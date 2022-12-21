FROM debian:bullseye-slim
MAINTAINER Odoo S.A. <info@odoo.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        libssl-dev \
        node-less \
        npm \
        python3-num2words \
        python3-pdfminer \
        python3-pip \
        python3-phonenumbers \
        python3-pyldap \
        python3-qrcode \
        python3-renderpm \
        python3-setuptools \
        python3-slugify \
        python3-vobject \
        python3-watchdog \
        python3-xlrd \
        python3-xlwt \
        xz-utils \
    && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.buster_amd64.deb \
    && echo 'ea8277df4297afc507c61122f3c349af142f31e5 wkhtmltox.deb' | sha1sum -c - \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ bullseye-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update  \
    && apt-get install --no-install-recommends -y postgresql-client \
    && rm -f /etc/apt/sources.list.d/pgdg.list \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

# Install Odoo
ENV ODOO_VERSION 15.0
ARG ODOO_RELEASE=20221216
ARG ODOO_SHA=db18fade18d691716ccb67ddbd583e288d4762be
RUN curl -o odoo.deb -sSL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
    && echo "${ODOO_SHA} odoo.deb" | sha1sum -c - \
    && apt-get update \
    && apt-get -y install --no-install-recommends ./odoo.deb \
    && rm -rf /var/lib/apt/lists/* odoo.deb

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /etc/odoo/odoo.conf \
    && mkdir -p /mnt/extra-addons \
    && mkdir -p /mnt/extra-addons/oca \
    && chown -R odoo /mnt/extra-addons\
    && chown -R odoo /mnt/extra-addons/oca

# Con lo siguiente, cambiamos al usuario odoo y el path donde ejecuta los
# comandos que se indiquen después.
# Odoo Community Association repositories
WORKDIR /mnt/extra-addons/oca
RUN git clone --branch 15.0 --depth 1 https://github.com/oca/account-analytic.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-budgeting.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-closing.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-consolidation.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-invoice-reporting.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-invoicing.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-financial-reporting.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-financial-tools.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-fiscal-rule.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/account-payment.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/bank-payment.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/bank-statement-import.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/bank-statement-reconcile.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/commission.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/community-data-files.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/connector-telephony.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/contract.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/crm.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/data-protection.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/e-commerce.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/event.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/hr.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/hr-timesheet.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/intrastat.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/knowledge.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/l10n-spain.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/management-system.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/manufacture.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/manufacture-reporting.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/margin-analysis.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/operating-unit.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/partner-contact.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/pos.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/product-attribute.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/product-variant.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/project.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/project-reporting.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/purchase-reporting.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/purchase-workflow.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/reporting-engine.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/rma.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/sale-financial.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/sale-reporting.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/sale-workflow.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/server-tools.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/server-ux.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/social.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/stock-logistics-barcode.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/stock-logistics-tracking.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/stock-logistics-transport.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/stock-logistics-warehouse.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/stock-logistics-workflow.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/web.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/website.git; \
    git clone --branch 15.0 --depth 1 https://github.com/oca/website-cms.git;


VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
