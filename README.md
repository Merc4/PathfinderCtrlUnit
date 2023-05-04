# Control Unit Deployment Guide

This guide is designed to run using at Northumbria University’s eduroam network, if you can have a LAN connection to your Pi and don’t require remote access to the printer setup, you can start at **Step 3. Docker Installation**.

# **Step 1. Setup the Raspberry Pi with Internet access.**

Install the latest Raspberry Pi OS (previously called ***Raspbian***) with desktop environment.

Using the latest Eduroam CAT tool to config your University Wi-Fi Access. (In most cases, change the network setting to NetworkManager before run the CAT tool is advised)

```bash
#Change the Network Config to NetworkManager
sudo raspi-confi
# Advanced Options > Network Config > NetworkManager > Reboot

#Setup Eduroam at Northumbria University using CAT 
#NU's CAT won't work under root privilege. 
sudo python /boot/eduroam-deploy.py
```

# Step 2. Setup Remote Access

We setup software-defined mesh virtual private network for easy access, in this deployment we using Tailscale.

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

This step is optional, it provide easy SSH access to the pi via Internet.

# **Step 3. Docker Installation**

Before install Docker, it's advisable to update the device’s software packages, in this example it was running with root privilege, otherwise you might need to add sudo before each command.

```bash
apt-get update
apt-get upgrade
```

After update, Install prerequisites

```bash
apt-get install \
apt-transport-https \
ca-certificates \
curl \
gnupg2 \
software-properties-common
```

After this, reboot the device before continue.

```bash
reboot
```

After device is rebooted, run Docker Install Script.

In this example it was running with root privilege, otherwise you might need to add sudo before each command.

```bash
curl -sSL https://get.docker.com | sh
```

Before proceed, check if Docker is up and running.

```bash
sudo docker ps
```

If success, the terminal should return following information.

```bash
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

We can then proceed to install Docker-Compose 

```bash
sudo apt-get install docker-compose
```

# Step 4. Setup Working Directory and docker-compose Configuration

First create directory for octoprint.

```bash
mkdir octoprint
```

Then, create a directory for the printer

```bash
cd octoprint
mkdir pathfinder
cd pathfinder
```

It is advisable to get the path to this directory, using pwd command.

```bash
pwd
# pwd command will return with the path of the current working directory.
# It should looks like:
# root@pathfinder:~/octoprint/pathfinder#
```

Return to octoprint directory

```bash
cd ..
# "cd .." command is to return to the parent directory
```

Before proceed to the next step, you need to identify the printer’s path. There are many method to achieve this, in this example, we use “dmesg” command to identify the printer.

First remove the USB Cable from the 3D-Printer

Then run “dmesg”

```bash
dmesg
```

In the return messages, there should be one looks like:

```bash
[ 2325.054125] usb 1-1.3: USB disconnect, device number 4
```

Reconnect the USB to your printer

Then run “dmesg” again

```bash
dmesg
```

In the return message, there should be one looks like:

```bash
[ 2362.728387] usb 1-1.3: new full-speed USB device number 5 using dwc_otg
[ 2362.861445] usb 1-1.3: New USB device found, idVendor=1eaf, idProduct=0004, bcdDevice= 2.00
[ 2362.861469] usb 1-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[ 2362.861477] usb 1-1.3: Product: Maple
[ 2362.861483] usb 1-1.3: Manufacturer: LeafLabs
[ 2362.862721] cdc_acm 1-1.3:1.0: ttyACM0: USB ACM device

```

Take note of the “ttyACM0”, this is the printer’s path, with this you can now proceed to the next step.

Create a docker-compose configuration file in /octopirnt directory

```bash
nano docker-compose.yml
```

```bash
version: '2.2'

services:
  octoprint_pathfinder:
    restart: unless-stopped
    image: octoprint/octoprint
    ports:
      - 4000:5000
    devices:
      - /dev/ttyACM0:/dev/ttyACM0
    volumes:
     - /root/octoprint/pathfinder:/home/octoprint
```

Save the file and proceed to next step.

# Step 5. Use docker-compose to start the octoprint instance

Run docker-compose up -d, the -d flag is to run the command detached, this will run it in the background and not hurting all the logging during the process.

```bash
docker-compose up -d
```

Get container id and then check logs if any failures has occur.

```bash
docker ps
```

#docker logs <containerid>

```bash
docker logs
```

Grant full permissions to printer directory

```bash
chmod -R 777 pathfinder
```

Restart the docker container

#docker restart <containerid>

```bash
docker restart
```

**Misc**

The control unit that was deployed at Northumbria University SmartDesignLab has a 2.2 inch display, a 6 keys keyboard, and an IR receiver/ transmitter, the following steps are for this setup only, ignore if it does not apply to you.

Install Adafruit 2.2" PiTFT HAT - 320x240 Display Install Python Script

```bash
cd ~
sudo apt-get update
sudo apt-get install -y git python3-pip
sudo pip3 install --upgrade adafruit-python-shell click
git clone https://github.com/adafruit/Raspberry-Pi-Installer-Scripts.git
cd Raspberry-Pi-Installer-Scripts
```

“Console Mode” Install Commands

```bash
sudo python3 adafruit-pitft.py --display=22 --rotation=270 --install-type=console
```