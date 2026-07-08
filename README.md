# 🗺️ Family Travel Tracker

A web app for tracking which countries each family member has visited, visualized on an interactive world map. Each person gets their own color, and their visited countries light up on the map accordingly. Deployed live, with the database hosted on Neon.

---

## 🚀 Live Deployment

🔗 App (Render): Deployed and live
🔗 Database: Hosted on Neon (PostgreSQL)

---

## 🛠 Tech Stack

- **Backend:** Node.js, Express
- **Templating:** EJS
- **Database:** PostgreSQL, hosted on Neon, accessed via `pg`
- **Frontend:** Static CSS + an inline SVG world map, colored dynamically per user via JavaScript
- **Middleware:** `body-parser` for form submissions
- **Deployment:** Render (backend), Neon (database)

---

## ✨ Features

- 👨‍👩‍👧‍👦 **Multiple family members**, each with their own name and assigned color
- 🌍 **Interactive world map** (SVG) — visited countries are filled in with that member's color
- ➕ **Add a visited country** by name (matched against a country lookup table, case-insensitive partial match)
- 🧑‍➕ **Add a new family member** with a name and a color picker
- 🔁 **Switch between family members** via tabs — the map and count update to reflect whoever is currently selected
- 🔢 **Live count** of total countries visited by the current user

---

## ⚙️ How It Works

1. On page load, the app fetches the current user's visited countries and all registered family members
2. The `/` route renders `index.ejs`, embedding a country-code list into a small inline script that colors in the corresponding paths on the SVG map
3. **Switching users:** submitting the member tabs (`POST /user`) sets which family member is "current" and re-renders the map for them
4. **Adding a new family member:** selecting "Add Family Member" renders `new.ejs`, a color-picker form; submitting (`POST /new`) inserts a new row into `users` and switches to them as the current user
5. **Adding a country:** typing a country name and submitting (`POST /add`) looks up the matching `country_code` from a `countries` table, then inserts a row into `visited_countries` linking that code to the current user

---

## 🗄 Database Schema

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(15) UNIQUE NOT NULL,
  color VARCHAR(15)
);

CREATE TABLE visited_countries (
  id SERIAL PRIMARY KEY,
  country_code CHAR(2) NOT NULL,
  user_id INTEGER REFERENCES users(id)
);

-- Required for country name → code lookups used by POST /add
CREATE TABLE countries (
  id SERIAL PRIMARY KEY,
  country_code CHAR(2) UNIQUE NOT NULL,
  country_name VARCHAR(100) NOT NULL
);
```

> **Note:** `countries` needs to be populated with a full country name/code reference list (e.g. via CSV import) before "Add a country" will work — see **Setup** below.

---

## 📁 Project Structure

```
.
├── index.js             # Express server + routes
├── queries.sql               # Schema + practice queries + seed data
├── views/
│   ├── index.ejs                # Main map view + family member tabs
│   └── new.ejs                    # Add family member form (name + color)
└── public/
    └── styles/
        ├── main.css                 # Map, tabs, and layout styling
        └── new.css                  # "Add family member" page styling
```

---

## 🧪 Run Locally

### 1. Clone the repository

### 2. Install dependencies
```
npm install
```

### 3. Set up environment variables
Create a `.env` file:
```
DATABASE_URL=""
```

### 4. Set up the database
- Run the schema portion of `queries.sql` (the `users` / `visited_countries` section) against your database
- Create the `countries` table shown above and import a country name/code dataset into it (CSV import via `psql`'s `\copy`, or a GUI client like pgAdmin)

### 5. Run the server
```
node solution.js
```

### 6. Open the app
Visit `http://localhost:3000/`

---

## ☁️ Deployment (Render + Neon)

- **Neon** hosts the PostgreSQL database — connection string used as `DATABASE_URL`, with SSL required
- **Render** hosts the Express app as a Web Service
  - Build command: `npm install`
  - Start command: `node solution.js`
  - Environment variable: `DATABASE_URL` set to the Neon connection string
  - Render provides `PORT` automatically at runtime

---

## ⚠️ Important Notes

- Database credentials are read from `DATABASE_URL` via environment variables — never commit a real `.env` file.
- The `countries` table is essential for the "Add a country" feature to work; without it populated, lookups will silently fail to find a match.
- Country matching uses a `LIKE '%...%'` partial match on lowercase names, so partial or slightly-off spellings can still resolve — but ambiguous partial matches will just take the first result.

---

## 🐛 Common Issues

### ❌ `relation "countries" does not exist`
- The `countries` table hasn't been created yet, or wasn't populated — see **Setup**, step 4

### ❌ `Cannot read properties of undefined (reading 'color')`
- The `users` table is empty, or the current user id doesn't match any row — make sure `users` has been seeded

### ❌ App fails to connect to the database on Render
- Confirm `DATABASE_URL` is set in Render's environment variables and includes `sslmode=require`
- Neon's free tier can auto-suspend after inactivity — the first request after idle time may just be slower, not broken

---

## 🔮 Future Improvements
- 🔐 User authentication (currently anyone can add/switch family members)
- 📅 Track visit dates per country, not just visited/not-visited
- 🏙 City-level tracking within a country
- 🖼 Photo uploads tied to specific trips

---

## 📜 License

This project is for learning and personal use.
