# 🔐 Auth System — Deployment Guide

## What you have

| File         | Purpose                                          |
|--------------|--------------------------------------------------|
| `server.js`  | Node.js + Express backend API with SQLite DB     |
| `login.html` | Drop-in login page for your static Netlify site  |
| `.env.example` | Environment variables template                 |

---

## Step 1 — Deploy the backend to Railway (free)

1. Go to **https://railway.app** and sign up (free)
2. Click **New Project → Deploy from GitHub repo**
3. Push this `auth-backend/` folder to a GitHub repo first, then connect it
   - Or use **Railway CLI**: `npm install -g @railway/cli && railway login && railway init && railway up`
4. In Railway dashboard → your project → **Variables**, add:
   ```
   JWT_SECRET=your-long-random-secret-here
   ALLOWED_ORIGIN=https://your-site.netlify.app
   ```
5. Railway will give you a public URL like:
   `https://auth-backend-production-abc123.railway.app`

> **Alternative**: Deploy to **Render.com** (also free):
> New Web Service → connect repo → set environment variables → Deploy

---

## Step 2 — Connect your Netlify site

1. Open `login.html`
2. Find this line near the bottom:
   ```javascript
   const API_URL = "https://YOUR-BACKEND.railway.app"; // ← CHANGE THIS
   ```
3. Replace it with your actual Railway URL:
   ```javascript
   const API_URL = "https://auth-backend-production-abc123.railway.app";
   ```
4. Save the file and upload it to your Netlify site (drag & drop or git push)

---

## Step 3 — Add the login link to your existing pages

In any HTML file on your site, just link to `login.html`:

```html
<a href="/login.html">Sign In</a>
```

Or redirect unauthenticated users in JavaScript:

```javascript
// Paste this at the top of any protected page
const token = localStorage.getItem('auth_token');
if (!token) window.location.href = '/login.html';
```

To show the logged-in username anywhere:

```javascript
const user = JSON.parse(localStorage.getItem('auth_user') || '{}');
if (user.username) {
  document.getElementById('greeting').textContent = `Hello, ${user.username}!`;
}
```

---

## API Endpoints

| Method | Endpoint        | Auth required | Description              |
|--------|-----------------|---------------|--------------------------|
| POST   | `/api/register` | No            | Create a new account     |
| POST   | `/api/login`    | No            | Login, returns JWT token |
| GET    | `/api/me`       | Yes (Bearer)  | Get current user info    |
| GET    | `/api/users`    | Yes (Bearer)  | List all users           |

### Example: calling the API from your own JS

```javascript
const API_URL = "https://your-backend.railway.app";

// Login
const res = await fetch(`${API_URL}/api/login`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ identifier: "user@email.com", password: "secret123" })
});
const { token, user } = await res.json();

// Protected request
const me = await fetch(`${API_URL}/api/me`, {
  headers: { Authorization: `Bearer ${token}` }
});
```

---

## Security notes

- Passwords are hashed with **bcrypt** (12 rounds) — never stored in plain text
- Sessions use **JWT tokens** (7-day expiry) stored in `localStorage`
- CORS is restricted to your `ALLOWED_ORIGIN` domain
- Change `JWT_SECRET` to a long random string before going live:
  `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`
