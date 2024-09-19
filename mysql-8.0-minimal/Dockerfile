FROM mysql:8.0-oracle
MAINTAINER groonga

ENV groonga_version=14.0.7 \
    mroonga_version=14.04

RUN mkdir -p /etc/mysql/mysql.conf.d && \
    touch /etc/mysql/mysql.conf.d/default-auth-override.cnf && \
    touch /etc/mysql/mysql.conf.d/lowercase-table-names.cnf && \
    microdnf update -y && \
    os_version=$(. /etc/os-release && echo $VERSION_ID | grep -oE '^[0-9]+') && \
    microdnf install -y \
      https://packages.groonga.org/almalinux/${os_version}/groonga-release-latest.noarch.rpm && \
    { \
    	echo '[mysql8.0-server-minimal]'; \
    	echo 'name=MySQL 8.0 Server Minimal'; \
    	echo 'enabled=1'; \
    	echo 'baseurl=https://repo.mysql.com/yum/mysql-8.0-community/docker/el/9/$basearch/'; \
    	echo 'gpgcheck=1'; \
    	echo 'gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql'; \
    	echo 'module_hotfixes=true'; \
    } | tee /etc/yum.repos.d/mysql-community-minimal.repo && \
    microdnf update -y && \
    microdnf install -y \
      groonga-bin=${groonga_version}-1 \
      groonga-normalizer-mysql \
      groonga-tokenizer-mecab=${groonga_version}-1 \
      mysql-community-8.0-mroonga=${mroonga_version}-1 && \
    sed \
      -i'' \
      -e 's,@MRN_DATA_DIR@,/usr/share/mroonga,g' \
      /usr/share/mroonga/install.sql && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir -p /docker-entrypoint-mroonga-initdb.d && \
    cp /usr/share/mroonga/install.sql \
       /docker-entrypoint-mroonga-initdb.d/00-install.sql && \
    sed \
      -i'' \
      -e 's,docker_process_init_files /,docker_process_init_files /docker-entrypoint-mroonga-initdb.d/* /,g' \
      /usr/local/bin/docker-entrypoint.sh

# mysql:8.0.30 image has DB in /var/lib/mysql/.
# This clears /var/lib/mysql/ to ensure creating a new DB on "docker run".
VOLUME ["/var/lib/mysql"]
