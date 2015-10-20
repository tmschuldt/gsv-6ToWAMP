#gsv-6ToWAMP
## GSV-6CPU Modul to WAMP
This shows how to hook up an Raspberry Pi2 to a WAMP router and display real-time GSV-6CPU readings in a browser, as well as configure the GSV-6CPU from the browser.

## What is an GSV6/GSV6-CPU Modul
[GSV6/GSV6-CPU](http://www.me-systeme.de/docs/de/flyer/flyer_gsv6.pdf) is an Measurement Amplifier from [ME-Messsysteme GmbH](http://www.me-systeme.de/)

## project status
in progress, NOT yet an release

## Dependencies
All Dependencies are included

`TODO: replace Highcharts by an MIT licensed version`

Dependencie | License
--- | ---
[Autobahn](http://autobahn.ws/) | MIT License
[Bootstrap](http://getbootstrap.com/) | MIT License
[jQuery](https://jquery.com/) | MIT License
[Smoothie](http://smoothiecharts.org/) | MIT License
[Highstock](http://www.highcharts.com/products/highstock) | Do you want to use Highcharts/Highstock for a personal or non-profit project? Then you can use Highchcarts/Hightock for free under the  Creative Commons Attribution-NonCommercial 3.0 License

## How it works

The `serial2ws` program will open a serial port connection with your RPi or PC. It will communicate over a specific protocol with your device.

### Control

The `serial2ws` program receives and call's procedures via WAMP.

### Sense

The GSV-6CPU-Modul will send sensor values by sending his protocol-data over serial. The data can contain an measure-frame, an answer-frame or an request-frame.
The `serial2ws` will receive those frames, parse each frame, and then receive or publish WAMP events with the payload consisting of the frames to the different WAMP-topic.


## How to run

You will need to have the following installed on the Rpi to run the project. 

* Python or PyPy(used in this installation tutorial)
* Twisted
* AutobahnPython
* PySerial

### Install
all downloads a dropped to ~/downloads and all installs dropped to ~/install

	mkdir ~/downloads
	mkdir ~/install
#### PyPy
pypy comes pre-installed on a Raspberry Pi with the raspian-image 2015-05-05. It comes with version 2.2.1 and we would use the version 2.6.1.
For the update, we have to download the latest pypy-version, extract files and update paths.
	
	cd ~/downloads
	wget https://bitbucket.org/pypy/pypy/downloads/pypy-2.6.1-linux-armhf-raspbian.tar.bz2
	cd ~/install
	tar xvjf ../downloads/pypy-2.6.1-linux-armhf-raspbian.tar.bz2
	echo "export PATH=\${HOME}/install/pypy-2.6.1-linux-armhf-raspbian/bin:\${PATH}" >> ~/.profile
	echo "export LD_LIBRARY_PATH=\${HOME}/install/pypy-2.6.1-linux-armhf-raspbian/lib:\${LD_LIBRARY_PATH}" >> ~/.profile
	source ~/.profile

#### pip
	cd ~/downloads
	wget https://bootstrap.pypa.io/get-pip.py
	pypy get-pip.py
	
#### twisted
	cd ~/downloads
	wget https://pypi.python.org/packages/source/T/Twisted/Twisted-15.4.0.tar.bz2
	cd ~/install
	tar xvjf ../downloads/Twisted-15.4.0.tar.bz2

##### now we have to comment one line in the twisted-source-code
	cd Twisted-15.4.0
	nano setup.py
	line 63 comment with a # -> -> #conditionalExtensions=getExtensions(),
	strg+o
	strg+x
	pypy setup.py install

#### PySerial
	pip install pyserial
	
#### Autobahn Framework
	pip install autobahn
	
#### install usbmount, for automount usb-store
	sudo apt-get install usbmount
change usbmount config

	sudo nano /etc/usbmount/usbmount.conf
goto FS_MOUNTOPTIONS="" and change it to

	FS_MOUNTOPTIONS="-fstype=vfat,gid=users,dmask=0007,fmask=0117"
	strg+o
	strg+x
	sudo reboot
	
#### Crosbar.io (WAMP-Router)
	sudo apt-get install build-essential libssl-dev libffi-dev python-dev
	pip install crossbar
	
### checkout from github
	cd ~/
	git clone https://github.com/flashbac/gsv-6ToWAMP.git
	
### set timezone
	cd ~/
	echo "TZ='Europe/Berlin';" >> ~/.profile
	echo "export TZ" >> ~/.profile
	source ~/.profile
	
### create folder for csv-files
	mkdir messungen
	
### run crossbar server
	cd <projectname>
	crossbar start &
	check with with the browser http://<ip>:8080 -> some information have to appear there
	
### run the serial2ws.py script
	pypy serial2ws.py --baud=115200 --port=/dev/ttyAMA0
	goto http://<ip>:8000

## start crossbar and serial2ws at systemstart
copy the crossbar-(start)-script and the serial2ws-(start)-script from scripts-folder to /etc/init.d/

	cd ~/gsv-6ToWAMP/scripts
	sudo cp crossbar /etc/init.d/
	sudo cp serial2ws /etc/init.d/
	
make the script runnable and add crossbar to rc.d

	sudo chmod +x /etc/init.d/crossbar
	sudo chmod +x /etc/init.d/serial2ws
	sudo update-rc.d crossbar defaults
	sudo update-rc.d serial2ws defaults
	
## establish an Wifi Accespoint with the RPi [Source](http://elinux.org/RPI-Wireless-Hotspot)
verify that your wifi-adapter is on the [compatible list](http://elinux.org/RPI-Wireless-Hotspot)
and make sure, that you connected via eth0 ( by cable )

	sudo apt-get install hostapd udhcpd

but pay attention to the compatible adapters, i use an EW-7811Un with Realtek RTL8188CUS Chipset and this one will not work out of the box with hostapd.
For the Raspberry Pi you can use an pachted binary from the binary folder or build an [patched Version](https://github.com/lostincynicism/hostapd-rtl8188)

### use pre-compield pachted hostapd from Bianry-Folder

copy hostapd-binary from git-binary Folder

	cd /usr/sbin
	sudo mv /usr/sbin/hostapd /usr/sbin/hostapd.bak
	sudo cp gsv-6ToWAMP/binary/hostapd hostapd
	sudo chown root:root hostapd
	sudo chmod 755 hostapd


### build hostapd-rtl8188 (patched Version)
first of all you have to clone the repo

	cd ~
	git clone https://github.com/lostincynicism/hostapd-rtl8188
then install dependecies

	sudo apt-get install libnl-3-dev libnl-genl-3-dev

build hostapd-rtl8188
	
	cd hostapd-rtl8188/hostapd
	make
	
copy hostapd-binary

	cd /usr/sbin
	sudo mv /usr/sbin/hostapd /usr/sbin/hostapd.bak
	sudo cp hostapd-rtl8188/hostapd/hostapd hostapd
	sudo chown root:root hostapd
	sudo chmod 755 hostapd


### configure the Accespoint
open uDHCP-config-File with follwong command
	
	sudo nano /etc/udhcpd.conf
	
Configure DHCP. Edit the file /etc/udhcpd.conf and configure it like this
	

	start 192.168.9.2 		# range of IPs that the hostspot will give to client devices.
	end 192.168.9.20
	interface wlan0 		# The device uDHCP listens on.
	remaining yes
	!!!    opt dns 8.8.8.8 4.2.2.2 # The DNS servers client devices will use. !!!
	opt subnet 255.255.255.0
	opt router 192.168.42.1	# The Pi's IP address on wlan0 which we will set up shortly.
	opt lease 864000 		# 10 day DHCP lease time in seconds
	
disable all other active options (opt) with an #
	
save udhcpd.conf changes and exit nano with
	
	ctrl+o
	ctrl+x

open uDHCP-default-File with follwong command

	sudo nano /etc/default/udhcpd

Edit the file /etc/default/udhcpd and change the line:

	DHCPD_ENABLED="no"
to

	#DHCPD_ENABLED="no"
and save and exit with
	
	ctrl+o
	ctrl+x
	
You will need to give the Pi a static IP address with the following command:

	sudo ifconfig wlan0 192.168.9.1
	
To set this up automatically on boot, edit the file /etc/network/interfaces. open it by typing following command

	sudo nano /etc/network/interfaces
	
and replace the line "iface wlan0 inet dhcp" to (If the line "iface wlan0 inet dhcp" is not present, add the above lines to the bottom of the file.)

	iface wlan0 inet static
	  address 192.168.9.1
	  netmask 255.255.255.0

Change the lines (they probably won't all be next to each other)

	allow-hotplug wlan0
	wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
	iface wlan0 inet dhcp

to

	#allow-hotplug wlan0
	#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
	#iface wlan0 inet manual
	
Configure HostAPD. Create a WPA-secured network. To create a WPA-secured network, open the file hostapd.conf

	sudo nano /etc/hostapd/hostapd.conf
	
and add the following lines and change the ssid-, channel- and wpa_passpharase-line to values of your choice
pay attention to the Passpharse, it`s look like that the password have to start wie Upper-Letter

	interface=wlan0
	driver=nl80211
	ssid=ME_AP
	hw_mode=g
	channel=6
	macaddr_acl=0
	auth_algs=1
	ignore_broadcast_ssid=0
	wpa=2
	wpa_passphrase=My_Passphrase
	wpa_key_mgmt=WPA-PSK
	wpa_pairwise=TKIP
	rsn_pairwise=CCMP

open  /etc/default/hostapd

	sudo nano  /etc/default/hostapd

and change

	#DAEMON_CONF=""
 
to

	DAEMON_CONF="/etc/hostapd/hostapd.conf"

exit with

	ctrl+o
	ctrl+x

Now run the following commands to start the access point:

	sudo service hostapd start
	sudo service udhcpd start
	
Your Pi should now be hosting a wireless hotspot. Test it
To get the hotspot to start on boot, run these additional commands:

	sudo update-rc.d hostapd enable
	sudo update-rc.d udhcpd enable

last stept reboot

	sudo reboot

## OPTIONAL: forward wlan-traffic to eth0
if you connected to a normal router with an internet connection on eth0, you can forward wlan-traffic

open /etc/udhcpd.conf
	
	sudo nano /etc/udhcpd.conf

add dns server (google-dns-server)
	
	opt dns 8.8.8.8 4.2.2.2
you can cahnge it to your own dns-server

Configure NAT, Linux supports NAT using iptables. First, enable IP forwarding in the kernel:

	sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
	
To set this up automatically on boot, edit the file /etc/sysctl.conf and add the following line to the bottom of the file:

	net.ipv4.ip_forward=1

Second, to enable NAT in the kernel, run the following commands:

	sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
	sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
	sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

These instructions don't give a good solution for rerouting https and for URLs referring to a page inside a domain, like www.nu.nl/38274.htm. The user will see a 404 error. Your Pi is now NAT-ing. To make this permanent so you don't have to run the commands after each reboot, run the following command:

	sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

Now edit the file /etc/network/interfaces and add the following line to the bottom of the file:
	
	up iptables-restore < /etc/iptables.ipv4.nat
	
reboot
	
	sudo reboot
