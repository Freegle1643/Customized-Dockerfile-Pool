## Running X server with spice in Docker in Brief

### Links that I refer to

The following Tutorial and Docker file section contain all the important links that I refer to through this building process. Most questions can be solved via these links. You may still want to Google your problem, but I guess I've already collect most of them.  

####Tutorial

[docker/Tutorials/GUI - ROS Wiki](http://wiki.ros.org/docker/Tutorials/GUI) Includes commands to run docker container that enable X server.

[user interface - Alternatives to ssh X11-forwarding for Docker containers - Stack Overflow](https://stackoverflow.com/questions/25281992/alternatives-to-ssh-x11-forwarding-for-docker-containers/25334301#25334301)

[Starting Xserver in Docker Ubuntu container - Stack Overflow](https://stackoverflow.com/questions/26075741/starting-xserver-in-docker-ubuntu-container/26104426#26104426) Suggests using `--privileged`

[docker run | Docker Documentation](https://docs.docker.com/engine/reference/commandline/run/)

[XSERVER](https://www.x.org/archive/current/doc/man/man1/Xserver.1.xhtml)

[Xspice in containers | S3hh's Blog](https://s3hh.wordpress.com/2014/04/18/xspice-in-containers/)

[Xspice server in a Container [Ghanima]](http://ghanima.net/doku.php?id=wiki:ghanima:xspiceservercontainer)

[xhost+ : How to Fix “Cannot Open Display” Error While Launching GUI on Remote Server](http://www.thegeekstuff.com/2010/06/xhost-cannot-open-display/)

[xorg - How to start a second X session? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/85383/how-to-start-a-second-x-session) Start X in a second display

[spice_for_newbies.pdf](https://www.spice-space.org/static/docs/spice_for_newbies.pdf) Includes some structural concepts of spice

[Spice User Manual](https://www.spice-space.org/spice-user-manual.html)

[Running GUI apps with Docker – Fábio Rehm](http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/) An example to run a GUI Firefox container 

[xserver-xspice : Trusty (14.04) : Ubuntu](https://launchpad.net/ubuntu/trusty/+package/xserver-xspice)



#### Dockerfile | Image that I refer to

[fcwu/docker-ubuntu-vnc-desktop: Docker image to provide HTML5 VNC interface to access Ubuntu 16.04 LXDE desktop environment](https://github.com/fcwu/docker-ubuntu-vnc-desktop) 

[docker-xorg/Dockerfile at xonly · Kry07/docker-xorg](https://github.com/Kry07/docker-xorg/blob/xonly/Dockerfile)

[docker-xorg/Dockerfile at trusty-xonly · Kry07/docker-xorg](https://github.com/Kry07/docker-xorg/blob/trusty-xonly/Dockerfile)



### Dockerfile that I use

```dockerfile
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
		firefox \
	&& apt-get autoclean \
	&& apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/*

WORKDIR /root

CMD /bin/bash

EXPOSE 5910
EXPOSE 22
```

This Dockerfile should create a image from Ubuntu 16.04 with basic network and installation tools. A GUI desktop called [XFCE4](https://xfce.org/) and basic components of X server, spice are also installed. 

### Steps to run

#### Build your image

Copy the content of `Dockerfile` above and put it into a file named `Dockerfile`, inside this directory, type:

```bash
docker build -t="<your name>/<image name>" .
```

#### Start a container

If you are in China, I highly recommend using mirrors from *[DaoCloud](https://www.daocloud.io/mirror#accelerator-doc)* to accelerate pull speed, otherwise just type the following command, it would give container a privilege to use some of the devices of the host and also start X:

```bash
docker run -ti \
       -e DISPLAY=$DISPLAY \
       -v /tmp/.X11-unix:/tmp/.X11-unix \
       -p <host port to run spice>:<port you EXPOSE to run spice> \
       --privileged \
       <your name>/<image name>
```

You may modify the command above according to your actual needs. These links should help for further information.

- [docker/Tutorials/GUI - ROS Wiki](http://wiki.ros.org/docker/Tutorials/GUI)
- [user interface - Alternatives to ssh X11-forwarding for Docker containers - Stack Overflow](https://stackoverflow.com/questions/25281992/alternatives-to-ssh-x11-forwarding-for-docker-containers/25334301#25334301)
- [Starting Xserver in Docker Ubuntu container - Stack Overflow](https://stackoverflow.com/questions/26075741/starting-xserver-in-docker-ubuntu-container/26104426#26104426) 
- [docker run | Docker Documentation](https://docs.docker.com/engine/reference/commandline/run/)

#### Inside your container

Here is where problem start to occur, inside the container we should start our spice server by, the port `5910` should match the one you opened as you run container.

```bash
sudo Xspice --port 5910 --disable-ticketing --tls-port 0 -noreset $DISPLAY
```

or

```bash
cp /usr/share/doc/xserver-xspice/spiceqxl.xorg.conf.example.gz /root
sudo gunzip spiceqxl.xorg.conf.example.gz
mv spiceqxl.xorg.conf.example spiceqxl.xorg.conf

/usr/bin/Xorg -config /root/spiceqxl.org.conf :<somenumber>
```

The two methods are based on the only two resources I find to enable spice with X in container:

- [Xspice server in a Container [Ghanima]](http://ghanima.net/doku.php?id=wiki:ghanima:xspiceservercontainer)
- [Xspice in containers | S3hh's Blog](https://s3hh.wordpress.com/2014/04/18/xspice-in-containers/)

Here we can see the first error messages. However, it still seems that connection has been successfully established.

```bash
_XSERVTransSocketCreateListener: failed to bind listener
_XSERVTransSocketUNIXCreateListener: ...SocketCreateListener() failed
_XSERVTransMakeAllCOTSServerListeners: failed to create listener for unix

X.Org X Server 1.18.4
Release Date: 2016-07-19
X Protocol Version 11, Revision 0
Build Operating System: Linux 4.4.0-97-generic x86_64 Ubuntu
Current Operating System: Linux a2a774c78e4e 4.4.0-97-generic #120~14.04.1-Ubuntu SMP Wed Sep 20 15:53:13 UTC 2017 x86_64
Kernel command line: BOOT_IMAGE=/boot/vmlinuz-4.4.0-97-generic root=UUID=8e0abfcc-784d-4d2a-a067-39ce42f33f96 ro find_preseed=/preseed.cfg auto noprompt priority=critical locale=en_US quiet
Build Date: 13 October 2017  01:57:05PM
xorg-server 2:1.18.4-0ubuntu0.7 (For technical support please see http://www.ubuntu.com/support) 
Current version of pixman: 0.33.6
	Before reporting problems, check http://wiki.x.org
	to make sure that you have the latest version.
Markers: (--) probed, (**) from config file, (==) default setting,
	(++) from command line, (!!) notice, (II) informational,
	(WW) warning, (EE) error, (NI) not implemented, (??) unknown.
(==) Log file: "/var/log/Xorg.1.log", Time: Thu Nov  2 09:02:46 2017
(++) Using config file: "/etc/X11/spiceqxl.xorg.conf"
(==) Using system config directory "/usr/share/X11/xorg.conf.d"
resizing surface0 to 16777216
memory space from 0x7f1952cd3010 to 0x7f1959cd0010
memory space from 0x7f1949cd2010 to 0x7f1951cd2010
resizing surface0 to 16777216
memory space from 0x7f1952cd3010 to 0x7f1959cd0010
memory space from 0x7f1949cd2010 to 0x7f1951cd2010
playback: no audio FIFO directory, audio is disabled
slots start: 0, slots end: 1
done reset
qxl_surface_create: Bad bpp: 1 (1)
qxl_surface_create: Bad bpp: 1 (1)
qxl_surface_create: Bad bpp: 1 (1)
main_channel_link: add main channel client
main_channel_handle_parsed: net test: latency 15.433000 ms, bitrate 1513673318 bps (1443.551367 Mbps)
red_dispatcher_set_cursor_peer: 
inputs_connect: inputs channel client create
```

#### Outside your container, i.e. host machine

Here  `spicy` is required to install, use `sudo apt install spice-client-gtk`. And then you can type the following to run spicy client. However, a spice window was opened without nothing in it.

```bash
freegle@ubuntu:~$ spicy -h 127.0.0.1 -p 5971

(spicy:70390): Gtk-WARNING **: Unable to locate theme engine in module_path: "adwaita",

(spicy:70390): GLib-CRITICAL **: Source ID 13 was not found when attempting to remove it
GSpice-Message: main channel: opened

(spicy:70390): GLib-CRITICAL **: Source ID 35 was not found when attempting to remove it

(spicy:70390): GLib-CRITICAL **: Source ID 66 was not found when attempting to remove it

(spicy:70390): GLib-CRITICAL **: Source ID 67 was not found when attempting to remove it

```

**Output**

![](http://markdownnotebucket-1251801748.cossh.myqcloud.com/201711Intel/spice01.png)

### Problems so far

After errors above occurred, I closed spice client to terminal in container to find out why. Inside container terminal, I used `ps -e|grep x` (and tried `spice` as well)  to find process, but nothing has shown. (**Don't  use `ps -ef`because it shows an output whatever you type**) 

#### Try to start X

I try to use `startx` and `xinit` to start X server first instead of heading to spice. But when I `xinit` failed with following error output.

```bash
root@a2a774c78e4e:~# xinit

_XSERVTransSocketUNIXCreateListener: ...SocketCreateListener() failed
_XSERVTransMakeAllCOTSServerListeners: server already running
(EE) 
Fatal server error:
(EE) Cannot establish any listening sockets - Make sure an X server isn't already running(EE) 
(EE) 
Please consult the The X.Org Foundation support 
	 at http://wiki.x.org
 for help. 
(EE) Please also check the log file at "/var/log/Xorg.0.log" for additional information.
(EE) 
(EE) Server terminated with error (1). Closing log file.
No protocol specified
xinit: giving up
xinit: unable to connect to X server: Resource temporarily unavailable
xinit: server error

```

However, `startx` succeeded. Since I run the container with `--privileged`, it also occupied the screen of my host. On the display, we see it start xfce4 successfully but without any response from mouse or keyboard.

```bash
root@a2a774c78e4e:~# startx

_XSERVTransSocketCreateListener: failed to bind listener
_XSERVTransSocketUNIXCreateListener: ...SocketCreateListener() failed
_XSERVTransMakeAllCOTSServerListeners: failed to create listener for unix

X.Org X Server 1.18.4
Release Date: 2016-07-19
X Protocol Version 11, Revision 0
Build Operating System: Linux 4.4.0-97-generic x86_64 Ubuntu
Current Operating System: Linux a2a774c78e4e 4.4.0-97-generic #120~14.04.1-Ubuntu SMP Wed Sep 20 15:53:13 UTC 2017 x86_64
Kernel command line: BOOT_IMAGE=/boot/vmlinuz-4.4.0-97-generic root=UUID=8e0abfcc-784d-4d2a-a067-39ce42f33f96 ro find_preseed=/preseed.cfg auto noprompt priority=critical locale=en_US quiet
Build Date: 13 October 2017  01:57:05PM
xorg-server 2:1.18.4-0ubuntu0.7 (For technical support please see http://www.ubuntu.com/support)
Current version of pixman: 0.33.6
        Before reporting problems, check http://wiki.x.org
        to make sure that you have the latest version.
Markers: (--) probed, (**) from config file, (==) default setting,
        (++) from command line, (!!) notice, (II) informational,
        (WW) warning, (EE) error, (NI) not implemented, (??) unknown.
(==) Log file: "/var/log/Xorg.2.log", Time: Thu Nov  2 11:06:01 2017
(==) Using system config directory "/usr/share/X11/xorg.conf.d"

```

![](http://markdownnotebucket-1251801748.cossh.myqcloud.com/201711Intel/spice02.png)

You can interrupt it by type `ctrl + C`, and some extra output came.

```bash
^Cxinit: connection to X server lost

waiting for X server to shut down (II) Server terminated successfully (0). Closing log file.

xinit: unexpected signal 2
root@a2a774c78e4e:~#

```



### Possible explanations and solutions

I believe there are some correct operation that we have done, maybe some minor adjustments is to be made to achieve the goal to enable Xspice in container. Here are some guess I can think of:

- Assign a new display other than host display to container to run, so it wouldn't occupy host's display
- Correspondingly assign that display for spice to use inside container

### Required help

