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
  `UUID=d2dXXXZYY-YYZZ-YXXXXX-XX116	/root/.config	ext4	discard,noauto	0	0`

