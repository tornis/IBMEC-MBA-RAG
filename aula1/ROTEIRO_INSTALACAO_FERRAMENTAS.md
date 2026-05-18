# ROTEIRO DE INSTALAÇÃO DAS FERRAMENTAS
## MBA RAG & CAG Aplicados a Direito e Segurança Pública — Aula 1

> **Versão:** 4.0 | **Atualizado:** Maio/2026
> **Sistema Operacional:** Windows 11 / macOS 13+ / Ubuntu 22.04 LTS
> **Python:** 3.10+ (3.11 recomendado)
> **Container runtime:** **Podman Desktop** (recomendado) | Docker Desktop (alternativa)
> **Ambiente de Execução:** Jupyter Local + VS Code Extension
> **Referência interativa:** `labs/LAB1_Setup_Ambiente_Completo.ipynb`

---

## ÍNDICE RÁPIDO

| # | Ferramenta | Tempo Estimado | Seção |
|---|-----------|----------------|-------|
| 1 | VS Code + Extensão Jupyter | 10 min | [→ 1](#1-vs-code-e-extensão-jupyter) |
| 2 | Python 3.10+ e venv | 10 min | [→ 2](#2-python-3-10-e-ambiente-virtual) |
| 3 | Ollama — LLM e Embeddings | 20 min | [→ 3](#3-ollama--servidor-de-llm-e-embeddings-local) |
| 4 | Pull dos Modelos Ollama | 15 min | [→ 4](#4-download-dos-modelos-ollama) |
| 5 | Podman Desktop + OpenSearch | 25 min | [→ 5](#5-podman-desktop-e-opensearch) |
| 6 | Dependências Python | 10 min | [→ 6](#6-dependências-python-do-curso) |
| 7 | Langfuse (local + cloud) | 15 min | [→ 7](#7-langfuse) |
| 8 | Variáveis de Ambiente | 5 min | [→ 8](#8-variáveis-de-ambiente) |
| 9 | Configurar Kernel Jupyter no VS Code | 5 min | [→ 9](#9-configurar-kernel-jupyter-no-vs-code) |
| 10 | Validação Final | 10 min | [→ 10](#10-validação-final-do-ambiente) |

**Tempo total estimado:** ~115 minutos (primeira vez) | ~5 minutos (sessões posteriores)

---

## PRÉ-REQUISITOS DE HARDWARE

| Componente | Mínimo | Recomendado |
|-----------|--------|-------------|
| RAM | 16 GB | 32 GB |
| GPU VRAM | 4 GB (qualquer GPU) | 8 GB (para llama3.1:8b) |
| Armazenamento | 30 GB livre | 60 GB livre |
| CPU | 4 cores | 8+ cores |
| Conexão | 10 Mbps | 100+ Mbps (download dos modelos) |

> **Sem GPU?** O Ollama roda em CPU. Os modelos menores (`llama3.2:3b`) respondem em 2-5 segundos em CPUs modernas — adequado para o curso.

---

## 1. VS Code e Extensão Jupyter

O VS Code é o ambiente de desenvolvimento do curso. A extensão Jupyter permite executar notebooks `.ipynb` diretamente no VS Code, com suporte a kernels Python, variáveis interativas e debug.

### 1.1 Instalar VS Code

**Windows:**
```
1. Acesse: https://code.visualstudio.com/download
2. Baixe o instalador "Windows x64 User Installer"
3. Execute e siga o assistente (aceite os padrões)
4. Reinicie o computador após a instalação
```

**macOS:**
```bash
# Via Homebrew (recomendado)
brew install --cask visual-studio-code

# Ou baixe em: https://code.visualstudio.com/download
```

**Ubuntu:**
```bash
# Via snap (mais simples)
sudo snap install code --classic

# Ou via repositório oficial:
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /tmp/packages.microsoft.gpg
sudo install -D -o root -g root -m 644 /tmp/packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
sudo apt-get update
sudo apt-get install -y code
```

### 1.2 Instalar a Extensão Jupyter

No VS Code, abra o painel de extensões com `Ctrl+Shift+X` (Windows/Linux) ou `Cmd+Shift+X` (macOS) e instale:

| ID da Extensão | Nome | Para quê |
|---------------|------|----------|
| `ms-toolsai.jupyter` | Jupyter | Executar notebooks `.ipynb` |
| `ms-python.python` | Python | IntelliSense e linting |
| `ms-toolsai.vscode-jupyter-cell-tags` | Jupyter Cell Tags | Navegação por células |

**Via linha de comando:**
```bash
code --install-extension ms-toolsai.jupyter
code --install-extension ms-python.python
code --install-extension ms-toolsai.vscode-jupyter-cell-tags
```

### 1.3 Verificar a Extensão

```
1. Abra qualquer arquivo .ipynb no VS Code
2. Deve aparecer uma barra superior com "Select Kernel"
3. Clique em "Select Kernel" para escolher o interpretador Python
```

---

## 2. Python 3.10+ e Ambiente Virtual

> **Versão recomendada:** Python 3.11. Versão mínima aceita: **3.10** (suporte a `match/case`, *type hints* modernos e `tomllib`).

### 2.1 Instalar Python 3.10+

**Windows:**
```
1. Acesse: https://www.python.org/downloads/windows/
2. Baixe "Python 3.11.x — Windows installer (64-bit)"
3. Execute o instalador
4. IMPORTANTE: Marque "Add Python to PATH" antes de instalar
5. Clique em "Install Now"
```

Verificar no PowerShell:
```powershell
python --version    # Python 3.11.x
pip --version
```

**macOS:**
```bash
brew install python@3.11
python3.11 --version
```

**Ubuntu:**
```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt-get update
sudo apt-get install -y python3.11 python3.11-venv python3.11-dev
python3.11 --version
```

### 2.2 Criar Ambiente Virtual do Curso

**Windows (PowerShell):**
```powershell
# Cria pasta do projeto
New-Item -ItemType Directory -Force -Path "$HOME\mba-rag"
Set-Location "$HOME\mba-rag"

# Cria ambiente virtual
python -m venv venv_rag

# Ativa o ambiente
.\venv_rag\Scripts\Activate.ps1

# Se der erro de política de execução:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Verificar — deve mostrar o caminho do venv
Get-Command python
```

**Ubuntu / macOS:**
```bash
mkdir -p ~/mba-rag && cd ~/mba-rag
python3.11 -m venv venv_rag
source venv_rag/bin/activate
which python    # deve apontar para ~/mba-rag/venv_rag/bin/python
```

> **Ativar o ambiente:** Sempre que abrir um novo terminal para trabalhar no curso, ative o venv primeiro (`source venv_rag/bin/activate` ou `.\venv_rag\Scripts\Activate.ps1`).

---

## 3. Ollama — Servidor de LLM e Embeddings Local

O **Ollama** é o servidor de LLMs locais utilizado neste curso. Ele baixa modelos em formato GGUF quantizado, serve via API REST (compatível com o padrão OpenAI), e roda em Windows, macOS e Linux — sem necessidade de configuração de CUDA ou drivers complexos.

> **Por que Ollama em vez de vLLM?**
> O vLLM é uma solução de produção excelente, mas exige Linux e GPU NVIDIA com CUDA configurado. O Ollama funciona em qualquer sistema operacional (incluindo Windows e macOS com Apple Silicon) e não exige configuração de drivers. Para o ambiente local de desenvolvimento do curso, o Ollama oferece a mesma API OpenAI-compatível com instalação em minutos.

### 3.1 Instalar Ollama no Windows

```
1. Acesse: https://ollama.com/download/windows
2. Baixe o instalador "OllamaSetup.exe"
3. Execute e siga o assistente
4. Após a instalação, o Ollama inicia automaticamente
   (ícone de llama na bandeja do sistema)
5. Verifique no PowerShell:
```

```powershell
ollama --version
# Saída esperada: ollama version X.Y.Z

# Verifica se o servidor está rodando
Invoke-RestMethod -Uri "http://localhost:11434" -Method GET
# Saída esperada: "Ollama is running"
```

### 3.2 Instalar Ollama no macOS

```bash
# Via Homebrew (recomendado)
brew install ollama

# Ou baixe em: https://ollama.com/download/mac
# Instale o app e abra pelo Launchpad

# Iniciar o servidor (se não iniciou automaticamente)
ollama serve &

# Verificar
curl http://localhost:11434
# Saída: "Ollama is running"
```

### 3.3 Instalar Ollama no Ubuntu / Linux

```bash
# Instalação oficial via script
curl -fsSL https://ollama.com/install.sh | sh

# Verificar instalação
ollama --version

# Iniciar como serviço systemd (recomendado)
sudo systemctl start ollama
sudo systemctl enable ollama    # Inicia automaticamente no boot

# Verificar status
sudo systemctl status ollama

# Testar API
curl http://localhost:11434
# Saída: "Ollama is running"
```

### 3.4 Iniciar o Servidor Ollama Manualmente

```bash
# Se o Ollama não estiver rodando (útil após reinicializações)
ollama serve

# Em background (Linux/macOS)
ollama serve &

# Verificar se está rodando
curl http://localhost:11434/api/tags
# Retorna JSON com lista de modelos instalados
```

---

## 4. Download dos Modelos Ollama

Após instalar o Ollama, baixe os modelos necessários para o curso. Os downloads são feitos uma única vez e ficam em cache local.

### 4.1 Modelos Obrigatórios

```bash
# Modelo de geração de texto — leve e eficiente (2.0 GB)
ollama pull llama3.2:3b

# Modelo de embeddings — padrão do curso (274 MB)
ollama pull nomic-embed-text

# Verificar modelos instalados
ollama list
```

**Saída esperada após o pull:**
```
NAME                    ID              SIZE      MODIFIED
llama3.2:3b             a80c4f17acd5    2.0 GB    Just now
nomic-embed-text        0a109f422b47    274 MB    Just now
```

### 4.2 Modelos Opcionais (Hardware Melhor)

```bash
# LLM mais capaz para tarefas jurídicas complexas (4.9 GB, requer 8GB RAM)
ollama pull llama3.1:8b

# Embedding de alta qualidade, 1024 dims (670 MB)
ollama pull mxbai-embed-large

# BGE-M3 multilíngue (para quem tem espaço) (~570 MB)
ollama pull bge-m3
```

### 4.3 Testar os Modelos

```bash
# Teste rápido de geração de texto
ollama run llama3.2:3b "O que é peculato? Responda em 2 frases."

# Teste de embedding via API REST
curl -s http://localhost:11434/api/embeddings \
  -d '{"model": "nomic-embed-text", "prompt": "crime de furto qualificado"}' | \
  python3 -c "import json,sys; d=json.load(sys.stdin); print(f'Embedding OK: {len(d[\"embedding\"])} dimensoes')"
# Saída esperada: Embedding OK: 768 dimensoes
```

### 4.4 Tabela de Modelos do Curso

| Uso | Modelo | Tamanho | RAM Mínima | Qualidade |
|-----|--------|---------|-----------|-----------|
| LLM (padrão) | `llama3.2:3b` | 2.0 GB | 8 GB | Boa |
| LLM (avançado) | `llama3.1:8b` | 4.9 GB | 16 GB | Excelente |
| Embedding (padrão) | `nomic-embed-text` | 274 MB | 4 GB | Boa (768 dims) |
| Embedding (avançado) | `mxbai-embed-large` | 670 MB | 8 GB | Excelente (1024 dims) |
| Embedding (multilíngue) | `bge-m3` | ~570 MB | 8 GB | Estado da arte |

---

## 5. Podman Desktop e OpenSearch

O **Podman Desktop** é o *runtime* de contêineres recomendado para esta disciplina por dois motivos principais: **(i)** é *daemonless* e *rootless* por padrão (menor superfície de ataque), e **(ii)** é Apache-2.0 sem restrições de licenciamento comercial — adequado para órgãos públicos. O Docker Desktop é uma **alternativa** funcionalmente equivalente para alunos que já o tenham instalado.

### 5.1 Instalar Podman Desktop (Recomendado)

**Windows:**
```powershell
# Pré-requisito: WSL 2 atualizado (PowerShell como Administrador)
wsl --status
wsl --update
wsl --set-default-version 2

# Opção A — winget (recomendada)
winget install -e --id RedHat.Podman-Desktop

# Opção B — Instalador oficial
# 1. Acesse: https://podman-desktop.io/downloads/windows
# 2. Baixe podman-desktop-x.y.z-setup-x64.exe
# 3. Execute o instalador

# Inicializar a máquina Podman (uma única vez)
podman machine init --cpus 4 --memory 6144 --disk-size 60
podman machine start

# Verificar
podman version
```

**macOS:**
```bash
brew install podman podman-desktop
podman machine init --cpus 4 --memory 6144
podman machine start
podman version
```

**Ubuntu:**
```bash
# Configurar vm.max_map_count (obrigatório para OpenSearch)
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
sudo sysctl -w vm.max_map_count=262144

# Instalar Podman e podman-compose
sudo apt-get update
sudo apt-get install -y podman
podman version
```

### 5.1b Alternativa: Docker Desktop

Se você já tem **Docker Desktop** instalado e prefere mantê-lo, basta substituir `podman` por `docker` nos comandos a seguir — o `docker-compose.yml` é idêntico. No Ubuntu, instale `docker.io docker-compose-plugin` em vez de `podman`.

### 5.2 Instalar `podman-compose`

Com o `venv_rag` ativo:

```bash
pip install podman-compose==1.2.0
podman-compose --version
```

### 5.3 Subir o OpenSearch com `podman-compose`

Crie o arquivo `docker-compose.yml` no diretório `~/mba-rag/infra/opensearch/`:

```bash
mkdir -p ~/mba-rag/infra/opensearch
```

Conteúdo do `docker-compose.yml`:

```yaml
version: "3.8"
services:
  opensearch-node1:
    image: docker.io/opensearchproject/opensearch:3.0.0
    container_name: opensearch-node1
    environment:
      - cluster.name=opensearch-cluster-rag
      - node.name=opensearch-node1
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - OPENSEARCH_JAVA_OPTS=-Xms2g -Xmx2g
      - DISABLE_SECURITY_PLUGIN=true
      - DISABLE_INSTALL_DEMO_CONFIG=true
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - opensearch-data1:/usr/share/opensearch/data
    ports:
      - "9200:9200"
      - "9600:9600"
    networks:
      - rag-network
    healthcheck:
      test: ["CMD-SHELL", "curl -sf http://localhost:9200/_cluster/health || exit 1"]
      interval: 15s
      timeout: 10s
      retries: 10

  opensearch-dashboards:
    image: docker.io/opensearchproject/opensearch-dashboards:3.0.0
    container_name: opensearch-dashboards
    ports:
      - "5601:5601"
    environment:
      OPENSEARCH_HOSTS: '["http://opensearch-node1:9200"]'
      DISABLE_SECURITY_DASHBOARDS_PLUGIN: "true"
    depends_on:
      opensearch-node1:
        condition: service_healthy
    networks:
      - rag-network

volumes:
  opensearch-data1:

networks:
  rag-network:
    driver: bridge
```

**Iniciar o OpenSearch:**
```bash
cd ~/mba-rag/infra/opensearch
podman-compose up -d        # ou: docker compose up -d (se usar Docker)

# Aguardar inicialização (~60-90 segundos)
sleep 75

# Verificar
curl -s http://localhost:9200/_cluster/health | python3 -m json.tool
```

**Acesso via navegador:** http://localhost:5601 (OpenSearch Dashboards)

---

## 6. Dependências Python do Curso

Com o ambiente virtual ativado, instale as bibliotecas:

```bash
# Ativar ambiente virtual (se ainda não estiver ativo)
# Windows: .\venv_rag\Scripts\Activate.ps1
# Linux/macOS: source ~/mba-rag/venv_rag/bin/activate

# Atualizar pip
pip install --upgrade pip

# Instalar dependências
pip install \
  sentence-transformers==3.0.1 \
  FlagEmbedding==1.2.11 \
  faiss-cpu==1.8.0 \
  opensearch-py==2.7.1 \
  langchain==0.3.0 \
  langchain-community==0.3.0 \
  langchain-openai==0.2.0 \
  openai>=1.40.0 \
  langfuse==2.36.1 \
  umap-learn==0.5.6 \
  matplotlib==3.9.2 \
  plotly==5.24.0 \
  seaborn==0.13.2 \
  pandas==2.2.2 \
  numpy==1.26.4 \
  tqdm==4.66.5 \
  python-dotenv==1.0.1 \
  requests==2.32.3 \
  psutil==6.0.0 \
  nltk==3.9.1 \
  scikit-learn==1.5.2 \
  ipykernel==6.29.5

# Registrar kernel Jupyter
python -m ipykernel install --user \
  --name=venv_rag \
  --display-name="MBA RAG (Python 3.11)"
```

---

## 7. Langfuse

Langfuse é a plataforma de observabilidade para rastreamento de chamadas LLM. Para esta disciplina, recomenda-se a **versão local (self-hosted)** por aderência ao sigilo funcional e à LGPD.

### 7.1 Opção A — Langfuse Local via `podman-compose` (Recomendado)

```bash
# Diretório dedicado
mkdir -p ~/mba-rag/infra/langfuse
cd ~/mba-rag/infra/langfuse

# Baixar o docker-compose.yml oficial do Langfuse v3
curl -L -o docker-compose.yml \
  https://raw.githubusercontent.com/langfuse/langfuse/main/docker-compose.yml

# Criar .env com secrets (Linux/macOS)
cat > .env <<EOF
NEXTAUTH_SECRET=$(openssl rand -hex 32)
SALT=$(openssl rand -hex 32)
ENCRYPTION_KEY=$(openssl rand -hex 32)
NEXTAUTH_URL=http://localhost:3000
DATABASE_URL=postgresql://postgres:postgres@postgres:5432/postgres
CLICKHOUSE_URL=http://clickhouse:8123
CLICKHOUSE_MIGRATION_URL=clickhouse://clickhouse:9000
TELEMETRY_ENABLED=false
EOF

# Subir o stack (Postgres + Clickhouse + Redis + MinIO + Web + Worker)
podman-compose up -d

# Aguardar ~90-120 segundos para o stack estabilizar
# Acessar: http://localhost:3000
```

No primeiro acesso a `http://localhost:3000`:

```
1. Crie sua conta (e-mail + senha gravados no Postgres local).
2. Crie a Organização: "mba-rag-direito".
3. Crie o Projeto: "aula1-fundamentos".
4. Vá em Settings → API Keys → Create new API key.
5. Anote (a Secret Key só é exibida UMA vez):
   - Public Key (pk-lf-...)
   - Secret Key (sk-lf-...)
6. Configure no ~/mba-rag/.env:
     LANGFUSE_HOST=http://localhost:3000
     LANGFUSE_PUBLIC_KEY=pk-lf-...
     LANGFUSE_SECRET_KEY=sk-lf-...
```

> **Windows (PowerShell)** — gerar secrets equivalentes ao `openssl rand -hex 32`:
> ```powershell
> -join ((1..32) | ForEach-Object { '{0:x2}' -f (Get-Random -Maximum 256) })
> ```

### 7.2 Opção B — Langfuse Cloud (Alternativa Rápida)

Para experimentação sem dados sensíveis, é possível usar a versão em nuvem:

```
1. Acesse: https://cloud.langfuse.com
2. Crie uma conta gratuita
3. Crie um novo projeto: "mba-rag-direito"
4. Settings → API Keys → Create new API Key
5. Configure no ~/mba-rag/.env:
     LANGFUSE_HOST=https://cloud.langfuse.com
     LANGFUSE_PUBLIC_KEY=pk-lf-...
     LANGFUSE_SECRET_KEY=sk-lf-...
```

> **Aviso:** **não** envie ao Langfuse Cloud trechos de processos sob segredo de justiça, dados pessoais sensíveis (LGPD art. 5º, II) ou conteúdo classificado. Para esses casos, use exclusivamente a opção local.

---

## 8. Variáveis de Ambiente

### 8.1 Criar o Arquivo .env

```bash
# Ubuntu/macOS
cat > ~/mba-rag/.env << 'EOF'
# ─── OpenSearch ──────────────────────────────────────────────
OPENSEARCH_URL=http://localhost:9200
OPENSEARCH_HOST=localhost
OPENSEARCH_PORT=9200

# ─── Ollama ──────────────────────────────────────────────────
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_LLM_MODEL=llama3.2:3b
OLLAMA_EMBED_MODEL=nomic-embed-text

# ─── LangFuse ────────────────────────────────────────────────
LANGFUSE_PUBLIC_KEY=pk-lf-SEU_TOKEN_AQUI
LANGFUSE_SECRET_KEY=sk-lf-SEU_TOKEN_AQUI
LANGFUSE_HOST=https://cloud.langfuse.com

# ─── Configurações do Curso ──────────────────────────────────
CHUNK_SIZE=512
CHUNK_OVERLAP=50
TOP_K_RETRIEVAL=5
TEMPERATURE_DEFAULT=0.2
MAX_TOKENS_DEFAULT=512
EOF

chmod 600 ~/mba-rag/.env
echo "Edite ~/mba-rag/.env e preencha LANGFUSE_PUBLIC_KEY e LANGFUSE_SECRET_KEY"
```

**Windows (PowerShell):**
```powershell
$envContent = @"
OPENSEARCH_URL=http://localhost:9200
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_LLM_MODEL=llama3.2:3b
OLLAMA_EMBED_MODEL=nomic-embed-text
LANGFUSE_PUBLIC_KEY=pk-lf-SEU_TOKEN_AQUI
LANGFUSE_SECRET_KEY=sk-lf-SEU_TOKEN_AQUI
LANGFUSE_HOST=https://cloud.langfuse.com
"@
$envContent | Out-File -FilePath "$HOME\mba-rag\.env" -Encoding utf8
```

### 8.2 Adicionar ao .gitignore

```bash
cat > ~/mba-rag/.gitignore << 'EOF'
.env
*.env
venv_rag/
__pycache__/
*.pyc
models/
*.gguf
EOF
```

---

## 9. Configurar Kernel Jupyter no VS Code

### 9.1 Selecionar o Kernel no VS Code

```
1. Abra qualquer notebook .ipynb no VS Code
2. Clique em "Select Kernel" no canto superior direito
3. Escolha "Python Environments..."
4. Selecione "MBA RAG (Python 3.11)" (o kernel registrado no passo 6)
   — ou selecione diretamente o interpretador em venv_rag/
```

### 9.2 Configurar o VS Code para Encontrar o venv

Crie um arquivo `.vscode/settings.json` no diretório do curso:

```json
{
  "python.defaultInterpreterPath": "${env:HOME}/mba-rag/venv_rag/bin/python",
  "jupyter.kernelSpecProvider.provider": "localKernelProvider",
  "jupyter.notebookFileRoot": "${workspaceFolder}"
}
```

**Windows:**
```json
{
  "python.defaultInterpreterPath": "${env:USERPROFILE}\\mba-rag\\venv_rag\\Scripts\\python.exe",
  "jupyter.kernelSpecProvider.provider": "localKernelProvider",
  "jupyter.notebookFileRoot": "${workspaceFolder}"
}
```

### 9.3 Verificar o Kernel Ativo

No VS Code, com um notebook aberto, execute na primeira célula:

```python
import sys
print(sys.executable)
# Deve mostrar o caminho do venv_rag
```

---

## 10. Validação Final do Ambiente

Execute o script abaixo para confirmar que tudo está funcionando:

```python
#!/usr/bin/env python3
"""
Validação do Ambiente — MBA RAG & CAG Aula 1
Execute com: python validar_ambiente.py
"""
import sys, requests
from datetime import datetime

resultados = []

def teste(nome, fn, opcional=False):
    try:
        fn()
        resultados.append(('OK', nome, ''))
    except Exception as e:
        if opcional:
            resultados.append(('WARN', f'{nome} (opcional)', str(e)[:80]))
        else:
            resultados.append(('FAIL', nome, str(e)[:80]))

print('=' * 65)
print(f'VALIDACAO DO AMBIENTE MBA RAG — {datetime.now().strftime("%d/%m/%Y %H:%M")}')
print('=' * 65)

# Python 3.11+
def check_python():
    assert sys.version_info >= (3, 11), f"Python 3.11+ necessario, encontrado: {sys.version}"
teste("Python 3.11+", check_python)

# Ollama rodando
def check_ollama():
    r = requests.get("http://localhost:11434/api/tags", timeout=5)
    assert r.status_code == 200
    modelos = [m["name"] for m in r.json().get("models", [])]
    assert len(modelos) > 0, "Nenhum modelo instalado. Execute: ollama pull llama3.2:3b"
    print(f"\n     Modelos: {modelos}", end="")
teste("Ollama (servidor + modelos)", check_ollama)

# Modelo LLM via Ollama
def check_llm():
    r = requests.post(
        "http://localhost:11434/api/generate",
        json={"model": "llama3.2:3b", "prompt": "OK", "stream": False},
        timeout=60
    )
    assert r.status_code == 200
    assert len(r.json()["response"]) > 0
teste("Ollama LLM (llama3.2:3b geracao)", check_llm)

# Embedding via Ollama
def check_embed():
    r = requests.post(
        "http://localhost:11434/api/embeddings",
        json={"model": "nomic-embed-text", "prompt": "crime de furto"},
        timeout=30
    )
    assert r.status_code == 200
    dims = len(r.json()["embedding"])
    assert dims > 0
    print(f"\n     Dimensoes: {dims}", end="")
teste("Ollama Embeddings (nomic-embed-text)", check_embed)

# OpenSearch
def check_opensearch():
    r = requests.get("http://localhost:9200", timeout=5)
    assert r.status_code == 200
    assert "version" in r.json()
teste("OpenSearch 3.x (servidor local)", check_opensearch)

# Bibliotecas Python
def check_libs():
    import sentence_transformers, faiss, opensearchpy, langchain, langfuse, umap
teste("Bibliotecas Python (sentence-transformers, faiss, langchain)", check_libs)

# LangFuse
def check_langfuse():
    import os
    from langfuse import Langfuse
    pk = os.environ.get("LANGFUSE_PUBLIC_KEY", "")
    sk = os.environ.get("LANGFUSE_SECRET_KEY", "")
    assert pk and not pk.endswith("_AQUI"), "Configure LANGFUSE_PUBLIC_KEY no .env"
    lf = Langfuse(public_key=pk, secret_key=sk)
    lf.auth_check()
teste("LangFuse (observabilidade)", check_langfuse)

# Kernel Jupyter
def check_jupyter():
    import ipykernel
    result = __import__("subprocess").run(
        [sys.executable, "-m", "jupyter", "kernelspec", "list"],
        capture_output=True, text=True
    )
    assert "venv_rag" in result.stdout or "mba-rag" in result.stdout.lower(), \
        "Kernel 'venv_rag' nao encontrado. Execute: python -m ipykernel install --user --name=venv_rag"
teste("Kernel Jupyter registrado (venv_rag)", check_jupyter)

# Imprimir resultados
print()
for status, nome, detalhe in resultados:
    icone = {"OK": "[OK]", "WARN": "[AV]", "FAIL": "[XX]"}[status]
    print(f"  {icone}  {nome}")
    if detalhe and status == "FAIL":
        print(f"       -> {detalhe}")

ok = sum(1 for s, _, _ in resultados if s == "OK")
total = len(resultados)
print()
print(f"Resultado: {ok}/{total} verificacoes aprovadas")

falhas = [n for s, n, _ in resultados if s == "FAIL"]
if not falhas:
    print()
    print("AMBIENTE COMPLETAMENTE VALIDADO!")
    print("Voce esta pronto para os laboratorios da Aula 1.")
else:
    print()
    print("Falhas detectadas — resolva antes de continuar:")
    for f in falhas:
        print(f"  - {f}")
```

**Saída esperada:**
```
==================================================================
VALIDACAO DO AMBIENTE MBA RAG — 14/05/2026 10:00
==================================================================

  [OK]  Python 3.11+
  [OK]  Ollama (servidor + modelos)
        Modelos: ['llama3.2:3b', 'nomic-embed-text']
  [OK]  Ollama LLM (llama3.2:3b geracao)
  [OK]  Ollama Embeddings (nomic-embed-text)
        Dimensoes: 768
  [OK]  OpenSearch 3.x (servidor local)
  [OK]  Bibliotecas Python (sentence-transformers, faiss, langchain)
  [OK]  LangFuse (observabilidade)
  [OK]  Kernel Jupyter registrado (venv_rag)

Resultado: 8/8 verificacoes aprovadas

AMBIENTE COMPLETAMENTE VALIDADO!
Voce esta pronto para os laboratorios da Aula 1.
```

---

## TROUBLESHOOTING

| Problema | Causa | Solução |
|----------|-------|---------|
| `ollama: command not found` | Ollama não instalado | Baixe em https://ollama.com/download |
| Ollama não responde em localhost:11434 | Servidor não iniciado | Execute `ollama serve` no terminal |
| `Error: model not found` | Modelo não baixado | `ollama pull llama3.2:3b` |
| OpenSearch não inicia (Ubuntu) | `vm.max_map_count` baixo | `sudo sysctl -w vm.max_map_count=262144` |
| `podman: command not found` | Podman Desktop não inicializou | Abra o app pelo Menu Iniciar e aguarde "Podman machine is running" |
| `podman machine start` falha no Windows | WSL 2 desatualizado | `wsl --update` no PowerShell admin |
| `podman-compose: command not found` | Não instalado no venv | `pip install podman-compose==1.2.0` com o venv ativo |
| Podman Machine sem memória | VM com defaults baixos | `podman machine stop && podman machine set --memory 6144 --cpus 4` |
| Docker Desktop sem memória (se usar Docker) | Limite padrão muito baixo | Docker Desktop → Settings → Resources → Memory: 6 GB |
| Langfuse: `Cannot connect to clickhouse` | Serviço ainda inicializando | Aguarde 60s — Clickhouse demora para subir |
| Portas em uso (`bind: address already in use`) | Outro serviço usando 9200/3000/5601 | `netstat -ano \| findstr :9200` (Win) ou `lsof -i :9200` (Linux) |
| `Activate.ps1 não pode ser carregado` (Windows) | Política de execução restrita | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` |
| Kernel `venv_rag` não aparece no VS Code | Não registrado | `python -m ipykernel install --user --name=venv_rag --display-name="MBA RAG (Python 3.11)"` |
| VS Code não encontra o Python do venv | Path incorreto | Abra o VS Code na pasta do curso e reselecione o kernel |
| LangFuse `401 Unauthorized` | Chaves expiradas | Regenere em cloud.langfuse.com → Settings → API Keys |
| `CUDA out of memory` (Ollama) | Modelo muito grande para GPU | Use `llama3.2:3b` ou adicione `OLLAMA_GPU_LAYERS=0` nas variáveis de ambiente |

---

## ESTRUTURA DE DIRETÓRIOS RECOMENDADA

```
~/mba-rag/
├── .env                              # Credenciais do curso (NUNCA no git!)
├── .gitignore
├── infra/
│   ├── opensearch/
│   │   └── docker-compose.yml        # OpenSearch + Dashboards
│   └── langfuse/
│       ├── docker-compose.yml        # Langfuse v3 (Web, Worker, Postgres, Clickhouse, Redis, MinIO)
│       └── .env                      # Secrets do Langfuse local
├── venv_rag/                         # Ambiente virtual Python 3.11
├── .vscode/
│   └── settings.json                 # Configurações do VS Code
├── aula1/
│   ├── teoria/
│   │   └── AULA1_TEORIA.md
│   ├── labs/
│   │   ├── LAB1_Setup_Ambiente_Completo.ipynb   # Setup integrado (Python, Podman, OpenSearch, Ollama, Langfuse)
│   │   ├── LAB2_Embeddings_BGE_M3_UMAP.ipynb     # Embeddings + UMAP
│   │   └── _deprecated/                          # Labs antigos (LAB1-4 originais, consulta histórica)
│   ├── exemplos/
│   │   └── EXEMPLO_Pipeline_RAG_Minimo.ipynb
│   └── datasets/
│       └── corpus_juridico_aula1.json
└── aula2/
    └── ...
```

---

*Roteiro de Instalação v4.0 — MBA RAG & CAG Aplicados a Direito e Segurança Pública*
*Adoção de Podman Desktop e Langfuse self-hosted em conformidade com LGPD e sigilo funcional.*
*Versão interativa (recomendada): `labs/LAB1_Setup_Ambiente_Completo.ipynb`.*
