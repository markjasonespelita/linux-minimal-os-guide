Setup Minimal Linux Operating System Distribution

Created by: Mark Jason Espelita

NOTE: this is my personal notes based from multiple sources I studied - Youtube, github /torvalds/linux.git, and other sources. :)

1. install the needed libraries
```bash
sudo apt install build-essential bc bison flex libssl-dev libelf-dev dwarves pkg-config libncurses-dev busybox-static libdw-dev
```

## Kernel

2. clone the linux source code
```bash
git clone https://github.com/torvalds/linux.git
```

3. perform this on terminal
```bash
cd linux
make tinyconfig
```

4. configure the kernel
```bash
[*] 64-bit kernel
general setup - [*] initial ram filesystem
general setup - initial ram filesystem - [*] support gzip
general setup - configure standard kernel features
general setup - configure standard kernel features - [*] Enable support for printk
device drivers - character devices - [*] enable tty
executable file formats - [*] kernel support for ELF binaries
```

5. build the kernel image
```bash
make -j 4
```
## Busybox

6. perform this on the terminal
```bash
cd ..

// make a filesystem
mkdir -p rootfs/{bin,sbin,etc,proc,sys,dev,usr/bin,usr/sbin}

// copy busybox binary into your rootfs /usr/bin
cd rootfs
which busybox
/usr/bin/busybox
cd bin
cp /usr/bin/busybox . --verbose

// create symlinks of the core utils
for i in $(./busybox --list); do ln -s busybox $i 2>/dev/null; done
```

7. create init (the first thing to load during after kernel bootstraping)
```bash
cd ../
vim init
```
8. init contents
```bash
#!/bin/sh

mount -t proc proc /proc
mount -t sysfs sys /sys
mount -t devtmpfs dev /dev

echo "Welcome to your custom Linux distro!"

exec /bin/sh
```

9. make init executable
```bash
chmod +x init
```

10. compile initramfs using .cpio.gz
```bash
find . | cpio -o -H newc | gzip > ../initramfs.cpio.gz
```

## Test your new OS
```bash
// go to the project root
cd ../
sudo qemu-system-x86_64 --enable-kvm -kernel linux/arch/x86/boot/bzImage initrd initramfs.cpio.gz
```