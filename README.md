# RAG Quiz

A **Retrieval-Augmented Generation (RAG) quiz microservice** built with FastAPI. The application allows users to upload PDF documents, extracts and chunks their content, stores vector representations in Pinecone, and uses Google Gemini to generate quiz questions from the indexed document content.

## Features

- 📄 Upload PDF documents
- 🔎 Extract text from PDFs using `pdfplumber`
- ✂️ Split extracted text into overlapping chunks
- 🧠 Generate embeddings for document chunks
- 🗄️ Store and retrieve document vectors using Pinecone
- 🤖 Generate MCQ and short-answer questions using Google Gemini
- 🔄 Regenerate questions for an existing document
- 📊 Track answered questions and user scores
- 🗃️ Store documents, questions, users, and progress using SQLAlchemy
- ⚡ FastAPI background tasks for document ingestion
- 🌐 CORS enabled for frontend integration

## Architecture

```text
                    ┌──────────────────────┐
                    │       Client         │
                    │  Web / Frontend App  │
                    └──────────┬───────────┘
                               │ HTTP
                               ▼
                    ┌──────────────────────┐
                    │       FastAPI        │
                    │        API           │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼─────────────────┐
              │                │                 │
              ▼                ▼                 ▼
        PDF Upload       Question API       Quiz Logic
              │                │                 │
              ▼                │                 ▼
       Text Extraction         │          User Progress
              │                │
              ▼                ▼
           Chunking      Retrieve Chunks
              │                │
              ▼                ▼
          Embeddings      Google Gemini
              │                │
              ▼                │
          Pinecone ◄───────────┘
              │
              ▼
       Document Context
```

## Tech Stack

| Technology | Purpose |
|---|---|
| Python | Backend language |
| FastAPI | REST API |
| SQLAlchemy | Database ORM |
| SQLite | Default local database |
| Pinecone | Vector database |
| Google Gemini | Question generation |
| pdfplumber | PDF text extraction |
| NumPy | Current test embedding generation |
| Pydantic Settings | Environment configuration |
| Uvicorn | ASGI server |

## Project Structure

```text
RAG-quiz-main/
│
├── app/
│   ├── api/
│   │   ├── questions.py       # Question generation/listing endpoints
│   │   ├── quiz.py            # Quiz answering and grading logic
│   │   ├── progress.py        # Progress endpoint
│   │   └── upload.py          # PDF upload endpoint
│   │
│   ├── config/
│   │   └── settings.py        # Environment/configuration settings
│   │
│   ├── db/
│   │   └── session.py         # SQLAlchemy engine and sessions
│   │
│   ├── models/
│   │   ├── document.py        # Document model
│   │   ├── question.py        # Question model
│   │   ├── progress.py        # User progress model
│   │   └── user.py            # User model
│   │
│   ├── services/
│   │   ├── chunking.py        # Text chunking
│   │   ├── embeddings.py      # Embedding generation
│   │   ├── extraction.py      # PDF extraction
│   │   ├── llm_client.py      # Gemini question generation
│   │   ├── progress_service.py
│   │   ├── question_service.py
│   │   ├── quiz_service.py
│   │   └── vector_db.py       # Pinecone operations
│   │
│   ├── tasks/
│   │   └── ingestion.py       # PDF → chunks → embeddings → Pinecone
│   │
│   └── main.py                # FastAPI application entry point
│
├── requirements.txt
├── schemas.txt
└── quiz.db                    # Created locally when SQLite is used
```

## How the RAG Pipeline Works

### 1. Upload a PDF

The client sends a PDF to:

```http
POST /upload/
```

The server:

1. Validates the file extension.
2. Saves the PDF in the configured upload directory.
3. Creates a document record in the database.
4. Starts ingestion as a FastAPI background task.

### 2. Extract Text

`pdfplumber` extracts text from every PDF page.

```text
PDF
 ↓
PDF text extraction
 ↓
Full document text
```

### 3. Chunk the Document

The extracted text is divided into overlapping chunks.

Default configuration:

```text
chunk_size = 800
chunk_overlap = 200
```

The overlap helps preserve context between adjacent chunks.

### 4. Generate Embeddings

Each chunk is converted into a vector representation.

The current repository contains a **testing implementation** that generates random 1024-dimensional vectors using NumPy.

For production RAG retrieval, replace this implementation with a real embedding model.

### 5. Store Vectors in Pinecone

The vectors are stored in Pinecone together with metadata such as:

```json
{
  "document_id": "...",
  "chunk_id": "...",
  "text_excerpt": "..."
}
```

### 6. Generate Questions

When question generation is requested, the application:

```text
Document ID
    ↓
Retrieve document chunks from Pinecone
    ↓
Send chunk content to Gemini
    ↓
Generate MCQs + short-answer questions
    ↓
Validate generated JSON
    ↓
Store questions in SQL database
```

The default API request can generate:

- 10 MCQs
- 5 short-answer questions

## Prerequisites

Install:

- Python 3.10+
- A Pinecone account/API key
- A Google Gemini API key

A local SQLite database is used by default, so PostgreSQL is not required for the basic local setup.

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/anmol567-lab/RAG_QUIZ.git
cd RAG_QUIZ
```

If you are working from the downloaded project instead, open the project directory in VS Code.

### 2. Create a virtual environment

Windows:

```bash
python -m venv venv
venv\Scripts\activate
```

macOS/Linux:

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

## Environment Variables

Create a `.env` file in the project root.

Example:

```env
# Database
DATABASE_URL=sqlite:///./quiz.db

# Gemini
GEMINI_API_KEY=your_gemini_api_key

# Pinecone
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_ENVIRONMENT=your_pinecone_environment
PINECONE_INDEX_NAME=rag-quiz
PINECONE_DIMENSION=1024

# File storage
UPLOAD_DIR=./uploads
MAX_FILE_SIZE=10485760

# Chunking
CHUNK_SIZE=800
CHUNK_OVERLAP=200

# Question generation
DEFAULT_QUESTION_COUNT=10
MAX_QUESTION_COUNT=50

# Security/configuration
SECRET_KEY=change_this_value
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Redis/Celery configuration
REDIS_URL=redis://localhost:6379/0
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0

# Application
LOG_LEVEL=INFO
ENVIRONMENT=development
DEBUG=true
```

> **Important:** Never commit `.env` or API keys to GitHub.

## Run the Application

Start the FastAPI server with:

```bash
uvicorn app.main:app --reload
```

The API will normally be available at:

```text
http://127.0.0.1:8000
```

Interactive API documentation:

```text
http://127.0.0.1:8000/docs
```

Alternative ReDoc documentation:

```text
http://127.0.0.1:8000/redoc
```

## API Endpoints

### Health / Root

```http
GET /
```

Response:

```json
{
  "message": "Welcome to RAG QUIZ Microservice"
}
```

### Upload PDF

```http
POST /upload/
```

Form-data:

```text
file=<your-pdf-file>
```

Example response:

```json
{
  "document_id": "document-uuid",
  "status": "processing"
}
```

### List Questions

```http
GET /questions/{document_id}
```

Returns questions that have already been generated for a document.

### Generate Questions

```http
POST /questions/generate
```

Example request:

```json
{
  "document_id": "your-document-id",
  "n_mcq": 10,
  "n_match": 5,
  "n_short": 5,
  "regenerate": false
}
```

### Get Next Question

```http
GET /questions/next?user_id=USER_ID&document_id=DOCUMENT_ID
```

Returns the next unanswered question for the user.

### Submit an Answer

```http
POST /questions/answer
```

Example:

```json
{
  "user_id": "user-123",
  "document_id": "document-123",
  "question_id": "question-123",
  "answer": {
    "selected_index": 1
  },
  "elapsed_seconds": 20
}
```

For a short-answer question:

```json
{
  "user_id": "user-123",
  "document_id": "document-123",
  "question_id": "question-123",
  "answer": {
    "text": "Your answer"
  },
  "elapsed_seconds": 35
}
```

## Question Types

### Multiple Choice

Questions are stored in the following form:

```json
{
  "type": "mcq",
  "question": "What is ...?",
  "options": [
    "A) Option one",
    "B) Option two",
    "C) Option three",
    "D) Option four"
  ],
  "answer": "A) Option one"
}
```

### Short Answer

```json
{
  "type": "short",
  "question": "Explain ...?",
  "answer": "Expected answer"
}
```

## Question Generation Strategy

The Gemini client distributes the requested number of questions across retrieved document chunks.

For example:

```text
5 chunks
10 MCQs
5 short answers

        ↓

Each chunk receives approximately:
2 MCQs
1 short-answer question
```

The generated response is parsed as JSON and validated before questions are saved to the database.

The application also contains retry logic for Gemini rate-limit (`429`) responses.

## Database Models

The application currently defines the following main tables:

### Users

```text
users
├── id
├── first_name
├── last_name
├── email
├── created_at
└── updated_at
```

### Documents

```text
documents
├── id
├── user_id
├── filename
├── file_path
├── status
├── excerpt
├── created_at
└── updated_at
```

### Questions

```text
questions
├── id
├── document_id
├── question_type
├── question_text
├── options
├── answer
├── metadata
└── created_at
```

### User Progress

```text
user_progress
├── user_id
├── document_id
├── answered_question_ids
├── score
├── created_at
└── updated_at
```

## Document Processing Status

Documents can move through states such as:

```text
uploaded
   ↓
processing
   ↓
indexed
```

If ingestion fails:

```text
processing
   ↓
failed
```

## Configuration

The main configuration is handled by `app/config/settings.py`.

Important settings include:

| Setting | Default / Purpose |
|---|---|
| `DATABASE_URL` | `sqlite:///./quiz.db` |
| `UPLOAD_DIR` | `./uploads` |
| `CHUNK_SIZE` | `800` |
| `CHUNK_OVERLAP` | `200` |
| `DEFAULT_QUESTION_COUNT` | `10` |
| `MAX_QUESTION_COUNT` | `50` |
| `MCQ_PASSING_SCORE` | `70` |
| `DESCRIPTIVE_PASSING_SCORE` | `60` |

## Important Development Notes

### Embeddings

The current `app/services/embeddings.py` uses randomly generated vectors for testing:

```python
np.random.rand(1024)
```

These vectors are **not semantic embeddings**, so they should be replaced with a real embedding model before relying on similarity-based retrieval.

### Pinecone Dimension

The Pinecone index dimension must match the dimension produced by the embedding function.

Currently the test embedding implementation produces:

```text
1024 dimensions
```

Therefore:

```env
PINECONE_DIMENSION=1024
```

should match the active implementation.

### Authentication

The configuration contains authentication-related settings, but the current upload route uses:

```text
DUMMY_USER_ID = "dummy-user-1"
```

A production version should replace this with authenticated user identity.

### CORS

The application currently allows all origins:

```python
allow_origins=["*"]
```

For production, restrict this to the actual frontend origin.

## Future Improvements

- [ ] Replace fake embeddings with a production embedding model
- [ ] Add authentication and authorization
- [ ] Replace `DUMMY_USER_ID` with authenticated users
- [ ] Add proper document status/progress polling
- [ ] Add frontend UI for PDF upload and quizzes
- [ ] Add richer semantic retrieval instead of dummy-vector retrieval
- [ ] Add PostgreSQL support for production deployment
- [ ] Move long-running ingestion to Celery/Redis
- [ ] Add automated tests
- [ ] Add Docker support
- [ ] Add logging and monitoring
- [ ] Add question difficulty levels
- [ ] Add explanations for answers
- [ ] Add quiz history and analytics

## Security

Do not expose API keys in source code.

Use environment variables:

```env
GEMINI_API_KEY=...
PINECONE_API_KEY=...
SECRET_KEY=...
```

Also avoid committing:

```text
.env
quiz.db
uploads/
__pycache__/
```

## Useful Documentation

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Pinecone Documentation](https://docs.pinecone.io/)
- [Google Gemini API Documentation](https://ai.google.dev/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [Pydantic Settings Documentation](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)
- [Uvicorn Documentation](https://www.uvicorn.org/)
- [pdfplumber Documentation](https://github.com/jsvine/pdfplumber)

## License

Add the license appropriate for your project before publishing the repository.

---

Built as a RAG-based document-to-quiz backend using **FastAPI + Pinecone + Gemini + SQLAlchemy**.
