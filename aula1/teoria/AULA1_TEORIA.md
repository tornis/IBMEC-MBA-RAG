# CURSO DE MBA: RAG & CAG APLICADOS A DIREITO E SEGURANÇA PÚBLICA
## Aula 1 — Fundamentos: NLP, Embeddings, LLMs e Setup do Ambiente
### Carga Horária: 5 horas | Proporção: 35% Teoria / 65% Prática

---

> **ABNT NBR 6023:2018** — Material didático de uso exclusivo do curso. Todos os conceitos são baseados nas referências listadas na Seção 8.

---

## SUMÁRIO

1. [Ementa e Objetivos](#1-ementa-e-objetivos)
2. [Tópico 1 — NLP: Fundamentos e Representação de Texto](#2-tópico-1--nlp-fundamentos-e-representação-de-texto)
3. [Tópico 2 — Embeddings Estáticos: Word2Vec, GloVe e fastText](#3-tópico-2--embeddings-estáticos-word2vec-glove-e-fasttext)
4. [Tópico 3 — Embeddings Contextuais: BERT, Sentence-Transformers e BGE-M3](#4-tópico-3--embeddings-contextuais-bert-sentence-transformers-e-bge-m3)
5. [Tópico 4 — Arquitetura Transformer: Atenção, Encoder-Decoder e Positional Encoding](#5-tópico-4--arquitetura-transformer-atenção-encoder-decoder-e-positional-encoding)
6. [Tópico 5 — Large Language Models: GPT, Llama, Mistral e Parâmetros de Geração](#6-tópico-5--large-language-models-gpt-llama-mistral-e-parâmetros-de-geração)
7. [Tópico 6 — O que é RAG? Por que surgiu e onde LLMs falham](#7-tópico-6--o-que-é-rag-por-que-surgiu-e-onde-llms-falham)
8. [Tópico 7 — Panorama das 25 Técnicas RAG do Curso](#8-tópico-7--panorama-das-25-técnicas-rag-do-curso)
9. [Exercícios de Fixação com Resoluções](#9-exercícios-de-fixação-com-resoluções)
10. [Referências Expandidas](#10-referências-expandidas)

---

## 1. EMENTA E OBJETIVOS

### 1.1 Ementa

Esta aula inaugura o curso com a construção sólida dos alicerces conceituais e práticos necessários para compreender e implementar sistemas RAG (Retrieval-Augmented Generation) e CAG (Cache-Augmented Generation) no contexto do Direito e da Segurança Pública. Partindo do zero em NLP, o aluno percorrerá a evolução histórica das representações de texto — desde contagens simples de palavras até modelos de linguagem com bilhões de parâmetros — compreendendo *por que* cada avanço foi necessário e *o que* ele resolve na prática.

### 1.2 Objetivos de Aprendizagem

Ao final desta aula, o aluno será capaz de:

- Compreender a evolução de modelos de linguagem: bag-of-words → Word2Vec → Transformers → LLMs
- Entender representações vetoriais de texto e métricas de similaridade semântica
- Configurar e validar o ambiente completo: OpenSearch, Ollama, LangFuse
- Executar o primeiro pipeline de embeddings e comparar modelos BGE-M3 vs sentence-transformers
- Compreender o problema que RAG resolve e visualizar o panorama das 25 técnicas do curso

### 1.3 Roteiro de 5 Horas

| Bloco | Conteúdo | Tempo |
|-------|----------|-------|
| Bloco 1 (Teoria) | Tópicos 1-4: NLP → Transformers | 45 min |
| Lab 1 + Lab 2 | Instalação Docker/OpenSearch + Setup Ambiente Local | 50 min |
| Bloco 2 (Teoria) | Tópicos 5-7: LLMs → RAG → Panorama | 30 min |
| Lab 3 + Lab 4 | Ollama + LangFuse | 45 min |
| Lab 5 | Embeddings + UMAP + Discussão | 60 min |
| Exercícios | Fixação e revisão | 30 min |

---

## 2. TÓPICO 1 — NLP: FUNDAMENTOS E REPRESENTAÇÃO DE TEXTO

### 2.1 Explicação Teórica

**O problema fundamental: computadores não entendem palavras**

Para um ser humano, a frase *"O réu foi absolvido por falta de provas"* é imediatamente compreensível. Para um computador, ela é apenas uma sequência de caracteres ASCII. O desafio central do Processamento de Linguagem Natural (NLP, do inglês *Natural Language Processing*) é criar pontes entre a linguagem humana — ambígua, contextual e repleta de nuances — e as operações matemáticas que os computadores executam com eficiência. Cada técnica que estudaremos neste curso é, em essência, uma resposta mais sofisticada a essa pergunta: *como representar texto de forma que o computador possa "entendê-lo"?*

Imagine que você é bibliotecário e precisa organizar 50.000 processos judiciais digitalizados de forma que qualquer advogado possa buscar casos relevantes em segundos. A abordagem mais ingênua seria criar um índice simples de palavras-chave: se alguém busca "homicídio culposo", você lista todos os documentos que contêm essas duas palavras. Isso é, essencialmente, o que as primeiras técnicas de NLP faziam. A **tokenização** é o processo de dividir o texto em unidades menores chamadas tokens — que podem ser palavras, subpalavras ou caracteres. O **stemming** reduz palavras à sua raiz morfológica: "absolvido", "absolvição" e "absolver" viram todas "absolv-", facilitando agrupamentos semânticos.

O modelo **Bag-of-Words (BoW)** representa um documento como um vetor de contagens de palavras, ignorando completamente a ordem em que aparecem. A frase "o juiz absolveu o réu" e "o réu absolveu o juiz" teriam representações idênticas em BoW — o que, do ponto de vista jurídico, é um problema gravíssimo. O aperfeiçoamento natural é o **TF-IDF** (Term Frequency-Inverse Document Frequency), que pondera cada palavra pela sua frequência no documento dividida pela sua frequência em todo o corpus. Palavras muito comuns como "o", "de", "que" recebem peso baixo, enquanto termos técnicos raros como "habeas corpus" ou "peculato" recebem peso alto. Isso melhora muito as buscas, mas ainda não captura semântica: "automóvel" e "veículo" são palavras diferentes para o TF-IDF, mesmo sendo quase sinônimos.

**Limitações críticas para o domínio jurídico:** No contexto de segurança pública e direito, as limitações do BoW e TF-IDF são particularmente problemáticas. Um sistema baseado em TF-IDF não consegue relacionar "arma branca" (faca) com "instrumento perfurocortante"; não entende que "flagrante" e "em flagrante delito" são conceitos relacionados; e, mais criticamente, não consegue inferir que um documento sobre "latrocínio" é relevante para uma busca sobre "roubo seguido de morte". Essas limitações foram o motor que impulsionou o desenvolvimento das técnicas de embeddings que veremos nos próximos tópicos.

---

> ### 💡 INSIGHT — A Falácia da Simplicidade
> TF-IDF ainda é amplamente usado em sistemas de busca em produção porque é **rápido, determinístico e explicável**. Em cenários jurídicos, onde a auditabilidade do resultado é obrigatória, saber exatamente *por que* um documento foi retornado tem valor. RAG híbrido (que veremos na Aula 8) combina TF-IDF com embeddings, aproveitando o melhor dos dois mundos.

---

> ### 🛠️ ATIVIDADE PRÁTICA
> Para uma demonstração interativa de como esses mecanismos funcionam na prática com a língua portuguesa, acesse o notebook:
> [tokenizacao_stemming_ptbr.ipynb](../scripts/tokenizacao_stemming_ptbr.ipynb)

---

### 2.2 Perguntas para Reflexão

**Pergunta 1:** Por que a tokenização por subpalavra (usada em modelos modernos como BERT e GPT) é superior à tokenização por palavra inteira para termos jurídicos incomuns?

> **Resposta:** Termos jurídicos raros como "desapropriação", "arrematação" ou "extorsão" podem nunca ter sido vistos no corpus de treinamento se tokenizarmos por palavra inteira — resultando em um token `[UNK]` (desconhecido). A tokenização por subpalavra (BPE — Byte Pair Encoding, usado no GPT; WordPiece, usado no BERT) divide "desapropriação" em unidades menores como "des", "apro", "priação" que, embora não sejam palavras do dicionário, preservam informação morfológica. O modelo pode assim fazer inferências razoáveis sobre termos nunca vistos, o que é essencial em domínios técnicos com vocabulário especializado extenso.

**Pergunta 2:** Um delegado propõe indexar todos os boletins de ocorrência usando apenas TF-IDF. Quais são os três principais riscos dessa abordagem para a qualidade das buscas?

> **Resposta:** (1) **Sinonímia não resolvida:** "veículo automotor", "carro" e "automóvel" seriam tratados como termos completamente diferentes, resultando em recall baixo nas buscas. (2) **Polissemia ignorada:** A palavra "banco" em "assalto a banco" vs "sentado no banco dos réus" teria o mesmo peso, gerando resultados irrelevantes (baixa precisão). (3) **Compostos jurídicos não capturados:** Expressões como "tráfico de influência" têm significado completamente diferente de "tráfico" + "influência" separados, mas TF-IDF trata cada palavra individualmente.

---

## 3. TÓPICO 2 — EMBEDDINGS ESTÁTICOS: WORD2VEC, GLOVE E FASTTEXT

### 3.1 Explicação Teórica

**Da contagem à geometria semântica**

A grande virada conceitual nos anos 2010 foi perceber que palavras poderiam ser representadas não como vetores esparsos de contagens (com milhares de dimensões, a maioria zeros), mas como vetores densos em espaços de baixa dimensionalidade (tipicamente 100 a 300 dimensões) onde a **proximidade geométrica codifica similaridade semântica**. Essa ideia, materializada no algoritmo **Word2Vec** publicado por Mikolov et al. em 2013, transformou fundamentalmente o campo do NLP. A intuição central é elegante: *"uma palavra é conhecida pela companhia que mantém"* (Firth, 1957). Se "juiz" frequentemente aparece perto de "sentença", "advogado", "réu" e "audiência", então essas palavras devem ter representações vetoriais próximas entre si.

O Word2Vec opera em dois modos: **CBOW** (Continuous Bag of Words), que prediz uma palavra a partir de seu contexto, e **Skip-gram**, que prediz o contexto a partir de uma palavra. Em ambos os casos, a tarefa de previsão é apenas um pretexto — o que realmente interessa são os pesos da rede neural que aprende a fazer essa previsão. Esses pesos se tornam os embeddings das palavras. A magia desse processo é que propriedades semânticas emergem naturalmente: o vetor de "rei" menos o vetor de "homem" mais o vetor de "mulher" resulta em um vetor muito próximo de "rainha". Aplicado ao domínio jurídico: vetor("delegado") - vetor("investigação") + vetor("julgamento") ≈ vetor("juiz").

O **GloVe** (Global Vectors for Word Representation, Pennington et al., 2014) complementa Word2Vec utilizando estatísticas globais de co-ocorrência de palavras no corpus, ao invés de apenas janelas locais de contexto. Isso permite que GloVe capture tanto padrões locais (relações sintáticas entre palavras adjacentes) quanto globais (relações temáticas entre palavras que raramente aparecem juntas mas compartilham contextos similares). O **fastText** (Bojanowski et al., 2017) foi além ao representar palavras como soma de embeddings de seus n-gramas de caracteres — o que permite gerar embeddings para palavras fora do vocabulário de treinamento, benefício enorme para termos técnicos novos ou palavras com erros ortográficos comuns em boletins de ocorrência.

**A limitação que mudou tudo:** Apesar de revolucionários, os embeddings estáticos têm um problema fundamental: cada palavra tem apenas *um* vetor, independente do contexto. A palavra "banco" tem o mesmo embedding em "assaltou o banco" e "sentado no banco dos réus". Para um sistema de busca jurídico, isso é crítico: uma consulta sobre "banco" poderia retornar tanto casos de crimes financeiros quanto de audiências judiciais. Essa limitação contextual foi o principal motivador para o desenvolvimento dos modelos Transformer e dos embeddings contextuais, que abordaremos no próximo tópico.

---

> ### 💡 INSIGHT — Visualizando Semântica
> Embeddings podem ser visualizados em 2D usando técnicas de redução de dimensionalidade como UMAP ou t-SNE. Em um corpus jurídico treinado corretamente, você esperaria ver clusters distintos para crimes patrimoniais, crimes contra a vida, crimes de trânsito e crimes digitais — com palavras semanticamente relacionadas agrupadas próximas. Essa visualização é exatamente o que faremos no Lab 5 desta aula.

---

### 3.2 Perguntas para Reflexão

**Pergunta 1:** Por que fastText é especialmente vantajoso para processar boletins de ocorrência escritos por policiais no campo, comparado ao Word2Vec tradicional?

> **Resposta:** Boletins de ocorrência frequentemente contêm erros tipográficos, abreviações e variações ortográficas: "homicidio" (sem acento), "vnculo" (faltando letra), "BDO" (abreviação). O fastText, ao representar palavras por seus n-gramas de caracteres, consegue gerar embeddings razoáveis para essas formas irregulares, pois "homicidio" e "homicídio" compartilham a maioria dos n-gramas. O Word2Vec, por trabalhar com palavras inteiras, retornaria `[UNK]` para qualquer forma não vista no treinamento.

**Pergunta 2:** Se você treinasse um modelo Word2Vec em um corpus de 1 milhão de acórdãos do STJ, qual seria a hipótese sobre a proximidade vetorial entre os termos "nulidade", "cerceamento de defesa" e "devido processo legal"?

> **Resposta:** A hipótese seria que esses três termos teriam representações vetoriais próximas no espaço de embeddings, pois frequentemente co-ocorrem nos mesmos contextos jurídicos (petições que alegam nulidade geralmente mencionam cerceamento de defesa como fundamento, e ambos invocam o princípio do devido processo legal como base constitucional). A distância coseno entre os vetores deveria ser menor que 0.3 (alta similaridade), enquanto a distância entre "nulidade" e "furto" deveria ser maior que 0.7 (baixa similaridade semântica). Isso poderia ser testado empiricamente com um corpus adequado.

---

## 4. TÓPICO 3 — EMBEDDINGS CONTEXTUAIS: BERT, SENTENCE-TRANSFORMERS E BGE-M3

### 4.1 Explicação Teórica

**O contexto muda tudo**

Em 2018, o Google publicou o BERT (Bidirectional Encoder Representations from Transformers, Devlin et al.), que representou uma ruptura fundamental: ao invés de atribuir um vetor fixo a cada palavra, o BERT gera representações diferentes para a mesma palavra dependendo de todo o seu contexto sentencial. A palavra "banco" em "assaltou o banco comercial" recebe um vetor completamente diferente de "banco" em "sentado no banco dos réus" — porque as outras palavras da frase influenciam dinamicamente o processo de representação. Isso é possível graças ao mecanismo de **atenção** (que exploraremos em profundidade no próximo tópico), que permite ao modelo "olhar" para todas as outras palavras da frase simultaneamente ao codificar cada token.

O BERT foi pré-treinado em duas tarefas: **Masked Language Modeling (MLM)**, onde o modelo precisa prever palavras mascaradas numa frase (ex.: "O réu foi [MASK] por falta de provas" → "absolvido"), e **Next Sentence Prediction (NSP)**, onde o modelo aprende se duas frases são consecutivas em um texto. Esse pré-treinamento massivo em bilhões de palavras (Wikipedia + BookCorpus) confere ao BERT um conhecimento profundo da estrutura da língua portuguesa (para modelos como BERTimbau, treinado em português) antes mesmo de ser ajustado para qualquer tarefa específica. O *fine-tuning* para classificação de sentenças, extração de informações jurídicas ou similaridade semântica é muito mais eficiente sobre esse fundamento.

Os **Sentence-Transformers** (Reimers & Gurevych, 2019) surgem de uma necessidade prática: o BERT original não foi projetado para gerar embeddings de frases completas — ele gera tokens individuais. Para comparar a similaridade entre dois documentos jurídicos usando BERT, seria necessário passar *ambos* os documentos concatenados, o que é computacionalmente impraticável para um corpus de 50.000 processos. Sentence-Transformers resolve isso com uma arquitetura *siamesa* de dois encoders BERT com pesos compartilhados, treinada para que frases semanticamente similares produzam embeddings próximos. O resultado é um encoder capaz de gerar um único vetor representativo para qualquer frase, permitindo comparações eficientes em escala.

O **BGE-M3** (BAAI General Embedding, Multi-lingual Multi-functionality Multi-granularity, Xiao et al., 2024) representa o estado da arte em 2024. Ele combina três funcionalidades em um único modelo: (1) **busca densa**, gerando embeddings de alta qualidade para similaridade semântica; (2) **busca esparsa** similar ao BM25, mas aprendida; (3) **multi-vetor** (ColBERT-style), onde cada token gera seu próprio vetor e o score é calculado por correspondência cruzada. Para o nosso curso, o BGE-M3 é a escolha principal por suportar **mais de 100 idiomas** incluindo português, ter janela de contexto de **8192 tokens** (suficiente para petições longas), e oferecer qualidade de recuperação superior em benchmarks públicos para textos técnicos.

---

> ### 💡 INSIGHT — Comprimento de Contexto é Crítico no Direito
> Uma petição inicial pode ter 50 páginas. Um acórdão do STJ pode ter 200 páginas. Modelos com janela de contexto pequena (BERT: 512 tokens ≈ 380 palavras) precisam ser usados com estratégias de chunking (Aula 4). O BGE-M3 com 8192 tokens ≈ 6.000 palavras já cobre a maioria dos documentos jurídicos sem truncamento, preservando a coerência do raciocínio jurídico que frequentemente se desenvolve ao longo de todo um documento.

---

### 4.2 Perguntas para Reflexão

**Pergunta 1:** Por que usar BERT diretamente para busca semântica em um banco de dados com 100.000 documentos jurídicos seria computacionalmente inviável, e como Sentence-Transformers resolve esse problema?

> **Resposta:** O BERT foi projetado para processamento *crossencoder*: recebe dois textos concatenados e produz um score de similaridade. Para encontrar o documento mais similar a uma query em 100.000 documentos, seriam necessárias 100.000 passagens pelo BERT, o que levaria horas para cada busca. Sentence-Transformers usa um *biencoder*: os 100.000 documentos são encodados *uma única vez* e seus embeddings são armazenados. Na hora da busca, apenas a query é encodada (1 passagem), e a similaridade é calculada por produto interno com os vetores pré-computados — operação vetorizada que leva milissegundos.

**Pergunta 2:** Em um sistema de busca jurídica, qual seria a diferença prática entre usar BGE-M3 em modo "densa" versus modo "esparsa" para uma query como "crime de peculato doloso"?

> **Resposta:** No modo **denso**, o BGE-M3 encodaria a query como um vetor de 1024 dimensões e buscaria documentos com vetores próximos no espaço semântico — encontraria documentos sobre peculato mesmo que usem a expressão "desvio de verbas públicas com intenção". No modo **esparso**, o modelo geraria um vetor esparso com altos pesos para tokens importantes como "peculato" e "doloso", funcionando de forma similar ao BM25 — garantindo que documentos que literalmente contêm essas palavras sejam retornados. Para sistemas jurídicos de alta precisão, a combinação dos dois modos (busca híbrida, Aula 8) oferece o melhor resultado: relevância semântica + precisão lexical.

---

## 5. TÓPICO 4 — ARQUITETURA TRANSFORMER: ATENÇÃO, ENCODER-DECODER E POSITIONAL ENCODING

### 5.1 Explicação Teórica

**A atenção é tudo que você precisa**

Em 2017, Vaswani et al. publicaram o artigo "Attention Is All You Need", introduzindo a arquitetura Transformer que se tornou a base de praticamente todos os modelos de linguagem modernos. O insight central é o mecanismo de **atenção multi-cabeça** (multi-head attention): ao processar cada palavra (token) de uma frase, o modelo calcula dinamicamente quanto "atenção" deve prestar a *cada outra palavra* da frase. Pense em um juiz lendo um acórdão: ao encontrar o pronome "ele", o juiz automaticamente rastreia de volta no texto para identificar a qual pessoa "ele" se refere — réu, promotor, testemunha? O mecanismo de atenção faz exatamente isso, só que para todos os tokens simultaneamente e de forma paramétrica aprendida.

Matematicamente, a atenção é calculada através de três matrizes aprendidas: **Query** (Q), **Key** (K) e **Value** (V). Para cada token, Q representa "o que estou procurando", K representa "o que ofereço" e V representa "o que contribuo se for selecionado". O score de atenção entre dois tokens é o produto escalar QK^T normalizado por √d_k (para estabilidade numérica), passado por uma função softmax que distribui os pesos de 0 a 1. O resultado final é uma soma ponderada dos vetores V. O mecanismo "multi-cabeça" (tipicamente 8 a 16 cabeças em modelos modernos) executa esse processo em paralelo com diferentes matrizes Q, K, V, capturando diferentes tipos de relações: sintáticas, semânticas, correferências.

A arquitetura Transformer original tinha duas partes: **Encoder** (que processa a entrada completa com atenção bidirecional, como o BERT) e **Decoder** (que gera a saída token por token, vendo apenas os tokens anteriores — como os modelos GPT). Modelos apenas-encoder (BERT, RoBERTa, BGE-M3) são ideais para *compreensão* de texto: classificação, extração de informações, embeddings. Modelos apenas-decoder (GPT, Llama, Mistral) são ideais para *geração* de texto: respostas a perguntas, sumarização, análise jurídica. Modelos encoder-decoder (T5, BART) são versáteis para tarefas de sequência-para-sequência como tradução e sumarização estruturada.

O **Positional Encoding** resolve um problema fundamental: diferente das redes recorrentes (LSTM, GRU), o Transformer processa todos os tokens em paralelo e, portanto, não tem noção inerente de ordem. Sem positional encoding, "o réu matou a vítima" e "a vítima matou o réu" seriam idênticos. O positional encoding adiciona um vetor matemático a cada embedding de token, codificando sua posição na sequência. Modelos modernos usam **RoPE** (Rotary Position Embeddings) ou **ALiBi**, que permitem generalização para sequências mais longas que as vistas no treinamento — fundamental para processar documentos jurídicos extensos.

---

> ### 💡 INSIGHT — Por que LLMs "Alucinam"?
> O Decoder Transformer é treinado para prever o próximo token mais provável dado o contexto anterior. Ele não tem um mecanismo explícito para verificar se a informação que está gerando é factualmente correta — apenas gera o que é estatisticamente plausível. É como um assistente muito eloquente que preenche lacunas de conhecimento com afirmações convincentes. RAG resolve isso ancorando a geração em documentos recuperados, forçando o modelo a usar informação verificável.

---

### 5.2 Perguntas para Reflexão

**Pergunta 1:** Por que um modelo apenas-decoder (como GPT-4 ou Llama) é mais adequado para gerar um parecer jurídico do que um modelo apenas-encoder (como BERT), mesmo que ambos sejam Transformers?

> **Resposta:** Modelos apenas-encoder foram treinados para *compreensão* — eles produzem representações ricas de texto de entrada, mas não foram projetados para gerar texto novo token a token. O BERT pode classificar um documento como "acórdão condenatório" ou extrair o "nome do réu", mas não consegue redigir um novo parágrafo de fundamentação jurídica. O Decoder, treinado com objetivo de linguagem causal (predizer próximo token), aprendeu padrões de geração de texto coerente e contextualmente consistente — exatamente o que é necessário para redigir pareceres, sumários ou análises jurídicas.

**Pergunta 2:** Considere um Transformer processando a frase "O suspeito, que foi visto na câmera portando a arma, negou envolvimento no crime". Como o mecanismo de atenção ajuda o modelo a resolver a correferência entre "que" e "suspeito"?

> **Resposta:** Ao processar o token "que" (pronome relativo), o mecanismo de atenção calcula scores de atenção com *todos* os outros tokens da frase. O token "suspeito" (antecedente) receberá um score de atenção muito alto porque padrões similares foram aprendidos durante o treinamento — pronomes relativos anafóricos tendem a se referir ao substantivo imediatamente anterior que coincide em gênero e número. O resultado é que a representação vetorial de "que" ficará altamente influenciada pelo vetor de "suspeito", permitindo que o modelo faça inferências corretas sobre quem portava a arma e quem negou o envolvimento.

---

## 6. TÓPICO 5 — LARGE LANGUAGE MODELS: GPT, LLAMA, MISTRAL E PARÂMETROS DE GERAÇÃO

### 6.1 Explicação Teórica

**Da escala emergem capacidades**

Large Language Models (LLMs) são modelos Transformer apenas-decoder treinados em escalas anteriormente inimagináveis: trilhões de tokens de texto, bilhões a trilhões de parâmetros, meses de treinamento em milhares de GPUs. O que torna os LLMs qualitativamente diferentes de modelos menores não é apenas a escala — é o fenômeno das **capacidades emergentes**: habilidades que simplesmente não existem em modelos menores e aparecem abruptamente quando um limiar de tamanho é ultrapassado. Raciocínio em múltiplos passos (chain-of-thought), aritmética, seguir instruções complexas, tradução zero-shot — nenhuma dessas capacidades foi explicitamente treinada, elas emergiram da escala.

Os principais modelos de código aberto relevantes para este curso são: **Llama 3.1** (Meta, 2024) disponível em versões 8B, 70B e 405B parâmetros, com contexto de 128k tokens; **Mistral 7B** (Mistral AI, 2023), notável por superar o Llama 2 13B com apenas 7B parâmetros através de atenção com janela deslizante eficiente; e **Mixtral 8x7B**, um modelo de mistura de especialistas (MoE) que ativa apenas 2 dos 8 "especialistas" por token, oferecendo qualidade de 47B com custo de 13B. Para ambientes locais (sem enviar dados sensíveis para APIs externas — requisito fundamental em sistemas jurídicos e de segurança pública), utilizaremos **Ollama** para servir esses modelos localmente via API REST (compatível com o padrão OpenAI), sem necessidade de configuração de CUDA ou drivers complexos.

Os **parâmetros de geração** controlam o comportamento do modelo durante a inferência. **Temperature** (valores 0.0 a 2.0) controla a "criatividade": temperature=0 torna o modelo completamente determinístico (sempre escolhe o token de maior probabilidade), temperature=1.0 é o padrão, temperature>1.5 gera texto cada vez mais caótico. Para análises jurídicas onde precisão é crítica, use temperature=0.1 a 0.3. Para geração de alternativas criativas em brainstorming, temperature=0.7 a 1.0. **Top-p** (nucleus sampling, valores 0.0 a 1.0) restringe a geração ao conjunto de tokens que perfazem uma probabilidade acumulada p — top-p=0.9 considera apenas os tokens que juntos respondem por 90% da probabilidade, ignorando os 10% menos prováveis. **Max_tokens** (ou max_new_tokens) limita o tamanho da resposta.

Para sistemas RAG em contexto jurídico e de segurança pública, a escolha do modelo local certo envolve trade-offs importantes: Llama 3.1 8B é adequado para a maioria das tarefas de extração e QA, roda em GPUs de 16GB; Llama 3.1 70B oferece qualidade próxima ao GPT-4 mas exige 40GB de VRAM ou quantização 4-bit; Mistral 7B é a melhor relação custo-benefício para ambientes com recursos limitados. A **quantização** (técnica de reduzir a precisão dos pesos de float32/float16 para int8 ou int4) reduz o consumo de memória até 4x com perda de qualidade frequentemente imperceptível — tornando modelos de 70B executáveis em hardware acessível.

---

> ### 💡 INSIGHT — Soberania de Dados em Sistemas Jurídicos
> Processos sob segredo de justiça, investigações policiais em curso e dados de vítimas não podem ser enviados para APIs externas (OpenAI, Anthropic, Google). O uso de LLMs locais via **Ollama** não é apenas uma escolha técnica — é muitas vezes um **requisito legal** (LGPD, sigilo funcional) e ético. Este curso prioriza soluções *on-premise* e *air-gapped* justamente por essa razão.

---

### 6.2 Perguntas para Reflexão

**Pergunta 1:** Um perito forense quer usar um LLM para analisar comunicações interceptadas legalmente. Por que ele **não deve** usar a API do ChatGPT/GPT-4, mesmo que ofereça a melhor qualidade de análise?

> **Resposta:** (1) **LGPD**: Dados de investigações criminais contendo informações pessoais de investigados são dados sensíveis; enviá-los para servidores da OpenAI (EUA) constitui transferência internacional de dados pessoais sem adequação legal. (2) **Sigilo funcional**: As comunicações interceptadas estão sob sigilo judicial; compartilhá-las com terceiros (mesmo uma empresa de tecnologia) pode configurar violação de sigilo profissional ou mesmo crime. (3) **Cadeia de custódia**: Em uma eventual ação penal, a defesa poderia questionar a integridade das provas processadas por sistemas externos não auditáveis. A alternativa correta é usar LLMs locais (Llama, Mistral via Ollama) em servidores sob controle da instituição.

**Pergunta 2:** Por que usar temperature=0.0 ao gerar uma conclusão jurídica, mas temperature=0.7 ao gerar sugestões de argumentos para uma peça processual?

> **Resposta:** Com temperature=0.0 (ou muito baixa), o LLM é **determinístico**: dado o mesmo contexto, sempre gera a mesma resposta. Para conclusões jurídicas onde precisamos de consistência e a "resposta mais correta" dado o contexto RAG recuperado, isso é ideal — não queremos que o modelo "invente" variações em sentenças factuais. Com temperature=0.7, o modelo é mais **criativo e variado**: cada execução gera argumentos ligeiramente diferentes, o que é valioso quando um advogado quer explorar diferentes ângulos argumentativos para uma peça, beneficiando-se da diversidade de perspectivas do modelo.

---

## 7. TÓPICO 6 — O QUE É RAG? POR QUE SURGIU E ONDE LLMS FALHAM

### 7.1 Explicação Teórica

**O problema do conhecimento congelado**

Imagine contratar o melhor consultor jurídico do mundo — alguém com conhecimento enciclopédico de toda a jurisprudência, doutrina e legislação até 2023. Perfeito, exceto por um detalhe: ele ficou em coma desde então. Você não pode atualizá-lo sobre a Reforma Tributária de 2024, as novas súmulas do STJ de março de 2025, ou o decreto presidencial assinado ontem. Quando consultado sobre esses temas, ele não diz "não sei" — ele *extrapola* com confiança a partir do que sabe, gerando respostas plausíveis mas potencialmente incorretas. Esse é exatamente o problema do **knowledge cutoff** dos LLMs: o modelo foi treinado até uma data específica e não tem conhecimento de eventos posteriores.

Mas o problema é ainda mais profundo. Mesmo para eventos *dentro* do período de treinamento, LLMs sofrem de **alucinação**: geram informações falsas com aparente confiança. Um LLM pode inventar números de processos inexistentes, citar artigos de lei que não existem, ou fabricar precedentes do STF. A raiz da alucinação está na natureza do treinamento: o modelo foi otimizado para gerar texto *plausível*, não texto *verdadeiro*. A distinção é sutil, mas crucial: "plausível" significa "estatisticamente similar ao que aparece nos dados de treinamento", não "factualmente correto". Para um sistema de apoio jurídico, gerar um precedente falso com aparência de real pode ter consequências devastadoras.

O **fine-tuning** — ajustar os pesos do modelo em dados específicos do domínio — parece uma solução óbvia, mas tem limitações sérias: é **caro** (treinar mesmo um modelo de 7B em um corpus jurídico requer semanas de GPU), **estático** (o conhecimento fica congelado novamente após o treino), e pode causar **catastrophic forgetting** (o modelo perde capacidades gerais ao ser especializado). Além disso, o fine-tuning não resolve a alucinação — um modelo que "sabe" mais sobre direito ainda pode fabricar precedentes convincentes.

**RAG (Retrieval-Augmented Generation)** resolve elegantemente esses três problemas. A ideia, introduzida por Lewis et al. (2020) no paper "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks", é simples: ao invés de confiar apenas na memória paramétrica do LLM, **recupere documentos relevantes do seu corpus** e forneça-os como contexto para a geração. O fluxo é: (1) usuário faz uma pergunta, (2) sistema recupera os K documentos mais relevantes do banco de dados vetorial, (3) documentos + pergunta são enviados ao LLM como contexto, (4) LLM gera resposta ancorada nos documentos recuperados. O conhecimento fica no banco de dados (atualizável em tempo real), não nos pesos do modelo; a alucinação é reduzida pois o modelo cita fontes verificáveis; o custo é vastamente inferior ao fine-tuning.

---

> ### 💡 INSIGHT — RAG como "Pejota Jurídico"
> Pense no RAG como um paralegal extremamente eficiente: quando o advogado faz uma pergunta complexa, o paralegal não responde de cabeça — vai à biblioteca, localiza os precedentes relevantes, organiza as cópias sobre a mesa do advogado, e *então* o advogado (LLM) constrói o raciocínio jurídico com as fontes físicas na frente. A qualidade da resposta depende tanto da capacidade do "advogado" (LLM) quanto da eficiência do "paralegal" (sistema de retrieval). É por isso que este curso dedica tanto tempo às técnicas de recuperação.

---

### 7.2 Perguntas para Reflexão

**Pergunta 1:** Um promotor usa um chatbot RAG para consultar jurisprudência e recebe uma resposta com 5 precedentes citados. Como ele deve verificar se os precedentes são reais e corretamente citados?

> **Resposta:** Um sistema RAG bem implementado deve sempre retornar as **fontes dos documentos recuperados** junto com a resposta (metadados: número do processo, órgão julgador, data, relator). O promotor deve: (1) verificar se os metadados dos documentos recuperados correspondem ao precedente citado na resposta, (2) acessar diretamente o sistema do tribunal (STJ, STF, TJ) para confirmar a existência e o teor do acórdão, (3) checar se a citação do trecho no contexto RAG é fiel ao documento original. Um sistema RAG confiável não deve gerar precedentes que não existam nos documentos recuperados — se o fizer, é sinal de alucinação mesmo com contexto fornecido, o que exige ajuste no prompt (instrução explícita: "cite apenas informações dos documentos fornecidos").

**Pergunta 2:** Qual a diferença fundamental entre RAG e fine-tuning para um sistema de IA jurídica, e em que cenário cada abordagem seria mais adequada?

> **Resposta:** **RAG** mantém o conhecimento *externo* aos pesos do modelo (no banco de dados), permitindo atualização instantânea, rastreabilidade das fontes e controle granular do que o modelo pode citar. É ideal para: consulta de jurisprudência atualizada, busca de precedentes específicos, resposta a perguntas factuais sobre legislação vigente. **Fine-tuning** incorpora conhecimento *interno* aos pesos, útil quando o modelo precisa aprender o *estilo* ou *estrutura* de um domínio (ex.: redigir petições no padrão do escritório, extrair entidades específicas como número de CPF/processo) ou quando não há acesso à internet para retrieval em tempo real. A combinação ideal é **RAG + fine-tuning leve** (PEFT/LoRA): o modelo aprende o estilo e terminologia do domínio, enquanto o conhecimento factual fica no banco de dados atualizado.

---

## 8. TÓPICO 7 — PANORAMA DAS 25 TÉCNICAS RAG DO CURSO

### 8.1 Mapa Visual do Curso

As 25 técnicas RAG estão distribuídas ao longo das 12 aulas em uma progressão lógica de complexidade:

```
╔══════════════════════════════════════════════════════════════╗
║          PANORAMA DO CURSO — 25 TÉCNICAS RAG                ║
╠══════════════════════════════════════════════════════════════╣
║ FUNDAÇÃO (Aulas 1-3)                                         ║
║  [1] Embeddings densos (BGE-M3, sentence-transformers)       ║
║  [2] FAISS — busca vetorial local                            ║
║  [3] OpenSearch — busca híbrida (densa + esparsa)            ║
║  [4] Chunking básico (fixed-size, overlap)                   ║
║                                                              ║
║ INGESTÃO INTELIGENTE (Aula 4)                                ║
║  [5] Docling — parsing PDF/DOCX estruturado                  ║
║  [6] Chunking semântico e hierárquico                        ║
║  [7] Metadata enrichment e filtragem                         ║
║                                                              ║
║ RETRIEVAL AVANÇADO (Aulas 5-6)                               ║
║  [8] Busca híbrida com RRF (Reciprocal Rank Fusion)          ║
║  [9] HyDE — Hypothetical Document Embeddings                 ║
║  [10] Multi-query retrieval                                  ║
║  [11] Contextual compression                                 ║
║  [12] Parent-child chunking                                  ║
║                                                              ║
║ RERANKING & RACIOCÍNIO (Aula 7)                              ║
║  [13] Cross-encoder reranking (bge-reranker)                 ║
║  [14] LLM reranking                                          ║
║  [15] Chain-of-thought RAG                                   ║
║  [16] Self-RAG (reflexivo)                                   ║
║                                                              ║
║ MCP & ORQUESTRAÇÃO (Aula 8)                                  ║
║  [17] Model Context Protocol                                  ║
║  [18] Agentic RAG com ferramentas                            ║
║  [19] Multi-agent RAG                                        ║
║                                                              ║
║ CAG & CACHE (Aula 9)                                         ║
║  [20] CAG — Cache-Augmented Generation                       ║
║  [21] KV-Cache management                                    ║
║  [22] Semantic caching                                       ║
║                                                              ║
║ AVALIAÇÃO & PRODUÇÃO (Aulas 10-12)                           ║
║  [23] RAGAs — Métricas de avaliação                          ║
║  [24] LangFuse — Observabilidade e rastreamento              ║
║  [25] Deploy Kubernetes — RAG em produção segura             ║
╚══════════════════════════════════════════════════════════════╝
```

### 8.2 Aplicações por Área no Domínio Jurídico e Segurança Pública

| Técnica | Aplicação Prática no Curso |
|---------|---------------------------|
| Embeddings BGE-M3 | Busca semântica em acórdãos e boletins de ocorrência |
| FAISS | Índice local de processos para uso off-line (tribunais sem internet) |
| OpenSearch híbrido | Motor de busca para corpus de 1M+ documentos jurídicos |
| Docling | Ingestão de PDFs de processos, laudos periciais, portarias |
| Chunking hierárquico | Preservar estrutura de acórdãos (ementa → relatório → votos) |
| HyDE | Melhorar busca por conceitos jurídicos abstratos |
| Reranking | Priorizar precedentes mais recentes e vinculantes |
| Chain-of-thought RAG | Raciocínio jurídico multi-etapa (subsunção fática → jurídica) |
| MCP | Integração com sistemas legados (PJe, SEEU, SisJuS) |
| CAG | Chatbot investigativo com contexto persistente de caso |
| RAGAs + LangFuse | Auditoria e explicabilidade das respostas geradas |
| Kubernetes | Deploy seguro em infraestrutura do Poder Judiciário/Polícia |

---

## 9. EXERCÍCIOS DE FIXAÇÃO COM RESOLUÇÕES

### Exercício 1 — Múltipla Escolha

**Qual das seguintes afirmações sobre TF-IDF é CORRETA?**

a) TF-IDF consegue identificar sinônimos como "automóvel" e "veículo" como semanticamente similares.
b) TF-IDF atribui peso alto a palavras raras no documento mas comuns no corpus.
c) TF-IDF atribui peso alto a palavras frequentes no documento mas raras no corpus.
d) TF-IDF usa um mecanismo de atenção para identificar termos relevantes.

> **Resposta: C.** TF-IDF = TF (frequência do termo no documento) × IDF (logaritmo do total de documentos dividido pelo número de documentos que contêm o termo). Palavras raras no corpus (alto IDF) que aparecem frequentemente num documento específico (alto TF) recebem alto peso. Palavras comuns como "o", "de", "que" têm baixo IDF (aparecem em quase todos os documentos), resultando em peso baixo mesmo que apareçam muito num documento específico. TF-IDF **não** captura sinônimos (a) e **não** usa atenção (d).

---

### Exercício 2 — Análise de Cenário

**Um delegado propõe usar Word2Vec treinado em processos criminais para criar um sistema de busca semântica de B.Os. Identifique duas limitações específicas dessa abordagem para o domínio investigativo.**

> **Resposta Esperada:**
>
> **Limitação 1 — Ausência de contexto:** Word2Vec atribui um único vetor fixo a cada palavra. A palavra "banco" teria o mesmo vetor em "assalto ao banco" (instituição financeira) e "réu sentado no banco" (mobiliário de audiência). Em buscas investigativas, isso geraria ruído: uma busca sobre crimes bancários poderia retornar B.Os sobre audiências, e vice-versa.
>
> **Limitação 2 — Erros ortográficos em B.Os:** Boletins de ocorrência frequentemente contêm erros tipográficos ("homicidio", "veiculo", "testemunha" escritas com variações). Word2Vec não conhece palavras fora do vocabulário de treinamento, retornando [UNK] para essas formas. fastText ou modelos de subpalavra seriam mais adequados por decompor palavras em n-gramas de caracteres.

---

### Exercício 3 — Implementação Conceitual

**Desenhe o fluxo de um sistema RAG simples para responder à pergunta "Quais são os precedentes do STJ sobre crime de peculato no setor de saúde?" Inclua os 4 componentes principais e descreva o que acontece em cada etapa.**

> **Resposta Esperada:**
>
> ```
> [Usuário] → "Quais precedentes STJ sobre peculato no setor saúde?"
>      ↓
> [1. ENCODER] Query → Vetor de embedding (BGE-M3)
>      ↓
> [2. RETRIEVER] Busca vetorial no índice de acórdãos → Top-5 acórdãos mais similares
>      ↓
> [3. CONTEXTO] Concatenação: prompt + acórdãos recuperados
>      ↓
> [4. LLM] Geração de resposta citando os precedentes recuperados
>      ↓
> [Usuário] ← Resposta com referências verificáveis
> ```
>
> **Etapa 1 (Encoder):** A query é convertida em um vetor de 1024 dimensões pelo BGE-M3. Esse vetor captura a semântica da busca: peculato + setor saúde + precedentes STJ.
>
> **Etapa 2 (Retriever):** O vetor da query é comparado por similaridade coseno com todos os vetores dos acórdãos indexados. Os 5 acórdãos com maior similaridade são recuperados com seus metadados (número, data, relator, ementa).
>
> **Etapa 3 (Contexto):** Um prompt estruturado é montado: "Baseado nos seguintes acórdãos do STJ: [Acórdão 1]... [Acórdão 5]... Responda: [pergunta original]".
>
> **Etapa 4 (LLM):** O Llama 3.1 8B processa o contexto completo e gera uma resposta estruturada, idealmente citando os números dos processos e trechos relevantes dos acórdãos fornecidos.

---

### Exercício 4 — Parâmetros de Geração

**Complete a tabela com os valores de temperatura mais adequados para cada tarefa:**

| Tarefa | Temperature Recomendada | Justificativa |
|--------|------------------------|---------------|
| Extrair o número do processo de um acórdão | ? | ? |
| Gerar 5 argumentos alternativos para uma peça de defesa | ? | ? |
| Resumir a ementa de um acórdão | ? | ? |
| Criar uma metáfora para explicar habeas corpus a leigos | ? | ? |

> **Resposta:**
>
> | Tarefa | Temperature | Justificativa |
> |--------|-------------|---------------|
> | Extrair número do processo | 0.0 | Tarefa determinística, única resposta correta |
> | Gerar 5 argumentos alternativos | 0.7–0.9 | Diversidade criativa desejada |
> | Resumir ementa | 0.1–0.3 | Fidelidade ao original, alguma variação aceitável |
> | Metáfora para leigos | 0.8–1.0 | Alta criatividade, múltiplas metáforas válidas |

---

### Exercício 5 — Verdadeiro ou Falso

Julgue cada afirmação:

1. "O BGE-M3 precisa de conexão com internet para gerar embeddings após o download inicial." (**Falso** — após o download, o modelo roda completamente local.)
2. "Fine-tuning resolve o problema de knowledge cutoff de forma permanente." (**Falso** — o fine-tuning cria um novo cutoff na data em que foi realizado; atualização contínua exigiria re-fine-tunings frequentes.)
3. "Sentence-Transformers e BERT usam exatamente a mesma arquitetura." (**Falso** — Sentence-Transformers usa BERT como backbone em uma arquitetura siamesa com loss especial para similaridade de frases.)
4. "Em um sistema RAG, o LLM pode gerar informações que não estão nos documentos recuperados." (**Verdadeiro** — alucinação ainda ocorre mesmo com RAG, especialmente se o prompt não instruir o modelo a se ater ao contexto fornecido.)
5. "Temperature=0 garante que o LLM nunca vai alucinar." (**Falso** — temperature controla aleatoriedade na seleção de tokens, não a factualidade; um modelo pode gerar consistentemente a mesma informação incorreta com temperature=0.)

---

## 10. REFERÊNCIAS EXPANDIDAS

### Referências Primárias (Artigos Fundacionais)

**VASWANI, A. et al.** Attention Is All You Need. *Advances in Neural Information Processing Systems (NeurIPS)*, v. 30, 2017. Disponível em: https://arxiv.org/abs/1706.03762. Acesso em: 14 abr. 2026.

**REIMERS, N.; GUREVYCH, I.** Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks. In: *Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing (EMNLP)*, 2019, p. 3982–3992. Disponível em: https://arxiv.org/abs/1908.10084. Acesso em: 14 abr. 2026.

**LEWIS, P. et al.** Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. *Advances in Neural Information Processing Systems (NeurIPS)*, v. 33, 2020. Disponível em: https://arxiv.org/abs/2005.11401. Acesso em: 14 abr. 2026.

**DEVLIN, J. et al.** BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding. In: *Proceedings of NAACL-HLT 2019*, 2019, p. 4171–4186. Disponível em: https://arxiv.org/abs/1810.04805. Acesso em: 14 abr. 2026.

**MIKOLOV, T. et al.** Distributed Representations of Words and Phrases and their Compositionality. *Advances in Neural Information Processing Systems*, v. 26, 2013. Disponível em: https://arxiv.org/abs/1310.4546. Acesso em: 14 abr. 2026.

**XIAO, S. et al.** BGE-M3: Multi-Lingual, Multi-Functionality, Multi-Granularity Text Embeddings Through Self-Knowledge Distillation. *arXiv:2402.03216*, 2024. Disponível em: https://arxiv.org/abs/2402.03216. Acesso em: 14 abr. 2026.

### Referências Complementares (Ferramentas e Implementações)

**META AI.** Llama 3: Open Foundation and Fine-Tuned Chat Models. *Technical Report*, 2024. Disponível em: https://ai.meta.com/llama/. Acesso em: 14 abr. 2026.

**PENNINGTON, J.; SOCHER, R.; MANNING, C. D.** GloVe: Global Vectors for Word Representation. In: *Proceedings of EMNLP 2014*, 2014, p. 1532–1543. Disponível em: https://nlp.stanford.edu/pubs/glove.pdf. Acesso em: 14 abr. 2026.

**BOJANOWSKI, P. et al.** Enriching Word Vectors with Subword Information. *Transactions of the Association for Computational Linguistics*, v. 5, p. 135–146, 2017. Disponível em: https://arxiv.org/abs/1607.04606. Acesso em: 14 abr. 2026.

**JIANG, A. Q. et al.** Mistral 7B. *arXiv:2310.06825*, 2023. Disponível em: https://arxiv.org/abs/2310.06825. Acesso em: 14 abr. 2026.

### Referências para o Domínio Jurídico e Segurança Pública

**CONSELHO NACIONAL DE JUSTIÇA (CNJ).** Glossário de Termos para Inteligência Artificial no Poder Judiciário. Brasília: CNJ, 2023.

**BRASIL.** Lei nº 13.709, de 14 de agosto de 2018. Lei Geral de Proteção de Dados Pessoais (LGPD). *Diário Oficial da União*, Brasília, DF, 15 ago. 2018.

**FIRTH, J. R.** A synopsis of linguistic theory 1930-1955. *Studies in linguistic analysis*, p. 1–32, 1957. (Origem da frase "a word is characterized by the company it keeps".)

---

*Fim do Documento Teórico — Aula 1*

*Curso de MBA: RAG & CAG Aplicados a Direito e Segurança Pública*
*Conforme ABNT NBR 6023:2018 para referências bibliográficas*
