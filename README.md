# Customized-Dockerfile-Pool
*This is a repository I use mainly to manage all my customized Dockerfiles in one place.*

git repository: https://github.com/Freegle1643/Customized-Dockerfile-Pool

using `git clone https://github.com/Freegle1643/Customized-Dockerfile-Pool.git` to clone repository.

#### Before we start

If you are in China and experience a significant slow pulling speed from [Docker Hub](hub.docker.com), you might want to try [*DaoCloud* mirror site](https://www.daocloud.io/mirror#accelerator-doc). It could dramatically make your learning process faster and more efficient.

## XSpice Dockerfile

This is a markdown note that document virtually every important process of my struggle to run X server and spice in a docker container. This is part also of my job as an Intern.

### Dockerfile_171026_01_XSpicetry01

It's my first try to write my own Dockerfile, which ends up not so good. I initially look for a image that have standard components, command, web browser, video player and X, VNC, XSpice installed. However, the disorganized `RUN` part leads to a bad result. 

####Output

Failed while installation. 

####Dockerfile

```dockerfile
FROM ubuntu:16.04
MAINTAINER Hao Yuan <freegleyuan@foxmail.com>

ENV HOME /root

#set up proxy for intel inner network
ENV https_proxy "https://child-prc.intel.com:913/"
ENV http_proxy "http://child-prc.intel.com:913/"
ENV ftp_proxy "http://child-prc.intel.com:913/"
ENV no_proxy "localhost,127.0.0.1,.intel.com"


RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  	&& echo "deb http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google.list \
	&& apt-get update \
	&& apt-get install -y supervisor \
		apt-utils \
		aufs-tools \
		automake \
		bash-completion \
		binutils-mingw-w64 \
		bsdmainutils \
		btrfs-tools \
		build-essential \
		clang \
		cmake \
		createrepo \
		curl \
		dpkg-sig \
		gcc-mingw-w64 \
		git \
		iptables \
		jq \
		less \
		libapparmor-dev \
		libcap-dev \
		libltdl-dev \
		libnl-3-dev \
		libprotobuf-c0-dev \
		libprotobuf-dev \
		libsystemd-journal-dev \
		libtool \
		libzfs-dev \
		mercurial \
		net-tools \
		pkg-config \
		protobuf-compiler \
		protobuf-c-compiler \
		python-dev \
		python-mock \
		python-pip \
		python-websocket \
		tar \
		ubuntu-zfs \
		vim \
		vim-common \
		xfsprogs \
		zip \
		openssh-server vim-tiny \
		xfce4 xfce4-goodies \
		x11vnc xvfb \
		firefox \
		google-chrome-stable \
		mplayer \
		xserver-xspice xinput \
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
```

#### What is to learn ?

Basic official images like `ubuntu:16.04` dose not come with some so-called basic commands, like `ifconfig`, `wget`, `curl`,`sudo` or `ping` etc. So we need to manual install them for either software installation or production usage.

Within basic images, `ls`, `echo` and `export` are good to use, however, if we need to set up some environment for software installation like Chrome, which requires adding key via `wget`. Always remember to install such command(or call it software if you will) prior to your software installation. 

*If the amount of things you need to install and run in `RUN` is huge, separate them into different section and using comment is good both for you can open source community.*

Dockerfile `RUN` part typical format: 

```dockerfile
RUN apt-get update \
	&& apt-get install -y supervisor \
		openssh-server vim-tiny \
		xfce4 xfce4-goodies \
		x11vnc xvfb \
		firefox \
	&& apt-get autoclean \
	&& apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/*
```

*Note that the first two and final three lines of code is usually seen in many dockerfiles, so better also keep them in your dockerfile.*

`apt-utils` : package management related utility programs

`curl` : command line tool for transferring data with URL syntax

`net` : A collection of programs that form the base set of the NET-3 networking distribution, e.g. `ifconfig`.



### Dockerfile_171026_01_XSpicetry02

#### Output

Installation was successful with a 2.6GB image created, this 

#### Dockerfile

```dockerfile
FROM ubuntu:16.04
MAINTAINER Hao Yuan <freegleyuan@foxmail.com>

ENV HOME /root
ENV DEBIAN_FRONTEND=noninteractive

#set up proxy for intel inner network
#ENV https_proxy "https://child-prc.intel.com:913/"
#ENV http_proxy "http://child-prc.intel.com:913/"
#ENV ftp_proxy "http://child-prc.intel.com:913/"
#ENV no_proxy "localhost,127.0.0.1,.intel.com"


RUN apt-get update \
	&& apt-get install -y supervisor \
		apt-utils \
		aufs-tools \
		automake \
		bash-completion \
		binutils-mingw-w64 \
		bsdmainutils \
		btrfs-tools \
		build-essential \
		clang \
		cmake \
		createrepo \
		curl \
		dpkg-sig \
		gcc-mingw-w64 \
		git \
		iptables \
		jq \
		less \
		net-tools \
		pkg-config \
		protobuf-compiler \
		protobuf-c-compiler \
		python-dev \
		python-mock \
		python-pip \
		python-websocket \
		sudo \
		tar \
		vim \
		vim-common \
		xfsprogs \
		zip \
		openssh-server vim-tiny \
		xfce4 xfce4-goodies \
		x11vnc xvfb \
		firefox \
		mplayer \
		software-properties-common \
		xserver-xspice xinput \
	&& apt-get autoclean \
	&& apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/*

RUN echo "root:Docker!" | chpasswd

WORKDIR /root

#ADD startup.sh ./
#ADD supervisord.conf ./

EXPOSE 5900
EXPOSE 5910
EXPOSE 22

#ENTRYPOINT ["./startup.sh"]
```



####What is to learn ?





#### Output

####Dockerfile

#### What is to learn ?



#### Output

#### Dockerfile

#### What is to learn ?



#### Output

#### Dockerfile

#### What is to learn ?



