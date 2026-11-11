# Nexo — Backend

API do **Nexo**, sistema de análise inteligente de reuniões desenvolvido para a TOTVS.  
Processa transcrições, gera insights e permite perguntas sobre as reuniões via RAG.

![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python)
![FastAPI](https://img.shields.io/badge/FastAPI-0.141-green?logo=fastapi)
![LangChain](https://img.shields.io/badge/LangChain-1.3-orange)
![Qdrant](https://img.shields.io/badge/Qdrant-vector%20DB-red)

## Visão geral

O backend é responsável por:

- Autenticação e gestão de usuários
- Cadastro e gerenciamento de reuniões e transcrições
- Pipeline de **RAG** (Retrieval-Augmented Generation) sobre as transcrições
- Chat contextual por reunião usando embeddings + Gemini
- Análise de sentimento e extração de informações

### Stack principal

| Tecnologia | Uso |
|------------|-----|
| **FastAPI** | API REST |
| **LangChain + LangGraph** | Orquestração do RAG |
| **Google Gemini** | Embeddings (`gemini-embedding-001`) + geração |
| **Qdrant** | Banco vetorial (busca por similaridade) |
| **SQLAlchemy** | Persistência relacional |
| **Uvicorn** | Servidor ASGI |

## Como rodar

### Pré-requisitos

- Python 3.11+
- Conta no Google AI Studio (chave do Gemini)
- Qdrant Cloud (ou local)

### 1. Variáveis de ambiente

Crie um arquivo `.env` na raiz:

```env
GOOGLE_API_KEY=sua_chave_gemini
QDRANT_URL=https://seu-cluster.qdrant.io
QDRANT_API_KEY=sua_chave_qdrant

# Opcional (banco)
DATABASE_URL=sqlite:///./mydatabase.db
```

### 2. Instalação

```bash
python -m venv venv
source venv/bin/activate   # Linux/Mac
# ou venv\Scripts\activate  # Windows

pip install -r requirements.txt
```

### 3. Executar

```bash
uvicorn app.main:app --reload
```

A API sobe em `http://127.0.0.1:8000`.  
Documentação interativa: `http://127.0.0.1:8000/docs`

## Endpoints principais

| Método | Rota | Descrição |
|--------|------|-----------|
| `POST` | `/auth/login` | Login |
| `POST` | `/meetings` | Criar reunião |
| `GET` | `/meetings` | Listar reuniões |
| `POST` | `/meetings/{id}/chat` | Chat RAG sobre a reunião |
| `GET` | `/users` | Gestão de usuários |

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

## Estrutura

```
app/
├── main.py                 # Entrada da aplicação
├── database.py
├── langchain/
│   ├── langchain.py
│   └── rag_service.py      # Lógica principal do RAG
├── models/
├── routes/
│   ├── auth_routes.py
│   ├── chat_routes.py
│   ├── meeting_routes.py
│   └── user_routes.py
└── enums/
```

## Observações

- O chat usa **Form Data** (`multipart/form-data`).
- Cada reunião tem filtro por `meeting_id` no Qdrant.
- A collection é criada automaticamente na primeira execução (3072 dimensões, distância cosseno).

---

Desenvolvido como parte do desafio FIAP / TOTVS.
