# Drone Pilot

 Pilot software (running on companion computers) for the flight controller autopilot's. It can control and fly several flight controllers, including Pixhawk's, APM's and MultiWii's.

## Current scripts:


### MultiWii scripts:

* mw-joystick.py -> Send joystick commands via UDP from a ground-station running Matlab to a FC running MultiWii software.

[![Multiwii joystick (naze32)](http://img.youtube.com/vi/XyyfGp-IomE/0.jpg)](http://www.youtube.com/watch?v=XyyfGp-IomE)

* mw-hover-controller.py -> Calculate commands to make a Multiwii multicopter hover over a specified x,y,z coordinate.



## Supported flight controllers:

* Multiwii boards (using MSP)

* Pixhawk / PX4 / APM (using mavlink & drone kit)



## Supported companion computers: 

* Raspberry Pi

* oDroid U3 

Note: Code is in python, so, any linux computer would be able tu run it.



## How to:

* Raspberry Pi (Rasbian)

<ol start="1"><li> Update your Pi... </li></ol>
```
sudo apt-get update
sudo apt-get upgrade
```

<ol start="2"><li> Next this one: </li></ol>
```
sudo apt-get install python-dev
```

<ol start="3"><li> More utilities: </li></ol>
```
sudo apt-get install screen python-wxgtk2.8 python-matplotlib python-opencv python-pip python-numpy python-serial
```

<ol start="4"><li> Then mavlink and mavproxy:</li></ol>
```
sudo pip install pymavlink
sudo pip install mavproxy
sudo pip install droneapi
```

<ol start="5"><li> Clone this repository:</li></ol>
```
git clone https://github.com/alduxvm/DronePilot.git
```

#### DroneAPI

You have to load the module inside mavproxy.py, to do that just type:

```
module load droneapi.module.api

```

Now you're ready to run .py files with droneapi/dronekit! 

#### Simulation

If you're doing tests with simulation, don't forget to turn off the parameter checking inside the mavproxy console, so that you can arm the vehicle:

```
param set ARMING_CHECK 0

```

All set!! go do tests!!

## Caution

This code is still under heavy development, everyday I add and remove stuff. Proceed with caution.

## Social networks:

You can follow us in this URL's:

* [Aldux.net](http://aldux.net/)
* [Aldux.net Facebook](https://www.facebook.com/AlduxNet)

## Assumptions:

For the 'pix-' scripts the following assumptions must be noted:

* This script assumes that the pixhawk is connected to the raspberry pi via the serial port (/dev/ttyAMA0)
* In our setup, telemetry port 2 is configured at 115200 on the pixhawk
* rpi camera connected (for computer vision) 

## Utilities:

#### ssh:

/etc/ssh/ssh_config
```
ServerAliveInterval 5
ClientAliveInterval 5
```

#### ad-hoc:

/etc/network/interfaces
```
auto lo
iface lo inet loopback
iface eth0 inet dhcp
 
auto wlan0
iface wlan0 inet static
  address 192.168.1.1
  netmask 255.255.255.0
  wireless-channel 1
  wireless-essid RPiAdHocNetwork
  wireless-mode ad-hoc
```

```
sudo ifdown wlan0
sudo ifup wlan0
```

```
sudo apt-get install dhcp3-server
/etc/dhcp/dhcpd.conf
```

```
ddns-update-style interim;
default-lease-time 600;
max-lease-time 7200;
authoritative;
log-facility local7;
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.5 192.168.1.150;
}
```

In case of failure starting DHCP:
```
sudo service ifplugd stop
```

#### Video

```
sudo modprobe -v bcm2835-v4l2
git clone https://github.com/mpromonet/h264_v4l2_rtspserver.git
cd h264_v4l2_rtspserver
cmake .
make install
h264_v4l2_rtspserver -H <height> -W <width> -F <fps>
```

To start:
```
rtsp://192.168.1.1:8554/unicast
```

#### updateDronePilot.sh 

```
#!/bin/bash

echo "Deleting previous directory..."
rm -rf DronePilot/
echo "Done."
echo "Cloning DronePilot repository: "
git clone https://github.com/alduxvm/DronePilot.git
echo "Copying mavinit.scr to proper directory..."
mkdir DronePilot/TestQuad
mv DronePilot/mavinit.scr DronePilot/TestQuad/
echo "Done."
echo "Ready to do: mavproxy.py --master=/dev/ttyAMA0 --baudrate 115200 --aircraft TestQuad"
echo "Dont forget to go inside the DronePilot directory."
```

### Low latency video

* Same DHCP configuration as before

* Install "hostapd" 
```
sudo apt-get install hostapd netcat-traditional

```

* Interfaces:
```
auto wlan0
iface wlan0 inet static
  address 85.85.85.1
  netmask 255.255.255.0
```

* /etc/hostapd/hostapd.conf 
```
interface=wlan0
ssid=HDFPV
hw_mode=g
channel=6
auth_algs=1
wmm_enabled=0
```

* /etc/default/hostapd
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

* On rpi, create fifo file:
```
mkfifo fifo.500 
```

* On client (mac):
```
brew install netcat
brew install mplayer
```

* To start (mac):
```
netcat -l -p 5000 | mplayer -fps 60 -cache 1024 -
```

* To start (rpi):
```
cat fifo.500 | nc.traditional 85.85.85.6 5000 &
/opt/vc/bin/raspivid -o fifo.500 -t 0 -b 50000
```
