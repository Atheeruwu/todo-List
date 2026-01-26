# 📝 Django Todo List Application

A full-stack Django web application that allows users to register, log in, and manage personal tasks with full CRUD functionality and search features.

---

## 🚀 Features

- User authentication (register, login, logout)
- Create, update, and delete tasks
- Mark tasks as complete/incomplete
- Search tasks by title
- User-specific task isolation (each user sees only their tasks)
- Secure password hashing and session management

---

## 🛠 Tech Stack

- **Backend**: Django 4.2.7 (Python)
- **Database**: SQLite3
- **Frontend**: HTML + Django Templates
- **Authentication**: Django built-in User model
- **Python Version**: 3.x

---

## 📂 Project Structure

todo_list/
├── base/
│ ├── models.py
│ ├── views.py
│ ├── urls.py
│ ├── templates/base/
│ └── migrations/
├── todo_list/
│ ├── settings.py
│ ├── urls.py
│ └── wsgi.py
├── manage.py
└── db.sqlite3


---

## 🧩 Database Model

```python
class Task(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, null=True, blank=True)
    title = models.CharField(max_length=200)
    description = models.TextField(null=True, blank=True)
    complete = models.BooleanField(default=False)
    created = models.DateTimeField(auto_now_add=True)
