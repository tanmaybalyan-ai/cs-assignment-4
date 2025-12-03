# cs-assignment-4
#!/usr/bin/env python3
"""
Simple console To-Do List
Features:
- Add task
- List tasks
- Mark complete
- Delete task
- Save/load tasks to tasks.json
"""

import json
import os
from datetime import datetime

DATA_FILE = "tasks.json"

def load_tasks():
    if not os.path.exists(DATA_FILE):
        return []
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except Exception:
        return []

def save_tasks(tasks):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(tasks, f, indent=2, ensure_ascii=False)

def new_task(tasks):
    title = input("Task title: ").strip()
    if not title:
        print("Cancelled (no title).")
        return
    due = input("Due date (optional, YYYY-MM-DD) or leave blank: ").strip()
    try:
        if due:
            datetime.strptime(due, "%Y-%m-%d")
    except ValueError:
        print("Invalid date format — saving without due date.")
        due = ""
    task = {
        "id": int(datetime.now().timestamp() * 1000),
        "title": title,
        "due": due,
        "done": False,
        "created": datetime.now().isoformat()
    }
    tasks.append(task)
    save_tasks(tasks)
    print("Task added.")

def list_tasks(tasks, show_all=False):
    if not tasks:
        print("No tasks yet.")
        return
    print("\nID        | Done | Due        | Title")
    print("-" * 50)
    for t in tasks:
        if not show_all and t.get("done"):
            continue
        id_short = str(t["id"])[-6:]
        done = "✓" if t.get("done") else " "
        due = t.get("due") or "--"
        print(f"{id_short:8} |  {done}  | {due:10} | {t['title']}")
    print()

def find_task(tasks, id_fragment):
    for t in tasks:
        if str(t["id"]).endswith(id_fragment) or str(t["id"]) == id_fragment:
            return t
    return None

def mark_done(tasks):
    id_fragment = input("Enter task ID (last 3-6 digits) to mark done: ").strip()
    t = find_task(tasks, id_fragment)
    if not t:
        print("Task not found.")
        return
    if t["done"]:
        print("Task already marked done.")
        return
    t["done"] = True
    t["completed_at"] = datetime.now().isoformat()
    save_tasks(tasks)
    print("Task marked done.")

def delete_task(tasks):
    id_fragment = input("Enter task ID (last 3-6 digits) to delete: ").strip()
    t = find_task(tasks, id_fragment)
    if not t:
        print("Task not found.")
        return
    tasks.remove(t)
    save_tasks(tasks)
    print("Task deleted.")

def show_menu():
    print("""
To-Do Console
1) List pending tasks
2) List all tasks
3) Add a task
4) Mark task complete
5) Delete a task
6) Exit
""")

def main():
    tasks = load_tasks()
    while True:
        show_menu()
        choice = input("Choose (1-6): ").strip()
        if choice == "1":
            list_tasks(tasks, show_all=False)
        elif choice == "2":
            list_tasks(tasks, show_all=True)
        elif choice == "3":
            new_task(tasks)
        elif choice == "4":
            mark_done(tasks)
        elif choice == "5":
            delete_task(tasks)
        elif choice == "6":
            print("Bye!")
            break
        else:
            print("Invalid choice. Try again.")

if __name__ == "__main__":
    main()
