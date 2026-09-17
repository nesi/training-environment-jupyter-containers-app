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

# the workshop example files, copied into the user's home directory at startup
# by template/script.sh.erb
# the revision is pinned in workshop-version.txt, which the github workflow
# reads as well so that the prebuilt images below match these definition files
COPY workshop-version.txt /tmp/workshop-version.txt
RUN git clone https://github.com/nesi/reannz-containers-workshop.git /tmp/workshop \
    && git -C /tmp/workshop checkout "$(cat /tmp/workshop-version.txt)" \
    && mkdir -p /opt/containers-workshop \
    && cp -r /tmp/workshop/examples /opt/containers-workshop/examples \
    && rm -rf /opt/containers-workshop/examples/09_running_containers_with_MPI \
    && rm -rf /tmp/workshop /tmp/workshop-version.txt

# the chapter 2 containers, built on the runner by .github/workflows/build_container.yml
# following https://nesi.github.io/reannz-containers-workshop/setup-containers/
# so that learners do not have to wait for them at the start of the workshop
# (this directory is empty when building the image by hand, which is fine - the
#  definition files are all still there to build from)
COPY prebuilt/ /tmp/prebuilt/
RUN find /tmp/prebuilt -name '*.sif' -exec mv {} /opt/containers-workshop/examples/02_basics_of_containers/ \; \
    && rm -rf /tmp/prebuilt

COPY workshop-readme.md /opt/containers-workshop/README.md
RUN chown -R root:root /opt/containers-workshop \
    && chmod -R o+rX /opt/containers-workshop

# install jupyterlab
# `notebook` is installed alongside it for the standalone terminal page at
# /terminals/1, which the "Terminal" button in view.html.erb links to
RUN pip3 --no-cache-dir install jupyterlab notebook
