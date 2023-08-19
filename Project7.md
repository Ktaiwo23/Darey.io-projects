# DevOps Tooling Website Solution

In previous project 6 we implemented a WordPress based solution that is ready to be filled with content and can be used as a full fledged website or blog. Moving further we will add some more value to our solutions that a DevOps team could utilize. We want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.

The tools we want our team to be able to use are well known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:

1. Jenkins – free and open source automation server used to build CI/CD pipelines.
2. Kubernetes – an open-source container-orchestration system for automating computer application deployment, scaling, and management.
3. Jfrog Artifactory – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
4. Rancher – an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.
5. Grafana – a multi-platform open source analytics and interactive visualization web application.
6. Prometheus – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
7. Kibana – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

### Set up and Technologies used:

In this project you will implement a solution that consists of following components:

1. Infrastructure: AWS
2. Webserver Linux: Red Hat Enterprise Linux 8
3. Database Server: Ubuntu 20.04 + MySQL
4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server
5. Programming Language: PHP
6. Code Repository: GitHub



On the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files.

![image](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/354e5b5d-9feb-424b-a99b-c9b8bcbfe17e)

It is important to know what storage solution is suitable for what use cases, for this – you need to answer following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Base on this you will be able to choose the right storage system for your solution

## Step 1: Prepare the NFS Server

1. Spin up a new EC2 instance 

2. Based on your LVM experience from Project 6, Configure LVM on the Server.

Instead of formating the disks as ext4 you will have to format them as xfs

Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs

Create mount points on /mnt directory for the logical volumes as follow:

Mount lv-apps on /mnt/apps – To be used by webservers

Mount lv-logs on /mnt/logs – To be used by webserver logs

Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

4. Install NFS server, configure it to start on reboot and make sure it is up and running

`sudo yum -y update`

`sudo yum install nfs-utils -y`

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`

![install NFS client](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/3088ef15-1585-4da9-8bcd-fb02f4092aa4)


Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security.

To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:

![getting the subnet CIDR of webserver1](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/42efd1b8-e9b1-4f99-8a97-5c58b36d63ae)


Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

`sudo chown -R nobody: /mnt/apps`

`sudo chown -R nobody: /mnt/logs`

`sudo chown -R nobody: /mnt/opt`

`sudo chmod -R 777 /mnt/apps`

`sudo chmod -R 777 /mnt/logs`

`sudo chmod -R 777 /mnt/opt`

`sudo systemctl restart nfs-server.service`

![setting up permisson to allow webserver read write execute](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/49ba1271-bae9-472d-b4d6-03e2b7c1cb29)


Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):


`sudo vi /etc/exports`

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

![VI into etc exports to configure access to NFS for clients within the same subnet](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/7a343287-6021-4971-adb6-262af46be889)


`sudo exportfs -arv`

Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

`rpcinfo -p | grep nfs`

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/c0a2da49-b28b-4107-9885-1cc99c122573)


 In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049

![edit inbound rule for TCP and UDP](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/948151f4-e9bb-4e3b-8b66-048f1d1e34e6)


## Step 2: Configure the Database Server

By now you should know how to install and configure a MySQL DBMS to work with remote Web Server

Install MySQL server

![changing bind address to 0 0 0 0](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/aa1cbd5a-c175-4e55-b7f2-8649d2348e48)


Create a database and name it tooling

Create a database user and name it webaccess

Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

![show database use tooling show tables show users](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/134e0f12-80cf-4fc7-8019-74bb18e4ba33)



## Step 3: Prepare the Web Servers

We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.

You already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

Configure NFS client (this step must be done on all three servers)

Deploy a Tooling application to our Web Servers into a shared NFS folder

Configure the Web Servers to work with a single MySQL database

Launch a new EC2 instance 

Install NFS client

`sudo yum install nfs-utils nfs4-acl-tools -y`

Mount /var/www/ and target the NFS server’s export for apps

`sudo mkdir /var/www`

`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

![make directory for mount point](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/8f24f825-0e30-4897-af70-83c78ba29cc8)

![mkdir var www and mount NFS target for mnt apps](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/868110de-2d2a-4202-ae68-a3e5e8a937f0)


Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:

`sudo vi /etc/fstab`

add following line

<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0

![add NFS IP to etc fstab](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/fa4ca360-e717-4f21-9d56-290cdf63b2a9)


Install Remi’s repository, Apache and PHP

`sudo yum install httpd -y`

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

`sudo dnf module reset php`

`sudo dnf module enable php:remi-7.4`

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`sudo setsebool -P httpd_execmem 1`

Repeat steps 1-5 for another 2 Web Servers.

![install RemiRepo Apache and PHP](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/0efd963b-784c-4ab9-b966-2e03c2a14521)



Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.

![creating test md](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/ac709231-0708-4c92-8f83-59ec26c4d410)


Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №4 to make sure the mount point will persist after reboot.

![Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/17d6d7da-c359-401e-b1cd-cb26460ab739)

![make sure mount point for apache on webserver persists](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/ff6d68f4-4f17-4307-bd7c-2edc85d53955)

Fork the tooling source code from Darey.io Github Account to your Github account.

![download git run git init fork the tooling source code from link provided](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/de74b56a-2442-4e00-a504-4ec6c975289a)


Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html

![deploy the tooling website code to the webserver](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/fe2d5f3f-5e9a-41f7-80b3-4e59bb932e85)


 Do not forget to open TCP port 80 on the Web Server.

 ![open http port 80 on web server](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/c5f18cfe-7e30-4f70-90ac-44e25455bc02)


If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux sudo setenforce 0
To make this change permanent – open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabledthen restrt httpd.

![disable selinux](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/0f0037e8-5e12-46de-82e1-be3660b901af)


Update the website’s configuration to connect to the database (in /var/www/html/functions.php file).

![uisng vi to update website configuration to connect to DB](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/fbcae4f2-4bf1-4c9e-a3f2-410fee1716f7)


Apply tooling-db.sql script to your database using this command mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql

![apply tooling script to database using command mysql -h -u](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/4116a0b1-988b-4737-9ba2-4a76b5126785)

![confirming the content of the html repo is same as in var www html](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/5abdf554-9739-4091-ba4a-d2386b1d7ec2)



Create in MySQL a new admin user with username: myuser and password: password:
INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);

Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and make sure you can login into the website with myuser user.
