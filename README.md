# Raspberry Pi specific fork

This fork repurposed this project and replaced obsolete librarary sysfs, and wiringpi as it was not working reliably enough to program without frequent mismatch errors or validation errors.

## Installation
This project has been tested on Raspberry Pi 4 with Bullseye 32-bit and Bookworm 32-bit and 64-bit.
It was checked only on cpld Xillinx XC9500XL series.


```
sudo apt install build-essential libusb-dev libftdi-dev libgpiod-dev git cmake
git clone https://github.com/arkadiuszmakarenko/xc3sprog.git
mkdir xc3sprog/build
cd xc3sprog/build
cmake ..
make
sudo make install
```
## Connecting GPIO 
cable type : gpiod_creator

|GPIO |Header Pin |Function|
--- | --- | ---|
|4|7|TMS|
|17|11|TCK|
|22|13|TDO|
|27|15|TDI|

## JTAG Chain discovery

```
sudo xc3sprog -c gpiod_creator -j

```

## Programming
```
xc3sprog -c gpiod_creator -v -p {chain poistion eg. 0} {filename}.jed
```

---
Old docs
---



# Installation
```
# Add repo and key
curl https://apt.matrix.one/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.matrix.one/raspbian $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/matrixlabs.list

# Update packages and install
sudo apt-get update
sudo apt-get upgrade

#Installation
sudo apt install matrixio-xc3sprog

```

# Spartan3, XCF and CPLD JTAG programmer and other utilities

Copyright (C) 2004 Andrew Rogers
          (C) 2005-2011 Uwe Bonnes bon@elektron.ikp.physik.tu-darmstadt.de

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program; if not, write to the Free Software
Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA 

Please also read the file "COPYING" which is a copy of the GNU General
Public License

## Prerequisites
This program should run without installation. For accessing USB cables, `libusb0`
is required as runtime dynamic linked library.

To compile, you need `CMAKE`, the static `libftdi` library and `usb.h`. 

## Compilation

```
$ mkdir build; cd build; cmake ..; make
```
To crosscompile for Win32 with mingw:

```
$ mkdir build-win32; cd build-win32; 
$ cmake -DCMAKE_TOOLCHAIN_FILE=../Toolchain-mingw32.cmake ..
```

## Options

####Get a description:
```
$ ./xc3sprog -h
```
Note: This will also list the supported cables.

####Get a chain description:
```
$ ./xcs3prog -j
```
Device description is searched in the file pointed to by the XCDB environment 
variable, or when not found the built-in list is used.

####Test the chain integrity:
```
$ ./xc3sprog -T -j
```
Here "-j" stops xc3sprog from entering the program/verify/read the part. If
things go wrong, an endless loop is entered to facilitate hardware degugging
with the scope.
Stop with `^C`.

###Programming
The Platform Flash PROM of the Xilinx Spartan3 Starter Kit can be programmed
by specifying it's location in the JTAG chain. Example command line below.
```
$ ./xc3sprog -c pp -p 1 <bitfile.bit>
```
Program the flash of an XC3S50AN by loading the bscan_spi bitfile
first. Aassume the XC3S50AN as single part in the jtag chain
```
$ ./xc3sprog  ../bscan_spi/xc3s50an.bit
```
Now program the flash
```
$ ./xc3sprog -I <your_bitfile.bit>
```
Verify the flash content against a file
```
$ ./xc3sprog -I -C <your_bitfile.bit>
```
You can readout XCF/ISF flash and CPLD

ISF flash probably mean Incircuit Serial Flash, as internal to XC3SAN or 
external connected to XC3SA/XC3SA or XC6S

First load an appropriate bitfile. Some bitfiles are in bscan_spi. Otherwise
use the appropriate HDL file from bscan_spi and a fitting UCF file to create
a ISE Project and run the Xilinx tools to generate the bitfile.

Load the bitfile, like
```
$ ./xc3sprog bscan_spi/xc6s_cs324.ucf
```
After loading the ISF Bitfile, you can now talk to the ISF Flash. When 
writing to the ISF, at the end the FPGA tries to reconfigure from flash
and the ISF Bitfile is lost.
```
$ ./xc3sprog -r (-I) <file to store>
```
xc3sprog handles XC3, XCF0x and XC95xxxXx. XC4 should work but is untested.

There is also a utility program included that parses and prints the header
of a Xilinx .bit file.
```
$ ./bitparse echo_out.bit
```
Jedecfiles for CPLD programming can be parsed with
```
$ ./jedecparse <jedecfile.jed>
```
When using a FT2232D|L programmer, speed is noticely enhance with a USB-2.0 
Hub between the adapter and the PC. Some effort has been made to concatenate 
as many JTAG actions as possible.

Example fallback multi boot setup on XC6S

Load intermediate BSCAN_SPI bitfile:
```
$ ./xc3sprog -I <package specific bscan_spi bitfile>
```
Eventually erase (not really needed):
```
$ ./xc3sprog -I -e
```

Prepare the Multiboot header from the Template. Adapt GENERAL2 and GENERAL4 for
your setup. Here a 32 MiBit Flash and a XC6SLX45 is used. The Bitstream length
is about 1484404 = 0x16a674 bytes, so with placing the golden image at 0x10000 
the golden image ends at 0x17a674 and with placing the normal image at 0x190000,
there should be enough spacing between both. Double check that GENERAL2 and 
GENERAL4 match the offsets.

Write the header
```
$ ./xc3sprog -I<boot_header_only_SPI_x1.hex> 
```

Write golden image
```
$ ./xc3sprog -I <golden image>:w:0x10000
```
 
Write normal image
```
$ ./xc3sprog -I <normal image>:w:0x190000
```

Reboot
```
$ ./xc3sprog -R
```

All all in one
```
$ ./xc3sprog -R -I<boot_header_only_SPI_x1.hex> <golden image>:w:0x10000 \
   <normal image>:w:0x190000
```

## Special Thanks

* nahitafu@nifty.com (naxjp, XC95X algorithm example),
* zoltan_csizmadia at yahoo dot (xilprg)
* Benedikt Heinz <Zn000h@googlemail.com> for the XC3SAN ISF 
* and many others
