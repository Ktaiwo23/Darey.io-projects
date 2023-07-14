## CLIENT-SERVER ARCHITECTURE WITH MYSQL

Understanding Client-Server Architecture
As you proceed your journey into the world of IT, you will begin to realise that certain concepts apply to many other areas. One of such concepts is – Client-Server architecture.

Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another. In their communication, each machine has its own role: the machine sending requests is usually referred as “Client” and the machine responding (serving) is called “Server”.
A simple diagram of Web Client-Server architecture is presented below:

![Web Client-Server architecture](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/6b338423-9a78-4757-bd36-146e3687c13e)

In the example above, a machine that is trying to access a Web site using Web browser or simply ‘curl’ command is a client and it sends HTTP requests to a Web server (Apache, Nginx, IIS or any other) over the Internet. If we extend this concept further and add a Database Server to our architecture, we can get this picture:

![Web Client-Server architecture with Database Server](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/0747b853-05e0-46f2-a019-58afa95f91fc)

In this case, our Web Server has a role of a “Client” that connects and reads/writes to/from a Database (DB) Server (MySQL, MongoDB, Oracle, SQL Server or any other), and the communication between them happens over a Local Network (it can also be Internet connection, but it is a common practice to place Web Server and DB Server close to each other in local network).
The setup on the diagram above is a typical generic Web Stack architecture that you have already implemented in previous projects (LAMP, LEMP, MEAN, MERN), this architecture can be implemented with many other technologies – various Web and DB servers, from small Single-page applications SPA to large and complex portals.

### IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)

To demonstrate a basic client-server using MySQL Relational Database Management System (RDBMS), Create and configure two Linux-based virtual servers (EC2 instances in AWS).

Server A name - `mysql server`
Server B name - `mysql client`
![Two Linux-based virtual servers (EC2 instances in AWS)](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/39465e91-f909-425c-8592-9e0ad1cd6fc2)

On mysql server Linux Server install MySQL Server software

On mysql client Linux Server install MySQL Client software 
![mysql server and mysql client installed](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/ab4d65ed-445a-4b01-8b33-8f9a2ea1bb2b)
By default, both of your EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’.
![Inbound rules of mysql server edited](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/b994b16c-c112-420c-b427-f3ef6c4e8e76)
To configure MySQL server to allow connections from remote hosts, run the command below.

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

Replace ‘127.0.0.1’ to ‘0.0.0.0’ like this:

![Configuration of MySQL server to allow connections from remote hosts](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/5712c03b-8d9c-456c-9c49-fee9ee85f62e)

set up a user, create a database and grant all privileges to the user created on mysql server

![mysql user and database setup](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/fed0aa83-9faa-42d4-a8aa-65c0ae2f8139)

From mysql client Linux Server connect remotely to mysql server Database Engine without using SSH. You must use the mysql utility to perform this action.

Check that you have successfully connected to a remote MySQL server and can perform SQL queries:

`Show databases;`

If you see an output similar to the below image, then you have successfully completed this project – you have deloyed a fully functional MySQL Client-Server set up.

![fully functional MySQL Client-Server set up](https://github.com/Ktaiwo23/Darey.io-projects/assets/134460769/f3765c3b-58d9-4fa8-a865-0d955ef1f581)
