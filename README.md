# go_server

A minimal but production-ready Go backend that provides authentication, user management, chirp posting, refresh tokens, and webhook handling — built with PostgreSQL, sqlc, and Goose migrations.

---

## 🚀 Features 

* **User system**: register, login, update profile
* **Authentication**:

  * Argon2id password hashing
  * JWT access tokens
  * Refresh tokens stored in DB
  * Token revocation
* **Chirps API**: create, list, filter, get by ID, delete
* **Webhook endpoint** for payment provider (Polka)
* **Metrics**: counts static file hits
* **PostgreSQL with sqlc** (type-safe queries)
* **Goose migrations**

---

## 🗄 Database Models (Important)

### **users**

```
id (uuid) PK
email (text, unique)
hashed_password (text)
is_chirpy_red (bool)
created_at, updated_at
```

### **chirps**

```
id (uuid) PK
body (text, max 140 chars, bad-word filtering)
user_id (uuid FK users)
created_at, updated_at
```

### **refresh_tokens**

```
token (text) PK
user_id (uuid FK users)
expires_at (timestamp)
revoked_at (timestamp nullable)
created_at, updated_at
```

---

## ⚙️ Environment Variables

Create a `.env` file:

```
DB_URL=postgres://user:pass@localhost:5432/go_server?sslmode=disable
PLATFORM=dev
SECRET_KEY=your_jwt_secret
POLKA_KEY=your_webhook_key
```

---

## 📦 Install & Run

### 1. Install dependencies

```bash
go mod tidy
```

### 2. Run Goose migrations

```bash
go install github.com/pressly/goose/v3/cmd/goose@latest
goose -dir sql/schema postgres "$DB_URL" up
```

### 3. Start the server

```bash
go run .
```

Server runs on:

```
http://localhost:8080
```

---

## 📡 Main API Endpoints

### Users & Auth

```
POST /api/users         # register
POST /api/login         # login → JWT + refresh token
POST /api/refresh       # new access token
POST /api/revoke        # revoke refresh token
PUT  /api/users         # update user (auth required)
```

### Chirps

```
POST   /api/chirps              # create (auth required)
GET    /api/chirps              # list (optional: ?sort=desc, ?author_id=uuid)
GET    /api/chirps/{id}         # get by ID
DELETE /api/chirps/{id}         # delete own chirp only
```

### Webhooks

```
POST /api/polka/webhooks   # requires Authorization: ApiKey <POLKA_KEY>
```

### Admin

```
GET  /api/healthz
POST /admin/reset
GET  /admin/metrics
```

---
