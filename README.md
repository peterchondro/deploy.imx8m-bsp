# deploy.imx8m-bsp
Preparing relevant environment to built on i.MX8M Plus BSP

# Prerequisite
1. OS Ubuntu >=20.04 LTS with HDD >= 500GB and Python3.7.
2. i.MX8M Plus BSP.
3. Micro SD Card >= 16GB.
4. Other Supporting Cables.
5. Internet Connection.

# Setup Instructions
Please follow these steps to setup your Linux environment to support i.MX8M Plus BSP:
### Base Dependencies Installation
```
$ sudo apt update
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio plocate vim
$ sudo apt-get install libegl1-mesa libsdl1.2-dev xterm curl repo liblz4-tool
$ sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev wget libbz2-dev
$ sudo add-apt-repository ppa:deadsnakes/ppa
$ sudo apt-get install python-is-python3.7 python3.7-pexpect python3.7-git python3.7-jinja2 xz-utils debianutils iputils-ping python3.7-pip
$ sudo apt-get install python3.7-distutils 
$ sudo apt-get install pylint3
```
[Optional] Should your Ubuntu system does not recognize pylint3, you may install pylint and create a symlink to replace pylint3 with pylint:
  ```
  $ sudo apt-get install pylint
  $ which pylint
  $ cd /usr/bin/
  $ sudo ln -s pylint pylint3
  ```
[Optional] Should your Ubuntu system defaultly use Python >= 3.7, you need to establish Python 3.7 as update alternatives:
  ```
  $ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 1
  ```
### Java Development Kit Installation
This setup recommends the installation of Java SE Development Kit 8u191.
Please visit https://www.oracle.com/tw/java/technologies/javase/javase8-archive-downloads.html
Find "Java SE Development Kit 8u191" and download the appropiate version depending on your system.

[Optional] Should you need to verify your system information, you may do the following:
  ```
  $ uname -a
  ```
Install JDK 8u191 as follows:
```
sudo mkdir /usr/java
locate jdk-8u191-linux-<system_version>.tar.gz
cd "path to jdk-8u191-linux-<system_version>.tar.gz"
sudo tar xf jdk-8u191-linux-<system_version>.tar.gz -C /usr/java
```
Establish JDK 8u191 installation directory to system's environment:
```
$ sudo vim /etc/profile
  -> export JAVA_HOME=/usr/java/jdk1.8.0_191
  -> export PATH=/usr/java/jdk1.8.0_191/bin:$PATH
$ source /etc/profile
$ sudo update-alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_191/bin/java 300
$ sudo update-alternatives --install /usr/bin/javac javac /usr/java/jdk1.8.0_191/bin/javac 300
$ sudo update-alternatives --install /usr/bin/javaws javaws /usr/java/jdk1.8.0_191/bin/javaws 300
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
$ sudo update-alternatives --config javaws
```
[Optional] To verify your JDK installation, please check if your Java shows a version "1.8.0_191" as:
  ```
  $ java -version
  ```
### Setup Manifest for i.MX BSP Release
Please setup your GitHub account in your terminal (if you haven't done):
```
$ git config --global user.name "user name"
$ git config --global user.email user.name@your-group.com
```
Install Repository Tool in your machine:
```
$ cd ~
$ mkdir ~/bin
$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
$ export PATH=~/bin:$PATH
```
Execute the RT in your machine:
```
$ mkdir "your_release_name"
$ cd "your_release_name"
$ repo init -u https://github.com/nxp-imx/imx-manifest -b imx-linux-hardknott [ -m <release manifest>]
$ repo sync
```
[Optional] To find your release manifest, please visit https://github.com/nxp-imx/imx-manifest/tree/imx-linux-hardknott or for this setup, you can use imx-6.1.1-1.0.1, which is:
  ```
  $ repo init -u https://github.com/nxp-imx/imx-manifest -b imx-linux-langdale -m imx-6.1.1-1.0.1.xml
  ```
Please setup the build folder for a BSP release:
```
$ EULA=1 [MACHINE=<machine>] [DISTRO=fsl-imx-<backend>] source ./imx-setup-release.sh -b bld-<backend>
```
[Optional] Please see imx-manifest/README-<demo> in https://github.com/nxp-imx/imx-manifest/tree/imx-linux-hardknott for further instructions or for this setup, you can use imx8mpevk or i.MX 8M Plus as:
  ```
  $ EULA=1 MACHINE=imx8mpevk DISTRO=fsl-imx-xwayland source ./imx-setup-release.sh -b buildxwayland
  ```
Update local config setting:
```
$ sudo vim ~/"your_release_name/buildxwayland/conf/local.conf
  -> IMAGE_INSTALL_append = "packagegroup-imx-ml"
```
Build BSP release:
```
$ bitbake imx-image-full
```
