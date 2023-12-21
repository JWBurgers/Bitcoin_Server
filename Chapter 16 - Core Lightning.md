# Core Lightning

The Lightning Network is a payment network that sits on top of the Bitcoin Network. Transactions on the Lightning Network are practically instant and significantly cheaper than standard Bitcoin transactions. They also offer substantially more transaction privacy.

Transactions on the Lightning Network operate via **payment channels**. Creating a payment channel on the Lightning Network requires a 2-out-of-2 multisignature output with a peer in a **funding transaction**, as well as two **commitment transactions**, one from each peer, which spend the multisignature output. While the funding transaction is published to the Block Chain, the two commitment transactions are not. After creating the channel, peers can make payments to each other by revoking the previous set of commitment transactions and creating a new set. 

Updated commitment transactions are only shared between the peers and not published to the Block Chain. This is why they are often called **off-chain** transactions. The fact that transactions are off-chain is an important way in which the Lightning Network can offer faster, cheaper, and more private payments to peers. Either peer can close the channel at any time by publishing the latest commitment transaction to the Block Chain. 

Channels can be **public** or **private**. The former channels are broadcasted to other peers in the network, while the latter are not. When channels are public, other peers on the Lightning network can see the **capacity** of a payment channel through the funding transaction. However, as the commitment transactions are only shared between the peers in an encrypted fashion, other peers will not directly know the **balance** of a payment channel. 

Currently, most Lightning channel funding and closing transactions are relatively easy to identify on the Bitcoin Block Chain. As more nodes move to using Taproot funding and closing transactions, however, these will be harder to distinguish from ordinary Bitcoin transactions. As long as both peers cooperate in the closing of the channel, a closing transaction will be harder to identify. All this will offer an additional degree of privacy for the Lightning Network, particularly for private channels.

Though payment channels can only be established between two peers, payments are not limited to your own channels. The Lightning Network has routing abilities. Currently those are implemented through what are called **hashed timelock contracts**. There will at some point be an upgrade to what are known as **point timelock contracts**. Any intermediary node which routes payments for others through one of their channels generally demands a fee for the service. That fee consists of both a flat charge per transaction and a variable charge which depends on the value of the payment. 

To update the channel balance, the peers revoke previous commitment transactions and create a new set. This revocation occurs by each peer sharing a **revocation key** with the other. If one of the peers breaks the agreement and publishes a revoked commitment transaction to the Block Chain, the other peer has the ability to claim all the funds in the channel with the help of the revocation key. This mechanism incentivizes peers to only close channels by publishing their latest commitment transaction. That is, the transaction which contains an accurate view of the balance in the channel considering all the payments that were made.

The revocation mechanism in a Lightning channel has two important security implications:

1. You need to monitor your channel partners in the network and ensure that they are not publishing revoked commitment transactions to the Block Chain. A channel peer might attempt such a move if the balance in a revoked commitment transaction is more favorable than the balance of the current commitment transaction. This monitoring activity requires your node to be online. You can acquire some support as well via **watchtowers**. These are servers that help monitor your previous commitment transactions.

2. If you close a Lightning channel, you must always ensure that you publish the last commitment transaction on your side. If you publish a revoked commitment transaction, your peer can take the whole balance of the channel. 

In this chapter, you will download, install, and configure a Lightning node on your Bitcoin server. Several interoperable node packages are available, including **LND**, **Core Lightning**, and **Éclair**. We will use Core Lightning by Blockstream, which is known for its stability. In the next chapter, we will discuss some aspects of managing your node and connect it to several wallets. 

Lets begin by downloading and verifying the tarball to install the Core Lightning binaries on our machine.  


## Download and verify the tarball

The Core Lightning project is located in the following Github repository: [https://github.com/ElementsProject/lightning](https://github.com/ElementsProject/lightning). 

You should first download the right tarball and the verification files. Proceed as follows:

* Move into your **`/btc-server directory`**. 
* Within the Github project, find the latest release resources for Core Lightning. In our case, the latest release was **`v23.11.1`**.
* Among the release assets, look for a tarball that was created for the most recent version of Ubuntu. Right click on the link and paste it into the following command on your server terminal: **`$ sudo wget [TARBALL LINK]`**
- Similarly copy the **`SHA256SUMS`** and **`SHA256SUMS.asc`** files onto your server.

We will need to verify the tarball. You can find the PGP public keys for all those contributors with commit access to the Core Lightning repository within the **`/contrib/keys`** directory. As of the time of writing, there were six contributors with commit access. Find out who of them created the latest release binaries. We will need to verify their PGP key against multiple sources.

Only full PGP keys are listed within the **`keys`** directory. It is easier to work with the fingerprint. Execute * **`$ sudo gpg --verify SHASUMS.asc`**. While you cannot verify the signature, this should output a PGP public key fingerprint for you.

You can, then, verify this fingerprint against other sources such as the Ubuntu keyserver. Once you are confident that the public key and identity belong together, you can import it onto your key ring from the Ubuntu key server and validate it with your own signing key. For Christian Decker's key, for example, the commands would be as follows:

* **`$ sudo gpg --keyserver keyserver.ubuntu.com --search B7C4BE81184FC203D52C35C51416D83DC4F0E86D`**
* **`$ sudo gpg --sign-key B7C4BE81184FC203D52C35C51416D83DC4F0E86D`**

If you now execute **`$ sudo gpg --list-keys`**, you should see the public key listed. At this point, you can also import all the public keys of all the others with commit access to the Core Lightning repository if you wish. You can remove the **`SHA256SUMS`** and **`SHA256SUMS.asc`** files from your server.


## Install Core Lightning

To install Core Lightning, first ensure that your system is up to date with the following commands:

* **`sudo apt update`**
* **`sudo apt upgrade`**

Next, move into your **`btc-server`** directory and unpack the tarball with the command below. You should note that this is a slightly different type of tarball than for Fulcrum in *Chapter 12*, so the two commands for unpacking the tarballs are not entirely the same. 

* **`$ sudo tar -x -f clightning-v23.11.1-Ubuntu-22.04.tar.xz`**

Core Lightning should now install. You should obtain a directory called **`usr`** within your **`btc-server`** directory. Let's organize a bit with the following instructions:

* **`$ sudo mv usr core-lightning**`**
* **`$ sudo rm -R clightning-v23.11.1-Ubuntu-22.04.tar.xz`**

If you enter into the **`core-lightning`** directory, you should see three subdirectories: **`bin`**, **`libexec`**, and **`share`**. The latter contains documentation and manuals. The **`libexec`** directory contains numerous supporting binaries for your server. They are not intended to be executed directly by you. The **`plugins`** subdirectory contains plugins that offer extra functionality for your Core Lightning node. Any additional plugins you want to install later will be placed within this directory. Finally, all the main binaries are included within the **`bin`** directory. They are as follows:

* **`lightningd`** is your node application.
* **`lightning-cli`** is a command line tool for sending requests to Lightningd.
* **`lightning-hsm`** is a tool for your **`hsm_secret`** file, which is used to derrive on-chain and off-chain keys.
* **`reckless`** is a plugin manager.

At this point, you can attempt to run the Lightningd help with the following command from the **`bin`** directory:

* **`$ sudo ./lightningd --help`**

You might receive an error about about a missing library called **`libpq5`**. To address the error, execute the following instruction:

* **`sudo apt install libpq5`**

You should now be able to run the Bitcoind binary with the **`help`** option without problems.


## Set up the Lightningd service

To set up the Lightningd service, let's first create the server account. We will call it "lightningd". Execute the following instruction: 

* **`$ sudo useradd --shell /usr/sbin/nologin --system -M lightningd`**

The account owner of the **`core-lightning`** directory should be the **`lightningd`** account. The group owner should be the **`lightningd`** group. We also need to set the read and write permissions. Within the **`btc-server`** directory, execute the following instructions:

* **`$ sudo chown -R lightningd core-lightning`**
* **`$ sudo chgrp -R lightningd core-lightning`**
* **`$ sudo chmod -R 750 core-lightning`**

To verify all the instructions you just passed were incorporated successfully, you can execute **`$ ls -l`** within the **`btc-server`** directory. 

To ensure that the **`administrator`** account can still navigate through the **`core-lightning`** directory with these permission settings, we need to add it to the **`lightningd`** group. Proceed as follows:

* **`$ sudo usermod -a -G lightningd administrator`**

You can verify that this instruction worked by executing **`$ groups administrator`**. 

Before starting Lightningd, we must also ensure the **`lightningd`** account has access to the **`bitcoin-core`** directory on our server. This is because it needs access to the Bitcoin command line tool. We can provide access by adding the **`lightningd`** account to the **`bitcoind`** group. Execute the following instruction: 

* **`$ sudo usermod -a -G bitcoind lightningd`**

If you now execute **`$ groups lightningd`**, you should see **`bitcoind`** listed as one of the secondary groups.

You should now reboot your system to ensure that all these group changes are incorporated. 

Once your system has rebooted, we can create the service file for Core Lightning. Enter into the **`/etc/systemd/system`** directory and execute the following instruction: 

* **`$ sudo touch lightningd.service`**

We can find recommendations for the service file within the **`contrib/init`** directory of the Core Lightning Github project. Our suggestion for the service file is seen below and it includes some adaptations. Open the **`lightningd.service`** file and save the content below into it. 

**Lightningd service file**

`[Unit]`
`Description=Core Lightning daemon`
`Requires=bitcoind.service`
`After=bitcoind.service`
`Wants=network-online.target`
`After=network-online.target`

`[Service]`
`ExecStart=/btc-server/core-lightning/bin/lightningd --conf /etc/lightningd/config --pid-file=/run/lightningd/lightningd.pid --daemon`
`Type=simple`
`PIDFile=/run/lightningd/lightningd.pid`

`Restart=on-failure`
`TimeoutSec=60`
`RestartSec=60`

`PermissionsStartOnly=true`
`ExecStartPre=/bin/chgrp lightningd /etc/lightningd`

`User=lightningd`
`Group=lightningd`

`RuntimeDirectory=lightningd`
`RuntimeDirectoryMode=0710`

`StateDirectory=lightningd`
`StateDirectoryMode=0710`

`PrivateTmp=true`

`ProtectSystem=full`

`NoNewPrivileges=true`

`PrivateDevices=true`

`MemoryDenyWriteExecute=true`

`[Install]`
`WantedBy=multi-user.target`


As mentioned, this service file contains some adaptations from the recommendations offered within the Core Lightning repository. The main differences are as follows:

* We set the **`PermissionsStartOnly=true`** option so that the permissions for Lightningd are only applied to the **`ExecStart`** process. 
* We added the **`--daemon`** option, so that Lightningd automatically runs in the background.
* We did not set the **`ConfigurationDirectory=lightningd`** option, as we already have that directory within the **`etc`** directory. To ensure that the **`lightningd`** account has the ability to read the file in the directory, we set **`ExecStartPre=/bin/chgrp lightningd /etc/lightningd`**. 
* We noticed that in case of failure, Lightningd would restart too quickly with the default settings. So we set **`RestartSec=60`**. We also set **`TimeoutSec`** to specify how long to wait when starting or stopping the service before marking it as failed or forcefully killing it.


## Configurations

We need to create the configuration file in the **`/etc/lightningd`** directory. Proceed as follows:

* Move into the **`/etc`** directory
* Execute **`$ sudo mkdir lightningd`**
* Move into the **`lightningd`** directory and execute **`sudo touch config`**

To send requests to the Bitcoind RPC API, Lightningd will require credentials. To create them, proceed as follows: 

* Travel to the **`bitcoin`** directory in the **`administrator`** account's home directory
* Move to the **`/share/rpauth`** directory
* Execute the following instruction: **`$ sudo python3 ./rpcauth.py lightningd`**.  
* After executing the instruction, the first line in the output will have the following format: `rpcauth=lightningd:[hex string]`. 
* At the bottom of the output is your password.    

You will need to place the credentials in your Bitcoind configuration file. Proceed as follows: 

- Open your **`bitcoin.conf`** file in the **`/etc/bitcoind`** directory. 
- Below the RPC credentials for your **`btc-rpc-explorer`** account, place those for your **`lightningd`** account. The relevant section of your Bitcoind configuration file should now look as follows:
    -`# [rpc]`
    -`# Username and hashed password combinations for JSON-RPC connections.`  
    - `rpcauth=administrator:[hex string]`
    - `rpcauth=electrs:[hex string]`
    - `rpcauth=btc-rpc-explorer:[hex string]`
    - `rpcauth=lightningd:[hex string]`
- Save and exit the file.

Copy your RPC password temporarily into a text document so that you don't lose it. 

We are now ready to populate the Lightningd configuration file. Move back into the **`/etc/lightningd`** directory and open the configuration file. Populate it with the following content:

`## Bitcoin control options`
`# This sets the network to which Core Lightning must synchronize. This is standardly "bitcoin".` 
`network=bitcoin`
`# The Lightning Daemon requires access to the Bitcoin command line interface tool. This option specifies its location.`
`bitcoin-cli=/btc-server/bitcoin-core/bin/bitcoin-cli`
`# This points Lightningd to the Bitcoind data directory`
`bitcoin-datadir=/btc-server/bitcoin-core/bitcoin-data`
`# This is the RPC username for the Bitcoind RPC API.`
`bitcoin-rpcuser=lightningd`
`# This is the password for using the Bitcoind RPC API.`
`bitcoin-rpcpassword=[PASSWORD]`
`# This identifies the IP address on which the Bitcoind RPC API is listening.`
`bitcoin-rpcconnect=127.0.0.1`
`# This identifies the port on which the Bitcoind RPC API is listening.`
`bitcoin-rpcport=8332`

`## Lightning daemon options`
`# This sets the working directory. A network-specific subdirectory will be created for each network with which you synchronize Core Lightning. Setting "network=bitcoin" above means a "bitcoin" directory will be created on startup.`
`lightning-dir=/btc-server/core-lightning`
`# This option sets where to output log data, instead of in the terminal. This must be set when running Lightningd as a daemon.`
`log-file=/btc-server/core-lightning/log`
`# This option ensures detailed log output. If you leave this to the default (i.e., "info"), you cannot really follow what your node is doing and will have a difficult time with any troubleshooting.`
`log-level=debug`
`# This option sets the account and group permissions for the lightning-rpc file. Access to this file is needed to command the lightning command line interface.`
`rpc-file-mode=0660`

`## Lightning node customization options`
`# This sets a name for your node.`
`alias=[ALIAS]`
`# This sets the flat fee for every payment that passes through your node as part of the routing system in the Lightning Network. Unless you are a high-volume node, you should set this base fee to 0.`
`fee-base=0`
`# This is a percentage of the total amount of the transaction to charge as a fee for any payment you route. It is expressed in millionths. So, for example, a fee of "10000" means you charge 0.1% of the total amount of the payment as a fee. A low-volume node can set it to "0", or at least a very low amount.
fee-per-satoshi=10`
`# This sets the minimum channel capacity in Satoshis. You should set this very high, as small channels typically do not benefit your liquidity and are not of benefit to the network as a whole. A minimum of "500000" satoshis is, therefore, a good minimal capacity, unless you have very specific reasons for allowing smaller channels.` 
`min-capacity-sat=500000`

`## Lightning channel and HTLC options`
`# This sets how many block confirmations a channel funding transaction requires before usage of the channel. The default is set to 1.`
`funding-confirms=3`

`## Networking options`
`# # The "addr" option is used to configure your inbound connections. The configured inbound connections are automatically announced to the network. We have set it twice here. The first setting ensures that your node listens for inbound IPv4 traffic on all network interfaces. The port number of "9735" is default. The second option instructs Lightningd to create a hidden service using the Tor service. The service will be static, meaning the address stays the same even after a restart of Lightningd. It is important that these two options are configured in the order below. The first option ensures that Lightningd also listens to port 9735 on the loopback address. This will be where the hidden service forwards inbound traffic to locally.`
`addr=0.0.0.0:9735`
`addr=statictor:127.0.0.1:9051`
`# This option sets the proxy for outbound connections using hidden service or standard Tor circuits.`
`proxy=127.0.0.1:9050`
`# This option can be set to "true" if you only want to make outbound connections via Tor hidden service or standard circuits. Setting it to "false" ensures you will make outbound connections both over Tor and clearnet.`
`always-use-proxy=false`

The explanations given for many of these configuration options should suffice on their own. We will discuss inbound and outbound connection settings within the next section. 


## Configuring the hidden service and inbound connections

Transactions in the Lightning Network are not publicly broadcasted. All connections between peers are authenticated and encrypted utilizing the public key IDs of nodes. Any transactions sent on the network also rely on onion routing scheme called **Sphinx**. So only the sending node of a payment will directly know with certainty all the hops involved and the destination. All the intermediate hops only directly know the previous and next hop. The destination only knows the last intermediate hop. 

All these features are some of the core reasons why Lightning payments generally offer much better transaction privacy than on-chain Bitcoin transactions. There are various ways in which you can use Lightning to bolster that transaction privacy. Using **multi-path-payments**, for example, will split the payment over different paths, so that no peers can directly know the amount of the transaction. For another example, if you ensure that the UTXOs used for your channels cannot be linked to your identity, you will have a higher assurance of transaction privacy. Otherwise, others will be able to easily deduce your channels, their capacity, and your channel partners. This information can help to form a picture of the transactions you are making on the Lightning network.

Running a Tor hidden service and setting all outbound and inbound connections to run over Tor circuits helps ensure that no one knows you are running a Lightning node. But it also impacts your transaction privacy. With certain techniques, attackers may identify that particular payments belong to your node. If they also know who you are, then, clearly this breaks your transaction privacy. 

Running your Lightning node only via Tor does not quite have the same security concerns as for your Bitcoin node. That said, it may decrease your connectivity and your ability to route payments for others. Given that your baseline transaction privacy is fairly good within the Lightning network, we suggested settings in the previous section that ensure your node has a mix of clearnet and Tor connections. Specifically, the networking settings in the suggested configuration file achieve the following: 

1. That your node accepts inbound connections over clearnet 
2. That your node accepts inbound connections via a hidden service
3. That your node makes outbound connections via ordinary Tor circuits to clearnet addresses
4. That your node makes outbound connections to hidden services
5. That your node makes outbound connections to clearnet addresses

At this point, we have to take some steps to ensure that all these network configuration settings for Lightningd will work. 

Let's start with the hidden service. In *Chapter 10*, we configured a hidden service for Bitcoind. The options we set, then, in the Tor configuration file (**`torrc`**) were as follows:

* **`Controlport 9051`**
* **`CookieAuthentication 1`**
* **`CookieAuthFileGroupReadable 1`**
* **`DataDirectoryGroupReadable 1`**

Setting those options allows Bitcoind to instruct Tor to create a hidden service. Lightningd has the same capabilities. We just need to make the **`lightningd`** account a part of the **`debian-tor`** group, so that the **`lightningd`** account can control Tor through the control port. Execute the following instruction:

* **`$ sudo usermod -a -G debian-tor lightningd`**

If you now execute **`$ groups lightningd`**, you should see **`debian-tor`** listed as one of the secondary groups.

Next, we need to ensure that all your inbound clearnet connections are properly set up. You first need to ensure that your server's firewall accepts requests to come in at port **`9735`** on any IPv4 network interface. Execute the following command:

* **`$ sudo ufw allow to any port 9735`**

If you now execute **`$ sudo ufw status`**, you should see that port **`9735`** accepts requests formatted according to both IPv4 and IPv6 standards. You now need to ensure that your router will listen for inbound connections on its external port **`9735`** and forwards the traffic to your server on port **`9735`**. The general steps are as follows:

* Log into your router. 
* Find the configuration options for **port forwarding**. It may have a slightly different name like **port mapping**.
* You will have to specify the IP address of your server on the local area network. You will also have to specify a port or perhaps a port range for your server. This should be **`9735`**. You will, then, have to specify an external port on your router, or perhaps a range of ports, from which it can forward traffic to your server. This external port is best also set to **`9735`**. 
* The traffic protocol should be **`TCP`**.
* You can only set these options for IPv4 traffic, as your IPv6 address is disabled for your router. 

Once these settings are in place, you should ***add to your server record that Lightningd is listening on port **`9735`** for inbound communications***. You can also ***record the mapping port on your router (that is, **`9735`**)***.  


## Start Lightningd

At this point, we can enable and start the Lightningd service. Execute the following two commands:

* **`$ sudo systemctl start lightningd.service`**
* **`$ sudo systemctl enable lightningd.service`**

Lightningd should now start and just run in the background. You can verify that the service is working properly with the following instruction: 

* **`$ sudo systemctl status lightningd`**

The output should tell you that the bitcoind service is **`active (running)`** highlighted in green. In case the service has failed to start, you should execute **`$ sudo journalctl -xeu bitcoind.service`**. The output will give you some indication of where the mistake might be. You can also enter into the **`/btc-server/core-lightning`** directory and open the **`log`** file if you see it. This will also help with troubleshooting.

Once you have the service running successfully, you should first enter into the **`btc-server`** directory. By default, Lightningd will make a **`bitcoin`** directory inside the **`core-lightning`** directory that only the service account, **`lightningd`**, will be able to access. But if you want to explore that data and also execute the command line tool with your administrator profile, you will need to change the permissions accordingly. Execute the following instructions: 

* **`$ sudo chmod -R 750 core-lightning`**

To verify that the instructions you just passed were incorporated successfully, you can execute **`$ ls -l`** within the **`btc-server`** directory. You should now be able to navigate through the **`core-lightning`** directory. 

To verify that the port for clearnet inbound transactions was opened successfully, you can use a port scanner (e.g., [https://www.ipfingerprints.com/portscan.php](https://www.ipfingerprints.com/portscan.php).

As a final step, move into the **`core-lightning`** directory and show the last 100 lines of the **`log`** file by executing **`$ sudo tail -n 100 log`**. You will see various messages about Lightningd's activities. 


## Configure Lightning-Cli and make peer connections

To use the **`lightning-cli`** tool in the **`core-lightning/bin`** directory, you will need to point it to the **`lightning-rpc`** file in the **`core-lightning/bitcoin`** directory. This is because you have a custom path to the file. One option is to specify the path with each separate instruction. That is rather tedious. Instead, it is better to create an alias with the custom path specified as an option. 

To start, let's first add the **`lightning-cli`** command to the system path variable for the **`administrator`** profile. This way you can execute it from any directory location. Proceed as follows:

* Open the **`.profile`** file in your administrator's home directory (that is, **`/home/administrator`**) with a text editor.
* Ensure that the last line reads as follows: export **`PATH=$PATH:/btc-server/bitcoin-core/bin:/btc-server/core-lightning/bin`**. 
* Save and exit the file.

Next, we will create the alias. Proceed as follows:

* In the **`/home/administrator`** directory open the **`.bashrc`** file with a text editor.
* At the bottom of the file, enter the following line: **`# Custom aliases`**. This creates a new section for your custom aliases.
* On the line below it, enter the following: **`alias ln-cli="lightning-cli --rpc-file=/btc-server/core-lightning/bitcoin/lightning-rpc"`**.
* Save and exit the file.

For these settings to take effect, you will have to reboot your system. Go ahead and do that now. Once it has rebooted, you can check whether all the settings worked by just executing **`$ ln-cli --help`** from anywhere in your system.

Now that you have setup the command line tool, you are ready to make peer connections and receive data about the Lightning Network. To start, first execute the following instructions:

* **`$ ln-cli listpeers`**
* **`$ ln-cli listnodes`**

Both queries will probably offer empty outputs. In order to start attracting inbound peers and to learn about other the Lightning Network, we will need to make an outbound connection first. You can find connection data on various platforms. We will use **Amboss**. 

* On a browser, visit [https://amboss.space](https://amboss.space).
* Find the page on the Lightning node owned by Kraken, a very popular node.
* Find their **node address** on the page. It should be a combination of a public key with network address information. Kraken only has a clearnet address, and it should be as follows: **`02f1a8c87607f415c8f22c00593002775941dea48869ce23096af27b0cfdcc0b69@52.13.118.208:9735`**.

To make a connection to Kraken, execute the following instruction:

* **`$ ln-cli connect 02f1a8c87607f415c8f22c00593002775941dea48869ce23096af27b0cfdcc0b69@52.13.118.208:9735`**

At this point, you should find a few more popular nodes and connect to them in the same manner. Try to obtain a combination of clearnet and Tor addresses for making these outbound connections. After you have made some connections, you can execute the following instructions again:

* **`$ ln-cli listpeers`**
* **`$ ln-cli listnodes`**

The first command should output the peers to which you have made outbound connections. The second instruction should show your connected peers, as well as a fair list of nodes about which you now have knowledge. Now move back into the **`/btc-server/core-lightning`** directory and show the last 100 lines of the log with the following command:

* **`$ sudo tail -n 100 log`**

You should see in the log that you are acquiring network information from the peers to which you have an outbound connections.