# Intelligent Document & Conversational AI API

A modular backend application built with **FastAPI** that combines document ingestion, semantic search, conversational Retrieval-Augmented Generation (RAG), Redis-based conversation memory, and LLM-powered interview booking.

The system allows users to upload documents, process and semantically search their contents, ask questions about the uploaded information through a conversational interface, and handle interview-booking requests through the same conversational API.

The project was developed as a backend-focused implementation of the PalmMind technical assignment.

---

## Overview

The application provides two primary capabilities:

1. **Document Intelligence & Conversational RAG**
   - Upload PDF and TXT documents
   - Extract and process document text
   - Apply selectable chunking strategies
   - Generate semantic embeddings
   - Store embeddings in Qdrant
   - Store document metadata in SQLite
   - Ask questions about uploaded documents
   - Retrieve relevant document content using semantic similarity
   - Generate contextual answers using an LLM

2. **Conversational Interview Booking**
   - Detect interview-booking requests
   - Extract booking information using an LLM
   - Collect missing information across multiple messages
   - Maintain conversation and booking state with Redis
   - Validate booking information
   - Persist completed bookings in SQLite

---

## Key Features

### Document Ingestion

- PDF and TXT file support
- PDF text extraction using `pypdf`
- Two selectable chunking strategies:
  - Fixed-size chunking with overlap
  - Sentence-based chunking
- Sentence Transformer embeddings
- Qdrant vector storage
- SQLite document and chunk metadata

### Document Question Answering

- Custom Retrieval-Augmented Generation pipeline
- Semantic similarity search using Qdrant
- Retrieval of relevant document chunks
- Context construction from retrieved information
- LLM-generated responses
- Conversation-aware document questions
- No `RetrievalQAChain` or pre-built RAG chain

### Conversational AI

- Multi-turn conversations
- Conversation history stored in Redis
- Context-aware responses
- Intent detection
- LLM-based response generation
- Conversation-specific state management

### Interview Booking

- Natural-language booking requests
- LLM-based extraction of:
  - Name
  - Email
  - Date
  - Time
- Multi-turn collection of missing information
- Email validation
- Date and time parsing
- Booking persistence in SQLite
- Temporary booking state stored in Redis

---

## Architecture

```text
                         ┌─────────────────────┐
                         │      FastAPI        │
                         │       REST API      │
                         └──────────┬──────────┘
                                    │
                 ┌──────────────────┴──────────────────┐
                 │                                     │
                 ▼                                     ▼
        ┌──────────────────┐                  ┌──────────────────┐
        │ Document Upload  │                  │   Chat Endpoint  │
        └────────┬─────────┘                  └────────┬─────────┘
                 │                                     │
                 ▼                                     ▼
        ┌──────────────────┐                  ┌──────────────────┐
        │ Text Extraction  │                  │ Intent Detection │
        └────────┬─────────┘                  └────────┬─────────┘
                 │                                     │
                 ▼                         ┌───────────┴───────────┐
        ┌──────────────────┐               │                       │
        │ Chunking         │               ▼                       ▼
        │ Fixed/Sentence   │        ┌──────────────┐       ┌──────────────┐
        └────────┬─────────┘        │ RAG Pipeline │       │    Booking   │
                 │                  └──────┬───────┘       └──────┬───────┘
                 ▼                         │                      │
        ┌──────────────────┐               ▼                      ▼
        │ Embeddings       │        ┌──────────────┐       ┌──────────────┐
        │ MiniLM           │        │    Qdrant    │       │    SQLite    │
        └────────┬─────────┘        │ Vector Store │       │   Bookings   │
                 │                  └──────────────┘       └──────────────┘
                 ▼
        ┌──────────────────┐
        │     Qdrant       │
        │  Vector Storage  │
        └──────────────────┘

                    Redis
                      │
                      ├── Conversation history
                      └── Temporary booking state
Technology Stack
Technology	Purpose
Python	Backend development
FastAPI	REST API framework
SQLAlchemy	Database ORM
SQLite	Metadata and booking persistence
Qdrant	Vector database
Redis	Conversation memory and state
Sentence Transformers	Text embeddings
PyTorch	Embedding model runtime
Ollama	Local LLM inference
Llama 3.2 3B	LLM used by the application
pypdf	PDF text extraction
pytest	Testing
Project Structure
Intelligent-Document-Conversational-AI/
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
Requirements

Before running the application, install:

Python 3.12+
Redis
Ollama
Ollama model: llama3.2:3b

The application currently uses local Redis and Ollama services.

Installation
1. Clone the repository
git clone https://github.com/AjurshaDahal/PalmMindTASK.git
cd PalmMindTASK

Replace the repository name above if the GitHub repository is renamed.

2. Create a virtual environment
python -m venv .venv
source .venv/bin/activate

On Windows:

.venv\Scripts\activate
3. Install Python dependencies
pip install -r requirements.txt
4. Initialize the database
python -m app.db.init_db
Redis Setup

Redis is used for:

Conversation history
Temporary interview-booking state
Multi-turn conversations

The application expects Redis to be available at:

localhost:6379

Start Redis before using the chat endpoints.

Ollama Setup

The application uses Ollama for local LLM inference.

Install the required model:

ollama pull llama3.2:3b

Make sure Ollama is running before using the conversational and booking functionality.

Running the Application

Start the FastAPI development server:

uvicorn app.main:app --reload

The API will be available at:

http://127.0.0.1:8000
API Documentation

FastAPI automatically provides interactive API documentation.

Swagger UI
http://127.0.0.1:8000/docs
OpenAPI Specification
http://127.0.0.1:8000/openapi.json
API Endpoints
1. Document Ingestion
POST /documents/upload

Uploads a PDF or TXT document and processes it for semantic retrieval.

Supported file types
.pdf
.txt
Chunking strategies
Fixed-size
fixed

Uses fixed character-size chunks with overlap.

Sentence-based
sentence

Groups a configurable number of sentences into each chunk.

Example

Using Swagger UI, provide:

file
strategy

Example:

fixed

The response includes information such as:

{
  "id": 1,
  "filename": "document.pdf",
  "status": "processed",
  "chunking_strategy": "fixed",
  "chunk_count": 12
}
2. Conversational RAG
POST /chat

Accepts a conversation ID and user message.

Request
{
  "conversation_id": "conversation-1",
  "message": "What does the document say about cloud computing?"
}
Example response
{
  "conversation_id": "conversation-1",
  "intent": "question",
  "answer": "..."
}

The RAG pipeline:

Receives the user's question
Generates an embedding for the question
Searches Qdrant for semantically similar document chunks
Builds context from the retrieved chunks
Retrieves previous conversation messages from Redis
Sends the context, conversation history, and question to the LLM
Returns the generated answer
Stores the new conversation messages in Redis

This allows the system to answer questions about uploaded documents while maintaining conversational context.

3. Conversational Interview Booking

The same /chat endpoint also supports interview-booking conversations.

The system can collect:

Name
Email
Date
Time

Information can be provided across multiple messages.

For example:

User: I want to book an interview.

Assistant: Sure! I still need your name, email address,
preferred interview date, and preferred interview time.

User: My name is Ajursha.

User: My email is ajursha@example.com.

User: October 5, 2026.

User: 2 PM.

The booking state is maintained in Redis until all required information is collected.

Once the information is complete:

Booking details are validated
Date and time are parsed
The booking is stored in SQLite
Temporary Redis booking state is cleared
A booking confirmation is returned
Data Storage

The application uses three storage systems for different purposes.

SQLite

Stores persistent application data:

Documents
Document chunks
Interview bookings
Qdrant

Stores:

Document chunk embeddings
Document IDs
Chunk text metadata
Redis

Stores temporary and conversational state:

Chat history
Interview booking state
Embedding Model

The project uses:

all-MiniLM-L6-v2

from Sentence Transformers.

The generated embeddings are stored in Qdrant and compared using cosine similarity for semantic retrieval.

Testing

Tests are included throughout the project for core services including:

Database
Booking extraction
Booking parsing
Booking service
Chunking
Embeddings
Intent detection
LLM service
PDF extraction
RAG
Redis
Search
Vector storage

Run the test suite with:

python -m pytest

Pytest discovery is configured through:

pytest.ini
Configuration

An example environment configuration is provided in:

.env.example

Do not commit private environment variables or credentials.

Runtime files such as databases, Qdrant storage, Redis dumps, uploaded documents, archives, virtual environments, and Python cache files are excluded through .gitignore.

Design Considerations
Modular Architecture

The application separates:

API routes
Database configuration
Models
Document processing
Embeddings
Vector search
RAG
Redis memory
Booking logic
LLM interaction

This keeps individual components easier to test and maintain.

Custom RAG Pipeline

The RAG implementation is intentionally built using individual retrieval and generation components rather than a pre-built RetrievalQAChain.

This provides explicit control over:

Document ingestion
Chunking
Embedding generation
Vector retrieval
Context construction
Conversation memory
LLM generation
Multi-turn Conversations

Redis allows conversation history and partially completed booking information to persist between requests using a conversation ID.

Local-first Development

Qdrant, SQLite, Redis, and Ollama can be used locally, making the project suitable for development without requiring a hosted vector database or hosted LLM API.

Limitations / Future Improvements

Potential future improvements include:

More advanced document parsing for complex PDF layouts
More sophisticated semantic chunking
Configurable embedding and LLM models through environment variables
Hosted Qdrant/Redis support
Authentication and authorization
Booking conflict detection
Time-zone aware interview scheduling
Improved API error handling
Automated CI testing
Expanded integration and API tests
License

This project was developed as a technical assignment and is intended for evaluation and demonstration purposes.


### One important thing

I **wouldn't rename every occurrence of `PalmMindTASK` inside the actual code/folder yet** just because we changed the README title.

There are three separate things:

- **Project display name:** `Intelligent Document & Conversational AI API`
- **Local folder:** can be renamed
- **GitHub repository:** `PalmMindTASK` → whatever new repo name you choose

If you want the GitHub repo to look professional on your CV, I'd use:

> **`intelligent-document-conversational-ai`**

And the README title:

> **Intelligent Document & Conversational AI API**
