# RaspberryPi-honeypot
#####A simple sniffing tool created using Raspberry Pi

### Setting up your Raspberry Pi

####Prerequisites:

1. A class 4 Micro SD card of at least 8GB size
2. A Raspberry Pi 3 board (obviously but you can also the same with a lower model along with a wifi-dongle)
3. A USB cable with an adapter to power the Pi up
4. Keyboard and Mouse (USB)
5. A Monitor or a TV as a display for thr Pi
6. Display connection cables: HDMI cable/ HDMI to VGA converter(if your monitor does not have an HDMI port)
7. Ethernet cable if you want to access internet through ethernet on the Pi which actualy we do want(Pi 3 comes with a built-in wireless LAN card which is very useful for our purpose, otherwise we would have needed a Wifi-Dongle)


####Getting the Operating System to install on the Pi

1. You need to install the latest version of NOOBS or Raspbian on your Pi, and for that you need a bootable SD card with the OS installed on it
2. You need to format your SD card first. Download [SD Formatter 4.0](https://www.sdcard.org/downloads/formatter_4/) for either Windows or Mac and install it
3. Follow the instructions on the software and using a USB Micro SD card reader or an adapter, format the SD card using you laptop or PC
4. Now you need to install the image of the OS on the Micro SD card. [Download](https://www.raspberrypi.org/downloads/) the image of the OS from the official website
5. Download the [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) and install it on your computer. Now run the application and choose the OS image and the SD card drive from the drop down or browse menu and click on write
6. Now you have your OS on the SD card and you are ready to use it to boot your Pi

####Plugging in your Raspberry Pi

1. Slot in your Micro SD card into the slot provided on the Raspberry Pi which would fit in only one way
2. Plug in your USB keyboard and mouse in the port provided on the Pi
3. Now for display, connect the HDMI cable from the Pi to the Monitor or TV depending on what you are using (you need to make sure that your monitor/TV is turned on and the appropriate mode is selected for display(HDMI/VGA/etc.)
4. Now plug in the ethernet cable into the ethernet port provided on th Pi neext to the USB ports (you can know if its working if your Pi shows a flickering green light when turned on)
5. When all these cables are plugged in properly, you are ready to fire up the Pi. Just plug in the micro USB power supply and this would turn on and boot your Raspberry Pi

####Logging into your Raspberry Pi

1. Now after the Pi has completed the boot process, a login will appear where you can use the default settings for login into the Pi: Username - pi, Password - raspberry
2. When you have succeessfully logged in, you will see the command line prompt pi@raspberrypi~$

