# Nexo

Plataforma de análise inteligente de reuniões desenvolvida para a TOTVS.  
Processa transcrições, gera insights e permite perguntas contextuais sobre as reuniões usando RAG.

![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python)
![FastAPI](https://img.shields.io/badge/FastAPI-green?logo=fastapi)
![React](https://img.shields.io/badge/React-19-blue?logo=react)
![Vite](https://img.shields.io/badge/Vite-purple?logo=vite)
![LangChain](https://img.shields.io/badge/LangChain-orange)
![Qdrant](https://img.shields.io/badge/Qdrant-red)

## Visão geral

O Nexo é composto por dois serviços:

- **Backend** — API FastAPI com pipeline de RAG (LangChain + Gemini + Qdrant)
- **Frontend** — Dashboard React com visualizações, KPIs e chat contextual por reunião

### Funcionalidades

- Upload e gestão de reuniões e transcrições
- Análise de sentimento e extração de insights
- Chat por reunião usando RAG (pergunta → busca no Qdrant → resposta do Gemini)
- Dashboard com gráficos, alertas de concorrentes e métricas
- Autenticação e controle de usuários

## Stack

| Camada | Tecnologias |
|--------|-------------|
| Frontend | React 19, Vite, Tailwind CSS 4, shadcn/ui, Recharts, React Router |
| Backend | FastAPI, LangChain, LangGraph, SQLAlchemy, Uvicorn |
| IA | Google Gemini (`gemini-embedding-001` + modelos generativos) |
| Vetor | Qdrant (busca por similaridade, 3072 dimensões) |

## Como rodar

### 1. Backend

#### Variáveis de ambiente

Crie um `.env` na pasta do backend:

```env
# Gemini + Qdrant (obrigatórios)
GOOGLE_API_KEY=sua_chave_gemini
QDRANT_URL=https://seu-cluster.qdrant.io
QDRANT_API_KEY=sua_chave_qdrant

# Banco de dados
DATABASE_URL=sqlite:///./mydatabase.db

# Usuário admin (criado automaticamente no startup)
ADMIN_NAME=Admin
ADMIN_EMAIL=admin@exemplo.com
ADMIN_PASSWORD=senha_forte_aqui
```

#### Instalação e execução

```bash
cd backend
python -m venv venv
source venv/bin/activate        # Linux/Mac
# ou venv\Scripts\activate      # Windows

pip install -r requirements.txt
uvicorn app.main:app --reload
```

API disponível em `http://127.0.0.1:8000`  
Documentação: `http://127.0.0.1:8000/docs`

### 2. Frontend

#### Variáveis de ambiente

Crie um `.env` na pasta do frontend:

```env
VITE_API_URL=http://127.0.0.1:8000
```

#### Instalação e execução

```bash
cd frontend
npm install
npm run dev
```

Aplicação disponível em `http://localhost:5173`

### Build de produção (frontend)

```bash
npm run build
npm run preview
```

## Arquitetura do RAG

```
Transcrição
    ↓
Chunking (RecursiveCharacterTextSplitter)
    ↓
Embeddings (gemini-embedding-001 → 3072 dims)
    ↓
Qdrant (collection: meeting_transcriptions)
    ↓
Pergunta do usuário → embedding
    ↓
Busca por similaridade (filtro por meeting_id)
    ↓
Contexto + prompt → Gemini
    ↓
Resposta
```

## Endpoints principais (backend)

| Método | Rota | Descrição |
|--------|------|-----------|
| `POST` | `/auth/login` | Login |
| `POST` | `/meetings` | Criar reunião |
| `GET` | `/meetings` | Listar reuniões |
| `POST` | `/meetings/{id}/chat` | Chat RAG sobre a reunião |

## Estrutura dos repositórios

```
backend/
├── app/
│   ├── main.py
│   ├── langchain/
│   │   └── rag_service.py
│   ├── routes/
│   └── models/
└── requirements.txt

frontend/
├── src/
│   ├── components/dashboard/
│   ├── pages/
│   ├── api/
│   └── router/
└── package.json
```

## Observações

- O chat usa **Form Data** (`multipart/form-data`).
- Cada reunião é filtrada por `meeting_id` no Qdrant.
- A collection no Qdrant é criada automaticamente na primeira execução.
- O usuário admin é criado no startup se as variáveis `ADMIN_*` estiverem definidas.

---

Desenvolvido como parte do desafio FIAP / TOTVS.
