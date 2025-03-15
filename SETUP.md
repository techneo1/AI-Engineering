# Project Setup Guide

## 1. Install VS Code  
Download and install [VS Code](https://code.visualstudio.com/)

## 2. Install **uv** (Python package/dependency manager)  
Install [uv](https://github.com/astral-sh/uv)  

### For macOS/Linux (run in terminal):
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### For Windows (run in PowerShell):
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

## 3. Install Python (using **uv**)  
To install Python 3.12:  
```bash
uv python install 3.12
```

## 4. Create and Activate Virtual Environment  

### Create Virtual Environment:  
```bash
uv venv --python 3.12
```

### Activate Virtual Environment:  

#### macOS/Linux:
```bash
source .venv/bin/activate
```

#### Windows:
```powershell
.\.venv\Scripts\activate
```

## 5. Install Required Packages  
```bash
uv pip install -r ./requirements.txt
```

## 6. Setup Groq Account  

1. Visit [Groq Cloud](https://groq.com/)
2. Create an account or login.
3. Navigate to **"API Keys"** → **"Create API Key"**.
4. Copy your newly generated API key.

## 7. Setup Environment Variables  

### Create `.env` file in project root:  

#### macOS/Linux:
```bash
touch .env
echo "GROQ_API_KEY=your_api_key_here" > .env
```

#### Windows (Command Prompt):
```cmd
echo GROQ_API_KEY=your_api_key_here > .env
```

> ⚠️ **Replace `your_api_key_here` with your actual Groq API key.**

## 8. Setup Back-end using FastAPI
#### Copy below content to back-end/main.py
```bash
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import os
from groq import Groq
import uuid
from fastapi.middleware.cors import CORSMiddleware


from dotenv import load_dotenv

load_dotenv()
# Initialize FastAPI app
app = FastAPI()

from fastapi.middleware.cors import CORSMiddleware

# Add after creating the FastAPI app
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",  # React development server
        "http://127.0.0.1:3000",   # Alternative local address
        # Add production URLs here when deployed
        # "https://your-production-domain.com",
    ],
    allow_credentials=True,
    allow_methods=["*"],  # Allows all methods
    allow_headers=["*"],  # Allows all headers
    expose_headers=["*"]
)
# Initialize Groq client
client = Groq(api_key=os.environ.get("GROQ_API_KEY"))

# System prompt configuration
system_prompt = {
    "role": "system",
    "content": "You are a helpful assistant. You reply with very short answers."
}

# In-memory storage for chat sessions
sessions = {}

# Pydantic models
class ChatRequest(BaseModel):
    message: str
    session_id: str

class ChatResponse(BaseModel):
    response: str
    session_id: str

class SessionResponse(BaseModel):
    session_id: str

# Endpoints
@app.post("/start-session", response_model=SessionResponse)
def start_session():
    """Create a new chat session"""
    session_id = str(uuid.uuid4())
    sessions[session_id] = [system_prompt.copy()]
    return {"session_id": session_id}

@app.post("/chat", response_model=ChatResponse)
def chat(request: ChatRequest):
    """Process a user message and return assistant response"""
    session_id = request.session_id
    
    # Validate session
    if session_id not in sessions:
        raise HTTPException(status_code=404, detail="Session not found")
    
    chat_history = sessions[session_id]
    
    # Add user message to history
    chat_history.append({"role": "user", "content": request.message})
    
    try:
        # Get AI response
        response = client.chat.completions.create(
            model="llama3-70b-8192",
            messages=chat_history,
            max_tokens=100,
            temperature=1.2
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
    
    # Extract and store assistant response
    assistant_message = response.choices[0].message.content
    chat_history.append({"role": "assistant", "content": assistant_message})
    
    return {"response": assistant_message, "session_id": session_id}
```

#### Run main.py
```bash
~/AI-Engineering/back-end $ uvicorn main:app --reload
``` 

## 9. Run Fast API
#### 1. Start a new session:
```bash
   curl -X POST http://localhost:8000/start-session
```
Returns: {"session_id": "uuid-here"}

#### 2. Send messages:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"message": "Capital City of Karnataka", "session_id": "your-session-id"}' http://localhost:8000/chat
```
Returns: {"response": "Bengaluru", "session_id": "your-session-id"}

## 10. Setup Frontend using Reactjs

#### 1. Create ReactJs application and install all the required dependencies:
```bash
~/Projects/AI-Engineering$ npx create-react-app frontend
~/Projects/AI-Engineering$ cd frontend
AI-Engineering/frontend$ npm install axios react-bootstrap bootstrap @popperjs/core react-syntax-highlighter
```
#### 2. Copy below code into App.js
```bash 
// src/App.js
import { useState, useEffect } from 'react';
import axios from 'axios';
import { Prism as SyntaxHighlighter } from 'react-syntax-highlighter';
import { atomDark } from 'react-syntax-highlighter/dist/esm/styles/prism';
import { 
  Container, 
  Card, 
  Form, 
  Button, 
  Alert, 
  Badge,
  Stack,
  ListGroup,
  Spinner
} from 'react-bootstrap';

function App() {
  // State declarations
  const [sessionId, setSessionId] = useState('');
  const [message, setMessage] = useState('');
  const [chatHistory, setChatHistory] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState('');

  // Start new session
  const startNewSession = async () => {
    try {
      const response = await axios.post('http://localhost:8000/start-session');
      setSessionId(response.data.session_id);
      setChatHistory([]);
      setError('');
    } catch (err) {
      setError('Failed to start new session');
    }
  };

  // Handle form submission
  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!message.trim() || !sessionId) return;

    setIsLoading(true);
    setError('');

    try {
      setChatHistory(prev => [...prev, { role: 'user', content: message }]);
      
      const response = await axios.post('http://localhost:8000/chat', {
        message: message,
        session_id: sessionId
      });

      setChatHistory(prev => [...prev, { 
        role: 'assistant', 
        content: response.data.response 
      }]);
      
      setMessage('');
    } catch (err) {
      setError(err.response?.data?.detail || 'Failed to send message');
    } finally {
      setIsLoading(false);
    }
  };

  // Initialize session on mount
  useEffect(() => {
    startNewSession();
  }, []); // eslint-disable-line react-hooks/exhaustive-deps

  // Format content with syntax highlighting
  const formatContent = (content) => {
    const codeBlocks = content.split(/```(\w+)?/);
    return codeBlocks.map((block, index) => {
      if (index % 2 === 1) {
        const language = codeBlocks[index] || 'javascript';
        return (
          <SyntaxHighlighter 
            key={index}
            language={language}
            style={atomDark}
            className="code-block"
          >
            {codeBlocks[index + 1]}
          </SyntaxHighlighter>
        );
      }
      return <div key={index} className="text-content">{block}</div>;
    });
  };

  return (
    <Container className="py-4">
      <h1 className="text-center mb-4">AI Chat Assistant</h1>
      <Card className="shadow-lg">
        <Card.Body className="p-0">
          <div className="chat-messages" style={{ height: '60vh', overflowY: 'auto', padding: '1rem' }}>
            <ListGroup variant="flush">
              {chatHistory.map((msg, index) => (
                <ListGroup.Item 
                  key={index}
                  className={`d-flex flex-column ${msg.role === 'user' ? 'align-items-end' : ''}`}
                  style={{ border: 'none', background: 'transparent' }}
                >
                  <Card 
                    className={`mb-2 ${msg.role === 'user' ? 'bg-primary text-white' : 'bg-light'}`}
                    style={{ maxWidth: '80%' }}
                  >
                    <Card.Body>
                      <Card.Subtitle className="mb-2 text-muted small">
                        {msg.role === 'user' ? 'You' : 'Assistant'}
                      </Card.Subtitle>
                      <div className="message-content">
                        {formatContent(msg.content)}
                      </div>
                    </Card.Body>
                  </Card>
                </ListGroup.Item>
              ))}
              {isLoading && (
                <ListGroup.Item className="d-flex justify-content-center">
                  <Spinner animation="border" role="status" variant="primary">
                    <span className="visually-hidden">Loading...</span>
                  </Spinner>
                </ListGroup.Item>
              )}
            </ListGroup>
          </div>

          <Form onSubmit={handleSubmit} className="p-3 border-top">
            <Stack direction="horizontal" gap={3}>
              <Form.Control
                type="text"
                value={message}
                onChange={(e) => setMessage(e.target.value)}
                placeholder="Type your message..."
                disabled={isLoading}
              />
              <Button 
                variant="primary" 
                type="submit" 
                disabled={isLoading}
                style={{ minWidth: '100px' }}
              >
                {isLoading ? 'Sending...' : 'Send'}
              </Button>
            </Stack>
          </Form>
        </Card.Body>

        <Card.Footer className="d-flex justify-content-between align-items-center">
          <Badge bg="secondary">Session ID: {sessionId}</Badge>
          <Button 
            variant="outline-primary" 
            onClick={startNewSession}
            size="sm"
          >
            New Session
          </Button>
        </Card.Footer>
      </Card>

      {error && (
        <Alert variant="danger" className="mt-3">
          {error}
        </Alert>
      )}
    </Container>
  );
}

export default App;
```

#### 3. Paste below code under src/index.js
```bash
import 'bootstrap/dist/css/bootstrap.min.css';
```

#### 4. Start the development server
```bash
npm start
```

---

✅ You're all set! Now you can start working on the project.
