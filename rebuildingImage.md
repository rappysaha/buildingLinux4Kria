# Why Rebuilding is Necessary for our case

- In our secda code, we use mmap() to access the memory during dma_init(). But, the mmap() function will not work if the CONFIG_STRICT_DEVMEM is enabled in the kernel. In default settings, the CONFIG_STRICT_DEVMEM is enabled in the kernel. So, we need to disable it to use mmap() function.

- Another error case: While you are running your secda_benchmark_suite.sh and after loading bit stream your binary will be failed immediately. In this case, you need to go to dma_init() and check is it due to mmap() function. If it is due to mmap() function, then you need to rebuild the kernel with the new configuration.

# Rebuilding an Image

We download the ubuntu image for Kria board and later find out that we need to change some configurations of linux kernel. So here we will rebuild the image with the new configurations.

We are following this website: <https://adaptivesupport.amd.com/s/feed/0D54U00007wzkKPSAY>

The following steps are also applicable to 20.04 - use ```focal``` where ```jammy``` is referenced below.

## Configure the Build Environment

Before fetching the Linux kernel sources, first configure the build environment with tools such as ```git```, Aarch64 ```gcc```, and ```fakeroot```.

```
echo "deb-src http://archive.ubuntu.com/ubuntu jammy main" | sudo tee -a /etc/apt/sources.list.d/jammy.list

sudo apt-get update

sudo apt-get build-dep linux

sudo apt-get install git fakeroot libncurses-dev gcc-aarch64-linux-gnu linux-tools-common
```

## Clone the Linux Kernel Source Code

```
git clone https://git.launchpad.net/~canonical-kernel/ubuntu/+source/linux-xilinx-zynqmp/+git/jammy
```

After cloning the source code, switch to the latest tag.  To check for the latest tag, look at the Git history with

```
cd jammy

git tag
```

Then checkout the latest tag with

```
git checkout tags/ubuntu-5.13.0-1005.5
```

## Configure the Build Environment

First, set the target architecture for the Linux kernel

```
export ARCH=arm64
```

Also, export the dpkg variables required for packaging Debian packages for the target (arm64) architecture

```
export $(dpkg-architecture -aarm64)
```

 if you see following warning:

```
rappy@lolland:~/workspace/buldingLinux/jammy$ export $(dpkg-architecture -aarm64)
dpkg-architecture: warning: specified GNU system type aarch64-linux-gnu does not match CC system type x86_64-linux-gnu, try setting a correct CC environment variable
```

Then

```
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu

export CC=aarch64-linux-gnu-gcc
export CXX=aarch64-linux-gnu-g++

export $(dpkg-architecture -aarm64)
```

Finally, if compiling in a cross-compilation environment set the CROSS_COMPILE variable. We are doing cross-compilation because we are compiling in an x86_64 environment for an arm64 target.

```
export CROSS_COMPILE=aarch64-linux-gnu-
```

## Edit the Linux Kernel Configuration

First lets make a quick edit to distinguish our kernel build from the binaries installed from Canonical.  Here we just added the ```+local``` to the version in the ```changelog``` to distinguish it from the running kernel.

```
fakeroot debian/rules clean
vi debian.zynqmp/changelog
```

Then edit the kernel configuration

If the Linux kernel configuration requires modification the standard Linux kernel menuconfig can be accessed with the standard Debian editconfigs rule.  This is the place in the process where new drivers, modules, or features of the Linux kernel can be enabled. If there are no edits required this section can be skipped.

In my case I needed to disable the ```CONFIG_STRICT_DEVMEM``` option.  This can be done with going to the ```Kernel hacking``` section and ```Filter access to /dev/mem``` option. Press ```N``` to disable it. Save the configuration and exit.

```
fakeroot debian/rules clean
fakeroot debian/rules editconfigs
```

## Build the Linux Kernel

To build the Linux kernel, use the standard Debian rules system with the ```binary``` target.  The output of this process will be a series of standard ```.deb``` Debian packages.

```
fakeroot debian/rules clean
```

There are two ways to build the kernel.

First,

```
do_tools=false fakeroot debian/rules binary
```

Second, If you are making changes to the configuration that may impact modules and/or the abi, then you may need to skip those checks as shown below. I chose the second to skip the checks.

```
do_tools=false skipmodule=true skipconfig=true skipabi=true fakeroot debian/rules binary
```

The generated ```.deb``` packages are located one directory higher than the Linux kernel source directory.

## Install the Linux Kernel Packages

If the kernel was compiled on the target, installing the new kernel is simple.  First, change to the directory where the build process placed the Debian packages and install them with the ```dpkg``` tool.

```
cd .. (or to wherever the .deb packages are located)
sudo dpkg -i *.deb
```

But, I used x86_64 to compile the kernel and I need to copy the generated .deb files to the Kria board and install them there. Copy the generated .deb files to the Kria board /boot/firmware directory by enabling root access to the Kria board.

```
cd /boot/firmware
sudo dpkg -i *.deb
```

After installing the new kernel, reboot the Kria board to boot the new kernel.

```
sudo reboot
```

## Verify the New Kernel

After the Kria board has rebooted, verify the new kernel is running with the ```uname``` command.

```
uname -r
```

The output should show the new kernel version.

Check the kernel configuration with the ```zcat``` command.

```
zcat /proc/config.gz | grep CONFIG_STRICT_DEVMEM
```

The output should show the ```CONFIG_STRICT_DEVMEM is not set``` indicating the configuration change was successful.

## If Kernel gets updated automatically

These instructions are for the case when the kernel gets updated automatically.  In this case, the kernel will be updated to the latest version and the configuration changes will be lost.  In this case, the kernel will need to be rebuilt with the desired configuration changes for the new kernel version.

- If you download the kernel source code from the website, jammy, remove the old source code and start from the beginning step [**Configure the Build Environment**](#configure-the-build-environment).
