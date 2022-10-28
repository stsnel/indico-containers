FROM ubuntu:22.04

ENV INDICO_VIRTUALENV="/opt/indico/.venv" INDICO_CONFIG="/opt/indico/etc/indico.conf"

ARG KRB_CACHE_DIR='/var/run/keytab'
ARG pip="${INDICO_VIRTUALENV}/bin/pip"
ARG tag="v3.2-testar"
ENV COVERAGE_DIR=/coverage

ENV KRB_CACHE_DIR ${KRB_CACHE_DIR}
ENV DEBIAN_FRONTEND noninteractive

USER root

RUN set -ex && \
    apt-get update && \
    apt-get -y install software-properties-common

RUN add-apt-repository ppa:deadsnakes/ppa

RUN set -ex && \
    apt-get update && \
    apt-get -y install python3.9 texlive-xetex libpq-dev postgresql-client vim less python3-virtualenv gcc gettext && \
    apt-get -y install -y libjpeg-turbo8-dev zlib1g-dev git npm sqlite3 python3.9 python3.9-dev python3.9-distutils && \
    apt-get clean

RUN ["/bin/bash", "-c", "mkdir -p --mode=775 /opt/indico/{etc,tmp,log,cache,archive,src} $COVERAGE_DIR"]

RUN  if [ "$tag" = "latest" ]; \
     then git clone --branch "v3.2-testar" https://github.com/stsnel/indico.git /opt/indico/src ;\
     else \
     git clone --branch "$tag" https://github.com/stsnel/indico.git /opt/indico/src ;\
     fi

RUN virtualenv --python /usr/bin/python3.9 ${INDICO_VIRTUALENV}

RUN ${pip} install --upgrade pip setuptools && \
    ${pip} install uwsgi raven flask_url_map_serializer && \
    ${pip} install git+https://github.com/stsnel/coveragepy.git@6.2-local && \
    ${pip} install git+https://github.com/stsnel/python-string-extractor.git@main

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
VOLUME /coverage

# OpenShift runs containers using an arbitrarily assigned user ID for security reasons
# This user is always in the root group so it is needed to grant privileges to group 0.
RUN chgrp -R 0 /opt/indico

COPY run_indico.sh run_celery.sh set_user.sh /opt/indico/
RUN chmod 755 /opt/indico/*.sh
RUN chmod g=u /etc/passwd

# Add extract strings scripts
COPY extract-strings.py /tmp
RUN install -m 0755 /tmp/extract-strings.py /usr/local/bin/extract-strings.py

ENV USE_PROXY ${use_proxy}
