# ⚡ LeadFlow CRM

A full-stack Customer Relationship Management web application built with Python, Django, MySQL, Bootstrap, and Django REST Framework.

---

## 📁 Project Structure

```
leadflow_crm/
├── leadflow/                  ← Django project config
│   ├── settings.py            ← All settings (DB, apps, etc.)
│   ├── urls.py                ← Root URL configuration
│   └── wsgi.py                ← Web server entry point
│
├── apps/
│   ├── authentication/        ← Login, Register, Logout, Profile
│   │   ├── models.py          ← UserProfile model
│   │   ├── views.py           ← Auth views
│   │   ├── forms.py           ← Login/Register forms
│   │   ├── urls.py            ← Auth URL routes
│   │   └── admin.py           ← Admin customization
│   │
│   └── leads/                 ← Core CRM functionality
│       ├── models.py          ← Lead + ActivityLog models
│       ├── views.py           ← CRUD views
│       ├── forms.py           ← Lead form
│       ├── urls.py            ← Lead URL routes
│       ├── api_views.py       ← REST API views
│       ├── api_urls.py        ← API URL routes
│       ├── serializers.py     ← DRF serializers
│       └── admin.py           ← Admin customization
│
├── templates/
│   ├── base.html              ← Master layout with sidebar
│   ├── authentication/
│   │   ├── login.html
│   │   ├── register.html
│   │   └── profile.html
│   └── leads/
│       ├── dashboard.html
│       ├── lead_list.html
│       ├── lead_form.html
│       ├── lead_detail.html
│       ├── lead_confirm_delete.html
│       └── activity_log.html
│
├── static/
│   ├── css/main.css
│   └── js/main.js
│
├── media/                     ← Uploaded files (profile pictures)
├── logs/                      ← Application logs
├── requirements.txt
├── .env.example
└── .gitignore
```

---

## 🚀 Installation (Local Development)

### Step 1: Clone the Project
```bash
git clone https://github.com/shubhamkumarus/leadflow-crm.git
cd leadflow-crm
```

### Step 2: Create Virtual Environment
```bash
# Create virtual environment
python -m venv venv

# Activate it
# On Windows:
venv\Scripts\activate
# On Linux/Mac:
source venv/bin/activate
```

### Step 3: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 4: Set Up MySQL Database
```sql
-- Run these commands in MySQL:
CREATE DATABASE leadflow_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'leadflow_user'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON leadflow_db.* TO 'leadflow_user'@'localhost';
FLUSH PRIVILEGES;
```

### Step 5: Configure Environment Variables
```bash
# Copy the example file
cp .env.example .env

# Edit .env with your values:
SECRET_KEY=your-random-secret-key
DEBUG=True
DB_NAME=leadflow_db
DB_USER=leadflow_user
DB_PASSWORD=your_password
DB_HOST=localhost
DB_PORT=3306
```

### Step 6: Run Migrations
```bash
# Creates all database tables
python manage.py makemigrations
python manage.py migrate
```

### Step 7: Create Superuser (Admin)
```bash
python manage.py createsuperuser
# Follow the prompts to set username, email, password
```

### Step 8: Create UserProfile for Superuser
```bash
python manage.py shell
```
```python
from django.contrib.auth.models import User
from apps.authentication.models import UserProfile
user = User.objects.get(username='your_superuser_name')
UserProfile.objects.create(user=user, role='admin')
exit()
```

### Step 9: Collect Static Files
```bash
python manage.py collectstatic
```

### Step 10: Run the Server
```bash
python manage.py runserver
```

Visit: **http://localhost:8000**

---

## 🗃️ Database Schema

### users (Django built-in)
| Column     | Type         | Description          |
|------------|--------------|----------------------|
| id         | INT PK       | Auto increment       |
| username   | VARCHAR(150) | Unique username      |
| email      | VARCHAR(254) | Email address        |
| password   | VARCHAR(128) | Hashed password      |
| first_name | VARCHAR(150) | First name           |
| last_name  | VARCHAR(150) | Last name            |
| is_active  | BOOLEAN      | Account active?      |

### authentication_userprofile
| Column          | Type        | Description            |
|-----------------|-------------|------------------------|
| id              | INT PK      | Auto increment         |
| user_id         | FK → User   | One-to-one with User   |
| role            | VARCHAR(20) | 'admin' or 'employee'  |
| phone           | VARCHAR(15) | Optional phone         |
| profile_picture | VARCHAR     | Image path             |
| created_at      | DATETIME    | Profile creation time  |

### leads_lead
| Column        | Type          | Description                          |
|---------------|---------------|--------------------------------------|
| id            | INT PK        | Auto increment                       |
| name          | VARCHAR(200)  | Lead's full name                     |
| email         | VARCHAR       | Email (optional)                     |
| phone         | VARCHAR(20)   | Phone number                         |
| company       | VARCHAR(200)  | Company (optional)                   |
| address       | TEXT          | Address (optional)                   |
| source        | VARCHAR(50)   | instagram/facebook/whatsapp etc.     |
| status        | VARCHAR(50)   | new/contacted/follow_up/converted/lost |
| priority      | VARCHAR(10)   | low/medium/high                      |
| budget        | DECIMAL(12,2) | Budget amount                        |
| notes         | TEXT          | Notes about the lead                 |
| follow_up_date| DATE          | Scheduled follow-up                  |
| assigned_to_id| FK → User     | Employee responsible                 |
| created_by_id | FK → User     | Who created the lead                 |
| created_at    | DATETIME      | Auto-set on creation                 |
| updated_at    | DATETIME      | Auto-updated on save                 |

### leads_activitylog
| Column      | Type        | Description               |
|-------------|-------------|---------------------------|
| id          | INT PK      | Auto increment            |
| lead_id     | FK → Lead   | Related lead (nullable)   |
| user_id     | FK → User   | Who performed the action  |
| action      | VARCHAR(50) | created/updated/deleted   |
| description | TEXT        | Human-readable description|
| created_at  | DATETIME    | When it happened          |

---

## 🌐 REST API Endpoints

Base URL: `http://localhost:8000/api/v1/`

### Get Authentication Token
```bash
POST /api/v1/token/
Body: { "username": "admin", "password": "yourpassword" }
Response: { "token": "abc123..." }
```

### List Leads
```bash
GET /api/v1/leads/
Header: Authorization: Token abc123...

# With filters:
GET /api/v1/leads/?status=new&source=instagram&search=john
```

### Create Lead
```bash
POST /api/v1/leads/
Header: Authorization: Token abc123...
Body: {
  "name": "John Doe",
  "phone": "9876543210",
  "email": "john@example.com",
  "source": "instagram",
  "status": "new",
  "priority": "high"
}
```

### Get Single Lead
```bash
GET /api/v1/leads/1/
Header: Authorization: Token abc123...
```

### Update Lead
```bash
PATCH /api/v1/leads/1/
Header: Authorization: Token abc123...
Body: { "status": "converted" }
```

### Delete Lead
```bash
DELETE /api/v1/leads/1/
Header: Authorization: Token abc123...
```

### Dashboard Stats
```bash
GET /api/v1/leads/stats/
Header: Authorization: Token abc123...
Response: {
  "total": 50,
  "by_status": { "new": 10, "converted": 15 },
  "by_source": { "instagram": 20, "facebook": 10 }
}
```

---

## 🔐 Role-Based Access

| Feature               | Admin | Employee |
|-----------------------|-------|----------|
| View all leads        | ✅    | ❌ (own only) |
| Add lead              | ✅    | ✅        |
| Edit lead             | ✅    | ✅ (own)  |
| Delete lead           | ✅    | ❌        |
| Assign leads          | ✅    | ❌        |
| View all activity logs| ✅    | ❌ (own)  |
| Admin panel (/admin/) | ✅    | ❌        |

---

## 🖥️ Deployment on VPS / Hostinger

### Step 1: Prepare Your VPS
```bash
# SSH into your server
ssh root@your-server-ip

# Update system
apt update && apt upgrade -y

# Install required packages
apt install python3 python3-pip python3-venv mysql-server nginx -y
```

### Step 2: Upload Project
```bash
# Option A: Git (recommended)
git clone https://github.com/yourusername/leadflow-crm.git /var/www/leadflow

# Option B: SCP from local
scp -r ./leadflow_crm root@your-server-ip:/var/www/leadflow
```

### Step 3: Set Up Python Environment
```bash
cd /var/www/leadflow
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Step 4: Configure Production .env
```bash
nano .env
# Set:
# DEBUG=False
# ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
# SECRET_KEY=very-long-random-production-key
# DB settings...
```

### Step 5: Run Migrations
```bash
python manage.py migrate
python manage.py collectstatic --noinput
python manage.py createsuperuser
```

### Step 6: Configure Gunicorn (WSGI Server)
```bash
# Test Gunicorn works:
gunicorn leadflow.wsgi:application --bind 0.0.0.0:8000

# Create systemd service:
nano /etc/systemd/system/leadflow.service
```
```ini
[Unit]
Description=LeadFlow CRM Gunicorn
After=network.target

[Service]
User=www-data
WorkingDirectory=/var/www/leadflow
ExecStart=/var/www/leadflow/venv/bin/gunicorn leadflow.wsgi:application --workers 3 --bind unix:/run/leadflow.sock
Restart=always

[Install]
WantedBy=multi-user.target
```
```bash
systemctl daemon-reload
systemctl start leadflow
systemctl enable leadflow
```

### Step 7: Configure Nginx (Web Server)
```bash
nano /etc/nginx/sites-available/leadflow
```
```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
        root /var/www/leadflow;
    }

    location /media/ {
        root /var/www/leadflow;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/leadflow.sock;
    }
}
```
```bash
ln -s /etc/nginx/sites-available/leadflow /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

### Step 8: Enable HTTPS (Free SSL)
```bash
apt install certbot python3-certbot-nginx -y
certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

---

## 📦 Git Commands

### Initial Setup
```bash
# Initialize git
git init

# Add all files
git add .

# First commit
git commit -m "Initial commit: LeadFlow CRM setup"

# Connect to GitHub (create repo first at github.com)
git remote add origin https://github.com/yourusername/leadflow-crm.git

# Push to GitHub
git push -u origin main
```

### Daily Workflow
```bash
# Check what changed
git status

# Stage changes
git add .

# Commit with a message
git commit -m "Add feature: quick status update"

# Push to GitHub
git push
```

### Deploy Updates to Server
```bash
# On your server:
cd /var/www/leadflow
git pull origin main
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py collectstatic --noinput
systemctl restart leadflow
```

---

## 🔑 Key Concepts Explained

**MVT Architecture**: Django uses Model-View-Template
- **Model** = Database table (Lead, UserProfile)
- **View** = Business logic (views.py)  
- **Template** = HTML files (.html)

**ORM**: Django's Object-Relational Mapper lets you write Python instead of SQL:
```python
# Instead of: SELECT * FROM leads WHERE status='new'
Lead.objects.filter(status='new')
```

**REST API**: JSON endpoints that mobile apps/external systems can use

**CSRF Protection**: Django auto-includes tokens in forms to prevent attacks

---

## 🧪 Testing the API

Use [Postman](https://postman.com) or `curl`:

```bash
# Get token
curl -X POST http://localhost:8000/api/v1/token/ \
  -d "username=admin&password=yourpassword"

# List leads
curl http://localhost:8000/api/v1/leads/ \
  -H "Authorization: Token YOUR_TOKEN_HERE"
```
