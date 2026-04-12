# UQ Multifunctional Composites Syringe Printer

AKA The Viedt Memorial Pen Plotter

By Sam Munro
---

The printer is heavily based upon the work of Jonathan Weiss and the Printess Bio Printer. The head is an exact rip of the Printess extruder setup attached to a fully custom gantry system.

It is suggested that you familiarise yourself with the documentation available on the [Printess GitHub](<https://github.com/weiss-jonathan/Printess-Low-Cost-3D-Printer>) before operating the VMPP.

Feel free to contact Sam - Mitch has my number
## The Printer
The bed size of the printer is 225x285mm. It has two syringe extruder heads. Each head can traverse vertically be 100mm (Z and A axes). The syringes may be depressed 50mm (B and C axes). Printess has a number of additions that can be made to the head which will work on this printer.

X and Y (multistepper in Y) are run by NEMA23 motors and TMC5160T drivers running on SPI. Vert. Axes are Alibaba NEMA17s running on TMC2209 running on UART. Control is run from an OCtopus Pro V1.1 (H723 chip).

Due to the requirement to run on the H723 chip, we are using Marlin 2.1.3-B2 which is a beta debug release of marlin. It is highly recomenned that when the stable release supports the H723 chip that the Firmware is updated to that version.

Currently the printer will home to x_max, y_max. Do not home any of the vert. axis as there are no end stops plugged it. Using sensorless homing is a possible future addition although of limited use as need to me homed manually. It is important to ensure that the vert axes will not crash when previewing the GCode. All other direction limits are software endstops so correct homing of X and Y are critical. The Octopus as 8 pins for endstops however due to the Marlin requirement of having a endstop defined for each axis it is only possible to define a min limit on one axis (one of the extra pins is used for Y2) (this may all be wrong).

The Ocotopus is running seperate power from 24V for logic and motors. The logic circuit will also be powered if you plug in the USBC. When turning on it is important to turn the main power on before connecting via USB C to avoid initialisation errors.

## Before Use and Important Notes

The printer should run both Y motors simutaneously. On occasion, often when homing has not worked properly, only one Y motor will run. If allowed this will damaged the machine. Immeadiatley switch off power (either at the wall or, if installed, on the EStop). This can be fixed by hitting the reset switch on the Octopus (above the USBC port) and rehoming.

It is imoportant to maintain lubrication of the machine both for good operation and to prevent rust. Ball screws and rails will rust if exposed. The rail carts can be lubricated by the screws on the ends. Ensure that the parts of the rails and screws that dont get oiled by movement are regularly sprayed with a lanolin or silicone grease. If parts do become rusty, gently remove with a scotch bright and then relubricate.

Before powering on the machine ensure that the Y Enstops (motor end) are equally space about 5mm from the endplates.

## Suggested Fixes and Improvements
- Estop On Motor Power Input
- The Switch on X_min is currently not used for anything other than decoration. can be wired to V pin easily.
- The Octopus Box needs a lid. DXF file is on USB or in DXF folder on CAD
- Buy and Add Feet to the machine
- 3DP a holder for the bed probe (already owned) and measure flatness of the bed. In initial testing the bed has been flat enough for small parts if you set Z at the centre of your print.
- Some of the screws don't fully engage the Nyloc (sorry). Make sure they stay tight and possibly replace.
- The Y Ball screws should probably be covered for safety
- Sensorless homing on Vertical Axes
- If a problem there is at least one spare endstop switch (V axis)
- lubricant of cart
- 3D printed box kinda sucks. Especially how the usb c doesn't fit

## Marlin and Flashing Firmware to Octopus
The Marlin file is a modified version of the one provided by Printess. Would have been easier to just start from a clean config. Feel free to improve.

The Octopus is currently setup to run over serial from Pronterface. However to flash firmware we use a MicroSD card.

Build from Marlin (ensure the printer isnt connected when building). From the PIO folder, grab firmware.bin and copy onto the SD card (delete firmware.cur if it is already in there). Plug microsd into unpowered octopus and power on over USB C. Wait for status LED to blink as standard, power off usb c and remove SD Card. The file should now be called firmware.cur. You can now power the machine over mains power and replug in the usbc to your computer after the status LED begins blinking.

## Printer Workflow
(note i am writing a lot of this from memory and bad notes after two weeks of Bintangs in Indo so some of the Gcode might be wrong)

Download Pronterface and Cura. Reread the Printess documentation.
Familiarise yourself with typical GCode and coroadinate setups (G90, G91 etc)

Turn on wall power and ensure status LED is blinking before connecting to Laptop over USB C. 

Call G28 X Y to home. Ensure that it backs off from both Y endstops after homing. If not manually move carts off endstops, reset the board via the ocotopus button and try again. ALWAYS CALL 'G28 X Y' as it will attempt to home other motors that do not have endstops if not. Call G90 to set the printer to absolute mode and call "G1 X0 Y0 F600" to send the printer to its start position

Setup Cura as described in Printess Documentation. I have included my Cura configs in the Cura folder but the settings are absolutley not dialled in for any particular reason. The most import thing is to set filament diamater to the internal diameter of your syringe and the extruder diamater to the ID of your chosen syringe. Create a STL of your part. I found it works to set the height as the expected height of the part. you made need to scale in Z in cura to get the correct amount of layers that you desire. Export the STl and bring into Cura and place as you would. Ensure the model sits flush on the build plate. Set up your parameters and slice. This will likely require a signfiicant amount of dialling in to get properties that you are happy with. good job for a RA. Export and save your GCode.

The GCode must be edited before it can be loaded into Pronterface. 
- Delete the Notes at the start
- Delete the notes at the end after the "Time Elapsed" line
- Delete the M104 bed heating commands
- Delete M107 (fans)
- Change the M82 (make sure it is M82 first) to G90. This sets it from extruder relative to total relative since we dont use extruders.
- Delete the "G1 F120 E-3" as we will do this ourselves later
- Use the Find and Replace Feature to change all the "E"s to "B"s. If using multi extruders this might be more complex sorry.

At the end of the GCode Add:
"
G91
G1 B-0.2 F100

G90
G1 Z5 F100
G1 X0 Y0 F400
"
To retract the syringe (if you deleted the extruder pullback in the direct marlin GCode) and to send the machine home

Fill the syringe enusring that no bubbles make it into the nozzle. Place the syringe into the provided holders on the machine. It may be neccessary to call "M18 B" (or other extruder axis) to disasble the stepper so that you can turn the extrusion axis by hand so that you can align the syringe holder with the plunger. Call G28 B0 after you do this. I have been doing a small initialising push in B to prime the nozzle manuallly in the pronterface terminal. If you wish you may add this to the begining of the GCode (before the G28 B0) command. I find 0.3mm of push in B is a good prepressurisation. Make sure there is a G28 B0 somewhere afterwards


Work out where in your build volume you placed the part in Cura (check the GCode against the cura preview). navigate the head to the centre of your part. Place your substrate here. You may need to lift it off the bed to ensure the nozzle can reach. used the most parralell piece of stock you can get your hands on if you need to do this. By hand rotate the Z axis until your syringe just touches the build plate. call "G28 Z0". After this point do not manually spin the Z axis. Call "G1 Z5 F200" to raise the nozzle off the build plate. You should now have set the correct Z height for your part.

When previewing the GCode it is important to preview the Min and Max values (especially for Z as it is homed by hand) to ensure the machine wont crash. go to the last extruion line and check the total change in B (or whatever extrusion axis you are using) and make sure that you fill the syringe to greater than that distance and you have sufficient leadscrew.

Have a final review of the GCode to ensure everything is within bounds and your Z never goes negative. Ensure your Feedrates are sensible. I found for printing around F400 was good. the machine will comfortably rapid X and Y at about F800. The vertical axis should not exceed F300. Test all this.

Load your GCode into the Pronterface using the "load file" button. Check everything again, especially Z heights and that you have correclty called all your G28s especially on Z. Cross your fingers and press print. Stay close to the machine and be ready to shut it down.

Goodluck and happy printing

## Other File
I have included my CAD here as well. It is a bit of a mess admititdley but I do not have Inventor on this Laptop to clean it up so my apologies. Hopefully you can make sense of it.

##






