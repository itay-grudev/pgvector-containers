# vim:set ft=dockerfile:
#
# Copyright The CloudNativePG Contributors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
FROM postgres:13.11-bullseye

# Do not split the description, otherwise we will see a blank space in the labels
LABEL name="PostgreSQL Container Images" \
      vendor="The CloudNativePG Contributors" \
      version="${PG_VERSION}" \
      release="7" \
      summary="PostgreSQL Container images." \
      description="This Docker image contains PostgreSQL and Barman Cloud based on Postgres 13.11-bullseye."

LABEL org.opencontainers.image.description="This Docker image contains PostgreSQL and Barman Cloud based on Postgres 13.11-bullseye."

COPY requirements.txt /

# Install pgvector
ARG PGVECTOR_VERSION=0.4.4
ARG PGVECTOR_REVISION=1982121694f7b4eeb206ee6d8dcd03b8b1316f13

RUN apt-get update && \
	apt-get install -y --no-install-recommends git ca-certificates build-essential postgresql-server-dev-${PG_MAJOR} && \
	git clone --branch v${PGVECTOR_VERSION} --depth 1 https://github.com/pgvector/pgvector.git /tmp/pgvector && \
	cd /tmp/pgvector && \
	git reset --hard ${PGVECTOR_REVISION} && \
	make clean && \
	make OPTFLAGS="" && \
	make install && \
	mkdir /usr/share/doc/pgvector && \
	cp LICENSE README.md /usr/share/doc/pgvector && \
	rm -r /tmp/pgvector && \
	apt-get remove -y git ca-certificates build-essential postgresql-server-dev-${PG_MAJOR} && \
	apt-get autoremove -y && \
	rm -rf /var/lib/apt/lists/*

# Install additional extensions
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		"postgresql-${PG_MAJOR}-pgaudit" \
		"postgresql-${PG_MAJOR}-pg-failover-slots" \
	; \
	rm -fr /tmp/* ; \
	rm -rf /var/lib/apt/lists/*;

# Install barman-cloud
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		python3-pip \
		python3-psycopg2 \
		python3-setuptools \
	; \
	pip3 install --upgrade pip; \
# TODO: Remove --no-deps once https://github.com/pypa/pip/issues/9644 is solved
	pip3 install --no-deps -r requirements.txt; \
	rm -rf /var/lib/apt/lists/*;

# Change the uid of postgres to 26
RUN usermod -u 26 postgres
USER 26
