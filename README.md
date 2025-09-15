# TimeMachine

<div align="center">

[![Chrome Web Store](https://img.shields.io/badge/Chrome%20Web%20Store-Available-brightgreen?logo=googlechrome)](https://chromewebstore.google.com/detail/timemachine/hjkicompionnablkpkgnplnacnnchjij)
[![Version](https://img.shields.io/badge/version-1.6.0-blue.svg)](https://github.com/HarshDev625/TimeMachine)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/HarshDev625/TimeMachine/pulls)

</div>

<p align="center">
  <img src="extension/icon128.png" width="128" height="128" alt="TimeMachine Logo">
</p>

<h3 align="center">Minimal time tracking, Focus Sessions, Guard blocking, and rich reports</h3>

TimeMachine is a lightweight Chrome extension + Node.js backend that automatically tracks active website time, groups it into categories, and generates rich PDF and HTML reports. It now includes Focus Sessions, a Guard for websites & keywords (with a themed blocked page), a Solver tracker, a polished Summary with medals, and 7 UI themes.

## ✨ Current Features

| Area | Highlights |
|------|------------|
| Auth | Inline email + password (30‑day JWT, no verification codes) |
| Tracking | Automatic per‑domain active time (minute slices, local timezone) |
| Classification | Editable categories: Work / Social / Entertainment / Professional / Other |
| Dashboard | Daily / Weekly / Monthly views + Quick Insights panel (Top Site, Focus vs Leisure, Balance Score, Category Mix) |
| Scoring | Productivity score (Work + Professional + 0.5×Other) + balance heuristic |
| Reports | Rich PDF (insights + ranked domains + per-domain session stats + charts) + HTML email reports via EmailJS (with charts) |
| Scheduling | Local daily / weekly / monthly report trigger (no external cron) |
| Resilience | Offline local buffering & retry; incremental 1‑min flush; 5‑min bulk sync |
| Focus Sessions | Preset durations with Start/Pause/Resume/Stop and daily stats |
| Guard | Block websites & keywords, Quick Block current site, in‑app confirm modal, optional in‑page keyword scan toggle |
| Blocked Page | Modern, theme‑aware page with Go Back / Start Focus / Open App buttons |
| Solver | Track problem‑solving sessions with categories and history cards |
| Theming | 7 UI themes (light, dark, cyberpunk, minimal, ocean, sunset, forest) |
| Feedback | In‑extension authenticated feedback submission |
| Privacy | Only domains + aggregated session durations stored (no full URLs) |

## 🆕 What’s New

### v1.6.0

- Added optional in‑page keyword scanning toggle (privacy control) in Guard tab
- Enhanced blocked page with action buttons (Close Tab, Open Dashboard, Start Focus)
- Pruned legacy modal authentication CSS (lighter bundle)
- Added `PRIVACY.md` with detailed data handling policy
- Manifest version bump, added `homepage_url`
- Minor copy and doc adjustments for clarity

### v1.5.1

- Removed all remaining device references (code & docs)
- Inline auth (no modal / verification codes) fully adopted
- Summary tab now contains the PDF Download button (removed from Settings)
- Report generator always returns a PDF (even with no activity – shows a note)
- Security docs aligned to bcrypt hashing

### v1.5.0

- Focus: refreshed Focus Sessions UI with presets and clear controls
- Guard: website + keyword blocking, Quick Block, in‑app confirm modal, and a modern theme‑aware blocked page
- Summary: top 3 sites highlighted with Gold/Silver/Bronze styling and normalized spacing
- Solver: redesigned session cards and quick start
- Theming: shared tokens across popup and blocked page
- Scheduler: “next scheduled” time shown in Settings
- Performance: popup.js memoized backend URL + event delegation for Guard lists
- Publish prep: cleaned manifest host permissions (removed localhost) and bumped version

### v2 Simplification

| Before | Now |
|--------|-----|
| Email verification codes | Direct email/password signup & login |
| Device tracking / IDs | Removed entirely |
| Separate device & utility scripts | Removed (cronJobs, dataCleanup, device-management) |
| Plain PDF summary | Enhanced PDF with charts + session table |
| UTC-based date (timezone drift) | Local date derivation for correct regional day boundaries |

## 📦 Repository Structure (Active Parts Only)

```
backend/
  index.js                # Express app + CORS + route mounting
  routes/
    auth.js               # /api/auth (signup, login, profile, settings)
    timeData.js           # /api/time-data (sync, reports, category patch)
    feedback.js           # /api/feedback (submit, list, admin ops)
    report.js             # /api/report (generate PDF)
  models/
  User.js               # Email/password user + settings & timezone
    TimeData.js           # Per user/date/domain aggregated sessions
    Feedback.js           # Feedback messages
  README.md               # Backend-only docs

extension/
  manifest.json           # MV3 config (service worker, permissions, resources)
  background.js           # Tracking engine (sessions, sync, idle handling)
  popup.html              # Main UI (tabs: Analytics, Summary, Focus, Guard, Solver)
  popup.js                # UI logic (charts, categories, focus, guard, solver, reports)
  blocked.html            # Theme-aware blocked page
  blocked.js              # Blocked page logic (actions & timer)
  guide.html         # In-extension user guide
  auth.js                 # Token storage & auth helpers
  config.js               # Dynamic base URL + overrides
  report-scheduler.js     # Local schedule logic (daily/weekly/monthly)
  css/
    style.css, analytics.css, summary.css, focus.css, guard.css, stopwatch.css, blocked.css
  icon16.png, icon48.png, icon128.png
```

Removed legacy files (device-authentication.js, utils/cronJobs.js, utils/dataCleanup.js, etc.) for clarity.

## 🔐 Authentication Flow

1. User signs up or logs in via inline form: POST `/api/auth/signup` or `/api/auth/login` (email, password).
2. Backend returns a JWT (30d). Token stored in both `localStorage` and `chrome.storage.local` (kept in sync lazily).
3. Popup load performs a cached token verify (POST `/api/auth/verify`, 5‑min cache) to avoid double login.
4. All protected calls send `Authorization: Bearer <token>`.
5. On expiry/invalid token it is cleared; user re-auths and any buffered sessions sync automatically.

## 🗄 Data Model (Essential Fields)

User:
```
email, password(hash:bcrypt), role, settings{ receiveReports, reportFrequency, categories(Map) }, timezone{name,offset}, lastActive
```
TimeData (unique per userEmail+date+domain):
```
userEmail, date(YYYY-MM-DD), domain, totalTime(ms), sessions[{startTime,endTime,duration}], category, timezone
```
Feedback:
```
userEmail, message, status, timestamp
```

## 🔌 Core API Endpoints

Auth:
- POST `/api/auth/signup`
- POST `/api/auth/login`
- POST `/api/auth/verify`
- GET  `/api/auth/profile`
- POST `/api/auth/update-settings`

Time Tracking:
- POST `/api/time-data/sync` (batch push sessions)
- GET  `/api/time-data/report/:userEmail?date=YYYY-MM-DD&endDate=...` (range list)
- PATCH `/api/time-data/category` (update a domain’s category for a date)

Reports:
- POST `/api/report/generate` (returns PDF binary) – includes charts + ranked domains + sessions (returns a minimal PDF with a “No activity” note if the day is empty).

Feedback:
- POST `/api/feedback/submit`
- GET  `/api/feedback/my`
- (Admin) GET `/api/feedback/all`, PATCH `/api/feedback/status/:id`

## 📄 Rich PDF Report Contents

| Section | Details |
|---------|---------|
| Header | Date, user, generated timestamp |
| Key Insights | Top site, main category share, unique domains, session medians/longest, focus ratio |
| Domain Table | Rank · Domain · Time · Category · Sessions · Avg Session · Longest Session · Active Span |
| Charts | Doughnut (category distribution) & Horizontal Bar (all site times) |

Rendered server‑side with `quickchart-js` + PDFKit (empty datasets produce a graceful minimal PDF instead of errors).

## ✉️ Email Reports (EmailJS)

- Send one-off reports or schedule them to send automatically from the background
- Works with your own EmailJS credentials (privacy-first, no central mail server)
- Uses HTML with embedded charts (QuickChart); plain-text fallback included
- Template tip: render the message variable using triple braces to avoid escaping HTML, e.g. `{{{message}}}`

Setup in the extension Settings:
1. Select EmailJS as the service
2. Enter Service ID, Template ID, and Public Key
3. In your EmailJS template, add variables: `to_email`, `subject`, `message`, `message_text`
4. Click “Send Test Email” to verify

## 🧠 Tracking Logic (background.js)

Event-driven session handling:
1. Tab activated / URL changed → close previous tab session (duration = now - start).
2. Start new session for active domain.
3. Alarms: incremental flush (1 min), bulk sync (5 min), stale session cutoff (15 min).
4. Idle / lock → end all active sessions; resume on activity.
5. Offline failures store sessions locally until next successful sync.

Date key is derived in LOCAL TIME (fixes prior off-by-one for positive timezones like IST).

## 🛠 Development Setup

Backend:
```bash
git clone https://github.com/HarshDev625/TimeMachine.git
cd TimeMachine/backend
npm install
cp .env.example .env   # (create one if not present)
# .env needs at least:
# MONGODB_URI=mongodb://localhost:27017/timemachine
# JWT_SECRET=your-long-secret
npm run dev
```

Extension (unpacked):
1. Open Chrome → `chrome://extensions` → enable Developer Mode.
2. Load unpacked → select `TimeMachine/extension` folder.
3. Click the extension icon → login → start browsing.
4. Click the help ( ? ) button anytime to open the bundled in‑extension user guide.

## 🔎 Table of Contents

1. Features
2. Quick Start
3. Architecture Overview
4. Data Model
5. API Endpoints
6. Tracking & Sync Logic
7. Reports
8. Development Setup
9. Security Notes
10. Email Reports (EmailJS)
11. Contribution Guide
12. License

## ⚡ Quick Start (End User)

1. Install from Chrome Web Store (link above) or load unpacked from `extension/`.
2. Open the popup → inline Sign up / Sign in (no verification codes).
3. Browse normally; active tab domain time is tracked automatically.
4. Reassign categories to adjust productivity scoring.
5. Guard tab: block distracting sites / keywords (use Quick Block for current site).
6. Focus Sessions: start a preset, pause/resume, stop to log.
7. Summary tab: use Download PDF for a daily report (Settings now only configures scheduling).
8. Press the ? help button for the bundled guide or view developer docs on GitHub.

## 🆘 In-Extension Help

An offline user guide (`extension/guide.html`) is bundled with the extension (open via the ? button) covering features, tabs, themes, scheduling, Guard, Focus, Solver, and troubleshooting. This README hosts developer documentation.

## ⚙ Tech Stack

| Layer | Tech |
|-------|------|
| Backend | Node.js, Express, Mongoose, PDFKit, quickchart-js, JSON Web Tokens |
| DB | MongoDB |
| Extension UI | Vanilla JS, Chart.js, HTML/CSS |
| Auth | JWT (Bearer), bcrypt password hashing |

## 🔒 Security Notes

- bcrypt hashing (configurable rounds) with per-hash salt
- JWT 30d expiry; no refresh token layer yet
- CORS restricted to extension origin(s) + localhost dev
- All time & feedback endpoints behind auth middleware

Planned improvements:
1. Short‑lived access + refresh token rotation
2. Rate limiting / anomaly detection
3. Optional encryption-at-rest for session payloads
4. Additional password strength validation

## 🧪 Testing Ideas (Not Yet Included)

Add tests for:
- auth (signup/login/verify invalid creds)
- time-data sync (capping >12h sessions, daily aggregation)
- report generation (PDF produced, MIME, size threshold)

## 🤝 Contributing

PRs welcome. Keep changes focused and include a brief description (screenshots for UI changes help). Open an issue to discuss bigger ideas first.

## 📜 License

MIT. See [LICENSE](LICENSE).

## 📑 Privacy

See [PRIVACY.md](PRIVACY.md) for details on data collected, processing, controls, and user choices (including disabling in‑page keyword scanning).

## 🛡️ Security

If you find a security issue, please open a private issue with minimal details and request a secure contact.

## ❤️ Credits

- Chart.js & QuickChart for visualization
- PDFKit for report generation
- All contributors & users providing feedback

---
<p align="center"><a href="https://github.com/HarshDev625/TimeMachine/issues">Report Bug</a> • <a href="https://github.com/HarshDev625/TimeMachine/issues">Request Feature</a></p>
