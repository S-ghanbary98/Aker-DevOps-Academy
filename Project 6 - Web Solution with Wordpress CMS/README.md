# Intro

In this project I will deploy a full-scale three tier web solution with the WordPress content management system. This will consist of: 
- Configuring storage system for Web and Database servers.
- Install and connect WordPress to MySQL database server.

![three-tier-arch](img/three-tier-architecture.png)

My three-tier architecture will involve:
- Client - My personal Laptop
- Web Server - A configured EC2 (wordpress)
- Database - A configured EC2 (DB)

The linux based distribution I will be using is 'RedHat'.


## Web Server

- Firstly we spin-up an ec2 'RedHat', that will serve as a our web-server.
- Next we create 3 new volumes in the same AZ as our web-server with each being 1gb and attach them to our ec2 instance.

![web-server](img//web-server.png)
![volumes](img/server-volumes.png)

- We can see the attached block devices to our web server through `lsblk`.

![attached-volumes](img/attached_volumes.png)


```
# to seee mounts
df -h 
```

![mounts](img/created-block-devices.png)

![mounts](img/created-block-devices.png)
![s](img/created-block-devices.png)


- Next we move on to creating one partition in each of the three volumes 'xvdf', 'xvdg', and 'xvdh'.

```
sudo gdisk /dev/xvdf
# input values include 'n', 'w', and 'yes'
```
![partioning](img/disk_partition.png)

- A check to view partitions `lsblk`.
![partioning](img/partioned_volumes.png)

- We can also view these partiotions via:

```
sudo ym install lvm2
sudo lvmdiskscan
```
![partioning](img/lvm_disk%20scan.png)



- Next we use `pvcreate` to mark each of the disks as physical volumes.

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

# verify
sudo pvs
```
![pvs](img/pvs.png)

- We then use `vgcreate` to add all three physical volumes to a volume group called <strong>webdata-vg</strong>.

```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

# verify
sudo vgs
```
![vgs](img/vgs.png)

- We next move on to creating our logical volumes (LV), we shall call it <strong>apps-lv</strong>, and <strong>logs-lv</strong>.

```
sudo lvcreate -n logs-lv -L 14G webdata-vg
sudo lvcreate -n apps-lv -L 14G webdata-vg

# verify 
sudo lvs
```

![lvs](img/lvs.png)

- Finally we verify the volume setup:

```
# Complete view
sudo vgdisplay -v 

sudo lsblk
```

- We use mkfs.ext4 to format the logical volumes with the ext4 filesystem. Furthermore we will also create the directories to store our website files and the backups of the log data.
- Additionally we will also mount the created directory to our logical volume 'apps-lv'.

```
# format logical volumes
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

# create directories for our filesystem.
sudo mkdir -p /var/www/html
sudo mkdir -p /home/recovery/logs

# mount the /var/www/html/directory to apps-lv logical volume.
sudo mount /dev/webdata-vg/apps-lv /var/www/html/

# Backup our files into log directory /var/log into the directory we just created /home/recovery/logs
sudo rsync -av /var/log/. /home/recovery/logs/

sudo mount /dev/webdata-vg/logs-lv /var/log

# Restore log files back into /var/log
sudo rsync -av /home/recovery/logs/. /var/log
```

## Ensure Mount Configuration Persistance

- We next move on to updating `/etc/fstab`, so that our mount configuration persists after our server is restarted.
- We add the UUID values to `/etc/fstab`.

```
# view UUID
sudo blkd

# update fstab file with new uuid
sudo vi /etc/fstab

```
![](img/uuid.png)
![](img/fstab_config.png)

- Finally we test our configuration.

```
sudo mount -a sudo systemctl daemon-reload
df -h
```
![](img/verify-setup.png)

## Installing WordPress on Web Server

```
sudo yum -y update

#install wget, apache, and dependencies
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

#start Apache
sudo systemctl enable httpd sudo systemctl start httpd


sudo systemctl restart httpd

# Donwload wordpress in var/www/.html

mkdir wordpress cd wordpress sudo wget http://wordpress.org/latest.tar.gz sudo tar xzvf latest.tar.gz sudo rm -rf latest.tar.gz cp wordpress/wp-config-sample.php wordpress/wp-config.php cp -R wordpress /var/www/html/
```