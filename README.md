# Customized-Dockerfile-Pool
*This is a repository I use mainly to manage all my customized Dockerfiles in one place.*

git repository: https://github.com/Freegle1643/Customized-Dockerfile-Pool

using `git clone https://github.com/Freegle1643/Customized-Dockerfile-Pool.git` to clone repository.

#### Before we start

If you are in China and experience a significant slow pulling speed from [Docker Hub](hub.docker.com), you might want to try [*DaoCloud* mirror site](https://www.daocloud.io/mirror#accelerator-doc). It could dramatically make your learning process faster and more efficient.

## XSpice Dockerfile

This is a markdown note that document virtually every important process of my struggle to run X server and spice in a docker container. This is part also of my job as an Intern.

###Phase 1

*This phase is a bad start, you may skip to Phase 2.*

*The following two Docker file can be find in First try directory.*

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

`net-tools` : A collection of programs that form the base set of the NET-3 networking distribution, e.g. `ifconfig`.



### Dockerfile_171026_01_XSpicetry02

#### Output

Installation was successful with a 2.6GB image created, this resembles XSpicetry01 very much, with some deletes and order adjustments. However, X server failed to start, I would assume it is because `xserver-xspice xinput` that I learned from [Xspice server in a Container [Ghanima]](http://ghanima.net/doku.php?id=wiki:ghanima:xspiceservercontainer). 

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

Unfortunately, there isn't much to learn even we have successfully build this image and run it interactively with command line, because accessing it graphically is instead our goal.

`RUN echo "root:Docker!" | chpasswd` : Since we added `sudo` package, we sometimes are required a root password, the above command perform a root password setting with `Docker!` to be its password.



###Phase 2

### Ubuntu-16.04-vnc-xfce 

I found a [xfce](https://xfce.org/) desktop and VNC installed Ubuntu Docker image on [Docker Hub](https://hub.docker.com/r/welkineins/ubuntu-xfce-vnc-desktop/). It works perfect and with some great fundamental features. It was built from a `ubuntu:14.04` image, I tried to modified it to `16.04` and added `sudo` in it because this package is no longer considered in the minimum image. I started with VNC because it is also a remote graphical desktop control solution with some installation and port setup, learning from here could be a reasonably good start.

#### Output

Xfce, when based on `ubuntu:16.04`, looks a bit better than `ubuntu:14.04`.  Inside the directory that contains `Dockefile`, `startup.sh`  and `supervisord.conf` ,, build image yourself and then run with following command

`docker run -i -t -p 5900:5900 <your name>/<your image name>`

*Notice, you might want to use `-d` (i.e. detached mode) instead of `-i -t` if you don't want closing terminal or exit tty to cause a stop of your container. `5900` on the left can be any available port you want to connect to your container using VNC.*

```
-t              : Allocate a pseudo-tty
-i              : Keep STDIN open even if not attached
```

####Dockerfile

```dockerfile
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
		xfce4 xfce4-goodies \
		x11vnc xvfb \
		firefox \
	&& apt-get autoclean \
	&& apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/*

WORKDIR /root

ADD startup.sh ./
ADD supervisord.conf ./

EXPOSE 5900
EXPOSE 22

ENTRYPOINT ["./startup.sh"]
```



#### What is to learn ?

After image has been created and container started, I discovered that 16.04 has a more minimized 'smallest' image comparing to 14.04, `ifconfig`, `ping` and some related `net-tools` and `iputils-ping`packages are not included. So I add it inside Dockerfile and build again(The Dockerfile above includes). 

**If `ENTRYPOINT` is a .sh script file?**

Under such condition, we **MUST** make this script file executable rather than just a plain text file by employee following command, or you should encounter errors indicating 'permission denied' as you run your container.

`sudo chmod +x /path/to/script.sh`  

I notice that many GUI container images on Docker Hub uses [XFCE](https://xfce.org/) or [LXDE](http://lxde.org/) instead of [GNOME](https://www.gnome.org/) as their desktop, I would assume it is because the first two are more light weight, and graphic performance and resources are something we don't have a lot inside a container. [Here](https://hub.docker.com/r/garland/dockerfile-ubuntu-gnome/) is a ubuntu image that come with GNOME, you may want to refer the Dockerfile to enable GNOME in your image.



### Enable XSpice by operating container

Here I don't jump to write my own Dockerfile to enable XSpice, because build could significantly occupies time and space. So based on the image above, I directly operate a container started from it trying to install XSpice in it. We assume we are to use 5910 port to run XSpice, so inside the Dockerfile, we additionally write `EXPOSE 5910`. 

#### Output

#### Dockerfile

#### What is to learn ?



#### Output

#### Dockerfile

#### What is to learn ?



