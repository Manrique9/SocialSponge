# SocialSponge

Mobile events + social app, written entirely in Python.

- **`mobile/`**: the app. [Flet](https://flet.dev) (Flutter under the hood) builds native iOS and Android binaries from Python.
- **`backend/`**: REST API. FastAPI + SQLAlchemy, SQLite in development, PostgreSQL when hosted.

## Run it locally

```bash
# Create a virtual environment
python -m venv .venv

# Activate the virtual environment on Windows
.\.venv\Scripts\Activate.ps1

# 1. Backend API
# Open the first terminal

pip install -r backend/requirements.txt
cd backend
python -m uvicorn app.main:app --reload

# API documentation:
# http://127.0.0.1:8000/docs

# 2. Mobile App
# Open a second terminal from the project root

# Activate the virtual environment
.\.venv\Scripts\Activate.ps1

# Install mobile dependencies
pip install -r mobile/requirements.txt

# Run the mobile app
cd mobile
python main.py
```

### macOS and Linux

### macOS and Linux

```bash
# Create a virtual environment
python3 -m venv .venv

# Activate the virtual environment
source .venv/bin/activate

# 1. Backend API
# Open the first terminal

pip install -r backend/requirements.txt
cd backend
python -m uvicorn app.main:app --reload
```

Open a second terminal from the main SocialSponge folder.

```bash
# 2. Mobile App

# Activate the virtual environment
source .venv/bin/activate

# Install mobile dependencies
pip install -r mobile/requirements.txt

# Run the mobile app
cd mobile
python main.py
```

Keep the backend running while using the mobile app.

Demo accounts: `admin@socialsponge.app` / `admin12345` (admin) and `<firstname>@school.edu` / `password123`.

Handy overrides:

| Variable | Purpose |
| --- | --- |
| `SC_API_URL` | Point the app at another API (e.g. `http://192.168.1.20:8000` for a real phone). Android emulator uses `10.0.2.2` automatically. |
| `SC_DEV_LOCATION` | `lat,lng` fake GPS for simulators/desktop. |
| `SC_DATABASE_URL` / `DATABASE_URL` | Backend DB. `postgresql://…` (or `postgres://…`) for a hosted instance. |
| `SC_SECRET_KEY` | **Required** when `SC_DEBUG=0`. |

## Hosted database

Create a managed Postgres (Neon, Supabase, Railway, RDS…), set `DATABASE_URL`, `SC_SECRET_KEY` and `SC_DEBUG=0` on your host, and start `uvicorn app.main:app`. Tables are created on startup; run `python -m app.seed --no-demo` once for categories, interests and clubs. Then set `PROD_API_URL` in `mobile/config.py`.

## Build binaries

```bash
cd mobile
flet build apk        # Android
flet build ipa        # iOS (needs macOS + Xcode)
```

Location and notification permission strings are declared in `mobile/pyproject.toml`.

## Tests

```bash
cd backend && pytest        # API: auth, events, moderation, RSVP, search, orders, admin, messages
cd mobile  && pytest tests  # builds every screen against a live API process with a fake page
```

## What's where

| Area | Backend | Mobile |
| --- | --- | --- |
| Auth, forgot password | `routers/auth.py` | `screens/auth.py` |
| Onboarding + interests | `routers/me.py` | `screens/onboarding.py`, `components/interests.py`, `screens/settings.py` |
| Dashboard, recommendations | `recommendations.py`, `routers/events.py` | `screens/home.py` |
| Search, filters, categories | `routers/events.py`, `catalog.py` | `screens/discover.py` |
| Near You map | `geo.py` | `screens/near.py` |
| Calendar | `/events/calendar/summary` | `screens/calendar.py` |
| Event page, RSVP, attendees, share | `routers/events.py` | `screens/event.py` |
| Create / edit / cancel event | `routers/events.py` | `screens/event_form.py` |
| Payment + tickets | `routers/orders.py`, `payments.py` | `screens/payment.py` |
| Clubs, messages | `routers/messages.py`, `catalog.py` | `screens/clubs.py`, `screens/messages.py` |
| Profile, settings | `routers/me.py` | `screens/profile.py`, `screens/settings.py` |
| Admin (users, events, categories) | `routers/admin.py` | `screens/admin.py` |

## Known gaps (be aware before launch)

- **Maps** use `flutter_map` with OpenStreetMap tiles, not the Google/Apple Maps SDKs. Swap `MAP_TILE_URL` for a commercial tile provider before release (OSM's public server is for light use only).
- **Payments** use a mock provider (`backend/app/payments.py`). Implement the same interface with Stripe or similar.
- **Push notifications**: the toggle is wired to the OS permission and stored per user, and in-app notifications work. There is no FCM/APNs sender yet; device tokens can already be registered via `POST /me/devices`.
- **Language** preference is saved, but UI strings are English only.
- **Email** (password reset codes) prints to the server console in dev; plug an SMTP/API sender into `backend/app/mailer.py`.
- The mobile UI has been exercised through the integration test above, not yet on a physical device or simulator.
