# Instructions to install Ubuntu 16.04 LTS on ASUS EeeBook X205TA

*NOTICE: at the time of writing this, bluetooth, sound, and mic does not work.* See [Linux kernel Bug 95681 - No sound on Asus EeeBook X205TA](https://bugzilla.kernel.org/show_bug.cgi?id=95681)

## Required items
* Separate system already running Ubuntu 16.04 - this is where you will build the 32bit boot loader and create the USB install flash drive.
* Bootable USB flash drive at least 2GB - ALL DATA WILL BE REMOVED FROM THIS DRIVE!

## Prepare the USB flashdrive to be used as the install media
**THIS WILL DELETE DATA ON /dev/sdb - MAKE SURE YOU KNOW WHAT YOU ARE DOING!**

On a separate system already running Ubuntu 16.04 and powered on, plug in the USB flash drive. Run the following to setup the install media:
```bash
apt-get install p7zip-full

wget http://releases.ubuntu.com/16.04/ubuntu-16.04-desktop-amd64.iso

# Assuming USB flashdrive assigned to /dev/sdb
# THIS WILL DELETE ALL DATA ON /dev/sdb - make sure you know what you are doing!
sgdisk --zap-all /dev/sdb
sgdisk --new=1:0:0 --typecode=1:ef00 /dev/sdb
mkfs.vfat -F32 /dev/sdb1

# Copy the ISO file to the USB drive:
mount -t vfat /dev/sdb1 /mnt
7z x ubuntu-16.04-desktop-amd64.iso -o/mnt/

# Create the 32bit boot loader and place it on USB install media
apt-get install git bison libopts25 libselinux1-dev autogen \
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
cp bootia32.efi /mnt/EFI/BOOT/

umount /mnt
```
Remove the USB flash drive.

## Installation on X205TA

### BIOS setup

* Make sure the X205TA is off
* Plug in the USB flash drive
* Start the X205TA and continue to press F2 to get into BIOS.
* Under `Advanced` tab, `USB Configuration` -> `USB Controller Select` set to `EHCI` otherwise mouse and keyboard will not work
* Under `Security` tab, `Secure Boot menu` -> `Secure Boot Control` set to `Disabled`
* Under `Save & Exit` tab, `Save Changes` (NOT `Save Chances and Exit`)
* Lastly, while still in `Save & Exit` tab, under `Boot Override`, select the USB flash drive.

### Installation

1. At the grub menu, select "Try Ubuntu without installing" (misleading because you will be installing). This is needed to get a shell to setup the wireless device. Network access is required for installation to remotely obtain the `grub-efi-ia32-bin` package.  Without this package, install will fail.
2. Once the desktop loads, press "ctrl+alt+t" to get a terminal window.
3. Type the following to setup the wireless network device:

   ```bash
   sudo cp /sys/firmware/efi/efivars/nvram-74b00bd9-805a-4d61-b51f-43268123d113 /lib/firmware/brcm/brcmfmac43340-sdio.txt
   sudo modprobe -v -r brcmfmac
   sudo modprobe -v brcmfmac
   ```

4. The wireless network device should now be working. Press alt+F10 (or click on the wireless/network icon on the indicator panel) to configure using your wireless network setup.
5. Double click on the Desktop icon labeled "Install Ubuntu 16.04 LTS" to start the installation process and install as usual.

*Note: Selecting `Encrypt the new Ubuntu installation for security` will require you to enter a password on boot - the keyboard will not work for this requiring you to use an external USB keyboard. You have been warned.*

### Post-installation

* Due to a known system freeze issue, update `/etc/default/grub` by updating the `GRUB_CMDLINE_LINUX_DEFAULT` line to:

  ```
  GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_idle.max_cstate=1"
  ```

  Then run `update-grub`.
* We will need to copy a file as we did during installation to get wifi working:

  ```bash
  sudo cp /sys/firmware/efi/efivars/nvram-74b00bd9-805a-4d61-b51f-43268123d113 /lib/firmware/brcm/brcmfmac43340-sdio.txt
  sudo modprobe -v -r brcmfmac
  sudo modprobe -v brcmfmac
  ```

  Then configure your wireless settings.
