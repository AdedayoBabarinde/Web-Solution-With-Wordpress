
#  Web Solution with Wordpress
In this project, a 3 tier architecture to host a WordPress site will be built. 



Network Information [All servers were hosted on AWS]

Webserver  :172.31.30.221

Database server  : 172.31.19.96

Client Machine : My Laptop




I prepared the Web Server by adding three new volumes of equal size and partition them as shown

![](https://github.com/drazen-dee28/Web-Solution-With-Wordpress/blob/main/img/partition.jpg)


I created three physical volumes as follows

`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /devxvdh1`

I created a volume group named `webdata-vg` from the three physical volumes as follows

`sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

I created two logical volumes named `app-lv`(storage of website data) and `logs-lv`(for logs storage) as follows

 `sudo lvcreate -n app-lv -L 14G webdata-vg`
  
`sudo lvcreate -n logs-lv -L 14G webdata-vg`
  
I checked the setup with `lsblk` command

![](https://github.com/drazen-dee28/Web-Solution-With-Wordpress/blob/main/img/check.jpg)


I formatted the logical volume with Ext4 filesystem

`sudo mkfs.ext4 /dev/webdata-vg/app-lv`


I created `/var/www/html` directory to store website files

`sudo mkdir -p /var/www/html`


I created `/home/recovery/logs` directory to store backup of log data

`sudo mkdir -p /home/recovery/logs`


I mounted /var/www/html on the `app` logical volume as shown

`sudo mount /dev/webdata-vg/app-lv /var/www/html`


I backed up the contents of `/var/log` directory before mounting on it as shown

`sudo rsync -av /var/log /home/recovery/logs`


I mounted the /var/log on the logs logical volume 

`sudo mount /dev/webdata-vg/logs-lv /var/log`


Since mounting will delete the contents of `/var/log` ,i copied back the backed up version

`sudo rsync -av /home/recovery/logs/log/ /var/log`