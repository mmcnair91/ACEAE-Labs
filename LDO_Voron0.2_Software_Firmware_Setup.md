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
 - there are two different mac addresses depending on if you are connected to WiFi or directly through an ethernet cable; "eth0" is for ethernet connections and "wlan0" is for WiFi
 - write down the mac address which is listed after "ether" and is in the format XX:XX:XX:XX:XX:XX where X can be a letter or number
 - write down the IP address for your chosen connection type (eth0/wlan0) which is listed after "inet" and should be something like ###.###.###.### where # is a number
SSH into the printer using Putty (Windows) or terminal (Mac) using the IP address you obtained; for help with this see this article (https://www.makeuseof.com/how-to-ssh-into-raspberry-pi-remote/)

## Use KIAUH to install Klipper, then Moonraker, and then Mainsail (if using a touchscreen go ahead and install KlipperScreen too). The KIAUH instructions are really clear and well laid out; if you want to read more on any of the programs it helps you install check the [Github](https://github.com/dw-0/kiauh)

 ![kiauh](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/3455d2ab-8fdc-46f2-8449-0757e43369c2)

  - Download KIAUH by typing ```sudo apt-get install git -y``` into the console and hitting enter
  - Once git is installed, use the following command to download KIAUH into your home-directory: ```cd ~&& git clone https://github.com/dw-0/kiauh.git```
  - Start KIAUH by running this command: ```./kiauh/kiauh.sh```
  - You should now find yourself in the main menu of KIAUH. You will see several actions to choose from depending on what you want to do. To choose an action, simply type the corresponding number into the "Perform action" prompt and confirm by hitting ENTER.
  - Install Klipper, then Moonraker, then Mainsail
  - If prompted to install macros or additional things go ahead and select Yes

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
   - Unplug the SKR Pico from the PC, remove any of the jumpers you added earlier, reconnect the SKR Pico USB-C that connects to your Raspberry Pi, push the SKR Pico reset button, and check if it's been flashed properly by running the following command: ```ls /dev/serial/by-id```
   - If it has been flashed properly the result should be similar to the one in the image below but your USB-ID will be different, but should start with usb-Klipper-rp2040

![SKR_pico_by-id_output](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/dabf21a8-d357-4b06-a402-0bdd17f78232)

**Raspberry Pi Method**
   - Note: If the Pico is not powered with 12-24V (via your printer's power supply) Klipper will be unable to communicate with the TMC drivers and the SKR Pico will automatically shut down
   - With the SKR Pico plugged into the Pi via USB-C and the boot jumper installed, press the reset button
   - Mount the SKR Pico to the Raspberry Pi to copy the file (klipper.uf2) over
   - Run the following commands: ```sudo mount /dev/sda1 /mnt`` then ```sudo cp out/klipper.uf2 /mnt``` then ```sudo umount /mnt```
   - Remove the jumpers that were installed earlier and reset the SKR Pico (see image of the board above if unsure where it is)
   - Run this command to retrieve the USB-ID for the SKR Pico ```ls /dev/serial/by-id```
   - If it has been flashed properly the result should be similar to the one in the image below but your USB-ID will be different, but should start with usb-Klipper-rp2040

![SKR_pico_by-id_output](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/dabf21a8-d357-4b06-a402-0bdd17f78232)

***Write down the USB-ID for the SKR Pico and make sure you denote that it is for the SKR Pico; confusing this USB-ID with the Picobilical USB-ID can result in unintended damage to your printer or mainboard***

### Flashing the firmware to the Picobilical
Preparing the Picobilical
 - Install the fan voltage jumpers (green things top right corner) corresponding to the rated voltage of your hotend (HEF) and part fans (PCF). In the photo shown below, the hotend fan is set to 5V while the part fan is set to 24V.  These kits comes with all 24v fans, so set both jumpers to 24v by placing the jumpers on the leftmost and middle pins.
 - Please double check the sticker on the fan to make sure you have 24v fans otherwise it will damage the fan.
  ![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/c95ec0a0-cde8-4998-96af-c2b01625eb0f)

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
 - Download the "printer.cfg" file and upload it to Mainsail under the Machine (Wrench icon) by drag and dropping the file
 - Edit the printer.cfg file, under the ```[mcu]``` section, replace ```{REPLACE WITH YOUR SERIAL}``` with the SKR Pico USB-ID you obtained in the previous steps
 - Edit the printer.cfg file, under the ```[mcu umb]``` section, replace ```{REPLACE WITH YOUR SERIAL}``` with the Picobilical USB-ID you obtained in the previous steps
 - Edit the printer.cfg file, so that the Mainsail.cfg file is included by deleting the "#" before ```[include mainsail.cfg]``` ![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/78744348-d9bf-48c8-a6ee-425d5b1b21a9)

 - FINAL WARNING: **DO NOT** mix up the USB-ID serial paths for the SKR Pico and Picobilical, doing so can result in unintended damage to your printer or mainboard

With all the configuration files in place, you should now be able to use Fluidd/Mainsail to perform basic controls on your 3D printer. However, there are still a few more steps you should follow before starting your first print.
  - Fine-tune sensorless homing by reading [this article](https://docs.vorondesign.com/community/howto/clee/sensorless_xy_homing.html) by clee (well known and respected member of the 3D printing community)

## Initial Startup ([based on instructions from Voron Design](https://docs.vorondesign.com/build/startup/))
 - Issue a RESTART command after every change to the config file to ensure that the change takes effect ( type ```restart``` into the Mainsail terminal then click "send").
 - It is also a good idea to issue a ```status``` command after every ```restart``` to verify that the config file is successfully loaded
 - Any time commands are requested to be issued, those will happen in the ‘Console’ tab of Mainsail in the box for entering commands directly.

![mainsail_terminal](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/95f50a4c-c856-49d3-a829-cf9d8bcd7c31)

 - Any time movements need to be made, those will happen in the ‘Dashboard’ tab / section of the Mainsail web UI. The numbers underneath X, Y, and Z control the movement distance.

![mainsail_controls](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/597b6a34-319d-4188-854e-72bb2b742dd9)

### Verify Temperature
 - Start by verifying that temperatures are being properly reported by navigating to the Mainsail temperature graph (in the Dashboard section)
 - Verify that the temperature of the nozzle and bed are present and **not increasing**. If it is increasing, remove power from the printer. If the temperatures are not accurate, review the ```sensor_type``` and ```sensor_pin``` settings for the extruder and/or bed in the printer.cfg file

### Verify Heaters
 - In the temperature graph section, type in "50" followed by ENTER in the “Extruder” temperature target field. The extruder temperature in the graph should start to increase (within about 10 seconds or so)
 - Go to the “Tool” temperature drop-down box and select “Off”/"0C". After several minutes the temperature should start to return to its initial room temperature value. If the temperature does not increase, then verify the ```heater_pin``` setting in the printer.cfg is correct
 - Perform the above steps again with the bed

### Stepper Motor Check
 - To verify each stepper motor is operating correctly, send the following command in the console: ```STEPPER_BUZZ STEPPER=stepper_x```
 - This command causes the given stepper to move one millimeter in a positive direction and then return to its starting position and repeat these actions 10 times.
 - We will verify direction later, but ideally all motors will be running correctly at the end of this test
 - We are not watching that the toolhead is moving when we issue these commands, but the pulley on the motors
 - See the table below for the expected motion for each command; run the command for each of the motors:

|Motor    |Expectation                                                      |
|---------|-----------------------------------------------------------------|
|stepper_x|The motor will rotate clockwise first, then back counterclockwise|
|stepper_y|The motor will rotate clockwise first, then back counterclockwise|
|stepper_z|The bed moves down, then back up                                 |
|extruder |Movement: Direction will be tested later                         |

 - If the stepper does not move at all, then verify the ```enable_pin``` and ```step_pin``` settings for the stepper
 - If the stepper motor moves but does not return to its original position then verify the ```dir_pin``` setting
 - If the stepper motor oscillates in an incorrect direction, this generally indicates that the ```dir_pin``` for the axis needs to be inverted. This is done by adding a "!" to the ```dir_pin``` in the printer.cfg file (or removing it if one is already there)
 - If the motor moves significantly more or significantly less than one millimeter, then verify the ```rotation_distance``` setting

### Endstop Checks
 - The beauty of an LDO kit is that their printer.cfg file already has sensorless homing set up for us on the X and Y and sensored homing on the Z!
 - Make sure the bed is not pushing on the Z endstop
 - In the console send a ```QUERY_ENDSTOPS``` command; the X, Y, and Z should all say "open"
 - If any of them say “triggered” instead of “open”, double-check to make sure none of them are pressed
 - Manually press the Z endstop switch
 - While holding it down send the ```QUERY_ENDSTOPS``` command again and make sure that the Z endstop says “triggered and the Y and X endstops stay open
 - If it is found that one of the endstops has inverted logic (i.e. it reads as “open” when it is pressed and “triggered” when not pressed), go into the printer configuration file (printer.cfg) and add or remove the "!" in front of the pin identifier. For example, if the X endstop was inverted, add a ! in front of the pin number as follows: ```enstop_pin: P1.28``` --> ```enstop_pin: !P1.28```
   
### XY Homing Check Preflight
 - **You need to be able to quickly stop the printer in case something goes wrong (e.g. the toolhead goes in the wrong direction)**
 - Have a computer right next to the printer with the ```RESTART``` or ```M112``` command already in the console in Mainsail. When you start homing the printer, if it goes in the wrong direction, quickly send the restart command and it will stop the printer.
 - As a 'nuclear' option, power off the printer with the power switch if something goes wrong. This is not ideal because it may corrupt the files on the SD card and to recover would require reinstalling everything from scratch

There are two ways to go about doing these checks: using the Mainsail Console or the Mainsail Dashboard

### XY Homing Check (Mainsail Console)
 - Once there is a tested process for stopping the printer in case of something going wrong, you can test X and Y movement
 - Note: you will need to test X AND Y before you can correctly determine what adjustments are needed
 - Send a ```G28 X``` command to only home X: The toolhead should move up slightly and then move to the right until it hits the X endstop
 - If it moves any other direction, **abort**, take note, but still move on to testing Y
 -  Next, test Y: run ```G28 Y```to home the Y axis; the toolhead should move to the back of the printer until it hits the Y endstop
 -  In a CoreXY configuration, both motors have to move in order to get the toolhead to go in only an X or Y direction (think Etch A Sketch)
 -  If the gantry moves downward first before moving to the right, you must reverse your z stepper directions in the config.
 -  If either axis does not move the toolhead in the expected or correct direction, refer to the table below to figure out how to correct it
 -  If you need to invert the direction of one of the motors, invert the direction pin definition by adding a "!" to the pin name. For example, ```dir_pin: PB2``` would become ```dir_pin: !PB2``` (if the entry already has a "!", remove it instead)
 -  If the motors are going in directions that match the lower row of the chart, physically swap your X and Y (A and B) motor connectors on the MCU
 -  [stepper x] = Motor B
 -  [stepper y] = Motor A

![V0-motor-configuration-guide](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/ecb2200c-a897-4ca1-92b7-a8c66c28d939)

### XY Homing Check (Mainsail Dashboard)
 - Once there is a tested process for stopping the printer in case of something going wrong, you can test X and Y movement
 - Note: you will need to test X AND Y before you can correctly determine what adjustments are needed
 - For the next few steps please refer to the image below

![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/4d4fea13-2937-42c7-9773-77193cfc1f83)

 - Click the orange "X" (circled in red) in the Toolhead menu to only home X: The toolhead should move up slightly and then move to the right until it hits the X endstop
 - If it moves any other direction, **abort**, take note, but still move on to testing Y
 - Next, test Y: Click the orange "Y" (circled in blue) in the Toolhead menu to only home Y; the toolhead should move to the back of the printer until it hits the Y endstop
 - If it moves any other direction, **abort**
 - In a CoreXY configuration, both motors have to move in order to get the toolhead to go in only an X or Y direction (think Etch A Sketch)
 -  If the gantry moves downward first before moving to the right, you must reverse your z stepper directions in the config.
 -  If either axis does not move the toolhead in the expected or correct direction, refer to the table below to figure out how to correct it
 -  If you need to invert the direction of one of the motors, invert the direction pin definition by adding a "!" to the pin name. For example, ```dir_pin: PB2``` would become ```dir_pin: !PB2``` (if the entry already has a "!", remove it instead)
 -  If the motors are going in directions that match the lower row of the chart, physically swap your X and Y (A and B) motor connectors on the MCU
 -  [stepper x] = Motor B
 -  [stepper y] = Motor A

![V0-motor-configuration-guide](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/ecb2200c-a897-4ca1-92b7-a8c66c28d939)

### Z Endstop Location
 - The Z endstop is located at the bottom of the machine at the end of the left linear rail
 - After homing Z (```G28 Z``` or clicking the orange Z button in Mainsail's Toolhead Dashboard) you can use the ```Z_ENDSTOP_CALIBRATE``` command to find the correct position_endstop value automatically
 - We will use a piece of copy paper to set the height of our nozzle relative to the endstop position (from here-on referred to as the "paper test"), do this test with your nozzle cold. When the nozzle is heated, its position (relative to the bed) changes due to thermal expansion. This thermal expansion is typically around a 100 microns, which is about the same thickness as a typical piece of printer paper. The exact amount of thermal expansion isn’t crucial, just as the exact thickness of the paper isn’t crucial. Start with the assumption that the two are equal.
 - Run the ```Z_ENDSTOP_CALIBRATE``` command, a dialog box will open that allows you to move the nozzle up and down by preset amounts
 - **The Paper Test**: Place a piece of copy paper under the nozzle and lower the nozzle in small increments, after each movement push the paper back and forth to check if the nozzle is in contact with the paper and to feel the amount of friction. Continue issuing commands until you feel a small amount of friction when testing with the paper. If too much friction is found then you can use a positive Z value to move the nozzle up.
 - Once you have found the proper height click the ACCEPT button. You then need to issue the SAVE_CONFIG command to save the value to the bottom of your config file. This will restart Mainsail.
 - This value that we just calculated is now in your config and it represents the distance from the point that the nozzle touches the bed surface to when the bed assembly triggers the z endstop switch. It also represents your maximum Z travel distance.
 - **This value can be edited manually as well and should be adjusted if using a different kind of buildplate and after bed leveling.**
   
## PID Tune Bed & Hotend
The PID tune is important for tuning the printer for a given hardware configuration to ensure that temperatures can remain as stable as possible during operation.
 - Move the nozzle to the center of the bed and approximately 5-10mm above the bed surface
 - Run: ```PID_CALIBRATE HEATER=heater_bed TARGET=100```
 - This performs a PID calibration routine that will last about 10-15 minutes. The console will not display anything while it is working but you can monitor the progress via the Dashboard temperature graph.
 -  Once finished, type ```SAVE_CONFIG``` which will save the parameters to the ```printer.cfg``` file
 - Set the part cooling fans to 25% in the dashboard or use ```M106 S64``` in the console
 - Run: ```PID_CALIBRATE HEATER=extruder TARGET=245```
 - This performs a PID calibration routine that will last about 5 minutes. Once finished, type ```SAVE_CONFIG``` which will save the parameters to the ```printer.cfg``` file

## Bed Leveling
The V0 uses manual bed leveling. The bed is small enough and thick enough that a mesh or other types of per print leveling should not be needed. There is a macro in Klipper to help with the manual bed leveling process: ```BED_SCREWS_ADJUST```

This macro will move the printer’s nozzle to each screw XY location and then move the nozzle to a Z=0.3 height. At this point one can use the “paper test” (in the Z Offset Adjustment section below) to adjust the bed screw directly under the nozzle. See the information described in “the paper test”, but adjust the bed screw instead of commanding the nozzle to different heights. Adjust the bed screw until there is a small amount of friction when pushing the paper back and forth. This process will move all three mounting points of your bed closer to the nozzle so it is critical that you re-run the Z offset adjust after completing this section.

Once the screw is adjusted so that a small amount of friction is felt, run either the ```ACCEPT``` or ```ADJUSTED``` command. Use the ```ADJUSTED``` command if the bed screw needed an adjustment (typically anything more than about 1/8th of a turn of the screw). Use the ```ACCEPT``` command if no significant adjustment is necessary. Both commands will cause the tool to proceed to the next screw. (When an ```ADJUSTED``` command is used, the tool will schedule an additional cycle of bed screw adjustments; the tool completes successfully when all bed screws are verified to not require any significant adjustments.) One can use the ```ABORT``` command to exit the tool early.

After the ```BED_SCREWS_ADJUST``` command has been completed rerun the ```Z_ENDSTOP_CALIBRATE``` command to to bring your nozzle to the correct Z=0 position.

## Z Offset Adjustment (additional methods)
You probably just completed this but if you wish to further refine it or need additional instructions you can follow the following steps
 - Home the X, Y, and Z axis then move the nozzle to the center of the bed if it is not already after homing
 - ***The Paper Test*** (additional instructions): Slowly move the nozzle toward the bed by using ```TESTZ Z=-1``` until the nozzle is relatively close to the bed, and then stepping down with ```TESTZ Z=-0.1``` until the nozzle touches a piece of paper on top of the build plate
 - If you go too far down, you can move the nozzle back up with: ```TESTZ Z=0.1```
 - Once you are satisfied with the nozzle height, run ```ACCEPT``` and then ```SAVE_CONFIG```
 - **Important**: Klipper assumes that this process is being done cold. If being performed hot, do an additional ```TESTZ Z=-0.1``` before accepting
 - If an “out of bounds” error occurs, send ```Z_ENDSTOP_CALIBRATE```, ```ACCEPT```, and then ```SAVE_CONFIG```. This will redefine the 0 bed height so you will be able to get closer.

### Fine Tuning Z Height
 - With LCD Screen: The Z offset can be adjusted during a print using the Tune menu on the display, and the printer configuration can be updated with this new value. Remember that higher values for the position_endstop means that the nozzle will be closer to the bed.
 - Mainsail: The "babystepping" controls may be used to fine tune the z-offset
 - Create Macros in your ```printer.cfg``` file so the commands are easier to remember/run
```
[gcode_macro ZUP]
gcode:
    SET_GCODE_OFFSET Z_ADJUST=0.01 MOVE=1

[gcode_macro ZDOWN]
gcode:
    SET_GCODE_OFFSET Z_ADJUST=-0.01 MOVE=1
```
 - Run ```zup``` or ```zdown``` (or the associated ```SET_GCODE_OFFSET``` command) as needed in the terminal window until you have perfected your squish
 - Run ```GET_POSITION``` and look for "gcode base". *Note the Z value*
*** - All of the above methods are “transient”. The changes are lost as soon as your printer restarts. Once you find an adjustment you are happy with, you may make it permanent, by applying it to the position_endstop in your config file: run the command ```Z_OFFSET_APPLY_ENDSTOP``` followed by ```SAVE_CONFIG```. This will restart your printer, with the adjustment permanently applied to the endstop position. ***

## Extruder Calibration (e-steps)
Before the first print, make sure that the extruder extrudes the correct amount of material.
 - First, make sure the extruder is running the correct direction: heat the hotend, and extrude 10mm or so of filament:
   - If the extruder pulls the filament in, all is well.
   - If the filament gets pushed back out the top, reverse the extruder in your ```printer.cfg``` by finding the ```[extruder] dir_pin```, and adding a ```!``` to the pin name. (if one is already present, remove it instead)
 - With the hotend at temperature, make a mark on the filament between the roll of filament and your extruder, between 120mm and 150mm away from the entrance to the extruder. Measure the distance from the entrance of the extruder to that mark.
 - Set the extrusion speed to 1mm/s, and extrude 50mm 2 times (for a total of 100mm since Klipper doesn’t allow you to extrude more than 50mm at a time)
 - Measure from the entrance of your extruder to the mark you made previously. In a perfect world, assuming the mark was at 120mm, it would measure 20mm (120mm - 20mm = 100mm), but usually won’t be.
 - Update ```rotation_distance``` in the extruder section of the configuration file using this formula:
   - New Config Value = Old Config Value * (Actual Extruded Amount/Target Extruded Amount)
*Note: a higher configuration value means that less filament is being extruded.*
 - Paste the new value into the configuration file, restart Klipper, and try again
 - Once the extrusion amount is within 0.5% of the target value (ie, 99.5-100.5mm for a target 100mm of extruded filament), the extruder is calibrated!
 - Typical rotation_distance values should be around 22.7 for Mini Stealthburner

## Setting Up A Display
There are multiple options for displays for the V0.2. The [LCD design by th0mpy](https://github.com/VoronDesign/Voron-Hardware/tree/master/V0_Display) is a great open source option and the [Waveshare 2.8" touchscreen display by hartk](https://github.com/VoronDesign/VoronUsers/tree/main/printer_mods/hartk1213/Voron0.2_2.8_WaveshareDisplay) is another. Both options have been summarized below.

# ******THIS SECTION IS STILL UNDER CONSTRUCTION DO NOT USE******
### V0 2.8" Waveshare Display (based on the [Waveshare installation instructions](https://www.waveshare.com/wiki/2.8inch_DSI_LCD) and [hartk's instructions](https://github.com/VoronDesign/VoronUsers/tree/main/printer_mods/hartk1213/Voron0.2_2.8_WaveshareDisplay))
 - SSH into the Raspberry Pi
 - Download and enter the Waveshare-DSI-LCD driver folder
```
git clone https://github.com/waveshare/Waveshare-DSI-LCD
cd Waveshare-DSI-LCD
```

 - Enter ```uname -a``` to view the kernel version of your Raspberry Pi; NOTE: waveshare screens may not be up to the latest version of your Raspberry Pi (and that's okay as long as the first two numbers and the first digit of the third number match)
 - ```cd``` to the corresponding file directory (e.g. ```cd 6.6.20```)
 - We know we set up our Raspberry Pi as a 32-bit system when we first flashed the operating system, so enter the ```32``` directory for 32-bit systems via ```cd 32```
 - Enter your corresponding model command to install the driver, pay attention to the selection of the I2C DIP switch
#2.8inch DSI LCD 480×640：```sudo bash ./WS_xinchDSI_MAIN.sh 28 I2C0```
 - Wait for a few seconds, when the driver installation is complete and no error is prompted, restart and load the DSI driver and it can be used normally by entering: ```sudo reboot```
 - make sure everything is fully up to date on your raspberry pi by running
```
sudo apt-get update
sudo apt-get full-upgrade
```
 - After the drivers are installed run the following command to open up the config.txt for the raspberry pi ```sudo nano /boot/firmware/config.txt```
 - During driver installation above you added this line:
```dtoverlay=WS_xinchDSI_Touch,invertedy,swappedxy,I2C_bus=10```
which needs to be updated to this:
```dtoverlay=WS_xinchDSI_Touch,invertedy,invertedx,I2C_bus=10```
 - Save and exit the config.txt file by pressing ```control+x``` and then ```y```
 - Go to this directory by ```cd /usr/share/X11/xorg.conf.d/``` and run
```git clone https://github.com/VoronDesign/VoronUsers/blob/main/printer_mods/hartk1213/Voron0.2_2.8_WaveshareDisplay/Software/90-monitor.conf```
 - if this command fails, you'll have to either copy the file to the directory using something like WinSCP to drag and drop the file into the directory OR you can make it yourself (directions follow)
 - Run ```sudo nano 90-monitor.conf``` which creates the file and opens the "nano" text editor
 - Copy the following into the file (if using Putty you can right click)
```
Section "Monitor"
    Identifier "DSI-1"
    # This identifier would be the same as the name of the connector printed by$
    # it can be "HDMI-0" "DisplayPort-0", "DSI-0", "DVI-0", "DPI-0" etc

    Option "Rotate" "left"
    # Valid rotation options are normal,inverted,left,right


    Option "PreferredMode" "640x480"
    # May be necesary if you are not getting your prefered resolution.
EndSection
```
 - Save and exit the config.txt file by pressing ```control+x``` and then ```y```

# ******END CONSTRUCTION SECTION THE INSTRUCTIONS BELOW WORK FINE******

### V0 LCD Display ([adapted from Mr Doctor Professor Patrick's Github](https://github.com/VoronDesign/Voron-Hardware/tree/master/V0_Display))
 - There are tons of sources for this display, pretty much any of them are fine or you can even build it yourself! Just don't buy from Blurolls ([explanation why on the Voron Discord](https://discord.com/channels/460117602945990666/696930677161197640/919787940807340072))
 - My guide assumes the display has not had firmware flashed to it previously to avoid confusion
 - Connect the board to the host Raspberry Pi via USB
 - Connect to your host raspberry pi via SSH
 - Run ```lsusb``` from the command prompt
   ![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/573e894b-267e-4e76-b37d-7d061f97dbfb)

 - Make sure you see an ```STM32``` in DFU mode listed
 -  - Write down the Device ID it will be in the format "xxxx:yyyy"
   ![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/6d99a8e3-3168-4aea-bd00-b9d8708fe18d)

 - If you don't already see the Device ID, Run ```dfu-util --list``` from the command prompt
   ![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/ee2562e5-d8bf-473a-bea8-9816e7fa8b8d)

 - Write down the Device ID
 - Run ```cd ~/klipper``` from the command line to enter the Klipper directory
 - Run ```make menuconfig``` and adjust the following
   - Micro-controller Architecture to ```STMicroelectronics STM32```
   - Processor model ```STM32F042```
   - Bootloader offset ```No bootloader```
   - Clock Reference ```Internal Clock```
   - Communication interface ```USB (on PA9/PA10))```
  - Set the "Optional features" to:
![Menuconfig_Optional_Options](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/6bdc7830-f5f2-4e50-bc12-f10ecbd51aa8)

 - If there additional options leave them unsupported (unchecked)
 - Hit Q to Exit and Save the settings when prompted
 - Run ```make clean``` to clean up the make environment
 - Run ```make flash FLASH_DEVICE=xxxx:yyyy``` (using xxxx:yyyy from above)
 - You may see what appears to be an "error" after flashing your board. (Blue box)
As long as you see the File downloded successfully text (Green box) you are good to proceed.
The error (Red box) seems to be caused by the controller immediately running the uploaded code and no longer appearing as a DFU device. This is not an issue, as long as the board reports a Klipper serial name. 

![dfu-util_Flashing_Error](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/637b603b-785c-4c88-a976-68beffb6b27f)

 - Remove the boot jumper and unplug the USB cord from the Raspberry Pi then plug it back in
 - Run ```ls /dev/serial/by-id/*``` should return a device begining with ```/dev/serial/by-id/usb-Klipper_stm32f042x6...```
 - Copy this serial port name (/dev/serial/by-id/usb-Klipper_stm32f042x6... )and place it in your [mcu display] section of the [display config file (download here)](https://github.com/VoronDesign/Voron-Hardware/blob/master/V0_Display/Software/V0Display.cfg)
 - Make sure to include the full path to the file just like in your ```printer.cfg``` file earlier when mapping the skr pico and picobilical
 - In Mainsail, upload the V0Display.cfg file under the Machine menu (Wrench icon) by drag and dropping the file
 - Add ```[include V0Display.cfg]``` to ```printer.cfg``` to include the file
   ![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/153e063a-0a69-4f9b-8005-7571a67b4484)

 - Restart the Firmware and the display should be functional!

# Changing Wifi Networks
The V0.2 is a great travel printer, but getting it connected to different Wifi networks is a bit of a pain. There are two ways you can go about it.

## Printer already connected to a network
There are two ways to go about this depending on if the new network you want to connect to is within range of the printer or not

### Within range of the new network
 - SSH into your printer
 - Run ```sudo raspi-config```
 - Press "Enter" to select "System Options"
 - Press "Enter" to select "S1 Wireless LAN"
 - Type in the SSID (network name) of your network then press "Enter"
 - Type in the password for your network then press "Enter"
 - If successful, navigate to the following directory s```cd /etc/NetworkManager/system-connections/``` and run the ```ls``` command and a file with called ```"Your Network SSID".nmconnection``` should be there
 - Check to make sure you are connected to the network by running the ```ifconfig``` command which should display something like this
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/43cdb363-2b9d-4ff3-94f4-2fd8844d604e)

### Out of range of the new network
 - SSH into your printer
 - Run ```cd /etc/NetworkManager/system-connections/ && ls```
 - Create the file for your network using the following command but replace the "Current Network Name" and "New Network SSID" portions as needed
```sudo cp "Current Network Name".nmconnection "New Network SSID.nmconnection ```
 - Adjust the following lines :
```
id="Your Network SSID"
ssid="Your Network SSID"
psk="Your Network Password"
```
 - Here is a sample version with the network name being "BubbaGumpShrimpComp" and the Password being "SuperSecretPassword"
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/8575d6de-19a7-4b23-aa44-9a86bee70385)

## Printer is not connected to a network
### Within range of the new network
 - Requires a screen to connect to the Rasbperry Pi (via HDMI (pi3) or mini HDMI (pi4) and a USB Keyboard)
 - Plug the Raspberry Pi into the display screen
 - Plug the keyboard into the Raspberry Pi
 - Turn on the Printer and wait for the command line to finish loading
 - Run ```sudo raspi-config```
 - Press "Enter" to select "System Options"
 - Press "Enter" to select "S1 Wireless LAN"
 - Type in the SSID (network name) of your network then press "Enter"
 - Type in the password for your network then press "Enter"
 - If successful, navigate to the following directory ```cd /etc/NetworkManager/system-connections/``` and run the ```ls``` command and a file with called ```"Your Network SSID".nmconnection``` should be there
 - Check to make sure you are connected to the network by running the ```ifconfig``` command which should display something like this
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/43cdb363-2b9d-4ff3-94f4-2fd8844d604e)

### Out of range of the new network
 - Requires a screen to connect to the Rasbperry Pi (via HDMI (pi3) or mini HDMI (pi4) and a USB Keyboard)
 - Plug the Raspberry Pi into the display screen
 - Plug the keyboard into the Raspberry Pi
 - Turn on the Printer and wait for the command line to finish loading
 - Run ```cd /etc/NetworkManager/system-connections/ && ls```
 - You should see a file named something like "preconfigured.nmconnection"
 - Create the file for your network using the following command but replace the "Current Network Name" and "New Network SSID" portions as needed ```sudo cp "Current Network Name".nmconnection "New Network SSID.nmconnection ```
 - Adjust the following lines :
```
id="Your Network SSID"
ssid="Your Network SSID"
psk="Your Network Password"
```
 - Here is a sample version with the network name being "BubbaGumpShrimpComp" and the Password being "SuperSecretPassword"
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/8575d6de-19a7-4b23-aa44-9a86bee70385)

# Input Shaping (optional but recommended)
Configuring Input Shaping allows you to print at much higher speeds by compensating for vibrations caused by the movement of the toolhead. 
 - This is a great video demonstrating [how input shaping works in a large scale system](https://www.youtube.com/watch?v=gzBhTrHv0-c)
 - This is a ;tldr style explanation I found on Reddit:
> Rapid printer movements create vibrations. This means that the printer is trying to make a movement, but because physics exists it'll actually wobble while doing so. This is visible in the printed object, and people really don't like it. The easiest way to avoid this is to slow down your printer. Alternatively, you can do input shaping [...] means the printer knows how much it is going to wobble, so it can take the original input and slightly modify it in such a way that the wobbles are countered! This allows you to speed up the print without getting ugly wobble artifacts.

Basically, it allows you to print faster without causing defects in the objects you print! Again this is specifically for the LDO Voron 0.2s1r1 kit with Raspberry Pi, SKR Pico, and Picobilical boards. These instructions are adapted from the [LDO Documentation](https://docs.ldomotors.com/adxl_tool) and the [Klipper Documentation](https://www.klipper3d.org/Measuring_Resonances.html).

## Setting Up Input Shaping

### Hardware Setup
 - Connect the FFC ribbon cable to each PCB (Raspberry Pi & Toolhead PCB)
    - Carefully pull up the black tab on the FFC connector
    - Insert the FFC ribbon cable and push the tab back in
    - Take care when pulling up the black tab! It is fragile and will break if pulled with excessive force
    - Make sure the polarity of the ribbon cable is correct
    - The ribbon cable is **NOT** to be run in cable chains and should be connected directly for input shaping and removed afterwards
    - Be sure the ribbon cable is facing the right direction during installation:
   ![20240603_142606](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/e986f768-7068-4f7a-9de4-546ba5b3fb1a)
   ![20240603_142600](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/a2a958ad-7b09-43bb-8690-40be42ac291b)

 ### Software Setup
  - Before you can start running input shaper calibration, you need to install a few software packages on your Raspberry Pi
  - SSH into your printer
  - Run the following to install the rc script which allows you to use the host as a secondary MCU:
```
cd ~/klipper/
sudo cp ./scripts/klipper-mcu.service /etc/systemd/system/
sudo systemctl enable klipper-mcu.service
```
 - Compile the Klipper micro-controller code, by configuring for the "Linux process":
```
cd ~/klipper/
make menuconfig
```
 - Set the "Microcontroller Architecture" to "Linux Process" then save and exit
 - Build and install the new micro-controller code by running:
```
sudo service klipper stop
make flash
sudo service klipper start
```
 - If klippy.log reports a "Permission denied" error when attempting to connect to ```/tmp/klipper_host_mcu``` then you need to add your user to the tty group. The following command will add the "pi" user to the tty group ```sudo usermod -a -G tty pi```
 - Run ``` sudo raspi-config``` and enable SPI under the "Interfacing Options" menu
 - Run
```sudo apt update
sudo apt install python3-numpy python3-matplotlib libatlas-base-dev libopenblas-dev
```
 - Then run: ```~/klippy-env/bin/pip install -v numpy```
 - Add the following to the ```printer.cfg``` file:
```
[mcu rpi]
serial: /tmp/klipper_host_mcu

[adxl345]
cs_pin: rpi:None

[resonance_tester]
accel_chip: adxl345
probe_points:
    60, 60, 20  # an example
```
 - It is advised to start with 1 probe point, in the middle of the print bed, slightly above it then restart the Printer Firmware in Mainsail
 - Check your setup by running the command ```ACCELEROMETER_QUERY``` in Mainsail. If everything is correct, you should see some measurements from the accelerometer on the console output

 ### Input Shaper Calibration
  - The easiest way to perform calibration is to simply run the command ```SHAPER_CALIBRATE``` in Mainsail. For more details and advanced commands and settings see [Klipper Documentation](https://www.klipper3d.org/Measuring_Resonances.html#input-shaper-auto-calibration)
 - This process takes ~5 minutes and at the end of it you will be prompted to run ```SAVE_CONFIG``` which will save the input shaper settings to your printer.cfg file an restart your firmware
 - You can now unplug the ribbon cable from the Raspberry Pi and the Toolhead PCB
 - Congrats! You have completed tuning your input shaping!

# Raspberry Pi Camera Installation & Setup (optional)
Installing a camera provides multiple advantages for your 3D printing experience. It can be used to monitor ongoing prints, record timelapse videos, and more. For this guide we'll be using an interface called Crowsnest and an official Raspberry Pi V2.1 camera. You'll need a 15 pin ribbon cable that is ~12" or longer, the camera unit, an m3 bolt (most mounts use 12-18mm lengths), and an m3 hexnut.

## Hardware Installation
### Mount Options
Multiple options for mounting the camera are out there here are a few that I like:
 - [For RPi V2 but there are multiple remixes for other cameras like Arducams](https://www.printables.com/model/146877-voron-0-voron-01-raspberry-pi-camera-mount)
 - [Corner Mount](https://www.printables.com/model/149493-voron-v01-raspberry-pi-camera-module-bracket)
 - [Between Z extrusions mount](https://www.printables.com/model/370373-voron-01-voron-02-raspberry-pi-camera-module-3-mou)
### Installation
 - Plug the ribbon cable into the Pi Camera making sure the contacts are facing towards the camera lens
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/d9f5d130-b3b4-4c39-80c9-1751e388ea25)
 - Screw the camera into the 3D printed mount (if required) using m2 self-tapping screws
 - (if applicable) Screw the m3 bolt (12-18mm) through the mount and begin the threading it into the m3 hexnut
 - Gently let the camera and mount hang and slip the ribbon cable between the back panel and the horizontal extrusion 
 - Plug the ribbon cable into the Raspberry Pi camera port with the contacts facing towards the hdmi/mini hdmi port
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/12ba8a46-3070-42b2-ad64-bee36b5cd553)
 - Install the mounted camera into the proper extrusion and (if applicable) tighten the m3 bolt to properly secure the camera

## Setup ([based on the official Crowsnest installation instructions](https://crowsnest.mainsail.xyz/configuration/sample-config))
 - SSH into your Raspberry Pi 
 - Use KIAUH to install Crowsnest by using the following command. ```./kiauh/kiauh.sh```
 - Install Crowsnest using KIAUH; to choose an action, simply type the corresponding number into the "Perform action" prompt and confirm by hitting ENTER
 - Crowsnest installation may take a few minutes to complete, if it prompts you for any decisions like asking for configuring the update manager in moonraker.conf; Select Yes and proceed with installation.
 - At the end of the installation, you can see a message indicating the process's success.
 - Do a hard restart of the entire printer (flip the power switch on the back), wait ~30seconds, then turn it back on
 - SSH into your Raspberry Pi again and navigate to the crowsnest logfile by running
```
cd /home/owner/printer_data/logs/
less crowsnest.log
```
 - Use the ENTER key to navigate through the file until you find the lines with the red underline and circled blue sections like in the image below:
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/34cb0d3c-edbb-435c-95b5-504682ca8bf2)
 - Record the full path to your camera (underlined in red in the image above)
 - Record the resolution and fps options of your camera (circled in blue in the image above)
 - Open Mainsail and go to the Machine (wrench) tab
 - Open the ```crowsnest.conf``` file
 - This config file is quite short luckily! There are only 8 lines you should adjust
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/59e7b66e-1c6e-41ed-bdce-650867ce8235)
 - Change the name of your camera (line 37 in the image above) to whatever you want but leave the "cam" portion alone as it is required for the configuration to work properly; the default name is [cam 1] but has been changed to [rpiv2.1] in the image above
 - Change the "mode" (line 38 in the image above) to your desired method but be sure your chosen camera supports that mode
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/0610a03f-ff92-4c43-a925-f4a37104f20e)
 - Change "enable_rtsp" (line 40 in the image above) to ```true``` if you chose ```camera-streamer``` for your mode
 - Change the "device" location (line 43 in the image above) to the path you recorded earlier from the ```crowsnest.log``` file
 - Change the "resolution" (line 44 in the image above) to your desired resolution based on the options you recorded earlier fom the ```crowsnest.log``` file
 - Change the "fps" (line 45 in the image above) to your desired frames per second based on the options you recorded earlier fom the ```crowsnest.log``` file
 - Click "Save and Restart" to return to the Machine (wrench) tab of Mainsail
 - Click the settings gears, then the "Webcams" tab, then click "Add Webcam" (NOTE: in the below images the camera has already been added and the "Add Webcam" button is beneath that to the right)
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/709ba564-2780-4d10-a902-4f3c59563a93)
![image](https://github.com/mmcnair91/ACEAE-Labs/assets/62910185/e622f369-3ca4-4482-804a-2634f6fdcced)
 - Name your webcam whatever you want; it does not have to be the same as in the ```crowsnest.conf``` file
 - If you chose ```camera-streamer``` for your mode, replace the path in the "URL Stream" box which defaults to ```/webcam/?action=stream``` with ```/webcam/webrtc```
 - If you chose ```camera-streamer``` for your mode, select ```WebRTC (camera-streamer)``` from the drop down menu in the "Service" box
 - If you chose ```ustreamer``` for your mode leave the path in the "URL Stream" box as the default which should be ```/webcam/?action=stream```
 - If you chose ```ustreamer``` for your mode, select ```Adaptive MJPEG-Streamer (experimental)``` from the drop down menu in the "Service" box
 - If you chose ```ustreamer``` for your mode, type your desired frames per second (fps) in the "Target FPS" field this should not exceed the value you put in the ```crowsnest.conf``` file
 - Click "Save Webcam"
 - If you want your webcam view in the Mainsail Dashboard, click the settings gears, then the "Dashboard" tab, then enable "Webcam" for whatever platform you want (desktop, mobile, tablet, or widescreen)
 - Congratulations! You should now have a live view of your printer!

# USB to Print (Requires a functioning display) [adapted from ShiftingTech's github](https://github.com/shiftingtech/Moonraker-loader/tree/main)
Klipper is designed to run on a network but some of us like to work offline. This modification allows you to save the gcode in the root (base) directory of a usb stick, plug the usb stick into your Raspberry Pi or USB extension cable and then copies all the files in that directory to the virtual SD card set up in the ```Mainsail.cfg``` file. It even shows a "Copy Complete" message on the display when it's done!
## Software Setup
 - SSH into your Raspberry Pi
 - Run the following commands:
```
git clone https://github.com/shiftingtech/Moonraker-loader.git
sudo ln -sf ~/Moonraker-loader/assets/89-moonraker-loader.rules /etc/udev/rules.d 
sudo ln -sf ~/Moonraker-loader/assets/*.sh /usr/local/sbin
```
 - Restart the Raspberry Pi
 - You should now be able to plug a USB stick/drive/thumbdrive/whatever the kids are calling it these days into a USB port on the Raspberry Pi and all the files should copy over automatically
 - You can now remove the USB once the "Copy Complete" message displays on your screen
 - That's it!
