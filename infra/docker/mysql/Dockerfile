FROM mysql/mysql-server:8.0
LABEL maintainer="yaand"

ENV TZ=UTC

COPY ./infra/docker/mysql/my.cnf /etc/my.cnf

RUN chmod 444 /etc/my.cnf
