# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A real-time multilingual chatroom application where each user can set their preferred language and receive all messages automatically translated. Built with FastAPI (backend) and vanilla JavaScript (frontend).

## Architecture

### Backend (`backend/main.py`)
- **FastAPI application** with WebSocket support for real-time chat
- **Translation engine**: Uses `deep-translator` library with Google Translate
- **ConnectionManager** (lines 114-173): Manages WebSocket connections and per-user target languages
  - `target_lang_by_ws`: Maps each WebSocket to user's chosen language
  - `translator_cache`: Caches GoogleTranslator instances by language code
  - `broadcast_translated()`: Translates original message into each connected user's target language
- **Language normalization** (`normalize_lang`, lines 50-72): Accepts both language codes ("fr") and names ("french")
- **Compatibility layer** (`get_supported_languages_dict`, lines 32-48): Handles differences between deep-translator versions

### Frontend (`frontend/index.html`)
- Single-page HTML with Bootstrap 5 styling
- WebSocket client connecting to `ws://localhost:8000/ws`
- Users set target language via input field or `/lang` command
- Displays translated messages with language code prefix

## Key Protocols

### WebSocket Message Flow
1. On connect: Backend sends welcome info and language setup instructions
2. User sets language: Send `/lang <code-or-name>` or enter language as first message
3. Chat messages: Original text → backend translates → broadcasts to all users in their language
4. Message format: `{"type": "chat", "sender": "user", "original": "...", "target": "fr", "translated": "..."}`

### Language Handling
- Supports both ISO codes ("en", "fr", "es") and full names ("english", "french", "spanish")
- Auto-detection for source language ("auto" used in GoogleTranslator)
- First message can be interpreted as language selection if no target set

## Development Commands

### Backend
```bash
# Install dependencies
cd backend
pip install -r requirements.txt

# Run development server (auto-reload on changes)
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Run production server
uvicorn main:app --host 0.0.0.0 --port 8000
```

### Frontend
```bash
# Serve frontend (any static file server)
cd frontend
python -m http.server 8080
# Or use: python3 -m http.server 8080
```

Then open `http://localhost:8080` in multiple browser windows to test multi-user chat.

### Testing Translation API
```bash
# Get supported languages
curl "http://localhost:8000/languages"

# Translate text via HTTP
curl "http://localhost:8000/translate?text=Hello&target=french"
curl "http://localhost:8000/translate?text=Bonjour&target=en&source=fr"
```

## Configuration Notes

- **CORS**: Currently set to allow all origins (`allow_origins=["*"]`) - suitable for dev/testing
- **WebSocket URL**: Hardcoded in `frontend/index.html` line 27 as `ws://localhost:8000/ws`
- **Logging**: Configured at INFO level (backend/main.py line 12)

## Common Patterns

### Adding New WebSocket Message Types
Modify `broadcast_translated()` or add handlers in the `/ws` endpoint (lines 178-226). Message types are distinguished by the `type` field in JSON payload.

### Extending Translation Features
The `ConnectionManager` class is the central point for managing translations. Translator instances are cached in `translator_cache` dict to avoid recreation overhead.

### Handling deep-translator Version Differences
The `get_supported_languages_dict()` function (lines 32-48) handles API differences between versions. Use this pattern when adding features that depend on deep-translator methods.
