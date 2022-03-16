# Hackintosh for MSI-b460m-Mortar + i5-10400

This is a working snapshot for opencore 0.7.9 + MacOS Monterey 12.1 (21C52)

All driver & kext are newest Debug version at 2022.03.08.

## Hardware:
- MSI-b460m-Mortar
- i5-10400
- SSD: RC10 500GB
- gxlinksta BCM94360CD with bluetooth
- ASUS 6600xt dual

## Feature:

+ [x] iGPU
+ [x] On-board audio
+ [x] Airdrop (no driver required)
+ [x] USB: all 3.0 is also 2.0 compatible. See noted below
+ [x] 6600xt working
+ [x] Ethernet
+ [x] iGPU acceleration
+ [ ] Hibernate


## Steps

### USB Drive Partition & Image Loading
just follow https://dortania.github.io/OpenCore-Install-Guide/installer-guide

Remind that this may take ~30GB disk space.

### EFI Preparation
Just like `OC` folder. Not included files shouldn't be necessary

`USBMap.kext` is created manually to fix usb 2.0/3.0. You can remove this during installation and add it back dursing post install.

### BIOS Configuration
Google. Lots of related material. See the reference.
To use dGPU as display and iGPU for acceleration, change IGD to PEG and enable integrated gpu.

### USB Customization
Since Big Sur 1.4, the 15 usb port limitation can not be easily bypassed, so we need to carefully select which port we use.
My case has 2 usb3.0 on the front, so there are 16 ports in total: 2(front) + 3(back) usb-A 3.0, 1(back) usb-C 3.0, 2(back) usb-A 2.0, plus 2 internal usb 2.0 (MYSTIC LIGHT + BCM94360CD's bluetooth). Each 3.0 interface needs to be 2.0 compatible, so it occupies 2 ports. That makes 16 ports in total. I removed one of the usb 2.0 port in my customized `USBMap.kext` (the top one, near PS2). As for the bluetooth, I connected it to the 2.0 pin close to back panel and preserved it in the kext.

### TODOs
- Hibernate

## Ref:

https://heipg.cn/tutorial/b460m-install-big-sur.html

https://github.com/maemual/MSI-B460M-10700-5500XT

https://dortania.github.io/OpenCore-Install-Guide/installer-guide

USB Custimization: 
https://dortania.github.io/OpenCore-Post-Install/usb/manual/manual.html#creating-a-personal-map