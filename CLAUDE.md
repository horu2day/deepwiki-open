# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeepWiki-Open is an AI-powered repository documentation generator that automatically creates interactive wikis with diagrams for GitHub, GitLab, and BitBucket repositories. It consists of:

- **Frontend**: Next.js 15 app (React 19, TypeScript, Tailwind CSS 4)
- **Backend**: FastAPI Python API with RAG (Retrieval Augmented Generation)
- **AI Integration**: Multi-provider support (Google Gemini, OpenAI, OpenRouter, Azure OpenAI, Ollama, AWS Bedrock)
- **Storage**: Local storage using adalflow framework (~/.adalflow/)

## Development Setup

### Initial Setup

```bash
# Install Python dependencies (from project root)
python -m pip install poetry==1.8.2 && poetry install -C api

# Install JavaScript dependencies
npm install
# or
yarn install

# Create .env file with required API keys
# See README.md for available environment variables
```

### Running the Application

```bash
# Start backend API (from project root)
python -m api.main
# API runs on http://localhost:8001 (configurable via PORT env var)

# Start frontend (in separate terminal)
npm run dev
# Frontend runs on http://localhost:3000
```

### Docker

```bash
# Build and run with Docker Compose
docker-compose up

# Or build locally
docker build -t deepwiki-open .
```

## Testing

```bash
# Run all tests
python tests/run_tests.py

# Run specific test categories
python tests/run_tests.py --unit         # Unit tests only
python tests/run_tests.py --integration  # Integration tests only
python tests/run_tests.py --api          # API tests only

# Run individual test files
python tests/unit/test_google_embedder.py
python tests/integration/test_full_integration.py
python tests/api/test_api.py
```

**Note**: API tests require the API server to be running on port 8001.

## Architecture

### Backend Architecture

The backend follows a provider-based model architecture:

1. **Configuration System** ([api/config.py](api/config.py))
   - JSON-based configuration files in [api/config/](api/config/)
   - `generator.json`: Text generation model configurations
   - `embedder.json`: Embedding model and retriever settings
   - `repo.json`: Repository filtering and processing rules
   - `lang.json`: Internationalization settings
   - Environment variable substitution with `${ENV_VAR}` placeholders
   - Custom config directory via `DEEPWIKI_CONFIG_DIR` env var

2. **Model Providers** (configured via [api/config/generator.json](api/config/generator.json))
   - **Google**: GoogleGenAIClient (default: gemini-2.5-flash)
   - **OpenAI**: OpenAIClient (supports custom base_url)
   - **OpenRouter**: OpenRouterClient (access to multiple models)
   - **Azure**: AzureAIClient (Azure OpenAI)
   - **Ollama**: OllamaClient (local models)
   - **Bedrock**: BedrockClient (AWS Bedrock)
   - **Dashscope**: DashscopeClient (Alibaba Qwen)

3. **Embedder System** ([api/tools/embedder.py](api/tools/embedder.py))
   - Switchable via `DEEPWIKI_EMBEDDER_TYPE` env var (openai|google|ollama)
   - **OpenAI**: text-embedding-3-small (default)
   - **Google**: text-embedding-004 (via GoogleEmbedderClient)
   - **Ollama**: Local embedding models
   - Configured in [api/config/embedder.json](api/config/embedder.json)

4. **Data Pipeline** ([api/data_pipeline.py](api/data_pipeline.py))
   - `download_repo()`: Clones repos with token auth support
   - `DatabaseManager`: Manages FAISS vector stores and document splitting
   - `process_documents()`: Creates embeddings with token limits
   - Stores data in `~/.adalflow/` (repos, databases, wikicache)

5. **RAG Implementation** ([api/rag.py](api/rag.py))
   - `CustomConversation`: Fixed conversation history management
   - `Memory`: Dialog turn tracking
   - `RAGAgent`: Retrieval-augmented generation with streaming
   - Uses FAISSRetriever for semantic search

6. **API Layer** ([api/api.py](api/api.py))
   - FastAPI with CORS enabled for all origins
   - Streaming WebSocket and SSE endpoints
   - Wiki generation and caching system
   - Authorization mode support (`DEEPWIKI_AUTH_MODE`, `DEEPWIKI_AUTH_CODE`)

### Frontend Architecture

1. **App Router Structure** ([src/app/](src/app/))
   - Dynamic routes: `[owner]/[repo]/page.tsx` for wiki display
   - API routes: `api/chat/stream/`, `api/models/config/`, etc.
   - Main page: [src/app/page.tsx](src/app/page.tsx) - repository input form

2. **Key Components**
   - [src/components/Mermaid.tsx](src/components/Mermaid.tsx): Renders Mermaid diagrams with auto-fix
   - [src/components/Ask.tsx](src/components/Ask.tsx): Chat interface with RAG
   - [src/components/ConfigurationModal.tsx](src/components/ConfigurationModal.tsx): Model/provider selection
   - [src/components/Markdown.tsx](src/components/Markdown.tsx): Markdown rendering with syntax highlighting

3. **State Management**
   - Local storage for repo configurations (`deepwikiRepoConfigCache`)
   - Language context via [src/contexts/LanguageContext.tsx](src/contexts/LanguageContext.tsx)
   - Custom hooks in [src/hooks/](src/hooks/)

## Important Development Patterns

### Adding a New Model Provider

1. Create client class in [api/](api/) (e.g., `api/newprovider_client.py`)
2. Add to `CLIENT_CLASSES` mapping in [api/config.py](api/config.py)
3. Update [api/config/generator.json](api/config/generator.json) with provider config
4. Frontend model selection automatically reflects JSON changes

### Working with Embeddings

- Token limits: MAX_EMBEDDING_TOKENS = 8192 (OpenAI), varies by provider
- Text splitting configured in [api/config/embedder.json](api/config/embedder.json)
- Switch embedders via `DEEPWIKI_EMBEDDER_TYPE=google|openai|ollama`
- Check embedder type with `get_embedder_type()` in [api/config.py](api/config.py)

### Repository Processing

Default exclusions defined in [api/config.py](api/config.py):
- Directories: `node_modules/`, `.git/`, `venv/`, `__pycache__/`, etc.
- Files: Lock files, config files, binaries, etc.
- Customize via [api/config/repo.json](api/config/repo.json) or UI filters

### Logging

Configured in [api/logging_config.py](api/logging_config.py):
- Set `LOG_LEVEL` env var (DEBUG|INFO|WARNING|ERROR|CRITICAL)
- Set `LOG_FILE_PATH` for custom log file (default: `api/logs/application.log`)
- Logs persisted via Docker volume mount in [docker-compose.yml](docker-compose.yml)

### Internationalization

- Supported languages in [api/config/lang.json](api/config/lang.json)
- Frontend translations in [messages/](messages/) directory
- Default language: English (en)

## Common Development Tasks

### Running Backend in Development Mode

```bash
# Auto-reload enabled (NODE_ENV != production)
python -m api.main
```

The development server uses watchfiles to monitor changes, excluding `api/logs/` directory.

### Building for Production

```bash
# Frontend build
npm run build
npm start

# Docker production build
docker build -t deepwiki-open .
docker run -p 8001:8001 -p 3000:3000 \
  -e GOOGLE_API_KEY=xxx \
  -e OPENAI_API_KEY=xxx \
  -v ~/.adalflow:/root/.adalflow \
  deepwiki-open
```

### Clearing Cache

Cached data stored in:
- `~/.adalflow/repos/` - Cloned repositories
- `~/.adalflow/databases/` - FAISS embeddings
- `~/.adalflow/wikicache/` - Generated wiki content

Delete specific cache via API endpoint or manually remove directories.

### Working with Private Repositories

The system supports personal access tokens for private repos:
- GitHub: Classic tokens or fine-grained PATs with repo access
- GitLab: Personal access tokens with read_repository scope
- Bitbucket: App passwords with repository read permission

Token handling in [api/data_pipeline.py](api/data_pipeline.py) `download_repo()` function.

### Adding Custom Model Parameters

Edit [api/config/generator.json](api/config/generator.json):
```json
{
  "providers": {
    "your_provider": {
      "default_model": "model-name",
      "models": {
        "model-name": {
          "temperature": 0.7,
          "top_p": 0.9,
          "max_tokens": 2048
        }
      }
    }
  }
}
```

## Key Files Reference

### Backend Core
- [api/main.py](api/main.py) - Entry point, environment setup
- [api/api.py](api/api.py) - FastAPI app, endpoints, models
- [api/config.py](api/config.py) - Configuration loader, provider mapping
- [api/rag.py](api/rag.py) - RAG implementation with conversation memory
- [api/data_pipeline.py](api/data_pipeline.py) - Repository processing, embeddings
- [api/prompts.py](api/prompts.py) - System prompts and templates

### Backend Client Implementations
- [api/openai_client.py](api/openai_client.py) - OpenAI API wrapper
- [api/openrouter_client.py](api/openrouter_client.py) - OpenRouter integration
- [api/google_embedder_client.py](api/google_embedder_client.py) - Google embeddings
- [api/azureai_client.py](api/azureai_client.py) - Azure OpenAI
- [api/bedrock_client.py](api/bedrock_client.py) - AWS Bedrock
- [api/ollama_patch.py](api/ollama_patch.py) - Ollama document processing

### Frontend Core
- [src/app/page.tsx](src/app/page.tsx) - Main repository input page
- [src/app/[owner]/[repo]/page.tsx](src/app/[owner]/[repo]/page.tsx) - Wiki display
- [src/components/Ask.tsx](src/components/Ask.tsx) - Chat/RAG interface
- [src/components/Mermaid.tsx](src/components/Mermaid.tsx) - Diagram renderer

### Configuration
- [api/config/generator.json](api/config/generator.json) - Model providers and defaults
- [api/config/embedder.json](api/config/embedder.json) - Embedding and retrieval config
- [api/config/repo.json](api/config/repo.json) - File filters and repo settings
- [api/config/lang.json](api/config/lang.json) - Language configurations

### Docker
- [Dockerfile](Dockerfile) - Multi-stage build for production
- [docker-compose.yml](docker-compose.yml) - Service orchestration with health checks

## Platform Compatibility

- Windows: Native support (path handling in [api/data_pipeline.py](api/data_pipeline.py))
- Linux/macOS: Full support
- Docker: Cross-platform containerized deployment

## Security Notes

- API keys stored in environment variables only (never in code)
- Authorization mode via `DEEPWIKI_AUTH_MODE` for controlled access
- Log file path validated to prevent path traversal attacks
- Private repo tokens authenticated via Git credential helpers
- Self-signed certificate support via Docker build arg `CUSTOM_CERT_DIR`
