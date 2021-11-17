# Myproject-7#
The diagram below shows a common pattern where several stateless Web Servers share a common database and also access the same files using Network File System (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files. IMAGE 01
![01](https://user-images.githubusercontent.com/91284177/141683652-03f06854-fdc3-4a8a-b2da-a466e2cb5ebd.png)

#STEP 1 – PREPARE NFS SERVER#

I spin up a new EC2 instance with RHEL Linux 8 Operating System. xfs was used in formatting the disks.
I created 3 Logical Volumes. lv-opts, lv-apps, and lv-logs. Physical volume pv, Group volume gv and a logical volume lv. IMAGE 02, 03, 04 and 05

![02](https://user-images.githubusercontent.com/91284177/141684021-29b7a45d-82a0-44b8-a8d0-6beb88428469.png)
![03](https://user-images.githubusercontent.com/91284177/141684027-cfb2cf07-4221-4b7d-85a1-d1b1968b6c0b.png)
![04](https://user-images.githubusercontent.com/91284177/141684035-6fddee5b-c653-477e-b0d3-8b084c443593.png)
![05](https://user-images.githubusercontent.com/91284177/141684045-8d58c33c-769e-4193-87eb-c2256a6ad5e6.png)

I installed NFS server, configure it to start on reboot and make sure it is up and running. IMAGE 06 & 07
![06](https://user-images.githubusercontent.com/91284177/141684131-3d7529b2-c445-4b4e-80ff-2831258d3584.png)
![07](https://user-images.githubusercontent.com/91284177/141684132-aa1fa957-f5b7-4b8e-a838-5605c01f9217.png)

I set up permission that will allow our Web servers to read, write and execute files on NFS. IMAGE 08 and 09

![08](https://user-images.githubusercontent.com/91284177/141684257-958fc887-bb10-40b9-becd-33537dd74361.png)
![09](https://user-images.githubusercontent.com/91284177/141684291-e3d72b51-1515-4051-b1e6-fbe0e78b6ad3.png)

I used <rpcinfo -p | grep nfs> to confirm where the port NFS is listening from. IMAGE 10
![10](https://user-images.githubusercontent.com/91284177/141684368-2fb4c9f9-de78-43f3-a45f-e3e98f5b7f91.png)

#STEP 2 — CONFIGURE THE DATABASE SERVER#

I installed mysql server. IMAGE 11
![11](https://user-images.githubusercontent.com/91284177/141685537-2d917ad6-4dfe-456d-addc-ef244073c263.png)

A database named "Tooling" was created. IMAGE 12

![12](https://user-images.githubusercontent.com/91284177/141686037-e1d718a7-e440-493e-9082-769c008fedc5.png)

I confirmed if the full web access was granted to the user is existing. IMAGE 13
![13](https://user-images.githubusercontent.com/91284177/141686607-82941865-b0a1-4203-9482-aaaa57993b26.png)

The mysql database configuration file was edited by changing the bind-address 127.0.0.1 to 0.0.0.0 for access from anywhere. IMAGE 14
![14](https://user-images.githubusercontent.com/91284177/141788701-8662668b-61d3-4912-94a2-7a6b9b6fc831.png)


#Step 3 — Prepare the Web Servers#

In step 3, Web servers was created to serve the same content from shared storage solutions, in our case – NFS Server and MySQL database. The DB can be accessed for reads and writes by multiple clients. For storing shared files that my Web Servers will use – I will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

This approach will make my Web Servers stateless, which means I will be able to add new ones or remove them whenever its necessary, and the integrity of the data (in the database and on NFS) will be preserved.

I launched a new EC2 instance with RHEL 8 Operating System. Installed NFS client on it.

For web server1, I created a directory /var/www and mounted the private IP address of NFS server on it. IMAGE 15
![15](https://user-images.githubusercontent.com/91284177/141792761-7ad59a7a-df56-4019-bc54-56cb7693fcf9.png)

I opened the fstab and saved the NFS private IP address there. IMAGE 16
![16](https://user-images.githubusercontent.com/91284177/141794944-7c393436-769f-4b11-b6aa-b9e2312546d7.png)
I then installed apache. IMAGE 17 and 18
![17](https://user-images.githubusercontent.com/91284177/141795244-1996d4fe-663b-42f1-8be0-cd8df548102d.png)
![18](https://user-images.githubusercontent.com/91284177/141820315-a2a07115-18e0-485e-8e43-3290631610d6.png)

I installed github, forked the tooling source code from  Darey.io Github Account using; <https://github.com/larrymie/Project-7.git>

IMAGE 19, 20 and 21

![19](https://user-images.githubusercontent.com/91284177/141957365-06717503-6d3a-42ad-8d00-d5c33dba31ce.png)
![20](https://user-images.githubusercontent.com/91284177/141957378-bdb533d5-6091-4be7-9959-224f7ce91fec.png)
![21](https://user-images.githubusercontent.com/91284177/141957386-3162d559-b1d0-4195-88f9-b060956441e7.png)
php was installed, I faced a major CHALLENGE of getting the latest version of the php. I troubleshoot to solve the problem using by installing the following;

First, I installed the EPEL repository.
 sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
Next, I installed yum utils and enable remi-repository using the command below.
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo dnf module list php
 sudo dnf module reset php
 sudo dnf module enable php:remi-8.0


Finally, I installed PHP, PHP-FPM (FastCGI Process Manager) and associated PHP modules using the command.

$ sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

IMAGE 22 confirmed the installed php as active
![22](https://user-images.githubusercontent.com/91284177/142046861-2006141d-b2a8-4981-981a-4a8d703fe71c.png)
