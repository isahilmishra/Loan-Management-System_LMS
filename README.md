<div align="center">

# 🏦 LendFlow
### *Loan Management System*

> A modern, production-grade LMS built as a CreditSea technical assignment — covering the full journey from borrower registration to loan settlement.

[![TypeScript](https://img.shields.io/badge/TypeScript-97.5%25-3178C6?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![Next.js](https://img.shields.io/badge/Frontend-Next.js-black?style=flat-square&logo=next.js)](https://nextjs.org/)
[![Express](https://img.shields.io/badge/Backend-Express.js-green?style=flat-square&logo=express)](https://expressjs.com/)
[![MongoDB](https://img.shields.io/badge/Database-MongoDB-47A248?style=flat-square&logo=mongodb)](https://www.mongodb.com/)

### 🎬 [Watch Demo Video](https://drive.google.com/file/d/1C7Z8QKNMpfErvI508quqElc6gU-oy7JH/view?usp=sharing)

</div>

---

## 📌 Table of Contents

1. [What is LendFlow?](#-what-is-lendflow)
2. [System Architecture](#-system-architecture)
3. [Loan Lifecycle](#-loan-lifecycle)
4. [Business Rule Engine (BRE)](#-business-rule-engine-bre)
5. [Role-Based Access](#-role-based-access)
6. [Sandbox Credentials](#-sandbox-tester-credentials)
7. [Testing Walkthrough](#-evaluator-testing-walkthrough)
8. [Running Locally](#-running-the-project-locally)
9. [Schema & Architecture Specs](#-schema--architecture-specifications)

---

## 💡 What is LendFlow?

LendFlow is a full-stack **Loan Management System** that simulates a real-world lending platform end-to-end.

**Borrowers** can:
- Register and fill in KYC details
- Undergo an instant BRE eligibility audit
- Upload salary slips and configure loan parameters via sliders
- Track their loan approval status in real time

**Operations Staff** can access a role-based dashboard covering:

| Department | Responsibility |
|---|---|
| 🧑‍💼 Sales | Track leads and pre-application stages |
| ✅ Sanction | Review and approve/reject loan applications |
| 💸 Disbursement | Release funds for sanctioned loans |
| 🗂️ Collection | Record repayments and manage loan closures |

---

## 🏗 System Architecture

```
flowchart TB
    subgraph FE["🖥️ Frontend — Next.js :3000"]
        BP["Borrower Portal\nMulti-step wizard · Status tracker"]
        OD["Operations Dashboard\nSales · Sanction · Disburse · Collection"]
    end

    subgraph BE["⚙️ Backend — Express API :5001"]
        AUTH["Auth Service\nJWT · role guard"]
        LOAN["Loan API\nCRUD · lifecycle"]
        BRE["BRE Engine\nAge · Salary · PAN · Employment"]
        PAY["Payments API\nUTR · auto-close"]
    end

    subgraph DB["🗄️ MongoDB"]
        U["Users\nRoles · hashed password"]
        L["Loans\nKYC · slip path · SI calc · status"]
        P["Payments\nUTR · amount · timestamp"]
    end

    BP -->|REST| AUTH
    BP -->|REST| LOAN
    BP -->|REST| BRE
    OD -->|REST| LOAN
    OD -->|REST| PAY
    OD -->|REST| AUTH

    AUTH --> U
    LOAN --> L
    LOAN --> BRE
    PAY --> P
    PAY -->|auto-close check| L
```

---

## 🔄 Loan Lifecycle

Every loan follows a strict progression through these states:

```
PRE_APPLIED → APPLIED → SANCTIONED → DISBURSED → CLOSED
                                   ↘ REJECTED
```

Here is the full end-to-end flow:

```
flowchart TD
    A([Borrower registers]) --> B[Enter KYC details\nAge · Salary · PAN · Employment]
    B --> C{BRE audit\nserver-side}
    C -->|fail| D([❌ Blocked\nFailed rules listed])
    C -->|pass| E[Upload salary slip\nPDF or image ≤ 5 MB]
    E --> F[Configure loan\nAmount + tenure sliders]
    F --> G[[PRE_APPLIED]]
    G --> H[[APPLIED\nAwaiting review]]

    H --> I[Sales exec\nreviews lead]
    I --> J[Sanction underwriter\nreviews slip]
    J -->|reject| K([❌ REJECTED])
    J -->|approve| L[[SANCTIONED]]

    L --> M[Disbursement officer\nreleases funds]
    M --> N[[DISBURSED\nLoan active]]

    N --> O[Collection exec\nrecords payments with UTR]
    O --> P{Outstanding\nbalance = ₹0?}
    P -->|no| O
    P -->|yes| Q([✅ CLOSED\nLoan settled])

    style D fill:#fee2e2,stroke:#ef4444,color:#7f1d1d
    style K fill:#fee2e2,stroke:#ef4444,color:#7f1d1d
    style Q fill:#dcfce7,stroke:#22c55e,color:#14532d
    style G fill:#e0f2fe,stroke:#0ea5e9,color:#0c4a6e
    style H fill:#e0f2fe,stroke:#0ea5e9,color:#0c4a6e
    style L fill:#f3e8ff,stroke:#a855f7,color:#3b0764
    style N fill:#ffedd5,stroke:#f97316,color:#7c2d12
```

---

## 🤖 Business Rule Engine (BRE)

The BRE runs **server-side** at KYC submission and instantly blocks ineligible applicants, listing every failed rule.

```
flowchart TD
    START([KYC submitted]) --> R1

    R1{"Age check\n23 ≤ age ≤ 50"}
    R1 -->|fail| F1(["❌ Blocked — age out of range"])
    R1 -->|pass| R2

    R2{"Salary check\nNet ≥ ₹25,000 / month"}
    R2 -->|fail| F2(["❌ Blocked — salary too low"])
    R2 -->|pass| R3

    R3{"PAN format\n^[A-Z]{5}[0-9]{4}[A-Z]$"}
    R3 -->|fail| F3(["❌ Blocked — invalid PAN"])
    R3 -->|pass| R4

    R4{"Employment check\nNot unemployed"}
    R4 -->|fail| F4(["❌ Blocked — unemployed applicant"])
    R4 -->|pass| OK

    OK(["✅ Eligible — proceed to salary slip upload"])

    style F1 fill:#fee2e2,stroke:#ef4444,color:#7f1d1d
    style F2 fill:#fee2e2,stroke:#ef4444,color:#7f1d1d
    style F3 fill:#fee2e2,stroke:#ef4444,color:#7f1d1d
    style F4 fill:#fee2e2,stroke:#ef4444,color:#7f1d1d
    style OK fill:#dcfce7,stroke:#22c55e,color:#14532d
```

**BRE Rules at a glance:**

| Rule | Criteria |
|---|---|
| 🎂 Age | Must be between **23 and 50** (calculated from DOB) |
| 💰 Salary | Net monthly income must be **≥ ₹25,000** |
| 🪪 PAN | Must match `^[A-Z]{5}[0-9]{4}[A-Z]{1}$` |
| 💼 Employment | Must **not** be unemployed |

---

## 👥 Role-Based Access

Each staff role sees only their part of the operations dashboard:

```
flowchart LR
    Admin["🔑 Admin\nadmin@lendflow.com"] --> S & SA & D & C

    S["🧑‍💼 Sales\nsales@lendflow.com\n\nViews pre-application leads"]
    SA["✅ Sanction\nsanction@lendflow.com\n\nApproves or rejects\nAPPLIED loans"]
    D["💸 Disbursement\ndisbursement@lendflow.com\n\nReleases funds for\nSANCTIONED loans"]
    C["🗂️ Collection\ncollection@lendflow.com\n\nRecords UTR payments\nTriggers auto-close"]

    style Admin fill:#fef9c3,stroke:#eab308,color:#713f12
    style S fill:#e0f2fe,stroke:#0ea5e9,color:#0c4a6e
    style SA fill:#f3e8ff,stroke:#a855f7,color:#3b0764
    style D fill:#ffedd5,stroke:#f97316,color:#7c2d12
    style C fill:#dcfce7,stroke:#22c55e,color:#14532d
```

---

## 🔑 Sandbox Tester Credentials

> All accounts share the password: **`Password123`**

| Role | Email | What to Test |
|---|---|---|
| 🔑 **System Administrator** | `admin@lendflow.com` | Access ALL modules |
| 🧑‍💼 **Sales Executive** | `sales@lendflow.com` | Lead tracking (pre-application stages) |
| ✅ **Sanction Underwriter** | `sanction@lendflow.com` | Approve or decline applied loans |
| 💸 **Disbursement Officer** | `disbursement@lendflow.com` | Release funds for sanctioned applications |
| 🗂️ **Collection Executive** | `collection@lendflow.com` | Record payments with UTR, auto-close |
| ⏳ **Borrower (Pending)** | `borrower2@lendflow.com` | Loan in APPLIED stage |
| ✅ **Borrower (Active)** | `borrower4@lendflow.com` | Loan in DISBURSED stage |

---

## 🚶 Evaluator Testing Walkthrough

> 🎬 **Prefer a visual overview first?** [Watch the full demo video](https://drive.google.com/file/d/1C7Z8QKNMpfErvI508quqElc6gU-oy7JH/view?usp=sharing) before going through the steps below.

Follow these 4 steps to experience the **complete loan lifecycle in ~2 minutes**.

---

### Step 1 — Create a Borrower Application

1. Go to `http://localhost:3000` → **Sign In**
2. Register a new account (e.g. `alex@lendflow.com`) or log in as a borrower
3. On **Step 2 (Eligibility)**, enter KYC details:
   - DOB: `1995-05-10` *(age must be 23–50)*
   - Salary: `₹45,000` *(must be ₹25,000+)*
   - PAN: `ABCDE1234F` *(alphanumeric regex checked)*
   - Employment: `Salaried`
   > 💡 Try entering invalid inputs (e.g. Unemployed or ₹15k salary) — the BRE will instantly block and list the exact failed rules!
4. Click **Validate** — watch the live rule scanning animation pass
5. On **Step 3 (Salary Slip)**, drag and drop any test image/PDF *(Max 5MB)*
6. On **Step 4 (Configure)**, select **₹2,50,000** for **180 Days** using the sliders — the live panel calculates the 12% p.a. simple interest. Click **Submit**
7. Status screen shows loan as **`APPLIED`** — log out

---

### Step 2 — Underwrite & Sanction the Loan

1. Log in as **Sanction** (`sanction@lendflow.com` / `Password123`)
2. Find `alex@lendflow.com`'s application in your queue
3. Click **"View / Download Salary Slip"** to verify income proof
4. Click **Approve & Sanction** → status updates to `SANCTIONED` — log out

---

### Step 3 — Disburse the Funds

1. Log in as **Disbursement** (`disbursement@lendflow.com` / `Password123`)
2. Find the sanctioned loan in the **Disbursement Queue** tab
3. Click **Release Funds** → status updates to `DISBURSED` — log out

---

### Step 4 — Record Payments & Auto-Close

1. Log in as **Collection** (`collection@lendflow.com` / `Password123`)
2. Open the **Collections Ledger** → find the active loan
3. Click **Record Payment** → enter UTR code (e.g. `UTR990088`) and a partial amount of `₹1,00,000`
4. Log a final payment for the remaining balance
5. Once outstanding balance hits **₹0**, the loan auto-transitions to **`CLOSED`**
6. Log back in as `alex@lendflow.com` — the tracker reads: **"Loan Closed & Settled - No outstanding dues!"**

> Here's a visual of this repayment flow:

```
flowchart LR
    REC["Collection Exec\nrecords payment\nwith UTR"] --> CHK{Outstanding\nbalance = 0?}
    CHK -->|No| PART["Partial payment\nlogged — balance\ndecreases"]
    PART --> REC
    CHK -->|Yes| CLOSE["✅ Loan auto-closes\nStatus → CLOSED\nBorrower notified"]

    style CLOSE fill:#dcfce7,stroke:#22c55e,color:#14532d
    style PART fill:#e0f2fe,stroke:#0ea5e9,color:#0c4a6e
```

---

## 🚀 Running the Project Locally

### Prerequisites

- **Node.js** v20 or higher
- **MongoDB** running locally at `mongodb://127.0.0.1:27017/lendflow` or a MongoDB Atlas connection string

---

### 1. Set Up Environment Variables

**`backend/.env`**
```env
PORT=5001
MONGO_URI=your_mongo_connection_string
JWT_SECRET=your_secret_key
```

**`frontend/.env.local`** *(optional — defaults to localhost:5001)*
```env
NEXT_PUBLIC_API_URL=http://localhost:5001/api
```

---

### 2. Install Dependencies & Seed the Database

Run from the **root workspace directory**:

```bash
# Install root, backend, and frontend packages concurrently
npm run install:all

# Seed the database with evaluator test profiles
npm run seed
```

---

### 3. Start Development Servers

Open two terminal windows:

**Terminal 1 — Backend API**
```bash
cd backend
npm run dev
# API starts at http://localhost:5001
```

**Terminal 2 — Frontend Portal**
```bash
nvm use 24   # Ensure Node 24 for native compiler bindings
cd frontend
npm run dev
# Portal starts at http://localhost:3000
```

---

### 4. (Alternative) Run with Docker

Skip Node, npm, and NVM setup entirely — boot the full platform with one command:

```bash
docker-compose up --build
```

| Service | URL |
|---|---|
| 🖥️ Next.js Frontend | `http://localhost:3000` |
| ⚙️ Express Backend API | `http://localhost:5001` |

---

## 📐 Schema & Architecture Specifications

### Database Collections

**Users**
- Handles logins, hashed passwords, and role classifications

**Loans**
- Stores step records, salary file paths, and calculated simple interest:
  ```
  SI = P × R × T / 36500
  ```
- Tracks lifecycle status: `PRE_APPLIED` → `APPLIED` → `SANCTIONED` → `DISBURSED` → `CLOSED` (or `REJECTED`)

**Payments**
- Logs each transaction with a **globally unique UTR** to prevent duplicates
- Auto-calculates total paid vs. repayment target to trigger automatic loan closure

---

### Simple Interest Calculation Flow

```
flowchart LR
    IN["Borrower inputs\nPrincipal (P)\nTenure in days (T)\nRate = 12% p.a."] --> CALC["SI = P × R × T ÷ 36500"]
    CALC --> OUT["Total Repayable\n= P + SI\nDisplayed on slider panel"]

    style IN fill:#e0f2fe,stroke:#0ea5e9,color:#0c4a6e
    style OUT fill:#dcfce7,stroke:#22c55e,color:#14532d
```

---

<div align="center">

Built with ❤️ by [isahilmishra](https://github.com/isahilmishra) · CreditSea Assignment

</div>
