# Allintitle Checker Tool

A simple, SEO-optimized single-page website to check keyword competition using Google's `allintitle:` operator.

## Files

1.  **index.html**: The frontend interface.
2.  **server.js**: The backend Node.js server that fetches data from Google.
3.  **package.json**: Dependency configuration.

## Setup & Deployment

### 1. Backend Server (Render.com or similar)
This tool requires a backend server to bypass CORS and query Google.

1.  Upload `server.js` and `package.json` to a GitHub repository.
2.  Connect the repository to **Render.com** (Web Service).
3.  Set the Build Command to `npm install`.
4.  Set the Start Command to `node server.js`.
5.  Copy the provided URL (e.g., `https://your-app.onrender.com`) and update `index.html` line 168:
    ```javascript
    const response = await fetch(`https://your-app.onrender.com/api/check?q=${encodeURIComponent(keyword)}`);
    ```
    *Note: The current code is pre-configured for `https://backend-server-pup6.onrender.com/`.*

### 2. Frontend (Local or Hosted)
*   Simply open `index.html` in any browser.
*   Or host it on Netlify, Vercel, or GitHub Pages.

## Troubleshooting "Connection Error"

If you see "Connection Error":
1.  **Server Sleeping**: Free tier hosting (like Render) puts apps to sleep after inactivity. The first request takes 30-60 seconds to wake it up. Wait a minute and try again.
2.  **CORS**: Ensure your server has `app.use(cors())`.
3.  **Google Blocking**: If the server returns 429 errors, Google has blocked the server IP. Try again later or rotate proxies.

## Features

*   **SEO Optimized**: Schema markup, Open Graph tags.
*   **Keyword Analysis**: Categorizes competition (KGR compliant).
*   **Responsive**: Tailwind CSS.
