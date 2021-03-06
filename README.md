[![Build Status](https://travis-ci.org/deferpanic/gorump.svg?branch=travis)](https://travis-ci.org/deferpanic/gorump)

# gorump

This contains code to run Go on the [Rumprun unikernel](https://github.com/rumpkernel/rumprun).

Rumprun is a special project because it allows you to run your Go apps
unmodified directly on the hypervisor of your choice such as KVM or Xen.

This allows faster boot times, smaller images and a much smaller attack
surface for security.

We believe unikernels are the future of infrastructure.

This repo is based on Go the source 1.5.1 stable c7d78ba4df574b5f9a9bb5d17505f40c4d89b81c
downloaded at https://storage.googleapis.com/golang/go1.5.1.src.tar.gz .

On top, the NetBSD platform has been modified to support Rumprun instead.
To generate a patch: `git diff go-1-5-1-upstream master`.

We don't intend to fork Go but there's quite a lot of work to do to get
it in enough shape to put into the main tree.

You can find the latest build of the modified Go @ [https://s3.amazonaws.com/dp-gorump/gorump.tar.gz](https://s3.amazonaws.com/dp-gorump/gorump.tar.gz) .

Please submit pull requests!

See `examples` directory on how to build and use.

### Getting Started

There are 2 ways you can install - from apt-get or by downloading the
source and the rumprun source and building it yourself.

#### Install from apt-get
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:engineering-s/gorump
sudo apt-get update
sudo apt-get install gorump
```

#### Install from source

##### Install dependencies
```
sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
sudo apt-get update -y
sudo apt-get install qemu-kvm -y
sudo apt-get install libxen-dev -y
sudo apt-get install g++-4.8 -y
```

##### Install Go to Bootstrap the Modified Go
```
wget https://storage.googleapis.com/golang/go1.5.2.linux-amd64.tar.gz
tar xzf go1.5*
sudo mv go /usr/local/go1.5
sudo ln -s /usr/local/go1.5 /usr/local/go
```

##### Add Env variables to your ~/.bashrc
```
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=/home/$(whoami)/go
export PATH=$PATH:$GOPATH/bin
```

##### Download and Install Rumprun

```
git clone https://github.com/rumpkernel/rumprun
git pull origin master
git submodule update --init
CC=cc ./build-rr.sh hw
```

##### Add the Rumprun env to your path
```
export PATH="${PATH}:/home/$(whoami)/rumprun/rumprun/bin"
```

##### Build the Modified Go
(from within this repository)
```
cd go/src GOROOT_BOOTSTRAP=/usr/local/go GOOS=netbsd GOARCH=amd64 ./make.bash
```

##### Install the Modified Go
(from within this repository)
```
sudo cp -R go /usr/local/go1.5-patched
sudo rm -rf /usr/local/go
sudo ln -s /usr/local/go1.5-patched /usr/local/go
```

### Create your first Rumprun Hello World Webserver
```
cd examples && make
```

#### Add Networking to your Image
```
sudo ip tuntap add tap0 mode tap
sudo ifconfig tap0 inet 10.181.181.181 up
```

#### Run the Rumprun kernel
```
rumprun qemu -i -g '-nographic -vga none' -D 1234 -I t,vioif,'-net tap,ifname=tap0,script=no' -W t,inet,static,10.181.181.180/24 httpd.bin
```

#### Test Your Go Rumprun server
```
curl http://10.181.181.180:3000/fast
```
