# Quick Start - My ChatGPT

#### Pre-requisites: Follow this [Setup Guide](https://github.com/techneo1/AI-Engineering/blob/master/SETUP.md)

✅ After Pre-requisites, You're all set! Now you can start working on the project.

#### 1. Run fastAPI Backend
```bash
cd back-end 
uvicorn main:app --reload
```

#### 2. Run ReactJs Frontend
```bash
cd front-end
npm start
```

#### 3. Start MLFLOW server for experiments
```bash
mlflow server --host 127.0.0.1 --port 5000
```

#### 4. Start MLFLOW Local tracking
```bash
mlflow ui --port 5001
```