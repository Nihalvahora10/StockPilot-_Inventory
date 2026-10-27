# StockPilot — Inventory & Order Management API

A production-style Python backend project for managing products, stock movements, and customer orders. It demonstrates layered architecture, validation, authentication, role-based access control, transactional inventory updates, audit events, tests, Docker, and CI.

> This is a portfolio/demo project, not a certified production system. Review security settings, migrations, observability, and deployment controls before real-world use.

## Features
- FastAPI REST API with generated OpenAPI docs
- SQLAlchemy 2 data models and SQLite-by-default configuration (PostgreSQL supported)
- JWT bearer authentication with password hashing
- Role-based access: `admin`, `manager`, `viewer`
- Product CRUD and low-stock filtering
- Inventory adjustments with a transaction ledger
- Order creation that reserves stock atomically; cancellation returns stock
- Audit log records for important business actions
- Pagination, request validation, consistent error responses
- Pytest tests, Dockerfile, Docker Compose, GitHub Actions CI
- Seed command for demo users and products

## Quick start

### 1. Create an environment
```bash
python -m venv .venv
# Windows PowerShell:
.venv\Scripts\Activate.ps1
# macOS/Linux:
source .venv/bin/activate
```

### 2. Install dependencies
```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Configure environment
```bash
cp .env.example .env
```
On Windows, copy `.env.example` to `.env` manually if `cp` is unavailable. Change `SECRET_KEY` before exposing the service.

### 4. Initialize and seed the database
```bash
python -m app.seed
```

### 5. Run the API
```bash
uvicorn app.main:app --reload
```
- Swagger UI: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc
- Health check: http://127.0.0.1:8000/health

## Demo accounts
Created by `python -m app.seed`:
- Admin: `admin@stockpilot.local` / `AdminPass123!`
- Manager: `manager@stockpilot.local` / `ManagerPass123!`
- Viewer: `viewer@stockpilot.local` / `ViewerPass123!`

Demo credentials are for local development only. Never deploy them.

## Main API routes

| Method | Route | Purpose | Access |
|---|---|---|---|
| POST | `/api/v1/auth/login` | Obtain access token | Public |
| GET | `/api/v1/auth/me` | Current user profile | Authenticated |
| GET/POST | `/api/v1/products` | List/create products | Viewer+ / Manager+ |
| GET/PATCH/DELETE | `/api/v1/products/{id}` | Read/update/delete product | Viewer+ / Manager+ |
| POST | `/api/v1/inventory/adjustments` | Adjust stock and create ledger entry | Manager+ |
| GET | `/api/v1/inventory/transactions` | Browse stock ledger | Authenticated |
| POST | `/api/v1/orders` | Create order and reserve stock | Manager+ |
| GET | `/api/v1/orders` | Browse orders | Authenticated |
| GET | `/api/v1/orders/{id}` | Order details | Authenticated |
| POST | `/api/v1/orders/{id}/cancel` | Cancel order and restore stock | Manager+ |
| GET | `/api/v1/audit-logs` | Browse audit history | Admin only |

`Manager+` means manager or admin. `Viewer+` means any authenticated role.

## Example API flow

1. Log in at `/api/v1/auth/login` with JSON:
   ```json
   {"email":"admin@stockpilot.local","password":"AdminPass123!"}
   ```
2. Copy `access_token` from the response.
3. In Swagger UI, click **Authorize** and enter `Bearer <token>` (or paste the token if Swagger's bearer dialog asks only for the token).
4. Create a product:
   ```json
   {
     "sku": "KB-100",
     "name": "Mechanical Keyboard",
     "description": "Compact keyboard",
     "unit_price": 79.99,
     "quantity": 25,
     "reorder_level": 5
   }
   ```
5. Create an order:
   ```json
   {
     "customer_name": "Jordan Lee",
     "items": [{"product_id": 1, "quantity": 2}]
   }
   ```

## Project layout
```text
stockpilot-python/
├── app/
│   ├── api/routes/        # HTTP endpoints
│   ├── core/              # Settings, security, permissions
│   ├── models/             # SQLAlchemy entities
│   ├── schemas/            # Request/response validation
│   ├── services/           # Business logic and transactions
│   ├── database.py
│   ├── main.py
│   └── seed.py
├── tests/
├── .github/workflows/ci.yml
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── .env.example
```

## Quality checks
```bash
pytest
ruff check .
```

## Design notes
- HTTP routing is kept separate from business logic.
- Stock changes and ledger writes happen in the same database transaction.
- Order creation checks availability and decrements stock in one transaction.
- `sku` and user email are unique.
- Passwords are hashed; plaintext passwords are never stored.
- Role checks are enforced at API boundaries.
- SQLite is convenient for local development; PostgreSQL is configured through `DATABASE_URL`.
- `create_all()` is used to keep this starter easy to run. For a production deployment, add Alembic migrations and a managed secrets strategy.
