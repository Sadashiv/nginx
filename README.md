Deploy the Django2.2 through uwsgi and nginx
============================================
Referance: https://www.digitalocean.com/community/tutorials/how-to-set-up-uwsgi-and-nginx-to-serve-python-apps-on-centos-7

sudo yum instal nginx -y
Nginx web server as a reverse proxy to the application server to provide more robust connection handling.

WSGI: A Python spec that defines a standard interface for communication between an application or framework
and an application/web server. This was created in order to simplify and standardize communication between
these components for consistency and interchangeability. This basically defines an API interface that can be used over other protocols.

uWSGI: An application server container that aims to provide a full stack for developing and deploying
web applications and services. The main component is an application server that can handle apps of different
languages. It communicates with the application using the methods defined by the WSGI spec, and with other
web servers over a variety of other protocols. This is the piece that translates requests from a conventional
web server into a format that the application can process.

uwsgi: A fast, binary protocol implemented by the uWSGI server to communicate with a more full-featured web server.
This is a wire protocol, not a transport protocol. It is the preferred way to speak to web servers that are proxying requests to uWSGI.

Assuming python and uwsgi installed

We want to mark the initial uwsgi process as a master and then spawn a number of worker processes. We will start with five workers
[uwsgi]
master = True
processes = 5

run uwsgi with particular user
uid = user
socket = /run/uwsgi/myapp.sock
chown-socket = user:nginx
chmod-socket = 660
vacuum = true
We’ll also add the vacuum option, which will remove the socket when the process stops:

die-on-term = true
creating a systemd file to start our application at boot. Systemd and uWSGI have different ideas about what
the SIGTERM signal should do to an application. 

Creating the systemd unti file to manage the app

sudo nano /etc/systemd/system/uwsgi.service

[Unit]
Description=uWSGI instance to serve myapp

[Service]
ExecStartPre=-/usr/bin/bash -c 'mkdir -p /run/uwsgi; chown user:nginx /run/uwsgi'
ExecStart=/usr/bin/bash -c 'cd /home/user/myapp; source myappenv/bin/activate; uwsgi --ini myapp.ini'

[Install]
WantedBy=multi-user.target

sudo systemctl start uwsgi
systemctl status uwsgi
sudo systemctl enable uwsgi

Nginx has the ability to proxy using the uwsgi protocol for communicating with uWSGI.
This is a faster protocol than HTTP and will perform better.

server {
    listen 80;
    server_name server_domain_or_IP;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/run/uwsgi/myapp.sock;
    }
}

