---
title: "Setting up postgres for python development"
date: 2017-12-02
categories: 
  - "tech"
---

Postgresql is my database of choice mainly because, almost (if not all) web applications in fedora infrastructure use postgres and i have been using the same. But, every time i use a fresh system, i face some issues with getting started. Almost all of them are for client authentication. This post is here in case it happens the same in future.

1. Install the dependencies:
    1. sudo dnf install postgresql postgresql-devel postgresql-server postgresql-contrib
2. Create a db user other than postgres:
    1. sudo -i
    2. su - postgres
    3. initdb
    4. Start the database using command: pg\_ctl -D /var/lib/pgsql/data -l logfile start
    5. psql -U postgres
    6. create database pagure;
    7. create user pagure with password 'pagure';
    8. grant all privileges on database pagure to pagure;
    9. show hba\_file;
    10. Go to that file and change the auth mode to trust;
3. If runs into: unregistered authentication agent for unix-process [https://techglimpse.com/solution-polkitd-postgresql-start-error/](https://techglimpse.com/solution-polkitd-postgresql-start-error/)
