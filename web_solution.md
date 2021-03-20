
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


To ensure that the mounts are persistent, i edited the `/etc/fstab` config file using the UUID

![](https://github.com/drazen-dee28/Web-Solution-With-Wordpress/blob/main/img/mountcheck.jpg)


I checked the mounts as shown
![](https://github.com/drazen-dee28/Web-Solution-With-Wordpress/blob/main/img/checkmount.jpg)


- Preparing Database Server

I repeated the same steps like i did for webserver above. I repaced the  `The app-lv` logical volume with `db-lv` and volume group is vg-database

I created a directory called `/db`

I mounted the `db-lv`  logical volume on `/db` directory

`sudo mount /dev/vg-database/db-lv /db`

I checked the mounts as shown
![](https://github.com/drazen-dee28/Web-Solution-With-Wordpress/blob/main/img/checkdb.jpg)


To ensure that the mounts are persistent, i edited the `/etc/fstab` config file using the UUID as executed on the webserver.
  
- Installation of Wordpress on the Webserver

I installed Apache(Httpd) and other dependencies as shown

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`


I installed the remaining `php` dependencies 

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo dnf module list php Remi's Modular repository for Enterprise Linux 8 -x86_64`

After installing the dependencies , i reset and enbled `php` as shown

`sudo dnf module reset php`

`sudo dnf module enable php:remi-7.4`


I installed `php` Fast Process Manager

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

I started and enabled the `php` Fast Process Manager

`sudo systemctl start php-fpm`
`sudo systemctl enable php-fpm`


In order to instruct SElinux to allow Apache execute `php` via `php-fpm` i ran the following

`sudo setsebool -P httpd_execmem 1`

Then ,i restarted Apache
`sudo systemctl restart httpd`


Installation of Wordpress

I created a folder named wordpress

`sudo mkdir -p wordpress`

I downloaded and extracted into the `wordpress` folder

`sudo wget http://wordpress.org/latest.tar.gz`

I extracted the wordpress that was downloaded

`sudo tar xzvf latest.tar.gz`

Credits

