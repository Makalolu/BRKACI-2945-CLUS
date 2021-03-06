# VERSION 1.0
FROM ubuntu:16.04

MAINTAINER Andy Gossett, agossett@cisco.com

ENV GIT_SSL_NO_VERIFY "true"
ARG APP_MODE=0
ARG DEV_MODE=0
ENV APP_MODE $APP_MODE
ENV DEV_MODE $DEV_MODE
ENV APP_DIR "/home/app"
ENV DATA_DIR "$APP_DIR/data"
ENV LOG_DIR "$APP_DIR/log"
ENV BACKEND_DIR "$APP_DIR/src/Service"

RUN apt-get update && \
    apt-get -y install apt-utils && \
    apt-get -y install \
        cron \
        curl \
        git \
        libffi-dev \
        libssl-dev \
        logrotate \
        ntp \
        python \
        python-dev \
        python-pip \
        rsyslog \
        vim

# install and configure apache
RUN apt-get -y install \
        apache2 \
        libapache2-mod-wsgi
COPY ./conf/apache2-000-default.conf /etc/apache2/sites-available/000-default.conf
COPY ./conf/apache2-default-ssl.conf /etc/apache2/sites-available/default-ssl.conf
RUN mkdir -p $LOG_DIR/apache2 && \
    chown www-data:www-data $LOG_DIR/apache2 && \
    /usr/sbin/a2enmod ssl && \
    /usr/sbin/a2dissite 000-default && \
    /usr/sbin/a2dissite default-ssl && \
    /usr/sbin/a2ensite 000-default && \
    /usr/sbin/a2ensite default-ssl

# install and configure mongodb 3.6
RUN apt-get install -y apt-transport-https && \
        apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2930ADAE8CAF5059EE73BB4B58712A2291FA4AD5 && \
        echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.6 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-3.6.list && \
        apt-get update && \
        apt-get install -y mongodb-org
COPY ./conf/mongodb-init.d /etc/init.d/mongodb
COPY ./conf/mongod.conf /etc/mongod.conf
RUN chmod 755 /etc/init.d/mongodb && \
    mkdir -p $LOG_DIR/mongo && \
    mkdir -p $DATA_DIR/db && \
    chown mongodb:mongodb $LOG_DIR/mongo && \
    chown mongodb:mongodb $DATA_DIR/db


# setup logrotate for minute granularity and add custom logging directories
RUN mkdir -p /etc/cron.minute && \
    chmod 755 /etc/cron.minute
COPY ./conf/crontab /etc/crontab
COPY ./conf/cron.minute.logrotate /etc/cron.minute/logrotate
COPY ./conf/logrotate.d.app /etc/logrotate.d/app

# upgrade pip
RUN pip install --upgrade pip

# copy python requirements and install
# pull backend source and install requirements
RUN mkdir -p $BACKEND_DIR
COPY ./requirements.txt $BACKEND_DIR/
RUN pip install -r $BACKEND_DIR/requirements.txt
WORKDIR $BACKEND_DIR
