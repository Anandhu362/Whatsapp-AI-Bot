# Vecta Assistant: Production-Grade WhatsApp AI Sales Companion

An event-driven, production-ready WhatsApp assistant engineered using **Node.js**, **TypeScript**, and **Gemini 2.5 Flash**. This system completely automates the pipeline from capturing unstructured text/media post-client meetings to structured dual-write persistence in Cloud Firestore and Google Sheets CRM, complete with an autonomous background scheduler.

---

## 🚀 The Core Philosophy

Manual data entry after client meetings is an energy drain. Getting a business card or contact number shouldn't trigger an hour of administrative busywork. 

**Vecta Assistant** allows a professional to send a raw text message or voice note directly inside WhatsApp. The underlying system parses the metadata, performs native date-math anchored to the **Asia/Dubai** timezone, secures the data downstream, updates corporate sheets, and establishes automated self-healing follow-up loops.

---

## 🗺️ System Architecture

The assistant operates as a containerized workload on a dedicated Google Cloud Platform (GCP) Virtual Machine. It decouples state by utilizing Firebase Realtime Database for stateless Baileys authentication handling.

<p align="center">
  <img src="./docs/images/Architecture.jpg" alt="System Architecture Diagram" width="85%">
</p>

### End-to-End Data Lifecycle:
1. **Ingestion:** Message payloads are picked up over a persistent WebSocket connection handled by the **Baileys Engine**.
2. **Security Check:** The `router.ts` engine matches sender data against an authorized whitelist, silently discarding unauthenticated spam.
3. **Idempotency Guarantee:** A Firestore transaction lock checks `msgId` to ensure identical network retries are never processed twice.
4. **Sanitization:** String fields run through a regex shield to block executable scripts and spreadsheet injection parameters.
5. **LLM Structured Extraction:** Gemini 2.5 Flash processes the unstructured string against a strict structural JSON schema.
6. **Dual-Write State Update:** Verified records are concurrently committed to **Cloud Firestore** and appended to a **Google Sheets CRM**.
7. **Background Cron Processing:** A high-frequency background worker sweeps the records every 60 seconds, handling event tracking, automated messaging loops, and error-recovery pipelines.

---

## 🛠️ Deep Dive: Core Features & Implementations

### 🔒 1. Edge Security & Meta Compliance (`router.ts`)
To protect internal cloud infrastructure and satisfy strict platform access policies, the perimeter is heavily guarded. 

<p align="center">
  <img src="./docs/images/Router.png" alt="Router Security Code Snippet" width="75%">
</p>

* **Implementation:** The incoming string is stripped down to its canonical phone number footprint and verified against a protected runtime array (`AUTHORIZED_SENDERS`). Unauthorized messages are dropped silently at the gate before consuming downstream AI tokens.

### 🧠 2. Contextual Data Extraction (`gemini.ts`)
Turning messy human dialogue into deterministic records requires pairing an LLM with strict response schemas.

<p align="center">
  <img src="./docs/images/WhatsApp Image 2026-07-01 at 02.13.29.jpeg" alt="WhatsApp Live Demo Screenshot" width="40%">
  <img src="./docs/images/carbon (3).png" alt="Gemini JSON Output Schema" width="45%">
</p>

* **Implementation:** Leverages the Gemini SDK with enforced JSON schemas (`responseSchema`). When given phrases like *"Remind me next Tuesday..."*, the system automatically computes future dates anchored to the `Asia/Dubai` timezone and generates crisp, structured database keys.

### 🛡️ 3. Defensive Data Sanitization (`sanitizer.ts`)
Because data is synchronized directly with external facing team spreadsheets, the application protects against CSV/Spreadsheet Formula Injection.

<p align="center">
  <img src="./docs/images/Sanitizer.png" alt="Sanitizer Logic Code Snippet" width="75%">
</p>

* **Implementation:** Any payload touching the database is processed by `escapeSpreadsheetFormula`. A regex pattern `/^[=+\-@]/` checks the first byte of incoming values. If an injection signature is discovered, the cell is prepended with a neutralizing single quote (`'`), rendering it as literal plain-text inside Google Sheets.

### 📁 4. Dual-Write State Synchronization
The architecture maintains complete separation of concerns between background worker states and operational user views.

<p align="center">
  <!-- Place your cropped Firestore screenshot here -->
  <img src="./docs/images/image_90540e.png" alt="Cloud Firestore Document Lifecycle Tracking" width="70%">
</p>

* **Cloud Firestore:** Serves as the transactional operational memory, maintaining status keys (`pending`, `processing`, `sent`, `failed`), retry indexes, and locking metrics.
* **Google Sheets CRM:** Functions as a human-readable ledger immediately accessible by internal office employees and field personnel without grant-level database access.

### ⚙️ 5. Self-Healing Background Scheduler (`reminderCron.ts`)
Real-world systems face unexpected processing lockups, network splits, and API timeouts. Vecta uses a fault-tolerant state-machine architecture.

<p align="center">
  <img src="./docs/images/Remindercron.png" alt="Reminder Cron Logic Code Snippet" width="75%">
</p>

* **Implementation:** The background engine scans for items with a state of `processing` that exceed a 3-minute lock window, alongside explicit `failed` transactions. If the transaction has failed permanently (`retryCount >= 3`), it drops into a **Dead-Letter Queue (DLQ)** for manual auditing. Otherwise, it increments the retry value and transparently re-injects the message into the parsing queue.

---

## 📦 Technology Stack & Environment

* **Runtime:** Node.js (v20+) with TypeScript
* **WhatsApp Gateway:** Baileys Core (WebSocket Layer)
* **AI Processing Engine:** Gemini 2.5 Flash API
* **Primary Database:** Google Cloud Firestore & Firebase Realtime DB
* **Business Interface:** Google Sheets Enterprise API
* **Infrastructure Host:** Google Cloud Platform (Compute Engine Dedicated Instance)
* **CI/CD Pipeline:** GitHub Actions (`deploy.yml`) integrated via automated container builds
