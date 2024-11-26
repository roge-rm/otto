# otto
##### a standalone battery (and pi) powered USB MIDI host & router

<img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_launchctl_deluge_exquis.jpg width=800>

USB MIDI is great but a real pain in the ass when you're away from a computer. This project aims to fix that by providing a cheap and simple way to DIY yourself out of the connection problem. 

Otto automatically connects USB MIDI devices to each other while providing its own power. Use it to connect your MIDI controller and groovebox (OP1, Deluge, M8), your grooveboxes to each other (M8 and OP1, anyone?), or as the host to a larger setup. As Otto brings his own battery all you need is the devices and the same cables you'd use to connect them to your computer. Easy peasy.

**The preconfigured image provided below should work on any Raspberry Pi capable of running the 32 bit/legacy Raspberry Pi OS.**<br>

<img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_1.jpg width=400>

**I have been notified that the Geekworm X306 board has been updated to remove the USB ports and is no longer an option for this project. This will work with any USB/battery hat (or combination thereof) that can be used with a Raspberry Pi so if you find another battery and another USB hub or some combination of those you can still make something like this - just not with the housing I designed.**

### Parts required:
* <a href=https://geekworm.com/products/x306>Geekworm X306 USB Hub/UPS Expansion board for Raspberry Pi Zero W/2W</a>
* Raspberry Pi Zero 2W or W
* 8GB or larger microSD card for preconfigured image (4GB required for manual setup)
* 18650 battery
* <a href=https://github.com/roge-rm/otto/blob/main/housing/HOUSING.md>3D printed housing</a> (not _required_ for functionality but maybe for the real world)

<img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_2.jpg width=400>

### Assembly: 

1. Mount the heatsink to the Pi by cutting the thermal tape to the size of the SOC and using it to hold the heatsink in place. 
2. Mount the Pi to the X306 board (USB and HDMI ports pointed away from the battery) with the provided screws.
3. Mount the battery. 
4. <a href=https://github.com/roge-rm/otto/blob/main/housing/HOUSING.md>Install the X306 board in the housing</a>, if you have decided to give Otto a home.
5. Prepare the SD card, see [card setup](#card-setup) below. <br>
You will need an 8GB or larger microSD card to use the preconfigured image, or 4GB or larger to manually set it up yourself.
6. Insert the microSD card into your completed Otto and power it on using the button on the front. You're ready to go! 

<img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto2_6.jpg width=400>

### Usage:

Usage is straight forward. Turn the power on, wait ~10-30 seconds (depending on speed of SD card), plug in USB devices you'd like to connect and voila! Turn the device off when done, as you've set the SD to read only there's no need to do a full shut down.

### Card Setup:

#### Preconfigured Image:

Download the latest preconfigured image <a href=https://github.com/roge-rm/otto/releases/tag/otto-legacy32lite-v01>at this link</a>.

```
This should work on any Raspberry Pi that can run a 32 Bit Raspberry Pi OS.
It has been tested on a Raspberry Pi Zero 2W as well as a Raspberry Pi 2B.
```

Extract the .img file from the zip and write it to an 8GB or larger microSD using <a href=https://rufus.ie/en/>Rufus</a> or <a href=https://win32diskimager.org/>Win32 Disk Imager</a> for Windows, or dd on linux/mac.

#### Manual Setup:
If you don't want to use the preconfigured image above you can set this up manually.

Steps are taken from <a href=http://hunke.ws/posts/orange-pi-usb-midi-host/>this guide for Orange Pi</a> and <a href=https://neuma.studio/raspberry-pi-as-usb-bluetooth-midi-host/>this guide for Raspberry Pi 3/4</a>:

1. Download the <a href=https://www.raspberrypi.com/software/>Raspberry Pi Imager</a> and set up the SD card. Choose the 32bit Bookworm lite image (no desktop is needed). You will need to enter in a username/password, set up WIFI (or use a USB ethernet adapter), and enable SSH access (on the second tab) in order to finish the setup. Setting the hostname can also be helpful.
2. Insert the SD in your assembled Otto and boot it up. It will take some time to connect to your WIFI, you can watch your router's DHCP logs to see what IP it pulls or try to ping the hostname you set until you see a response.
3. Connect to the Pi using SSH
4. Perform `sudo apt update` and `sudo apt upgrade` to update all current packages
5. Perform `sudo apt install git ruby` to install required packages
6. Create and edit the connection script `sudo nano /usr/local/bin/connectall.rb`<br>
Paste the following code as the contents:
```
#!/usr/bin/ruby
#

t = `aconnect -i -l`
ports = []
t.lines.each do |l|
  /client (\d*)\:/=~l
  port = $1
  # we skip empty lines and the "Through" port
  unless $1.nil? || $1 == '0' || /Through/=~l
    ports << port
    #names << name
  end  
end

ports.each do |p1|
  ports.each do |p2|
    unless p1 == p2 # probably not a good idea to connect a port to itself
      system  "aconnect #{p1}:0 #{p2}:0"
    end
  end
end
```
7. Save and exit the file (Ctrl+X, then Y, then enter)
8. Set permissions on the file `sudo chmod +x /usr/local/bin/connectall.rb`
9. Connect one or more USB devices to otto and issue `connectall.rb` to perform a test connection<br>
Check the results with `aconnect -l`, it should show connected devices
10. Create a udev rule to run the script when USB devices are connected `sudo nano /etc/udev/rules.d/33-midiusb.rules`<br>
Paste the following as contents:
```
ACTION=="add|remove", SUBSYSTEM=="usb", DRIVER=="usb", RUN+="/usr/local/bin/connectall.rb"
```
11. Restart the udev service to enable `sudo udevadm control --reload` and `sudo service udev restart`
12. Create a systemd service to configure any attached devices at boot `sudo nano /lib/systemd/system/midi.service`
Paste the following as contents:
```
[Unit]
Description=Initial USB MIDI connect

[Service]
ExecStart=/usr/local/bin/connectall.rb

[Install]
WantedBy=multi-user.target
```
13. Reload the systemctl daemon `sudo systemctl daemon-reload`
14. Enable the service `sudo systemctl enable midi.service` and start it `sudo systemctl start midi.service`
15. Edit the crontab to run the connectall script upon boot, connecting any devices that were already connected ahead of time `crontab -e`
Select the nano editor if prompted and you are not sure.
16. Add the following to the bottom of the crontab:
```
@reboot ruby /usr/local/bin/connectall.rb &
```

At this point USB MIDI routing should be working as expected, you can plug and unplug devices as desired and they should be automatically connected to each other.

#### BLE MIDI:
If you'd like to add BLE (Bluetooth Low Energy) MIDI support you'll need to <a href=https://github.com/arkq/bluez-alsa/wiki/Installation-from-source>compile and install BlueALSA</a>. The standard bluetooth stack does not support BLE MIDI so we will add it in.

17. Install required packages to compile BlueALSA:
```
sudo apt-get install git automake build-essential libtool pkg-config python3-docutils
sudo apt-get install libasound2-dev libbluetooth-dev libdbus-1-dev libglib2.0-dev libsbc-dev
```
18. Download, compile, and install bluez-alsa from source
```
mkdir ~/dev
git clone https://github.com/arkq/bluez-alsa.git /home/otto/dev/bluez-alsa
cd ~/dev/bluez-alsa
autoreconf --install --force
mkdir build && cd build
../configure --enable-systemd --enable-midi
make
sudo make install
```
19. Create a udev rule to run the connectall script when the BLE server is started 
`sudo nano /etc/udev/rules.d/44-bt.rules`
```
ACTION=="add|remove", SUBSYSTEM=="bluetooth", RUN+="/usr/local/bin/connectall.rb"
```
20. Restart the udev service to enable `sudo udevadm control --reload` and `sudo service udev restart`
21. Create a systemd service to enable BLE MIDI server at boot `sudo nano /lib/systemd/system/btmidi.service`
```
[Unit]
Description=MIDI Bluetooth connect
After=bluetooth.target sound.target multi-user.target
Requires=bluetooth.target sound.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/home/otto
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=btmidi
Restart=always
ExecStart=/usr/bin/bluealsad -p midi --midi-advertisement

[Install]
WantedBy=multi-user.target
```
22. Enable the service `sudo systemctl enable btmidi.service` and start it `sudo systemctl start btmidi.service`
23. You should now be able to scan for BLE MIDI devices (eg from your iPad, from BLE MIDI app on Android) and see an 'otto' device available to connect to

At this point BLE MIDI should be set up and working as expected, and devices that connect to the 'otto' device will be able to exchange MIDI data with any devices connected to the Pi.

#### Read-Only Mode:
In order to prevent SD card corruption you will want to set it up as read-only.

24. Reconnect to the device via SSH
25. Clone the rpi-readonly git `git clone https://gitlab.com/larsfp/rpi-readonly`
26. Enter the directory `cd rpi-readonly` and run the setup `sudo ./setup.sh`

When setup is complete your install will be set to read only mode. You are now ready to make music!

**If you want to make changes in the future you can turn read/write mode on with the command `rw` <br> 
When done making changes you can turn read-only mode back on using the command `ro`**
