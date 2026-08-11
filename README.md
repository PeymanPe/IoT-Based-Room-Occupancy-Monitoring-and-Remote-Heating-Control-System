# IoT-Based Room Occupancy Monitoring and Remote Heating Control System

The final project for the **Internet of Things (52104S-3006)** course at the **University of Oulu** is presented here. In this project, room occupancy is estimated at the edge using a Raspberry Pi and camera-based person detection. Users can monitor sensor readings and the estimated number of occupants in real time and remotely control the heating system.

## Devices

* Raspberry Pi 3
* Raspberry Pi Pico W
* Raspberry Pi Camera Module
* RS-485 CAN HAT for Raspberry Pi 3
* Pico-2CH-RS485 HAT for Raspberry Pi Pico W
* Compatible 4G/5G USB dongle with Raspberry Pi OS
* Magnetic door switch
* LED and 10 kΩ resistor
* DB9 cable
* BMP280 sensor
* HC-SR04 ultrasonic sensor

## Dependencies

### LabVIEW

LabVIEW 2024 Q3 with the following packages and toolkits:

* [LabVIEW Hobbyist Toolkit 2024 Q3](https://www.ni.com/en/support/downloads/tools-network/download.labview-hobbyist-toolkit.html)
* NI-VISA
* [MQTT Broker 4.0.2.17 by G Open Source Project for LabVIEW](https://www.vipm.io/package/labview_open_source_project_lib_mqtt_broker/)
* MQTT Client 4.0.0.15 by G Open Source Project for LabVIEW
* MQTT Connection 4.0.0.25 by G Open Source Project for LabVIEW
* MQTT Control Packets 3.1.4.10 by G Open Source Project for LabVIEW
* MQTT LocalQueue Connection 4.0.0.5 by G Open Source Project for LabVIEW
* MQTT TCP Connection 4.0.0.7 by G Open Source Project for LabVIEW
* MQTT WebSockets Connection 4.0.0.10 by G Open Source Project for LabVIEW

### Raspberry Pi Image

* [Raspberry Pi OS with LabVIEW Preinstalled](https://github.com/LVMakerHub/LINX/wiki/installing-an-image-with-LabVIEW-pre-installed)

## Repository Structure

```text
.
├── Pico/
│   ├── main.py
│   └── Test/
├── RPI/
│   ├── test/
│   └── udp_person_counter.cpp
├── Services/
│   ├── udp_person_counter.service
│   ├── reverse-ssh-tunnel.service
│   └── hivemq.service
└── Docs/
```

## Raspberry Pi 3 Configuration

### Operating System Installation

1. Install [Raspberry Pi Imager](https://www.raspberrypi.com/software/) or a similar tool such as [Rufus](https://rufus.ie/en/).
2. Flash Raspberry Pi OS with the appropriate LabVIEW version preinstalled.
3. Reinsert the SD card into your PC.
4. Open the `cmdline.txt` file and append the following configuration to assign a temporary static IP:

```bash
ip=192.168.1.99::192.168.1.50:255.255.255.0:rpi:eth0:off
```

Where:

* `192.168.1.99` = Raspberry Pi IP address
* `192.168.1.50` = Host PC IP address (gateway)

> **Note:** If you are using Raspberry Pi Compute Modules, you may need the `rpiboot` utility installed through Raspberry Pi Imager.

### Initial Setup

1. Insert the SD card into the Raspberry Pi.
2. Connect the Raspberry Pi to your PC using an Ethernet cable.
3. Configure your PC with the static IP address:

```text
192.168.1.50
```

4. Open NetworkManager:

```bash
sudo nmtui
```

5. Remove the extra Ethernet interface and edit the remaining one:

   * Enable **Automatically connect**
   * Enable **Never use this network as default router**

6. Update installed packages:

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Remove Temporary Static IP

Edit:

```bash
sudo nano /boot/firmware/cmdline.txt
```

Remove the IP configuration added earlier.

### Install Required Packages

Install the Raspberry Pi camera stack:

```bash
sudo apt install -y libcamera-apps libcamera-dev
```

Install OpenCV:

```bash
sudo apt install -y libopencv-dev
```

Download MobileNet-SSD:

* https://github.com/nikmart/pi-object-detection

Compile the occupancy detection application:

```bash
g++ -o udp_person_counter udp_person_counter.cpp $(pkg-config --cflags --libs opencv4)
```

### Configure Startup Services

Use:

* [udp_person_counter.service](Services/udp_person_counter.service)
* [reverse-ssh-tunnel.service](Services/reverse-ssh-tunnel.service)

Enable services:

```bash
sudo systemctl daemon-reload

sudo systemctl enable udp_person_counter.service
sudo systemctl enable reverse-ssh-tunnel.service

sudo systemctl start udp_person_counter.service
sudo systemctl start reverse-ssh-tunnel.service
```

### Deploy LabVIEW Application

1. Open the IoT LabVIEW project.
2. Right-click **Build Specifications**.
3. Select:

```text
New → Real-Time Application
```

4. Under **Source Files**, select:

```text
main_v6.vi
```

as the Startup VI.

5. Build and deploy the application.

## Raspberry Pi Pico W Configuration

1. Install MicroPython.
2. Create and upload:

```text
Pico/main.py
```

3. Configure Wi-Fi credentials and communication settings as required.

## Hardware Setup

### Raspberry Pi Pico W Connections

| Device          | Pico Pin |
| --------------- | -------- |
| HC-SR04 Echo    | GPIO 4   |
| HC-SR04 Trigger | GPIO 5   |
| HC-SR04 VCC     | Pin 40   |
| HC-SR04 GND     | Pin 38   |
| BMP280 VCC      | Pin 36   |
| BMP280 GND      | Pin 33   |
| BMP280 SCL      | Pin 27   |
| BMP280 SDA      | Pin 26   |
| Magnetic Switch | Pin 38   |
| Magnetic Switch | Pin 19   |

For RS-485 communication:

| RS485 HAT | DB9   |
| --------- | ----- |
| A         | Pin 1 |
| B         | Pin 2 |
| G         | Pin 5 |

### Raspberry Pi 3 Connections

| Raspberry Pi Pin | Function                   |
| ---------------- | -------------------------- |
| Pin 6            | DB9 Pin 5                  |
| Pin 13 and Pin 9 | LED and resistor in series |

## Private Server Configuration

Ubuntu 22.04 LTS was used as the private server.

### Install HiveMQ Broker

Update packages:

```bash
sudo apt update
sudo apt install git
```

Create a workspace:

```bash
mkdir broker
cd broker
```

Download HiveMQ Community Edition:

* https://github.com/hivemq/hivemq-community-edition/releases

Example:

```bash
curl -L -o hivemq-ce-2024.9.zip https://github.com/hivemq/hivemq-community-edition/releases/download/2024.9/hivemq-ce-2024.9.zip
```

Install unzip:

```bash
sudo apt update
sudo apt install unzip
```

Extract:

```bash
unzip hivemq-ce-2024.9.zip
```

Install Java:

* https://www.oracle.com/java/technologies/javase/jdk23-archive-downloads.html

Install JDK:

```bash
sudo dpkg -i jdk-23_linux-x64_bin.deb
```

Start HiveMQ:

```bash
./hivemq-ce-2024.9/bin/run.sh
```

### Create a Systemd Service

Create:

```bash
sudo nano /etc/systemd/system/hivemq.service
```

Use the configuration provided in:

* [hivemq.service](Services/hivemq.service)

Enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable hivemq.service
sudo systemctl start hivemq.service
```

## Throughput Testing

### RS-485 Throughput Test

Use:

* `Pico/Test/test.py`
* `RPI/test/test.vi`

### Wi-Fi UDP Throughput Test

Use:

* `Pico/Test/test2.py`
* `RPI/test/test2.vi`

Update:

* `SSID`
* `PASSWORD`
* `UDP_TARGET_IP`

according to your network configuration.

To determine the Raspberry Pi IP address:

```bash
ifconfig
```

## License

The original source code and documentation contained in this repository are
licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

This project depends on third-party software, libraries, models, and tools,
which remain subject to their respective licenses. Such components are not
covered by the MIT License provided in this repository.

## Third-Party Components

This project uses or depends on the following third-party software:

- OpenCV
- HiveMQ Community Edition
- Raspberry Pi OS
- MicroPython
- LabVIEW and associated toolkits
- MobileNet-SSD model

Please refer to the respective projects for their license terms.
