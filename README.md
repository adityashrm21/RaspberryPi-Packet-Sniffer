# RaspberryPi-Packet-Sniffer
##### A simple HTTP and HTTPS sniffing tool created using Raspberry Pi (only for educational purposes)

### Setting up your Raspberry Pi

#### Prerequisites:

1. A class 4 Micro SD card of at least 8GB size
2. A Raspberry Pi 3 board (obviously, but you can also do the same with a lower model Pi along with a wifi-dongle)
3. A USB cable with an adapter to power the Pi up
4. Keyboard and Mouse (USB)
5. A Monitor or a TV as a display for thr Pi
6. Display connection cables: HDMI cable/ HDMI to VGA converter(if your monitor does not have an HDMI port)
7. Ethernet cable if you want to access internet through ethernet on the Pi which actualy we do want(Pi 3 comes with a built-in wireless LAN card which is very useful for our purpose, otherwise we would have needed a Wifi-Dongle)


#### Getting the Operating System to install on the Pi

1. You need to install the latest version of NOOBS or Raspbian on your Pi, and for that you need a bootable SD card with the OS installed on it
2. You need to format your SD card first. Download [SD Formatter 4.0](https://www.sdcard.org/downloads/formatter_4/) for either Windows or Mac and install it
3. Follow the instructions on the software and using a USB Micro SD card reader or an adapter, format the SD card using your laptop or PC
4. Now you need to install the image of the OS on the Micro SD card. [Download](https://www.raspberrypi.org/downloads/) the image of the OS from the official website
5. Download the [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) and install it on your computer. Now run the application and choose the OS image and the SD card drive from the drop down or browse menu and click on write
6. Now you have your OS on the SD card and you are ready to use it to boot your Pi

#### Plugging in your Raspberry Pi

1. Slot in your Micro SD card into the slot provided on the Raspberry Pi which would fit in only one way
2. Plug in your USB keyboard and mouse in the port provided on the Pi
3. Now for display, connect the HDMI cable from the Pi to the Monitor or TV depending on what you are using (you need to make sure that your monitor/TV is turned on and the appropriate mode is selected for display(HDMI/VGA/etc.))
4. Now plug in the ethernet cable into the ethernet port provided on th Pi next to the USB ports (you can know if its working if your Pi shows a flickering green light when turned on)
5. When all these cables are plugged in properly, you are ready to fire up the Pi. Just plug in the micro USB power supply and this would turn on and boot your Raspberry Pi

#### Logging into your Raspberry Pi

1. Now after the Pi has completed the boot process, a login will appear where you can use the default settings for login into the Pi: Username - pi, Password - raspberry
2. When you have succeessfully logged in, you will see the command line prompt pi@raspberrypi~$
3. Now once you are logged into you Pi, run

  ```bash
  sudo apt-get update
  ```
and

  ```bash
  sudo apt-get upgrade
  ```
to update your Pi to the newest available updates

#### Steps to create a Wifi-access point

1. If you have an ethernet cable plugged in into your Pi, you can start the web browser and see if the internet is working or not
2. Now type ifconfig in the terminal and note the IP address of your Pi in the eth0 interface(this would be the IP address of the Pi)
3. You now want to create a wifi-hotspot using the wifi-card on the Pi. This can be achieved using a service called hostapd but you don't just want the hotspot, you also want the internet access through the wireless access point. You also install the dnsmasq service for this purpose which is an easy to configure DNS and DHCP server
4. Use the following command and hit **y** when prompted to do so

  ```bash
  sudo apt-get install dnsmasq hostapd
  ```
5. The next step you need to do is to provide your wlan0 interface with a static IP. We already have our raspberry pi connected to the ethernet cable from whihc we will be sharing our internet
6. We will be using dhcpcd(most feature-rich open source DHCP client) to configure our interface configuration so open it up using

  ```bash
  sudo nano /etc/dhcpcd.conf
  ```
7. We need to tell it that our wlan0 has a static IP. So add these lines to it at the bottom of the file:

  ```bash
  interface wlan0
      static ip_address=172.24.1.1/24
  ```
8. We also need to prevent **wpa_supplicant** from running and interfering with setting up **wlan0** in access point mode. To do this open up the interface configuration file with

  ```bash
  sudo nano /etc/network/interfaces
  ```
and comment out the line containing **wpa-conf** in the **wlan0** section, so that it looks like this

  ```bash
  allow-hotplug wlan0  
  iface wlan0 inet manual  
  #    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
  ```
9. Now restart **dhcpcd** with

  ```bash
  sudo service dhcpcd restart
  ```
and it should assign **wlan0** with a static IP address
10. Now we need to configure **hostapd**. Change the configuration file for hostapd using

  ```bash
  sudo nano /etc/hostapd/hostapd.conf
  ```
with the contents given in the [hostapd.conf](https://github.com/adityashrm21/RaspberryPi-Packet-Sniffer/blob/master/hostapd.conf) file
11. To check whether all we've been doing is working or not, just run this command

  ```bash
  sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf
  ```
If everything goes well, you should be able to see the network Pi3-AP from your mobile phone or laptop device. You can try connecting to it in whoch case you would see some output from the Pi but you won't be allotted an IP address until we configure dnsmasq. So press **Ctrl + c** to stop it
12. Right now, hostapd is not configured to work on a fresh boot. So we also need to tell hostapd where to look for the config file when it starts up on boot. Open up the default configuration file with

  ```bash
  sudo nano /etc/default/hostapd
  ```
  and find the line **#DAEMON_CONF=""** and replace it with **DAEMON_CONF="/etc/hostapd/hostapd.conf"** and this would do the job

#### Setting up dnsmasq

1. The dnsmasq config file that comes preinstalled contains a lot of functionalities that we don't require at all so we delete it and create a new one using and paste the contents of [dnsmasq.conf](https://github.com/adityashrm21/RaspberryPi-Packet-Sniffer/blob/master/dnsmasq.conf) into it:

  ```bash
  sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig  
  sudo nano /etc/dnsmasq.conf
  ```
2. Now we need to enable packet forwarding. For this we need to open sysctl.conf using:

  ```bash
  sudo nano /etc/sysctl.conf
  ```
  and uncommenting the line **net.ipv4.ip_forward=1** and it will be enabled on the next boot
3. But to enable it for this session we quickly do:

  ```bash
  sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
  ```
4. Now we also need to share our Pi's internet to the devices connected to it throught the Wifi by configuring a NAT between the **eth0** and **wlan0** interface. We do this using the following commands:

  ```bash
  sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
  sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT  
  sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
  ```
5. But to enable the above settings everytime we boot, we need to do:

  ```bash
  sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
  ```
  and this will copy the settings to **iptables.ipv4.nat** file

6. now we need dhcpcd to run this and we do this by opening:

  ```bash
  sudo nano /lib/dhcpcd/dhcpcd-hooks/70-ipv4-nat
  ```
  and adding this to the file and saving it:

  ```bash
  iptables-restore < /etc/iptables.ipv4.nat  
  ```

7. now we are just one step behind sharing our internet through the Pi, just do:

  ```bash
  sudo service hostapd start  
  sudo service dnsmasq start
  ```
  and reboot the Pi for rechecking everything worked correctly using:

  ```bash
  sudo reboot
  ```
Now you would be able to connect to the internet through the Pi's network!

#### Man in the Middle Pi

Now we would tweak some settings and configurations and use **mitmproxy** to set up a man in the middle attack using our Raspberry Pi on it's hotspot

1. First you would need to install mitmproxy and any dependencies related to it:

  ```bash
  sudo pip install mitmproxy
  ```
2. Now we need to set up a **transparent proxy** using the iptables which can be done using the commands in the [mitm.sh](https://github.com/adityashrm21/RaspberryPi-Packet-Sniffer/blob/master/mitm.sh) file
3. Now run the **mitm.sh** file using:

  ```bash
  sudo ./mitm.sh
  ```
4. Now connect your phone to the Pi's hotspot and open your browser and browse some sites and you will see the data being generated in the console will all the http requests and responses
5. You can use the [mitmproxy documentation](http://docs.mitmproxy.org/en/stable/mitmproxy.html) for more information on how to use, look and store the data collected by mitmproxy
6. So we are set up as a man in the middle for the users connected to our Pi's network. But note here that we are only able to get information about the **HTTP** requests and not the **HTTPS** requests which are encrypted and need further hacking to break into which we do below

#### Configuring mitmproxy for secure connections

1. To get mitmproxy working for secure sites, you need to make a fake SSL certificate for the site you want to sniff and this would work even when the certificate is invalid because of the reasons given in [Priyank's blog](http://priyaaank.tumblr.com/post/81172916565/validating-ssl-certificates-in-mobile-apps) which you can go through
2. So now follow the steps given below to create your fake certificate:

  ```bash  
  openssl genrsa -out myown.cert.key 8192
  openssl req -new -x509 -key myown.cert.key -out fakesite.cert
  ```
  Specify all values like Company, BU, Country etc, as they appear in real certificate

  ```bash
  cat myown.cert.key fakesite.cert > fakesite.pem
  ```
3. Now you can run mitmproxy using this command:

  ```bash
  mitmproxy -p 8888 –cert=fakesite.pem
  ```
  Note: You can use any available port number in place of 8888
4. To connect to the network use the same port in advance options setting of the wifi network and then connect
5. Now you would be able to see request data from the secured site as well using mitmproxy

###### So this is how you can create a Raspberry Pi Sniffer. You can tweak the steps and do something really different on your own!
##### Sources:
1. [Raspberry Pi Official Documentation](https://www.raspberrypi.org/help/noobs-setup/)
2. [Frillip's Blog](https://frillip.com/using-your-raspberry-pi-3-as-a-wifi-access-point-with-hostapd/)
3. [Marxy's Blog](http://blog.marxy.org/2013/08/reverse-engineering-network-traffic.html)
4. [Priyank's Blog](http://priyaaank.tumblr.com/post/81172916565/validating-ssl-certificates-in-mobile-apps)
