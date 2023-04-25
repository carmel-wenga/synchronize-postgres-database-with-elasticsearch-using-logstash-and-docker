# base image
FROM postgres:13-alpine

# run init.sql on init
COPY init.sql /docker-entrypoint-initdb.d
