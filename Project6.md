## WEB SOLUTION WITH WORDPRESS

In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).
The project consistes of two parts:
1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give us practical experience of working with disks, partitions and volumes in Linux.
2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify our skills of deploying Web and DB tiers of Web solution.

#### Three-tier Architecture
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture. Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

![1-Three-tier Architecture](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/700866a0-f8f6-4b12-a09b-950e9cfbe9ba)

1. Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
2. Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
3. Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server

   In this project, we will have the hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively. We will be working working with several storage and disk management concepts.

   The 3-Tier Setup
1. A Laptop or PC to serve as a client
2. An EC2 Linux Server as a web server (This is where you will install WordPress)
3. An EC2 Linux server as a database (DB) server

In previous projects we used ‘Ubuntu’, but it is better to be well-versed with various Linux distributions, thus, for this projects we will use very popular distribution called ‘RedHat’ (it also has a fully compatible derivative – CentOS). For Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like ec2-user@<Public-IP>

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/96a27db4-538b-47a9-9201-1ffeecd2eb02)
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/5cc441bd-2b8e-4e41-b9ad-72e5c5786943)
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/4ffe5f6b-09da-47be-9d94-71f7ae8ce83d)

Open up the Linux terminal to begin configuration

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/f42c1950-d294-4ebc-82aa-c5959a602d8d)

Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/2be95566-4c66-4f92-a66b-80c08b465504)

Use df -h command to see all mounts and free space on your server

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/bdc07c70-1c70-4de5-a31f-735d08f49387)

Use gdisk utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/nvme1n1`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/a36c04c0-2d75-431f-9d25-ce51b47465cf)

Use lsblk utility to view the newly configured partition on each of the 3 disks.

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/9bd84f60-3a8f-4fea-a1e3-55e365cd2ff4)

Install lvm2 package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/26217e8e-4625-4b90-907c-5d0938d8e363)
![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/d2246938-ba0a-4494-83d3-fb9348767f61)

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
`sudo pvcreate /dev/nvme1n1p1`
`sudo pvcreate /dev/nvme2n1p2`
`sudo pvcreate /dev/nvme3n1p3`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/ca00690e-756a-40e8-b46d-2a119e2686ce)

Verify that your Physical volume has been created successfully by running `sudo pvs`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/4d64038e-a0d9-4b52-8e51-877fed8de50f)

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
`sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p2 /dev/nvme3n1p3`

Verify that your VG has been created successfully by running `sudo vgs`

Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

`sudo lvcreate -n apps-lv -L 14G webdata-vg`

`sudo lvcreate -n logs-lv -L 14G webdata-vg`

Verify that your Logical Volume has been created successfully by running `sudo lvs`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/bbfaa119-0a64-442e-9a48-1f4931fa7c5a)

Verify the entire setup

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

`sudo lsblk`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/eb7f2592-f2d6-4122-a8c8-272ec65b717b)

Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/8d20d6f5-edee-4867-9041-672ee3f4769a)

Create /var/www/html directory to store website files

`sudo mkdir -p /var/www/html`

Create /home/recovery/logs to store backup of log data

`sudo mkdir -p /home/recovery/logs`

Mount /var/www/html on apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/5e507fb7-6d7f-4a74-ba2e-82ade3ace33f)

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

Restore log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/log/. /var/log`

Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file;

`sudo blkid`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/f0efee7c-fb46-4861-a7d0-32d94746bdf0)

`sudo vi /etc/fstab`

Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/58d1da4c-4a98-44af-b575-52c892dc277a)

Test the configuration and reload the daemon

`sudo mount -a`

`sudo systemctl daemon-reload`

Verify your setup by running `df -h`, output must look like this:

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/393ca96a-1c6d-4a89-a2ea-77e1db8ecd1c)

## PREPARE THE DATABASE SERVER

Launch a second RedHat EC2 instance that will have a role – ‘DB Server’ Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/f2d8f2f2-49dc-4668-9e1d-9e30cd01e392)

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/bcb5d4f8-d1f2-4b48-9ae4-65850762a2e4)

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/d813f208-b58a-45e8-af48-759902a80023)

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/cfc88481-cd7e-497c-8107-4e6ab2ef79d2)

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/ddf54801-3eeb-4124-b461-ce1a9699e41b)

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/259adf1f-96a7-4e13-8875-6f86989dcdc8)

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/b5935cae-d58f-4012-934e-690ceb286b92)

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/2b0eaead-7544-45c0-9852-d8062a1957f4)

## Install WordPress on your Web Server EC2

Update the repository

`sudo yum -y update`

Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/8d4cb017-f139-4e5d-8cf0-7f932dc134c4)

Start Apache

`sudo systemctl enable httpd`

`sudo systemctl start httpd`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/50f0ec11-6231-44b6-807a-e8d25e4646f8)

To install PHP and it’s depemdencies

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo yum module list php`

`sudo yum module reset php`

`sudo yum module enable php:remi-7.4`

`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`setsebool -P httpd_execmem 1`

Restart Apache

`sudo systemctl restart httpd`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/80d2fd0a-500b-4160-aa46-8c1ef68c2ffa)

Download wordpress and copy wordpress to var/www/html

  `mkdir wordpress`
  
  `cd   wordpress`
  
  `sudo wget http://wordpress.org/latest.tar.gz`
  
  `sudo tar xzvf latest.tar.gz`
  
  `sudo rm -rf latest.tar.gz`
  
  `sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php`
  
  `sudo cp -R wordpress /var/www/html/`

Configure SELinux Policies

  `sudo chown -R apache:apache /var/www/html/wordpress`
  
  `sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`
  
  `sudo setsebool -P httpd_can_network_connect=1`

  ![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/ecca70a5-9144-443f-9482-17f0989f69ce)

## Install MySQL on your DB Server EC2

`sudo yum update -y`

`sudo yum install mysql-server`

Verify that the service is up and running by using `sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:

`sudo systemctl restart mysqld`

`sudo systemctl enable mysqld`

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/38aeb897-1ab6-4900-9679-7d6f04b11ffc)

## Configure DB to work with WordPress

`sudo mysql`

CREATE DATABASE wordpress;

CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';

GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';

FLUSH PRIVILEGES;

SHOW DATABASES;

exit

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/41bd44f3-034d-4f7a-a79b-349932ad376f)

## Configure WordPress to connect to remote database.

Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/61f267e7-fce9-4aa7-b4be-9280057acbbd)

Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

`sudo yum install mysql`

`sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`

Verify if you can successfully execute `SHOW DATABASES;` command and see a list of existing databases.

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/c433bddb-b1dd-4344-ba31-07c5d1764554)

Change permissions and configuration so Apache could use WordPress:

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/d535ddab-3f53-4dd6-ba64-fc27780a3685)

Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/7904abb1-38a3-4929-9296-ad68afc4b0f9)
