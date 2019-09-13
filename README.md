# VFA fix summary, firmware compilation, and shopping list

VFA's are a solved issue with 0.9 degree stepper motors. This post is a summary of what is needed to use 0.9 degree motors.

# Now based on Prusa 3.8.0 (actually the newer than release, MK3 branch)
This branch adds support for...

0.9 degree motors on XYZ

Geared Extruders

Slice thermistor

Slice Magnum Mosquito

E3D Volcano

Linear Advance 1.5

Filament load/unload and z dimension changes for Slice magnum, Bondtech Prusa Upgrade MK3 & MK3S extruders, Skelestruder

# MUST UPDATE RAMBO BOARD In ARDUINO IDE TO PRUSA RESEARCH VERSION!!!!!!
As of 3.8.0, Prusa moved to new board definition. You can no longer compile for the Ultimachines board. New instructions for obtaining Prusa version are in compilation directions below.



## Firmware
You will need this 0.9 degree motor firmware.

Be able to compile this firmware before doing any hardware changes. Once motors have been changed, the first thing you need to do is update the firmware to this 0.9 motor support version.

Extruder microstep rate has been reduced to avoid overrunning EINSY. E-steps are now 1/2 of what was needed for prior BNB firmware. 

Set e-steps to new values via terminal window.
-
PLEASE DO A FACTORY RESET WITH DATA ERASURE after installing this firmware. Failure to clear out old EEPROM setttings can produce odd printer behavior.
-

### Compiling my firmware for 0.9 degree motor support
You may wish to refer to the Prusa README.md document for more detailed instructions regarding compilation of the firmware. 
Warning: You are in experimental firmware territory here.

#### Obtain Arduino IDE

Visit https://www.arduino.cc/en/Main/Software and download the Arduino IDE for your OS. 
I successfully use Arduino 1.8.9 for OSX. 
Prusa instructions mention they internally using version 1.8.5
Install Arduino compiler on your computer

#### Prepare Arduino IDE to handle EINSY board
As downloaded, the Arduino IDE does not know about the EINSY RAMBO board. You must adjust some IDE settings to download board info from Prusa (no longer Ultimachines).

1. Launch Arduino.

2. In Preferences -> Settings Tab

Additional Boards Manager URLs textfield enter...
```
https://raw.githubusercontent.com/prusa3d/Arduino_Boards/master/IDE_Board_Manager/package_prusa3d_index.json
```

3. Accept (OK) new preference setting

4. Tools -> Board -> Boards Manager

5. Select and install the Prusa Research AVR MK3 RAMBo EINSY board
Board info will download become noted as "Installed." Depending on server load this can take anywhere from seconds to minutes.

You many need to "update" the board to get the latest version. That is 1.0.1 as of this writing.

6. Close Board Manager

7. Tools -> Board, select PrusaResearch EINSY RAMBo as target board. Do not select any other board.

8. QUIT Arduino IDE

9. (Setting compiler flags step appears to be already done by Prusa. So this step may no longer be needed)
Set compiler flags in platform.txt for newly installed RAMBo board
Use your OS file search function to find platform.txt

Under OSX, it will be in
```
~/Library/Arduino15/packages/PrusaResearchRambo/hardware/avr/1.0.1/platform.txt
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
Verify you are in my 0.9-Degree-Stepper-Support branch

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


#### Set Language Support
(This has already been done for you in my Prusa-Firmware-MK3-3.8.0-with-0.9-motors-LA15-Slice-Skelestruder-BMG branch)
Language support in config.h must be primary language only. Otherwise, firmware will not run properly. Instead, your LCD will display random letters and likely boot loop.

The changes needed are near bottom of the file. It should look like this...
```
//LANG - Multi-language support
#define LANG_MODE 0 //Kuo primary language only
//#define LANG_MODE 1 // sec. language support
#define LANG_SIZE_RESERVED 0x2f00 // reserved space for secondary language (12032 bytes)
```
You can make the edit from within Arduino IDE.


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
//Geared extruders now set for lower microstepping to avoid overruning EINSY during fast retracts or MMU2S filament moves. 
//Factory reset and delete all data after installing this firmware. Otherwise EEPROM settings override settings in this firmware.
//After installing this firmware, send M350 and M92 commands to force correct micro-stepping and e-step rates. M350 must be first
//because M350 command will sometimes alter existing M92 setting
//
//e-steps values for M92 depend on your extruder gearing.
//xxx = 280 for non-geared extruder
//xxx = 415 for BMG extruder (special Bondtech compensated value)
//xxx = 420 for 3:1 extruder
//xxx = 473 for BNBSX with 54:16 gearing
//xxx = 490 for BNBSX, Short Ears, Skelestruder with 56:16 gearing
//
//non-geared extruder, 1.8 degree motor
//M350 E32
//M92 E280
//M500
//
//non-geared extruder, 0.9 degree motor
//M350 E16
//M92 E280
//M500
//
//geared extruder, 1.8 degree motor
//M350 E16
//M92 Exxx
//M500
//
//geared extruder, 0.9 degree motor
//M350 E8
//M92 Exxx
//M500
//
//Follow with power off/on and M503 to verify settings are correct.

//====== Kuo Uncommented def(s) below specify 0.9 degree stepper motors on x, y, z, e axis
//Motors used should be 1 amp or lower current rating to avoid overheating TMC2130 drivers in Stealthchop.
//Kuo recommended 0.9 degree motors for X, Y, or direct drive E are Moons MS17HA2P4100 or OMC 17HM15-0904S 
//
#define X_AXIS_MOTOR_09 //kuo exper X axis
#define Y_AXIS_MOTOR_09 //kuo exper Y axis
//#define Z_AXIS_MOTOR_09 //kuo exper Z axis
//#define E_AXIS_MOTOR_09 //kuo exper EXTRUDER
```

My _AXIS_MOTOR_09 defines are probably all you need to modify. The rest of my firmware changes are controlled by these defines.

Uncomment only the axes that you want to be 0.9 degree motors and any other extruder options you are using. For example, if you have 0.9 degree motors only on X & Y, a BNBSX Extruder and Slice thermistor it would look like...
```
#define X_AXIS_MOTOR_09 //kuo exper X axis
#define Y_AXIS_MOTOR_09 //kuo exper Y axis
//#define Z_AXIS_MOTOR_09 //kuo exper Z axis
//#define E_AXIS_MOTOR_09 //kuo exper EXTRUDER

//====== Kuo Uncomment ONLY ONE or NONE of below for geared extruders
//Don't forget to also send gcode to set e-steps as detailed earlier
//Reversion back from geared extruder requires sending M92 E280 & M500 to printer
//
//#define SKELESTRUDER // Uncomment if you have a skelestruder. Applies the patches for load distances and Z height.
//#define BONDTECH_PRUSA_UPGRADE_MK3 //Kuo Uncomment for Bondtech MK3 extruder upgrade. 3:1 extruder. This also sets Z_MAX_POS 205.
//#define BONDTECH_PRUSA_UPGRADE_MK3S //Kuo Uncomment for Bondtech MK3S extruder upgrade. (Note the S!!!!) 3:1 extruder. This also sets Z_MAX_POS 205.
//#define EXTRUDER_GEARRATIO_30 //Kuo Uncomment for extruder with gear ratio 3.0. 
#define EXTRUDER_GEARRATIO_3375 //Kuo Uncomment for extruder with gear ratio 3.375 like 54:16 BNBSX.
//#define EXTRUDER_GEARRATIO_35 //Kuo Uncomment for extruder with gear ratio 3.5 like 56:16 Bunny and Bear Short Ears or Skelestruder.

//====== Kuo E3D Volcano Support
//#define E3D_VOLCANO //uncomment to adjust Z_MAX_POS to accomodate 8.5 mm greater Volcano extruder height
//====== Kuo Slice Support
#define SLICETHERMISTOR //uncomment for Slice Thermistor
//#define SLICEMAGNUM //uncomment to adjust MMU2S filament laod/unload distances for Slice Magnum

//====== Kuo End of defines one normally needs to change ======
```



Save your changes

5. Test compile with Sketch -> Verify/Compile
Compiler will complete the job. 
If you see a warning about a missing bootloader, you probably have an older, RAMBo board 1.0.0 definition installed as your target board.

6. To compile and upload to the printer, connect your computer to the USB port of EINSY. You likely need to also select serial port in Arduino. For instance, on my Mac I have to select /dev/cu.usbmodem14101 for uploads.

Sketch -> Upload
The firmware will compile and upload to printer. Do NOT interrupt the update!!!! Let it complete.

Do a full factory reset with data erase. 

Don't let setup wizard run 1st time. You should set e-steps and microstepping first.

#Microstepping and e-steps
If you are using a geared extruder, you must also set e-steps and micro-stepping. Although my branch firmware includes settings for those items, they are not always accepted by the printer.

Use a terminal to make the setting and verify they have been accepted. For instance on a BNBSX extruder you would issue

M350 E16 //set extruder microstepping
M92 E473 //set e-steps
M500 //store the settings

M503 //read current settings so you can verify the extruder motor microsteps and e-steps are 16 and 473

NB: M350 must be BEFORE M92. Otherwise, M350 may alter existing the e-steps value

Once you have verified e-steps and microstepping are correct, you can proceed with setup wizard.

## Hardware
### Motors
Select either below listed Moons or OMC 0.9 degree steppers. Both dramatically reduce VFA's compared to any 1.8 degree motor. 

OMC motor is 1/2 cost of Moons but requires soldering of cable adapter. Moon's are plug-in compatible with Yotino cable harness.

Moon's are best at reducing VFA's with linearity correction OFF. They do not tune better with linearity correction. However, the Moon's have suffer a signficant rate of defective units shipped. After receiving a Moons' check it spins with very little notchinesss.

OMC's are baseline slightly worse than Moons, but can be tuned to achieve better than Moons with linearity correction (1.130 - 1.140). However, forget to set linearity correction and the OMCs are worse than Moons. 

NB: Prusa firmware older than 3.8.0 does not store linearity correction settings to EEPROM unless you let the menu time out by itself.

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

Wiring is as in pictures. Pay attention to wire order and pin positions at both EINSY and motor ends
NB: if using retrograde motor extruder like BNBSX Short Ears, plug EINSY end in 180 degrees rotated
NB2: LDO's use different pinout not detailed here.

![EINSY end of cable](https://github.com/guykuo/Prusa-Firmware/blob/0.9-Degree-Stepper-Support/einsy-end-of-cable.jpg)

![Moons motor connector wire order detail](https://github.com/guykuo/Prusa-Firmware/blob/0.9-Degree-Stepper-Support/moons-end-of-cable.jpg)

![OMC cable connector](https://github.com/guykuo/Prusa-Firmware/blob/0.9-Degree-Stepper-Support/OMC%20stepper%20online%20wiring%20adapt.JPG)

If you want pre-assembled connector and wire assemblies, while not specifically intended for stepper motors, these RGB LED strip connectors are a great alternative.  They have color-coded wires that match the standard stepper motor wire colors, they are meant to handle a couple of amps or more.  These are great for steppers that have wires already attached.

[DIY Stepper Cables: RGB LED Strjp Cables](https://www.amazon.com/gp/product/B01DC0KKJU/) 	$9
BTF-LIGHTING 10 Pairs 4pin SM JST 15cm Cable Female/Male connectors for Led Strip RGB 5050 3528 WS2801 APA02


### Drive Pulleys (optional but recommended)
[10mm Drive Pulley for GT2](https://www.amazon.com/gp/product/B07BH26P2D) $8.90 for 4
BALITENSEN GT2 Timing Pulley 16 Teeth 5mm Bore, Width 10mm for GT2 Belt 
(optional) Replace X and Y motor pulleys with these to reduce 2mm, vertical GT2 tooth artifact. This particular drive pulley yielded lower tooth engagement 2mm artifact during 1st phase testing.

### E-Steps and microstepping
Geared extruders now set for lower microstepping to avoid overruning EINSY during fast retracts or MMU2S filament moves. 
Factory reset and delete all data after installing this firmware. Otherwise EEPROM settings override settings in this firmware.
After installing this firmware, send M350 and M92 commands to force correct micro-stepping and e-step rates. M350 should be done FIRST because M350 will sometimes modified prior M92 value.
```
e-steps values for M92 depend on your extruder gearing.
xxx = 280 for non-geared extruder
xxx = 415 for BMG extruder (special Bondtech compensated value)
xxx = 420 for 3:1 extruder
xxx = 473 for BNBSX with 54:16 gearing
xxx = 490 for BNBSX, Short Ears, Skelestruder with 56:16 gearing

non-geared extruder, 1.8 degree motor
M350 E32
M92 E280
M500

non-geared extruder, 0.9 degree motor
M350 E16
M92 E280
M500

geared extruder, 1.8 degree motor
M350 E16
M92 Exxx
M500

geared extruder, 0.9 degree motor
M350 E8
M92 Exxx
M500

Follow with power off/on and M503 to verify settings are correct.
```
=======
END OF KUO MATERIAL
---

# Table of contents

<!--ts-->
   * [Linux build](#linux)
   * Windows build
     * [Using Arduino](#using-arduino)
     * [Using Linux subsystem](#using-linux-subsystem-under-windows-10-64-bit)
     * [Using Git-bash](#using-git-bash-under-windows-10-64-bit)
   * [Automated tests](#3-automated-tests)
   * [Documentation](#4-documentation)
   * [FAQ](#5-faq)
<!--te-->


# Build
## Linux

1. Clone this repository and checkout the correct branch for your desired release version.

2. Set your printer model. 
   - For MK3 --> skip to step 3. 
   - If you have a different printer model, follow step [2.b](#2b) from Windows build
   
3. Run `sudo ./build.sh`
   - Output hex file is at `"PrusaFirmware/lang/firmware.hex"` . In the same folder you can hex files for other languages as well.

4. Connect your printer and flash with PrusaSlicer ( Configuration --> Flash printer firmware ) or Slic3r PE.
   - If you wish to flash from Arduino, follow step [2.c](#2c) from Windows build first.


_Notes:_

The script downloads Arduino with our modifications and Rambo board support installed, unpacks it into folder `PF-build-env-\<version\>` on the same level, as your Prusa-Firmware folder is located, builds firmware for MK3 using that Arduino in Prusa-Firmware-build folder on the same level as Prusa-Firmware, runs secondary language support scripts. Firmware with secondary language support is generated in lang subfolder. Use firmware.hex for MK3 variant. Use `firmware_\<lang\>.hex` for other printers. Don't forget to follow step [2.b](#2b) first for non-MK3 printers.

## Windows
### Using Arduino
_Note: Multi language build is not supported._

#### 1. Development environment preparation

**a.** Install `"Arduino Software IDE"` from the official website `https://www.arduino.cc -> Software->Downloads` 
   
   _It is recommended to use version `"1.8.5"`, as it is used on out build server to produce official builds._

**b.** Setup Arduino to use Prusa Rambo board definition

* Open Arduino and navigate to File -> Preferences -> Settings
* To the text field `"Additional Boards Manager URLSs"` add `https://raw.githubusercontent.com/prusa3d/Arduino_Boards/master/IDE_Board_Manager/package_prusa3d_index.json`
* Open Board manager (`Tools->Board->Board manager`), and install `Prusa Research AVR MK3 RAMBo EINSy board`

**c.** Modify compiler flags in `platform.txt` file
     
* The platform.txt file can be found in Arduino instalation directory, or after Arduino has been updated at: `"C:\Users\(user)\AppData\Local\Arduino15\packages\arduino\hardware\avr\(version)"` If you can locate the file in both places, file from user profile is probably used.
       
* Add `"-Wl,-u,vfprintf -lprintf_flt -lm"` to `"compiler.c.elf.flags="` before existing flag "-Wl,--gc-sections"  

    For example:  `"compiler.c.elf.flags=-w -Os -Wl,-u,vfprintf -lprintf_flt -lm -Wl,--gc-sections"`
   
_Notes:_


_In the case of persistent compilation problems, check the version of the currently used C/C++ compiler (GCC) - should be at leas `4.8.1`; 
If you are not sure where the file is placed (depends on how `"Arduino Software IDE"` was installed), you can use the search feature within the file system_

_Name collision for `"LiquidCrystal"` library known from previous versions is now obsolete (so there is no need to delete or rename original file/-s)_

#### 2. Source code compilation

**a.** Clone this repository`https://github.com/prusa3d/Prusa-Firmware/` to your local drive.

**b.**<a name="2b"></a> In the subdirectory `"Firmware/variants/"` select the configuration file (`.h`) corresponding to your printer model, make copy named `"Configuration_prusa.h"` (or make simple renaming) and copy it into `"Firmware/"` directory.  

**c.**<a name="2c"></a> In file `"Firmware/config.h"` set LANG_MODE to 0.

**d.** Run `"Arduino IDE"`; select the file `"Firmware.ino"` from the subdirectory `"Firmware/"` at the location, where you placed the source code `File->Open` Make the desired code customizations; **all changes are on your own risk!**  

**e.** Select the target board `"Tools->Board->PrusaResearch Einsy RAMBo"`  

**f.** Run the compilation `Sketch->Verify/Compile`  

**g.** Upload the result code into the connected printer `Sketch->Upload`  

* or you can also save the output code to the file (in so called `HEX`-format) `"Firmware.ino.rambo.hex"`:  `Sketch->ExportCompiledBinary` and then upload it to the printer using the program `"FirmwareUpdater"`  
_note: this file is created in the directory `"Firmware/"`_  

### Using Linux subsystem under Windows 10 64-bit
_notes: Script and instructions contributed by 3d-gussner. Use at your own risk. Script downloads Arduino executables outside of Prusa control. Report problems [there.](https://github.com/3d-gussner/Prusa-Firmware/issues) Multi language build is supported._
- follow the Microsoft guide https://docs.microsoft.com/en-us/windows/wsl/install-win10
  You can also use the 'prepare_winbuild.ps1' powershell script with Administrator rights
- Tested versions are at this moment
  - Ubuntu other may different
  - After the installation and reboot please open your Ubuntu bash and do following steps
  - run command `apt-get update`
  - to install zip run `apt-get install zip`
  - add few lines at the top of `~/.bashrc` by running `sudo nano ~/.bashrc`
	
	export OS="Linux"
	export JAVA_TOOL_OPTIONS="-Djava.net.preferIPv4Stack=true"
	export GPG_TTY=$(tty)
	
	use `CRTL-X` to close nano and confirm to write the new entries
  - restart Ubuntu bash
Now your Ubuntu subsystem is ready to use the automatic `PF-build.sh` script and compile your firmware correctly

#### Some Tips for Ubuntu
- Linux is case sensetive so please don't forget to use capital letters where needed, like changing to a directory
- To change the path to your Prusa-Firmware location you downloaded and unzipped
  - Example: You files are under `C:\Users\<your-username>\Downloads\Prusa-Firmware-MK3`
  - use under Ubuntu the following command `cd /mnt/c/Users/<your-username>/Downloads/Prusa-Firmware-MK3`
    to change to the right folder
- Unix and windows have different line endings (LF vs CRLF), try dos2unix to convert
  - This should fix the `"$'\r': command not found"` error
  - to install run `apt-get install dos2unix`

#### Compile Prusa-firmware with Ubuntu Linux subsystem installed
- open Ubuntu bash
- change to your source code folder (case sensitive)
- run `./PF-build.sh`
- follow the instructions

### Using Git-bash under Windows 10 64-bit
_notes: Script and instructions contributed by 3d-gussner. Use at your own risk. Script downloads Arduino executables outside of Prusa control. Report problems [there.](https://github.com/3d-gussner/Prusa-Firmware/issues) Multi language build is supported._
- Download and install the 64bit Git version https://git-scm.com/download/win
- Also follow these instructions https://gist.github.com/evanwill/0207876c3243bbb6863e65ec5dc3f058
- Download and install 7z-zip from its official website https://www.7-zip.org/
  By default, it is installed under the directory /c/Program Files/7-Zip in Windows 10
- Run `Git-Bash` under Administrator privilege
- navigate to the directory /c/Program Files/Git/mingw64/bin
- run `ln -s /c/Program Files/7-Zip/7z.exe zip.exe`

#### Compile Prusa-firmware with Git-bash installed
- open Git-bash
- change to your source code folder
- run `bash PF-build.sh`
- follow the instructions


# 3. Automated tests
## Prerequisites
* c++11 compiler e.g. g++ 6.3.1
* cmake
* build system - ninja or gnu make

## Building
Create a folder where you want to build tests.

Example:

`cd ..`

`mkdir Prusa-Firmware-test`

Generate build scripts in target folder.

Example:

`cd Prusa-Firmware-test`

`cmake -G "Eclipse CDT4 - Ninja" ../Prusa-Firmware`

or for DEBUG build:

`cmake -G "Eclipse CDT4 - Ninja" -DCMAKE_BUILD_TYPE=Debug ../Prusa-Firmware`

Build it.

Example:

`ninja`

## Runing
`./tests`

# 4. Documentation
run [doxygen](http://www.doxygen.nl/) in Firmware folder

# 5. FAQ
Q:I built firmware using Arduino and I see "?" instead of numbers in printer user interface.

A:Step 1.c was ommited or you updated Arduino and now platform.txt located somewhere in your user profile is used.

Q:I built firmware using Arduino and printer now speaks Klingon (nonsense characters and symbols are displayed @^#$&*°;~ÿ)

A:Step 2.c was omitted.

Q:What environment does Prusa use to build the firmware in the first place?

A:Our production builds are 99.9% equivalent to https://github.com/prusa3d/Prusa-Firmware#linux this is also easiest way to build as only one step is needed - run single script, which downloads patched Arduino from github, builds using it, then extracts translated strings and creates language variants (for MK2x) or language hex file for external SPI flash (MK3x). But you need Linux or Linux in virtual machine. This is also what happens when you open pull request to our repository - all variants are built by Travis http://travis-ci.org/ (to check for compilation errors). You can see, what is happening in .travis.yml. It would be also possible to get hex built by travis, only deploy step is missing in .travis.yml. You can get inspiration how to deploy hex by travis and how to setup travis in https://github.com/prusa3d/MM-control-01/ repository. Final hex is located in ./lang/firmware.hex Community reproduced this for Windows in https://github.com/prusa3d/Prusa-Firmware#using-linux-subsystem-under-windows-10-64-bit or https://github.com/prusa3d/Prusa-Firmware#using-git-bash-under-windows-10-64-bit .

Q:Why are build instructions for Arduino mess.

Y:We are too lazy to ship proper board definition for Arduino. We plan to swich to cmake + ninja to be inherently multiplatform, easily integrate build tools, suport more IDEs, get 10 times shorter build times and be able to update compiler whenewer we want.

