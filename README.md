# IoT-Based Room Occupancy Monitoring and Remote Heating Control-System
The final project for "Internet of Things 52104S-3006" course at University of Oulu is presented here. In this project, occupancy count is estimated at edge. The user can monitor the sensors readings and calculated numbers of residents in real-time.

## Devices
* Raspberry Pi 3
* Raspberry Pi Pico
* Raspberry Pi camera module
* RS-485 CAN HAT for Raspberry Pi 3
* Pico- 2CH-RS485 Hat for raspberry pi pico
* compatible 4G/5G USB dongle with Raspbian OS
## Dependencies

* LabVIEW 2024 Q3 with the following packages and toolkits:
  * [Hobbyist version 2024 Q3](https://www.ni.com/en/support/downloads/tools-network/download.labview-hobbyist-toolkit.html?srsltid=AfmBOootf6Z0s5w6TBujDSQhxd81qou_WAUbxogZ993ae2nn6GK9WtHZ#569351)
  * NI-VISA
  * [MQTT Broker 4.0.2.17 by G Open Source Project for LabVIEW](https://www.vipm.io/package/labview_open_source_project_lib_mqtt_broker/)
  * MQTT Client 4.0.0.15 by G Open Source Project for LabVIEW
  * MQTT Connection 4.0.0.25 by G Open Source Project for LabVIEW
  * MQTT Control Packets 3.1.4.10 by G Open Source Project for LabVIEW
  * MQTT LocalQueue Connection 4.0.0.5 by G Open Source Project for LabVIEW
  * MQTT TCP Connection 4.0.0.7 by G Open Source Project for LabVIEW
  * MQTT Websockets Connection 4.0.0.10 by G Open Source Project for LabVIEW
* [Raspbian with LabVIEW pre installed](https://github.com/LVMakerHub/LINX/wiki/installing-an-image-with-LabVIEW-pre-installed)

## Raspbian Configuration for Raspberry Pi 3:
* Install [Raspberry Pi Imager](https://www.raspberrypi.com/software/) or other similar programs such as [Rufus](https://rufus.ie/en/) to fomat Raspberry Pi
* Install a bootable Raspbian OS with the appropriate LabVIEW version preinstalled
* Reinsert the memory to PC and look for a file named cmdline open it with a text editor such as Notepad plus and add the following at the end of string. It gives an static IP to the Raspberry Pi:
  ip=192.168.1.99::192.168.1.50:255.255.255.0:rpi:eth0:off
  in which 192.168.1.99 is the configured static IP address for raspberry Pi and 192.168.1.50 is the IP address which Raspberry Pi expects as a connecting device (for example your PC) as gateway

**Note**: if you are using Compute Raspberry Pi models, you will need rpiboot tool which is installed with Raspberry Pi imager
* Plug in the Raspberry Pi with its memory and connect it to the PC with Ethernet cable
* Configure your static IP in your PC same as 192.168.1.50
* use this command  `sudo nmtui` to remove the extra Ethernet 0 interface and edit the configuration for the other one:
  * Tick **Automatically connect** and also **Never use this nethwork as default router** options
* update the installed packages with `sudo apt-get update` and `sudo apt-get upgrade`
* Change the cmdline file again ` sudo nano /boot/firmware/cmdline.txt` and remove the command that was added previously
* Install camera stack with `sudo apt install -y libcamera-apps libcamera-dev`
* Install Opencv with `sudo apt install -y libopencv-dev`
* [Download the MobileNet-SSD model](https://github.com/nikmart/pi-object-detection)
