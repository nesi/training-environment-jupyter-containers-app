FROM ubuntu:22.04

# run unminimise, e.g. for man pages to work
RUN yes | unminimize

# install system dependencies
# TODO: move ldap-utils, libnss-ldapd, libpam-ldapd, nscd, nslcd to base image??
RUN echo 'tzdata tzdata/Areas select Pacific' | debconf-set-selections \
    && echo 'tzdata tzdata/Zones/Pacific select Auckland' | debconf-set-selections \
    && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
        bash-completion \
        ca-certificates \
        curl \
        fakeroot \
        file \
        git \
        ldap-utils \
        libnss-ldapd \
        libpam-ldapd \
        less \
        locales \
        man-db \
        nano \
        nodejs \
        nscd \
        nslcd \
        python-is-python3 \
        python3 \
        python3-pip \
        rsync \
        software-properties-common \
        squashfs-tools \
        tree \
        tzdata \
        uidmap \
        unzip \
        vim \
        wget \
        zip \
    && rm -rf /var/lib/apt/lists/* \
    && localedef -i en_NZ -c -f UTF-8 -A /usr/share/locale/locale.alias en_NZ.UTF-8

ENV LANG en_NZ.utf8

# install apptainer
# apptainer-suid is what makes `apptainer build --fakeroot` work for the
# training users, it also requires `enable_privileged_pods: true` in the
# training environment config and the subuid/subgid mounts in submit.yml.erb
RUN add-apt-repository -y ppa:apptainer/ppa \
    && apt update \
    && apt install -y apptainer-suid \
    && rm -rf /var/lib/apt/lists/*

# example definition files, copied into the user's home directory at startup by
# template/script.sh.erb
COPY examples /opt/apptainer-examples
RUN chown -R root:root /opt/apptainer-examples \
    && chmod -R o+rX /opt/apptainer-examples

# install jupyterlab
# `notebook` is installed alongside it for the standalone terminal page at
# /terminals/1, which the "Terminal" button in view.html.erb links to
RUN pip3 --no-cache-dir install jupyterlab notebook
