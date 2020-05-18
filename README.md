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

```
sudo /etc/init.d/nginx start
```

### Database setup
```bash
sudo mkdir -p /usr/local/pgsql/data
sudo chown postgres /usr/local/pgsql/data
sudo -u postgres /usr/lib/postgresql/11/bin/initdb -D /usr/local/pgsql/data
sudo -u postgres /usr/lib/postgresql/11/bin/pg_ctl -D /usr/local/pgsql/data/-l logfile start &
sudo -u postgres psql
```

Replace "mysite" with actual project name, "myuser" with actual database user and "mypass" with acutal database password.
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
pip install python-dotenv
```

Autogenerate some code around Django project
```
django-admin.py startproject mysite .
```

Clone this repo into the django-project folder.

Create .env file for database connection configuration

```
DATABASE_NAME=mysite
DATABASE_USER=myuser
DATABASE_PASS=mypass
DATABASE_HOST=myhost
```

To make the Django project see your dotenv configuration, first load it at the top of wsgi.py and manage.py

```
from dotenv import load_dotenv, find_dotenv
load_dotenv(find_dotenv())
```

then adjust settings.py in mysite directory

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv("DATABASE_NAME"),
        'USER': os.getenv("DATABASE_USER"),
        'PASSWORD': os.getenv("DATABASE_PASS"),
        'HOST': os.getenv("DATABASE_HOST"),
        'PORT': '5432',
    }
}
```

The project should now have the following structure, where the Django folder holds our Python virtual environment, the mysite folder is populated by Django, and the polls folder contains the result of the completed tutorial [Writing your first Django app](https://docs.djangoproject.com/en/3.0/intro/).

```
django-project/
├── django
│   └── ...
├── .env
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

Run the app and serve it on the local network.
```
python manage.py runserver $(hostname -I | awk '{print $1}'):8000
```

### Use uwsgi

Configure nginx to talk to uwsgi. For that, update the file /etc/nginx/sites-enabled/mysite_nginx.conf

```
# the upstream component nginx needs to connect to
upstream django {
    server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    #server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /path/to/your/mysite/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /etc/nginx/uwsgi_params;
    }
}
```

Add the following to mysite/settings.py

```
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

and run

```
python manage.py collectstatic
```

Restart nginx for changes to take effect

```
sudo /etc/init.d/nginx restart
```

Run 

```
uwsgi --socket mysite.sock --module mysite.wsgi --chmod-socket=666
```

and check that everything is working, e.g. by visiting [localhost:8000](localhost:8000).
