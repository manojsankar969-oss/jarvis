# JARVIS - Voice AI Assistant Manual

## Overview

JARVIS (Just A Rather Very Intelligent System) is a voice-first AI assistant inspired by Tony Stark's AI from the MCU films. Originally designed for macOS, it has been modified to run on Windows.

**Key Features:**
- Voice interaction with JARVIS personality (British butler, dry wit)
- WebSocket-based voice interface (browser audio ↔ LLM ↔ TTS)
- Integration with Anthropic Claude API for intelligence
- Fish Audio TTS for JARVIS voice synthesis
- SQLite-based memory system
- Task management and project awareness

---

## Required API Keys

Create a `.env` file in the project root with the following:

```env
# REQUIRED API Keys
ANTHROPIC_API_KEY=your-anthropic-api-key-here
FISH_API_KEY=your-fish-audio-api-key-here

# OPTIONAL: Fish Audio voice model (defaults to JARVIS MCU voice)
# FISH_VOICE_ID=612b878b113047d9a770c069c8b4fdfe

# OPTIONAL: Your name (JARVIS will address you by name)
# USER_NAME=Tony

# OPTIONAL: Specific calendar accounts (macOS only)
# CALENDAR_ACCOUNTS=you@gmail.com,work@company.com
```

**Where to get API keys:**
- **ANTHROPIC_API_KEY**: https://console.anthropic.com (sign up and create API key)
- **FISH_API_KEY**: https://fish.audio (sign up and generate API key)

---

## Windows Compatibility Changes

The original JARVIS codebase was macOS-only, using AppleScript for:
- Terminal control (Terminal.app)
- Calendar access (Calendar.app)
- Mail access (Mail.app)
- Notes access (Notes.app)
- Screenshot capture (`screencapture`)
- System Events integration

### Modifications Made for Windows

#### 1. **server.py**
- Added platform detection using `sys.platform`
- Conditionally imports macOS modules only on Darwin (macOS)
- On Windows/Linux: Uses stub functions that return safe defaults
- Stubbed functions include:
  - `execute_action()`, `open_terminal()`, `open_browser()`
  - `get_active_windows()`, `take_screenshot()`
  - `get_todays_events()`, `get_upcoming_events()` (calendar)
  - `get_unread_messages()`, `search_mail()` (mail)
  - `get_recent_notes()`, `create_apple_note()` (notes)

#### 2. **actions.py**
- Added `IS_MACOS` and `IS_WINDOWS` platform flags
- `open_terminal()`: Uses `cmd` on Windows instead of Terminal.app
- `open_browser()`: Uses Windows `start` command instead of AppleScript
- `open_claude_in_project()`: Launches claude via cmd on Windows
- `prompt_existing_terminal()`: Returns "not supported" message on Windows
- `get_chrome_tab_info()`: Returns empty dict on Windows

#### 3. **screen.py**
- Added platform detection
- `get_active_windows()`: Uses PowerShell with Win32 API on Windows
- `take_screenshot()`: Uses PowerShell with System.Drawing on Windows
- On macOS: Uses original AppleScript + `screencapture` method

---

## Setup Instructions for Windows

### Prerequisites
- Python 3.10+ installed and in PATH
- Node.js and npm installed
- OpenSSL (for SSL certificate generation)
- Git (optional, for cloning)

### Step 1: Clone/Download the Repository
```bash
cd C:\workspace
# If using git:
git clone <repository-url> jarvis
# Or extract the ZIP file to C:\workspace\jarvis\jarvis-main
```

### Step 2: Create and Configure .env File
```bash
cd C:\workspace\jarvis\jarvis-main
copy .env.example .env
notepad .env
```
Add your API keys to the `.env` file.

### Step 3: Install Python Dependencies
```bash
cd C:\workspace\jarvis\jarvis-main
pip install anthropic fastapi uvicorn websockets pyyaml
pip install sniffio anyio distro docstring-parser jiter starlette annotated-doc
```

Or install from requirements.txt (some packages may need --no-deps):
```bash
pip install -r requirements.txt
```

### Step 4: Install Frontend Dependencies
```bash
cd C:\workspace\jarvis\jarvis-main\frontend
npm install
```

### Step 5: Generate SSL Certificates
Required for audio functionality in browsers:
```bash
cd C:\workspace\jarvis\jarvis-main
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=localhost"
```

### Step 6: Run the Backend Server
Open a new terminal:
```bash
cd C:\workspace\jarvis\jarvis-main
python server.py
```

You should see:
```
INFO:     Started server process [...]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

### Step 7: Run the Frontend
Open another terminal:
```bash
cd C:\workspace\jarvis\jarvis-main\frontend
npm run dev
```

You should see:
```
VITE v... ready in ... ms
➜  Local:   http://localhost:5173/
```

### Step 8: Access JARVIS
1. Open Chrome browser
2. Navigate to `http://localhost:5173`
3. Click to enable audio/microphone permissions
4. Speak to JARVIS!

---

## Feature Compatibility Matrix

| Feature | macOS | Windows | Notes |
|---------|-------|---------|-------|
| Voice interaction | ✅ | ✅ | Requires microphone + API keys |
| TTS (Fish Audio) | ✅ | ✅ | Requires FISH_API_KEY |
| LLM (Claude) | ✅ | ✅ | Requires ANTHROPIC_API_KEY |
| Memory system | ✅ | ✅ | SQLite-based, fully functional |
| Task management | ✅ | ✅ | Fully functional |
| Terminal control | ✅ | ⚠️ | Basic cmd support on Windows |
| Browser control | ✅ | ✅ | Uses `start` command on Windows |
| Calendar access | ✅ | ❌ | Apple Calendar only |
| Mail access | ✅ | ❌ | Apple Mail only |
| Notes access | ✅ | ❌ | Apple Notes only |
| Screenshot | ✅ | ⚠️ | PowerShell-based on Windows |
| Screen awareness | ✅ | ⚠️ | Limited on Windows |
| Claude Code spawn | ✅ | ⚠️ | Basic support via cmd |

---

## Architecture

```
JARVIS Architecture (Windows-Compatible)
├── Backend (Python/FastAPI)
│   ├── server.py (main server, WebSocket handler)
│   ├── actions.py (platform-aware system actions)
│   ├── screen.py (platform-aware screen capture)
│   ├── memory.py (SQLite memory system)
│   ├── work_mode.py (Claude Code sessions)
│   ├── browser.py (Playwright web automation)
│   └── Stub modules (calendar, mail, notes - Windows)
├── Frontend (Vite/TypeScript)
│   ├── orb.ts (Three.js particle visualization)
│   ├── voice.ts (Web Speech API + audio)
│   └── main.ts (Frontend state machine)
└── Communication
    └── WebSocket (JSON messages + binary audio)
```

---

## Troubleshooting

### Server won't start
- **ModuleNotFoundError**: Run the pip install commands above
- **ANTHROPIC_API_KEY not set**: Create `.env` file with your API key
- **Port 8000 in use**: Change port in server.py or kill existing process

### Frontend won't start
- **npm not found**: Install Node.js from https://nodejs.org
- **Dependencies missing**: Run `npm install` in the frontend folder

### No audio/voice detection
- **Microphone permission**: Allow mic access in Chrome
- **SSL required**: Ensure key.pem and cert.pem are generated
- **Fish API key**: Verify FISH_API_KEY is set correctly

### Features not working on Windows
- Calendar, Mail, and Notes features are macOS-only (Apple apps)
- Terminal control is basic on Windows (no Terminal.app equivalent)
- Screenshots use PowerShell (may need Windows SDK installed)

---

## Development Notes

### Files Modified for Windows Compatibility
1. `server.py` - Platform detection and conditional imports
2. `actions.py` - Platform guards for macOS-specific functions
3. `screen.py` - PowerShell-based alternatives for Windows

### Files NOT Modified (macOS-only)
- `calendar_access.py` - Not imported on Windows
- `mail_access.py` - Not imported on Windows
- `notes_access.py` - Not imported on Windows
- `helpers/get_events.py` - Not used on Windows
- `helpers/get_events.sh` - Not used on Windows

### Running in Development Mode
Backend with auto-reload:
```bash
cd C:\workspace\jarvis\jarvis-main
uvicorn server:app --reload --host 127.0.0.1 --port 8000
```

Frontend with hot reload:
```bash
cd C:\workspace\jarvis\jarvis-main\frontend
npm run dev
```

---

## License and Credits

JARVIS is inspired by Tony Stark's AI from the Marvel Cinematic Universe.
Built with:
- FastAPI (Python web framework)
- Anthropic Claude API (LLM intelligence)
- Fish Audio API (Text-to-Speech)
- Three.js (3D orb visualization)
- Web Speech API (Voice recognition)
- Vite (Frontend build tool)

---

**Last Updated**: May 2026
**Platform**: Windows (modified from original macOS-only version)
