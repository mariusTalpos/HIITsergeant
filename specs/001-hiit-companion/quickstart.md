# Quickstart: HIITsergeant Development

**Branch**: `001-hiit-companion` | **Date**: 2026-02-24

This guide helps developers get the HIITsergeant codebase running locally for the 001-hiit-companion feature.

## Package Manager & Node Version Policy

- **Package manager**: `pnpm` (required for monorepo). Install: `npm install -g pnpm` or `corepack enable && corepack prepare pnpm@latest --activate`
- **Node.js**: 20+ (LTS). Use `nvm use 20` or `.nvmrc` if present. CI and contributors MUST use Node 20+.
- **npm**: Supported as fallback; `pnpm` preferred for workspace hoisting and speed.

## Prerequisites

- **Node.js** 20+
- **pnpm** 8+
- **PostgreSQL** 15+ (local or Docker)
- **Expo**: Use `npx expo` (no global install). Expo SDK via `apps/mobile` dependencies.
- **iOS**: Xcode 15+ (macOS only), iOS Simulator
- **Android**: Android Studio with SDK 34+, Android Emulator

## Repository Structure

```text
HIITsergeant/
├── api/                 # Backend (Node.js + Fastify)
├── apps/
│   ├── mobile/          # React Native (Expo) - iOS + Android
│   └── web/             # Web app (Phase 2+)
├── packages/
│   └── shared/          # Shared types, validation
└── specs/001-hiit-companion/   # This feature's specs
```

---

## MVP Setup (Backend + Mobile)

### 1. Clone and Install

```bash
git clone https://github.com/your-org/HIITsergeant.git
cd HIITsergeant
pnpm install
```

### 2. Environment Variables

Copy the example env and configure:

```bash
cp api/.env.example api/.env
```

**Env var matrix**:

| Variable | Required | MVP Default | Notes |
|----------|----------|-------------|-------|
| `DATABASE_URL` | Yes | `postgresql://localhost:5432/hiitsergeant` | PostgreSQL connection string |
| `NODE_ENV` | No | `development` | |
| `PORT` | No | `3000` | API port |
| `FIREBASE_PROJECT_ID` | Auth | (empty) | Firebase project for passwordless auth |
| `FIREBASE_CLIENT_EMAIL` | Auth | (empty) | Service account for token verification |
| `FIREBASE_PRIVATE_KEY` | Auth | (empty) | Service account key (base64) |
| `STRIPE_SECRET_KEY` | Payments | `sk_test_...` | Stripe test key for web |
| `STRIPE_WEBHOOK_SECRET` | Webhooks | (empty) | For local: use Stripe CLI |
| `REVENUECAT_API_KEY` | Mobile IAP | (empty) | RevenueCat API key |
| `API_BASE_URL` | Mobile | `http://localhost:3000` | Override for device networking (see below) |

**Local auth/payment limitations**:
- **Auth**: Firebase emulator or test project. Production Firebase requires configured OAuth clients (Apple/Google). For local dev, use Firebase Auth emulator or mock tokens if implemented.
- **Stripe**: Use `sk_test_*` keys. Webhooks: run `stripe listen --forward-to localhost:3000/api/webhooks/stripe` for local testing.
- **RevenueCat**: Sandbox IAP only on simulator/emulator. Physical device required for real IAP testing.

### 3. Database

```bash
cd api
pnpm db:migrate
pnpm db:seed
```

### 4. Run Backend

```bash
cd api
pnpm dev
```

API at `http://localhost:3000`. Health check: `curl http://localhost:3000/health` (or `/` if defined).

### 5. Run Mobile (Expo)

Use `npx expo` (no global Expo CLI):

```bash
cd apps/mobile
npx expo start
```

- **iOS Simulator**: Press `i` in terminal, or `npx expo start --ios`
- **Android Emulator**: Press `a` in terminal, or `npx expo start --android`
- **Physical device**: Install Expo Go, scan QR code. Ensure device and machine are on same network.

---

## Mobile-to-Local API Networking

| Target | API_BASE_URL | Notes |
|--------|--------------|-------|
| **iOS Simulator** | `http://localhost:3000` | Simulator shares host network |
| **Android Emulator** | `http://10.0.2.2:3000` | Emulator uses 10.0.2.2 for host loopback |
| **Physical device** | `http://<your-machine-ip>:3000` | e.g. `http://192.168.1.5:3000`; ensure firewall allows port 3000 |

Set in `apps/mobile/.env` or `apps/mobile/app.config.js`:

```bash
# apps/mobile/.env
EXPO_PUBLIC_API_URL=http://localhost:3000
```

For Android emulator, use `http://10.0.2.2:3000`. For physical device, use your machine's LAN IP.

---

## Optional: Phase 2 Web Setup

Web app is Phase 2+; not required for MVP.

```bash
cd apps/web
pnpm install
pnpm dev
```

Web at `http://localhost:5173` (Vite default). Ensure API is running and `VITE_API_URL` points to backend.

---

## Key Scripts

| Command | Location | Purpose |
|---------|----------|---------|
| `pnpm dev` | api/ | Start API dev server |
| `pnpm test` | api/ | Run Vitest (unit + integration) |
| `pnpm test:contract` | api/ | Run API contract tests |
| `npx expo start` | apps/mobile | Start Expo dev server |
| `pnpm test` | apps/mobile | Run Jest unit tests |
| `maestro test flows/` | apps/mobile | Run Maestro E2E |
| `pnpm lint` | root | Lint all packages |

---

## Feature Flag Examples

Flags are read from env or a backend config. For local dev:

```bash
# api/.env
FEATURE_GROUP_WORKOUTS=false
FEATURE_ENTERPRISE=false
FEATURE_WEB_WORKOUT=false
```

Or in code (example):

```typescript
// packages/shared/src/features.ts
export const features = {
  groupWorkouts: process.env.FEATURE_GROUP_WORKOUTS === 'true',
  enterprise: process.env.FEATURE_ENTERPRISE === 'true',
  webWorkout: process.env.FEATURE_WEB_WORKOUT === 'true',
};
```

MVP: all Phase 2 flags `false`. Phase 2: enable `groupWorkouts`, `enterprise` as needed.

---

## Seed Data Scenarios

| Scenario | Command | Purpose |
|----------|---------|---------|
| **Minimal** | `pnpm db:seed` | 1 user, 2 plans, 3 sessions |
| **With achievements** | `pnpm db:seed -- --achievements` | Adds unlocked achievements |
| **Enterprise** | `pnpm db:seed -- --enterprise` | Org, invites, members (Phase 2) |
| **Reset** | `pnpm db:migrate:reset && pnpm db:seed` | Fresh DB with seed |

(Adjust to match actual seed script interface in repo.)

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| **`pnpm install` fails** | Ensure Node 20+ (`node -v`). Delete `node_modules`, `pnpm-lock.yaml`, run `pnpm install` again. |
| **Expo "Unable to resolve module"** | Run `pnpm install` from repo root. Clear Metro cache: `npx expo start --clear` |
| **Mobile can't reach API** | Check `EXPO_PUBLIC_API_URL`. Simulator: `localhost`. Android emulator: `10.0.2.2`. Physical: machine IP. |
| **Database connection refused** | Ensure PostgreSQL is running. Check `DATABASE_URL`. Docker: `docker run -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:15` |
| **Firebase auth errors** | Verify `FIREBASE_*` env vars. Use emulator or test project. |
| **Stripe webhook signature invalid** | Use Stripe CLI: `stripe listen --forward-to localhost:3000/api/webhooks/stripe`; set `STRIPE_WEBHOOK_SECRET` from CLI output. |
| **Maestro E2E fails** | Ensure app is built and running. Use `maestro test --env APP_URL=...` if needed. |

---

## First Successful Run Verification Checklist

- [ ] `pnpm install` completes without errors
- [ ] `cd api && pnpm db:migrate && pnpm db:seed` succeeds
- [ ] `cd api && pnpm dev` starts; `curl http://localhost:3000/health` returns 200
- [ ] `cd apps/mobile && npx expo start` starts; no red errors in Metro
- [ ] iOS Simulator: press `i`; app loads (splash or login)
- [ ] Android Emulator: press `a`; app loads
- [ ] Create a plan in app (or via API)
- [ ] Start and complete a solo workout; session appears in history
- [ ] `pnpm lint` passes at repo root
- [ ] `cd api && pnpm test` passes

---

## Constitution Alignment

- **Linting**: Run `pnpm lint` before commit; CI enforces.
- **Tests**: Run `pnpm test` in api/ and apps/mobile; CI blocks on failure.
- **Performance**: Timer precision validated in E2E; profile with React DevTools / Flipper.

## Further Reading

- [plan.md](./plan.md) – Implementation plan and architecture
- [data-model.md](./data-model.md) – Entity definitions
- [contracts/api-overview.md](./contracts/api-overview.md) – API contract
- [research.md](./research.md) – Technology decisions
- [.specify/memory/constitution.md](../../.specify/memory/constitution.md) – Code quality principles
