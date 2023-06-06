# deploy.imx8m-bsp
Preparing relevant environment to built on i.MX8M Plus BSP

# Prerequisite
1. OS Ubuntu 18.04 LTS with HDD >= 500GB.
2. i.MX8M Plus BSP.
3. Micro SD Card >= 16GB.
4. Other Supporting Cables.
5. Internet Connection.

It is important to pay attention on OS Ubuntu 18.04 LTS and your HDD/SSD storage capacity.
Using Ubuntu >18 will result in failure to build the BSP release due to incompactibilities in its kernel (no change in BSP release >=6).
Using HDD/SSD <500GB may result in failure to build the BSP release as it requires a lot of space.

# Setup Instructions
Please follow these steps to setup your Linux environment to support i.MX8M Plus BSP:
### Base Dependencies Installation
```
$ sudo apt update
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio mlocate vim
$ sudo apt-get install libegl1-mesa libsdl1.2-dev xterm curl liblz4-tool zstd
$ sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev wget libbz2-dev
$ sudo apt-get install python-is-python3 python3-pexpect python3-git python3-jinja2 xz-utils debianutils iputils-ping python3-pip
$ sudo apt-get install python3-distutils 
$ sudo apt-get install pylint3
```
[Optional] Should your Ubuntu system does not recognize pylint3, you may install pylint and create a symlink to replace pylint3 with pylint:
  ```
  $ sudo apt-get install pylint
  $ which pylint
  $ cd /usr/bin/
  $ sudo ln -s pylint pylint3
  ```
[Optional] Should your Ubuntu system defaultly use Python!=3.8, you need to establish Python 3.8 as update alternatives:
  ```
  $ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1
  $ sudo update-alternatives --config python3
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
$ git config --global user.name "your user name"
$ git config --global user.email "your user's email address"
```
Install Repository Tool in your machine:
```
$ cd ~
$ mkdir ~/bin
$ curl http://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
$ export PATH=~/bin:$PATH
```
Execute the RT in your machine:
```
$ mkdir "your_project_name"
$ cd "your_project_name"
$ repo init -u https://github.com/nxp-imx/imx-manifest -b imx-linux-langdale [ -m <release manifest>]
$ repo sync
```
[Recommendation] To find your release manifest, please visit https://github.com/nxp-imx/imx-manifest/tree/imx-linux-langdale or for this setup, you can use imx-5.10.72-2.2.3, which is:
  ```
  $ repo init -u https://github.com/nxp-imx/imx-manifest  -b imx-linux-hardknott -m imx-5.10.72-2.2.3.xml
  ```
Please setup the build folder for a BSP release:
```
$ EULA=1 [MACHINE=<machine>] [DISTRO=fsl-imx-<backend>] source ./imx-setup-release.sh -b bld-<backend>
```
[Recommendation] Please see imx-manifest/README-<demo> in https://github.com/nxp-imx/imx-manifest/tree/imx-linux-langdale for further instructions or for this setup, you can use imx8mpevk or i.MX 8M Plus as:
  ```
  $ EULA=1 MACHINE=imx8mpevk DISTRO=fsl-imx-xwayland source ./imx-setup-release.sh -b buildxwayland
  ```
Prior to building BSP Release, please modify the local configuration file:
```
$ sudo vim "your_project_name"/buildxwayland/conf/local.conf
  -> IMAGE_INSTALL_append = "packagegroup-imx-ml"
```
[Optional] To avoid CPU exhaustion during build process for the BSP release, you are strongly advised to limit the number of threads. You can check how many CPU threads you have with:
  ```
  $ lscpu | egrep 'CPU\(s\)'
  ```
  Take that number of CPU thread and you may take just 50%. For example, if you have 12 threads, you may use 6 and do as follows:
  ```
  $ sudo vim "your_project_name"/buildxwayland/conf/local.conf
    -> BB_NUMBER_THREADS = "6"
    -> PARALLEL_MAKE = "-j 6"
  ```
Build BSP release:
```
$ bitbake imx-image-full
```
If you encounter this error exit 1, you can just simply run the bitbake again and it will resume from the point you stop.
You may need to repeat resuming bitbake for several time depending on your hardware situation.

[Optional] If you encountered an occuring error, even though you had been resuming bitbake for several time.
You could do the following:
  ```
  $ bitbake -f -c cleanall "name_of_the_package_with_the_same_error"
  $ bitbake imx-image-full
  ```
Build SDK:
```
$ bitbake imx-image-full -c populate_sdk
$ ./"your_project_name"/buildxwayland/tmp/deploy/sdk/fsl-imx-xwayland-glibc-x86_64-imx-image-full-aarch64-imx8mpevk-toolchain-5.4-zeus.sh
```
