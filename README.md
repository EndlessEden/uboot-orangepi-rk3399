# uboot-rk3399-orangepi-4

Works on the Orange Pi RK3399, Orange Pi 4 and Orange Pi 4 LTS.

## Installation (Image)

Note: Replace "uboot-orangepi-4-2023.01-2-aarch64.img" with "uboot-orangepi-4-lts-2023.01-2-aarch64.img" for OrangePi 4 LTS

1. Download a pre-built image, and dd to the SD card.
        ```
        wget https://github.com/endlesseden/uboot-orangepi-rk3399/releases/download/2023.01-2/uboot-orangepi-4-2023.01-2-aarch64.img{.sig,.md5}
        dd if=uboot-orangepi-4-2023.01-2-aarch64.img of=/dev/sdX
        ```
	Where 'X' is your SD-Card

2. Insert the micro SD card into the board, connect ethernet and apply power.

3. Use the serial console or SSH to the IP address given to the board by your router.
        * Login as the default user *alarm* with the password *alarm*.
        * The default root password is *root*.

4. Initialize the pacman keyring and populate the Arch Linux ARM [package signing](https://archlinuxarm.org/about/package-signing) keys.
        ```
        pacman-key --init
        pacman-key --populate archlinuxarm
        ```

## Installation (Manual)
Based on installation instructions from [Arch Linux ARM](https://archlinuxarm.org/). 

Replace **sdX** in the following instructions with the device name for the SD card as it appears on your computer.

1. Zero the beginning of the SD card:
`dd if=/dev/zero of=/dev/sdX bs=1M count=32`

2. Start fdisk to partition the SD card:
`fdisk /dev/sdX`

3. At the fdisk prompt, create the new partition:
	a. Type **o**. This will clear out any partitions on the drive.
	b. Type **p** to list partitions. There should be no partitions left.
	c. Type **n**, then **p** for primary, **1** for the first partition on the drive, **32768** for the first sector, and then press **ENTER** to accept the default last sector.
	d. Write the partition table and exit by typing **w**.
	
4. Create the ext4 filesystem:
`mkfs.ext4 /dev/sdX1 -l ROOT`

5. Mount the filesystem:
	```
	mkdir root
	mount /dev/sdX1 root
	```

6. Download and extract the root filesystem (as root, not via sudo):
	```
	wget http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
	bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C root
	```


7. Download a pre-built u-boot package, and copy to the SD card
	```
	wget https://github.com/endlesseden/uboot-orangepi-rk3399/releases/download/2023.01-2/uboot-orangepi-4-2023.01-2-aarch64.pkg.tar.zst
	cp uboot-orangepi-rk3399-2023.01-2-aarch64.pkg.tar.zst root/home/alarm/
	```
	Note: Replace "uboot-orangepi-4-2023.01-2-aarch64.tar.zst" with "uboot-orangepi-4-lts-2023.01-2-aarch64.tar.zst" for OrangePi 4 LTS.

8. Install the u-boot package in the new root
	```
	arch-chroot ./root pacman -U /home/alarm/uboot-orangepi-rk3399-2023.01-2-aarch64.pkg.tar.zst
	```
	When asked, don't attempt to flash the new version of u-boot onto the card.

9. Manually flash u-boot to the SD card:
	```
	arch-chroot ./root dd if=/boot/idbloader.img of=/dev/sdX seek=64 conv=notrunc
	arch-chroot ./root dd if=/boot/u-boot.itb of=/dev/sdX seek=16384 conv=notrunc
	```

10. Insert the micro SD card into the board, connect ethernet and apply power.

11. Use the serial console or SSH to the IP address given to the board by your router.
	* Login as the default user *alarm* with the password *alarm*.
	* The default root password is *root*.

12. Initialize the pacman keyring and populate the Arch Linux ARM [package signing](https://archlinuxarm.org/about/package-signing) keys.
	```
	pacman-key --init
	pacman-key --populate archlinuxarm
	```

## Building (x86_64 host)
(NOTE: You will need the 'qemu-user-static' package to chroot into this root)
1. Clone this repository
`git clone https://github.com/endlesseden/uboot-orangepi-rk3399 -b orangepi4`

2. Build
`cd uboot-orangepi-rk3399 && makepkg -s`

3. Mount the SDcard
```
        mkdir root
        mount /dev/sdX1 root
```

4. copy and install 
```
cp uboot-orangepi-rk3399-2023.01-2-aarch64.pkg.tar.zst root/home/alarm/
arch-chroot ./root pacman -U /home/alarm/uboot-orangepi-rk3399-2023.01-2-aarch64.pkg.tar.zst
```

5. Manually flash u-boot to the SD card:
```
arch-chroot ./root dd if=/boot/idbloader.img of=/dev/sdX seek=64 conv=notrunc
arch-chroot ./root dd if=/boot/u-boot.itb of=/dev/sdX seek=16384 conv=notrunc
```

## Building (native)
1. Install and boot as above

2. Clone this repository
`git clone https://github.com/endlesseden/uboot-orangepi-rk3399 -b orangepi4`

3. Build and install
`cd uboot-orangepi-rk3399 && makepkg -si`

4. flash u-boot to the SD card:
```
dd if=/boot/idbloader.img of=/dev/sdX seek=64 conv=notrunc
dd if=/boot/u-boot.itb of=/dev/sdX seek=16384 conv=notrunc
```
