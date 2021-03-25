# FRRouting Build & Topotests in LXC containers

(Instructions should contain enough details for anyone without LXC experience)

## Requirements

Only tested on Ubuntu 18.04 LTS and 20.04 LTS as the LXC Server host, but 
might work on other Distros and versions

It is tested on Intel 64bit Architecture (x86_64) and Raspberry Pi 4.

For Raspberry Pi 4, only modules with 4 GB RAM are supported.
As a base OS for Raspberry, we use ARM8 Ubuntu 20.04 from
https://ubuntu.com/download/raspberry-pi
(Pick Ubuntu 20.04 `64-bit` image)

For Intel, It is suggested to install a Ubuntu 20.04LTS as the server

## Installation of LXC Server

### Base LXC Installation

I would suggest to remove the LXD tools to avoid confusion between LXC and LXD
as they don't mix well:

	apt purge lxd lxd-client

Install lxc and distrobuilder:

	apt install lxc-utils

then follow guide at https://linuxcontainers.org/lxc/getting-started/ to
create `~/.config/lxc/default.conf` and `/etc/lxc/lxc-usernet`

For Topotests, it is recommended to change the DHCP range for the containers
away from the 10.x.x.x network to 192.168.x.x range to avoid conflicts with
tests. To do this, edit `/etc/default/lxc-net` and (as a example) set the following:

	LXC_NETWORK="192.168.121.0/24"
	LXC_ADDR="192.168.121.1"
	LXC_DHCP_RANGE="192.168.121.100,192.168.121.254"
	LXC_DHCP_MAX="150"

### Distrobuilder installation

Build and install distrobuilder from source. I suggest to install it under `/opt`
with a symlink in `/usr/local/bin`:

	apt install golang-go debootstrap rsync gpg squashfs-tools git
	GOPATH=/opt/go; export GOPATH
	go get -d -v github.com/lxc/distrobuilder/distrobuilder
	cd /opt/go/src/github.com/lxc/distrobuilder
	make
	ln -s /opt/go/bin/distrobuilder /usr/local/bin/distrobuilder

### Extra steps on LXC hosts to run Topotest containers

Not needed if only building FRR inside LXC. This is only needed to run
TopoTests, but it does not impact build containers. 

LXC containers can't load extra modules inside the container, so we have to always
load the MPLS modules on the host to have them accessable by the containers.

Edit `/etc/modules` and add

	mpls_router
	mpls_iptunnel

Disable apport on LXC host to avoid Ubuntu moving dumps:

	echo "enabled=0" >> /etc/apport

Modify `/etc/security/limits.conf` to enable core dumps (item core) and 
enough virtual terminal connections (item soft). Add the following lines:

	#<domain>      <type>  <item>         <value>
	*               soft    core          unlimited
	root            soft    core          unlimited
	*               hard    core          unlimited
	root            hard    core          unlimited
        *               soft    nofile        unlimited
        root            soft    nofile        unlimited

Reboot LXC Host to activate the changes

## Building and Installing Template

1. Checkout this git on the LXC host

2. Build LXC Image Template

(Example based on Ubuntu 18.04 AMD64 Template. Use Template as needed)

	cd ContainerImages/ubuntu
	distrobuilder build-lxc topotest_ubuntu1804_amd64.yaml

or to override the Distrobution to Ubuntu 16.04 (xenial) use

	distrobuilder build-lxc topotest_ubuntu1804_amd64.yaml -o image.release=xenial

or to build i386 Ubuntu 16.04:

	distrobuilder build-lxc topotest-ubuntu-18.04-amd64.yaml -o image.release=xenial -o image.architecture=i386

Other tested containers:

- Build on Intel (on x86_64 Ubuntu 20.04 host)
	- Ubuntu 16.04, i386:  `distrobuilder build-lxc build-ubuntu-18.04-amd64.yaml -o image.release=xenial -o image.architecture=i386`
	- Ubuntu 16.04, x86_64:  `distrobuilder build-lxc build-ubuntu-18.04-amd64.yaml -o image.release=xenial -o image.architecture=x86_64`
	- Ubuntu 18.04, x86_64:  `distrobuilder build-lxc build-ubuntu-18.04-amd64.yaml -o image.release=bionic -o image.architecture=x86_64`
	- Ubuntu 20.04, x86_64:  `distrobuilder build-lxc build-ubuntu-18.04-amd64.yaml -o image.release=focal -o image.architecture=x86_64`

- Build on ARM (on Raspberry Pi4, Ubuntu 20.04 host)
	- Ubuntu 16.04, armhf:  `distrobuilder build-lxc build-ubuntu-18.04-arm64.yaml -o image.release=xenial -o image.architecture=armhf`
	- Ubuntu 16.04, aarch64:  `distrobuilder build-lxc build-ubuntu-18.04-arm64.yaml -o image.release=xenial -o image.architecture=aarch64`
	- Ubuntu 18.04, armhf:  `distrobuilder build-lxc build-ubuntu-18.04-arm64.yaml -o image.release=bionic -o image.architecture=armhf`
	- Ubuntu 18.04, aarch64:  `distrobuilder build-lxc build-ubuntu-18.04-arm64.yaml -o image.release=bionic -o image.architecture=aarch64`
	- Ubuntu 20.04, armhf:  `distrobuilder build-lxc build-ubuntu-18.04-arm64.yaml -o image.release=focal -o image.architecture=armhf`
	- Ubuntu 20.04, aarch64:  `distrobuilder build-lxc build-ubuntu-18.04-arm64.yaml -o image.release=focal -o image.architecture=aarch64`

- TopoTest on Intel (on x86_64 Ubuntu 20.04 host)
	- Ubuntu 16.04, i386:  `distrobuilder build-lxc topotest-ubuntu-18.04-amd64.yaml -o image.release=xenial -o image.architecture=i386`
	- Ubuntu 16.04, x86_64:  `distrobuilder build-lxc topotest-ubuntu-18.04-amd64.yaml -o image.release=xenial -o image.architecture=x86_64`
	- Ubuntu 18.04, x86_64:  `distrobuilder build-lxc topotest-ubuntu-18.04-amd64.yaml -o image.release=bionic -o image.architecture=x86_64`

- TopoTest on ARM (on Raspberry Pi4, Ubuntu 20.04 host)
	- Ubuntu 16.04, armhf:  `distrobuilder build-lxc topotest-ubuntu-18.04-arm64.yaml -o image.release=xenial -o image.architecture=armhf`
	- Ubuntu 16.04, aarch64:  `distrobuilder build-lxc topotest-ubuntu-18.04-arm64.yaml -o image.release=xenial -o image.architecture=aarch64`
	- Ubuntu 18.04, armhf:  `distrobuilder build-lxc topotest-ubuntu-18.04-arm64.yaml -o image.release=bionic -o image.architecture=armhf`
	- Ubuntu 18.04, aarch64:  `distrobuilder build-lxc topotest-ubuntu-18.04-arm64.yaml -o image.release=bionic -o image.architecture=aarch64`

3. Add build LXC image to the local containers

Image is built, needs to be added to local image storage

	lxc-create -n topotest_ubuntu1804_amd64 -t local -- --metadata meta.tar.xz --fstree rootfs.tar.xz

This creates a new `topotest_ubuntu1804_amd64` container.

Step 2 & 3 can be repeated for each LXC image you want to build

## Start a copy of the locally installed image

Create a local shared directory to exchange files between container and
host. (Easy way to share files)

	mkdir /root/shared

Create and start a ephemeral copy of the image (the ephemeral images
only save changes to template and are deleted on shutdown).

	lxc-copy -n topotest_ubuntu1804_amd64 -N mytopotest -e -m bind=/root/shared:/root/shared

`topotest_ubuntu1804_amd64` is the name of the templated container created in previous
step. {Check `lxc-ls -f` for a list of locally available containers)
`mytopotest` is the name of the new container created and started.

Please be aware, that topotest LXC containers need to be priviledged.

## Accessing the LXC 
Execute tests as needed with `lxc-attach` and stop container with `lxc-stop` afterwards
`lxc-attach` has some limited environment. For a full shell experience, use SSH to 
login. However, you need to set passwords or add a authorized_keys file first.

The `/root/shared` folder can be used to exchange data between container and 
host.

