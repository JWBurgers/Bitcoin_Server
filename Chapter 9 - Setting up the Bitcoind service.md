# Setting up the Bitcoind service

You currently know about the root and administrator accounts on your system. But it may surprise you to learn that you, in fact, have many other accounts. To see them, execute the following two instructions:

* **`$ cd /etc`**
* **`$ cat passwd`**

Each line in this file describes an account on your system. Every line has seven descriptors which are seperated by colons. The very first descriptor on each line refers to the account name. 

The accounts besides root and administrator are for Linux services. These Linux services include **system services**, which are related to the functioning of your operating system, as well as **non-system services** such as Tor. We call these accounts for Linux services specifically **service accounts**, in order to distinguish them from human **user accounts** such as admin and root.  

Many services you run on a Linux system will need to be run from a service account dedicated uniquely to it. This general practice has both security and operational benefits, though the extent of these benefits depends on how well your service account and **service file**, a configuration file for the service, are set up. 

In the previous chapter, you learned that Tor automatically creates a service file for the application. What we did not discuss at the time is that the Tor installation also automatically created an account called **debian-tor** for running the service. To see this account, execute the following instructions:

* **`$ sudo apt install htop`**
* **`$ htop`**

The last command will show you all running processes and their respective accounts. Somewhere in the list, you should see the Tor application running under the service account debian-tor. 

Now there are some nuances and exceptions to this practice of running Linux services via dedicated service accounts. Take as an example NGINX, pronounced as “engine X”, a popular web server application which also can be utilized for a variety of other purposes, including as a reverse proxy and load balancer. The NGINX service actually is setup to work (partially) with the root account. While this would a terrible idea for Bitcoind, it is safe to do for NGINX. 

In contrast to services like Tor and NGINX, Bitcoin Core does not automatically create a service with a dedicated service account for Bitcoind upon installation. So you will have to set all of this up on your own for running Bitcoind as a service. We will start in the next section with creating the service account. We will then make the necessary ownership changes for this service account in our directories and files. Finally, we will create the service file. 


## Creating a service account

You should name a service account for Bitcoind so that it is congruent with the application name. For bitcoind, you can choose something like “bitcoind”, “bitcoin”, or “bitcoin-core”. For the instructions, we will assume you will set it as “bitcoind”. To create the service account called "bitcoind", execute the following instruction:

* **`$ sudo useradd --shell /usr/sbin/nologin --system -M bitcoind`**

The **`shell`** option specifies the default shell for the account. In this case, you have pointed it to a file that will prevent logging into the bitcoind account. If you attempt a log in from your terminal window in the same way as for the administrator or root account, you will receive a message that “This account is currently not available”. This setting offer an extra layer of protection and is typical for service accounts. 

The **`system`** option in the instruction above ensures that the bitcoind account recieves both a user and a group ID that is lower than 1000. These IDs correspond to the third and fourth descriptors in the passwd file in the /etc directory (you can have a look now). 

Accounts from 1000 upwards are intended for user accounts, while those between 1 and 1000 are intended for service accounts. While there is no technical difference between service and user accounts, the numbering system helps with identification. The root account, also a user account, is unique and receives both a user and group ID of 0.  

The last option (**`-M`**) in the instruction above specifies that the new account has ***no*** home directory. This is generally recommended for service accounts. By default, Bitcoind will search for a configuration file in the home directory of the user owner. As the bitcoind service account will be the user owner, Bitcoind will by default search for a configuration file in /home/bitcoind/.bitcoin. But as bitcoind has no home directory, you will have to specify an alternative location for the configuration file in the system service file.
 
If you now look into the passwd file within the **`/etc`** directory, you should see the **`bitcoind`** service account at the end of the list. 


## Ownership changes

Any Linux account generally belongs to a **primary group** of the same name (unless you specifically allocated it to another primary group). In addition, it may belong to any number of **secondary groups**. 

Group membership is important for access to directories and files. Every Linux file and directory will have both an account owner and a group owner. And you can specify the rights of both the account and group owner separately. If an account belongs to a secondary group that is the owner of a particular file or directory, then that account will have the rights as specified for the group.

We will assign various directories to have the **`bitcoind`** account as the account owner and the **`bitcoind`** group as the group owner. But as we will generally navigate our system using the administrator account, we will need to ensure that it is added to the bitcoind group. Proceed as follows:

* **`$ sudo usermod -a -G bitcoind administrator`**

You can verify that this instruction worked by executing **`$ groups administrator`**. You should see the bitcoind group listed as one of the secondary groups for the administrator account (the primary group being the administrator group). 

Note that the **`bitcoind`** account only belongs to a primary group, namely **`bitcoind`**. We are not adding the **`bitcoind`** account to any secondary groups, as we did when creating the **`administrator`** account. This is because the former account is merely for running a service. Hence, we want to restrict what it can access as much as possible.  

By default, all the binaries for the Bitcoin Core directory will have been installed with the **`root`** account as an account owner and the **`root`** group as the group owner. In addition, permissions will have automatically been set for the **`root`** account, the **`root`** group, and everyone else. You can see all this information by executing **`$ ls -l`** in the **`btc-server`** directory. 

In order to run Bitcoind from the **`bitcoind`** service account, you need to make changes to these settings. Within the **`btc-server`** directory, first execute the following instruction:

* **`$ sudo chown -R bitcoind bitcoin-core`**

This changes the owner of the Bitcoin Core directory and all its contents from the root account to the bitcoind account. Next, you need to change the group owner of the directory and all its contents with the following instruction:

* **`$ sudo chgrp -R bitcoind bitcoin-core`**

Finally, we need to specify the permissions for the account owner, the group owner, and everyone else regarding the Bitcoin Core directory and all its contents. Linux permissions are described by three categories of rights: read (r), write (w), and execution (x) privileges. We need to set the permissions as follows:

* The account owner of the Bitcoin Core directory, the bitcoind account, should have read, write, and execution privileges to the directory. The root account automatically has the same privileges as the owner. 
* The group owner of the Bitcoin Core directory, the bitcoind group, has two members: the bitcoind service account and the administrator account. As the bitcoind service account already has all rights to the directory as an account owner, the specification of the rights for the group owner does not impact it. These rights, however, will matter to the administrator account. As you will want to be able to navigate the Bitcoin Core directory from your administrator account, you need to give the bitcoind group read and execution privileges to the directory. We will not set writing rights for the administrator account, as they are not really needed.  
* Finally, any account that is not root, administrator, or bitcoind has no real business in the Bitcoin Core directory. Hence, other accounts should have no read, write, or execution permissions. 

Read, write, and execution permissions are set in Linux using a numbering system. A 0 indicates no permissions, while a 7 indicates full permissions. Numbers in between have various intermediate meanings. The number 5 specifically indicates read and execution permissions. To specify the permissions as discussed above, you, therefore, need to execute the following instruction:

* **`$ sudo chmod -R 750 bitcoin-core`**

The 750 instruction sets the privileges for the account owner, group owner, and everyone else respectively. 

To verify all the instructions you just passed were incorporated successfully, you can execute **`$ ls -l`** again within the btc-server directory. You should now see that the Bitcoin Core directory has an account owner bitcoind and a group owner bitcoind. The permissions are specified in the initial part of the entry with the labels r (read), w (write), and x (execution) for the account owner, group owner, and everyone else, respectively (the "d" preceding the labels indicates that Bitcoin Core is a directory). 


## Creating the service file

We will not sugarcoat anything: creating your own service files for applications is a giant pain in the ass. Many beginners spend a substantial amount of time looking through the Linux manual, blog posts, and stack exchange conversations when creating their first service files. Most of all, they tend to spend it breaking things.  

If you have no experience with service files, we recommend you first become familiar with some of the basics in order to follow the discussion in this section better. There are many blog posts, articles, and videos online that can give you a good introduction.<sup>[1](#footnote1)</sup>

Once you understand some of the basics of service files, you can start by creating an empty Bitcoind service file in the **`/etc/systemd/system`** directory, where all administrator-created service files are typically placed. Proceed as follows:

* Enter into the **`/etc/systemd/system`** directory
* Execute the following instruction: **`$ sudo touch bitcoind.service>`**

The specifications we would recommend for the service file are listed below. They are based on the template offered in the Bitcoin Core repository.<sup>[2](#footnote2)</sup> We did make some minor alterations. Our main philosophy is to only put what is absolutely necessary in the service file. Anything that is not strictly necessary has been left out. 

Importantly, this service file is sensitive in its formatting (e.g., capitalization, spacing, tabs). After copying the contents below into your own Bitcoind service file, make sure your remove any excess spaces and tabs, so that everything looks exactly as below. 

Pay attention particularly to the ExecStart item when copying. All the options—**`daemon`**, **`pid`**, and **`conf`**—should all be on a single continuous line. Also note that the items in brackets—**`[Unit]`**, **`[Service]`**, and **`[Target]`**—should be highlighted in green. If not, it is likely due to excess spaces or tabs. 

**Bitcoin Core service file**

`[Unit]`
`Description=Bitcoin Daemon`
`After=network-online.target`

`[Service]`
`ExecStart=/btc-server/bitcoin-core/bin/bitcoind -daemon -pid=/run/bitcoind/bitcoind.pid -conf=/etc/bitcoind/bitcoin.conf`
`Type=forking`
`PIDFile=/run/bitcoind/bitcoind.pid`

`Restart=on-failure`
`TimeoutStartSec=infinity`
`TimeoutStopSec=600`

`PermissionsStartOnly=true`
`ExecStartPre=/bin/chgrp bitcoind /etc/bitcoind`

`User=bitcoind`
`Group=bitcoind`

`RuntimeDirectory=bitcoind`
`RuntimeDirectoryMode=0710`

`ConfigurationDirectory=bitcoind`
`ConfigurationDirectoryMode=0710`

`StateDirectory=bitcoind`
`StateDirectoryMode=0710`

`PrivateTmp=true`

`ProtectSystem=full`

`ProtectHome=true`

`NoNewPrivileges=true`

`PrivateDevices=true`

`MemoryDenyWriteExecute=true`

`[Install]`
`WantedBy=multi-user.target`

Some of the configuration options in the service file require fairly advanced knowledge of operating systems to fully understand. In general, it is helpful to realize that many of them are intended to **sandbox** the Bitcoind application: that is, to restrict the service application’s access to only those system resources needed for running successfully. This is done as a security precaution.<sup>[3](#footnote3)</sup>

Below we explain the meaning of the configuration options in the service file.  

- ***Description=Bitcoin Daemon***: This is just a description for the service.
- ***After=network-online.target***: This ensures Bitcoind will not start until your network connections are up. What that means exactly depends on your network management software. Usually, "up" means that you have a routable IP address.  
- ***ExecStart=/btc-server/bitcoin-core/bin/bitcoind***: This configuration option points to the starting process for the Bitcoind service. As you made bitcoind the owner of the bitcoin-core directory earlier, the service account should have no issues launching this process. This option was also accompanied by three flags. 
    - The **`daemon`** option instructs Bitcoind that it needs to run as a background service which can accept instructions. If you do not specify this option in either the service file or the configuration file, Bitcoind will not activate as a service. The choice on where to put the flag does not really matter.   
    - The **`pid`** option creates a PID file for Bitcoind, or a "process identification" file, in the specified location, namely **`/run/bitcoin/bitcoind.pid`**.<sup>[4](#footnote4)</sup> Without this specification, Bitcoind would attempt to create the file in the home directory for the **`bitcoind`** account, which does not actually exist. When running Bitcoind as a service, you need to specifically flag this option in the service file. The service will not start up if you merely flag it in the configuration file. 
    - By default the configuration file for Bitcoind is placed in the home directory for the **`bitcoind`** account (specifically, in the directory ~/.bitcoin). But the **`bitcoind`** service account has no home directory. So the configuration file needs an alternative location. It is typical to place it within the **`/etc`** directory. 
    - If you look at the Bitcoin Core template for the service file, it also includes a location for the data directory (i.e., a **`datadir`** option). We decided not to include this option in the service file, but in the configuration file. In any case, you would not want to copy Bitcoin Core’s template exactly, as your data directory will need to point to your **`btc-server`** directory.
- ***Type=forking***: This option means that the starting process of the service, as defined in ExecStart, will exit at some point and that the main process of the service is actually a child process of that defined by ExecStart. The service manager will consider the Bitcoind service started when the parent process exits.
- ***PIDFile=/run/bitcoind/bitcoind.pid***: Any time you specify the type of a service as **`forking`** in the service file (as we did above), it is recommended that you also specify the location for the PID file. This file contains metadata about the process on your system. 
- ***Restart=on-failure***: This option ensures that the Bitcoind service will restart on its own in case of failure.
- ***TimeoutStartSec=infinity***: This option specifies how long systemd should try to start up the service before considering the start up a failure and shutting it down. The selection of **`infinity`** here implies that this option is turned off for Bitcoind.
- ***TimeoutStopSec=600***: This option specifies how long systemd should wait for a service to stop before forcibly shutting it down. The selection of "600" indicates that systemd will wait 10 minutes on the bitcoind service to stop properly. 
- ***PermissionsStartOnly=true***: This option means that the permissions for the bitcoind service account are only applied to the ExecStart process.
- ***ExecStartPre=/bin/chgrp bitcoind /etc/bitcoin***: This process executes before ExecStart. It ensures that the configuration file, which we will place in this directory, is accessible to the users of the **`bitcoind`** group (i.e., **`bitcoind`**, `**administrator`**, and `**root`**). 
- ***User=bitcoind***: This specifies the service account for the Bitcoind application.
- ***Group=bitcoind***: This specifies the user group for the Bitcoind application.
- ***RuntimeDirectory=bitcoind***: This specifies a particular runtime directory to be created at the startup of your service. This directory will include your PID file. The directory is automatically removed when the service closes.
- ***RuntimeDirectoryMode=0710***: This specifies the permissions regarding the runtime directory **`/run/bitcoind`**. Standardly, only the **`root`** account has the ability to write anywhere in the `**/run`** directory. You are making an exception here for the **`bitcoind`** service account in **`/run/bitcoind`**, who has full permissions. Anyone in the **`bitcoind`** group has execution rights. Anyone else has no rights regarding the directory.
- ***StateDirectory=bitcoind***: This specifies a particular state directory to be created at startup of the service. This includes data that is modified by Bitcoind while running.
- ***StateDirectoryMode=0710***: This specifies the permissions regarding the state directory. The owner of the service has full permissions. Anyone in the **`bitcoind`** group has execution rights. Anyone else has no rights regarding the directory.
- ***PrivateTmp=true***: The **`/tmp`** directory is where you system stores temporary files. This option creates a unique **`/tmp`** directory for the Bitcoind service, which other users and services cannot view.
- ***ProtectSystem=full***: This option makes the **`/usr`**, **`/boot`**, **`/efi`**, and **`/etc`** directories read-only for the service. The exception is the **`/etc/bitcoin`** directory as defined by ExecStartPre above.
- ***ProtectHome=true***: This option ensures that the service cannot read or write to the home directory. This is typically recommended for services. We did not create a home directory for the service account, so this is probably redundant given our setup. 
- ***NoNewPrivileges=true***: This stops the startup process of the service and any child processes from obtaining new privileges.
- ***PrivateDevices=true***: This creates a private device file for the service when you start it up.
- ***MemoryDenyWriteExecute=true***: This is to prevent the process from modifying any running code in memory.
- ***WantedBy=multi-user.target***: This is a typical setting for servers. It basically ensures the Bitcoind service starts up whenever you launch your system in command line mode, but not in graphical user mode.

In the next chapter, we will configure and run the Bitcoind service. You may have to return to this chapter if there are any issues with starting the service. 


## Notes

<a name="footnote1">1</a>. See, for example, Shell hacks, “Systemd service file examples”, March 20, 2018 (available at https://www.shellhacks.com/systemd-service-file-example/); Justin Ellingwood, “Systemd essentials: Working with services, units, and the journal”, April 20, 2015 (available at https://www.digitalocean.com/community/tutorials/systemd-essentials-working-with-services-units-and-the-journal). You can also consult the following three-part video series: BeginLinux Guru, “Systemd service files”, 2019 (part I is available at https://www.youtube.com/watch?v=xtIBJNZb1jE).  

<a name="footnote2">2</a>. This is available here: https://github.com/bitcoin/bitcoin/blob/master/contrib/init/bitcoind.service. 

<a name="footnote3">3</a>. See Elliot Cooper, “How to sandbox processes with systemd on Ubuntu 20.04”, September 16, 2020 (available at https://www.digitalocean.com/community/tutorials/how-to-sandbox-processes-with-systemd-on-ubuntu-20-04). 

<a name="footnote4">4</a>. A process in Linux is any program that is running. Any running application will have one or more processes associated with it (the latter if the application consists of multiple programs such as the Google Chrome and Firefox browsers). Each running process has a unique ID and other metadata associated with it. This information is included in the PID file. 