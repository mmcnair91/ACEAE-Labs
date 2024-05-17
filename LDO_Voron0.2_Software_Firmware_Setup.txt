LDO Kit Voron 0.2 setup instructions
Raspberry Pi OS Setup
 - Download and run the Raspberry Pi Imager.
 - Under Advanced options, enable SSH, set a username and password, and enter your WIFI info.
 - Under Operating System Choose Raspberry Pi OS (other) -> Raspberry Pi OS Lite (32bit)).
 - Insert the SD card provided in the kit and choose it under storage. 
 - Adjust the advanced options and provide the WiFi network SSID and passkey you want it to connect to. Make sure to also enable SSH capabilities
 - Write down the name you give the printer (should be something like myprinter.local), the username and password you choose. You'll need them later
 - Click Write and wait for the Imager to finish writing the data into the SD card.
 - Remove the SD card from your computer and insert it into the Raspberry Pi
 - Turn on the printer
Obtain the printer's IP and MAC address (you'll need a display and a keyboard)
 - the easiest way to get these is to plug a display and keyboard directly into the Raspberry Pi when booting (via ribbon cable, hdmi, or minihdmi)
 - once the Pi is fully booted the console should automatically show on the screen
 - type "sudo apt-get update" and hit enter to update the Pi to the latest firmware; this may take a couple minutes
 - type in "ifconfig" and press enter
 - write down the mac address which is listed after "ether" and is in the format XX:XX:XX:XX:XX:XX where X can be a letter or number
 - there are two different mac addresses depending on if you are connected to WiFi or directly through an ethernet cable; "eth0" is for ethernet connections and "wlan0" is for WiFi
 - write down the IP address for your chosen connection type (eth0/wlan0) which is listed after "inet" and should be something like ###.###.###.### where # is a number
SSH into the printer using Putty (Windows) or terminal (Mac) using the IP address you obtained; for help with this see this article (https://www.makeuseof.com/how-to-ssh-into-raspberry-pi-remote/)
Use KIAUH to install Klipper, then Moonraker, and then Mainsail (if using a touchscreen go ahead and install KlipperScreen too)
The KIAUH instructions are really clear and well laid out; if you want to read more on any of the programs it helps you install check the Github (https://github.com/dw-0/kiauh)
  - Download KIAUH by typing "sudo apt-get install git -y" into the console and hitting enter
  - Once git is installed, use the following command to download KIAUH into your home-directory: "cd ~&& git clone https://github.com/dw-0/kiauh.git"
  - Start KIAUH by running this command: "./kiauh/kiauh.sh"
  - You should now find yourself in the main menu of KIAUH. You will see several actions to choose from depending on what you want to do. To choose an action, simply type the corresponding number into the "Perform action" prompt and confirm by hitting ENTER.
Flashing firmware to the SKR Pico and Picobilical
  - Note: this is one of the more problematic steps in this process so pay close attention to instructions; I find this confusing because there are multiple instruction sources (LDO, Voron, Discords, etc.) and they all differ
  - Make sure the Pi and SKR Pico are connected via USB
  - SSH into the Raspbeery Pi
  - Run the following: "sudo apt install make" then "cd ~/klipper" then "make menuconfig"
  - enable extra low-level configuration options by pressing ENTER; an asterisk should appear between the brackets in the top left corner of the window
  - Ensure that the micro-controller architecture is set to 'Raspberry Pi RP2040' by using the arrow keys to highlight that part of the menu, pressing ENTER and selecting "Raspberry Pi RP2040" by pressing ENTER
  - Once the proper configuration is selected, press Q to exit and Yes if asked to save the configuration
  - Run the following: "make clean" then "make"; the "make" command creates a firmware file called klipper.uf2 and should be located in the directory "~/klipper/out" on the Rapsberry Pi
  - Install a jumper on the Boot pins and (*only if your SKR Pico is NOT connected to your power supply already*) install a jumper for USB power
