# Deployment Guide

This guide covers deployment strategies for Telegram bots and Django websites.

## Telegram Bot Deployment

### Deployment Options

#### 1. VPS/Cloud Server (Recommended)
- **Providers**: DigitalOcean, AWS EC2, Google Cloud, Azure
- **Pros**: Full control, always running
- **Cons**: Requires server management

#### 2. Heroku
- **Pros**: Easy deployment, free tier available
- **Cons**: May have cold starts

#### 3. PythonAnywhere
- **Pros**: Python-focused, easy setup
- **Cons**: Limited free tier

### VPS Deployment Steps

1. **Prepare your server:**
   ```bash
   # Update system
   sudo apt update && sudo apt upgrade -y
   
   # Install Python and pip
   sudo apt install python3 python3-pip python3-venv -y
   
   # Install git
   sudo apt install git -y
   ```

2. **Clone your bot:**
   ```bash
   git clone <your-repo-url>
   cd panda/telegram_bots/your_bot
   ```

3. **Set up virtual environment:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

4. **Configure environment variables:**
   ```bash
   nano .env
   # Add your BOT_TOKEN and other variables
   ```

5. **Run with systemd (recommended):**
   ```bash
   sudo nano /etc/systemd/system/your_bot.service
   ```
   
   Add:
   ```ini
   [Unit]
   Description=Telegram Bot
   After=network.target
   
   [Service]
   Type=simple
   User=your_user
   WorkingDirectory=/path/to/bot
   ExecStart=/path/to/venv/bin/python bot.py
   Restart=always
   
   [Install]
   WantedBy=multi-user.target
   ```
   
   Enable and start:
   ```bash
   sudo systemctl enable your_bot
   sudo systemctl start your_bot
   sudo systemctl status your_bot
   ```

### Using Docker for Bots

Create `Dockerfile`:
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "bot.py"]
```

Create `docker-compose.yml`:
```yaml
version: '3.8'
services:
  bot:
    build: .
    restart: always
    env_file:
      - .env
```

Deploy:
```bash
docker-compose up -d
```

## Django Website Deployment

### Deployment Options

#### 1. VPS with Nginx + Gunicorn (Recommended)
- **Providers**: DigitalOcean, Linode, AWS
- **Pros**: Full control, best performance
- **Cons**: Requires configuration

#### 2. PaaS Solutions
- **Heroku**: Easy deployment
- **PythonAnywhere**: Django-friendly
- **Railway**: Modern alternative to Heroku

#### 3. Container Platforms
- **Docker + AWS ECS**
- **Google Cloud Run**
- **Azure Container Instances**

### VPS Deployment with Nginx + Gunicorn

1. **Prepare server:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install python3 python3-pip python3-venv nginx -y
   ```

2. **Install PostgreSQL (if needed):**
   ```bash
   sudo apt install postgresql postgresql-contrib -y
   sudo -u postgres psql
   CREATE DATABASE your_db;
   CREATE USER your_user WITH PASSWORD 'your_password';
   GRANT ALL PRIVILEGES ON DATABASE your_db TO your_user;
   \q
   ```

3. **Clone and setup project:**
   ```bash
   git clone <your-repo-url>
   cd panda/django_websites/your_project
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   pip install gunicorn
   ```

4. **Configure Django settings:**
   ```bash
   nano .env
   # Add:
   # DEBUG=False
   # SECRET_KEY=your-secret-key
   # ALLOWED_HOSTS=your-domain.com
   # DATABASE_URL=postgresql://user:password@localhost/dbname
   ```

5. **Prepare static files:**
   ```bash
   python manage.py collectstatic --noinput
   python manage.py migrate
   ```

6. **Configure Gunicorn:**
   ```bash
   sudo nano /etc/systemd/system/gunicorn.service
   ```
   
   Add:
   ```ini
   [Unit]
   Description=Gunicorn daemon for Django project
   After=network.target
   
   [Service]
   User=your_user
   Group=www-data
   WorkingDirectory=/path/to/project
   ExecStart=/path/to/venv/bin/gunicorn --workers 3 --bind unix:/path/to/project.sock project_name.wsgi:application
   
   [Install]
   WantedBy=multi-user.target
   ```
   
   Enable and start:
   ```bash
   sudo systemctl enable gunicorn
   sudo systemctl start gunicorn
   ```

7. **Configure Nginx:**
   ```bash
   sudo nano /etc/nginx/sites-available/your_project
   ```
   
   Add:
   ```nginx
   server {
       listen 80;
       server_name your-domain.com;
   
       location = /favicon.ico { access_log off; log_not_found off; }
       
       location /static/ {
           root /path/to/project;
       }
       
       location /media/ {
           root /path/to/project;
       }
   
       location / {
           proxy_pass http://unix:/path/to/project.sock;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
   }
   ```
   
   Enable site:
   ```bash
   sudo ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl restart nginx
   ```

8. **Setup SSL with Let's Encrypt:**
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   sudo certbot --nginx -d your-domain.com
   ```

### Using Docker for Django

Create `Dockerfile`:
```dockerfile
FROM python:3.11-slim

ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN python manage.py collectstatic --noinput

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "project_name.wsgi:application"]
```

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=your_db
      - POSTGRES_USER=your_user
      - POSTGRES_PASSWORD=your_password
  
  web:
    build: .
    command: gunicorn project_name.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - .:/app
      - static_volume:/app/static
      - media_volume:/app/media
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

## Monitoring and Maintenance

### Logging

For Telegram bots:
```python
import logging
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
```

For Django:
```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': '/path/to/django.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}
```

### Monitoring Tools

- **Uptime monitoring**: UptimeRobot, Pingdom
- **Error tracking**: Sentry
- **Performance**: New Relic, DataDog
- **Server monitoring**: Netdata, Prometheus + Grafana

### Backups

Regular backups are essential:

For PostgreSQL:
```bash
# Backup
pg_dump -U user dbname > backup.sql

# Restore
psql -U user dbname < backup.sql
```

Automate with cron:
```bash
crontab -e
# Add: Daily backup at 2 AM
0 2 * * * pg_dump -U user dbname > /backups/db_$(date +\%Y\%m\%d).sql
```

## Continuous Deployment

### Using GitHub Actions

Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /path/to/project
            git pull
            source venv/bin/activate
            pip install -r requirements.txt
            python manage.py migrate
            python manage.py collectstatic --noinput
            sudo systemctl restart gunicorn
```

## Security Checklist

- [ ] Use HTTPS/SSL certificates
- [ ] Keep dependencies updated
- [ ] Use environment variables for secrets
- [ ] Enable firewall (ufw on Ubuntu)
- [ ] Regular security updates
- [ ] Use strong passwords
- [ ] Enable fail2ban for SSH protection
- [ ] Regular backups
- [ ] Monitor logs for suspicious activity

## Resources

- [Django Deployment Checklist](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/)
- [Digital Ocean Django Deployment Guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Gunicorn Documentation](https://docs.gunicorn.org/)
