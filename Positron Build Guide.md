# Positron V3.2 Build Log - LDO Kit Batch 1
Complaints as I build:
 - LDO build guide is pretty good and easy to follow; however, I had a number of issues with the images not loading, links to photos of parts redirecting to a different part of the guide, and/or the photos not be correct for the hardware it was supposed to represent.
 - The physical parts checklist is a nice touch, but having a QR code that takes you to the build manual on LDO or Positron's website would be nice.
 - The brass standoffs for the mainboard shear off far too easily even just using the provided hex wrench.
 - If printing in pulltruded PET filament dialing in your extrusion multiplier will greatly improve part tolerances; however, for small, fine parts scaling up the parts by 1-5% can really help with getting hardware to fit properly (looking at you glass bed mounts)

## Initial Checks
- turn on the printer
- Using the touchscreen select the settings gear-wheel with ```More``` under it
- Select the ```Network``` and enable the built in wifi by touching the

## Z-Endstop Calibration
- Manually drop the Z height to Z=0 then manually measure the height from the tip of the nozzle to the bed with a ruler or calipers if you have them. After my build was complete, my Z=0 was ~10mm away from the bed initially. 
- In the ```printer.cfg``` file there is a setting called ```position_min``` under the Z-Stepper section (image below). 
![image](https://github.com/user-attachments/assets/b7a3c2a1-29a6-49a6-a4da-f3013b63fc24)
- Set it to "**-**(*your measured value here*)-1" so in my case ```position_min= -11```. This ensures that you will be able to drop the bed all the way until it touches the nozzle and you can properly perform the paper test.
- Home all using the Tool menu in the home screen or typing ```G28``` in the console
![image](https://github.com/user-attachments/assets/728499ee-7b75-4ba9-ab08-64dd74b176e8)
- Move the nozzle to the center of the bed (~X=95)
- Move the toolhead to Y=0
- Perform **The Paper Test**: Place a piece of copy paper under the nozzle and lower the nozzle in small increments, after each movement push the paper back and forth to check if the nozzle is in contact with the paper and to feel the amount of friction. Continue issuing commands until you feel a small amount of friction when testing with the paper. If too much friction is found then you can use a positive Z value to move the nozzle up.
- Once you are happy with the position of the nozzle relative to the bed (just barely touching based on the paper test) type ```Z_OFFSET_APPLY_ENDSTOP``` in the console and then issue the ```SAVE_CONFIG``` command in the console to save the value to the bottom of your config file. Restart the firmware to apply the changes. This value that we just calculated is now in your config!

## PID Tune Bed & Hotend
The PID tune is important for tuning the printer for a given hardware configuration to ensure that temperatures can remain as stable as possible during operation.
 - Move the nozzle to the center of the bed and approximately 5-10mm above the bed surface
 - Run: ```PID_CALIBRATE HEATER=heater_bed TARGET=80```
 - This performs a PID calibration routine that will last about 10-15 minutes. The console will not display anything while it is working but you can monitor the progress via the Dashboard temperature graph.
 -  Once finished, type ```SAVE_CONFIG``` which will save the parameters to the ```printer.cfg``` file
 - Set the part cooling fans to 25% in the dashboard or use ```M106 S64``` in the console
 - Run: ```PID_CALIBRATE HEATER=extruder TARGET=245```
 - This performs a PID calibration routine that will last about 5 minutes. Once finished, type ```SAVE_CONFIG``` which will save the parameters to the ```printer.cfg``` file

## Bed Leveling
The Positron uses manual bed leveling. There is a macro in Klipper to help with the manual bed leveling process: ```BED_SCREWS_ADJUST```

This macro will move the printer’s nozzle to each screw XY location and the rear center of the bed and then move the nozzle to a Z=0.3 height. At this point one can use the “paper test” (in the Z Offset Adjustment section above) to adjust the bed screw for that portion of the bed. See the information described in “the paper test”, but adjust the bed screw instead of commanding the nozzle to different heights. Adjust the bed screw until there is a small amount of friction when pushing the paper back and forth. This process will move all three mounting points of your bed closer to the nozzle so it is critical that you re-run the Z offset adjust after completing this section.

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
