#
# Docker container-provisioning file for a container running a LAMP
# stack containing the XDMoD web application.
#
# Build
#
#   $ docker build --rm --tag local/xdmod:9.5.0 .
#
# Run
#
#   $ docker run --detach --restart unless-stopped --name XDMoD-Caviness \
#     --volume "$(pwd)/ingest-queue:/var/lib/XDMoD-ingest-queue:rw" \
#     --volume "$(pwd)/database:/var/lib/mysql:rw" \
#     --env CLUSTER_NAME="<cluster_name_for_xdmod-shredder>" \
#     --env RESOURCE_LOG_FORMAT="<resource_log_format_for_xdmod-shredder" \
#     --publish 8080:8080 \
#     local/xdmod:9.5.0
#

FROM    centos:7
LABEL   version="9.5.0" \
        description="XDMoD bundle containing database and web server" \
        maintainer="renfro@tntech.edu"

# Mongo repository
COPY    mongodb-org-3.6.repo /etc/yum.repos.d

#
# Install tools, libraries:
#
RUN     yum -y update && \
        yum -y install  epel-release && \
        yum -y install  sudo wget gmp-devel cronie logrotate \
                        coreutils shadow-utils tar xz bzip2 gzip zip jq \
                        libreoffice chromium-headless librsvg2-tools openssh-clients mongodb-org-shell 

RUN     set -o pipefail && echo "root:${ROOT_PASSWORD}" | chpasswd

#
# Install Apache and PHP:
#
RUN     yum -y install  httpd php php-cli php-gd php-gmp \
                        php-pdo php-xml php-pear-MDB2 php-pear-MDB2-Driver-mysql \
                        php-mbstring php-pecl-apcu perl-Image-ExifTool

#
# Install MySQL:
#
RUN     yum -y install  mariadb-server mariadb

#
# Install NodeJS repo
#
RUN     curl -sL https://rpm.nodesource.com/setup_14.x | bash -
#
# Install XDMoD:
#
RUN     yum -y install \
        https://github.com/ubccr/xdmod/releases/download/v9.5.0/xdmod-9.5.0-1.0.el7.noarch.rpm \
        https://github.com/ubccr/xdmod-supremm/releases/download/v9.5.0/xdmod-supremm-9.5.0-1.0.el7.noarch.rpm \
        https://github.com/ubccr/supremm/releases/download/1.4.1/supremm-1.4.1-1.el7.x86_64.rpm && \
        yum -y install python-pip && \
        pip install "pymongo==3.*" --upgrade && \
        yum clean all

#
# XDMoD uses an Apache virtual host on port 8080:
#
EXPOSE  8080/tcp

#
# Ensure a mountpoint/directory exists for the data ingest pipeline:
#
RUN     mkdir --parents --mode=0755 /var/lib/XDMoD-ingest-queue/{in,out,error} /home/tntech.edu/supremm

#
# Command to execute when container is started:
#
COPY    --chown=0:0 XDMoD-start /usr/sbin/XDMoD-start
RUN     chmod 0755 /usr/sbin/XDMoD-start
CMD     /usr/sbin/XDMoD-start
