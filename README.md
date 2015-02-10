# BuildServer
CSC591-Homework
##Build up Vagrant Machine
Create a VM to host your build server.  Note, it must be a 64 bit image, otherwise docker will not be supported!

    vagrant init hashicorp/precise64
##Configure VagrantFile to enable public network and forwarded port
	vim Vagrantfile
	config.vm.network "forwarded_port", guest: 6060, host: 6767
	config.vm.network "forwarded_port", guest: 8080, host: 8181

	# Create a private network, which allows host-only access to the machine
	# using a specific IP.
	config.vm.network "private_network", ip: "192.168.33.11"

	# Create a public network, which generally matched to bridged network.
	# Bridged networks make the machine appear as another physical device on
	# your network.
	config.vm.network "public_network"

## Preparing machine for docker

install the backported kernel

    sudo apt-get update
    sudo apt-get -y install linux-image-generic-lts-raring linux-headers-generic-lts-raring

reboot

    sudo reboot

Add the repository to your APT sources

    sudo sh -c "echo deb https://get.docker.com/ubuntu docker main > /etc/apt/sources.list.d/docker.list"

Then import the repository key

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9

Install docker

    sudo apt-get update
    sudo apt-get install -y lxc-docker

Install some basics    
   
    sudo apt-get -y install git vim


### Creating docker image

Build a docker environment for running build.  Create a "Dockerfile" and place this content inside:

    FROM ubuntu:13.10
    MAINTAINER Chris Parnin, chris.parnin@ncsu.edu
    
    RUN echo "deb http://archive.ubuntu.com/ubuntu saucy main universe" > /etc/apt/sources.list
    RUN apt-get -y update && apt-get -y upgrade
    RUN apt-get install -y wget openjdk-7-jdk curl unzip

    RUN apt-get -y install git
    RUN apt-get -y install maven
    RUN apt-get -y install libblas*
    RUN ldconfig /usr/local/cuda/lib64
    
    ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64

Build the docker image

    sudo docker build -t ncsu/buildserver .
    
Test the image

    sudo docker images
    sudo docker run -it ncsu/buildserver mvn --versio
