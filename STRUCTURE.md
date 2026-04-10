# PDP-OJ Project Structure

Generated: April 10, 2026  
Project: VNOI OJ → PDP-OJ (Lightweight Online Judge)

---

## Directory Tree (Simplified)

```
pdp-oj/
├── README.md                          ✅ UPDATED - PDP-OJ branding
├── QUICKSTART.md                      ✅ NEW - 5-minute setup
├── REFACTOR_GUIDE.md                  ✅ NEW - Technical reference
├── DEPLOYMENT.md                      ✅ NEW - Production guide
├── PROJECT_SUMMARY.md                 ✅ NEW - Complete overview
├── CHANGES_CHECKLIST.md               ✅ NEW - Detailed checklist
│
├── dmoj/                              Django project root
│   ├── __init__.py
│   ├── settings.py                    ✅ MODIFIED - Branding + settings cleanup
│   ├── urls.py                        ✅ MODIFIED - 26 routes removed
│   ├── wsgi.py
│   ├── wsgi_async.py
│   ├── celery.py
│   ├── local_settings.py              📝 TO CREATE - Local configuration
│   ├── middleware.py
│   ├── throttle_discord_webhook.py
│   ├── throttle_mail.py
│   ├── admin/
│   │   └── (admin site config)
│   └── ...
│
├── judge/                             Core OJ application
│   ├── __init__.py
│   ├── apps.py
│   ├── models/                        ✅ UNCHANGED - All intact
│   │   ├── __init__.py
│   │   ├── problem.py
│   │   ├── submission.py
│   │   ├── contest.py
│   │   ├── runtime.py
│   │   ├── comment.py                 ⚠️  DISABLED (model exists, not used)
│   │   ├── ticket.py                  ⚠️  DISABLED (model exists, not used)
│   │   └── ...
│   │
│   ├── views/                         ✅ UNCHANGED - Core views intact
│   │   ├── problem.py                 ✅ Problem display
│   │   ├── submission.py              ✅ Submission handling
│   │   ├── contests.py                ✅ Contest management
│   │   ├── comment.py                 ⚠️  NOT IMPORTED (disabled)
│   │   ├── ticket.py                  ⚠️  NOT IMPORTED (disabled)
│   │   ├── blog.py                    ✅ Blog (can be disabled)
│   │   ├── user.py                    ✅ User profiles
│   │   └── ...
│   │
│   ├── admin/                         ✅ MODIFIED - Comment/Ticket removed
│   │   └── (admin site configuration)
│   │
│   ├── fixtures/
│   │   ├── language_small.json        📚 Original language data
│   │   ├── language_pdp.json          ✅ NEW - Only C++17, Python3
│   │   ├── language_all.json          📚 Original (for reference)
│   │   ├── demo.json                  📚 Demo data
│   │   └── navbar.json                📚 Navigation
│   │
│   ├── migrations/                    ✅ UNCHANGED - All intact
│   ├── management/
│   ├── tests/
│   ├── utils/
│   ├── widgets/
│   ├── jinja2/
│   │
│   ├── comments.py                    ⚠️  EXISTS but NOT USED
│   ├── event_poster.py                ✅ Basic poster
│   ├── event_poster_amqp.py           ⚠️  EXISTS but NOT USED
│   ├── event_poster_ws.py             ⚠️  EXISTS but NOT USED (WebSocket)
│   ├── social_auth.py                 ⚠️  EXISTS but NOT USED
│   ├── feed.py                        ✅ Feed generation
│   ├── caching.py                     ✅ Cache layer (local memory)
│   ├── signals.py                     ✅ Django signals
│   ├── sitemap.py                     ✅ Sitemap
│   ├── fulltext.py                    ✅ Full-text search
│   ├── performance_points.py          ✅ Rating calculation
│   ├── highlight_code.py              ✅ Code highlighting
│   ├── lxml_tree.py
│   ├── custom_translations.py
│   ├── dblock.py
│   ├── judgeapi.py                    ✅ Judge communication
│   ├── ip_auth.py
│   ├── judge_priority.py
│   ├── ratings.py
│   ├── timezone.py
│   ├── user_log.py
│   ├── user_translations.py
│   └── ...
│
├── websocket/                         ⚠️  REMOVED (can delete)
│   ├── daemon.js
│   ├── queue.js
│   ├── types.d.ts
│   └── config.js (gitignored)
│
├── templates/
│   ├── base.html                      ✅ MODIFIED - Footer updated
│   ├── home.html                      ✅ Home page
│   │
│   ├── problem/
│   │   ├── list.html                  ✅ Problem list
│   │   ├── detail.html                ✅ Problem display
│   │   ├── submit.html                ✅ Submission form
│   │   └── ...
│   │
│   ├── submission/
│   │   ├── detail.html                ✅ Submission detail
│   │   ├── list.html                  ✅ Submission list
│   │   └── ...
│   │
│   ├── contest/
│   │   ├── list.html                  ✅ Contest list
│   │   ├── detail.html                ✅ Contest detail
│   │   ├── standings.html             ✅ Scoreboard
│   │   └── ...
│   │
│   ├── user/
│   │   ├── profile.html               ✅ User profile
│   │   ├── dashboard.html             ✅ User dashboard
│   │   └── ...
│   │
│   ├── registration/
│   │   ├── login.html                 ✅ Login page
│   │   ├── register.html              ✅ Registration
│   │   └── ...
│   │
│   ├── admin/
│   │   └── (admin templates)
│   │
│   ├── comments/                      ⚠️  DISABLED (can delete)
│   │   └── (comment templates)
│   │
│   ├── ticket/                        ⚠️  DISABLED (can delete)
│   │   └── (ticket templates)
│   │
│   ├── blog/
│   │   ├── list.html                  ✅ Blog list
│   │   ├── post.html                  ✅ Blog detail
│   │   └── ...
│   │
│   ├── stats/
│   │   └── (statistics pages)
│   │
│   ├── organization/
│   │   └── (organization pages)
│   │
│   └── ... (other templates)
│
├── resources/                         CSS/JS/Static files
│   ├── style.scss                     ✅ Main stylesheet
│   ├── base.scss                      ✅ Base styles
│   ├── problem.scss                   ✅ Problem styles
│   ├── submission.scss                ✅ Submission styles
│   ├── contest.scss                   ✅ Contest styles
│   ├── navbar.scss                    ✅ Navigation styles
│   ├── comments.scss                  ⚠️  DISABLED (can delete)
│   ├── ticket.scss                    ⚠️  DISABLED (can delete)
│   │
│   ├── common.js                      ✅ Common JS
│   ├── event.js                       ✅ Event handling (polling fallback)
│   ├── user_profile.js                ✅ User profile JS
│   ├── blog-toc.js                    ✅ Blog table of contents
│   │
│   ├── ace/                           ✅ Ace editor (code)
│   ├── icons/                         ✅ Icon set
│   ├── libs/                          ✅ Third-party libraries
│   ├── select2/                       ✅ Select2 dropdown
│   ├── admin/                         ✅ Admin styles
│   ├── vnoj/                          📚 Theme files
│   ├── wpadmin/                       📚 Admin theme
│   └── ...
│
├── locale/
│   ├── en/                            ✅ English translations
│   │   └── LC_MESSAGES/
│   │       ├── django.po
│   │       └── django.mo
│   │
│   └── vi/                            ✅ Vietnamese translations (default)
│       └── LC_MESSAGES/
│           ├── django.po
│           └── django.mo
│
├── django_ace/                        ACE editor integration
│   ├── __init__.py
│   ├── widgets.py
│   └── static/
│
├── martor/                            Markdown editor
│   ├── __init__.py
│   ├── api.py
│   ├── views.py
│   ├── widgets.py
│   ├── static/
│   └── templates/
│
├── urlshortener/                      URL shortener service
│   └── ...
│
├── scripts/
│   ├── check-package-installed.js
│   └── ...
│
├── manage.py                          Django management
├── package.json                       ✅ MODIFIED - Name updated
├── package-lock.json
├── requirements.txt                   ✅ MODIFIED - Dependencies cleaned
├── additional_requirements.txt        ✅ MODIFIED - websocket-client removed
├── .flake8                            Python linting
├── .prettierrc                        JS formatting
├── .prettierignore
├── .gitignore                         Git ignores
├── .gitmodules
├── LICENSE                            AGPLv3
├── contributing.md                    Contribution guide
├── codecov.yml                        Code coverage
├── robots.txt                         SEO
├── logo.png                           Site logo
├── 502.html                           Error page
├── make_style.sh                      Build styles
├── dmoj_bridge_async.py              Judge bridge
├── dmoj_celery.py                    Celery config
├── dmoj_install_pymysql.py           MySQL setup
│
└── static/                            Generated static files
    ├── css/
    ├── js/
    ├── fonts/
    ├── icons/
    └── ... (collectstatic output)
```

---

## Detailed Status by Component

### ✅ WORKING (Fully Functional)
- User authentication (register, login, profiles)
- Problem management (list, display, submit)
- Code submission and judging
- Judge verdicts (AC, WA, TLE, MLE, RE, etc.)
- Scoreboard and rankings
- Contest creation and management
- Admin panel
- Code highlighting
- Markdown rendering
- Problem ratings and difficulty
- User badges and achievements
- Django admin interface
- Localization (English, Vietnamese)
- Static file serving
- Database models

### ⚠️ DISABLED (Code exists but unused)
- Comments system (URLs removed, model intact)
- Ticket/Issue system (URLs removed, model intact)
- WebSocket real-time updates (code kept for reference)
- Social authentication (dependency removed)
- Discord webhook notifications (optional)
- Blog system (can be re-enabled)
- RabbitMQ/pika queue (not used)

### ❌ REMOVED (Deleted)
- WebSocket client library (requirement)
- Redis cache library (requirement)
- Social auth dependencies (requirement)
- Discord webhook library (requirement)
- RabbitMQ client (requirement)
- WebSocket URL routes
- Comment URL routes
- Ticket URL routes
- Comment/Ticket markdown styles
- Comment/Ticket form imports
- Comment/Ticket view imports

### 📚 REFERENCES (For future use)
- Original language fixtures (language_all.json)
- Original theme files (vnoj/, wpadmin/)
- DMOJ original implementations

---

## Data Flow

### User Submission Flow
```
Browser
  ↓
Django View (submission.py)
  ↓
Database Store
  ↓
Celery Task Queue (async)
  ↓
Judge Worker
  ↓
Execute Code (C++17 or Python3)
  ↓
Compare Output
  ↓
Store Verdict (AC/WA/TLE/etc)
  ↓
Update Scoreboard
  ↓
Browser Refresh (polling, no WebSocket)
```

### Cache Architecture
```
Browser
  ↓
Nginx (static files, caching)
  ↓
Gunicorn (Django app)
  ↓
Local Memory Cache (CACHES = {})
  ↓
PostgreSQL Database
```

**No Redis, no distributed cache needed**

---

## Key Differences from VNOI OJ

| Feature | VNOI | PDP-OJ |
|---------|------|--------|
| Users | 100+ | 3-5 |
| Runtimes | 20+ | 2 |
| Real-time | WebSocket | Polling |
| Comments | Yes | No |
| Tickets | Yes | No |
| Cache | Redis | Local memory |
| Locales | Many | 2 (EN, VI) |
| Blog | Yes | Disabled |
| Social Auth | Yes | No |
| Notifications | Discord | Email |
| Complexity | High | Low |

---

## Installation Folder Structure

### Development
```
~/projects/pdp-oj/                   (repository clone)
├── manage.py
├── db.sqlite3                       (dev database)
├── venv/                            (virtual environment)
└── ... (all code)
```

### Production
```
/home/pdp_oj/pdp-oj/                 (repository clone)
├── manage.py
├── static/                          (collectstatic output)
├── logs/                            (log files)
├── venv/                            (virtual environment)
└── data/                            (problem test cases)

/home/pdp_oj/backups/                (database backups)
├── pdp_oj_20260410.sql.gz
└── pdp_data_20260410.tar.gz

/var/log/
├── nginx/
└── supervisor/
```

---

## File Sizes (Approximate)

| Item | Size |
|------|------|
| Repository (code only) | ~150 MB |
| Database (fresh) | ~5 MB |
| Database (with problems) | 50-500 MB |
| Problem test cases | 100MB-10GB+ |
| Static files (compiled) | ~50 MB |
| Documentation | ~6 MB |

---

## Generated Files by Phase

### Phase 1: Executable Files
- manage.py
- dmoj_celery.py
- dmoj_bridge_async.py
- make_style.sh

### Phase 2: Configuration Files
- requirements.txt
- additional_requirements.txt
- package.json
- .flake8
- .prettierrc

### Phase 3: Documentation Files (NEW)
- README.md (rewritten)
- QUICKSTART.md (new)
- REFACTOR_GUIDE.md (new)
- DEPLOYMENT.md (new)
- PROJECT_SUMMARY.md (new)
- CHANGES_CHECKLIST.md (new)
- STRUCTURE.md (this file)

### Phase 4: Data Files
- judge/fixtures/language_pdp.json (new)
- db.sqlite3 (created on migrate)

---

## Git History

### Preserved
- All original code (safe to revert)
- All models and migrations
- All template files
- All style files
- All javascript

### Modified in-place
- dmoj/settings.py
- dmoj/urls.py
- templates/base.html
- package.json
- requirements.txt
- additional_requirements.txt
- README.md

### New Files
- QUICKSTART.md
- REFACTOR_GUIDE.md
- DEPLOYMENT.md
- PROJECT_SUMMARY.md
- CHANGES_CHECKLIST.md
- judge/fixtures/language_pdp.json

**No files permanently deleted.** All changes can be reverted if needed.

---

## Performance Characteristics

### Memory Usage
| Component | Memory |
|-----------|--------|
| Django process | 100-200 MB |
| Celery worker | 100-150 MB |
| PostgreSQL | 100-200 MB |
| Nginx | 10-20 MB |
| OS baseline | 200-300 MB |
| **Total at rest** | **~600-900 MB** |
| Peak with traffic | ~1.5-2 GB |

### Database Size
- SQLite: 10-20 MB (small deployments)
- PostgreSQL: 50-500 MB (depends on data)
- Backups: 5-50 MB (compressed)

### Disk Space
- Code: 150 MB
- Static files: 50 MB
- Database: 100-500 MB
- Problem data: 100MB-10GB+
- Backups: 50-500 MB
- **Total needed: 500MB-15GB**

---

**Project Structure: Complete and Documented** ✅
