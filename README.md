# Translation Chatroom

A real-time multilingual chat application where messages are automatically translated into multiple languages. Each user can customize their view while sharing a global language pool.

## Features

- 🌍 **Real-time Translation**: Messages are automatically translated into all active languages
- 👥 **Live User List**: See who's currently in the chat with colored name badges
- 🎨 **Unique User Colors**: Each user gets a randomly assigned pastel color
- 🕐 **Timestamps**: Every message shows when it was sent
- 🙈 **Personal Language Filtering**: Hide specific language translations from your view
- ⚡ **Global Language Management**: Any user can add/remove languages for everyone
- 📱 **Clean UI**: Modern sidebar layout with scrollable messages

## How It Works

1. **Join the chat** - Set your display name
2. **Add languages** - Type a language (e.g., "french", "spanish", "korean", or codes like "fr", "es", "ko")
3. **Start chatting** - Your messages get translated to all active languages
4. **Customize your view** - Hide languages you don't want to see (click 🙈/👁️ button)

### Translation Behavior

- Languages are **global** - when anyone adds a language, everyone's messages translate to it
- Visibility is **personal** - hide languages from your view without affecting others
- Example:
  - User A adds French, Spanish
  - User B adds Korean
  - Everyone's messages now translate to French, Spanish, Korean
  - User A can hide Korean from their view while User B sees all three

## Tech Stack

### Backend
- **FastAPI** - Modern Python web framework
- **WebSockets** - Real-time bidirectional communication
- **deep-translator** - Google Translate integration
- **Python 3.12+**

### Frontend
- **Vanilla JavaScript** - No frameworks, pure JS
- **WebSocket API** - Real-time chat
- **localStorage** - Persist language visibility preferences
- **CSS Flexbox** - Responsive layout

## Installation

### Prerequisites
- Python 3.12 or higher
- pip (Python package manager)

### Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd translation_chatroom-main
   ```

2. **Install backend dependencies**
   ```bash
   cd backend
   pip install -r requirements.txt
   ```

## Running the Application

### Start the Backend Server
```bash
cd backend
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

The backend will be available at `http://localhost:8000`

### Start the Frontend Server
Open a new terminal:
```bash
cd frontend
python3 -m http.server 8080
```

The frontend will be available at `http://localhost:8080`

## Usage

### Setting Your Name
```
1. Enter your name in the "Your Name" field
2. Click "Set"
3. You'll appear in the Active Users list
```

### Managing Languages
```
Add: Type language name or code (e.g., "french" or "fr") → Click "Add"
Remove: Click × button next to language
Hide/Show: Click 🙈/👁️ button to toggle visibility
```

### Sending Messages
```
1. Type your message in the bottom input
2. Click "Send" or press Enter
3. Message appears with translations in all active languages
```

### Commands
- `/name <your-name>` - Set your display name
- `/add-lang <language>` - Add a language (e.g., `/add-lang french`)
- `/remove-lang <language>` - Remove a language (e.g., `/remove-lang fr`)

## Supported Languages

Supports 100+ languages through Google Translate, including:
- English (en)
- French (fr)
- Spanish (es)
- German (de)
- Italian (it)
- Portuguese (pt)
- Arabic (ar)
- Hindi (hi)
- Japanese (ja)
- Korean (ko)
- Russian (ru)
- Swahili (sw)
- Chinese (zh-CN)
- And many more...

## Project Structure

```
translation_chatroom-main/
├── backend/
│   ├── main.py              # FastAPI server with WebSocket endpoints
│   └── requirements.txt     # Python dependencies
├── frontend/
│   └── index.html          # Single-page application
├── CLAUDE.md               # Claude Code development guide
└── README.md               # This file
```

## API Endpoints

### HTTP Endpoints
- `GET /languages` - Get list of supported languages
- `GET /translate?text=...&target=...&source=...` - Translate text

### WebSocket
- `ws://localhost:8000/ws` - Real-time chat connection

## Development

The application uses auto-reload for both frontend and backend during development:
- Backend: `uvicorn` with `--reload` flag watches for Python file changes
- Frontend: Refresh browser to see HTML/JS/CSS changes

## License

This project is open source and available under the MIT License.

## Contributing

Contributions are welcome! Feel free to submit issues and pull requests.
