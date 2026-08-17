# 📱 Enterprise Multimodal WhatsApp AI Assistant & Cross-Project Ledger Engine

[![TypeScript](https://img.shields.io/badge/TypeScript-5.4-blue.svg)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-20.x-green.svg)](https://nodejs.org/)
[![Gemini AI](https://img.shields.io/badge/Gemini%20AI-2.5%20Flash-orange.svg)](https://deepmind.google/technologies/gemini/)
[![Firebase](https://img.shields.io/badge/Firebase-Admin%20SDK-yellow.svg)](https://firebase.google.com/)
[![Docker](https://img.shields.io/badge/Docker-Multi--stage-blue.svg)](https://www.docker.com/)

An event-driven, enterprise-grade WhatsApp AI platform built using **TypeScript**, **Node.js**, **Google Gemini 2.5 Flash**, **Firebase Admin SDK**, **Google BigQuery**, and **Google Sheets API**.

This platform combines two core enterprise subsystems:
1. **Multimodal Sales CRM & Task Scheduler**: Bypasses manual entry by extracting contact/meeting details from raw text or business card images, enforcing GST (`Asia/Dubai`) timezone math, and syncing dual-write updates to Cloud Firestore and Google Sheets CRM.
2. **Cross-Project Cash Reconciliation & Ledger Reversal Engine ("Ferrari Foods")**: Solves the administrative pain of manually searching and correcting database entries by utilizing AI OCR image parsing, multi-branch parallel queries, atomic 5-collection Firestore batch deletions, dynamic vault note inventory updates, and immutable non-repudiation audit logging across isolated GCP environments.

---

## 🗺️ Multi-Cloud & Cross-Project Architecture

The microservice bridges two independent Google Cloud Platform / Firebase projects through IAM Service Accounts:

```
+------------------------------------------------------------------------------------+
|                             PROJECT A (Local Bot Engine)                           |
|  - Baileys WhatsApp WebSocket Socket Gateway                                      |
|  - Persistent Auth State in Firebase Realtime DB (whatsapp_sessions)               |
|  - LID Authorization & Branch Directory (authorized_users)                         |
|  - Atomic Idempotency & Zombie Recovery Queue (inbound_messages)                   |
|  - Temporary Multimodal Session State (pending_approvals)                          |
+------------------------------------------------------------------------------------+
                                           |
                         IAM Application Default Credentials (ADC)
                                           |
                                           v
+------------------------------------------------------------------------------------+
|                  PROJECT B (Remote Corporate Accounting System)                    |
|  - Live Ledger: branches/{branchId}/live_ledger/{txId}                             |
|  - Daily Metrics: branches/{branchId}/daily_stats/{date}                           |
|  - Accountant Vault: branches/{branchId}/temp_vault_inventory/{noteValue}          |
|  - CEO Vault: branches/{branchId}/vault_inventory/{noteValue}                       |
|  - Denomination History: branches/{branchId}/daily_denominations/{date}            |
|  - Audit Trail: branches/{branchId}/reversal_logs/{logId}                          |
+------------------------------------------------------------------------------------+
```

---

## 🔥 Key Feature Modules

### 1. ⚡ Cross-Project Automated Ledger Reversal Engine
- **The Problem Solved**: Eliminates the tedious, error-prone manual task of logging into Firebase/BigQuery consoles to find, delete, and adjust multi-collection ledger entries and physical currency inventory.
- **Image/Caption OCR Extraction**: Users send a cash ledger receipt image or brief message (e.g., *"remove desk entry pending ceo 500 AED Johny"*). Gemini 2.5 Flash extracts date, entity name, cash total, and denomination breakdowns (e.g. 5x 500 AED, 10x 100 AED notes).
- **Multi-Branch Concurrent Search**: Checks user branch permissions from Project A (`authorized_users/{cleanLid}`) and queries all authorized branch databases in Project B simultaneously.
- **Smart Fallback Matching**: Bypasses human spelling variations (e.g. `JOHNY` vs `JOHNNY`) if a unique exact-amount match exists for today's transactions.
- **Atomic 5-Collection Batch Deletion**: Upon user reply of `"YES"`, an atomic batch transaction:
  1. Deletes the target document from `live_ledger`.
  2. Decrements `daily_stats/{date}` inflow metrics.
  3. Automatically decrements currency note counts in `temp_vault_inventory` (Accountant Vault for `PENDING CEO`) or `vault_inventory` (CEO Vault for `APPROVED`).
  4. Rolls back `daily_denominations/{date}` snapshot records.
  5. Appends an unalterable audit log to `reversal_logs` recording the executor's WhatsApp LID.

### 2. 📇 Multimodal Sales CRM & Business Card Capture
- **In-Memory OCR Extraction**: Photos of business cards or store fronts are processed directly in memory buffer with Gemini 2.5 Flash Vision, returning company name, contact number, and confidence score.
- **Human-in-the-Loop (HITL) Gate**: Caches extracted details in `pending_approvals/{userId}`. Only upon receiving confirmation (`"Approve"`, `"Save"`, `"OK"`) is the payload committed.
- **Spreadsheet Formula Defense**: Inputs are sanitized with `escapeSpreadsheetFormula()` to strip injection triggers (`=`, `+`, `-`, `@`) before appending to Google Sheets CRM.

### 3. ⏰ Contextual Task & Meeting Scheduler
- **Temporal Date Math**: Parses relative expressions (e.g. *"Remind me next Tuesday at 3 PM"*) and computes explicit ISO timestamps anchored to **Gulf Standard Time (`Asia/Dubai`)**.
- **Automated Reminder Cron Engine**: A high-frequency worker checks pending reminders every 60 seconds and dispatches meeting notifications (30 minutes prior) or daily follow-ups.

### 4. 🤖 Anti-Bot "Human Mimic" Telemetry Choreography
- Protects WhatsApp accounts against automated bot detection heuristics by simulating realistic typing sequences:
  `Read Receipt (Blue Ticks)` ➔ `Reading Pause (1.2s)` ➔ `Typing Leg 1 (45%)` ➔ `Thinking Pause (0.7s)` ➔ `Typing Leg 2 (55%)` ➔ `Drop Status` ➔ `Send Message`.
- Employs a 3-turn delay memory buffer with random jitter calculation to prevent repetitive interval signatures.

### 5. 🛡️ Self-Healing Zombie Queue & Resilient State
- **Stateless Cloud Run Architecture**: Stores Baileys WhatsApp authentication credentials in Firebase Realtime DB, allowing container restarts without re-scanning QR codes.
- **Zombie Recovery Worker**: Scans `inbound_messages` every minute for transactions locked in `processing` (> 3 mins) or `failed`. Automatically re-queues items up to 3 retries before dead-lettering to `dead_letter`.

---

## 🛠️ Tech Stack

- **Runtime**: Node.js (v20+ Alpine) with TypeScript 5.4
- **WhatsApp Client**: `@whiskeysockets/baileys` v6.6
- **AI Processing**: Google Gemini 2.5 Flash API (`@google/generative-ai`)
- **Databases**: Google Cloud Firestore & Firebase Realtime DB (Dual Project)
- **CRM Integration**: Google Sheets Enterprise API v4
- **Containerization**: Multi-stage Docker Dockerfile (Alpine Linux)

---

## 📋 System Requirements & Environment Setup

Create a `.env` file in the root directory:

```env
# Runtime & Timezone
NODE_ENV=production
PORT=8080
TZ=Asia/Dubai

# WhatsApp Session
WA_SESSION_ID=vecta-prod-session-01

# Project A (Local Bot Database)
FIREBASE_PROJECT_ID=your-bot-firebase-project-id
FIREBASE_DATABASE_URL=https://your-bot-firebase-project-id-default-rtdb.firebaseio.com
GCP_SERVICE_ACCOUNT_PATH=./src/config/whatsapp-assistant-sa.json

# Project B (Remote Corporate Cash Management System)
FERRARI_FIRESTORE_PROJECT_ID=your-ferrari-firestore-project-id
FERRARI_BQ_PROJECT_ID=your-ferrari-bq-project-id
FERRARI_BQ_DATASET=cash_management
FERRARI_BQ_TABLE=live_ledger

# External Services
GOOGLE_SHEET_ID=your-google-sheet-id
GEMINI_API_KEY=your-gemini-api-key
DEFAULT_REMINDER_DAYS=2
```

---

## 🚀 Quick Start Guide

### 1. Local Development
```bash
# Install dependencies
npm install

# Run in development mode (watch mode)
npm run dev

# Build TypeScript to /dist
npm run build

# Start production runtime
npm start
```

### 2. Docker Deployment
```bash
# Build multi-stage production image
docker build -t whatsapp-ai-assistant .

# Run container with volume mount for authentication cache
docker run -d \
  --name whatsapp-bot \
  -p 8080:8080 \
  --env-file .env \
  --memory="3.5g" \
  whatsapp-ai-assistant
```

---

## 📄 License
ISC License. Developed for Enterprise Automation.
