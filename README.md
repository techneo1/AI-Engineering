# Project Setup Guide
#### 1. Run fastAPI Backend
```bash
Projects/backend$ uvicorn main:app --reload
```

#### 2. Run ReactJs Frontend
```bash
Projects/frontend$ npm start
```

#### 3. Start MLFLOW server for experiments
```bash
mlflow server --host 127.0.0.1 --port 5000
```

#### 4. Start MLFLOW Local tracking
```bash
mlflow ui --port 5001
```


---

✅ You're all set! Now you can start working on the project.
