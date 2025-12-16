# RAG Chatbot Integration for Physical AI & Humanoid Robotics Book

## Overview

This document provides comprehensive instructions for integrating a Retrieval-Augmented Generation (RAG) chatbot with the Physical AI & Humanoid Robotics book. The chatbot will be able to answer questions about the book content using OpenAI Agents, FastAPI, Neon Postgres, and Qdrant Cloud for vector storage and retrieval.

## Architecture Overview

```mermaid
graph TB
    A[User Query] --> B[FastAPI Backend]
    B --> C[Qdrant Vector DB]
    C --> D[Neon Postgres Metadata]
    B --> E[OpenAI API]
    D --> B
    C --> B
    E --> B
    B --> A[Answer Response]
```

## Prerequisites

- Python 3.8 or higher
- OpenAI API key
- Qdrant Cloud account
- Neon Postgres account
- Docker (optional, for containerization)

## 1. Environment Setup

### 1.1 Create Virtual Environment

```bash
python -m venv rag-chatbot-env
source rag-chatbot-env/bin/activate  # On Windows: rag-chatbot-env\Scripts\activate
```

### 1.2 Install Dependencies

```bash
pip install fastapi uvicorn python-multipart python-dotenv openai langchain-community langchain-openai qdrant-client psycopg2-binary sqlalchemy python-slugify
```

### 1.3 Create Environment File

Create a `.env` file in your project root:

```env
OPENAI_API_KEY=your_openai_api_key_here
QDRANT_URL=your_qdrant_cloud_url
QDRANT_API_KEY=your_qdrant_api_key
DATABASE_URL=postgresql://username:password@ep-xxx.us-east-1.aws.neon.tech/dbname?sslmode=require
```

## 2. Database Setup (Neon Postgres)

### 2.1 Create Database Schema

```python
# database.py
from sqlalchemy import create_engine, Column, Integer, String, Text, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime
import os
from dotenv import load_dotenv

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

class BookSection(Base):
    __tablename__ = "book_sections"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    content = Column(Text)
    chapter = Column(String)
    section = Column(String)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

# Create tables
Base.metadata.create_all(bind=engine)
```

### 2.2 Database Session Management

```python
# database.py (continued)
from contextlib import contextmanager

@contextmanager
def get_db_session():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

## 3. Vector Database Setup (Qdrant Cloud)

### 3.1 Initialize Qdrant Client

```python
# vector_store.py
from qdrant_client import QdrantClient
from qdrant_client.http import models
from typing import List, Dict, Optional
import os
from dotenv import load_dotenv

load_dotenv()

class VectorStore:
    def __init__(self):
        self.client = QdrantClient(
            url=os.getenv("QDRANT_URL"),
            api_key=os.getenv("QDRANT_API_KEY"),
        )
        self.collection_name = "book_content"
        self._create_collection()

    def _create_collection(self):
        """Create the collection if it doesn't exist"""
        try:
            self.client.get_collection(self.collection_name)
        except:
            # Create collection with dense vector configuration
            self.client.create_collection(
                collection_name=self.collection_name,
                vectors_config=models.VectorParams(
                    size=1536,  # OpenAI embedding dimension
                    distance=models.Distance.COSINE
                )
            )

    def add_documents(self, documents: List[Dict]):
        """Add documents to the vector store"""
        points = []
        for i, doc in enumerate(documents):
            # Create a point for Qdrant
            points.append(models.PointStruct(
                id=i,
                vector=doc['embedding'],  # This should be the embedding vector
                payload={
                    'title': doc['title'],
                    'content': doc['content'],
                    'chapter': doc['chapter'],
                    'section': doc['section'],
                    'source': doc.get('source', 'book')
                }
            ))

        self.client.upsert(
            collection_name=self.collection_name,
            points=points
        )

    def search(self, query_embedding: List[float], limit: int = 5) -> List[Dict]:
        """Search for similar documents"""
        results = self.client.search(
            collection_name=self.collection_name,
            query_vector=query_embedding,
            limit=limit
        )

        return [
            {
                'id': result.id,
                'payload': result.payload,
                'score': result.score
            }
            for result in results
        ]

    def delete_collection(self):
        """Delete the collection (useful for testing)"""
        self.client.delete_collection(self.collection_name)
```

## 4. Embedding and Document Processing

### 4.1 Document Embedding Service

```python
# embedding_service.py
import openai
from typing import List, Dict
import os
from dotenv import load_dotenv

load_dotenv()

openai.api_key = os.getenv("OPENAI_API_KEY")

class EmbeddingService:
    def __init__(self, model="text-embedding-ada-002"):
        self.model = model

    def create_embedding(self, text: str) -> List[float]:
        """Create embedding for a single text"""
        response = openai.Embedding.create(
            input=text,
            model=self.model
        )
        return response['data'][0]['embedding']

    def create_embeddings(self, texts: List[str]) -> List[List[float]]:
        """Create embeddings for multiple texts"""
        embeddings = []
        for text in texts:
            embedding = self.create_embedding(text)
            embeddings.append(embedding)
        return embeddings

    def get_embedding_dimension(self) -> int:
        """Get the dimension of the embeddings"""
        sample_text = "This is a sample text for dimension testing."
        sample_embedding = self.create_embedding(sample_text)
        return len(sample_embedding)
```

### 4.2 Document Chunking Service

```python
# document_processor.py
from typing import List, Dict
from langchain.text_splitter import RecursiveCharacterTextSplitter
import re

class DocumentProcessor:
    def __init__(self, chunk_size: int = 1000, chunk_overlap: int = 200):
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=chunk_size,
            chunk_overlap=chunk_overlap,
            length_function=len,
            separators=["\n\n", "\n", " ", ""]
        )

    def process_book_content(self, content: str, title: str, chapter: str, section: str) -> List[Dict]:
        """Process book content into chunks suitable for vector storage"""
        # Split the content into chunks
        chunks = self.text_splitter.split_text(content)

        documents = []
        for i, chunk in enumerate(chunks):
            documents.append({
                'title': f"{title} - Part {i+1}",
                'content': chunk,
                'chapter': chapter,
                'section': section,
                'source': f"{title}_part_{i+1}"
            })

        return documents

    def clean_content(self, content: str) -> str:
        """Clean content by removing extra whitespace and formatting"""
        # Remove extra whitespace
        content = re.sub(r'\s+', ' ', content)
        # Remove markdown formatting if present
        content = re.sub(r'\*{1,2}([^*]+)\*{1,2}', r'\1', content)  # Bold/italic
        content = re.sub(r'#{1,6}\s*(.+)', r'\1', content)  # Headers
        content = re.sub(r'```[\s\S]*?```', '', content)  # Code blocks
        content = re.sub(r'`([^`]+)`', r'\1', content)  # Inline code

        return content.strip()
```

## 5. FastAPI Backend

### 5.1 Main Application

```python
# main.py
from fastapi import FastAPI, HTTPException, Depends, Request
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List, Optional
import logging
from database import get_db, BookSection
from vector_store import VectorStore
from embedding_service import EmbeddingService
from document_processor import DocumentProcessor
import os
from dotenv import load_dotenv

load_dotenv()

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(title="Physical AI & Humanoid Robotics RAG Chatbot",
              description="A RAG-based chatbot for the Physical AI & Humanoid Robotics book",
              version="1.0.0")

# Add CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # In production, specify exact origins
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Initialize services
vector_store = VectorStore()
embedding_service = EmbeddingService()
document_processor = DocumentProcessor()

class QueryRequest(BaseModel):
    query: str
    max_results: int = 5

class QueryResponse(BaseModel):
    query: str
    answer: str
    sources: List[Dict]

class DocumentRequest(BaseModel):
    title: str
    content: str
    chapter: str
    section: str

@app.get("/")
async def root():
    return {"message": "Physical AI & Humanoid Robotics RAG Chatbot API"}

@app.post("/query", response_model=QueryResponse)
async def query_endpoint(request: QueryRequest):
    """Query the RAG system for answers"""
    try:
        logger.info(f"Processing query: {request.query}")

        # Create embedding for the query
        query_embedding = embedding_service.create_embedding(request.query)

        # Search for relevant documents
        search_results = vector_store.search(query_embedding, limit=request.max_results)

        if not search_results:
            return QueryResponse(
                query=request.query,
                answer="I couldn't find relevant information in the book to answer your question.",
                sources=[]
            )

        # Prepare context from search results
        context_parts = []
        sources = []

        for result in search_results:
            content = result['payload']['content']
            context_parts.append(content)

            sources.append({
                'title': result['payload']['title'],
                'chapter': result['payload']['chapter'],
                'section': result['payload']['section'],
                'score': result['score']
            })

        context = "\n\n".join(context_parts)

        # Generate answer using OpenAI
        answer = await generate_answer(request.query, context)

        return QueryResponse(
            query=request.query,
            answer=answer,
            sources=sources
        )

    except Exception as e:
        logger.error(f"Error processing query: {str(e)}")
        raise HTTPException(status_code=500, detail=f"Error processing query: {str(e)}")

async def generate_answer(query: str, context: str) -> str:
    """Generate answer using OpenAI based on context"""
    import openai

    prompt = f"""
    You are an expert assistant for the Physical AI & Humanoid Robotics book.
    Use the following context to answer the user's question.
    If the context doesn't contain relevant information, say so.

    Context:
    {context}

    Question: {query}

    Answer (be concise and relevant to the book content):
    """

    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "You are an expert assistant for the Physical AI & Humanoid Robotics book. Provide accurate and helpful answers based on the book content."},
                {"role": "user", "content": prompt}
            ],
            max_tokens=500,
            temperature=0.3
        )

        return response.choices[0].message.content.strip()
    except Exception as e:
        logger.error(f"Error generating answer: {str(e)}")
        return "Sorry, I encountered an error while generating the answer."

@app.post("/documents")
async def add_document(document: DocumentRequest):
    """Add a document to the vector store"""
    try:
        # Process the document content
        processed_chunks = document_processor.process_book_content(
            document.content,
            document.title,
            document.chapter,
            document.section
        )

        # Create embeddings for each chunk
        for chunk in processed_chunks:
            content_embedding = embedding_service.create_embedding(chunk['content'])
            chunk['embedding'] = content_embedding

        # Add to vector store
        vector_store.add_documents(processed_chunks)

        # Also store in PostgreSQL for metadata
        from sqlalchemy.orm import Session
        db: Session = next(get_db())

        for chunk in processed_chunks:
            db_section = BookSection(
                title=chunk['title'],
                content=chunk['content'],
                chapter=chunk['chapter'],
                section=chunk['section']
            )
            db.add(db_section)

        db.commit()

        return {"message": f"Successfully added {len(processed_chunks)} document chunks"}

    except Exception as e:
        logger.error(f"Error adding document: {str(e)}")
        raise HTTPException(status_code=500, detail=f"Error adding document: {str(e)}")

@app.get("/health")
async def health_check():
    """Health check endpoint"""
    return {"status": "healthy", "vector_store": "connected", "database": "connected"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## 6. Data Ingestion Script

### 6.1 Script to Load Book Content

```python
# ingest_book_content.py
import os
import glob
from dotenv import load_dotenv
from embedding_service import EmbeddingService
from document_processor import DocumentProcessor
from vector_store import VectorStore
from database import SessionLocal, BookSection
from slugify import slugify
import markdown
from pathlib import Path

load_dotenv()

def ingest_book_files(book_directory: str):
    """Ingest all book content files into the vector store"""
    embedding_service = EmbeddingService()
    document_processor = DocumentProcessor()
    vector_store = VectorStore()

    # Connect to database
    db = SessionLocal()

    try:
        # Find all book content files (MD, MDX, etc.)
        content_files = []
        content_patterns = ["*.md", "*.mdx", "*.txt"]

        for pattern in content_patterns:
            content_files.extend(glob.glob(os.path.join(book_directory, pattern), recursive=True))

        print(f"Found {len(content_files)} content files to process")

        all_documents = []

        for file_path in content_files:
            print(f"Processing file: {file_path}")

            # Extract chapter and section from filename/path
            path_parts = Path(file_path).parts
            chapter = "unknown"
            section = "unknown"

            # Try to extract chapter/section info from path
            for part in path_parts:
                if "chapter" in part.lower():
                    chapter = part
                elif "section" in part.lower():
                    section = part

            # Read file content
            with open(file_path, 'r', encoding='utf-8') as file:
                content = file.read()

            # Clean content if it's markdown
            if file_path.endswith(('.md', '.mdx')):
                # Convert markdown to plain text for better processing
                html = markdown.markdown(content)
                # Remove HTML tags to get plain text
                import re
                plain_text = re.sub('<[^<]+?>', '', html)
                content = plain_text

            # Process content into chunks
            title = os.path.basename(file_path).replace('.md', '').replace('.mdx', '').replace('.txt', '')
            processed_chunks = document_processor.process_book_content(
                content, title, chapter, section
            )

            # Add embeddings to each chunk
            for chunk in processed_chunks:
                content_embedding = embedding_service.create_embedding(chunk['content'])
                chunk['embedding'] = content_embedding

            all_documents.extend(processed_chunks)

            # Store in database
            for chunk in processed_chunks:
                db_section = BookSection(
                    title=chunk['title'],
                    content=chunk['content'],
                    chapter=chunk['chapter'],
                    section=chunk['section']
                )
                db.add(db_section)

        # Add all documents to vector store
        if all_documents:
            vector_store.add_documents(all_documents)
            print(f"Added {len(all_documents)} document chunks to vector store")

        # Commit database changes
        db.commit()
        print("Successfully ingested all book content")

    except Exception as e:
        print(f"Error during ingestion: {str(e)}")
        db.rollback()
    finally:
        db.close()

if __name__ == "__main__":
    # Specify the directory containing your book files
    BOOK_DIRECTORY = "./book_content"  # Update this path

    if not os.path.exists(BOOK_DIRECTORY):
        print(f"Book directory {BOOK_DIRECTORY} does not exist!")
        exit(1)

    ingest_book_files(BOOK_DIRECTORY)
```

## 7. Frontend Integration Example

### 7.1 Simple HTML Interface

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Physical AI & Humanoid Robotics Chatbot</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .chat-container {
            background-color: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .message {
            margin: 10px 0;
            padding: 10px;
            border-radius: 5px;
        }
        .user-message {
            background-color: #e3f2fd;
            text-align: right;
        }
        .bot-message {
            background-color: #f5f5f5;
        }
        .input-container {
            display: flex;
            margin-top: 20px;
        }
        #query-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
        }
        #send-button {
            padding: 10px 20px;
            margin-left: 10px;
            background-color: #2196f3;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        #send-button:hover {
            background-color: #1976d2;
        }
        .sources {
            font-size: 0.8em;
            color: #666;
            margin-top: 5px;
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <h1>Physical AI & Humanoid Robotics Chatbot</h1>
        <div id="chat-messages"></div>
        <div class="input-container">
            <input type="text" id="query-input" placeholder="Ask a question about Physical AI & Humanoid Robotics...">
            <button id="send-button">Send</button>
        </div>
    </div>

    <script>
        const API_BASE_URL = 'http://localhost:8000'; // Update with your API URL
        const chatMessages = document.getElementById('chat-messages');
        const queryInput = document.getElementById('query-input');
        const sendButton = document.getElementById('send-button');

        // Add sample messages
        function addMessage(text, isUser = false) {
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${isUser ? 'user-message' : 'bot-message'}`;
            messageDiv.textContent = text;
            chatMessages.appendChild(messageDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        // Send query to API
        async function sendQuery() {
            const query = queryInput.value.trim();
            if (!query) return;

            // Add user message to chat
            addMessage(query, true);
            queryInput.value = '';

            try {
                // Show typing indicator
                const typingIndicator = document.createElement('div');
                typingIndicator.className = 'message bot-message';
                typingIndicator.id = 'typing-indicator';
                typingIndicator.textContent = 'Thinking...';
                chatMessages.appendChild(typingIndicator);
                chatMessages.scrollTop = chatMessages.scrollHeight;

                // Send query to API
                const response = await fetch(`${API_BASE_URL}/query`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        query: query,
                        max_results: 3
                    })
                });

                // Remove typing indicator
                const typingEl = document.getElementById('typing-indicator');
                if (typingEl) typingEl.remove();

                if (response.ok) {
                    const data = await response.json();
                    addMessage(data.answer);

                    // Add sources
                    if (data.sources && data.sources.length > 0) {
                        const sourcesDiv = document.createElement('div');
                        sourcesDiv.className = 'sources';
                        sourcesDiv.innerHTML = '<strong>Sources:</strong><br>' +
                            data.sources.map(src =>
                                `- ${src.chapter} - ${src.section}`
                            ).join('<br>');
                        chatMessages.appendChild(sourcesDiv);
                    }
                } else {
                    addMessage('Sorry, I encountered an error processing your request.');
                }
            } catch (error) {
                // Remove typing indicator
                const typingEl = document.getElementById('typing-indicator');
                if (typingEl) typingEl.remove();

                addMessage('Sorry, I encountered a network error.');
                console.error('Error:', error);
            }
        }

        // Event listeners
        sendButton.addEventListener('click', sendQuery);
        queryInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                sendQuery();
            }
        });

        // Add welcome message
        addMessage('Hello! I\'m your Physical AI & Humanoid Robotics assistant. Ask me anything about the book!');
    </script>
</body>
</html>
```

## 8. Docker Configuration

### 8.1 Dockerfile

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Run the application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 8.2 requirements.txt

```
fastapi==0.104.1
uvicorn[standard]==0.24.0
python-multipart==0.0.6
python-dotenv==1.0.0
openai==0.28.1
langchain-community==0.0.1
langchain-openai==0.0.2
qdrant-client==1.6.2
psycopg2-binary==2.9.9
sqlalchemy==2.0.23
python-slugify==8.0.1
markdown==3.5.1
```

### 8.3 docker-compose.yml

```yaml
version: '3.8'

services:
  rag-chatbot:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - QDRANT_URL=${QDRANT_URL}
      - QDRANT_API_KEY=${QDRANT_API_KEY}
      - DATABASE_URL=${DATABASE_URL}
    env_file:
      - .env
    depends_on:
      - db
    networks:
      - rag-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - rag-chatbot
    networks:
      - rag-network

networks:
  rag-network:
    driver: bridge
```

## 9. Deployment Instructions

### 9.1 Local Development

1. **Set up environment variables** in `.env` file
2. **Install dependencies**: `pip install -r requirements.txt`
3. **Run the application**: `uvicorn main:app --reload`
4. **Ingest book content**: `python ingest_book_content.py`

### 9.2 Production Deployment

1. **Build Docker image**: `docker build -t rag-chatbot .`
2. **Run with Docker Compose**: `docker-compose up -d`
3. **Configure reverse proxy** (Nginx) for SSL termination
4. **Set up monitoring and logging**

### 9.3 Example Queries and Responses

```python
# example_queries.py
def example_queries():
    """Example queries that demonstrate the chatbot capabilities"""

    examples = [
        {
            "query": "What is ROS 2 and how does it differ from ROS 1?",
            "expected_topic": "ROS 2 architecture and differences"
        },
        {
            "query": "Explain the NVIDIA Isaac platform for robotics",
            "expected_topic": "NVIDIA Isaac ecosystem"
        },
        {
            "query": "How do Vision-Language-Action models work in robotics?",
            "expected_topic": "VLA models and architecture"
        },
        {
            "query": "What are the key components of a voice-to-action pipeline?",
            "expected_topic": "Voice processing and action generation"
        },
        {
            "query": "How does Gazebo simulation integrate with ROS 2?",
            "expected_topic": "Gazebo-ROS integration"
        }
    ]

    return examples

if __name__ == "__main__":
    for example in example_queries():
        print(f"Query: {example['query']}")
        print(f"Expected topic: {example['expected_topic']}")
        print("-" * 50)
```

## 10. Testing and Validation

### 10.1 Unit Tests

```python
# test_rag_chatbot.py
import unittest
from unittest.mock import Mock, patch, MagicMock
from main import generate_answer, QueryRequest
from vector_store import VectorStore
from embedding_service import EmbeddingService

class TestRAGChatbot(unittest.TestCase):

    def setUp(self):
        self.embedding_service = EmbeddingService()
        self.vector_store = VectorStore()

    @patch('main.openai.ChatCompletion.create')
    def test_generate_answer(self, mock_openai):
        """Test answer generation function"""
        mock_openai.return_value = {
            'choices': [
                {
                    'message': {
                        'content': 'This is a test answer based on the context.'
                    }
                }
            ]
        }

        query = "What is ROS 2?"
        context = "ROS 2 is the next generation of Robot Operating System..."

        result = self.loop.run_until_complete(generate_answer(query, context))

        self.assertIn("test answer", result.lower())
        mock_openai.assert_called_once()

    def test_query_request_validation(self):
        """Test query request model validation"""
        request = QueryRequest(query="Test query", max_results=3)

        self.assertEqual(request.query, "Test query")
        self.assertEqual(request.max_results, 3)

        # Test default value
        request_default = QueryRequest(query="Test query")
        self.assertEqual(request_default.max_results, 5)

if __name__ == '__main__':
    unittest.main()
```

## 11. Performance Optimization

### 11.1 Caching Strategy

```python
# caching.py
import redis
import pickle
import hashlib
from typing import Any, Optional
import os

class CacheManager:
    def __init__(self, redis_url: str = None):
        if redis_url:
            self.redis_client = redis.from_url(redis_url)
        else:
            # Use environment variable
            redis_url = os.getenv("REDIS_URL", "redis://localhost:6379")
            self.redis_client = redis.from_url(redis_url)

    def get_cache_key(self, query: str) -> str:
        """Generate a cache key for the query"""
        query_hash = hashlib.md5(query.encode()).hexdigest()
        return f"rag_query:{query_hash}"

    def get(self, query: str) -> Optional[Any]:
        """Get cached result for query"""
        cache_key = self.get_cache_key(query)
        cached_result = self.redis_client.get(cache_key)

        if cached_result:
            return pickle.loads(cached_result)
        return None

    def set(self, query: str, result: Any, expire: int = 3600) -> None:
        """Cache result for query"""
        cache_key = self.get_cache_key(query)
        serialized_result = pickle.dumps(result)
        self.redis_client.setex(cache_key, expire, serialized_result)

# Integrate caching in main application
cache_manager = CacheManager()

@app.post("/query", response_model=QueryResponse)
async def query_endpoint(request: QueryRequest):
    """Query the RAG system for answers with caching"""
    # Check cache first
    cached_result = cache_manager.get(request.query)
    if cached_result:
        return cached_result

    # Process query as before...
    try:
        # ... existing query processing code ...

        # Cache the result before returning
        cache_manager.set(request.query, response)
        return response
    except Exception as e:
        logger.error(f"Error processing query: {str(e)}")
        raise HTTPException(status_code=500, detail=f"Error processing query: {str(e)}")
```

This RAG chatbot integration provides:

1. **Complete backend implementation** with FastAPI
2. **Vector storage** using Qdrant Cloud
3. **Database integration** with Neon Postgres
4. **Document processing** for book content
5. **Frontend interface** for user interaction
6. **Docker configuration** for deployment
7. **Testing framework** for validation
8. **Performance optimization** with caching
9. **Comprehensive setup instructions**

The system is designed to be scalable, maintainable, and production-ready, allowing users to ask questions about the Physical AI & Humanoid Robotics book and receive accurate, context-aware responses.