**Installing R1soft**

Debian official installation docs if you need them are here: http://wiki.r1soft.com/display/ServerBackup/Install+Server+Backup+Manager+on+Debian+and+Ubuntu
```
cd /tmp
wget http://repo.r1soft.com/r1soft.asc
apt-key add r1soft.asc
echo "deb http://repo.r1soft.com/apt stable main" >> /etc/apt/sources.list
apt-get update
apt-get install serverbackup-enterprise
serverbackup-setup --user admin --pass <PASSWORD>
/etc/init.d/cdp-server restart
```

Redhat official installation docs if you need them are here: http://wiki.r1soft.com/display/ServerBackupManager/Install+and+Upgrade+Server+Backup+Manager+on+CentOS%2C+RHE%2C+and+Fedora
```
vi /etc/yum.repos.d/r1soft.repo

[r1soft]
name=R1Soft Repository Server
baseurl=http://repo.r1soft.com/yum/stable/$basearch/
enabled=1
gpgcheck=0

yum install serverbackup-enterprise
serverbackup-setup --user admin --pass <PASSWORD>
/etc/init.d/cdp-server restart
```
Now you can access the web interface with the credentials set above.

**Setting up R1soft**

1. First we create a new volume (this will be our Netapp volume that we have mounted on the server)
```bash
http://backupserver.domain.com -> Settings -> Volumes -> New Volume (give it description and full path: /backup/vol <-- r1soft will create the 'vol' folder)
```

2. Before we add a device we need to install an agent on the target device
```bash
(apt-get|yum) install serverbackup-enterprise-agent
 ```
 
3. Then we add a device
```bash
http://backupserver.domain.com -> Protected Machines -> New Machine -> untick "Enable Encryption" and "Deploy agent software now"
on the agent/target device run serverbackup-setup --get-key http://backupserver.domain.com
```
 
4. Test agent connection
```bash
http://backupserver.domain.com -> Protected Machines -> <DEVICE> -> Actions -> Test Agent Connection
```
 
5. Now we add disk safes
```bash
http://backupserver.domain.com -> Disk Safes -> <DISKSAFE> -> Actions -> Edit Disk Safe -> Devices -> untick "Automatically add new devices" and add all mount points -> Save
```

**Building r1soft kernel module**

If you get the following error in the log files in /usr/sbin/r1soft/logs/server.log
```bash
2018-03-20 11:32:26,200 ERROR  - TASK(3ecfv330-86h-4f2d-1124-2c32v07adbbe:test-server.domain.com)<-AGENT(test-server.domain.com): An exception occurred during the request. Unable to find device with content id (d987994b-275h-3532-abf0-05a01179aa46)
```
it means you will need to build the kernel module for r1soft on that device. To do that run the following command on the machine:
```bash
serverbackup-setup --get-module
```

Or if you want to force build a module based on the current kernel headers without checking for existing drivers:
```bash
serverbackup-setup --get-module --no-binary
```

This should compile and install the module, however it may error out if you don't have kernel headers installed:
```
Please install the kernel headers for your operating system.
To install kernel headers execute:
apt-get install linux-headers-`uname -r`
```

In this case just run the command above and then try reinstalling the module. On a Redhat system do:
```bash
yum install kernel-headers-`uname -r` kernel-devel-`uname -r`
```

If you can't compile you can try downloading the binary module from http://repo.r1soft.com/modules or http://beta.r1soft.com/modules, placing it in _/lib/modules/r1soft/_ and restarting the _cdp-server_ daemon. In any other case you will need to raise a ticket with r1soft at https://r1soft.force.com.

**Performing a bare metal restore**

```
download a boot cd from http://repo.r1soft.com/bm/serverbackup-bootcd-agent.iso
wipe the target drive
boot the target device into the bootcd iso
set up networking on it manually or via 'sudo ceni' command
put r1soft into recovery mode with 'touch /usr/sbin/r1soft/.recovery-mode' and restart the agent
log into backupserver.domain.com -> Actions -> Open Recovery Points -> Choose a restore point -> Bare Metal Restore
follow the steps until done, it will take roughly an hour for every 50G
```

**Migrating R1soft**

If you don't want to set up backups from scratch and want to migrate the existing installation instead. Official migration docs if you need them are here: http://wiki.r1soft.com/display/CDP/Migrating+Enterprise+Edition

On the NEW backup server:
```
/etc/init.d/cdp-server stop
rm -rf /usr/sbin/r1soft/conf
rm -rf /usr/sbin/r1soft/log
rm -rf /usr/sbin/r1soft/data
iptables -I INPUT 1 -p tcp -s <OLDBACKUPSERVER> --dport 22 -j ACCEPT (allow OLD backup server to connect to the new one on port 22, insert the rule to position 1 before any explicit deny)
scp -r <USERNAME>@<OLDBACKUPSERVER>:/usr/sbin/r1soft/conf /usr/sbin/r1soft/
scp -r <USERNAME>@<OLDBACKUPSERVER>:/usr/sbin/r1soft/data /usr/sbin/r1soft/
scp -r <USERNAME>@<OLDBACKUPSERVER>:/usr/sbin/r1soft/log /usr/sbin/r1soft/
/etc/init.d/cdp-server start
```

Proceed to move disk safes one by one from the OLD backup server to the NEW one, official docs are here if you need them: http://wiki.r1soft.com/display/CDP/Copying+and+Moving+Disk+Safes  

For every disk safe we move, we first need to:
```
 - on the OLD server:
   * http://oldbackupserver.domain.com -> Disk Safes -> <DISKSAFE> -> click on it to expand and get the disk safe ID, we will need it for the step below
   * http://oldbackupserver.domain.com -> Disk Safes -> <DISKSAFE> -> Stop 
   * http://oldbackupserver.domain.com -> Disk Safes -> <DISKSAFE> -> Detach

 - on the NEW server:
   * scp -r <USERNAME>@<OLDBACKUPSERVER>:/backup/vol/<DISKSAFE> /backup/vol/
   * /etc/init.d/cdp-server restart
   * http://newbackupserver.domain.com -> Disk Safes -> Attach Existing Disk Safe (give it a description and paste the full path to the disk safe: /backup/vol/<DISKSAFE>)
   * http://newbackupserver.domain.com -> Disk Safes -> <DISKSAFE> -> Open (click a little blue triangle)
```

Repeat that for every disk safe. After everything has been moved across, delete the firewall rule that you previously added with:
```
iptables -D INPUT 1
```
