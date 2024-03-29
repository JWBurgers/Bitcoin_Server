# Chapter 4. Remote server usage

We will generally access our server remotely. To set up remote access in a convenient and secure way, we will need to enable SSH connections, create a static IP address for the server, and secure the server and our general network. In addition, we will need to configure a static IP address for our managing computer and install an application called **PuTTy** on it.   

Note that we will disable IPv6 and set a static IPv4 address for our server. If you want to configure your server via IPv6, you need to figure it out on your own.


## Enable SSH connections

*Secure shell* (SSH) is a protocol which enables you to operate a computer remotely. 

SSH should have been installed as a Linux service during your Debian installation. To check that it's indeed running, open a terminal and execute **`$ sudo systemctl status ssh`**. If not, you need to install SSH with the package manager (**`sudo apt install open-ssh`**) and enable it as a service (**`sudo systemctl enable ssh`**). You can now execute **`$ sudo systemctl status ssh`** again and see it running. 


## Create a static IP address for your server

To connect to your Bitcoin server with the SSH protocol from a remote computer, you will need the local IPv4 address of the server. By default, the local IPv4 address of your active network interface card is dynamically managed by your router’s **DHCP server**. That means the card’s IP address can change over time. Remote access will be much easier if you assign the appropriate network card a permanent local IP address. 

There are two approaches available. First, you can instruct your DHCP server to reserve a local IP address on the basis of the card’s MAC address. Second, you can manually configure a **static IP address** for it. From a security standpoint, a static IP address is probably better. You just need to ensure that the static IP address you select is ***not*** within the range of IP addresses available to the router's DHCP server. Otherwise, two devices may attempt to communicate under the same IP address in your local network. That will cause problems.

An easy method to avoid address conflicts is to limit the range of available addresses for your router's DHCP server, and then select a static IP address for your server’s network card accordingly. Suppose, for example, that your local network address is 192.168.2.0 and that the address for your router (and DHCP server) is 192.168.2.254. Your network broadcast address will, then, be 192.168.2.255. (This is a typical local addressing space on a router. You may also have an addressing space that starts with a "10" or "172". Typically you will have 256 local addresses available.) 

In this case, you could limit the range of your DHCP server from, for instance, 192.168.2.1 to 192.168.2.50, and then set a static IP address for your server at any valid local IP address outside this range, such as 192.168.2.150. As 50 addresses is more than enough for a typical home network, this restriction for the router's DHCP server should not lead to any real problems. You can always allow a wider range if necessary.  

Let's now assign your Bitcoin server’s network card a static IP address. Start with changing the DHCP range of available addresses. Each router will differ in the details, but broadly the steps are as follows:  

* Log into your router. If you have never logged in before, you should be able to find the relevant instructions for your model online or obtain them from your Internet service provider. 
* Determine the range of acceptable IPv4 addresses for your router's DHCP server. Typically, you will have 253 addresses at your disposal. The addresses ending in 0 and 255 are usually your local network address and broadcast address respectively. Your router’s local IP address normally ends with 1 or 254. The other addresses you can normally use.  
* Choose a particular static IPv4 address that falls within the range for your home network, but outside any range you need for the DHCP server. Make the address easy to remember. ***Write down this information on your server record.*** We will use this information shortly. We will assume it is "192.168.2.150" for the instructions. If you have this IP address available given your local network circumstances, it would be convenient to use this setting. 
* Find the settings for your router's DHCP server. Set the IP range so that the static IP address for your Bitcoin server’s network card does ***not*** fall within this range. 
* Save the settings and log out of your router.

Next, you can set the static IP address for your server’s network card. We will do this from the desktop environment for convenience. Proceed as follows:

* Click on **`Activities`**. Search for and launch **`Settings`**. 
* Given that we recommended an ethernet connection, you should see this listed as your active interface. On our system it was identified as **`enp1s0`**. The "en" stands for "ethernet". The "p1" here stands for a bus number. The "s0" stands for a slot number. You should see a similar name (unless you are using a WiFi card). Select "+" to add a connection profile for your ethernet card. 
* Under the **`Identity`** tab give it the name **`Static profile`**. 
* Move to the IPv6 tab and select **`Disable`**.
* On the IPv4 tab, select **`Manual`**. How you fill this in will depend on your local network circumstances. But using the example values from earlier, it would be as follows:
    - Address: 192.168.2.150 (your desired local IPv4 address for the system)
    - Netmask: 255.255.255.0 (standard value, typical for most home routers)
    - Gateway: 192.168.2.254 (this is your router's local address)
    - DNS: 192.168.2.254 (typically, the same as the router's local address)
    - Routes: "Automatic"
* You can run the **`Static profile`** by double-clicking on it. You can leave the original, dynamic profile in place for convenience. If you ever have connection issues, you can revert back to it. However, do make sure you turn off the **`Connect automatically`** setting under the **`Details`** tab of this fallback connection profile. This helps avoid using the wrong connection profile unintentionally.  
* Open the terminal and execute **`$ ip a`**. You should see various network interfaces. The **`lo`** interface refers to the **loopback address**. You may have several other network interfaces listed. Only one interface, however, should be without the **`NO-CARRIER`** designation: the Ethernet interface you just configured. You should see the local IPv4 address you just set (e.g., "192.168.2.150"). Your ethernet card now has a static IP address assigned to it. 

As a final check, carry out the following instructions:

* **`$ cd /etc`**
* **`$ sudo cat resolve.conf`**

Make sure that you see the right nameserver listed here (192.168.2.254 in our example above). 


## Configuring your SSH server

Enabling SSH on your Bitcoin server introduces new security risks. You can take some further security measures to protect yourself. We will start by configuring your SSH server. 

To begin, you will want to also set a static IP address for your managing computer. This allows you to restrict access to the server via SSH in a much better way. Proceed as follows on your managing computer:

* Click on the Windows start button
* Select **`PC Settings`**
* Select **`Network and internet`**
* Select **`Change adaptor options`**
* You will probably see several network interface cards listed here. Find the network card you are using, and right-click on it. 
* Select **`Properties`**
* Scroll down to the **`TCP/IPv4`** options (unless you are using IPv6), select it, and click on the properties button.
* In the dialog box, you can specify the IP address you want to use for the computer. If possible, select **`192.168.2.200`**. ***Write down this information on your server record.***.  
* Your subnet mask on a normal network will be **`255.255.255.0`**, while your default gateway is your router’s IP address. The preferred DNS server should typically also be on your router’s IP address. (This was **`192.168.2.254`** in our example from the previous section).
* Select **`Ok`** and close the dialog box.
* Restart your managing computer. 

Your managing computer now also has a static IP address. Next, we will change some of the main options in the configuration file for the SSH daemon on your Bitcoin server. Proceed as follows on your Bitcoin server:

* Open a terminal and go to the folder **`/etc/ssh`**
* Open the sshd_config file with **`$ sudo nano sshd_config`**. Do not confuse this file with the **`ssh_config`** file (without the “d” in the name). The first file is for the SSH server application on your Bitcoin server, while the other is for the client application. 
* At the end of the file, insert **`AllowUsers administrator@192.168.2.200`** (or your own user account and managing computer's static IP address) on a new line. This ensures that only your managing computer can log into the Bitcoin server remotely, specifically into the administrator account. (You really should only manage the server from one computer, but you can provide access to more machines by setting a static IP address for them and adding them to the SSH daemon configuration file on your Bitcoin server in the same way.) 
* Scroll back up and find the option **`#Port 22`**. Remove the “#” tag and change the port number to a random port. You should avoid **well-known** ports from 0 to 1,023, but really can choose many options between 1,024 and 65,535. Change it to **`Port 16222`** if you are worried about picking a port on your own. Attackers are more likely to operate under the assumption that the SSH port is at 22. 
* Find the option **`#ClientAliveInterval 0`**, remove the **`#`** tag, and change the period from **`0`** seconds to **`300`** seconds. This means any open SSH connections will automatically shut down if inactive for five minutes. 
* Find the option **`X11Forwarding yes`** (the one aligned to the left, not indented) and change it to **`X11Forwarding no`**. This disables the ability to remotely display applications from your Bitcoin server via their GUI. Allowing GUI displays remotely only introduces security risks, and you do not need this functionality.  
* Find the option **`#PermitRootLogin prohibit-password`**, remove the **`#`** tag, and change it to **`PermitRootLogin no`**. This explicitly denies any remote login via the **`root`** account. 
* Save and exit the file. 
* ***Write down the SSH port information on your server record.***

There are numerous other security measures you can take with regards to your SSH configurations. For instance, you can also disable passwords alltogether and use SSH keys to enable your authentication. Though you should now be reasonably secure, you are invited to research SSH security more on your own.


## State of your server record

At this point, you should have a fairly extensive record of data about your server. In the subsequent instructions, we will continue to assume that you have a local network at 192.168.2.0, and that you have selected as 192.168.2.150 as the static IP address of your Bitcoin server. We will also assume that your managing computer is at 192.168.2.200. Finally, we will assume that your SSH server is running on port 16,222. Hence, we will assume that the details are as follows:

* User name of the administrator account administrator: administrator
* Regular name belonging to the server administrator: Server Administrator
* Password to the server administrator's account: *******
* Password to the root account: *******
* Host name of your computer: btc-server
* Directory for your bitcoin applications and data: /btc-server
* Static IPv4 address of your server: 192.168.2.150
* Static IPv4 address of your managing computer: 192.168.2.200
* SSH server port: 16022

If you any of your values diverge from those above, ***you need to continue to adapt the subsequent instructions accordingly***.


## Setting up a firewall

Next to configuring the SSH daemon application, there are various other measures you can take to improve the security of your Bitcoin server now that SSH connections are enabled.

To start, you should set up a firewall on your server. Even if you already have a hardware firewall through your router, it is still recommended that you also use a software firewall on your system. 

The **ufw application** (“uncomplicated firewall” application) for Linux systems is relatively easy to use. It actually provides a more convenient command line interface to the **iptables application**. To start, we will install the ufw application and enable it as a Linux service.

* Open a terminal window on your server
* Execute **`$ sudo apt install ufw`**
* After installation, executive **`$ sudo ufw status`**. It should be “inactive”.
* Enable ufw with the following command: **`$ sudo ufw enable`**.

We will explain **Linux services** in more detail at a later point. For now, just understand that it means the ufw application will start running automatically in the background each time you start the server. If you now run the **`$ sudo ufw status`** instructions, all you should see is a message that the firewall is **`active`**. 

By default, the ufw application typically allows all outbound connections and denies all inbound connections. An **outbound connection** occurs when your Bitcoin server initiates a communication session with a remote computer. An **inbound connection** occurs when a remote computer initiates a communication session with your Bitcoin server. 

This setting is a good starting point for your configurations. To ensure the options are indeed set this way, you can execute the following two instructions:

* **`$ sudo ufw default allow outgoing`**
* **`$ sudo ufw default deny incoming`**

Customizing outbound connections on your firewall has marginal security benefits at best. And it just creates extra hassle. So the default of allowing all outbound connections is fine. Just do not use your Bitcoin server for browsing the Web. 

With regards to inbound connections, however, we will want to allow SSH connections on the custom port we specified earlier, as long as the connection is made from the IP address of our managing computer. So we need to make an exception. (As we move along, you will need to make additional exceptions for certain software applications.) Assuming that your custom SSH port is set at 16222 and your managing computer’s static IP address is 192.168.2.200, you can make an exception with the following instruction:

* **`$ sudo ufw allow from 192.168.2.200 to any port 16222`**

All your network interfaces have a port 16222. This instruction tells ufw to allow inbound connections on any these ports, as long as they come from your managing computer. Given that we are only using a single ethernet interface, we could have technically restricted this further. But this would not really benefit us in any way. 

Before we altered the configuration file for the SSH daemon so that only connections from 192.168.2.200 to the administrator profile would be accepted. We have now further increased our security even further. The SSH daemon should now never even receive requests from other computers than the one we have specified due to the software firewall.

If at any point you have made a mistake in setting the rules for your firewall, you can easily make deletions. These works as follows:

* To delete a specific rule, first execute **`$ sudo ufw status`**. Find the number of the rule you want to delete, starting the counting from "1". To delete the specific rule number ***x***, just execute **`$ sudo ufw delete x`**. 
* To delete all rules, just execute **`$ sudo ufw reset`**. 


## Establishing a remote connection

You are now ready to connect remotely. To start, you will have to set the Bitcoin server to boot into the command line. This **command line interface mode** (CLI mode) consumes much less of your system resources. So you have a higher assurance that your Bitcoin server can handle all of its tasks properly. In addition, the CLI mode is less susceptible to system failures and security threats. You can configure the booting into CLI as follows:

* Open a terminal
* Execute **`$sudo systemctl set-default multi-user.target`**
* Execute **`$ sudo reboot`**

You should now see a log in screen on the command line. First enter **`administrator`** after the **`btc-server login: `** prompt. Then enter your **`administrator`** account password. Whenever your system now reboots, it will again restart in command line mode.  

Next, you need to install PuTTy on your managing computer. This is a client-side application that allows access to other computers using the secure shell protocol. (You could also just log into your Bictoin server from the Windows command prompt or Powershell if you wish.) Proceed as follows:

* Surf to www.putty.org.
* Click on the link provided in the section **`Download PuTTy`**.
* In the section **`Package files`** find the installers for Windows.
* You most likely have a 64-bit processor so you will need to download the 64-bit installer. Only if your managing computer is really old are you going to need the 32-bit installer.   
* After downloading the installer, execute the installer on your computer.

Logging into your Bitcoin server is now a piece of cake. Take the following steps:  

* Launch PuTTy on your managing computer.
* On the top of the screen, you should see a text box for **`Host name`**.
* Type in the static IP address for your Bitcoin server that we created earlier: 192.168.2.150.  
* Ensure that the port number is set to the SSH port number: **`16222`**. It should not be port **`22`**, the default value.  
* Ensure that the connection type is set to **`SSH`**.  
* Press **`Open`** at the bottom of the dialogue box
* A terminal window should now appear with the prompt **`login as: `**. You should enter the account **`administrator`**. 
* Enter your password for the account.
* You should now be logged into your Bitcoin server remotely.

If your screen did not show the login prompt (even after waiting a few moments), it really can only be one of three problems. First, you have not entered the right local IP address for your Bitcoin server. It could be that you mistyped the address. Second, you have not specified the correct port number. Third, either your own computer or your Bitcoin server is not online. This should help you troubleshoot any problems. 

Instead of finding your server by the static IP address, you can also use the server's **`hostname`**. In that case, you should replace **`192.168.2.150`** by **`btc-server.local`** when prompted for the **`Host name`** by PuTTy.

You are now ready to start controlling your Bitcoin server remotely. To shut down your server from the managing computer's terminal window execute **`$ sudo shutdown`**. As you have already seen earlier, you can reboot with the **`$ sudo reboot`** instruction. Take note that a full shutdown of your server will take some time. You will know that it has completed when you receive a message “Remote side unexpectedly closed network connection” on your managing computer's terminal window. 

If you ever want to make the graphical environment available again, you will have to hook up a screen, keyboard, and mouse directly to the server. You can, then, execute the following commands: 

* Execute **`$ sudo systemctl set-default graphical.target`**. 
* Execute **`$ sudo reboot`**

You will then be logged back into the graphical environment. This can sometimes be convenient for troubleshooting purposes. Remember that you cannot access your server’s GUI remotely. So you will only see the GUI if you have a screen, keyboard, and mouse directly connected to the server. 