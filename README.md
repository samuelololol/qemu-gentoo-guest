# qemu-gentoo-guest

reference:
* bridge: https://wiki.gentoo.org/wiki/Network_bridge
* qemu: https://wiki.gentoo.org/wiki/QEMU/Linux_guest
    * qemu with bridge setting: https://wiki.archlinux.org/title/QEMU#Bridged_networking_using_qemu-bridge-helper
* gentoo handbook: https://wiki.gentoo.org/wiki/Handbook:AMD64

## qemu image

`qemu-img create -f qcow2 Gentoo-VM.img 15G`

## download images

choose the minimal installation CD is fine

https://www.gentoo.org/downloads/

## script#1 for testing the boot

use the `start_Gentoo_VM.sh` script with command line arguments

```
./start_Gentoo_VM.sh -boot d -cdrom install-amd64-minimal-20120621.iso
```

## script#2 for launch the LiveCD with image for installing gentoo

update the script with bridge network supported. `br0` is the bridge interface
name on the host system

```
...
    -netdev bridge,id=net0,br=br0 \
    -device virtio-net-pci,netdev=net0 \
...
```

## boot LiveCD system

after running the CLI command with arguments, add the
parameter: console=ttyS0, to the grub entry and boot

```
 │setparams 'Boot LiveCD (kernel: gentoo)'                                    │
 │                                                                            │
 │        linux /boot/gentoo root=/dev/ram0 init=/linuxrc dokeymap looptype=s\│
 │quashfs loop=/image.squashfs cdroot console=ttyS0                           │
 │        initrd /boot/gentoo.igz                                             │
 │                                                                            │
```

## check the LiveCD system

```
$ ifconfig # check network works in bridge
...
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.34  netmask 255.255.255.0  broadcast 192.168.0.255
...

$ fdisk -l
....
Disk /dev/vda: 15 GiB, 16106127360 bytes, 31457280 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
...
```

* the enp0s3 interface is bridging the host network
* the /dev/vga is mapping to the Gentoo-VM.img file on the host

## install gentoo system

* The network setting of this minimal installation LiveCD is ready(bridged host network)

go preparing the disk

### disk

#### partition
reference: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks#Viewing_the_current_partition_layout_2

```
$ fdisk /dev/vda
....

```

follow the layout of 3 partitions
* /dev/vda1 boot
* /dev/vda2 swap
* /dev/vda3 root

#### stage3(try systemd)

reference: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage
download site: https://gentoo.osuosl.org/releases/amd64/autobuilds/current-stage3-amd64-systemd/

#### chroot

reference: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base

#### kernel

reference: https://wiki.gentoo.org/wiki/Handbook:AMD65/Installation/Kernel

#### tools

reference: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Tools

## script#3 for boot with kernel image

```
#!/bin/bash
KRNL_FILE="vmlinuz-5.15.88-gentoo-x60"
IMG_NAME="gentoo.img"
MEM_SIZE=2G
CPU_CNT=2

qemu-system-x86_64 \
    -name "Gentoo VM" \
    -cpu host \
    -m $MEM_SIZE -smp $CPU_CNT \
    -enable-kvm \
    -kernel $KRNL_FILE \
    -append "root=/dev/vda3 console=ttyS0" \
    -drive file=$IMG_NAME,index=0,media=disk,if=virtio \
    -device virtio-rng-pci -netdev bridge,id=net0,br=br0 -device virtio-net-pci,netdev=net0 \
    -nographic -boot c
```
