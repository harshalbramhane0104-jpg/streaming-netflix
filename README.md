# 🎬 Netflix Clone

A Netflix-style streaming UI: sign up / log in, browse rows of titles pulled live from **TMDB**, search, save titles to "My List", watch trailers, and manage your account — built as a **3-tier application** and deployed on **AWS** using a highly available, auto-scaling cloud architecture.

---

## 📌 Overview

| | |
|---|---|
| **Frontend** | React + Tailwind CSS (Netflix-style black/red theme) |
| **Backend** | Python — FastAPI (JWT auth, bcrypt password hashing) |
| **Database** | MySQL |
| **Cloud** | AWS — VPC, EC2, Auto Scaling, ALB, CloudFront, RDS, NAT Gateway |

---

## ✨ Features

- Sign up / log in with JWT-based authentication
- Browse rows of titles pulled live from the TMDB API
- Live search
- Watch trailers in an autoplaying modal (YouTube trailers via TMDB's video API)
- Save/remove titles to a personal **My List**
- Account page with profile info and sign-out

---

## 🏗️ Application Architecture (3-Tier)

1. **Presentation tier** — `frontend/` — React UI. Talks to the backend only over HTTP (`REACT_APP_BACKEND_URL`).
2. **Application/logic tier** — `backend/` — FastAPI. Handles auth, business rules, and all reads/writes to the database. The only tier that talks to the database.
3. **Data tier** — **MySQL** — `users` and `mylist` tables. Nothing outside the backend touches it directly.

Each tier can run on its own machine — the frontend just needs the backend's URL, and the backend just needs the MySQL host/port/credentials in `.env`.

---

## ☁️ Cloud Deployment Architecture (AWS)

The app is designed to be deployed on AWS with high availability and auto-scaling across two Availability Zones, isolating the backend and database in private subnets.

![Netflix Clone AWS Architecture](docs/architecture.png)

**Traffic flow:** Internet → CloudFront → Frontend ALB → Frontend EC2 (Auto Scaling, public subnets) → Backend ALB (internal) → Backend EC2 (Auto Scaling, private subnets) → RDS MySQL (Multi-AZ, private subnets)

**Key components:**
- **Amazon VPC** — custom network (`10.0.0.0/16`) spanning 2 AZs with public/private subnet isolation
- **Internet Gateway** — public ingress for the frontend tier
- **NAT Gateway** — outbound-only internet access for the private backend tier (OS updates, TMDB API calls)
- **Auto Scaling Groups** — independent scaling for frontend (`netflix-frontend-asg`) and backend (`netflix-backend-asg`)
- **Application Load Balancers** — internet-facing ALB for the frontend, internal ALB for backend service discovery
- **Amazon CloudFront** — global CDN and HTTPS termination in front of the frontend
- **Amazon RDS (MySQL, Multi-AZ)** — isolated in private subnets, reachable only via least-privilege security groups

A full step-by-step deployment walkthrough (VPC setup, security groups, launch templates, ASG/ALB configuration, CloudFront setup) is available in [`docs/aws-deployment-guide.md`](docs/aws-deployment-guide.md).

---

## 🛠️ Tech Stack

**Frontend:** React, React Router, Tailwind CSS, Axios, Lucide Icons
**Backend:** FastAPI, aiomysql, Pydantic, PyJWT, Passlib (bcrypt), Uvicorn
**Database:** MySQL 8
**Infrastructure:** AWS (VPC, EC2, Auto Scaling, ALB, CloudFront, RDS, NAT Gateway, IAM), Nginx, Linux (Ubuntu)

---

## 🚀 Local Development Setup

### 1. Set up MySQL (data tier)

You need a MySQL server running somewhere reachable by the backend. Options:

**Local install (Windows):** install [MySQL Community Server](https://dev.mysql.com/downloads/mysql/) (the installer includes MySQL Workbench, a GUI you can use for the next step).

**Local install (Mac):** `brew install mysql && brew services start mysql`

**Docker (any OS), fastest if you have Docker:**
```bash
docker run --name netflix-mysql -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 -d mysql:8
```

Once MySQL is running, create the database and an app user (run in `mysql` CLI, MySQL Workbench, or `docker exec -it netflix-mysql mysql -uroot -proot`):
```sql
CREATE DATABASE netflix_clone;
CREATE USER 'netflix_app'@'%' IDENTIFIED BY 'netflix_pw_123';
GRANT ALL PRIVILEGES ON netflix_clone.* TO 'netflix_app'@'%';
FLUSH PRIVILEGES;
```
(Change the password — this is just a placeholder.)

The backend auto-creates the `users` and `mylist` tables on first run — you don't need to run `backend/schema.sql` yourself unless you want to inspect it or set the schema up ahead of time.

### 2. Configure the backend

Edit `backend/.env` if your MySQL credentials differ from the defaults:
```
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USER=netflix_app
MYSQL_PASSWORD=netflix_pw_123
MYSQL_DB=netflix_clone
```

### 3. Run the backend (application tier)

```bash
cd backend
python -m venv venv
venv\Scripts\Activate.ps1      # Mac/Linux: source venv/bin/activate
pip install -r requirements.txt
uvicorn server:app --reload --port 8000
```
Ready when you see `Connected to MySQL database 'netflix_clone'` and `Uvicorn running on http://0.0.0.0:8000`.

### 4. Run the frontend (presentation tier)

```bash
cd frontend
npm install
npm start
```
Opens `http://localhost:3000`. It already points at `http://localhost:8000` via `frontend/.env` — change `REACT_APP_BACKEND_URL` if your backend runs elsewhere.

---

## 🧪 Try It

1. Sign up on the landing page → lands you on the browse dashboard (rows load live from TMDB).
2. Search bar in the nav — type and get live results.
3. Click a title to open a modal that autoplays its YouTube trailer (via TMDB's video API — it only ever streams trailers, never full films).
4. Click **+** on any card to save it to My List; view saved titles under **My List**.
5. **Account** (profile menu) shows your info and lets you sign out.

---

## 🔍 Inspecting Your Data

```sql
USE netflix_clone;
SELECT id, email, name, created_at FROM users;
SELECT * FROM mylist;
```
Passwords are stored as bcrypt hashes, never in plain text.

---

## 📁 Project Structure

```
netflix-clone/
├── frontend/              # React app (presentation tier)
│   ├── src/
│   │   ├── components/    # Navbar, Hero, MovieCard, MovieModal, Row, WatchPlayer
│   │   ├── pages/         # Landing, Browse, Login, Signup, Search, MyList, Account
│   │   ├── context/       # AuthContext
│   │   └── services/      # tmdb.js (TMDB API client)
│   └── .env
├── backend/               # FastAPI app (application tier)
│   ├── server.py
│   ├── schema.sql
│   ├── requirements.txt
│   └── .env
├── docs/
│   ├── architecture.png           # AWS cloud architecture diagram
│   └── aws-deployment-guide.md    # Full AWS deployment walkthrough
└── README.md
```

---

## 📄 License

This is a personal/educational project built for learning full-stack development and AWS cloud architecture. Not affiliated with Netflix.
