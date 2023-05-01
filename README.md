# Rikaline-GPS-6010-Firmware-Fix
Updated firmware for old Rikaline GPS-6010 SiRFstarIIe/LP based serial GPS mouse and updated chiplist.txt for SiRFFlash utility to allow reprogramming other flash parts. The SiRF firmware version patched was SW Version 231.000.000.

## Problem

An old Rikaline GPS I found, was acquiring satellites, giving correct position fixes, downloading almanacs, which could be dumped using the SiRFDemo utility, but discovered the almanacs were not being saved to flash upon first valid position fix. This resulted in VERY LONG times to all initial position fixes. The out-of-date factory supplied almanac was used at each and every startup. Replaced the memory backup battery - no effect - though at least my SiRFDemo configurations were now being saved to SRAM.

A 2nd Rikaline GPS-6010 was acquired and found to have the same problem.

## Clue

When attempting to read out the binary firmware image, I also discovered cases where the SiRFFlash utility either failed to recognize the flash part or recognized the flash part but the GPS was unable to commit the downloaded almanac to flash memory after a successful fix. Suspected some connection here.

## Solution

1. Determined which flash parts were actually used by the GPS mice. It was a Fujitsu MBM29LV400BC flash memory in one unit and a Macronix MX29LV400B in another unit.<br>
2. Could SiRFFlash be corrected? Yes, it turns out flash programming parameters are specified by SiRFFlash's "chiplist.txt" file. Added the device specification to chiplist.txt. This can be found in SiRFDemo's installation directory. The GPS's binary image could then be successfully dumped and saved.<br>
3. Review binary image using HxD for clues to flash devices in the code. Luckily, it turned out that that the manufacturer tried to take into account that different flash memory components could be used during manufacture. A BIG help was that the flash device vendor names and part numbers were stored in ASCII in the GPS firmware image. Discovered that support for these flash parts was missing.<br>
4. The firmware was likely originally compiled in C so went about looking for which might be the binary representation of a structure array or anything like that. Reverse engineered the binary image to locate pointers to the flash part and vendor strings. Located the flash programming parameter structures. Determined the binary data was similar to the binary representation of the flash programming parameters seen in the chiplist.txt file.<br>
5. Replaced existing but unused AMD and Toshiba flash devices in the image with the correct parameters for Fujitsu MBM29LV400BC and Macronix MX29LV400B parts. Updated manufacturer name, device name, flash programming parameters and various pointers. Care has to be taken to change strings and pointers so that the replacement manufacturer's names and part numbers can fit into the space of the existing list of strings.<br>
6. Use SiRFFlash to upload the new firmware to the device.<br>
7. Confirmed almanac was being downloaded and correctly saved to flash memory after initial valid position fix and then power cycling the GPS. SiRFDemo can be used to download the almanac in the device to confirm this. Quick times to first fix on subsequent power ups. Got lucky this time - Success!

Both the original unmodified firmware image and the patched image have been provided so you can compare the images to see what changes were made. Use HxD for this.

My guess was the manufacturer might have produced this GPS mouse with a number of different flash memory parts over time and the part my particular GPS mouse was manufactured with was not specified in the firmware. Though the GPS must have seemed to work, it was not realized at the factory that the almanac was being thrown away after every power-off. The bug was not caught during original product testing and release.

The fixed image and GPS has being successfully used in an APRS tracker these past few years. The image has also successfully been used to update a 2nd Rikaline GPS-6010 serial mouse from SiRF firmware version 2.2.0 to 2.3.1 while correcting its flash problem.

## Tools used

SiRFFlash and SiRFDemo* - https://web.archive.org/web/20190831163705/https://www.falcom.de/support/software-tools/sirf <br>
HxD - https://mh-nexus.de/en/hxd/ <br>
Rikaline GPS-6010 manual - https://www.manualslib.com/manual/261000/Rikaline-Gps-6010.html <br>

(*) The original Falcom links to SiRFFlash and SiRFDemo apparently longer active.
