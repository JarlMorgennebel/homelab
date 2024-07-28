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
- Create an 100 MByte file to be used as virtual disk image
    dd if=/dev/urandom of=/root/.configurationdisk bs=32k count=3200
