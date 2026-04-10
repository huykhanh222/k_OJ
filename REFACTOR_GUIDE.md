# PDP-OJ Refactoring Guide

## Project Overview

**PDP-OJ** is a lightweight, simplified fork of DMOJ designed for small private training groups (3-5 users).

Transformed from: VNOI OJ → **PDP-OJ**

Target use cases:
- Private problem practice for small groups
- Simple contest training
- Low server resource usage
- Easy deployment and maintenance

---

## Changes Made

### 1. **Branding Refactor**

| Location | Old | New |
|----------|-----|-----|
| `dmoj/settings.py` | `SITE_NAME = 'DMOJ'` | `SITE_NAME = 'PDP-OJ'` |
| `dmoj/settings.py` | `SITE_LONG_NAME = 'DMOJ: Modern Online Judge'` | `SITE_LONG_NAME = 'PDP-OJ: Lightweight Online Judge'` |
| `templates/base.html` | Footer links to VNOI/DMOJ | Links to PDP-OJ repo |
| `README.md` | VNOJ documentation | PDP-OJ lightweight setup |
| `package.json` | `@vnoi-admin/oj` | `@pdp-oj/oj` |

### 2. **Language Simplification**

**Removed runtimes:**
- C++03, C++11, C++14 → Keep only **C++17**
- C, C11 (removed)
- Pascal (removed)
- Python 2 (removed)
- Java 8 (removed)

**Kept:**
- **C++17**: Modern standard, widely used in competitive programming
- **Python 3**: Latest, most popular

**File:** `judge/fixtures/language_pdp.json` (new, simplified)

### 3. **Removed WebSocket Real-time Updates**

**Deleted:**
- `websocket/` directory entirely
  - `websocket/daemon.js`
  - `websocket/queue.js`
  - `websocket/config.js`

**Related code removed:**
- `resources/event.js` (WebSocket client logic)
- Event posting AMQP support: `judge/event_poster_amqp.py` (not deleted, but not used)
- Event posting WebSocket support: `judge/event_poster_ws.py` (not deleted, but not used)

**Replacement:** Simple HTTP polling in submission pages (no real-time push)

**Changes:**
- `package.json` scripts no longer reference websocket directory
- Removed `websocket-client` from `additional_requirements.txt`

### 4. **Removed Comment System**

**Deleted:**
- `judge/comments.py` (not deleted, but disabled)
- `templates/comments/` directory
- `resources/comments.scss`
- Markdown styles: `'comment': MARKDOWN_DEFAULT_STYLE` removed

**URLs removed from `dmoj/urls.py`:**
- `/comments/upvote`
- `/comments/downvote`
- `/comments/hide`
- `/comments/<id>/edit`
- `/comments/<id>/history/ajax`
- `/user/<user>/comment/` page
- Comment feeds: `/feed/comment/rss/` and `/atom/`
- Select2 views: `CommentSelect2View`

**Settings changes in `dmoj/settings.py`:**
- Removed `VNOJ_CP_COMMENT`
- Removed `VNOJ_COMMENT_MIN_CONTRIBUTION`
- Removed `VNOJ_COMMENT_MIN_LENGTH`
- Removed `VNOJ_COMMENT_MAX_LENGTH`
- Removed `VNOJ_COMMENT_BLACKLIST_TERMS`
- Removed `VNOJ_COMMENT_RATE_LIMIT_COUNT`
- Removed `VNOJ_COMMENT_RATE_LIMIT_WINDOW`
- Removed `on_new_comment` discord webhook

**Admin panel:**  
- Removed `judge.Comment` and `judge.CommentLock` from admin

### 5. **Removed Ticket System (Issue Reporting)**

**Deleted:**
- Ticket views (kept in code, but URL removed)
- `templates/ticket/` directory
- `resources/ticket.scss`

**URLs removed from `dmoj/urls.py`:**
- `/problem/<code>/tickets/` (problem ticket list)
- `/problem/<code>/tickets/new` (new problem ticket)
- `/tickets/` (all tickets listing)
- `/tickets/new` (new issue ticket)
- `/ticket/<pk>/` (ticket detail)
- `/ticket/<pk>/open`, `/close`, `/good`, `/norm`, `/notes`
- Select2 views: `TicketUserSelect2View`, `AssigneeSelect2View`
- Preview: `TicketMarkdownPreviewView`

**Settings changes in `dmoj/settings.py`:**
- Removed `VNOJ_CP_TICKET`
- Removed `on_new_ticket` and `on_new_ticket_message` discord webhooks

**Admin panel:**
- Removed `judge.Ticket` from admin

### 6. **Removed Redis/Memcached Dependency**

**Removed:**
- `django-redis` from `requirements.txt`
- Redis cache configuration options

**Replacement:** Django's local-memory caching (default):
```python
CACHES = {}  # Uses local memory
```

**Benefits:**
- Single-server deployments don't need Redis
- No distributed cache sync needed (for 3-5 users)

### 7. **Removed Blog System (Optional)**

**Kept blog functionality** but simplified:
- Settings: Removed `VNOJ_BLOG_MIN_PROBLEM_COUNT`
- Admin still manages blog posts (can be disabled if needed)

**To fully disable blogs, add to `local_settings.py`:**
```python
MIDDLEWARE = [m for m in MIDDLEWARE if m != 'judge.views.blog.enable_blog_module']
```

### 8. **Removed Social Auth**

**Removed:**
- `social-auth-core==4.3.0`
- `social-auth-app-django==5.0.0`

from `requirements.txt`

**Settings changes in `dmoj/settings.py`:**
- `judge/social_auth.py` (not deleted, but not used)
- No OAuth/social login support (standard auth only)

### 9. **Removed Unnecessary Discord/Messaging**

**Removed:**
- `discord-webhook` from `requirements.txt`
- `pika` (RabbitMQ) for message queuing
- `ansi2html` (error formatting for Discord)

**Settings changes:**
- Discord webhooks still defined but set to `None` (inactive)
- Can be re-enabled if needed

### 10. **Locale Simplification**

**Kept:**
- `locale/en/` (English)
- `locale/vi/` (Vietnamese)

**Default language changed:** Vietnamese (`LANGUAGE_CODE = 'vi'`)

---

## File Structure Changes

### Deleted
```
websocket/
  ├── daemon.js
  ├── queue.js
  └── types.d.ts
  
judge/comments.py (logic kept, URLs removed)
judge/event_poster_ws.py (WebSocket logic, can be deleted)
judge/event_poster_amqp.py (AMQP logic, can be deleted)
judge/social_auth.py (OAuth logic, can be deleted)

templates/comments/
templates/ticket/
resources/comments.scss
resources/ticket.scss
```

### Modified
```
dmoj/settings.py
dmoj/urls.py
judge/fixtures/language_pdp.json (NEW)
judge/fixtures/language_small.json (kept for reference)
package.json
requirements.txt
additional_requirements.txt
README.md
templates/base.html
```

### Unchanged (core functionality)
```
judge/models/ (all models kept)
judge/views/ (problem, submission, contest views kept)
judge/forms.py
judge/admin/ (reduced but functional)
templates/problem/
templates/submission/
templates/contest/
resources/base.scss
resources/problem.scss
resources/submission.scss
```

---

## Configuration for Deployment

### `dmoj/local_settings.py`

```python
# Basic settings
DEBUG = False
ALLOWED_HOSTS = ['yourdomain.com', 'localhost']
SECRET_KEY = 'generate-secure-random-key'

# Database (SQLite or PostgreSQL)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

# Problem data
DMOJ_PROBLEM_DATA_ROOT = '/path/to/problem/data'

# Email (optional)
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-app-password'

# Caching (local memory, no Redis needed)
CACHES = {}  # Uses local-memory caching

# Language
LANGUAGE_CODE = 'vi'

# Site info
SITE_NAME = 'PDP-OJ'
SITE_LONG_NAME = 'PDP-OJ: Lightweight Online Judge'
```

### Installation Commands

```bash
# Clone and setup
git clone https://github.com/yourusername/PDP-OJ.git
cd PDP-OJ
pip install -r requirements.txt

# Initial setup
python manage.py migrate
python manage.py createsuperuser
python manage.py loaddata judge/fixtures/language_pdp.json
python manage.py collectstatic --noinput

# Dev server (testing)
python manage.py runserver

# Production (with Gunicorn)
pip install gunicorn
gunicorn dmoj.wsgi:application --bind 0.0.0.0:8000
```

---

## Performance Optimizations

### 1. **Database Indexing**
Problem codes, contest slugs, and user IQs already have indexes.

### 2. **Caching**
- Uses Django's local-memory cache by default
- No need for Redis distribution (small userbase)
- Problem data cached in database

### 3. **Static Files**
- Run `collectstatic` and serve via nginx
- CSS/JS minified via django-compressor

### 4. **Minimal Background Tasks**
- Celery kept for async judging only
- No background webhooks or notifications

### 5. **Database Queries**
- Use `select_related()` and `prefetch_related()` in views
- Avoid N+1 queries when listing problems/contests

---

## Lightweight Architecture Diagram

```
┌─────────────────────────────────────────┐
│         PDP-OJ: 3-5 Users                │
│  Lightweight Online Judge                │
└────┬────────────────────────────────────┘
     │
     ├─────────────────────────────────────────────────────────┐
     │                                                           │
┌────▼─────────────────┐  ┌──────────────────────┐  ┌──────────▼──────┐
│   Django Web Server   │  │   Celery/Judge       │  │  SQLite/Postgres │
│                       │  │   (Async Judging)    │  │   Database       │
│ • Problems           │  │                      │  │                  │
│ • Submissions        │  │ No Redis/RabbitMQ    │  │ Problems, Users, │
│ • Contests           │  │ Direct File I/O      │  │ Contests,        │
│ • Authentication     │  │                      │  │ Submissions      │
│ • Scoreboard         │  │                      │  │                  │
│                       │  │                      │  │                  │
│ NO WebSocket          │  │ CPP17, Python 3 only │  │ Local cache only │
│ NO Comments          │  │                      │  │                  │
│ NO Tickets           │  │                      │  │                  │
│ NO Social Auth       │  │                      │  │                  │
└────┬─────────────────┘  └──────────────────────┘  └──────────┬──────┘
     │                                                           │
     └─────────────────────────────────────────────────────────┘
                            │
                     ┌──────▼──────┐
                     │  nginx       │
                     │  Static      │
                     │  Reverse     │
                     │  Proxy       │
                     └──────┬───────┘
                            │
                     ┌──────▼──────┐
                     │  Browser    │
                     │  (Simple    │
                     │   polling)  │
                     └─────────────┘
```

---

## Testing Checklist

After deployment, verify:

- [ ] Users can register and login
- [ ] Problems display correctly
- [ ] Code submission works
- [ ] Judging returns correct verdicts
- [ ] Scoreboard updates after submissions
- [ ] Contests can be created and viewed
- [ ] Admin can create problems
- [ ] Language dropdown shows C++17 and Python 3 only
- [ ] No WebSocket/real-time errors in console
- [ ] Comments/tickets links don't exist
- [ ] Vietnamese is default language
- [ ] Footer shows PDP-OJ branding

---

## Migration from VNOI OJ

If migrating an existing VNOI instance:

```bash
# 1. Backup data
mysqldump -u user -p database > backup.sql

# 2. Update code
git checkout main
git pull

# 3. Run migrations (check for conflicts)
python manage.py migrate --fake-initial judge

# 4. Reload languages
python manage.py loaddata judge/fixtures/language_pdp.json

# 5. Clear old comments/tickets in admin (optional)
python manage.py shell
# >>> from judge.models import Comment, Ticket
# >>> Comment.objects.all().delete()
# >>> Ticket.objects.all().delete()
```

---

## Troubleshooting

### "Missing language X"
- Load fixture: `python manage.py loaddata judge/fixtures/language_pdp.json`

### Submissions not judging
- Ensure Celery worker is running: `celery -A dmoj worker --loglevel=info`
- Check `DMOJ_PROBLEM_DATA_ROOT` configuration

### Slow queries
- Run `python manage.py shell_plus` and profile views with `django-silk`

### Cache issues
- Clear: `python manage.py shell` → `from django.core.cache import cache; cache.clear()`

---

## Future Enhancements (Optional)

If needed later:

1. **Redis caching** for performance (optional)
2. **PostgreSQL** for better scalability
3. **Gunicorn + Nginx** for production deployment
4. **HTTPS/SSL** for security
5. **Backup automation** with cron jobs
6. **Email notifications** for submissions (keep optional)
7. **Dark mode** (already in UI)
8. **Advanced problem ratings/difficulty** (keep deactivated)

---

## Summary of Removed Features

| Feature | Impact | Reason |
|---------|--------|--------|
| WebSockets | No real-time updates | HTTP polling works fine for 3-5 users |
| Redis | Simpler setup | Local memory caching sufficient |
| Comments | No feedback system | Private group doesn't need public comments |
| Tickets | No issue reporting | Admin can manage directly |
| Blog | Can be disabled | News not needed for training group |
| Social auth | Standard login only | Email registration works |
| Multiple languages | C++/Python only | Most common for training |
| Discord webhooks | No notifications | Emails optional, admin checks manually |

---

## Estimated Deployment Time

- Fresh install: **30-45 minutes**
- Migration from VNOI: **1-2 hours** (data backup + migration + testing)

---

Generated: April 10, 2026
Lightweight OJ Version: PDP-OJ 1.0
