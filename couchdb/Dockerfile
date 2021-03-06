
# Licensed under the Apache License, Version 2.0 (the "License"); you may not
# use this file except in compliance with the License. You may obtain a copy of
# the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations under
# the License.

FROM debian:buster-slim

LABEL maintainer="CouchDB Developers dev@couchdb.apache.org"

# Add CouchDB user account to make sure the IDs are assigned consistently
RUN groupadd -g 5984 -r couchdb && useradd -u 5984 -d /opt/couchdb -g couchdb couchdb

# be sure GPG and apt-transport-https are available and functional
RUN set -ex; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        dirmngr \
        gnupg \
     ; \
    rm -rf /var/lib/apt/lists/*

# grab gosu for easy step-down from root and tini for signal handling and zombie reaping
# see https://github.com/apache/couchdb-docker/pull/28#discussion_r141112407
ENV GOSU_VERSION 1.11
ENV TINI_VERSION 0.18.0
RUN set -ex; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends wget; \
    rm -rf /var/lib/apt/lists/*; \
    \
    dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')"; \
    \
# install gosu
    wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-$dpkgArch"; \
    wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc"; \
    export GNUPGHOME="$(mktemp -d)"; \
    echo "disable-ipv6" >> ${GNUPGHOME}/dirmngr.conf; \
    for server in $(shuf -e pgpkeys.mit.edu \
        ha.pool.sks-keyservers.net \
        hkp://p80.pool.sks-keyservers.net:80 \
        pgp.mit.edu) ; do \
    gpg --batch --keyserver $server --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 && break || : ; \
    done; \
    gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
    rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc; \
    chmod +x /usr/local/bin/gosu; \
    gosu nobody true; \
    \
# install tini
    wget -O /usr/local/bin/tini "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini-$dpkgArch"; \
    wget -O /usr/local/bin/tini.asc "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini-$dpkgArch.asc"; \
    export GNUPGHOME="$(mktemp -d)"; \
    echo "disable-ipv6" >> ${GNUPGHOME}/dirmngr.conf; \
    for server in $(shuf -e pgpkeys.mit.edu \
        ha.pool.sks-keyservers.net \
        hkp://p80.pool.sks-keyservers.net:80 \
        pgp.mit.edu) ; do \
    gpg --batch --keyserver $server --recv-keys 595E85A6B1B4779EA4DAAEC70B588DFF0527A9B7 && break || : ; \
    done; \
    gpg --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini; \
    rm -rf "$GNUPGHOME" /usr/local/bin/tini.asc; \
    chmod +x /usr/local/bin/tini; \
    apt-get purge -y --auto-remove wget; \
    tini --version

# http://docs.couchdb.org/en/latest/install/unix.html#installing-the-apache-couchdb-packages
ENV GPG_COUCH_KEY \
# gpg: key D401AB61: public key "Bintray (by JFrog) <bintray@bintray.com> imported
    8756C4F765C9AC3CB6B85D62379CE192D401AB61
RUN set -xe; \
    export GNUPGHOME="$(mktemp -d)"; \
    echo "disable-ipv6" >> ${GNUPGHOME}/dirmngr.conf; \
    for server in $(shuf -e pgpkeys.mit.edu \
        ha.pool.sks-keyservers.net \
        hkp://p80.pool.sks-keyservers.net:80 \
        pgp.mit.edu) ; do \
        gpg --batch --keyserver $server --recv-keys $GPG_COUCH_KEY && break || : ; \
    done; \
    gpg --batch --export $GPG_COUCH_KEY > /etc/apt/trusted.gpg.d/couchdb.gpg; \
    command -v gpgconf && gpgconf --kill all || :; \
    rm -rf "$GNUPGHOME"; \
    apt-key list

ENV COUCHDB_VERSION 3.1.1

RUN echo "deb https://apache.bintray.com/couchdb-deb buster main" > /etc/apt/sources.list.d/couchdb.list

# https://github.com/apache/couchdb-pkg/blob/master/debian/README.Debian
RUN set -xe; \
    apt-get update; \
    \
    echo "couchdb couchdb/mode select none" | debconf-set-selections; \
# we DO want recommends this time
    DEBIAN_FRONTEND=noninteractive apt-get install -y --allow-downgrades --allow-remove-essential --allow-change-held-packages \
            couchdb="$COUCHDB_VERSION"~buster \
    ; \
# Undo symlinks to /var/log and /var/lib
    rmdir /var/lib/couchdb /var/log/couchdb; \
    rm /opt/couchdb/data /opt/couchdb/var/log; \
    mkdir -p /opt/couchdb/data /opt/couchdb/var/log; \
    chown couchdb:couchdb /opt/couchdb/data /opt/couchdb/var/log; \
    chmod 777 /opt/couchdb/data /opt/couchdb/var/log; \
# Remove file that sets logging to a file
    rm /opt/couchdb/etc/default.d/10-filelog.ini; \
# Check we own everything in /opt/couchdb. Matches the command in dockerfile_entrypoint.sh
    find /opt/couchdb \! \( -user couchdb -group couchdb \) -exec chown -f couchdb:couchdb '{}' +; \
# Setup directories and permissions for config. Technically these could be 555 and 444 respectively
# but we keep them as 755 and 644 for consistency with CouchDB defaults and the dockerfile_entrypoint.sh.
    find /opt/couchdb/etc -type d ! -perm 0755 -exec chmod -f 0755 '{}' +; \
    find /opt/couchdb/etc -type f ! -perm 0644 -exec chmod -f 0644 '{}' +; \
# only local.d needs to be writable for the docker_entrypoint.sh
    chmod -f 0777 /opt/couchdb/etc/local.d; \
# apt clean-up
    rm -rf /var/lib/apt/lists/*;

# Add configuration
COPY --chown=couchdb:couchdb 10-docker-default.ini /opt/couchdb/etc/default.d/
COPY --chown=couchdb:couchdb vm.args /opt/couchdb/etc/

COPY docker-entrypoint.sh /usr/local/bin
RUN ln -s usr/local/bin/docker-entrypoint.sh /docker-entrypoint.sh # backwards compat
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]

VOLUME /opt/couchdb/data

# 5984: Main CouchDB endpoint
# 4369: Erlang portmap daemon (epmd)
# 9100: CouchDB cluster communication port
EXPOSE 5984 4369 9100
CMD ["/opt/couchdb/bin/couchdb"]