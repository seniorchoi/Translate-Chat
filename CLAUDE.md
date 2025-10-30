# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A real-time multilingual chatroom where messages are automatically translated into multiple languages. Uses a **global language pool** architecture: all connected users share the same set of translation languages, but each user can independently filter which languages they see. Built with FastAPI (backend) and vanilla JavaScript (frontend).

## Architecture

### Backend (`backend/main.py`)
- **FastAPI application** with WebSocket support for real-time chat
- **Static file serving** (lines 33-47): Serves `frontend/index.html` from root `/` path, works both locally and on Render deployment
- **Translation engine**: Uses `deep-translator` library with Google Translate
- **ConnectionManager** (lines 136-300): Central state manager for WebSocket connections
  - `global_languages`: Single shared list of language codes that apply to all users
  - `username_by_ws`: Maps each WebSocket to display name
  - `username_colors`: Assigns each user a unique pastel color (generated via HSL conversion)
  - `translator_cache`: Caches GoogleTranslator instances by language code
  - `broadcast_translated()`: Translates original message into ALL global languages, sends to all users
  - `broadcast_language_update()`: Syncs global language list to all connected clients
  - `broadcast_users_update()`: Syncs active user list to all connected clients
- **Language normalization** (`normalize_lang`, lines 72-94): Accepts both language codes ("fr") and names ("french"), returns ISO code or None
- **Compatibility layer** (`get_supported_languages_dict`, lines 54-70): Handles differences between deep-translator versions (class method vs instance method)

### Frontend (`frontend/index.html`)
- Single-page application with vanilla JavaScript, no frameworks
- **WebSocket URL auto-detection** (lines 452-454): Dynamically constructs `ws://` or `wss://` based on page protocol, works for both local dev and production
- **Global language management**: Any user can add/remove languages via sidebar or commands
- **Personal visibility filtering** (lines 470, 580-589): Users can hide/show specific languages from their view using localStorage
- **Mobile responsive** (lines 292-403, 668-737): Hamburger menu, touch-friendly buttons, responsive sidebar overlay
- **Message display**: Shows sender name with color, timestamp, original text, and translations in all active languages (respecting personal visibility filters)

## Key Protocols

### WebSocket Message Flow
1. **On connect**: Backend sends welcome info, command help, current global language list, and active users list
2. **Set name**: `/name <your-name>` → backend stores username → broadcasts updated user list to all
3. **Add language**: `/add-lang <code-or-name>` → backend adds to global list → broadcasts language update to all
4. **Remove language**: `/remove-lang <code-or-name>` → backend removes from global list → broadcasts language update to all
5. **Chat message**: Original text → backend translates to ALL global languages → broadcasts to all users with full translation set
6. **On disconnect**: Backend removes user → broadcasts updated user list to remaining users

### Message Format
```json
{
  "type": "chat",
  "sender": "username",
  "color": "#a8c2e0",
  "original": "Hello world",
  "translations": {
    "FR": "Bonjour le monde",
    "ES": "Hola mundo",
    "KO": "안녕하세요 세계"
  },
  "timestamp": "2025-01-15T10:30:00.123456"
}
```

### Language Architecture: Global Pool + Personal Filters
- **Global**: Languages are shared across all users. When User A adds French, everyone's messages translate to French.
- **Personal**: Each user can hide specific languages from their view via 🙈/👁️ buttons. Hidden languages are stored in browser localStorage and only affect that user's display (frontend filtering, lines 646-647).
- Supports both ISO codes ("en", "fr", "es") and full names ("english", "french", "spanish")
- Source language auto-detection ("auto" used in GoogleTranslator)

## Development Commands

### Local Development (Separate Backend/Frontend)
```bash
# Terminal 1: Start backend
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Terminal 2: Start frontend
cd frontend
python3 -m http.server 8080

# Open http://localhost:8080 in multiple browser windows to test multi-user chat
```

### Local Development (Integrated - Frontend Served by Backend)
```bash
# Single terminal: Backend serves frontend
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Open http://localhost:8000 (backend serves frontend/index.html at root)
```

### Testing Translation API (HTTP Endpoints)
```bash
# Get supported languages dictionary
curl "http://localhost:8000/languages"

# Translate text directly
curl "http://localhost:8000/translate?text=Hello&target=french"
curl "http://localhost:8000/translate?text=Bonjour&target=en&source=fr"
```

## Deployment

### Render Configuration (`render.yaml`)
- Service type: `web` (Python environment)
- Build: `pip install -r backend/requirements.txt`
- Start: `uvicorn backend.main:app --host 0.0.0.0 --port $PORT`
- Health check: `/languages` endpoint
- Backend serves frontend static files from root path

## Configuration Notes

- **CORS**: Set to `allow_origins=["*"]` (line 25) for dev/testing - consider restricting for production
- **WebSocket URL**: Auto-detected from page protocol and host (frontend lines 452-454), works for both `ws://localhost:8000` and `wss://` deployments
- **Logging**: INFO level (line 16)
- **Static files**: Frontend served via StaticFiles mount at `/static` and root index at `/` (lines 33-47)

## Common Patterns

### Adding New WebSocket Commands
Add command handlers in `/ws` endpoint (lines 305-375). Commands start with `/` and use `data.strip().lower().startswith("/command-name")` pattern. Always send personal responses via `manager.send_personal(websocket, {...})` and broadcast updates via `manager.broadcast_*()` methods.

### Adding New Message Types
Backend broadcasts messages with `type` field. Frontend handles them in `ws.onmessage` (lines 487-506). Existing types: `"language_update"`, `"users_update"`, `"chat"`, plus `info` and `error` fields for system messages.

### Extending ConnectionManager State
Add new state dictionaries/lists to `ConnectionManager.__init__` (lines 137-142). Create methods to modify state and broadcast updates (follow pattern of `add_language`/`broadcast_language_update`).

### Handling deep-translator Version Differences
Use the `get_supported_languages_dict()` pattern (lines 54-70): try class method first, fall back to instance method. Cache results globally to avoid repeated calls.
