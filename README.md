# VFA fix summary, firmware compilation, and shopping list

VFA's are a solved issue with 0.9 degree stepper motors. This post is a summary of what is needed to use 0.9 degree motors.


## Firmware
You will need this 0.9 degree motor firmware (now includes BMG extruder support)

Be able to compile this firmware before doing any hardware changes. Once motors have been changed, the first thing you need to do is update the firmware to this 0.9 motor support version.


### Compiling my firmware for 0.9 degree motor support
You may wish to refer to the Prusa README.md document for more detailed instructions regarding compilation of the firmware. 
Warning: You are definitely in experimental firmware territory here.

#### Obtain Arduino IDE

Visit https://www.arduino.cc/en/Main/Software and download the Arduino IDE for your OS. 
I successfully use Arduino 1.8.8 for OSX. 
Prusa instructions mention their internally using version 1.8.5
Install Arduino compiler on your computer

#### Prepare Arduino IDE to handle EINSY board
As downloaded, the Arduino IDE does not know about the EINSY RAMBO board. You must adjust some IDE settings to download board info from Ultimachine.

1. Launch Arduino.

2. In Preferences -> Settings Tab

Additional Boards Manager URLs textfield enter...
```
https://raw.githubusercontent.com/ultimachine/ArduinoAddons/master/package_ultimachine_index.json
```

3. Accept (OK) new preference setting

4. Tools -> Board -> Boards Manager
Select the RAMBo board, which is listed something like "RepRap Arduino-compabilty Mother Board (RAMBo) by Ultimachine"
Board info will download become noted as "Installed." Depending on server load this can take anywhere from seconds to minutes.

You many need to "update" the board to get the latest version. That is 1.0.1 as of this writing.

5. Close Board Manager

6. Tools -> Board, select RAMBo as target board. Do not select any other board.

7. QUIT Arduino IDE

8. Set compiler flags in platform.txt for newly installed RAMBo board
Use your OS file search function to find platform.txt

Under OSX, it will be in
```
~/Library/Arduino15/packages/rambo/hardware/avr/1.0.1
```
Open platform.txt

Add "-Wl,-u,vfprintf -lprintf_flt -lm" to "compiler.c.elf.flags=" before existing flag "-Wl,--gc-sections"
On my system that meant adding the following line in platform.txt

```
compiler.c.elf.flags=-w -Os -Wl,-u,vfprintf -lprintf_flt -lm -Wl,--gc-sections
```
Save your changes to platform.txt

#### Obtain firmware files.

0.9 degree motors require my firmware branch that contains 0.9 degree stepper support (which is this page)
https://github.com/guykuo/Prusa-Firmware/tree/0.9-Degree-Stepper-Support
Verify you are in my 0.9 Degree Stepper Support branch

Click on "Clone or Download" and DOWNLOAD ZIP to your computer

Unzip the newly downloaded Prusa-Firmware-0.9-Degree-Stepper-Support.zip

Inside the unzipped folder "Prusa-Firmware-0.9-Degree-Stepper-Support" you will find a folder named "Firmware"
That folder contains the firmware files you need. Place the firmware folder where you wish to keep it on your drive.

#### Set Printer Variant

Use the correct variant file for your printer model.
In Firmware/variants you will find several header (.h) files. Each printer model has one specific file.

Mk3 needs 1_75mm_MK3-EINSy10a-E3Dv6full.h

Mk3s needs 1_75mm_MK3S-EINSy10a-E3Dv6full.h

Choose the correct variant file for your printer model!!!!!!

Rename the variant file for your printer to be...

Configuration_prusa.h

Move Configuration_prusa.h to the main firmware folder. 


#### Set Language Support (already done for you in my branch)
Normally you would need to set language support in config.h to primary language only.
I have already done so in my branch. So you do not need to do this. 

However, if you build from a Prusa branch you MUST do this. 
Otherwise, firmware will not run properly. Instead, your LCD will display random letters and likely boot loop.

The changes needed are near bottom of the file. It should look like this...
```
//LANG - Multi-language support
#define LANG_MODE 0 // primary language only
//#define LANG_MODE 1 // sec. language support
#define LANG_SIZE_RESERVED 0x2f00 // reserved space for secondary language (12032 bytes)
```
You can make the edit from within Arduino IDE, but again, this has already been done for you in my branch.


#### Set Motor Defines, Compile, Upload

My firmware branch includes changes for 0.9 degree motor support. All the important edits have been tagged with a comment.
Search for Kuo in all sketch tabs to view my changes.

For most users, you only need to specify which motors are 0.9 degree units.

1. Launch Arduino IDE

2. Open the Firmware.ino file which is inside your firmware folder

3. Select the Configuration_prusa.h tab

4. You must adjust defines in Configuration_prusa.h to match your actual motor setup for X Y and E axes. Also, if you have a BMG extruder,  uncomment #define BMG_EXTRUDER

Look for...

```
/*------------------------------------
 AXIS SETTINGS
 *------------------------------------*/
//Uncomment def(s) below for 0.9 degree stepper motors on x, y, e axis
//Motors used should be 1 amp or lower current rating to avoid overheating TMC2130 drivers in Stealthchop.
//My recommended 0.9 degree motors for X, Y, or direct drive E are Moons MS17HA2P4100 or OMC 17HM15-0904S 
#define X_AXIS_MOTOR_09 //kuo exper
#define Y_AXIS_MOTOR_09 //kuo exper
#define E_AXIS_MOTOR_09 //kuo exper

//Uncomment below for BMG Extruder
//#define BMG_EXTRUDER //kuo exper implements changes based on Chris Warkocki BMG firmware mods

```

My _AXIS_MOTOR_09 defines are probably all you need to modify. The rest of my firmware changes are controlled by these defines.

Uncomment only the axes that you want to be 0.9 degree motors. In the above example, x and y are set to be 0.9 degree motors, but not the extruder.

Save your changes

5. Test compile with Sketch -> Verify/Compile
Compiler will complete the job. 
If you see a warning about a missing bootloader, you probably have an older, RAMBo board 1.0.0 definition installed as your target board.

6. To compile and upload to the printer, connect your computer to the USB port of EINSY. 
Sketch -> Upload
The firmware will compile and upload to printer. Do NOT interrupt the update!!!! Let it complete.

You may also need to use Tools --> Port to select the correct USB port of your computer.





## Hardware
### Motors
Select either below listed Moons or OMC 0.9 degree steppers. Both dramatically reduce VFA's compared to any 1.8 degree motor. 

OMC motor is 1/2 cost of Moons but requires soldering of cable adapter. Moon's are plug-in compatible with Yotino cable harness.

Moon's are best at reducing VFA's with linearity correction OFF. They do not tune better with linearity correction.

OMC's are baseline slightly worse than Moons, but can be tuned to achieve better than Moons with linearity correction (1.130 - 1.140). However, forget to set linearity correction and the OMCs are worse than Moons. 

NB: Prusa firmware does not store linearity correction settings to EEPROM unless you let the menu time out by itself.

OMC's are a little bit louder during printing, but not by much. 

Both choices of motor require grinding a shaft flat for the y-axis.

It's a matter of cost, install effort, and whether one can be bothered to set linearity correction (once) for X and Y. Either motor choice greatly reduces VFA's.

[Moons Stepper Motors](https://www.amazon.com/MOONS-Stepper-Accuracy-Cable00723-MS17HA2P4100/dp/B072HT8PLM)	$53 x 2
Moons 0.9	MS17HA2P4100 
Voltage	3.9 volt
Resistance	3.9 ohm
Current	1.0 A
Hold Torque	0.39 Nm
Weight	290 gm
inductance	11.2 mH

or

[OMC Stepper Motors](https://www.amazon.com/gp/product/B00W98OYE4)	$20 x 2
STEPPERONLINE 0.9° OMC 17HM15-0904S
Voltage: 12-24V
Resistance	6.0ohms
Hold Torque 0.36 Nm
Current	0.9A 
Weight	280 gm
Inductance	12.0mH

[JST-PH Connector Kit](https://www.amazon.com/gp/product/B07DCL9J8K)	$15
2.0mm JST-PH Connector Kit, with JST-PH 5/6/7 Pin Housing 
6 pin connector to adapt OMC motor wires to harness.
Optionally, skip these connectors and directly solder motor wires to harness
I prefer to add standardized connector because I test multiple motors.



### Heatsinks for TMC2130's
Obtain heatsinks for TMZ2130's x all 4 axes. May as well cool Z and E while taking care of X & Y)
Required to avoid 0.9 motors overheating TMC2130 drivers, especially if run in Stealth mode. Heatsinks attach to NON-COMPONENT side of EINSY PC board. They do NOT attach on the TMC2130 chips themselves, but to thermal vias on empty side of EINSY board. Must also cut holes in back of EINSY case for fitment & adequate ventilation. Clean PC board with IPA to let self-stick thermal tape do its job. Position on the vias!.

[Driver Heatsinks](https://www.amazon.com/gp/product/B07GWR3TWZ)	$9
Driver Heatsinks for TMC2130 (12 pcs)
Use these or similar self-stick, driver heatsinks for stock EINSY enclosure. 

or

Reverse orientation EINSY case users must use low profile heatsinks because reverse case orients heatsinks towards heatbed. Extruder cable will catch on high profile heatsinks. Use low profile, Pi heatsinks.

[Raspberry Pi Heatsink Kit](https://www.amazon.com/gp/product/B07217N5LS)	$8
Raspberry Pi Heatsink Kit (lower profile than usual driver heatsinks)
I use these Pi heatsinks with my reverse EINSY case. Smaller and lower profile but still get the job done.

### Motor Cables
[Stepper Motor Cables](https://www.amazon.com/gp/product/B07CBV8DVZ)	$7
YOTINO Bipolar Stepper Motor Cables, 4 x 100cm Long NEMA 17 Extended Connector Cable (XH2.54 4Pin-6Pin)


### Drive Pulleys (optional but recommended)
[10mm Drive Pulley for GT2](https://www.amazon.com/gp/product/B07BH26P2D) $8.90 for 4
BALITENSEN GT2 Timing Pulley 16 Teeth 5mm Bore, Width 10mm for GT2 Belt 
(optional) Replace X and Y motor pulleys with these to reduce 2mm, vertical GT2 tooth artifact. This particular drive pulley yielded lower tooth engagement 2mm artifact during 1st phase testing.
