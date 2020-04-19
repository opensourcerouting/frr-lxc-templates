# FRRouting Build & Topotests in LXC containers

## Requirements

Only tested on Ubuntu 18.04LTS, but might work on other Distros and versions
Install lxc and distrobuilder:

	apt install lxc-utils
	snap install distrobuilder --classic

## Building and Installing Template

1. Build LXC Image Template

(Example based on Ubuntu 18.04 AMD64 Template. Use Template as needed)

	cd ContainerImages/ubuntu
	distrobuilder build-lxc ubuntu-18.04-amd64.yaml

2. Add Image to LXC Images

Image is built, needs to be added to local image storage

	lxc-create -n ubuntu-18.04-amd64 -t local -- --metadata meta.tar.xz --fstree rootfs.tar.xz

## Start a copy of the locally installed image

Create and start a ephemeral copy of the image

	lxc-copy -n U1804 -N Ucopy -e -m bind=/root/test:/root/test

Execute tests as needed with `lxc-attach` and stop container with `lxc-stop` afterwards
