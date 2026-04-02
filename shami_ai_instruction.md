# Shami AI — Complete Agent Instruction Manual
> EduSHAMIIT · School/College SaaS Platform — Shami Innovation and Technologies LLP  
> Version 1.0 · Python + LangChain + Supabase + ESP32 IoT  
> Author: Naresh — Shami Innovation and Technologies LLP

---

## Table of Contents

1. [Project Identity](#1-project-identity)
2. [Complete Tech Stack](#2-complete-tech-stack)
3. [Project Structure](#3-project-structure)
4. [Roles & Permissions](#4-roles--permissions)
5. [LLM Selection & Routing Strategy](#5-llm-selection--routing-strategy)
6. [Multimodal Input/Output](#6-multimodal-inputoutput)
7. [LangChain Agent Architecture](#7-langchain-agent-architecture)
8. [All Tool Definitions](#8-all-tool-definitions)
9. [RAG Pipeline](#9-rag-pipeline)
10. [IoT Device Control (ESP32 + MQTT)](#10-iot-device-control-esp32--mqtt)
11. [Camera Attendance (Computer Vision)](#11-camera-attendance-computer-vision)
12. [Supabase Schema](#12-supabase-schema)
13. [FastAPI Endpoints](#13-fastapi-endpoints)
14. [Flutter Integration](#14-flutter-integration)
15. [Security & Auth](#15-security--auth)
16. [Environment Variables](#16-environment-variables)
17. [Deployment](#17-deployment)
18. [Cost Optimisation](#18-cost-optimisation)
19. [Build Order](#19-build-order)

---

## 1. Project Identity

**Product name:** EduSHAMIIT  
**AI assistant name:** Shami  
**Market:** Indian school/college SaaS  
**Target users:** Students, Teachers, Parents, Principals, Admins, Support Staff — 17 roles across 7 tiers  
**India-specific defaults:**
- Currency: INR with Indian number formatting (₹1,23,456)
- Date format: DD/MM/YYYY
- Phone prefix: +91
- Languages: English + Hindi (Hinglish mix accepted in voice)
- Payments: Razorpay, UPI, BBPS
- Communication: WhatsApp Business API (Gupshup or Meta Cloud API)

**Shami AI — The EduSHAMIIT Intelligence (inspired by J.A.R.V.I.S. from Iron Man):**
- Proactive, intelligent, always on — like Jarvis for Tony Stark, Shami is the school's always-available intelligent assistant
- Warm, concise, authoritative
- Responds in the user's language (Hindi or English, auto-detected)
- Never hallucinates student data — always fetches from tools
- Addresses teachers formally, students encouragingly
- Proactively surfaces insights the user didn't ask for ("Sir, 3 teachers are absent today — shall I arrange substitutes?")
- Acknowledges when it cannot do something and explains why

---

## 2. Complete Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | Flutter + Riverpod | Mobile app (3 apps: EduSHAMIIT Academic, EduSHAMIIT Family, EduSHAMIIT Admin Suite) |
| State management | Riverpod | Auth state, chat state, role switching |
| Backend framework | FastAPI (Python 3.11+) | REST API + SSE streaming |
| AI orchestration | LangChain 0.3 | Agent, tools, RAG, memory |
| Primary LLM | Gemini 1.5 Flash | Text Q&A, RAG, vision, question generation |
| Tool call LLM | Claude Haiku 3.5 | Structured JSON tool calls, IoT commands |
| Voice STT | OpenAI Whisper-1 | Hindi + English transcription |
| Voice TTS | Google TTS / ElevenLabs | AI voice responses |
| Embeddings | text-embedding-3-small | RAG vector generation |
| Database | Supabase (PostgreSQL) | All structured data |
| Vector store | Supabase pgvector | RAG curriculum embeddings |
| Auth | Supabase Auth + JWT | User authentication |
| File storage | Supabase Storage | Documents, voice files, face images |
| Real-time | Supabase Realtime | Live dashboard updates |
| IoT messaging | MQTT (Mosquitto) | ESP32 relay board control |
| IoT hardware | ESP32 + 4CH relay board | Classroom device control (fans, lights) |
| Camera AI | YOLOv8 + DeepFace | Student count + attendance |
| Payments | Razorpay SDK | Fee collection + payment links |
| WhatsApp | Gupshup / Meta Cloud API | Parent notifications |
| Deployment | Railway.app | Python backend hosting |
| Cache | Redis | Response caching, device state |

---

## 3. Project Structure

```
edushamiit-ai-backend/
├── app/
│   ├── main.py                    # FastAPI app entry point
│   ├── models.py                  # Pydantic request/response schemas
│   ├── api/
│   │   ├── chat.py                # POST /chat/message  (text + streaming)
│   │   ├── voice.py               # POST /chat/voice    (audio upload)
│   │   ├── image.py               # POST /chat/image    (image upload)
│   │   └── iot.py                 # POST /iot/control, GET /iot/status/{room}
│   ├── agents/
│   │   ├── shamiit_agent.py      # Main LangChain agent builder
│   │   ├── router.py              # Task detection → LLM routing
│   │   └── prompts.py             # Role-specific system prompts
│   ├── tools/
│   │   ├── student_tools.py       # Student profile, attendance, results
│   │   ├── fee_tools.py           # Fee status, Razorpay links
│   │   ├── academic_tools.py      # Lesson plans, question generator
│   │   ├── whatsapp_tools.py      # WhatsApp parent messaging
│   │   ├── rag_tools.py           # Curriculum search (pgvector)
│   │   └── iot_tools.py           # Device control LangChain tools
│   ├── services/
│   │   ├── iot_controller.py      # MQTT + HTTP IoT service (singleton)
│   │   ├── camera_attendance.py   # YOLOv8 + DeepFace classroom camera
│   │   ├── rag_ingestion.py       # PDF → chunk → embed → store
│   │   ├── whisper_service.py     # Audio → text transcription
│   │   └── tts_service.py         # Text → audio response
│   ├── supabase/
│   │   ├── client.py              # Supabase singleton (service role)
│   │   └── auth.py                # JWT validation middleware
│   └── cache/
│       └── redis_client.py        # Response + device state cache
├── esp32/
│   └── relay_controller/
│       └── relay_controller.ino   # ESP32 Arduino firmware
├── scripts/
│   └── ingest_documents.py        # CLI: load PDFs into knowledge base
├── requirements.txt
├── Dockerfile
└── .env
```

---

## 4. Roles & Permissions

EduSHAMIIT has 17 roles across 7 tiers. Shami's tool access is gated by role.

| Tier | Role | App | Shami Capabilities |
|---|---|---|---|
| 0 — Platform | Super Admin | Admin Suite | Full access, all schools |
| 1 — Institution | Director | Admin Suite | Multi-school analytics, executive briefing |
| 2 — School Head | Principal | Admin Suite | School-wide reports, staff management |
| 3 — Academic | Teacher / HOD | Academic | Lesson plans, question paper, student insights |
| 3 — Academic | Student | Academic | Q&A, homework help, syllabus search (RAG) |
| 4 — Operations | Admin Manager | Admin Suite | Admissions, timetable, fee ops |
| 4 — Operations | Finance Officer | Admin Suite | Fee collection, Razorpay, defaulter list |
| 4 — Operations | HR Manager | Admin Suite | Staff attendance, payroll queries |
| 4 — Operations | Transport Coordinator | Admin Suite | Bus routes, driver management |
| 4 — Operations | Librarian | Admin Suite | Book catalog, issue/return |
| 4 — Operations | Sports Coordinator | Admin Suite | Events, student club management |
| 5 — Family | Parent | Family App | Child attendance, fees, results, WhatsApp alerts |
| 5 — Family | Admission Applicant | Family App | Application status, document queries |
| 5 — Family | Guest / Enquiry | Family App | School info chatbot (no auth required) |
| 6 — Support | Security Guard | Admin Suite | Gate QR, visitor log (no AI) |
| 6 — Support | Support Staff | Admin Suite | Task checklist (no AI) |
| 6 — Support | Bus Driver | Admin Suite | Route view, boarding log (no AI) |

**Tool permission matrix:**

```python
ROLE_TOOLS = {
    "student":    ["search_curriculum", "get_homework", "get_results", "get_timetable"],
    "teacher":    ["search_curriculum", "get_student_profile", "get_attendance_report",
                   "get_results", "generate_questions", "create_lesson_plan",
                   "get_fee_status", "send_whatsapp",
                   "control_classroom_device", "get_classroom_device_status"],
    "parent":     ["get_child_attendance", "get_child_results", "get_fee_status",
                   "create_razorpay_link"],
    "principal":  ["*"],   # all tools
    "admin":      ["get_student_profile", "get_fee_status", "create_razorpay_link",
                   "send_whatsapp", "control_classroom_device", "get_attendance_report"],
    "finance":    ["get_fee_status", "create_razorpay_link", "send_whatsapp"],
    "guest":      ["get_school_info", "search_curriculum"],  # very limited
}
```

---

## 5. LLM Selection & Routing Strategy

**Core principle:** Never use one model for everything. Route each task to the cheapest capable model.

### Models

| Model | Provider | Cost (input/output per 1M) | Use for |
|---|---|---|---|
| `gemini-1.5-flash-latest` | Google | $0.075 / $0.30 | Text Q&A, RAG, vision, question generation |
| `claude-haiku-4-5-20251001` | Anthropic | $0.80 / $4.00 | Tool calls, IoT JSON commands, DB operations |
| `whisper-1` | OpenAI | $0.006/min | Voice STT (Hindi + English) |
| Google TTS / ElevenLabs | Google / ElevenLabs | $0.000015/char | Voice TTS output |
| `text-embedding-3-small` | OpenAI | $0.02/1M | RAG embeddings (one-time per document) |
| Llama 3.2 Vision (local) | Meta (Ollama) | $0.00 | Camera attendance — on-premise, biometric privacy |

### Routing Logic

```python
# app/agents/router.py

from enum import Enum

class TaskType(Enum):
    QA          = "qa"           # student Q&A, explanations
    GENERATION  = "generation"   # question papers, lesson plans
    TOOL_CALL   = "tool_call"    # DB ops, fee, attendance fetch
    IOT_CONTROL = "iot_control"  # device commands (fan, light, camera)
    VISION      = "vision"       # image analysis (diagrams, exam sheets)
    RAG         = "rag"          # curriculum document retrieval + answer

IOT_KEYWORDS  = ["fan", "light", "lights", "camera", "turn on", "turn off", 
                  "switch", "projector", "ac", "bulb", "switch off", "band karo"]
TOOL_KEYWORDS = ["fee", "fees", "attendance", "marks", "result", "schedule",
                  "timetable", "leave", "salary", "payment", "dues", "absent"]

def detect_task(message: str, has_image: bool = False) -> TaskType:
    if has_image:
        return TaskType.VISION
    msg = message.lower()
    if any(k in msg for k in IOT_KEYWORDS):
        return TaskType.IOT_CONTROL
    if any(k in msg for k in TOOL_KEYWORDS):
        return TaskType.TOOL_CALL
    if any(k in msg for k in ["explain", "solve", "what is", "how does", "help me"]):
        return TaskType.RAG   # try RAG first, fallback to QA
    return TaskType.QA

def get_llm(task: TaskType):
    """Return the appropriate LLM for this task type."""
    if task in (TaskType.TOOL_CALL, TaskType.IOT_CONTROL):
        return ChatAnthropic(model="claude-haiku-4-5-20251001", temperature=0.0)
    return ChatGoogleGenerativeAI(model="gemini-1.5-flash-latest", temperature=0.3)
```

---

## 6. Multimodal Input/Output

### Text Input → Text Output (primary)
- User sends text message via Flutter
- FastAPI receives → LangChain agent processes → streams SSE response
- Supports Markdown in responses (rendered by `flutter_markdown`)

### Voice Input → Text/Voice Output
```
Flutter records audio (.m4a / .wav)
  → POST /chat/voice (multipart)
  → Whisper-1 transcribes (Hindi + English)
  → LangChain agent processes transcribed text
  → Response text returned
  → Optional: TTS converts response to audio
  → Flutter plays audio bubble
```

**Implementation:**
```python
# services/whisper_service.py
from openai import AsyncOpenAI

oai = AsyncOpenAI()

async def transcribe(audio_bytes: bytes, filename: str, content_type: str) -> str:
    transcript = await oai.audio.transcriptions.create(
        model="whisper-1",
        file=(filename, audio_bytes, content_type),
        language="hi",       # Hindi detection; also handles English auto
        response_format="text"
    )
    return transcript
```

### Image Input → Text Output
- User uploads image (handwritten answer, diagram, whiteboard photo, ID card)
- Gemini Flash Vision analyzes the image
- Returns text analysis, explanation, or structured data
- Relay images NOT sent to Gemini — face data stays on-premise

```python
# api/image.py
import base64
from anthropic import AsyncAnthropic

async def analyze_image(image_bytes: bytes, content_type: str, question: str) -> str:
    b64 = base64.b64encode(image_bytes).decode()
    # Use Gemini Flash for image analysis
    from langchain_google_genai import ChatGoogleGenerativeAI
    from langchain_core.messages import HumanMessage
    llm = ChatGoogleGenerativeAI(model="gemini-1.5-flash-latest")
    msg = HumanMessage(content=[
        {"type": "image_url", "image_url": {"url": f"data:{content_type};base64,{b64}"}},
        {"type": "text", "text": question or "Describe and analyze this image."}
    ])
    response = await llm.ainvoke([msg])
    return response.content
```

### Tool Result → Structured Card (Flutter)
- When agent calls a tool (fee status, attendance, device control)
- Response JSON includes `tool_result` key with structured data
- Flutter renders role-specific widget: `FeeStatusCard`, `AttendanceCard`, `PaymentLinkCard`

**Response envelope:**
```json
{
  "type": "text",
  "content": "Rohan's fee of ₹12,500 is pending since 15/01/2025.",
  "tool_result": {
    "type": "fee_status",
    "data": {
      "student_name": "Rohan Sharma",
      "amount_due": 12500,
      "due_date": "15/01/2025",
      "payment_url": "https://rzp.io/i/abc123"
    }
  }
}
```

---

## 7. LangChain Agent Architecture

### Main Agent Builder

```python
# app/agents/shamiit_agent.py
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_anthropic import ChatAnthropic
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.messages import HumanMessage, AIMessage
from app.agents.router import detect_task, get_llm
from app.tools import ALL_TOOLS
from app.agents.prompts import get_system_prompt

def build_agent(role: str, school_id: str, task_type: str = "qa") -> AgentExecutor:
    """
    Build a role-scoped, task-appropriate LangChain agent.
    
    Args:
        role:      User role string (e.g. 'teacher', 'student')
        school_id: UUID of the school — all queries scoped to this
        task_type: Detected task type for LLM selection
    """
    llm   = get_llm(task_type)
    tools = filter_tools_by_role(ALL_TOOLS, role, school_id)

    prompt = ChatPromptTemplate.from_messages([
        ("system", get_system_prompt(role, school_id)),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
        MessagesPlaceholder("agent_scratchpad"),
    ])

    agent = create_tool_calling_agent(llm, tools, prompt)
    return AgentExecutor(
        agent=agent,
        tools=tools,
        return_intermediate_steps=True,
        max_iterations=5,
        verbose=True,
    )

async def process_message(
    text: str = "",
    image_b64: str = None,
    audio_path: str = None,
    user=None,
    session_id: str = "",
) -> dict:
    """Single entry point for all message types."""

    # Step 1: Transcribe voice if provided
    if audio_path:
        from app.services.whisper_service import transcribe_file
        text = await transcribe_file(audio_path)

    # Step 2: Handle image directly (skip agent for pure vision tasks)
    if image_b64:
        from app.services.vision_service import analyze_image
        result = await analyze_image(image_b64, text)
        return {"type": "vision", "output": result, "tool_result": None}

    # Step 3: Detect task type and build agent
    task     = detect_task(text)
    agent    = build_agent(user.role, user.school_id, task.value)
    history  = load_history(session_id)

    # Step 4: Run agent
    result = await agent.ainvoke({
        "input": text,
        "chat_history": history,
    })

    # Step 5: Save conversation
    await save_message(session_id, user.id, "user", text)
    await save_message(session_id, user.id, "assistant", result["output"],
                       tool_data=extract_tool_results(result))

    return {
        "type": "text",
        "output": result["output"],
        "tool_result": extract_tool_results(result),
        "intermediate_steps": result.get("intermediate_steps", []),
    }
```

### System Prompts

```python
# app/agents/prompts.py

BASE_PROMPT = """You are Shami, the AI assistant for EduSHAMIIT — a school & college management platform by Shami Innovation and Technologies LLP.

IDENTITY:
- Name: Shami
- Tone: Warm, professional, concise
- Language: Match the user's language (Hindi or English). Hinglish is fine.
- Never invent student data. Always use tools to fetch real information.
- Format numbers in Indian style: ₹1,23,456 not ₹123456
- Dates in DD/MM/YYYY format

SCHOOL CONTEXT:
- School ID: {school_id}
- All data queries are automatically scoped to this school

TOOL USAGE RULES:
- Always call tools to fetch live data before answering factual questions
- For device control (fans/lights): confirm the action with the user if ambiguous
- For fee payment: always generate a Razorpay link, never ask for card details
- For WhatsApp: confirm message content with the user before sending

When you cannot help, say so clearly and suggest who can help instead.
"""

ROLE_CONTEXT = {
    "student": """You are talking to a student. Be encouraging and supportive.
Help them understand concepts, solve problems, and track their academic progress.
Suggest study strategies. Never give direct answers to exam questions — guide instead.""",

    "teacher": """You are talking to a teacher. Be efficient and professional.
Help generate lesson plans, question papers, analyze student performance, and automate admin tasks.
You can control classroom devices (fans, lights, projectors) on their behalf.""",

    "parent": """You are talking to a parent. Be reassuring and transparent.
Help them track their child's attendance, results, and fees.
You can generate payment links and send WhatsApp notifications.""",

    "principal": """You are talking to the school principal. Provide strategic insights.
Give school-wide analytics, staff summaries, and actionable recommendations.
You have access to all school data.""",

    "admin": """You are talking to an admin staff member. Be precise and task-oriented.
Help with admissions, fee collection, timetables, and parent communication.""",

    "finance": """You are talking to a finance officer.
Help with fee collection, pending dues, Razorpay links, and payment reports.""",

    "guest": """You are talking to a prospective parent exploring the school.
Provide school information, admission process details, and answer FAQs.
Do not share private student data. Encourage them to apply.""",
}

def get_system_prompt(role: str, school_id: str) -> str:
    context = ROLE_CONTEXT.get(role, "You are a helpful school assistant.")
    return BASE_PROMPT.format(school_id=school_id) + "\n\n" + context
```

---

## 8. All Tool Definitions

### Student Tools

```python
# app/tools/student_tools.py
from langchain_core.tools import tool
from app.supabase.client import get_supabase
import json

def get_student_tools(school_id: str) -> list:

    @tool
    def get_student_profile(student_name_or_id: str) -> str:
        """Look up a student's profile — name, class, section, roll number, parent contact."""
        sb = get_supabase()
        result = sb.table("students")\
            .select("id,name,class,section,roll_no,parent_phone,parent_email")\
            .eq("school_id", school_id)\
            .ilike("name", f"%{student_name_or_id}%")\
            .execute()
        return json.dumps(result.data)

    @tool
    def get_attendance_report(class_name: str, date: str = "today") -> str:
        """Get attendance for a class on a given date. date format: DD/MM/YYYY or 'today'."""
        from datetime import datetime
        query_date = datetime.now().strftime("%Y-%m-%d") if date == "today" \
                     else datetime.strptime(date, "%d/%m/%Y").strftime("%Y-%m-%d")
        result = get_supabase().table("attendance")\
            .select("*, students(name, roll_no)")\
            .eq("school_id", school_id)\
            .eq("class", class_name)\
            .eq("date", query_date)\
            .execute()
        return json.dumps(result.data)

    @tool
    def mark_attendance(student_id: str, date: str, status: str) -> str:
        """Mark a student's attendance. status: 'present' | 'absent' | 'late'."""
        get_supabase().table("attendance").upsert({
            "student_id": student_id, "date": date,
            "status": status, "school_id": school_id
        }).execute()
        return f"Attendance marked: {status} for student {student_id} on {date}"

    @tool
    def get_results(student_id: str, exam: str = "") -> str:
        """Get exam results for a student. Optionally filter by exam name."""
        query = get_supabase().table("results")\
            .select("subject, marks_obtained, max_marks, grade, exam_name")\
            .eq("student_id", student_id)
        if exam:
            query = query.ilike("exam_name", f"%{exam}%")
        return json.dumps(query.execute().data)

    return [get_student_profile, get_attendance_report, mark_attendance, get_results]
```

### Fee Tools

```python
# app/tools/fee_tools.py
from langchain_core.tools import tool
import razorpay, json, os

@tool
def get_fee_status(student_id: str) -> str:
    """Get pending fee dues and payment history for a student."""
    result = get_supabase().table("fees")\
        .select("amount_due, due_date, fee_type, status, academic_year")\
        .eq("student_id", student_id)\
        .eq("status", "pending")\
        .execute()
    return json.dumps(result.data)

@tool
def create_razorpay_payment_link(student_id: str, amount: float, description: str = "") -> str:
    """
    Create a Razorpay payment link for fee collection.
    Returns a short URL the parent can use to pay immediately.
    amount: in INR (e.g. 12500 for ₹12,500)
    """
    client = razorpay.Client(auth=(os.getenv("RAZORPAY_KEY_ID"), os.getenv("RAZORPAY_SECRET")))
    student = get_supabase().table("students")\
        .select("name, parent_phone, parent_email")\
        .eq("id", student_id).single().execute()
    s = student.data
    link = client.payment_link.create({
        "amount": int(amount * 100),   # paise
        "currency": "INR",
        "description": description or f"School fee — {s['name']}",
        "customer": {
            "name": s["name"],
            "contact": s["parent_phone"],
            "email": s["parent_email"],
        },
        "notify": {"sms": True, "email": True},
        "reminder_enable": True,
    })
    return json.dumps({"payment_url": link["short_url"], "amount_inr": amount,
                        "student": s["name"]})
```

### Academic Tools

```python
# app/tools/academic_tools.py
from langchain_core.tools import tool
from langchain_google_genai import ChatGoogleGenerativeAI

@tool
def generate_questions(subject: str, topic: str, grade: str,
                        question_type: str = "mixed", count: int = 10,
                        difficulty: str = "medium") -> str:
    """
    Generate exam/practice questions.
    subject: 'Mathematics' | 'Physics' | 'Hindi' | etc.
    topic: specific topic within subject
    grade: 'Class 10' | 'Class 12' etc.
    question_type: 'mcq' | 'short_answer' | 'long_answer' | 'mixed'
    difficulty: 'easy' | 'medium' | 'hard'
    """
    llm = ChatGoogleGenerativeAI(model="gemini-1.5-flash-latest", temperature=0.7)
    prompt = f"""Generate {count} {difficulty} {question_type} questions for {grade} {subject} on topic: {topic}.
    
    Format each question with:
    - Question number
    - Question text
    - For MCQ: 4 options (A, B, C, D) and mark the correct answer
    - For subjective: mention marks and expected answer length
    - NCERT aligned where applicable
    
    Output as clean JSON array."""
    
    response = llm.invoke(prompt)
    return response.content

@tool
def create_lesson_plan(subject: str, topic: str, grade: str,
                        duration_minutes: int = 45) -> str:
    """
    Generate a structured lesson plan for a teacher.
    Returns a detailed plan with objectives, activities, and assessment.
    """
    llm = ChatGoogleGenerativeAI(model="gemini-1.5-flash-latest", temperature=0.4)
    prompt = f"""Create a {duration_minutes}-minute lesson plan for:
    Subject: {subject}
    Topic: {topic}
    Grade: {grade}
    
    Include: Learning objectives, teaching materials, step-by-step activities,
    Q&A section, assessment questions, homework assignment.
    NCERT curriculum aligned. Practical and easy to follow."""
    
    response = llm.invoke(prompt)
    return response.content

@tool
def get_timetable(class_name: str, day: str = "today") -> str:
    """Get the class timetable for a specific day."""
    from datetime import datetime
    if day == "today":
        day = datetime.now().strftime("%A")   # e.g. "Monday"
    result = get_supabase().table("timetable")\
        .select("*, teachers(name)")\
        .eq("class", class_name)\
        .eq("day", day)\
        .order("period_number")\
        .execute()
    return json.dumps(result.data)
```

### WhatsApp Tools

```python
# app/tools/whatsapp_tools.py
from langchain_core.tools import tool
import requests, os

@tool
def send_whatsapp_message(phone: str, message: str) -> str:
    """
    Send a WhatsApp message to a parent.
    phone: Indian format without spaces (e.g. 919876543210 — country code + number)
    message: plain text message (max 1000 chars)
    """
    response = requests.post(
        "https://api.gupshup.io/wa/api/v1/msg",
        headers={"apikey": os.getenv("WHATSAPP_API_KEY")},
        data={
            "channel": "whatsapp",
            "source": os.getenv("WHATSAPP_NUMBER"),
            "destination": phone,
            "message": message,
            "src.name": "EduSHAMIIT"
        }
    )
    return "Message sent successfully" if response.ok else f"Failed: {response.text}"

@tool
def send_bulk_whatsapp(class_name: str, message: str, filter_status: str = "all") -> str:
    """
    Send a WhatsApp message to all parents in a class.
    filter_status: 'all' | 'fee_pending' | 'absent_today'
    """
    # Fetch parent phones from Supabase based on class + filter
    students = get_supabase().table("students")\
        .select("parent_phone, name")\
        .eq("class", class_name)\
        .execute()
    
    sent_count = 0
    for s in students.data:
        if s["parent_phone"]:
            personalized = message.replace("{student_name}", s["name"])
            result = requests.post(...)   # same as above
            if result.ok:
                sent_count += 1
    
    return f"WhatsApp sent to {sent_count}/{len(students.data)} parents in {class_name}"
```

### IoT Tools

```python
# app/tools/iot_tools.py
from langchain_core.tools import tool
from app.services.iot_controller import iot

@tool
def control_classroom_device(room: str, device: str, action: str) -> str:
    """
    Control an electrical device in a classroom over WiFi.
    room:   classroom ID — e.g. 'class_10a', 'staff_room', 'library', 'principal_office'
    device: 'fan' | 'light1' | 'light2' | 'projector' | 'ac' | 'all' | 'all_lights'
    action: 'on' | 'off' | 'toggle'
    
    Examples:
      User: "Turn off the fan in 10A"
        → room='class_10a', device='fan', action='off'
      User: "Switch on lights in staffroom"
        → room='staff_room', device='all_lights', action='on'
      User: "Band karo sab lights class 9B mein"  (Hindi)
        → room='class_9b', device='all_lights', action='off'
    """
    result = iot.control_device(room, device, action)
    if result["ok"]:
        return f"Done — {device} in {room} has been turned {action}."
    return f"Could not reach {room}. The device board may be offline."

@tool
def get_classroom_device_status(room: str) -> str:
    """Get current on/off status of all devices in a classroom."""
    status = iot.get_room_status(room)
    if not status:
        return f"Cannot fetch status for {room}. Device may be offline."
    return str(status)

@tool
def schedule_device_action(room: str, device: str, action: str, time_str: str) -> str:
    """
    Schedule a device to turn on/off at a specific time.
    time_str: '14:30' (24h format) or '2 minutes from now'
    Example: Schedule all lights in class 10A to turn off at 17:00
    """
    # Parse time and insert into Supabase scheduled_actions table
    get_supabase().table("iot_scheduled_actions").insert({
        "room": room, "device": device,
        "action": action, "scheduled_time": parse_time(time_str),
        "status": "pending"
    }).execute()
    return f"Scheduled: {device} in {room} will {action} at {time_str}"
```

### ALL_TOOLS Assembly

```python
# app/tools/__init__.py
from app.tools.student_tools import get_student_tools
from app.tools.fee_tools import get_fee_tools
from app.tools.academic_tools import get_academic_tools
from app.tools.whatsapp_tools import get_whatsapp_tools
from app.tools.rag_tools import get_rag_tools
from app.tools.iot_tools import get_iot_tools

def get_all_tools(school_id: str) -> list:
    return [
        *get_student_tools(school_id),
        *get_fee_tools(school_id),
        *get_academic_tools(school_id),
        *get_whatsapp_tools(school_id),
        *get_rag_tools(school_id),
        *get_iot_tools(school_id),
    ]

def filter_tools_by_role(tools: list, role: str, school_id: str) -> list:
    from app.agents.router import ROLE_TOOLS
    allowed = ROLE_TOOLS.get(role, [])
    if "*" in allowed:
        return tools
    return [t for t in tools if t.name in allowed]
```

---

## 9. RAG Pipeline

RAG (Retrieval Augmented Generation) allows Shami to answer questions using the school's actual curriculum: NCERT textbooks, past papers, school policies, notes uploaded by teachers.

### Supabase pgvector Setup

```sql
-- Enable pgvector
CREATE EXTENSION IF NOT EXISTS vector;

-- Knowledge base table
CREATE TABLE knowledge_base (
  id         uuid    DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id  uuid    REFERENCES schools(id),
  subject    text,          -- 'Mathematics', 'Physics', 'History'
  grade      text,          -- 'Class 10', 'Class 12'
  source     text,          -- 'NCERT_Math_Ch5', 'Past_Paper_2023'
  content    text,          -- raw text chunk (500 words)
  embedding  vector(1536),  -- OpenAI small embedding dimension
  metadata   jsonb,         -- page number, chapter, etc.
  created_at timestamptz DEFAULT now()
);

-- IVFFlat index for fast cosine similarity search
CREATE INDEX ON knowledge_base
  USING ivfflat (embedding vector_cosine_ops)
  WITH (lists = 100);

-- RLS: Each school can only read its own knowledge base
ALTER TABLE knowledge_base ENABLE ROW LEVEL SECURITY;
CREATE POLICY "school_kb" ON knowledge_base
  USING (school_id = (SELECT school_id FROM user_profiles WHERE id = auth.uid()));
```

### Document Ingestion

```python
# services/rag_ingestion.py
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import SupabaseVectorStore

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

async def ingest_pdf(pdf_path: str, school_id: str, subject: str, 
                      grade: str, source: str):
    """Load a PDF, chunk it, embed it, and store in Supabase pgvector."""
    loader  = PyPDFLoader(pdf_path)
    docs    = loader.load()
    
    splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
    chunks   = splitter.split_documents(docs)
    
    for chunk in chunks:
        chunk.metadata.update({
            "school_id": school_id,
            "subject":   subject,
            "grade":     grade,
            "source":    source,
        })
    
    vector_store = SupabaseVectorStore(
        client=get_supabase(),
        embedding=embeddings,
        table_name="knowledge_base",
        query_name="match_documents",
    )
    await vector_store.aadd_documents(chunks)
    return f"Ingested {len(chunks)} chunks from {source}"
```

### RAG Tool

```python
# app/tools/rag_tools.py
from langchain_core.tools import tool
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import SupabaseVectorStore

def get_rag_tools(school_id: str) -> list:
    embeddings   = OpenAIEmbeddings(model="text-embedding-3-small")
    vector_store = SupabaseVectorStore(
        client=get_supabase(),
        embedding=embeddings,
        table_name="knowledge_base",
        query_name="match_documents",
    )

    @tool
    def search_curriculum(query: str, subject: str = "", grade: str = "") -> str:
        """
        Search the school's curriculum, textbooks, past papers, and study materials
        to find relevant content for answering academic questions.
        
        Use this tool FIRST for any academic question before generating an answer.
        query: the student's question in natural language
        subject: optional filter (e.g. 'Mathematics', 'Physics')
        grade: optional filter (e.g. 'Class 10', 'Class 12')
        """
        filter_dict = {"school_id": school_id}
        if subject: filter_dict["subject"] = subject
        if grade:   filter_dict["grade"]   = grade
        
        docs = vector_store.similarity_search(query, k=5, filter=filter_dict)
        if not docs:
            return "No relevant curriculum content found. Answering from general knowledge."
        
        context = "\n\n---\n\n".join([
            f"[Source: {d.metadata.get('source','Unknown')}]\n{d.page_content}"
            for d in docs
        ])
        return context

    return [search_curriculum]
```

---

## 10. IoT Device Control (ESP32 + MQTT)

### Hardware: ESP32 4-Channel Relay Board

- **Board:** ESP32 WROOM-32E + 4CH relay (220V AC powered, opto-isolated)
- **Cost:** ₹500–700 per room
- **Size:** ~85mm × 60mm (fits in standard switchboard gang box)
- **Channels:** 4 relays × 10A / 250V AC each
- **Relay logic:** Active LOW (GPIO LOW = relay ON, HIGH = relay OFF)
- **Available on:** Flipkart, Robu.in, etstore.in

**Channel mapping per room:**
```
CH1 → GPIO23 → Ceiling Fan
CH2 → GPIO22 → Tube Light Row 1 (front)
CH3 → GPIO21 → Tube Light Row 2 (back)
CH4 → GPIO19 → Projector / Smart TV
```

### ESP32 Firmware (Arduino IDE)

Flash with Arduino IDE. Board: `ESP32 Dev Module`.

```cpp
// esp32/relay_controller/relay_controller.ino
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include <WebServer.h>
#include <Preferences.h>   // NVS: saves state across power cuts

// ── Config — edit per device ────────────────────────────────
const char* WIFI_SSID   = "School_WiFi";
const char* WIFI_PASS   = "schoolwifipass";
const char* MQTT_SERVER = "192.168.1.100";   // Python server LAN IP
const char* DEVICE_ID   = "room_10a_board1"; // unique per room
const char* DEVICE_ROOM = "class_10a";

// ── Relay GPIO pins (active LOW) ─────────────────────────────
const int  RELAY_PINS[]  = {23, 22, 21, 19};
const char* RELAY_NAMES[] = {"fan", "light1", "light2", "projector"};
const int  NUM_RELAYS    = 4;

WiFiClient   espClient;
PubSubClient mqtt(espClient);
WebServer    http(80);
Preferences  prefs;

void onMqttMessage(char* topic, byte* payload, unsigned int len) {
  StaticJsonDocument<128> doc;
  deserializeJson(doc, payload, len);
  String action = doc["action"].as<String>();  // "on" | "off" | "toggle"
  String device = doc["device"].as<String>();  // "fan" | "light1" | "all"
  handleCommand(device, action);
}

void handleCommand(String device, String action) {
  for (int i = 0; i < NUM_RELAYS; i++) {
    if (device == RELAY_NAMES[i] || device == "all") {
      bool state = (action == "on");
      if (action == "toggle") state = digitalRead(RELAY_PINS[i]) == HIGH;
      digitalWrite(RELAY_PINS[i], state ? LOW : HIGH);  // active LOW!
      prefs.putBool(RELAY_NAMES[i], state);              // persist
    }
  }
  publishStatus();
}

void publishStatus() {
  String json = buildStatusJson();
  String topic = String("school/") + DEVICE_ROOM + "/status";
  mqtt.publish(topic.c_str(), json.c_str(), true);  // retained
}

void setupHttpServer() {
  http.on("/status",  HTTP_GET,  []() {
    http.send(200, "application/json", buildStatusJson());
  });
  http.on("/control", HTTP_POST, []() {
    StaticJsonDocument<128> doc;
    deserializeJson(doc, http.arg("plain"));
    handleCommand(doc["device"], doc["action"]);
    http.send(200, "application/json", "{\"ok\":true}");
  });
  http.begin();
}

void setup() {
  for (int i = 0; i < NUM_RELAYS; i++) {
    pinMode(RELAY_PINS[i], OUTPUT);
    bool saved = prefs.getBool(RELAY_NAMES[i], false);  // restore state
    digitalWrite(RELAY_PINS[i], saved ? LOW : HIGH);
  }
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  mqtt.setServer(MQTT_SERVER, 1883);
  mqtt.setCallback(onMqttMessage);
  setupHttpServer();
}

void loop() {
  if (!mqtt.connected()) reconnectMqtt();
  mqtt.loop();
  http.handleClient();
}

void reconnectMqtt() {
  if (mqtt.connect(DEVICE_ID)) {
    String topic = String("school/") + DEVICE_ROOM + "/control";
    mqtt.subscribe(topic.c_str());
    publishStatus();
  }
}
```

### Python IoT Controller Service

```python
# app/services/iot_controller.py
import paho.mqtt.client as mqtt
import httpx, json
from app.supabase.client import get_supabase

class IoTController:
    def __init__(self):
        self._mqtt = mqtt.Client(client_id="edushamiit-python")
        self._mqtt.on_message = self._on_status_update
        self._mqtt.connect("localhost", 1883)
        self._mqtt.subscribe("school/+/status")
        self._mqtt.loop_start()   # non-blocking background thread

    def control_device(self, room: str, device: str, action: str) -> dict:
        payload = json.dumps({"device": device, "action": action})
        topic   = f"school/{room}/control"
        result  = self._mqtt.publish(topic, payload, qos=1)
        if result.rc == 0:
            self._log(room, device, action)
            return {"ok": True, "method": "mqtt"}
        return self._http_fallback(room, device, action)

    def _http_fallback(self, room, device, action) -> dict:
        ip = self._get_device_ip(room)
        if not ip:
            return {"ok": False, "error": "device unreachable"}
        try:
            r = httpx.post(f"http://{ip}/control",
                           json={"device": device, "action": action}, timeout=3)
            return {"ok": r.status_code == 200, "method": "http"}
        except Exception:
            return {"ok": False, "error": "http timeout"}

    def get_room_status(self, room: str) -> dict:
        ip = self._get_device_ip(room)
        if not ip:
            return {}
        try:
            r = httpx.get(f"http://{ip}/status", timeout=3)
            return r.json()
        except Exception:
            return {}

    def _get_device_ip(self, room: str) -> str | None:
        result = get_supabase().table("iot_devices")\
            .select("ip_address").eq("room_id", room).single().execute()
        return result.data.get("ip_address") if result.data else None

    def _on_status_update(self, client, userdata, message):
        data = json.loads(message.payload)
        get_supabase().table("iot_device_states").upsert(data).execute()

    def _log(self, room, device, action):
        get_supabase().table("iot_control_log").insert({
            "room": room, "device": device, "action": action
        }).execute()

# Singleton — import this everywhere
iot = IoTController()
```

### MQTT Broker Setup

```bash
# Install on Ubuntu (same server as Python backend)
sudo apt install mosquitto mosquitto-clients
sudo systemctl enable --now mosquitto

# Or Docker:
docker run -d -p 1883:1883 --name mosquitto eclipse-mosquitto

# Test:
mosquitto_sub -t "school/class_10a/status"
mosquitto_pub -t "school/class_10a/control" -m '{"device":"fan","action":"on"}'
```

---

## 11. Camera Attendance (Computer Vision)

**Privacy rule:** All face processing runs on a local Python server inside the school. No student face images are sent to any cloud API.

**Hardware needed per school:**
- IP camera with RTSP support (₹2,000–5,000 each, 1 per classroom)
- Local mini-PC / NUC or spare laptop with NVIDIA GPU (RTX 3060 or better)
- Runs Ollama + YOLOv8 + DeepFace locally

```python
# app/services/camera_attendance.py
import cv2
from deepface import DeepFace
from ultralytics import YOLO
import asyncio
from app.supabase.client import get_supabase

yolo = YOLO("yolov8n.pt")    # nano model — fast, runs on CPU too

class ClassroomCamera:
    def __init__(self, rtsp_url: str, class_id: str, school_id: str):
        self.cap      = cv2.VideoCapture(rtsp_url)
        self.class_id = class_id
        self.school_id = school_id
        self.face_db  = f"/faces/{school_id}/{class_id}"  # local enrolled faces

    async def take_attendance(self) -> dict:
        ret, frame = self.cap.read()
        if not ret:
            return {"error": "camera feed unavailable"}

        # Step 1: Count people with YOLO (fast)
        results     = yolo.predict(frame, classes=[0], verbose=False)  # 0 = person
        headcount   = len(results[0].boxes)

        # Step 2: Identify faces with DeepFace (slower but accurate)
        present_ids = []
        for box in results[0].boxes:
            x1, y1, x2, y2 = map(int, box.xyxy[0])
            face_crop = frame[y1:y2, x1:x2]
            try:
                matches = DeepFace.find(
                    img_path=face_crop,
                    db_path=self.face_db,
                    model_name="ArcFace",
                    enforce_detection=False,
                    silent=True,
                )
                if matches and not matches[0].empty:
                    student_id = matches[0].iloc[0]["identity"].split("/")[-2]
                    present_ids.append(student_id)
            except Exception:
                pass

        # Step 3: Write to Supabase
        from datetime import datetime
        today = datetime.now().strftime("%Y-%m-%d")
        
        # Get all students in class
        all_students = get_supabase().table("students")\
            .select("id").eq("class_id", self.class_id).execute()
        all_ids = [s["id"] for s in all_students.data]
        
        records = []
        for sid in all_ids:
            records.append({
                "student_id":  sid,
                "class_id":    self.class_id,
                "school_id":   self.school_id,
                "date":        today,
                "status":      "present" if sid in present_ids else "absent",
                "method":      "camera_ai",
            })
        
        if records:
            get_supabase().table("attendance").upsert(records).execute()

        return {
            "headcount":  headcount,
            "identified": len(present_ids),
            "present_ids": present_ids,
            "date":       today,
        }
```

### Enroll Student Faces (One-time Setup)

```python
# scripts/enroll_faces.py
# Run once per student to add their face to the local DB

import cv2, os, shutil

def enroll_student(student_id: str, class_id: str, school_id: str,
                    photo_path: str):
    """Save student face photo to local face database."""
    face_dir = f"/faces/{school_id}/{class_id}/{student_id}"
    os.makedirs(face_dir, exist_ok=True)
    shutil.copy(photo_path, f"{face_dir}/face_01.jpg")
    print(f"Enrolled: {student_id} → {face_dir}")
```

---

## 12. Supabase Schema

```sql
-- ── Core Auth & Users ──────────────────────────────────────

CREATE TABLE schools (
  id         uuid  DEFAULT gen_random_uuid() PRIMARY KEY,
  name       text  NOT NULL,
  address    text,
  phone      text,
  logo_url   text,
  created_at timestamptz DEFAULT now()
);

CREATE TABLE user_profiles (
  id         uuid  REFERENCES auth.users(id) PRIMARY KEY,
  school_id  uuid  REFERENCES schools(id),
  role       text  NOT NULL,  -- 'student'|'teacher'|'parent'|'principal'|etc.
  name       text,
  phone      text,
  created_at timestamptz DEFAULT now()
);

-- ── Academic ────────────────────────────────────────────────

CREATE TABLE students (
  id           uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id    uuid REFERENCES schools(id),
  user_id      uuid REFERENCES user_profiles(id),
  name         text NOT NULL,
  class        text,           -- 'X', 'XI', 'XII'
  section      text,           -- 'A', 'B', 'C'
  roll_no      int,
  parent_phone text,
  parent_email text,
  dob          date,
  created_at   timestamptz DEFAULT now()
);

CREATE TABLE attendance (
  id         uuid  DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id  uuid  REFERENCES schools(id),
  student_id uuid  REFERENCES students(id),
  class_id   text,
  date       date  NOT NULL,
  status     text  DEFAULT 'absent',  -- 'present'|'absent'|'late'
  method     text  DEFAULT 'manual',  -- 'manual'|'camera_ai'
  UNIQUE(student_id, date)
);

CREATE TABLE results (
  id              uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id       uuid REFERENCES schools(id),
  student_id      uuid REFERENCES students(id),
  exam_name       text,
  subject         text,
  marks_obtained  numeric,
  max_marks       numeric,
  grade           text,
  exam_date       date,
  created_at      timestamptz DEFAULT now()
);

CREATE TABLE fees (
  id           uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id    uuid REFERENCES schools(id),
  student_id   uuid REFERENCES students(id),
  fee_type     text,      -- 'tuition'|'transport'|'hostel'|'activity'
  amount_due   numeric,
  due_date     date,
  status       text DEFAULT 'pending',  -- 'pending'|'paid'|'partial'
  academic_year text,
  paid_at      timestamptz,
  razorpay_link text,
  created_at   timestamptz DEFAULT now()
);

CREATE TABLE timetable (
  id            uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id     uuid REFERENCES schools(id),
  class         text,
  section       text,
  day           text,   -- 'Monday'...'Saturday'
  period_number int,
  subject       text,
  teacher_id    uuid REFERENCES user_profiles(id),
  room          text,
  start_time    time,
  end_time      time
);

-- ── AI Conversations ────────────────────────────────────────

CREATE TABLE ai_conversations (
  id         uuid  DEFAULT gen_random_uuid() PRIMARY KEY,
  session_id uuid  NOT NULL,
  school_id  uuid  REFERENCES schools(id),
  user_id    uuid  REFERENCES user_profiles(id),
  role       text  NOT NULL,  -- 'user' | 'assistant'
  content    text  NOT NULL,
  tool_data  jsonb,           -- structured tool result attached to this message
  created_at timestamptz DEFAULT now()
);

CREATE INDEX ON ai_conversations(session_id, created_at);

-- ── Knowledge Base (RAG) ────────────────────────────────────

CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE knowledge_base (
  id        uuid   DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id uuid   REFERENCES schools(id),
  subject   text,
  grade     text,
  source    text,
  content   text,
  embedding vector(1536),
  metadata  jsonb,
  created_at timestamptz DEFAULT now()
);

CREATE INDEX ON knowledge_base USING ivfflat (embedding vector_cosine_ops)
  WITH (lists = 100);

-- ── IoT Device Management ───────────────────────────────────

CREATE TABLE iot_devices (
  id         uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id  uuid REFERENCES schools(id),
  room_id    text NOT NULL,        -- 'class_10a', 'staff_room'
  device_id  text UNIQUE NOT NULL, -- 'room_10a_board1'
  ip_address text,
  num_relays int  DEFAULT 4,
  last_seen  timestamptz,
  is_online  boolean DEFAULT true
);

CREATE TABLE iot_device_states (
  device_id  text PRIMARY KEY,
  room_id    text,
  fan        text DEFAULT 'off',
  light1     text DEFAULT 'off',
  light2     text DEFAULT 'off',
  projector  text DEFAULT 'off',
  updated_at timestamptz DEFAULT now()
);

CREATE TABLE iot_control_log (
  id           uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id    uuid,
  room_id      text,
  device       text,
  action       text,
  triggered_by text DEFAULT 'ai_assistant',  -- 'ai_assistant'|'teacher_manual'|'schedule'
  user_id      uuid,
  created_at   timestamptz DEFAULT now()
);

CREATE TABLE iot_scheduled_actions (
  id             uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  room_id        text,
  device         text,
  action         text,
  scheduled_time timestamptz,
  status         text DEFAULT 'pending',  -- 'pending'|'done'|'failed'
  created_at     timestamptz DEFAULT now()
);

-- ── Row Level Security ──────────────────────────────────────

ALTER TABLE students       ENABLE ROW LEVEL SECURITY;
ALTER TABLE attendance     ENABLE ROW LEVEL SECURITY;
ALTER TABLE results        ENABLE ROW LEVEL SECURITY;
ALTER TABLE fees           ENABLE ROW LEVEL SECURITY;
ALTER TABLE knowledge_base ENABLE ROW LEVEL SECURITY;
ALTER TABLE ai_conversations ENABLE ROW LEVEL SECURITY;

-- Users see only their school's data
CREATE POLICY "school_isolation" ON students
  USING (school_id = (SELECT school_id FROM user_profiles WHERE id = auth.uid()));

-- Users see only their own conversations
CREATE POLICY "own_conversations" ON ai_conversations
  USING (user_id = auth.uid());
```

---

## 13. FastAPI Endpoints

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api import chat, voice, image, iot

app = FastAPI(title="Shami AI — EduSHAMIIT | Shami Innovation and Technologies", version="1.0.0")
app.add_middleware(CORSMiddleware, allow_origins=["*"],
                   allow_methods=["*"], allow_headers=["*"])

app.include_router(chat.router,  prefix="/chat")
app.include_router(voice.router, prefix="/chat")
app.include_router(image.router, prefix="/chat")
app.include_router(iot.router,   prefix="/iot")
```

```
POST   /chat/message          Text message → streaming SSE response
POST   /chat/voice            Audio file → transcribe + agent + response
POST   /chat/image            Image file + optional question → vision response
GET    /chat/history/{session_id}   Load past messages for a session

POST   /iot/control           Control a device: {room, device, action}
GET    /iot/status/{room}     Get current device states in a room
POST   /iot/schedule          Schedule a future device action
GET    /iot/devices           List all registered ESP32 devices

POST   /rag/ingest            Upload PDF → chunk → embed → store
GET    /rag/documents         List ingested documents for a school
DELETE /rag/documents/{id}    Remove a document from knowledge base
```

**Streaming SSE response format:**

```
data: {"type":"text","content":"Rohan's attendance is "}
data: {"type":"text","content":"94% this month."}
data: {"type":"tool_result","tool":"get_attendance_report","data":{...}}
data: {"type":"done"}
```

---

## 14. Flutter Integration

### AI Service (HTTP + SSE Streaming)

```dart
// lib/services/ai_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class AiService {
  final String _base = 'https://your-api.railway.app';

  Stream<AiChunk> streamMessage({
    required String message,
    required String sessionId,
    required String token,
  }) async* {
    final req = http.Request('POST', Uri.parse('$_base/chat/message'))
      ..headers = {
        'Authorization': 'Bearer $token',
        'Content-Type': 'application/json',
      }
      ..body = jsonEncode({'message': message, 'session_id': sessionId});

    final response = await http.Client().send(req);
    await for (final line in response.stream
        .transform(utf8.decoder)
        .transform(const LineSplitter())) {
      if (line.startsWith('data: ')) {
        final data = jsonDecode(line.substring(6));
        yield AiChunk.fromJson(data);
      }
    }
  }

  Future<AiVoiceResponse> sendVoice(String filePath, String token) async {
    final req = http.MultipartRequest('POST', Uri.parse('$_base/chat/voice'))
      ..headers['Authorization'] = 'Bearer $token'
      ..files.add(await http.MultipartFile.fromPath('audio', filePath));
    final res = await req.send();
    return AiVoiceResponse.fromJson(
        jsonDecode(await res.stream.bytesToString()));
  }

  Future<String> sendImage(String filePath, String question, String token) async {
    final req = http.MultipartRequest('POST', Uri.parse('$_base/chat/image'))
      ..headers['Authorization'] = 'Bearer $token'
      ..fields['question'] = question
      ..files.add(await http.MultipartFile.fromPath('image', filePath));
    final res = await req.send();
    final body = jsonDecode(await res.stream.bytesToString());
    return body['output'];
  }
}
```

### Riverpod Chat State

```dart
// lib/providers/ai_chat_provider.dart
final aiChatProvider = StateNotifierProvider.autoDispose
    <AiChatNotifier, AiChatState>((ref) {
  return AiChatNotifier(ref.read(aiServiceProvider), ref.read(authProvider).token);
});

class AiChatNotifier extends StateNotifier<AiChatState> {
  AiChatNotifier(this._service, this._token) : super(AiChatState.empty());

  Future<void> sendText(String text) async {
    state = state.addUserMessage(text).startStreaming();
    await for (final chunk in _service.streamMessage(message: text, ...)) {
      if (chunk.type == 'text')        state = state.appendToken(chunk.content);
      if (chunk.type == 'tool_result') state = state.addToolResult(chunk);
      if (chunk.type == 'done')        state = state.finishStreaming();
    }
  }
}
```

### Chat Message Widget

```dart
// lib/widgets/chat_message_widget.dart
Widget buildMessage(ChatMessage msg) {
  return switch (msg.type) {
    MsgType.text       => MarkdownBubble(text: msg.text, streaming: msg.isStreaming),
    MsgType.image      => ImageMessageBubble(url: msg.imageUrl),
    MsgType.voice      => VoiceMessageBubble(audioUrl: msg.audioUrl, transcript: msg.transcript),
    MsgType.feeCard    => FeeStatusCard(data: FeeData.fromJson(msg.toolData)),
    MsgType.payLink    => PaymentLinkCard(url: msg.toolData['payment_url']),
    MsgType.attendance => AttendanceCard(data: msg.toolData),
    MsgType.deviceCtrl => DeviceControlCard(room: msg.toolData['room'], result: msg.toolData),
    _                  => TextBubble(text: msg.text),
  };
}
```

---

## 15. Security & Auth

### JWT Middleware

```python
# app/supabase/auth.py
from fastapi import HTTPException, Depends
from fastapi.security import HTTPBearer
from jose import jwt, JWTError
import os

security = HTTPBearer()

async def get_current_user(token=Depends(security)) -> UserContext:
    try:
        payload = jwt.decode(
            token.credentials,
            os.getenv("SUPABASE_JWT_SECRET"),
            algorithms=["HS256"],
            options={"verify_aud": False}
        )
        user_id = payload["sub"]
        profile = get_supabase().table("user_profiles")\
            .select("role, school_id, name")\
            .eq("id", user_id).single().execute()
        return UserContext(
            id=user_id,
            role=profile.data["role"],
            school_id=profile.data["school_id"],
            name=profile.data["name"],
        )
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid or expired token")
```

### Security Rules

- Python backend uses `SUPABASE_SERVICE_ROLE_KEY` (bypasses RLS — server-side only)
- Flutter uses only `SUPABASE_ANON_KEY` + user JWT (RLS enforced)
- API keys (Anthropic, Google, OpenAI, Razorpay) live only in backend `.env`
- Never put any API key inside the Flutter app
- IoT MQTT broker runs on LAN only — not exposed to internet
- Face image data never leaves the school premises

---

## 16. Environment Variables

```bash
# .env

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJ...      # server only — never expose to client
SUPABASE_ANON_KEY=eyJ...              # safe for Flutter
SUPABASE_JWT_SECRET=your-jwt-secret   # from Supabase dashboard → Settings → API

# AI Models
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=AIza...                # Gemini Flash
OPENAI_API_KEY=sk-...                 # Whisper + Embeddings

# Payments
RAZORPAY_KEY_ID=rzp_live_...
RAZORPAY_SECRET=...

# WhatsApp
WHATSAPP_API_KEY=...                  # Gupshup API key
WHATSAPP_NUMBER=917XXXXXXXXX          # your WhatsApp Business number

# TTS
GOOGLE_TTS_API_KEY=...               # or ElevenLabs
ELEVENLABS_API_KEY=...

# IoT
MQTT_BROKER_HOST=localhost
MQTT_BROKER_PORT=1883

# Redis (optional — for caching)
REDIS_URL=redis://localhost:6379

# App
APP_ENV=production
```

---

## 17. Deployment

### Python Backend → Railway.app

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

```bash
# Deploy
railway login
railway init
railway add            # add environment variables from .env
railway up
```

### MQTT Broker → Same Server or Docker

```bash
docker run -d \
  --name mosquitto \
  --restart always \
  -p 1883:1883 \
  -v ./mosquitto.conf:/mosquitto/config/mosquitto.conf \
  eclipse-mosquitto
```

### Camera Service → On-Premise School Server

```bash
# Run camera attendance service locally at school
pip install deepface ultralytics opencv-python
python -m app.services.camera_attendance --class class_10a --rtsp rtsp://192.168.1.50/stream
```

### ESP32 → Flash per Room

1. Edit `WIFI_SSID`, `WIFI_PASS`, `MQTT_SERVER`, `DEVICE_ID`, `DEVICE_ROOM`
2. Flash with Arduino IDE
3. Add device to Supabase `iot_devices` table with its LAN IP address
4. Verify with: `mosquitto_sub -t "school/class_10a/status"`

---

## 18. Cost Optimisation

### Monthly Estimate (500 students, 50 teachers)

| Service | Est. Cost/Month |
|---|---|
| Gemini 1.5 Flash (2M tokens/day) | ~$18 |
| Whisper STT (200 min/day) | ~$36 |
| Claude Haiku (500 tool calls/day) | ~$12 |
| text-embedding-3-small (5K queries/day) | ~$1 |
| Google TTS (50K chars/day) | ~$22 |
| Camera AI (on-premise) | $0 |
| Railway + Supabase hosting | ~$25 |
| **Total** | **~$114 (~₹9,500/month)** |

### Cost Reduction Strategies

1. **Cache repeat answers** — Cache common Q&A in Redis. Same question from 100 students = 1 LLM call, not 100.
2. **Per-user daily limits** — Free tier: 20 AI queries/day. Premium: unlimited.
3. **Batch embedding** — Run PDF ingestion at night using Gemini's batch API (50% discount).
4. **Local LLM fallback** — Run Llama 3.2 on school server for simple one-word answers.
5. **Response deduplication** — Hash the question; return cached response if identical query within 1 hour.

---

## 19. Build Order

Follow this sequence. Each step depends on the previous.

```
PHASE 1 — Foundation (Week 1)
  [ ] 1. Supabase project setup + all schema tables created
  [ ] 2. RLS policies enabled
  [ ] 3. FastAPI project scaffolded (main.py, models.py)
  [ ] 4. JWT auth middleware working
  [ ] 5. Supabase singleton client (service role)

PHASE 2 — Core AI Agent (Week 2)
  [ ] 6. Gemini Flash LLM connected via langchain-google-genai
  [ ] 7. Claude Haiku connected via langchain-anthropic
  [ ] 8. Router (detect_task → get_llm) working
  [ ] 9. System prompts for all roles written
  [ ] 10. build_agent() function working
  [ ] 11. /chat/message endpoint with SSE streaming

PHASE 3 — Tools (Week 3)
  [ ] 12. student_tools.py (profile, attendance, results)
  [ ] 13. fee_tools.py (fee status, Razorpay link)
  [ ] 14. academic_tools.py (question generator, lesson plan)
  [ ] 15. whatsapp_tools.py (Gupshup integration)
  [ ] 16. Tool permission filtering by role
  [ ] 17. Chat history load/save to Supabase

PHASE 4 — RAG (Week 4)
  [ ] 18. pgvector extension enabled in Supabase
  [ ] 19. rag_ingestion.py — PDF → chunks → embeddings → store
  [ ] 20. rag_tools.py — search_curriculum tool
  [ ] 21. /rag/ingest endpoint for teacher document uploads
  [ ] 22. RAG tested end-to-end with NCERT sample PDF

PHASE 5 — Multimodal (Week 5)
  [ ] 23. /chat/voice — Whisper STT working
  [ ] 24. /chat/image — Gemini Flash vision working
  [ ] 25. TTS response (Google TTS → audio URL returned)
  [ ] 26. Flutter: voice recording, image picker, audio player

PHASE 6 — IoT (Week 6)
  [ ] 27. MQTT broker (Mosquitto) running on server
  [ ] 28. ESP32 firmware flashed on first test board
  [ ] 29. iot_controller.py MQTT service working
  [ ] 30. iot_tools.py LangChain tools connected
  [ ] 31. /iot/control and /iot/status endpoints
  [ ] 32. Test: AI command → MQTT → relay fires

PHASE 7 — Camera (Week 7–8)
  [ ] 33. YOLOv8 installed on local school server
  [ ] 34. DeepFace installed + student face enrollment
  [ ] 35. camera_attendance.py reading RTSP stream
  [ ] 36. Attendance auto-written to Supabase
  [ ] 37. Teacher can ask AI: "Who's absent today?" → accurate answer

PHASE 8 — Flutter UI (Parallel with above)
  [ ] 38. AiService (HTTP client + SSE streaming)
  [ ] 39. Riverpod AiChatNotifier state management
  [ ] 40. Chat screen: text, voice, image input buttons
  [ ] 41. Message widgets: text bubble, voice bubble, image bubble
  [ ] 42. Tool result cards: FeeStatusCard, AttendanceCard, PaymentLinkCard, DeviceControlCard

PHASE 9 — Production Hardening
  [ ] 43. Rate limiting (per user per day)
  [ ] 44. Redis response caching
  [ ] 45. Error handling + fallback messages
  [ ] 46. Logging (LangSmith or custom)
  [ ] 47. Railway deployment + env vars
  [ ] 48. OTA firmware update for ESP32 boards
```

---

*This document is the single source of truth for building Shami AI on EduSHAMIIT — by Shami Innovation and Technologies LLP.*  
*Every AI agent, code generator, or developer working on this project should start here.*

---
**Last updated:** April 2026  
**Maintainer:** Naresh — EduSHAMIIT
