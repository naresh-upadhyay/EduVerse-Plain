# EduSHAMIIT AI — Complete Backend Specification (Distributed System)

> **PURPOSE**: SELF-CONTAINED BLUEPRINT for building the EduSHAMIIT backend — a distributed Python FastAPI system serving the Flutter app. Designed to scale from 100 to 1,000,000+ users.

> **PRODUCT**: EduSHAMIIT
> **AI ASSISTANT NAME**: Shami
> **MARKET**: Indian school/college SaaS
> **BACKEND**: Python 3.11+ / FastAPI / LangChain / Supabase (PostgreSQL)
> **MULTI-SCHOOL**: All data partitioned by `school_id`
> **SCALING**: Distributed architecture with Docker Compose + Kubernetes

---

## 📋 TABLE OF CONTENTS

1. [Architecture Overview](#1-architecture-overview)
2. [Distributed System Design](#2-distributed-system-design)
3. [Docker Compose Setup](#3-docker-compose-setup)
4. [Database Partitioning (school_id)](#4-database-partitioning)
5. [Complete REST API Endpoints](#5-complete-rest-api-endpoints)
6. [AI Tools (40 Tools)](#6-ai-tools)
7. [RAG Pipeline](#7-rag-pipeline)
8. [IoT Device Control](#8-iot-device-control)
9. [Security & Auth](#9-security--auth)
10. [Deployment & Scaling](#10-deployment--scaling)

---

## 1. ARCHITECTURE OVERVIEW

```
                         ┌─────────────────┐
                         │   Flutter App   │
                         │  (Web/Mobile)   │
                         └────────┬────────┘
                                  │ HTTPS
                                  ▼
                    ┌─────────────────────────────┐
                    │        Nginx Load Balancer  │
                    │     (rate limiting, SSL)     │
                    └──────────────┬──────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
         ┌────┴────┐         ┌────┴────┐         ┌────┴────┐
         │FastAPI 1│         │FastAPI 2│         │FastAPI N│
         │ (REST)  │         │ (REST)  │         │ (REST)  │
         └────┬────┘         └────┬────┘         └────┬────┘
              │                    │                    │
    ┌─────────┴────────────────────┴────────────────────┴─────────┐
    │                    Service Layer                             │
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐  │
    │  │  Auth   │  │Academic │  │  Comms  │  │ AI Service  │  │
    │  │ Service │  │ Service │  │ Service │  │  (LangChain)│  │
    │  └─────────┘  └─────────┘  └─────────┘  └─────────────┘  │
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐  │
    │  │ Payment │  │  File   │  │  IoT    │  │  Camera AI  │  │
    │  │ Service │  │ Service │  │ Service │  │  Service    │  │
    │  └─────────┘  └─────────┘  └─────────┘  └─────────────┘  │
    └────────────────────────┬──────────────────────────────────┘
                             │
    ┌────────────────────────┼──────────────────────────────────┐
    │                    Data Layer                              │
    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
    │  │  PostgreSQL  │  │   Redis     │  │   RabbitMQ      │  │
    │  │  Primary     │  │   Cache     │  │   Queue         │  │
    │  │  (Write)     │  │  (Hot Data) │  │  (Async Tasks)  │  │
    │  └──────┬──────┘  └─────────────┘  └─────────────────┘  │
    │         │                                                 │
    │  ┌──────┴──────────────────┐                              │
    │  │   PostgreSQL Replicas   │                              │
    │  │  (Read 1)  (Read 2)     │                              │
    │  └─────────────────────────┘                              │
    └──────────────────────────────────────────────────────────┘
```

### Microservices Split

| Service | Responsibility | Port |
|---|---|---|
| `api-gateway` (nginx) | Load balancing, SSL, rate limiting | 443/80 |
| `auth-service` | Login, JWT, password reset, school config | 8001 |
| `academic-service` | Timetable, results, exams, courses, grading | 8002 |
| `communication-service` | Messages, notices, notifications, live classes | 8003 |
| `ai-service` | Shami AI agent, tools, RAG, voice, vision | 8004 |
| `payment-service` | Razorpay, fee management, salary | 8005 |
| `file-service` | Upload/download, study materials, documents | 8006 |
| `iot-service` | ESP32 control, MQTT, device management | 8007 |
| `camera-service` | YOLOv8, DeepFace, attendance camera | 8008 |

---

## 2. DISTRIBUTED SYSTEM DESIGN

### Request Flow with school_id

Every API request MUST include `school_id` in the JWT token or request body. The backend uses `school_id` for:

1. **Database partitioning** — queries hit only the relevant partition
2. **Cache scoping** — Redis keys prefixed with `school_id:`
3. **Message queue routing** — RabbitMQ exchanges per school
4. **Rate limiting** — per-school rate limits
5. **Analytics** — school-scoped metrics

### JWT Token Structure

```json
{
  "sub": "user-uuid",
  "school_id": "school-uuid",
  "role": "student",
  "class": "X-A",
  "exp": 1712345678,
  "iat": 1712259278
}
```

### Response Format (All Endpoints)

Every response includes `school_id` for fast client-side scoping:

```json
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": { ... },
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 156,
    "total_pages": 8
  },
  "timestamp": "2026-04-02T18:30:00Z"
}
```

### Error Response Format

```json
{
  "success": false,
  "school_id": "uuid-school-1",
  "error": {
    "code": "STUDENT_NOT_FOUND",
    "message": "Student not found in this school",
    "details": "The student_id does not belong to this school"
  },
  "timestamp": "2026-04-02T18:30:00Z"
}
```

---

## 3. DOCKER COMPOSE SETUP

### docker-compose.yml

```yaml
version: '3.8'

services:
  # ============================================
  # DATABASE LAYER
  # ============================================
  postgres-primary:
    image: supabase/postgres:15.1.0
    container_name: eduverse-db-primary
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: eduverse
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql
    command: >
      postgres
        -c wal_level=replica
        -c max_wal_senders=3
        -c max_replication_slots=3
        -c hot_standby=on
    networks:
      - eduverse-net

  postgres-replica-1:
    image: supabase/postgres:15.1.0
    container_name: eduverse-db-replica-1
    environment:
      POSTGRES_PRIMARY_HOST: postgres-primary
      POSTGRES_REPLICATION_USER: replicator
      POSTGRES_REPLICATION_PASSWORD: ${REPLICATION_PASSWORD}
    depends_on:
      - postgres-primary
    networks:
      - eduverse-net

  redis:
    image: redis:7-alpine
    container_name: eduverse-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes --maxmemory 512mb --maxmemory-policy allkeys-lru
    networks:
      - eduverse-net

  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: eduverse-rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - eduverse-net

  # ============================================
  # APPLICATION LAYER
  # ============================================
  api-gateway:
    image: nginx:alpine
    container_name: eduverse-gateway
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - fastapi-app-1
      - fastapi-app-2
      - fastapi-app-3
    networks:
      - eduverse-net

  fastapi-app-1:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: eduverse-api-1
    environment:
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres-primary:5432/eduverse
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://rabbitmq:5672
      - SUPABASE_URL=http://supabase-kong:8000
      - SUPABASE_SERVICE_KEY=${SUPABASE_SERVICE_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - GOOGLE_API_KEY=${GOOGLE_API_KEY}
    depends_on:
      - postgres-primary
      - redis
      - rabbitmq
    networks:
      - eduverse-net

  fastapi-app-2:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: eduverse-api-2
    environment:
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres-primary:5432/eduverse
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - postgres-primary
      - redis
      - rabbitmq
    networks:
      - eduverse-net

  fastapi-app-3:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: eduverse-api-3
    environment:
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres-primary:5432/eduverse
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - postgres-primary
      - redis
      - rabbitmq
    networks:
      - eduverse-net

  # ============================================
  # AI SERVICES
  # ============================================
  whisper-service:
    build:
      context: ./services/whisper
      dockerfile: Dockerfile
    container_name: eduverse-whisper
    ports:
      - "8009:8009"
    networks:
      - eduverse-net

  camera-service:
    build:
      context: ./services/camera
      dockerfile: Dockerfile
    container_name: eduverse-camera
    ports:
      - "8010:8010"
    networks:
      - eduverse-net

  mosquitto:
    image: eclipse-mosquitto:2
    container_name: eduverse-mqtt
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - mosquitto_data:/mosquitto/data
    networks:
      - eduverse-net

volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:
  mosquitto_data:

networks:
  eduverse-net:
    driver: bridge
```

### Backend Dockerfile

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### Nginx Configuration

```nginx
# nginx/nginx.conf
upstream fastapi_cluster {
    least_conn;
    server fastapi-app-1:8000;
    server fastapi-app-2:8000;
    server fastapi-app-3:8000;
}

server {
    listen 80;
    server_name api.eduverse.com;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=30r/m;

    location / {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://fastapi_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # SSE support
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 86400s;
    }

    location /chat/message {
        limit_req zone=api burst=10 nodelay;
        proxy_pass http://fastapi_cluster;
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 300s;
    }
}
```

---

## 4. DATABASE PARTITIONING

### Partition Strategy (school_id)

All major tables are partitioned by `school_id` using PostgreSQL LIST partitioning:

```sql
-- Create partitioned tables
CREATE TABLE profiles (
  id UUID NOT NULL,
  school_id UUID NOT NULL,
  full_name TEXT NOT NULL,
  role TEXT NOT NULL,
  class TEXT,
  -- ... other columns
  PRIMARY KEY (id, school_id)
) PARTITION BY LIST (school_id);

CREATE TABLE attendance (
  id UUID DEFAULT gen_random_uuid(),
  school_id UUID NOT NULL,
  student_id UUID NOT NULL,
  subject_id UUID NOT NULL,
  date DATE NOT NULL,
  status TEXT NOT NULL,
  PRIMARY KEY (id, school_id)
) PARTITION BY LIST (school_id);

CREATE TABLE results (
  id UUID DEFAULT gen_random_uuid(),
  school_id UUID NOT NULL,
  student_id UUID NOT NULL,
  subject_id UUID NOT NULL,
  marks_obtained DECIMAL(5,2),
  max_marks DECIMAL(5,2),
  PRIMARY KEY (id, school_id)
) PARTITION BY LIST (school_id);

CREATE TABLE fees (
  id UUID DEFAULT gen_random_uuid(),
  school_id UUID NOT NULL,
  student_id UUID NOT NULL,
  amount DECIMAL(10,2),
  status TEXT,
  PRIMARY KEY (id, school_id)
) PARTITION BY LIST (school_id);

CREATE TABLE homework (
  id UUID DEFAULT gen_random_uuid(),
  school_id UUID NOT NULL,
  teacher_id UUID NOT NULL,
  title TEXT NOT NULL,
  target_class TEXT NOT NULL,
  PRIMARY KEY (id, school_id)
) PARTITION BY LIST (school_id);

CREATE TABLE messages (
  id UUID DEFAULT gen_random_uuid(),
  school_id UUID NOT NULL,
  sender_id UUID NOT NULL,
  content TEXT NOT NULL,
  PRIMARY KEY (id, school_id)
) PARTITION BY LIST (school_id);

CREATE TABLE notifications (
  id UUID DEFAULT gen_random_uuid(),
  school_id UUID NOT NULL,
  user_id UUID NOT NULL,
  title TEXT NOT NULL,
  PRIMARY KEY (id, school_id)
) PARTITION BY LIST (school_id);

-- Create partition for each school (auto-generated)
-- Example for school-uuid-1:
CREATE TABLE profiles_school_1 PARTITION OF profiles
  FOR VALUES IN ('school-uuid-1');
CREATE TABLE attendance_school_1 PARTITION OF attendance
  FOR VALUES IN ('school-uuid-1');
CREATE TABLE results_school_1 PARTITION OF results
  FOR VALUES IN ('school-uuid-1');
CREATE TABLE fees_school_1 PARTITION OF fees
  FOR VALUES IN ('school-uuid-1');
CREATE TABLE homework_school_1 PARTITION OF homework
  FOR VALUES IN ('school-uuid-1');
CREATE TABLE messages_school_1 PARTITION OF messages
  FOR VALUES IN ('school-uuid-1');
CREATE TABLE notifications_school_1 PARTITION OF notifications
  FOR VALUES IN ('school-uuid-1');
```

### Auto-Create Partition Function

```sql
-- Function to auto-create partitions when new school is added
CREATE OR REPLACE FUNCTION create_school_partitions(school_uuid UUID)
RETURNS VOID AS $$
BEGIN
  EXECUTE format('CREATE TABLE IF NOT EXISTS profiles_%s PARTITION OF profiles FOR VALUES IN (%L)',
    replace(school_uuid::text, '-', '_'), school_uuid);
  EXECUTE format('CREATE TABLE IF NOT EXISTS attendance_%s PARTITION OF attendance FOR VALUES IN (%L)',
    replace(school_uuid::text, '-', '_'), school_uuid);
  EXECUTE format('CREATE TABLE IF NOT EXISTS results_%s PARTITION OF results FOR VALUES IN (%L)',
    replace(school_uuid::text, '-', '_'), school_uuid);
  EXECUTE format('CREATE TABLE IF NOT EXISTS fees_%s PARTITION OF fees FOR VALUES IN (%L)',
    replace(school_uuid::text, '-', '_'), school_uuid);
  EXECUTE format('CREATE TABLE IF NOT EXISTS homework_%s PARTITION OF homework FOR VALUES IN (%L)',
    replace(school_uuid::text, '-', '_'), school_uuid);
  EXECUTE format('CREATE TABLE IF NOT EXISTS messages_%s PARTITION OF messages FOR VALUES IN (%L)',
    replace(school_uuid::text, '-', '_'), school_uuid);
  EXECUTE format('CREATE TABLE IF NOT EXISTS notifications_%s PARTITION OF notifications FOR VALUES IN (%L)',
    replace(school_uuid::text, '-', '_'), school_uuid);
END;
$$ LANGUAGE plpgsql;

-- Trigger: auto-create partitions when new school is added
CREATE OR REPLACE FUNCTION trigger_create_partitions()
RETURNS TRIGGER AS $$
BEGIN
  PERFORM create_school_partitions(NEW.id);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER school_partition_trigger
  AFTER INSERT ON schools
  FOR EACH ROW
  EXECUTE FUNCTION trigger_create_partitions();
```

### Cache Strategy (Redis)

```python
# app/cache/redis_client.py
import redis
import json
import os

redis_client = redis.Redis.from_url(os.getenv("REDIS_URL"), decode_responses=True)

def cache_key(school_id: str, resource: str, resource_id: str = "") -> str:
    """Generate school-scoped cache key"""
    return f"{school_id}:{resource}:{resource_id}"

async def get_cached(school_id: str, resource: str, resource_id: str = ""):
    key = cache_key(school_id, resource, resource_id)
    data = redis_client.get(key)
    return json.loads(data) if data else None

async def set_cached(school_id: str, resource: str, data: dict, resource_id: str = "", ttl: int = 300):
    key = cache_key(school_id, resource, resource_id)
    redis_client.setex(key, ttl, json.dumps(data))

async def invalidate_school_cache(school_id: str, resource: str = "*"):
    pattern = f"{school_id}:{resource}:*"
    for key in redis_client.scan_iter(match=pattern):
        redis_client.delete(key)
```

---

## 5. COMPLETE REST API ENDPOINTS

### 5.1 Authentication Endpoints

#### POST /api/auth/login
```json
// Request
{
  "email": "rohan@school.com",
  "password": "password123",
  "role": "student"
}

// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": "uuid-user-1",
      "full_name": "Rohan Sharma",
      "role": "student",
      "class": "X-A",
      "school_id": "uuid-school-1",
      "avatar_url": "https://..."
    }
  }
}
```

#### POST /api/auth/refresh
#### POST /api/auth/forgot-password
#### POST /api/auth/reset-password

### 5.2 Student Dashboard Endpoints

#### GET /api/student/dashboard?user_id={id}&school_id={school_id}
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "user": {
      "full_name": "Rohan Sharma",
      "class": "X-A",
      "xp_points": 2450,
      "learning_streak": 18,
      "avatar_url": "https://..."
    },
    "stats": {
      "attendance_pct": 94.0,
      "avg_score": 91.4,
      "class_rank": 3,
      "xp_points": 2450
    },
    "today_schedule": [
      {
        "subject": "Mathematics",
        "icon": "📐",
        "start_time": "08:00",
        "end_time": "08:45",
        "room": "101",
        "is_now": true
      }
    ],
    "pending_homework": [
      {
        "id": "uuid-hw-1",
        "title": "Trigonometry Problems",
        "subject": "Mathematics",
        "icon": "📐",
        "due_date": "2026-04-05",
        "status": "due_soon"
      }
    ],
    "quick_access": [
      {"title": "Timetable", "icon": "📅", "route": "/student/timetable"}
    ]
  }
}
```

### 5.3 Timetable Endpoints

#### GET /api/student/timetable?class=X-A&day=monday&school_id={school_id}
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "class": "X-A",
    "day": "Monday",
    "schedule": [
      {
        "start_time": "08:00",
        "end_time": "08:45",
        "subject": "Mathematics",
        "icon": "📐",
        "room": "101",
        "teacher": "Mr. Sharma"
      },
      {
        "start_time": "08:45",
        "end_time": "09:00",
        "is_break": true,
        "break_name": "Short Break"
      }
    ]
  }
}
```

#### GET /api/teacher/timetable?teacher_id={id}&school_id={school_id}
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "teacher_id": "uuid-teacher-1",
    "day": "Monday",
    "schedule": [
      {
        "subject": "Mathematics",
        "class": "X-A",
        "icon": "📐",
        "start_time": "08:00",
        "end_time": "08:45",
        "room": "101"
      }
    ]
  }
}
```

### 5.4 Results Endpoints

#### GET /api/student/results?student_id={id}&school_id={school_id}&category=Mid-Term
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "student_id": "uuid-user-1",
    "overall": {
      "avg_score": 91.4,
      "grade": "A+",
      "total_exams": 12
    },
    "results": [
      {
        "subject": "Mathematics",
        "icon": "📐",
        "exam_category": "Mid-Term",
        "marks_obtained": 95,
        "max_marks": 100,
        "grade": "A+",
        "date": "2026-03-15"
      }
    ]
  }
}
```

### 5.5 Exam Endpoints

#### GET /api/student/exams?class=X-A&school_id={school_id}
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "next_exam_days": 5,
    "exams": [
      {
        "id": "uuid-exam-1",
        "title": "Mathematics Mid-Term",
        "subject": "Mathematics",
        "icon": "📐",
        "date": "2026-04-10",
        "time": "09:00",
        "duration": 90,
        "venue": "Exam Hall A",
        "total_marks": 100,
        "type": "offline",
        "status": "upcoming"
      }
    ]
  }
}
```

#### POST /api/teacher/exams/create
```json
// Request
{
  "school_id": "uuid-school-1",
  "teacher_id": "uuid-teacher-1",
  "subject_id": "uuid-subject-1",
  "title": "Mathematics Mid-Term",
  "exam_type": "offline",
  "exam_date": "2026-04-10",
  "start_time": "09:00",
  "duration_minutes": 90,
  "total_marks": 100,
  "venue": "Exam Hall A",
  "target_classes": ["X-A", "X-B"]
}

// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "exam_id": "uuid-exam-1",
    "message": "Exam created successfully"
  }
}
```

#### POST /api/teacher/exams/generate-questions
```json
// Request
{
  "school_id": "uuid-school-1",
  "teacher_id": "uuid-teacher-1",
  "subject": "Mathematics",
  "topic": "Trigonometry",
  "num_mcq": 10,
  "num_subjective": 5,
  "difficulty": "medium",
  "total_marks": 100
}

// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "questions": [
      {
        "number": 1,
        "section": "objective",
        "type": "mcq",
        "text": "What is sin(30°)?",
        "options": ["0.5", "0.707", "0.866", "1.0"],
        "correct_answer": 0,
        "marks": 4
      }
    ]
  }
}
```

### 5.6 Fee Endpoints

#### GET /api/student/fees?student_id={id}&school_id={school_id}
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "total_outstanding": 12500.00,
    "total_paid": 37500.00,
    "pending_fees": [
      {
        "id": "uuid-fee-1",
        "fee_type": "Tuition Fee",
        "amount": 12500.00,
        "amount_paid": 0,
        "due_date": "2026-04-15",
        "status": "pending"
      }
    ],
    "recent_payments": [
      {
        "amount": 12500.00,
        "method": "UPI",
        "date": "2026-03-01",
        "transaction_id": "TXN123456"
      }
    ]
  }
}
```

#### POST /api/student/fees/create-payment-link
```json
// Request
{
  "school_id": "uuid-school-1",
  "student_id": "uuid-user-1",
  "fee_id": "uuid-fee-1",
  "amount": 12500.00
}

// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "payment_url": "https://rzp.io/i/abc123",
    "expires_at": "2026-04-15T23:59:59Z"
  }
}
```

### 5.7 Homework Endpoints

#### GET /api/student/homework?class=X-A&status=pending&school_id={school_id}
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "homework": [
      {
        "id": "uuid-hw-1",
        "title": "Trigonometry Problems Ch-8",
        "subject": "Mathematics",
        "icon": "📐",
        "due_date": "2026-04-05",
        "max_marks": 25,
        "status": "pending",
        "submission_status": null
      }
    ]
  }
}
```

#### POST /api/student/homework/submit
```json
// Request
{
  "school_id": "uuid-school-1",
  "student_id": "uuid-user-1",
  "homework_id": "uuid-hw-1",
  "submission_text": "Here is my solution...",
  "attachment_url": "https://storage.supabase.co/..."
}

// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "submission_id": "uuid-sub-1",
    "message": "Homework submitted successfully"
  }
}
```

#### POST /api/teacher/homework/create
```json
// Request
{
  "school_id": "uuid-school-1",
  "teacher_id": "uuid-teacher-1",
  "subject_id": "uuid-subject-1",
  "title": "Trigonometry Problems Ch-8",
  "description": "Solve problems 1-20 from Chapter 8",
  "due_date": "2026-04-10T23:59:00",
  "max_marks": 25,
  "target_class": "X-A"
}
```

### 5.8 Attendance Endpoints

#### GET /api/student/attendance?student_id={id}&school_id={school_id}
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "overall_pct": 94.0,
    "present_days": 94,
    "absent_days": 4,
    "late_days": 2,
    "total_days": 100,
    "subject_wise": [
      {"subject": "Mathematics", "present": 19, "total": 20, "pct": 95.0}
    ]
  }
}
```

#### POST /api/teacher/attendance/mark
```json
// Request
{
  "school_id": "uuid-school-1",
  "teacher_id": "uuid-teacher-1",
  "class": "X-A",
  "subject_id": "uuid-subject-1",
  "date": "2026-04-02",
  "attendance_records": [
    {"student_id": "uuid-s1", "status": "present"},
    {"student_id": "uuid-s2", "status": "absent"}
  ]
}
```

### 5.9 Notice Endpoints

#### GET /api/student/notices?school_id={school_id}&category=General
#### POST /api/teacher/notices/create
```json
// Request
{
  "school_id": "uuid-school-1",
  "title": "Holiday Notice - Eid",
  "content": "School will remain closed on 10th April...",
  "category": "General",
  "author_id": "uuid-teacher-1",
  "is_urgent": false
}
```

### 5.10 Transport Endpoints

#### GET /api/student/transport?student_id={id}&school_id={school_id}
```json
// Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "route": {
      "name": "Route 5 - Dehradun",
      "bus_number": "UK 07 AB 1234",
      "driver_name": "Mr. Singh",
      "driver_phone": "+91-9876543210"
    },
    "your_stop": "ISBT Dehradun",
    "live_location": {
      "latitude": 30.3165,
      "longitude": 78.0322,
      "speed": 35.5,
      "eta_minutes": 12
    }
  }
}
```

### 5.11 Messaging Endpoints

#### GET /api/messages?user_id={id}&school_id={school_id}
#### POST /api/messages/send
```json
// Request
{
  "school_id": "uuid-school-1",
  "sender_id": "uuid-user-1",
  "receiver_id": "uuid-user-2",
  "content": "Hello, can you help with Math homework?"
}
```

#### GET /api/groups?user_id={id}&school_id={school_id}
#### POST /api/groups/create
```json
// Request
{
  "school_id": "uuid-school-1",
  "name": "X-A Study Group",
  "description": "For Math and Physics discussions",
  "created_by": "uuid-user-1"
}
```

### 5.12 Live Class Endpoints

#### GET /api/student/live-classes?class=X-A&school_id={school_id}
#### POST /api/teacher/live-classes/start
```json
// Request
{
  "school_id": "uuid-school-1",
  "teacher_id": "uuid-teacher-1",
  "subject_id": "uuid-subject-1",
  "title": "Trigonometry Live Class",
  "scheduled_at": "2026-04-02T10:00:00Z",
  "duration_minutes": 60,
  "target_class": "X-A"
}
```

### 5.13 Teacher-Specific Endpoints

#### GET /api/teacher/dashboard?teacher_id={id}&school_id={school_id}
#### GET /api/teacher/classes?teacher_id={id}&school_id={school_id}
#### GET /api/teacher/class-detail?class=X-A&school_id={school_id}
#### GET /api/teacher/submissions?teacher_id={id}&school_id={school_id}
#### POST /api/teacher/submissions/grade
```json
// Request
{
  "school_id": "uuid-school-1",
  "submission_id": "uuid-sub-1",
  "marks": 22,
  "grade": "A",
  "remarks": "Excellent work!"
}
```

#### GET /api/teacher/gradebook?class=X-A&subject_id={id}&school_id={school_id}
#### PUT /api/teacher/grading-config
```json
// Request
{
  "school_id": "uuid-school-1",
  "teacher_id": "uuid-teacher-1",
  "class": "X-A",
  "subject_id": "uuid-subject-1",
  "mid_term_weight": 30,
  "final_term_weight": 40,
  "attendance_weight": 5,
  "assignment_weight": 10,
  "class_test_weight": 10,
  "lab_weight": 5
}
```

#### GET /api/teacher/salary?teacher_id={id}&school_id={school_id}
#### POST /api/teacher/leave/apply
#### POST /api/teacher/materials/upload
```json
// Request
{
  "school_id": "uuid-school-1",
  "teacher_id": "uuid-teacher-1",
  "title": "Trigonometry Notes",
  "material_type": "Notes",
  "target_class": "X-A",
  "attachment_urls": ["https://storage.supabase.co/..."]
}
```

### 5.14 User Settings Endpoints

#### GET /api/user/settings?user_id={id}&school_id={school_id}
#### PUT /api/user/settings
```json
// Request
{
  "school_id": "uuid-school-1",
  "user_id": "uuid-user-1",
  "push_notifications": true,
  "dark_mode": false,
  "language": "en"
}
```

### 5.15 Notification Endpoints

#### GET /api/notifications?user_id={id}&school_id={school_id}
#### PUT /api/notifications/{id}/read

### 5.16 Leave Endpoints

#### GET /api/leave?user_id={id}&school_id={school_id}
#### POST /api/student/leave/apply
```json
// Request
{
  "school_id": "uuid-school-1",
  "student_id": "uuid-user-1",
  "leave_type": "sick",
  "start_date": "2026-04-05",
  "end_date": "2026-04-06",
  "reason": "Not feeling well"
}
```

### 5.17 Achievement Endpoints

#### GET /api/student/achievements?student_id={id}&school_id={school_id}
#### GET /api/student/leaderboard?class=X-A&school_id={school_id}

### 5.18 Event Endpoints

#### GET /api/student/events?school_id={school_id}
#### POST /api/student/events/{id}/register

### 5.19 Library Endpoints

#### GET /api/student/library?student_id={id}&school_id={school_id}
#### GET /api/student/library/books?school_id={school_id}

### 5.20 Profile Endpoints

#### GET /api/student/profile?student_id={id}&school_id={school_id}
#### PUT /api/student/profile
#### GET /api/teacher/profile?teacher_id={id}&school_id={school_id}
#### PUT /api/teacher/profile

---

## 6. AI TOOLS (40 TOOLS)

### 6.1 Student AI Tools (20)

All tools are implemented as LangChain Python tools. See Python implementations below.

| # | Tool | Supabase Query | school_id Scope |
|---|---|---|---|
| 1 | get_timetable | SELECT * FROM timetable WHERE class=? AND day=? | ✅ |
| 2 | get_homework | SELECT * FROM homework WHERE target_class=? | ✅ |
| 3 | get_fee_status | SELECT * FROM fees WHERE student_id=? | ✅ |
| 4 | get_attendance | SELECT * FROM attendance WHERE student_id=? | ✅ |
| 5 | get_exam_info | SELECT * FROM exams WHERE target_classes CONTAINS ? | ✅ |
| 6 | get_bus_location | SELECT * FROM bus_locations WHERE route_id=? | ✅ |
| 7 | get_performance | Aggregate from results + attendance | ✅ |
| 8 | generate_study_plan | LLM generates from exam data + weak areas | ✅ |
| 9 | explain_concept | LLM generates explanation | N/A |
| 10 | get_notifications | SELECT * FROM notifications WHERE user_id=? | ✅ |
| 11 | get_library_status | SELECT * FROM library_borrows WHERE student_id=? | ✅ |
| 12 | get_achievements | SELECT * FROM student_achievements WHERE student_id=? | ✅ |
| 13 | get_events | SELECT * FROM events WHERE event_date >= today | ✅ |
| 14 | get_leaderboard | SELECT * FROM profiles WHERE class=? ORDER BY xp_points | ✅ |
| 15 | submit_homework | INSERT INTO homework_submissions | ✅ |
| 16 | apply_leave | INSERT INTO leave_applications | ✅ |
| 17 | get_notices | SELECT * FROM notices WHERE status='published' | ✅ |
| 18 | answer_general | LLM generates answer | N/A |
| 19 | generate_practice | LLM generates questions | N/A |
| 20 | get_live_class | SELECT * FROM live_classes WHERE target_class=? | ✅ |

### 6.2 Teacher AI Tools (20)

| # | Tool | Supabase Query | school_id Scope |
|---|---|---|---|
| 1 | get_class_students | SELECT * FROM profiles WHERE class=? AND role='student' | ✅ |
| 2 | get_class_performance | Aggregate from results + attendance | ✅ |
| 3 | get_at_risk_students | Aggregate from results + attendance | ✅ |
| 4 | generate_questions | LLM generates questions | N/A |
| 5 | generate_lesson_plan | LLM generates plan | N/A |
| 6 | auto_grade_homework | UPDATE homework_submissions SET status='graded' | ✅ |
| 7 | get_submission_status | SELECT * FROM homework_submissions WHERE homework_id=? | ✅ |
| 8 | generate_remedial_plan | LLM generates plan | N/A |
| 9 | create_notice | INSERT INTO notices | ✅ |
| 10 | get_attendance_stats | Aggregate from attendance | ✅ |
| 11 | get_teacher_schedule | SELECT * FROM timetable WHERE teacher_id=? | ✅ |
| 12 | get_pending_tasks | Aggregate from homework_submissions + leave_applications | ✅ |
| 13 | generate_report | LLM generates report | N/A |
| 14 | explain_pedagogy | LLM generates strategies | N/A |
| 15 | get_leave_balance | SELECT * FROM leave_applications WHERE applicant_id=? | ✅ |
| 16 | get_salary_info | SELECT * FROM salary WHERE teacher_id=? | ✅ |
| 17 | get_parent_communications | SELECT * FROM messages WHERE receiver_id=? | ✅ |
| 18 | upload_material | INSERT INTO study_materials | ✅ |
| 19 | get_exam_analytics | Aggregate from exam_sessions | ✅ |
| 20 | answer_general | LLM generates answer | N/A |

### Python Tool Implementation Examples

```python
# app/tools/student_tools.py
from langchain_core.tools import tool
from app.supabase.client import get_supabase
from app.cache.redis_client import get_cached, set_cached

@tool
def get_timetable(day: str = "today", school_id: str = "") -> str:
    """Get the student timetable for a specific day or week.
    Input: day name or 'week' for full week."""
    # Check cache first
    cached = await get_cached(school_id, "timetable", f"{get_user_class()}_{day}")
    if cached:
        return cached

    sb = get_supabase()
    user_id = get_current_user_id()
    profile = sb.table("profiles").select("class").eq("id", user_id).single().execute()
    student_class = profile.data["class"]

    day_map = {"monday":0,"tuesday":1,"wednesday":2,"thursday":3,"friday":4,"saturday":5}
    query = sb.table("timetable")\
        .select("*, subjects(name, icon, color)")\
        .eq("school_id", school_id)\
        .eq("class", student_class)\
        .order("start_time")

    if day.lower() in day_map:
        query = query.eq("day_of_week", day_map[day.lower()])

    response = query.execute()
    result = _format_timetable(response.data, day)

    # Cache for 5 minutes
    await set_cached(school_id, "timetable", result, f"{student_class}_{day}", ttl=300)
    return result

@tool
def get_fee_status(school_id: str = "") -> str:
    """Check fee balance and payment history."""
    sb = get_supabase()
    user_id = get_current_user_id()

    fees = sb.table("fees")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("student_id", user_id)\
        .order("due_date").execute().data

    # ... format response with school_id in output
    return _format_fees(fees, school_id)
```

### AI Service Endpoints (Chat)

#### POST /api/chat/message (SSE Streaming)
```json
// Request
{
  "message": "What classes do I have today?",
  "session_id": "uuid-session-1"
}

// SSE Response (streamed)
data: {"type":"text","content":"📅 Your Monday Schedule:\n"}
data: {"type":"text","content":"  📐 Mathematics - 08:00 to 08:45 (Room 101)\n"}
data: {"type":"text","content":"  ⚛️ Physics - 09:00 to 09:45 (Lab-1)\n"}
data: {"type":"tool_result","tool":"get_timetable","data":{...}}
data: {"type":"done"}
```

#### POST /api/chat/voice
#### POST /api/chat/image
#### GET /api/chat/history/{session_id}

---

## 7. RAG PIPELINE

### Document Ingestion

```python
# POST /api/rag/ingest
# Request (multipart)
{
  "school_id": "uuid-school-1",
  "subject": "Mathematics",
  "grade": "X",
  "source": "NCERT Textbook Ch-8"
}
# File: trigonometry_ncert.pdf

# Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "chunks_ingested": 47,
    "source": "NCERT Textbook Ch-8"
  }
}
```

### Curriculum Search Tool

```python
@tool
def search_curriculum(query: str, subject: str = "", grade: str = "", school_id: str = "") -> str:
    """Search the school's curriculum and study materials."""
    # school_id scoping ensures only this school's materials are searched
    filter_dict = {"school_id": school_id}
    if subject: filter_dict["subject"] = subject
    if grade: filter_dict["grade"] = grade

    docs = vector_store.similarity_search(query, k=5, filter=filter_dict)
    return "\n\n".join([f"[{d.metadata.get('source')}]\n{d.page_content}" for d in docs])
```

---

## 8. IoT DEVICE CONTROL

### ESP32 + MQTT

```python
# POST /api/iot/control
# Request
{
  "school_id": "uuid-school-1",
  "room": "class_10a",
  "device": "fan",
  "action": "on"
}

# Response
{
  "success": true,
  "school_id": "uuid-school-1",
  "data": {
    "room": "class_10a",
    "device": "fan",
    "action": "on",
    "method": "mqtt",
    "status": "success"
  }
}
```

### IoT Tools (LangChain)

```python
@tool
def control_classroom_device(room: str, device: str, action: str, school_id: str = "") -> str:
    """Control an electrical device in a classroom over WiFi."""
    result = iot.control_device(room, device, action)
    if result["ok"]:
        return f"Done — {device} in {room} has been turned {action}."
    return f"Could not reach {room}. The device board may be offline."
```

---

## 9. SECURITY & AUTH

### JWT Middleware (school_id extraction)

```python
# app/middleware/auth.py
from fastapi import HTTPException, Depends
from fastapi.security import HTTPBearer
from jose import jwt, JWTError
import os

security = HTTPBearer()

async def get_current_user(token=Depends(security)):
    try:
        payload = jwt.decode(
            token.credentials,
            os.getenv("SUPABASE_JWT_SECRET"),
            algorithms=["HS256"],
            options={"verify_aud": False}
        )
        return UserContext(
            id=payload["sub"],
            school_id=payload["school_id"],  # ← ALWAYS extracted
            role=payload["role"],
            class_name=payload.get("class"),
        )
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

# Dependency injection for school_id
async def require_school_id(user: UserContext = Depends(get_current_user)):
    if not user.school_id:
        raise HTTPException(status_code=400, detail="school_id required")
    return user.school_id
```

### Security Rules
- Python backend uses `SUPABASE_SERVICE_ROLE_KEY` (server-side only)
- Flutter uses only `SUPABASE_ANON_KEY` + user JWT
- Every API request includes `school_id` from JWT
- Rate limiting per school_id
- IoT MQTT broker runs on LAN only
- Face data never leaves school premises

---

## 10. DEPLOYMENT & SCALING

### Scaling Strategy

| Users | Infrastructure |
|---|---|
| 0-1,000 | Single FastAPI + Supabase + Redis |
| 1,000-10,000 | 3 FastAPI replicas + Nginx LB + Redis |
| 10,000-100,000 | Microservices + PostgreSQL replicas + RabbitMQ |
| 100,000-1,000,000 | Kubernetes + Sharded DB + CDN + Dedicated AI servers |

### Kubernetes Deployment (Production)

```yaml
# k8s/fastapi-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eduverse-api
spec:
  replicas: 5
  selector:
    matchLabels:
      app: eduverse-api
  template:
    metadata:
      labels:
        app: eduverse-api
    spec:
      containers:
      - name: api
        image: eduverse/api:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: eduverse-api-service
spec:
  selector:
    app: eduverse-api
  ports:
  - port: 80
    targetPort: 8000
  type: LoadBalancer
```

### Environment Variables

```bash
# .env
DATABASE_URL=postgresql://postgres:password@localhost:5432/eduverse
REDIS_URL=redis://localhost:6379
RABBITMQ_URL=amqp://localhost:5672
SUPABASE_URL=http://localhost:54321
SUPABASE_SERVICE_ROLE_KEY=eyJ...
SUPABASE_JWT_SECRET=your-secret
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=AIza...
RAZORPAY_KEY_ID=rzp_...
RAZORPAY_SECRET=...
MQTT_BROKER_HOST=localhost
MQTT_BROKER_PORT=1883
```

---

*This document is the single source of truth for the EduSHAMIIT AI backend — by Shami Innovation and Technologies LLP.*

---
**Last updated:** April 2026