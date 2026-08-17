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

http://ip

## Expected Containers

registration-proxy
registration-frontend
registration-backend
registration-db

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
