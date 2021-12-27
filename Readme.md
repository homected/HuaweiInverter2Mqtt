![huawei logo](logo.png)
# HuaweiInverter2Mqtt
A python gateway to publish [Huawei](https://solar.huawei.com/) solar inverter data values to an MQTT broker.

## Credits
The source code for reading the data values from the solar inverter is based on the code of [ayasystems](https://github.com/ayasystems/Huawei-TCP-Modbus) and this is based on the script of [Pedestre](https://www.dropbox.com/s/9zaa1zexnr6cv60/detalles_modbus-tcp.py?dl=0).

## Introduction
I have created my own version of the script to get the data values from the [Huawei](https://solar.huawei.com/) Sun2000 solar inverter to adapt it to my [HomeAssistant](https://www.home-assistant.io/) based home automation system. 

The main idea is to be able to display data of interest from the solar inverter such as consumption and production data, which can be stored in an [InfluxDB](https://www.influxdata.com/) database and can be plotted in [Grafana](https://grafana.com/).

To do this here I will show the necessary configuration to achieve this by means of a [Raspberry Pi](https://www.raspberrypi.org/) that runs the code in Python and is responsible for sending the data to a MQTT server through which [HomeAssistant](https://www.home-assistant.io/) will be able to access.

## Equipment used

This code is checked with the following hardware:

- [Huawei SUN2000-5KTL-L1](https://solar.huawei.com/en-GB/download?p=%2f-%2fmedia%2fSolar%2fattachment%2fpdf%2feu%2fdatasheet%2fSUN2000-2-6KTL-L1.pdf)
- [Raspberry Pi](https://www.raspberrypi.org/) 1 Model B

## Installation

### Raspberry Pi

1. First, update your distribution.

   ```sh
   sudo apt update
   sudo apt upgrade
   ```
   
2. Install Python pip and some dependencies.

   ```sh
   sudo apt install python3-pip
   sudo pip install -U pymodbus
   pip3 install paho-mqtt
   ```
 
3. Install git and clone the repository.

   ```sh
   sudo apt install git
   cd~
   git clone https://github.com/homected/HuaweiInverter2Mqtt.git huaweiInverter2mqtt
   ```

4. Check USB device for Modbus-RTU is detected.

   ```sh
   usb-devices
   ```

4. Find serial port for USB device for Modbus-RTU with this command, first without the adapter connected and then with the adapter connected.

   ```sh
   ls /dev/tty
   ```

## Configuration

1. Set your own configuration parameters editing the program file.

   ```sh
   cd huaweiInverter2mqtt
   sudo nano huaweiInverter2mqtt.py
   ```

	You have to replace the text between quotes with the correct values for your configuration:
  
  	- **COM_PORT**: Something like /dev/ttyUSB0 or COM1 or /dev/serial/by-id/usb-Prolific_Technology_Inc._USB-Serial_Controller-if00-port0;
  	- **MQTT_Host**: The IP Address of the MQTT broker;
  	- **MQTT_Port**: Port of the MQTT broker, for example 1883;
  	- **MQTT_User**: Username to authenticate into the MQTT broker;
  	- **MQTT_Password**: Password to authenticate into the MQTT broker;
  	- **MQTT_Topic**: The topic of the MQTT broker where publish the data;
  	- **MQTT_QoS**: Quality of Service level (0, 1 or 2);
  	- **MQTT_Retain**: True for retain MQTT messages of False for not retain;

  	Save and close the file with: Control+O, Enter, Control+X
  
  
2. Optionally, you can set this program runs automatically when the raspberry boots with these commands:

   ```sh
   sudo cp rfxcom2mqtt.service /lib/systemd/system/rfxcom2mqtt.service
   sudo chmod 644 /lib/systemd/system/rfxcom2mqtt.service
   sudo systemctl daemon-reload
   sudo systemctl enable rfxcom2mqtt.service
   ```

## Run

1. For start manually the program you can use this command:

   ```sh
   python3 rfxcom2mqtt.py
   ```

	If you set the program to run after boots, reboot the raspberry with this command:

   ```sh
   sudo reboot
   ```
   
	After boot, you can control the process with these commands:

   ```sh
   sudo systemctl start rfxcom2mqtt
   sudo systemctl status rfxcom2mqtt
   sudo systemctl stop rfxcom2mqtt
   ```

## Usage

The values of the energy monitor will be published under the topic set in **MQTT_Topic** inside the program file and these are the published endpoints:

- **Monitor temperature**: *MQTT_Topic*/CurrentCost/Temperature
- **Total power**: *MQTT_Topic*/CurrentCost/Power/Total/chX, where X is the channel number (1, 2 or 3).
- **Appliance power**: *MQTT_Topic*/CurrentCost/Power/ApplianceY/ChX, where Y is the appliance number (1 to 9) and X is the channel number (1, 2 or 3).
- **Impulse sensor**: *MQTT_Topic*/CurrentCost/Meter/SensorY, where Y is the impulse meter sensor number (1 to 9).

### Home Assistant

To get the values of the energy monitor in Home Assistant, enter a sensor entry for each monitored value in your configuration.yaml file:

   ```yaml
   sensor:
     - platform: mqtt
       state_topic: "MQTT_Topic/CurrentCost/Temperature"
       name: "currentcost_temperature"
       unit_of_measurement: "°C"
       value_template: '{{ value_json.state }}'

     - platform: mqtt
       state_topic: "MQTT_Topic/CurrentCost/Power/Total/Ch1"
       name: "currentcost_totalpower"
       unit_of_measurement: "W"
       value_template: '{{ value_json.state }}'
    
     - platform: mqtt
       state_topic: "MQTT_Topic/CurrentCost/Power/Appliance1/Ch1"
       name: "currentcost_power1"
       unit_of_measurement: "W"
       value_template: '{{ value_json.state }}'
   ```
