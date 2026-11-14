# mirea

import sqlite3

db = sqlite3.connect("tasks.db")
db.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, login TEXT UNIQUE, password TEXT)")
db.execute("CREATE TABLE IF NOT EXISTS tasks (id INTEGER PRIMARY KEY, user_id INTEGER, title TEXT)")

user = None

print("Привет! Рад тебя видеть.")

while True:
    if user is None:
        choice = input("Что хочешь сделать? ")
        if choice == "регистрация":
            login = input("Придумай логин: ")
            password = input("Придумай пароль: ")
            db.execute("INSERT INTO users (login, password) VALUES (?, ?)", (login, password))
            db.commit()
            print(f"Готово, {login} зарегистрирован!")
        elif choice == "вход":
            login = input("Логин: ")
            password = input("Пароль: ")
            user = db.execute(
                "SELECT id, login, password FROM users WHERE login = ? AND password = ?",
                (login, password),
            ).fetchone()
            if user is None:
                print("Не получилось войти, проверь логин и пароль.")
            else:
                print(f"Добро пожаловать, {login}!")
        elif choice == "выход":
            break
    else:
        choice = input("Что дальше? ")
        if choice == "добавить":
            title = input("Какая задача? ")
            db.execute("INSERT INTO tasks (user_id, title) VALUES (?, ?)", (user[0], title))
            db.commit()
            print("Задача добавлена.")
        elif choice == "задачи":
            rows = db.execute("SELECT title FROM tasks WHERE user_id = ?", (user[0],)).fetchall()
            if not rows:
                print("Пока задач нет.")
            for (title,) in rows:
                print("-", title)
        elif choice == "/я":
            print(f"Твой логин: {user[1]}")
            print(f"Твой пароль: {user[2]}")
        elif choice == "выйти":
            print("Пока! Ты вышел из аккаунта.")
            user = None
        elif choice == "выход":
            break

db.close()
print("До встречи!")
