# MBA RAG & CAG — AULA 1: FUNDAMENTOS
## Índice de Materiais

> **Carga Horária:** 5 horas | **Proporção:** 30% Teoria / 70% Prática
> **Versão:** 4.0 — Maio/2026 (consolidação dos labs de ambiente)

---

## DOCUMENTOS DA AULA

### Teoria
| Arquivo | Conteúdo | Formato |
|---------|----------|---------|
| `teoria/AULA1_TEORIA.md` | 7 tópicos teóricos com analogias, insights, perguntas e resoluções, exercícios e referências ABNT | Markdown |

### Laboratórios (Jupyter Local + VS Code)
| Arquivo | Conteúdo | Tempo |
|---------|----------|-------|
| `labs/LAB1_Setup_Ambiente_Completo.ipynb` | Setup integrado: **Python 3.10+ / Jupyter**, **Podman Desktop** (Windows), **OpenSearch + Dashboards** via `podman-compose`, **Ollama** + modelos locais, **Langfuse local** via `podman-compose`, validação final | 120 min |
| `labs/LAB2_Embeddings_BGE_M3_UMAP.ipynb` | BGE-M3 vs sentence-transformers, similaridade coseno, UMAP 2D | 60 min |

> Os antigos LAB1–LAB4 foram **consolidados** no novo LAB1. Os arquivos originais permanecem em `labs/_deprecated/` apenas para consulta histórica.

### Exemplos
| Arquivo | Conteúdo | Tipo |
|---------|----------|------|
| `exemplos/EXEMPLO_Pipeline_RAG_Minimo.ipynb` | Pipeline RAG completo: corpus → embedding → FAISS → Ollama → LangFuse | Demo |

### Datasets
| Arquivo | Conteúdo | Formato |
|---------|----------|---------|
| `datasets/corpus_juridico_aula1.json` | 10 documentos jurídicos fictícios: acórdãos, BO, laudos, sentenças | JSON |

### Roteiro de Instalação (complementar)
| Arquivo | Conteúdo |
|---------|----------|
| `ROTEIRO_INSTALACAO_FERRAMENTAS.md` | Guia textual passo a passo do mesmo conteúdo do LAB1 — útil como referência rápida fora do notebook |

---

## STACK TECNOLÓGICO DESTA AULA

```
Python 3.10+ (3.11 recomendado)  → Linguagem do curso
Jupyter + VS Code                → Ambiente de execução dos labs (local)
Podman Desktop                   → Runtime OCI sem daemon, sem licenciamento comercial (recomendado)
Docker Desktop                   → Alternativa (apenas se já instalado no ambiente)
podman-compose                   → Orquestração de contêineres no formato docker-compose
OpenSearch 3.x                   → Motor de busca vetorial + textual
OpenSearch Dashboards            → Interface visual para exploração
Ollama                           → Servidor de LLMs e embeddings local (Windows/macOS/Linux)
llama3.2:3b                      → LLM padrão do curso (leve, 2 GB)
llama3.1:8b                      → LLM avançado (mais capaz, 5 GB)
nomic-embed-text                 → Modelo de embedding via Ollama (768 dims)
mxbai-embed-large                → Embedding alternativo (1024 dims)
Langfuse (self-hosted)           → Observabilidade local em Postgres+Clickhouse via compose
Langfuse Cloud                   → Alternativa rápida (https://cloud.langfuse.com)
sentence-transformers            → Framework de embeddings (local, sem Ollama)
BGE-M3 (BAAI)                    → Modelo de embedding estado da arte
FAISS                            → Índice vetorial local
UMAP                             → Redução dimensional para visualização
```

---

## CRITERIOS DE AVALIACAO

| Item | Descrição | Pontos |
|------|-----------|--------|
| LAB 1 | Checklist de 8 itens (ambiente local 100% funcional, validação integrada OK) | 50 pts |
| LAB 2 | Checklist de 7 itens (UMAP com clusters visíveis, BGE-M3 funcional) | 30 pts |
| Exercícios | 5 exercícios de fixação no LAB1 + exercícios da teoria | 20 pts |
| **TOTAL** | | **100 pts** |

---

## SEQUÊNCIA RECOMENDADA

```
[1] Leia teoria/AULA1_TEORIA.md — Tópicos 1-4 (45 min)
    |
[2] Execute LAB1_Setup_Ambiente_Completo — instalação integrada (120 min)
        - Etapa 1: Python 3.10+ e venv_rag
        - Etapa 2: Jupyter + VS Code
        - Etapa 3: Podman Desktop
        - Etapa 4: OpenSearch + Dashboards (podman-compose)
        - Etapa 5: Ollama + modelos
        - Etapa 6: Langfuse local (podman-compose)
        - Etapa 7: Validação integrada (todos os checks OK)
    |
[3] Leia teoria/AULA1_TEORIA.md — Tópicos 5-7 (30 min)
    |
[4] Execute LAB2_Embeddings_BGE_M3_UMAP — embeddings + UMAP (60 min)
    |
[5] Resolva os Exercícios de Fixação (40 min)
    |
[6] BONUS: Execute o EXEMPLO_Pipeline_RAG_Minimo.ipynb
```

---

## AMBIENTES SUPORTADOS

| Componente | Windows 11 | macOS 13+ | Ubuntu 22.04 |
|-----------|-----------|-----------|-------------|
| VS Code + Jupyter | OK | OK | OK |
| Podman Desktop | OK (WSL2) | OK | OK (podman nativo) |
| Docker Desktop (alternativa) | OK (WSL2) | OK | OK |
| Ollama | OK (nativo) | OK (nativo) | OK (nativo) |
| OpenSearch (containers) | OK | OK | OK |
| Langfuse local (containers) | OK | OK | OK |

> **Todos os labs funcionam 100% no Jupyter Local via VS Code — sem necessidade de Google Colab ou GPU obrigatória.**
> **Toda a infraestrutura é *self-hosted*** — alinhada aos requisitos de LGPD e sigilo funcional do Poder Judiciário e da Segurança Pública.

---

*MBA RAG & CAG Aplicados a Direito e Segurança Pública — Aula 1*
