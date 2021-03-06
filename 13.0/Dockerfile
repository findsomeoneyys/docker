FROM ubuntu:18.04
MAINTAINER Odoo S.A. <info@odoo.com>

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -f -y --no-install-recommends \
            ca-certificates \
            wget \
            curl \
            git \
            dirmngr \
            fonts-noto-cjk \
            gnupg \
            node-less \
            nodejs-dev \
            npm \
            gcc \
            libpq-dev \
            python3 \
            python3-dev \
            python3-pip \
            python3-wheel \
            build-essential \
            libxml2-dev \
            libxslt1-dev \
            zlib1g-dev \
            libffi-dev \
            libldap2-dev \
            libsasl2-dev \
            python3-renderpm \
            python3-setuptools \
            python3-slugify \
            python3-watchdog \
            xz-utils \
        && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb \
        && apt-get install -f -y --no-install-recommends ./wkhtmltox.deb \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# install latest postgresql-client
RUN set -x; \
        echo 'deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
        && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
        && apt-get update  \
        && apt-get install -f -y postgresql-client

# Install rtlcss (on Debian buster)
RUN set -x; \
    npm install -g rtlcss

RUN pip3 install num2words xlwt xlrd phonenumbers qrcode wheel
# Install Odoo
ENV ODOO_VERSION 13.0
ARG ODOO_RELEASE=20200407
RUN wget http://nightly.odoo.com/${ODOO_VERSION}/nightly/src/odoo_${ODOO_VERSION}.${ODOO_RELEASE}.tar.gz \
    && tar -zxf odoo_${ODOO_VERSION}.${ODOO_RELEASE}.tar.gz
RUN cd odoo-${ODOO_VERSION}.post${ODOO_RELEASE} && pip3 install -r requirements.txt && pip3 install .
RUN cd ${GITHUB_WORKSPACE} && ls | grep 'odoo.${ODOO_VERSION}.*' | xargs rm -rf
        
# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN useradd -ms /bin/bash  odoo
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
