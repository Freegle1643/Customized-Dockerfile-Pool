FROM ubuntu:16.04
MAINTAINER Hao Yuan <freegleyuan@foxmail.com>

ENV HOME /root
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update \
	&& apt-get install -y supervisor \
		sudo \
		wget \
		curl \
		net-tools \
		iputils-ping \
		openssh-server vim-tiny \
		xserver-xorg-core \
		xfonts-base \
		x11-session-utils x11-utils x11-xfs-utils \
	    xauth x11-common \
	    xinit \
		xserver-xspice xinput\
		xfce4 xfce4-goodies \
		xvfb \
		firefox \
	&& apt-get autoclean \
	&& apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/*

WORKDIR /root

CMD /bin/bash

EXPOSE 5900
EXPOSE 5901
EXPOSE 22