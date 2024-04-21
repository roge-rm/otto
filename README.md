# otto
##### a standalone battery (and pi) powered USB MIDI host & router

<img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_3.jpg width=800>

USB MIDI is great but a real pain in the ass when you're away from a computer. This project aims to fix that by providing a cheap and simple way to DIY yourself out of the connection problem.

### Parts required:
* <a href=https://geekworm.com/products/x306>Geekworm X306 USB Hub/UPS Expansion board for Raspberry Pi Zero W/2W</a> (heatsink optional?)
* Raspberry Pi Zero W or 2W
* 4GB or larger microSD card
* 18650 battery

 <img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_1.jpg width=400>

### Assembly: 

 <img src=https://raw.githubusercontent.com/roge-rm/otto/main/pictures/otto_2.jpg width=400>

Mount the heatsink to the Pi by cutting the thermal tape to the size of the SOC and using it to hold the heatsink in place. 
Mount the Pi to the X306 board (USB and HDMI ports pointed away from the battery) with the provided screws. 

Install the microSD card (see card setup below) and 18650 battery and you're ready to go!

### Card Setup:

I will provide a flashable image sometime in the future but in the meantime you can set this up manually:

1. Download the <a href=https://www.raspberrypi.com/software/>Raspberry Pi Imager</a> and set up the SD card. Choose the Legacy 32 Lite image (no desktop is needed). You will need to enter in a username/password, set up wifi, and enable SSH access (on the second tab) in order to do finish the setup. Setting the hostname can also be helpful.
2. Insert the SD in your assembled otto and boot it up. It will take some time to connect to your wifi, you can watch your router's DHCP logs to see what IP it pulls or try to ping the hostname you set until you see a response.
3. Connect to the pi using SSH
4. Complete setup (steps coming)
