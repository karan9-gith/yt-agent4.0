# ContentPilot AI — Comprehensive Setup Guide

This guide walks you through setting up ContentPilot AI for local development and operations.

---

## 1. Prerequisites & System Dependencies

1. **Node.js**: Version `v20.x` or later (LTS recommended).
2. **FFmpeg & FFprobe**: Version `6.0` or later installed and accessible via system `PATH`.
   - **Windows**: Install via `winget install Gyan.FFmpeg` or extract from [gyan.dev](https://www.gyan.dev/ffmpeg/builds/) and add `bin` to PATH. Verify via `ffmpeg -version`.
   - **macOS**: `brew install ffmpeg`
   - **Linux (Ubuntu/Debian)**: `sudo apt-get update && sudo apt-get install -y ffmpeg`
3. **SQLite**: Embedded automatically via Prisma (no standalone daemon required).

---

## 2. Repository Installation

Clone the repository and install dependencies in both backend and frontend workspaces:

```bash
# Clone the repository
git clone https://github.com/your-org/contentpilot-ai.git
cd contentpilot-ai

# Install root dependencies
npm install

# Install backend dependencies
cd backend
npm install

# Install client dependencies
cd ../client
npm install
cd ..
```

---

## 3. Environment Configuration

Copy the environment template and configure your secrets:

```bash
cp .env.example .env
```

### Essential Environment Variables:

| Variable | Description | Default / Example |
| :--- | :--- | :--- |
| `NODE_ENV` | Application environment (`development`, `production`, `test`) | `development` |
| `PORT` | Backend HTTP API port | `3001` |
| `CORS_ORIGIN` | Allowed client origin for CORS | `http://localhost:5173` |
| `APP_SECRET_KEY` | 32+ byte string used for AES-256 token encryption | *Set a secure random key* |
| `DATABASE_URL` | SQLite file connection string | `file:./dev.db` |
| `AUTO_PUBLISH` | Auto-publishing policy (**Must default to false**) | `false` |
| `REQUIRE_HUMAN_APPROVAL`| Mandatory human editorial review | `true` |
| `GEMINI_API_KEY` | Google AI Studio API key | Obtain from Google AI Studio |
| `GEMINI_MODEL` | Gemini LLM model | `gemini-2.5-flash` |
| `YOUTUBE_CLIENT_ID` | Google Cloud OAuth 2.0 Client ID | `your_oauth_client_id` |
| `YOUTUBE_CLIENT_SECRET` | Google Cloud OAuth 2.0 Client Secret | `your_oauth_client_secret` |
| `YOUTUBE_REDIRECT_URI` | OAuth 2.0 authorized callback redirect URI | `http://localhost:3001/api/auth/youtube/callback` |
| `STORAGE_DIR` | Directory for rendered videos and audio assets | `./storage` |

---

## 4. Google Gemini API Setup

1. Visit [Google AI Studio](https://aistudio.google.com/).
2. Create or select a Google Cloud project.
3. Generate a new API key.
4. Set in `.env`:
   ```env
   GEMINI_API_KEY=your_gemini_api_key
   GEMINI_MODEL=gemini-2.5-flash
   ```

---

## 5. YouTube Data API v3 & OAuth 2.0 Setup

1. Open [Google Cloud Console](https://console.cloud.google.com/).
2. Enable the **YouTube Data API v3**.
3. Navigate to **APIs & Services -> OAuth consent screen**:
   - User Type: External
   - Add scopes:
     - `https://www.googleapis.com/auth/youtube.upload`
     - `https://www.googleapis.com/auth/youtube.readonly`
     - `https://www.googleapis.com/auth/youtube.force-ssl`
     - `https://www.googleapis.com/auth/yt-analytics.readonly`
4. Navigate to **APIs & Services -> Credentials**:
   - Create Credentials -> **OAuth 2.0 Client IDs**.
   - Application type: **Web application**.
   - Name: `ContentPilot AI Studio`.
   - Authorized redirect URIs:
     `http://localhost:3001/api/auth/youtube/callback`
5. Copy Client ID and Client Secret into `.env`:
   ```env
   YOUTUBE_CLIENT_ID=your_client_id.apps.googleusercontent.com
   YOUTUBE_CLIENT_SECRET=your_client_secret
   YOUTUBE_REDIRECT_URI=http://localhost:3001/api/auth/youtube/callback
   ```

---

## 6. Database Initialization

Run Prisma code generation and apply database schema migrations:

```bash
cd backend
npx prisma generate
npx prisma db push
cd ..
```

---

## 7. Development & Running Commands

### Run Backend Development Server:
```bash
cd backend
npm run dev
# Starts backend server on http://localhost:3001
```

### Run Frontend Development Dashboard:
```bash
cd client
npm run dev
# Starts Vite React dashboard on http://localhost:5173
```

---

## 8. Test Commands

Run the comprehensive test suite (all 9 suites, unit, integration, and E2E):

```bash
cd backend
npm test
```

Run TypeScript compilation check:
```bash
cd backend
npm run typecheck

cd ../client
npm run build
```

---

## 9. Database Backup Instructions

Because ContentPilot AI uses SQLite with Write-Ahead Logging (`WAL`), create consistent hot backups using the safe SQLite backup API or copy when idle:

```bash
# Hot backup using sqlite3 CLI
sqlite3 backend/prisma/dev.db ".backup 'backend/prisma/backup-$(date +%Y%m%d%H%M%S).db'"
```

---

## 10. Known Limitations

1. **Daily YouTube Quota**: Standard Google Cloud projects receive 10,000 YouTube Data API units/day. A video upload costs 1,600 units. ContentPilot AI auto-fix schedules backoff retries when quota is exhausted.
2. **Local FFmpeg Performance**: Video render speed depends on CPU hardware encoding capabilities.
