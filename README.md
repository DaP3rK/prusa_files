# Skelestruder Firmware Notes for Prusa MK3/MK3S

I'm using [JLTX's Skelestruder](https://www.thingiverse.com/thing:2845416) on my Prusa MK3S.

This repository contains updated patch files, sourced from [JLTX's Skelestruder Supplemental](https://jltxplore.dozuki.com/Wiki/Skelestruder_Information#main) with these additional modifications

- Menu change - PET renamed to PETG
- Includes patch for MMU2
- Changed printer name to Skelestruder MK3
- Using stock Prusa Z height - `210`

Patch files can be found in [fw_patch_files](https://github.com/tprelog/prusa_files/tree/master/fw_patch_files)

I've also included a copy of my compiled Skelestruder MK3S Firmware in [skelestruder_fw](https://github.com/tprelog/prusa_files/tree/master/skelestruder_fw)

---

## Basic steps for patching the firmware ( 3.9.1 ) using Linux

```bash
# create an empty directory and `cd` into it
mkdir fw_391
cd fw_391

# download and extract the source code for prusa firmware
wget -O Prusa-Firmware-3.9.1.zip https://github.com/prusa3d/Prusa-Firmware/archive/v3.9.1.zip
unzip Prusa-Firmware-3.9.1.zip

# `cd` into unzipped directory and download the skelestrude fw patch
cd Prusa-Firmware-3.9.1
wget -O skelestruder-3.9.1.patch https://raw.githubusercontent.com/tprelog/prusa_files/master/fw_patch_files/skelestruder-3.9.1.patch
```

#### *Optional* - Test the patch file

```bash
patch -p1 --dry-run < skelestruder-3.9.1.patch
```

*Example - test command with output*

```bash
$ patch -p1 --dry-run < skelestruder-3.9.1.patch

checking file Firmware/Marlin_main.cpp
checking file Firmware/mmu.cpp
checking file Firmware/ultralcd.cpp
checking file Firmware/variants/1_75mm_MK3-EINSy10a-E3Dv6full.h
checking file Firmware/variants/1_75mm_MK3S-EINSy10a-E3Dv6full.h
```

### Apply the patch file

```bash
patch -p1 < skelestruder-3.9.1.patch
```

#### *Optional* - Additional Firmware Changes

Edit the corresponding file for your printer

- MK3S - `Firmware/variants/1_75mm_MK3S-EINSy10a-E3Dv6full.h`
- MK3 - `Firmware/variants/1_75mm_MK3-EINSy10a-E3Dv6full.h`

Change the Printer Name

- Name dispays on the LCD screen
- Search for, then edit `#define CUSTOM_MENDEL_NAME`

```C
// Printer name
#define CUSTOM_MENDEL_NAME "SKELESTRUDER MK3S"
```

Change the Max Z Height

- JLTX's firmware patch uses `220`
- Search for, then edit `#define Z_MAX_POS`

```C
// Travel limits after homing
#define Z_MAX_POS 210
```

### Compile the Firmware

You can compile the firmware using `./PF-build.sh` - This script optionally takes three (3) arguments

- First argument defines which variant of the Prusa Firmware will be compiled. Valid options for Skelestruder are
  - Prusa MK3S - `1_75mm_MK3S-EINSy10a-E3Dv6full.h`
  - Prusa MK3 - `1_75mm_MK3-EINSy10a-E3Dv6full.h`
- Second argument defines if it is an english only version. Known values `EN_ONLY` or `ALL`
- Third argument is the DEV_STATUS. Valid options are `GOLD`, `RC`, `BETA`, `ALPHA`, `DEVEL` or `DEBUG`

*Options for `PF-build.sh` were found by reviewing the [source code](https://github.com/prusa3d/Prusa-Firmware/blob/MK3/PF-build.sh#L422)*

Compile skelestruder firmware for the Prusa MK3S

```bash
./PF-build.sh 1_75mm_MK3S-EINSy10a-E3Dv6full.h EN_ONLY GOLD
```

*Example - build command with output*

```bash
$ ./PF-build.sh 1_75mm_MK3S-EINSy10a-E3Dv6full.h EN_ONLY GOLD

Linux 64-bit found

Script path : /ssd/_prusa_files/src/fw_391/Prusa-Firmware-3.9.1
OS          :
OS type     : linux

Ardunio IDE : 1.8.5
Build env   : 1.0.6
Board       : PrusaResearchRambo
Specific Lib: PrusaLibrary


Variant    : 1_75mm_MK3S-EINSy10a-E3Dv6full
Firmware   : 391
Build #    : 2869
Dev Check  : 391
DEV Status : GOLD
Motherboard: BOARD_EINSY_1_0a
Languages  : EN_ONLY
Hex-file Folder: PF-build-hex/FW391-Build2869/BOARD_EINSY_1_0a


English only language firmware will be built


Start to build Prusa Firmware ...
Using variant 1_75mm_MK3S-EINSy10a-E3Dv6full
Sketch uses 241730 bytes (95%) of program storage space. Maximum is 253952 bytes.
Global variables use 6042 bytes of dynamic memory.

Copying English only firmware to PF-build-hex folder
1

Build done, please use Slic3rPE > 1.41.0 to upload the firmware
more information how to flash firmware https://www.prusa3d.com/drivers/
```

---

### Flashing the firmware

You can flash your custom firmware using [Slic3rPE](https://help.prusa3d.com/en/article/firmware-updating-and-flashing_2227) or the [Octo-Print Firmware Updater](https://github.com/OctoPrint/OctoPrint-FirmwareUpdater/blob/master/README.md#octoprint-firmware-updater).

*Optional* - In order to make sure the firmware changes are loaded, you may want to send `M502` and then `M500` to your printer after the update. This recalls defaults from firmware and saves them to EEPROM.
