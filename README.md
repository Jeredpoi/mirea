# mirea

import sqlite3

db = sqlite3.connect("tasks.db")
db.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, login TEXT UNIQUE, password TEXT)")
db.execute("CREATE TABLE IF NOT EXISTS tasks (id INTEGER PRIMARY KEY, user_id INTEGER, title TEXT, done INTEGER DEFAULT 0)")
user = None  # (id, login, password)

while True:
    if user is None:
        print("1 - регистрация, 2 - вход, 0 - выход")
        choice = input("Выбор: ").strip()
        if choice in ("1", "2"):
            login = input("Логин: ").strip()
            password = input("Пароль: ").strip()
            if choice == "1":
                try:
                    db.execute("INSERT INTO users (login, password) VALUES (?, ?)", (login, password))
                    db.commit()
                    print("Готово.")
                except sqlite3.IntegrityError:
                    print("Логин занят.")
            else:
                user = db.execute("SELECT id, login, password FROM users WHERE login = ? AND password = ?", (login, password)).fetchone()
                print("Вход выполнен." if user else "Неверные данные.")
        elif choice == "0":
            break
    else:
        print("1 - добавить, 2 - список, 3 - выполнить, 4 - выйти, /я - мои данные, 0 - выход")
        choice = input("Выбор: ").strip()
        if choice == "1":
            db.execute("INSERT INTO tasks (user_id, title) VALUES (?, ?)", (user[0], input("Задача: ").strip()))
            db.commit()
        elif choice == "2":
            for tid, title, done in db.execute("SELECT id, title, done FROM tasks WHERE user_id = ?", (user[0],)):
                print(f"{tid}: [{'✓' if done else ' '}] {title}")
        elif choice == "3":
            cur = db.execute("UPDATE tasks SET done = 1 WHERE id = ? AND user_id = ?", (input("id: ").strip(), user[0]))
            db.commit()
            print("Отмечено." if cur.rowcount else "Нет такой задачи.")
        elif choice == "4":
            user = None
        elif choice == "/я":
            print(f"Логин: {user[1]}\nПароль: {user[2]}")
        elif choice == "0":
            break

db.close()
