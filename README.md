# Todo List Application

A Django-based web application for managing personal tasks with user authentication and task management features.

## Project Overview

This is a full-stack Django application that allows users to create accounts, log in, and manage their personal to-do list. The application features user authentication, task creation, updating, deletion, and a search functionality.

## Architecture & Technology Stack

- **Framework**: Django 4.2.7
- **Database**: SQLite3
- **Frontend**: HTML/Django Templates
- **Authentication**: Django's built-in User model and authentication system
- **Python Version**: 3.x

## Project Structure

```
todo_list/
├── base/                          # Main app directory
│   ├── models.py                 # Database models
│   ├── views.py                  # View logic and request handlers
│   ├── urls.py                   # URL routing
│   ├── admin.py                  # Admin configuration
│   ├── templates/base/           # HTML templates
│   │   ├── login.html
│   │   ├── register.html
│   │   ├── task_list.html
│   │   ├── task.html
│   │   ├── task_form.html
│   │   └── task_confirm_delete.html
│   └── migrations/               # Database migrations
├── todo_list/                     # Project settings
│   ├── settings.py               # Django configuration
│   ├── urls.py                   # Main URL configuration
│   └── wsgi.py                   # WSGI configuration
├── manage.py                      # Django management script
└── db.sqlite3                     # Database file
```

---

## Database Models

### Task Model

Located in `base/models.py`, the Task model represents individual todo items:

```python
class Task(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, null=True, blank=True)
    title = models.CharField(max_length=200)
    description = models.TextField(null=True, blank=True)
    complete = models.BooleanField(default=False)
    created = models.DateTimeField(auto_now_add=True)
```

**Fields:**
- **user**: Foreign key to Django's User model - associates each task with a specific user. If a user is deleted, all their tasks are deleted (CASCADE).
- **title**: Required field (max 200 characters) - the task name/title
- **description**: Optional text field for additional task details
- **complete**: Boolean flag to mark tasks as completed (defaults to False)
- **created**: Automatically set to the current date/time when task is created

**Meta Options:**
- Tasks are ordered by the `complete` field, so incomplete tasks appear first in lists

---

## Views & Logic

Located in `base/views.py`, the application uses class-based views for clean, reusable code:

### 1. **CustomLoginView**
```
Route: /login/
Purpose: User login page
```
- Inherits from Django's `LoginView`
- Uses custom template `base/login.html`
- After successful login, redirects users to the task list page
- Handles user authentication

### 2. **RegisterPage**
```
Route: /register/
Purpose: New user registration
```
- Inherits from `FormView` for form handling
- Uses Django's `UserCreationForm` for secure password validation
- **Features:**
  - Automatically logs in users after successful registration
  - Redirects already authenticated users to task list (prevents re-registration)
  - Enforces password strength requirements (length, complexity, etc.)

### 3. **TaskList**
```
Route: / (root path)
Purpose: Display all user's tasks with search capability
```
- Inherits from `LoginRequiredMixin` (only authenticated users can access)
- Inherits from `ListView` to display a list of tasks
- **Features:**
  - Filters tasks to show only current user's tasks
  - Counts incomplete tasks for dashboard display
  - **Search functionality**: Users can search tasks by title using the `search-area` query parameter
  - Search is case-insensitive and uses partial matching

### 4. **TaskDetail**
```
Route: /task/<int:pk>/
Purpose: Display detailed view of a single task
```
- Shows complete information for one specific task
- Requires authentication
- Displays task title, description, completion status, and creation date

### 5. **TaskCreate**
```
Route: /task-create/
Purpose: Create a new task
```
- Inherits from `LoginRequiredMixin` and `CreateView`
- Form fields: title, description, complete
- **Key Logic**: Automatically assigns the task to the current logged-in user via `form_instance.user = self.request.user`
- Redirects to task list after successful creation

### 6. **TaskUpdate**
```
Route: /task-update/<int:pk>/
Purpose: Edit existing task
```
- Allows users to modify title, description, and completion status
- Requires authentication
- Redirects to task list after successful update

### 7. **DeleteView**
```
Route: /task-delete/<int:pk>/
Purpose: Delete a task
```
- Removes a task from the database
- Requires authentication
- Shows confirmation page before deletion
- Redirects to task list after deletion

---

## URL Routing

Defined in `base/urls.py`:

| Route | View | Purpose |
|-------|------|---------|
| `/login/` | CustomLoginView | User login |
| `/logout/` | LogoutView (Django built-in) | User logout |
| `/register/` | RegisterPage | New user registration |
| `/` | TaskList | View all tasks with search |
| `/task/<id>/` | TaskDetail | View single task details |
| `/task-create/` | TaskCreate | Create new task |
| `/task-update/<id>/` | TaskUpdate | Edit task |
| `/task-delete/<id>/` | DeleteView | Delete task |

---

## Authentication Flow

1. **Registration**:
   - User submits username and password
   - Password is validated and hashed
   - User account is created
   - User is automatically logged in
   - Redirect to task list

2. **Login**:
   - User enters credentials
   - Django authenticates user
   - Session is created
   - Redirect to task list

3. **Protected Routes**:
   - All task-related views use `LoginRequiredMixin`
   - Unauthenticated users are redirected to login page
   - `LOGIN_URL = 'login'` setting (in settings.py) defines redirect destination

4. **Logout**:
   - Session is destroyed
   - User is logged out
   - Redirect to login page

---

## Key Features & Logic

### User Isolation
- Tasks are filtered by user: `context['tasks'] = context['tasks'].filter(user=self.request.user)`
- Each user only sees their own tasks
- Cannot access other users' tasks

### Search Functionality
- Case-insensitive search: `filter(title__icontains=search_input)`
- Searches task titles for partial matches
- Search term is preserved in context for display

### Task Completion Tracking
- Incomplete task counter: `context['count'] = context['tasks'].filter(complete=False).count()`
- Shows number of incomplete tasks on dashboard
- Tasks can be marked complete without deletion

### Automatic User Assignment
- When creating a task: `form.instance.user = self.request.user`
- Ensures task ownership before saving

---

## Django Configuration

Key settings in `settings.py`:

- **DEBUG = True**: Development mode enabled
- **INSTALLED_APPS**: Includes 'base' app and Django's default apps
- **LOGIN_URL = 'login'**: Redirect unauthenticated users to login view
- **DATABASE**: SQLite3 for development
- **MIDDLEWARE**: Security, sessions, CSRF protection, authentication

---

## Forms Used

1. **UserCreationForm** (Django built-in):
   - Used in RegisterPage
   - Validates passwords match
   - Enforces password strength rules
   - Hashes passwords securely

2. **Model Forms** (auto-generated):
   - TaskCreate and TaskUpdate use auto-generated forms from Task model
   - Fields: title, description, complete

---

## Templates

- **login.html**: Login form
- **register.html**: Registration form with password fields
- **task_list.html**: Displays all tasks with search bar and counts
- **task.html**: Individual task detail view
- **task_form.html**: Form for creating/updating tasks
- **task_confirm_delete.html**: Confirmation page before task deletion

---

## Running the Application

1. **Install dependencies**:
   ```bash
   pip install django
   ```

2. **Apply migrations**:
   ```bash
   python manage.py migrate
   ```

3. **Run development server**:
   ```bash
   python manage.py runserver
   ```

4. **Access the application**:
   - Visit `http://localhost:8000/`
   - You'll be redirected to login page
   - Register a new account or use existing credentials

---

## Security Notes

- Passwords are hashed using Django's password hashing system
- CSRF protection enabled via middleware
- SQL injection prevention through ORM
- User authentication prevents unauthorized access
- Session-based authentication for stateful requests

---

## Future Enhancement Ideas

- Task categories or tags
- Priority levels
- Due dates
- Task sharing between users
- Email notifications
- Task recurring/scheduling
- Dark mode UI
- Mobile responsive design improvements
#   t o d o - L i s t  
 