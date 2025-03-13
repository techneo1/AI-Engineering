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

## 8. Setup Backend using FastAPI
#### Copy content of main.py under Backend/main.py
```bash
~/AI-Engineering/Backend $ uvicorn main:app --reload
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



---

✅ You're all set! Now you can start working on the project.
