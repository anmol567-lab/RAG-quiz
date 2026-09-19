# RAG Quiz

A FastAPI-based document quiz generation service that uploads PDFs, extracts text, chunks the content, stores embeddings, and generates quiz questions from the document context using a RAG-style flow.

## Features

- PDF upload endpoint
- Text extraction using `pdfplumber`
- Chunking and embedding pipeline
- Pinecone vector storage for document chunks
- Quiz question generation with Gemini / LLM-backed logic
- Question listing and regeneration APIs
- SQLAlchemy persistence for documents and generated questions

## Tech Stack

- Python 3.11+
- FastAPI
- SQLAlchemy
- SQLite (default database)
- Pinecone vector DB
- Gemini API
- pdfplumber

## Project Structure

```text
app/
├── api/
│   ├── questions.py
│   ├── upload.py
│   └── quiz.py
├── config/
│   └── settings.py
├── db/
│   └── session.py
├── models/
│   ├── document.py
│   ├── progress.py
│   ├── question.py
│   └── user.py
├── services/
│   ├── chunking.py
│   ├── embeddings.py
│   ├── extraction.py
│   ├── llm_client.py
│   ├── question_service.py
│   └── vector_db.py
├── tasks/
│   └── ingestion.py
├── main.py
└── __init__.py
```

## Prerequisites

- Python 3.11+
- pip
- A valid `.env` file with the required environment variables
- Pinecone API access if using vector storage
- Gemini API key if using the LLM question-generation flow

## Environment Variables

Create a `.env` file in the project root with values like:

```env
DATABASE_URL=sqlite:///./quiz.db
GEMINI_API_KEY=your_gemini_key
PINECONE_API_KEY=your_pinecone_key
PINECONE_ENVIRONMENT=us-east-1
PINECONE_INDEX_NAME=rag-quiz-index
PINECONE_DIMENSION=1024
UPLOAD_DIR=./uploads
MAX_FILE_SIZE=10485760
SECRET_KEY=super-secret-key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
REDIS_URL=redis://localhost:6379/0
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0
```

Note: the application uses `pydantic-settings` to load these values from `.env` automatically.

## Installation

```bash
python -m venv .venv
source .venv/bin/activate   # Linux/macOS
# or .venv\Scripts\activate  # Windows

pip install -r requirements.txt
```

## Run the Application

```bash
uvicorn app.main:app --reload
```

The API will be available at:

```text
http://127.0.0.1:8000
```

## API Endpoints

### Upload a PDF

```bash
curl -X POST "http://127.0.0.1:8000/upload/" \
  -F "file=@sample.pdf"
```

Response:

```json
{
  "document_id": "<uuid>",
  "status": "processing"
}
```

### List questions for a document

```bash
curl "http://127.0.0.1:8000/questions/<document_id>"
```

### Generate questions for a document

```bash
curl -X POST "http://127.0.0.1:8000/questions/generate" \
  -H "Content-Type: application/json" \
  -d '{
    "document_id": "<document_id>",
    "n_mcq": 10,
    "n_match": 5,
    "n_short": 5,
    "regenerate": false
  }'
```

## Notes

- The project includes fallback / mock logic in some service layers for development and testability.
- Embedding generation currently uses a deterministic pseudo-random generator in `app/services/embeddings.py` unless replaced with a real embedding provider.
- The app is designed to work with Pinecone for indexing and retrieval, but the fallback logic can still support limited local testing when external services are unavailable.

## License

This project does not include a license file. Add one if you plan to distribute or publish the repository.
