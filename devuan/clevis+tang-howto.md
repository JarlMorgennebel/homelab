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

