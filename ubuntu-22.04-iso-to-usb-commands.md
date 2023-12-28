# Instructions to install ISO file to USB via CLI

1. Install `ddrescue`

   ```
   sudo apt install ddrescue
   ```

2. Determine the device path for USB device

   For this example, we will use `/dev/sdf` as our USB device.

3. Make sure USB device is not mounted

   ```
   sudo umount /dev/sdf
   ```

4. Create correct partition table with `fdisk`

   ```bash
   sudo ddrescue imagefile.iso /dev/sdf --force --odirect
   ```
