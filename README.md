Check last release with binaries distribution for Windows and Linux platforms here :   
https://github.com/TheKikGen/stm32-tkg-hid-bootloader/releases

# STM32F10x TKG-HID-BOOTLOADER

This is a HID (Human Interface Device) driverless booloader for the STM32F10x family line, including the famous Bluepill.  It was inspired by  https://github.com/Serasidis/STM32_HID_Bootloader from Vassilis Serasidis (respect to him !), but is not compatible anymore due to a lot of enhancements, code optimizations and bug corrections.

The bootloder **supports transparently low-medium and high density devices without recompilation**. 
The tkg-hid-bootloader doesn't use any ST libraries, but only CMSIS from the ST SDK. So, the bootloader size is under 4 Kbytes, allowing more space for user programs (user flash memory starts at 0x08001000). 

It was not possible (reasonable) to keep the size under the 2K, because before jumping to the user application, the bootloader must ensure that the current state of the MCU is clean. More, depending on your own hardware configuration, the size can vary with GPIO settings. So 4K is a good compromize between size and code quality, and will allow new features !

The TKG-FLASH has many new features, like info, dump, simulation mode, progression bar,compatibilty with the STM32DUINO platform, and can be easily integrated in the Arduino IDE

Since version 3.10, a checksum control is done at the end of the flashing process.

# Entering the bootloader

* "Double push" on the reset button
* Automatic when flashing with TKG-FLASH if the right com port is given to toggle DTR (not always reliable)
* BOOT1 set to HIGH (permanent with jumper set fro ultimate case !)
* Writing the value **0x424C** in the DR10 backup register (ex : from your own firmware before rebooting) 
* A START command from TKG-Flash tool


<img alt="TKG Bootloader logic" style="border-width:0" src="https://github.com/TheKikGen/stm32-tkg-hid-bootloader/blob/master/doc/TKG-HID-BOOTLOADER%20LOGIC.png?raw=true" /></a>


# TKG-FLASH 3.1
``````
+-----------------------------------------------------------------------+
|             TKG-Flash v3.1 STM32F103 HID Bootloader Flash Tool        |
|                     High density device support.                      |
|       (c) 2020 - The KikGen Labs     https://github.com/TheKikGen     |
+-----------------------------------------------------------------------+
Build : 3.210120.1702

  Usage: tkg-flash <firmware file name> [<options>]
         tkg-flash -info

  Options are :
  -info         : get only some information from the MCU without flashing.

  Flashing options :
  -d=16 -d=32   : hexa dump sectors by 16 or 32 bytes line length.
  -ide          : ide embedded, no progression bar, basic info.
  -o=<n>        : offset the default flash start page of 1-255 page(s)
  -p=<com port> : serial com port used to toggle DTR for MCU reset.
  -sim          : flashing simulation.
  -w=<time s>   : HID device waiting time (10s default).

  Examples :
  tkg-flash myfirmare.bin -p=COM4 -w=30
  tkg-flash myfirmare.bin -d=16 -s

``````

The TKG-FLASH tool has the following features :
* Permanent flashing capability if com port passed in the command line
* "embedded" mode when tkg-flash must be integrated in an IDE, like for example,the Arduino one.
* Waiting time parameter to adjust the waiting time for toggling serial port and  HID device to be ready
* Flashing simulation : allows same behaviour without writing the firmware to the flash memory
* Dump firmware file feature
* Page offset : flash a firmware beyond the tkg bootloader base address (first page is 4 or 2 depending on density). 

The page offset permits to flash a firmware that was not linked with the tkg-hid-bootloader FLASH_BASE_ADDRESS at 0x08001000.  For example, a firmware compiled with the stm32duino bootloader will be linked with a base address at 0x08004000. So to load that firmware correctly, you can offset of 1 page for a high density device (2048 bytes per pages), or 2 for a medium density one (1024 bytes par page). 

Examples :

To flash myfirmware.bin under windows, using COM1 DTR as reset method :
           
    tkg-flash myfirmware.bin -p=COM1

To do the same under Linux, with an offset of 2 pages :

    tkg-flash myfirmware.bin -p=ttyACM0s -o=2

To get informations from a device in bootloader mode :

``````
# tkf-flash -info

+-----------------------------------------------------------------------+
|             TKG-Flash v3.1 STM32F103 HID Bootloader Flash Tool        |
|                     High density device support.                      |
|       (c) 2020 - The KikGen Labs     https://github.com/TheKikGen     |
+-----------------------------------------------------------------------+
Build : 3.210120.1702

> Searching for [1209:BEBA] HID device...|
> [1209:BEBA] HID device found !
  INFO - Informations reported by the bootloader :
      Firmware version       : 0310
      MCU Flash memory size  : 128 K (medium density device)
      Page size              : 1024 bytes
      Page offset            : 4 pages
      Flash base address     : 0x08001000
  Bootloader mode still active.
> TKG-Flash end.

``````

# How to upgrade from your current bootoader

A special stm32duino sketch project can be found at https://github.com/TheKikGen/stm32-tkg-hid-bootloader/tree/master/tools/tkg_hid_btl_uploader

* Load the project in the Arduino IDE
* Specify the right uploading method in th "tools" menu, corresponding to your current bootloader
* uncomment one of the target in the .ino source. Usually for a Bluepill STM32F103C you will choose GENERIC_PC13.
``````
// --- TARGETS ---
#define GENERIC_PC13
//#define MIDITECH_MIDIFACE
//#define MIDIPLUS_SMART_PAD
``````
* The flash operation will start and the new bootloader will be written at 0x08000000.
* You can open a serial terminal to check eventual error messages.
* "Double click" reset button to go directly in bootloader mode and flash your own firmware with tkg-flash
* You can use "tkg-flash -info" to check the current bootloader firmware.
* You can also use CTRL + ALT + S to get a binary in the tkg_hid_btl_uploader directory.

# Adding a new upload method to the Arduino platform (short description)

First, you must add a new upload method in board.txt (usually located at ..\Arduino\hardware\Arduino_STM32\STM32F1

Exemple for a STM32103C Bluepill :
``````
genericSTM32F103C.menu.upload_method.HIDUploadMethod2K=TKG-HID bootloader
genericSTM32F103C.menu.upload_method.HIDUploadMethod2K.upload.tool=tkg-hid
genericSTM32F103C.menu.upload_method.HIDUploadMethod2K.build.upload_flags=-DSERIAL_USB -DGENERIC_BOOTLOADER
genericSTM32F103C.menu.upload_method.HIDUploadMethod2K.build.vect=VECT_TAB_ADDR=0x8001000
genericSTM32F103C.menu.upload_method.HIDUploadMethod2K.build.ldscript=ld/hid_bootloader_tkg_cb.ld
``````
You must also edit the linker script file mentioned in the upload method (ex above is hid_bootloader_tkg_cb.ld) in the (...)\STM32F1\variants\generic_stm32f103c\ld and adjust the ram and rom origin and lengths. Ram origin start at 0x20000000. The length is the RAM size of your MCU (20K for the STM32103CB). Rom origin is the flash memory available , starts at 0x08000000 + TKG bootloader size (4K = 0x1000).  The flash memory size is the MCU one minus the TKG bootloader size. 

``````
MEMORY
{
  ram (rwx) : ORIGIN = 0x20000000, LENGTH = 20K
  rom (rx)  : ORIGIN = 0x08001000, LENGTH = 124K
}

/* Provide memory region aliases for common.inc */
REGION_ALIAS("REGION_TEXT", rom);
REGION_ALIAS("REGION_DATA", ram);
REGION_ALIAS("REGION_BSS", ram);
REGION_ALIAS("REGION_RODATA", rom);

/* Let common.inc handle the real work. */
INCLUDE common.inc
``````
You must then add an upload method in platform.txt :

    # TKG-HID upload 2.2.2
    tools.tkg_hid_upload.cmd=tkg_hid_upload
    tools.tkg_hid_upload.cmd.windows=tkg-flash.exe
    tools.tkg_hid_upload.cmd.macosx=tkg_flash
    tools.tkg_hid_upload.path={runtime.hardware.path}/tools/win
    tools.tkg_hid_upload.path.macosx={runtime.hardware.path}/tools/macosx
    tools.tkg_hid_upload.path.linux={runtime.hardware.path}/tools/linux
    tools.tkg_hid_upload.path.linux64={runtime.hardware.path}/tools/linux64
    tools.tkg_hid_upload.upload.params.verbose=-d
    tools.tkg_hid_upload.upload.params.quiet=n
    tools.tkg_hid_upload.upload.pattern="{path}/{cmd}" "{build.path}/{build.project_name}.bin" -p={serial.port.file} -ide

and copy the tkg-flash tool in the Arduino\hardware\Arduino_STM32\tool\win .
You need to restart the Arduino IDE to see your changes.

