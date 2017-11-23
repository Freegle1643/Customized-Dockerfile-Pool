## Enable Xspice in Container with Docker

Previously I've tried multiple times on enabling Xspice in container but failed with some seemingly unsolvable errors. Lucky enough, I've made some progress recently and managed to run Xspice in a Docker container. In this document, I will list the essential steps to fire up the Xspice remote desktop setup and hopefully show you how to make further customization on the Docker image and enabling audio, etc.

### Pre-requests

#### Environment

Due to the disturbance of glab and the complex network setup with those machine, I again use my VM in VMware Workstation instead to do this experiment. Of course you can do this on virtually any Linux with Docker installed.

```
OS: Ubuntu 16.04 64 bit
CPU: Intel® Core™ i7-6700HQ CPU @ 2.60GHz × 4 
Memory: 3.8 GiB
Graphics: Gallium 0.4 on SVGA3D; build: RELEASE;  LLVM;
Docker Version: 17.04.0-ce
```

#### Spice Client

On the host machine, we need a spice client in order to interact with our container using spice. here we chose spicy, type following command to install

```bash
sudo apt install spice-client-gtk
```

### Build Docker Image

Obviously we need a Docker image for this purpose, here is the Dockerfile I use. Simple use `touch Dockerfile` then paste the following content into it in an empty directory.

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
```

Then use the following command to build the image where our Dockerfile stored.

```bash
docker build -t="haoyuan/ubuntu1604-xspice-xfce" .
```

*If you are in China and want to accelerate the pull speed of Docker image, try [DaoCloud](https://www.daocloud.io/mirror#accelerator-doc) mirrors.*

This Dockerfile should create a image from Ubuntu 16.04 with basic network and installation tools. A GUI desktop called [XFCE4](https://xfce.org/) and basic components of X server, spice are also installed. I wrote the X server part referring to some Dockerfile on [Docker Hub](hub.docker.com). As for spice, I used a Ubuntu package named [`xserver-xspice`](https://launchpad.net/ubuntu/trusty/+package/xserver-xspice), which will help us set up Xspice in container. Port 5900 and 5901 are default port for spice and tls port.

### Run Container

If you are not sure, use `docker images` to see if there is a image named as above. Now we can run our container use the following command.

```bash
docker run -ti \
       -e DISPLAY=$DISPLAY \
       -v /tmp/.X11-unix:/tmp/.X11-unix \
       -p <host port to run spice>:5900 \
       -p <host port for tls connect>:5901 \
       
       haoyuan/ubuntu1604-xspice-xfce
```

`--privileged` Tried previously, but it seems that we should not apply this parameter for safe concern.

`--rm` If you only want to run your container for only once, add this parameter.

[Here](https://stackoverflow.com/questions/25281992/alternatives-to-ssh-x11-forwarding-for-docker-containers/25334301#25334301) is a further explanation I find in Stack Overflow, sure, you will always want to check [official document](https://docs.docker.com/engine/reference/run/) to see what's what.

### Enable spice server in Docker container

inside our container, we can use the following command to enable spice server

```bash
sudo Xspice --port 5900 --disable-ticketing --tls-port 0 -noreset :2
```

`:2` is to say we will enable spice on DISPLAY `:2`. 

There is an alternative way to run Xspice with further configuration by modifying the configure file.

```bash
cp /usr/share/doc/xserver-xspice/spiceqxl.xorg.conf.example.gz /root
cd /root
gunzip spiceqxl.xorg.conf.example.gz
```

Inside this file (could be read by `cat`), you can change as many parameters as you wish. For convenience reason, we disable ticking so that we don't need to set up a password.

```bash
/usr/bin/Xorg -config /root/spiceqxl.xorg.conf.example :2
```

`:2` again, this represents the display we want to assign

Either way, we shall now see output like this

```bash
X.Org X Server 1.18.4
Release Date: 2016-07-19
X Protocol Version 11, Revision 0
Build Operating System: Linux 4.4.0-97-generic x86_64 Ubuntu
Current Operating System: Linux 5978e713edb4 4.10.0-38-generic #42~16.04.1-Ubuntu SMP Tue Oct 10 16:32:20 UTC 2017 x86_64
Kernel command line: BOOT_IMAGE=/boot/vmlinuz-4.10.0-38-generic root=UUID=53640fd5-1501-43e4-8635-2f115bddcddf ro find_preseed=/preseed.cfg auto noprompt priority=critical locale=en_US quiet
Build Date: 13 October 2017  01:57:05PM
xorg-server 2:1.18.4-0ubuntu0.7 (For technical support please see http://www.ubuntu.com/support) 
Current version of pixman: 0.33.6
	Before reporting problems, check http://wiki.x.org
	to make sure that you have the latest version.
Markers: (--) probed, (**) from config file, (==) default setting,
	(++) from command line, (!!) notice, (II) informational,
	(WW) warning, (EE) error, (NI) not implemented, (??) unknown.
(==) Log file: "/var/log/Xorg.2.log", Time: Thu Nov 23 10:05:26 2017
(++) Using config file: "/etc/X11/spiceqxl.xorg.conf"
(==) Using system config directory "/usr/share/X11/xorg.conf.d"
resizing surface0 to 16777216
memory space from 0x7f590d307010 to 0x7f5914304010
memory space from 0x7f5904306010 to 0x7f590c306010
resizing surface0 to 16777216
memory space from 0x7f590d307010 to 0x7f5914304010
memory space from 0x7f5904306010 to 0x7f590c306010
playback: no audio FIFO directory, audio is disabled
slots start: 0, slots end: 1
done reset
qxl_surface_create: Bad bpp: 1 (1)
qxl_surface_create: Bad bpp: 1 (1)
qxl_surface_create: Bad bpp: 1 (1)

```

### Connect to container using spice client

Now we have our spice connection established, what we need to do now is to remotely access to our container using a valid client. As I mentioned above, `spicy` is our choice. Simply tying the following command in the host terminal.

*Note that 5926 is the port I project to 5900 inside container*

```bash
spicy -h 127.0.0.1 -p 5926
```

Now a client with a black window has been popped up.

![Spicy client](http://markdownnotebucket-1251801748.file.myqcloud.com/201711Intel/spice_s1.png)

### Run GUI software on DISPLAY

The reason why it's pure black is that we have nothing running on display `:2` , now we need to access the container with a new tty because the previous one has been occupied by spice output. Type the following to do so. Use `docker ps` to see your container's name.

```bash
docker exec -it <container name> bash
```

Now we are inside the container with another tty. Try these following command to fire up GUI software. 

```bash
DISPLAY=:2 xterm
```

![Xterm in spice](http://markdownnotebucket-1251801748.file.myqcloud.com/201711Intel/spice_s2.png)

It's a totally interactive terminal that we can type things on.

Back to the first terminal we identify more output caused by these interactions.

```bash
main_channel_link: add main channel client
main_channel_handle_parsed: net test: latency 22.375000 ms, bitrate 8126984126 bps (7750.496031 Mbps)
red_dispatcher_set_cursor_peer: 
inputs_connect: inputs channel client create
qxl_surface_create: Bad bpp: 1 (1)
qxl_surface_create: Bad bpp: 1 (1)
```

Get bored? Use `ctrl` + `C` in previous terminal to stop xterm.

Now you may want to try more complex application like a desktop. Sure, let's fire up xfce4.

```bash
DISPLAY=:2 startxfce4
```

It may take a few seconds due to its complexity, and it may produce many output in the terminal as well. But after that we shall see this. An interactive desktop!

![Xfce4 in Xspice](http://markdownnotebucket-1251801748.file.myqcloud.com/201711Intel/spice_s3.png)

There are actually some minor issues with text display, like some mysterious shading. But overall a GUI interactive desktop is ready!

If you install a `ubuntu-desktop` by add it in your Dockerfile instead of our `xfce4`, you may apply `unity` to start it. Basically, it can be any GUI software, desktops that could be executed using terminal command.

## XSpice V.S. X11 VNC

### Brief concepts



### Performance

#### Graphics

#### Bandwidth

#### Audio support



### Further improvement 

- Enable audio
- Hardware acceleration

 