Snip.ly — URL Shortener
A full-featured URL shortener web application with user authentication, analytics, custom slugs, expiration dates, and QR code generation.

 Features
FeatureDetailsUser AuthRegister, log in, log out with hashed passwordsURL ShorteningBase62 key generation (6–7 char unique slugs)Custom SlugsChoose your own short key (3–30 chars, unique)Click AnalyticsPer-link click counts, daily trends, top referrersExpirationSet links to expire after N daysQR CodesGenerate & download QR codes for any short linkDashboardFull CRUD — view, edit, delete your linksResponsive UIClean dark UI, works on mobile and desktop

🗂️ Project Structure
urlshortener/
── app.py                  # Main Flask application
── urlshortener.db         # SQLite database (auto-created on first run)
── templates/
│   ├── base.html           # Shared layout, nav, QR modal
│   ├── index.html          # Landing page
│   ├── login.html          # Login page
│   ├── register.html       # Registration page
│   ├── dashboard.html      # URL list + stats
│   ├── create.html         # Create short URL form
│   ├── edit.html           # Edit URL form
│   ├── stats.html          # Per-link analytics
│   ├── 404.html            # Not found
│   └── expired.html        # Expired link page
└── static/
    ├── css/
    ├── js/
    └── qr/                 # QR code output (if saved server-side)

 Quick Start
1. Create a virtual environment
bashpython -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
3. Install dependencies
bashpip install flask werkzeug
4. Run the app
bashpython app.py
The app will be available.
The SQLite database is created automatically on first run.

Configuration
Set these environment variables before running (optional but recommended for production):
VariableDefaultDescriptionSECRET_KEYRandom bytesFlask session secret — set this in productionPORT5000Port to listen on
bashexport SECRET_KEY="your-random-secret-key-here"
python app.py

How It Works
Short Key Generation (Base62)
Each URL gets a unique 7-character key generated from a SHA-256 hash of the URL + a random salt, encoded in base62 (A–Z, a–z, 0–9). This gives 62⁷ ≈ 3.5 trillion possible keys.
pythonBASE_CHARS = string.ascii_letters + string.digits  # 62 chars

def base62_encode(num: int) -> str:
    result = []
    while num:
        result.append(BASE_CHARS[num % 62])
        num //= 62
    return "".join(reversed(result))
Collision is handled by retrying up to 10 times with a new random salt.
Custom Slugs
Users can optionally specify their own short key:

Must match ^[A-Za-z0-9_-]{3,30}$
Checked for uniqueness before saving

Expiration
Links can be set to expire after a configurable number of days. On redirect, the expiry timestamp is checked and expired links return HTTP 410 Gone.
Analytics
Every redirect is logged to a clicks table with:

Timestamp
Referrer URL
User-Agent string

The dashboard aggregates these into daily click counts and top referrers.

 Database Schema
sql-- Users
CREATE TABLE users (
    id       INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT    UNIQUE NOT NULL,
    email    TEXT    UNIQUE NOT NULL,
    password TEXT    NOT NULL,       -- bcrypt via werkzeug
    created  TEXT    DEFAULT (datetime('now'))
);

-- Short URLs
CREATE TABLE urls (
    id        INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id   INTEGER NOT NULL REFERENCES users(id),
    long_url  TEXT    NOT NULL,
    short_key TEXT    UNIQUE NOT NULL,
    title     TEXT,
    clicks    INTEGER DEFAULT 0,
    created   TEXT    DEFAULT (datetime('now')),
    expires   TEXT,                  -- NULL = never expires
    is_active INTEGER DEFAULT 1
);

-- Click events
CREATE TABLE clicks (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    url_id     INTEGER NOT NULL REFERENCES urls(id),
    clicked    TEXT    DEFAULT (datetime('now')),
    referrer   TEXT,
    user_agent TEXT
);

 API
The app exposes a lightweight JSON API for programmatic use.
POST /api/shorten
Shorten a URL (requires login session).
Request body:
json{ "url": "https://example.com/very/long/path" }
GET /api/check-key?key=myslug
Check if a custom slug is available.
Response:
json{ "available": true }

 Tech Stack

Backend: Python 3.12 + Flask 3.x
Database: SQLite (via Python's built-in sqlite3)
Auth: werkzeug.security (PBKDF2-SHA256 password hashing)
Frontend: Vanilla HTML/CSS/JS, Google Fonts (Syne + DM Mono)
QR Codes: Canvas-based QR generation (no external library)
No ORM — raw SQL for full transparency and zero dependencies


 Security Notes

Passwords are hashed with PBKDF2-SHA256 via Werkzeug (never stored in plain text)
All DB queries use parameterized statements (no SQL injection)
Session cookies are signed with SECRET_KEY
Short key lookups use indexed columns for performance
is_active flag allows soft-disabling links without deletion


 React Prototype
A self-contained React prototype (App.jsx) is included for rapid UI development and demo purposes. It runs entirely in the browser with mock data (no backend needed).
Demo credentials: alice / password
To run the prototype, paste App.jsx into StackBlitz or any React sandbox.

 Roadmap

 -Password reset via email
 -Custom domain support
 -Bulk URL import (CSV)
 -Link preview (Open Graph metadata fetch)
 -API key authentication
 -Rate limiting per user
- Docker / docker-compose setup


 License
MIT License — see LICENSE for details.

 Contributing
Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

Fork the repo
-Create a feature branch (git checkout -b feature/my-feature)
-Fork the repo
-Create a feature branch (git checkout -b feature/my-feature)
-Commit your changes (git commit -m 'Add my feature')
-Push to the branch (git push origin feature/my-feature)
-Open a Pull Request
Commit your changes (git commit -m 'Add my feature')
Push to the branch (git push origin feature/my-feature)
Open a Pull Request
