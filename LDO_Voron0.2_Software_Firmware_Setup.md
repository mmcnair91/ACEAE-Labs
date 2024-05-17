# LDO Kit Voron 0.2 setup instructions

 ## Raspberry Pi OS Setup
 - Download and run the Raspberry Pi Imager (https://www.raspberrypi.com/software/)
 - Under Advanced options, enable SSH, set a username and password, and enter your WIFI info.
 - Under Operating System Choose Raspberry Pi OS (other) -> Raspberry Pi OS Lite (32bit)).
 - Insert the SD card provided in the kit and choose it under storage. 
 - Adjust the advanced options and provide the WiFi network SSID and passkey you want it to connect to. Make sure to also enable SSH capabilities
 - Write down the name you give the printer (should be something like myprinter.local), the username and password you choose. You'll need them later
 - Click Write and wait for the Imager to finish writing the data into the SD card.
 - Remove the SD card from your computer and insert it into the Raspberry Pi
 - Turn on the printer

## Obtain the printer's IP and MAC address (you'll need a display and a keyboard)
 - the easiest way to get these is to plug a display and keyboard directly into the Raspberry Pi when booting (via ribbon cable, hdmi, or minihdmi)
 - once the Pi is fully booted the console should automatically show on the screen
 - type ```sudo apt-get update``` and hit enter to update the Pi to the latest firmware; this may take a couple minutes
 - type in ```ifconfig``` and press enter
 - write down the mac address which is listed after "ether" and is in the format XX:XX:XX:XX:XX:XX where X can be a letter or number
 - there are two different mac addresses depending on if you are connected to WiFi or directly through an ethernet cable; "eth0" is for ethernet connections and "wlan0" is for WiFi
 - write down the IP address for your chosen connection type (eth0/wlan0) which is listed after "inet" and should be something like ###.###.###.### where # is a number
SSH into the printer using Putty (Windows) or terminal (Mac) using the IP address you obtained; for help with this see this article (https://www.makeuseof.com/how-to-ssh-into-raspberry-pi-remote/)

## Use KIAUH to install Klipper, then Moonraker, and then Mainsail (if using a touchscreen go ahead and install KlipperScreen too). The KIAUH instructions are really clear and well laid out; if you want to read more on any of the programs it helps you install check the [Github](https://github.com/dw-0/kiauh)

 ![kiauh](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/3455d2ab-8fdc-46f2-8449-0757e43369c2)

  - Download KIAUH by typing ```sudo apt-get install git -y``` into the console and hitting enter
  - Once git is installed, use the following command to download KIAUH into your home-directory: ```cd ~&& git clone https://github.com/dw-0/kiauh.git```
  - Start KIAUH by running this command: ```./kiauh/kiauh.sh```
  - You should now find yourself in the main menu of KIAUH. You will see several actions to choose from depending on what you want to do. To choose an action, simply type the corresponding number into the "Perform action" prompt and confirm by hitting ENTER.

## Flashing firmware to the SKR Pico and Picobilical (based on instructions from [Voron Design](https://docs.vorondesign.com/build/software/skrPico_klipper.html) and [LDO](https://docs.ldomotors.com/en/voron/voron01/Picobilical#uploading-firmware))
 - Note: this is one of the more problematic steps in this process so pay close attention to instructions; I find this confusing because there are multiple instruction sources (LDO, Voron, Discords, etc.) and they all differ
 - Make sure the Pi and SKR Pico are connected via USB
 - SSH into the Raspberry Pi
 - Run the following: ```sudo apt install make``` then ```cd ~/klipper``` then ```make menuconfig```
 - enable extra low-level configuration options by pressing ENTER; an asterisk should appear between the brackets in the top left corner of the window
 - Ensure that the micro-controller architecture is set to "Raspberry Pi RP2040" by using the arrow keys to highlight that part of the menu, pressing ENTER and selecting "Raspberry Pi RP2040" by pressing ENTER

![skrPico_klipper_menuconfig](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/806ed432-81b3-46b0-9eb5-1c7ab5d8eb6f)

 - Once the proper configuration is selected, press Q to exit and Yes if asked to save the configuration
 - Run the following: ```make clean``` then ```make```; the "make" command creates a firmware file called klipper.uf2 and should be located in the directory ```~/klipper/out``` on the Rapsberry Pi
 - Install a jumper on the Boot pins and (*only if your SKR Pico is NOT connected to your power supply already*) install a jumper for USB power

![SKR_Pico_Pin_Flashing](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/3ecbfde0-33dc-4b0f-b6ef-5aaae2e87cfa)

### Flashing the firmware to the SKR Pico
**PC Method**
   - Copy the klipper.uf2 file to a directory that you can access via mainsail with this command: ```cp out/klipper.uf2 ~/printer_data/config/klipper.uf2```
   - After running that command you should find the klipper.uf2 file in the config section of your UI (mainsail)
   - Right click and download this file to your PC to flash the SKR Pico
   - Connect the SKR Pico to your PC using the USB-C connector, then push the reset button on the SKR Pico (see image of the board above if unsure where it is); after pressing, it should show up just like a flash drive would on your computer

 ![windows_explorer_mounted_drive](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/0d19fa92-88b3-4767-acd8-26cd6abdf45d)

   - Copy the file you saved to your computer above (klipper.uf2) onto the SKR Pico just like you would any file to a flash drive. As soon as you copy the file over, the SKR Pico will reboot automatically and it will be flashed with Klipper
   - Unplug the SKR Pico from the PC, remove any of the jumpers you added earlier, plug the USB into your Raspberry Pi, push the SKR Pico reset button, and check if it's been flashed properly by running the following command: ```ls /dev/serial/by-id
   - If it has been flashed properly the result should be similar to the one in the image below but your USB-ID will be different, but should start with usb-Klipper-rp2040

![SKR_pico_by-id_output](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/dabf21a8-d357-4b06-a402-0bdd17f78232)

**Raspberry Pi Method**
   - Note: If the Pcio is not powered with 12-24V (via your printer's power supply) Klipper will be unable to communicate with the TMC drivers and the SKR Pico will automatically shut down
   - With the SKR Pico plugged into the Pi via USB-C and the boot jumper installed, press the reset button
   - Mount the SKR Pico to the Raspberry Pi to copy the file (klipper.uf2) over
   - Run the following commands: ```sudo mount /dev/sda1 /mnt`` then ```sudo cp out/klipper.uf2 /mnt``` then ```sudo umount /mnt```
   - Remove the jumpers that were installed earlier and reset the SKR Pico (see image of the board above if unsure where it is)
   - Run this command to retrieve the USB-ID for the SKR Pico ```ls /dev/serial/by-id```
   - If it has been flashed properly the result should be similar to the one in the image below but your USB-ID will be different, but should start with usb-Klipper-rp2040

![SKR_pico_by-id_output](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/dabf21a8-d357-4b06-a402-0bdd17f78232)

***Write down the USB-ID for the SKR Pico and make sure you denote that it is for the SKR Pico; confusing this USB-ID with the Picobilical USB-ID can result in unintended damage to your printer or mainboard***

### Flashing the firmware to the Picobilical
There are two methods for uploading the firmware: 
- UF2 Bootloader Mode (when no klipper firmware was previously installed)
- Klipper Make (if you already have klipper installed on the Picobilical)

Most LDO V0 kits ship with the Picobilical firmware already flashed but it is advisable to update it via the Klipper Make option during initial setup. You can tell if your Picobilical has firmware flashed already or not by if the PWR (Power) LED turns on or not (On=installed; Off=not installed).

**UF2 Bootloader Mode**
  - Make sure the USB power pins are shorted with a jumper on the Picobilical - this allows the mcu on this PCB to be powered from the USB port (circled in red in image below)
  ![picobilical_hef_5v_usb_jumper_labelled](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/ded49f9b-9eb8-40f8-98cf-aff84f6cb271)
  - Connect the Picobilical to your PC using the USB-C connector; the power LED on the PCB should light up
  - Simultaneously press the BootSel and Reset button, then release Reset first then followed by BootSel (both buttons are in the top left corner of the image above; partially hidden by the RPI PWR cable)
  - On your computer, the Picobilical should now show up just like a flash drive would on your computer
  - Copy the file you saved to your computer above (klipper.uf2) onto the Picobilical just like you would any file to a flash drive. As soon as you copy the file over the Picobilical will disappear from your computer, indicating that the new firmware was successfully uploaded to the microcontroller
  - Unplug the Picobilical from the PC, remove any of the jumpers you added earlier, plug the USB into your Raspberry Pi
 **Klipper Make**
  - SSH into your Raspberry Pi and run ```ls /dev/serial/by-id``` to find the USB-ID of your Picobilical
  - The USB-ID should have a similar format to this: ```usb-Klipper_rp2040_1234567890000000-if00```
  - Make sure the USB-ID is different than the one for the SKR Pico; if it is not unplug the SKR Pico USB from the Raspberry Pi, restart your printer and repeat the two steps above; plug the SKR Pico USB back into the Raspberry Pi after completing this step
  - Write down the USB-ID for the Picobilical and make sure you denote that it is for the Picobilical; confusing this USB-ID with the SKR Pico USB-ID can result in unintended damage to your printer or mainboard
  - Run the following commands to automatically compile and upload the firmware to the Picobilical MCU:

```
cd ~/klipper
make clean
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/<your USB ID>
sudo service klipper start
```
  - if you encounter any connection issues after flashing the new firmware, reboot your printer
  - Your Picobilical should now have the newest firmware

## Klipper Configuration
These configuration files tell Klipper how our printer is wired. It also contains other useful data like custom macros, tuning values, etc.
 - Open your printer's web interface (mainsail) by entering your Raspberry Pi's IP address in any browser. Initially you will see an error message indicating a missing "printer.cfg" file
 - We followed the LDO wiring guide during installation so we can use their [pre-made configuration files](https://github.com/MotorDynamicsLab/LDOVoron0/tree/v02/Firmware) they also have a [separate file for the Picobilical](https://github.com/MotorDynamicsLab/LDO-Picobilical/tree/master/Klipper_Configs)
 - Download the "printer.cfg" bile and upload it to Mainsail under the Machine (Wrench icon) by drag and dropping the file
 - Edit the printer.cfg file, under the ```[mcu]``` section, replace ```{REPLACE WITH YOUR SERIAL}``` with the SKR Pico USB-ID you obtained in the previous steps
 - Edit the printer.cfg file, under the ```[mcu umb]``` section, replace ```{REPLACE WITH YOUR SERIAL}``` with the Picobilical USB-ID you obtained in the previous steps
 - FINAL WARNING: **DO NOT** mix up the USB-ID serial paths for the SKR Pico and Picobilical, doing so can result in unintended damage to your printer or mainboard

With all the configuration files in place, you should now be able to use Fluidd/Mainsail to perform basic controls on your 3D printer. However, there are still a few more steps you should follow before starting your first print.
  - Follow the [initial startup guide](https://docs.vorondesign.com/build/startup/) from Voron Design
  - Fine-tune sensorless homing by reading [this article](https://docs.vorondesign.com/community/howto/clee/sensorless_xy_homing.html) by clee (well known and respected member of the 3D printing community)
