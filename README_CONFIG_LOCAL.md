# Guia para avaliação e instalação das aplicações em ambiente local

## Contexto

Análise das aplicações identificadas no projeto **MBA RAG & CAG Aplicados a Direito e Segurança Pública** (aulas 1–12) para orientar a configuração de VMs Ubuntu com acesso local. O objetivo é separar aplicações que exigem GPU das que podem rodar em container Podman ou instalação direta no Ubuntu, informando os requisitos mínimos de cada aplicação e como instalá-las.

> O presente guia é uma sugestão para utilização em ambiente coorporativo com restrição ao uso de aplicações comerciais e poucos recursos de hardware. 


## Decisões e justificativas

- **vLLM isolado em VM com GPU**: exige CUDA e VRAM dedicada; compartilhar com VM de CPU degradaria todos os serviços simultaneamente.
- **Ollama na mesma VM GPU que o vLLM**: se a VRAM for ≥ 12 GB, a convivência é possível; em cenários de restrição de hardware (ex: 6 GB de VRAM), deve-se rodar apenas um dos serviços por vez para evitar Out of Memory (OOM) da GPU. 
- -- Recomendação de criar uma VM para cada aplicação ollama / vLLM
- -- Dimensionar, se possível, a VM para rodar os modelos na VRAM por longos períodos (ollama run <model_name> --keepalive -1)
- **RAGatouille e LLMLingua como bibliotecas pip, sem servidor próprio**: não expõem porta, são chamadas diretamente pelo código do notebook; instalar na VM GPU onde o processo Python tem acesso nativo aos drivers NVIDIA. **Atenção aos conflitos de dependência do PyTorch** ao instalá-las.
- **OpenSearch via Podman rootless**: evita dependência de Docker daemon; volume nomeado garante persistência; a configuração `vm.max_map_count` é um ajuste de kernel obrigatório no host para evitar falhas de inicialização do mmap.
- **Langfuse self-hosted via podman-compose**: requer PostgreSQL + Redis + servidor web; a forma oficial de self-hosting é via compose; senhas criptográficos (senha do banco, JWT secret e salt de hashing) devem ser gerados via `openssl rand` e armazenados em `.env` fora do repositório; o acesso à interface web usa login por conta de usuário (e-mail + senha criados no primeiro acesso em `http://localhost:3000`), independente das credenciais do banco.
- **Langflow na VM CPU**: ambos são leves, sem requisito de GPU; Langflow pode ser instalado via pip ou container; FastAPI não tem servidor próprio nos labs, mas a orientação cobre deploy com Podman para a Aula 12.
- **FastAPI na VM CPU** as aplicações fastApi compartilhadas pelos usuários, aplicações meio, podem ser instalada em uma VM com CPU. 
- **FastApi do usuário** Aplicações individuais de testes para consumir modelos devem ser instaladas no ambiente do usuário.
- **Versões mais recentes estáveis**: vLLM 0.19.1, OpenSearch 3.6.0, Langfuse 3.174.1, Ollama 0.24.0, Langflow 1.9.2, FastAPI 0.136.1, Uvicorn 0.47.0, Docling 2.93.0 (é mandatório verificar as releases oficiais no momento da instalação).

## Tabela resumo

| Aplicação | GPU? | Deploy recomendado | Versão estável | RAM mín. | Porta |
|---|---|---|---|---|---|
| vLLM | ✅ Obrigatório | VM GPU — processo local ou Podman+NVIDIA | 0.19.1 | 8–16 GB | 8000 |
| Ollama | ✅ Recomendado (CPU opcional) | VM GPU (junto ao vLLM) | 0.24.0 | 8 GB | 11434 |
| RAGatouille / ColBERT | ✅ Recomendado | pip na VM GPU | 0.0.8.post4 | 4 GB | — |
| LLMLingua | ⚠️ GPU opcional | pip na VM GPU | 0.2.2+ | 2 GB | — |
| OpenSearch | ❌ CPU | Podman single-node | 3.6.0 | 4 GB | 9200 |
| Langfuse | ❌ CPU | Podman Compose (PG + Redis + web) | 3.174.1 | 2 GB | 3000 |
| Langflow | ❌ CPU | Podman container ou pip | 1.9.2 | 2 GB | 7860 |
| FastAPI + Uvicorn | ❌ CPU | Ubuntu nativo ou Podman | 0.136.1 / 0.47.0 | 512 MB | 8080 |
| Tesseract-OCR | ❌ CPU | apt-get no Ubuntu | 5.x | 512 MB | — |
| Docling | ❌ CPU (OCR GPU opcional) | pip no Ubuntu | 2.93.0 | 1 GB | — |

## Perfil de cada aplicação

| Aplicação | Tipo | Persiste dados? | Requer login? | Comportamento |
|---|---|---|---|---|
| vLLM | Servidor HTTP | ❌ Não | ❌ Não (API key opcional) | Stateless — recebe prompt via REST, retorna tokens. Sem banco, sem sessão. |
| Ollama | Servidor HTTP | ⚠️ Modelos em disco | ❌ Não | Stateless para inferência. Persiste apenas os arquivos de modelo baixados. |
| RAGatouille | Biblioteca Python | ⚠️ Índice ColBERT em disco | ❌ Não | Carregado em processo Python. Pode gravar índice local para reutilização. |
| LLMLingua | Biblioteca Python | ❌ Não | ❌ Não | Stateless — recebe texto, retorna texto comprimido. Sem escrita em disco. |
| OpenSearch | Banco de dados | ✅ Sim — índices vetoriais e documentos | ✅ Sim (usuário admin) | Persiste todos os documentos e vetores indexados. Requer credenciais para escrita e leitura. |
| Langfuse | Plataforma web | ✅ Sim — traces, scores, avaliações (PostgreSQL) | ✅ Sim (conta de usuário) | Registra histórico completo de todas as chamadas LLM. Possui interface web com login, projetos e chaves de API por projeto. |
| Langflow | Plataforma web | ✅ Sim — flows salvos em banco local | ✅ Sim (conta de usuário) | Interface visual drag-and-drop. Persiste flows criados pelo usuário. Requer login quando `LANGFLOW_AUTO_LOGIN=false`. |
| FastAPI + Uvicorn | Servidor HTTP | ❌ Não (por padrão) | ❌ Depende da implementação | Stateless na Aula 12 — processa requisição e retorna resposta. Sem persistência própria; delega ao OpenSearch/Langfuse. |
| Tesseract-OCR | Biblioteca / CLI | ❌ Não | ❌ Não | Stateless — converte imagem/PDF em texto. Sem servidor, sem banco. |
| Docling | Biblioteca Python | ❌ Não (cache de modelos em disco) | ❌ Não | Stateless — converte documento em estrutura semântica. Faz download de modelos de layout na primeira execução e os armazena em cache. |

> **Resumo prático**: as únicas aplicações que exigem criação de conta/login são **OpenSearch** (usuário `admin`), **Langfuse** (conta web + chaves de projeto) e **Langflow** (quando autenticação habilitada). As demais são stateless e acessadas diretamente por API ou chamada de função.

## Requisitos consolidados por VM

### VM GPU — vLLM + Ollama + RAGatouille + LLMLingua

| Componente | RAM | vCPU | Disco | VRAM |
|---|---|---|---|---|
| vLLM (Qwen 2.5 3B float16) | 8 GB | 4 | 60 GB | 6 GB |
| vLLM (Llama 3.1 8B) | 16 GB | 8 | 60 GB | 16 GB |
| Ollama (modelos ≤ 7B) | 8 GB | 4 | 40 GB | 8 GB |
| RAGatouille / ColBERT | 4 GB | 4 | 10 GB | 4 GB |
| LLMLingua | 2 GB | 2 | 5 GB | 2 GB |
| **Total recomendado** | **32 GB** | **8+** | **150 GB SSD** | **≥ 8 GB** |
| **Total mínimo absoluto** | **16 GB** | **4** | **80 GB SSD** | **6 GB** |


### VM CPU — OpenSearch + Langfuse + Langflow + FastAPI

| Componente | RAM | vCPU | Disco |
|---|---|---|---|
| OpenSearch (JVM heap) | 4 GB | 2 | 30 GB |
| Langfuse (PG + Redis + web) | 2 GB | 2 | 10 GB |
| Langflow | 2 GB | 2 | 5 GB |
| FastAPI + Uvicorn | 512 MB | 1 | 2 GB |
| Tesseract-OCR + Docling | 1 GB | 1 | 3 GB |
| SO + overhead | 4 GB | — | — |
| **Total recomendado** | **16 GB** | **8** | **50 GB SSD** |
| **Total mínimo absoluto** | **12 GB** | **4** | **30 GB SSD** |

> **Nota sobre FastApi**: Provavelmente o uso da FastApi será individual para consumo das outras aplicações, não sendo necessário considerar no dimensionamento.

## Diagrama de arquitetura

```
┌──────────────────────────────────────────────────────┐
│  VM GPU  (32 GB RAM / 8 vCPU / 150 GB SSD / GPU)    │
│                                                      │
│  ├─ vLLM serve    :8000  — API OpenAI-compatível     │
│  ├─ Ollama        :11434 — modelos alternativos      │
│  └─ pip (sem porta):                                 │
│       RAGatouille, LLMLingua, FlagEmbedding          │
└──────────────────────────┬───────────────────────────┘
                           │ HTTP (localhost ou LAN)
┌──────────────────────────▼───────────────────────────┐
│  VM CPU  (16 GB RAM / 8 vCPU / 50 GB SSD)           │
│                                                      │
│  ├─ opensearch    :9200  (Podman, volume nomeado)    │
│  ├─ langfuse      :3000  (Podman Compose)            │
│  ├─ langflow      :7860  (Podman ou pip)             │
│  ├─ fastapi       :8080  (Podman ou nativo, opcional)│
│  └─ apt: tesseract-ocr   pip: docling                │
└──────────────────────────────────────────────────────┘
```

---

## Grupo A — Aplicações com GPU

---

### 1. vLLM

#### Função no projeto
Servidor de inferência LLM com API compatível com OpenAI. Usado nas aulas 1 (LAB3), 4 (LAB5 — Contextual Retrieval), 5 (RAGAS LLM Judge), 7, 8 (Self-RAG), 9 (LightRAG), 10 (Agentic RAG) e 12.

#### Versão estável recomendada
**0.19.1** — versão estável mais recente disponível no PyPI (maio 2026). Verificar `pip index versions vllm` antes de instalar.

#### Requisitos mínimos

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM sistema | 8 GB | 16 GB |
| vCPU | 4 | 8 |
| Disco (modelos + cache HF) | 60 GB SSD | 150 GB SSD |
| VRAM GPU | 6 GB (Qwen 3B) | 16 GB (Llama 3.1 8B) |
| NVIDIA Driver | ≥ 550 | — |
| CUDA | 12.x | 12.8 |
| Python | 3.11+ | 3.11 |

#### Instalação no Ubuntu Server

```bash
# 1. Verificar driver e CUDA
nvidia-smi

# 2. Instalar Python 3.11 se necessário
sudo apt-get update && sudo apt-get install -y python3.11 python3.11-venv python3-pip

# 3. Criar venv dedicado
python3.11 -m venv ~/.venvs/vllm
source ~/.venvs/vllm/bin/activate

# 4. Instalar vLLM (sem pinagem estrita para obter a mais recente ou aplicar patch local se necessário)
pip install --upgrade pip
pip install vllm

# 5. Criar arquivo de configuração de ambiente (NUNCA commitar este arquivo)
mkdir -p ~/.config/vllm
cat > ~/.config/vllm/vllm.env <<'EOF'
HUGGING_FACE_HUB_TOKEN=hf_SEU_TOKEN_AQUI
VLLM_MODEL=Qwen/Qwen2.5-3B-Instruct
VLLM_PORT=8000
VLLM_GPU_MEM_FRAC=0.90
VLLM_MAX_CTX=1024
EOF
chmod 600 ~/.config/vllm/vllm.env

# 6. Testar inicialização manual
source ~/.config/vllm/vllm.env
vllm serve "$VLLM_MODEL" \
  --host 0.0.0.0 \
  --port "$VLLM_PORT" \
  --gpu-memory-utilization "$VLLM_GPU_MEM_FRAC" \
  --max-model-len "$VLLM_MAX_CTX" \
  --dtype half

# 7. Instalar como serviço systemd
sudo tee /etc/systemd/system/vllm.service > /dev/null <<EOF
[Unit]
Description=vLLM OpenAI-compatible LLM server
After=network.target

[Service]
Type=simple
User=${USER}
EnvironmentFile=/home/${USER}/.config/vllm/vllm.env
ExecStart=/home/${USER}/.venvs/vllm/bin/vllm serve \${VLLM_MODEL} \
  --host 0.0.0.0 \
  --port \${VLLM_PORT} \
  --gpu-memory-utilization \${VLLM_GPU_MEM_FRAC} \
  --max-model-len \${VLLM_MAX_CTX} \
  --dtype half
Restart=on-failure
RestartSec=15
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable vllm
sudo systemctl start vllm
journalctl -u vllm -f  # acompanhar logs
```

#### Instalação via container Podman

```bash
# 1. Instalar NVIDIA Container Toolkit (rootless Podman)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit

# Configurar runtime para Podman rootless
sudo nvidia-ctk runtime configure --runtime=crun \
  --config="$HOME/.config/containers/containers.conf"

# 2. Criar arquivo .env local (NUNCA commitar)
cat > ~/vllm.env <<'EOF'
HUGGING_FACE_HUB_TOKEN=hf_SEU_TOKEN_AQUI
EOF
chmod 600 ~/vllm.env

# 3. Executar container
podman run -d --name vllm-server \
  --device nvidia.com/gpu=all \
  -p 8000:8000 \
  -v "$HOME/.cache/huggingface:/root/.cache/huggingface:z" \
  --env-file ~/vllm.env \
  docker.io/vllm/vllm-openai:latest \
  --model Qwen/Qwen2.5-3B-Instruct \
  --gpu-memory-utilization 0.90 \
  --max-model-len 1024 \
  --dtype half

# 4. Verificar
curl http://localhost:8000/v1/models

# 5. Auto-start com systemd (gerado pelo Podman)
podman generate systemd --name vllm-server --files --new
mkdir -p ~/.config/systemd/user
mv container-vllm-server.service ~/.config/systemd/user/
systemctl --user enable container-vllm-server
systemctl --user start container-vllm-server
loginctl enable-linger "$USER"  # iniciar sem login interativo
```

---

### 2. Ollama

#### Função no projeto
Servidor LLM alternativo ao vLLM, com foco em facilidade de uso e troca rápida de modelos. Usado nas aulas 8 (LAB1 — Self-RAG) e 11 (LAB5 — DSPy). Suporta GPU (NVIDIA/AMD) e CPU (modelos quantizados Q4).

#### Versão estável recomendada
**0.24.0** — verificar `https://github.com/ollama/ollama/releases` antes de instalar.

#### Requisitos mínimos

| Recurso | GPU (recomendado) | CPU (mínimo) |
|---|---|---|
| RAM sistema | 8 GB | 16 GB |
| vCPU | 4 | 8 |
| Disco | 40 GB SSD | 40 GB SSD |
| VRAM GPU | 8 GB (7B Q4) | — |
| NVIDIA Driver | ≥ 550 | — |

> **Conflito de hardware**: Ollama consome VRAM dedicada na inicialização e troca de contexto. Se estiver compartilhando ambiente com o vLLM, certifique-se de baixar/encerrar instâncias simultâneas.

#### Instalação no Ubuntu Server


```bash
# 1. Instalação via script oficial
curl -fsSL https://ollama.com/install.sh | sh

# O script detecta GPU NVIDIA automaticamente e instala drivers CUDA se necessário

# 2. Verificar serviço
systemctl status ollama

# 3. Configurar porta e host (padrão: só localhost)
sudo mkdir -p /etc/systemd/system/ollama.service.d
sudo tee /etc/systemd/system/ollama.service.d/override.conf > /dev/null <<'EOF'
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
EOF

# avaliar a exposição da porta 11343
# uma alternativa é utilizar um proxy reverso com autenticação para expor essa porta


sudo systemctl daemon-reload
sudo systemctl restart ollama

# 4. Baixar modelos
ollama pull qwen2.5:3b       # ~2 GB — GPU 6GB ou CPU 8GB
ollama pull llama3.1:8b      # ~5 GB — requer GPU >= 8GB VRAM

# 5. Testar
ollama run qwen2.5:3b "Explique RAG em uma linha"
```

#### Instalação via container Podman

```bash
# Requer NVIDIA Container Toolkit instalado (ver seção vLLM)

# Criar volume para modelos
podman volume create ollama-models


# Executar container
podman run -d --name ollama \
  --device nvidia.com/gpu=all \
  -p 11434:11434 \
  -v ollama-models:/root/.ollama:z \
  docker.io/ollama/ollama:latest

# Baixar modelo dentro do container
podman exec ollama ollama pull qwen2.5:3b

# Verificar
curl http://localhost:11434/api/tags

# Auto-start
podman generate systemd --name ollama --files --new
mv container-ollama.service ~/.config/systemd/user/
systemctl --user enable container-ollama
systemctl --user start container-ollama
```

---

### 3. RAGatouille / ColBERT

#### Função no projeto
Retriever denso baseado em ColBERT para recuperação multi-vetor de alta precisão. Usado na aula 11 (LAB3 — Advanced Retrieval). Não é um servidor — é uma biblioteca Python chamada diretamente nos notebooks.

#### Versão estável recomendada
**0.0.8.post4** — versão mais recente disponível (PyPI). Verificar compatibilidade com a versão instalada do PyTorch.

#### Requisitos mínimos

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM sistema | 4 GB | 8 GB |
| vCPU | 4 | 4 |
| Disco | 10 GB | 20 GB |
| VRAM GPU | 4 GB (recomendado) | 8 GB |

#### Instalação no Ubuntu Server (VM GPU)

> **Aviso crítico de dependência**: RAGatouille instala pacotes do ecosistema PyTorch. É altamente recomendável utilizar um ambiente virtual (venv) estritamente isolado daquele utilizado pelo vLLM para evitar a sobrescrita das versões compiladas do PyTorch/CUDA requeridas pelos motores de inferência.

```bash
python3.11 -m venv ~/.venvs/ragatouille
source ~/.venvs/ragatouille/bin/activate

pip install --upgrade pip
pip install ragatouille

# Verificar
python -c "from ragatouille import RAGPretrainedModel; print('OK')"
```

---

### 4. LLMLingua

#### Função no projeto
Compressão de contexto para reduzir tokens enviados ao LLM sem perda significativa de informação. Usado na aula 11 (LAB2 — Context Compression). GPU é opcional — CPU funciona para compressão de textos curtos a moderados.

#### Versão estável recomendada
**0.2.2** — verificar `pip index versions llmlingua`.

#### Requisitos mínimos

| Recurso | CPU | GPU (recomendado) |
|---|---|---|
| RAM | 2 GB | 2 GB |
| vCPU | 2 | 2 |
| Disco | 5 GB | 5 GB |
| VRAM | — | 2 GB |

#### Instalação no Ubuntu Server

```bash
# Pode compartilhar venv com o projeto base (caso as dependências do torch permitam)
source ~/.venvs/ragatouille/bin/activate

pip install llmlingua

# Verificar
python -c "from llmlingua import PromptCompressor; print('OK')"
```

---

## Grupo B — Aplicações sem GPU

---

### 5. OpenSearch

#### Função no projeto
Motor de busca distribuído com suporte a BM25, kNN vetorial e busca híbrida. É o vector store principal do projeto, presente nas aulas 1, 3, 4, 7, 9, 11 e 12.

#### Versão estável recomendada
**3.6.0** — série 3.x agora estável (verificar https://opensearch.org/downloads.html). A série 2.x (última: 2.19.0) permanece em suporte LTS para quem precisar de estabilidade máxima ou já tiver índices em 2.x — migração entre major versions exige reindexação.

#### Requisitos mínimos

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 4 GB (JVM heap 2 GB) | 8 GB (JVM heap 4 GB) |
| vCPU | 2 | 4 |
| Disco | 20 GB SSD | 50 GB SSD |
| SO | Ubuntu 22.04+ | — |

> **Regra de ouro do OpenSearch**: JVM heap = metade da RAM disponível, com teto máximo de 32 GB. Com 4 GB de RAM total, a atribuição exata é `-Xms2g -Xmx2g`.

#### Instalação no Ubuntu Server (nativo)

```bash
# 1. Ajuste de kernel obrigatório
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 2. Adicionar repositório APT
curl -o- https://artifacts.opensearch.org/publickeys/opensearch.pgp \
  | sudo gpg --dearmor -o /usr/share/keyrings/opensearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring.gpg] \
  https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" \
  | sudo tee /etc/apt/sources.list.d/opensearch.list

# 3. Instalar
sudo apt-get update
sudo OPENSEARCH_INITIAL_ADMIN_PASSWORD="MinhaS3nh@F0rte!" \
  apt-get install -y opensearch

# 4. Configurar heap no arquivo jvm.options
sudo tee /etc/opensearch/jvm.options.d/heap.options > /dev/null <<'EOF'
-Xms2g
-Xmx2g
EOF

# 5. Habilitar e iniciar
sudo systemctl enable opensearch
sudo systemctl start opensearch

# 6. Verificar
curl -ku admin:"$OPENSEARCH_INITIAL_ADMIN_PASSWORD" https://localhost:9200
```

#### Instalação via container Podman

```bash
# 1. Ajuste de kernel obrigatório no HOST (não no container)
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 2. Criar volume nomeado para persistência dos índices
podman volume create opensearch-data

# 3. Definir senha em variável de ambiente (NUNCA hardcodar em scripts permanentes)
read -s -p "Senha admin OpenSearch: " OPENSEARCH_ADMIN_PASSWORD
export OPENSEARCH_ADMIN_PASSWORD

# 4. Executar container (flags `:z` para compatibilidade com SELinux e volumes mapeados)
podman run -d --name opensearch \
  -p 9200:9200 \
  -p 9600:9600 \
  -e "discovery.type=single-node" \
  -e "OPENSEARCH_INITIAL_ADMIN_PASSWORD=${OPENSEARCH_ADMIN_PASSWORD}" \
  -e "OPENSEARCH_JAVA_OPTS=-Xms2g -Xmx2g" \
  -v opensearch-data:/usr/share/opensearch/data:z \
  docker.io/opensearchproject/opensearch:3.6.0

# 5. Verificar (aguardar ~30s para inicializar a JVM)
curl -ku admin:"${OPENSEARCH_ADMIN_PASSWORD}" https://localhost:9200

# 6. OpenSearch Dashboards (opcional — interface gráfica)
podman run -d --name opensearch-dashboards \
  -p 5601:5601 \
  -e "OPENSEARCH_HOSTS=https://opensearch:9200" \
  -e "OPENSEARCH_USERNAME=admin" \
  -e "OPENSEARCH_PASSWORD=${OPENSEARCH_ADMIN_PASSWORD}" \
  --link opensearch:opensearch \
  docker.io/opensearchproject/opensearch-dashboards:3.6.0

# 7. Auto-start com systemd
podman generate systemd --name opensearch --files --new
mv container-opensearch.service ~/.config/systemd/user/
systemctl --user enable container-opensearch
systemctl --user start container-opensearch
loginctl enable-linger "$USER"
```

**Variáveis de ambiente necessárias nos notebooks:**
```bash
OPENSEARCH_HOST=localhost
OPENSEARCH_PORT=9200
OPENSEARCH_USER=admin
OPENSEARCH_PASS=<senha definida acima>
# Desativar verificação TLS em labs locais
OPENSEARCH_USE_SSL=false
OPENSEARCH_VERIFY_CERTS=false

# Perfil PRODUÇÃO
OPENSEARCH_USE_SSL=true
OPENSEARCH_VERIFY_CERTS=true
```

---

### 6. Langfuse

#### Função no projeto
Plataforma de observabilidade e rastreamento de pipelines RAG. Registra traces, spans, scores e avaliações. Usada nas aulas 1, 3, 4, 5, 7, 10, 11 e 12.

#### Versão estável recomendada
**3.174.1** (self-hosted) — verificar https://github.com/langfuse/langfuse/releases. Atenção: A versão 3 introduziu breaking changes estruturais no schema do banco em relação à v2.x. Jamais integre ou mescle dados entre as duas versões sem rotinas estritas de migração.

#### Requisitos mínimos (stack completa: PG + Redis + Web)

| Componente | RAM | Disco |
|---|---|---|
| PostgreSQL 15 | 512 MB | 5 GB |
| Redis 7 | 256 MB | 1 GB |
| Langfuse server | 1 GB | 2 GB |
| **Total** | **2 GB** | **8 GB** |
| **Recomendado** | **4 GB** | **15 GB** |

#### Instalação via Podman Compose

```bash
# 1. Instalar podman-compose
pip install podman-compose
# ou via gerenciador de pacotes
sudo apt-get install -y podman-compose

# 2. Gerar segredos criptográficos (NUNCA commitar estes valores)
LANGFUSE_DB_PASSWORD=$(openssl rand -hex 16)
LANGFUSE_NEXTAUTH_SECRET=$(openssl rand -hex 32)
LANGFUSE_SALT=$(openssl rand -hex 16)

# 3. Criar arquivo de variáveis de ambiente
cat > ~/langfuse.env <<EOF
LANGFUSE_DB_PASSWORD=${LANGFUSE_DB_PASSWORD}
LANGFUSE_NEXTAUTH_SECRET=${LANGFUSE_NEXTAUTH_SECRET}
LANGFUSE_SALT=${LANGFUSE_SALT}
EOF
chmod 600 ~/langfuse.env

# 4. Criar arquivo de compose
cat > ~/langfuse-compose.yml <<'EOF'
services:
  langfuse-db:
    image: docker.io/library/postgres:15
    restart: unless-stopped
    environment:
      POSTGRES_USER: langfuse
      POSTGRES_PASSWORD: ${LANGFUSE_DB_PASSWORD}
      POSTGRES_DB: langfuse
    volumes:
      - langfuse-db:/var/lib/postgresql/data:z
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U langfuse"]
      interval: 10s
      timeout: 5s
      retries: 5

  langfuse-cache:
    image: docker.io/library/redis:7-alpine
    restart: unless-stopped
    volumes:
      - langfuse-cache:/data:z

  langfuse-server:
    image: docker.io/langfuse/langfuse:3
    restart: unless-stopped
    depends_on:
      langfuse-db:
        condition: service_healthy
      langfuse-cache:
        condition: service_started
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://langfuse:${LANGFUSE_DB_PASSWORD}@langfuse-db:5432/langfuse
      REDIS_URL: redis://langfuse-cache:6379
      NEXTAUTH_URL: http://localhost:3000
      NEXTAUTH_SECRET: ${LANGFUSE_NEXTAUTH_SECRET}
      SALT: ${LANGFUSE_SALT}
      TELEMETRY_ENABLED: "false"
      LANGFUSE_INIT_ORG_ID: mba-rag
      LANGFUSE_INIT_ORG_NAME: "IBMEC MBA RAG"
      LANGFUSE_INIT_PROJECT_ID: aulas
      LANGFUSE_INIT_PROJECT_NAME: "Aulas RAG"

volumes:
  langfuse-db:
  langfuse-cache:
EOF

# 5. Iniciar
podman-compose --env-file ~/langfuse.env -f ~/langfuse-compose.yml up -d

# 6. Verificar
curl http://localhost:3000/api/public/health

# Acesso: http://localhost:3000
# Primeiro acesso: criar conta de administrador via interface web
```

**Variáveis de ambiente necessárias nos notebooks:**
```bash
LANGFUSE_HOST=http://localhost:3000
LANGFUSE_PUBLIC_KEY=<obter na interface web após criar projeto>
LANGFUSE_SECRET_KEY=<obter na interface web após criar projeto>
```

---

### 7. Langflow

#### Função no projeto
Builder visual de pipelines RAG baseado em componentes drag-and-drop. Usado na aula 7 (LAB6).

#### Versão estável recomendada
**1.9.2** — verificar https://github.com/langflow-ai/langflow/releases.

#### Requisitos mínimos

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 2 GB | 4 GB |
| vCPU | 2 | 4 |
| Disco | 5 GB | 10 GB |

#### Instalação no Ubuntu Server (via pip)

```bash
# Instalação direta no host em virtual environment
python3.11 -m venv ~/.venvs/langflow
source ~/.venvs/langflow/bin/activate

pip install --upgrade pip
pip install langflow

# Iniciar
langflow run --host 0.0.0.0 --port 7860

# Acesso: http://localhost:7860
```

#### Instalação via container Podman

```bash
# Criar volume para persistir flows salvos
podman volume create langflow-data

# Gerar senha administrativa antes do podman run
LANGFLOW_ADMIN_PASSWORD=$(openssl rand -hex 16)
export LANGFLOW_ADMIN_PASSWORD

# Executar container
podman run -d --name langflow \
  -p 7860:7860 \
  -v langflow-data:/app/langflow:z \
  -e LANGFLOW_AUTO_LOGIN=false \
  -e LANGFLOW_SUPERUSER=admin \
  -e LANGFLOW_SUPERUSER_PASSWORD="${LANGFLOW_ADMIN_PASSWORD}" \
  docker.io/langflowai/langflow:latest

# Verificar
curl http://localhost:7860/health

# Auto-start
podman generate systemd --name langflow --files --new
mv container-langflow.service ~/.config/systemd/user/
systemctl --user enable container-langflow
systemctl --user start container-langflow
```

---

### 8. FastAPI + Uvicorn

#### Função no projeto
Framework para exposição do pipeline RAG como API REST. Usado na aula 12 (LAB3 — Pipeline em Produção).

#### Versão estável recomendada
FastAPI **0.136.1** | Uvicorn **0.47.0** — verificar PyPI.

#### Requisitos mínimos

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 512 MB | 1 GB |
| vCPU | 1 | 2 |
| Disco | 2 GB | 5 GB |

#### Instalação no Ubuntu Server

```bash
# Certificar-se de executar no venv correto do projeto
source ~/.venvs/projeto/bin/activate

pip install "fastapi[standard]" "uvicorn[standard]"

# Iniciar aplicação (a partir do diretório com main.py)
uvicorn main:app --host 0.0.0.0 --port 8080 --reload

# Para produção (sem reload):
uvicorn main:app --host 0.0.0.0 --port 8080 --workers 2
```

#### Instalação via container Podman

```bash
# Criar Containerfile mínimo (equivalente ao Dockerfile)
cat > Containerfile <<'EOF'
FROM docker.io/library/python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8080
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080", "--workers", "2"]
EOF

# Build
podman build -t fastapi-rag:latest .

# Executar
podman run -d --name fastapi-rag \
  -p 8080:8080 \
  --env-file ~/.config/rag/.env \
  fastapi-rag:latest

# Verificar
curl http://localhost:8080/docs
```

---

### 9. Tesseract-OCR

#### Função no projeto
Motor de OCR para extração de texto de imagens e PDFs escaneados. Usado no material_docling_extra (LAB2) e internamente pelo Docling.

#### Versão estável recomendada
**5.x** — disponível via APT no Ubuntu 22.04+.

#### Requisitos mínimos

| Recurso | Mínimo |
|---|---|
| RAM | 512 MB por processo |
| vCPU | 1 |
| Disco | 1 GB (+ 500 MB por pacote de idioma adicional) |

#### Instalação no Ubuntu Server

```bash
# Instalar Tesseract com suporte a português e inglês
sudo apt-get update
sudo apt-get install -y \
  tesseract-ocr \
  tesseract-ocr-por \
  tesseract-ocr-eng

# Verificar instalação
tesseract --version
tesseract --list-langs

# Testar com imagem
tesseract imagem.png saida -l por
cat saida.txt
```

> **Nota**: Tesseract é instalado diretamente no SO hospedeiro. Não há imagem de container recomendada para uso como biblioteca pelos notebooks — o Docling e o EasyOCR o chamam internamente via subprocess.

---

### 10. Docling

#### Função no projeto
Biblioteca para parsing avançado de documentos (PDF, DOCX, PPTX, HTML). Extrai estrutura semântica, tabelas, figuras e metadados. Usada nas aulas 2 (todos os labs), 11 e 12, e no material_docling_extra.

#### Versão estável recomendada
**2.93.0** — verificar `pip index versions docling`.

#### Requisitos mínimos

| Recurso | Mínimo | Recomendado |
|---|---|---|
| RAM | 1 GB | 2 GB |
| vCPU | 2 | 4 |
| Disco | 5 GB (modelos de layout interno) | 10 GB |
| GPU | Não obrigatória | Acelera OCR neural |

#### Instalação no Ubuntu Server

```bash
# Pré-requisito: Tesseract instalado no SO (ver seção anterior)

# Instalar Docling no escopo do projeto
source ~/.venvs/projeto/bin/activate
pip install docling

# Instalar com suporte a OCR avançado via EasyOCR
pip install "docling[ocr]"

# Verificar
python -c "from docling.document_converter import DocumentConverter; print('OK')"

# Testar conversão
python - <<'EOF'
from docling.document_converter import DocumentConverter
converter = DocumentConverter()
result = converter.convert("documento.pdf")
print(result.document.export_to_markdown()[:500])
EOF
```

---

## Pendências e Riscos Estruturais

- **vm.max_map_count**: o ajuste de kernel para OpenSearch precisa ser feito no host antes de subir o container; falhar nesta etapa resultará em colapso da JVM com erro de mmap.
- **podman-compose vs docker-compose**: a sintaxe é declarativamente idêntica, mas algumas imagens testam apenas flags baseadas no daemon do Docker. Monitore os logs de startup do Langfuse na primeira execução.
- **Volumes Podman rootless**: O uso do flag `:z` foi consolidado nos comandos acima por prover a marcação SELinux adequada de diretórios mapeados, garantindo perms consistentes entre o container rootless e o file system hospedeiro. Em distribuições baseadas unicamente no AppArmor (como no padrão do Ubuntu Server 22.04 LTS), ele será ignorado com segurança sem prejuízos.
- **Tracking de Versão**: Todas as versões indexadas apontam para referências testadas (maio 2026). Dada a alta volatilidade do ecossistema AI, é necessário executar `pip index versions <pacote>` nos deploy scripts antes de promover ao ambiente de produção ou validação final.
- **Restrições Ollama CPU**: Funcionalidade garantida para sandboxes baseados em modelos Q4 (≤ 3B), mas as latências (10–30s de intersecção) o tornam proibitivo para avaliações sistemáticas via frameworks RAGAS em lote.
- 
- # Antes da instalação em cada VM:
pip index versions vllm
pip index versions langflow
pip index versions fastapi
pip index versions uvicorn
pip index versions docling
# e validar releases oficiais de OpenSearch, Langfuse e Ollama


