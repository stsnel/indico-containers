FROM ubuntu:22.04

ENV INDICO_VIRTUALENV="/opt/indico/.venv" INDICO_CONFIG="/opt/indico/etc/indico.conf"

ARG KRB_CACHE_DIR='/var/run/keytab'
ARG pip="${INDICO_VIRTUALENV}/bin/pip"
ARG tag="v3.2-testar-plain"

ENV KRB_CACHE_DIR ${KRB_CACHE_DIR}
ENV DEBIAN_FRONTEND noninteractive

USER root

RUN set -ex && \
    apt-get update && \
    apt-get -y install texlive-xetex libpq-dev postgresql-client vim less python3-virtualenv gcc gettext && \
    apt-get -y install -y libjpeg-turbo8-dev zlib1g-dev git npm && \
    apt-get clean

RUN ["/bin/bash", "-c", "mkdir -p --mode=775 /opt/indico/{etc,tmp,log,cache,archive,src}"]

RUN  git clone --branch "$tag" https://github.com/stsnel/indico.git /opt/indico/src

RUN virtualenv ${INDICO_VIRTUALENV}

RUN ${pip} install --upgrade pip setuptools && \
    ${pip} install uwsgi raven flask_url_map_serializer

RUN apt-get install -y libldap2-dev

RUN ${pip} install -e /opt/indico/src
WORKDIR /opt/indico/src

RUN npm ci

RUN . ${INDICO_VIRTUALENV}/bin/activate && exec python3 /opt/indico/src/bin/maintenance/build-assets.py indico

RUN ${INDICO_VIRTUALENV}/bin/indico setup create-symlinks /opt/indico
RUN ${INDICO_VIRTUALENV}/bin/indico setup create-logging-config /opt/indico/etc

COPY indico.conf logging.yaml /opt/indico/etc/

EXPOSE 59999

COPY uwsgi.ini /etc/uwsgi.ini

VOLUME ${KRB_CACHE_DIR}

# OpenShift runs containers using an arbitrarily assigned user ID for security reasons
# This user is always in the root group so it is needed to grant privileges to group 0.
RUN chgrp -R 0 /opt/indico

COPY run_indico.sh run_celery.sh set_user.sh /opt/indico/
RUN chmod 755 /opt/indico/*.sh
RUN chmod g=u /etc/passwd

ENV USE_PROXY ${use_proxy}
