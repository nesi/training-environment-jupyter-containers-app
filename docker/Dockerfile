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
        dnsutils \
        fakeroot \
        file \
        git \
        jq \
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

# install kubectl, required for running on the k8s cluster
ARG KUBECTL_VERSION=v1.28.5
RUN curl -LO https://dl.k8s.io/release/$KUBECTL_VERSION/bin/linux/amd64/kubectl \
    && mv kubectl /usr/local/bin/kubectl \
    && chmod +x /usr/local/bin/kubectl

# copy in ood k8s utils
# template/before.sh.erb sources find_host_port, save_passwd_as_secret and
# create_salt_and_sha1 from here to set up the connection details, so the app
# will not connect without them
ARG UTILS_HASH=6298fb01f7a7c66a8454e3f0fd74437a32491423
RUN git clone https://github.com/nesi/training-environment-k8s-utils.git /opt/ood-k8s-utils \
    && cd /opt/ood-k8s-utils \
    && git checkout $UTILS_HASH \
    && chmod +x /opt/ood-k8s-utils/files/* \
    && mv /opt/ood-k8s-utils/files/* /bin/ \
    && rm -rf /opt/ood-k8s-utils

# make a dummy module command to avoid warnings from ondemand job_script_content.sh
RUN echo "#!/bin/bash" > /bin/module \
    && chmod +x /bin/module

# install code-server, used by the code-server app which runs this same image
ARG CODE_RELEASE=4.137.0
RUN mkdir -p /app/code-server \
    && curl -sL -o /tmp/code-server.tar.gz \
        "https://github.com/coder/code-server/releases/download/v${CODE_RELEASE}/code-server-${CODE_RELEASE}-linux-amd64.tar.gz" \
    && tar xf /tmp/code-server.tar.gz -C /app/code-server --strip-components=1 \
    && rm -f /tmp/code-server.tar.gz \
    && ln -s /app/code-server/bin/code-server /usr/local/bin/code-server

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
