# Getting started with KRIA

This guide will help you get started with KRIA.

<https://github.com/Xilinx/Kria-PYNQ>

We target the Ubuntu Desktop 22.04 LTS image for KRIA because KRIA-PYNQ supports this version.
Download 22.04 Desktop image from

<https://ubuntu.com/download/amd#kria-k26>

Follow the instructions to install the image on the KRIA board from this link:

<https://xilinx.github.io/kria-apps-docs/kv260/2022.1/linux_boot/ubuntu_22_04/build/html/docs/sdcard.html>

# Some Tips

- Always shutdown the board properly. Do not unplug the power supply directly.
```sudo shutdown -h now```

- make sure update and upgrade the system once you login to the board. this has been described in the step 5(Boot linux) of the above link.
<https://xilinx.github.io/kria-apps-docs/kv260/2022.1/linux_boot/ubuntu_22_04/build/html/docs/sdcard.html> Upgrade packages on the system :Install the application specific repositories, Ubuntu updates, and upgrade the system (which may take 10-20 minutes to complete):

```
sudo add-apt-repository ppa:xilinx-apps --yes &&
sudo add-apt-repository ppa:ubuntu-xilinx/sdk --yes &&
sudo add-apt-repository ppa:xilinx-apps/xilinx-drivers --yes &&
sudo add-apt-repository ppa:lely/ppa --yes &&
sudo apt update --yes &&
sudo apt upgrade --yes
```

- check the firmware version.
```uname -r```

- check the configuCheck the kernel configuration with the ```zcat``` command.
```zcat /proc/config.gz | grep CONFIG_STRICT_DEVMEM```
The output should show the ```CONFIG_STRICT_DEVMEM is not set``` indicating the configuration is correct.
Otherwise, you need to rebuild the kernel, follow the **rebuildingImage.md** guide.

- check the CMA allcoation
```cat /proc/meminfo | grep -i cma``` or ```sudo dmesg | grep -i cma```
