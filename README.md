# Parent Dashboard Backend

Backend API for the Parent Dashboard AI Assistant component. This backend uses RAG (Retrieval-Augmented Generation) with Chroma DB for vector storage and Groq LLM for generating parent-friendly responses about speech therapy and child development.

## Architecture

- **Framework**: FastAPI
- **AI Approach**: RAG (Retrieval-Augmented Generation)
- **Vector Database**: Chroma DB (local persistent storage)
- **LLM Provider**: Groq
- **Embeddings**: Sentence Transformers (all-MiniLM-L6-v2)

## Project Structure

```
backend/
├── parentdashboard/
│   ├── api/
│   │   └── routes.py              # API endpoints
│   ├── rag/
│   │   ├── loader.py              # PDF loading
│   │   ├── chunker.py             # Text chunking
│   │   ├── embeddings.py          # Embedding generation
│   │   ├── vector_store.py        # Chroma DB management
│   │   ├── retriever.py           # Document retrieval
│   │   └── rag_pipeline.py        # End-to-end RAG pipeline
│   ├── ai/
│   │   ├── llm.py                 # Groq LLM integration
│   │   └── prompt.py              # Prompt templates
│   ├── data/
│   │   └── pdfs/                  # Knowledge base PDFs (add your PDFs here)
│   ├── services/
│   │   └── qa_service.py          # QA service (RAG + LLM)
│   ├── schemas/
│   │   ├── request.py             # Request models
│   │   └── response.py            # Response models
│   └── config.py                  # Configuration
├── main.py                         # FastAPI entry point
├── .env                            # Environment variables (create from .env.example)
├── requirements.txt
└── README.md
```

## Setup

### 1. Install Dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 2. Configure Environment Variables

1. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` and add your Groq API key:
   ```
   GROQ_API_KEY=your_actual_groq_api_key_here
   ```

   Get your API key from: https://console.groq.com/

### 3. Add Knowledge Base PDFs

Place your PDF files in the `backend/parentdashboard/data/pdfs/` directory. The system will automatically load and index them when the API starts.

**Important**: The PDFs should contain information about:
- Phonological issues
- Speech therapy techniques
- Child speech development guidance

### 4. Run the Backend

```bash
# Development mode
python main.py

# Or using uvicorn directly
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

The API will be available at:
- API: http://localhost:8000
- Interactive Docs: http://localhost:8000/docs
- Alternative Docs: http://localhost:8000/redoc

## API Endpoints

### POST `/parentdashboard/ask`

Ask a question to the AI assistant.

**Request:**
```json
{
  "question": "How can I help my child with R sounds?"
}
```

**Response:**
```json
{
  "answer": "Based on the provided materials, here are some helpful strategies..."
}
```

### POST `/parentdashboard/reload`

Reload the knowledge base from PDFs. Use this after adding new PDFs.

**Response:**
```json
{
  "status": "Knowledge base reloaded successfully"
}
```

### GET `/parentdashboard/health`

Health check endpoint.

**Response:**
```json
{
  "status": "healthy",
  "service": "Parent Dashboard AI Assistant"
}
```

## How It Works

### RAG Pipeline

1. **PDF Loading**: PDFs are loaded from `parentdashboard/data/pdfs/`
2. **Chunking**: Text is split into overlapping chunks (1000 chars with 200 char overlap)
3. **Embedding**: Each chunk is converted to a vector using sentence transformers
4. **Storage**: Embeddings are stored in Chroma DB (local persistent storage)
5. **Retrieval**: When a question is asked, relevant chunks are retrieved using cosine similarity
6. **Generation**: Retrieved context is passed to Groq LLM to generate a parent-friendly answer

### AI Assistant Behavior

- Answers **ONLY** using information from the provided PDFs
- If information is not available, responds with: "This information is not available in the provided materials."
- Uses simple, parent-friendly language
- Avoids medical jargon
- Focuses on: phonological issues, speech therapy, and child speech development

## Adding New PDFs

1. Place PDF files in `backend/parentdashboard/data/pdfs/`
2. Call the `/parentdashboard/reload` endpoint or restart the server
3. The system will automatically:
   - Load the new PDFs
   - Chunk the text
   - Generate embeddings
   - Add to the vector store

## Chroma DB

Chroma DB stores embeddings locally in the `chroma_db/` directory (created automatically). The database persists between server restarts, so you don't need to reload PDFs every time unless you add new ones.

## Development Notes

- The backend is designed to be modular and extendable
- Other team members can add their components following the same structure
- Frontend integration is limited to the `Parent Awareness` folder only
- All configuration is in `config.py` and can be adjusted as needed

## Troubleshooting

### "GROQ_API_KEY not found"
- Make sure you've created a `.env` file with your Groq API key
- Check that `python-dotenv` is installed

### "No PDFs found"
- Ensure PDF files are in `backend/parentdashboard/data/pdfs/`
- Check file permissions

### Import errors
- Make sure all dependencies are installed: `pip install -r requirements.txt`
- Verify you're running from the `backend/` directory

## Production Considerations

- Replace CORS `allow_origins=["*"]` with your frontend URL
- Use environment variables for all sensitive configuration
- Consider adding authentication/authorization
- Set up proper logging
- Use a production ASGI server (e.g., Gunicorn with Uvicorn workers)

