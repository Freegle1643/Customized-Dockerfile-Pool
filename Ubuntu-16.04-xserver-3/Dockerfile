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
		x11-xserver-utils \
		xfonts-base \
	    xauth \
	    xinit \
		xserver-xspice xinput\
		xfce4 xfce4-goodies \
		firefox \
	&& apt-get autoclean \
	&& apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/*

WORKDIR /root

CMD /bin/bash

EXPOSE 5910
EXPOSE 22