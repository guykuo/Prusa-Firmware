VFA's are a solved issue with 0.9 degree stepper motors. This post is a summary of what is needed to use 0.9 degree motors.


=== Firmware ===
You will need the changes contained in my 0.9 degree motor firmware (now includes BMG extruder support)
Please follow the directions at user-mods-octoprint-enclosures-nozzles- ... ml#p133678

Be able to compile the firmware before doing any hardware changes. Once motors have been changed, the first thing you need to do is update the firmware to my 0.9 motor support version.



=== Hardware ===
--- Motors ---
Select either below listed Moons or OMC 0.9 degree steppers. Both dramatically reduce VFA's compared to any 1.8 degree motor. 

OMC motor is 1/2 cost of Moons but requires soldering of cable adapter. Moon's are plug-in compatible with Yotino cable harness.

Moon's are best at reducing VFA's with linearity correction OFF. They do not tune better with linearity correction.

OMC's are baseline slightly worse than Moons, but can be tuned to achieve better than Moons with linearity correction (1.130 - 1.140). However, forget to set linearity correction and the OMCs are worse than Moons. 

NB: Prusa firmware does not store linearity correction settings to EEPROM unless you let the menu time out by itself.

OMC's are a little bit louder during printing, but not by much. 

Both choices of motor require grinding a shaft flat for the y-axis.

It's a matter of cost, install effort, and whether one can be bothered to set linearity correction (once) for X and Y. Either motor choice greatly reduces VFA's.


Moons Stepper Motors
https://www.amazon.com/MOONS-Stepper-Ac ... B072HT8PLM	$53 x 2
Moons 0.9	MS17HA2P4100 
Voltage	3.9 volt
Resistance	3.9 ohm
Current	1.0 A
Hold Torque	0.39 Nm
Weight	290 gm
inductance	11.2 mH

-- or ---
OMC Stepper Motors
https://www.amazon.com/gp/product/B00W98OYE4	$20 x 2
STEPPERONLINE 0.9° OMC 17HM15-0904S
Voltage: 12-24V
Resistance	6.0ohms
Hold Torque 0.36 Nm
Current	0.9A 
Weight	280 gm
Inductance	12.0mH

https://www.amazon.com/gp/product/B07DCL9J8K	$15
2.0mm JST-PH Connector Kit, with JST-PH 5/6/7 Pin Housing 
6 pin connector to adapt OMC motor wires to harness.
Optionally, skip these connectors and directly solder motor wires to harness
I prefer to add standardized connector because I test multiple motors.



-- Heatsinks for TMC2130's ---
Obtain heatsinks for TMZ2130's x all 4 axes. May as well cool Z and E while taking care of X & Y)
Required to avoid 0.9 motors overheating TMC2130 drivers, especially if run in Stealth mode. Heatsinks attach to NON-COMPONENT side of EINSY PC board. They do NOT attach on the TMC2130 chips themselves, but to thermal vias on empty side of EINSY board. Must also cut holes in back of EINSY case for fitment & adequate ventilation. Clean PC board with IPA to let self-stick thermal tape do its job. Position on the vias!.

https://www.amazon.com/gp/product/B07GWR3TWZ	$9
Driver Heatsinks for TMC2130 (12 pcs)
Use these or similar self-stick, driver heatsinks for stock EINSY enclosure. 

--or--
Reverse orientation EINSY case users must use low profile heatsinks because reverse case orients heatsinks towards heatbed. Extruder cable will catch on high profile heatsinks. Use low profile, Pi heatsinks.

https://www.amazon.com/gp/product/B07217N5LS	$8
Raspberry Pi Heatsink Kit (lower profile than usual driver heatsinks)
I use these Pi heatsinks with my reverse EINSY case. Smaller and lower profile but still get the job done.

-- Wiring Cable --
https://www.amazon.com/gp/product/B07CBV8DVZ	$7
YOTINO Bipolar Stepper Motor Cables, 4 x 100cm Long NEMA 17 Extended Connector Cable (XH2.54 4Pin-6Pin)


-- Drive Pulleys (optional but recommended) --
https://www.amazon.com/gp/product/B07BH26P2D $8.90 for 4
BALITENSEN GT2 Timing Pulley 16 Teeth 5mm Bore, Width 10mm for GT2 Belt 
(optional) Replace X and Y motor pulleys with these to reduce 2mm, vertical GT2 tooth artifact. This particular drive pulley yielded lower tooth engagement 2mm artifact during 1st phase testing.
