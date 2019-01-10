# Anycubic i3 Mega Marlin 1.1.9 by wattie2k1 davidramiro

This is my forked version of davidramiros version of Marlin firmware for the Anycubic i3 Mega. This version has been modified to be able to use a BLTouch sensor for automatic bed leveling.

## How to level your bed with a BLTouch sensor

At first meassure the offset of the nozzle:

- G28
- G90
- G1 Z10
- G1 X40 Y40 F4000
- M280 P0 S10
- G91
- Now move the nozzle down with G1 Z-1 or G1 Z-0.1 until the BLTouch sensor is activated.
- acknowledge the BLTouch with M280 P0 S160
- M114 
- G90
- G1 X72 Y64 F4000
- G91
- G1 Z-0.1
- M114
- M851 Z-<Offset>
- M500

This only has to be done once, it should be done again if you changes are done to the bed itself.

- Heat up your bed
- G28
- G29
- M500

Bed leveling is done at this point, the only thing left to do is make sure you enabled using the measured bed level point by using M420 S1 in your start script. Simply set this command after G28.

=======
While the i3 Mega is a great printer for its price and produces fantastic results in stock, there are some issues that are easily addressed:

- Many people have issues getting the Ultrabase leveled perfectly, using Manual Mesh Bed Leveling the printer generates a mesh of the flatness of the bed and compensates for it on the Z-axis for perfect prints without having to level with the screws.
- Much more efficient bed heating by using PID control. This uses less power and holds the temperature at a steady level. Highly recommended for printing ABS.
- Fairly loud fans, while almost every one of them is easily replaced, the stock FW only gives out 9V instead of 12V on the parts cooling fan so some fans like Noctua don't run like they should. This is fixed in this firmware.
- Even better print quality by adding Linear Advance, S-Curve Acceleration and some tweaks on jerk and acceleration.
- Thermal runaway protection: Reducing fire risk by detecting a faulty or misaligned thermistor. 
- Very loud stock stepper motor drivers, easily replaced by Watterott or FYSETC TMC2208. To do that, you'd usually have to flip the connectors on the board, this is not necessary using this firmware.
- No need to slice and upload custom bed leveling tests, simply start one with a simple G26 command.
- Easily start an auto PID tune or mesh bed leveling via the special menu (insert SD card, select special menu and press the round arrow)

## How to flash this?

I provided three different precompiled hex files: One for no modifications on the stepper motor drivers - good for people who didn't touch anything yet, one for boards with TMC2208 installed and where the connectors have been flipped and one with TMC2208 and the connectors in original orientation.

### Choose your precompiled hex:

- Download the precompiled firmware here: [Releases](https://github.com/davidramiro/Marlin-AI3M/releases)
- Choose the correct hex file:
- For TMC2208 with connectors in original orientation, use `Marlin-AI3M-XXXXXX-TMC2208.hex`
- If you use TMC2208 and already reversed your connectors, use `Marlin-AI3M-XXXXXX-TMC2208_reversed.hex`
- If you use a newer version of the TMC2208 that doesn't require the connector to be reversed (TMC2208 "v2.0" written on the PCB, chip on the top side), please also use `Marlin-AI3M-XXXXXX-TMC2208_reversed.hex`.
- If you use the original stepper motor drivers, use `Marlin-AI3M-XXXXXX-stock_drivers.hex`.

### Or compile it yourself:

- Download Arduino IDE
- Clone or download this repo
- In the IDE, under `Tools -> Board` select `Genuino Mega 2560` and `ATmega2560`
- Open Marlin.ino in the Marlin directory of this repo
- Customize if needed and under `Sketch`, select `Export compiled binary`
- Look for the .hex file in your temporary directory, e.g. `.../AppData/Local/Temp/arduino_build_xxx/` (only the `Marlin.ino.hex`, not the `Marlin.ino.with_bootloader.hex`!)

### After obtaining the hex file: 

- Flash the hex with Cura, OctoPrint or similar
- Use a tool with a terminal (OctoPrint, Pronterface, Repetier Host, ...) to send commands to your printer. 
- Connect to the printer and send the following commands:
- `M502` - load hard coded default values
- `M500` - save them to EEPROM

## Calibration & Tuning

### Manual Mesh Bed Leveling

If you have issues with an uneven bed, this is a great feature.

- Level your preheated bed as well as you can
- Send `G29 S1`, your nozzle will go to the first calibration position
- Don't adjust the bed itself with screws, only use software from here on:
- Use a paper (I recommend using thermopaper like a receipt or baking paper)
- Use the onscreen controls or a tool like OctoPrint to lower or raise your nozzle until you feel a light resistance
- If 0.1 mm steps are not enough, you can send specific commands down to 0.02 mm via those three commands:
- To raise: `G91`, `G1 Z+0.02`, `G90` (one after another, not in one line)
- To lower: `G91`, `G1 Z-0.02`, `G90`
- I also added fine Z axis controls to the special menu, might be easier to use.
- When done, send `G29 S2` and repeat the process for the next level point. Continue with `G29 S2`every time.
- After finishing the 25 points, the printer will beep and calculate. 
- After seeing `ok` in the console, send `M500` to save.
- Reboot the printer.
- To ensure your mesh gets used on every print from now on, go into your slicer settings and look for the start GCode
- Look for the Z-homing (either just `G28` or `G28 Z0`) command and insert these two right underneath it:
```
M501
M420 S1
```
- Enjoy never having to worry about an uneven bed again!

Note: By default, this firmware probes 25 points. This might take little while, if you don't need an as precise mesh, you can change the grid size by sending `G29 Px` before starting the leveling. E.g., `G29 P3` results in a 3x3 mesh, thus only 9 points.

### Testing your bed leveling

- No need to download or create a bed leveling test, simply send those commands to your printer:
```
G28
G26 C H200 P25 R25
```
- To adjust your filament's needed temperature, change the number of the `H` parameter
- If your leveling is good, you will have a complete pattern of your mesh on your bed that you can peel off in one piece
- Optional: Hang it up on a wall to display it as a trophy of how great your leveling skills are.


### Calibrating extruder & PID


### Extruder steps

- Preheat the hotend with `M104 S220`
- Send `M83` to prepare the extruder
- Use a caliper or measuring tape and mark 120 mm (measured downwards from the extruder intake) with a pencil on the filament
- Send `G1 E100 F100`
- Your extruder will feed 100 mm of filament now (takes 60 seconds)
- Measure where your pencil marking is now. If it's exactly 20 mm to the extruder, it's perfectly calibrated
- If it's less or more than 20 mm, subtract that value from 120 mm, e.g.:
- If you measure 25 mm, your result would be 95 mm. If you measure 15 mm, your result would be 105 mm
- Calculate your new value: (100 mm / actually extruded filament) * 92.6
- For example, if your markings are at 15 mm, you'd calculate: (100/105) * 92.6 = 88.19
- Put in the new value like this: `M92 X80.00 Y80.00 Z400.00 Exxx.xx`, replacing `x` with your value
- Save with `M500`
- Finish with `M82`


### PID tuning

**PID calibration is only necessary if you experience fluctuating temperatures.**

- Turn on parts cooling fan If you have a radial blower fan like the original one, I generally recommend running it at 70% because of the 12V mod (`M106 S191`). Remember to also limit it in your slicer.
- Send `M303 E0 S210 C6 U1` to start extruder PID auto tuning
- Wait for it to finish
- Send `M303 E-1 S60 C6 U1` to start heatbed PID auto tuning
- Wait for it to finish
- Save with `M500`, turn off fan with `M106 S0`

Note: These commands are tweaked for PLA printing at up to 210/60 °C. If you run into issues at higher temperatures (e.g. PETG & ABS), simply change the `S` parameter to your desired temperature

**Reminder**: PID tuning sometimes fails. If you get fluctuating temperatures or the heater even fails to reach your desired temperature after tuning, you can always go back to the stock settings by sending `M301 P15.94 I1.17 D54.19` and save with `M500`.

## Updating

### Back up & restore your settings

Some updates require the storage to be cleared (`M502`), if mentioned in the update log. In those cases, before updating, send `M503` and make a backup of all the lines starting with:

```
M92
G29
M301
M304
```

After flashing the new version, issue a `M502` and `M500`. After that, enter every line you saved before and finish by saving with `M500`.


## Detailed changes:

- Thermal runaway protection enabled
- Stepper orientation flipped (you don't have to flip the connectors on the board anymore)
- Linear advance unlocked (Off by default. [Research, calibrate](http://marlinfw.org/docs/features/lin_advance.html) and then enable with `M900 Kx`)
- S-Curve Acceleration enabled
- G26 Mesh Validation enabled
- Some redundant code removed to save memory
- Manual mesh bed leveling enabled ([check this link](https://github.com/MarlinFirmware/Marlin/wiki/Manual-Mesh-Bed-Leveling) to learn more about it)
- Heatbed PID mode enabled
- Minor tweaks on default jerk and acceleration
>>>>>>> upstream/master


## Changes by derhopp:

- 12V capability on FAN0 (parts cooling fan) enabled
- Buzzer disabled (e.g. startup beep)
- Subdirectory support: Press the round arrow after selecting a directory
- Special menu in the SD file menu: Press the round arrow after selecting `Special menu`



## About Marlin

<img align="right" src="../../raw/1.1.x/buildroot/share/pixmaps/logo/marlin-250.png" />

Marlin is an optimized firmware for [RepRap 3D printers](http://reprap.org/) based on the [Arduino](https://www.arduino.cc/) platform. First created in 2011 for RepRap and Ultimaker printers, today Marlin drives a majority of the world's most popular 3D printers. Marlin delivers outstanding print quality with unprecedented control over the process.

[![Coverity Scan Build Status](https://scan.coverity.com/projects/2224/badge.svg)](https://scan.coverity.com/projects/2224)
[![Travis Build Status](https://travis-ci.org/MarlinFirmware/Marlin.svg)](https://travis-ci.org/MarlinFirmware/Marlin)
[![Flattr Us!](http://api.flattr.com/button/flattr-badge-large.png)](https://flattr.com/submit/auto?user_id=ErikZalm&url=https://github.com/MarlinFirmware/Marlin&title=Marlin&language=&tags=github&category=software)


### Contributing to Marlin

If you have coding or writing skills you're encouraged to contribute to Marlin. You may also contribute suggestions, feature requests, and bug reports through the Marlin Issue Queue.

Before contributing, please read our [Contributing Guidelines](https://github.com/MarlinFirmware/Marlin/blob/1.1.x/.github/contributing.md) and [Code of Conduct](https://github.com/MarlinFirmware/Marlin/blob/1.1.x/.github/code_of_conduct.md).

### Marlin Resources

- [Marlin Home Page](http://marlinfw.org/) - The latest Marlin documentation.
- [Marlin Releases](https://github.com/MarlinFirmware/Marlin/releases) - All Marlin releases with release notes.
- [RepRap.org Wiki Page](http://reprap.org/wiki/Marlin) - An overview of Marlin and its role in RepRap.
- [Marlin Firmware Forum](http://forums.reprap.org/list.php?415) - Get help with configuration and troubleshooting.
- [Marlin Firmware Facebook group](https://www.facebook.com/groups/1049718498464482) - Help from the community. (Maintained by [@thinkyhead](https://github.com/thinkyhead).)
- [@MarlinFirmware](https://twitter.com/MarlinFirmware) on Twitter - Follow for news, release alerts, and tips. (Maintained by [@thinkyhead](https://github.com/thinkyhead).)

### Credits

Marlin's administrators are:
 - Scott Lahteine [[@thinkyhead](https://github.com/thinkyhead)]
 - Roxanne Neufeld [[@Roxy-3D](https://github.com/Roxy-3D)]
 - Bob Kuhn [[@Bob-the-Kuhn](https://github.com/Bob-the-Kuhn)]
 - Erik van der Zalm [[@ErikZalm](https://github.com/ErikZalm)]

Notable contributors include:
 - Alexey Shvetsov [[@alexxy](https://github.com/alexxy)]
 - Andreas Hardtung [[@AnHardt](https://github.com/AnHardt)]
 - Ben Lye [[@benlye](https://github.com/benlye)]
 - Bernhard Kubicek [[@bkubicek](https://github.com/bkubicek)]
 - Bob Cousins [[@bobc](https://github.com/bobc)]
 - Petr Zahradnik [[@clexpert](https://github.com/clexpert)]
 - Jochen Groppe [[@CONSULitAS](https://github.com/CONSULitAS)]
 - David Braam [[@daid](https://github.com/daid)]
 - Eduardo José Tagle [[@ejtagle](https://github.com/ejtagle)]
 - Ernesto Martinez [[@emartinez167](https://github.com/emartinez167)]
 - Edward Patel [[@epatel](https://github.com/epatel)]
 - F. Malpartida [[@fmalpartida](https://github.com/fmalpartida)]
 - João Brazio [[@jbrazio](https://github.com/jbrazio)]
 - Kai [[@Kaibob2](https://github.com/Kaibob2)]
 - Luc Van Daele [[@LVD-AC](https://github.com/LVD-AC)]
 - Alberto Cotronei [[@MagoKimbra](https://github.com/MagoKimbra)]
 - Marcio Teixeira [[@marcio-ao](https://github.com/marcio-ao)]
 - Chris Palmer [[@nophead](https://github.com/nophead)]
 - Chris Pepper [[@p3p](https://github.com/p3p)]
 - Steeve Spaggi [[@studiodyne](https://github.com/studiodyne)]
 - Thomas Moore [[@tcm0116](https://github.com/tcm0116)]
 - Teemu Mäntykallio [[@teemuatlut](https://github.com/teemuatlut)]
 - Nico Tonnhofer [[@Wurstnase](https://github.com/Wurstnase)]
 - [[@android444](https://github.com/android444)]
 - [[@bgort](https://github.com/bgort)]
 - [[@GMagician](https://github.com/GMagician)]
 - [[@Grogyan](https://github.com/Grogyan)]
 - [[@maverikou](https://github.com/maverikou)]
 - [[@oysteinkrog](https://github.com/oysteinkrog)]
 - [[@paclema](https://github.com/paclema)]
 - [[@paulusjacobus](https://github.com/paulusjacobus)]
 - [[@psavva](https://github.com/psavva)]
 - [[@Tannoo](https://github.com/Tannoo)]
 - [[@TheSFReader](https://github.com/TheSFReader)]
 - ...and many others

## License

Marlin is published under the [GPLv3 license](https://github.com/MarlinFirmware/Marlin/blob/1.0.x/COPYING.md) because we believe in open development. The GPL comes with both rights and obligations. Whether you use Marlin firmware as the driver for your open or closed-source product, you must keep Marlin open, and you must provide your compatible Marlin source code to end users upon request. The most straightforward way to comply with the Marlin license is to make a fork of Marlin on Github, perform your modifications, and direct users to your modified fork.

