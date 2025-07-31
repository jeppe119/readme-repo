# 🛠️ Warrant Art Gallery – Fullstack Docker Monorepo Setup

This guide explains how to run the entire Warrant Art Gallery project (Django backend + React frontend) using Docker Compose. It includes all necessary file configurations, networking, environment management, and tips for extending or debugging the stack.

---

## 📦 Monorepo Structure Overview

```
fullstack-w/
├── warrant-backend/      # Django backend codebase
│   └── Core/
│       ├── backend/          # Django apps
│       ├── .env              # Backend environment variables
│       ├── Dockerfile        # (should exist or be created)
│       └── ...
├── warrant-webpage/      # React frontend codebase
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── Dockerfile         # (create, see below)
│   ├── nginx.conf         # (create, see below)
│   └── .env               # (recommended, see below)
├── docker-compose.yml     # Root Compose file (create at repo root)
└── README.md
```

---

## 🚦 Step-by-Step Setup

### 1. Prerequisites

- Install [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/).
- (Optional, recommended) Install [Node.js](https://nodejs.org/) and [Python](https://www.python.org/) for local development/debugging.

---

### 2. Prepare the React Frontend for Docker

**a. Add a Dockerfile (`warrant-webpage/Dockerfile`):**
```dockerfile
# Build the React app
FROM node:18-alpine AS build
WORKDIR /app
COPY package.json yarn.lock* package-lock.json* ./
RUN yarn install --frozen-lockfile || npm install
COPY . .
RUN yarn build || npm run build

# Serve with Nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**b. Add an Nginx config (`warrant-webpage/nginx.conf`):**
```nginx
server {
  listen 80;
  server_name _;
  root /usr/share/nginx/html;
  location / {
    try_files $uri /index.html;
  }
}
```

---

### 3. Configure Frontend Environment Variables

**a. Create `.env` in `warrant-webpage/`:**
```
REACT_APP_API_URL=http://backend:8000
```
- **Why?** This lets your React app make API calls to the Django backend by Docker-internal hostname.
- **Reference in JS:**  
  ```js
  const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:8000';
  fetch(`${API_URL}/api/your-endpoint`)
  ```

---

### 4. Backend Configuration

- Ensure `warrant-backend/Core/.env` has correct database and allowed host settings. Example:
  ```
  DEBUG=True
  SECRET_KEY=...
  POSTGRES_HOST=db
  POSTGRES_DB=coredb
  POSTGRES_USER=coreuser
  POSTGRES_PASSWORD=supersecure
  POSTGRES_PORT=5432
  DJANGO_ALLOWED_HOSTS=localhost,backend,127.0.0.1,0.0.0.0
  ```
- **Make sure your backend has a Dockerfile** (if not, create one similar to the official Django Dockerfile or ask for a template).

---

### 5. Create/Update Root `docker-compose.yml`

At the root of your repo:

```yaml
version: "3.9"
services:
  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_DB: coredb
      POSTGRES_USER: coreuser
      POSTGRES_PASSWORD: supersecure
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build:
      context: ./warrant-backend/Core
      dockerfile: Dockerfile
    command: >
      sh -c "python backend/manage.py migrate &&
             python backend/manage.py runserver 0.0.0.0:8000"
    volumes:
      - ./warrant-backend/Core:/app
    environment:
      - POSTGRES_HOST=db
      - POSTGRES_DB=coredb
      - POSTGRES_USER=coreuser
      - POSTGRES_PASSWORD=supersecure
      - DJANGO_ALLOWED_HOSTS=localhost,backend,127.0.0.1
      - DEBUG=True
    ports:
      - "8000:8000"
    depends_on:
      - db

  frontend:
    build:
      context: ./warrant-webpage
      dockerfile: Dockerfile
    ports:
      - "3000:80"
    depends_on:
      - backend

volumes:
  postgres_data:
```

---

### 6. Build and Run All Services

From the **project root**, run:

```bash
docker-compose up --build
```

- This starts the **database**, **backend**, and **frontend**.
- Default URLs:
  - Frontend: [http://localhost:3000](http://localhost:3000)
  - Backend/API: [http://localhost:8000](http://localhost:8000)
  - Django admin: [http://localhost:8000/admin/](http://localhost:8000/admin/)

---

### 7. How Components Communicate

- **Frontend → Backend:**  
  Use `http://backend:8000` from inside containers, or `http://localhost:8000` from your host browser.
- **Backend → DB:**  
  Use `db` as the hostname in Django settings (`POSTGRES_HOST=db`).

---

### 8. Managing Environment Variables

- **Backend**: Set in `warrant-backend/Core/.env` (and referenced in Compose via `env_file` or `environment`).
- **Frontend**: Set in `warrant-webpage/.env`.  
- **Secrets**: Never commit real secrets. Use `.env.template` for safe sharing.

---

### 9. Stopping and Cleaning Up

To stop all services:
```sh
docker-compose down
```
To remove all containers, networks, and volumes (⚠️ will delete DB data):
```sh
docker-compose down -v
```

---

## 💡 Key Features & Architecture

- **Frontend**: React 18, modular structure, glassmorphism UI, login/register, marketplace, JWT auth.
- **Backend**: Django, REST API, JWT (SimpleJWT), PostgreSQL, optional Redis.
- **Networking**: All containers on the same Compose network; use service names (e.g., `backend`, `db`) for internal calls.
- **One-command startup**: `docker-compose up --build` for the entire stack.

---

## 🛠️ Development Tips

- **Hot Reload (Frontend):**  
  You can still use `npm start` in `warrant-webpage/` for local hot-reloading outside Docker.
- **Debug (Backend):**  
  Use Django shell, logs (`docker-compose logs backend`), or attach to the container.
- **Database Access:**  
  Use a tool like DBeaver, Postico, or `psql` to connect to `localhost:5432` (see DB credentials in Compose or `.env`).

---

## 🚀 Production Recommendations

- Set `DEBUG=False` and use strong, unique secrets.
- Use a production-ready database (managed Postgres, etc).
- Harden Compose file: add `docker-compose.prod.yml`, restrict ports, use `nginx` as a reverse proxy, enable HTTPS.
- Use Docker volumes for persistent storage.
- See `Docker_Production_TODO.md` for security and deployment tips.

---

## 📝 Troubleshooting

- **Frontend not connecting to backend?**
  - Check `REACT_APP_API_URL` and CORS settings in Django.
- **Migrations or DB errors?**
  - Run `docker-compose exec backend python manage.py migrate`
- **Port already in use?**
  - Change the port mapping in `docker-compose.yml`.

---

## 📚 References & Further Reading

- Frontend: `warrant-webpage/README.md`
- Backend: `warrant-backend/Core/README.md`
- Auth: `AUTH_GUARD_DOCUMENTATION.md`
- Production: `Docker_Production_TODO.md`

---

**With this setup, you have a professional, maintainable, and reproducible fullstack art marketplace—start and stop everything with one command!**