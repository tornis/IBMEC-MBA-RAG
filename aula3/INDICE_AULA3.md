# Índice — Aula 3: Advanced RAG e Modular RAG
## Refinamento do Pipeline
### MBA em RAG & CAG Aplicados a Direito e Segurança Pública

**Aula:** 3 de 12 | **Carga:** 5h | **Proporção:** 30% teoria / 70% prática  
**Pré-requisito:** Aula 2 concluída (Naive RAG + Baseline registrado) | **Stack:** BGE-Reranker · LangChain · LangFuse · OpenSearch · ChromaDB · **Groq API** (`llama-3.1-8b-instant`) com **Ollama local** como fallback

---

## Estrutura de Arquivos

```
aula3/
│
├── INDICE_AULA3.md                                   ← Este arquivo
├── AVALIACAO_AULA3.md                                ← Rubricas e critérios (professor)
│
├── teoria/
│   └── AULA3_TEORIA.md                               ← Material teórico completo (7 seções)
│
├── labs/                                             ← 6 Laboratórios práticos (30% teoria / 70% prática)
│   ├── LAB1_Query_Rewriting.ipynb                   ← Query rewriting com Llama 3.1 — original vs reformulada
│   ├── LAB2_BGE_Reranker.ipynb                      ← Reranking com BGE-Reranker após retrieval OpenSearch
│   ├── LAB3_Analise_Qualitativa.ipynb               ← Impacto das melhorias: métricas + análise comparativa
│   ├── LAB4_Modular_RAG_Pipeline.ipynb              ← Pipeline Modular RAG com troca de retriever
│   ├── LAB5_LangFuse_Instrumentacao.ipynb           ← Instrumentação completa com traces e spans
│   └── LAB6_Analise_Trace_Otimizacao.ipynb          ← Análise do trace: módulo mais lento + proposta
│
├── exemplos/
│   ├── EXEMPLO1_Query_Rewriting.ipynb               ← Técnicas de reescrita de queries (referência rápida)
│   └── EXEMPLO2_BGE_Reranker.ipynb                  ← BGE-Reranker básico (referência rápida)
│
└── datasets/
    └── corpus_juridico_aula3.json                   ← 10 docs + 10 queries baseline (herda aula2 + expansão)
```

---

## Roteiro da Aula (5 horas)

| Bloco | Duração | Tipo | Conteúdo | Arquivo |
|---|---|---|---|---|
| **1. Revisão + Motivação** | 15 min | Teoria | Limitações do Naive RAG identificadas no baseline | `teoria/AULA3_TEORIA.md §1` |
| **2. Advanced RAG — Teoria** | 30 min | Teoria | Query rewriting, reranking, tri-layer pipeline | `teoria/AULA3_TEORIA.md §2–4` |
| **3. LAB 1 — Query Rewriting** | 40 min | Prática | 3 técnicas de reescrita, comparação query original vs reformulada | `labs/LAB1_Query_Rewriting.ipynb` |
| **4. LAB 2 — BGE-Reranker** | 40 min | Prática | Reranking top-K, tabela comparativa antes/depois | `labs/LAB2_BGE_Reranker.ipynb` |
| **5. LAB 3 — Análise de Impacto** | 30 min | Prática | Medir melhoria qualitativa, comparar com baseline aula 2 | `labs/LAB3_Analise_Qualitativa.ipynb` |
| **6. Modular RAG — Teoria** | 20 min | Teoria | Módulos intercambiáveis, arquitetura desacoplada | `teoria/AULA3_TEORIA.md §5` |
| **7. LAB 4 — Modular RAG** | 40 min | Prática | Trocar retriever OpenSearch ↔ ChromaDB sem alterar demais módulos | `labs/LAB4_Modular_RAG_Pipeline.ipynb` |
| **8. LangFuse — Teoria** | 15 min | Teoria | Traces, spans, generations — arquitetura de observabilidade | `teoria/AULA3_TEORIA.md §6` |
| **9. LAB 5 — Instrumentação** | 30 min | Prática | Adicionar trace completo com latência por módulo | `labs/LAB5_LangFuse_Instrumentacao.ipynb` |
| **10. LAB 6 — Otimização** | 20 min | Prática | Analisar trace, identificar gargalo, propor melhoria | `labs/LAB6_Analise_Trace_Otimizacao.ipynb` |

---

## Objetivos de Aprendizagem (conforme ementa)

Ao final desta aula, o aluno será capaz de:

1. **Identificar** os pontos de falha do Naive RAG em cenários jurídicos reais
2. **Implementar query rewriting** com 3 técnicas distintas (paraphrase, HyDE-lite, step-back)
3. **Aplicar reranking** com cross-encoder BGE-Reranker e comparar resultados top-5 antes/depois
4. **Arquitetar um pipeline Modular RAG** com módulos intercambiáveis no Google Colab
5. **Instrumentar o pipeline** com LangFuse e interpretar o primeiro trace completo com latência por módulo

---

## Stack Tecnológico

| Componente | Ferramenta | Papel no Pipeline |
|---|---|---|
| Query Rewriting | **`llama-3.1-8b-instant` via Groq API** | Reformulação de queries antes do retrieval |
| Reranking | **BGE-Reranker-v2-m3** (BAAI) | Cross-encoder para reordenar top-K resultados |
| Embeddings | **BGE-M3** (BAAI, dim=1024) — servido via Ollama | Vetorização multilíngue (herdado da Aula 2); fallback HuggingFace |
| Vector Store Principal | **OpenSearch kNN** | Retriever primário |
| Vector Store Alternativo | **ChromaDB** | Retriever intercambiável (demo Modular RAG) |
| LLM | **`llama-3.1-8b-instant`** (Groq) ou `llama3.2:3b` (Ollama local) | Geração de respostas jurídicas |
| Servidor LLM primário | **Groq Cloud API** (OpenAI-compatible, baixa latência) | LLM-as-a-service para todos os labs |
| Servidor LLM fallback | **Ollama local** em `http://localhost:11434/v1` | Provisionado na Aula 1; usado automaticamente se Groq indisponível |
| Orquestração | **LangChain LCEL** | Pipeline modular e composável |
| Observabilidade | **LangFuse** | Traces, spans, latência por módulo (provider e modelo propagados via metadata) |

---

## Fichas de Técnicas RAG — Esta Aula

### Ficha T02 — Advanced RAG

| Campo | Conteúdo |
|---|---|
| **ID** | #T02 |
| **Categoria** | RAG Básico |
| **Subtítulo** | Pré e pós-processamento avançado |
| **Descrição** | Adiciona camadas de refinamento antes (query rewriting/reformulation) e depois (reranking, context compression) do retrieval. Representa o padrão mais adotado em ambientes de produção. |
| **Aplicabilidades** | Assistentes jurídicos de alta precisão; análise de documentos financeiros; suporte técnico enterprise; pesquisa científica assistida |
| **Vantagens** | Melhoria significativa de relevância e faithfulness; suporte a queries complexas e ambíguas |
| **Limitações** | Maior latência (múltiplas etapas), custo operacional mais elevado |
| **Lab** | LAB1 (query rewriting) + LAB2 (reranking) + LAB3 (análise impacto) |
| **Referência** | Gao et al. (2023). arXiv:2312.10997. |

### Ficha T03 — Modular RAG

| Campo | Conteúdo |
|---|---|
| **ID** | #T03 |
| **Categoria** | RAG Básico |
| **Subtítulo** | Módulos intercambiáveis |
| **Descrição** | Decomposição do pipeline em módulos independentes: retriever, reranker, reader, memory. Cada módulo pode ser substituído sem afetar os demais, permitindo experimentação e customização ágil. |
| **Aplicabilidades** | Plataformas enterprise multi-tenant; sistemas customizáveis por domínio; MLOps de pipelines LLM; experimentação e pesquisa RAG |
| **Vantagens** | Flexibilidade arquitetural máxima, fácil manutenção, suporte à experimentação rápida |
| **Limitações** | Overhead de integração, complexidade de orquestração entre módulos |
| **Lab** | LAB4 (Modular RAG com troca de retriever) |
| **Referência** | Gao et al. (2023). Modular RAG. arXiv:2312.10997. |

---

## Avaliação

Ver `AVALIACAO_AULA3.md` para rubricas completas.

| Entregável | Peso | Lab |
|---|---|---|
| Pipeline Advanced RAG com query rewriting + reranking funcional | 40% | LAB1 + LAB2 |
| Análise comparativa: melhoria vs baseline Aula 2 | 25% | LAB3 |
| Pipeline Modular RAG com troca de retriever demonstrada | 20% | LAB4 |
| Trace completo no LangFuse com latência por módulo visível | 15% | LAB5 + LAB6 |

---

## Referências Bibliográficas (ABNT)

GAO, Y. et al. **Retrieval-Augmented Generation for Large Language Models: A Survey**. arXiv:2312.10997, 2023.

NOGUEIRA, R.; CHO, K. **Passage Re-ranking with BERT**. arXiv:1901.04085, 2019.

CHEN, J. et al. **BGE M3-Embedding: Multi-Lingual, Multi-Functionality, Multi-Granularity Text Embeddings Through Self-Knowledge Distillation**. arXiv:2309.07597, 2024.

LANGFUSE. **LangFuse Documentation**. Disponível em: <https://langfuse.com/docs>. Acesso em: abr. 2026.

GROQ INC. **Groq API Documentation — OpenAI-compatible LLM Inference**. Disponível em: <https://console.groq.com/docs>. Acesso em: maio 2026.

OLLAMA. **Ollama Documentation — Local LLM Serving with OpenAI-compatible API**. Disponível em: <https://github.com/ollama/ollama/blob/main/docs/openai.md>. Acesso em: maio 2026.

MA, X. et al. **Query Rewriting for Retrieval-Augmented Large Language Models**. arXiv:2305.14283, 2023.
