# Configuring and run Bitcoind

In the previous two chapters, we have built Bitcoin Core from the source code and set up the Bitcoind service. In this chapter, we will first set some general configuration options and ensure that we can run Bitcoind as a service. We will, then, ensure that we can issue commands to Bitcoind using the Bitcoin-cli tool. Finally, we will configure Bitcoind to accept both clearnet and hidden service outbound and inbound connections as discussed in *Chapter 7*. 


## General configurations for Bitcoind

By default, Bitcoind will search for a configuration file with the name **`bitcoin.conf`** in the directory **`~/.bitcoin`** when your run it. However, in the service file you wrote in the previous chapter, you specified an alternative directory for it, namely **`/etc/bitcoind`**. 

We will now create the configuration file and set some general configuration options. Many of these configuration options could have been set in the service file. Yet, we prefer to only include in the service file what is absolutely necessary, so most of our configuration options will be set here.  

To create the empty configuration file, proceed as follows: 

* **`$ cd /etc`**
* **`$ sudo mkdir bitcoind`**
* **`$ cd bitcoind`**
* **`$ sudo touch bitcoin.conf`**

Rather than completely writing the configuration file from scratch, you can use a very handy tool that was made by Bitcoin developer Jameson Lopp. It is located at the following Web address: [https://jlopp.github.io/bitcoin-core-config-generator](https://jlopp.github.io/bitcoin-core-config-generator). 

All the configuration options for Bitcoind are on the left side of the screen, subdivided into a number of sections. Any changes you make that deviate from the default values will automatically be placed into the configuration file on the right side of the screen. Using the configurator, proceed as follows:    

1. Select the “Linux” option at the top of the page, as it is set to “Windows” by default.
2. ***Data storage location*** (Bitcoin Core): This option determines where the Block Chain data as well as other data will be stored. Delete the default location and specify the following directory instead: **`/btc-server/bitcoin-core/bitcoin-data`**. ***If you do not set this option properly, Bitcoin Core will try to write data to a home directory for the **`bitcoind`** service account which does not exist.*** Be sure not to confuse this option with the "Blocks data storage location". This latter option will only determine the data directory for the blocks, not any of the other Bitcoin Core data. 
3. ***Index transactions*** (Bitcoin Core): You need to turn this option “on”. By default Bitcoind indexes all valid blocks into a database. This is, however, not the case for transactions, where only transactions belonging to your own wallet are indexed. Setting this option to "on" ensures that we can easily query transactions on the basis of their ID. It is also required for certain applications, such as certain electrum servers, Lightning nodes, and so on.  
4. ***Listen for incoming connections*** (Networking): This option should be turned “on” in most cases. Accepting incoming connections will increase the CPU and bandwith load of your node. But it is generally safe and you support the network as a **supernode**. However, if you have very little bandwidth or if you only want connections over the Tor service, you need to turn this configuration option to "off". By default, Bitcoind will listen to port 8333 for inbound network connections.    
5. ***MAX peer connections*** (Networking): By default, this is set to “125”, which includes a maximum of 10 outbound connections and a maximum of 115 inbound connections. The 10 outbound connections are hardwired into Bitcoin Core. To avoid server overload, start by setting the maximum connections at “25” (10 inbound and 15 outbound). If this setting does not cause any problems for your local network, you can increase the maximum number of peers gradually.
6. **Enable RPC server** (RPC API): You need to turn this on, which will set "server=1" in the configuration file. Without this setting, you will only be able to make RPC requests of Bitcoind with the Bitcoin-cli tool. But you will need to make requests also with other applications, such as Electrs. 

Next, you need to create the bitcoin-data directory, which you have just specified in the configuration file. Move into the **`/btc-server/bitcoin-core`** directory and execute the following instruction:

* **`$ sudo mkdir bitcoin-data`**

You should now have both a "bin" and a "bitcoin-data" directory within the bitcoin-core directory. Make sure that this new bitcoin-data directory is also controlled by the **`bitcoind`** service account and group, and not the **`root`** account and group, by executing the following instructions: 

* **`$ sudo chown bitcoind bitcoin-data`**
* **`$ sudo chgrp bitcoind bitcoin-data`**
* **`$ sudo chmod 750 bitcoin-data`**

As a final step, copy all the contents of the configuration file on your Web browser into the **`bitcoin.conf`** file in the **`/etc/bitcoind`** directory. Ensure that no proper formatting was lost in the transfer.


## Start Bitcoin Core  
 
At this point, we can start the Bitcoind service. We still have not yet configured Bitcoind to work with Tor. But it is better to wait until after the Block Chain has downloaded, as Tor connections may slow down that syncing process. This does not really impact any additional transaction privacy assurances you can achieve with Tor.

Remember that Bitcoind is now configured as a service under the control of the **`bitcoind`** account. You cannot log into this account to start Bitcoind. Instead, you need to use the **systemctl** tool to give instructions to systemd. Execute the following instructions:

* **`$ sudo systemctl enable bitcoind`**
* **`$ sudo systemctl start bitcoind`**

Bitcoind should now start and just run in the background. It will start automatically each time your server starts up. You can verify that the service is working properly with the following instruction: 

* **`$ sudo systemctl status bitcoind`**

The output should tell you that the bitcoind service is active (highlighted in green). If not, you may have some complicated troubleshooting ahead of you. In case the service has failed to start, you should execute **`$ sudo journalctl -xeu bitcoind.service`**. The output will give you some indication of where the mistake might be. It is likely one of the following possibilities: 

* You may have made a formatting mistake with the Bitcoind service file in the previous chapter (**`bitcoind.service`** in the `**/etc/systemd/system directory`**). Make sure you have no capitalization, spelling, or spacing errors anywhere in the file.
* You may have also set particular file or directory paths in the wrong way. Ensure that every file and directory path is correct and starts with the root directory ("/"). 
* You did not set permissions for the **`bitcoind` account in the right way. 

In case you have found one or more mistakes and fixed them, you should execute the following commands: 

* **`$ sudo systemctl stop bitcoind`**
* **`$ sudo systemctl daemon-reload`**
* **`$ sudo systemctl start bitcoind`**

You can again check to see if everything is working by executing **`$ sudo systemctl status bitcoind`**. Only if you see “active” highlighted in green is the service working properly.

If you are still running into problems at this point, the problem is probably still one of those listed above or a combination of them. As it's difficult to say anything more about the possible issues generally, you will have to start troubleshooting on your own. (You can use the the **`$ sudo journalctl -xeu bitcoind.service`** instruction to help you.)

A service file is one type of systemd unit file. To see the status of all systemd units, execute the following instruction:

* **`$ sudo systemctl list-units`**

Once you have the Bitcoind service running, you should see that it loaded successfully (as indicated by “loaded” under the LOAD column), and that it is “active” and “running”. The latter two options respectively mean the configuration options for the service file were successfully executed, and that the unit is running one or more active processes. 

In the same section as for Bitcoind, you should also see information about the SSH, Tor, Fail2ban, and (possibly) Log2ram services that you have enabled at earlier points in this guide. 


## Initial Block Chain download

Once your Bitcoind service is running properly, move into the **`/btc-server/bitcoin-core/bitcoin-data`** directory. If you execute **`$ ls`**, you should no longer see an empty directory, but one with various subdirectories and data files. 

We will explore this **`bitcoin-data`** directory more fully in a later section. For now, you will want to just have a look into the log file named **`debug.log`**. This keeps track of all the operations that Bitcoind is performing in the background. 

As this log file can become very large, you will want to use the **`tail`** command for exploring it. This command allows you to specify how many of the most recent lines added to the file, you want to output (it is set to 10 lines by default). To start, execute the following instruction:

* **`$sudo tail -n 100 debug.log`**

This will show you operations that have been performed by Bitcoind, including obtaining new blocks, finding peers, loading configuration options, and so on. 

If you scroll upwards, you might at some point see the following line: “Bitcoin Core version v25.0 (release build)”. This is the first line Bitcoind outputs when it starts running. If you cannot find this line, it means Bitcoind has already performed too many operations for you to see its starting point with the tail instruction above. 

At this point, you should just let Bitcoind synchronize with the network. This synchronization process can take a substantial amount of time. If you have a decent system and Internet connection, it should probably be ready in a day or so. There are some methods that you can deploy to shorten the waiting time. But unless you are in some urgent rush, we would just suggest you patiently wait out the process rather than complicate matters further.   

In order to verify that you have synchronized, just check the Bitcoind log file at any time, using the following instruction:

* **`$ sudo tail -n 20 debug.log`**

The last twenty lines of the log file will be output and the activities are likely to be related to the discovery of new blocks. Look for the last line that says “UpdateTip”, which concerns the discovery of a new valid block by your node. Moving to the right on the line, you will see the timestamp associated with it. You are synchronized with the network if the timestamp is very recent, typically from within the last ten minutes (though it may take longer sometimes).   


## Using Bitcoin-CLI

Once the Block Chain has downloaded completely, you are ready to configure the Bitcoin-cli tool. It makes remote procedure calls to Bitcoind via the latter's HTTP JSON-RPC API. Let’s unpack this statement.

A **procedure call** (also known as a **function call**) occurs whenever one application requests another application to perform a procedure on its behalf and share the result. A **remote procedure call** occurs whenever the two applications occupy a different virtual address space, and the request needs some transport protocol (such as http). Sometimes these two applications are on the same computer (so are not “remote” in the physical sense). 

A remote procedure call is possible if an application has an application programming interface, or API, that is listening on a certain logical port. Sometimes this is a logical port on the same computer. But in our modern age, API calls are frequently made between computers over a network (typically, the Internet). 

There are different frameworks for structuring the data that is transferred with remote procedure calls. The one standardly employed between Bitcoin-cli and Bitcoind is known as JSON-RPC. It can be used with various transport protocols. Typically that transport protocol is http, as is the case here. 

Instead of the JSON-RPC framework, you can also make remote procedure calls to Bitcoind using the latter's REST API. This protocol is the main data structuring standard for APIs on the World Wide Web, though we will not utilize it anywhere in this guide.

By default, Bitcoind listens to requests intended for its JSON-RPC API on port 8332. Also by default, the API will only listen for requests from applications on the same system and only accept them if they originate with the Bitcoin-cli tool. Nevertheless, you can configure Bitcoind to accept API requests from other applications, and even from other computers. 

In order to use the Bitcoin-cli tool, you should first add its directory to your PATH variable. Proceed as follows:

* Open the “.profile” file in your administrator home directory (**`/home/administrator`**) by executing **`$ nano .profile`**.
* Add the following line at the end of the text file: **`export PATH=$PATH:/btc-server/bitcoin-core/bin`**. 
* Save the file and reboot your system.

The Bitcoin-cli application is in your **`bin`** directory for Bitcoin Core. Adding that directory to the PATH variable ensures that you can easily load this tool from anywhere in your system with the simple instruction **`$ bitcoin-cli`**. Otherwise, you would have to specify the directory of the binary each time you want to use it.

Now first check whether the directory was properly added to your PATH variable, by executing **`$ echo $PATH`**. The directory should be at the end of the output. Next, lets test to see if the tool works. Executing the following instruction requests Bitcoind to return the number of network nodes that are connected to yours:

* **`$ bitcoin-cli getconnectioncount`**

You should now receive an error, which states that ***it cannot locate your RPC credentials or cookie file***. The output should also specify the location Bitcoin-cli has used for configuration options, namely **`/home/admin/.bitcoin/bitcoin.conf`**. Lets explore what is happening.

Before any application on your system, including Bitcoin-cli, can make requests to Bitcoind’s JSON-RPC API, it needs to prove that it is allowed to make such requests. There are two methods an application has for authentication to the API:

* ***Authentication cookie***. An authentication cookie is just a file which contains a long, random string of characters. Each time Bitcoind starts, it will create a new authentication cookie in the main data directory. Given the settings in our configuration file, this will be within the **`/btc-server/bitcoin-0.21.0/bitcoin-data`** directory. One way an application can authenticate requests to the Bitcoind JSON-RPC API is by providing the random string located within the authentication cookie. 
* ***JSON-RPC credentials***. RPC credentials include a username and password. While Bitcoind does not recognize any RPC credentials by default, you can set them in the configuration file. If any application then provides the right credentials, it can make requests to the API.

We have not yet set any RPC credentials in the configuration file for Bitcoind. So Bitcoin-cli can definitely not authenticate any API requests in this manner. Bitcoind did, however, create a cookie file which Bitcoin-cli might attempt to access from the administrator profile. To display it, just move into the **`bitcoin-data`** directory and execute **`$ ls -a`**. You should see a file called **`.cookie`**. 

Using this cookie with the administrator profile, however, comes with two problems. First, each time you invoke Bitcoin-cli, it will not search the **`bitcoin-data`** directory for the cookie file. By default, Bitcoin-cli will instead search for the cookie in the same default directory for the configuration file (i.e., **`/home/administrator/.bitcoin`**). As no valid cookie is located in this directory, Bitcoin-cli cannot authenticate its remote procedure call to Bitcoind.

Second, each time Bitcoind creates a cookie file on startup, it does not provide any permissions to the **`bitcoind`** group. So the administrator has no rights with regards to the cookie, even if they are a member of the **`bitcoind`** group. So even if you point Bitcoin-cli to the **`bitcoin-data`** directory, it cannot actually read the contents of the cookie file when used from the **`administrator`** account.

While you could probably resolve these issues and make Bitcoin-cli work with the authentication cookie from the administrator account, setting RPC credentials is a much more elegant solution. Creating RPC credentials for Bitcoind’s API requires a python3 application called “rpcauth”. Proceed as follows:

* Travel to the **`bitcoin`** directory in your home directory
* Move to the **`/share/rpauth`** directory
* Execute the following instruction: **`$ sudo python3 ./rpcauth.py administrator`**. The last item in this instruction, “administrator”, creates the name for the RPC user. Given that you are using these credentials to access the API from the administrator account, we would recommend you just use the same name. 
* After executing the instruction, the first line in the output will have the following format: `rpcauth=administrator:[hex string]`. This hex string is not your password, but a hash of your RPC password. 
* At the bottom of the output is your password. This string is expressed in base64 format.   

Next, you will have to place these credentials in the right locations on your system. Proceed as follows: 

- Open your bitcoin.conf file in the **`/etc/bitcoind`** directory. 
- After your `# [network]` section, create a new section called `# [rpc]`. This is where you will enter your RPC username, administrator, and the hash of your password. Make sure you are entering the hash of your password and not your actual password. You should specifically enter the following lines:
    -`# [rpc]`
    -`# Username and hashed password combinations for JSON-RPC connections.`  
    - `rpcauth=administrator:[hex string]`
- Save and exit the file.
- Now move into **`/home/administrator`** directory.
- Execute **`$ mkdir .bitcoin`** in case the .bitcoin directory does not yet exist.
- Enter into the directory.
- Execute **`$ touch bitcoin.conf`**.
- Open the bitcoin.conf file with a text editor and enter the following two lines:
    - `rpcuser=admin`
    - `rpcpassword=[password]`
- Save and exit the file.

So here is how all of this now works: When Bitcoin-cli is invoked from the administrator profile, it will search for a bitcoin configuration file in **`/home/admin/.bitcoin`** directory. Instead of looking for a cookie to authenticate its request to Bitcoind’s API, it sees and loads the RPC username and password. It will, then, send requests to port 8332—the port at which the API listens for requests—with the username and a hash of the password. 

Bitcoind will, then, check the username and hash against the information specified in its configuration file under the **`rpcauth`** option. After validation of your credentials, Bitcoind will perform the request and sends the result back to the Bitcoin-cli application. 

Passwords are not stored directly into the Bitcoind configuration file for security reasons. It is much more difficult to break into multiple accounts and computers to expose credentials, than to ascertain them all from one configuration file. This is a standard practice in accessing server applications.

In order for all of this to now work, you will need to restart the bitcoind service. Execute the following commands:

* **`sudo systemctl stop bitcoind`**
* **`sudo systemctl daemon-reload`**
* **`sudo systemctl start bitcoind`**

You can now execute the instruction **`$ bitcoin-cli getconnectioncount`** again from the admin account. It should give you a result. Even if Bitcoind has been running for a while, you will not be connected to more than 10 peers, even though you configured Bitcoind to accept up to 25 peers.

The reason is as follows: While you turned on the “listen” option in the configuration file (option (4) in the *General configurations for bitcoind* section), your node cannot accept inbound connections. Your router and firewall are blocking them. Hence, it can only make the maximum number of outbound peer connections, namely 10. We will fix this in the next section when we configure Tor.

From here on out, we will start to open a lot of ports for communication. It can be convenient to make of these ports on your server record. So at this point, ***make sure you record that the Bitcoind JSON-RPC API is listening locally at port 8332 (that is, at "127.0.0.1:8332)***. 


## Configuring inbound connections

Let's now configure our inbound connections. We will first need to ensure that our firewall, **`ufw`**, accepts requests to come in at port 8333 on any network interface if forwarded from our router. Execute the following command:

* **`$ sudo ufw allow to any port 8333`**

If you now execute **`$ sudo ufw status`**, you should see that port 8333 accepts requests formatted according to both IPv4 and IPv6 standards. You can check that you completed this step correctly by opening up powershell on your managing computer and executing the following command: **`$ Test-Netconnection 192.168.2.150 -Port 8333`**. It should state "true" for "TCPtestsucceeded".

You now need to ensure that your router will listen for inbound connections on its own port 8333 and forwards the traffic to your server. While each router will work differently, the general steps are as follows:

* Log into your router
* Find configuration options for **port forwarding**. It may have a slightly different name like **port mapping**.
* You will have to specify the IP address of your server on the local area network. You will also have to specify a port or perhaps a port range for your server. This should be 8333. You will, then, have to specify a forwarding port on your router, or perhaps a range of ports. This is best also set to 8333. 
* The traffic protocol should be "TCP".
* You can only set these options for IPv4 traffic, as your IPv6 address is disabled. 

You should verify that things are working properly. First, obtain your router's public IP address. You can find it, for example, with a website such as [http://whatismyipaddress.com/](http://whatismyipaddress.com/). Then using a tool from bitnodes at [https://bitnodes.io/](https://bitnodes.io/), you can check that indeed other nodes can reach port 8333 on your server. The tool should also recognize your Bitcoin node, the version, and your current block height.     

It will probably take some time before anyone makes an inbound connection to you, even if everything is working properly. But at some point, you should see some inbound connections. You can verify the existence of inbound connections in two ways:

* First, if you execute **`$ bitcoin-cli getconnectioncount`** and obtain a number greater than 10, you know that inbound connections are active. 
* Second, if you execute **`$ bitcoin-cli getpeerinfo`** you can see the "connection type" for each of your connections. 

Typically you will see inbound connections after 30 to 60 minutes.

At this point, you should also ***add to your server record that Bitcoind is listening on port 8333 for inbound communications and can receive those locally as well as from other computers (that is, at "0.0.0.0:8333)***. You can also ***record the public IP address of your router and the mapping port (that is, [public IP]:8333).  


## Configuring Tor

At this point, we are ready to configure Bitcoin Core with the Tor service. We will first walk through the configuration options for Tor and, then, implement those necessary for running an onion service for inbound and outbound connections. 

Let's start with the configuration options purely for outbound connections over Tor. Suppose you are not wanting to run a hidden service on your own and only want outbound connections. In that case, you cannot run both Tor and clearnet outbound connections at the same time. You would have to make a choice. If you wanted to run your outbound connections over Tor, you would proceed as follows:

* The Tor service offers a SOCKS5 interface that can serve as a proxy server for all your outbound Bitcoind connections and ensure they are only made via ordinary and hidden service circuits. To use this proxy server, you are required to set the **`proxy`** configuration option within the Bitcoind configuration file. By default, the SOCKS5 proxy server runs on port "9050" and only listens for applications on your own computer to communicate to it at the loopback address (127.0.0.1). Any data sent from an application on your computer to this address stays within your computer, and is received by whatever local application is listening on the specified port number. 
* If you have configured the SOCKS proxy server to work on a different port than 9050 for hidden service connections, then you also need to specify this port within the Bitcoind configuration file. The configuration option is **`onion`** and requires both the loopback IP address and the custom port you set. Typically, you would not separate your ordinary and hidden service circuit connections in this manner. If both are the default "9050", for example, you only need to set the **`proxy`** configuration option. 
* In case you are not interested in making outbound connections to hidden services, you can also disable them by setting **`onion=0`**. With this configuration, you will only make connections over ordinary Tor circuits. Conversely, you can also choose to make only outbound connections to hidden services using the **`onlynet=onion`** option. 
* Within the web configurator we have been using, all these options can be adjusted as follows:
    - The **`proxy`** option is adjusted via "Networking/Proxy Connection".
    - The **`onion`** option is adjusted via "Networking/Tor Proxy".
    - The **`onlynet=0`** option is adjusted via "Networking/Only Use Specific Network"

We will not alter these settings for the reasons discussed in *Chapter 7*. However, if you are indeed concerned about hiding your use of Bitcoind, you should also configure your outbound connections appropriately. In most cases, it would typically be enough to just add the following configuration option: **`proxy=127.0.0.1:9050`**. This will ensure your outbound connections only use ordinary and hidden service circuits. There will be no clearnet connections. Any outbound peer will see the IP address of a Tor exit node, and not that of your router.

Let's now turn to the configuration options for the Tor hidden service. We will turn this on, because it will create both inbound and outbound connections over hidden service circuits. To set up a hidden service for Bitcoind, you will first need to give it access to the Tor service's control port. To start, proceed as follows:

1. Enter the Tor configuration file. It is in the path /etc/tor and called **`torrc`**. 
2. Open the file with a text editor.
3. Ensure that you remove the `#` sign before the `Controlport 9051` and `CookieAuthentication 1` options.
4. Add two lines below the `CookieAuthentication 1` line which read `CookieAuthFileGroupReadable 1` and `DataDirectoryGroupReadable 1`, respectively.
5. Exit and save the file.

Setting open the control port allows applications, such as Bitcoind, to send instructions to and control the Tor application. The other settings ensures that access to the control port is determined by a cookie and that any account within the **`debian-tor`** group—which by default only includes the **`debian-tor`** account—has access to it. All you need to do now to ensure that the **`bitcoind`** account has access to the control port is to add it to the **`debian-tor`** group. Execute the following instruction:

* **`$ sudo usermod -a -G debian-tor bitcoind`**

If you now execute **`$ groups bitcoind`**, you should see **`debian-tor`** listed as one of the secondary groups. You can now restart the Tor service, so that all your new settings will be incorporated. Execute the following instruction:

* **`$ sudo systemctl restart tor`**

As a last step, you will now need to restart the bitcoind service. Execute the following instruction:

* **`$ sudo systemctl restart bitcoind`**

Bitcoind will now automatically create the hidden service after restarting. The Tor service will automatically ensure that traffic to your hidden service is forwarded through your router, and you do not need to configure anything on your router on your own.  

To verify it all worked, let's start by traveling to the **`/btc-server/bitcoin-core/bitcoin-data`** directory. You should now see a file called **`onion_v3_private_key`**. This is the private key that determines the public key for your service. 

Next, let's inspect Bitcoind’s log file with the **`tail`** command. If you have already finished synchronizing with the Block Chain, just trying the last 100 lines with **`$ sudo tail -n 100 debug.log`** instruction should be sufficient to show all the log from the beginning of the last restart. You should, then, see your most recent startup of bitcoind with a line that reads as follows (where "[onion]" is your onion address):

* **`tor: Got service ID [onion], advertising service [onion].onion:8333`**

In case this didn't work, you can also just search the log file with the **`grep`** command and "advertising" as a search word. You can do this as follows:

* **`$ grep advertising debug.log`**

You should now see the line that describes the hidden service. 


## Analyzing the connections

To analyze the connections to Bitcoind, execute the following command:

* **`$ bitcoin-cli getpeerinfo | less`**

This will return a JSON object with information about your peer connections. The less command ensures that you can scroll the results. 

Each of your peers should have an "id" number. Each peer should also be described by a variable called "inbound", which receives "false" if it is an outbound connection and "true" otherwise. Any outbound connections over clearnet should have the following types of values for “addr” (the origin of the traffic), “addrbind” (the destination of the traffic), and “addrlocal” (the local source of the traffic):

* **addr**: A public IP address and port, typically port "8333". The public IP is typically that of a home router behind which the other node sits. 
* **addrbind**: The local network address on which your server receives data from the outbound peer (forwarded by the router). This will be a port dedicated to communication with the outbound node. 
* **addrlocal**: The public IP address of your router and a communication port dedicated to the connection. This should standardly be the same port on which your local network address receives data.

As an example, you could see output with the following data:

* "addr": "34.26.202.111:8333"
* "addrbind": "192.168.2.150:33220"
* "addrlocal": "52.35.322.134:33220"

Any outbound connections made by your Tor hidden service to another Tor hidden service will list the onion address of the counterparty for the "addr" variable. You will see a loopback address with a port for the "addrbind" variable. The "network" should also be listed as onion. As an example, you might see something like the following:

* "addr": "zhlpzola3s8w7offksdl4r4zp2rtvm9nb3sosdvgnpos4ezgxxj4eiad.onion:8333",
* "addrbind": "127.0.0.1:57410"
* "network": "onion",

For your inbound peers (inbound: “true”), you can look at the same variables. In the case of inbound peers over clearnet, you will usually see only the "addr" and "addrbind" fields listed, but with the same type of data as for outbound clearnet connections. Your "addrlocal" is namely always your router's public IP address with port as per your port forwarding specifications (standardly also 8333). While your outbound clearnet connections are all made directly to other nodes or (more typically) the routers behind which they sit, the nodes on the other side of these inbound clearnet connections can vary. Their traffic can come directly from them or their home router. It could also come via an ordinary Tor circuit or some othe proxy.  

In the case of inbound connections via your Tor hidden service, you should see similar information for the "addr" and "addrbind" fields as for clearnet connections, but an "addrlocal" field that lists "127.0.0.1:8333". This indicates that you are receiving communications locally from the Tor service and not your router. 


## Cleaning up the configuration file

As a last step, let's make a few stylistic changes to the comments, so everything is a bit more clear and you only have what you need. Adapt it to look as follows: 

`# Generated by https://jlopp.github.io/bitcoin-core-config-generator/`

`# This config should be located in following custom path:`
`# /etc/bitcoind/bitcoin.conf`

`# [core]`  
`# Specify a non-default location to store blockchain data as well as other data.`  
`datadir=/btc-server/bitcoin-core/bitcoin-data`  
`# Maintain a full transaction index, used by the getrawtransaction rpc call. The electrum server requires this option.`  
`txindex=1`  

`# [network]`  
`# Maintain at most N connections to peers.`  
`maxconnections=25`  

`# [rpc]`
`# Accept command line and JSON-RPC commands`
`server=1`
`# Username and hashed password for JSON-RPC connections.`
`# Connection for the administrator to the Bitcoin-cli tool.`
`rpcauth=administrator:[hex string]`  


## Notes

<a name="footnote1">1</a>. See, for example, Shell hacks, “Systemd service file examples”, March 20, 2018 (available at https://www.shellhacks.com/systemd-service-file-example/); Justin Ellingwood, “Systemd essentials: Working with services, units, and the journal”, April 20, 2015 (available at https://www.digitalocean.com/community/tutorials/systemd-essentials-working-with-services-units-and-the-journal). You can also consult the following three-part video series: BeginLinux Guru, “Systemd service files”, 2019 (part I is available at https://www.youtube.com/watch?v=xtIBJNZb1jE).  

<a name="footnote2">2</a>. This is available here: https://github.com/bitcoin/bitcoin/blob/master/contrib/init/bitcoind.service. 

<a name="footnote3">3</a>. See Elliot Cooper, “How to sandbox processes with systemd on Ubuntu 20.04”, September 16, 2020 (available at https://www.digitalocean.com/community/tutorials/how-to-sandbox-processes-with-systemd-on-ubuntu-20-04). 

<a name="footnote4">4</a>. A process in Linux is any program that is running. Any running application will have one or more processes associated with it (the latter if the application consists of multiple programs such as the Google Chrome and Firefox browsers). Each running process has a unique ID and other metadata associated with it. This information is included in the PID file. 