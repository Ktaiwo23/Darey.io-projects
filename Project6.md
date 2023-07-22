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





