## Chapter 5. Further security and performance measures

Before turning to the installation of your Bitcoin server applications, we will tend to some final security and performance measures.


## Harden your home network

To start, we will have a look at your home router. You can find many resources online regarding router security. I will just provide you with some of the main points of attention here. These are not just a benefit to your Bitcoin server, but more generally for your home network. Log into your router and attempt the following changes:

1. Change the default name, or SSID (“service set identifier”), of your wireless access point. (You can also prevent this name from publicly broadcasting if you wish, but this does not add much in terms of security.)
2. Change the default password to your wireless access point. It is probably best for you to use a long random string and store it in a password manager. Unless you take other measures, this default password is the only source of encryption for your wireless traffic.  
3. Change the default password to log into your router. Again, it is probably best to use a long random string and store it in a password manager. 
4. Ensure that your home network has WPA-3 enabled (or at the very least WPA-2) with regards to wireless encryption. 
6. Set your firewall settings to the highest possible level. (If you discover that this gives substantial issues down the line, you can lower them.)
7. Update your router’s firmware to the latest version. Be careful during this process not to pull out the power, or you may **brick** your router. Router producers are not great at keeping their firmware updated, but definitely make updates if you can find them. 

An additional good security practice is not to provide network access to guests. Everyone who has your wireless access point password can basically decrypt all your wireless communications unless you take additional measures. If you do really want to let guests use your home network, many routers typically have the option to broadcast a guest network. 

These points of attention above are not magic, but they will help you improve home network security.  


## Install Fail2ban

Next, we will install Fail2ban. This application further protects your Bitcoin server now that you have set up an SSH connection. Specifically, it will temporarily ban any IP address that has had a certain number of failed login attempts. This can be particularly helpful against bots that attempt to brute force your connection. 

To install fail2ban, execute the following instructions:

* **`$ sudo apt update`**
* **`$ sudo apt upgrade`**
* **`$ sudo apt install fail2ban`**

You will need to configure Fail2ban. As the configuration files that come with the software package can be overwritten with updates, you will need to create a local configuration file first. Proceed as follows:

* Move into the /etc/fail2ban directory. The main configuration file you will need is “jail.conf”. There is also an operational configuration file called “fail2ban.conf”. 
* To create a local copies of these files, execute **`$ sudo cp jail.conf jail.local`** and **`$ sudo cp fail2ban.conf fail2ban.local`**. 

Next, you need to make some changes to the jail.local configuration file. Proceed as follows:

•	Open the jail.local file with a text editor (e.g., **`$ sudo nano jail.local`**). 
•	Scroll down a bit and find the activated “bantime” entry. Change it to “30m”. This increases the length of time any IP address is banned from 10 to 30 minutes. 
•	Set the “maxretry” parameter to “3”. This is the maximum number of times an IP address can try to log in unsuccesfully within a frame set by “findtime” (10 minutes by default).
•	Set the “destemail” parameter to one of your e-mail addresses that you commonly use. The application will send you alerts when needed. 
•	Save and exit the jail.local file.

While the configuration file of Fail2ban should work just fine for your purposes, you can also do much more with it such as blacklisting and whitelisting IP addresses. There are many good sources online if you would like to explore further.<sup>[1](#footnote1)</sup>

You will have to run Fail2ban as a Linux service. As with our firewall, it means that Fail2ban will start running automatically in the background each time you start the server. To enable the service, execute the following instruction:

* **`$ sudo systemctl enable fail2ban`**

You can check that it is working properly by executing **`$ sudo systemctl status fail2ban`**. This should ouptut that the Fail2ban service is active. You can exit the status screen by pressing “Ctrl + c” on the keyboard.

While fail2ban should just work out of the box, it may be the case that a package is missing which is causing it not to run. You should check the error messages on the systemctl status check if you are having any issues. In our installations, we ran into problems because a package called "syslog" was not installed. If you run into problems, you could see if an installation resolves the problems. Execute the following commands:

* **`$ sudo apt install syslog-ng`**
* **`$ sudo systemctl restart fail2ban`**
* **`$ sudo systemctl status fail2ban`**

If you were having issues, hopefully this package has resolved the problem. If not, you will have to perform further troubleshooting on your own.


## SD card performance measures

If you are running your operating system on an HDD or SSD, you can safely skip this last section. That said, if you have a significant amount of RAM (say 16GB or more), you might want to turn off the swap file as explained below.  

If you are running your operating system from a (micro) SD card, you need to take some measures to extend the life of your SD card, which is much more vulnerable than an HDD or SSD. There are at least two measures you can take if you are running an operating system from an SD card. 

First, you need to turn off the swap file feature if it is enabled on your system. A swap file is a location on your OS drive which can be used for primary memory if you run out of RAM. We need to turn the swap file off because it can contribute to wear on your micro SD card. On Debian, you can proceed as follows:

* Open a terminal
* Execute **`sudo swapoff -a`**
* Execute **`sudo nano /etc/fstab`**
* You will see a line in the file that refers to the swap partition. You can comment it out using a pound sign ("#") in front of the line. Save and exit the fstab file.

You can now reboot your Bitcoin server and execute **`$ free -m`**. If everything worked well, there should be 0 megabytes available for the swap file. 

Second, you will have to alter the way your operating system maintains logs. By default, it will probably log many events directly onto your micro SD card. This can significantly increase the wear on your card. If you jump into the directory /var/log, you can see all the log files your system maintains. 

Using an application called **Log2ram**, we can mount this /var/log directory into RAM and, then, write our logs only periodically to the micro SD card. Proceed as follows:

* Change into the home directory for the “administrator” profile.
* Check the web address for the log2ram git repository. It should be https://github.com/azlux/log2ram.git.
* You probably don't yet have the git application installed, so execute **`sudo apt install git`**. 
* Execute the following command to copy the repository into your home directory: **`$ git clone https://github.com/azlux/log2ram.git>`**. 
* Change into the log2ram directory you just created.
* Execute the **`$ ls -l`** instruction to see the details of files and directories. The install script (install.sh) should have execution permissions for the administrator profile. If not, you have to alter them using the chmod command.  
* Execute the install script with the following instruction: **`$ sudo ./install.sh`**. The “./” is needed in front of the file name because this directory is not included in your system’s PATH variable. After installation, you should receive a message that the application will run automatically after rebooting. 
* Open the configuration file for Log2ram using the nano text editor with the following instruction: **`$ sudo nano /etc/log2ram.conf`**. The default size of the RAM directory for your logs is 40 megabytes. Hopefully, you have at least 4GB of RAM or more (as per *Chapter 2* recommendations). You can change it to 100 megabytes. Save and close the file.
* Reboot your Bitcoin server.

After rebooting your server, you can run the instruction **`$ df -h`** to check that Log2ram is in the list, which means it is indeed working properly. The service will now automatically startup each time you start your computer. 

These two measures will help to preserve your SD card. You should search online for any other measures that might help preserve your SD card.


# Notes

<a name="footnote1">1</a>. See, e.g., Abhishek Prakash, “Secure your Linux server with Fail2ban: A beginner’s guide”, March 26 (2020), available at https://linuxhandbook.com/fail2ban-basic/.