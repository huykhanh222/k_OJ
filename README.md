# PDP-OJ: Lightweight Online Judge

A lightweight, simplified fork of DMOJ designed for small private training groups (3-5 users).

**PDP-OJ** is a minimal online judge focused on:
- Private problem practice
- Simple contest training  
- Low server resource usage
- Easy deployment and maintenance

## Features

Core OJ functionality:
- User authentication (register/login)
- Problem list and statements
- Code submission and judging
- Judge verdicts (AC, WA, TLE, etc.)
- Scoreboard
- Contest mode
- Admin problem creation

Lightweight design:
- C++17 and Python 3 runtimes only
- No real-time WebSocket updates
- No Redis/complex caching
- No comments, tickets, or messaging
- Vietnamese + English (Vietnamese default)
- Simple local database access

## Installation

1. Clone this repository
2. Install requirements: `pip install -r requirements.txt`
3. Configure `dmoj/local_settings.py`:
   ```python
   ALLOWED_HOSTS = ['yourdomain.com']
   DATABASES = {...}
   DMOJ_PROBLEM_DATA_ROOT = '/path/to/problem/data'
   SECRET_KEY = 'your-secret-key'
   ```
4. Run migrations: `python manage.py migrate`
5. Create superuser: `python manage.py createsuperuser`
6. Load fixture: `python manage.py loaddata judge/fixtures/language_small.json`
7. Collect static files: `python manage.py collectstatic`
8. Start server: `python manage.py runserver`

## Configuration

### Database
Configure in `dmoj/local_settings.py`:
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',  # or postgresql
        'NAME': 'db.sqlite3',
    }
}
```

### Language Support
Only C++17 and Python 3 are enabled. Edit judge fixtures if needed.

### Cache
Uses local-memory caching by default. No Redis required.

## Contributing ![PR's Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat)

Take a look at [our contribution guideline](contributing.md).

If you find any bug, please feel free to contact us via Discord [![Discord Chat](https://img.shields.io/discord/660930260405190688?color=%237289DA&label=Discord&logo=Discord)](https://discord.gg/TDyYVyd) or open an issue.

Pull requests are welcome as well. Before you submit your PR, please check your code with [flake8](https://flake8.pycqa.org/en/latest/) and format it if needed. There's also `prettier` if you need to format JS code (in `websocket/`).

Translation contributions are also welcome.
