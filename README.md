# Digital Voice Modem Firmware (Hotspot)

The DVM hotspot firmware provides the embedded microcontroller implementation of a dedicated-mode DMR, P25 or NXDN hotspot system. The firmware is the portion of a complete Over-The-Air modem implementation that uses an ADF7021 to provide a raw RF interface.

This project is a direct fork of the MMDVM_HS (https://github.com/juribeparada/MMDVM_HS) project.

Please feel free to reach out to us for help, comments or otherwise, on our Discord: https://discord.gg/3pBe8xgrEz

> **_NOTE:_**  Before beginning the firmware update process please see the various Makefile's included in the project for more information. This project includes a few Makefiles to target different hardware. (All following information assumes familiarity with the standard Linux make system.)
> - EG: Makefile.STM32FX - This makefile is used for targeting a generic STM32F103 with an ADF7021 RF SoC device.
  
## Firmware Preparation and Building


* For STM32F103 using Ubuntu based OS, install the standard ARM embedded toolchain (typically arm-gcc-none-eabi).

  - ```sudo apt install gcc-arm-none-eabi libnewlib-arm-none-eabi build-essential```

* From the directory of your choosing clone this repository. It will create a folder named dvmfirmeare-hs. Ensure that when you clone you are using the ```--recurse-submodules``` option, otherwise the STM32 platform files will be missing! 

  - ```git clone --recurse-submodules https://github.com/DVMProject/dvmfirmware-hs.git```

* To build the firmware, use the ```make``` command, followed by -f and the correct makefile, followed by the type of board you are using. 

  - EG: ```make -f Makefile.STM32FX mmdvm-hs-hat-dual``` for a full duplex modem hotspot, attached to GPIO.

## Raspberry Pi Preparation Notes

Some extra notes for those who are using the Raspberry Pi, default Raspbian OS or Debian OS installations. You will not be able to flash or access the STM32 modem unless you do some things beforehand.

1. Disable the Bluetooth services. Bluetooth will share the GPIO serial interface on `/dev/ttyAMA0`. On Rasbian OS or Debian OS, this is done by: `sudo systemctl disable bluetooth` then adding `dtoverlay=disable-bt` to `/boot/config.txt`.
1. The default Rasbian OS and Debian OS will have a getty instance listening on `/dev/ttyAMA0`. This can conflict with the STM32, and is best if disabled. On Rasbian OS or Debian OS, this is done by: `systemctl disable serial-getty@ttyAMA0.service`
1. On modern OS version (Bookworm / Debian 12.0 or newer), the getty instance on `/dev/ttyAMA0` gets rebuilt on boot via a systemd generator, even if you've already disabled it.  You'll need to disable this generator with: `sudo systemctl mask serial-getty@ttyAMA0.service`
1. There's a default boot option which is also listening on the GPIO serial interface. This **must be disabled**. Open the `/boot/cmdline.txt` file in your favorite editor (vi or pico) and remove the `console=serial0,115200` part.

The steps above can be done by the following commands:

```shell
sudo systemctl disable bluetooth.service serial-getty@ttyAMA0.service
sudo systemctl mask serial-getty@ttyAMA0.service
grep '^dtoverlay=disable-bt' /boot/config.txt || echo 'dtoverlay=disable-bt' | sudo tee -a /boot/config.txt
sudo sed -i 's/^console=serial0,115200 *//' /boot/cmdline.txt
```

After finishing these steps, reboot.

### Install the firmware via GPIO on Raspberry Pi

> **_NOTE:_**  Your mileage may vary with these instructions, the hotspot boards are loosely designed around a common factor but not all are created equally.

> **_NOTE:_**  Most sets of instructions reccomend to download stm32flash from online, however we have found the prepackaged version to work fine.

Once the hotspot is back on, install stm32flash:

  - ```sudo apt install stm32flash```

Once that is complete put a jumper across the JP1 points on the board, and the RED heartbeat LED may or may not stop flashing. This jumper will need to remain for the duration of the firmware upgrade process.

<img src=https://github.com/user-attachments/assets/220b2e12-61b6-429e-98cc-a489918c0aad>

  - Image credit to [DF2ET](https://github.com/phl0)


Once confirmed navigate to the build folder where you compiled the firmware and run the below command to flash. Sudo is required on most systems to access GPIO pins.

  - ```sudo stm32flash -v -w dvm-firmware-hs_f1.bin -i 532,-533,533,-520 -R /dev/ttyAMA0```

> **_NOTE:_**  On legacy raspbian versions , the way GPIO chips are numbered was different. If you're using raspbian Bullseye (Debian 11) or older, use this command instead: ```sudo stm32flash -v -w dvm-firmware-hs_f1.bin -i 20,-21,21,-20 -R /dev/ttyAMA0```

You should see a result similar to the below output if the board flashed successfully.
```
Wrote and verified address 0x0800deec (100.00%) Done.

Resetting device... 
Reset done.
```
There are reports that when using MMDVM Duplex Hats (such as the AURSINC) JP1 can remain jumpered during normal operation if you do not wish to de-solder for ease of future updating. As we cannot guarantee this will remain true this decision will remain at the discretion of the end user. (If unexpected behavior is experienced, first step would be de-solder JP1 and see if the concern is resolved.)

## Notes

**NXDN Support Note**: NXDN support is currently experimental.

## License

This project is licensed under the GPLv2 License - see the [LICENSE.md](LICENSE.md) file for details. Use of this project is intended, for amateur and/or educational use ONLY. Any other use is at the risk of user and all commercial purposes is strictly discouraged.

