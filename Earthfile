FROM ubuntu:20.10

apache-php:
    FROM DOCKERFILE ./apache-php
    SAVE IMAGE apache-php

couchdb:
    FROM DOCKERFILE ./couchdb
    SAVE IMAGE couchdb

mysql-server:
    FROM DOCKERFILE ./mysql-server
    SAVE IMAGE mysql-server

nodejs-mongo-mongoose:
    FROM DOCKERFILE ./nodejs-mongo-mongoose
    SAVE IMAGE nodejs-mongo-mongoose

all:
    BUILD +apache-php
    BUILD +couchdb
    BUILD +mysql-server
    BUILD +nodejs-mongo-mongoose