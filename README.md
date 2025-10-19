# Project-Multi-Container-To-Do-App-Flask-PostgreSQL-Docker-Compose-
# docker-todo-app/
│
├── backend/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── docker-compose.yml
├── .env
└── README.md
## Flask Backend

backend/app.py

from flask import Flask, request, jsonify
import psycopg2
import os

app = Flask(__name__)

def get_db_connection():
    conn = psycopg2.connect(
        host=os.environ['DB_HOST'],
        database=os.environ['DB_NAME'],
        user=os.environ['DB_USER'],
        password=os.environ['DB_PASS']
    )
    return conn

@app.route('/todos', methods=['GET'])
def get_todos():
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute('SELECT * FROM todos;')
    todos = cur.fetchall()
    cur.close()
    conn.close()
    return jsonify(todos)

@app.route('/todos', methods=['POST'])
def add_todo():
    data = request.get_json()
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute('INSERT INTO todos (task) VALUES (%s);', (data['task'],))
    conn.commit()
    cur.close()
    conn.close()
    return jsonify({'message': 'Task added successfully!'}), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
    ## backend/Dockerfile

FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000
CMD ["python", "app.py"]
## Docker Compose File

docker-compose.yml

version: "3.8"

services:
  web:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: todosdb
      DB_USER: postgres
      DB_PASS: example
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: todosdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: example
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
  ### intialize Database

After containers start:

docker exec -it docker-todo-app-db-1 psql -U postgres -d todosdb
## A simple multi-container application using Docker Compose.

## 🚀 Features
- Flask REST API
- PostgreSQL database
- Dockerized environment
- Persistent storage

## 🧠 What I Learned
- Docker Compose for multi-container orchestration
- Environment variables & volumes
- Flask ↔ PostgreSQL connection
- Managing container dependencies
