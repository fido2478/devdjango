# devdjango
install django, gunicorn, nignx on ubuntu 16.04

sudo apt-get update
sudo apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx

sudo -u postgres psql

- paste this code in postgres console:
CREATE DATABASE django_project;
CREATE USER username WITH PASSWORD 'xxxx';
ALTER ROLE username SET client_encoding TO 'utf8';
ALTER ROLE username SET default_transaction_isolation TO 'read committed';
ALTER ROLE username SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE django_project TO username;
\q

pip3 install django gunicorn python3-psycopg2
django-admin.py startproject devdjango
nano devdjango/settings.py

- add your Amazon Lightsail IP to the ALLOWED_HOSTS variable, example:

ALLOWED_HOSTS = ['*']

- add database configuration:

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'devdjango',
        'USER': 'devdjango',
        'PASSWORD': 'xxxx',
        'HOST': 'localhost',
    }
}

- add static root to the end of the settings:
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')

- save the settings.py file

cd ~/devdjango
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py createsuperuser
python3 manage.py collectstatic

sudo ufw allow 8001
./manage.py runserver 0.0.0.0:8001
- ctrl + c

cd ~/devdjango
gunicorn --bind 0.0.0.0:8001 devdjango.wsgi:application
- ctrl + c

sudo nano /etc/systemd/system/gunicorn.service

- paste inside gunicorn.service:

[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/devdjango
ExecStart=/usr/local/bin/gunicorn --bind 192.168.0.11:8001 devdjango.wsgi:application

[Install]
WantedBy=multi-user.target


sudo systemctl start gunicorn
sudo systemctl enable gunicorn
sudo nano /etc/nginx/sites-available/devdjango
- copy inside:
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /var/www/devdjango;
    }
    location / {
        include proxy_params;
        proxy_pass http://192.168.0.11:8001;
    }
}

- save the file
- remove or rename default
mv /etc/nginx/sites-available/default /etc/nginx/sites-available/default.df

sudo ln -s /etc/nginx/sites-available/devdjango /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'
