# otto
##### a standalone battery (and pi) powered USB MIDI host & router

<img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_3.jpg width=800>

USB MIDI is great but a real pain in the ass when you're away from a computer. This project aims to fix that by providing a cheap and simple way to DIY yourself out of the connection problem. 

Otto automatically connects USB MIDI devices to each other while providing its own power. Use it to connect your MIDI controller and groovebox (OP1, Deluge, M8), your grooveboxes to each other (M8 and OP1, anyone?), or as the host to a larger setup. As Otto brings his own battery all you need is the devices and the same cables you'd use to connect them to your computer. Easy peasy.

### Parts required:
* <a href=https://geekworm.com/products/x306>Geekworm X306 USB Hub/UPS Expansion board for Raspberry Pi Zero W/2W</a>
* Raspberry Pi Zero 2W or W
* 4GB or larger microSD card
* 18650 battery

 <img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_1.jpg width=400>

### Assembly: 

 <img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_2.jpg width=400>

Mount the heatsink to the Pi by cutting the thermal tape to the size of the SOC and using it to hold the heatsink in place. 
Mount the Pi to the X306 board (USB and HDMI ports pointed away from the battery) with the provided screws. 

Install the microSD card (see card setup below) and 18650 battery and you're ready to go!

### Card Setup:

I will provide a flashable image sometime in the future but in the meantime you can set this up manually. Steps are taken from <a href=http://hunke.ws/posts/orange-pi-usb-midi-host/>this guide for Orange Pi</a> and <a href=https://neuma.studio/raspberry-pi-as-usb-bluetooth-midi-host/>this guide for Raspberry Pi 3/4</a>:

1. Download the <a href=https://www.raspberrypi.com/software/>Raspberry Pi Imager</a> and set up the SD card. Choose the Legacy 32 Lite image (no desktop is needed). You will need to enter in a username/password, set up wifi, and enable SSH access (on the second tab) in order to do finish the setup. Setting the hostname can also be helpful.
2. Insert the SD in your assembled Otto and boot it up. It will take some time to connect to your wifi, you can watch your router's DHCP logs to see what IP it pulls or try to ping the hostname you set until you see a response.
3. Connect to the pi using SSH
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
7. Save and exit the file (Ctrl+X, then Y)
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
14. Enable `sudo systemctl enable midi.service` the service.
15. Reboot the device `sudo reboot`

At this point USB MIDI routing should be working as expected, you can plug and unplug devices as desired and they should be automatically connected to each other.

In order to prevent SD card corruption you will want to set it up as read-only.

16. Reconnect to the device via SSH
17. Clone the rpi-readonly git `git clone https://gitlab.com/larsfp/rpi-readonly`
18. Enter the directory `cd rpi-readonly` and run the setup `sudo ./setup.sh`

When setup is complete your install will be set to read only mode. You are now ready to make music!

**If you want to make changes in the future you can turn read/write mode on with the command `rw` <br> 
When done making changes you can turn read-only mode back on using the command `ro`**
