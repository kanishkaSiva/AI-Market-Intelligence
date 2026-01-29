# AI Market — Requirements
_Last updated: 2026-01-29_

## 1. Overview
AI Market is a web application that lists AI tools/products, lets users search/filter, view details, and (optionally) sign in to save favorites and submit new listings. The system includes an admin workflow to review/approve submissions.

## 2. Goals
- Help users quickly discover relevant AI tools by category, pricing, and use case.
- Provide accurate, structured listing pages (features, pricing, links, tags).
- Enable safe community submissions with moderation.

## 3. Non‑Goals
- Hosting/serving the AI tools themselves.
- Processing payments (unless added later).
- Real-time chat/support (unless added later).

## 4. Personas
- **Visitor:** browses listings, searches, views details.
- **Registered User:** can favorite/save lists, submit listings, manage profile.
- **Admin/Moderator:** reviews submissions, edits listings, manages categories.

## 5. Functional Requirements

### 5.1 Public Browsing
- View home page with featured/new/trending listings.
- Browse by categories (e.g., Writing, Image, Video, DevTools, Productivity).
- Listing cards show: name, short description, tags, pricing badge, rating/likes (optional).
- Listing detail page includes: long description, screenshots/logo, pricing model, website link, tags, category, last updated.

### 5.2 Search & Filtering
- Full-text search over name, description, tags.
- Filters:
  - Category (single/multi)
  - Pricing model (Free, Freemium, Paid, Trial)
  - Platform (Web, iOS, Android, Desktop, API)
  - Popularity (views/likes) and recency
- Sort options: Relevance, Newest, Most Popular, Rating.

### 5.3 Authentication & Accounts (optional but recommended)
- Email/password or OAuth (Google) sign-in.
- Password reset flow.
- Profile: display name, avatar (optional), saved favorites, submitted listings.

### 5.4 Favorites / Collections
- Users can favorite/unfavorite listings.
- View favorites list.
- (Optional) Collections: create named lists and add/remove listings.

### 5.5 Submissions
- Authenticated users can submit a listing with:
  - Name (required, unique-ish)
  - Website URL (required)
  - Short description (required)
  - Full description (optional)
  - Category (required)
  - Tags (optional)
  - Pricing model (required)
  - Screenshots/logo (optional)
- Submissions go to **Pending** state until approved.
- User can view submission status and edit/resubmit if rejected.

### 5.6 Admin / Moderation
- Admin dashboard to:
  - Review pending submissions (approve/reject with reason).
  - Edit existing listings (content, tags, category, status).
  - Manage categories and tags (create/rename/disable).
  - View audit log of actions.
- Basic abuse controls:
  - Rate limit submissions per user/day.
  - Blocklist domains if needed.

### 5.7 Content & Quality
- Validate URLs (format + reachable check optional).
- Prevent duplicate listings (same domain).
- Basic markdown support for descriptions (optional).
- Image upload constraints (size/type).

## 6. Non‑Functional Requirements

### 6.1 Performance
- Home and browse pages load in < 2s on typical broadband.
- Search/filter results returned in < 500ms for typical queries (excluding network).

### 6.2 Reliability & Availability
- Target uptime: 99.5% for MVP.
- Graceful error pages and retry on transient failures.

### 6.3 Security
- Passwords hashed (bcrypt/argon2).
- JWT or session-based auth.
- Input validation and output escaping to prevent XSS/SQL injection.
- CSRF protections if cookies are used.
- Role-based access control for admin endpoints.

### 6.4 Privacy
- Collect minimal personal data.
- Clear privacy policy for stored emails and analytics.

### 6.5 Observability
- Structured logging (request id, latency).
- Basic metrics (requests, errors).
- Admin audit log.

### 6.6 Accessibility
- WCAG-inspired: keyboard navigation, labels, contrast.

## 7. Data Model (High-Level)
- **User**: id, email, password_hash, role, profile fields, created_at
- **Listing**: id, name, domain, url, short_desc, long_desc, category_id, pricing_model, platform, tags[], status, created_at, updated_at
- **Submission**: id, listing_payload, submitter_id, status, moderator_notes, timestamps
- **Category**: id, name, slug, enabled
- **Tag**: id, name, slug, enabled
- **Favorite**: user_id, listing_id, created_at
- **AuditLog**: id, actor_id, action, entity_type, entity_id, metadata, created_at

## 8. API Requirements (Example)
- `GET /api/listings` (query, filters, paging, sort)
- `GET /api/listings/:id`
- `POST /api/auth/register`, `POST /api/auth/login`
- `POST /api/listings/:id/favorite` / `DELETE ...`
- `POST /api/submissions`
- Admin:
  - `GET /api/admin/submissions?status=pending`
  - `POST /api/admin/submissions/:id/approve`
  - `POST /api/admin/submissions/:id/reject`

## 9. Acceptance Criteria (MVP)
- Users can browse listings by category and search by keyword.
- Listing detail pages render correctly with validated links.
- Submissions workflow works end-to-end: submit → pending → approve → visible.
- Admin can approve/reject and edit listing content.
- Basic auth + favorites (if included) function correctly.

## 10. Open Questions
- Will ratings/reviews be part of MVP?
- What is the source of “trending” (views/likes/time-decay)?
- Do we need multi-language support?
