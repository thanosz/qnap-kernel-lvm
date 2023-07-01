# QNAP kernel and lvm

## Introduction
If your QNAP device is dead you cannot access the files on the disk on a linux machine. This is because QNAP has modified the kernel and lvm code. With this kernel and initrd, you can spawn a VM to access the disk contents. 

I have never owned a QNAP device. This was a side project to recover the files in a QNAP disk for a colleague.

## How to use
You need a Linux host with qemu to spawn the VM. 
1. Plugin the disk to your linux host (via a usb dock)
2. Activate the proper mdraid array (if applicable). Typically you have opted for RAID1 setup. The contents should be on the third partition.
3. ```cat /proc/mdstat``` and activate the array with ```mdadm --run /dev/mdXXX```
4. Then spawn a VM using the kernel and initrd passing the mdraid array. The /dev/sda3 here is a second disk in order to copy your files
   ```
   qemu-system-x86_64 \
   -kernel vmlinuz-3.12.6 \
   -initrd initrd-lvm.img-3.12.6 \
   -append "init=/bin/busybox console=ttyS0" \
   -nographic -m 128M \
   -drive file=/dev/md125 \
   -drive file=/dev/sda3 \
   -serial mon:stdio
6. Once booted you should be able to activate LVM volumes and copy your files
   ```
   vgchange -ay
   mkdir -p /mnt/{src,dst}
   mount /dev/mapper/vg1-lv1 /mnt/src
   mount /dev/sdb /mnt/dst
   rsync -av /mnt/src /mnt/dst
8. The initrd image contains an rsync binary for convinience and also a full build of busybox with all modules (bin/busybox2)
