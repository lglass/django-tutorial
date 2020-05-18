# django-tutorial

This repo follows the Django tutorial [Writing your first Django app](https://docs.djangoproject.com/en/3.0/intro/). I've included the installation steps for one possible setup with 

- nginx as the web server, 
- uWSGI as web server gateway interface (WSGI) and 
- Postgres as database.

Another would be e.g. Apache web server with mod_wsgi as WSGI and some database other than Postgres.

## Prerequisites

```bash
sudo apt install python3-pip
sudo apt install python3-venv
sudo apt install postgresql
sudo apt install nginx
```

### Database setup
```bash
sudo mkdir -p /usr/local/pgsql/data
sudo chown postgres /usr/local/pgsql/data
sudo -u postgres /usr/lib/postgresql/11/bin/initdb -D /usr/local/pgsql/data
sudo -u postgres /usr/lib/postgresql/11/bin/pg_ctl -D /usr/local/pgsql/data/-l logfile start &
sudo -u postgres psql
```

Replace "mysite" with your project name, "myuser" with your database user and "mypass" with your database password.
```SQL
CREATE DATABASE mysite;
CREATE USER myuser WITH ENCRYPTED PASSWORD 'mypass';
GRANT ALL PRIVILEGES ON DATABASE mysite TO myuser;
ALTER USER myuser CREATEDB;
\q
```

### Create a Django project
```bash
mkdir django-project && cd django-project
python3 -m venv django
source django/bin/activate
pip install Django
pip install psycopg2
pip install uwsgi
```

Autogenerate some code around Django project
```
django-admin.py startproject mysite .
```

Clone this repo into your django-project folder.

This results in the following structure, where the django folder holds our Python virtual environment, mysite folder is populated by Django, and polls folder contains all the steps completed in [Writing your first Django app](https://docs.djangoproject.com/en/3.0/intro/).

```
django-project/
├── django
│   └── ...
├── mysite
│   ├── asgi.py
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
├── polls
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   └── __init__.py
│   ├── models.py
│   ├── static
│   │   └── polls
│   ├── templates
│   │   └── polls
│   ├── tests.py
│   ├── urls.py
│   └── views.py
├── settings.py
└── templates
    └── admin
        └── base_site.html
```

Create database tables
```
python manage.py migrate
```

Run the app
```
python manage.py runserver $(hostname -I | awk '{print $1}'):8000
```
