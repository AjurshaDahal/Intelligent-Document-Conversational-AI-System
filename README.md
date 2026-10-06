
# Intelligent Document & Conversational AI API

A modular backend application built with **FastAPI** that combines document ingestion, semantic search, Retrieval-Augmented Generation (RAG), conversational memory, and LLM-powered interview booking.

The system allows users to upload PDF and TXT documents, process their contents, generate semantic embeddings, retrieve relevant information, and interact with the documents through a conversational interface. It also supports multi-turn interview-booking conversations using LLM-based intent detection and information extraction.

The project was developed as a backend-focused implementation of the PalmMind technical assignment.

## Where the project stands

The current implementation includes:

- PDF and TXT document ingestion
- PDF text extraction
- Configurable document chunking
- Fixed-size chunking with overlap
- Sentence-based chunking
- Sentence Transformer embeddings
- Qdrant vector storage
- Semantic similarity search
- Custom Retrieval-Augmented Generation pipeline
- Redis-based conversational memory
- LLM-powered document question answering
- Intent detection
- Multi-turn interview booking
- LLM-based booking information extraction
- SQLite persistence
- Automated tests for core services
- FastAPI REST API
- Interactive Swagger/OpenAPI documentation

## What the system does

| Component | Responsibility |
|---|---|
| Document Ingestion | Upload and process PDF/TXT documents |
| Text Extraction | Extract text from uploaded documents |
| Chunking | Split documents using fixed-size or sentence-based strategies |
| Embeddings | Convert document chunks into semantic vectors |
| Qdrant | Store and retrieve document embeddings |
| RAG Pipeline | Retrieve relevant document context and generate answers |
| Redis | Maintain conversation history and temporary booking state |
| Intent Detection | Identify document questions and booking requests |
| Booking System | Extract, validate, and store interview details |
| SQLite | Persist document metadata, chunks, and bookings |
| FastAPI | Expose the system through REST endpoints |

## RAG Pipeline

Document Upload → Text Extraction → Chunking → Embedding Generation → Qdrant Vector Storage → Query Embedding → Semantic Search → Relevant Document Chunks → Conversation History → Context Construction → LLM → Generated Answer

The RAG pipeline is implemented using individual retrieval and generation components rather than relying on a pre-built `RetrievalQAChain`.

## Conversational Booking

The same conversational API also supports interview booking.

The system can collect:

- Name
- Email
- Date
- Time

Information can be provided across multiple messages.

Example conversation:

> User: I want to book an interview.
>
> Assistant: Sure! I need your name, email, preferred date, and preferred interview time.
>
> User: My name is ......
>
> User: My email is ....@....com.
>
> User: October 5, 2026.
>
> User: 2 PM.

The application maintains incomplete booking information in Redis until all required information is collected. Once complete, the information is validated and persisted in SQLite.

## Technology Stack

- **Python** — Backend development
- **FastAPI** — REST API framework
- **SQLAlchemy** — Database ORM
- **SQLite** — Persistent metadata and booking storage
- **Qdrant** — Vector database
- **Redis** — Conversation memory and temporary state
- **Sentence Transformers** — Text embeddings
- **PyTorch** — Embedding model runtime
- **Ollama** — Local LLM inference
- **Llama 3.2 3B** — Local language model
- **pypdf** — PDF text extraction
- **pytest** — Automated testing

## Project Structure

    intelligent-document-conversational-ai/
    │
    ├── app/
    │   ├── api/
    │   │   └── routes/
    │   │       ├── chat.py
    │   │       └── documents.py
    │   │
    │   ├── core/
    │   │   └── config.py
    │   │
    │   ├── db/
    │   │   ├── base.py
    │   │   ├── init_db.py
    │   │   ├── session.py
    │   │   └── test_db.py
    │   │
    │   ├── models/
    │   │   ├── booking.py
    │   │   └── document.py
    │   │
    │   ├── services/
    │   │   ├── booking_extractor.py
    │   │   ├── booking_parser.py
    │   │   ├── booking_service.py
    │   │   ├── booking_validation.py
    │   │   ├── chunk_service.py
    │   │   ├── document_service.py
    │   │   ├── embedding_service.py
    │   │   ├── intent_service.py
    │   │   ├── llm_service.py
    │   │   ├── pdf_service.py
    │   │   ├── rag_service.py
    │   │   ├── redis_service.py
    │   │   ├── search_service.py
    │   │   └── vector_service.py
    │   │
    │   ├── utils/
    │   │
    │   └── main.py
    │
    ├── tests/
    │
    ├── .env.example
    ├── .gitignore
    ├── pyproject.toml
    ├── pytest.ini
    ├── requirements.txt
    └── README.md

## Quick Setup for Developers

### Requirements

Before running the application, install:

- Python 3.12+
- Redis
- Ollama
- Ollama model: `llama3.2:3b`

The application currently uses local Redis and Ollama services.

### Clone the Repository

    git clone https://github.com/AjurshaDahal/PalmMindTASK.git
    cd PalmMindTASK

Update the repository name above if the GitHub repository has been renamed.

### Create a Virtual Environment

    python -m venv .venv
    source .venv/bin/activate

On Windows:

    .venv\Scripts\activate

### Install Dependencies

    pip install -r requirements.txt

### Initialize the Database

    python -m app.db.init_db

## Redis Setup

Redis is used for:

- Conversation history
- Multi-turn conversation state
- Temporary interview-booking state

The application expects Redis to be available at:

    localhost:6379

Start Redis before using the chat endpoints.

## Ollama Setup

The application uses Ollama for local LLM inference.

Pull the required model:

    ollama pull llama3.2:3b

Make sure Ollama is running before using the conversational and booking functionality.

## Running the Application

Start the FastAPI development server:

    uvicorn app.main:app --reload

The API will be available at:

    http://127.0.0.1:8000

## API Documentation

FastAPI automatically provides interactive API documentation.

### Swagger UI

    http://127.0.0.1:8000/docs

### OpenAPI Specification

    http://127.0.0.1:8000/openapi.json

## API Endpoints

### Document Upload

`POST /documents/upload`

Uploads and processes a PDF or TXT document for semantic retrieval.

Supported formats:

- `.pdf`
- `.txt`

Available chunking strategies:

- `fixed`
- `sentence`

Example response:

    {
      "id": 1,
      "filename": "document.pdf",
      "status": "processed",
      "chunking_strategy": "fixed",
      "chunk_count": 12
    }

### Conversational RAG

`POST /chat`

Accepts a conversation ID and user message.

Example request:

    {
      "conversation_id": "conversation-1",
      "message": "What does the document say about cloud computing?"
    }

Example response:

    {
      "conversation_id": "conversation-1",
      "intent": "question",
      "answer": "..."
    }

The RAG process:

1. Receives the user's question
2. Generates an embedding for the question
3. Searches Qdrant for semantically similar document chunks
4. Retrieves relevant document content
5. Retrieves previous conversation history from Redis
6. Builds the LLM context
7. Sends the context, conversation history, and question to the LLM
8. Generates a response
9. Stores the conversation in Redis

### Interview Booking

Interview booking is handled through the same conversational `/chat` endpoint.

The system detects booking-related requests and extracts:

- Name
- Email
- Date
- Time

Missing information can be collected across multiple messages.

The booking flow is:

    User Request
         │
         ▼
    Intent Detection
         │
         ▼
    Booking Information Extraction
         │
         ▼
    Check Missing Fields
         │
         ├── Missing ──► Ask User
         │
         ▼
    Validation
         │
         ▼
    Date/Time Parsing
         │
         ▼
    Store Booking in SQLite
         │
         ▼
    Clear Temporary Redis State
         │
         ▼
    Return Confirmation

## Data Storage

The application uses different storage technologies for different responsibilities.

### SQLite

Used for persistent application data:

- Documents
- Document chunks
- Interview bookings

### Qdrant

Used as the vector database for:

- Document embeddings
- Document IDs
- Chunk metadata
- Semantic similarity search

### Redis

Used for temporary and conversational state:

- Conversation history
- Multi-turn conversation state
- Temporary interview-booking information

## Embedding Model

The project uses:

`all-MiniLM-L6-v2`

from Sentence Transformers.

Document chunks and user queries are converted into vector embeddings.

Qdrant uses these embeddings to perform semantic similarity search using cosine similarity.

## Testing

The project includes tests for core application components, including:

- Database
- Booking extraction
- Booking parsing
- Booking service
- Chunking
- Embeddings
- Intent detection
- LLM service
- PDF extraction
- RAG
- Redis
- Search
- Vector storage

Run the test suite with:

    python -m pytest

Pytest discovery is configured through:

    pytest.ini

## Configuration

Environment configuration is provided through:

    .env.example

Do not commit private environment variables, credentials, or API keys.

Runtime files such as databases, Qdrant storage, Redis dumps, uploaded documents, archives, virtual environments, and Python cache files should be excluded through `.gitignore`.

## Design Considerations

### Modular Architecture

The application separates responsibilities across:

- API routes
- Database configuration
- Data models
- Document processing
- PDF extraction
- Chunking
- Embedding generation
- Vector search
- RAG
- Redis memory
- Intent detection
- Booking logic
- LLM interaction

This keeps individual components easier to test, maintain, and extend.

### Custom RAG Pipeline

The RAG implementation is built from individual retrieval and generation components rather than using a pre-built `RetrievalQAChain`.

This provides explicit control over:

- Document ingestion
- Text extraction
- Chunking
- Embedding generation
- Vector retrieval
- Context construction
- Conversation memory
- LLM generation

### Multi-Turn Conversations

Redis allows conversation history and partially completed booking information to persist between requests using a conversation ID.

### Local-First Development

The application can run locally using:

- SQLite
- Qdrant
- Redis
- Ollama

This allows development without requiring a hosted vector database or external LLM API.

## Limitations & Future Improvements

Potential improvements include:

- Advanced PDF parsing for complex layouts
- Table and image-aware document extraction
- More sophisticated semantic chunking
- Configurable embedding models
- Configurable LLM providers
- Hosted Qdrant and Redis support
- Authentication and authorization
- Booking conflict detection
- Time-zone-aware scheduling
- Improved API error handling
- Automated CI/CD
- Expanded integration and API tests
- Containerized deployment
- Cloud deployment

## Important Notes

- This repository is primarily a backend-focused implementation.
- The document intelligence and conversational RAG components are the core functionality of the project.
- Interview booking is implemented as an additional conversational capability.
- Qdrant, Redis, SQLite, and Ollama can be run locally.
- Runtime databases, vector storage, uploaded documents, virtual environments, and cache files should not be committed to Git.
- The project was developed as a technical assignment and is intended for evaluation, learning, and demonstration purposes.

## License

This project was developed as a technical assignment and is intended for evaluation, learning, and demonstration purposes.
```
