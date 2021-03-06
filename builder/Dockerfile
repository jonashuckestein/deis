FROM deis/base:latest
MAINTAINER OpDemand <info@opdemand.com>

# install docker-in-docker
RUN echo "deb http://get.docker.io/ubuntu docker main" > /etc/apt/sources.list.d/docker.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
# install builder, docker, and hook dependencies
RUN apt-get update && apt-get install -yq \
    openssh-server git \
    aufs-tools iptables lxc \
    curl \
    lxc-docker-1.1.2

# configure ssh server
RUN rm /etc/ssh/ssh_host_*
RUN dpkg-reconfigure openssh-server
RUN mkdir -p /var/run/sshd

# install recent pip
RUN wget -qO- https://raw.githubusercontent.com/pypa/pip/1.5.5/contrib/get-pip.py | python -

# install hook dependencies
RUN pip install pyyaml requests

# install all i18n locales
RUN ln -s /usr/share/i18n/SUPPORTED /var/lib/locales/supported.d/all && locale-gen

# install git and configure gituser
ENV GITHOME /home/git
ENV GITUSER git
RUN useradd -d $GITHOME $GITUSER
RUN mkdir -p $GITHOME/.ssh && chown git:git $GITHOME/.ssh
RUN chown -R $GITUSER:$GITUSER $GITHOME

# let the git user run `sudo /home/git/builder` (not writeable)
RUN echo "%git    ALL=(ALL:ALL) NOPASSWD:/home/git/builder" >> /etc/sudoers

# HACK: import progrium/cedarish as a tarball
# see https://github.com/deis/deis/issues/1027
RUN wget -O /progrium_cedarish.tar --progress=dot:giga \
        https://s3-us-west-2.amazonaws.com/opdemand/progrium_cedarish_cedar.tar

# define the execution environment
VOLUME /var/lib/docker
WORKDIR /app
ENTRYPOINT ["/app/bin/entry"]
CMD ["/app/bin/boot"]
EXPOSE 22

ADD . /app
ADD sshd_config /etc/ssh/sshd_config
RUN chown -R root:root /app
