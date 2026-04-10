# PDP-OJ Deployment Guide

## Overview

This guide covers deploying PDP-OJ to a production server for 3-5 users.

**Estimated Time:** 1-2 hours
**Server:** Ubuntu 20.04 LTS or later
**Memory:** 1-2 GB  
**Storage:** 10-50 GB (depends on problem data)

---

## Phase 1: Server Setup

### 1.1 Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### 1.2 Install Dependencies
```bash
sudo apt install -y \
  python3-pip \
  python3-dev \
  postgresql \
  postgresql-contrib \
  nginx \
  supervisor \
  git \
  build-essential \
  libpq-dev \
  python3-venv
```

### 1.3 Create Application User
```bash
sudo useradd -m -s /bin/bash pdp_oj
sudo usermod -aG sudo pdp_oj
sudo su - pdp_oj
```

---

## Phase 2: Database Setup

### 2.1 PostgreSQL Configuration
```bash
sudo -u postgres psql

# In PostgreSQL console:
CREATE DATABASE pdp_oj;
CREATE USER pdp_user WITH PASSWORD 'secure_password_123';
ALTER ROLE pdp_user SET client_encoding TO 'utf8';
ALTER ROLE pdp_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE pdp_user SET default_transaction_deferrable TO on;
ALTER ROLE pdp_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE pdp_oj TO pdp_user;
\q
```

### 2.2 Test Connection
```bash
psql -U pdp_user -h localhost -d pdp_oj
\q  # Exit
```

---

## Phase 3: Application Setup

### 3.1 Clone Repository
```bash
cd /home/pdp_oj
git clone https://github.com/yourusername/PDP-OJ.git pdp-oj
cd pdp-oj
```

### 3.2 Create Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install gunicorn
pip install psycopg2-binary  # PostgreSQL adapter
```

### 3.3 Create Settings File
```bash
cat > dmoj/local_settings.py << 'EOF'
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

DEBUG = False
SECRET_KEY = 'generate-a-random-key-here'  # Use: python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# PostgreSQL Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'pdp_oj',
        'USER': 'pdp_user',
        'PASSWORD': 'secure_password_123',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

# Problem Data Root (CRITICAL)
DMOJ_PROBLEM_DATA_ROOT = '/home/pdp_oj/data'

# Caching (local memory, no Redis)
CACHES = {}

# Email Settings (Optional)
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-app-password'
DEFAULT_FROM_EMAIL = 'noreply@yourdomain.com'

# Language (Vietnamese default)
LANGUAGE_CODE = 'vi'
TIME_ZONE = 'Asia/Ho_Chi_Minh'

# Security
CSRF_TRUSTED_ORIGINS = ['https://yourdomain.com', 'https://www.yourdomain.com']
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_SECURITY_POLICY = {
    'default-src': ("'self'",),
}

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': '/home/pdp_oj/pdp-oj/logs/django.log',
        },
    },
    'root': {
        'handlers': ['file'],
        'level': 'INFO',
    },
}
EOF
```

### 3.4 Create Problem Data Directory
```bash
mkdir -p /home/pdp_oj/data
mkdir -p /home/pdp_oj/pdp-oj/logs
```

### 3.5 Run Migrations
```bash
cd /home/pdp_oj/pdp-oj
source venv/bin/activate
python manage.py migrate
python manage.py createsuperuser  # Create admin account
python manage.py loaddata judge/fixtures/language_pdp.json
python manage.py collectstatic --noinput
deactivate
```

---

## Phase 4: Supervisor Configuration

Supervisor keeps Django and Celery running automatically.

### 4.1 Create Gunicorn Config
```bash
sudo tee /etc/supervisor/conf.d/pdp-oj-web.conf > /dev/null << 'EOF'
[program:pdp-oj-web]
directory=/home/pdp_oj/pdp-oj
command=/home/pdp_oj/pdp-oj/venv/bin/gunicorn \
    --workers 4 \
    --bind 127.0.0.1:8000 \
    --timeout 60 \
    dmoj.wsgi:application
user=pdp_oj
autostart=true
autorestart=true
stderr_logfile=/home/pdp_oj/pdp-oj/logs/gunicorn.err.log
stdout_logfile=/home/pdp_oj/pdp-oj/logs/gunicorn.out.log
EOF
```

### 4.2 Create Celery Worker Config
```bash
sudo tee /etc/supervisor/conf.d/pdp-oj-worker.conf > /dev/null << 'EOF'
[program:pdp-oj-worker]
directory=/home/pdp_oj/pdp-oj
command=/home/pdp_oj/pdp-oj/venv/bin/celery \
    -A dmoj \
    worker \
    --loglevel=info \
    --concurrency=2
user=pdp_oj
autostart=true
autorestart=true
stderr_logfile=/home/pdp_oj/pdp-oj/logs/celery.err.log
stdout_logfile=/home/pdp_oj/pdp-oj/logs/celery.out.log
EOF
```

### 4.3 Start Services
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl status
```

---

## Phase 5: Nginx Configuration

### 5.1 Create Nginx Config
```bash
sudo tee /etc/nginx/sites-available/pdp-oj > /dev/null << 'EOF'
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL Certificates (Let's Encrypt)
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Static Files
    location /static/ {
        alias /home/pdp_oj/pdp-oj/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Media Files
    location /media/ {
        alias /home/pdp_oj/pdp-oj/media/;
    }

    # Gunicorn Proxy
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
        client_max_body_size 10M;  # Allow large submissions
    }
}
EOF
```

### 5.2 Enable Nginx Site
```bash
sudo ln -s /etc/nginx/sites-available/pdp-oj /etc/nginx/sites-enabled/
sudo nginx -t  # Test config
sudo systemctl restart nginx
```

---

## Phase 6: SSL Certificate (Let's Encrypt)

### 6.1 Install Certbot
```bash
sudo apt install -y certbot python3-certbot-nginx
```

### 6.2 Get Certificate
```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

### 6.3 Auto-Renewal
```bash
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

---

## Phase 7: Firewall Configuration

```bash
# Enable UFW
sudo ufw enable

# Allow SSH, HTTP, HTTPS
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Check status
sudo ufw status
```

---

## Phase 8: Backup Strategy

### 8.1 Daily Database Backup
```bash
sudo tee /etc/cron.daily/pdp-oj-backup > /dev/null << 'EOF'
#!/bin/bash
BACKUP_DIR="/home/pdp_oj/backups"
mkdir -p $BACKUP_DIR
pg_dump -U pdp_user pdp_oj | gzip > $BACKUP_DIR/pdp_oj_$(date +\%Y\%m\%d).sql.gz
# Keep only 7 days of backups
find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete
EOF
sudo chmod +x /etc/cron.daily/pdp-oj-backup
```

### 8.2 Problem Data Backup
```bash
sudo tee /etc/cron.weekly/pdp-oj-data > /dev/null << 'EOF'
#!/bin/bash
BACKUP_DIR="/home/pdp_oj/backups"
mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/pdp_data_$(date +\%Y\%m\%d).tar.gz /home/pdp_oj/data/
# Keep only 4 weeks
find $BACKUP_DIR -name "pdp_data_*.tar.gz" -mtime +28 -delete
EOF
sudo chmod +x /etc/cron.weekly/pdp-oj-data
```

---

## Phase 9: Monitoring

### 9.1 Check Service Status
```bash
# Web server
sudo supervisorctl status pdp-oj-web

# Worker
sudo supervisorctl status pdp-oj-worker

# Nginx
sudo systemctl status nginx
```

### 9.2 View Logs
```bash
# Gunicorn
tail -f /home/pdp_oj/pdp-oj/logs/gunicorn.out.log

# Celery Worker
tail -f /home/pdp_oj/pdp-oj/logs/celery.out.log

# Nginx
sudo tail -f /var/log/nginx/error.log
```

### 9.3 System Health
```bash
# CPU/Memory
free -h
top

# Disk
df -h

# Database
sudo -u postgres psql -c "SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database WHERE datname = 'pdp_oj';"
```

---

## Phase 10: First Test

1. Navigate to `https://yourdomain.com`
2. Login with admin account
3. Create a test problem
4. Submit a solution
5. Check if judging works

---

## Troubleshooting

### Submissions not judging
```bash
# Check worker
sudo supervisorctl status pdp-oj-worker

# Restart if needed
sudo supervisorctl restart pdp-oj-worker

# Check logs
tail -f /home/pdp_oj/pdp-oj/logs/celery.out.log
```

### Pages slow or timing out
```bash
# Check Gunicorn workers
ps aux | grep gunicorn

# Increase workers (edit supervisor config)
--workers 8  # Instead of 4

# Restart
sudo supervisorctl restart pdp-oj-web
```

### Database connection errors
```bash
# Test PostgreSQL
sudo -u postgres psql pdp_oj

# Check settings in local_settings.py
cat dmoj/local_settings.py | grep DATABASE
```

### Certificate renewal issues
```bash
# Test renewal
sudo certbot renew --dry-run

# Manual renewal
sudo certbot renew --force-renewal
```

---

## Performance Tuning

### PostgreSQL
```bash
sudo tee -a /etc/postgresql/12/main/postgresql.conf > /dev/null << 'EOF'
# Increase shared buffers
shared_buffers = 256MB

# Increase effective cache size
effective_cache_size = 1GB

# Increase work_mem
work_mem = 64MB
EOF
sudo systemctl restart postgresql
```

### Nginx Caching
```bash
# In /etc/nginx/sites-available/pdp-oj, add:
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=pdp_cache:10m;

location / {
    proxy_cache pdp_cache;
    proxy_cache_valid 200 10m;
    ...
}
```

---

## Maintenance Tasks

### Weekly
- Check server disk space
- Verify backups exist
- Monitor error logs

### Monthly
- Update system: `sudo apt update && sudo apt upgrade`
- Review user accounts
- Test backup restoration

### Quarterly
- Database optimization: `VACUUM; ANALYZE;`
- Security audit
- Performance review

---

## Shutdown Procedure
```bash
# Graceful shutdown
sudo supervisorctl stop pdp-oj-worker
sudo supervisorctl stop pdp-oj-web
sudo systemctl stop nginx
sudo systemctl stop postgresql
```

---

## Support

For issues, check:
- `REFACTOR_GUIDE.md` (architecture)
- `QUICKSTART.md` (setup basics)
- Official DMOJ docs (core OJ features)

---

**Deployment Complete! 🚀**
