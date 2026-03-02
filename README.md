# Vikara — Voice Scheduling Agent

A real-time voice assistant that books Google Calendar events through natural conversation. Just speak your name, preferred date, start and end time, and meeting title — Vikara handles the rest.

**Live demo:** https://karan0sidhu.github.io/Vikara---Voice-Scheduling-Agent/

---

## Demo Video

[![Watch the demo](https://cdn.loom.com/sessions/thumbnails/42bd693cf2234c30b5c678511a99877f-with-play.gif)](https://www.loom.com/share/42bd693cf2234c30b5c678511a99877f)

▶️ **[Watch on Loom](https://www.loom.com/share/42bd693cf2234c30b5c678511a99877f)**

---

## How It Works

1. Click **Start Agent** and speak naturally
2. Vikara asks for your name, date, start time, end time, and meeting title
3. It reads back the full details for confirmation
4. Confirms and creates a real Google Calendar event with the correct start and end time

---

## Setup Instructions

### Step 1 — Get a Free Groq API Key (LLM)

Vikara uses Llama 3.1 via Groq — completely free, no credit card required.

1. Go to [console.groq.com](https://console.groq.com)
2. Sign up for a free account
3. Go to **API Keys** → **Create Key**
4. Copy the key (starts with `gsk_...`)
5. Paste it into the **Groq API Key** field on the app

> Free tier: 6,000 tokens/min, unlimited daily requests.

---

### Step 2 — Set Up Google Calendar API

#### 2a. Create a Google Cloud Project

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Click the project dropdown at the top → **New Project**
3. Give it a name (e.g. "Vikara") → **Create**

#### 2b. Enable the Google Calendar API

1. Go to **APIs & Services → Enable APIs & Services**
2. Search for **Google Calendar API**
3. Click it → **Enable**

#### 2c. Configure the OAuth Consent Screen

1. Go to **APIs & Services → OAuth consent screen**
2. Select **External** → **Create**
3. Fill in:
   - App name: `Vikara`
   - User support email: your Gmail
   - Developer contact email: your Gmail
4. Click **Save and Continue** through all steps
5. Back on the consent screen page, under **Test users** → **Add Users**
6. Add your Gmail address (e.g. `karan02sidhu@gmail.com`)
7. Click **Save**

> ⚠️ You must add yourself as a test user or Google will block the OAuth login with an "Access blocked" error.

#### 2d. Create OAuth Credentials

1. Go to **APIs & Services → Credentials**
2. Click **Create Credentials → OAuth Client ID**
3. Application type: **Web Application**
4. Under **Authorized JavaScript Origins**, add:
   ```
   https://karan0sidhu.github.io
   ```
5. Under **Authorized Redirect URIs**, add:
   ```
   https://karan0sidhu.github.io/Vikara---Voice-Scheduling-Agent/
   ```
6. Click **Create**
7. Copy the **Client ID** (ends in `.apps.googleusercontent.com`)
8. Paste it into the **Google OAuth Client ID** field on the app

---

### Step 3 — Use the App

1. Open the live URL in **Chrome or Edge** (required for voice recognition)
2. Paste your **Groq API key**
3. Select **Google Calendar** from the Calendar dropdown
4. Paste your **Google OAuth Client ID**
5. Click **Start Agent →**
6. Click the microphone button and speak naturally:
   - *"My name is Karan"*
   - *"Let's meet Monday from 4 to 6"*
   - *"Call it Product Sync"*
7. Vikara confirms the details including start and end time — click **Create Event ✓**
8. A Google OAuth popup will appear — sign in and allow access
9. The event is created and you'll see a **View event →** link

---

## Calendar Integration — How It Works

Vikara integrates with Google Calendar using the **Google Calendar REST API v3** with an **OAuth 2.0 implicit flow** — no backend server required.

### Authentication Flow

When the user clicks "Create Event", the app opens a Google OAuth popup requesting the `https://www.googleapis.com/auth/calendar.events` scope. This scope is the minimum required — it allows creating and editing events without access to other calendar data.

Google redirects back to the app URL with an `access_token` in the URL hash fragment. The popup detects it has landed back on the same origin and posts the token to the parent window via `window.postMessage`. If the popup is blocked by the browser, the app falls back to a same-tab redirect — saving the pending booking to `sessionStorage` first, then restoring and completing the booking automatically once the user returns.

The token is stored in `sessionStorage` so it persists across conversation resets within the same browser tab without requiring re-authorization.

### Event Creation

Once authenticated, the app makes a direct `POST` request to:

```
https://www.googleapis.com/calendar/v3/calendars/primary/events
```

The event payload is constructed from the details collected during the voice conversation. Start and end times are tracked as separate fields — if the user specifies a range like "4 to 6", both times are captured and used directly. If only a start time is given, the end time defaults to +1 hour automatically.

```json
{
  "summary": "Product Sync",
  "description": "Scheduled via Vikara voice assistant",
  "start": { "dateTime": "2026-03-07T16:00:00", "timeZone": "America/Toronto" },
  "end":   { "dateTime": "2026-03-07T18:00:00", "timeZone": "America/Toronto" }
}
```

The timezone is detected automatically from the user's browser using `Intl.DateTimeFormat().resolvedOptions().timeZone`, so events are always created in the user's local timezone.

On success, the Google Calendar API returns the created event object including an `htmlLink` — a direct link to view the event in Google Calendar, which is displayed to the user.

### Why No Backend?

The implicit OAuth flow and direct API calls from the browser mean there is no server, no token storage, and no credentials at risk. The access token lives only in `sessionStorage` for the duration of the tab session and is never sent anywhere except Google's own API endpoints.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Voice Input | Web Speech API (browser built-in) |
| Voice Output | Speech Synthesis API (browser built-in) |
| LLM | Llama 3.1 8B via Groq API |
| Calendar | Google Calendar API v3 |
| Frontend | Vanilla HTML/CSS/JS |
| Hosting | GitHub Pages |

---

## Troubleshooting

**"Access blocked: has not completed the Google verification process"**
→ Go to Google Cloud Console → OAuth consent screen → Test users → add your Gmail address.

**"redirect_uri_mismatch"**
→ Make sure the exact URL `https://karan0sidhu.github.io/Vikara---Voice-Scheduling-Agent/` (with trailing slash) is added to Authorized Redirect URIs in your OAuth client.

**No popup appears when clicking Create Event**
→ Allow popups for the site in your browser settings, or the app will fall back to a same-tab redirect automatically.

**Voice not working**
→ Use Chrome or Edge. Safari and Firefox do not support the Web Speech API. Make sure your microphone is allowed in browser permissions.

**Event created but not showing in Google Calendar**
→ Check that you're looking at the correct Google account. The event is added to the **primary** calendar of whichever account you authorized.

**Groq error: 401**
→ Your Groq API key is invalid or expired. Generate a new one at [console.groq.com](https://console.groq.com).

---

## Local Development

No build step needed — it's a single HTML file.

```bash
git clone https://github.com/karan0sidhu/Vikara---Voice-Scheduling-Agent.git
cd Vikara---Voice-Scheduling-Agent
# Open index.html in Chrome
```

> Note: Google OAuth will not work on `file://` URLs. For local OAuth testing, use a local server like `npx serve .` and add `http://localhost:3000` to your OAuth authorized origins.
