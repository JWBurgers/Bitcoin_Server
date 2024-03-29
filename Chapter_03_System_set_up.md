# Chapter 3. System set up

In this chapter, we will take the main steps to prepare your system. We will start with an explanation for why to use Linux with your server, and then show how to configure and install the Debian operating system. We will, then, create a Bitcoin services directory, mounting any drives if necessary.

In this chapter, we will start with recording key information about your server. This really should be done on paper given that some of the information is quite sensitive. We will refer to this paper as your ***server record***. You can use the template in *Appendix A* to keep track of that information.  


## Why Linux? 

**Linux** is a family of operating systems, one that is much less centralized than the Windows family of operating systems. Popular desktop examples of **Linux distributions** are Debian, Ubuntu, Fedora, Red Hat Enterprise Linux, and CentOS. While there is often much more overlap in these operating systems, what makes them Linux distributions is that they all use the **Linux kernel**. A **kernel** here is the part of the operating system that handles its primary tasks.  

Linux distributions are widely used for servers. There are various reasons why Linux distributions, at least those with a very free and open source nature, are advantageous for usage in servers (as well as in personal or office computers). These reasons include the following:

* They better protect your privacy. You are much less likely to be violated through tracking and monitoring in a Linux distribution than in Windows or MacOS. 
* They typically offer a high degree of stability. The majority of the modern server market as well as almost all of the world’s most powerful supercomputers use Linux operating systems.
* They commonly offer a high degree of security. At least one key factor here is that the free and open source nature of many Linux distributions also ensures that vulnerabilities can be quickly spotted and fixed by a large community. You are not reliant on the actions of a single company.
* They offer more flexibility and control. Unlike for Windows, for example, automatic updates are not typically forced down your throat with Linux distributions. For a somewhat different example, there is a wide variety of choices in UX for the user. If you like the design and feel of MacOS, for instance, there are Linux distributions such as Gmac that look very similar.
* They are normally freeware, as is most of the software that you can use on them. That matters a lot, particularly if you are trying to set up simple, economical servers.

Because of the types of reasons mentioned above, many people favor Linux distributions both for servers as well as personal and office computers. We might also see a growing popularity of free and open source Linux distributions for smart phones in the coming years such as **Ubuntu Touch** and **SailfishOS**.

Liberal and democratic government—and freedom and privacy more specifically—is under threat from the misuse of technology. Government agencies have been actively building mass surveillance systems to track the activities of their own citizens. Technology companies can now track our movements to an extremely high degree of accuracy via our “smart” phones. Thousands of history books can explain to you why all of this is a terrible idea. From a security perspective, moving to Linux-based systems is a step in the right direction.


## Obtaining the Debian installation image

For all the reasons mentioned above, we will also utilize a Linux distribution for your Bitcoin server: **Debian**. This distribution is fairly conservative with regards to implementing new features. 

Let's first obtain an installation image of Debian. Proceed as follows:

* Make sure you have an empty USB stick with at least 5GB of storage space. 
* On your managing computer, travel to the official webpage for obtaining Debian: https://www.debian.org/distrib/. 
* Here you have two options for downloading a Debian installation image. Select the link for the "complete installation image".
* Click on the link for downloading the images using HTTP. 
* Click on the link that will show you the stable releases.
* Select the DVD image link that matches the architecture of the processor on your server (typically AMD64 or ARM64). If you are not sure, you can easily Google the architectural information about your processor. 
* Download the ISO file onto your managing computer. If more than one ISO file is available, only download the first in the series (i.e., "debian-something-DVD-1.iso").
* Leave the web page for downloading the installation image open for now.

At this point, you should verify the authenticity of the image. Specifically, you will want to ensure that the image was not corrupted during transfer or that you have obtained a malicious image. You can use the hashes files and signature files on the Debian download web page for the verification process.

If you have never verified files in this manner, you really need to get into the habit. Free and open source applications are often not verified automatically by your system upon installation. And without manually verifying the authenticity of such applications can create security risks. To verify the authenticity of the Debian installation image on Windows, you need to take the following steps:  

* If it is not yet on your system, install GPG4win with Kleopatra (see [https://gpg4win.org/download.html](https://gpg4win.org/download.html)). If it is not yet on your system, install the MD5 & SHA checksum utility (see [https://download.cnet.com/md5-sha-checksum-utility/3000-2092_4-10911445.html](https://download.cnet.com/md5-sha-checksum-utility/3000-2092_4-10911445.html)). 
* Create your own private-public keypair using Kleopatra. You will use this keypair to validate imported public keys. 
* Find the latest public key associated with the release signatures for Debian installation images. You can find it on the official website ([https://www.debian.org/DVD/verify](https://www.debian.org/DVD/verify)). 
* Verify the legitimacy of the public key against other sources such as the entry on the Ubuntu key server.
* If the public key has been verified, import it onto your GPG keyring using Kleopatra.
* From the Debian installation image download page, download either the SHA-256 or SHA-512 hashes and signature files.
* Find the SHA-256 or SHA-512 hash of your installation image in the hashes file. Use the checksum utility to confirm the hash value in the hashes file matches that of the installation image on your managing computer. 
* Verify with Kleopatra that the signature file contains a valid signature over the hashes file. 

Writing out the exact details of these steps would consume a lot of space and we will not do it here. If this is your first time verifying software in this manner, you should look for resources online that can walk you through a more detailed explanation of these steps. We will show in detail later how to verify the authenticity of the source code for bitcoin core and other packages. Please be aware that your first attempt at software verification will probably require quite a bit of time and troubleshooting. So be patient. 

Once you have verified the Debian installation image, you can delete the hashes file and the signature file from your managing computer and move onto the installation of Debian. 


## Debian installation

We will first create a bootable USB stick using the Debian installation image. Proceed as follows:

* Download a utility called Rufus (https://rufus.ie/en/#).
* Plug the USB stick into your managing computer.
* Execute Rufus as ***an administrator***. Fill in the options as follows:
    - **`"Device"`**: Select your USB stick
    - **`"Boot selection"`**: Set to **`"Disk or ISO image"`**
    - **`"SELECT"`**: Click on select and choose the Debian installation image you downloaded in the previous step
    - Leave **`"Persistent partition size"`** on a value of **`0`**
    - **`"Partition Scheme"`**: Set to **`"GPT"`** (which stands for **GUID partition table**)
    - **`"Target system"`**: Set to **`"UEFI (Non CSM)"`**
    - **`"Volume label"`**: You can identify the volume as **`"Debian 12.2.0 amd64"`** (replace **`"amd64"`** with the right abbreviation for a different architecture)
    - **`"File system"`**: Leave on **`"FAT32"`** (it should be the default)
    - **`"Cluster size"`**: Leave on the default (it is probably **`"16KB"`** or **`"32KB"`**)
    - Press **`"Start"`**
    - Select **`"Write in DD image mode"`** (the default **`"Write in ISO image mode"`** has been known to change the ISO image)
    - After the process completes, click on **`"Close"`**
    
Once you have finished the bootable USB, plug it into your Bitcoin server. Make sure your Bitcoin server is connected to the Internet via an ethernet cable and turn on your server with a screen, mouse, and keyboard attached. You should now be guided through the setup for Debian. Below are suggestions for the setup choices that we were offered installing ***Debian Bookworm 12.2.0***. Different releases should offer similar choices, so you can use the suggestions below as a general guide. The suggestions are mostly recommendations, you are free to make adaptations as you see fit. 

* Choose the **`"Graphical install"`** option.
* Set **`"English"`** as your preferred language.
* Provide your location information.
* Select your preferred keyboard layout.
* If you are asked about your network connection, choose your Ethernet interface (not the Wifi interface).
* For your hostname, you can choose **`"btc-server"`**. If you are very concerned about privacy and security, you can also choose a less distinct or even misleading name (e.g., **`"my_server"`**, **`"rainbow"`**). Make sure you only use lowercase letters, the numbers 0 through 9, and the hyphen (i.e., "-") in the new host name. ***Write down this information on your server record.***
* The domain name you can just leave blank.
* At this point, you have to choose a root user password. Your **root account** is an account that will allow you to access and manipulate any files on your system. If someone obtains access to your root account, they can mess with anything. So a good option for the password is to pick a short, but highly unusual sentence or phrase that you can remember. ***Write down this information on your server record.*** 
* Set the real name of your user account to **`"Server Administrator"`**. ***Write down this information on your server record.***
* Set the username of your user account to **`"administrator"`** (it is also common to use "admin", but this is not allowed in Debian). ***Write down this information on your server record.***
* Choose a password for your user account that is different from the root user password. Again, a good option is to pick a short, but highly unusual sentence or phrase that you can remember. ***Write down this information on your server record.***
* Select **`"Use an entire disk"`** for the installation of the Debian operating system.
* Choose which disk you want to use for the Debian operating system.
* It is recommended to select **`"All files in one partition"`**.
* Confirm the installation by selecting **`"Finish partitioning and write changes to disk"`**.
* Select **`"Yes"`** when asked **`"Write changes to disks?"`**
* Select **`"Yes"`** when asked **`"Use a network mirror?"`**
* Choose your location to determine the mirror.
* Select **`"deb.debian.org"`** if available; otherwise just try another mirror.
* You can leave the HTTP proxy information blank.
* For privacy, it is strongly suggested to say **`"No"`** when asked **`"Participate in the package usage survey?"`**
* You need to install the **`"SSH server"`** and **`"Standard system utilities"`**. You will use the SSH server application to remotely control your Bitcoin server from your managing computer. Given that you will manage your server remotely through a command line interface, you arguablly do not really need the desktop environment. You could only work with a **headless** version of Debian: that is, a version of Debian which only provides a command line environment. Nevertheless, especially if you are a novice to servers, having the option to boot into the desktop environment can be very helpful, particularly for troubleshooting purposes. Hence, we would strongly recommend you also select **`"Debian Desktop Environment"`** and **`"GNOME"`**.
* After the installation, reboot your computer.
* On startup, you will be asked several more questions: confirm the keyboard settings; skip the Wifi settings; turn off the location services; and skip connecting your online accounts. 


## Add sudo and obtain updates 

We will now check whether there are any further updates for your operating system. 

To start, launch the terminal application. You should automatically be logged into your **`“administrator”`** account. This account only has limited privileges to make changes to your system. Going forward, you will want to be able to precede any administrator command with the **`$ sudo`** modifier to invoke privileges for the root group. To set this up, first enter into your root account. 

* Execute **`$ su root`** and enter your password. The **`su`** command stands for “switch user”.  

Next, you will need to assure that the **`usermod`** utility is available from anywhere on the command line. Proceed as follows:

* Execute **`$ echo $PATH`** to view which directory paths are included in the PATH variable. 
* Probably, the PATH variable does not include **`/usr/sbin`**. This directory includes the **`usermod`** utility as well as other utilities that are often needed. To add **`/usr/sbin`** to your path variable, move into the **`/home/administrator`** directory and open the **`.profile`** file. Add the following line to the end of the file: **`$ export PATH=$PATH:/usr/sbin`**. Save and exit the file.

Next, you will need to ensure that the **`administrator`** user account is added to the **`sudo`** group. Proceed as follows:

* Check the groups for the account using **`$ groups administrator`**. The first group listed will be **`administrator`**, the **primary group**, while all the **secondary groups** follow it. (Any user in Linux is by default part of a primary group with the same name.) The list of secondary groups should look something like the following: **`administrator cdrom floppy audio dip video plugdev users netdev bluetooth lpadmin scanner`**. 
* In order to add **`sudo`** to the list of secondary groups, execute **`$ usermod -a -G sudo administrator`**. 

You can now log back into your administrator account by executing **`su administrator`**. If you now execute **`$ echo $PATH`**, you will see the **`/usr/sbin` directory also added to the **`PATH`** variable. If you execute **`$ groups administrator`**, you should see **`sudo`** in the list of secondary groups.

To ensure that all updates are grabbed from online repositories, not any of the installation images, proceed as follows:

* Enter the **`/etc/apt`** directory.
* Open the **`sources.list`** file with the following command: **`$ sudo nano sources.list`**.
* You should ensure any line that mentions a "CD-ROM" is commented out using a pound sign (i.e., “#”). 

Finally, we can verify that our installation contains all the latest stable versions of our packages and kernel. Proceed as follows:

* Execute **`$ sudo apt full-upgrade`**
* Enter your password
* Install any remaining available updates

The full-upgrade command will also delete existing packages on your system if needed for the upgrade. You will only want to execute this command in case of a major upgrade. Typically, you should execute **`$ apt update`** and **`$ apt upgrade`**. 


## Creating the Bitcoin services directory (option 1)

If you plan to have a Bitcoin services directory on a different drive than your operating system, ***you have to skip this section and move onto the "Unmounted drives" section below***. If you will install your Bitcoin services directory on the same drive as your operating system, ***you need to complete this section and then skip to the "Server record" section below***. 

To have your Bitcoin services directory on the same drive as your operating system, you merely need to create a directory. You are free to choose where to create the directory, and not really bound by Linux directory conventions given that this is a personal server. We recommend to place it just within the root directory. Proceed as follows:

* Move into the root directory by executing **`$ cd /`**
* Execute **`$ sudo mkdir btc-server`**

As you created this directory with **`sudo`**, the permissions will by default not allow the **`administrator`** user to write data to it. You can see the ownership of the directory and the permissions with the following instruction:

* **`$ ls -l`**

The first column of the output indicates the read, write, and execution rights of the user owner, the group owner, and others within the directory. Next are listed the user and group owners. You should have **`root`** for each owner. To allow the **`administrator`** to write to this directory, now execute the following command from the root directory: 

* **`$ sudo chown -R administrator btc-server`**

This instructs the operating system to change the user owner of the **`/btc-server`** directory and all its contents to the **`administrator`**. If you now execute **`$ ls -l`**, you should see the **`administrator`** as the user owner.

It makes sense to also change the group owner in the same way. Execute the following: 

* **`$ sudo chgrp -R administrator btc-server`**

All the permissions for the account owner, group owner, and everybody else can be set as follows: 

* **`$ sudo chmod -R 711 btc-server`**

This instruction basically ensures that only the **`administrator`** and **`root`** user can make changes to the directory. Any other account can only look inside the directory and execute applications, not read or write any content. 


## Unmounted drives (option 2)

If you have a dedicated drive for your Bitcoin services directory, you might have already noticed that you cannot access it. That is, you cannot actually read or write to the drive. While the Debian operating system will recognize your dedicated Bitcoin drive, you first need to create a volume on the drive and then mount it onto your system. Only then can you read and write to the drive.  

To start, make sure your system has recognized your dedicated drive. Execute the following command:

	**`$ lsblk`**

All the drives recognized by your system will now be listed. One drive will be for your operating system. You should see mount points in the **`boot`** and **`root`** directories. Typically this primary device would be an HDD or SSD. It could also be a (micro) SD card if you have a single-board computer like a Raspberry Pi system. Your primary drive is listed in the **`lsblk`** output using the following conventions:  

* An NVME SSD would be identified as **`“nvme0n1”`**. The three partitions for the Debian operating system would be listed as **`“nvme0n1p1”`**, **`“nvme0n1p2”`**, and **`“nvme0n1p3”`**. Any subsequent NVME SSD would be identified by **`“nvme0n2”`**, **`“nvme0n3”`**, and so on. 
* A SATA SSD or HDD would be identified as **`“sda”`**. The three different partitions for the Debian operating system would be listed as **`“sda1”`**, **`“sda2”`**, and **`“sda3”`**. The **`“sda”`** identifier is also used for (micro) SD cards and USB drives. Any subsequent drives would be identified by **`“sdb”`**, **`“sdc”`**, and so on. 

Next to the primary drive, you should see your intended SSD or HDD for bitcoin applications and data. It should be unmounted. The naming conventions for this drive logically follow from the system explained above. Suppose, for example, that your primary drive were a SATA SSD with three partitions and your secondary drive also a, yet unmounted, SATA SSD. In that case, you should see the following:

* A primary drive identified by **`“sda”`** and three partitions: **`“sda1”`**, **`“sda2”`**, and **`“sda3”`**. 
* A second drive just identified by **`“sdb”`** (or perhaps **`“sdc”`** if you still have a USB plugged in). 

You can find all your device files, including for storage drives, in the **`/dev`** directory (i.e., “devices" directory). 

Let’s now turn to the task of mounting your drive for Bitcoin applications and data. 


## Partitioning and Formatting the drive (option 2)

We first need to create a single volume on your drive. By a **volume** here, we mean a logical partition which has a file system installed on it. (Your drive may have already have one or more logical partitions, but we suggest just to redo all of it on your own.)

We will use a graphical partition editor called **gparted** (or “Gnome Partition Editor”). To install and open the application, execute the following instructions:

* **`$ sudo apt install gparted`**
* **`$ sudo gparted`**

As a first step, we need to create a **partition table**. This is a table with metadata about your drive that the operating system will read on startup. The type of partition table we will use is known as the **GUID partition table** (“gpt”), a newer standard.  

On the gparted application window, you will probably see information about your primary drive. It will likely have a FAT32 partition, an ext4 partition, and some unallocated space. The ext4 partition has most of the storage space that is not required by the operating system. The operating system is stored on the FAT32 partition.  

In order to install the GUID partition table on your umounted drive, proceed as follows:

1. Select your dedicated Bitcoin services drive from the drop-down menu on the right. Make sure you grab the right drive. ***Definitely be careful not to select your primary drive with operating system!***
2. If there are no partitions on the drive yet, just move to step (3). If there are visible partitions, right-click on each of them and select **`unmount`**. Then, for each partition, click **`Partition`** from the menu and select **`delete`**. Be sure to click the green check mark at the top of the application window in order for the deletion operation(s) to execute. 
3. On the **`Device`** menu, choose **`Create Partition Table`**.
4. It automatically selects the **`MS DOS`** partition table, an older standard. Instead, select **`gpt`**. 
5. Click **`Ok`**. 

You really only need one volume on your drive. The best file system for it is **ext4**, which is optimal for Linux systems. Take the following steps to create the volume:

1. Make sure the right drive is still selected from the drop-down menu on the right. 
2. On the menu, click **`Partition`** and select **`New`**.
3. The information on the left of the menu pertains to the size of the intended partition. By default, it should cover all of the available space. 
4. On the right side of the menu, proceed as follows:
    - Select **`primary`** partition for the type (here not related to whether an operating system runs on it). 
    - The partition name you can leave blank.
    - The file system you want is **`ext4`**.  
    - You can put the label **`btc-server`**. 
5. Once you have addressed all the options, click **`Add`**.
6. Click the green check mark on the top of the screen to create the volume.
7. After completion of the process, reboot your system.


## Mounting the drive (option 2)

After creating the volume, we will now need to ensure your dedicated Bitcoin services drive mounts automatically when you boot your system into the command line environment. 

To start, check that your partition has been added to the **`dev`** directory. Depending on your setup, it is likely listed as **`sda1`** or **`sdb1`**, as explained in the **Unmounted drives** section above.  

Next, we have to decide on a mount point. Given that this is our own personal server, we will not abide by any Linux conventions and just create a directory for the server within our root directory. Open your terminal window and proceed with the following instructions: 

* **`$ cd /`**
* **`$ sudo mkdir btc-server`**

As we created this directory with **`sudo`**, the permissions will by default not allow the **`administrator`** user to write data to it. You can see the ownership of the directory and the permissions with the following instruction:

* **`$ ls -l`**

The first column indicates the read, write, and execution rights of the account owner, the group owner, and others within the directory. Next are listed the account and group owners. You should have **`root`** in both cases. The easiest approach to allowing the **`administrator`** to write to this directory is by moving into the root directory and executing the following command: 

* **`$ sudo chown -R administrator btc-server`**

This changes account owner of the **`/btc-server`** directory to the **`administrator`**. The option **`-R`** means that the same ownership privileges also apply to all subdirectories and files within the **`/btc-server`** branch (even though there is nothing yet inside the directory for which this would apply). If you now execute **`$ ls -l`**, you should see the **`administrator`** as the account owner.       

It makes sense to also change the group owner in the same way. Proceed as follows: 

* **`$ sudo chgrp -R administrator btc-server`**

All the permissions for the account owner, group owner, and everybody else can then be set as follows: 

* **`$ sudo chmod -R 711 btc-server`**

This basically ensures that only the **`administrator`** and **`root`** accounts can make changes to the directory. Any other account can only look inside the directory and execute applications, not read or write any content. 

Next, we need to alter details about the drive in a configuration file called **`fstab`**, which stands for “file system table”. It is one of the most important configuration files in Debian and other Linux distributions. To open it, proceed as follows:

* **`$ cd /etc`**
* **`$ sudo nano fstab`**

The table configures the volumes on your system. At this point you should have only three volumes listed in the file: the three on your primary drive. To automatically boot your additional's drive's volume in command line mode, you need add the following line at the end of the fstab file, replace **`[volume]`** with either **`sda1`**, **`sdb1`**, and so on.

* `/dev/[volume] /btc-server ext4    defaults,noatime	0	0`

A tab space should be between each of the items. 

The left entry indicates the location of the volume. The second entry indicates the desired mount point. The third entry indicates the type of file system on the volume. The fourth entry lists various mount options. The label **`defaults`** puts in place a set of common mount options. By default, the access time field is updated whenever you read or write to a file or directory. The **`noatime`** removes the recording of any read event and, thus, reduces the load on your drive. The last two options pertain to backup operations and file system checks.  

After saving the fstab file, you need to close the terminal window and restart your system. If you now move to the **`/btc-server`** directory in a terminal window, you should see a **`lost + found`** directory. This indicates that your drive has mounted successfully from the configuration options you set in the fstab file. You can just leave this **`lost + found`** directory in place.   


## Server record

At this point, we have already added a lot of information to our server record. The details that you should have so far are as follows:

* User name of the administrator account: administrator
* Regular name belonging to the server administrator's user: Server Administrator
* Password to the server administrator's account: *******
* Password to the root account: *******
* Host name of your computer: btc-server
* Directory for your Bitcoin services: /btc-server