# EduSHAMIIT AI — Python Backend Complete Specification

> **PURPOSE**: This document is the SELF-CONTAINED BLUEPRINT for the Shami AI Python backend. Any AI model or developer reading ONLY this file should be able to build the complete EduSHAMIIT AI assistant — a FastAPI + LangChain backend serving the Flutter EduSHAMIIT AI Academic App.

> **PRODUCT**: EduSHAMIIT
> **AI ASSISTANT NAME**: Shami (inspired by J.A.R.V.I.S.)
> **MARKET**: Indian school/college SaaS
> **BACKEND**: Python 3.11+ / FastAPI / LangChain 0.3 / Supabase (PostgreSQL)
> **MULTI-SCHOOL**: All data scoped by `school_id` for multi-institution support

---

## 📋 TABLE OF CONTENTS

1. [Project Identity & Tech Stack](#1-project-identity--tech-stack)
2. [Project Structure](#2-project-structure)
3. [Roles & Permissions](#3-roles--permissions)
4. [LLM Selection & Routing Strategy](#4-llm-selection--routing-strategy)
5. [Multimodal Input/Output](#5-multimodal-inputoutput)
6. [LangChain Agent Architecture](#6-langchain-agent-architecture)
7. [Student AI Tools (20 Tools)](#7-student-ai-tools-20-tools)
8. [Teacher AI Tools (20 Tools)](#8-teacher-ai-tools-20-tools)
9. [System Prompts](#9-system-prompts)
10. [RAG Pipeline](#10-rag-pipeline)
11. [IoT Device Control (ESP32 + MQTT)](#11-iot-device-control-esp32--mqtt)
12. [Camera Attendance (Computer Vision)](#12-camera-attendance-computer-vision)
13. [FastAPI Endpoints](#13-fastapi-endpoints)
14. [Supabase Schema (with school_id)](#14-supabase-schema-with-school_id)
15. [Flutter Integration](#15-flutter-integration)
16. [Security & Auth](#16-security--auth)
17. [Environment Variables](#17-environment-variables)
18. [Deployment](#18-deployment)
19. [Cost Optimisation](#19-cost-optimisation)
20. [Build Order](#20-build-order)

---

## 1. PROJECT IDENTITY & TECH STACK

**Product name:** EduSHAMIIT
**AI assistant name:** Shami
**India-specific defaults:**
- Currency: INR with Indian number formatting (₹1,23,456)
- Date format: DD/MM/YYYY
- Phone prefix: +91
- Languages: English + Hindi (Hinglish accepted in voice)

**Shami AI personality:**
- Proactive, intelligent, always on — like Jarvis for Tony Stark
- Warm, concise, authoritative
- Responds in the user's language (Hindi or English, auto-detected)
- Never hallucinates student data — always fetches from tools
- Addresses teachers formally, students encouragingly
- Proactively surfaces insights ("Sir, 3 teachers are absent today — shall I arrange substitutes?")
- Acknowledges when it cannot do something and explains why

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | Flutter + Riverpod | Mobile app (EduSHAMIIT Academic) |
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
| File storage | Supabase Storage | Documents, voice files |
| Real-time | Supabase Realtime | Live dashboard updates |
| IoT messaging | MQTT (Mosquitto) | ESP32 relay board control |
| IoT hardware | ESP32 + 4CH relay board | Classroom device control |
| Camera AI | YOLOv8 + DeepFace | Student count + attendance |
| Payments | Razorpay SDK | Fee collection + payment links |
| WhatsApp | Gupshup / Meta Cloud API | Parent notifications |
| Deployment | Railway.app | Python backend hosting |
| Cache | Redis | Response caching, device state |

---

## 2. PROJECT STRUCTURE

```
edushamiit-ai-backend/
├── app/
│   ├── main.py                    # FastAPI app entry point
│   ├── models.py                  # Pydantic request/response schemas
│   ├── config.py                  # Environment config loader
│   ├── api/
│   │   ├── chat.py                # POST /chat/message (text + streaming)
│   │   ├── voice.py               # POST /chat/voice (audio upload)
│   │   ├── image.py               # POST /chat/image (image upload)
│   │   └── iot.py                 # POST /iot/control, GET /iot/status/{room}
│   ├── agents/
│   │   ├── shamiit_agent.py       # Main LangChain agent builder
│   │   ├── router.py              # Task detection → LLM routing
│   │   └── prompts.py             # Role-specific system prompts
│   ├── tools/
│   │   ├── student_tools.py       # Student profile, attendance, results
│   │   ├── teacher_tools.py       # Class management, grading, analytics
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

## 3. ROLES & PERMISSIONS

EduSHAMIIT has roles. Shami's tool access is gated by role.

| Role | Shami Capabilities |
|---|---|
| student | Profile, attendance, results, timetable, homework, fees, library, achievements, events, live classes, notices, leaderboard, leave application, bus tracking, study plans, concept explanation, practice questions |
| teacher | Class students, class performance, at-risk students, question generation, lesson plans, auto-grading, submission status, remedial plans, notices, attendance stats, schedule, pending tasks, reports, pedagogy, leave balance, salary, parent comms, materials upload, exam analytics |
| parent | Child attendance, child results, fee status, Razorpay payment links |
| principal | All tools (full access) |
| admin | Student profile, fee status, Razorpay links, WhatsApp |
| finance | Fee status, Razorpay links, WhatsApp |

```python
ROLE_TOOLS = {
    "student":  ["get_timetable", "get_homework", "get_fee_status", "get_attendance",
                 "get_exam_info", "get_bus_location", "get_performance", "generate_study_plan",
                 "explain_concept", "get_notifications", "get_library_status", "get_achievements",
                 "get_events", "get_leaderboard", "submit_homework", "apply_leave",
                 "get_notices", "answer_general", "generate_practice", "get_live_class"],
    "teacher":  ["get_class_students", "get_class_performance", "get_at_risk_students",
                 "generate_questions", "generate_lesson_plan", "auto_grade_homework",
                 "get_submission_status", "generate_remedial_plan", "create_notice",
                 "get_attendance_stats", "get_teacher_schedule", "get_pending_tasks",
                 "generate_report", "explain_pedagogy", "get_leave_balance", "get_salary_info",
                 "get_parent_communications", "upload_material", "get_exam_analytics",
                 "answer_general"],
    "parent":   ["get_attendance", "get_results", "get_fee_status", "create_razorpay_link"],
    "principal": ["*"],   # all tools
    "admin":    ["get_student_profile", "get_fee_status", "create_razorpay_link", "send_whatsapp"],
    "finance":  ["get_fee_status", "create_razorpay_link", "send_whatsapp"],
}
```

---

## 4. LLM SELECTION & ROUTING STRATEGY

**Core principle:** Route each task to the cheapest capable model.

| Model | Use for |
|---|---|
| `gemini-1.5-flash-latest` | Text Q&A, RAG, vision, question generation |
| `claude-haiku-4-5-20251001` | Tool calls, IoT JSON commands, DB operations |
| `whisper-1` | Voice STT (Hindi + English) |
| Google TTS / ElevenLabs | Voice TTS output |
| `text-embedding-3-small` | RAG embeddings |

### Routing Logic

```python
# app/agents/router.py

from enum import Enum

class TaskType(Enum):
    QA          = "qa"
    GENERATION  = "generation"
    TOOL_CALL   = "tool_call"
    IOT_CONTROL = "iot_control"
    VISION      = "vision"
    RAG         = "rag"

IOT_KEYWORDS  = ["fan", "light", "lights", "camera", "turn on", "turn off",
                  "switch", "projector", "ac", "bulb", "band karo"]
TOOL_KEYWORDS = ["fee", "fees", "attendance", "marks", "result", "schedule",
                  "timetable", "leave", "salary", "payment", "dues", "absent",
                  "homework", "exam", "bus", "library", "achievement", "leaderboard",
                  "event", "notice", "live class", "class", "student", "performance",
                  "grade", "submission", "task"]

def detect_task(message: str, has_image: bool = False) -> TaskType:
    if has_image:
        return TaskType.VISION
    msg = message.lower()
    if any(k in msg for k in IOT_KEYWORDS):
        return TaskType.IOT_CONTROL
    if any(k in msg for k in TOOL_KEYWORDS):
        return TaskType.TOOL_CALL
    if any(k in msg for k in ["explain", "solve", "what is", "how does", "help me"]):
        return TaskType.RAG
    return TaskType.QA

def get_llm(task: TaskType):
    if task in (TaskType.TOOL_CALL, TaskType.IOT_CONTROL):
        from langchain_anthropic import ChatAnthropic
        return ChatAnthropic(model="claude-haiku-4-5-20251001", temperature=0.0)
    from langchain_google_genai import ChatGoogleGenerativeAI
    return ChatGoogleGenerativeAI(model="gemini-1.5-flash-latest", temperature=0.3)
```

---

## 5. MULTIMODAL INPUT/OUTPUT

### Text Input → Text Output (primary)
- User sends text message via Flutter
- FastAPI receives → LangChain agent processes → streams SSE response

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

```python
# services/whisper_service.py
from openai import AsyncOpenAI

oai = AsyncOpenAI()

async def transcribe(audio_bytes: bytes, filename: str, content_type: str) -> str:
    transcript = await oai.audio.transcriptions.create(
        model="whisper-1",
        file=(filename, audio_bytes, content_type),
        language="hi",
        response_format="text"
    )
    return transcript
```

### Image Input → Text Output
```python
# api/image.py
import base64
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.messages import HumanMessage

async def analyze_image(image_bytes: bytes, content_type: str, question: str) -> str:
    b64 = base64.b64encode(image_bytes).decode()
    llm = ChatGoogleGenerativeAI(model="gemini-1.5-flash-latest")
    msg = HumanMessage(content=[
        {"type": "image_url", "image_url": {"url": f"data:{content_type};base64,{b64}"}},
        {"type": "text", "text": question or "Describe and analyze this image."}
    ])
    response = await llm.ainvoke([msg])
    return response.content
```

### Tool Result → Structured Card (Flutter)
When agent calls a tool, response JSON includes `tool_result` key with structured data. Flutter renders role-specific widget.

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

## 6. LANGCHAIN AGENT ARCHITECTURE

### Main Agent Builder

```python
# app/agents/shamiit_agent.py
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_anthropic import ChatAnthropic
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from app.agents.router import detect_task, get_llm
from app.tools import get_all_tools, filter_tools_by_role
from app.agents.prompts import get_system_prompt
from app.supabase.client import get_supabase

def build_agent(role: str, school_id: str, task_type: str = "qa") -> AgentExecutor:
    llm   = get_llm(task_type)
    tools = filter_tools_by_role(get_all_tools(school_id), role)

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
    if audio_path:
        from app.services.whisper_service import transcribe_file
        text = await transcribe_file(audio_path)

    if image_b64:
        from app.services.vision_service import analyze_image
        result = await analyze_image(image_b64, text)
        return {"type": "vision", "output": result, "tool_result": None}

    task     = detect_task(text)
    agent    = build_agent(user.role, user.school_id, task.value)
    history  = load_history(session_id)

    result = await agent.ainvoke({
        "input": text,
        "chat_history": history,
    })

    await save_message(session_id, user.id, "user", text)
    await save_message(session_id, user.id, "assistant", result["output"],
                       tool_data=extract_tool_results(result))

    return {
        "type": "text",
        "output": result["output"],
        "tool_result": extract_tool_results(result),
    }
```

---

## 7. STUDENT AI TOOLS (20 TOOLS)

All student tools query Supabase with `school_id` scoping.

### Tool 1: get_timetable
```python
from langchain_core.tools import tool
from app.supabase.client import get_supabase
import json

@tool
def get_timetable(day: str = "today", school_id: str = "") -> str:
    """Get the student timetable for a specific day or the whole week.
    Input: day name like "monday", "tuesday", etc., or "week" for full week.
    Use when student asks about their schedule, classes, or what they have today."""
    from datetime import datetime

    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles").select("class").eq("id", user_id).single().execute()
    student_class = profile.data["class"]

    day_map = {"monday":0,"tuesday":1,"wednesday":2,"thursday":3,"friday":4,"saturday":5}
    day_lower = day.lower().strip()

    query = sb.table("timetable")\
        .select("*, subjects(name, icon, color)")\
        .eq("school_id", school_id)\
        .eq("class", student_class)\
        .order("start_time")

    if day_lower in day_map:
        query = query.eq("day_of_week", day_map[day_lower])

    response = query.execute()
    return _format_timetable(response.data, day_lower)

def _format_timetable(data, day):
    if not data:
        return f"No classes scheduled{' for ' + day if day != 'week' else ''}. Enjoy! 🎉"
    days = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday']
    buf = []
    current_day = None
    for item in data:
        dow = item['day_of_week']
        if current_day != dow:
            current_day = dow
            buf.append(f"\n📅 {days[dow]}:")
        if item.get('is_break'):
            buf.append(f"  ☕ {item.get('break_name','Break')} ({item['start_time']} - {item['end_time']})")
        elif item.get('is_free_period'):
            buf.append(f"  🆓 Free Period ({item['start_time']} - {item['end_time']})")
        else:
            subj = item.get('subjects',{})
            buf.append(f"  {subj.get('icon','📚')} {subj.get('name','Unknown')} - {item['start_time']} to {item['end_time']} (Room {item.get('room','TBD')})")
    return "\n".join(buf)
```

### Tool 2: get_homework
```python
@tool
def get_homework(filter_status: str = "all", school_id: str = "") -> str:
    """Get homework assignments. Filter: "pending", "submitted", "graded", or "all".
    Use when student asks about homework, assignments, or what is due."""
    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles").select("class").eq("id", user_id).single().execute()
    student_class = profile.data["class"]

    homework = sb.table("homework")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .eq("target_class", student_class)\
        .eq("status", "active")\
        .order("due_date").execute().data

    submissions = sb.table("homework_submissions")\
        .select("homework_id, status, grade, marks")\
        .eq("student_id", user_id).execute().data

    submitted_ids = {s["homework_id"] for s in submissions}

    filtered = []
    for hw in homework:
        sub = next((s for s in submissions if s["homework_id"] == hw["id"]), None)
        if filter_status == "pending" and hw["id"] not in submitted_ids:
            filtered.append((hw, None))
        elif filter_status == "submitted" and sub and sub["status"] == "submitted":
            filtered.append((hw, sub))
        elif filter_status == "graded" and sub and sub["status"] == "graded":
            filtered.append((hw, sub))
        elif filter_status == "all":
            filtered.append((hw, sub))

    if not filtered:
        return "No homework found! Enjoy your free time! 🎉"

    buf = []
    pending_count = 0
    for hw, sub in filtered:
        icon = hw.get("subjects",{}).get("icon","📝")
        title = hw["title"]
        subject = hw.get("subjects",{}).get("name","Unknown")
        due = hw.get("due_date","TBD")

        if sub and sub["status"] == "graded":
            status = f"✅ Graded: {sub.get('grade','N/A')} ({sub.get('marks','N/A')} marks)"
        elif sub:
            status = "📤 Submitted - Awaiting review"
        else:
            pending_count += 1
            from datetime import datetime
            due_dt = datetime.try_parse(due) if due != "TBD" else None
            if due_dt and due_dt < datetime.now():
                status = "🔴 OVERDUE!"
            else:
                status = "📋 Pending"

        buf.append(f"{icon} {title}\n   Subject: {subject}\n   Due: {due}\n   Status: {status}\n")

    if pending_count > 0:
        buf.append(f"⚠️ You have {pending_count} pending assignment(s). Submit before deadline!")
    return "\n".join(buf)
```

### Tool 3: get_fee_status
```python
@tool
def get_fee_status(school_id: str = "") -> str:
    """Check fee balance, pending fees, and payment history.
    Use when student asks about fees, payments, or outstanding balance."""
    sb = get_supabase()
    user_id = get_current_user_id()

    fees = sb.table("fees").select("*")\
        .eq("school_id", school_id)\
        .eq("student_id", user_id)\
        .order("due_date").execute().data

    payments = sb.table("payments").select("*")\
        .eq("student_id", user_id)\
        .order("paid_at", ascending=False).execute().data

    total_due = 0
    total_paid = 0
    pending = []
    for fee in fees:
        amount = float(fee["amount"])
        paid = float(fee.get("amount_paid", 0))
        status = fee["status"]
        if status in ("pending","partial","overdue"):
            total_due += (amount - paid)
            pending.append(fee)
        total_paid += paid

    buf = [f"💰 Fee Status Summary\n",
           f"Total Outstanding: ₹{total_due:,.0f}",
           f"Total Paid This Year: ₹{total_paid:,.0f}\n"]

    if pending:
        buf.append("📋 Pending Fees:")
        for fee in pending:
            remaining = float(fee["amount"]) - float(fee.get("amount_paid",0))
            emoji = "🔴" if fee["status"] == "overdue" else ("🟡" if fee["status"] == "partial" else "📋")
            buf.append(f"  {emoji} {fee.get('fee_type','Unknown')}: ₹{remaining:,.0f} remaining (Due: {fee.get('due_date','TBD')})")
    else:
        buf.append("✅ All fees are paid! Great job!")

    if payments:
        buf.append("\n📝 Recent Payments:")
        for p in payments[:3]:
            buf.append(f"  ✅ ₹{float(p['amount']):,.0f} via {p.get('payment_method','N/A')} - {p.get('paid_at','')}")

    return "\n".join(buf)
```

### Tool 4: get_attendance
```python
@tool
def get_attendance(school_id: str = "") -> str:
    """Get attendance percentage, both overall and per subject.
    Use when student asks about their attendance."""
    sb = get_supabase()
    user_id = get_current_user_id()

    attendance = sb.table("attendance")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .eq("student_id", user_id).execute().data

    if not attendance:
        return "No attendance records found yet."

    total = len(attendance)
    present = sum(1 for a in attendance if a["status"] == "present")
    absent = sum(1 for a in attendance if a["status"] == "absent")
    late = sum(1 for a in attendance if a["status"] == "late")
    overall_pct = (present / total * 100) if total > 0 else 0

    subject_map = {}
    for a in attendance:
        name = a.get("subjects",{}).get("name","Unknown")
        subject_map.setdefault(name, {"total":0,"present":0})
        subject_map[name]["total"] += 1
        if a["status"] == "present":
            subject_map[name]["present"] += 1

    buf = [f"📊 Attendance Summary\n",
           f"Overall: {overall_pct:.1f}% ({present}/{total} days)",
           f"  ✅ Present: {present} days",
           f"  ❌ Absent: {absent} days",
           f"  ⏰ Late: {late} days\n",
           f"📚 Subject-wise:"]
    for subj, counts in subject_map.items():
        pct = (counts["present"]/counts["total"]*100) if counts["total"]>0 else 0
        emoji = "✅" if pct>=90 else ("⚠️" if pct>=75 else "🔴")
        buf.append(f"  {emoji} {subj}: {pct:.0f}% ({counts['present']}/{counts['total']})")

    if overall_pct < 85:
        buf.append("\n⚠️ Your attendance is below 85%. Attend all upcoming classes.")

    return "\n".join(buf)
```

### Tool 5: get_exam_info
```python
@tool
def get_exam_info(school_id: str = "") -> str:
    """Get upcoming exam schedule, dates, and details.
    Use when student asks about exams, next exam, exam dates."""
    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles").select("class").eq("id", user_id).single().execute()
    student_class = profile.data["class"]

    from datetime import date
    today = date.today().isoformat()

    exams = sb.table("exams")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .contains("target_classes", f'["{student_class}"]')\
        .gte("exam_date", today)\
        .order("exam_date").limit(10).execute().data

    if not exams:
        return "No upcoming exams found. Enjoy the break! 🎉"

    buf = ["📝 Upcoming Exams:\n"]
    for exam in exams:
        subj = exam.get("subjects",{})
        buf.append(f"{subj.get('icon','📝')} {exam.get('title','Untitled')}")
        buf.append(f"   📅 Date: {exam.get('exam_date','TBD')}")
        buf.append(f"   ⏰ Time: {exam.get('start_time','TBD')} ({exam.get('duration_minutes',0)} minutes)")
        buf.append(f"   📍 Venue: {exam.get('venue','TBD')}")
        buf.append(f"   📊 Total Marks: {exam.get('total_marks',100)}")
        buf.append(f"   📋 Type: {exam.get('exam_type','offline').upper()}")
        buf.append("")

    from datetime import datetime
    next_date = datetime.fromisoformat(exams[0]["exam_date"])
    days_left = (next_date.date() - date.today()).days
    buf.append(f"⏰ Next exam is in {days_left} day(s). Start preparing!")

    return "\n".join(buf)
```

### Tool 6: get_bus_location
```python
@tool
def get_bus_location(school_id: str = "") -> str:
    """Get real-time bus location, ETA, and route information.
    Use when student asks about their bus, where is the bus, bus timing."""
    sb = get_supabase()
    user_id = get_current_user_id()

    transport = sb.table("student_transport")\
        .select("*, bus_routes(*), bus_stops(*)")\
        .eq("student_id", user_id).maybe_single().execute()

    if not transport or not transport.data:
        return "You are not assigned to any bus route. Contact the transport office."

    route = transport.data["bus_routes"]
    stop = transport.data.get("bus_stops",{})

    bus_loc = sb.table("bus_locations")\
        .select("*").eq("route_id", route["id"])\
        .order("recorded_at", ascending=False)\
        .limit(1).maybe_single().execute()

    buf = [f"🚌 Bus Tracking Information\n",
           f"Route: {route['route_name']}",
           f"Bus Number: {route.get('bus_number','N/A')}",
           f"Driver: {route.get('driver_name','N/A')}",
           f"Driver Phone: {route.get('driver_phone','N/A')}",
           f"Your Stop: {stop.get('stop_name','N/A')}\n"]

    if bus_loc and bus_loc.data:
        eta = bus_loc.data.get("eta_minutes")
        speed = bus_loc.data.get("speed")
        buf.append("📍 Live Location:")
        buf.append(f"   ETA to your stop: {eta if eta else 'Calculating...'} minutes")
        if speed:
            buf.append(f"   Current Speed: {speed} km/h")
        buf.append(f"   Last updated: {bus_loc.data.get('recorded_at','')}")
    else:
        buf.append("📍 Bus location data not available yet.")

    return "\n".join(buf)
```

### Tool 7: get_performance
```python
@tool
def get_performance(school_id: str = "") -> str:
    """Get academic performance summary including scores, grades, rank, and trends.
    Use when student asks about their overall performance, grades, how they are doing."""
    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles")\
        .select("full_name, class, xp_points, learning_streak, best_streak")\
        .eq("id", user_id).single().execute().data

    results = sb.table("results")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .eq("student_id", user_id)\
        .order("created_at", ascending=False).execute().data

    attendance = sb.table("attendance")\
        .select("status").eq("student_id", user_id).execute().data

    total_days = len(attendance)
    present_days = sum(1 for a in attendance if a["status"] == "present")
    att_pct = (present_days/total_days*100) if total_days > 0 else 0

    total_score = 0
    score_count = 0
    subject_scores = {}
    for r in results:
        obtained = r.get("marks_obtained")
        max_marks = r.get("max_marks")
        if obtained is not None and max_marks and max_marks > 0:
            pct = (float(obtained)/float(max_marks))*100
            total_score += pct
            score_count += 1
            name = r.get("subjects",{}).get("name","Unknown")
            subject_scores.setdefault(name,[]).append(pct)

    avg_score = (total_score/score_count) if score_count > 0 else 0

    buf = [f"📊 Academic Performance Summary for {profile['full_name']}",
           f"Class: {profile['class']}\n",
           f"🏆 Overall Stats:",
           f"   Average Score: {avg_score:.1f}%",
           f"   Attendance: {att_pct:.1f}%",
           f"   XP Points: {profile.get('xp_points',0)}",
           f"   Current Streak: {profile.get('learning_streak',0)} days",
           f"   Best Streak: {profile.get('best_streak',0)} days\n",
           f"📚 Subject-wise Performance:"]
    for subj, scores in subject_scores.items():
        subj_avg = sum(scores)/len(scores) if scores else 0
        emoji = "✅" if subj_avg>=90 else ("⚠️" if subj_avg>=75 else "🔴")
        buf.append(f"   {emoji} {subj}: {subj_avg:.1f}%")

    return "\n".join(buf)
```

### Tool 8: generate_study_plan
```python
@tool
def generate_study_plan(subject: str = "all", school_id: str = "") -> str:
    """Generate a personalized study plan based on upcoming exams and weak areas.
    Input: subject name or "all" for all subjects.
    Use when student asks for help preparing for exams."""
    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles").select("class").eq("id", user_id).single().execute().data
    student_class = profile["class"]

    from datetime import date
    exams = sb.table("exams")\
        .select("*, subjects(name)")\
        .eq("school_id", school_id)\
        .contains("target_classes", f'["{student_class}"]')\
        .gte("exam_date", date.today().isoformat())\
        .order("exam_date").limit(5).execute().data

    results = sb.table("results")\
        .select("*, subjects(name)")\
        .eq("school_id", school_id)\
        .eq("student_id", user_id)\
        .order("created_at", ascending=False).limit(20).execute().data

    weak_subjects = []
    subject_scores = {}
    for r in results:
        obtained = r.get("marks_obtained")
        max_marks = r.get("max_marks")
        if obtained and max_marks and max_marks > 0:
            pct = (float(obtained)/float(max_marks))*100
            name = r.get("subjects",{}).get("name","")
            subject_scores.setdefault(name,[]).append(pct)
    for name, scores in subject_scores.items():
        avg = sum(scores)/len(scores)
        if avg < 75:
            weak_subjects.append((name, avg))

    buf = ["📋 Personalized Study Plan\n"]
    if exams:
        buf.append("📝 Upcoming Exams:")
        for exam in exams:
            from datetime import datetime
            exam_date = datetime.fromisoformat(exam["exam_date"]).date()
            days_left = (exam_date - date.today()).days
            buf.append(f"   📅 {exam['title']} - {exam['exam_date']} ({days_left} days left)")
        buf.append("")

    if weak_subjects:
        buf.append("⚠️ Areas Needing Focus:")
        for name, avg in weak_subjects:
            buf.append(f"   🔴 {name} (Average: {avg:.1f}%)")
        buf.append("")

    buf.extend([
        "💡 Recommended Study Schedule:",
        "   Morning (6-8 AM): Revise weak topics",
        "   Afternoon (2-4 PM): Practice problems",
        "   Evening (7-9 PM): Review notes and formulas\n",
        "🎯 Tips:",
        "   • Study in 25-min blocks with 5-min breaks (Pomodoro)",
        "   • Focus on understanding concepts, not memorizing",
        "   • Practice previous year questions",
        "   • Take a full mock test every weekend"
    ])
    return "\n".join(buf)
```

### Tool 9: explain_concept
```python
@tool
def explain_concept(concept: str) -> str:
    """Explain an academic concept in simple terms with examples.
    Input: the concept to explain.
    Use when student asks to explain something like integration, Newton laws, etc."""
    return f"Please explain the following academic concept in simple, easy-to-understand terms with examples: {concept}"
```

### Tool 10: get_notifications
```python
@tool
def get_notifications(school_id: str = "") -> str:
    """Get recent notifications and alerts for the student.
    Use when student asks about notifications, alerts, or what is new."""
    sb = get_supabase()
    user_id = get_current_user_id()

    notifications = sb.table("notifications")\
        .select("*").eq("user_id", user_id)\
        .order("created_at", ascending=False)\
        .limit(10).execute().data

    if not notifications:
        return "No new notifications. You're all caught up! ✅"

    buf = ["🔔 Recent Notifications:\n"]
    unread = 0
    for n in notifications:
        is_read = n.get("is_read", False)
        if not is_read:
            unread += 1
        prefix = "  " if is_read else "🔴"
        buf.append(f"{prefix} {n['title']}")
        buf.append(f"   {n.get('body','')}")
        buf.append(f"   {n.get('created_at','')}\n")

    if unread > 0:
        buf.append(f"📬 You have {unread} unread notification(s).")
    return "\n".join(buf)
```

### Tool 11: get_library_status
```python
@tool
def get_library_status(school_id: str = "") -> str:
    """Get library borrow status, currently borrowed books, and due dates.
    Use when student asks about library books, borrowed books."""
    sb = get_supabase()
    user_id = get_current_user_id()

    borrows = sb.table("library_borrows")\
        .select("*, library_books(title, author, isbn)")\
        .eq("school_id", school_id)\
        .eq("student_id", user_id)\
        .eq("status", "borrowed")\
        .order("due_at").execute().data

    if not borrows:
        return "You have no books currently borrowed from the library. 📚"

    buf = ["📚 Currently Borrowed Books:\n"]
    from datetime import datetime
    for b in borrows:
        book = b.get("library_books",{})
        due = b.get("due_at","TBD")
        due_dt = datetime.fromisoformat(due) if due != "TBD" else None
        is_overdue = due_dt and due_dt < datetime.now()

        buf.append(f"📖 {book.get('title','Unknown')} by {book.get('author','Unknown')}")
        buf.append(f"   Due: {due} {'⚠️ OVERDUE' if is_overdue else '✅'}")
        buf.append(f"   Renewals: {b.get('renewals_used',0)}/{b.get('max_renewals',2)} used")
        if b.get("fine_amount",0) > 0:
            buf.append(f"   Fine: ₹{b['fine_amount']}")
        buf.append("")

    return "\n".join(buf)
```

### Tool 12: get_achievements
```python
@tool
def get_achievements(school_id: str = "") -> str:
    """Get achievements, badges, XP points, and rankings.
    Use when student asks about their badges, XP, achievements."""
    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles")\
        .select("xp_points, learning_streak, best_streak")\
        .eq("id", user_id).single().execute().data

    achievements = sb.table("student_achievements")\
        .select("*, achievements(name, description, icon, xp_reward, rarity)")\
        .eq("school_id", school_id)\
        .eq("student_id", user_id)\
        .order("earned_at", ascending=False).execute().data

    buf = [f"🏆 Achievements Summary\n",
           f"⭐ Total XP: {profile.get('xp_points',0)}",
           f"🔥 Current Streak: {profile.get('learning_streak',0)} days",
           f"🏅 Best Streak: {profile.get('best_streak',0)} days\n",
           f"🎖️ Badges Earned ({len(achievements)}):\n"]

    for a in achievements:
        ach = a.get("achievements",{})
        buf.append(f"  {ach.get('icon','🏅')} {ach.get('name','Unknown')} (+{ach.get('xp_reward',0)} XP) [{ach.get('rarity','common')}]")
        if ach.get("description"):
            buf.append(f"     {ach['description']}")

    return "\n".join(buf)
```

### Tool 13: get_events
```python
@tool
def get_events(school_id: str = "") -> str:
    """Get upcoming school events, competitions, and activities.
    Use when student asks about events, competitions."""
    sb = get_supabase()
    from datetime import date
    events = sb.table("events")\
        .select("*")\
        .eq("school_id", school_id)\
        .gte("event_date", date.today().isoformat())\
        .order("event_date").limit(10).execute().data

    if not events:
        return "No upcoming events at the moment. Stay tuned! 📅"

    buf = ["📅 Upcoming Events:\n"]
    for e in events:
        buf.append(f"🎉 {e['title']}")
        buf.append(f"   📅 {e.get('event_date','TBD')} at {e.get('event_time','TBD')}")
        buf.append(f"   📍 {e.get('venue','TBD')}")
        if e.get("description"):
            buf.append(f"   📝 {e['description']}")
        max_p = e.get("max_participants")
        if max_p:
            buf.append(f"   👥 {e.get('current_participants',0)}/{max_p} registered")
        buf.append("")

    return "\n".join(buf)
```

### Tool 14: get_leaderboard
```python
@tool
def get_leaderboard(school_id: str = "") -> str:
    """Get class or school leaderboard rankings.
    Use when student asks about rankings, who is top of the class."""
    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles")\
        .select("class, xp_points")\
        .eq("id", user_id).single().execute().data

    student_class = profile["class"]

    leaderboard = sb.table("profiles")\
        .select("id, full_name, xp_points, learning_streak")\
        .eq("school_id", school_id)\
        .eq("class", student_class)\
        .eq("role", "student")\
        .order("xp_points", ascending=False)\
        .limit(10).execute().data

    buf = [f"🏆 Class {student_class} Leaderboard:\n"]
    medals = {1:'🥇',2:'🥈',3:'🥉'}
    for rank, student in enumerate(leaderboard, 1):
        is_me = student["id"] == user_id
        prefix = medals.get(rank, f"{rank}.")
        name = f"{student['full_name']} (You)" if is_me else student["full_name"]
        buf.append(f"  {prefix} {name} - {student.get('xp_points',0)} XP (🔥 {student.get('learning_streak',0)} days)")

    return "\n".join(buf)
```

### Tool 15: submit_homework
```python
@tool
def submit_homework(homework_title: str, school_id: str = "") -> str:
    """Submit a homework assignment. Input: homework title or ID.
    Use when student says they want to submit homework."""
    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles").select("class").eq("id", user_id).single().execute().data
    student_class = profile["class"]

    homework = sb.table("homework")\
        .select("*, subjects(name)")\
        .eq("school_id", school_id)\
        .eq("target_class", student_class)\
        .ilike("title", f"%{homework_title}%")\
        .limit(1).maybe_single().execute()

    if not homework or not homework.data:
        return f'Could not find homework matching "{homework_title}". Check the title.'

    existing = sb.table("homework_submissions")\
        .select("id")\
        .eq("homework_id", homework.data["id"])\
        .eq("student_id", user_id)\
        .maybe_single().execute()

    if existing and existing.data:
        return "You have already submitted this homework. ✅"

    sb.table("homework_submissions").insert({
        "homework_id": homework.data["id"],
        "student_id": user_id,
        "school_id": school_id,
        "submission_text": "Submitted via Shami AI Assistant",
        "status": "submitted",
    }).execute()

    subj_name = homework.data.get("subjects",{}).get("name","Unknown")
    return f'✅ Homework "{homework.data["title"]}" ({subj_name}) submitted successfully! Your teacher will review it shortly.'
```

### Tool 16: apply_leave
```python
@tool
def apply_leave(leave_reason: str, leave_type: str = "sick", school_id: str = "") -> str:
    """Apply for leave. Input: reason and optionally leave_type (sick/casual/urgent/family_event).
    Use when student says they need leave, sick leave."""
    sb = get_supabase()
    user_id = get_current_user_id()

    from datetime import date, timedelta
    tomorrow = date.today() + timedelta(days=1)
    day_after = date.today() + timedelta(days=2)

    sb.table("leave_applications").insert({
        "applicant_id": user_id,
        "applicant_role": "student",
        "school_id": school_id,
        "leave_type": leave_type,
        "start_date": tomorrow.isoformat(),
        "end_date": day_after.isoformat(),
        "reason": leave_reason,
        "status": "pending",
    }).execute()

    return (f"✅ Leave application submitted!\n"
            f"Type: {leave_type.upper()}\n"
            f"From: {tomorrow.isoformat()}\n"
            f"To: {day_after.isoformat()}\n"
            f"Status: PENDING\n"
            f"Your class teacher will review and approve it.")
```

### Tool 17: get_notices
```python
@tool
def get_notices(school_id: str = "") -> str:
    """Get recent school notices and announcements.
    Use when student asks about notices, announcements, circulars."""
    sb = get_supabase()
    notices = sb.table("notices")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("status", "published")\
        .order("published_at", ascending=False)\
        .limit(10).execute().data

    if not notices:
        return "No notices at the moment. 📢"

    buf = ["📢 Recent Notices:\n"]
    for n in notices:
        is_urgent = n.get("is_urgent", False)
        prefix = "🚨" if is_urgent else "📋"
        buf.append(f"{prefix} {n['title']} [{n.get('category','General')}]")
        buf.append(f"   {n.get('content','')[:200]}")
        buf.append(f"   📅 {n.get('published_at','')}\n")

    return "\n".join(buf)
```

### Tool 18: answer_general
```python
@tool
def answer_general(question: str) -> str:
    """Answer general questions about the school, timings, policies.
    Use when student asks general questions that don't require database queries."""
    return f"Answer the following general school question: {question}"
```

### Tool 19: generate_practice
```python
@tool
def generate_practice(subject_and_topic: str) -> str:
    """Generate practice questions for a given subject and topic.
    Input: subject name and topic.
    Use when student wants practice problems, worksheets."""
    return (f"Generate 5 practice questions with solutions for: {subject_and_topic}. "
            f"Include a mix of easy, medium, and hard difficulty levels. "
            f"Provide step-by-step solutions for each question.")
```

### Tool 20: get_live_class
```python
@tool
def get_live_class(school_id: str = "") -> str:
    """Get live class status, upcoming classes, and recorded sessions.
    Use when student asks about live classes, online classes, recordings."""
    sb = get_supabase()
    user_id = get_current_user_id()

    profile = sb.table("profiles").select("class").eq("id", user_id).single().execute().data
    student_class = profile["class"]

    live_classes = sb.table("live_classes")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .eq("target_class", student_class)\
        .order("scheduled_at", ascending=False)\
        .limit(10).execute().data

    if not live_classes:
        return "No live classes scheduled at the moment. 🎥"

    buf = ["🎥 Live Classes:\n"]
    for lc in live_classes:
        subj = lc.get("subjects",{})
        is_live = lc.get("is_live", False)
        status = lc.get("status","scheduled")

        if is_live:
            status_str = f"🔴 LIVE NOW ({lc.get('viewer_count',0)} watching)"
        elif status == "completed":
            status_str = "✅ Completed"
        elif status == "recorded":
            status_str = "📹 Recorded"
        else:
            status_str = f"📅 Scheduled: {lc.get('scheduled_at','TBD')}"

        buf.append(f"{subj.get('icon','🎥')} {lc.get('title','Untitled')}")
        buf.append(f"   {status_str}")
        buf.append(f"   Subject: {subj.get('name','Unknown')}\n")

    return "\n".join(buf)
```

### getStudentTools() Assembly

```python
# app/tools/student_tools.py
def get_student_tools(school_id: str) -> list:
    return [
        get_timetable,
        get_homework,
        get_fee_status,
        get_attendance,
        get_exam_info,
        get_bus_location,
        get_performance,
        generate_study_plan,
        explain_concept,
        get_notifications,
        get_library_status,
        get_achievements,
        get_events,
        get_leaderboard,
        submit_homework,
        apply_leave,
        get_notices,
        answer_general,
        generate_practice,
        get_live_class,
    ]
```

---

## 8. TEACHER AI TOOLS (20 TOOLS)

### Tool 1: get_class_students
```python
@tool
def get_class_students(class_name: str, school_id: str = "") -> str:
    """List all students in a specific class. Input: class name like "X-A".
    Use when teacher asks about students in a class."""
    sb = get_supabase()
    students = sb.table("profiles")\
        .select("id, full_name, roll_number, xp_points, learning_streak")\
        .eq("school_id", school_id)\
        .eq("class", class_name)\
        .eq("role", "student")\
        .order("roll_number").execute().data

    if not students:
        return f"No students found in class {class_name}."

    buf = [f"👥 Students in Class {class_name} ({len(students)} total):\n"]
    for s in students:
        buf.append(f"  {s.get('roll_number','?')}. {s['full_name']} (XP: {s.get('xp_points',0)}, Streak: {s.get('learning_streak',0)} days)")
    return "\n".join(buf)
```

### Tool 2: get_class_performance
```python
@tool
def get_class_performance(class_name: str, school_id: str = "") -> str:
    """Get class performance statistics including average score, attendance, top performers.
    Input: class name. Use when teacher asks how a class is performing."""
    sb = get_supabase()
    students = sb.table("profiles")\
        .select("id, full_name, xp_points")\
        .eq("school_id", school_id)\
        .eq("class", class_name)\
        .eq("role", "student").execute().data

    if not students:
        return f"No students found in class {class_name}."

    student_ids = [s["id"] for s in students]

    results = sb.table("results")\
        .select("student_id, marks_obtained, max_marks")\
        .in_("student_id", student_ids).execute().data

    attendance = sb.table("attendance")\
        .select("student_id, status")\
        .in_("student_id", student_ids).execute().data

    total_pct = sum(
        (float(r["marks_obtained"])/float(r["max_marks"])*100)
        for r in results if r.get("marks_obtained") and r.get("max_marks") and float(r["max_marks"]) > 0
    )
    avg_score = (total_pct / len(results)) if results else 0

    total_att = len(attendance)
    present_att = sum(1 for a in attendance if a["status"] == "present")
    att_pct = (present_att / total_att * 100) if total_att > 0 else 0

    at_risk = []
    student_scores = {}
    for r in results:
        sid = r["student_id"]
        if r.get("marks_obtained") and r.get("max_marks") and float(r["max_marks"]) > 0:
            pct = float(r["marks_obtained"])/float(r["max_marks"])*100
            student_scores.setdefault(sid, []).append(pct)
    for sid, scores in student_scores.items():
        avg = sum(scores)/len(scores)
        if avg < 50:
            name = next((s["full_name"] for s in students if s["id"] == sid), sid)
            at_risk.append(name)

    buf = [f"📊 Class {class_name} Performance:",
           f"Total Students: {len(students)}",
           f"Average Score: {avg_score:.1f}%",
           f"Average Attendance: {att_pct:.1f}%\n"]
    if at_risk:
        buf.append("⚠️ At-Risk Students (below 50%):")
        for name in at_risk:
            buf.append(f"  🔴 {name}")

    return "\n".join(buf)
```

### Tool 3: get_at_risk_students
```python
@tool
def get_at_risk_students(class_name: str, school_id: str = "") -> str:
    """Identify students who are struggling academically or have low attendance.
    Input: class name. Use when teacher asks who needs help."""
    sb = get_supabase()
    students = sb.table("profiles")\
        .select("id, full_name, roll_number")\
        .eq("school_id", school_id)\
        .eq("class", class_name)\
        .eq("role", "student").execute().data

    if not students:
        return f"No students found in class {class_name}."

    student_ids = [s["id"] for s in students]

    results = sb.table("results")\
        .select("student_id, marks_obtained, max_marks")\
        .in_("student_id", student_ids).execute().data

    attendance = sb.table("attendance")\
        .select("student_id, status")\
        .in_("student_id", student_ids).execute().data

    at_risk = []
    for student in students:
        sid = student["id"]
        s_results = [r for r in results if r["student_id"] == sid]
        s_att = [a for a in attendance if a["student_id"] == sid]

        avg_score = 0
        if s_results:
            scores = [
                float(r["marks_obtained"])/float(r["max_marks"])*100
                for r in s_results if r.get("marks_obtained") and r.get("max_marks") and float(r["max_marks"]) > 0
            ]
            avg_score = sum(scores)/len(scores) if scores else 0

        total_days = len(s_att)
        present_days = sum(1 for a in s_att if a["status"] == "present")
        att_pct = (present_days/total_days*100) if total_days > 0 else 100

        if avg_score < 50 or att_pct < 75:
            reason = "Low Score" if avg_score < 50 else "Low Attendance"
            at_risk.append({
                "name": student["full_name"],
                "roll": student.get("roll_number"),
                "avg_score": avg_score,
                "attendance": att_pct,
                "reason": reason,
            })

    if not at_risk:
        return f"✅ No at-risk students found in class {class_name}. All students are performing well!"

    buf = [f"⚠️ At-Risk Students in Class {class_name}:\n"]
    for s in at_risk:
        buf.append(f"🔴 {s['name']} (Roll {s['roll']})")
        buf.append(f"   Score: {s['avg_score']:.1f}% | Attendance: {s['attendance']:.1f}%")
        buf.append(f"   Issue: {s['reason']}\n")
    buf.append("💡 Consider scheduling remedial sessions for these students.")

    return "\n".join(buf)
```

### Tool 4: generate_questions
```python
@tool
def generate_questions(requirements: str) -> str:
    """Generate exam questions for a given subject, topic, difficulty, and marks distribution.
    Input: subject, topic, number of questions, difficulty level.
    Use when teacher needs to create questions."""
    return (f"Generate exam questions based on: {requirements}. "
            f"Include a mix of MCQ and subjective questions. "
            f"For MCQs, provide 4 options with the correct answer marked. "
            f"For subjective questions, provide expected key points in the answer.")
```

### Tool 5: generate_lesson_plan
```python
@tool
def generate_lesson_plan(topic_details: str) -> str:
    """Create a detailed lesson plan for a given topic.
    Input: subject, topic, duration, and class level.
    Use when teacher needs a lesson plan."""
    return (f"Create a detailed lesson plan for: {topic_details}. "
            f"Include: Learning Objectives, Duration breakdown, Teaching Methods, "
            f"Materials needed, Activities, Assessment methods, and Homework. "
            f"Follow NCERT/CBSE guidelines.")
```

### Tool 6: auto_grade_homework
```python
@tool
def auto_grade_homework(homework_title: str, school_id: str = "") -> str:
    """Auto-grade homework submissions using AI. Can grade objective/numerical questions.
    Input: homework title or ID. Use when teacher wants to auto-grade pending submissions."""
    sb = get_supabase()
    user_id = get_current_user_id()

    homework = sb.table("homework")\
        .select("*, subjects(name)")\
        .eq("school_id", school_id)\
        .ilike("title", f"%{homework_title}%")\
        .eq("teacher_id", user_id)\
        .limit(1).maybe_single().execute()

    if not homework or not homework.data:
        return f'Could not find homework matching "{homework_title}".'

    submissions = sb.table("homework_submissions")\
        .select("*, profiles(full_name)")\
        .eq("homework_id", homework.data["id"])\
        .eq("status", "submitted").execute().data

    if not submissions:
        return f'No pending submissions for "{homework.data["title"]}".'

    max_marks = float(homework.data.get("max_marks", 25))
    graded = 0
    for sub in submissions:
        text = sub.get("submission_text", "")
        if len(text) > 500:
            marks, grade = max_marks, "A+"
        elif len(text) > 200:
            marks, grade = max_marks * 0.8, "B+"
        elif text:
            marks, grade = max_marks * 0.6, "C"
        else:
            marks, grade = 0, "F"

        sb.table("homework_submissions").update({
            "status": "graded",
            "marks": marks,
            "grade": grade,
            "teacher_remarks": "Auto-graded by Shami AI",
            "graded_by": user_id,
            "graded_at": "now()",
        }).eq("id", sub["id"]).execute()
        graded += 1

    return f'✅ Auto-grading complete! Graded {graded} submission(s) for "{homework.data["title"]}".'
```

### Tool 7: get_submission_status
```python
@tool
def get_submission_status(school_id: str = "") -> str:
    """Check homework submission statistics - submitted, pending, graded.
    Use when teacher asks about submission status."""
    sb = get_supabase()
    user_id = get_current_user_id()

    homeworks = sb.table("homework")\
        .select("*, subjects(name)")\
        .eq("school_id", school_id)\
        .eq("teacher_id", user_id)\
        .eq("status", "active").execute().data

    if not homeworks:
        return "No active homework assignments."

    buf = ["📋 Homework Submission Status:\n"]
    for hw in homeworks:
        submissions = sb.table("homework_submissions")\
            .select("status").eq("homework_id", hw["id"]).execute().data
        total = len(submissions)
        graded = sum(1 for s in submissions if s["status"] == "graded")
        buf.append(f"📝 {hw['title']} ({hw.get('subjects',{}).get('name','')})")
        buf.append(f"   Submitted: {total} | Graded: {graded} | Pending: {total-graded}\n")

    return "\n".join(buf)
```

### Tool 8: generate_remedial_plan
```python
@tool
def generate_remedial_plan(details: str) -> str:
    """Create a remedial teaching plan for struggling students.
    Input: class name and weak topics. Use when teacher wants to help weak students."""
    return (f"Create a remedial teaching plan for: {details}. "
            f"Include: Target students criteria, Topics to cover, "
            f"Session schedule (2-3 extra classes per week), "
            f"Teaching approach (simplified explanations, more examples), "
            f"Practice worksheets, and Progress tracking method.")
```

### Tool 9: create_notice
```python
@tool
def create_notice(notice_topic: str) -> str:
    """Draft and create a school notice or circular.
    Input: notice topic and content details. Use when teacher wants to create a notice."""
    return (f"Draft a professional school notice based on: {notice_topic}. "
            f"Format with: Title, Date, Target Audience, Content (clear and concise), "
            f"and action items. Use formal language appropriate for school communication.")
```

### Tool 10: get_attendance_stats
```python
@tool
def get_attendance_stats(class_name: str, school_id: str = "") -> str:
    """Get attendance analytics and trends for a class.
    Input: class name. Use when teacher asks about attendance trends."""
    sb = get_supabase()
    students = sb.table("profiles")\
        .select("id, full_name")\
        .eq("school_id", school_id)\
        .eq("class", class_name)\
        .eq("role", "student").execute().data

    if not students:
        return f"No students found in class {class_name}."

    student_ids = [s["id"] for s in students]

    attendance = sb.table("attendance")\
        .select("student_id, status, date")\
        .in_("student_id", student_ids)\
        .order("date", ascending=False).execute().data

    student_att = {}
    for a in attendance:
        sid = a["student_id"]
        student_att.setdefault(sid, {"total":0,"present":0})
        student_att[sid]["total"] += 1
        if a["status"] == "present":
            student_att[sid]["present"] += 1

    class_avg = 0
    low_att = []
    for sid, data in student_att.items():
        pct = (data["present"]/data["total"]*100) if data["total"] > 0 else 0
        class_avg += pct
        if pct < 75:
            name = next((s["full_name"] for s in students if s["id"] == sid), sid)
            low_att.append(f"{name} ({pct:.0f}%)")

    class_avg = class_avg / len(student_att) if student_att else 0

    buf = [f"📊 Attendance Statistics for Class {class_name}:",
           f"Total Records: {len(attendance)}",
           f"Class Average Attendance: {class_avg:.1f}%\n"]
    if low_att:
        buf.append("⚠️ Students Below 75%:")
        for name in low_att:
            buf.append(f"  🔴 {name}")

    return "\n".join(buf)
```

### Tool 11: get_teacher_schedule
```python
@tool
def get_teacher_schedule(school_id: str = "") -> str:
    """Get the teacher's daily schedule and timetable.
    Use when teacher asks about their schedule, what classes they have today."""
    sb = get_supabase()
    user_id = get_current_user_id()

    from datetime import datetime
    today = datetime.now().weekday()  # 0=Mon

    schedule = sb.table("timetable")\
        .select("*, subjects(name, icon)")\
        .eq("school_id", school_id)\
        .eq("teacher_id", user_id)\
        .eq("day_of_week", today)\
        .order("start_time").execute().data

    if not schedule:
        return "No classes scheduled for today. Enjoy your free day! 🎉"

    days = ['Monday','Tuesday','Wednesday','Thursday','Friday','Saturday']
    buf = [f"📅 Your Schedule for {days[today]}:\n"]
    for s in schedule:
        if s.get("is_break"):
            buf.append(f"☕ {s.get('break_name','Break')} ({s['start_time']} - {s['end_time']})")
        elif s.get("is_free_period"):
            buf.append(f"🆓 Free Period ({s['start_time']} - {s['end_time']})")
        else:
            subj = s.get("subjects",{})
            buf.append(f"{subj.get('icon','📚')} {subj.get('name','Unknown')} - Class {s.get('class','')}")
            buf.append(f"   ⏰ {s['start_time']} - {s['end_time']} | Room {s.get('room','TBD')}")

    return "\n".join(buf)
```

### Tool 12: get_pending_tasks
```python
@tool
def get_pending_tasks(school_id: str = "") -> str:
    """Get pending tasks that need teacher attention - ungraded homework, pending leaves.
    Use when teacher asks what needs their attention."""
    sb = get_supabase()
    user_id = get_current_user_id()

    buf = ["📋 Pending Tasks:\n"]

    homeworks = sb.table("homework")\
        .select("*, subjects(name)")\
        .eq("school_id", school_id)\
        .eq("teacher_id", user_id)\
        .eq("status", "active").execute().data

    for hw in homeworks:
        submissions = sb.table("homework_submissions")\
            .select("status")\
            .eq("homework_id", hw["id"])\
            .eq("status", "submitted").execute().data
        if submissions:
            buf.append(f"📝 {hw['title']} - {len(submissions)} submissions to grade")

    leaves = sb.table("leave_applications")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("status", "pending")\
        .eq("applicant_role", "student").execute().data

    if leaves:
        buf.append(f"🏖️ {len(leaves)} leave application(s) pending approval")

    return "\n".join(buf)
```

### Tool 13: generate_report
```python
@tool
def generate_report(class_name: str) -> str:
    """Generate a class performance report with analytics.
    Input: class name. Use when teacher wants a performance report."""
    return (f"Generate a comprehensive performance report for Class {class_name}. "
            f"Include: Overall class average, Subject-wise performance, "
            f"Top 5 performers, At-risk students, Attendance summary, "
            f"Comparison with previous term, and Recommendations for improvement.")
```

### Tool 14: explain_pedagogy
```python
@tool
def explain_pedagogy(topic_and_challenge: str) -> str:
    """Suggest teaching strategies and methods for a given topic.
    Input: topic and any specific challenges.
    Use when teacher asks how to teach something."""
    return (f"Suggest effective teaching strategies for: {topic_and_challenge}. "
            f"Include: Multiple teaching approaches (visual, auditory, kinesthetic), "
            f"Real-world examples, Common student misconceptions, "
            f"Assessment methods, and Differentiation strategies.")
```

### Tool 15: get_leave_balance
```python
@tool
def get_leave_balance(school_id: str = "") -> str:
    """Get teacher leave balance and history.
    Use when teacher asks about their leave status."""
    sb = get_supabase()
    user_id = get_current_user_id()

    leaves = sb.table("leave_applications")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("applicant_id", user_id)\
        .order("created_at", ascending=False).execute().data

    total_quota = 12
    used = sum(1 for l in leaves if l["status"] == "approved")
    pending = sum(1 for l in leaves if l["status"] == "pending")
    remaining = total_quota - used

    buf = [f"🏖️ Leave Summary:",
           f"Total Quota: {total_quota} days",
           f"Used: {used} days",
           f"Pending: {pending} applications",
           f"Remaining: {remaining} days\n",
           f"📋 Recent Applications:"]
    for l in leaves[:5]:
        status = l["status"]
        emoji = {"approved":"✅","pending":"⏳","rejected":"❌"}.get(status,"📋")
        buf.append(f"  {emoji} {l.get('leave_type','')} - {l.get('start_date','')} to {l.get('end_date','')} [{status}]")

    return "\n".join(buf)
```

### Tool 16: get_salary_info
```python
@tool
def get_salary_info(school_id: str = "") -> str:
    """Get teacher salary details and breakup.
    Use when teacher asks about their salary."""
    sb = get_supabase()
    user_id = get_current_user_id()

    salary = sb.table("salary")\
        .select("*")\
        .eq("school_id", school_id)\
        .eq("teacher_id", user_id)\
        .order("created_at", ascending=False)\
        .limit(1).maybe_single().execute()

    if not salary or not salary.data:
        return "No salary records found."

    s = salary.data
    buf = [f"💰 Salary Details - {s.get('month','')}:\n",
           f"Gross Salary: ₹{s.get('gross_salary',0):,.0f}",
           f"  Basic Pay: ₹{s.get('basic_pay',0):,.0f}",
           f"  HRA: ₹{s.get('hra',0):,.0f}",
           f"  DA: ₹{s.get('da',0):,.0f}",
           f"  Special Allowance: ₹{s.get('special_allowance',0):,.0f}\n",
           f"Deductions:",
           f"  PF: -₹{s.get('pf_deduction',0):,.0f}",
           f"  TDS: -₹{s.get('tds_deduction',0):,.0f}",
           f"  Professional Tax: -₹{s.get('professional_tax',0):,.0f}\n",
           f"💵 Net Salary: ₹{s.get('net_salary',0):,.0f}",
           f"Status: {s.get('status','')}"]

    return "\n".join(buf)
```

### Tool 17: get_parent_communications
```python
@tool
def get_parent_communications(school_id: str = "") -> str:
    """Get recent parent messages and communications.
    Use when teacher asks about parent messages."""
    sb = get_supabase()
    user_id = get_current_user_id()

    messages = sb.table("messages")\
        .select("*, profiles(full_name, role)")\
        .eq("school_id", school_id)\
        .eq("receiver_id", user_id)\
        .order("created_at", ascending=False)\
        .limit(10).execute().data

    if not messages:
        return "No new parent messages."

    buf = ["💬 Recent Messages:\n"]
    for m in messages:
        sender = m.get("profiles",{})
        is_parent = sender.get("role") == "parent"
        prefix = "👨‍👩‍👦" if is_parent else "👤"
        buf.append(f"{prefix} {sender.get('full_name','Unknown')}:")
        buf.append(f"   {m.get('content','')}")
        buf.append(f"   {m.get('created_at','')}\n")

    return "\n".join(buf)
```

### Tool 18: upload_material
```python
@tool
def upload_material(material_details: str) -> str:
    """Upload study material for a class.
    Input: material title, type, and target class.
    Use when teacher wants to upload notes or materials."""
    return (f"To upload study material, please use the Materials screen in the app. "
            f"You can upload Notes, PPTs, Videos, or Worksheets for any class. "
            f"Requested: {material_details}")
```

### Tool 19: get_exam_analytics
```python
@tool
def get_exam_analytics(school_id: str = "") -> str:
    """Get exam statistics including pass rates, score distribution, and trends.
    Input: class name or exam title. Use when teacher asks about exam results."""
    sb = get_supabase()
    user_id = get_current_user_id()

    exams = sb.table("exams")\
        .select("*, subjects(name)")\
        .eq("school_id", school_id)\
        .eq("teacher_id", user_id)\
        .order("exam_date", ascending=False)\
        .limit(5).execute().data

    if not exams:
        return "No exams found."

    buf = ["📊 Exam Analytics:\n"]
    for exam in exams:
        sessions = sb.table("exam_sessions")\
            .select("total_marks, status")\
            .eq("exam_id", exam["id"]).execute().data

        if sessions:
            scores = [float(s.get("total_marks",0)) for s in sessions]
            avg = sum(scores)/len(scores) if scores else 0
            pass_count = sum(1 for s in scores if s >= 40)
            pass_rate = (pass_count/len(scores)*100) if scores else 0

            buf.append(f"📝 {exam['title']} ({exam.get('subjects',{}).get('name','')})")
            buf.append(f"   Average: {avg:.1f}% | Pass Rate: {pass_rate:.0f}% | Total Students: {len(sessions)}\n")

    return "\n".join(buf)
```

### Tool 20: answer_general (Teacher)
```python
@tool
def teacher_answer_general(question: str) -> str:
    """Answer general questions about school policies, timings, etc.
    Use when teacher asks general questions."""
    return f"Answer the following general school question: {question}"
```

### getTeacherTools() Assembly

```python
# app/tools/teacher_tools.py
def get_teacher_tools(school_id: str) -> list:
    return [
        get_class_students,
        get_class_performance,
        get_at_risk_students,
        generate_questions,
        generate_lesson_plan,
        auto_grade_homework,
        get_submission_status,
        generate_remedial_plan,
        create_notice,
        get_attendance_stats,
        get_teacher_schedule,
        get_pending_tasks,
        generate_report,
        explain_pedagogy,
        get_leave_balance,
        get_salary_info,
        get_parent_communications,
        upload_material,
        get_exam_analytics,
        teacher_answer_general,
    ]
```

### Tools __init__.py

```python
# app/tools/__init__.py
from app.tools.student_tools import get_student_tools
from app.tools.teacher_tools import get_teacher_tools
from app.tools.fee_tools import get_fee_tools
from app.tools.academic_tools import get_academic_tools
from app.tools.whatsapp_tools import get_whatsapp_tools
from app.tools.rag_tools import get_rag_tools
from app.tools.iot_tools import get_iot_tools

def get_all_tools(school_id: str) -> list:
    return [
        *get_student_tools(school_id),
        *get_teacher_tools(school_id),
        *get_fee_tools(school_id),
        *get_academic_tools(school_id),
        *get_whatsapp_tools(school_id),
        *get_rag_tools(school_id),
        *get_iot_tools(school_id),
    ]

def filter_tools_by_role(tools: list, role: str) -> list:
    from app.agents.router import ROLE_TOOLS
    allowed = ROLE_TOOLS.get(role, [])
    if "*" in allowed:
        return tools
    return [t for t in tools if t.name in allowed]
```

---

## 9. SYSTEM PROMPTS

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
}

def get_system_prompt(role: str, school_id: str) -> str:
    context = ROLE_CONTEXT.get(role, "You are a helpful school assistant.")
    return BASE_PROMPT.format(school_id=school_id) + "\n\n" + context
```

---

## 10. RAG PIPELINE

### Supabase pgvector Setup

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE knowledge_base (
  id         uuid    DEFAULT gen_random_uuid() PRIMARY KEY,
  school_id  uuid    REFERENCES schools(id),
  subject    text,
  grade      text,
  source     text,
  content    text,
  embedding  vector(1536),
  metadata   jsonb,
  created_at timestamptz DEFAULT now()
);

CREATE INDEX ON knowledge_base
  USING ivfflat (embedding vector_cosine_ops)
  WITH (lists = 100);
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

## 11. IoT DEVICE CONTROL (ESP32 + MQTT)

### Hardware
- ESP32 WROOM-32E + 4CH relay board (₹500-700 per room)
- 4 relays × 10A / 250V AC each, active LOW logic
- Channel mapping: CH1=Fan, CH2=Light1, CH3=Light2, CH4=Projector

### ESP32 Firmware
See `esp32/relay_controller/relay_controller.ino` — handles WiFi, MQTT, HTTP fallback, NVS state persistence.

### Python IoT Controller

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
        self._mqtt.loop_start()

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

iot = IoTController()
```

### IoT Tools

```python
# app/tools/iot_tools.py
from langchain_core.tools import tool
from app.services.iot_controller import iot

@tool
def control_classroom_device(room: str, device: str, action: str) -> str:
    """Control an electrical device in a classroom over WiFi.
    room: classroom ID (e.g. 'class_10a', 'staff_room')
    device: 'fan' | 'light1' | 'light2' | 'projector' | 'ac' | 'all' | 'all_lights'
    action: 'on' | 'off' | 'toggle'"""
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
    """Schedule a device to turn on/off at a specific time.
    time_str: '14:30' (24h format) or '2 minutes from now'"""
    get_supabase().table("iot_scheduled_actions").insert({
        "room": room, "device": device,
        "action": action, "scheduled_time": parse_time(time_str),
        "status": "pending"
    }).execute()
    return f"Scheduled: {device} in {room} will {action} at {time_str}"
```

---

## 12. CAMERA ATTENDANCE (COMPUTER VISION)

All face processing runs on a local Python server inside the school. No student face images sent to cloud.

```python
# app/services/camera_attendance.py
import cv2
from deepface import DeepFace
from ultralytics import YOLO
from app.supabase.client import get_supabase

yolo = YOLO("yolov8n.pt")

class ClassroomCamera:
    def __init__(self, rtsp_url: str, class_id: str, school_id: str):
        self.cap      = cv2.VideoCapture(rtsp_url)
        self.class_id = class_id
        self.school_id = school_id
        self.face_db  = f"/faces/{school_id}/{class_id}"

    async def take_attendance(self) -> dict:
        ret, frame = self.cap.read()
        if not ret:
            return {"error": "camera feed unavailable"}

        results     = yolo.predict(frame, classes=[0], verbose=False)
        headcount   = len(results[0].boxes)

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

        from datetime import datetime
        today = datetime.now().strftime("%Y-%m-%d")

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

---

## 13. FASTAPI ENDPOINTS

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

### Endpoint Summary

| Method | Path | Purpose |
|--------|------|---------|
| POST | /chat/message | Text message → streaming SSE response |
| POST | /chat/voice | Audio file → transcribe + agent + response |
| POST | /chat/image | Image file + optional question → vision response |
| GET | /chat/history/{session_id} | Load past messages for a session |
| POST | /iot/control | Control a device: {room, device, action} |
| GET | /iot/status/{room} | Get current device states in a room |
| POST | /iot/schedule | Schedule a future device action |
| GET | /iot/devices | List all registered ESP32 devices |
| POST | /rag/ingest | Upload PDF → chunk → embed → store |
| GET | /rag/documents | List ingested documents for a school |
| DELETE | /rag/documents/{id} | Remove a document from knowledge base |

### SSE Streaming Response Format

```
data: {"type":"text","content":"Rohan's attendance is "}
data: {"type":"text","content":"94% this month."}
data: {"type":"tool_result","tool":"get_attendance_report","data":{...}}
data: {"type":"done"}
```

---

## 14. SUPABASE SCHEMA (WITH SCHOOL_ID)

### Core Tables

```sql
-- Enable extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- SCHOOLS (multi-school support)
CREATE TABLE schools (
  id         uuid  DEFAULT gen_random_uuid() PRIMARY KEY,
  name       text  NOT NULL,
  address    text,
  phone      text,
  logo_url   text,
  created_at timestamptz DEFAULT now()
);

-- PROFILES (extends auth.users, supports both students and teachers)
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  school_id UUID REFERENCES schools(id),
  user_id TEXT NOT NULL,
  full_name TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('student', 'parent', 'teacher')),
  class TEXT,
  department TEXT,
  designation TEXT,
  roll_number INT,
  employee_id TEXT,
  gender TEXT CHECK (gender IN ('Male', 'Female', 'Other')),
  date_of_birth DATE,
  blood_group TEXT,
  email TEXT,
  phone TEXT,
  admission_number TEXT,
  nationality TEXT DEFAULT 'Indian',
  religion TEXT,
  category TEXT CHECK (category IN ('General','OBC','SC','ST','EWS')),
  address TEXT,
  house TEXT,
  avatar_url TEXT,
  father_name TEXT,
  father_occupation TEXT,
  father_phone TEXT,
  mother_name TEXT,
  mother_occupation TEXT,
  mother_phone TEXT,
  local_guardian TEXT,
  qualification TEXT,
  experience_years INT DEFAULT 0,
  rating DECIMAL(2,1) DEFAULT 0.0,
  xp_points INT DEFAULT 0,
  learning_streak INT DEFAULT 0,
  best_streak INT DEFAULT 0,
  last_login DATE DEFAULT CURRENT_DATE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(school_id, user_id)
);
```

### Academic Tables

```sql
-- SUBJECTS
CREATE TABLE subjects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  name TEXT NOT NULL,
  icon TEXT,
  color TEXT,
  teacher_id UUID REFERENCES profiles(id),
  total_chapters INT DEFAULT 0,
  class TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- TIMETABLE
CREATE TABLE timetable (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  day_of_week INT NOT NULL CHECK (day_of_week BETWEEN 0 AND 5),
  start_time TIME NOT NULL,
  end_time TIME NOT NULL,
  room TEXT,
  class TEXT NOT NULL,
  is_break BOOLEAN DEFAULT FALSE,
  break_name TEXT,
  is_free_period BOOLEAN DEFAULT FALSE,
  free_period_activity TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- COURSES
CREATE TABLE courses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  syllabus_coverage DECIMAL(5,2) DEFAULT 0,
  total_chapters INT DEFAULT 0,
  completed_chapters INT DEFAULT 0,
  class TEXT NOT NULL,
  academic_year TEXT,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- RESULTS
CREATE TABLE results (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  exam_category TEXT NOT NULL CHECK (exam_category IN ('Class Test','Lab Test','Assignment','Mid-Term','End-Term','Other')),
  academic_year TEXT NOT NULL,
  marks_obtained DECIMAL(5,2),
  max_marks DECIMAL(5,2),
  grade TEXT,
  remarks TEXT,
  published_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAMS
CREATE TABLE exams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  exam_type TEXT NOT NULL CHECK (exam_type IN ('offline','online','hybrid')),
  exam_category TEXT CHECK (exam_category IN ('Class test','Lab test','Assignment','Mid term','End term','Other')),
  exam_date DATE,
  start_time TIME,
  duration_minutes INT DEFAULT 90,
  total_marks INT DEFAULT 100,
  venue TEXT,
  instructions TEXT,
  status TEXT DEFAULT 'upcoming' CHECK (status IN ('draft','upcoming','ongoing','completed','graded')),
  target_classes JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM QUESTIONS
CREATE TABLE exam_questions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  exam_id UUID REFERENCES exams(id) ON DELETE CASCADE,
  question_number INT NOT NULL,
  section TEXT NOT NULL CHECK (section IN ('objective','subjective')),
  question_text TEXT NOT NULL,
  question_type TEXT NOT NULL CHECK (question_type IN ('mcq','true_false','short_answer','long_answer')),
  options JSONB,
  correct_answer INT,
  positive_marks INT DEFAULT 4,
  negative_marks INT DEFAULT 1,
  max_marks INT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM SUBMISSIONS
CREATE TABLE exam_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  exam_id UUID REFERENCES exams(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  question_id UUID REFERENCES exam_questions(id) ON DELETE CASCADE,
  selected_answer INT,
  subjective_answer TEXT,
  attachment_url TEXT,
  is_correct BOOLEAN,
  marks_awarded DECIMAL(5,2),
  is_reviewed BOOLEAN DEFAULT FALSE,
  reviewed_by UUID REFERENCES profiles(id),
  submitted_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM SESSIONS
CREATE TABLE exam_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  exam_id UUID REFERENCES exams(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  started_at TIMESTAMPTZ,
  submitted_at TIMESTAMPTZ,
  time_remaining_seconds INT,
  total_attempted INT DEFAULT 0,
  total_correct INT DEFAULT 0,
  total_marks DECIMAL(5,2) DEFAULT 0,
  xp_earned INT DEFAULT 0,
  integrity_score DECIMAL(3,2) DEFAULT 1.00,
  screen_switches INT DEFAULT 0,
  status TEXT DEFAULT 'in_progress' CHECK (status IN ('in_progress','submitted','auto_submitted'))
);

-- ATTENDANCE
CREATE TABLE attendance (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  date DATE NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('present','absent','late','excused')),
  marked_at TIMESTAMPTZ DEFAULT NOW(),
  remarks TEXT,
  UNIQUE(student_id, subject_id, date)
);
```

### Finance Tables

```sql
-- FEES
CREATE TABLE fees (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  fee_type TEXT NOT NULL,
  fee_period TEXT,
  amount DECIMAL(10,2) NOT NULL,
  due_date DATE,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','partial','paid','overdue')),
  amount_paid DECIMAL(10,2) DEFAULT 0,
  academic_year TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- PAYMENTS
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  fee_id UUID REFERENCES fees(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  amount DECIMAL(10,2) NOT NULL,
  payment_method TEXT CHECK (payment_method IN ('upi','card','netbanking')),
  transaction_id TEXT UNIQUE,
  receipt_number TEXT,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','success','failed','refunded')),
  paid_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- SALARY (Teacher)
CREATE TABLE salary (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  teacher_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  month TEXT NOT NULL,
  gross_salary DECIMAL(10,2),
  basic_pay DECIMAL(10,2),
  hra DECIMAL(10,2),
  da DECIMAL(10,2),
  special_allowance DECIMAL(10,2),
  pf_deduction DECIMAL(10,2),
  tds_deduction DECIMAL(10,2),
  professional_tax DECIMAL(10,2),
  net_salary DECIMAL(10,2),
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','credited')),
  credited_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Homework Tables

```sql
-- HOMEWORK
CREATE TABLE homework (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  due_date TIMESTAMPTZ NOT NULL,
  max_marks INT,
  attachment_url TEXT,
  reference_material TEXT,
  rubric TEXT,
  target_class TEXT NOT NULL,
  status TEXT DEFAULT 'active' CHECK (status IN ('active','closed')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- HOMEWORK SUBMISSIONS
CREATE TABLE homework_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  homework_id UUID REFERENCES homework(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  submission_text TEXT,
  attachment_url TEXT,
  file_name TEXT,
  submitted_at TIMESTAMPTZ DEFAULT NOW(),
  status TEXT DEFAULT 'submitted' CHECK (status IN ('submitted','graded','late')),
  grade TEXT,
  marks DECIMAL(5,2),
  teacher_remarks TEXT,
  graded_by UUID REFERENCES profiles(id),
  graded_at TIMESTAMPTZ,
  is_plagiarism_flagged BOOLEAN DEFAULT FALSE
);
```

### Communication Tables

```sql
-- NOTICES
CREATE TABLE notices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  category TEXT NOT NULL CHECK (category IN ('Urgent','General','Event','Academic')),
  author_id UUID REFERENCES profiles(id),
  author_name TEXT,
  target_audience TEXT DEFAULT 'all',
  attachment_url TEXT,
  is_pinned BOOLEAN DEFAULT FALSE,
  is_urgent BOOLEAN DEFAULT FALSE,
  status TEXT DEFAULT 'published' CHECK (status IN ('draft','published','scheduled')),
  scheduled_at TIMESTAMPTZ,
  published_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ
);

-- EVENTS
CREATE TABLE events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  title TEXT NOT NULL,
  description TEXT,
  event_date DATE,
  event_time TIME,
  venue TEXT,
  banner_url TEXT,
  gradient_colors JSONB,
  max_participants INT,
  current_participants INT DEFAULT 0,
  registration_deadline DATE,
  is_registration_open BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EVENT REGISTRATIONS
CREATE TABLE event_registrations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  event_id UUID REFERENCES events(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  registered_at TIMESTAMPTZ DEFAULT NOW(),
  status TEXT DEFAULT 'registered',
  UNIQUE(event_id, student_id)
);

-- CHAT GROUPS
CREATE TABLE chat_groups (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  name TEXT NOT NULL,
  description TEXT,
  icon_url TEXT,
  created_by UUID REFERENCES profiles(id),
  member_count INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- CHAT GROUP MEMBERS
CREATE TABLE chat_group_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  group_id UUID REFERENCES chat_groups(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  role TEXT DEFAULT 'member' CHECK (role IN ('admin','member')),
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(group_id, user_id)
);

-- MESSAGES
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  sender_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  receiver_id UUID REFERENCES profiles(id),
  group_id UUID REFERENCES chat_groups(id),
  content TEXT NOT NULL,
  attachment_url TEXT,
  is_read BOOLEAN DEFAULT FALSE,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- NOTIFICATIONS
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  type TEXT NOT NULL,
  reference_id UUID,
  reference_type TEXT,
  is_read BOOLEAN DEFAULT FALSE,
  priority TEXT DEFAULT 'normal',
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Leave, Achievement, Library, Transport, Live Classes, Study Materials, Settings Tables

```sql
-- LEAVE APPLICATIONS
CREATE TABLE leave_applications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  applicant_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  applicant_role TEXT CHECK (applicant_role IN ('student','teacher')),
  leave_type TEXT NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  reason TEXT NOT NULL,
  attachment_url TEXT,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','approved','rejected','cancelled')),
  approved_by UUID REFERENCES profiles(id),
  approved_at TIMESTAMPTZ,
  remarks TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ACHIEVEMENTS
CREATE TABLE achievements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  name TEXT NOT NULL,
  description TEXT,
  icon TEXT,
  badge_color JSONB,
  xp_reward INT DEFAULT 0,
  criteria TEXT,
  rarity TEXT DEFAULT 'common',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- STUDENT ACHIEVEMENTS
CREATE TABLE student_achievements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  achievement_id UUID REFERENCES achievements(id) ON DELETE CASCADE,
  earned_at TIMESTAMPTZ DEFAULT NOW(),
  progress DECIMAL(5,2) DEFAULT 100,
  UNIQUE(student_id, achievement_id)
);

-- LIBRARY BOOKS
CREATE TABLE library_books (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  title TEXT NOT NULL,
  author TEXT,
  isbn TEXT,
  category TEXT,
  shelf_location TEXT,
  total_copies INT DEFAULT 1,
  available_copies INT DEFAULT 1,
  cover_url TEXT,
  is_digital BOOLEAN DEFAULT FALSE,
  digital_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- LIBRARY BORROWS
CREATE TABLE library_borrows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  book_id UUID REFERENCES library_books(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  borrowed_at TIMESTAMPTZ DEFAULT NOW(),
  due_at TIMESTAMPTZ NOT NULL,
  returned_at TIMESTAMPTZ,
  renewals_used INT DEFAULT 0,
  max_renewals INT DEFAULT 2,
  status TEXT DEFAULT 'borrowed',
  fine_amount DECIMAL(10,2) DEFAULT 0
);

-- BUS ROUTES
CREATE TABLE bus_routes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  route_name TEXT NOT NULL,
  bus_number TEXT,
  driver_name TEXT,
  driver_phone TEXT,
  total_capacity INT DEFAULT 40,
  current_passengers INT DEFAULT 0,
  status TEXT DEFAULT 'active'
);

-- BUS STOPS
CREATE TABLE bus_stops (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  route_id UUID REFERENCES bus_routes(id) ON DELETE CASCADE,
  stop_name TEXT NOT NULL,
  latitude DECIMAL(10,8) NOT NULL,
  longitude DECIMAL(11,8) NOT NULL,
  stop_order INT NOT NULL,
  estimated_arrival TIME,
  is_student_stop BOOLEAN DEFAULT FALSE
);

-- BUS LOCATIONS
CREATE TABLE bus_locations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  route_id UUID REFERENCES bus_routes(id) ON DELETE CASCADE,
  latitude DECIMAL(10,8) NOT NULL,
  longitude DECIMAL(11,8) NOT NULL,
  speed DECIMAL(5,2),
  heading DECIMAL(5,2),
  eta_minutes INT,
  recorded_at TIMESTAMPTZ DEFAULT NOW()
);

-- LIVE CLASSES
CREATE TABLE live_classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  scheduled_at TIMESTAMPTZ NOT NULL,
  duration_minutes INT DEFAULT 60,
  platform TEXT DEFAULT 'EduSHAMIIT',
  stream_url TEXT,
  recording_url TEXT,
  is_live BOOLEAN DEFAULT FALSE,
  viewer_count INT DEFAULT 0,
  target_class TEXT,
  status TEXT DEFAULT 'scheduled' CHECK (status IN ('draft','scheduled','live','completed','recorded')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- LIVE CLASS COMMENTS
CREATE TABLE live_class_comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  live_class_id UUID REFERENCES live_classes(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  comment TEXT NOT NULL,
  is_pinned BOOLEAN DEFAULT FALSE,
  likes INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- STUDY MATERIALS
CREATE TABLE study_materials (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  material_type TEXT CHECK (material_type IN ('Notes','PPTs','Videos','Worksheets')),
  target_class TEXT,
  attachment_urls JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- USER SETTINGS
CREATE TABLE user_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE UNIQUE,
  push_notifications BOOLEAN DEFAULT TRUE,
  sms_alerts BOOLEAN DEFAULT TRUE,
  email_reports BOOLEAN DEFAULT FALSE,
  ai_personalization BOOLEAN DEFAULT TRUE,
  biometric_login BOOLEAN DEFAULT TRUE,
  dark_mode BOOLEAN DEFAULT FALSE,
  language TEXT DEFAULT 'en',
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- DOCUMENTS
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  document_type TEXT NOT NULL,
  file_url TEXT NOT NULL,
  file_name TEXT,
  verification_status TEXT DEFAULT 'pending',
  uploaded_at TIMESTAMPTZ DEFAULT NOW()
);

-- GRADING POLICIES
CREATE TABLE grading_policies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  teacher_id UUID REFERENCES profiles(id),
  class TEXT NOT NULL,
  subject_id UUID REFERENCES subjects(id),
  mid_term_weight INT DEFAULT 30,
  final_term_weight INT DEFAULT 40,
  attendance_weight INT DEFAULT 5,
  assignment_weight INT DEFAULT 10,
  class_test_weight INT DEFAULT 10,
  lab_weight INT DEFAULT 5,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- AI CHAT HISTORY
CREATE TABLE ai_chat_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  session_id UUID,
  role TEXT CHECK (role IN ('user','assistant')),
  content TEXT NOT NULL,
  tool_calls JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- KNOWLEDGE BASE (RAG)
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE knowledge_base (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject TEXT,
  grade TEXT,
  source TEXT,
  content TEXT,
  embedding vector(1536),
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX ON knowledge_base USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

-- IoT TABLES
CREATE TABLE iot_devices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  room_id TEXT NOT NULL,
  device_id TEXT UNIQUE NOT NULL,
  ip_address TEXT,
  num_relays INT DEFAULT 4,
  last_seen TIMESTAMPTZ,
  is_online BOOLEAN DEFAULT true
);

CREATE TABLE iot_device_states (
  school_id UUID REFERENCES schools(id),
  device_id TEXT PRIMARY KEY,
  room_id TEXT,
  fan TEXT DEFAULT 'off',
  light1 TEXT DEFAULT 'off',
  light2 TEXT DEFAULT 'off',
  projector TEXT DEFAULT 'off',
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE iot_control_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  room_id TEXT,
  device TEXT,
  action TEXT,
  triggered_by TEXT DEFAULT 'ai_assistant',
  user_id UUID,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE iot_scheduled_actions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  room_id TEXT,
  device TEXT,
  action TEXT,
  scheduled_time TIMESTAMPTZ,
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- STUDENT TRANSPORT ASSIGNMENT
CREATE TABLE student_transport (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  route_id UUID REFERENCES bus_routes(id),
  stop_id UUID REFERENCES bus_stops(id),
  seat_no TEXT,
  assigned_at TIMESTAMPTZ DEFAULT NOW()
);
```

### RLS Policies with school_id

```sql
-- Enable RLS on all user-data tables
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE results ENABLE ROW LEVEL SECURITY;
ALTER TABLE attendance ENABLE ROW LEVEL SECURITY;
ALTER TABLE fees ENABLE ROW LEVEL SECURITY;
ALTER TABLE payments ENABLE ROW LEVEL SECURITY;
ALTER TABLE salary ENABLE ROW LEVEL SECURITY;
ALTER TABLE homework_submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE exam_submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE exam_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE leave_applications ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE event_registrations ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_achievements ENABLE ROW LEVEL SECURITY;
ALTER TABLE library_borrows ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_settings ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;
ALTER TABLE grading_policies ENABLE ROW LEVEL SECURITY;
ALTER TABLE ai_chat_history ENABLE ROW LEVEL SECURITY;

-- School isolation: users see only their school's data
CREATE POLICY "school_isolation_profiles" ON profiles
  USING (school_id = (SELECT school_id FROM profiles WHERE id = auth.uid()));

CREATE POLICY "school_isolation_results" ON results
  USING (school_id = (SELECT school_id FROM profiles WHERE id = auth.uid()));

CREATE POLICY "school_isolation_attendance" ON attendance
  USING (school_id = (SELECT school_id FROM profiles WHERE id = auth.uid()));

CREATE POLICY "school_isolation_fees" ON fees
  USING (school_id = (SELECT school_id FROM profiles WHERE id = auth.uid()));

CREATE POLICY "school_isolation_notifications" ON notifications
  USING (school_id = (SELECT school_id FROM profiles WHERE id = auth.uid()));

-- Own data policies
CREATE POLICY "own_profile_view" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "own_profile_update" ON profiles FOR UPDATE USING (auth.uid() = id);

CREATE POLICY "own_results" ON results FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "own_attendance" ON attendance FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "own_fees" ON fees FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "own_salary" ON salary FOR SELECT USING (auth.uid() = teacher_id);
CREATE POLICY "own_leave" ON leave_applications FOR SELECT USING (auth.uid() = applicant_id);
CREATE POLICY "own_messages" ON messages FOR SELECT USING (auth.uid() = sender_id OR auth.uid() = receiver_id);
CREATE POLICY "own_notifications" ON notifications FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "own_ai_chats" ON ai_chat_history FOR SELECT USING (auth.uid() = user_id);

-- Teacher policies
CREATE POLICY "teachers_view_all_results" ON results FOR SELECT USING (true);
CREATE POLICY "teachers_insert_results" ON results FOR INSERT WITH CHECK (true);
CREATE POLICY "teachers_view_all_attendance" ON attendance FOR SELECT USING (true);
CREATE POLICY "teachers_insert_attendance" ON attendance FOR INSERT WITH CHECK (true);
CREATE POLICY "teachers_view_submissions" ON homework_submissions FOR SELECT USING (true);
CREATE POLICY "teachers_grade_submissions" ON homework_submissions FOR UPDATE USING (true);
CREATE POLICY "teachers_view_exam_submissions" ON exam_submissions FOR SELECT USING (true);
CREATE POLICY "teachers_grade_exams" ON exam_submissions FOR UPDATE USING (true);
CREATE POLICY "teachers_approve_leaves" ON leave_applications FOR UPDATE USING (true);

-- Public read policies
CREATE POLICY "anyone_read_subjects" ON subjects FOR SELECT USING (true);
CREATE POLICY "anyone_read_timetable" ON timetable FOR SELECT USING (true);
CREATE POLICY "anyone_read_notices" ON notices FOR SELECT USING (true);
CREATE POLICY "anyone_read_events" ON events FOR SELECT USING (true);
CREATE POLICY "anyone_read_homework" ON homework FOR SELECT USING (true);
CREATE POLICY "anyone_read_exams" ON exams FOR SELECT USING (true);
CREATE POLICY "anyone_read_achievements" ON achievements FOR SELECT USING (true);
CREATE POLICY "anyone_read_library" ON library_books FOR SELECT USING (true);
CREATE POLICY "anyone_read_bus_routes" ON bus_routes FOR SELECT USING (true);
CREATE POLICY "anyone_read_bus_stops" ON bus_stops FOR SELECT USING (true);
CREATE POLICY "anyone_read_live_classes" ON live_classes FOR SELECT USING (true);
CREATE POLICY "anyone_read_materials" ON study_materials FOR SELECT USING (true);
```

### Enable Realtime

```sql
ALTER PUBLICATION supabase_realtime ADD TABLE bus_locations;
ALTER PUBLICATION supabase_realtime ADD TABLE messages;
ALTER PUBLICATION supabase_realtime ADD TABLE live_classes;
ALTER PUBLICATION supabase_realtime ADD TABLE notifications;
ALTER PUBLICATION supabase_realtime ADD TABLE exam_sessions;
ALTER PUBLICATION supabase_realtime ADD TABLE live_class_comments;
ALTER PUBLICATION supabase_realtime ADD TABLE attendance;
```

### Seed Data

```sql
-- Sample school
INSERT INTO schools (name, address, phone) VALUES
  ('Shami Innovation Academy', 'Dehradun, Uttarakhand', '+91-135-2765432');

-- Sample subjects (with school_id)
INSERT INTO subjects (school_id, name, icon, color, total_chapters, class) VALUES
  ((SELECT id FROM schools LIMIT 1), 'Mathematics', '📐', '#4F46E5', 42, 'X-A'),
  ((SELECT id FROM schools LIMIT 1), 'Physics', '⚛️', '#3B82F6', 38, 'X-A'),
  ((SELECT id FROM schools LIMIT 1), 'Chemistry', '⚗️', '#F59E0B', 35, 'X-A'),
  ((SELECT id FROM schools LIMIT 1), 'English', '📖', '#10B981', 28, 'X-A');

-- Sample achievements (with school_id)
INSERT INTO achievements (school_id, name, description, icon, badge_color, xp_reward, criteria, rarity) VALUES
  ((SELECT id FROM schools LIMIT 1), 'Academic Excellence', 'Scored 90%+ in term exams', '🏆', '["#FCD34D","#F59E0B"]', 500, 'Score 90%+ in any term', 'rare'),
  ((SELECT id FROM schools LIMIT 1), '18-Day Streak', 'Logged in 18 consecutive days', '🔥', '["#60A5FA","#3B82F6"]', 300, 'Maintain login streak', 'uncommon'),
  ((SELECT id FROM schools LIMIT 1), 'Zero Late Submissions', 'All homework on time this term', '✅', '["#86EFAC","#22C55E"]', 200, 'Submit all homework before deadline', 'common');
```

---

## 15. FLUTTER INTEGRATION

### AI Service (HTTP + SSE Streaming)

```dart
// lib/core/services/ai_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class AiService {
  // Python backend URL (update for production)
  static const String _base = 'http://127.0.0.1:8000';

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
// lib/core/providers/ai_chat_provider.dart
final aiChatProvider = StateNotifierProvider.autoDispose
    <AiChatNotifier, AiChatState>((ref) {
  return AiChatNotifier(ref.read(aiServiceProvider), ref.read(authProvider).token);
});

class AiChatNotifier extends StateNotifier<AiChatState> {
  AiChatNotifier(this._service, this._token) : super(AiChatState.empty());

  Future<void> sendText(String text) async {
    state = state.addUserMessage(text).startStreaming();
    await for (final chunk in _service.streamMessage(
      message: text,
      sessionId: state.sessionId,
      token: _token,
    )) {
      if (chunk.type == 'text')        state = state.appendToken(chunk.content);
      if (chunk.type == 'tool_result') state = state.addToolResult(chunk);
      if (chunk.type == 'done')        state = state.finishStreaming();
    }
  }
}
```

### Chat Message Widget

```dart
// lib/shared/widgets/chat_message_widget.dart
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

## 16. SECURITY & AUTH

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
        profile = get_supabase().table("profiles")\
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
- API keys live only in backend `.env`
- Never put any API key inside the Flutter app
- IoT MQTT broker runs on LAN only — not exposed to internet
- Face image data never leaves the school premises

---

## 17. ENVIRONMENT VARIABLES

```bash
# .env

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJ...
SUPABASE_ANON_KEY=eyJ...
SUPABASE_JWT_SECRET=your-jwt-secret

# AI Models
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=AIza...
OPENAI_API_KEY=sk-...

# Payments
RAZORPAY_KEY_ID=rzp_live_...
RAZORPAY_SECRET=...

# WhatsApp
WHATSAPP_API_KEY=...
WHATSAPP_NUMBER=917XXXXXXXXX

# TTS
GOOGLE_TTS_API_KEY=...
ELEVENLABS_API_KEY=...

# IoT
MQTT_BROKER_HOST=localhost
MQTT_BROKER_PORT=1883

# Redis
REDIS_URL=redis://localhost:6379

# App
APP_ENV=production
```

---

## 18. DEPLOYMENT

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
railway add
railway up
```

### MQTT Broker → Docker

```bash
docker run -d \
  --name mosquitto \
  --restart always \
  -p 1883:1883 \
  -v ./mosquitto.conf:/mosquitto/config/mosquitto.conf \
  eclipse-mosquitto
```

### ESP32 → Flash per Room
1. Edit WIFI_SSID, WIFI_PASS, MQTT_SERVER, DEVICE_ID, DEVICE_ROOM
2. Flash with Arduino IDE
3. Add device to Supabase `iot_devices` table
4. Verify with: `mosquitto_sub -t "school/class_10a/status"`

---

## 19. COST OPTIMISATION

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
1. **Cache repeat answers** — Same question from 100 students = 1 LLM call
2. **Per-user daily limits** — Free tier: 20 AI queries/day
3. **Batch embedding** — Night-time PDF ingestion with 50% discount
4. **Local LLM fallback** — Llama 3.2 for simple one-word answers
5. **Response deduplication** — Hash question; return cached response within 1 hour

---

## 20. BUILD ORDER

```
PHASE 1 — Foundation (Week 1)
  [ ] 1. Supabase project setup + all schema tables created
  [ ] 2. RLS policies enabled with school_id
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
  [ ] 12. student_tools.py (20 student tools)
  [ ] 13. teacher_tools.py (20 teacher tools)
  [ ] 14. fee_tools.py (fee status, Razorpay link)
  [ ] 15. academic_tools.py (question generator, lesson plan)
  [ ] 16. whatsapp_tools.py (Gupshup integration)
  [ ] 17. Tool permission filtering by role
  [ ] 18. Chat history load/save to Supabase

PHASE 4 — RAG (Week 4)
  [ ] 19. pgvector extension enabled
  [ ] 20. rag_ingestion.py — PDF → chunks → embeddings → store
  [ ] 21. rag_tools.py — search_curriculum tool
  [ ] 22. /rag/ingest endpoint for teacher uploads
  [ ] 23. RAG tested with NCERT sample PDF

PHASE 5 — Multimodal (Week 5)
  [ ] 24. /chat/voice — Whisper STT working
  [ ] 25. /chat/image — Gemini Flash vision working
  [ ] 26. TTS response (Google TTS → audio URL)
  [ ] 27. Flutter: voice recording, image picker, audio player

PHASE 6 — IoT (Week 6)
  [ ] 28. MQTT broker (Mosquitto) running
  [ ] 29. ESP32 firmware flashed on first test board
  [ ] 30. iot_controller.py MQTT service working
  [ ] 31. iot_tools.py LangChain tools connected
  [ ] 32. /iot/control and /iot/status endpoints
  [ ] 33. Test: AI command → MQTT → relay fires

PHASE 7 — Camera (Week 7–8)
  [ ] 34. YOLOv8 installed on local school server
  [ ] 35. DeepFace installed + student face enrollment
  [ ] 36. camera_attendance.py reading RTSP stream
  [ ] 37. Attendance auto-written to Supabase

PHASE 8 — Flutter UI (Parallel with above)
  [ ] 38. AiService (HTTP client + SSE streaming)
  [ ] 39. Riverpod AiChatNotifier state management
  [ ] 40. Chat screen: text, voice, image input buttons
  [ ] 41. Message widgets: text bubble, voice bubble, image bubble
  [ ] 42. Tool result cards: FeeStatusCard, AttendanceCard, PaymentLinkCard

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

---
**Last updated:** April 2026
**Maintainer:** Naresh — EduSHAMIIT