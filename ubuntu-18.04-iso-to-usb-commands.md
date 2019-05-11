# Instructions to install ISO file to USB via CLI

1. Determine the device path for USB device

   For this example, we will use `/dev/sdf` as our USB device.

2. Make sure USB device is not mounted

   ```
   sudo umount /dev/sdf
   ```

3. Create correct partition table with `fdisk`

   ```bash
   # IF USING UEFI
   sudo fdisk /dev/sdf
   # g create a new empty GPT partition table
   # w write table to disk and exit
   ```

   ```bash
   # IF USING BIOS
   sudo fdisk /dev/sdf
   # o create a new empty DOS partition table
   # w write table to disk and exit
   ```

3. Run dd to write ISO to USB device

   ```
   sudo dd bs=4M status=progress if=image.iso of=/dev/sdf
   ```

4. Run `sync` to make sure all data written to USB before removing it
   ```
   sync
   ```
