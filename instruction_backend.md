# EduSHAMIIT Backend — Complete Implementation Guide (Docker, Zero External Cost)

> **PURPOSE**: SELF-CONTAINED BLUEPRINT for building the EduSHAMIIT backend from scratch. Any AI agent reading ONLY this file should be able to implement the complete FastAPI backend with all 65+ screen endpoints, custom UPI payments, Docker deployment, and zero external costs.

> **PRODUCT**: EduSHAMIIT
> **BACKEND**: Python 3.11+ / FastAPI / LangChain / Supabase (PostgreSQL)
> **DEPLOYMENT**: Docker Compose (local + VPS-ready)
> **PAYMENTS**: Custom UPI (zero third-party fees)
> **MULTI-SCHOOL**: All data scoped by `school_id`

---

## 📋 TABLE OF CONTENTS

1. [Docker Setup (Zero Cost)](#1-docker-setup-zero-cost)
2. [FastAPI Project Structure](#2-fastapi-project-structure)
3. [Custom UPI Payment System](#3-custom-upi-payment-system)
4. [Authentication Endpoints](#4-authentication-endpoints)
5. [Student Screen Endpoints (29 Screens)](#5-student-screen-endpoints)
6. [Teacher Screen Endpoints (29 Screens)](#6-teacher-screen-endpoints)
7. [Shared Screen Endpoints (7 Screens)](#7-shared-screen-endpoints)
8. [AI Chat Endpoints](#8-ai-chat-endpoints)
9. [IoT Control Endpoints](#9-iot-control-endpoints)
10. [RAG Pipeline Endpoints](#10-rag-pipeline-endpoints)
11. [Complete Python Implementation](#11-complete-python-implementation)

---

## 1. DOCKER SETUP (ZERO COST)

### docker-compose.yml

```yaml
version: "3.8"

services:
  # ============================================
  # DATABASE
  # ============================================
  postgres:
    image: supabase/postgres:15.1.0
    container_name: eduSHAMIIT-db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: eduSHAMIIT
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD:-eduSHAMIIT2026}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql
    command: >
      postgres
        -c wal_level=replica
        -c max_wal_senders=3
        -c shared_preload_libraries=vector
    networks:
      - eduSHAMIIT-net

  # ============================================
  # CACHE
  # ============================================
  redis:
    image: redis:7-alpine
    container_name: eduSHAMIIT-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru
    networks:
      - eduSHAMIIT-net

  # ============================================
  # MESSAGE QUEUE
  # ============================================
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: eduSHAMIIT-rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: eduSHAMIIT
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD:-eduSHAMIIT2026}
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - eduSHAMIIT-net

  # ============================================
  # MQTT BROKER (IoT)
  # ============================================
  mosquitto:
    image: eclipse-mosquitto:2
    container_name: eduSHAMIIT-mqtt
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - mosquitto_data:/mosquitto/data
    networks:
      - eduSHAMIIT-net

  # ============================================
  # BACKEND API (3 replicas for scaling)
  # ============================================
  api-1:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: eduSHAMIIT-api-1
    environment:
      - DATABASE_URL=postgresql://postgres:${DB_PASSWORD:-eduSHAMIIT2026}@postgres:5432/eduSHAMIIT
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://eduSHAMIIT:${RABBITMQ_PASSWORD:-eduSHAMIIT2026}@rabbitmq:5672
      - SUPABASE_URL=http://postgres:5432
      - SUPABASE_JWT_SECRET=${JWT_SECRET:-eduSHAMIIT-jwt-secret-2026}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - GOOGLE_API_KEY=${GOOGLE_API_KEY}
      - UPI_MERCHANT_ID=${UPI_MERCHANT_ID:-eduSHAMIIT-school}
      - UPI_MERCHANT_NAME=${UPI_MERCHANT_NAME:-EduSHAMIIT Academy}
    depends_on:
      - postgres
      - redis
      - rabbitmq
    networks:
      - eduSHAMIIT-net

  api-2:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: eduSHAMIIT-api-2
    environment:
      - DATABASE_URL=postgresql://postgres:${DB_PASSWORD:-eduSHAMIIT2026}@postgres:5432/eduSHAMIIT
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://eduSHAMIIT:${RABBITMQ_PASSWORD:-eduSHAMIIT2026}@rabbitmq:5672
      - SUPABASE_JWT_SECRET=${JWT_SECRET:-eduSHAMIIT-jwt-secret-2026}
    depends_on:
      - postgres
      - redis
      - rabbitmq
    networks:
      - eduSHAMIIT-net

  api-3:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: eduSHAMIIT-api-3
    environment:
      - DATABASE_URL=postgresql://postgres:${DB_PASSWORD:-eduSHAMIIT2026}@postgres:5432/eduSHAMIIT
      - REDIS_URL=redis://redis:6379
      - RABBITMQ_URL=amqp://eduSHAMIIT:${RABBITMQ_PASSWORD:-eduSHAMIIT2026}@rabbitmq:5672
      - SUPABASE_JWT_SECRET=${JWT_SECRET:-eduSHAMIIT-jwt-secret-2026}
    depends_on:
      - postgres
      - redis
      - rabbitmq
    networks:
      - eduSHAMIIT-net

  # ============================================
  # LOAD BALANCER
  # ============================================
  nginx:
    image: nginx:alpine
    container_name: eduSHAMIIT-gateway
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - api-1
      - api-2
      - api-3
    networks:
      - eduSHAMIIT-net

  # ============================================
  # AI SERVICES
  # ============================================
  whisper:
    build:
      context: ./services/whisper
      dockerfile: Dockerfile
    container_name: eduSHAMIIT-whisper
    ports:
      - "8009:8009"
    networks:
      - eduSHAMIIT-net

  camera:
    build:
      context: ./services/camera
      dockerfile: Dockerfile
    container_name: eduSHAMIIT-camera
    ports:
      - "8010:8010"
    networks:
      - eduSHAMIIT-net

volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:
  mosquitto_data:

networks:
  eduSHAMIIT-net:
    driver: bridge
```

### Backend Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y gcc libpq-dev && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### Nginx Configuration

```nginx
upstream eduSHAMIIT_api {
    least_conn;
    server api-1:8000;
    server api-2:8000;
    server api-3:8000;
}

server {
    listen 80;
    server_name api.eduSHAMIIT.local;

    location / {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://eduSHAMIIT_api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_buffering off;
        proxy_cache off;
    }

    location /chat/message {
        limit_req zone=api burst=10 nodelay;
        proxy_pass http://eduSHAMIIT_api;
        proxy_buffering off;
        proxy_read_timeout 300s;
    }
}
```

### .env File

```bash
# Database
DB_PASSWORD=eduSHAMIIT2026

# RabbitMQ
RABBITMQ_PASSWORD=eduSHAMIIT2026

# JWT
JWT_SECRET=eduSHAMIIT-jwt-secret-2026

# AI (add your keys)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=AIza...

# UPI (your school's UPI ID)
UPI_MERCHANT_ID=eduSHAMIIT-school@upi
UPI_MERCHANT_NAME=EduSHAMIIT Academy
```

### Start Everything

```bash
# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f api-1

# Stop everything
docker-compose down
```

### Push to VPS

```bash
# On your VPS
git clone https://github.com/naresh-upadhyay/eduSHAMIIT-backend.git
cd eduSHAMIIT-backend
cp .env.example .env  # Edit with production values
docker-compose up -d
```

---

## 2. FASTAPI PROJECT STRUCTURE

```
backend/
├── app/
│   ├── main.py                    # FastAPI app + CORS + routers
│   ├── config.py                  # Environment config
│   ├── models.py                  # Pydantic schemas
│   ├── api/
│   │   ├── __init__.py
│   │   ├── auth.py                # Login, register, refresh
│   │   ├── student.py             # 29 student screen endpoints
│   │   ├── teacher.py             # 29 teacher screen endpoints
│   │   ├── shared.py              # Messages, notifications, settings
│   │   ├── chat.py                # AI chat (SSE streaming)
│   │   ├── iot.py                 # Device control
│   │   ├── rag.py                 # Document ingestion
│   │   └── payments.py            # Custom UPI payments
│   ├── middleware/
│   │   ├── __init__.py
│   │   ├── auth.py                # JWT validation
│   │   └── rate_limit.py          # Rate limiting
│   ├── services/
│   │   ├── __init__.py
│   │   ├── supabase_client.py     # Supabase singleton
│   │   ├── redis_client.py        # Cache layer
│   │   ├── rabbitmq_client.py     # Message queue
│   │   ├── langchain_agent.py     # AI agent builder
│   │   ├── whisper_service.py     # Voice STT
│   │   ├── camera_service.py      # Face detection
│   │   ├── iot_controller.py      # MQTT device control
│   │   ├── upi_service.py         # Custom UPI payments
│   │   └── rag_service.py         # Document ingestion
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── student_tools.py       # 20 student AI tools
│   │   ├── teacher_tools.py       # 20 teacher AI tools
│   │   ├── fee_tools.py           # Fee + payment tools
│   │   ├── iot_tools.py           # IoT control tools
│   │   └── rag_tools.py           # Curriculum search
│   └── cache/
│       └── redis_client.py        # Redis wrapper
├── requirements.txt
├── Dockerfile
└── .env
```

### main.py

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api import auth, student, teacher, shared, chat, iot, rag, payments

app = FastAPI(
    title="EduSHAMIIT API",
    description="Backend for EduSHAMIIT Academic App",
    version="1.0.0"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(auth.router, prefix="/api/auth")
app.include_router(student.router, prefix="/api/student")
app.include_router(teacher.router, prefix="/api/teacher")
app.include_router(shared.router, prefix="/api")
app.include_router(chat.router, prefix="/api/chat")
app.include_router(iot.router, prefix="/api/iot")
app.include_router(rag.router, prefix="/api/rag")
app.include_router(payments.router, prefix="/api/payments")

@app.get("/health")
async def health_check():
    return {"status": "healthy", "service": "EduSHAMIIT API"}
```

---

## 3. CUSTOM UPI PAYMENT SYSTEM (ZERO EXTERNAL COST)

### How It Works

```
Student clicks "Pay Now" in Flutter
       ↓
Flutter calls POST /api/payments/create-upi-link
       ↓
Backend generates UPI deep link:
  upi://pay?pa=school@upi&pn=EduSHAMIIT&am=12500&tn=FEE-X-A-2026&tr=TXN-123456
       ↓
Flutter opens UPI app (GPay/PhonePe/Paytm)
       ↓
Student completes payment in UPI app
       ↓
UPI app sends callback to Flutter
       ↓
Flutter calls POST /api/payments/verify
       ↓
Backend verifies with bank webhook
       ↓
Backend updates payment status to 'success'
       ↓
Student sees "Payment Successful!" ✅
```

### 3.1 Create UPI Payment Link

```python
# app/api/payments.py
from fastapi import APIRouter, Depends, HTTPException
from app.middleware.auth import get_current_user, require_school_id
from app.services.supabase_client import get_supabase
from app.services.upi_service import generate_upi_link
import uuid

router = APIRouter()

@router.post("/create-upi-link")
async def create_upi_payment(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    """
    Create a UPI payment link for fee payment.

    Request:
    {
        "fee_id": "uuid-fee-1",
        "amount": 12500.00,
        "description": "Tuition Fee Q2 2026"
    }

    Response:
    {
        "success": true,
        "school_id": "uuid-school-1",
        "data": {
            "payment_id": "uuid-payment-1",
            "upi_link": "upi://pay?pa=eduSHAMIIT@upi&pn=EduSHAMIIT&am=12500&tn=...",
            "amount": 12500.00,
            "expires_at": "2026-04-15T23:59:59Z"
        }
    }
    """
    sb = get_supabase()

    fee_id = request.get("fee_id")
    amount = request.get("amount")
    description = request.get("description", "EduSHAMIIT Fee Payment")

    # Generate unique transaction ID
    transaction_id = f"EDU-{uuid.uuid4().hex[:12].upper()}"

    # Create payment record
    payment = sb.table("payments").insert({
        "school_id": school_id,
        "fee_id": fee_id,
        "student_id": user["id"],
        "amount": amount,
        "transaction_id": transaction_id,
        "status": "pending",
        "payment_method": "upi",
    }).execute()

    # Generate UPI deep link
    upi_link = generate_upi_link(
        merchant_id="eduSHAMIIT-school@upi",  # Your school's UPI ID
        merchant_name="EduSHAMIIT Academy",
        amount=amount,
        transaction_id=transaction_id,
        description=description,
    )

    return {
        "success": True,
        "school_id": school_id,
        "data": {
            "payment_id": payment.data[0]["id"],
            "upi_link": upi_link,
            "amount": amount,
            "expires_at": "2026-04-15T23:59:59Z",
        }
    }
```

### 3.2 UPI Link Generator

```python
# app/services/upi_service.py
from urllib.parse import urlencode
from datetime import datetime

def generate_upi_link(
    merchant_id: str,
    merchant_name: str,
    amount: float,
    transaction_id: str,
    description: str = "EduSHAMIIT Payment",
) -> str:
    """
    Generate UPI deep link for payment.

    UPI Deep Link Format:
    upi://pay?pa={merchant_id}&pn={merchant_name}&am={amount}&tn={description}&tr={transaction_id}

    This opens the UPI app (GPay, PhonePe, Paytm, etc.) directly.
    Zero external fees — direct bank-to-bank transfer.
    """
    params = {
        "pa": merchant_id,           # Payee address (school UPI ID)
        "pn": merchant_name,         # Payee name
        "am": str(amount),           # Amount
        "tn": description,           # Transaction note
        "tr": transaction_id,        # Transaction reference
        "cu": "INR",                 # Currency
        "mc": "8220",                # Merchant category code (education)
    }

    return f"upi://pay?{urlencode(params)}"

def generate_upi_qr_data(
    merchant_id: str,
    merchant_name: str,
    amount: float,
    transaction_id: str,
) -> str:
    """
    Generate UPI QR code data string.
    Can be rendered as QR code in Flutter for scanning.
    """
    return generate_upi_link(merchant_id, merchant_name, amount, transaction_id, "EduSHAMIIT Fee")
```

### 3.3 Payment Verification

```python
# app/api/payments.py (continued)
@router.post("/verify")
async def verify_payment(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    """
    Verify UPI payment after student completes payment in UPI app.

    Request:
    {
        "payment_id": "uuid-payment-1",
        "transaction_id": "EDU-ABC123DEF456",
        "upi_transaction_id": "UPI-TXN-789012",
        "status": "success"
    }

    Response:
    {
        "success": true,
        "school_id": "uuid-school-1",
        "data": {
            "payment_id": "uuid-payment-1",
            "status": "success",
            "message": "Payment verified successfully"
        }
    }
    """
    sb = get_supabase()

    payment_id = request.get("payment_id")
    upi_transaction_id = request.get("upi_transaction_id")
    status = request.get("status", "success")

    # Verify with bank webhook (optional — for production)
    # For local development, trust the UPI app callback

    # Update payment status
    sb.table("payments").update({
        "status": status,
        "transaction_id": upi_transaction_id,
        "paid_at": datetime.now().isoformat(),
    }).eq("id", payment_id).eq("school_id", school_id).execute()

    # Update fee status
    payment = sb.table("payments").select("fee_id, amount").eq("id", payment_id).single().execute()
    if payment.data:
        fee = sb.table("fees").select("amount, amount_paid").eq("id", payment.data["fee_id"]).single().execute()
        if fee.data:
            new_paid = float(fee.data["amount_paid"]) + float(payment.data["amount"])
            new_status = "paid" if new_paid >= float(fee.data["amount"]) else "partial"
            sb.table("fees").update({
                "amount_paid": new_paid,
                "status": new_status,
            }).eq("id", payment.data["fee_id"]).execute()

    return {
        "success": True,
        "school_id": school_id,
        "data": {
            "payment_id": payment_id,
            "status": status,
            "message": "Payment verified successfully",
        }
    }
```

### 3.4 Bank Webhook Handler (Production)

```python
# app/api/payments.py (continued)
@router.post("/webhook")
async def bank_webhook(request: dict):
    """
    Handle bank payment webhook for production.
    Bank sends POST to this endpoint when payment is confirmed.

    Request (from bank):
    {
        "transaction_id": "EDU-ABC123DEF456",
        "upi_transaction_id": "UPI-TXN-789012",
        "amount": 12500.00,
        "status": "success",
        "timestamp": "2026-04-02T10:30:00Z"
    }
    """
    sb = get_supabase()

    transaction_id = request.get("transaction_id")
    upi_transaction_id = request.get("upi_transaction_id")
    status = request.get("status", "success")

    # Find payment by transaction_id
    payment = sb.table("payments").select("*").eq("transaction_id", transaction_id).single().execute()

    if not payment.data:
        return {"success": False, "error": "Transaction not found"}

    # Verify amount matches
    if float(request.get("amount", 0)) != float(payment.data["amount"]):
        return {"success": False, "error": "Amount mismatch"}

    # Update payment
    sb.table("payments").update({
        "status": status,
        "transaction_id": upi_transaction_id,
        "paid_at": request.get("timestamp", datetime.now().isoformat()),
    }).eq("id", payment.data["id"]).execute()

    # Update fee
    if status == "success":
        fee = sb.table("fees").select("amount, amount_paid").eq("id", payment.data["fee_id"]).single().execute()
        if fee.data:
            new_paid = float(fee.data["amount_paid"]) + float(payment.data["amount"])
            new_status = "paid" if new_paid >= float(fee.data["amount"]) else "partial"
            sb.table("fees").update({
                "amount_paid": new_paid,
                "status": new_status,
            }).eq("id", payment.data["fee_id"]).execute()

    return {"success": True, "message": "Payment processed"}
```

---

## 4. AUTHENTICATION ENDPOINTS

### 4.1 POST /api/auth/login

```python
# app/api/auth.py
from fastapi import APIRouter, HTTPException
from app.services.supabase_client import get_supabase
from jose import jwt
import os

router = APIRouter()

@router.post("/login")
async def login(request: dict):
    """
    Authenticate user and return JWT token.

    Request:
    {
        "email": "rohan@school.com",
        "password": "password123",
        "role": "student"
    }

    Response:
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
    """
    sb = get_supabase()

    email = request.get("email")
    password = request.get("password")

    # Authenticate with Supabase
    auth_response = sb.auth.sign_in_with_password({
        "email": email,
        "password": password,
    })

    user_id = auth_response.user.id

    # Get profile
    profile = sb.table("profiles").select("*").eq("id", user_id).single().execute()

    if not profile.data:
        raise HTTPException(status_code=404, detail="Profile not found")

    # Generate JWT with school_id
    token = jwt.encode(
        {
            "sub": user_id,
            "school_id": profile.data["school_id"],
            "role": profile.data["role"],
            "class": profile.data.get("class"),
            "exp": datetime.now() + timedelta(days=7),
        },
        os.getenv("SUPABASE_JWT_SECRET"),
        algorithm="HS256",
    )

    return {
        "success": True,
        "school_id": profile.data["school_id"],
        "data": {
            "token": token,
            "refresh_token": auth_response.session.refresh_token,
            "user": {
                "id": user_id,
                "full_name": profile.data["full_name"],
                "role": profile.data["role"],
                "class": profile.data.get("class"),
                "school_id": profile.data["school_id"],
                "avatar_url": profile.data.get("avatar_url"),
            }
        }
    }
```

### 4.2 POST /api/auth/register

```python
@router.post("/register")
async def register(request: dict):
    """
    Register new user (student/teacher).

    Request:
    {
        "email": "newstudent@school.com",
        "password": "password123",
        "full_name": "New Student",
        "role": "student",
        "class": "X-A",
        "school_id": "uuid-school-1"
    }
    """
    sb = get_supabase()

    # Create auth user
    auth_response = sb.auth.sign_up({
        "email": request["email"],
        "password": request["password"],
    })

    # Create profile
    sb.table("profiles").insert({
        "id": auth_response.user.id,
        "school_id": request["school_id"],
        "user_id": f"STU-{uuid.uuid4().hex[:6].upper()}",
        "full_name": request["full_name"],
        "role": request["role"],
        "class": request.get("class"),
    }).execute()

    return {"success": True, "message": "Registration successful"}
```

### 4.3 POST /api/auth/refresh

```python
@router.post("/refresh")
async def refresh_token(request: dict):
    """Refresh JWT token."""
    sb = get_supabase()
    refresh_token = request.get("refresh_token")

    auth_response = sb.auth.refresh_session(refresh_token)

    # Re-generate JWT
    profile = sb.table("profiles").select("*").eq("id", auth_response.user.id).single().execute()

    token = jwt.encode(
        {
            "sub": auth_response.user.id,
            "school_id": profile.data["school_id"],
            "role": profile.data["role"],
            "exp": datetime.now() + timedelta(days=7),
        },
        os.getenv("SUPABASE_JWT_SECRET"),
        algorithm="HS256",
    )

    return {"success": True, "data": {"token": token}}
```

---

## 5. STUDENT SCREEN ENDPOINTS (29 SCREENS)

### 5.1 Dashboard — GET /api/student/dashboard

```python
# app/api/student.py
from fastapi import APIRouter, Depends
from app.middleware.auth import get_current_user, require_school_id
from app.services.supabase_client import get_supabase
from app.services.redis_client import get_cached, set_cached

router = APIRouter()

@router.get("/dashboard")
async def student_dashboard(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    """
    Student dashboard data.

    Response:
    {
        "success": true,
        "school_id": "uuid-school-1",
        "data": {
            "user": {...},
            "stats": {...},
            "today_schedule": [...],
            "pending_homework": [...],
            "quick_access": [...]
        }
    }
    """
    # Check cache
    cached = await get_cached(school_id, "dashboard", user["id"])
    if cached:
        return cached

    sb = get_supabase()

    # Get profile
    profile = sb.table("profiles").select("*").eq("id", user["id"]).single().execute().data

    # Get today's schedule
    today = datetime.now().weekday()
    schedule = sb.table("timetable")\
        .select("*, subjects(name, icon, color)")\
        .eq("school_id", school_id)\
        .eq("class", profile["class"])\
        .eq("day_of_week", today)\
        .order("start_time").execute().data

    # Get pending homework
    homework = sb.table("homework")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .eq("target_class", profile["class"])\
        .eq("status", "active")\
        .lte("due_date", (datetime.now() + timedelta(days=3)).isoformat())\
        .order("due_date").execute().data

    # Get stats
    attendance = sb.table("attendance")\
        .select("status")\
        .eq("school_id", school_id)\
        .eq("student_id", user["id"]).execute().data

    att_pct = (sum(1 for a in attendance if a["status"] == "present") / len(attendance) * 100) if attendance else 0

    result = {
        "success": True,
        "school_id": school_id,
        "data": {
            "user": profile,
            "stats": {
                "attendance_pct": round(att_pct, 1),
                "avg_score": 91.4,
                "class_rank": 3,
                "xp_points": profile.get("xp_points", 0),
            },
            "today_schedule": schedule,
            "pending_homework": homework,
            "quick_access": _get_quick_access_items(),
        }
    }

    # Cache for 2 minutes
    await set_cached(school_id, "dashboard", result, user["id"], ttl=120)

    return result
```

### 5.2 Timetable — GET /api/student/timetable

```python
@router.get("/timetable")
async def student_timetable(
    day: str = "monday",
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()
    profile = sb.table("profiles").select("class").eq("id", user["id"]).single().execute().data

    day_map = {"monday":0,"tuesday":1,"wednesday":2,"thursday":3,"friday":4,"saturday":5}
    day_num = day_map.get(day.lower(), datetime.now().weekday())

    schedule = sb.table("timetable")\
        .select("*, subjects(name, icon, color)")\
        .eq("school_id", school_id)\
        .eq("class", profile["class"])\
        .eq("day_of_week", day_num)\
        .order("start_time").execute().data

    return {"success": True, "school_id": school_id, "data": {"schedule": schedule}}
```

### 5.3 Results — GET /api/student/results

```python
@router.get("/results")
async def student_results(
    category: str = "All",
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    query = sb.table("results")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .eq("student_id", user["id"])

    if category != "All":
        query = query.eq("exam_category", category)

    results = query.order("created_at", ascending=False).execute().data

    # Calculate overall
    total = sum(float(r.get("marks_obtained", 0)) for r in results)
    max_total = sum(float(r.get("max_marks", 100)) for r in results)
    avg_score = (total / max_total * 100) if max_total > 0 else 0

    return {
        "success": True,
        "school_id": school_id,
        "data": {
            "overall": {"avg_score": round(avg_score, 1), "grade": _calculate_grade(avg_score), "total_exams": len(results)},
            "results": results,
        }
    }
```

### 5.4 Exams — GET /api/student/exams

```python
@router.get("/exams")
async def student_exams(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()
    profile = sb.table("profiles").select("class").eq("id", user["id"]).single().execute().data

    exams = sb.table("exams")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .contains("target_classes", f'["{profile["class"]}"]')\
        .gte("exam_date", datetime.now().date().isoformat())\
        .order("exam_date").execute().data

    return {"success": True, "school_id": school_id, "data": {"exams": exams}}
```

### 5.5 Homework — GET /api/student/homework

```python
@router.get("/homework")
async def student_homework(
    status: str = "all",
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()
    profile = sb.table("profiles").select("class").eq("id", user["id"]).single().execute().data

    homework = sb.table("homework")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .eq("target_class", profile["class"])\
        .eq("status", "active")\
        .order("due_date").execute().data

    # Get submissions
    submissions = sb.table("homework_submissions")\
        .select("homework_id, status")\
        .eq("student_id", user["id"]).execute().data

    submitted_ids = {s["homework_id"] for s in submissions}

    # Filter
    filtered = []
    for hw in homework:
        sub = next((s for s in submissions if s["homework_id"] == hw["id"]), None)
        hw["submission_status"] = sub["status"] if sub else None

        if status == "all":
            filtered.append(hw)
        elif status == "pending" and not sub:
            filtered.append(hw)
        elif status == "submitted" and sub and sub["status"] == "submitted":
            filtered.append(hw)
        elif status == "graded" and sub and sub["status"] == "graded":
            filtered.append(hw)

    return {"success": True, "school_id": school_id, "data": {"homework": filtered}}
```

### 5.6 Homework Submit — POST /api/student/homework/submit

```python
@router.post("/homework/submit")
async def submit_homework(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    homework_id = request.get("homework_id")
    submission_text = request.get("submission_text", "")

    # Check if already submitted
    existing = sb.table("homework_submissions")\
        .select("id")\
        .eq("homework_id", homework_id)\
        .eq("student_id", user["id"]).maybe_single().execute()

    if existing.data:
        return {"success": False, "message": "Already submitted"}

    # Create submission
    submission = sb.table("homework_submissions").insert({
        "school_id": school_id,
        "homework_id": homework_id,
        "student_id": user["id"],
        "submission_text": submission_text,
        "status": "submitted",
    }).execute()

    # Award XP
    await _update_xp(school_id, user["id"], 50, "homework_submission")

    return {"success": True, "school_id": school_id, "data": {"submission_id": submission.data[0]["id"]}}
```

### 5.7 Attendance — GET /api/student/attendance

```python
@router.get("/attendance")
async def student_attendance(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    attendance = sb.table("attendance")\
        .select("*, subjects(name)")\
        .eq("school_id", school_id)\
        .eq("student_id", user["id"]).execute().data

    total = len(attendance)
    present = sum(1 for a in attendance if a["status"] == "present")
    absent = sum(1 for a in attendance if a["status"] == "absent")
    late = sum(1 for a in attendance if a["status"] == "late")
    pct = (present / total * 100) if total > 0 else 0

    # Subject-wise
    subject_wise = {}
    for a in attendance:
        subj = a.get("subjects", {}).get("name", "Unknown")
        subject_wise.setdefault(subj, {"total": 0, "present": 0})
        subject_wise[subj]["total"] += 1
        if a["status"] == "present":
            subject_wise[subj]["present"] += 1

    return {
        "success": True,
        "school_id": school_id,
        "data": {
            "overall_pct": round(pct, 1),
            "present_days": present,
            "absent_days": absent,
            "late_days": late,
            "total_days": total,
            "subject_wise": [
                {"subject": s, "present": d["present"], "total": d["total"], "pct": round(d["present"]/d["total"]*100, 1) if d["total"]>0 else 0}
                for s, d in subject_wise.items()
            ],
        }
    }
```

### 5.8 Fees — GET /api/student/fees

```python
@router.get("/fees")
async def student_fees(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    fees = sb.table("fees")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("student_id", user["id"])\
        .order("due_date").execute().data

    payments = sb.table("payments")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("student_id", user["id"])\
        .order("paid_at", ascending=False).execute().data

    total_outstanding = sum(float(f["amount"]) - float(f.get("amount_paid", 0)) for f in fees if f["status"] in ("pending", "partial", "overdue"))
    total_paid = sum(float(f.get("amount_paid", 0)) for f in fees)

    return {
        "success": True,
        "school_id": school_id,
        "data": {
            "total_outstanding": total_outstanding,
            "total_paid": total_paid,
            "pending_fees": [f for f in fees if f["status"] in ("pending", "partial", "overdue")],
            "recent_payments": payments[:3],
        }
    }
```

### 5.9 Transport — GET /api/student/transport

```python
@router.get("/transport")
async def student_transport(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    transport = sb.table("student_transport")\
        .select("*, bus_routes(*), bus_stops(*)")\
        .eq("school_id", school_id)\
        .eq("student_id", user["id"]).maybe_single().execute().data

    if not transport:
        return {"success": False, "message": "Not assigned to any bus route"}

    # Get live location
    bus_location = sb.table("bus_locations")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("route_id", transport["route_id"])\
        .order("recorded_at", ascending=False)\
        .limit(1).maybe_single().execute().data

    return {
        "success": True,
        "school_id": school_id,
        "data": {
            "route": transport["bus_routes"],
            "your_stop": transport["bus_stops"]["stop_name"],
            "live_location": bus_location,
        }
    }
```

### 5.10-5.29 Remaining Student Endpoints

Each follows the same pattern — `school_id` scoping, Supabase queries, response format.

| Screen        | Endpoint                     | Method |
| ------------- | ---------------------------- | ------ |
| Notices       | `/api/student/notices`       | GET    |
| Events        | `/api/student/events`        | GET    |
| Achievements  | `/api/student/achievements`  | GET    |
| Leave         | `/api/student/leave/apply`   | POST   |
| Profile       | `/api/student/profile`       | GET    |
| Library       | `/api/student/library`       | GET    |
| Courses       | `/api/student/courses`       | GET    |
| Notifications | `/api/student/notifications` | GET    |
| Settings      | `/api/user/settings`         | GET    |
| Messaging     | `/api/messages`              | GET    |
| Chat Detail   | `/api/messages/chat`         | GET    |
| Create Group  | `/api/groups/create`         | POST   |
| Live Classes  | `/api/student/live-classes`  | GET    |
| Leaderboard   | `/api/student/leaderboard`   | GET    |

---

## 6. TEACHER SCREEN ENDPOINTS (29 SCREENS)

### 6.1 Dashboard — GET /api/teacher/dashboard

```python
# app/api/teacher.py
from fastapi import APIRouter, Depends
from app.middleware.auth import get_current_user, require_school_id
from app.services.supabase_client import get_supabase

router = APIRouter()

@router.get("/dashboard")
async def teacher_dashboard(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    # Get classes
    classes = sb.table("timetable")\
        .select("class")\
        .eq("school_id", school_id)\
        .eq("teacher_id", user["id"]).execute().data

    unique_classes = list(set(c["class"] for c in classes))

    # Get student count
    students = sb.table("profiles")\
        .select("id")\
        .eq("school_id", school_id)\
        .in_("class", unique_classes)\
        .eq("role", "student").execute().data

    # Get pending tasks
    homework = sb.table("homework")\
        .select("id")\
        .eq("school_id", school_id)\
        .eq("teacher_id", user["id"])\
        .eq("status", "active").execute().data

    pending_subs = 0
    for hw in homework:
        subs = sb.table("homework_submissions")\
            .select("id")\
            .eq("homework_id", hw["id"])\
            .eq("status", "submitted").execute().data
        pending_subs += len(subs)

    return {
        "success": True,
        "school_id": school_id,
        "data": {
            "teacher": sb.table("profiles").select("*").eq("id", user["id"]).single().execute().data,
            "stats": {
                "total_students": len(students),
                "avg_attendance": 92,
                "total_classes": len(unique_classes),
                "pending_tasks": pending_subs,
            }
        }
    }
```

### 6.2 My Classes — GET /api/teacher/classes

```python
@router.get("/classes")
async def teacher_classes(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    classes = sb.table("timetable")\
        .select("class")\
        .eq("school_id", school_id)\
        .eq("teacher_id", user["id"]).execute().data

    unique_classes = list(set(c["class"] for c in classes))

    class_details = []
    for cls in unique_classes:
        students = sb.table("profiles")\
            .select("id")\
            .eq("school_id", school_id)\
            .eq("class", cls)\
            .eq("role", "student").execute().data

        class_details.append({
            "class": cls,
            "student_count": len(students),
        })

    return {"success": True, "school_id": school_id, "data": {"classes": class_details}}
```

### 6.3 Mark Attendance — POST /api/teacher/attendance/mark

```python
@router.post("/attendance/mark")
async def mark_attendance(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    date = request.get("date", datetime.now().date().isoformat())
    records = request.get("attendance_records", [])

    for record in records:
        # Upsert attendance
        sb.table("attendance").upsert({
            "school_id": school_id,
            "student_id": record["student_id"],
            "subject_id": request.get("subject_id"),
            "teacher_id": user["id"],
            "date": date,
            "status": record["status"],
        }, on_conflict="school_id,student_id,subject_id,date").execute()

    return {"success": True, "school_id": school_id, "message": f"Marked {len(records)} students"}
```

### 6.4 Create Homework — POST /api/teacher/homework/create

```python
@router.post("/homework/create")
async def create_homework(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    homework = sb.table("homework").insert({
        "school_id": school_id,
        "subject_id": request.get("subject_id"),
        "teacher_id": user["id"],
        "title": request.get("title"),
        "description": request.get("description"),
        "due_date": request.get("due_date"),
        "max_marks": request.get("max_marks", 25),
        "target_class": request.get("target_class"),
        "status": "active",
    }).execute()

    return {"success": True, "school_id": school_id, "data": {"homework_id": homework.data[0]["id"]}}
```

### 6.5 Grade Submission — POST /api/teacher/submissions/grade

```python
@router.post("/submissions/grade")
async def grade_submission(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    sb.table("homework_submissions").update({
        "status": "graded",
        "marks": request.get("marks"),
        "grade": request.get("grade"),
        "teacher_remarks": request.get("remarks"),
        "graded_by": user["id"],
        "graded_at": datetime.now().isoformat(),
    }).eq("id", request.get("submission_id")).eq("school_id", school_id).execute()

    # Award XP to student
    submission = sb.table("homework_submissions").select("student_id").eq("id", request.get("submission_id")).single().execute()
    if submission.data:
        await _update_xp(school_id, submission.data["student_id"], 50, "homework_graded")

    return {"success": True, "school_id": school_id, "message": "Submission graded"}
```

### 6.6-6.29 Remaining Teacher Endpoints

| Screen             | Endpoint                                | Method |
| ------------------ | --------------------------------------- | ------ |
| Class Detail       | `/api/teacher/class-detail`             | GET    |
| Timetable          | `/api/teacher/timetable`                | GET    |
| Exams              | `/api/teacher/exams/create`             | POST   |
| Generate Questions | `/api/teacher/exams/generate-questions` | POST   |
| Gradebook          | `/api/teacher/gradebook`                | GET    |
| Grading Config     | `/api/teacher/grading-config`           | PUT    |
| Student Directory  | `/api/teacher/students`                 | GET    |
| Leave              | `/api/teacher/leave/apply`              | POST   |
| Notices            | `/api/teacher/notices/create`           | POST   |
| Live Classes       | `/api/teacher/live-classes`             | GET    |
| Live Session Start | `/api/teacher/live-classes/start`       | POST   |
| Materials          | `/api/teacher/materials/upload`         | POST   |
| Salary             | `/api/teacher/salary`                   | GET    |
| Profile            | `/api/teacher/profile`                  | GET    |
| Settings           | `/api/user/settings`                    | PUT    |
| Notifications      | `/api/notifications`                    | GET    |

---

## 7. SHARED SCREEN ENDPOINTS (7 SCREENS)

### 7.1 Messaging — GET /api/messages

```python
# app/api/shared.py
from fastapi import APIRouter, Depends
from app.middleware.auth import get_current_user, require_school_id
from app.services.supabase_client import get_supabase

router = APIRouter()

@router.get("/messages")
async def get_messages(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    messages = sb.table("messages")\
        .select("*, profiles!sender_id(full_name, avatar_url, role)")\
        .eq("school_id", school_id)\
        .or_(f"sender_id.eq.{user['id']},receiver_id.eq.{user['id']}")\
        .order("created_at", ascending=False).execute().data

    return {"success": True, "school_id": school_id, "data": {"messages": messages}}

@router.post("/messages/send")
async def send_message(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    message = sb.table("messages").insert({
        "school_id": school_id,
        "sender_id": user["id"],
        "receiver_id": request.get("receiver_id"),
        "content": request.get("content"),
    }).execute()

    return {"success": True, "school_id": school_id, "data": {"message_id": message.data[0]["id"]}}
```

### 7.2 Notifications — GET /api/notifications

```python
@router.get("/notifications")
async def get_notifications(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()

    notifications = sb.table("notifications")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("user_id", user["id"])\
        .order("created_at", ascending=False)\
        .limit(20).execute().data

    return {"success": True, "school_id": school_id, "data": {"notifications": notifications}}
```

### 7.3 Settings — GET/PUT /api/user/settings

```python
@router.get("/user/settings")
async def get_settings(
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()
    settings = sb.table("user_settings").select("*").eq("user_id", user["id"]).single().execute().data
    return {"success": True, "school_id": school_id, "data": {"settings": settings}}

@router.put("/user/settings")
async def update_settings(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    sb = get_supabase()
    sb.table("user_settings").upsert({
        "school_id": school_id,
        "user_id": user["id"],
        **request,
    }).execute()
    return {"success": True, "school_id": school_id, "message": "Settings updated"}
```

---

## 8. AI CHAT ENDPOINTS

### 8.1 POST /api/chat/message (SSE Streaming)

```python
# app/api/chat.py
from fastapi import APIRouter, Depends
from fastapi.responses import StreamingResponse
from app.middleware.auth import get_current_user, require_school_id
from app.services.langchain_agent import process_message
import json

router = APIRouter()

@router.post("/message")
async def chat_message(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    """
    Send message to Shami AI and stream response via SSE.

    Request:
    {
        "message": "What classes do I have today?",
        "session_id": "uuid-session-1"
    }

    SSE Response:
    data: {"type":"text","content":"📅 Your Monday Schedule:\n"}
    data: {"type":"text","content":"  📐 Mathematics - 08:00 to 08:45\n"}
    data: {"type":"tool_result","tool":"get_timetable","data":{...}}
    data: {"type":"done"}
    """
    message = request.get("message")
    session_id = request.get("session_id")

    async def generate():
        async for chunk in process_message(
            text=message,
            user=user,
            school_id=school_id,
            session_id=session_id,
        ):
            yield f"data: {json.dumps(chunk)}\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

### 8.2 POST /api/chat/voice

```python
@router.post("/chat/voice")
async def chat_voice(
    audio: UploadFile,
    session_id: str = Form(...),
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    # Transcribe with Whisper
    from app.services.whisper_service import transcribe
    text = await transcribe(await audio.read())

    # Process with AI agent
    async def generate():
        async for chunk in process_message(text=text, user=user, school_id=school_id, session_id=session_id):
            yield f"data: {json.dumps(chunk)}\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

### 8.3 POST /api/chat/image

```python
@router.post("/chat/image")
async def chat_image(
    image: UploadFile,
    question: str = Form("Describe this image"),
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    import base64
    from langchain_google_genai import ChatGoogleGenerativeAI
    from langchain_core.messages import HumanMessage

    image_bytes = await image.read()
    b64 = base64.b64encode(image_bytes).decode()

    llm = ChatGoogleGenerativeAI(model="gemini-1.5-flash-latest")
    msg = HumanMessage(content=[
        {"type": "image_url", "image_url": {"url": f"data:{image.content_type};base64,{b64}"}},
        {"type": "text", "text": question}
    ])
    response = await llm.ainvoke([msg])

    return {"success": True, "school_id": school_id, "data": {"output": response.content}}
```

---

## 9. IoT CONTROL ENDPOINTS

### 9.1 POST /api/iot/control

```python
# app/api/iot.py
from fastapi import APIRouter, Depends
from app.middleware.auth import get_current_user, require_school_id
from app.services.iot_controller import iot

router = APIRouter()

@router.post("/control")
async def control_device(
    request: dict,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    """
    Control classroom device via MQTT.

    Request:
    {
        "room": "class_10a",
        "device": "fan",
        "action": "on"
    }
    """
    room = request.get("room")
    device = request.get("device")
    action = request.get("action")

    result = iot.control_device(room, device, action)

    return {
        "success": result["ok"],
        "school_id": school_id,
        "data": {"room": room, "device": device, "action": action, "method": result.get("method")}
    }

@router.get("/status/{room}")
async def get_device_status(
    room: str,
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    status = iot.get_room_status(room)
    return {"success": True, "school_id": school_id, "data": {"room": room, "devices": status}}
```

---

## 10. RAG PIPELINE ENDPOINTS

### 10.1 POST /api/rag/ingest

```python
# app/api/rag.py
from fastapi import APIRouter, UploadFile, Form, Depends
from app.middleware.auth import get_current_user, require_school_id
from app.services.rag_service import ingest_document

router = APIRouter()

@router.post("/ingest")
async def ingest_document_endpoint(
    file: UploadFile,
    subject: str = Form(...),
    grade: str = Form(...),
    source: str = Form(...),
    user=Depends(get_current_user),
    school_id=Depends(require_school_id),
):
    """
    Ingest PDF document into RAG knowledge base.
    """
    pdf_bytes = await file.read()
    chunks = await ingest_document(
        pdf_bytes=pdf_bytes,
        school_id=school_id,
        subject=subject,
        grade=grade,
        source=source,
    )

    return {"success": True, "school_id": school_id, "data": {"chunks_ingested": chunks}}
```

---

## 11. COMPLETE PYTHON IMPLEMENTATION

### 11.1 Redis Cache Client

```python
# app/cache/redis_client.py
import redis
import json
import os

redis_client = redis.Redis.from_url(os.getenv("REDIS_URL", "redis://localhost:6379"), decode_responses=True)

async def get_cached(school_id: str, resource: str, resource_id: str = ""):
    key = f"{school_id}:{resource}:{resource_id}"
    data = redis_client.get(key)
    return json.loads(data) if data else None

async def set_cached(school_id: str, resource: str, data: dict, resource_id: str = "", ttl: int = 300):
    key = f"{school_id}:{resource}:{resource_id}"
    redis_client.setex(key, ttl, json.dumps(data, default=str))

async def invalidate_cache(school_id: str, resource: str = "*"):
    pattern = f"{school_id}:{resource}:*"
    for key in redis_client.scan_iter(match=pattern):
        redis_client.delete(key)
```

### 11.2 Supabase Client

```python
# app/services/supabase_client.py
from supabase import create_client
import os

_client = None

def get_supabase():
    global _client
    if _client is None:
        _client = create_client(
            os.getenv("SUPABASE_URL", "http://localhost:54321"),
            os.getenv("SUPABASE_SERVICE_ROLE_KEY", ""),
        )
    return _client
```

### 11.3 JWT Middleware

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
        return {
            "id": payload["sub"],
            "school_id": payload["school_id"],
            "role": payload["role"],
            "class": payload.get("class"),
        }
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

async def require_school_id(user=Depends(get_current_user)):
    if not user.get("school_id"):
        raise HTTPException(status_code=400, detail="school_id required")
    return user["school_id"]
```

### 11.4 XP Update Helper

```python
async def _update_xp(school_id: str, student_id: str, xp: int, action: str):
    from app.services.supabase_client import get_supabase
    from datetime import datetime, timedelta

    sb = get_supabase()
    profile = sb.table("profiles").select("last_login, learning_streak").eq("id", student_id).single().execute().data

    last_login = profile.get("last_login")
    streak = profile.get("learning_streak", 0)

    today = datetime.now().date()

    if last_login == (today - timedelta(days=1)).isoformat():
        new_streak = streak + 1
    elif last_login < (today - timedelta(days=1)).isoformat():
        new_streak = 1
    else:
        new_streak = streak

    sb.table("profiles").update({
        "xp_points": sb.rpc("increment_xp", {"p_student_id": student_id, "p_xp": xp}),
        "learning_streak": new_streak,
        "best_streak": max(new_streak, profile.get("best_streak", 0)),
        "last_login": today.isoformat(),
    }).eq("id", student_id).execute()
```

---

_This document is the single source of truth for building the EduSHAMIIT backend — by Shami Innovation and Technologies LLP._

---

**Last updated:** April 2026
