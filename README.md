<div align="center">

# ☕ BrewCo CRM

### A customer-growth CRM for a coffee brand: RFM segments, churn-risk scoring, campaigns with delivery tracking, and an "Ask Your Data" box that turns plain English into a validated, parameterized query.

<br/>

![React](https://img.shields.io/badge/React_18-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite_6-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind_3-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![Python](https://img.shields.io/badge/Python_3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Neon](https://img.shields.io/badge/Neon-00E599?style=for-the-badge&logo=neon&logoColor=black)
![Clerk](https://img.shields.io/badge/Clerk_Auth-6C47FF?style=for-the-badge&logo=clerk&logoColor=white)
![Groq](https://img.shields.io/badge/Groq-gpt--oss--120b-F55036?style=for-the-badge)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-Frontend-000000?style=for-the-badge&logo=vercel&logoColor=white)
![Render](https://img.shields.io/badge/Render-Backend-46E3B7?style=for-the-badge&logo=render&logoColor=black)
![Tests](https://img.shields.io/badge/tests-130_automated_+_27_regression_cases-2EA44F?style=for-the-badge)

**2 FastAPI services · 5 PostgreSQL tables · 19 CRM endpoints (17 behind Clerk JWT) · 1 scikit-learn model + 1 transparent churn heuristic · 12 test files**

**Maintained by:** Saksham Dhiman

[Live Demo](#-live-demo) · [System Architecture](#-system-architecture) · [Ask Your Data](#-ask-your-data) · [Campaign Delivery Pipeline](#-campaign-delivery-pipeline) · [Security Hardening](#-security-hardening) · [Local Setup](#-local-setup)

</div>

---

## 🚀 Live Demo

| Resource | Link |
|---|---|
| 🖥️ **Frontend** (Vercel) | https://brewco-crm-pi.vercel.app |
| ⚙️ **Backend API** (Render) | https://brewco-crm-backend-7xrd.onrender.com |
| 📦 **Repository** | https://github.com/SakshamDhiman2202/brewco-crm |

> **Signing in:** the app is gated by Clerk. *Continue with Google* is the quickest route in.
>
> **Treat the demo as public.** The live sign-in screen is a Clerk **development-mode** instance with self-service sign-up, so any signed-in user can use every route. The data is synthetic (100 customers, 300 orders) and message delivery is simulated, so please don't enter real personal data. See [Known Limitations](#-known-limitations).

---

## 📚 Table of Contents

- [Live Demo](#-live-demo)
- [At a Glance](#-at-a-glance)
- [Overview](#-overview)
- [Features](#-features)
- [System Architecture](#-system-architecture)
- [Authentication Flow](#-authentication-flow)
- [Ask Your Data](#-ask-your-data)
- [AI and PII Boundary](#-ai-and-pii-boundary)
- [Churn Risk and RFM Segmentation](#-churn-risk-and-rfm-segmentation)
- [Campaign Delivery Pipeline](#-campaign-delivery-pipeline)
- [Data Model](#-data-model)
- [API Endpoints](#-api-endpoints)
- [Security Hardening](#-security-hardening)
- [Frontend](#-frontend)
- [Backend and Repository Structure](#-backend-and-repository-structure)
- [Local Setup](#-local-setup)
- [Deployment](#-deployment)
- [Screenshots and Walkthrough](#-screenshots-and-walkthrough)
- [Engineering Decisions](#-engineering-decisions)
- [Testing](#-testing)
- [Known Limitations](#-known-limitations)
- [Future Work](#-future-work)
- [Tech Stack](#-tech-stack)

---

## 🧭 At a Glance

```mermaid
mindmap
  root((BrewCo CRM))
    Know the customer
      RFM clusters from KMeans
      Churn risk heuristic
      Health badge VIP or At Risk or Active
    Reach the customer
      Validated segment filters
      AI drafted campaign copy
      Email SMS WhatsApp channels
    Measure the result
      KPI dashboard
      Delivery funnel
      Ask Your Data in plain English
    Trust boundaries
      Clerk RS256 JWT with issuer and azp
      Query spec not raw SQL
      PII sanitizer before the LLM
      Shared secret between services
```

### A marketing user's day

```mermaid
journey
    title From sign-in to a measured campaign
    section Understand
      Sign in with Clerk: 5: User
      Scan KPIs and the revenue trend: 4: User
      Ask a question in plain English: 5: User, Groq
    section Target
      Review customer health badges: 4: User
      Convert an RFM cluster into a segment: 5: User
    section Reach out
      Draft campaign copy with AI: 4: User, Groq
      Launch the campaign: 5: User
    section Learn
      Watch the delivery funnel fill in: 5: User, Channel service
```

### Codebase snapshot

| | Lines | Notes |
|---|---:|---|
| 🐍 CRM API (`backend/`, excluding tests, scripts, seed) | ~2,000 | 9 routers, 4 services, 2 clients, 3 core modules |
| 📨 Channel service (`channel-service/`) | ~320 | Delivery simulator with authenticated `/send` |
| ⚛️ Frontend (`frontend/src`) | ~2,800 | 37 files: 4 pages, 7 hooks, 7 service modules |
| 📜 Scripts, migrations, seed | ~800 | RFM training, scoring, guarded dev reset |
| 🧪 Tests | ~3,600 | More test code than application code |

### Design principles

| Principle | What it looks like in the code |
|---|---|
| **Constrain the model, never trust it** | The LLM emits a JSON *spec* from a closed vocabulary. It never writes SQL. |
| **Fail closed** | Missing issuer, secret, or origin list stops the app at boot or returns an error. It never silently relaxes. |
| **Keep PII out of prompts** | Name, email, and phone are unselectable in analytics and stripped again before the summary call. |
| **Honest metrics** | A circular pseudo-label churn classifier was retired in favour of a transparent heuristic. |
| **Test the boundaries** | Every security fix ships with a regression suite that locks it in. |

---

## 📖 Overview

BrewCo CRM is built for a marketing user who wants to **understand customers, target them, contact them, and see whether it worked**. It is a full-stack project with two FastAPI services, a React front end, a PostgreSQL database, and a small offline ML layer.

| Question the user has | How the product answers it |
|---|---|
| *Who are my customers?* | KMeans on recency, frequency, and spend produces **discovered segments** that convert into targetable ones. |
| *Who might leave?* | A transparent **churn-risk heuristic** (recency 50% · frequency 30% · monetary 20%) drives the **At Risk** badge. See [Churn Risk and RFM Segmentation](#-churn-risk-and-rfm-segmentation) for why it is deliberately not a trained classifier. |
| *What does my data say?* | **Ask Your Data**: a plain-English question becomes a validated query spec, a parameterized read-only query, and a short summary. |
| *Did the campaign work?* | A channel service simulates delivery and posts authenticated callbacks. The dashboard shows the **sent → delivered → opened → clicked** funnel. |

**AI:** Groq (`openai/gpt-oss-120b`) suggests segment filters, drafts campaign copy, generates query specs, and summarizes results.
**ML:** one scikit-learn model (StandardScaler + KMeans over RFM features) runs in offline scripts, never in the request path. Churn risk is a deterministic heuristic evaluated at read time.

---

## ✨ Features

| Area | What you get |
|---|---|
| 📊 **Dashboard** | KPI cards (customers, orders, revenue, campaigns, delivery / open / click rates), a 30-day revenue trend, customers-by-city chart, per-campaign funnel with a campaign picker, top-10 customers, a real **recent activity** feed built with `UNION ALL`, and a sample-feedback panel. |
| 💬 **Ask Your Data** | Natural-language analytics. LLM → validated JSON spec → parameterized read-only SQL → short summary and a results table. |
| 👥 **Customers** | Client-side search by name, email, or city. A detail panel shows contact info, totals, last order, and a **VIP / At Risk / Active / Unscored** health badge. |
| 🧩 **Segments** | Saved segments with strict filter validation, an **AI segment assistant** that proposes filters from a prompt, and **discovered RFM clusters** that convert into segments with one click. Deleting a segment that a campaign uses is blocked (409). |
| 📣 **Campaigns** | Create a campaign for a segment over email, SMS, or WhatsApp (labels only). **AI-drafted copy**, a stats drawer fetched on open, and a status badge that understands `processing`, `sent`, `partially_failed`, and `failed`. |
| 📡 **Delivery tracking** | A separate channel service simulates *sent → delivered / failed → opened → clicked* and reports back through an authenticated `/receipt` callback. |
| 🔐 **Authentication** | Clerk sign-in in the browser. FastAPI verifies the RS256 session JWT (signature, expiry, issuer, authorized party) on 17 routes. |
| 📱 **Responsive shell** | Collapsible desktop sidebar, slide-over mobile navigation, loading skeletons, empty states, and error states on every page. |

---

## 🧱 System Architecture

```mermaid
flowchart TB
    User(["👤 Marketing user"])

    subgraph FE["🌐 Vercel · React 18 + Vite + Tailwind"]
        direction TB
        Pages["pages/<br/>Dashboard · Customers · Segments · Campaigns"]
        Hooks["hooks/ → services/"]
        Axios["Axios client<br/>Authorization: Bearer JWT"]
        Pages --> Hooks --> Axios
    end

    Clerk["🔐 Clerk<br/>sign-in · session JWT · JWKS"]

    subgraph CRM["🐍 Render · CRM API · FastAPI · backend/"]
        direction TB
        CORS["CORS allow-list<br/>ALLOWED_ORIGINS"]
        Auth{{"core/auth.py<br/>verify_clerk_token<br/>RS256 · issuer · azp"}}
        Routers["routers/<br/>customers · orders · segments · campaigns<br/>dashboard · ai · analytics · receipts · root"]
        Svc["services/<br/>sql_guard · segment_filters<br/>campaign_processor · churn_risk"]
        Cli["clients/<br/>ai_client · channel_client"]
        Pool["core/database.py<br/>asyncpg pool 1 to 10"]
        CORS --> Auth -->|valid token| Routers
        Routers --> Svc
        Routers --> Pool
        Svc --> Cli
        Svc --> Pool
    end

    subgraph CS["📨 Channel service · FastAPI · channel-service/"]
        Sim["Simulated delivery lifecycle<br/>shared-secret on /send"]
    end

    Scripts["📜 scripts/ + models/*.pkl<br/>offline RFM training and scoring"]
    DB[("🐘 PostgreSQL · Neon")]
    Groq["🧠 Groq API<br/>gpt-oss-120b"]

    User --> FE
    FE <-->|sign-in and session token| Clerk
    Axios -->|HTTPS + Bearer JWT| CORS
    Auth -.->|JWKS signing keys| Clerk
    Pool <--> DB
    Cli <-->|JSON mode prompts| Groq
    Cli -->|POST /send + X-Channel-Service-Key| Sim
    Sim -->|POST /receipt + X-Channel-Service-Key| Routers
    Scripts -->|writes churn_score and cluster_id| DB

    classDef client fill:#1e3a5f,stroke:#60a5fa,color:#fff
    classDef api fill:#4a2c1d,stroke:#c8852e,color:#fff
    classDef data fill:#14532d,stroke:#4ade80,color:#fff
    classDef sec fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef ext fill:#1f2937,stroke:#9ca3af,color:#fff
    classDef ml fill:#581c87,stroke:#c084fc,color:#fff
    class Pages,Hooks,Axios client
    class CORS,Routers,Svc,Cli,Pool,Sim api
    class DB data
    class Auth sec
    class Clerk,Groq ext
    class Scripts ml
```

| Component | Responsibility |
|---|---|
| **Frontend** | Four routed pages behind Clerk's signed-in gate. Hooks wrap per-resource fetching, services wrap Axios, and an interceptor attaches the Clerk session token. |
| **CRM API** | Auth, validation, persistence, analytics, and campaign orchestration. Every route except `/` and `/receipt` requires a Clerk JWT. |
| **Channel service** | Pretends to be an email, SMS, or WhatsApp provider. It requires `X-Channel-Service-Key` on `/send` and sends the same key back on `/receipt`. |
| **Offline scripts** | Train the RFM model, assign cluster ids, and optionally persist churn scores. They are run by hand, never from a request. |
| **PostgreSQL (Neon)** | System of record. The analytics path uses a read-only transaction. |
| **Groq** | Four narrow JSON-mode prompts. No customer records are sent. See [AI and PII Boundary](#-ai-and-pii-boundary). |

### One authenticated request, end to end

```mermaid
sequenceDiagram
    autonumber
    participant B as Browser (React)
    participant C as Clerk
    participant A as FastAPI
    participant P as asyncpg pool
    participant D as PostgreSQL

    B->>C: window.Clerk.session.getToken()
    C-->>B: short-lived RS256 JWT
    B->>A: GET /customers with Bearer JWT
    A->>A: CORS check, then verify_clerk_token
    A->>P: acquire connection
    P->>D: parameterized query
    D-->>P: rows
    A->>A: churn risk computed per row
    A->>P: release connection
    A-->>B: 200 JSON
    Note over B,A: Auth failures return 401 before any database access
```

### Product flow

```mermaid
flowchart LR
    A["📥 Customers and orders<br/>bulk import or seed"] --> B["🧮 Read-time churn risk<br/>+ offline RFM clusters"]
    B --> C["🧩 Segments<br/>AI-suggested or RFM-discovered"]
    C --> D["📣 Campaign<br/>AI-drafted copy"]
    D --> E["📨 Channel service<br/>simulated delivery"]
    E -->|authenticated receipts| F["📡 Delivery stats<br/>sent · delivered · opened · clicked"]
    F --> G["📊 Dashboard funnel"]
    G -.->|insights guide the next campaign| C
    H["💬 Ask Your Data"] -.-> G

    classDef n fill:#4a2c1d,stroke:#c8852e,color:#fff
    class A,B,C,D,E,F,G,H n
```


---

## 🔐 Authentication Flow

Clerk handles sign-in in the browser. The CRM API trusts nothing except a correctly signed, unexpired Clerk session JWT that was issued by the configured Clerk instance **and** minted for an allow-listed origin.

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant FE as React app
    participant C as Clerk
    participant API as FastAPI
    participant J as Clerk JWKS endpoint

    U->>FE: open any page
    FE->>C: is there a session?
    alt signed out
        FE-->>U: redirect to Clerk sign-in
        U->>C: authenticate
        C-->>FE: session established
    end
    FE->>C: session.getToken()
    C-->>FE: RS256 JWT
    FE->>API: request with Bearer JWT
    API->>J: PyJWKClient fetches the signing key for the token kid
    J-->>API: public key (cached by PyJWKClient)
    API->>API: verify signature, exp, issuer
    API->>API: check azp against CLERK_AUTHORIZED_PARTIES
    alt every check passes
        API-->>FE: 200 response
    else any check fails
        API-->>FE: 401 Unauthorized
    end
```

### What `verify_clerk_token` decides

```mermaid
flowchart TD
    Start(["Request reaches a protected route"]) --> H{"Authorization header present?"}
    H -- no --> E1["401 Missing Authorization header"]
    H -- yes --> B{"Starts with Bearer?"}
    B -- no --> E2["401 Invalid Authorization header"]
    B -- yes --> I{"CLERK_ISSUER configured?"}
    I -- no --> E3["500 Server misconfiguration<br/>fail closed"]
    I -- yes --> S["PyJWKClient resolves the signing key by kid"]
    S --> D{"jwt.decode with RS256<br/>signature · exp · issuer"}
    D -- invalid --> E4["401 Invalid or expired Clerk token"]
    D -- valid --> Z{"CLERK_AUTHORIZED_PARTIES set?"}
    Z -- no --> OK
    Z -- yes --> A{"token azp in the allow-list?"}
    A -- no --> E5["401 Authorized party not allowed"]
    A -- yes --> OK["✅ payload returned · route runs"]

    classDef bad fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef good fill:#14532d,stroke:#4ade80,color:#fff
    classDef step fill:#1f2937,stroke:#9ca3af,color:#fff
    class E1,E2,E3,E4,E5 bad
    class OK good
    class Start,H,B,I,S,D,Z,A step
```

| Design choice | Why |
|---|---|
| **Issuer is mandatory** | A token from *any* Clerk instance would otherwise validate. The app refuses to boot without `CLERK_ISSUER`, and the dependency also fails closed. |
| **`azp` is checked** | Binds a token to the front-end origin that requested it. `CLERK_AUTHORIZED_PARTIES` defaults to `ALLOWED_ORIGINS`. |
| **Audience check is off** (`verify_aud=False`) | Clerk session tokens carry no per-service audience. Issuer plus `azp` plus signature carry the trust instead. |
| **No per-user authorization** | The token payload is not used to scope data. Every signed-in user sees every record. This is a documented single-tenant design, see [Known Limitations](#-known-limitations). |
| **`/receipt` is not a Clerk route** | The channel service is a machine caller, so it authenticates with a shared secret instead. See [Campaign Delivery Pipeline](#-campaign-delivery-pipeline). |

> ⚠️ **Heads-up for local setup:** an empty `CLERK_AUTHORIZED_PARTIES=` line in `.env` parses to an empty allow-list, which **disables** the `azp` check. Delete the line or give it a value. In production the app refuses to start without it.

---

## 💬 Ask Your Data

A plain-English question becomes an answer without ever letting a language model write SQL. The model produces a **JSON query spec** from a closed vocabulary, and the backend turns that spec into a parameterized query that runs inside a read-only transaction.

```mermaid
flowchart TD
    Q["💬 'Which city drives the most revenue?'"] --> G1["🧠 Groq call 1<br/>generate_analytics_query_spec<br/>JSON-object mode"]
    G1 --> SPEC["📋 Candidate JSON spec"]
    SPEC --> V{"Pydantic QuerySpec<br/>Literal tables · columns · aggregates · operators"}
    V -- "unknown name or shape" --> R400["❌ 400 invalid spec"]
    V -- ok --> AL{"Per-table column allow-list<br/>PII and churn_score are not selectable"}
    AL -- "disallowed column" --> R400
    AL -- ok --> J{"Fixed join map<br/>between consecutive tables"}
    J -- "unsupported join" --> R400
    J -- ok --> B["🔧 build_analytics_query<br/>values become $1 $2 parameters"]
    B --> RO["🔒 asyncpg read-only transaction<br/>LIMIT capped at 100"]
    RO --> ROWS["📄 Result rows"]
    ROWS --> PII["🧼 PII sanitizer<br/>drops name · email · phone keys"]
    PII --> G2["🧠 Groq call 2<br/>generate_sql_summary"]
    G2 --> OUT["✅ sql + results + summary"]

    classDef llm fill:#581c87,stroke:#c084fc,color:#fff
    classDef guard fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef ok fill:#14532d,stroke:#4ade80,color:#fff
    classDef step fill:#1f2937,stroke:#9ca3af,color:#fff
    class G1,G2 llm
    class V,AL,J,PII,RO guard
    class OUT ok
    class Q,SPEC,B,ROWS,R400 step
```

### Six independent controls on one path

```mermaid
flowchart LR
    L1["1️⃣ Closed vocabulary<br/>Literal types"] --> L2["2️⃣ Column allow-list<br/>per table"] --> L3["3️⃣ Fixed join map"] --> L4["4️⃣ Bound parameters<br/>values never in SQL text"] --> L5["5️⃣ Read-only<br/>transaction"] --> L6["6️⃣ LIMIT at most 100<br/>+ PII stripped before summary"]
    classDef g fill:#7f1d1d,stroke:#f87171,color:#fff
    class L1,L2,L3,L4,L5,L6 g
```

| Layer | What it blocks |
|---|---|
| **Closed vocabulary** | Anything not in the `Literal` lists for tables, columns, aggregates, operators, and directions fails validation. There is no free-text field that reaches SQL. |
| **Column allow-list** | Identifiers come only from `VALID_TABLE_COLUMNS`, never from model output. `customers.name`, `email`, `phone`, and `churn_score` are not on it. |
| **Fixed join map** | Joins are limited to `customers ↔ orders`, `segments ↔ campaigns`, `campaigns ↔ communications`, and `customers ↔ communications`. Any other pair is rejected. |
| **Bound parameters** | Filter values are emitted as `$1`, `$2`, and so on. A value such as `Delhi'; DROP TABLE customers; --` is just a string argument. |
| **Read-only transaction** | Even a bug in the builder could not write, because the transaction itself is read-only. |
| **Row cap and PII sanitizer** | `limit` must be at most 100. Before the summary call, rows are filtered again for `name`, `email`, and `phone` keys. |

### Allow-listed columns

| Table | Selectable columns |
|---|---|
| `customers` | `id` · `city` · `total_orders` · `total_spent` · `last_order_date` · `cluster_id` · `created_at` |
| `orders` | `id` · `customer_id` · `amount` · `created_at` |
| `segments` | `id` · `name` · `customer_count` · `created_at` |
| `campaigns` | `id` · `name` · `segment_id` · `channel` · `status` · `created_at` |
| `communications` | `campaign_id` · `customer_id` · `status` · `sent_at` · `delivered_at` · `opened_at` · `clicked_at` |

Operators: `=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`, `ILIKE`, `IS NULL`, `IS NOT NULL`. Aggregates: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.

### Worked examples (generated by the real `build_analytics_query`)

**1. "Which city has the highest total revenue?"**

```jsonc
// What the LLM returns
{
  "tables": ["customers", "orders"],
  "select": [
    { "table": "customers", "column": "city" },
    { "table": "orders", "column": "amount", "agg": "SUM" }
  ],
  "group_by": [{ "table": "customers", "column": "city" }],
  "order_by": [{ "table": "orders", "column": "amount", "agg": "SUM", "direction": "DESC" }],
  "limit": 1
}
```

```sql
-- What the backend builds
SELECT customers.city, SUM(orders.amount)
FROM customers JOIN orders ON customers.id = orders.customer_id
GROUP BY customers.city
ORDER BY SUM(orders.amount) DESC
LIMIT 1
```

**2. "Top 5 cities by number of orders worth at least 500"** keeps the number out of the SQL text:

```sql
SELECT customers.city, COUNT(orders.id)
FROM customers JOIN orders ON customers.id = orders.customer_id
WHERE orders.amount >= $1
GROUP BY customers.city
ORDER BY COUNT(orders.id) DESC
LIMIT 5
-- params: [500]
```

**What gets refused**

| Attempt | Result |
|---|---|
| Select `customers.email` | ❌ `400` validation error: not in the `Literal` / allow-list |
| `limit: 5000` | ❌ `400` validation error: limit must be at most 100 |
| Tables `orders` + `segments` | ❌ `400` "Unsupported join between orders and segments" |
| Filter value `Delhi'; DROP TABLE customers; --` | ✅ accepted, but only as parameter `$1`. It is data, never SQL. |

### Response shape

```json
{
  "sql": "SELECT customers.city, SUM(orders.amount) FROM ...",
  "results": [{ "city": "<city>", "sum": 0.0 }],
  "summary": "A one-sentence answer written by the model"
}
```

Values above are placeholders. The failure modes are deliberately distinct:

| Situation | Response |
|---|---|
| Groq fails while writing the spec | `500` "Failed to generate query spec from AI." |
| Spec fails validation or the join is unsupported | `400` "Generated query is invalid or unsafe: ..." |
| Postgres rejects the query | `400` "Database execution error: ..." |
| Query returns no rows | `200` with `"summary": "No results found."` |
| Only the summary call fails | `200` with the rows and `"summary": "Summary generation failed."` |

---

## 🔏 AI and PII Boundary

```mermaid
flowchart LR
    subgraph Inside["🏠 Your infrastructure"]
        direction TB
        DB[("PostgreSQL<br/>names · emails · phones")]
        UI["Customers page<br/>signed-in users only"]
        Guard["Allow-list excludes<br/>name · email · phone · churn_score"]
        San["PII sanitizer"]
    end
    subgraph Outside["🌍 Groq"]
        LLM["gpt-oss-120b"]
    end

    DB --> UI
    DB --> Guard --> San
    San -->|"question + schema text<br/>+ allow-listed rows only"| LLM
    Prompts["Segment and message prompts<br/>user text only"] --> LLM

    classDef a fill:#14532d,stroke:#4ade80,color:#fff
    classDef b fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef c fill:#581c87,stroke:#c084fc,color:#fff
    class DB,UI a
    class Guard,San b
    class LLM,Prompts c
```

| Groq call | Triggered by | What leaves the system | Customer records? |
|---|---|---|---|
| `generate_segment_filter` | `POST /ai/suggest-segment` | The user's prompt plus fixed instructions | None |
| `generate_campaign_message` | `POST /ai/draft-message` | The campaign goal plus fixed instructions | None |
| `generate_analytics_query_spec` | `POST /analytics/query` | The question plus schema text | None |
| `generate_sql_summary` | `POST /analytics/query` | The question plus allow-listed result rows | No name, email, or phone keys |

The Customers page does show names, emails, and phone numbers to any signed-in user, because `GET /customers` returns full rows. The boundary above is about what reaches the **language model**.


---

## 🧠 Churn Risk and RFM Segmentation

Two different tools answer two different questions. **RFM clustering** discovers *groups* of similar customers. A **churn-risk heuristic** scores *each* customer's behavioural risk. They run on different schedules and neither sits behind a trained classifier.

```mermaid
flowchart TD
    A[("customers<br/>total_orders · total_spent<br/>last_order_date · created_at")]

    subgraph Read["⚡ Read-time path · every GET /customers"]
        direction TB
        R1["services/churn_risk.py<br/>calculate_churn_risk"] --> R2["churn_score overwritten in the response"]
        R2 --> R3["utils/customerHealth.js<br/>VIP · At Risk · Active · Unscored"]
    end

    subgraph Offline["📜 Offline path · scripts run by hand"]
        direction TB
        O1["train_rfm_clusters.py<br/>days since order · orders · spend<br/>StandardScaler then KMeans<br/>k from 2 to 8 by silhouette"] --> O2["models/rfm_model.pkl<br/>models/rfm_scaler.pkl"]
        O2 --> O3["update_customer_clusters.py<br/>scaler.transform then model.predict"]
        O3 --> O4["customers.cluster_id"]
        P1["update_churn_scores.py<br/>optional: persist the heuristic"] --> P2["customers.churn_score<br/>not read by the API"]
    end

    A --> R1
    A --> O1
    A --> P1
    O4 --> S1["GET /segments/discovered<br/>cluster stats + request-time labels"]
    S1 --> S2["POST /segments/discovered/id/convert<br/>saves filter cluster_id"]
    S2 --> S3["📣 Campaign targeting"]

    classDef ml fill:#581c87,stroke:#c084fc,color:#fff
    classDef api fill:#4a2c1d,stroke:#c8852e,color:#fff
    classDef data fill:#14532d,stroke:#4ade80,color:#fff
    classDef ui fill:#1e3a5f,stroke:#60a5fa,color:#fff
    class A data
    class R1,R2,S1,S2 api
    class R3,S3 ui
    class O1,O2,O3,O4,P1,P2 ml
```

### The churn-risk heuristic

```text
R = min(days_since_last_order / 90, 1)          # recency   (weight 0.50)
F = 1 / (1 + total_orders)                      # frequency (weight 0.30)
M = 1 / (1 + total_spent / 1000)                # monetary  (weight 0.20)

score = 0.50·R + 0.30·F + 0.20·M                # bounded to [0, 1], 4 decimals
```

```mermaid
pie showData title Churn-risk score weights
    "Recency (90-day horizon)" : 50
    "Frequency (habituation)" : 30
    "Monetary (1000 rupee scale)" : 20
```

| Edge case | Behaviour |
|---|---|
| Has orders but `last_order_date` is missing | `R = 1.0` (treated as dormant) |
| Zero orders, `created_at` known | `R = min(account_tenure_days / 90, 1)`, so a brand-new sign-up is not instantly "dormant" |
| Zero orders and no dates | `R = 1.0` |
| Future dates (clock skew) | Clamped to 0 days |
| NaN, negative, or non-numeric inputs | Cleaned to 0 |

**Tiers:** low is under 0.40, moderate is 0.40 to 0.70, high is above 0.70. The front end labels a customer **At Risk** at `score > 0.70`.

**Worked examples** (computed with the real function):

| Customer | Orders | Spent | Last order | R | F | M | Score | Badge |
|---|---:|---:|---|---:|---:|---:|---:|---|
| Loyal regular | 15 | ₹7,500 | 3 days ago | 0.03 | 0.06 | 0.12 | **0.06** | 🏆 VIP |
| Solid buyer | 6 | ₹3,000 | 3 weeks ago | 0.23 | 0.14 | 0.25 | **0.21** | 🟢 Active |
| Occasional | 3 | ₹900 | 2 months ago | 0.67 | 0.25 | 0.53 | **0.51** | 🟢 Active |
| One-off buyer | 1 | ₹150 | 4 months ago | 1.00 | 0.50 | 0.87 | **0.82** | 🔴 At Risk |
| High spender gone quiet | 8 | ₹6,200 | 130 days ago | 1.00 | 0.11 | 0.14 | **0.56** | 🏆 VIP |
| New sign-up, no orders | 0 | ₹0 | none | 0.11 | 1.00 | 1.00 | 0.56 | ⚪ Unscored |

### Health badge logic (front end)

```mermaid
flowchart TD
    S(["Customer row from GET /customers"]) --> V{"total_orders > 4<br/>AND total_spent > 5000?"}
    V -- yes --> VIP["🏆 VIP"]
    V -- no --> Z{"total_orders = 0?"}
    Z -- yes --> UN["⚪ Unscored"]
    Z -- no --> H{"churn_score > 0.70?"}
    H -- yes --> AR["🔴 At Risk"]
    H -- no --> AC["🟢 Active"]

    classDef a fill:#14532d,stroke:#4ade80,color:#fff
    classDef b fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef c fill:#92400e,stroke:#fbbf24,color:#fff
    classDef d fill:#1f2937,stroke:#9ca3af,color:#fff
    class AC a
    class AR b
    class VIP c
    class UN,S,V,Z,H d
```

> **Why a heuristic and not a trained model?** The dataset has no observed churn events such as cancellations or deactivations. Fitting a logistic regression to labels derived from the same recency, frequency, and spend features is circular: it would "learn" the rule that produced the labels and report a flattering accuracy. The earlier classifier was retired, and `train_churn_model.py` now just prints a retirement notice. The 90-day horizon, the 50/30/20 weights, and the ₹1,000 scale are **chosen parameters, not calibrated ones**. The score is a relative risk index, not a probability.
>
> **Design trade-off to know about:** the **VIP rule wins over risk**. A high spender who has been silent for months still shows as VIP, as the "gone quiet" row above shows.

### RFM clustering

| Step | Detail |
|---|---|
| **Features** | `[days since last order, total_orders, total_spent]` (999 days if the customer never ordered) |
| **Scaling** | `StandardScaler` |
| **Model** | `KMeans(random_state=42, n_init=10)`, trying k = 2 to 8 and keeping the best silhouette score |
| **Artifacts** | `models/rfm_model.pkl` and `models/rfm_scaler.pkl` are committed, so the two training scripts are optional on a fresh clone. The committed model has **k = 3**. |
| **Assignment** | `update_customer_clusters.py` loads both pickles and writes `customers.cluster_id` |
| **Serving** | `GET /segments/discovered` aggregates per-cluster averages and applies labels **at request time** |

Cluster names are not stored. They come from a ranking applied to whatever clusters exist:

```mermaid
flowchart LR
    C["All clusters"] --> L1["① highest avg orders × avg spend<br/>→ Loyal High-Value"]
    L1 --> L2["② of the rest, highest avg recency<br/>→ At Risk"]
    L2 --> L3["③ of the rest, fewest avg orders<br/>→ New/Occasional"]
    L3 --> L4["④ of the rest, highest avg spend<br/>→ High Spenders"]
    L4 --> L5["⑤ remainder<br/>→ Standard Group or Standard Group N"]

    classDef n fill:#581c87,stroke:#c084fc,color:#fff
    class C,L1,L2,L3,L4,L5 n
```

With the committed k = 3 model, only the first three labels can appear.

| Caveat | Why it matters |
|---|---|
| **Cluster ids are not stable across retrains** | Re-running training can renumber clusters. A converted segment stores `{"cluster_id": n}`, so it could later match a different group. |
| **Labels are relative** | The cluster with the highest recency is named *At Risk* even if no cluster is truly at risk. |
| **The scaler is frozen at training time** | Recency is measured when the scripts run. As data ages, retrain before re-assigning clusters. |

---

## 📣 Campaign Delivery Pipeline

Creating a campaign returns immediately. Sending happens in a FastAPI background task, and delivery outcomes come back later through an authenticated callback.

```mermaid
sequenceDiagram
    autonumber
    participant FE as Frontend
    participant API as CRM API · POST /campaigns
    participant BG as process_campaign · BackgroundTask
    participant DB as PostgreSQL
    participant CS as Channel service · simulator
    participant RC as CRM · POST /receipt

    FE->>API: name, segment_id, channel, message
    API->>DB: check the segment exists, INSERT campaign as processing
    API-->>FE: 200 with campaign_id
    API->>BG: schedule background task
    BG->>DB: load segment, re-validate filter_json, select customers
    alt no customers match
        BG->>DB: UPDATE campaign status failed
    else customers found
        BG->>DB: INSERT one communication per customer as processing
        loop every communication, started concurrently
            BG->>CS: POST /send with X-Channel-Service-Key
            CS-->>BG: 200 queued
            BG->>DB: set sent_at and status sent, unless a receipt already landed
        end
        BG->>DB: UPDATE campaign to sent, partially_failed, or failed
    end
    loop per message, asynchronously
        CS->>CS: sleep 2 to 5 seconds, then roll the dice
        CS->>RC: POST /receipt delivered or failed, then opened, then clicked
        RC->>DB: monotonic status update plus timestamps
    end
    FE->>API: GET /campaigns/{id}/stats
    API-->>FE: total, sent, delivered, opened, clicked, failed
```

### Campaign status

```mermaid
stateDiagram-v2
    [*] --> processing: POST /campaigns
    processing --> sent: every dispatch accepted
    processing --> partially_failed: some accepted and some rejected
    processing --> failed: all rejected, empty segment, or processor error
    sent --> [*]
    partially_failed --> [*]
    failed --> [*]
```

> `sent` means *handed to the channel service*, not *delivered*. Real delivery is tracked per message in the funnel.

### Per-message status

```mermaid
stateDiagram-v2
    [*] --> processing: row inserted before dispatch
    processing --> sent: channel service accepted
    processing --> failed: dispatch rejected
    sent --> failed: failed receipt
    sent --> delivered: delivered receipt
    processing --> delivered: receipt beats the dispatch update
    delivered --> opened: opened receipt
    sent --> opened: opened receipt backfills delivered_at
    opened --> clicked: clicked receipt
    delivered --> clicked: clicked receipt backfills opened_at
    clicked --> [*]
    failed --> [*]
```

| Rule enforced in SQL | Effect |
|---|---|
| Dispatch result uses `CASE WHEN status IN (delivered, opened, clicked, failed)` | A slow dispatch update never overwrites a receipt that already arrived. |
| A `failed` receipt only applies when status is **not** delivered, opened, or clicked | A confirmed delivery cannot be downgraded. |
| A `delivered` receipt keeps `opened` and `clicked` | Late receipts cannot move a row backwards. |
| `opened` and `clicked` receipts `COALESCE` earlier timestamps | An out-of-order click still produces a complete timeline. |

### What the channel service simulates

```mermaid
flowchart LR
    S["POST /send<br/>X-Channel-Service-Key required"] --> W1["sleep 2 to 5 s"]
    W1 --> R1{"random below 0.80?"}
    R1 -- "20%" --> F["receipt: failed"]
    R1 -- "80%" --> D["receipt: delivered"]
    D --> W2["sleep 2 to 5 s"] --> R2{"random below 0.60?"}
    R2 -- "40%" --> N1["stop: not opened"]
    R2 -- "60%" --> O["receipt: opened"]
    O --> W3["sleep 2 to 5 s"] --> R3{"random below 0.30?"}
    R3 -- "70%" --> N2["stop: not clicked"]
    R3 -- "30%" --> K["receipt: clicked"]

    classDef bad fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef good fill:#14532d,stroke:#4ade80,color:#fff
    classDef step fill:#1f2937,stroke:#9ca3af,color:#fff
    class F bad
    class D,O,K good
    class S,W1,W2,W3,R1,R2,R3,N1,N2 step
```

Expected funnel for every 100 messages (expected values, not guarantees):

| Sent | Delivered | Opened | Clicked | Failed |
|---:|---:|---:|---:|---:|
| 100 | 80 | 48 | 14.4 | 20 |

### The callback boundary

`POST /receipt` is the one CRM route that does **not** use a Clerk token, because its caller is the channel service rather than a person. It authenticates with the shared secret and compares it in constant time.

| Condition | Response |
|---|---|
| `X-Channel-Service-Key` header absent | `401` Missing credential |
| Header present but wrong (`hmac.compare_digest`) | `403` Invalid credential |
| Server secret not set | `500` Server misconfiguration |
| Status is not `delivered`, `failed`, `opened`, or `clicked` | `400` Invalid status |
| All checks pass | `200`, row updated |

The same secret protects `/send` on the channel service (`401` absent, `403` wrong, `500` unset). The CRM no longer sends a callback URL in the `/send` payload. The channel service reads its own `CRM_RECEIPT_URL`, so a caller cannot redirect receipts.

---

## 💾 Data Model

Five tables, plus two nullable columns added by offline scripts.

```mermaid
erDiagram
    CUSTOMERS ||--o{ ORDERS : "places"
    CUSTOMERS ||--o{ COMMUNICATIONS : "receives"
    SEGMENTS ||--o{ CAMPAIGNS : "targeted by"
    CAMPAIGNS ||--o{ COMMUNICATIONS : "sends"

    CUSTOMERS {
        serial id PK
        varchar name "NOT NULL"
        varchar email UK "NOT NULL"
        varchar phone
        varchar city
        int total_orders "DEFAULT 0"
        numeric total_spent "DEFAULT 0"
        timestamp last_order_date
        float churn_score "DEFAULT 0.0 · written by script"
        int cluster_id "written by script"
        timestamp created_at
    }
    ORDERS {
        serial id PK
        int customer_id FK "ON DELETE CASCADE"
        numeric amount "NOT NULL"
        jsonb items "NOT NULL"
        timestamp created_at "may be backdated on import"
    }
    SEGMENTS {
        serial id PK
        varchar name "NOT NULL"
        text description
        jsonb filter_json "NOT NULL"
        int customer_count "cached at creation"
        timestamp created_at
    }
    CAMPAIGNS {
        serial id PK
        varchar name "NOT NULL"
        int segment_id FK "ON DELETE CASCADE"
        text message "NOT NULL"
        varchar channel "NOT NULL"
        varchar status "processing sent partially_failed failed"
        timestamp created_at
    }
    COMMUNICATIONS {
        serial id PK
        int campaign_id FK "ON DELETE CASCADE"
        int customer_id FK "ON DELETE CASCADE"
        varchar status "processing sent delivered opened clicked failed"
        timestamp sent_at
        timestamp delivered_at
        timestamp opened_at
        timestamp clicked_at
    }
```

### What deletion does

```mermaid
flowchart LR
    DC["🗑️ Delete customer"] --> O["orders removed"]
    DC --> CM["communications removed"]
    DS["🗑️ Delete segment via API"] --> G{"any campaign uses it?"}
    G -- yes --> X["409 Conflict, nothing deleted"]
    G -- no --> OK["segment removed"]
    DK["🗑️ Delete campaign via API"] --> T["transaction: communications first, then the campaign"]
    SQL["⚠️ DELETE segment directly in SQL"] -.->|ON DELETE CASCADE| CP["campaigns then communications removed"]

    classDef bad fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef good fill:#14532d,stroke:#4ade80,color:#fff
    classDef step fill:#1f2937,stroke:#9ca3af,color:#fff
    class X,SQL bad
    class OK,T good
    class DC,O,CM,DS,G,DK,CP step
```

The 409 guard is **application-level only**. The database itself cascades, so deleting a segment with raw SQL also removes its campaigns and their communications.

| Table | Notes |
|---|---|
| `customers` | `email` is `UNIQUE NOT NULL`, which makes bulk import idempotent through `ON CONFLICT`. `total_orders`, `total_spent`, and `last_order_date` are denormalized and kept in sync by bulk order insert (`last_order_date` uses `GREATEST`). |
| `orders` | `items` is `JSONB` for flexible line items. `created_at` may be backdated on import, so charts reflect real order dates. |
| `segments` | Stores the filter as `JSONB`. `customer_count` is a snapshot taken at creation and is **not refreshed** later. |
| `campaigns` | `channel` is a label (email, SMS, WhatsApp). The schema default is `pending`, but the API always writes `processing` first. |
| `communications` | One row per campaign and customer in practice. The schema does **not** enforce uniqueness of that pair. |

**Indexes (10):** `customers(email)`, `customers(city)`, `orders(customer_id)`, `orders(created_at)`, `segments(name)`, `campaigns(segment_id)`, `campaigns(status)`, `communications(campaign_id)`, `communications(customer_id)`, `communications(status)`.


---

## 🔌 API Endpoints

FastAPI also serves interactive Swagger docs at `/docs` on any running instance. Authentication is a Clerk session JWT in `Authorization: Bearer <token>` unless noted.

### CRM API (19 endpoints, 17 behind Clerk)

| Group | Method | Endpoint | Purpose | Auth |
|---|---|---|---|---|
| Service | `GET` | `/` | Service status and version | none |
| Receipts | `POST` | `/receipt` | Delivery callback from the channel service | `X-Channel-Service-Key` |
| Customers | `POST` | `/customers/bulk` | Bulk insert in one transaction, skipping duplicate emails | Clerk |
| Customers | `GET` | `/customers` | List customers with live churn score. Optional `city`, `min_spent`, `max_spent`, `min_orders` | Clerk |
| Orders | `POST` | `/orders/bulk` | Bulk insert (may be backdated). Updates customer totals, rolls back on an unknown customer | Clerk |
| Segments | `POST` | `/segments` | Create from a validated `filter_json` | Clerk |
| Segments | `GET` | `/segments` | List saved segments | Clerk |
| Segments | `DELETE` | `/segments/{id}` | Delete (`404` if missing, `409` if a campaign uses it) | Clerk |
| Segments | `GET` | `/segments/discovered` | RFM clusters with averages and request-time labels | Clerk |
| Segments | `POST` | `/segments/discovered/{cluster_id}/convert` | Save a cluster as a segment | Clerk |
| Campaigns | `POST` | `/campaigns` | Create, then dispatch in the background | Clerk |
| Campaigns | `GET` | `/campaigns` | List with segment name | Clerk |
| Campaigns | `DELETE` | `/campaigns/{id}` | Delete the campaign and its communications in one transaction | Clerk |
| Campaigns | `GET` | `/campaigns/{id}/stats` | Funnel counts for one campaign | Clerk |
| Dashboard | `GET` | `/dashboard/stats` | KPIs, rates, recent campaigns, recent activity | Clerk |
| Dashboard | `GET` | `/dashboard/revenue-trend` | Daily revenue for the last 30 days | Clerk |
| AI | `POST` | `/ai/suggest-segment` | Prompt → suggested `filter_json` (validated when saved) | Clerk |
| AI | `POST` | `/ai/draft-message` | Campaign goal → drafted message | Clerk |
| Analytics | `POST` | `/analytics/query` | Question → `{ sql, results, summary }` | Clerk |

### Channel service (2 endpoints)

| Method | Endpoint | Purpose | Auth |
|---|---|---|---|
| `GET` | `/` | Service status | none |
| `POST` | `/send` | Accept a message and simulate its lifecycle | `X-Channel-Service-Key` |

<details>
<summary><b>📎 Request and response examples</b></summary>

<br/>

**Create a campaign**

```bash
curl -X POST "$API/campaigns" \
  -H "Authorization: Bearer $CLERK_JWT" \
  -H "Content-Type: application/json" \
  -d '{
        "name": "Cold brew comeback",
        "segment_id": 3,
        "channel": "email",
        "message": "We saved a cold brew for you."
      }'
# → { "success": true, "campaign_id": 12, "status": "processing" }
```

**Poll its funnel**

```bash
curl "$API/campaigns/12/stats" -H "Authorization: Bearer $CLERK_JWT"
# → { "campaign_id": 12, "total": 32, "sent": 32, "delivered": 26,
#     "opened": 15, "clicked": 4, "failed": 6 }
```

**Ask a question**

```bash
curl -X POST "$API/analytics/query" \
  -H "Authorization: Bearer $CLERK_JWT" \
  -H "Content-Type: application/json" \
  -d '{ "question": "Which city has the highest total revenue?" }'
# → { "sql": "SELECT customers.city, SUM(orders.amount) ...",
#     "results": [ ... ], "summary": "..." }
```

**Bulk import customers** (the live demo database can only be filled this way, because the seed script refuses remote hosts)

```bash
curl -X POST "$API/customers/bulk" \
  -H "Authorization: Bearer $CLERK_JWT" \
  -H "Content-Type: application/json" \
  -d '{ "customers": [
        { "name": "Asha Rao", "email": "asha@example.com", "phone": "9000000001", "city": "Pune" }
      ] }'
# → { "success": true, "inserted": 1, "skipped": 0, "attempted": 1 }
```

**Simulate a receipt** (what the channel service sends back)

```bash
curl -X POST "$API/receipt" \
  -H "X-Channel-Service-Key: $CHANNEL_SERVICE_SECRET" \
  -H "Content-Type: application/json" \
  -d '{ "campaign_id": 12, "customer_id": 7, "status": "delivered" }'
```

</details>

**Status codes you will see:** `400` invalid or unsafe input · `401` missing or invalid credential · `403` wrong shared secret · `404` unknown segment or campaign · `409` segment still used by a campaign · `422` request body fails Pydantic validation · `500` server or upstream failure.

---

## 🔒 Security Hardening

### Trust boundaries

```mermaid
flowchart LR
    subgraph Z1["🌍 Untrusted · the internet"]
        U["Browser and React app"]
    end
    subgraph Z2["🔐 Identity provider"]
        K["Clerk"]
    end
    subgraph Z3["🏠 Application tier"]
        A["CRM API"]
        C["Channel service"]
    end
    subgraph Z4["💾 Data tier"]
        D[("PostgreSQL")]
    end
    subgraph Z5["🧠 Third-party AI"]
        G["Groq"]
    end

    U -->|"① CORS allow-list<br/>② RS256 JWT: signature · exp · issuer · azp"| A
    U <-->|session token| K
    A -.->|JWKS| K
    A -->|"③ X-Channel-Service-Key on /send"| C
    C -->|"④ X-Channel-Service-Key on /receipt"| A
    A -->|"⑤ bound parameters<br/>read-only analytics transaction"| D
    A -->|"⑥ no customer records<br/>PII keys stripped"| G

    classDef z1 fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef z2 fill:#1f2937,stroke:#9ca3af,color:#fff
    classDef z3 fill:#4a2c1d,stroke:#c8852e,color:#fff
    classDef z4 fill:#14532d,stroke:#4ade80,color:#fff
    classDef z5 fill:#581c87,stroke:#c084fc,color:#fff
    class U z1
    class K z2
    class A,C z3
    class D z4
    class G z5
```

### Controls

| Risk | Control | Where |
|---|---|---|
| Forged, foreign, or replayed tokens | RS256 signature, `exp`, mandatory issuer, authorized-party (`azp`) check. Fails closed if the issuer is unset. | `core/auth.py` |
| Cross-origin abuse | CORS allow-list from `ALLOWED_ORIGINS`. Wildcards, missing schemes, and trailing slashes are rejected at boot. Production requires an explicit list. | `core/config.py`, `main.py` |
| Anonymous calls to the machine endpoints | `/send` and `/receipt` need `X-Channel-Service-Key`, compared with `hmac.compare_digest`. The CRM no longer sends a callback URL. | `channel-service/main.py`, `routers/receipts.py` |
| SQL injection through segments | `filter_json` is validated by a strict schema (unknown keys forbidden, negatives rejected, `min > max` rejected, empty rejected). Values are bound parameters. Filters are re-validated at send time. | `services/segment_filters.py` |
| SQL injection and data exfiltration through Ask Your Data | Closed-vocabulary spec, column allow-list, fixed joins, bound parameters, read-only transaction, `LIMIT ≤ 100`. | `schemas.py`, `services/sql_guard.py`, `routers/analytics.py` |
| PII reaching the LLM | Name, email, and phone are not selectable. Rows are sanitized again before the summary call. | `services/sql_guard.py`, `clients/ai_client.py` |
| False "success" reporting | Campaign status reflects real dispatch outcomes. Late dispatch updates never overwrite receipts. | `services/campaign_processor.py`, `routers/receipts.py` |
| Half-applied bulk imports | Customer and order imports run in one transaction and report accurate `inserted` and `skipped` counts. | `routers/customers.py`, `routers/orders.py` |
| Wiping the wrong database | Seed and reset scripts demand a development `APP_ENV`, a dedicated `DEV_DATABASE_URL`, a **loopback-only** host, and an explicit confirmation flag. They never print credentials. | `seed.py`, `scripts/reset_dev_db.py` |
| Missing secrets | The API will not start without `DATABASE_URL`, `GROQ_API_KEY`, `CLERK_JWKS_URL`, `CLERK_ISSUER`, or `CHANNEL_SERVICE_SECRET`. | `core/config.py` |

### Which test guards which risk

```mermaid
flowchart LR
    T1["🔐 Token and origin binding"] --> F1["tests_security_fix1.py<br/>26 tests"]
    T2["📨 Callback credentials"] --> F2["tests_receipt_security.py<br/>channel-service tests<br/>3 + 8 tests"]
    T3["💬 Query spec safety"] --> F3["tests_analytics_security.py<br/>13 cases"]
    T4["🔏 PII boundary"] --> F4["tests_analytics_pii_security.py"]
    T5["🧩 Segment filter safety"] --> F5["tests_segment_security.py<br/>14 cases"]
    T6["📣 Honest delivery reporting"] --> F6["tests_campaign_reporting_fix2.py<br/>22 tests"]
    T7["🧠 Churn methodology"] --> F7["tests_churn_methodology_fix3.py<br/>27 tests"]
    T8["📥 Bulk import integrity"] --> F8["tests_bulk_transactions_fix4.py<br/>13 tests"]
    T9["💾 Database workflow safety"] --> F9["tests_database_workflow_fix5.py<br/>23 tests"]
    T10["🚀 Deploy routing"] --> F10["tests_vercel_routing_fix6.py<br/>7 tests"]

    classDef r fill:#7f1d1d,stroke:#f87171,color:#fff
    classDef t fill:#14532d,stroke:#4ade80,color:#fff
    class T1,T2,T3,T4,T5,T6,T7,T8,T9,T10 r
    class F1,F2,F3,F4,F5,F6,F7,F8,F9,F10 t
```

---

## 🎨 Frontend

React 18 with Vite 6, Tailwind 3, React Router 7, Clerk for auth, Axios for HTTP, Recharts for charts, Sonner for toasts, and Lucide for icons.

```mermaid
flowchart LR
    subgraph Shell["🧱 Shell"]
        direction TB
        Main["main.jsx<br/>ClerkProvider"] --> App["App.jsx<br/>SignedIn gate + Router"]
        App --> Layout["AppLayout<br/>Sidebar · MobileSidebar · PageHeader"]
    end
    subgraph Pages["📄 Pages"]
        direction TB
        P1["Dashboard"]
        P2["Customers"]
        P3["Segments"]
        P4["Campaigns"]
    end
    subgraph Hooks["🪝 hooks/"]
        direction TB
        H1["useDashboard"]
        H2["useCustomers"]
        H3["useSegments<br/>useDiscoveredSegments"]
        H4["useCampaigns<br/>useCampaignStats"]
    end
    subgraph Svc["🔌 services/"]
        direction TB
        S0["apiClient.js<br/>Axios + token interceptor"]
        S1["dashboardApi · analyticsApi"]
        S2["customersApi"]
        S3["segmentsApi · aiApi"]
        S4["campaignsApi"]
    end

    Layout --> P1 & P2 & P3 & P4
    P1 --> H1 & H2 & H4
    P2 --> H2
    P3 --> H3
    P4 --> H4 & H3
    H1 --> S1
    H2 --> S2
    H3 --> S3
    H4 --> S4
    S1 & S2 & S3 & S4 --> S0
    S0 -->|Bearer JWT| API["🐍 CRM API"]

    classDef a fill:#1e3a5f,stroke:#60a5fa,color:#fff
    classDef b fill:#4a2c1d,stroke:#c8852e,color:#fff
    class Main,App,Layout,P1,P2,P3,P4 a
    class H1,H2,H3,H4,S0,S1,S2,S3,S4,API b
```

| Page | Route | Data it loads | Notable behaviour |
|---|---|---|---|
| **Dashboard** | `/` | `/dashboard/stats`, `/dashboard/revenue-trend`, `/customers`, `/campaigns`, `/campaigns/{id}/stats`, `/analytics/query` | KPI cards, Ask Your Data, city chart (top 8 of all cities), 30-day revenue trend, funnel with campaign picker, top-10 customers, recent activity. Revenue is formatted as INR. |
| **Customers** | `/customers` | `/customers` | Client-side search across name, email, and city. Selecting a row opens a detail panel. |
| **Segments** | `/segments` | `/segments`, `/segments/discovered`, `/ai/suggest-segment` | The create form needs an AI suggestion first, and the suggested filter is shown read-only. The form clears after a successful create. Cluster cards convert in one click. |
| **Campaigns** | `/campaigns` | `/campaigns`, `/segments`, `/ai/draft-message`, `/campaigns/{id}/stats` | Create form with an AI draft helper. Clicking a row opens a stats drawer, which fetches once on open. |

**Cross-cutting patterns**

| Pattern | Detail |
|---|---|
| **Auth gate** | Signed-out visitors are redirected to Clerk sign-in. |
| **Token injection** | An Axios request interceptor reads the session token from Clerk and sets the `Authorization` header. |
| **Per-resource hooks** | Each hook owns `data`, `loading`, `error`, and a refetch function, so pages stay declarative. |
| **UI states** | Shared `LoadingSkeleton`, `EmptyState`, `ErrorState`, `Drawer`, `MetricCard`, and `StatusBadge` components cover every page. |
| **Health badge** | `utils/customerHealth.js` mirrors the backend thresholds. See [Churn Risk and RFM Segmentation](#-churn-risk-and-rfm-segmentation). |
| **Deep links** | `frontend/vercel.json` rewrites unknown paths to the SPA entry so `/campaigns` survives a refresh. |

---

## 🧰 Backend and Repository Structure

```mermaid
flowchart LR
    main["main.py<br/>CORS · lifespan · routers"] --> routers
    main --> core
    subgraph core["core/"]
        cfg["config.py<br/>fail-fast env checks"]
        db["database.py<br/>asyncpg pool"]
        au["auth.py<br/>verify_clerk_token"]
    end
    subgraph routers["routers/ · 9 modules"]
        r1["customers · orders · segments · campaigns"]
        r2["dashboard · ai · analytics · receipts · root"]
    end
    subgraph services["services/"]
        s1["sql_guard"]
        s2["segment_filters"]
        s3["campaign_processor"]
        s4["churn_risk"]
    end
    subgraph clients["clients/"]
        c1["ai_client"]
        c2["channel_client"]
    end
    schemas["schemas.py<br/>Pydantic models + allow-lists"]

    routers --> au
    routers --> db
    routers --> schemas
    r1 --> s2 & s3 & s4
    r2 --> s1 & c1
    s3 --> c2
    s3 --> s2
    au --> cfg
    db --> cfg
    c1 --> cfg
    c2 --> cfg

    classDef a fill:#4a2c1d,stroke:#c8852e,color:#fff
    classDef b fill:#1e3a5f,stroke:#60a5fa,color:#fff
    classDef c fill:#14532d,stroke:#4ade80,color:#fff
    class cfg,db,au a
    class r1,r2 b
    class s1,s2,s3,s4,c1,c2,schemas c
```

```text
brewco-crm/
├── backend/                     CRM API (FastAPI)
│   ├── main.py                  app, CORS, lifespan, router registration
│   ├── schemas.py               Pydantic models and query-spec allow-lists
│   ├── schema.sql               canonical schema (includes churn_score and cluster_id)
│   ├── seed.py                  guarded synthetic data seeder
│   ├── core/                    config (fail-fast) · asyncpg pool · Clerk JWT dependency
│   ├── routers/                 customers · orders · segments · campaigns · dashboard
│   │                            ai · analytics · receipts · root
│   ├── services/                sql_guard · segment_filters · campaign_processor · churn_risk
│   ├── clients/                 ai_client (Groq) · channel_client (channel service)
│   ├── scripts/                 additive migrations · RFM training and scoring
│   │                            guarded dev reset (reset_dev_db.py / .sql)
│   ├── models/                  rfm_model.pkl · rfm_scaler.pkl
│   └── tests_*.py               11 regression suites
├── channel-service/             delivery simulator (FastAPI)
│   ├── main.py
│   └── tests_channel_service_security.py
├── frontend/                    React app (Vite)
│   └── src/                     pages · layout · components · hooks · services · utils · styles
└── Screenshots/                 images used in this README
```


---

## 🔧 Local Setup

**Prerequisites:** Python **3.11+** (verified on 3.12), Node.js **20+**, PostgreSQL, a [Groq](https://console.groq.com) API key, and a [Clerk](https://clerk.com) application.

```mermaid
flowchart LR
    A["① Clone and install<br/>venv + requirements"] --> B["② Create the schema<br/>psql -f schema.sql"]
    B --> C["③ Fill the three .env files"]
    C --> D["④ Seed demo data<br/>guarded, local only"]
    D --> E["⑤ Optional ML scripts<br/>cluster ids + churn scores"]
    E --> F["⑥ Run three terminals<br/>API 8000 · channel 8001 · UI 5173"]

    classDef n fill:#4a2c1d,stroke:#c8852e,color:#fff
    class A,B,C,D,E,F n
```

### 1. Clone

```bash
git clone https://github.com/SakshamDhiman2202/brewco-crm.git
cd brewco-crm
```

### 2. Database

There are three workflows. Pick the one that matches your situation.

| Situation | Command | Notes |
|---|---|---|
| **Fresh database** | `psql "$DATABASE_URL" -f backend/schema.sql` | Idempotent (`IF NOT EXISTS`). Already includes `churn_score` and `cluster_id`. |
| **Older database from before the ML features** | `python backend/scripts/01_add_churn_score_column.py` then `python backend/scripts/02_add_cluster_id_column.py` | Additive migrations. They never drop data. |
| **Wipe and rebuild a local dev database** | `python backend/scripts/reset_dev_db.py --confirm-destructive-reset`, then re-apply `schema.sql` | ⚠️ Destructive. Guarded: needs a development `APP_ENV`, `DEV_DATABASE_URL` on a loopback host, and the confirmation flag. |

### 3. Environment files

```bash
cp backend/.env.example backend/.env
cp channel-service/.env.example channel-service/.env
cp frontend/.env.example frontend/.env
```

A working local `backend/.env` looks like this:

```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/brewco_dev
DEV_DATABASE_URL=postgresql://postgres:postgres@localhost:5432/brewco_dev
GROQ_API_KEY=gsk_your_key
CLERK_JWKS_URL=https://<your-instance>.clerk.accounts.dev/.well-known/jwks.json
CLERK_ISSUER=https://<your-instance>.clerk.accounts.dev
CHANNEL_SERVICE_SECRET=<run: python -c "import secrets; print(secrets.token_hex(32))">
CHANNEL_SERVICE_URL=http://localhost:8001/send
ALLOWED_ORIGINS=http://localhost:5173
APP_ENV=development
# CLERK_AUTHORIZED_PARTIES  <- leave this line out. An empty value disables the azp check.
```

> 💡 Locally, point `DATABASE_URL` and `DEV_DATABASE_URL` at the **same** database. The seed script writes to `DEV_DATABASE_URL`, while the API and the ML scripts read `DATABASE_URL`.

Use the same `CHANNEL_SERVICE_SECRET` in `channel-service/.env`, and add your Clerk publishable key to `frontend/.env`:

```env
VITE_API_URL=http://localhost:8000
VITE_CLERK_PUBLISHABLE_KEY=pk_test_...
```

### 4. Run the three services

**Terminal 1: CRM API** (`http://localhost:8000`)

```bash
cd backend
python -m venv venv && source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
pip install faker                                    # only needed for the dev seeder
python seed.py --confirm-reset                       # 100 customers and 300 orders, local DBs only
uvicorn main:app --reload --port 8000
```

**Terminal 2: Channel service** (`http://localhost:8001`). It has no requirements file, so install its four dependencies directly:

```bash
cd channel-service
python -m venv venv && source venv/bin/activate
pip install fastapi uvicorn httpx python-dotenv
uvicorn main:app --reload --port 8001
```

**Terminal 3: Frontend** (`http://localhost:5173`)

```bash
cd frontend
npm install
npm run dev
```

### 5. Optional: ML scripts

The trained RFM pickles are committed, so the training step is optional on a fresh clone. Run these from `backend/`:

```bash
python scripts/train_rfm_clusters.py        # retrain: picks k from 2 to 8 by silhouette
python scripts/update_customer_clusters.py  # assigns customers.cluster_id
python scripts/update_churn_scores.py       # optional: persists the churn heuristic
```

> These scripts import `core.config`, so they need **every** backend variable, including Groq and Clerk values, even though they never call those services.

### Environment variables

| Variable | Service | Required | Purpose |
|---|---|:---:|---|
| `DATABASE_URL` | backend | ✅ | PostgreSQL connection string for the API and ML scripts |
| `GROQ_API_KEY` | backend | ✅ | Groq API key |
| `CLERK_JWKS_URL` | backend | ✅ | Clerk JWKS endpoint used to verify session tokens |
| `CLERK_ISSUER` | backend | ✅ | Must equal the token `iss` claim. The app will not boot without it. |
| `CHANNEL_SERVICE_SECRET` | backend, channel | ✅ | Shared secret for `/send` and `/receipt`. The API will not boot without it. |
| `ALLOWED_ORIGINS` | backend | prod ✅ | Comma-separated CORS origins. No wildcards, no trailing slash. Defaults to `http://localhost:5173` outside production. |
| `CLERK_AUTHORIZED_PARTIES` | backend | optional | Allowed `azp` origins. Defaults to `ALLOWED_ORIGINS`. An empty value disables the check. |
| `APP_ENV` or `ENVIRONMENT` | backend | optional | `production` turns on strict startup checks. Seed and reset require an explicit development value. |
| `DEV_DATABASE_URL` | backend | seed and reset | Local loopback database for destructive dev tools |
| `CONFIRM_SEED_RESET`, `CONFIRM_DESTRUCTIVE_RESET` | backend | optional | Environment alternatives to the `--confirm-*` flags |
| `CHANNEL_SERVICE_URL` | backend | optional | Channel `/send` URL (default `http://localhost:8001/send`) |
| `CRM_RECEIPT_URL` | channel | optional | CRM `/receipt` URL (default `http://localhost:8000/receipt`) |
| `VITE_API_URL` | frontend | ✅ | Backend base URL. The example file points at the live API, so change it for local work. |
| `VITE_CLERK_PUBLISHABLE_KEY` | frontend | ✅ | Clerk publishable key (not included in `.env.example`) |

### Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| `Exception: CLERK_ISSUER missing in .env` at startup | A required variable is unset. The API and every script that imports `core.config` check all five required values. |
| Every request returns `401` | `CLERK_ISSUER` must match the token `iss` exactly. The browser origin must also be in `ALLOWED_ORIGINS` or `CLERK_AUTHORIZED_PARTIES`. |
| Browser CORS error | The origin is missing from `ALLOWED_ORIGINS`, or it has a trailing slash. |
| Funnel numbers never move | The channel service is not running, or its secret differs from the API's (the callback gets `403`). |
| `ModuleNotFoundError: dotenv` in the channel service | Install `python-dotenv` as shown above. |
| Discovered segments are empty or error out | Apply the `cluster_id` migration, then run `update_customer_clusters.py`. |
| Seed or reset says `REFUSED` | Check `APP_ENV`, make `DEV_DATABASE_URL` a loopback host, and pass the confirmation flag. |
| Ask Your Data says the query is "invalid or unsafe" | That is the guard working. PII columns and `churn_score` cannot be queried, so rephrase the question. |

---

## 🌐 Deployment

```mermaid
flowchart LR
    U["👤 Browser"] -->|HTTPS| V["▲ Vercel<br/>frontend/<br/>npm run build → dist"]
    V -->|"HTTPS + Clerk JWT"| R["🟢 Render web service<br/>backend/ · uvicorn"]
    R -->|"asyncpg · TLS"| N[("🐘 Neon PostgreSQL")]
    R -->|HTTPS| G["🧠 Groq"]
    R -->|"POST /send + shared key"| CSV["📨 Render web service<br/>channel-service/"]
    CSV -->|"POST /receipt + shared key"| R
    V <-->|sessions| CL["🔐 Clerk"]
    R -.->|JWKS| CL

    classDef a fill:#1e3a5f,stroke:#60a5fa,color:#fff
    classDef b fill:#4a2c1d,stroke:#c8852e,color:#fff
    classDef c fill:#14532d,stroke:#4ade80,color:#fff
    classDef d fill:#1f2937,stroke:#9ca3af,color:#fff
    class U,V a
    class R,CSV b
    class N c
    class G,CL d
```

Only `frontend/vercel.json` is committed as deployment configuration. It builds with `npm run build`, serves `dist`, and rewrites every path to `index.html` so deep links such as `/campaigns` survive a refresh. The Render settings below are the recommended setup, not files in the repo.

| Service | Root directory | Build command | Start command |
|---|---|---|---|
| **Frontend** (Vercel) | `frontend` | `npm run build` | static output `dist` |
| **CRM API** (Render) | `backend` | `pip install -r requirements.txt` | `uvicorn main:app --host 0.0.0.0 --port $PORT` |
| **Channel service** (Render) | `channel-service` | `pip install fastapi uvicorn httpx python-dotenv` | `uvicorn main:app --host 0.0.0.0 --port $PORT` |

| Variable | Vercel | CRM API | Channel service |
|---|:---:|:---:|:---:|
| `VITE_API_URL`, `VITE_CLERK_PUBLISHABLE_KEY` | ✅ | | |
| `DATABASE_URL` (Neon, `sslmode=require`) | | ✅ | |
| `GROQ_API_KEY` | | ✅ | |
| `CLERK_JWKS_URL`, `CLERK_ISSUER` | | ✅ | |
| `ALLOWED_ORIGINS` (the Vercel URL, no trailing slash) | | ✅ | |
| `APP_ENV=production` | | ✅ | |
| `CHANNEL_SERVICE_SECRET` (identical value) | | ✅ | ✅ |
| `CHANNEL_SERVICE_URL` (`https://<channel>/send`) | | ✅ | |
| `CRM_RECEIPT_URL` (`https://<crm>/receipt`) | | | ✅ |

**Rollout order**

1. Create the Neon database and run `psql "$DATABASE_URL" -f backend/schema.sql`.
2. Deploy the channel service and note its URL.
3. Deploy the CRM API with the variables above, then set the channel service's `CRM_RECEIPT_URL`.
4. Deploy the frontend, then add its domain in the Clerk dashboard.
5. Smoke test: `GET /` returns `"status": "running"`, sign in, create a campaign, and watch the funnel fill in.

**Populating a remote database.** `seed.py` refuses non-loopback hosts on purpose, so load demo data through `POST /customers/bulk` and `POST /orders/bulk` with a Clerk token (see the examples in [API Endpoints](#-api-endpoints)). Then run the RFM scripts with `DATABASE_URL` pointing at Neon.

**Startup guards that protect a deploy**

| Guard | If it fails |
|---|---|
| A required variable is missing | The process exits at import with a clear message. |
| A wildcard, scheme-less, or trailing-slash origin | The process exits at import. |
| `APP_ENV=production` without an explicit `ALLOWED_ORIGINS` | The process exits at import. |
| `APP_ENV=production` with an empty authorized-party list | The process exits at import. |

> 🔓 **Before sharing a deployment widely:** Clerk development instances allow self-service sign-up, and the CRM treats every signed-in user as fully authorized. Restrict sign-up in the Clerk dashboard or add roles first.

## 📸 Screenshots and Walkthrough

> **Heads-up:** the Dashboard, Customers, and Segments captures predate the INR and sample-feedback changes: they show `$` amounts and a "Customer reviews" panel, while the current code renders ₹ and a **"Sample customer feedback"** panel. Re-capture before treating them as current.

| Authentication | Dashboard |
|---|---|
| ![Authentication](Screenshots/authentication.png) | ![Dashboard](Screenshots/dashboard-overview.png) |

| Dashboard: funnel, top customers | Customers |
|---|---|
| ![Dashboard lower](Screenshots/dashboard-funnel.png) | ![Customers](Screenshots/customers.png) |

| Segments | Campaign creation |
|---|---|
| ![Segments](Screenshots/segments.png) | ![Campaigns](Screenshots/campaigns.png) |

![Campaign analytics](Screenshots/campaign-analytics.png)

| Page | What you see | Backend support |
|---|---|---|
| **Dashboard** | KPI cards, Ask Your Data, city chart, revenue trend, funnel, top customers, activity, sample feedback | `/dashboard/*`, `/analytics/query`, `/customers`, `/campaigns/{id}/stats` |
| **Customers** | Search box, table with health badges, detail panel | `/customers` |
| **Segments** | Saved segments, discovered RFM clusters, create form, AI assistant | `/segments*`, `/ai/suggest-segment` |
| **Campaigns** | History table, create form with AI drafting, stats drawer | `/campaigns*`, `/ai/draft-message` |

**Feedback panel:** ten hard-coded quotes with static ratings under the placeholder name "Sample customer". It is **illustrative UI content, not real customer reviews**; no review source is connected.



---

## 🧩 Engineering Decisions

| Decision | Reason and trade-off |
|---|---|
| **Query spec instead of LLM-written SQL** | Validating a closed JSON schema is easier to reason about than filtering arbitrary SQL. *Cost:* only what the schema allows (single-level aggregates, fixed joins) is expressible. |
| **Allow-list, not block-list** | Naming what *is* queryable (per-table columns, operators, aggregates) fails safe when the schema grows. A block-list fails open. |
| **Parameterized SQL + read-only transaction** | Values never touch the SQL text, and the database itself refuses writes. *Cost:* read-only is per transaction, not a separate DB role. |
| **Heuristic churn index, not a classifier** | With no observed churn events, a model trained on rule-derived labels only reproduces the rule. A transparent index is honest and testable. *Cost:* the parameters are chosen, not calibrated. |
| **Score churn at read time** | Customer rows are always fresh and no model loads in the API. *Cost:* a little arithmetic per row. |
| **Offline clustering** | KMeans is cheap to serve and slow to retrain. Scripts keep the request path light. *Cost:* clusters go stale until the scripts rerun. |
| **Separate channel service** | Real delivery is asynchronous, so a second service forces the right callback design: shared secret, fixed destination, idempotent receipts. *Cost:* another process, and it is a simulator. |
| **Monotonic receipt updates in SQL** | Out-of-order or duplicate callbacks cannot move a message backwards. *Cost:* the rules live in SQL, so they need tests (they have them). |
| **Clerk + JWKS** | No password handling and no shared JWT secret. *Cost:* a hosted-identity dependency and no roles yet. |
| **Groq** | Fast, short JSON and summary calls. *Cost:* prompts leave the system, hence the PII boundary. |
| **FastAPI + asyncpg, no ORM** | Async I/O for the database and raw SQL that keeps the query builder transparent. *Cost:* no ORM and no migration framework. |
| **Fail closed at boot** | Missing secrets, wildcard origins, or an unset issuer stop the process instead of degrading security silently. |

### Hardening history

Each fix shipped with a regression suite, so the behaviour cannot quietly regress.

```mermaid
timeline
    title Six hardening passes and the suites that lock them in
    Fix 1 : Security boundary : Channel key on send and receipt : CORS allow-list : Clerk issuer and azp checks
    Fix 2 : Honest delivery reporting : HTTP errors are not success : partially_failed status : No receipt overwrites
    Fix 3 : Honest churn methodology : Retired the circular classifier : Transparent heuristic
    Fix 4 : Bulk import integrity : Accurate inserted and skipped counts : All or nothing transactions
    Fix 5 : Database workflow safety : Canonical schema and additive migrations : Guarded seed and reset
    Fix 6 : Deploy routing : Vercel SPA rewrites for deep links
```

---

## 🧪 Testing

Twelve test files guard the security and correctness boundaries. They hold more code than the application itself. There is **no CI pipeline or coverage report yet**, and the frontend has no unit tests.

```mermaid
pie showData title Automated tests by area, 130 test functions
    "Churn methodology" : 27
    "Auth, CORS, channel key (Fix 1)" : 26
    "Database workflow (Fix 5)" : 23
    "Campaign reporting (Fix 2)" : 22
    "Bulk imports (Fix 4)" : 13
    "Channel service" : 8
    "Vercel routing (Fix 6)" : 7
    "Receipt security" : 3
    "PII boundary" : 1
```

| File | Verifies | Count |
|---|---|---:|
| `tests_security_fix1.py` | Clerk issuer and `azp` handling, CORS configuration, channel-key enforcement | 26 |
| `tests_campaign_reporting_fix2.py` | HTTP 4xx, 5xx, and timeouts are not success, mixed outcomes give `partially_failed`, empty segments give `failed`, and a late dispatch never overwrites a receipt | 22 |
| `tests_churn_methodology_fix3.py` | Determinism, `[0, 1]` bounds, monotonicity, bad inputs, alignment with the front-end badges, and retirement of the pseudo-label model | 27 |
| `tests_bulk_transactions_fix4.py` | Accurate inserted and skipped counts, all-or-nothing rollback, and a `404` with full rollback for an unknown customer. Uses an in-memory mock connection. | 13 |
| `tests_database_workflow_fix5.py` | Reset SQL drops in foreign-key order, `schema.sql` is repeatable DDL with no `DROP` or `DELETE`, migrations are additive, nothing resets on startup, and seed and reset guards refuse remote hosts | 23 |
| `tests_vercel_routing_fix6.py` | Valid `vercel.json`, preserved build settings, and the catch-all rewrite for every client route | 7 |
| `tests_receipt_security.py` | Missing key → 401, wrong key → 403, correct key passes the auth check | 3 |
| `tests_analytics_security.py` | Unknown tables and columns, direct PII, dangerous functions, malicious operators, limit above 100, multi-statement input, valid queries and joins | 13 cases |
| `tests_segment_security.py` | Empty, unknown, negative, and mistyped filters, `min > max`, injection-style values, valid combinations | 14 cases |
| `tests_analytics_pii_security.py` | With Groq mocked, name, email, and phone never reach the summary prompt while city and spend do | 1 |
| `tests_historical_orders.py` | Backdated bulk orders never move `last_order_date` backwards | needs a live database |
| `channel-service/tests_channel_service_security.py` | `/send` returns 401 without a key, 403 with a wrong one, 200 queued with the right one, and 500 when unconfigured. No CORS middleware is registered. | 8 |

**Run them.** The backend suites need five environment variables because `core/config.py` fails fast. Dummy values are enough. Run **one file per process**: some suites set environment variables at import time, and two are `async` tests that expect to be launched with `python`.

```bash
cd backend && pip install pytest
export DATABASE_URL=postgresql://u:p@localhost:5432/x GROQ_API_KEY=dummy \
       CLERK_JWKS_URL=https://example.clerk.accounts.dev/.well-known/jwks.json \
       CLERK_ISSUER=https://example.clerk.accounts.dev \
       CHANNEL_SERVICE_SECRET=test-secret APP_ENV=development

for f in tests_security_fix1 tests_campaign_reporting_fix2 tests_churn_methodology_fix3 \
         tests_bulk_transactions_fix4 tests_database_workflow_fix5 tests_vercel_routing_fix6 \
         tests_receipt_security; do
  python -m pytest -q "$f.py"
done

python tests_analytics_security.py
python tests_segment_security.py
python tests_analytics_pii_security.py

# channel service
cd ../channel-service && CHANNEL_SERVICE_SECRET=x python -m pytest -q tests_channel_service_security.py
```

> Running every file in a single `pytest` session currently produces a few failures from import-time environment setup and async tests, not from product bugs. Consolidating them with `pytest-asyncio` and a `conftest.py` is on the roadmap below.

---

## 🚧 Known Limitations

This project is a portfolio prototype, and these are the edges worth knowing about.

| Area | Limitation |
|---|---|
| **Delivery** | Simulated. No real email, SMS, or WhatsApp is sent, and outcomes are random. |
| **Access control** | Single-tenant with no roles. Any valid Clerk user can use every route, and the demo's Clerk instance allows self-service sign-up. The Customers page exposes names, emails, and phones to every signed-in user. |
| **Durability** | Campaign processing uses FastAPI background tasks. A restart mid-dispatch can leave a campaign in `processing`, and there are no retries. |
| **Dispatch concurrency** | All sends for a campaign start together. The `Semaphore(5)` limits only the follow-up database writes, so very large segments would need real throttling. |
| **Churn scoring** | A heuristic with chosen, uncalibrated parameters, not a validated prediction. The VIP badge overrides risk. |
| **Clusters** | `cluster_id` is not stable across retrains, names are relative rankings, and scoring is manual. With the committed k = 3 model only three labels appear. The persisted `churn_score` column is not read by the API. |
| **Async hygiene** | The Groq SDK and the JWKS fetch on a cache miss are synchronous calls inside async code, so they can briefly block the event loop. |
| **Error detail** | Analytics returns Postgres error text on a rejected query, and the AI routes return exception text on failure. |
| **Scale** | `GET /customers` returns every row without pagination, and the dashboard derives its city chart and top-10 from it. The AI endpoints have no rate limiting. |
| **Schema** | The 409 segment guard lives in the API. The database cascades. `communications` has no unique `(campaign_id, customer_id)`, and `customer_count` is a snapshot. |
| **Receipts** | A receipt for a campaign and customer pair that does not exist still returns success. |
| **Read-only isn't a role** | Analytics shares the API's connection pool, and results are not cached. |
| **Sample data** | The feedback panel is static, all customers and orders are synthetic, and the 30-day chart empties as seeded data ages. Observability is limited to `print` and standard logging. |

---

## 🔭 Future Work

```mermaid
quadrantChart
    title Roadmap by impact and effort
    x-axis Low effort --> High effort
    y-axis Low impact --> High impact
    quadrant-1 Plan carefully
    quadrant-2 Do first
    quadrant-3 Quick wins
    quadrant-4 Park for later
    Restrict sign-up and add roles: [0.30, 0.90]
    Durable job queue with retries: [0.80, 0.80]
    Real churn labels and evaluation: [0.90, 0.85]
    CI with one pytest session: [0.35, 0.65]
    Rate limit AI endpoints: [0.25, 0.58]
    Paginate customers: [0.30, 0.50]
    Read-only DB role for analytics: [0.38, 0.55]
    Throttle dispatch fan-out: [0.18, 0.45]
    Async Groq client: [0.20, 0.32]
    Scheduled RFM retraining: [0.58, 0.48]
```

| Priority | Item |
|---|---|
| 🥇 | Restrict sign-up, add roles, and scope data per tenant |
| 🥇 | CI that runs every suite, with `pytest-asyncio` and a shared `conftest.py` |
| 🥈 | A durable job queue for dispatch, with retries and idempotent receipts |
| 🥈 | Bounded dispatch concurrency, rate limiting on AI routes, and paginated customer lists |
| 🥈 | A dedicated read-only database role for Ask Your Data |
| 🥉 | Real churn outcomes, held-out evaluation, and scheduled rescoring and reclustering |
| 🥉 | A connected review source in place of the static feedback panel |

---

## 🐳 Tech Stack

| Layer | Choice |
|---|---|
| **Frontend** | React 18, Vite 6, Tailwind CSS 3, React Router 7, Axios, Recharts, Sonner, Lucide icons, `@clerk/clerk-react` |
| **Backend** | Python 3.11+, FastAPI, Pydantic 2, `asyncpg`, PyJWT with `cryptography`, `httpx`, `email-validator` |
| **Database** | PostgreSQL (Neon in production), JSONB columns, 10 indexes |
| **Auth** | Clerk, RS256 session JWTs verified through JWKS |
| **AI** | Groq SDK, model `openai/gpt-oss-120b`, JSON-object mode |
| **ML** | scikit-learn (KMeans, StandardScaler, silhouette score) and NumPy, with pickled artifacts |
| **Extra service** | FastAPI channel-delivery simulator |
| **Testing** | pytest and unittest-style suites plus script-style regression runners |
| **Dev tooling** | Faker for seeding, Vite dev server |
| **Deployment** | Vercel (frontend), Render (CRM API and channel service), Neon (database) |

---

<div align="center">

Built by **Saksham Dhiman** · B.Tech CSE · [GitHub @SakshamDhiman2202](https://github.com/SakshamDhiman2202)

</div>
