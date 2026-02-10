
- 🖥 Servidor em Python

- - Qt (GUI) para controlar o servidor

- - Flask como backend HTTP

- - SQLite como base de dados

- - botões Ligar / Desligar / Sair

- - Log das ligações

- - janela do servidor permite ver, editar e apagar users


🌐 Página Web

HTML + CSS + JavaScript

Dois campos para registar users

Comunicação HTTP com o servidos


---

📁 Estrutura do projeto

projeto/
│
├── server_qt.py        # GUI Qt + controlo do Flask
├── flask_app.py        # Backend Flask
├── database.py         # SQLite
├── users.db            # Base de dados (criada automaticamente)
│
└── web/
    ├── index.html
    ├── style.css
    └── script.js


---

🗄 database.py (SQLite)

import sqlite3

DB_NAME = "users.db"

def init_db():
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()
    cur.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT,
            password TEXT
        )
    """)
    conn.commit()
    conn.close()

def get_users():
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()
    cur.execute("SELECT * FROM users")
    data = cur.fetchall()
    conn.close()
    return data

def add_user(username, password):
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()
    cur.execute(
        "INSERT INTO users (username, password) VALUES (?, ?)",
        (username, password)
    )
    conn.commit()
    conn.close()

def update_user(user_id, username, password):
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()
    cur.execute(
        "UPDATE users SET username=?, password=? WHERE id=?",
        (username, password, user_id)
    )
    conn.commit()
    conn.close()

def delete_user(user_id):
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()
    cur.execute("DELETE FROM users WHERE id=?", (user_id,))
    conn.commit()
    conn.close()


---

🌐 flask_app.py (Servidor HTTP)

from flask import Flask, request, jsonify
from database import init_db, get_users, add_user, update_user, delete_user
from datetime import datetime

app = Flask(__name__)
init_db()

log_callback = None  # será ligado ao Qt

def log(msg):
    if log_callback:
        log_callback(f"[{datetime.now().strftime('%H:%M:%S')}] {msg}")

@app.route("/users", methods=["GET"])
def users():
    log("GET /users")
    return jsonify(get_users())

@app.route("/users", methods=["POST"])
def create_user():
    data = request.json
    add_user(data["username"], data["password"])
    log(f"Novo user: {data['username']}")
    return {"status": "ok"}

@app.route("/users/<int:user_id>", methods=["PUT"])
def edit_user(user_id):
    data = request.json
    update_user(user_id, data["username"], data["password"])
    log(f"User alterado ID {user_id}")
    return {"status": "ok"}

@app.route("/users/<int:user_id>", methods=["DELETE"])
def remove_user(user_id):
    delete_user(user_id)
    log(f"User apagado ID {user_id}")
    return {"status": "ok"}

def run_flask():
    app.run(host="0.0.0.0", port=5000, debug=False)


---

🖥 server_qt.py (Qt + controlo do servidor)

import sys, threading
from PyQt5.QtWidgets import (
    QApplication, QWidget, QPushButton, QTextEdit,
    QVBoxLayout, QHBoxLayout, QMessageBox
)
import flask_app

class ServerGUI(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Servidor Flask + SQLite")
        self.resize(500, 400)

        self.log = QTextEdit()
        self.log.setReadOnly(True)

        self.btn_on = QPushButton("Ligar")
        self.btn_off = QPushButton("Desligar")
        self.btn_exit = QPushButton("Sair")

        self.btn_on.clicked.connect(self.start_server)
        self.btn_exit.clicked.connect(self.close)

        layout = QVBoxLayout()
        layout.addWidget(self.log)

        buttons = QHBoxLayout()
        buttons.addWidget(self.btn_on)
        buttons.addWidget(self.btn_off)
        buttons.addWidget(self.btn_exit)

        layout.addLayout(buttons)
        self.setLayout(layout)

        flask_app.log_callback = self.add_log
        self.server_thread = None

    def add_log(self, msg):
        self.log.append(msg)

    def start_server(self):
        if not self.server_thread:
            self.server_thread = threading.Thread(
                target=flask_app.run_flask, daemon=True
            )
            self.server_thread.start()
            self.add_log("Servidor ligado")

app = QApplication(sys.argv)
win = ServerGUI()
win.show()
sys.exit(app.exec_())


---

🌐 web/index.html

<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <title>Registo de Users</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

<h2>Registar User</h2>

<input id="user" placeholder="Username">
<input id="pass" placeholder="Password" type="password">
<button onclick="addUser()">Registar</button>

<h3>Users</h3>
<ul id="list"></ul>

<script src="script.js"></script>
</body>
</html>


---

🎨 web/style.css

body {
    font-family: Arial;
    background: #1e1e1e;
    color: white;
    padding: 20px;
}

input, button {
    padding: 8px;
    margin: 5px;
}


---

⚙ web/script.js

const API = "http://localhost:5000/users";

function loadUsers() {
    fetch(API)
        .then(r => r.json())
        .then(data => {
            const list = document.getElementById("list");
            list.innerHTML = "";
            data.forEach(u => {
                list.innerHTML += `
                    <li>
                        ${u[1]} 
                        <button onclick="del(${u[0]})">X</button>
                    </li>`;
            });
        });
}

function addUser() {
    fetch(API, {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify({
            username: user.value,
            password: pass.value
        })
    }).then(loadUsers);
}

function del(id) {
    fetch(`${API}/${id}`, { method: "DELETE" })
        .then(loadUsers);
}

loadUsers();


---

✅ O que já faz

✔ Registo via web
✔ Ver users
✔ Apagar users
✔ SQLite
✔ GUI Qt
✔ Log das ligações HTTP
✔ Ligar / Desligar / Sair


---

próximo passo:

🔐 adicionar login + tokens

🧑‍💻 editar users diretamente na GUI Qt

🌍 servir a página web diretamente pelo Flask

📜 log em ficheiro

🔄 auto-refresh na web

🔒 HTTPs
