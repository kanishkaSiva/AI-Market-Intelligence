# AI Market — Design
_Last updated: 2026-01-29_

## 1. Architecture Overview
A typical MVP can be built as a **single-page web app (SPA)** + **REST API**:

- **Frontend:** React (Vite) or Next.js
- **Backend:** FastAPI (Python) or Node/Express
- **Database:** SQLite for local/MVP, Postgres for production
- **Storage (optional):** S3-compatible bucket for images
- **Auth:** JWT (bearer token) or cookie sessions

### Suggested Component Diagram
- Browser (React)
  - calls → API (FastAPI)
    - talks to → DB (SQLite/Postgres)
    - uploads to → Object Storage (optional)

## 2. Frontend Design

### 2.1 Routes
- `/` Home (featured/new/trending)
- `/browse` Search + filters + paging
- `/listing/:id` Listing detail
- `/login` `/register`
- `/account` Profile, favorites, submissions
- `/submit` Submit listing form
- `/admin` Admin dashboard (role protected)

### 2.2 State & Data Fetching
- Use a single API client module.
- Cache list/search results (React Query recommended).
- Debounced search input (300–500ms).

### 2.3 UI Components
- Header: logo, search bar, auth buttons
- Filters panel: category chips, pricing select, platform select
- Listing card grid
- Pagination / infinite scroll
- Listing detail view (tabs: Overview, Pricing, Links)
- Admin table for submissions (approve/reject modal)

## 3. Backend Design

### 3.1 API Style
- REST endpoints with predictable query params:
  - `q` (search query)
  - `category`, `pricing`, `platform`
  - `sort` (newest/popular)
  - `page`, `page_size`

### 3.2 Validation
- Pydantic models for request/response schemas.
- URL validation:
  - Must be HTTPS unless explicitly allowed.
  - Extract domain for de-duplication.

### 3.3 Auth & RBAC
- `/auth/login` returns JWT access token.
- Middleware to set `request.user`.
- Role guard for admin endpoints (`role == "admin"`).

### 3.4 Moderation Workflow
- `Submission` table stores payload + status.
- On approve:
  - Upsert into `Listing` table
  - Mark submission approved
  - Log audit event
- On reject:
  - Mark rejected + reason stored
  - Notify user (email optional)

## 4. Database Design (Relational)
### 4.1 Tables (Simplified)
- `users(id, email, password_hash, role, created_at)`
- `categories(id, name, slug, enabled)`
- `tags(id, name, slug, enabled)`
- `listings(id, name, domain, url, short_desc, long_desc, category_id, pricing_model, platform, status, created_at, updated_at)`
- `listing_tags(listing_id, tag_id)`
- `favorites(user_id, listing_id, created_at)`
- `submissions(id, submitter_id, payload_json, status, moderator_notes, created_at, updated_at)`
- `audit_logs(id, actor_id, action, entity_type, entity_id, metadata_json, created_at)`

### 4.2 Indexing
- Index on `listings(domain)` (unique)
- Index on `listings(category_id, pricing_model)`
- Full-text index (Postgres `tsvector`) on name/description/tags if available

## 5. Search Design
### MVP
- Simple LIKE/ILIKE search on name/description/tags (SQLite/Postgres).
### Upgrade Path
- Postgres full-text search with ranking.
- Optional Elasticsearch/Meilisearch later.

## 6. Deployment Design
### Development
- Run backend on `localhost:8000`
- Run frontend on `localhost:5173`
- Configure CORS for dev.

### Production
- Containerize (Docker):
  - Frontend served by Nginx (or Vercel for Next.js)
  - Backend served by Uvicorn/Gunicorn
- Use Postgres
- Environment variables:
  - `DATABASE_URL`
  - `JWT_SECRET`
  - `CORS_ORIGINS`
  - `STORAGE_BUCKET` (optional)

## 7. Error Handling & Observability
- Consistent error envelope:
  - `{"error": {"code": "...", "message": "...", "details": ...}}`
- Server logs: request id, method, path, status, latency
- Basic rate limiting on auth + submissions

## 8. Security Considerations
- Input sanitization for descriptions (if markdown/HTML is allowed).
- CORS locked to known origins.
- Admin endpoints protected by RBAC.
- Prevent SSRF if doing URL reachability checks.

## 9. Testing Strategy
- Backend:
  - Unit tests for validation and moderation logic
  - Integration tests for `/listings` search + filters
- Frontend:
  - Component tests for filters and listing cards
  - E2E smoke test: browse → detail → submit (optional)

## 10. Future Enhancements
- User reviews/ratings and reporting
- Verified listings and badges
- Personalized recommendations
- Import pipeline (curated sources)
