# Jogjagem API 🏯

The backend for the **Jogjagem** tourism ecosystem in Yogyakarta, Indonesia. It powers the public site and business portal ([`jogjagem`](../jogjagem)) and the internal staff console ([`jogjagem-admin`](../jogjagem-admin)).

Jogjagem API is a modular, production-oriented Go REST service built on the **Pleco** foundation (`pleco-api` is still the Go module name). It goes well beyond auth: it serves the tourism catalogue, AI travel features, business/listing workflows, advertising, payments, sales commissions, scraping, analytics, and the full admin/RBAC surface.

- Setup guide: [INSTALLATION.md](./INSTALLATION.md)
- Common issues: [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)
- Contributing: [CONTRIBUTING.md](./CONTRIBUTING.md)

[![Go](https://img.shields.io/badge/Go-1.25+-00ADD8?logo=go)](https://go.dev)
[![Gin](https://img.shields.io/badge/Gin-HTTP%20Framework-009688)](https://gin-gonic.com)
[![Database](https://img.shields.io/badge/Database-PostgreSQL%20%7C%20MySQL-336791)](https://github.com/kanjengadipati/jogjagem-api)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker)](https://www.docker.com)
[![AI Powered](https://img.shields.io/badge/AI-Multi--Provider-ff6b35)](https://github.com/kanjengadipati/jogjagem-api)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## Overview

Jogjagem API is organized around domain-focused modules. Each module owns its own handler, service, repository, and model, so the tourism catalogue, identity, monetization, AI, and operations surfaces can evolve independently.

**Tourism catalogue**

- Destinations, events, hotels, restaurants, guides, souvenirs, rentals, stories, articles, promotions, and reviews
- Config: categories, sub-regions, quotes, and SEO settings
- Location/region endpoints and a generated `sitemap.xml`
- Public image-report endpoint and moderated image reports

**AI features** (multi-provider, optional)

- Tourist assistant: chat query, image search, recommendations (single + multi), trending, journey planning, route timeline, next-stop
- AI content generation for destinations, events, and articles, with a content-quality gate
- AI-assisted audit-log investigation for admins
- AI-powered error optimization and optional AI error monitoring

**Identity, auth, and RBAC**

- Email/password, passwordless OTP (email/WhatsApp), magic links, and social login (Google, Facebook, Apple)
- JWT access tokens + rotating HttpOnly refresh cookie, per-device sessions, trusted devices
- Database-driven RBAC with fine-grained per-route permission checks
- Admin user/role/permission management and audit trail

**Business & monetization**

- Business registration/verification, listing claims, team invites
- Promotions, review replies, subscriptions
- Ad campaigns, house ads, placement pricing, impression/click tracking
- Midtrans payments and webhooks
- Sales referral codes, commissions, and bonuses

**Operations**

- Content scraping (Jadesta, InJourney, VisitingJogja) with a staging review queue
- Staging/content-queue review and AI review
- Analytics dashboard, notifications, monitoring (Sentry/Datadog)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Go 1.25+ |
| HTTP | Gin |
| ORM | GORM (`gorm.io/gorm`) |
| Database | PostgreSQL 15+ (default) or MySQL 8.4 |
| Migrations | golang-migrate v4 (embedded) |
| Auth | JWT (HS256), refresh-token rotation, RBAC |
| Cache / rate limit | Redis (fallback: in-memory) |
| Email | SMTP, SendGrid, Resend, MailerSend |
| WhatsApp / OTP | Fonnte, Meta WhatsApp Cloud API |
| Payments | Midtrans |
| AI | mock / Ollama / OpenAI / Gemini / Anthropic / Groq |
| Web search | Tavily |
| Scraping | `goquery` (HTML) |
| Monitoring | Sentry, Datadog |

---

## Architecture

![Architecture](docs/architecture.svg)

Client → Nginx gateway → Gin router → modules → services → PostgreSQL/Redis. Cross-cutting concerns (auth, RBAC, CORS, rate limiting, request IDs, structured logging, recovery, security headers) live in `internal/middleware`.

---

## Quickstart

### Local

```bash
cp .env.example .env
go run ./cmd/migrate
go run ./cmd/seed
go run ./cmd/api
```

The API listens on **http://localhost:8081** by default (`PORT` defaults to `8081` in code).

### Docker

```bash
cp .env.docker.example .env.docker
make docker-up
```

The gateway is exposed at **http://localhost**.

### Test

```bash
make test
# or
go test ./...
```

### Releases

Tagged releases publish prebuilt binary bundles for Linux, macOS, and Windows. Each archive contains the `api`, `migrate`, and `seed` executables plus the setup and troubleshooting docs, so you can bootstrap a host without compiling Go.

---

## Project Structure

```text
.
├── cmd/              # api, migrate, seed, sync_seeds, youtube entrypoints
├── docs/             # OpenAPI spec, Swagger UI, architecture diagram
├── internal/
│   ├── appsetup/     # composition root: router, server, bootstrap
│   ├── ai/           # multi-provider AI clients (6 providers) + fallback
│   ├── cache/        # Redis-or-memory cache store
│   ├── config/       # env, db, and app config with startup validation
│   ├── contentquality/  # AI copy quality gate
│   ├── domain/       # shared APIError and error codes
│   ├── erroroptimizer/  # AI-friendly error classification
│   ├── httpx/        # response envelope, pagination, validation
│   ├── middleware/   # auth, RBAC, CORS, rate limit, logging, security
│   ├── modules/      # 34 domain modules (see below)
│   ├── otp/          # OTP / magic-link channel interface
│   ├── providers/    # email, WhatsApp, payment provider factories
│   ├── scraper/      # Jadesta / InJourney / VisitingJogja + scheduler
│   ├── search/       # Tavily web-research client
│   ├── seeds/        # seed logic + destinations.json (362 records)
│   └── services/     # JWT, password hashing, email, monitoring
├── migrations/       # PostgreSQL SQL migrations (+ migrations/mysql/)
├── postman/          # Newman collections and environment
├── queries/          # ad-hoc SQL helpers
├── scripts/          # shell helpers (e.g. extensions setup)
├── tests/            # cross-module integration/handler tests + mocks
├── docker/           # db-setup image
├── nginx/            # reverse-proxy config
├── dockerfile, docker-compose*.yaml, Makefile
```

### Modules (`internal/modules/`)

`adcampaign`, `analytics`, `article`, `audit`, `auth`, `bonus`, `business`, `commission`, `config`, `destination`, `event`, `guide`, `hotel`, `imagereport`, `listingclaim`, `notification`, `payment`, `permission`, `promotion`, `quota`, `rental`, `restaurant`, `review`, `role`, `sitemap`, `social`, `souvenir`, `staging`, `story`, `subscription`, `token`, `tourist`, `trips`, `user`.

Each module follows the same layout: `dto.go`, `handler.go`, `model.go`, `module.go`, `repository.go`, `routes.go`, `service.go`.

---

## Environment Configuration

Copy one of the example files depending on your workflow:

- Local development: [`.env.example`](.env.example)
- Docker: [`.env.docker.example`](.env.docker.example)
- Production: [`.env.production.example`](.env.production.example)

The app validates critical configuration at startup and exits early (with an aggregated list of problems) when required values are missing.

### Core

```env
PORT=8081
APP_BASE_URL=http://localhost:8081
FRONTEND_URL=http://localhost:3001
CORS_ALLOWED_ORIGINS=http://localhost:3001,http://127.0.0.1:3001
TRUSTED_PROXIES=127.0.0.1,::1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
REQUEST_BODY_LIMIT_BYTES=1048576
# Optional (read directly, not in the example files):
GIN_MODE=release
COOKIE_DOMAIN=.jogjagem.com
```

### Database

```env
DB_DRIVER=postgres
DATABASE_URL=postgresql://postgres:password@localhost:5432/jogjagem?sslmode=disable
AUTO_RUN_MIGRATIONS=false
AUTO_RUN_SEEDS=false
DB_MAX_OPEN_CONNS=5
DB_MAX_IDLE_CONNS=2
DB_CONN_MAX_LIFETIME_MINUTES=30
```

### Auth / JWT

```env
JWT_SECRET=replace-with-a-strong-secret-at-least-32-bytes
ACCESS_TOKEN_EXPIRY_MINUTES=15
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=supersecret
```

### Redis

```env
REDIS_URL=
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0
```

### Email

```env
EMAIL_PROVIDER=disabled          # disabled | smtp | sendgrid | resend | mailersend
EMAIL_FROM=
EMAIL_FROM_NAME=Jogjagem
EMAIL_SMTP_HOST=
EMAIL_SMTP_PORT=587
EMAIL_SMTP_MODE=starttls         # starttls | tls | plain
```

### WhatsApp / OTP

```env
WA_PROVIDER=disabled             # disabled | fonnte | whatsapp_cloud
OTP_RATE_LIMIT_REQUESTS=5
OTP_RATE_LIMIT_WINDOW_SECONDS=3600
OTP_TARGET_COOLDOWN_SECONDS=60
```

### Social login

```env
SOCIAL_ACTIVE_PROVIDERS=google   # comma list: google, facebook, apple
SOCIAL_GOOGLE_CLIENT_ID=
SOCIAL_GOOGLE_CLIENT_SECRET=
SOCIAL_FACEBOOK_CLIENT_ID=
SOCIAL_FACEBOOK_CLIENT_SECRET=
SOCIAL_APPLE_CLIENT_ID=
SOCIAL_APPLE_CLIENT_SECRET=
```

### AI

```env
AI_ENABLED=false
AI_PROVIDER=gemini               # mock | ollama | openai | gemini | anthropic | groq
AI_MODEL=gemini-3.5-flash-lite
AI_API_KEY=
AI_BASE_URL=                     # required only for ollama
AI_TIMEOUT_SECONDS=30
AI_FALLBACK_PROVIDER=groq
AI_FALLBACK_MODEL=llama-3.3-70b-versatile
AI_FALLBACK_BASE_URL=https://api.groq.com/openai
AI_ADMIN_ENABLED=false
AI_ADMIN_PROVIDER=
AI_ADMIN_MODEL=gemini-3.5-flash
AI_ADMIN_API_KEY=
```

### Payments, search, scraping, monitoring

```env
MIDTRANS_MERCHANT_ID=
MIDTRANS_SERVER_KEY=
MIDTRANS_CLIENT_KEY=
MIDTRANS_IS_PRODUCTION=false

TAVILY_ENABLED=false
TAVILY_API_KEY=
TAVILY_MAX_RESULTS=5

SCRAPER_ENABLED=false
SCRAPER_DEST_SCHEDULE=0 0 1 * *
SCRAPER_EVENT_SCHEDULE=0 0 */3 * *
YOUTUBE_API_KEY=

MONITORING_PROVIDER=none         # none | sentry | datadog
SENTRY_DSN=
DATADOG_API_KEY=
AI_MONITORING_ENABLED=false
AI_MONITORING_ERROR_THRESHOLD=5
```

### Notes

- `DATABASE_URL` is the primary connection setting. `DB_DRIVER` supports `postgres`/`mysql`; if omitted it is inferred from the URL scheme.
- `AUTO_RUN_MIGRATIONS` / `AUTO_RUN_SEEDS` are optional startup flags. Keep them `false` for local and Docker workflows and run migrations/seeds manually instead.
- `PORT` defaults to `8081`. The Docker image exposes `8080` and nginx proxies to `app:8080`, so set `PORT=8080` inside containers (the production compose file pins it).
- `CORS_ALLOWED_ORIGINS` should list explicit frontend origins; credentialed refresh cookies are unsafe with a wildcard.
- `EMAIL_PROVIDER` also supports `resend` and `mailersend` via `EMAIL_API_KEY`.
- Without Redis, rate limiting and caching fall back to in-memory stores — fine for single-instance local development only.
- Some variables read by the code are not yet in the example files: `MIDTRANS_*`, `SCRAPER_SOURCES`, `AUTH_LOGIN_RATE_LIMIT_*`, `AUTH_REGISTER_RATE_LIMIT_*`, `GIN_MODE`, `COOKIE_DOMAIN`, and `ENV_NAME`. Add them if you need those features. `COOKIE_DOMAIN` defaults to `.jogjagem.com` when `GIN_MODE=release`.

---

## AI Capabilities

### Tourist features (`/ai`)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/ai/query` | Conversational travel assistant |
| POST | `/ai/image-search` | Find destinations from an image |
| GET | `/ai/recommend` | Personalized recommendations |
| GET | `/ai/recommend/multi` | Multi-traveller recommendations |
| GET | `/ai/trending` | Trending picks |
| POST | `/ai/journey` | Generate a journey/itinerary |
| GET | `/ai/route-timeline` | Route timeline for a trip |
| GET | `/ai/next-stop` | Suggest the next stop |
| POST | `/ai/generate-destination`, `/ai/generate-event`, `/ai/generate-article` | Content generation |

Providers are hand-rolled HTTP clients supporting **mock, Ollama, OpenAI, Gemini, Anthropic, and Groq**, with an optional fallback provider chain. `AI_ENABLED=false` keeps the whole platform usable without AI. Content generation is passed through a quality gate (`internal/contentquality`) that detects clichés and triggers regeneration.

### Audit log investigator

Admin-only, permission-gated workflow over the audit trail:

1. Filter logs with `GET /auth/admin/audit-logs`
2. Run an investigation with `POST /auth/admin/audit-logs/investigations`
3. Review the structured `summary`, `timeline`, `suspicious_signals`, and `recommendations`
4. Re-open saved investigations with `GET /auth/admin/audit-logs/investigations[/:id]`

Permissions: `audit.read` (list/export/read) and `audit.investigate` (create). Identical requests over the same snapshot are deduplicated, and creating an investigation is itself audited.

### Error optimization

When `AI_ENABLED=true`, backend errors are classified into internal codes and rewritten into user-friendly messages with actionable suggestions, then cached in Redis. If AI is unavailable, Pleco-style generic messages are returned as a safe fallback.

---

## Main Endpoints

Auth, admin, and health are documented in detail below. The tourism, business, and monetization surfaces are summarized by prefix.

### Auth

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Register a new user |
| POST | `/auth/login` | Login, receive an access token, set refresh cookie |
| POST | `/auth/passwordless/check` | Validate an email/WhatsApp identity |
| POST | `/auth/passwordless/start` | Start OTP or magic-link delivery |
| POST | `/auth/magic-link/verify` | Verify a magic link |
| POST | `/auth/request-otp`, `/auth/verify-otp` | OTP login |
| POST | `/auth/refresh` | Rotate the access token using the refresh cookie |
| GET | `/auth/verify` | Verify email |
| POST | `/auth/resend-verification`, `/auth/forgot-password`, `/auth/reset-password` | Recovery flows |
| POST | `/auth/social-login` | Google, Facebook, or Apple login |
| GET/PATCH | `/auth/profile` | Read / update the current profile |
| PATCH | `/auth/change-password` | Change password |
| GET | `/auth/social/:provider/account` | Linked social account |
| GET | `/auth/sessions` | List active sessions |
| POST | `/auth/logout`, `/auth/logout-all`, `/auth/logout-others` | Session revocation |
| DELETE | `/auth/sessions/:id` | Revoke a session |
| GET/POST | `/auth/referral-code` | Read / regenerate your referral code |

### Admin (identity & audit)

| Method | Endpoint | Description |
|---|---|---|
| GET/POST | `/auth/admin/users` | List / create users |
| GET/PUT/DELETE | `/auth/admin/users/:id` | Manage a user |
| GET | `/auth/admin/users/:id/permissions` | Effective permissions |
| GET | `/auth/admin/roles`, `/auth/admin/roles/:id` | Roles |
| GET | `/auth/admin/permissions` | Permissions |
| GET/PUT | `/auth/admin/roles/:id/permissions` | Role permissions |
| GET | `/auth/admin/audit-logs`, `/export` | Audit logs / CSV export |
| POST/GET | `/auth/admin/audit-logs/investigations[/:id]` | AI investigations |
| GET | `/auth/admin/promotions/pending` | Pending promotions |
| POST | `/auth/admin/promotions/:id/approve`, `/:id/reject` | Moderate promotions |
| GET/POST | `/auth/admin/businesses`, `/pending`, `/:id`, `/:id/approve`, `/:id/reject`, `/:id/suspend` | Moderate businesses |

### Tourism catalogue (public reads, permission-gated writes)

| Prefix | Resources |
|---|---|
| `/destinations` | list, search, hidden gems, by category, by id; `my-status` for users; image report |
| `/events` | list, search, by id |
| `/hotels`, `/restaurants`, `/guides`, `/souvenirs`, `/rentals`, `/stories`, `/promotions` | list, search, by id |
| `/articles` | list, search, by category/slug, by id |
| `/reviews` | list, search, by id; admin moderation |
| `/config` | categories, sub-regions, quotes, SEO |
| `/locations` | regions |
| `/trips` | authenticated trip planning (all routes require auth) |
| `/notifications` | authenticated notifications |
| `/sitemap.xml` | generated sitemap |

### Business & monetization

| Prefix | Purpose |
|---|---|
| `/businesses` | register a business, name check, invite accept |
| `/businesses/me/**` | profile, listings, members/invites, promotions, review replies, subscription, ad campaigns |
| `/listing-claims` | submit and track listing claims |
| `/admin/listing-claims` | approve/reject claims |
| `/ads` | banners, house ads, ecosystem ads, pricing, impression/click tracking |
| `/admin/analytics` | overview, top destinations, categories, sub-regions, activity, reports |
| `/admin/staging`, `/admin/content-queue` | content review workflows |
| `/admin/payments`, `/admin/bonuses`, `/admin/bonus-rules` | payments, bonuses, rules |
| `/sales/me/commissions`, `/bonuses/me` | sales self-service |
| `/admin/scrape` | trigger scrapers |
| `/webhooks/midtrans/notification` | Midtrans webhook (HMAC-SHA512 verified, no JWT) |

### Health

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Health check (legacy) |
| GET | `/health/live` | Liveness probe |
| GET | `/health/ready` | Readiness probe (checks DB) |
| GET | `/docs`, `/docs/openapi.yaml` | Swagger UI and OpenAPI spec |

---

## API Conventions

- Authenticated routes require `Authorization: Bearer <access_token>`.
- Admin routes use permission checks (`RequirePermission`), not role-only checks.
- Refresh tokens are issued as the `pleco_refresh_token` HttpOnly cookie and are only valid for `POST /auth/refresh`. Device identity is `pleco_device_id`.
- Access tokens carry a token-version claim; after a password change/reset, role change, or `logout-all`, previously issued tokens return `401`.
- Success envelope: `status`, `message`, optional `data`, optional `meta`.
- Error envelope: `status`, `message`, optional `errors`.
- OpenAPI reference: [`docs/openapi.yaml`](docs/openapi.yaml) (covers health/auth/admin; the tourism and business surfaces are broader). Swagger UI is served at `/docs`.

---

## Example Requests

### Login

```bash
BASE_URL=http://localhost:8081
COOKIE_JAR=/tmp/jogjagem-cookies.txt

TOKENS=$(curl -s -c "$COOKIE_JAR" -X POST "$BASE_URL/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email": "tester@example.com", "password": "Secret123!"}')

ACCESS_TOKEN=$(echo "$TOKENS" | jq -r '.data.access_token')
```

The response also sets the `pleco_refresh_token` and `pleco_device_id` HttpOnly cookies.

```json
{
  "status": "success",
  "message": "Login success",
  "data": { "access_token": "<jwt>" }
}
```

### Authenticated profile

```bash
curl "$BASE_URL/auth/profile" -H "Authorization: Bearer $ACCESS_TOKEN"
```

### Refresh and logout

```bash
curl -X POST "$BASE_URL/auth/refresh" -b "$COOKIE_JAR" -c "$COOKIE_JAR" \
  -H "Content-Type: application/json" -d '{}'

curl -X POST "$BASE_URL/auth/logout" \
  -H "Authorization: Bearer $ACCESS_TOKEN" -b "$COOKIE_JAR"
```

### Public catalogue

```bash
curl "$BASE_URL/destinations?page=1&limit=10"
curl "$BASE_URL/destinations/search?q=prambanan"
curl "$BASE_URL/events?page=1&limit=10"
curl "$BASE_URL/articles/slug/tips-berlibur-di-jogja"
```

### Admin: users and audit

```bash
curl "$BASE_URL/auth/admin/users?page=1&limit=10" \
  -H "Authorization: Bearer $ACCESS_TOKEN"

curl "$BASE_URL/auth/admin/audit-logs?resource=auth&status=failed" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Social login, sessions, passwordless OTP, and the full audit-investigation request/response shapes are documented in [`docs/openapi.yaml`](docs/openapi.yaml) and the Postman collections under [`postman/`](postman).

---

## Local Development

```bash
cp .env.example .env    # point DATABASE_URL at your database
go run ./cmd/migrate    # apply migrations
go run ./cmd/seed       # seed roles, permissions, admin, catalogue
go run ./cmd/api        # start the API on :8081
```

PostgreSQL needs the `pg_trgm` and `unaccent` extensions. Use [`scripts/setup-extensions.sh`](scripts/setup-extensions.sh) (or enable them from your provider console on managed databases).

---

## Docker Workflow

The Compose stack includes the app, Redis, an Nginx gateway, and a `db-setup` container that runs migrations and seeding.

```bash
make docker-up
make docker-down
make docker-logs
make docker-rebuild
```

- `DB_DRIVER=postgres` (default) uses `docker-compose.yaml` with PostgreSQL + PgBouncer.
- `DB_DRIVER=mysql` uses `docker-compose.mysql.yaml` with MySQL.
- Postgres is published on `127.0.0.1:5433`; MySQL on `127.0.0.1:3307`. The gateway is exposed at `http://localhost`.

### Production Deployment

Deploy only the API image and connect it to managed/private infrastructure:

- Managed PostgreSQL or MySQL via `DATABASE_URL`
- Managed Redis via `REDIS_URL` so rate limits and cache entries are shared across replicas
- TLS termination and public traffic at a load balancer / reverse proxy
- `AUTO_RUN_MIGRATIONS=false` and `AUTO_RUN_SEEDS=false` on the long-running service; run migrations once per deploy as a release/CI step

Use [`docker-compose.prod.example.yaml`](docker-compose.prod.example.yaml) as a minimal reference. Do not publish database ports such as `5432`/`3306` in production.

---

## Database & Migrations

- **PostgreSQL** (default) and **MySQL** are supported, selected by `DB_DRIVER` or inferred from the URL scheme.
- Migrations use golang-migrate v4 and are embedded in the binary (`migrations/embed.go`). Run them with `go run ./cmd/migrate`, via the `migrate` CLI Makefile targets, or with `AUTO_RUN_MIGRATIONS=true`.
- The embedded runner auto-recovers from a dirty state by forcing the previous version and retrying.
- PostgreSQL migrations live in `migrations/`; MySQL migrations in `migrations/mysql/`.
- Seeding (`go run ./cmd/seed`) creates roles, permissions, the admin user, and catalogue data (`internal/seeds/destinations.json` holds 362 destination records).

### Database tasks

```bash
make migrate-up
make migrate-down
make migrate-down-all
make migrate-status
make migrate-create NAME=create_example_table
make migrate-force VERSION=1
make migrate-drop CONFIRM=1
make seed
make db-setup
```

---

## Response Caching

When Redis is configured, hot auth/admin reads are cached; otherwise an in-memory store is used.

| Endpoint | Cache key | TTL |
|---|---|---|
| `GET /auth/admin/users/:id/permissions` | `user:permissions:{userID}` | 10 min |
| `GET /auth/profile` | `user:profile:{userID}` | 5 min |
| `GET /auth/admin/roles` | `roles` | 20 min |
| `GET /auth/admin/roles/:id` | `role:{roleID}` | 15 min |
| `GET /auth/admin/roles/:id/permissions` | `role:{roleID}:permissions` | 15 min |
| `GET /auth/admin/users/:id` | `user:detail:{userID}` | 5 min |
| `GET /auth/social/:provider/account` | `social:account:{userID}:{provider}` | 15 min |

Permission checks are cached for 10 minutes (`role:permission:{role}:{permission}`). User/role writes invalidate the related entries.

---

## Testing

```bash
make test          # go test ./...
make check         # fmt + test
```

Tests live alongside the code and under `tests/` (integration/handler tests using `testify` and `sqlmock`). Set `TEST_DATABASE_URL` to run the PostgreSQL integration tests.

### Manual testing with Postman / Newman

Collections: `pleco.postman_collection.json`, `pleco.docker.postman_collection.json`, `pleco.smoke.postman_collection.json`, `pleco.negative.postman_collection.json`, and `pleco.local.postman_environment.json`.

```bash
npm install            # installs Newman
npm run postman:local  # smoke suite (make postman-test)
npm run postman:manual # full manual collection
npm run postman:negative
npm run postman:all    # smoke + negative
```

The collections expect the API to already be running at the configured `base_url`.

---

## Makefile Reference

```bash
make help
make fmt
make test
make check
make postman-test
make postman-negative
make postman-all
make migrate-up
make migrate-down
make migrate-down-all
make migrate-status
make migrate-create NAME=create_example_table
make migrate-force VERSION=1
make migrate-drop CONFIRM=1
make seed
make db-setup
make docker-up
make docker-down
make docker-logs
make docker-rebuild
```

---

## Security Notes

- Never commit real secrets. Use secret managers / platform-managed env vars in production, and rotate any credential ever exposed.
- `JWT_SECRET` must be at least 32 bytes.
- Refresh tokens are HttpOnly, rotated on every use, and tracked by family so reuse revokes the whole family.
- Magic links are stored hashed and single-use; OTP codes are hashed, expire, and are guarded by cooldown/target limits.
- Rate limits cover login, register, refresh, social, OTP, and ad tracking. The default store is in-memory; configure Redis for multi-instance deployments.
- Security headers include CSP, HSTS (on HTTPS), `X-Content-Type-Options`, and `X-Frame-Options`; request IDs propagate via `X-Request-ID`.
- The app does not redirect HTTP → HTTPS; terminate TLS at the gateway/load balancer.

---

## Monitoring & Observability

Optional error monitoring via Sentry or Datadog:

```env
MONITORING_PROVIDER=sentry
SENTRY_DSN=https://key@sentry.io/project
# or
MONITORING_PROVIDER=datadog
DATADOG_API_KEY=your_key
```

AI-powered error analysis can classify patterns and store root causes, sampling errors to control cost:

```env
MONITORING_PROVIDER=sentry
SENTRY_DSN=...
AI_MONITORING_ENABLED=true
AI_MONITORING_ERROR_THRESHOLD=5
```

---

## Troubleshooting

See [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) for the full guide. Common cases:

- **`relation "users" does not exist`** — migrations have not run. Run `go run ./cmd/migrate` or `make db-setup`.
- **`email not verified` on login** — click the verification link, resend it, or mark the user verified in development.
- **AI errors** — verify `AI_ENABLED=true` and provider keys; for a quick check use `AI_PROVIDER=mock`.
- **Social login `email not available`** — request the email scope in the frontend OAuth flow.
- **Docker container unhealthy** — confirm `PORT=8080` inside the container and use `make docker-logs`.

---

## Project Metadata

- License: [MIT](LICENSE)
- Contributing: [CONTRIBUTING.md](CONTRIBUTING.md)
- Security policy: [SECURITY.md](SECURITY.md)
- Code of conduct: [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- CI workflow: [ci.yml](.github/workflows/ci.yml)

### Contact

- **GitHub Issues:** [kanjengadipati/jogjagem-api](https://github.com/kanjengadipati/jogjagem-api/issues)
- **Upstream foundation:** [pleco-dev/pleco-api](https://github.com/pleco-dev/pleco-api)

For security vulnerabilities, report privately — see [SECURITY.md](SECURITY.md).
