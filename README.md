# ProctorAI — Smart Online Proctoring & Exam Integrity System

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        PROCTOR AI SYSTEM                        │
├────────────────┬───────────────────────┬────────────────────────┤
│   FRONTEND     │       BACKEND         │      AI MODULES        │
│  (HTML/JS)     │  (Node.js + Express)  │   (face-api.js +CDN)  │
├────────────────┼───────────────────────┼────────────────────────┤
│ • Student View │ • REST API (Express)  │ • Face Detection       │
│ • Admin View   │ • WebSocket (Socket.io│ • Eye/Gaze Tracking    │
│ • Camera Feed  │ • JWT Auth            │ • Multi-Person Detect  │
│ • AI Panel     │ • MongoDB (Mongoose)  │ • Audio Monitoring     │
│ • Violation Log│ • Rate Limiting       │ • Tab Switch (browser) │
│ • Q&A Engine   │ • AI Report Gen       │ • Paste Detect (real)  │
│ • Live Alerts  │ • Violation Logger    │ • Object Detection     │
└────────────────┴───────────────────────┴────────────────────────┘
                          ↕ Socket.io ↕
┌─────────────────────────────────────────────────────────────────┐
│                       DATABASE (MongoDB)                        │
│  Users | Exams | ExamSessions | Violations | Alerts            │
└─────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
proctor-ai/
├── frontend/
│   └── index.html          ← Complete frontend (student + admin)
├── backend/
│   ├── server.js           ← Express + Socket.io + MongoDB
│   ├── seed.js             ← Database seeder with sample data
│   ├── package.json        ← Dependencies
│   └── .env.example        ← Environment variables template
└── README.md
```

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, Vanilla JS |
| AI (Frontend) | face-api.js (TensorFlow.js), Web Audio API |
| Real-time | Socket.io (WebSocket) |
| Backend | Node.js, Express.js |
| Database | MongoDB + Mongoose |
| Auth | JWT (JSON Web Tokens) |
| AI Reports | Anthropic Claude API |
| Security | Helmet, CORS, Rate Limiting, bcrypt |

---

## Module 1 — Frontend (frontend/index.html)

### Pages
- **Login** — Role selection (Student / Admin)
- **Pre-Exam Setup** — Camera check, face detection verification, system checks
- **Exam Interface** — Questions + AI monitoring panel side by side
- **Admin Dashboard** — Live student grid + alerts + AI report generator

### Features
- Real camera access via `getUserMedia`
- Real audio monitoring via Web Audio API
- Real tab switch detection via `visibilitychange`
- Real copy/paste blocking via `paste`/`copy` events
- Real keyboard shortcut blocking
- Fullscreen enforcement
- Socket.io client for real-time backend sync

---

## Module 2 — Backend (backend/server.js)

### REST API Endpoints

#### Auth
| Method | Endpoint | Description |
|---|---|---|
| POST | /api/auth/register | Register student or admin |
| POST | /api/auth/login | Login and get JWT token |
| GET | /api/auth/me | Get current user |

#### Exams
| Method | Endpoint | Description |
|---|---|---|
| POST | /api/exams | Create exam (admin) |
| GET | /api/exams | List all exams |
| GET | /api/exams/code/:code | Get exam by code |
| PUT | /api/exams/:id | Update exam |
| DELETE | /api/exams/:id | Delete exam |

#### Sessions
| Method | Endpoint | Description |
|---|---|---|
| POST | /api/sessions/start | Start exam session |
| POST | /api/sessions/:id/answer | Save student answer |
| POST | /api/sessions/:id/violation | Log AI violation |
| POST | /api/sessions/:id/submit | Submit exam |
| GET | /api/sessions/active | Active sessions (admin) |
| PATCH | /api/sessions/:id/action | Warn / Terminate / Clear |

#### Admin
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/admin/dashboard | Dashboard stats |
| GET | /api/admin/reports/:id | Generate AI report |
| GET | /api/admin/students | All students + stats |
| GET | /api/alerts | Violation alerts |

### Socket.io Events

#### Student → Server
| Event | Payload | Description |
|---|---|---|
| join_session | sessionId | Join exam room |
| ai_frame_data | { gaze, face, audio } | Send AI frame data |
| heartbeat | sessionId | Keep-alive ping |
| student_help | { sessionId, message } | Request help |

#### Server → Admin
| Event | Description |
|---|---|
| violation | Student violation detected |
| live_frame_data | Real-time AI data |
| student_joined | Student started exam |
| student_disconnected | Student went offline |
| session_submitted | Student submitted |

#### Server → Student
| Event | Description |
|---|---|
| admin_action | Admin warn/terminate |
| proctor_message | Message from proctor |
| heartbeat_ack | Heartbeat acknowledged |

### Database Models

**User** — name, email, password (hashed), role, studentId, department

**Exam** — title, subject, examCode, questions[], duration, settings{}

**ExamSession** — exam, student, answers[], violations[], integrityScore, tabSwitches, status

**Alert** — session, student, type, message, severity, isRead

---

## Module 3 — AI Modules

### 1. Face Detection (face-api.js — TinyFaceDetector)
- Real TensorFlow.js model loaded from CDN
- Detects face presence and confidence score
- Draws bounding box with corner brackets
- Plots 68 facial landmarks
- Triggers alert if face disappears for 3+ seconds

### 2. Eye & Gaze Tracking (face-api.js landmarks)
- Computes gaze direction from eye landmark positions
- Calculates nose-to-eye offset ratio for direction estimation
- Detects: CENTER, LEFT, RIGHT, UP, DOWN
- Logs violation after 4 seconds of looking away

### 3. Multiple Person Detection (face-api.js)
- Detects all faces in frame simultaneously
- Flags if count > 1 with CRITICAL severity
- Draws red bounding box on unrecognized person

### 4. Audio Monitoring (Web Audio API — real)
- Connects to microphone stream
- Real FFT frequency analysis
- Live 12-bar audio level meter
- Alerts on sustained high audio (>65% level)

### 5. Tab Switch Detection (Browser visibilitychange — real)
- Listens to `document.visibilitychange`
- Counts every tab switch
- Logs severity: MED (≤2), HIGH (>2)
- Sent to admin via Socket.io instantly

### 6. Copy-Paste Detection (Browser events — real)
- Intercepts and cancels `paste`, `copy`, `cut` events
- Blocks Ctrl+C, Ctrl+V, Ctrl+X, Ctrl+A, Ctrl+U
- Right-click context menu disabled
- F12 developer tools blocked

### 7. Object Detection (Simulated — YOLO stub)
- Simulates phone, book, earphone, second screen detection
- To connect real YOLO: send frames to `/api/detect` Python endpoint

### 8. Screen Activity Monitor
- Tracks mouse movement patterns
- Detects rapid scrolling and excessive cursor movement

---

## Setup & Installation

### Prerequisites
- Node.js 18+
- MongoDB (local or Atlas)
- npm

### 1. Install Backend

```bash
cd backend
npm install
cp .env.example .env
# Edit .env with your MongoDB URI and API keys
```

### 2. Seed Database

```bash
node seed.js
```

### 3. Start Backend

```bash
npm run dev
# Server starts on http://localhost:5000
```

### 4. Open Frontend

```bash
# Open frontend/index.html in browser
# Or serve with any static server:
npx serve frontend/
# Opens at http://localhost:3000
```

### 5. Test Login Credentials

```
Admin:    admin@proctorai.edu  /  Admin@123
Student:  priya@student.edu   /  Student@123
Exam Code: CS401-FINAL-2025
```

---

## AI Model Loading

face-api.js models load automatically from CDN (jsdelivr).
If offline, system runs in **simulation mode** with realistic behavior patterns.

For production, download models locally:
```bash
mkdir -p frontend/models
# Download from: https://github.com/vladmandic/face-api/tree/master/model
```
Then update MODELS_URL in index.html to `/models/`

---

## Future Enhancements

| Feature | Implementation |
|---|---|
| Real object detection | YOLOv8 Python microservice |
| Voice-to-text monitoring | OpenAI Whisper API |
| Session recording | MediaRecorder API + S3 upload |
| Email alerts | Nodemailer on critical violations |
| Report PDF export | PDFKit backend route |
| Face recognition login | face-api.js face descriptor matching |
| Scalable deployment | Docker + Nginx + PM2 + MongoDB Atlas |

---

## Security Features

- Passwords hashed with bcrypt (12 rounds)
- JWT tokens with expiry
- Rate limiting (200 req/15min)
- Helmet.js security headers
- CORS configured for allowed origins
- Input validation with express-validator
- Admin-only route middleware
- Student can only access their own session
