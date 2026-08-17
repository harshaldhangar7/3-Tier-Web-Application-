# 3-Tier Registration Project
## React + NGINX + Flask + MySQL + Docker Compose

This project demonstrates a simple production-style 3-tier web application.

## Architecture

Browser
   |
   v
NGINX Reverse Proxy
   |
   +--------------------+
   |                    |
   v                    v
React Frontend       Flask Backend
                         |
                         v
                       MySQL

## Technologies

- React
- Vite
- NGINX
- Python Flask
- MySQL
- mysql-connector-python
- JWT
- Docker
- Docker Compose

## Important

This project does NOT use:

- SQLAlchemy
- Any ORM

Flask communicates directly with MySQL using `mysql-connector-python`.

## Directory Structure

three-tier-flask-registration/
│
├── docker-compose.yml
│
├── proxy/
│   ├── Dockerfile
│   └── nginx.conf
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── index.html
│   ├── vite.config.js
│   └── src/
│       ├── main.jsx
│       └── style.css
│
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│       ├── __init__.py
│       ├── __main__.py
│       └── main.py
│
└── db/
    └── init.sql

## Start Project

Run:

docker compose up -d --build

Check:

docker compose ps

Open:

http://localhost

## Expected Containers

registration-proxy
registration-frontend
registration-backend
registration-db

## Test Health

Browser:

http://localhost/health

Or:

curl http://localhost/health

Expected:

{
  "status": "ok",
  "service": "flask-backend",
  "database": "connected"
}

## Register User

POST:

/api/register

Example:

curl -X POST http://localhost/api/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Harshal","email":"harshal@example.com","password":"password123"}'

## Login

POST:

/api/login

Example:

curl -X POST http://localhost/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"harshal@example.com","password":"password123"}'

The response contains a JWT token.

## Protected API

GET:

/api/me

Use:

Authorization: Bearer <TOKEN>

Example:

curl http://localhost/api/me \
  -H "Authorization: Bearer YOUR_TOKEN"

## View Database

Enter MySQL container:

docker exec -it registration-db mysql -uappuser -papppassword registration_db

Run:

SHOW TABLES;

SELECT * FROM users;

## Logs

All services:

docker compose logs -f

Backend:

docker compose logs -f backend

Proxy:

docker compose logs -f proxy

Database:

docker compose logs -f db

## Stop

docker compose down

## Delete Database Data

WARNING: This removes the MySQL Docker volume.

docker compose down -v

## Docker Networking

The browser only knows:

localhost:80

The proxy communicates with:

frontend:80
backend:5000

The Flask backend communicates with:

db:3306

Inside a Docker container, do NOT use localhost to reach another container.

Correct:

DB_HOST=db

Incorrect:

DB_HOST=localhost

## Learning Flow

1. User opens http://localhost
2. Request reaches NGINX
3. NGINX sends page request to React container
4. React sends /api/register request
5. NGINX sends /api/register to Flask
6. Flask validates input
7. Flask connects directly to MySQL
8. MySQL stores user
9. Flask returns JSON
10. React displays the result

## Production Notes

Before real production use:

- Change MySQL passwords
- Change JWT_SECRET
- Use HTTPS
- Store secrets in Docker secrets or a secret manager
- Add rate limiting
- Add stronger validation
- Add database migrations
- Do not use development credentials
