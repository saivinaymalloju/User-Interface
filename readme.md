# Django User Registration & Login Project

> A complete beginner-friendly Django web application covering user registration, login, dashboard, and logout — explained step by step.

---

## Table of Contents

- [What is This Project?](#what-is-this-project)
- [What is Django?](#what-is-django)
- [Technologies Used](#technologies-used)
- [Project Folder Structure](#project-folder-structure)
- [Step 1 — manage.py](#step-1--managepy)
- [Step 2 — settings.py](#step-2--settingspy)
- [Step 3 — URL Files](#step-3--url-files-traffic-controller)
- [Step 4 — forms.py](#step-4--formspy)
- [Step 5 — views.py](#step-5--viewspy)
- [Step 6 — Templates](#step-6--templates-html-files)
- [Step 7 — How Sessions Work](#step-7--how-sessions-work)
- [Complete Request Flow](#complete-request-flow)
- [All URLs at a Glance](#all-urls-at-a-glance)
- [Key Django Concepts](#key-django-concepts-summary)
- [How to Run the Project](#how-to-run-the-project)

---

## What is This Project?

This is a **Django Web Application** that allows users to:

- **Register** — Create a new account
- **Login** — Sign in with an existing account
- **View Dashboard** — A protected page, only logged-in users can see it
- **Logout** — Sign out safely

---

## What is Django?

Django is a **Python Web Framework** — it helps you build websites quickly using Python.

```
User opens Browser
       ↓
   Sends Request  →  Django handles it  →  Sends back HTML page
```

Django follows the **MVT Pattern:**

| Letter | Stands For | Job |
|--------|------------|-----|
| M | Model | Handles Database |
| V | View | Handles Logic |
| T | Template | Handles HTML (what user sees) |

---

## Technologies Used

| Technology | Version | Purpose |
|------------|---------|---------|
| Python | 3.11.9 | Programming language |
| Django | 4.2 | Web framework |
| SQLite | Built-in | Database |
| Bootstrap | 5.3 | CSS styling |
| HTML/CSS | — | Frontend templates |

---

## Project Folder Structure

```
django_auth_project/          ← Root folder (whole project lives here)
│
├── manage.py                 ← Django's command tool (run server, migrate, etc.)
│
├── django_auth_project/      ← Project settings folder
│   ├── __init__.py           ← Tells Python "this is a package"
│   ├── settings.py           ← All project configurations
│   ├── urls.py               ← Main URL routes (traffic controller)
│   └── wsgi.py               ← Deployment entry point
│
└── accounts/                 ← Our App (handles users)
    ├── __init__.py           ← Tells Python "this is a package"
    ├── views.py              ← Logic (what happens on each page)
    ├── urls.py               ← URL routes for accounts app
    ├── forms.py              ← Registration & Login forms
    └── templates/
        └── accounts/
            ├── base.html     ← Common layout (navbar, messages)
            ├── register.html ← Registration page
            ├── login.html    ← Login page
            └── dashboard.html← Dashboard page (protected)
```

> **Note:** `forms.py` is NOT auto-created by Django. You must create it manually inside the `accounts/` folder.

---

## 🔷 Step 1 — `manage.py`

This file is **auto-created** by Django. You never edit it.

You use it to run commands:

```bash
python manage.py runserver    # starts the website
python manage.py migrate      # creates database tables
python manage.py startapp     # creates a new app
```

> Think of it as a **remote control** for your Django project.

---

## Step 2 — `settings.py` — Project Configuration

This is the **brain/config file** of the entire project. Everything is configured here.

### INSTALLED_APPS

```python
INSTALLED_APPS = [
    'django.contrib.admin',      # Django admin panel
    'django.contrib.auth',       # Built-in user login/logout system
    'django.contrib.sessions',   # Handles user sessions
    'accounts',                  # OUR custom app
]
```

> Think of this as a **list of plugins** your project uses.

---

### DATABASES

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

> We use **SQLite** — a simple file-based database. Perfect for learning. All user data is stored in `db.sqlite3` file.

---

### LOGIN Settings

```python
LOGIN_URL = '/accounts/login/'               # If not logged in → go here
LOGIN_REDIRECT_URL = '/accounts/dashboard/'  # After login → go here
LOGOUT_REDIRECT_URL = '/accounts/login/'     # After logout → go here
```

> These 3 lines control **where Django redirects** users automatically.

---

## Step 3 — URL Files (Traffic Controller)

### Main `urls.py` (Project level)

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/', include('accounts.urls')),  # hand over to accounts app
    path('', lambda request: redirect('accounts:login')),  # root → login
]
```

### `accounts/urls.py` (App level)

```python
app_name = 'accounts'   # namespace — avoids name conflicts

urlpatterns = [
    path('register/',  views.register_view,  name='register'),
    path('login/',     views.login_view,     name='login'),
    path('logout/',    views.logout_view,    name='logout'),
    path('dashboard/', views.dashboard_view, name='dashboard'),
]
```

### How URLs work together:

```
User visits → /accounts/register/
                    ↓
         Main urls.py sees "accounts/"
                    ↓
         Passes to accounts/urls.py
                    ↓
         Matches "register/" → calls register_view
```

---

## Step 4 — `forms.py` — The Forms

Forms handle **user input** and **validation**.

### RegisterForm

```python
class RegisterForm(UserCreationForm):  # extends Django's built-in form
    email = forms.EmailField(...)      # adds email field
    username = forms.CharField(...)    # username field
    password1 = forms.CharField(...)   # password field
    password2 = forms.CharField(...)   # confirm password field

    def clean_email(self):             # custom validation
        email = self.cleaned_data.get('email')
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError("This email is already registered.")
        return email
```

> `UserCreationForm` is Django's **built-in form** that already handles password matching, strength checks etc. We just **extend it** and add email.

### LoginForm

```python
class LoginForm(AuthenticationForm):  # extends Django's built-in login form
    username = forms.CharField(...)
    password = forms.CharField(...)
```

> `AuthenticationForm` already handles **username + password checking** against the database. We just style it.

### What `clean_email()` does:

```
User submits email → clean_email() runs
         ↓
Checks if email exists in DB
         ↓
If YES → show error 
If NO  → allow registration 
```

---

## Step 5 — `views.py` — The Logic

Views are **Python functions** that handle what happens when a URL is visited.

### register_view

```python
def register_view(request):

    # If already logged in → go to dashboard
    if request.user.is_authenticated:
        return redirect('accounts:dashboard')

    if request.method == 'POST':       # form was submitted
        form = RegisterForm(request.POST)
        if form.is_valid():            # all fields are correct
            form.save()               # save user to database
            messages.success(request, 'Account created! Please login.')
            return redirect('accounts:login')   # go to login page
        else:
            messages.error(request, 'Please fix the errors.')
    else:                              # page just opened (GET request)
        form = RegisterForm()          # show empty form

    return render(request, 'accounts/register.html', {'form': form})
```

### Flow Diagram:

```
User visits /register/
        ↓
GET request? → Show empty form
        ↓
User fills form → clicks Submit
        ↓
POST request → validate form
        ↓
Valid?   → Save to DB → Redirect to Login 
Invalid? → Show form again with errors   
```

---

### login_view

```python
def login_view(request):
    if request.method == 'POST':
        form = LoginForm(request, data=request.POST)
        if form.is_valid():
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            user = authenticate(request, username=username, password=password)
            if user is not None:
                login(request, user)    # creates session for user
                return redirect('accounts:dashboard')
        else:
            messages.error(request, 'Invalid username or password.')
```

### Key Functions:

| Function | What it does |
|----------|-------------|
| `authenticate()` | Checks username & password against the DB |
| `login()` | Creates a **session** — remembers the user is logged in |
| `logout()` | Destroys the session — user is logged out |

---

### dashboard_view

```python
@login_required          # ← This is a DECORATOR
def dashboard_view(request):
    return render(request, 'accounts/dashboard.html', {'user': request.user})
```

> `@login_required` is a **decorator** — it protects this page. If a user is **not logged in** and tries to visit `/dashboard/`, Django automatically sends them to the **login page**.

---

## Step 6 — Templates (HTML Files)

### `base.html` — The Master Layout

```html
<!-- All other pages EXTEND this -->
<nav>...</nav>                         <!-- Navbar shown on all pages -->
{% for message in messages %}
    <!-- Flash messages shown here -->
{% endfor %}
{% block content %}{% endblock %}      <!-- Child pages fill this area -->
```

> Think of `base.html` like a **picture frame** — every page uses the same frame, only the picture (content) changes.

---

### `register.html`

```html
{% extends 'accounts/base.html' %}    <!-- use the base frame -->

{% block content %}
<form method="POST">
    {% csrf_token %}                   <!-- security token (REQUIRED) -->
    {% for field in form %}            <!-- loop through each form field -->
        {{ field }}                    <!-- render input box -->
    {% endfor %}
    <button>Register</button>
</form>
{% endblock %}
```

---

### What is `{% csrf_token %}`?

**CSRF = Cross Site Request Forgery**

```
Without CSRF token:
Evil website → sends fake form → your server accepts it 

With CSRF token:
Django checks hidden token → fake request rejected 
```

> Always include `{% csrf_token %}` inside every `<form>` tag. It is a security best practice.

---

## Step 7 — How Sessions Work

```
User logs in
     ↓
Django creates a SESSION (stored in database)
     ↓
Sends SESSION ID to browser as a COOKIE
     ↓
Every future request → browser sends cookie → Django knows who you are
     ↓
User logs out → session deleted → cookie cleared
```

> This is how Django **remembers** a logged-in user across multiple pages.

---

## Complete Request Flow

```
[Browser]  →  types URL  →  [Django urls.py]
                                    ↓
                            matches URL pattern
                                    ↓
                            calls View function
                                    ↓
                         View talks to Form / Model
                                    ↓
                         View returns Template (HTML)
                                    ↓
                        [Browser] shows the page
```

---

## All URLs at a Glance

| URL | View | What Happens |
|-----|------|-------------|
| `/` | — | Redirects to login |
| `/accounts/register/` | register_view | Show registration form |
| `/accounts/login/` | login_view | Show login form |
| `/accounts/dashboard/` | dashboard_view | Show dashboard *(login required)* |
| `/accounts/logout/` | logout_view | Log out and redirect to login |

---

## Key Django Concepts Summary

| Concept | What it is | Example in this Project |
|---------|-----------|------------------------|
| **App** | A module inside a project | `accounts` app |
| **View** | Function that handles logic | `register_view`, `login_view` |
| **URL** | Maps web address to a view | `/register/` → `register_view` |
| **Form** | Handles input & validation | `RegisterForm`, `LoginForm` |
| **Template** | HTML file shown to user | `register.html`, `login.html` |
| **Model** | Represents a DB table | Django's built-in `User` model |
| **Session** | Remembers logged-in user | Created on `login()` call |
| **Decorator** | Adds extra behavior to view | `@login_required` |
| **CSRF Token** | Security against fake requests | `{% csrf_token %}` in forms |
| **Messages** | Flash notifications | `messages.success(...)` |

---

## How to Run the Project

### 1. Install Django

```bash
pip install django==4.2
```

### 2. Create the Project & App

```bash
django-admin startproject django_auth_project
cd django_auth_project
python manage.py startapp accounts
```

### 3. Apply Migrations

```bash
python manage.py migrate
```

### 4. Run the Server

```bash
python manage.py runserver
```

### 5. Open in Browser

```
http://127.0.0.1:8000/
```

---

## Common Error & Fix

### `ModuleNotFoundError: No module named 'accounts.forms'`

**Cause:** `forms.py` is not auto-created by Django. You must create it manually.

**Fix:** Create a new file called `forms.py` inside the `accounts/` folder and add your form classes.

---

> Once you understand this project, you have a strong foundation to build any Django application confidently!