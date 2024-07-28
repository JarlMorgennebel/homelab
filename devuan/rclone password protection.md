# Rclone .config protection

Situation:
* /root is not encrypted
* Your storage partitions are encrypted and protected against theft

## Result
An attacker/intruder may gain access to files by restoring the backup from rclone to an unencrypted disk.
