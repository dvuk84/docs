This document will explain how to upgrade Cisco IOS on Catalyst switches.

* save running config, upload running IOS and config to the FTP server (ftp.server.com:/home/tftp/switch)
* then download the new IOS image and set it to boot and reload the switch

```
# wr
# copy flash: tftp:
# copy run tftp:
# copy tftp: flash:
# dir flash:
# conf t
# boot system flash:/c2960s-universalk9-mz.150-2.SE12.bin
# wr
# show boot
# reload
```
