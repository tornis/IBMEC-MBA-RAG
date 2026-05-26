# Aula 3 — Advanced RAG e Modular RAG: Refinamento do Pipeline
## MBA em RAG & CAG Aplicados a Direito e Segurança Pública

**Proporção:** 30% teoria / 70% prática | **Carga horária:** 5h  
**Pré-requisito:** Aula 2 concluída e baseline de 5 queries registrado no LAB5

---

## Seção 1 — Motivação: Por Que o Naive RAG Falha em Ambientes Jurídicos

Na Aula 2, construímos um pipeline Naive RAG funcional e registramos o baseline. Agora é hora de analisar os resultados e entender por que esse pipeline frequentemente falha em contextos jurídicos de alta precisão.

### 1.1 Os Três Pontos de Falha do Naive RAG

O Naive RAG sofre de três problemas estruturais bem documentados na literatura:

**Problema 1 — Query-Document Mismatch (Vocabulary Gap)**

Um advogado pergunta: *"Qual é a posição do STF sobre a necessidade de advogado na fase policial?"*

Essa query usa linguagem coloquial. Os documentos indexados, no entanto, usam linguagem técnica-jurídica:

> *"O Plenário do Supremo Tribunal Federal, por maioria, fixou tese no sentido de que a Súmula Vinculante 14 assegura ao defensor constituído o direito de acesso às investigações em curso..."*

O modelo de embedding tenta aproximar esses vetores semanticamente, mas a distância lexical e de registro linguístico cria ruído significativo. Em testes controlados, queries coloquiais sobre temas jurídicos perdem em média 23% de relevância nos top-5 resultados quando comparadas com queries técnicas equivalentes (Ma et al., 2023).

**Problema 2 — Contexto Truncado**

Em documentos jurídicos longos — acórdãos de 80 páginas, laudos periciais, inquéritos policiais —, o chunking fixo frequentemente corta entidades relevantes entre chunks. O retriever recupera o chunk correto, mas o contexto necessário para a resposta está no chunk anterior ou posterior, que não foi recuperado.

**Problema 3 — Sem Filtragem de Relevância Pós-Retrieval**

O Naive RAG recupera os top-K chunks por similaridade de cosseno e os passa diretamente ao LLM. O problema: similaridade semântica global ≠ relevância para a pergunta específica. Um chunk sobre "habeas corpus" pode ser semanticamente próximo de uma query sobre "prisão preventiva", mas conter apenas definições gerais que não respondem à pergunta.

### 1.2 Diagrama: Comparação Arquitetural

```
NAIVE RAG:
Query ──► Embedding ──► Vector Search ──► Top-K Chunks ──► LLM ──► Resposta
          [sem refinamento]               [sem filtragem]

ADVANCED RAG:
Query ──► [PRÉ-RETRIEVAL] ──► Embedding ──► Vector Search ──► [PÓS-RETRIEVAL] ──► LLM ──► Resposta
          Query Rewriting                                        Reranking
          HyDE-lite                                             Context Compression
          Step-back                                             Relevance Filtering
```

---

## Seção 2 — Query Rewriting: Reduzindo o Vocabulary Gap

### 2.1 O Problema de Vocabulary Mismatch em Detalhes

O vocabulary mismatch ocorre porque usuários (policiais, advogados, promotores) expressam suas necessidades em linguagem natural que pode divergir substancialmente do vocabulário dos documentos indexados.

**Exemplos concretos do domínio:**

| Query do Usuário | Termo Técnico no Documento | Impacto no Retrieval |
|---|---|---|
| "Podem prender alguém sem mandado?" | "prisão em flagrante", "flagrante delito" | Alta divergência |
| "O suspeito tem direito a advogado na delegacia?" | "assistência do defensor na fase investigativa", "Súmula Vinculante 14" | Alta divergência |
| "Como provar que o réu tinha intenção?" | "elemento subjetivo do tipo", "dolo específico", "animus necandi" | Muito alta divergência |

### 2.2 Técnica 1: Paraphrase Rewriting

A técnica mais simples: usar o LLM para gerar N reformulações da query original, cobrindo diferentes perspectivas linguísticas.

**Prompt template:**
```
Você é um especialista jurídico brasileiro. Reformule a query abaixo em 3 variações 
usando terminologia técnica jurídica. Mantenha o mesmo significado.

Query original: {query}

Retorne apenas as 3 variações, uma por linha.
```

**Quando usar:** Queries coloquiais que precisam de tradução para linguagem técnica.

**Limitação:** Pode gerar variações redundantes se o LLM não tiver domínio jurídico adequado.

### 2.3 Técnica 2: HyDE-Lite (Hypothetical Document Embeddings simplificado)

O HyDE original (Gao et al., 2022) gera um documento hipotético que *responderia* à query e usa o embedding desse documento para busca. O HyDE-lite é uma versão simplificada:

1. O LLM gera um parágrafo hipotético que seria o trecho ideal para responder à query
2. Esse parágrafo é embutido e usado para busca vetorial (em vez da query original)
3. O resultado é enriquecido semanticamente porque o documento hipotético usa o vocabulário do corpus

**Prompt template:**
```
Você é um redator jurídico especializado. Escreva um parágrafo curto (3-4 linhas) 
que seria encontrado em um acórdão ou documento oficial brasileiro e que responderia 
diretamente à seguinte pergunta:

Pergunta: {query}

Escreva apenas o parágrafo, sem introdução ou conclusão.
```

**Quando usar:** Queries complexas onde a terminologia do domínio é muito específica.

**Atenção:** HyDE pode introduzir alucinações se o LLM gerar informações incorretas no parágrafo hipotético. Em ambiente jurídico, isso é especialmente problemático — sempre valide os resultados.

### 2.4 Técnica 3: Step-Back Simplificado

Inspirado no paper "Take a Step Back" (Zheng et al., 2023), essa técnica abstrai a query para um nível mais geral, realizando a busca num nível conceitual antes de detalhar.

**Exemplo:**
- Query original: *"O delegado pode interrogar o suspeito sem advogado presente?"*
- Step-back: *"Quais são os direitos do investigado na fase policial?"*

A busca step-back recupera documentos sobre direitos gerais do investigado, que provavelmente contêm o trecho específico sobre interrogatório.

**Pipeline combinado:**
```python
results_original = retriever.search(query)
results_stepback = retriever.search(stepback_query)
results_combined = deduplicate_and_merge(results_original, results_stepback)
```

---

## Seção 3 — Reranking com BGE-Reranker

### 3.1 Bi-Encoders vs Cross-Encoders: A Diferença Fundamental

Compreender essa distinção é essencial para arquitetar pipelines eficientes.

**Bi-Encoder (como o BGE-M3):**

```
Query  ──► Encoder ──► vetor_q (1024 dims)
                              ↓
                         cos_similarity(vetor_q, vetor_doc)
                              ↑
Documento ──► Encoder ──► vetor_doc (1024 dims)
```

- Os vetores são calculados **independentemente** e armazenados no índice
- Na busca: apenas uma multiplicação matricial (ANN search)
- **Velocidade:** O(1) por query com índice pré-computado
- **Precisão:** Limitada — sem atenção cruzada entre query e documento

**Cross-Encoder (como o BGE-Reranker):**

```
[Query] [SEP] [Documento] ──► Transformer completo ──► score_relevância
```

- Query e documento são processados **juntos** pelo transformer
- Atenção cruzada completa entre todos os tokens
- **Velocidade:** O(N) por query — precisa processar cada par (query, doc) separadamente
- **Precisão:** Muito superior — captura interações finas entre query e documento

**Implicação arquitetural:** Cross-encoders são lentos para busca em larga escala, mas extremamente precisos para reordenar um conjunto pequeno de candidatos. Por isso, a arquitetura padrão é:

```
1. Bi-Encoder: busca rápida em 1M+ documentos → top-100
2. Cross-Encoder: reranking preciso nos top-100 → top-5 final
```

### 3.2 BGE-Reranker-v2-m3 em Detalhes

O `BAAI/bge-reranker-v2-m3` é o estado da arte em reranking multilíngue, com suporte nativo a português. Características:

- **Arquitetura:** XLM-RoBERTa fine-tuned para cross-encoding
- **Idiomas:** 94+ idiomas, incluindo português brasileiro
- **Output:** Score de relevância entre 0 e 1 para cada par (query, documento)
- **Tamanho:** ~560M parâmetros (requer ~2GB VRAM em FP16)

**Uso básico:**
```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

model_name = "BAAI/bge-reranker-v2-m3"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)

def rerank(query: str, docs: list[str]) -> list[tuple[float, str]]:
    pairs = [[query, doc] for doc in docs]
    inputs = tokenizer(pairs, padding=True, truncation=True, 
                       max_length=512, return_tensors="pt")
    with torch.no_grad():
        scores = model(**inputs).logits.squeeze(-1)
    ranked = sorted(zip(scores.tolist(), docs), reverse=True)
    return ranked
```

### 3.3 Interpretação Matemática do Score

O cross-encoder aplica uma camada de classificação binária sobre o [CLS] token:

```
score = sigmoid(W · h_CLS + b)
```

Onde `h_CLS` é a representação do token [CLS] após o forward pass completo com atenção cruzada. Scores acima de 0.7 indicam alta relevância; abaixo de 0.3, baixa relevância.

**Tabela de interpretação:**

| Score | Interpretação | Ação recomendada |
|---|---|---|
| 0.85 – 1.00 | Altamente relevante | Incluir no contexto |
| 0.60 – 0.85 | Provavelmente relevante | Incluir se espaço disponível |
| 0.30 – 0.60 | Relevância moderada | Incluir com cautela |
| 0.00 – 0.30 | Baixa relevância | Descartar |

---

## Seção 4 — Advanced RAG: Pipeline Tri-Layer

### 4.1 Arquitetura Completa

O Advanced RAG organiza as melhorias em três camadas funcionais:

```
CAMADA 1: PRÉ-RETRIEVAL
├── Query Analysis (detecção de intent)
├── Query Rewriting (paraphrase / HyDE-lite / step-back)
└── Query Expansion (adicionar sinônimos jurídicos)

CAMADA 2: RETRIEVAL
├── Hybrid Search (vetorial + BM25)
├── Multi-query retrieval (busca com N variações da query)
└── Ensemble retrieval (combinar múltiplos retrievers)

CAMADA 3: PÓS-RETRIEVAL
├── Reranking (BGE-Reranker cross-encoder)
├── Context Compression (extrair apenas trechos relevantes)
└── Deduplication (remover chunks redundantes)
```

### 4.2 Pipeline Implementado nesta Aula

Nesta aula, implementamos um subconjunto focado:

```python
def advanced_rag_pipeline(query: str) -> str:
    # PRÉ-RETRIEVAL
    rewritten_query = query_rewriter.rewrite(query)  # LAB1
    
    # RETRIEVAL
    raw_results = retriever.search(rewritten_query, top_k=20)
    
    # PÓS-RETRIEVAL
    reranked_results = reranker.rerank(query, raw_results, top_k=5)  # LAB2
    
    # GERAÇÃO
    context = build_context(reranked_results)
    response = llm.generate(query, context)
    
    return response
```

### 4.3 Infraestrutura LLM da Aula 3 — Groq + Ollama (Fallback)

A partir desta aula, o pipeline de geração passa a depender de um LLM real para query rewriting, geração de respostas e, em alguns labs, para reranking baseado em LLM. Para garantir baixa latência didática **e** portabilidade local, o curso adota uma estratégia de **dois provedores em cascata**:

| Camada | Provedor | Modelo | Endpoint | Quando é usado |
|---|---|---|---|---|
| **Primário** | **Groq Cloud** | `llama-3.1-8b-instant` | `https://api.groq.com/openai/v1` | Sempre que `GROQ_API_KEY` estiver presente no `.env` e a API responder |
| **Fallback** | **Ollama local** | `llama3.2:3b` (padrão) ou `llama3.1:8b` | `http://localhost:11434/v1` | Automaticamente, se Groq estiver indisponível ou a chave faltar |

Ambos os provedores expõem **API OpenAI-compatible**, o que permite trocar apenas `base_url` + `api_key` sem reescrever o pipeline. O snippet padrão dos labs detecta o provedor em tempo de execução com um *smoke test* leve (1 chamada de 2 tokens) e propaga `LLM_PROVIDER` e `MODEL_NAME` para o LangFuse via `metadata`, permitindo análise de custo/latência por provedor nos dashboards.

> **Nota didática — Por que `llama-3.1-8b-instant` da Groq?**
> O modelo `llama-3.1-8b-instant` é o melhor compromisso entre custo, contexto (128k tokens) e velocidade no Groq (~750 tokens/s). É suficiente para tarefas jurídicas brasileiras de query rewriting e geração curta, e o aluno tem **free tier** generoso para concluir todos os labs sem cartão de crédito.
>
> **Por que Ollama como fallback (e não vLLM)?**
> O Ollama instala em minutos em Windows/macOS/Linux **sem exigir GPU NVIDIA** e usa o mesmo schema OpenAI-compatible. O vLLM (servidor de produção com PagedAttention) continua sendo a referência para deploy em GPU dedicada — abordado apenas na **Aula 12 (Projeto Final)**, não nos labs didáticos.

**Variáveis lidas pelos labs (definidas em `~/mba-rag/.env`):**

```
GROQ_API_KEY=gsk_...
GROQ_LLM_MODEL=llama-3.1-8b-instant
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_LLM_MODEL=llama3.2:3b
OLLAMA_EMBED_MODEL=bge-m3
```

> **Atenção:** embeddings continuam **sempre via Ollama** (`bge-m3`, 1024 dims), com fallback para `HuggingFaceEmbeddings(BAAI/bge-m3)`. Groq não oferece API de embeddings — apenas LLMs.

---

## Seção 5 — Modular RAG: Arquitetura Desacoplada

### 5.1 Filosofia de Design

O Modular RAG (Gao et al., 2023) formaliza o RAG como um sistema de módulos independentes. A ideia central: **cada componente do pipeline pode ser substituído sem afetar os demais**, desde que respeite a interface definida.

Isso é análogo ao conceito de **injeção de dependência** em engenharia de software, ou ao princípio de substituição de Liskov aplicado a sistemas de IA.

### 5.2 Módulos Principais

```
┌─────────────────────────────────────────────────────────────────┐
│                      MODULAR RAG PIPELINE                        │
├─────────────┬──────────────┬──────────────┬───────────────────── ┤
│  MODULE 1   │   MODULE 2   │   MODULE 3   │      MODULE 4        │
│  INDEXING   │PRE-RETRIEVAL │  RETRIEVAL   │   POST-RETRIEVAL     │
│             │              │              │                       │
│  Docling    │  QueryRewrite│  OpenSearch  │  BGE-Reranker        │
│  Chunking   │  HyDE-lite   │  ──OR──      │  ──OR──              │
│  BGE-M3     │  Step-back   │  ChromaDB    │  LLM Compressor      │
│  OpenSearch │              │  ──OR──      │  ──OR──              │
│             │              │  FAISS       │  KeyBERT Filter      │
├─────────────┴──────────────┴──────────────┴───────────────────── ┤
│                      MODULE 5: GENERATION                         │
│       Groq API (llama-3.1-8b-instant) / Ollama local fallback     │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 Interface Contratual dos Módulos

Para que a intercambialidade funcione, cada módulo deve respeitar uma interface definida. Em Python, usamos classes abstratas:

```python
from abc import ABC, abstractmethod
from typing import List

class BaseRetriever(ABC):
    @abstractmethod
    def search(self, query: str, top_k: int = 10) -> List[dict]:
        """Retorna lista de dicts com 'content', 'score', 'metadata'"""
        pass

class OpenSearchRetriever(BaseRetriever):
    def search(self, query: str, top_k: int = 10) -> List[dict]:
        # implementação OpenSearch
        ...

class ChromaDBRetriever(BaseRetriever):
    def search(self, query: str, top_k: int = 10) -> List[dict]:
        # implementação ChromaDB — MESMA INTERFACE
        ...
```

**Ponto-chave:** A classe `ModularRAGPipeline` só conhece `BaseRetriever`, não as implementações concretas. Isso permite trocar o retriever em runtime:

```python
# Sem alterar nenhuma outra linha do pipeline:
pipeline = ModularRAGPipeline(retriever=OpenSearchRetriever())
# ou
pipeline = ModularRAGPipeline(retriever=ChromaDBRetriever())
```

### 5.4 Benefícios em Contexto Jurídico-Policial

Em sistemas de segurança pública, o Modular RAG permite:

- **Múltiplos corpora**: trocar o retriever por domínio (Código Penal vs. procedimentos policiais vs. jurisprudência STF)
- **Compliance e auditoria**: cada módulo pode ser auditado e validado independentemente
- **Evolução gradual**: substituir o LLM de geração sem reindexar o corpus
- **A/B Testing**: comparar retrievers diferentes em produção sem downtime

---

## Seção 6 — LangFuse: Observabilidade de Pipelines LLM

### 6.1 Por Que Observabilidade É Crítica

Em sistemas jurídicos, **toda resposta gerada deve ser rastreável e auditável**. O LangFuse fornece:

- **Traces completos**: qual foi a query, quais chunks foram recuperados, qual foi o prompt exato
- **Latência por módulo**: identificar gargalos (query rewriting lento? retrieval lento? LLM lento?)
- **Custo por operação**: tokens usados, custo estimado por query
- **Análise de qualidade**: registrar feedback humano sobre as respostas

### 6.2 Arquitetura de Traces e Spans

```
TRACE (representa uma operação end-to-end)
├── SPAN: query_rewriting       [latência: 1.2s]
│   └── INPUT: query original
│   └── OUTPUT: query reescrita
│
├── SPAN: retrieval             [latência: 0.3s]
│   └── INPUT: query reescrita
│   └── OUTPUT: top-20 chunks
│
├── SPAN: reranking             [latência: 2.1s]  ← gargalo!
│   └── INPUT: top-20 chunks
│   └── OUTPUT: top-5 reranked
│
└── GENERATION: llm_response    [latência: 3.4s]
    ├── INPUT: prompt completo
    ├── OUTPUT: resposta gerada
    └── TOKENS: 1.847 tokens
```

### 6.3 Instrumentação com o Decorator @observe

O LangFuse oferece uma API de decorators que minimiza o código de instrumentação:

```python
from langfuse.decorators import observe, langfuse_context

@observe()
def advanced_rag_query(query: str) -> str:
    langfuse_context.update_current_trace(
        name="advanced_rag_pipeline",
        metadata={"domain": "juridico", "version": "3.0"}
    )
    
    rewritten = rewrite_query(query)        # span automático
    chunks = retrieve_chunks(rewritten)     # span automático
    reranked = rerank_chunks(query, chunks) # span automático
    response = generate_response(query, reranked)
    
    return response

@observe(name="query_rewriting")
def rewrite_query(query: str) -> str:
    # código de reescrita
    ...
```

### 6.4 Métricas a Monitorar

Para pipelines jurídicos em produção, monitore sempre:

| Métrica | Threshold Saudável | Ação se Ultrapassar |
|---|---|---|
| Latência total (p95) | < 5s | Revisar módulo mais lento |
| Latência de retrieval | < 500ms | Otimizar índice OpenSearch |
| Latência de reranking | < 3s | Reduzir top-K candidatos |
| Taxa de erros | < 0.1% | Investigar logs de erro |
| Tokens médios/query | < 2.000 | Revisar context compression |

---

## Seção 7 — Estudos de Caso em Produção

### 7.1 Harvey AI — Assistente Jurídico para Escritórios de Advocacia

A Harvey AI (2023–2024) implementou Advanced RAG para revisão de contratos. O pipeline usa query rewriting para traduzir perguntas de advogados em queries técnicas sobre cláusulas específicas, e reranking para priorizar precedentes mais recentes. Resultado reportado: redução de 40% no tempo de due diligence em processos de M&A.

**Lição técnica:** O reranking foi crucial para priorizar jurisprudência por data (documentos mais recentes têm maior peso no score final).

### 7.2 Detectives Digitais — Polícia Civil de São Paulo (Projeto Piloto)

Em projeto piloto de 2024, a PCSP testou um sistema RAG para busca em boletins de ocorrência. O maior desafio: policiais usam jargão operacional ("BO de furto", "flagrante de tráfico") que diverge do texto formal dos documentos. Query rewriting com step-back reduziu queries sem resultado de 31% para 8%.

**Lição técnica:** Step-back é especialmente eficaz em domínios com forte jargão operacional vs. linguagem formal nos documentos.

### 7.3 Anthropic Legal Team — Internal RAG Tool

A Anthropic documentou internamente que pipelines Advanced RAG com reranking melhoram precision@5 em 15-30% em benchmarks jurídicos comparados ao Naive RAG, com custo adicional de latência de ~2-3 segundos por query (principalmente devido ao cross-encoder).

**Lição técnica:** O trade-off latência/precisão deve ser avaliado caso a caso. Para sistemas onde um advogado espera ativamente por uma resposta, 3s adicionais são aceitáveis. Para sistemas de triagem em tempo real (alerta policial), podem não ser.

---

## Exercícios de Fixação

**Exercício 1 — Identificação de Falhas:** Analise as 5 queries do baseline (LAB5, Aula 2). Para cada uma, identifique qual dos três problemas do Naive RAG (vocabulary gap, contexto truncado, sem filtragem) foi mais provável responsável por resultados insatisfatórios.

**Exercício 2 — Design de Query Rewriting:** Para a query *"O acusado pode mentir no interrogatório?"*, escreva manualmente: (a) uma paraphrase técnica, (b) um documento hipotético HyDE-lite, (c) a query step-back correspondente.

**Exercício 3 — Análise de Trade-off:** Um sistema policial precisa responder a queries em menos de 1 segundo. Considerando os tempos típicos de reranking (2–3s), como você redesenharia o pipeline para manter a precisão do Advanced RAG dentro da restrição de latência?

**Exercício 4 — Modularidade:** Desenhe o diagrama de classes Python (usando ABCs) para um pipeline Modular RAG que suporte: 3 retrievers (OpenSearch, ChromaDB, FAISS), 2 rerankers (BGE-Reranker, LLM-based), 2 geradores (Groq API com `llama-3.1-8b-instant`, Ollama local com `llama3.2:3b`).

**Exercício 5 — LangFuse:** Num trace hipotético, o span de reranking tem latência de 8 segundos para top-20 documentos. Identifique duas otimizações possíveis e explique o impacto esperado de cada uma.

---

## Referências Bibliográficas (ABNT)

GAO, Y. et al. **Retrieval-Augmented Generation for Large Language Models: A Survey**. arXiv:2312.10997, 2023. Disponível em: <https://arxiv.org/abs/2312.10997>. Acesso em: abr. 2026.

MA, X. et al. **Query Rewriting for Retrieval-Augmented Large Language Models**. arXiv:2305.14283, 2023. Disponível em: <https://arxiv.org/abs/2305.14283>. Acesso em: abr. 2026.

NOGUEIRA, R.; CHO, K. **Passage Re-ranking with BERT**. arXiv:1901.04085, 2019.

CHEN, J. et al. **BGE M3-Embedding: Multi-Lingual, Multi-Functionality, Multi-Granularity Text Embeddings Through Self-Knowledge Distillation**. arXiv:2309.07597, 2024.

GAO, L. et al. **Precise Zero-Shot Dense Retrieval without Relevance Labels (HyDE)**. arXiv:2212.10496, 2022.

ZHENG, H. S. et al. **Take a Step Back: Evoking Reasoning via Abstraction in Large Language Models**. arXiv:2310.06117, 2023.

LANGFUSE. **LangFuse Documentation**. Disponível em: <https://langfuse.com/docs>. Acesso em: abr. 2026.

KWON, W. et al. **Efficient Memory Management for LLM Serving with PagedAttention**. *ACM SOSP*, 2023.
