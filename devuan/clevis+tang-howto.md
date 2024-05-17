## Clevis and Tang with Devuan 5.0

Clevis and Tang is a security suite for LUKS encrypted disks and Network based policies (decryption) during boot time.

See: https://github.com/latchset/clevis

### Tang Server
Anything really power efficient. A Raspberry Pi Zero 2 works fine with Raspberry OS Server edition.
Tang is available via apt.

Setup keys using:

    $ sudo jose jwk gen -i '{"alg":"ES512"}' -o /var/lib/tang/YYYYMMsig.jwk
    $ sudo jose jwk gen -i '{"alg":"ECMR"}' -o /var/lib/tang/YYYYMMexc.jwk

where YYYYMM stands for YearMonth.

Enable tang-service and reboot. Setup DNS for Tang server (in this example http://tang.mydomain.lan).

### Clevis
Install Devuan 5.0 as per your preferences using Expert Install, Manual partition.

Use LVM-over-LUKS, e.g.

    root@gclevis:/etc/boot.d# lsblk
    NAME                MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
    sda                   8:0    0 29.8G  0 disk  
    ├─sda1                8:1    0  365M  0 part  /boot/efi
    ├─sda2                8:2    0  954M  0 part  /boot
    └─sda3                8:3    0 28.5G  0 part  
      └─sda3_crypt      254:0    0 28.5G  0 crypt 
        ├─FlashMem-root 254:1    0 15.8G  0 lvm   /var/lib/docker
        │                                         /
        ├─FlashMem-swap 254:2    0  3.7G  0 lvm   [SWAP]
        └─FlashMem-opt  254:3    0    9G  0 lvm   /opt

This design is 32 GB disk sda. EFI (384 MByte in sda1) and /boot (ext4, 1 GByte in sda2) are not encrypted.
/dev/sda3 is encrypted (sda3_crypt) with remaining space. LVM is on top and provides a 17GB logical
/ using ext4, 4 GB swap and remaining /opt using ext4.

Configure a static IP address per your network. Configure DNS.

Devuan has two challenges to work with Tang:

* The kernel has no working network and is missing curl for clevis during very early boot stages
* The kernel has no working DNS resolution during very early boot stages.

We will modify initramfs to archive the same.

#### DNS resolution
Create /etc/initramfs-tools/hooks/curl and copy content from file here.

#### IP network during early boot
Create /etc/boot.d/add-to-initramfs.conf and copy content from file here.

Make both files executable.

Add to //etc/initramfs-tools/modules the module for your network card (use lsusb or lspci to identify).

#### Final configuration
Reboot to enable extended initramfs.conf - with monitor and keyboard to enter the LUKS password

Enable clevis:

    clevis luks bind -d /dev/sda3 tang '{"url":"http://tang.mydomain.lan"}'

Update initram and grub

    update-initramfs -u -k 'all'
    update-grub

