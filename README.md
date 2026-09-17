# My-concert-tracker

# `/lab2` Folder Documentation

This folder contains all the JavaScripts files.

# `/lab2/public` Folder Documentation

This folder contains all the static files (front-end HTML/CSS/JS) that make up the interface of the **Concert Tracker / My Music** application.

## File Structure

web_proj/lab2/
├── data/                        
├── public/                      
│   ├── style.css            
│   ├── img/                     
│   ├── index.html               
│   ├── pf.html                  
│   ├── add.html                 
│   ├── stat.html                
│   ├── about.html               
│   ├── login.html               
│   └── admin.html               
├── audit.js                     
├── authentificationRutes.js     
├── createHash.js                
├── password.js                 
└── server.js                

## Design & Style (`style.css`)

- **Night theme**: Dark background (`#050b2e` / `#101a4b`) with a pink/fuchsia accent color (`#ff4fa3`).
- **Typography**: Poppins via Google Fonts.
- **Custom cursors**: Default musical note cursor (`img/musical-note.png`) and star cursor on hover for clickable elements (`img/star.png`).
- **Key components**:
  - Fixed header with `backdrop-filter: blur()` effect.
  - Responsive grids (`.cards-grid`) for laying out artist/album cards.
  - Styled forms (`.card-form`) with hover animations on buttons (`.btn`, `.gradient-btn`).

## Pages & Associated JavaScript

### 1. `index.html` — Home & Recommendations

**Purpose**: General site presentation and recommendations catalog.
**UI Navigation**: Tab system to filter between Tracks (Music), Albums, and Artists.

**JS Preview:**
```javascript
// Handle audio playback on click of the interactive element
const audio = document.getElementById('heroAudio');
const img = document.getElementById('heroAudioImg');

img.addEventListener('click', () => {
  if (audio.paused) {
    audio.play();
  } else {
    audio.pause();
  }
});

// Send a recommendation to the server
document.getElementById('form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const formData = new FormData(e.target);
  const data = Object.fromEntries(formData);

  const res = await fetch('/recommendations', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });

  if (res.ok) {
    alert('Recommendation sent!');
    e.target.reset();
  }
});
```

### 2. `pf.html` — User Profile

**Purpose**: Personal dashboard listing attended concerts and the wishlist.

**JS Preview:**
```javascript
// Load the user's concerts
async function loadMyConcerts() {
  const res = await fetch('/api/my-concerts');
  if (res.status === 401) {
    window.location.href = '/login.html';
    return;
  }
  const data = await res.json();
  renderArtistsTable(data.pastConcerts);
  renderFutureList(data.futureConcerts);
}

// Dynamic rendering and toggling of details on row click
function renderArtistsTable(concerts) {
  const tbody = document.getElementById('artistsTableBody');
  tbody.innerHTML = '';

  // Group by artist
  const grouped = concerts.reduce((acc, c) => {
    acc[c.artist] = acc[c.artist] || [];
    acc[c.artist].push(c);
    return acc;
  }, {});

  Object.entries(grouped).forEach(([artist, list]) => {
    const row = document.createElement('tr');
    row.innerHTML = `<td><strong>${artist}</strong></td><td>${list.length} time(s)</td>`;

    const detailRow = document.createElement('tr');
    detailRow.className = 'details-row hidden';
    detailRow.innerHTML = `<td colspan="2">
      ${list.map(c => `<div>${c.date} - ${c.location} (${c.type}) : ${c.rating}/10</div>`).join('')}
    </td>`;

    row.addEventListener('click', () => detailRow.classList.toggle('hidden'));
    tbody.appendChild(row);
    tbody.appendChild(detailRow);
  });
}

loadMyConcerts();
```

### 3. `add.html` — Add a Concert

**Purpose**: Two-tab form (Add a past concert / Save a future concert).

**JS Preview:**
```javascript
// Handle tab switching
const tabs = document.querySelectorAll('.tab-btn');
tabs.forEach(tab => {
  tab.addEventListener('click', () => {
    document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
    document.getElementById(tab.dataset.target).classList.add('active');
  });
});

// Submit a past concert
document.getElementById('pastConcertForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const payload = {
    artist: document.getElementById('pastArtist').value,
    date: document.getElementById('pastDate').value,
    type: document.getElementById('pastType').value,
    location: document.getElementById('pastLocation').value,
    rating: Number(document.getElementById('pastRating').value)
  };

  const res = await fetch('/concerts/past', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });

  if (res.ok) window.location.href = '/pf.html';
});
```

### 4. `stat.html` — Statistics

**Purpose**: Analytics view (KPIs, top-rated concerts ranking, most visited venues).

**JS Preview:**
```javascript
async function initStats() {
  const res = await fetch('/api/my-concerts');
  const { pastConcerts, futureConcerts } = await res.json();

  // KPI calculations
  const totalArtists = new Set(pastConcerts.map(c => c.artist)).size;
  const totalLocations = new Set(pastConcerts.map(c => c.location)).size;
  const avgRating = (pastConcerts.reduce((acc, c) => acc + c.rating, 0) / pastConcerts.length).toFixed(1);

  document.getElementById('kpiArtists').textContent = totalArtists;
  document.getElementById('kpiWishlist').textContent = futureConcerts.length;
  document.getElementById('kpiLocations').textContent = totalLocations;
  document.getElementById('kpiAvgRating').textContent = isNaN(avgRating) ? '-' : avgRating;

  // Top 10 best-rated concerts
  const top10 = [...pastConcerts].sort((a, b) => b.rating - a.rating).slice(0, 10);
  renderTopConcerts(top10);
}

initStats();
```

### 5. `about.html` — About & Gallery

**Purpose**: Personal presentation page, media (concert photos/videos), and secondary audio player.

**JS Preview:**
```javascript
// Easter egg audio on header image
const aboutAudio = new Audio('audio/abba.mp3');
const heroImg = document.querySelector('.about-hero img');

heroImg.addEventListener('click', () => {
  aboutAudio.paused ? aboutAudio.play() : aboutAudio.pause();
});
```

### 6. `login.html` — Login

**Purpose**: User authentication interface with local validation before submission.

**JS Preview:**
```javascript
function validate() {
  const username = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value.trim();
  const errorEl = document.getElementById('errorMsg');

  if (username.length < 5 || username.length > 30) {
    errorEl.textContent = 'Username must be between 5 and 30 characters.';
    return false;
  }

  if (password.length < 10) {
    errorEl.textContent = 'Password must be at least 10 characters long.';
    return false;
  }

  return true;
}

document.getElementById('loginForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  if (!validate()) return;

  const res = await fetch('/checkLogin', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      username: e.target.username.value,
      password: e.target.password.value
    })
  });

  if (res.ok) {
    window.location.href = '/pf.html';
  } else {
    document.getElementById('errorMsg').textContent = 'Invalid credentials.';
  }
});
```

### 7. `admin.html` — Admin Panel

**Purpose**: Moderation dashboard (recommendations, users, system logs).

**JS Preview:**
```javascript
// Load system logs
async function loadAuditLog() {
  const res = await fetch('/api/admin/audit-log');
  const logs = await res.json();
  const container = document.getElementById('auditLogContainer');

  container.innerHTML = logs.map(log => `
    <div class="log-item">
      <span class="date">${new Date(log.timestamp).toLocaleString()}</span>
      <span class="user">[${log.username}]</span>
      <span class="action">${log.action}</span>
    </div>
  `).join('');
}

// Block / Unblock an account
async function toggleBlockUser(userId, currentStatus) {
  await fetch('/api/admin/toggle-block', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ userId, block: !currentStatus })
  });
  loadUsers();
}
```

## Consumed API Routes

| Endpoint | Method | Front-end Usage |
|---|---|---|
| `/recommendations` | POST / GET | Submission (`index.html`) & Review (`admin.html`) |
| `/concerts/past` | POST | Create a past concert entry (`add.html`) |
| `/concerts/future` | POST | Add to wishlist (`add.html`) |
| `/api/my-concerts` | GET | Load data (`pf.html`, `stat.html`) |
| `/checkLogin` | POST | User authentication (`login.html`) |
| `/addNewUser` | POST | Create account via admin (`admin.html`) |
| `/api/admin/users` | GET | User management (`admin.html`) |
| `/api/admin/toggle-block` | POST | Account moderation (`admin.html`) |
| `/api/admin/audit-log` | GET | Display audit logs (`admin.html`) |

# 📊 `/data` Folder — JSON File Structure

This document describes the structure and fields of the 4 JSON files used to store the application's data.

## `sessions.json` — User Account Management

This file is an array of objects representing registered users. It is used for authentication and access rights.

- **`username`** (String): The user's unique login identifier (e.g. `"Aenlpb"`).
- **`name`** (String): The person's full name or display first name (e.g. `"Aénor"`).
- **`password`** (String): The password stored as a secure SHA-256 hash.
- **`admin`** (Boolean): Defines privileges. If `true`, the user has access to the `/admin.html` panel.
- **`isBlocked`** (Boolean, optional): Indicates whether the account has been locked by an administrator (`true`/`false`).

## `user_concerts.json` — Concert History & Wishlist

This file is an array of objects where each entry groups a user's music data into two sub-arrays (`past` and `future`).

- **`username`** (String): The identifier of the user the list belongs to.
- **`past`** (Array of objects): The list of previously attended events.
  - **`artist`** (String): Name of the artist or band.
  - **`date`** (String): Concert date in ISO format (`YYYY-MM-DD`).
  - **`location`** (String): Venue or festival name.
  - **`rating`** (String): Rating given to the performance (from 1 to 10).
  - **`eventType`** (String, optional): Type of event (`"concert"` or `"festival"`).
- **`future`** (Array of objects): The list of upcoming concerts/tracks.
  - **`artist`** (String): Name of the artist to see.
  - **`song`** (String): Title of an associated song.

## `recommendations.json` — Shared Recommendations

This file is an array of objects storing recommendations submitted by users on the home page.

- **`username`** (String): The author of the recommendation.
- **`type`** (String): The recommended category (`"artist"` or `"album"`).
- **`title`** (String): The name of the artist or the title of the recommended album.
- **`reason`** (String): The justification or opinion left by the user.
- **`date`** (String): Exact creation timestamp in ISO format.

## `audit_log.json` — Security & Moderation Log

This file is an array of objects recording the history of all key actions performed on the site, viewable by the administrator.

- **`username`** (String): The user who triggered the action.
- **`action`** (String): The recorded event (e.g. `"USER_LOGIN"`, `"BLOCKED_USER: Alexou"`, `"UNBLOCKED_USER: Alexou"`).
- **`timestamp`** (String): The exact date and time of the event in full ISO format (`YYYY-MM-DDTHH:mm:ss.sssZ`).
