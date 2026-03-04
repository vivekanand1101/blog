---
title: "Django with uwsgi and nginx on fedora"
date: 2017-08-08
categories: 
  - "tech"
---

Today, i deployed a [django](https://github.com/django/django) project using [uwsgi](https://github.com/unbit/uwsgi) and [nginx](https://github.com/nginx/nginx) on a fedora 26 instance on _AWS_. I will talk about the same here.

I had used [gunicorn](https://github.com/benoitc/gunicorn) in the past but never uwsgi. Getting started with gunicorn was a little bit easier for me than uwsgi primarily because i didn't know i had to install _uwsgi-plugin-python_ package to use uwsgi with django. This took me a while because, there were no errors. There was a "no app could be loaded" problem but, on internet most of this kind of error is for [flask](https://github.com/pallets/flask). Flask exposes it's application object as app and uwsgi looks to load application which it fails to find.

The steps are:

1. Install dependencies and the project itself:
    - _sudo dnf install uwsgi uwsgi-plugin-python nginx_

1. Create a configuration file for uwsgi: _uwsgi.ini_
2. Change nginx config to pass on to uwsgi for incoming requests on _/ethcld/_ (the mount point that i used)

 

Here is my uwsgi file:

_\[uwsgi\]_ _chdir = /home/fedora/ethcld_ _plugin = python_ _\# Django's wsgi file_ _\# module = ethereal.wsgi:application_ _mount = /ethcld=ethereal.wsgi:application_ _manage-script-name = True_ _master = True_ _\# maximum number of worker processes_ _processes = 4_ _\# Threads per process_ _threads = 2_ _socket = 127.0.0.1:8001_ _\# clear environment on exit_ _vacuum = true_

uwsgi asks for the directory of the project. In my case, it was _/home/fedora/ethcld/_. _Mount_ is optional, if you want to run the application under some namespace, you will have to use mount. Also, if mount is getting used, you should not need _module_.

Manage Script name (manage-script-name)is important while mounting otherwise you will get a bunch of 404s. Usually, the request comes for something like: _GET /username/_  but, without _manage-script-name_ option, nginx will not map _/username/_ to _/ethcld/username._

For socket, i could have used unix socket instead of port one. But, somehow i settled for port.

On the nginx side, i did the following changes:

_http {_

       _include /etc/nginx/mime.types;_

       _server {_

                _location ^~/static/ {_                        _autoindex on;_                        _alias /opt/ethcld/static/;_                 _}_

                _location ^~ /ethcld {_                         _include uwsgi\_params;_                         _uwsgi\_pass 127.0.0.1:8001;_                 _}_

         _}_

_}_

Nginx supports uwsgi protocol by default, so options are already there, we just need to call them. For serving static files, for django, it is recommended that we do:

- _python manage.py collectstatic_

This will copy (by default) all the static files in the django app to the location specified in _STATIC\_ROOT_ in settings. From there we can serve the static files. There are possible optimizations that can be done like gzip. But, i did this for testing only.

You need to include _mime.types_, otherwise browsers will keep rejecting files. By default, the mime type is _'text/plain'_.
