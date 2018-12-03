FROM debian:stretch as builder
MAINTAINER Aidan Campbell

# Install packages

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -y update
RUN apt-get -yy upgrade
ENV BUILD_DEPS="git autoconf pkg-config libssl-dev libpam0g-dev \
    libx11-dev libxfixes-dev libxrandr-dev nasm xsltproc flex \
    bison libxml2-dev dpkg-dev libcap-dev"
RUN apt-get -yy install  sudo apt-utils software-properties-common libtool-bin $BUILD_DEPS
RUN apt-get clean


# Build xrdp

WORKDIR /tmp
RUN git clone --branch v0.9.7 --recursive https://github.com/neutrinolabs/xrdp.git
WORKDIR /tmp/xrdp
RUN ./bootstrap
RUN ./configure
RUN make
RUN make install

FROM debian:stretch
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -y update
RUN apt install -yy vim wget ca-certificates xorgxrdp xrdp\
  xfce4 xfce4-terminal xfce4-screenshooter xfce4-taskmanager \
  xfce4-clipman-plugin xfce4-cpugraph-plugin xfce4-netload-plugin \
  xfce4-xkb-plugin xauth supervisor uuid-runtime locales \
  firefox-esr openssh-server sudo
RUN apt-get clean
ADD bin /usr/bin
ADD etc /etc


# Configure
RUN mkdir -p /var/run/dbus
RUN cp /etc/X11/xrdp/xorg.conf /etc/X11
RUN sed -i "s/console/anybody/g" /etc/X11/Xwrapper.config
RUN sed -i "s/xrdp\/xorg/xorg/g" /etc/xrdp/sesman.ini
RUN locale-gen en_US.UTF-8
RUN echo "xfce4-session" > /etc/skel/.Xclients
RUN cp -r /etc/ssh /ssh_orig
RUN rm -rf /etc/ssh/*
RUN rm -rf /etc/xrdp/rsakeys.ini /etc/xrdp/*.pem

# Add sample user
# sample user uses uid 999 to reduce conflicts with user ids when mounting an existing home dir
RUN addgroup --gid 998 debian
RUN useradd -m -u 998 -s /bin/bash -g debian debian
RUN echo "debian:debian" | /usr/sbin/chpasswd
RUN echo "debian    ALL=(ALL) ALL" >> /etc/sudoers

# Docker config

VOLUME ["/etc/ssh","/home"]
EXPOSE 3389 22 9001
ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]
CMD ["supervisord"]