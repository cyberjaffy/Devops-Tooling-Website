# DevOps-Tooling-Website-Solution

In a previous project, we implemented a WordPress-based solution that can be used as a fully-fledged website or blog. Moving forward, we want to introduce a set of DevOps tools that will help your team in day-to-day activities in managing, developing, testing, deploying, and monitoring different projects.

## Setup and Technologies Used in this Project

You will implement a tooling website solution that provides easy access to DevOps tools within the corporate infrastructure. The solution will include the following components:

1. Infrastructure: AWS
2. Webserver Linux: Red Hat Enterprise Linux 8
3. Database Server: Ubuntu 24.04 + MySQL
4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server
5. Programming Language: PHP
6. Code Repository: GitHub

## Architecture

In this project, you will implement a tooling website solution that makes access to DevOps tools within the corporate infrastructure easily accessible.
The architecture consists of the following components:

1. Infrastructure: AWS Cloud
2. Web Server OS: Red Hat Enterprise Linux 8
3. Database Server OS: Ubuntu 24.04 + MySQL
4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server
5. Programming Language: PHP
6. Code Repository: GitHub

The infrastructure will follow a three-tier architecture pattern with stateless Web Servers sharing a common database and using Network File System (NFS) as shared storage.

## Architecture Diagram

<img width="1660" height="1052" alt="image" src="https://github.com/user-attachments/assets/ba1363bf-e069-463f-a8c2-1b5b9ff9a89f" />


It is important to know what storage solution is suitable for what use cases, for this - we need to answer following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Base on this you will be able to choose the right storage system for your solution.

## Step 1 -  Prepare NFS Server

1. Spin up a new EC2 instance with RHEL Linux 9 Operating System.
2. Based on your LVM experience from Project 6, Configure LVM on the Server. Create 3 volumes in the same AZ as your Web Server EC2.

<img width="2254" height="452" alt="image" src="https://github.com/user-attachments/assets/4670be83-d91b-4bba-ba9e-3b510e6b627f" />

3. SSH into your instance and update your machine:

   ```
   sudo yum update -y
   ```
  <img width="2874" height="1684" alt="image" src="https://github.com/user-attachments/assets/ee35a1e4-722a-48d4-913e-0c07d4144407" />

   
5. List the disk:

   ```
   lsblk
   ```

6. Use df -h command to see all mounts and free space:

   ```
   df -h
   ```

7. Use gdisk utility to create a single partition on each of the 3 disks, might need to download gdisk for RHEL with wget:

   ```
   wget https://mirror.stream.centos.org/9-stream/AppStream/x86_64/os/Packages/gdisk-1.0.7-5.el9.x86_64.rpm 
   ```
   ```
   sudo yum install ./gdisk-1.0.7-5.el9.x86_64.rpm
   ```

   ```
   sudo gdisk /dev/nvme1n1
   ```
   ```
   sudo gdisk /dev/nvme2n1
   ```
   ```
   sudo gdisk /dev/nvme3n1
   ```   
<img width="1274" height="1418" alt="image" src="https://github.com/user-attachments/assets/713fb632-a494-428d-8d87-7a268181efed" />


9. Install lvm2 package: creating logical volumes, which can be resized or moved without needing to unmount file systems.

    ```
    sudo yum install lvm2 -y
    ```

10. Check for available partitions.

    ```
    sudo lvmdiskscan
    ```

11. Create Physical Volumes Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

    ```
    sudo pvcreate /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
    ```

12. Verify that your Physical volume has been created successfully

    ```
    sudo pvs
    ```

13. Use vgcreate utility to add all 3 PVs to a volume group (VG) Name the VG webdata-vg

    ```
    sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
    ```
    <img width="733" height="47" alt="Screenshot 2025-12-14 at 02 15 01" src="https://github.com/user-attachments/assets/dfb4f63a-4e59-4a11-bf87-86b5cd3f6c25" />


14. Verify that your VG has been created successfully

    ```
    sudo vgs
    ```

15. Create Logical Volumes Use lvcreate utility to create logical volumes

    ```
    sudo lvcreate -L 10G -n lv-apps webdata-vg
    sudo lvcreate -L 10G -n lv-logs webdata-vg
    sudo lvcreate -L 10G -n lv-opt  webdata-vg
    ```

16. Verify that our Logical Volume has been created successfully

    ```
    sudo lvs
    ```

   <img width="1398" height="150" alt="image" src="https://github.com/user-attachments/assets/23120442-4173-4b29-9314-8f7993d51467" />

17. Verify the entire setup #view complete setup - VG , PV, and LV

    ```
    sudo vgdisplay -v
    ```

18. Format the disks as xfs instead ext4 you will have to format them as xfs. Ensure there are 3 Logical Volumes lv-opt lv-apps, and lv-logs

    ```
    sudo lsblk
    ```

19. Format the Logical Volumes as XFS:

    ```
    sudo mkfs.xfs /dev/webdata-vg/lv-apps
    sudo mkfs.xfs /dev/webdata-vg/lv-logs
    sudo mkfs.xfs /dev/webdata-vg/lv-opt
    ```

20. Create mount points on /mnt directory for the logical volumes:

    ```
    sudo mkdir -p /mnt/apps /mnt/logs /mnt/opt
    ```

21. Mount Logical Volumes

    ```
    sudo mount /dev/webdata-vg/lv-apps /mnt/apps
    sudo mount /dev/webdata-vg/lv-logs /mnt/logs
    sudo mount /dev/webdata-vg/lv-opt /mnt/opt
    ```

22. Add Mount Points to /etc/fstab

    ```
    sudo vi /etc/fstab
    ```

24. run `blkid` to get the uuid of the logical volumes 
25. Add the following lines:

    ```
   UUID=67c20376-144e-44a2-a275-6cae7dc65503  /mnt/opt   xfs  defaults  0 0
   UUID=9d0fef3f-fe1a-4de2-902f-4dbaa1618c4c  /mnt/apps  xfs  defaults  0 0
   UUID=b1dc575a-a985-4217-8184-77bfb9dad7cf  /mnt/logs  xfs  defaults  0 0
    ```
   <img width="1838" height="492" alt="image" src="https://github.com/user-attachments/assets/98ad14ce-ae7d-4f68-9c01-4add8392432e" />

24. Verify Mounts:

    ```
    sudo mount -a
    ```

25. Install NFS server, configure it to start on reboot and make sure it is u and running. Update the System and Install NFS Utilities:

    ```
    sudo yum -y update
    sudo yum install nfs-utils -y
    sudo systemctl start nfs-server.service
    sudo systemctl enable nfs-server.service
    sudo systemctl status nfs-server.service
    ```

26.  Export the NFS Mounts Use subnet cidr to connect as clients. For simplicity, we will install our all three Web Servers inside the same subnet, but in production set up would probably want to separate each tier inside its own subnet for higher level of security. To check our subnet cidr - open our EC2 details in AWS web console and locate Networking tab and open a Subnet link:

27. Set Permissions on Mount Points:

    ```
     sudo chown -R nobody:nobody /mnt/apps
     sudo chown -R nobody:nobody /mnt/logs
     sudo chown -R nobody:nobody /mnt/opt
     sudo chmod -R 777 /mnt/apps
     sudo chmod -R 777 /mnt/logs
     sudo chmod -R 777 /mnt/opt
    ```
    
28. Restart NFS Server
    ```
    sudo systemctl restart nfs-server.service
    ```

29. Configure access to NFS for clients within the same subnet (example of Subnet CIDR - 172.31.32.0/20 ):

    ```
    sudo vi /etc/exports
    ```

30. Add the following lines:

    ```
    /mnt/apps 172.31.32.0/20(rw,sync,no_all_squash,no_root_squash)
    /mnt/logs 172.31.32.0/20(rw,sync,no_all_squash,no_root_squash)
    /mnt/opt 172.31.32.0/20(rw,sync,no_all_squash,no_root_squash)
    ```
   <img width="960" height="168" alt="image" src="https://github.com/user-attachments/assets/bc59c205-72ae-4aef-83d0-15b18c3fb264" />

31. Export the NFS Shares:

    ```
    sudo exportfs -arv
    ```
   <img width="730" height="170" alt="image" src="https://github.com/user-attachments/assets/dc6401ca-6f09-4e98-a835-5bf138c71c80" />


32. Check which port is used by NFS and open necessary ports in Security Groups (add new Inbound Rule). Check NFS Ports:

    ```
    rpcinfo -p | grep nfs
    ```
   <img width="730" height="170" alt="image" src="https://github.com/user-attachments/assets/c29b9fc6-e235-490c-8c3b-0fb7584df770" />

    
### Important note: In order for NFS server to be accessible from our client,we open following ports: TCP 111, UDP 111, UDP 2049.

<img width="1137" height="257" alt="Screenshot 2025-12-14 at 04 37 35" src="https://github.com/user-attachments/assets/086c634b-a6e4-4a28-b645-b6bc9ebed216" />

## Step 2 - Configure the database server

Create EC2 instance of t2.micro type with Ubuntu Server launch in the default region.

   ![image 8](https://github.com/Captnfresh/DevOps-Tooling-Website-Solution/blob/main/DevOps%20Tooling%20Website/image%208.jpg)

1. Update and Upgrade the server:

   ```
   sudo apt update -y

   sudo apt upgrade -y
   ```

2. Install MySQL server:

   ```
   sudo apt install mysql-server
   ```
3. Check status:

   ```
   sudo systemctl status mysql
   ```
4. Enable MySQL to start on boot:

   ```
   sudo systemctl enable mysql
   ```
5. Create a database Tooling and User name it webaccess.

   ```
   sudo mysql
   CREATE DATABASE tooling;
   CREATE USER 'webaccess'@'%' IDENTIFIED BY 'mypass';
   GRANT ALL PRIVILEGES ON tooling. * TO 'webaccess'@'%' WITH GRANT OPTION;
   exit
   ```

<img width="1206" height="694" alt="image" src="https://github.com/user-attachments/assets/3dca4c4c-effa-4313-9268-94619ad24748" />

6. Update the bind-address in the /etc/mysql/mysql.conf.d/mysqld.cnf file to allow remote connections:

   ```
   sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
   ```
   <img width="1578" height="1690" alt="image" src="https://github.com/user-attachments/assets/e390a233-c880-4d37-878c-512a79eb85ab" />


##  Step 3 - Prepare the webservers

1. Update machine:

   ```
   sudo yum update -y
   ```

2. Install NFS CLIENT:

   ```
   sudo yum install nfs-utils nfs4-acl-tools -y
   ```

3. Mount /var/www/ and target the NFS server's export for apps

   ```
   sudo mkdir /var/www
   sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
   ```
   
4. Verify that NFS was mounted successfully

   ```
   df -h
   ```  
   <img width="1394" height="446" alt="image" src="https://github.com/user-attachments/assets/1bd446ae-9ac0-4a2c-8177-f9155b59b8e3" />

   
5. Make sure that the changes will persist on Web Server after reboot:

   ```
   sudo vi /etc/fstab
   ```

6. Add the following line:

   ```
   <NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
   ```

7. Install Remi's repository, Apache and PHP

   ```
   sudo yum install httpd -y
   sudo systemctl enable httpd.service
   sudo systemctl start httpd.service
   sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
   sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
   sudo dnf module reset php
   sudo dnf module enable php:remi-7.4
   sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   setsebool -P httpd_execmem 1
   ```

8. Repeat steps 1-5 for another 2 Web Servers.

9. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files - it means NFS is mounted correctly. `cd /var/www`

10. You can try to create a new file.

    ```
    sudo touch test.txt
    ```
    
11. Fork the tooling source code from `Darey.io` Github Account to your Github account. Download git.

    ```
    sudo yum install git -y
    ```

12. Clone the repository you forked the project into

    ```
    sudo git clone https://github.com/StegTechHub/tooling.git
    ```


13. Configure Application Database and Admin User

Now we configure the application database and create an initial admin user.

Apply Database Schema (tooling-db.sql)
	Access the tooling-db.sql file from the cloned repository on the Web Server and configure the credentials to that of the database.
	Connect to the MySQL server from the Web Server and apply the schema:
   
```   
mysql -h <DB-Private-IP-Address> -u webaccess -p tooling < tooling-db.sql
```

This command creates all required tables for the tooling application.

 Create an Admin User in MySQL

Log in to the database:

```
mysql -h <DB-Private-IP-Address> -u webaccess -p tooling
```
Insert the admin user without specifying the id (to avoid duplicate key errors):

```
INSERT INTO users (username, password, email, user_type, status)
VALUES (
  'myuser',
  'mypassword',
  'user@mail.com',
  'admin',
  1
);
```

Verify the user was created:

```
SELECT id, username, user_type, status FROM users;
```
```
Exit MySQL:
```
```
EXIT;
```

15. Update Application Database Configuration

Edit the database connection file on the Web Server:

sudo vi /var/www/html/functions.php

Ensure the database credentials match what you created:

$db = mysqli_connect(
    '<DB-Private-IP-Address>',
    'webaccess',
    'mypass',
    'tooling'
);

Save and exit.

⸻

16. Configure Security Groups for MySQL Access

On the MySQL EC2 Security Group, add an inbound rule:

Type	Protocol	Port	Source
MySQL/Aurora	TCP	3306	Web Server Security Group or Web Server Private IP

This allows the Web Servers to connect securely to the database.


17. Access the Tooling Website

Open a browser and navigate to:

http://<Web-Server-Public-IP-Address>/index.php

Log in using:
	•	Username: myuser
	•	Password: password

✅ You should now have full admin access to the DevOps Tooling Website.

18.   Open Website in Browser: Open the website using `http://<Web-Server-Public-IP-Address>/index.php` and ensure you can log in with the credentials you created.

<img width="2424" height="1330" alt="image" src="https://github.com/user-attachments/assets/bc736b50-f2e0-4998-ace7-5c3169ae07ef" />





