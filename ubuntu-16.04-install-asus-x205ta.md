# Instructions to create Ubuntu 16.04 LTS install media for ASUS EeeBook X205TA

After Ubuntu 16.04.2 updated fresh installs to have Linux kernel 4.8.0, which broke several things such as the keyboard, and many asking if I can just provide an ISO for the X205TA, I have changed this doc to give instructions on how to create an ISO with X205TA changes and Linux kernel 4.10. I also provided the resulting ISO. I provided the instructions on how to build the ISO from trusted sources because it can be dangerous running code from someone you don't know.

## Download of ISO

This document was used to create the following ISO:

[ubuntu-16.04.2-desktop-amd64-asus-x205ta-4.10-kernel.iso](https://drive.google.com/uc?export=download&id=0Bwc6mx-58gRQaHlPM3JFbWhlY28)

(SHA256 checksum: e87454446d85fbc1260ac7ec8950f38dde79411cb27347bb72157e7cb900326b)

## Required items to build the ISO

* Separate system already running Ubuntu 16.04 - this is where you will build the 32bit boot loader, create the LiveCD/ISO, and create the USB install flash drive.
* Bootable USB flash drive at least 2GB - ALL DATA WILL BE REMOVED FROM THIS DRIVE!

## Instructions to create the LiveCD

All of these commands were executed as root - I use `sudo -i`.

```bash
### Create the 32bit EFI boot loader
mkdir ~/boot32
cd ~/boot32
apt -y install git bison libopts25 libselinux1-dev autogen \
  m4 autoconf help2man libopts25-dev flex libfont-freetype-perl \
  automake autotools-dev libfreetype6-dev texinfo
git clone git://git.savannah.gnu.org/grub.git
cd grub
./autogen.sh
./configure --with-platform=efi --target=i386 --program-prefix=''
make
cd grub-core
../grub-mkimage -d . -o bootia32.efi -O i386-efi -p /boot/grub \
  ntfs hfs appleldr boot cat efi_gop efi_uga elf fat hfsplus iso9660 linux keylayouts \
  memdisk minicmd part_apple ext2 extcmd xfs xnu part_bsd part_gpt search search_fs_file \
  chain btrfs loadbios loadenv lvm minix minix2 reiserfs memrw mmap msdospart scsi loopback \
  normal configfile gzio all_video efi_gop efi_uga gfxterm gettext echo boot chain eval
# Store bootia32.efi in home dir to be copied later
mv ~/boot32/grub/grub-core/bootia32.efi ~

### Customize the LiveCD - based on https://help.ubuntu.com/community/LiveCDCustomization
# Install required tools
apt -y install squashfs-tools genisoimage

# Obtain 16.04.2 Desktop 64-bit ISO and extract content to work with
mkdir ~/livecdwip
cd ~/livecdwip
wget http://releases.ubuntu.com/16.04.2/ubuntu-16.04.2-desktop-amd64.iso
mkdir isomnt
mount -o loop ubuntu-16.04.2-desktop-amd64.iso isomnt
mkdir livecd
rsync --exclude=/casper/filesystem.squashfs -a isomnt/ livecd
unsquashfs isomnt/casper/filesystem.squashfs
mv squashfs-root systemroot
rmdir isomnt

# Backup specific files and dirs to be restored at cleanup
cp -a systemroot/etc/hosts systemroot/etc/hosts.orig
cp -a systemroot/root systemroot/root.orig
cp -a systemroot/tmp systemroot/tmp.orig

# Setup chroot environment
mount --bind /run/ systemroot/run
mount --bind /dev/ systemroot/dev
cp /etc/hosts systemroot/etc/

# chroot - change root to become systemroot
chroot systemroot
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devpts none /dev/pts
export HOME=/root
export LC_ALL=C

# chroot - download and install packaged kernel 4.10 - from http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/
cd /tmp
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/linux-headers-4.10.0-041000-generic_4.10.0-041000.201702191831_amd64.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/linux-headers-4.10.0-041000_4.10.0-041000.201702191831_all.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.10/linux-image-4.10.0-041000-generic_4.10.0-041000.201702191831_amd64.deb
dpkg -i *.deb

# chroot - remove previous kernel resources
apt -y purge linux-image-4.8.0-36-generic
apt -y purge linux-headers-4.8.0-36-generic
apt -y purge linux-headers-4.8.0-36
apt -y autoremove

# clean, exit chroot, and restore previously backed up files/dir
umount /proc
umount /sys
umount /dev/pts
exit
umount systemroot/run
umount systemroot/dev
mv systemroot/etc/hosts.orig systemroot/etc/hosts
rm -fr systemroot/root
mv systemroot/root.orig  systemroot/root
rm -fr systemroot/tmp
mv systemroot/tmp.orig systemroot/tmp

# install Broadcom 43340 wireless adapter config file
# file is available on a running x205ta system in /sys/firmware/efi/efivars/nvram-74b00bd9-805a-4d61-b51f-43268123d113
wget http://lopaka.github.io/files/instructions/brcmfmac43340-sdio.txt -O systemroot/lib/firmware/brcm/brcmfmac43340-sdio.txt

# Prep for ISO creation
chmod +w livecd/casper/filesystem.manifest
chroot systemroot dpkg-query -W --showformat='${Package} ${Version}\n' > livecd/casper/filesystem.manifest
cp livecd/casper/filesystem.manifest livecd/casper/filesystem.manifest-desktop
sed -i '/ubiquity/d' livecd/casper/filesystem.manifest-desktop
sed -i '/casper/d' livecd/casper/filesystem.manifest-desktop
# Move the bootia32.efi file previously created
mv ~/bootia32.efi livecd/EFI/BOOT/
mksquashfs systemroot livecd/casper/filesystem.squashfs
echo $(du -sx --block-size=1 systemroot | cut -f1) > livecd/casper/filesystem.size
# Overwrite livecd files to use new kernel and wireless adapter
cp systemroot/boot/vmlinuz-4.10.0-041000-generic livecd/casper/vmlinuz.efi
mkdir initrd
cd initrd
lzma -dc -S .lz ../livecd/casper/initrd.lz | cpio -imvd --no-absolute-filenames
rm -fr lib/modules/4.8.0-36-generic
cp -a ../systemroot/lib/modules/4.10.0-041000-generic lib/modules/
rm -fr lib/firmware/4.8.0-36-generic
cp -a ../systemroot/lib/firmware/4.10.0-041000-generic lib/firmware/
mkdir -p lib/firmware/brcm
cp -a ../systemroot/lib/firmware/brcm/brcmfmac43340-sdio* lib/firmware/brcm/
find . | cpio --quiet --dereference -o -H newc | lzma -7 > ../livecd/casper/initrd.lz
cd ..

# Edit livecd/README.diskdefines at this point if you wish to change name

# Create ISO
cd livecd
find -type f -print0 | xargs -0 md5sum | grep -v isolinux/boot.cat > md5sum.txt
mkisofs -J -l -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -z -iso-level 4 -c isolinux/isolinux.cat -joliet-long -o ../ubuntu-16.04.2-desktop-amd64-asus-x205ta-4.10-kernel.iso .
cd ..

# Write ISO to USB
# Assuming USB flashdrive assigned to /dev/sdb
# THIS WILL DELETE ALL DATA ON /dev/sdb - make sure you know what you are doing!
sgdisk --zap-all /dev/sdb
sgdisk --new=1:0:0 --typecode=1:ef00 /dev/sdb
mkfs.vfat -F32 /dev/sdb1
mount -t vfat /dev/sdb1 /mnt
7z x ubuntu-16.04.2-desktop-amd64-asus-x205ta-4.10-kernel.iso -o/mnt/
umount /mnt
```
Remove the USB flash drive.

## Installation on X205TA

### BIOS setup

* Make sure the X205TA is off
* Plug in the USB flash drive
* Start the X205TA and continue to press `F2` to get into BIOS.
* Under `Advanced` tab, `USB Configuration` -> `USB Controller Select` set to `EHCI` otherwise mouse and keyboard will not work
* Under `Security` tab, `Secure Boot menu` -> `Secure Boot Control` set to `Disabled`. Otherwise, you may get a **SECURE BOOT VIOLATION** on boot. This is due to the factory defaut keys not working for Ubuntu's EFI bootloader. Instead of disabling `Secure Boot Control`, you can 1) `Key Management` -> `Delete All Secure Boot Variables` or 2) `Key Management` and manually load all of the variables.  Disabling `Secure Boot Control` was easier. [More information on SecureBoot and Ubuntu](http://web.dodds.net/~vorlon/wiki/blog/SecureBoot_in_Ubuntu_12.10/)
* Under `Save & Exit` tab, `Save Changes` (NOT `Save Chances and Exit`)
* Lastly, while still in `Save & Exit` tab, under `Boot Override`, select the USB flash drive.

### Installation

Install as normal

*Note: Selecting `Encrypt the new Ubuntu installation for security` will require you to enter a password on boot - the keyboard will not work for this requiring you to use an external USB keyboard. You have been warned.*

## Bluetooth setup

The firmware for the bluetooth hardware needs to be installed and the service started in order to function. The file needed for this is [BCM43341B0.hcd](http://lopaka.github.io/files/instructions/BCM43341B0.hcd)

###Quick note on how firmware file was obtained:
1. From https://software.intel.com/en-us/iot/hardware/edison/downloads click on **Latest Yocto* Poky image** to download a file similar to `iot-devkit-prof-dev-image-edison-20160606-patch.zip`
2. Unzip the file and you will find a file called `edison-image-edison.ext4`
3. Mount this file: `mount edison-image-edison.ext4 /mnt`
4. The file `/mnt/etc/firmware/bcm43341.hcd` is what we are looking for to be renamed `BCM43341B0.hcd`

###Steps to setup bluetooth

```bash
# Download firmware file and install it
wget http://lopaka.github.io/files/instructions/BCM43341B0.hcd -O /lib/firmware/brcm/BCM43341B0.hcd

# Create systemd service file
cat >/etc/systemd/system/btattach.service <<EOL
[Unit]
Description=Btattach

[Service]
Type=simple
ExecStart=/usr/bin/btattach --bredr /dev/ttyS1 -P bcm
ExecStop=/usr/bin/killall btattach

[Install]
WantedBy=multi-user.target
EOL

# Enable service
systemctl enable btattach
```

## *EXPERIMENTAL* Kernel changes for audio support

The section was created thanks to the great work done by those on [UbuntuForums](https://ubuntuforums.org/showthread.php?t=2254322&page=126&p=13592053#post13592053).

The following steps must be done on the x205ta after installation to provide *experimental* audio support:

```bash
# Required lib and packages not installed by default
apt -y install git libssl-dev

# Retrieve the Linux kernel source tree fork - will take some time
git clone https://github.com/plbossart/sound.git -b experimental/codecs
cd sound

# Obtain the kernel config already done - otherwise you will have to run
# 'make localmodconfig', 'make menuconfig', and answer questions.
# Original file from:
#   ftp://x205ta.myftp.org:1337/kernel/.config
wget http://lopaka.github.io/files/instructions/x205ta.config -O .config

# reverse patch the commit that causes the keyboard to malfunction
git diff 3ae02c1^ 3ae02c1 | patch -Rp1

# Add patch that attempts to fix non-functioning FN-keys
# Original file from:
#   https://raw.githubusercontent.com/harryharryharry/x205ta-patches/master/fn-brightness-hack.patch
wget http://lopaka.github.io/files/instructions/fn-brightness-hack.patch
patch -p1 < fn-brightness-hack.patch

# Build - will take some time
make -j6

# Install modules
make modules_install

# Install kernel to the boot dir
export KERNELRELEASE=$(<include/config/kernel.release)
cp -va arch/x86/boot/bzImage /boot/vmlinuz-$KERNELRELEASE

# Build initramfs
update-initramfs -c -k $KERNELRELEASE

# Rebuild /boot/grub/grub.cfg
update-grub

# Obtain HiFi.conf and install it at /usr/share/alsa/ucm/chtrt5645/
# Original files from:
#   https://raw.githubusercontent.com/plbossart/UCM/master/chtrt5645/HiFi.conf
#   https://raw.githubusercontent.com/plbossart/UCM/master/chtrt5645/chtrt5645.conf
mkdir -p /usr/share/alsa/ucm/chtrt5645
wget http://lopaka.github.io/files/instructions/HiFi.conf -O /usr/share/alsa/ucm/chtrt5645/HiFi.conf
wget http://lopaka.github.io/files/instructions/chtrt5645.conf -O /usr/share/alsa/ucm/chtrt5645/chtrt5645.conf

# Install audio packages
apt -y install pulseaudio alsa-base alsa-utils pavucontrol

# Reboot and use GUI to set default output - Sound Settings...
shutdown -r now
```
