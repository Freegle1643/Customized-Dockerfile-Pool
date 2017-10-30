FROM ubuntu:16.04
MAINTAINER Hao Yuan <freegleyuan@foxmail.com>

ENV HOME /root
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update \
	&& apt-get install -y supervisor \
		sudo \
		net-tools \
		iputils-ping \
		openssh-server vim-tiny \
		xauth xorg openbox lightdm plymouth \
		xfce4 xfce4-goodies \
		x11vnc xvfb \
		firefox \
		xserver-xspice \
	&& apt-get autoclean \
	&& apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/*

WORKDIR /root

ADD startup.sh ./
ADD supervisord.conf ./

EXPOSE 5900
EXPOSE 5910
EXPOSE 22

ENTRYPOINT ["./startup.sh"]