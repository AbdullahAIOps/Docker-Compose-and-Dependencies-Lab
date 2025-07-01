# 🔗 Lab 17: Compose with Dependencies

## 🎯 Objectives

By the end of this lab, you will be able to:

- Use `depends_on` to manage service startup order in Docker Compose
- Scale services using Docker Compose replicas
- Test service-to-service communication (e.g., web app to Redis)

---

## 📋 Prerequisites

- Docker 20.10.0+ installed
- Docker Compose 2.0.0+ installed
- Basic knowledge of Docker and YAML
- Terminal access

---

## ⚙️ Lab Setup

```bash
mkdir compose-dependencies-lab
cd compose-dependencies-lab
```

---

## 🧱 Task 1: Using `depends_on`

### 🔹 Subtask 1.1: Create `docker-compose.yml`

```yaml
version: '3.8'

services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

  webapp:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - redis
```

---

### 🔹 Subtask 1.2: Understanding `depends_on`

- Ensures Redis **starts before** Nginx
- ❗ Does *not* wait for Redis to be fully *ready*, only *started*

---

### 🔹 Subtask 1.3: Start the Services

```bash
docker-compose up -d
```

✅ Expected:
- Redis starts first
- Nginx starts after Redis
- Both services should run without error

```bash
docker-compose ps
```

---

## 📈 Task 2: Scale Service Replicas

### 🔹 Subtask 2.1: Update Compose File

```yaml
version: '3.8'

services:
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

  webapp:
    image: nginx:alpine
    ports:
      - "8080-8082:80"
    depends_on:
      - redis
    deploy:
      replicas: 3
```

---

### 🔹 Subtask 2.2: Start Scaled Services

```bash
docker-compose up -d --scale webapp=3
```

---

### 🔹 Subtask 2.3: Verify Scaling

```bash
docker-compose ps
docker-compose exec webapp hostname
```

✅ Expected:
- One Redis container
- Three Nginx webapp containers (with different hostnames)

---

## 🔌 Task 3: Test Inter-Service Connectivity

### 🔹 Subtask 3.1: Create a Test Flask App

#### 📝 `app.py`
```python
from flask import Flask
import redis
import os

app = Flask(__name__)
redis_host = os.environ.get('REDIS_HOST', 'redis')
redis_client = redis.Redis(host=redis_host, port=6379)

@app.route('/')
def hello():
    count = redis_client.incr('hits')
    return f'Hello World! This page has been viewed {count} times.\n'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

#### 🐳 `Dockerfile`
```Dockerfile
FROM python:3.9-alpine
WORKDIR /app
COPY . .
RUN pip install flask redis
CMD ["python", "app.py"]
```

---

### 🔹 Subtask 3.2: Update Compose File

```yaml
version: '3.8'

services:
  redis:
    image: redis:alpine

  webapp:
    build: .
    ports:
      - "5000:5000"
    environment:
      - REDIS_HOST=redis
    depends_on:
      - redis
```

---

### 🔹 Subtask 3.3: Build and Test

```bash
docker-compose up -d
```

Test the app in browser:  
[http://localhost:5000](http://localhost:5000)

Or with curl:
```bash
curl http://localhost:5000
```

✅ Expected Output:
```
Hello World! This page has been viewed 1 times.
```

Refresh:
```
Hello World! This page has been viewed 2 times.
```

🔁 Page counter increments with each request  
🚫 No connection errors with Redis

---

## ✅ Summary

| Task                        | Command/Tool                        | Purpose                                       |
|-----------------------------|-------------------------------------|-----------------------------------------------|
| Start services              | `docker-compose up -d`             | Launch multi-container app                    |
| Control order               | `depends_on` in YAML                | Ensure service startup order                  |
| Scale services              | `--scale webapp=3`                  | Run multiple instances of a service           |
| Test connectivity           | `curl http://localhost:5000`       | Verify Redis → Flask connection               |
| Build image from code       | `Dockerfile + build:` in Compose   | Custom Python webapp container                |

---

## 👨‍💻 Author

**Abdullha Saleem** – DevOps | Docker | Python | Flask | Compose | Redis  
📫 *Connect with me on GitHub or LinkedIn for more labs and projects.*
