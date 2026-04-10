# PDP-OJ: Quick Start Guide

## What is PDP-OJ?

A lightweight, simplified online judge for small private training groups (3-5 users).

- **Simple:** Core OJ features only (problems, submissions, contests)
- **Lightweight:** No WebSockets, no Redis, no real-time push
- **Easy to deploy:** Single server, SQLite or PostgreSQL
- **Vietnamese-friendly:** Default language is Vietnamese

---

## Installation (5 minutes)

### 1. Prerequisites
```bash
# Python 3.10+, pip
python --version  # 3.10+
pip --version     # Latest
```

### 2. Clone and Install
```bash
git clone https://github.com/yourusername/PDP-OJ.git
cd PDP-OJ
pip install -r requirements.txt
```

### 3. Configure
Create `dmoj/local_settings.py`:

```python
import os

DEBUG = False
SECRET_KEY = 'your-random-secret-key-here'  # Generate with: python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
ALLOWED_HOSTS = ['localhost', '127.0.0.1']

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

# Problem Data
DMOJ_PROBLEM_DATA_ROOT = '/path/to/problem/data'  # Create this directory

# Cache (local memory, no Redis)
CACHES = {}

# Language
LANGUAGE_CODE = 'vi'
```

### 4. Initialize Database
```bash
python manage.py migrate
python manage.py createsuperuser  # Create admin account
python manage.py loaddata judge/fixtures/language_pdp.json  # Load C++17 and Python 3
python manage.py collectstatic --noinput  # Collect static files
```

### 5. Run!
```bash
python manage.py runserver 0.0.0.0:8000
```

Visit: **http://localhost:8000**

---

## Create Your First Problem

1. Go to **admin/** (http://localhost:8000/admin)
2. Login with superuser account
3. Navigate to **Problems** → **Add Problem**
4. Fill in:
   - **Code:** `hello` (unique identifier)
   - **Name (English):** `Hello, World!`
   - **Name (Vietnamese):** `Xin chào, Thế giới!`
   - **Type:** Standard
   - **Time limit:** 1.0 second
   - **Memory limit:** 256 MB
   - **Description:** (HTML or Markdown)
   - **Points:** 100
5. Click **Save**

### Problem Data Files

In `/path/to/problem/data/hello/`:
```
hello/
  ├── input/
  │   ├── 1.txt    # Sample input
  │   ├── 2.txt
  │   └── ...
  ├── output/
  │   ├── 1.txt    # Expected output
  │   ├── 2.txt
  │   └── ...
  ├── init.yml     # Test case metadata (optional)
  └── statement.pdf (optional)
```

<br>

**Example:**
```
# input/1.txt
5

# output/1.txt
120
```

---

## Running the Judge

The judge must run separately from the web server:

```bash
# Terminal 1: Web Server
python manage.py runserver 0.0.0.0:8000

# Terminal 2: Judge Worker
celery -A dmoj worker --loglevel=info

# Terminal 3: Celery Beat (schedule tasks, optional)
celery -A dmoj beat --loglevel=info
```

Or use Supervisor to manage processes.

---

## User Management

### Create User (Admin Only)
In admin panel: **Auth** → **Users** → **Add User**

### Create User (Self-register)
Users can register at: `/accounts/register/`

### Manage Users
- Ban user: Admin panel → **User** → **Ban User**
- Change permissions: Admin panel → **User** → Edit

---

## Create a Contest

1. Admin → **Contests** → **Add Contest**
2. Fill in:
   - **Key:** `intro2024` (unique)
   - **Name:** `Introduction Contest`
   - **Start:** Date/time
   - **Duration:** 2 hours
   - **Format:** `IOI` (standard) or `ICPC`
   - **Problems:** Add problems
3. Save and publish

Users register and participate via `/contests/`

---

## Supported Languages

Only 2 languages (lightweight setup):

1. **C++17** (competitive programming standard)
2. **Python 3** (AI/scripting)

To add more languages:
1. Download judge binaries from DMOJ
2. Edit judge worker config
3. Load new language fixture

---

## Common Tasks

### Check Submissions
```
Admin → Judge → Submissions
```

### View Scoreboard
```
/submissions/ (all)
/contest/key/standings/ (per contest)
```

### Backup Database
```bash
# SQLite
cp db.sqlite3 db.sqlite3.backup

# PostgreSQL
pg_dump databasename > backup.sql
```

### Clear Cache
```bash
python manage.py shell
>>> from django.core.cache import cache
>>> cache.clear()
```

### Rejudge Problem
```bash
Admin → Judge → Problems → Select → Rejudge Submissions
```

---

## Production Deployment

### Using Gunicorn

```bash
pip install gunicorn
gunicorn dmoj.wsgi:application --bind 127.0.0.1:8000 --workers 4

# With Supervisor (keep running)
# /etc/supervisor/conf.d/pdp-oj.conf
[program:pdp-oj]
directory=/home/pdp-oj
command=/home/pdp-oj/venv/bin/gunicorn dmoj.wsgi:application --bind 127.0.0.1:8000
autostart=true
autorestart=true
```

### Using Nginx

```nginx
# /etc/nginx/sites-available/pdp-oj
server {
    listen 80;
    server_name yourdomain.com;

    location /static/ {
        alias /home/pdp-oj/static/;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Enable HTTPS

```bash
# Using Let's Encrypt
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

---

## Troubleshooting

### "No such file or directory: /path/to/problem/data"
Create the directory and add problem data files.

### Submissions not judging
1. Ensure Celery worker is running
2. Check worker logs: `celery -A dmoj worker --loglevel=debug`
3. Check database: admin → **Judge** → **Submissions**

### Pages slow to load
1. Run collectstatic: `python manage.py collectstatic --noinput`
2. Use nginx to serve static files
3. Consider PostgreSQL for larger datasets

### Forgot admin password
```bash
python manage.py changepassword username
```

### Import data from VNOI
```bash
# Backup VNOI database
mysqldump -u user -p database > vnoi_backup.sql

# Edit and import structure, then migrate
python manage.py migrate
```

---

## Advanced Configuration

### Email Notifications
```python
# dmoj/local_settings.py
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-app-password'  # Not your real password
```

### PostgreSQL Database
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'pdp_oj',
        'USER': 'pdp_user',
        'PASSWORD': 'secure_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

### Enable Redis Caching (Optional)
```python
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}
```

---

## Support

- **Documentation:** See `REFACTOR_GUIDE.md`
- **Issues:** Create on GitHub
- **Discussions:** GitHub Discussions

---

## License

Same as DMOJ (AGPLv3)

---

**Happy coding! 🎉**
