# WEB-SOLUTION-WITH-WORDPRESS

# PROJECT 6 - WEB SOLUTION WITH WORDPRESS

## WEB SOLUTION WITH WORDPRESS

In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. 
WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

This Project consists of two parts:

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.

2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.


### Three-tier Architecture
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

![image](https://user-images.githubusercontent.com/88560609/198341981-7810eae4-e3ea-411e-9cf4-894b68faaba7.png)

1. **Presentation Layer (PL)**: This is the user interface such as the client server or browser on your laptop.
2. **Business Layer (BL)**: This is the backend program that implements business logic. Application or Webserver
3. **Data Access or Management Layer (DAL)**: This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server
In this project, you will have the hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on
the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.


#### Your 3-Tier Setup
1. A Laptop or PC to serve as a client
2. An EC2 Linux Server as a web server (This is where you will install WordPress)
3. An EC2 Linux server as a database (DB) server
Use RedHat OS for this project and set the security group to all traffic (not recommend for advanced or real projects)

![image](https://user-images.githubusercontent.com/88560609/198509636-74bf7b1a-195d-4c2b-9efd-163477d389a4.png)

#### Step 1 — Prepare a Web Server

1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

![image](https://user-images.githubusercontent.com/88560609/198510188-6e176967-0820-451b-9915-6743e9388c53.png)

2. Attach all three volumes one by one to your Web Server EC2 instance.
    select each volume, go to action and select attach volume, select the webserver instance and save

Open up the Linux terminal to begin configuration

3. Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. 
All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices
there – their names will likely be xvdf, xvdh, xvdg.

```
lsblk 
```

<img width="413" alt="image" src="https://user-images.githubusercontent.com/88560609/198510423-bd5d19b7-da86-4551-8906-16503129d95b.png">


4. Use (df -h) command to see all mounts and free space on your server

5. Use gdisk utility to create a single partition on each of the 3 disks

```
sudo gdisk /dev/xvdf
```

you will see this: 

<img width="356" alt="image" src="https://user-images.githubusercontent.com/88560609/198510857-87b453a7-2c0d-4fb5-b6a3-844527a0102e.png">


<img width="490" alt="image" src="https://user-images.githubusercontent.com/88560609/198511033-520722cc-e8a9-412b-91ab-0ccf93bd2c6d.png">

```
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): p
Disk /dev/xvdf: 20971520 sectors, 10.0 GiB
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): B505F4F7-5D34-4BE7-A6c7-DFCA67AA55D2
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        20971486   10.0 GiB    8E00  Linux LVM

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): yes
OK; writing new GUID partition table (GPT) to /dev/xvdf.
The operation has completed successfully.
Now,  your changes has been configured succesfuly, exit out of the gdisk console and do the same for the remaining disks.

```
repeat same steps for all disks 

```
sudo gdisk /dev/xvdg
```

```
sudo gdisk /dev/xvdh
```

Use lsblk utility to view the newly configured partition on each of the 3 disks.

<img width="392" alt="image" src="https://user-images.githubusercontent.com/88560609/198511635-e127cdf4-9051-48fc-9acb-1229e584af10.png">



6. Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.


```
sudo yum install lvm2 -y
sudo lvmdiskscan 
```

7. Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

```
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
```

8. Verify that your Physical volume has been created successfully by running sudo pvs

```
sudo pvs
```

<img width="454" alt="image" src="https://user-images.githubusercontent.com/88560609/198512091-93b1014f-f17b-4476-af90-1ac16e87d28d.png">


9. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

```
sudo vgcreate vg-webdata /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```

10. Verify that your VG has been created successfully by running

```
sudo vgs
```

<img width="490" alt="image" src="https://user-images.githubusercontent.com/88560609/198512290-37d6706c-c6b5-4e63-a731-cbd8814965d7.png">


11. Use lvcreate utility to create 2 logical volumes. app-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
NOTE: app-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.


```
sudo lvcreate -n app-lv -L 14G vg-webdata
sudo lvcreate -n logs-lv -L 14G vg-webdata
```

12. Verify that your Logical Volume has been created successfully by running 

```
sudo lvs
```

<img width="492" alt="image" src="https://user-images.githubusercontent.com/88560609/198512476-20250f68-b1c7-4d2b-9055-444ef092bc6d.png">


13. Verify the entire setup

```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk
```

14. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
sudo mkfs.ext4 /dev/vg-webdata-vg/app-lv
sudo mkfs.ext4 /dev/vg-webdata-vg/logs-lv
```

15. Create /var/www/html directory to store website files


```
sudo mkdir -p /var/www/html
```

16. Create /home/recovery/logs to store backup of log data

```
sudo mkdir -p /home/recovery/logs
```

17. Mount /var/www/html on app-lv logical volume

```
sudo mount /dev/vg-webdata/app-lv /var/www/html/
```

18. Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

```
sudo rsync -av /var/log/. /home/recovery/logs/
```

19. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

```
sudo mount /dev/vg-webdata/logs-lv /var/log
```

20. Restore log files back into /var/log directory

```
sudo rsync -av /home/recovery/logs/. /var/log
```


21. Update /etc/fstab file so that the mount configuration will persist after restart of the server.

  
#### UPDATE THE `/ETC/FSTAB` FILE

The UUID of the device will be used to update the /etc/fstab file;

```
sudo blkid
```

<img width="737" alt="image" src="https://user-images.githubusercontent.com/88560609/198514178-b10f55a1-bc47-4b7c-8af6-da0605d6e0b5.png">


```
sudo vi /etc/fstab
```

##### Step 1 - Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

<img width="488" alt="image" src="https://user-images.githubusercontent.com/88560609/198514315-adf7c371-a837-4a6a-b561-92b658f6f88a.png">

1. Test the configuration and reload the daemon

```
sudo mount -a
sudo systemctl daemon-reload
```

2. Verify your setup by running df -h, output must look like this:

```
df -h
```

<img width="466" alt="image" src="https://user-images.githubusercontent.com/88560609/198514410-8bab0a95-7cd8-41d2-8f49-5236288abda0.png">


##### Step 2 — Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of app-lv create db-lv and mount it to /db directory instead of /var/www/html/.


##### Step 3 — Install WordPress on your Web Server EC2
1. Update the repository

```
sudo yum -y update
```

2. Install wget, Apache and it’s dependencies

```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```


3. Start Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd
```


4. To install PHP and it’s dependencies

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

https://www.tecmint.com/install-php-8-on-centos/


5. Restart Apache

```
sudo systemctl restart httpd
```


6. Download wordpress and copy wordpress to var/www/html

```
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp -R wordpress/wp-config-sample.php wordpress/wp-config.php
cd wordpress
sudo cp -R wordpress/. /var/www/html/  
```
  
  
7. Configure SELinux Policies

```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

##### Step 4 — Install MySQL on your DB Server EC2

```
sudo yum update
sudo yum install mysql-server -y
```

Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

```
sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo systemctl status mysqld
```


##### Step 5 — Configure DB to work with WordPress

```
sudo mysql
CREATE DATABASE wordpress;

CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY 'wordpress';
GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

SHOW DATABASES;
exit
```

<img width="425" alt="image" src="https://user-images.githubusercontent.com/88560609/198516904-57fe4243-9d89-407d-9f50-090a733f7dc2.png">


<img width="424" alt="image" src="https://user-images.githubusercontent.com/88560609/198516971-c794fa3d-920d-4ae2-9ffb-c004c1368a5e.png">


##### Step 6 — Configure WordPress to connect to remote database.
Set this up if you did not allow all traffic in your security group
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32


1. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)
```
sudo vi /etc/my.cnf
```

<img width="413" alt="image" src="https://user-images.githubusercontent.com/88560609/198517696-add85923-ad8e-4e86-9c07-7968b9a0a679.png">


From the web server 
```
sudo vi wp-config.php
```
update the DB_NAME, DB_USER DB_PASSWORD and DB_HOST

<img width="509" alt="image" src="https://user-images.githubusercontent.com/88560609/198518553-3e0989c3-f8be-449d-86eb-ed1b79ba57b6.png">

```
sudo systemctl restart httpd
```

2. Change permissions and configuration so Apache could use WordPress on the webserver:

https://www.thegeekdiary.com/how-to-disable-the-default-apache-welcome-page-in-centos-rhel-7/#:~:text=Method%201%20%3A%20removing%2Frenaming%20Welcome%20Page&text=In%20order%20to%20disable%20this,you%20don't%20need%20it.

```
sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
```

3. Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

```
sudo yum install mysql
sudo mysql -u wordpress -p -h <DB-Server-Private-IP-address>
```

4. Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.
```
show databases;
```
exit mysql

<img width="517" alt="image" src="https://user-images.githubusercontent.com/88560609/198519611-900bfd53-598b-4359-894d-1131b4eee5ad.png">

```  
select user, host from mysql.user;
``` 
```
sudo chown -R apache:apache /var/www/html/
```

<img width="423" alt="image" src="https://user-images.githubusercontent.com/88560609/198520352-d131dfbb-11d7-4cad-a516-3c7c6085c26d.png">

```
sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R
```
```
sudo setsebool -P httpd_can_network_connect=1
```
```
sudo setsebool -P httpd_can_network_connect_db 1
```

<img width="500" alt="image" src="https://user-images.githubusercontent.com/88560609/198520492-814a3c17-edaa-4f66-8d29-c85d5cd38038.png">

5. Try to access from your browser the link to your WordPress
http://<Web-Server-Public-IP-Address>/wordpress/

you see this:
    
<img width="770" alt="image" src="https://user-images.githubusercontent.com/88560609/198668339-70e305ca-349c-4cb3-96a1-b556dbe04ca6.png">


Fill out your DB credentials and click on `install wordpress`:


<img width="764" alt="image" src="https://user-images.githubusercontent.com/88560609/198669480-38446780-35c9-4673-95c1-2b0a17cb56dc.png">


If you see this message – it means your WordPress has successfully connected to your remote MySQL database


<img width="713" alt="image" src="https://user-images.githubusercontent.com/88560609/198670008-6aa985ab-bb53-42e5-9a58-30468d41e4c6.png">

<img width="793" alt="image" src="https://user-images.githubusercontent.com/88560609/198670965-659708ba-183d-4413-99c2-5d3d28cb1de8.png">


Important: Do not forget to STOP your EC2 instances after completion of the project to avoid extra costs.

You have learned how to configure Linux storage susbystem and have also deployed a full-scale Web Solution using WordPress CMS and MySQL RDBMS!
