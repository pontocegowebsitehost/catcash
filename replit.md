# Workspace

## Overview

pnpm workspace monorepo using TypeScript. **CatCash** — personal finance web app for two private users (Cezar and Lexie).

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **Frontend**: React + Vite + Tailwind CSS + Framer Motion + Recharts
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Auth**: Cookie-based sessions with bcryptjs password hashing
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)

## Structure

```text
artifacts-monorepo/
├── artifacts/              # Deployable applications
│   ├── api-server/         # Express API server (auth, categories, transactions, goals, dashboard)
│   └── finance-app/        # React + Vite frontend (personal finance dashboard)
├── lib/                    # Shared libraries
│   ├── api-spec/           # OpenAPI spec + Orval codegen config
│   ├── api-client-react/   # Generated React Query hooks
│   ├── api-zod/            # Generated Zod schemas from OpenAPI
│   └── db/                 # Drizzle ORM schema + DB connection
├── scripts/                # Utility scripts
│   └── src/seed-users.ts   # Seeds Cezar & Lexie accounts + default categories
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── tsconfig.json
└── package.json
```

## Application Features

### Authentication
- Private app — only 2 users allowed: `cezar` and `lexie`
- Cookie-based session auth (express-session + connect-pg-simple)
- Username whitelist enforced on backend
- Login, logout, change password
- Default password: `finance123` (users should change on first login)

### Accounts & Balances
- Create, edit, delete financial accounts (Checking, Wallet, Savings, etc.)
- Starting balance + automatic current balance calculation
- Balance = starting_balance + sum(income) - sum(expenses) per account
- Total balance across all accounts shown on dashboard
- Transactions can be assigned to accounts; balance updates automatically
- Ownership enforced: users can only see/modify their own accounts

### Dashboard
- Monthly income/expense/savings summary
- Category breakdown charts (Recharts)
- Per-account balances with color-coded positive/negative
- Total balance across all accounts
- Financial goals progress
- Recent transactions

### Transactions
- Create, edit, delete transactions (income/expense)
- Assign transactions to accounts (optional)
- Filter by month, year, category
- CSV export
- Recurring transaction flag

### Categories
- User-specific custom categories
- Default categories seeded: Food & Dining, Transportation, Housing, Utilities, Entertainment, Shopping, Health, Education, Savings, Other

### Goals
- Create financial goals with target amounts
- Track progress with current amount vs target
- Optional deadline

### UI
- Dark/light mode toggle
- Framer Motion animations
- Mobile-responsive sidebar navigation
- Stripe-inspired SaaS design

### PWA (Progressive Web App)
- manifest.json with app icons for iOS and Android
- Service worker (sw.js) with stale-while-revalidate caching for static assets
- Network-first with cache fallback for API data (dashboard, transactions, accounts)
- Apple-specific meta tags for standalone full-screen mode on iPhone
- Safe area inset handling for notched devices
- Service worker only registers in production builds
- Icons generated from logo: 120, 152, 167, 180, 192, 512px

### Security
- CSV export sanitizes formula injection characters (=, +, -, @)
- Category ownership validated on transaction create/update
- Account ownership validated on transaction create/update
- Error messages properly extracted from ApiError objects

## Database Schema

- **users**: id, username, display_name, password_hash, created_at
- **categories**: id, user_id, name
- **accounts**: id, user_id, name, starting_balance, current_balance, created_at, updated_at
- **transactions**: id, user_id, amount, type (income/expense), category_id, account_id, date, notes, is_recurring, created_at
- **goals**: id, user_id, name, target_amount, current_amount, deadline, created_at
- **session**: sid, sess, expire (managed by connect-pg-simple)

## API Routes

All routes under `/api`:
- `POST /auth/login` — Login with username/password
- `POST /auth/logout` — Logout
- `GET /auth/me` — Get current user
- `POST /auth/change-password` — Change password
- `GET /categories` — List categories
- `POST /categories` — Create category
- `PATCH /categories/:id` — Update category
- `DELETE /categories/:id` — Delete category
- `GET /accounts` — List accounts
- `POST /accounts` — Create account
- `PATCH /accounts/:id` — Update account
- `DELETE /accounts/:id` — Delete account (unlinks transactions)
- `GET /transactions` — List transactions (with month/year/categoryId filters)
- `POST /transactions` — Create transaction
- `PATCH /transactions/:id` — Update transaction
- `DELETE /transactions/:id` — Delete transaction
- `GET /transactions/export` — Export as CSV
- `GET /goals` — List goals
- `POST /goals` — Create goal
- `PATCH /goals/:id` — Update goal
- `DELETE /goals/:id` — Delete goal
- `GET /dashboard/summary` — Dashboard summary data

## Commands

- `pnpm run typecheck` — Full typecheck
- `pnpm --filter @workspace/api-spec run codegen` — Regenerate API client/Zod
- `pnpm --filter @workspace/db run push` — Push DB schema changes
- `pnpm --filter @workspace/scripts run seed-users` — Seed users and default categories
