# Instructions to install Ubuntu 14.10 on ASUS EeeBook X205TA

*NOTICE: because of the lack of hardware support on the X205TA at the time of writing this doc, I have returned the device.  The items that were the deal breakers: wireless and sound.*

## BIOS
press F2 before boot to get into BIOS.
Under Security, set 'Secure Boot Control' to Disabled
Under Advanced, set 'USB Controller Select' to EHCI
'Save Changes and Exit'

## Few notes:

After initial install, the following devices are setup:
```
/boot = /dev/mmcblk0p1
/     = /dev/mmcblk0p2
```

WIFI does not work out of the box. I had [this USB WIFI adapter](http://www.newegg.com/Product/Product.aspx?Item=N82E16833315091) laying around and it worked perfectly.


## Create bootia32.efi

On a freshly installed Ubuntu 14.10 system
```bash
apt-get install git bison libopts25 libselinux1-dev autogen m4 autoconf help2man libopts25-dev flex libfont-freetype-perl automake autotools-dev libfreetype6-dev texinfo

# from http://www.gnu.org/software/grub/grub-download.html
git clone git://git.savannah.gnu.org/grub.git

cd grub

./autogen.sh

./configure --with-platform=efi --target=i386 --program-prefix=''

make

cd grub-core

../grub-mkimage -d . -o bootia32.efi -O i386-efi -p /boot/grub ntfs hfs appleldr boot cat efi_gop efi_uga elf fat hfsplus iso9660 linux keylayouts memdisk minicmd part_apple ext2 extcmd xfs xnu part_bsd part_gpt search search_fs_file chain btrfs loadbios loadenv lvm minix minix2 reiserfs memrw mmap msdospart scsi loopback normal configfile gzio all_video efi_gop efi_uga gfxterm gettext echo boot chain eval

# copy bootia32.efi to /tmp directory to be used later
cp bootia32.efi /tmp
```

## Prepare the USB flashdrive to be used as the install media
**THIS WILL DELETE DATA ON /dev/sdb - MAKE SURE YOU KNOW WHAT YOU ARE DOING!**

On a freshly installed Ubuntu 14.10 system:
```bash
apt-get install p7zip-full

wget http://releases.ubuntu.com/14.10/ubuntu-14.10-desktop-amd64.iso

# Assuming USB flashdrive assigned to /dev/sdb
# THIS WILL DELETE ALL DATA ON /dev/sdb - make sure you know what you are doing!

sgdisk --zap-all /dev/sdb
sgdisk --new=1:0:0 --typecode=1:ef00 /dev/sdb
mkfs.vfat -F32 /dev/sdb1
```
Run the following to check if Disklabel type is `gpt` and Type is `EFI System`
```bash
fdisk -l /dev/sdb
```
Output should match:
```
Disk /dev/sdb: 14.9 GiB, 16008609792 bytes, 31266816 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: EE8E4AB6-A349-426E-81C9-F37EB3DFBFF9

Device     Start      End  Sectors  Size Type
/dev/sdb1   2048 31266782 31264735 14.9G EFI System
``` 

Continue copying the ISO file to the USB drive:
```bash
mount -t vfat /dev/sdb1 /mnt
7z x ubuntu-14.10-desktop-amd64.iso -o/mnt/

# The bootia32.efi file from last section
cp /tmp/bootia32.efi /mnt/EFI/BOOT/

umount /mnt
```
Plug the USB stick into the X205TA, start the system, and continue to press `F2` to get the BIOS.  From there, go to `Save & Exit` tab, and under `Boot Overrid` select the USB flash drive.

Install Ubuntu as usual.

# Set the new install to boot with 32bit grub

The newly installed system does not have a 32bit bootloader installed.  The following must be done.

## Get the bootloader on the USB drive to run the install on the system

Boot again with the USB stick by pressing `F2`.  Once at the grub selection screen, press `c` to edit commands.

Type the following at the `grub>` prompt (tab autocomplete can also be used here):
```
linux (hd1,gpt2)/boot/vmlinuz-3.16-0-23-generic root=/dev/mmcblk0p2 reboot=pci,force
initrd (hd1,gpt2)/boot/initrd-3.16-0-23-generic
boot
```

# After initial boot - steps to install 32bit grub

*After the system boots, you can remove the USB key if you wish*

These steps closely match what we did on the system we used to create the USB flashdrive.

```bash
apt-get install git bison libopts25 libselinux1-dev autogen m4 autoconf help2man libopts25-dev flex libfont-freetype-perl automake autotools-dev libfreetype6-dev texinfo

# from http://www.gnu.org/software/grub/grub-download.html
git clone git://git.savannah.gnu.org/grub.git

cd grub

./autogen.sh

./configure --with-platform=efi --target=i386 --program-prefix=""

make

cd grub-core
../grub-install -d . --efi-directory /boot/efi/ --target=i386
cd /boot/efi/EFI
cp grub/grubia32.efi ubuntu/
```
