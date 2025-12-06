# India-First GEO/AEO Platform

> **Production-ready MVP for tracking AI search visibility across ChatGPT, Perplexity, Gemini, Claude, Google AI, Grok, and Meta AI. Built specifically for Indian SaaS, fintech, e-commerce, and agency teams.**

[![Status](https://img.shields.io/badge/status-production--ready-green)]()
[![License](https://img.shields.io/badge/license-MIT-blue)]()

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Features](#-features)
- [Crawler Requirements](#-crawler-requirements)
- [Quick Start](#-quick-start)
- [Configuration](#-configuration)
- [API Documentation](#-api-documentation)
- [How Everything Works](#-how-everything-works)
  - [How Authentication Works](#-how-authentication-works)
  - [How the Crawler System Works](#️-how-the-crawler-system-works)
  - [How Metrics Are Calculated](#-how-metrics-are-calculated)
  - [How the Database Schema Works](#-how-the-database-schema-works)
  - [How Frontend Communicates with Backend](#-how-frontend-communicates-with-backend)
  - [How Each Feature Works](#-how-each-feature-works)
  - [How Deployment Works](#-how-deployment-works)
  - [How to Extend the System](#-how-to-extend-the-system)
- [Deployment](#-deployment)
- [Contributing](#-contributing)

---

## 🎯 Overview

This platform helps Indian brands track and optimize their visibility across AI answer engines. Monitor brand mentions, competitor analysis, sentiment tracking, and get actionable insights to improve your AI search presence.

### Key Capabilities

- **Multi-Platform Tracking**: Monitor visibility across 7 AI platforms
- **Real-time Metrics**: Visibility scores, trend analysis, competitor insights
- **Automated Crawling**: Scheduled scans every 6 hours
- **India-First**: DPDP compliance, INR pricing, regional language support
- **Enterprise Ready**: Team collaboration, API access, white-label options

---

## 🏗️ Architecture

### System Architecture Diagram

```mermaid
graph TB
    User[Users<br/>Web Browser] --> Frontend[React Frontend<br/>Vite]
    Frontend --> API[FastAPI Backend]
    API --> Auth[Auth Service]
    API --> Metrics[Metrics Service]
    API --> Crawler[Crawler Service]
    Auth --> DB[(PostgreSQL)]
    Metrics --> DB
    Crawler --> ChatGPT[ChatGPT]
    Crawler --> Perplexity[Perplexity]
    Crawler --> Other[Gemini/Claude/Grok/Meta AI]
    
    style User fill:#e1f5ff
    style Frontend fill:#e8f5e9
    style API fill:#fff9c4
    style Auth fill:#ffe0b2
    style Metrics fill:#ffe0b2
    style Crawler fill:#ffe0b2
    style DB fill:#f3e5f5
    style ChatGPT fill:#ffcdd2
    style Perplexity fill:#ffcdd2
    style Other fill:#ffcdd2
```

### Data Flow Diagram

```mermaid
flowchart TD
    Start([Start]) --> User[User Creates Query]
    User --> Scheduler[Scheduler<br/>6h interval]
    Scheduler --> Crawler[Crawler Executes]
    Crawler --> ChatGPT[ChatGPT]
    Crawler --> Perplexity[Perplexity]
    Crawler --> Other[Other Platforms]
    ChatGPT --> Process[Parse & Analyze]
    Perplexity --> Process
    Other --> Process
    Process --> DB[(Store Citations)]
    DB --> Metrics[Calculate Metrics]
    Metrics --> Dashboard[Update Dashboard]
    Dashboard --> End([End])
    
    style Start fill:#c8e6c9
    style User fill:#bbdefb
    style Scheduler fill:#fff9c4
    style Crawler fill:#ffe0b2
    style ChatGPT fill:#ffcdd2
    style Perplexity fill:#ffcdd2
    style Other fill:#ffcdd2
    style Process fill:#e1bee7
    style DB fill:#b2ebf2
    style Metrics fill:#f8bbd0
    style Dashboard fill:#c8e6c9
    style End fill:#ffcdd2
```

### Database Schema Diagram

```mermaid
erDiagram
    Organizations ||--o{ Users : "has many"
    Organizations ||--o{ Brands : "has many"
    Brands ||--o{ Queries : "has many"
    Queries ||--o{ ScanRuns : "has many"
    ScanRuns ||--|| Citations : "has one"
    
    Organizations {
        int id PK
        string name
        string country
        string billing_plan
    }
    
    Users {
        int id PK
        string email
        int org_id FK
        string role
        string password_hash
    }
    
    Brands {
        int id PK
        string name
        int org_id FK
        string domain
    }
    
    Queries {
        int id PK
        int brand_id FK
        string text
        string language
        json platforms
        bool active
    }
    
    ScanRuns {
        int id PK
        int query_id FK
        string platform
        datetime run_at
        string status
    }
    
    Citations {
        int id PK
        int scan_run_id FK
        bool brand_mentioned
        int brand_position
        string snippet
        string sentiment
        json competitors
    }
```

### Component Architecture

```mermaid
graph TB
    Frontend[Frontend<br/>React + TypeScript<br/>Dashboard | Brands | Metrics | Reports] --> Backend[Backend<br/>FastAPI<br/>REST API | OAuth 2.0 | Webhooks]
    Backend --> Auth[Auth Service]
    Backend --> Metrics[Metrics Service]
    Backend --> Crawler[Crawler Service]
    Auth --> DB[(PostgreSQL)]
    Metrics --> DB
    Crawler --> DB
    Crawler --> AI[AI Platforms<br/>ChatGPT | Perplexity | Gemini | Claude | Grok | Meta AI]
    
    style Frontend fill:#bbdefb
    style Backend fill:#c8e6c9
    style Auth fill:#fff9c4
    style Metrics fill:#fff9c4
    style Crawler fill:#fff9c4
    style DB fill:#e1bee7
    style AI fill:#ffcdd2
```

---

## 🕷️ Crawler Requirements

### API Keys & Authentication

**Good News: No API keys required for web scraping approach!**

The crawlers use web scraping (Playwright/Selenium) instead of official APIs, which means:

#### ✅ **No API Keys Required**
- **Perplexity**: ✅ No authentication needed - works via public web scraping
- **Gemini**: ⚠️ Placeholder - Would need API key if using official API
- **Claude**: ⚠️ Placeholder - Would need API key if using official API  
- **Google AI**: ⚠️ Placeholder - No API key needed for web scraping

#### 🔐 **Session Cookies Required** (Optional for Enhanced Access)
- **ChatGPT**: Requires logged-in session cookies
  - **How to get**: Export cookies from browser after logging into chat.openai.com
  - **File path**: Set `COOKIES_PATH` environment variable or pass to crawler
  - **Without cookies**: Crawler will return error message (won't crash)
  
- **Grok (X/Twitter)**: Requires X (Twitter) authentication cookies
  - **How to get**: Export cookies from browser after logging into x.com
  - **Requires**: X Premium subscription for Grok access
  - **Without cookies**: Crawler will return error message
  
- **Meta AI**: Requires Facebook authentication cookies
  - **How to get**: Export cookies from browser after logging into facebook.com
  - **Without cookies**: Crawler will return error message

### Crawler Configuration

```python
# Example: Using cookies for ChatGPT
from app.crawler.chatgpt import ChatGPTClient

client = ChatGPTClient(
    cookies_path="/path/to/chatgpt_cookies.json",  # Optional
    headless=True,
    timeout=60000
)

result = await client.run_query("best CRM software", "Zoho")
```

### Cookie Export Guide

1. **Install browser extension** (e.g., "Get cookies.txt" for Chrome)

2. **Login** to the platform (ChatGPT, X, Facebook)
3. **Export cookies** as JSON format
4. **Save** to a file (e.g., `chatgpt_cookies.json`)
5. **Configure** in crawler or environment variable

**Note**: Cookies expire periodically. You'll need to refresh them when they expire.

---

## 🚀 Quick Start

### Prerequisites

- Python 3.11+
- Node.js 18+
- PostgreSQL (or Supabase free tier)
- Git

### Installation

#### 1. Clone Repository
```bash
git clone <repository-url>
cd geo-platform
```

#### 2. Backend Setup

```bash
cd backend

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create .env file
cat > .env << EOF
DATABASE_URL=sqlite:///./geo_platform.db
SECRET_KEY=$(python3 -c "import secrets; print(secrets.token_urlsafe(32))")
BACKEND_URL=http://localhost:8000
FRONTEND_URL=http://localhost:5173
EOF

# Run migrations
alembic upgrade head

# Start server
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

#### 3. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

#### 4. Access Application

- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs

---

## ⚙️ Configuration

### Required Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/geo_platform

# Security (REQUIRED)
SECRET_KEY=your-secret-key-here

# URLs
BACKEND_URL=http://localhost:8000
FRONTEND_URL=http://localhost:5173
```

### Optional Environment Variables

```bash
# Email (SendGrid)
SENDGRID_API_KEY=your_sendgrid_key
SENDGRID_FROM_EMAIL=noreply@yourdomain.com

# OAuth Providers
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret

# Error Tracking
SENTRY_DSN=your_sentry_dsn

# Crawler Cookies (Optional)
CHATGPT_COOKIES_PATH=/path/to/chatgpt_cookies.json
GROK_COOKIES_PATH=/path/to/grok_cookies.json
META_AI_COOKIES_PATH=/path/to/meta_cookies.json
```

**See `REQUIRED_API_KEYS.md` for detailed setup instructions.**

---

## 📊 Features

### Phase 1: MVP ✅ Complete
- User authentication (OAuth 2.0 + Email/Password)
- Brand & Query management
- AI visibility tracking (7 platforms)
- Metrics dashboard
- Citation tracking with sentiment analysis
- Competitor analysis
- Weekly email reports

### Phase 2: Advanced Features ✅ Complete
- Team collaboration & role-based access
- API keys & webhooks
- Integrations (Slack, Google Sheets, Zapier, HubSpot)
- Advanced reporting & PDF export
- Content optimization engine
- Competitive intelligence
- DPDP compliance

### Phase 3: Enterprise Features ✅ Complete
- Predictive analytics
- Content generation workflow
- Industry benchmarking
- Multi-channel tracking
- SSO/SAML support
- Agency platform
- Advanced workflows & automation

---

## 📚 API Documentation

### Interactive Documentation

Once the backend is running:
- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

### Key Endpoints

```
POST   /v1/auth/register          # Register new user
POST   /v1/auth/login             # Login
GET    /v1/brands                  # List brands
POST   /v1/brands                  # Create brand
GET    /v1/brands/{id}/metrics     # Get metrics
GET    /v1/citations               # List citations
POST   /v1/citations/ingest        # Ingest citation (API key required)
```

**Full API documentation available at `/docs` endpoint.**

---

## 🚢 Deployment

### Backend (Railway/Render)

1. **Push to GitHub**
2. **Connect repository** to Railway/Render
3. **Set environment variables**:
   - `DATABASE_URL`
   - `SECRET_KEY`
   - `FRONTEND_URL`
   - Optional: `SENDGRID_API_KEY`, OAuth credentials
4. **Deploy**

### Frontend (Vercel)

1. **Push to GitHub**
2. **Import project** in Vercel
3. **Set environment variable**: `VITE_API_URL`
4. **Deploy**

### Database (Supabase)

1. **Create project** at https://supabase.com
2. **Get connection string** from Settings → Database
3. **Update** `DATABASE_URL` in backend `.env`
4. **Run migrations**: `alembic upgrade head`

### Crawler (GitHub Actions)

1. **Set up GitHub Secrets**:
   - `DATABASE_URL`
   - `API_URL`
   - `API_TOKEN` (optional)

2. **Workflow runs automatically** every 6 hours

**See `DEPLOYMENT.md` for detailed deployment guide.**

---

## 🧪 Testing

### Backend Tests
```bash
cd backend
pytest
```

### Frontend Tests
```bash
cd frontend
npm test
```

### Manual Testing

1. **Register** a new account at http://localhost:5173/signup
2. **Create a brand**
3. **Add queries**
4. **Wait for crawler** to run (or trigger manually)
5. **View metrics** in dashboard

---

## 📁 Project Structure

```
.
├── backend/
│   ├── app/
│   │   ├── api/routes/          # API endpoints
│   │   ├── models/              # Database models
│   │   ├── schemas/             # Pydantic schemas
│   │   ├── services/            # Business logic
│   │   ├── crawler/             # Web scraping clients
│   │   │   ├── base.py          # Base client interface
│   │   │   ├── chatgpt.py       # ChatGPT scraper
│   │   │   ├── perplexity.py   # Perplexity scraper
│   │   │   ├── gemini.py        # Gemini (placeholder)
│   │   │   ├── claude.py        # Claude (placeholder)
│   │   │   ├── google_ai.py     # Google AI (placeholder)
│   │   │   ├── grok.py          # Grok scraper
│   │   │   └── meta_ai.py        # Meta AI scraper
│   │   └── core/                # Config & utilities
│   ├── alembic/                 # Database migrations
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── pages/               # Page components
│   │   ├── components/          # Reusable components
│   │   ├── hooks/               # Custom hooks
│   │   └── api/                 # API client
│   └── package.json
└── .github/workflows/           # GitHub Actions
```

---

## 🔧 How Everything Works

This section provides detailed explanations of how each component of the platform works.

### 🔐 How Authentication Works

The platform uses **OAuth 2.0** as the primary authentication mechanism, with support for email/password login and external OAuth providers (Google, GitHub).

#### **1. OAuth 2.0 Flow (Internal)**

**Registration Process:**
1. User submits registration form with email, password, and organization name
2. Backend creates:
   - A new `Organization` record
   - A new `User` record with hashed password (bcrypt)
   - A default `OAuthClient` for the organization
   - An `OAuthToken` (access token + refresh token) for immediate use
3. Tokens are returned to frontend and stored in `localStorage`

**Login Process:**
1. User submits email and password
2. Backend verifies credentials using `verify_password()` (bcrypt comparison)
3. If valid, backend:
   - Retrieves or creates default `OAuthClient` for user's organization
   - Generates new `OAuthToken` (access token expires in 1 hour, refresh token in 30 days)
   - Returns tokens to frontend
4. Frontend stores tokens and fetches user info from `/v1/oauth/userinfo`

**Token Refresh:**
1. When access token expires (401 error), frontend automatically calls `/v1/auth/refresh`
2. Backend validates refresh token and issues new access/refresh tokens
3. Original request is retried with new access token

**Authorization:**
- All protected endpoints use `get_current_user()` dependency
- This function:
  1. Extracts `Bearer` token from `Authorization` header
  2. Tries to find `OAuthToken` in database (OAuth flow)
  3. Falls back to JWT decode (backward compatibility)
  4. Returns `User` object if valid, raises 401 if invalid

#### **2. External OAuth Providers (Google, GitHub)**

**Google OAuth Flow:**
1. User clicks "Sign in with Google" button
2. Frontend redirects to `/v1/oauth/google/login`
3. Backend redirects to Google OAuth consent screen
4. User authorizes, Google redirects to `/v1/oauth/google/callback` with `code`
5. Backend exchanges `code` for access token via Google API
6. Backend fetches user info from Google API
7. Backend creates/finds user account and generates platform tokens
8. Backend redirects to frontend with tokens in URL fragment
9. Frontend extracts tokens and stores them

**GitHub OAuth Flow:**
- Same flow as Google, but uses GitHub OAuth endpoints

**Implementation Details:**
- OAuth provider credentials stored in environment variables
- State parameter used to prevent CSRF attacks
- PKCE (Proof Key for Code Exchange) supported for enhanced security
- Callback URLs must match registered redirect URIs

#### **3. Password Reset Flow**

1. User requests password reset via `/v1/auth/forgot-password`
2. Backend generates secure token and stores in `PasswordResetToken` table
3. Backend sends email with reset link containing token
4. User clicks link, frontend shows reset form
5. User submits new password to `/v1/auth/reset-password` with token
6. Backend validates token (checks expiration), hashes new password, updates user
7. Token is deleted after use

---

### 🕷️ How the Crawler System Works

The crawler system automatically queries AI answer engines and stores results in the database.

#### **1. Crawler Architecture**

**Base Client Interface:**
- All crawlers inherit from `AnswerEngineClient` (abstract base class)
- Each crawler implements `run_query(query: str, brand_name: str) -> NormalizedResult`
- `NormalizedResult` contains:
  - `brand_mentioned`: Boolean indicating if brand was found
  - `brand_position`: Position in list (1-based, None if not in list)
  - `snippet`: Text excerpt from answer
  - `sentiment`: "positive", "neutral", or "negative"
  - `competitors`: List of competitor mentions with positions

**Crawler Implementation (Perplexity Example):**
1. Uses Playwright to launch headless browser
2. Navigates to Perplexity website
3. Finds search input and submits query
4. Waits for answer to load
5. Extracts answer text using multiple CSS selectors (fallback strategy)
6. Parses text to find:
   - Brand mentions (case-insensitive, fuzzy matching)
   - Brand position (if in numbered list)
   - Competitor mentions
   - Sentiment (using `analyze_sentiment()` service)
7. Returns `NormalizedResult`

**ChatGPT/Grok/Meta AI:**
- Similar flow but requires cookies for authenticated access
- Cookies loaded from JSON file (exported from browser)
- Without cookies, crawler returns error (doesn't crash)

#### **2. Scheduler System**

**Embedded Scheduler (Optional):**
- Uses APScheduler (BackgroundScheduler)
- Scheduled job runs every 6 hours (0:00, 6:00, 12:00, 18:00 UTC)
- Job function: `run_crawls_sync()` (synchronous wrapper)
- Calls `run_crawls()` (async function) using `asyncio.run()`

**GitHub Actions Scheduler (Recommended for Production):**
- Workflow file in `.github/workflows/crawler.yml`
- Runs on cron schedule: `0 */6 * * *` (every 6 hours)
- Executes crawler script that:
  1. Connects to database
  2. Fetches all active brands and queries
  3. For each query, runs crawler for each platform
  4. Posts results to `/v1/citations/ingest` endpoint

**Crawler Execution Flow:**
1. Scheduler triggers `run_crawls()`
2. Function queries database for all active brands
3. For each brand, queries all active queries
4. For each query, iterates through platforms (from `query.platforms` field)
5. Initializes appropriate crawler client
6. Calls `client.run_query(query.text, brand.name)`
7. Prepares citation data from `NormalizedResult`
8. Posts to `/v1/citations/ingest` endpoint (requires API token)
9. Endpoint creates `ScanRun` and `Citation` records in database

#### **3. Data Ingestion**

**Citation Ingestion Endpoint (`/v1/citations/ingest`):**
- Protected by `verify_api_token()` dependency
- Accepts `CitationIngest` schema:
  - `query_id`: ID of the query that was crawled
  - `platform`: Platform name (e.g., "chatgpt", "perplexity")
  - `brand_mentioned`: Boolean
  - `brand_position`: Integer or None
  - `snippet`: Text excerpt
  - `sentiment`: String
  - `competitors`: List of competitor objects
  - `run_at`: Timestamp
- Creates `ScanRun` record (tracks the crawl execution)
- Creates `Citation` record (stores the actual result)
- Returns `CitationResponse` with created citation data

**Database Relationships:**
- `Query` → `ScanRun` (1:N) - One query can have many scan runs
- `ScanRun` → `Citation` (1:1) - Each scan run produces one citation
- `Query` → `Brand` (N:1) - Many queries belong to one brand
- `Brand` → `Organization` (N:1) - Many brands belong to one organization

---

### 📊 How Metrics Are Calculated

The metrics system calculates visibility scores, trends, and competitor insights from citation data.

#### **1. Visibility Score Calculation**

**Formula:**
```
Visibility Score = (Mention Rate × 0.5 + Position Score × 0.3 + Sentiment Score × 0.2) × 100
```

**Components:**

**Mention Rate (50% weight):**
- `mention_rate = citations_with_brand / total_scans`
- Measures how often the brand appears in results
- Range: 0.0 to 1.0

**Position Score (30% weight):**
- `avg_position = average of all brand_position values`
- `position_score = max(0, 1 - (avg_position - 1) / 10)`
- Lower positions (1, 2, 3) score higher
- Range: 0.0 to 1.0

**Sentiment Score (20% weight):**
- `sentiment_score = positive_citations / total_citations`
- Positive mentions boost score
- Range: 0.0 to 1.0

**Final Score:**
- Weighted sum multiplied by 100
- Clamped to 0-100 range
- Calculated per platform, per time period

**Implementation:**
- Function: `calculate_visibility_score(brand_id, platform, days, db)`
- Queries citations from last N days
- Filters by platform and brand
- Calculates all components and returns final score

#### **2. Trend Data Calculation**

**Daily Trend:**
1. For each day in the last 30 days:
   - Calculate visibility score for that day (using 1-day window)
   - Average across all platforms
   - Store as `{date, score}`

2. Compare recent period (last 7 days) vs older period (7-14 days ago):
   - Calculate average score for each period
   - Determine trend: "up", "down", or "stable"
   - Calculate percentage change

**Platform-Specific Trends:**
- Same calculation but per platform
- Shows which platforms are improving/declining

**Implementation:**
- Function: `get_brand_metrics_overview(brand_id, days, db)`
- Queries citations grouped by date and platform
- Calculates daily scores
- Aggregates for trend analysis

#### **3. Competitor Analysis**

**Competitor Mention Counting:**
1. Query all citations for brand in time period
2. Extract `competitors` array from each citation
3. Count mentions per competitor name
4. Sort by frequency (most mentioned first)
5. Return top 10 competitors with mention counts

**Share of Voice:**
- `share_of_voice = brand_mentions / (brand_mentions + competitor_mentions) × 100`
- Shows brand's visibility relative to competitors

**Implementation:**
- Function: `_get_competitor_analysis_logic(brand_id, days, db)`
- Aggregates competitor data from citations
- Returns sorted list of top competitors

#### **4. Query Performance Metrics**

**Per-Query Analysis:**
1. For each query:
   - Count total citations (scans)
   - Count citations where brand was mentioned
   - Calculate `visibility_rate = mentions / total_scans`
   - Sort by mention count

**Top Performing Queries:**
- Queries with highest mention rates
- Shows which search terms drive visibility

**Implementation:**
- Function: `get_query_metrics_logic(brand_id, days, db)`
- Groups citations by query
- Calculates metrics per query
- Returns top 10 queries

---

### 💾 How the Database Schema Works

The database uses PostgreSQL with SQLAlchemy ORM. Here's how the schema is organized:

#### **Core Tables**

**Organizations:**
- `id`: Primary key
- `name`: Organization name
- `country`: Country code (default: "IN")
- `billing_plan`: Subscription plan ("free", "pro", "enterprise")
- Relationships: Has many Users, Brands

**Users:**
- `id`: Primary key
- `email`: Unique email address
- `password_hash`: Bcrypt hashed password
- `role`: User role ("owner", "member", "admin")
- `org_id`: Foreign key to Organizations
- Relationships: Belongs to Organization, has many OAuthTokens, APIKeys

**Brands:**
- `id`: Primary key
- `name`: Brand name
- `domain`: Brand website domain
- `org_id`: Foreign key to Organizations
- Relationships: Belongs to Organization, has many Queries

**Queries:**
- `id`: Primary key
- `text`: Search query text
- `brand_id`: Foreign key to Brands
- `platforms`: JSON array of platform names (e.g., ["chatgpt", "perplexity"])
- `active`: Boolean flag
- `language`: Query language code
- Relationships: Belongs to Brand, has many ScanRuns

**ScanRuns:**
- `id`: Primary key
- `query_id`: Foreign key to Queries
- `platform`: Platform name ("chatgpt", "perplexity", etc.)
- `run_at`: Timestamp of crawl execution
- `status`: Status ("success", "failed", "pending")
- Relationships: Belongs to Query, has one Citation

**Citations:**
- `id`: Primary key
- `scan_run_id`: Foreign key to ScanRuns
- `brand_mentioned`: Boolean
- `brand_position`: Integer (1-based position in list)
- `snippet`: Text excerpt from answer
- `sentiment`: String ("positive", "neutral", "negative")
- `competitors`: JSON array of competitor objects
- `created_at`: Timestamp
- Relationships: Belongs to ScanRun

#### **OAuth Tables**

**OAuthClients:**
- `id`: Primary key
- `client_id`: Unique client identifier
- `client_secret`: Hashed client secret
- `client_name`: Human-readable name
- `redirect_uris`: Allowed redirect URIs
- `org_id`: Foreign key to Organizations
- `user_id`: Foreign key to Users
- Relationships: Belongs to Organization and User, has many OAuthTokens

**OAuthTokens:**
- `id`: Primary key
- `client_id`: Foreign key to OAuthClients
- `user_id`: Foreign key to Users
- `access_token`: Access token string
- `refresh_token`: Refresh token string
- `token_type`: Token type ("Bearer")
- `scope`: OAuth scopes
- `expires_at`: Access token expiration
- `refresh_expires_at`: Refresh token expiration
- Relationships: Belongs to OAuthClient and User

**OAuthAuthorizationCodes:**
- Stores authorization codes for OAuth flow
- Supports PKCE with `code_challenge` and `code_challenge_method`

#### **Additional Tables**

- **APIKeys**: API keys for programmatic access
- **Webhooks**: Webhook configurations for event notifications
- **TeamMembers**: Team collaboration (invites, roles)
- **Alerts**: Alert configurations (thresholds, notifications)
- **Integrations**: Third-party integrations (Slack, Google Sheets, etc.)
- **Reports**: Custom report configurations
- **AgencyClients**: Agency-client relationships
- **SSOSessions**: SSO/SAML session data
- And many more for Phase 2 and Phase 3 features

**Migration System:**
- Uses Alembic for database migrations
- Migrations in `backend/alembic/versions/`
- Run migrations: `alembic upgrade head`
- Create migration: `alembic revision --autogenerate -m "description"`

---

### 🌐 How Frontend Communicates with Backend

The frontend is a React + TypeScript application that communicates with the FastAPI backend via REST API.

#### **1. API Client Setup**

**Base Configuration:**
- Axios instance created in `frontend/src/api/client.ts`
- Base URL from `VITE_API_URL` environment variable (defaults to `http://localhost:8000`)
- Content-Type header set to `application/json`

**Request Interceptor:**
- Adds `Authorization: Bearer {token}` header to all requests
- Token retrieved from `localStorage.getItem('token')`
- Ensures authenticated requests automatically include token

**Response Interceptor:**
- Handles 401 (Unauthorized) errors
- If 401 and refresh token exists:
  1. Calls `/v1/auth/refresh` with refresh token
  2. Stores new access/refresh tokens
  3. Retries original request with new access token
- If refresh fails, redirects to `/signin`

#### **2. Authentication Flow**

**Login:**
1. User submits email/password
2. Frontend calls `POST /v1/auth/login`
3. Receives `{access_token, refresh_token}`
4. Stores tokens in `localStorage`
5. Calls `GET /v1/oauth/userinfo` to get user data
6. Updates React context with user info

**OAuth Provider Login:**
1. User clicks "Sign in with Google/GitHub"
2. Frontend redirects to `/v1/oauth/{provider}/login`
3. Backend redirects to provider OAuth page
4. User authorizes, provider redirects to callback
5. Backend processes callback and redirects to frontend with tokens
6. Frontend extracts tokens from URL and stores them

**Protected Routes:**
- `useAuth()` hook provides `user` state and `isAuthenticated` flag
- Routes wrapped in `<ProtectedRoute>` component
- If not authenticated, redirects to `/signin`

#### **3. Data Fetching**

**React Query (Optional):**
- Can use React Query for caching and state management
- Provides `useQuery` and `useMutation` hooks
- Automatic refetching and caching

**Manual Fetching:**
- Components call API functions directly
- API functions use `apiClient` (Axios instance)
- Responses handled with try/catch blocks
- Loading states managed with React `useState`

**Example:**
```typescript
const fetchBrands = async () => {
  const response = await apiClient.get('/v1/brands')
  return response.data
}
```

#### **4. Error Handling**

**Network Errors:**
- Axios interceptor catches network errors
- Displays user-friendly error messages
- Logs errors to console (or Sentry if configured)

**Validation Errors:**
- Backend returns 422 (Unprocessable Entity) with error details
- Frontend displays field-specific error messages
- Form validation prevents invalid submissions

**Authentication Errors:**
- 401 errors trigger token refresh
- If refresh fails, user is logged out
- Redirects to sign-in page

---

### 🎯 How Each Feature Works

#### **1. Brand Management**

**Creating a Brand:**
1. User fills form with brand name and domain
2. Frontend calls `POST /v1/brands` with brand data
3. Backend creates `Brand` record linked to user's organization
4. Frontend refreshes brand list

**Listing Brands:**
1. Frontend calls `GET /v1/brands`
2. Backend queries brands filtered by `org_id`
3. Returns list of brands with metrics summary
4. Frontend displays in dashboard

**Brand Metrics:**
1. Frontend calls `GET /v1/brands/{id}/metrics`
2. Backend calculates visibility scores, trends, competitor data
3. Returns comprehensive metrics object
4. Frontend displays in charts and tables

#### **2. Query Management**

**Adding Queries:**
1. User selects brand and enters query text
2. User selects platforms to track (checkboxes)
3. Frontend calls `POST /v1/brands/{id}/queries`
4. Backend creates `Query` record
5. Query is automatically included in next crawl

**Query Execution:**
1. Scheduler triggers crawl
2. For each active query, crawler runs on selected platforms
3. Results stored as citations
4. Metrics updated automatically

#### **3. Citation Tracking**

**Viewing Citations:**
1. Frontend calls `GET /v1/citations/brand/{brand_id}`
2. Backend queries citations with filters (platform, sentiment, date range)
3. Returns paginated list of citations
4. Frontend displays in table with filters

**Citation Details:**
- Each citation shows:
  - Platform name
  - Query text
  - Snippet from answer
  - Brand position (if in list)
  - Sentiment
  - Competitor mentions
  - Timestamp

#### **4. Competitor Analysis**

**Competitor Detection:**
1. Crawler parses answer text
2. Identifies brand names (excluding tracked brand)
3. Extracts positions if in numbered list
4. Stores in `competitors` JSON field

**Competitor Dashboard:**
1. Frontend calls `GET /v1/brands/{id}/competitors`
2. Backend aggregates competitor mentions from citations
3. Returns top competitors with mention counts
4. Frontend displays in chart/table

#### **5. Email Reports**

**Weekly Report Generation:**
1. Scheduler runs every Monday at 9 AM
2. For each organization:
   - Queries metrics for all brands
   - Generates HTML email with:
     - Visibility scores
     - Top queries
     - Competitor insights
     - Trend analysis
3. Sends email via SendGrid (if configured)
4. Stores report record in database

**Report Content:**
- Summary statistics
- Top performing queries
- Competitor mentions
- Platform breakdown
- Trend indicators

#### **6. Team Collaboration**

**Inviting Team Members:**
1. Owner/admin clicks "Invite Member"
2. Enters email address and role
3. Frontend calls `POST /v1/team/invite`
4. Backend creates `TeamInvite` record
5. Sends invitation email with token
6. User clicks link, creates account
7. Account automatically linked to organization

**Role-Based Access:**
- `owner`: Full access, can manage team
- `admin`: Can manage brands, queries, view all data
- `member`: Can view data, limited editing

**Implementation:**
- `get_current_user()` returns user with role
- Endpoints check role before allowing actions
- Frontend conditionally shows/hides features based on role

#### **7. API Keys & Webhooks**

**Creating API Key:**
1. User navigates to API settings
2. Clicks "Create API Key"
3. Frontend calls `POST /v1/api-keys`
4. Backend generates secure token
5. Returns token (shown once, must be saved)
6. Token stored in `APIKey` table with user/org association

**Using API Key:**
1. Client includes `Authorization: Bearer {api_key}` header
2. `verify_api_token()` dependency validates token
3. Finds `APIKey` record, checks expiration
4. Returns user associated with key
5. Request proceeds with user context

**Webhook Configuration:**
1. User creates webhook with URL and events
2. Backend stores in `Webhook` table
3. When event occurs (e.g., new citation):
   - Backend queries active webhooks for event type
   - Sends HTTP POST to webhook URL with event data
   - Retries on failure (exponential backoff)

#### **8. Integrations**

**Slack Integration:**
1. User connects Slack workspace
2. OAuth flow authorizes bot
3. Bot token stored in `Integration` record
4. When alert triggers:
   - Backend sends message to Slack channel
   - Includes metrics summary and link to dashboard

**Google Sheets Integration:**
1. User authorizes Google account
2. Access token stored
3. User selects spreadsheet and sheet
4. Backend periodically exports metrics to sheet
5. Uses Google Sheets API to update cells

**Zapier Integration:**
1. User creates Zapier webhook
2. Zapier receives webhook events
3. User configures Zapier workflow
4. Triggers actions in other apps

#### **9. Advanced Features (Phase 2 & 3)**

**Predictive Analytics:**
- Uses historical citation data
- Calculates trends and extrapolates
- Predicts future visibility scores
- Identifies growth opportunities

**Content Optimization:**
- Analyzes top-performing queries
- Suggests content improvements
- Provides keyword recommendations
- Generates content briefs

**Industry Benchmarking:**
- Aggregates anonymous metrics
- Compares brand performance to industry
- Shows percentile rankings
- Identifies competitive gaps

**Multi-Channel Tracking:**
- Extends beyond AI platforms
- Tracks YouTube, TikTok, Reddit, LinkedIn
- Unified dashboard for all channels
- Cross-channel insights

**Agency Platform:**
- White-label dashboard for agencies
- Client management interface
- Reseller program support
- Custom branding

---

### 🚀 How Deployment Works

#### **1. Backend Deployment (Railway/Render)**

**Preparation:**
1. Ensure all environment variables are documented
2. Test database migrations locally
3. Verify all dependencies in `requirements.txt`

**Deployment Steps:**
1. Push code to GitHub repository
2. Connect repository to Railway/Render
3. Configure environment variables:
   - `DATABASE_URL`: PostgreSQL connection string
   - `SECRET_KEY`: Secure random string
   - `FRONTEND_URL`: Frontend domain
   - Optional: `SENDGRID_API_KEY`, OAuth credentials, etc.
4. Set build command: `pip install -r requirements.txt`
5. Set start command: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
6. Deploy

**Post-Deployment:**
1. Run migrations: `alembic upgrade head` (via CLI or one-time job)
2. Verify health endpoint: `GET /health`
3. Test authentication flow
4. Monitor logs for errors

#### **2. Frontend Deployment (Vercel)**

**Preparation:**
1. Ensure `VITE_API_URL` is set correctly
2. Test build locally: `npm run build`
3. Verify all API calls use environment variables

**Deployment Steps:**
1. Push code to GitHub
2. Import project in Vercel
3. Configure environment variable:
   - `VITE_API_URL`: Backend API URL (e.g., `https://api.yourdomain.com`)
4. Deploy

**Post-Deployment:**
1. Verify frontend loads correctly
2. Test authentication flow
3. Check API connectivity
4. Monitor for CORS errors

#### **3. Database Setup (Supabase)**

**Setup Steps:**
1. Create project at https://supabase.com
2. Go to Settings → Database
3. Copy connection string (format: `postgresql://user:password@host:port/dbname`)
4. Update `DATABASE_URL` in backend environment variables
5. Run migrations: `alembic upgrade head`

**TimescaleDB (Optional):**
- Supabase supports TimescaleDB extension
- Enables time-series optimizations
- Useful for metrics and analytics
- Enable via Supabase dashboard

#### **4. Crawler Deployment (GitHub Actions)**

**Setup Steps:**
1. Go to repository Settings → Secrets and variables → Actions
2. Add secrets:
   - `DATABASE_URL`: Database connection string
   - `API_URL`: Backend API URL
   - `API_TOKEN`: Optional API key for authentication
3. Workflow file (`.github/workflows/crawler.yml`) automatically runs
4. Schedule: Every 6 hours (configurable in workflow)

**Workflow Execution:**
1. GitHub Actions runner starts
2. Checks out code
3. Sets up Python environment
4. Installs dependencies
5. Runs crawler script
6. Script connects to database and API
7. Executes crawls and posts results
8. Logs output for debugging

**Monitoring:**
- View workflow runs in GitHub Actions tab
- Check logs for errors
- Set up notifications for failures

#### **5. Domain & SSL**

**Custom Domain:**
1. Add custom domain in Vercel/Railway settings
2. Update DNS records as instructed
3. SSL certificate automatically provisioned (Let's Encrypt)

**CORS Configuration:**
- Backend CORS middleware allows frontend domain
- Update `allow_origins` in `app/main.py` if needed
- Include both `www` and non-`www` variants

---

### 🔌 How to Extend the System

#### **1. Adding a New Crawler**

**Steps:**
1. Create new file in `backend/app/crawler/` (e.g., `new_platform.py`)
2. Inherit from `AnswerEngineClient`:
```python
from .base import AnswerEngineClient, NormalizedResult

class NewPlatformClient(AnswerEngineClient):
    async def run_query(self, query: str, brand_name: str = None) -> NormalizedResult:
        # Implementation here
        return NormalizedResult(...)
```
3. Implement web scraping logic
4. Parse results and return `NormalizedResult`
5. Add to scheduler in `backend/app/crawler/scheduler.py`:
```python
elif platform == "new_platform":
    client = NewPlatformClient(headless=True)
```

**Testing:**
- Test crawler independently
- Verify `NormalizedResult` format
- Test with various queries
- Handle errors gracefully

#### **2. Adding a New API Endpoint**

**Steps:**
1. Create or update route file in `backend/app/api/routes/`
2. Define endpoint with decorator:
```python
@router.get("/new-endpoint")
async def new_endpoint(
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    # Implementation
    return {"data": "result"}
```
3. Include router in `backend/app/main.py`:
```python
from app.api.routes import new_route
app.include_router(new_route.router)
```
4. Create Pydantic schema in `backend/app/schemas/` if needed
5. Update frontend API client if needed

#### **3. Adding a New Database Model**

**Steps:**
1. Create model file in `backend/app/models/`:
```python
from sqlalchemy import Column, Integer, String
from app.core.db import Base

class NewModel(Base):
    __tablename__ = "new_table"
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
```
2. Create Alembic migration:
```bash
alembic revision --autogenerate -m "Add new_table"
```
3. Review migration file
4. Run migration: `alembic upgrade head`
5. Create Pydantic schema in `backend/app/schemas/`

#### **4. Adding a New Frontend Page**

**Steps:**
1. Create page component in `frontend/src/pages/`
2. Add route in `frontend/src/App.tsx`:
```typescript
<Route path="/new-page" element={<NewPage />} />
```
3. Add navigation link in `Layout.tsx` if needed
4. Create API function in `frontend/src/api/` if needed
5. Use `useAuth()` hook for authentication

#### **5. Adding a New Integration**

**Steps:**
1. Create integration service in `backend/app/services/integrations/`
2. Implement OAuth flow if needed
3. Store credentials in `Integration` model
4. Create API endpoints for setup
5. Implement event handlers to trigger integration
6. Add frontend UI for configuration

**Example (Slack):**
1. Create Slack app and get OAuth credentials
2. Implement OAuth callback handler
3. Store bot token in database
4. Create function to send messages
5. Call function when alerts trigger

---

---

## 🔒 Security

- ✅ OAuth 2.0 authentication
- ✅ API key authentication
- ✅ Rate limiting
- ✅ CORS protection
- ✅ Input validation
- ✅ SQL injection protection (SQLAlchemy ORM)
- ✅ DPDP compliance features

---

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

---

## 📄 License

MIT License

---

## 🙏 Acknowledgments

Built with 100% free-tier services to enable bootstrap MVP development.

**Services Used:**
- Supabase (Database)
- Railway/Render (Backend hosting)
- Vercel (Frontend hosting)
- SendGrid (Email - free tier)
- GitHub Actions (Crawler scheduling)

---

## 📞 Support

- **Documentation**: See `START_HERE.md` for quick start
- **API Keys**: See `REQUIRED_API_KEYS.md` for setup
- **Issues**: Open an issue on GitHub

---

**Status**: ✅ Production Ready | **Version**: 1.0.0 | **Last Updated**: 2024
