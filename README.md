# Rikaline-GPS-6010-Firmware-Fix
Updated firmware for old Rikaline GPS-6010 SiRFstar II based serial GPS mouse and updated chiplist.txt for SiRFFlash utility to allow reprogramming the device. The SiRF firmware version fixed was 2.3.1.

Problem:

A old Rikaline GPS I found, was acquiring satellites, giving correct position fixes, downloading almanacs, which could be dumped using the SiRFDemo utility, but discovered the almanacs were not being saved to flash upon first valid position fix. This resulted in VERY LONG times to initial position fix given the out-of-date factory supplied almanac was used at each and every startup. Replaced the memory backup battery - no effect - though at least my SiRFDemo configurations were now being saved to SRAM.

Clue:

Then, when attempting to read out the binary firmware image, I also discovered the SiRFFlash utility failed to recognize the flash part. Suspected some connection here.

Solution:

1. Determined which flash part was actually used by this particular GPS mouse. It was a Fujitsu MBM29LV400BC flash memory in my device.<br>
2. Could SiRFFlash be corrected? Yes, it turns out flash parts are specified by SiRFFlash's "chiplist.txt" file. Added the device specification to chiplist.txt. The GPS's binary image could then be successfully dumped and saved.<br>
3. Review binary image using HxD for clues to flash devices in the code. Luckily, it turned out that that the manufacturer tried to take into account different flash memory components could be used. A BIG help was that the flash device manufacturer names and part numbers were stored in ASCII in the GPS firmware image.<br>
4. The firmware was likely originally compiled in C so went about looking for which might be the binary representation of a structure array or something like that. Reverse engineered the binary image to locate pointers to and the flash part data structures. Examined and determined the binary data was simular to the binary representation of the data in the chiplist.txt file.<br>
5. Replaced an existing but unused Toshiba flash device in the image with the correct data for the Fujitsu MBM29LV400BC part, manufacturer name, device name, and flash programming parameters and various pointers. Care has to be taken to swap an existing data structure with strings the same length or greater length than the new device so that the replacement manufacturer's name and part number can fit into the space of the existing data structure.<br>
6. Use SiRFFlash to upload the new firmware to the device. The suspense builds. :-)<br>
7. Confirmed almanac was being downloaded and correctly saved to flash memory. Quick times to first fix on subsequent power ups. Got lucky this time - Success!

My guess was the manufacturer might have produced this GPS mouse with a number of different flash memory parts over time and the part my particular GPS mouse was manufactured with was not specified in the firmware. Though the GPS must have seemed to work, it was not realized at the factory that the almanac was being thrown away after every power-off. The bug was not caught during product testing and release.

The fixed image and GPS is being successfully used in an APRS tracker. The image has also successfully be used to update a 2nd Rikaline GPS-6010 serial mouse from SiRF firmware version 2.2.0 to 2.3.1.

Tools used:

SiRFFlash and SiRFDemo - https://www.falcom.de/support/software-tools/sirf/ <br>
HxD - https://mh-nexus.de/en/hxd/ <br>
