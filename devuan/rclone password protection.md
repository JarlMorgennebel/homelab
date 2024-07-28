# Rclone .config protection

Situation:
* /root is not encrypted
* Your storage partitions are encrypted and protected against theft/attackers

## Result
* An attacker/intruder may gain access to files by restoring the backup from rclone to an unencrypted disk
* Your data becomes visible even if the encrypted storage areas are not decrypted

## Solution
* Encrypt /root/.config/rclone.conf using LUKS and clevis & tang

## Step-by-Step guide
- Create an 100 MByte file to be used as virtual disk image:
  `dd if=/dev/urandom of=/root/.configurationdisk bs=32k count=3200`
- Partition virtual disk image, create a single partition:
  `fdisk  /root/.configurationdisk`
- Create LUKS encryption on the virtual disk:
  `cryptsetup luksFormat /root/.configurationdisk`
- Open LUKS encrypted disk and name it *config*:
  `cryptsetup luksOpen /root/.configurationdisk config`
- Create filesystem on encrypted storage:
  `mkfs.ext4 /dev/mapper/config`
- Get UUID from virtual encrypted disk and add to fstab:
  `blkid /dev/mapper/config`
- Modify fstab entry and use *noauto* flag:
  
  ```UUID=d2dXXXZYY-YYZZ-YXXXXX-XX116	/root/.config	ext4	discard,noauto	0	0```
- Implement a Tang-Server, see https://github.com/latchset/clevis, and configure keys
  A Raspberry Pi Zero 2 is a very solid start. Continue when tang is working.
- Install clevis: `apt install clevis clevis-initramfs clevis-luks`
- My network card using driver r8169 does not support clevis-initramfs (well, it does not support setting an IP in early-boot-stages) and r8168-dkms sucks in a different way
- Alternative: create /etc/boot.d and place the following script:

  ````
  #!/bin/sh
  sleep 4
  
  clevis luks unlock -d /root/.configurationdisk && mount /root/.config
  
  sleep 1
  if [[ $(findmnt -M "/root/.config") ]]; then
      /etc/init.d/docker start
  fi
  ````
- Bind clevis to tang: `clevis luks bind -d /root/.configurationdisk tang '{"url":"http://mytangserver.lan"}
- Disable docker start, will only be started if disk could be decrypted and mounted

## Result
After reboot your rclone configuration is uncrypted and cronjobs can work. 
