# Chapter 3. System set up

In this chapter, we will take the main steps to prepare your system. 


## Why Linux? 

Linux distributions are widely used for servers. There are various reasons why Linux distributions, at least those with a very free and open source nature, are advantageous for usage in servers (as well as in personal or office computers). These reasons include the following:

* They will better protect your privacy. You are much less likely to be violated through tracking and monitoring in a Linux distribution like Debian, for example, than in Windows or MacOS. This, of course, does not mean you no longer have to worry about privacy at all when you use a Linux distribution, only that you are better protected.
* They typically offer a high degree of stability. This at least one reason why many people favor them, particularly for servers. It is no surprise that the majority of the modern server market as well as almost all of the world’s most powerful supercomputers use Linux operating systems.
* They commonly offer a high degree of security. Their Unix origin means they are operating systems designed to be used by multiple users (Unix was created at a time that people still generally sat behind terminals to access a central computer). Windows, by contrast, was more designed to be used by particular individuals. The free and open source nature of many Linux distributions also ensures that vulnerabilities can be quickly spotted and fixed by a large community. You are not reliant on the actions of a single company.
* They offer more flexibility and control to the user in how their operating system works. Unlike for Windows 10, for example, you will not face a situation where it is seemingly impossible to stop automatic updates. This flexibility and control also ensures a wide variety of choices for the average user. For instance, if you like the design and feel of MacOS, there are Linux distributions such as Gmac that look very similar; all this was created by a community of enthusiasts who took advantage of the flexibility and control you have with most Linux distributions.
* They are normally freeware, as is most of the software that you can use on them. That matters a lot, particularly if you are trying to set up simple, economical servers.

Because of the types of reasons mentioned above, many people favor Linux distributions both for servers as well as personal and office computers. We might also see a growing popularity of free and open source Linux distributions for smart phones in the coming years such as Ubuntu Touch and SailfishOS.

Liberal and democratic government—and freedom and privacy more specifically—is under threat from the misuse of technology. Government agencies have been actively building mass surveillance systems to track the activities of their own citizens. Technology companies can now track our movements to an extremely high degree of accuracy via our “smart” phones. Thousands of history books can explain to you why all of this is a terrible idea. This is not a right or left issue. From a security perspective, moving to Linux-based systems is a step in the right direction.


## Obtaining the Debian installation image

For all the reasons mentioned above, we will also utilize a Linux distribution for the Bitcoin server: **Debian**. While the instructions provided in this guide will work similarly on other Linux distributions, you will probably have to do some tweaking and adapting when using alternatives. You are free to work with an alternative distribution if you think you can handle that challenge on your own.

Let's first obtain an installation image of Debian. Proceed as follows:

* Make sure you have a USB stick with sufficient storage space available. You will probably need around 4 to 5GB of storage space for the image. 
* On your managing computer, travel to the official webpage for obtaining Debian on a browser: https://www.debian.org/distrib/. 
* Here you have two options for downloading a Debian installation image: the small or complete version. Select the link for the "complete installation image".
* Click on the link for downloading the images using HTTP. 
* Click on the link that will show you the stable releases.
* Select the DVD image link that matches the architecture of the processor on your server (typically AMD64 or ARM64). If you are not sure, you can easily Google the architectural information about your processor. 
* Download the ISO file onto your managing computer. If there is more than one ISO file available, you only need to download the first in the series (i.e., "debian-something-DVD-1.iso").
* Leave the web page for downloading the installation image open for now.

At this point, you will want to verify the authenticity of the image. Specifically, you will want to ensure that the image was not corrupted during transfer or that you have obtained a malicious image. You can use the hashes files and signature files on this web page for the verification process.

If you have never verified files in this manner, you really need to get into the habit. Free and open source applications are often not verified automatically by your system upon installation. Not verifying the authenticity of your applications can create security risks. To verify the authenticity of the Debian installation image on Windows, you need to take the following steps:  

* If it is not yet on your system, install GPG4win with Kleopatra (see https://gpg4win.org/download.html). If it is not yet on your system, install the MD5 & SHA checksum utility (see https://download.cnet.com/md5-sha-checksum-utility/3000-2092_4-10911445.html). 
* Create your own private-public keypair using Kleopatra. You will use this keypair to certify imported public keys. 
* Find the latest public key associated with the release signatures for Debian installation images. You can find it on the official website (https://www.debian.org/DVD/verify). 
* Verify the legitimacy of the public key against other sources such as the entry on the Ubuntu key server.
* If the public key has been verified, import it onto your GPG keyring using Kleopatra.
* From the Debian installation image download page, download either the SHA-256 or SHA-512 hashes and signature files.
* Find the SHA-256 or SHA-512 hash of your installation image in the hashes file. Use the checksum utility to confirm the hash value in the hashes file matches that calculated by the checksum utility of the installation image on your managing computer. 
* Verify with Kleopatra that the signature file contains a valid signature over the hashes file. 

Writing out the exact details of these steps would consume a lot of space and we will not do it here. If this is your first time verifying software in this manner, you should look for resources online that can take you through a more detailed explanation of these steps. We will show in detail later how to verify the authenticity of the source code for bitcoin core and other packages. Please be aware that your first attempt at software verification will probably require quite a bit of time and troubleshooting. 

Once you have verified the Debian installation image, you can delete the hashes file and signature file from your managing computer. 


## Debian installation

We will first create a bootable USB stick using the Debian installation image. Proceed as follows:

* Download a utility called Rufus (https://rufus.ie/en/#).
* Plug the USB stick into your managing computer.
* Execute Rufus as ***an administrator***. Fill in the options as follows:
    - "Device": Select your USB stick
    - "Boot selection": Set to "Disk or ISO image"
    - "SELECT": Click on select and choose the Debian installation image you downloaded in the previous step
    - Leave "Persistent partition size" on a value of 0
    - "Partition Scheme": Set to "GPT" (which stands for GUID partition table)
    - "Target system": Set to "UEFI (Non CSM)"
    - "Volume label": You can identify the volume as "Debian 12.2.0 amd64" (replace "amd64" with the right abbreviation for a different architecture)
    - "File system": Leave on "FAT32" (it should be the default)
    - "Cluster size": Leave on the default (it is probably "16KB" or "32KB")
    - Press "Start"
    - Select "Write in DD image mode" (the default "Write in ISO image mode" has been known to change the ISO image)
    - After the process completes, click on "Close"
    
Once you have finished creating the bootable USB, plug it into your Bitcoin server. Make sure your Bitcoin server is connected to the Internet via an ethernet cable and turn on your server. You should now be guided through the setup for Debian. Below are suggestions for the setup choices that we were offered installing Debian Bookworm 12.2.0. Different releases should offer similar choices, so you can use the suggestions below as a general guide. The suggestions are mostly recommendations, you are free to make adaptations as you see fit. 

* Choose the "Graphical install" option
* Set "English" as your preferred language
* Provide your location information
* Select your preferred keyboard layout
* If you are asked about your network connection, choose your Ethernet interface (not the Wifi interface)
* For your hostname, you can choose "btc-server" or something similar. If you are very concerned about privacy and security, you can also choose a less distinct or even misleading name (e.g., "my_server", "rainbow"). Make sure you only use lowercase letters, the numbers 0 through 9, and the “-“ sign in the new host name. 
* The domain name you can just leave blank.
* At this point, you have to choose a root user password. Your root user account is an account that will allow you to access and manipulate any files on your system. If someone obtains access to your root user profile, they can mess with anything in your system. So a good option for the password is to pick a short, but highly unusual sentence or phrase that you can remember. ***Make sure you write down the passphrase on a piece of paper.*** 
* The real name of your user account you can set to "Server Administrator". ***Make sure you write down this information.***
* The username of your user account you can set to "administrator" (it is common to use "admin", but this is not allowed in Debian). ***Make sure you write down this information.***
* Choose a password for your user account that is different from the root user password. Again, a good option for the password is to pick a short, but highly unusual sentence or phrase that you can remember. ***Make sure you write down this information.***
* Select "Use an entire disk" for the installation of the Debian operating system
* Choose which disk you want to use for the Debian operating system
* It is recommended to select "All files in one partition"
* Confirm the installation by selecting "Finish partitioning and write changes to disk"
* Select "Yes" when asked "Write changes to disks?"
* Select "Yes" when asked "Use a network mirror?"
* Choose your location
* Select "deb.debian.org" if available; otherwise just try another option
* You can leave the HTTP proxy information blank
* It is suggested to say "No" when asked "Participate in the package usage survey?"
* You need to install the SSH server and standard system utilities. You will use the SSH server application to remotely control your Bitcoin server from your managing computer. Given that you will manage your server remotely through a command line interface, you arguably do not really need the desktop environment. Arguably, you could work with a **headless** version of Debian: that is, a version which only provides a command line environment to the operating system. Nevertheless, especially if you are a novice to servers, having the option to boot into the desktop environment can be very helpful, particularly for troubleshooting purposes. Hence, we would strongly recommend you also select "Debian Desktop Environment" and "GNOME".
* After the installation, reboot your computer
* On startup, you will be asked several more questions: confirm the keyboard settings; skip the Wifi settings; turn off the location services; and skip connecting your online accounts. 


## Add sudo and obtain updates 

We will now check whether there are any further updates for your operating system. 

To start, launch the terminal application. You should be automatically logged into your “administrator” account. 

This account only has limited privileges to make changes to your system. Going forward, you will want to be able to precede any command with the **`$sudo`** modifier to invoke administrative privileges. To set this up, first enter into your root account. 

* Execute **`$ su root`** and enter your password. The “su” command stands for “switch user”.  

Next, you will need to assure that the **`usermod`** utility is directly available on the command line. Proceed as follows:

* Execute **`$ echo $PATH`** to view which directory paths are included in the PATH variable. 
* Probably, the PATH variable does not include **`/usr/sbin`**. This directory includes the “usermod” utility as well as other utilities that are often needed. To add **`/usr/sbin`** to your path variable, execute the following command: **`$ export PATH=$PATH:/usr/sbin`**. 
* If you now execute **`$ echo $PATH`**, you should see the **`/usr/sbin` directory added to the PATH variable.

Finally, you will need to ensure that the “administrator” user account is added to the “sudo” group. Proceed as follows:

* Check the groups for the “administrator” account using **`$ groups administrator`**. The first group listed will be “administrator”, the primary group, while all the secondary groups follow it. (Any user in Linux is by default part of a primary group with the same name.) The list of secondary groups for “administrator” should look something like the following: **`administrator cdrom floppy audio dip video plugdev users netdev bluetooth lpadmin scanner`**. 
* In order to add “sudo” to the list of secondary groups, execute **`$ usermod -a -G sudo administrator`**. 
* You can check that the command was executed successfully using **`$ groups administrator`**. You should now see “sudo” in the list of secondary groups.

You can now log back into your administrator account by executing **`su administrator`**. 

Next, you need to ensure that all updates are grabbed from online repositories, not any of the installation images. Proceed as follows:

* Enter the /etc/apt directory.
* Open the “sources.list” file with the following command: **`$ sudo nano sources.list`**.
* You should ensure any line that refers to a CD-ROM is commented out using “#”. 

Finally, we need to ensure that our installation indeed contains all the latest stable versions of our packages and kernel. Proceed as follows:

* Open a terminal window 
* Execute **`$ sudo apt full-upgrade`**
* Enter your password
* Install any remaining available updates

The full-upgrade command will also delete existing packages on your system if needed for the upgrade. You will only want to execute this command in case of a major upgrade. Typically, you should execute **`$ apt update`** and **`$ apt upgrade`**. 

## Creating the Bitcoin server directory

If will install your Bitcoin server applications and data on a different drive as your operating system, ***you can skip this section and move onto the next***. If you will install your Bitcoin server applications and data on the same drive as your operating system, ***you need to complete this section and then skip to the "Records" section below***. 

Installing your Bitcoin server applications and data on the same drive as your operating system, you merely need to create a directory for your server. You are free to choose where to create the directory, and not really bound by Linux directory conventions given that this is your personal server. A convenient place we recommend is just within the root directory. Proceed as follows:

* Move into the root directory
* Execute **`$ sudo mkdir btc-server`**

As you created this directory with sudo, the permissions will by default not allow the “administrator” user to write data to it. You can see the ownership of the directory and the permissions with the following instruction:

* **`$ ls -l`**

The first column indicates the read, write, and execution rights of the user owner, the group owner, and others within the directory. Next are listed the user and group owners. You should have “root” for each owner. 

The easiest approach to allowing the "administrator" to write to this directory is to execute the following command from the root directory: 

* **`$ sudo chown -R administrator /btc-server`**

This instructs the operating system to change the user owner of the /btc-server directory to “administrator”. The option -R means that the same ownership privileges also apply to all subdirectories and files within the /btc-server branch (even though there is nothing yet inside the directory for which this would apply). If you now execute **`ls -l`**, you should see the "administrator" as the user owner.  


## Unmounted drives

It is recommended that you install all your Bitcoin server applications and data on one drive for convenience. This should be either an HDD or an SSD. 

If your Bitcoin server drive is different drive than your primary drive, you might have already noticed that you cannot access it. That is, you cannot actually read or write to it. While the Debian operating system will recognize your Bitcoin server drive, you first need to create one or more volumes on the drive and then mount these volumes onto your system. Only then can you read and write to the drive. This is in contrast to USB sticks that are mounted automatically. 

To start, you should make sure that your system indeed recognized your Bitcoin server drive. Execute the following command:

	**`$ lsblk`**

All the drives recognized by your system will now be listed. One drive will be for your operating system. You should see mount points in the “boot” and root directories. Typically this primary device would be an HDD, SSD, or a (micro) SD card. The latter is sometimes the case if you have a single-board computer like a Raspberry Pi system. Your primary drive is listed in the output using the following conventions:  

* An SSD with NVME would be identified as “nvme0n1”. With the three different partitions necessary for the operating system, you would see “nvme0n1p1”, “nvme0n1p2”, and “nvme0n1p3” to identify each of them. Any subsequent NVME drive would be identified by “nvme0n2”, “nvme0n3”, and so on. 
* A SATA SSD or HDD would be identified as “sda”. With the three different partitions necessary for the operating system, you would see “sda1”, “sda2”, and “sda3” to identify each of them. The “sda” identifier is also used for (micro) SD cards and USB drives. Any subsequent drives would be identified by “sdb”, “sdc”, and so on. 

Next to the primary drive, you should see your intended SSD or HDD. It should be unmounted. The naming conventions for this drive logically follow the system explained above. Suppose, for example, that your primary drive were a SATA SSD with three partitions and your secondary drive also a, yet unmounted, SATA SSD. In that case, you should see the following:

* A primary drive identified by “sda” and three partitions: “sda1”, “sda2”, and “sda3”. 
* A second drive just identified by “sdb” (or perhaps “sdc” if you have a USB plugged in). 

You can find all your device files, including for storage drives, in the “dev” directory (i.e., “devices directory) within the root of your file system. 

Let’s now turn to the task of mounting your server drive. 


## Partitioning and Formatting the drive 

Lets first turn to the task of creating a single volume on your drive. By a **volume** here, we mean a logical partition which has a file system installed on it. (Your drive may have already come with one or more logical partitions, but I would just redo all of it on your own.)

There are various applications for partitioning and formatting in Debian. We will use a partition editor called gparted (or “Gnome Partition Editor”). To install the gparted application, execute the following command:

	**`$ sudo apt install gparted`**

To open gparted, execute the following command:

	**`$ sudo gparted`**

The graphical application will now open on your desktop. 

As a first step, we need to create a partition table on our drive. This is a table with metadata about your drive that the operating system will read on startup. The type of partition table we will use is known as the GUID partition table (“gpt”), a newer standard (GUID = globally unique identifiers). 

On the gparted application window, you will probably see information about your primary drive. It will likely have a FAT32 partition, an ext4 partition, and some unallocated space. The additional ext4 partition has most of the storage space that is not required by the operating system.  

In order to install the GUID partition table, proceed as follows:

1. Select your drive from the drop-down menu on the right. If you have multiple alternative drives, make sure you grab the right one. ***Definitely be careful not to select your primary drive with operating system!***
2. If there are any visible partitions on your selected drive, right-click on each of them and select “unmount”. Next, for each partition, click “Partition” from the menu and select “delete”. Be sure to click the green check mark at the top of the application window in order for the delete operation to execute. If there are no partitions on the SSD yet, just move to step (3).
3. On the “Device” menu, choose “Create Partition Table.”
4. It automatically selects the MS DOS partition table, an older standard. Instead, select “gpt”. 
5. Click “Ok”. 

You really only need one volume on your drive. The best file system for it is ext4, which is optimal for Linux systems. Its performance is much better than FAT32. We only have that file system for a partition on the primary drive in order to load the operating system. 

With the GUID partition table installed, we can create a single partition for the drive and format it with the ext4 file system. Take the following steps:

1. Make sure the right drive is still selected from the drop-down menu on the right. 
2. On the menu, click “Partition” and select “New.”
3. The information on the left of the menu that opens up pertains to the size of the intended partition. By default, it should cover all of the space you have available on the drive. 
4. On the right side of the menu, proceed as follows for the options:
    - Select “primary” partition for the type. 
    - The partition name you can leave blank.
    - The file system you want to create is “ext4”.  
    - You can put the label “btc-server” here. 
5. Once you have addressed all the options, click “Add.”
6. Click the green check mark on the top of the screen to create the volume.
7. After completion of the process, reboot your system.


## Mounting the drive

We will now need to ensure your drive mounts automatically when you boot your system into the command line environment. 

To start, check that your single partition has been added to the "dev" directory. Depending on your setup, it is likely listed as "sda1" or "sdb1", as explained in the **Unmounted Drives** section above.  

Next, we have to decide on a mount point. It is common practice to mount additional drives into their own subdirectories within the /mnt directory. However, given that this is our own personal server, we will not abide by this convention and instead just create a directory for the server within the root directory.  

We will start by creating the necessary directory for the drive’s mount point. Open your terminal window and proceed with the following instructions: 

* **`$ cd /`**
* **`$ sudo mkdir btc-server`**

As we created this directory with sudo, the permissions will by default not allow the “administrator” user to write data to it. You can see the ownership of the directory and the permissions with the following instruction:

* **`$ ls -l`**

The first column indicates the read, write, and execution rights of the user owner, the group owner, and others within the directory. Next are listed the user and group owners. You should have “root” for each owner. 

The easiest approach to allowing the "administrator" to write to this directory is by moving into the root directory and executing the following command: 

* **`$ sudo chown -R administrator /btc-server`**

This instructs the operating system to change the user owner of the /mnt directory to “administrator”. The option -R means that the same ownership privileges also apply to all subdirectories and files within the /btc-server branch ((even though there is nothing yet inside the directory for which this would apply). If you now execute **`ls -l`**, you should see the "administrator" as the user owner.       

Next, we need to alter details about the drive in a configuration file called “fstab”, which stands for “file system table”. It is one of the most important configuration files in Linux. To open it, proceed as follows:

* **`$ cd /etc`**
* **`$ sudo nano fstab`**

The table configures the volumes on your system. At this point you should have only three volumes listed in the file: the three on your primary drive. To automatically boot your additional's drive's volume in command line mode, you need add the following line at the end of the fstab file, replace [volume] with either sda1, sdb1, and so on.

* /dev/[volume] /btc-server ext4    defaults,noatime	0	0

A tab space should be between each of the items. 

The left entry indicates the location in your Linux system of the volume. The second entry indicates the desired mount point. The third entry indicates the type of file system on the volume. The fourth entry lists various mount options. The label “defaults” puts in place a set of common mount options. By default, the access time field is updated whenever you read or write to a file or directory. The “noatime” removes the recording of any read event and, thus, reduces the load on your drive. The last two options pertain to backup operations and file system checks.  

After saving the fstab file, you need to close the terminal window and restart your system. If you now move to /btc-server, you should see a lost + found directory. This indicates that your drive has mounted successfully from the configuration options you set in the fstab file. You can just leave this directory in place.   


## Records

At this point, we have already done a lot and you really need to start recording certain matters for convenience later. So far, the main details that would probably be convenient to record are as follows:

* The name of your server administrator's account: administrator
* The regular user name belonging to the server administrator's account: Server Administrator
* The password to the server administrator's account: *******
* The password to the root account: *******
* The host name of your computer: btc-server
* The directory of your server: /btc-server

While you could write many of these details down on a piece of paper safely, the passwords to the administrator and root accounts are sensitive information. Whether you write them down on paper, store them in a password manager, or apply some other storage method, be sure that they are recorded securely. 

As we move along, you will need to add more information to the record above. 