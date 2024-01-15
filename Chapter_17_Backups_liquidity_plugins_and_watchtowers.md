# Backups, liquidity, plugins, and watchtowers

Now that your Core Lightning node is up and running, we will need to perform some initial node management tasks: making backups, obtaining liquidity, installing plugins, and setting up watchtowers. We will do these from the command line. In the next Chapter, we will set up Ride the Lightning that will allow performing some of these tasks from a graphical user interface. 

Let's start with ensuring that we have a backup system for any bitcoin that we place into our Lightning node's on-chain wallet or commit to channels with peers. If our node ever real data loss issues, at least we will not lose our funds. 


## Backup of hsm_secret

Each lightning node has a master private-public keypair generated with a seed and, possibly, a password. This master keypair is used to generate standard on-chain bitcoin addresses, as well as all the keys necessary for creating and updating payment channels. 

If you travel to the **`/btc-server/core-lightning/bitcoin`** directory, you will see a file called **`hsm_secret`**. This file contains the seed for your Core Lightning wallet's master private-public keypair. ***A backup of this file is a necessary condition for restoring on-chain and channel funds***. For on-chain funds, a backup of this file is also a sufficient condition for restoration. This is ***not*** the case for channel funds, which also requires a backup of your channel data. 

The **`hsm_secret`** file is in binary format. But you can easily output the content in hexadecimal format. To do so, proceed as specified below:

* Install an application called **`xxd`** with the instruction **`sudo apt install xxd`**.
* From within the **`/btc-server/core-lightning/bitcoin`** directory, execute **`sudo xxd hsm_secret`**

You will see an output that has two rows with hexadecimal content. You can ignore the content at the end of each of the hexadecimal rows. **You should record the hexadecimal content of the hsm_secret file on your server record.** Following is an example you can follow for the format of you recording. 

* **`00: 30cc f221 94e1 7f01 cd54 d68c a1ba f124`**
* **`10: e1f3 1d45 d904 823c 77b7 1e18 fd93 1676`**

This is all you need for backing up the seed in your Core Lightning node. We will test the backup in the *Obtaining Liquidity* section, once you have some funds stored and a channel open. 

Note that in order to make payments, your Lightning Node will need access to the **`hsm_secret`** file. Any funds stored on your node are, thus, stored in a ***hot wallet***, at least as long as your server is online and your Lightning node is in operation. This required hot storage of funds stands in sharp contrast to proper practices with ordinary management of on-chain funds, where the general prescription is to always keep your private keys offline. This hot wallet situation on your Lightning node is unavoidable, and one of the main ways in which Lightning wallets come with different risks than purely on-chain wallets such as Sparrow and Specter. It is part of the price for faster, cheaper, and more private transactions. 

The **`hsm_secret`** file can be encrypted. Doing this means that your wallet will have to be decrypted each time your node is restarted. One can configure the unlocking to happen automatically. Given that the **`hsm_secret`** file needs to be unlocked for the operation of your node, encrypting it has really limited security benefits. The main benefit is if you ever want to move the seed file to another device. Hence, we will not set up the encryption and automatic unlocking here. You are free to set this up on your own. 


## Channel backups

The **`hsm_secret`** file is sufficient to recover your Lightning node's on-chain funds from the Block Chain. So it is important in case you lose the node data on your server through a hardware failure, accidental data deletion, file corruption, and so on. While the wallet seed is also necessary for recovering channel funds, it is **not sufficient** for guaranteed recovery. The seed, namely, does not provide any information on the channels you had in operation, in particular the latest state of commitment transactions. While restoring channel funds from only the seed is not impossible, you are at risk of losing channel funds. 

All your open channels, closed channels, and the transactions made on those channels are stored in a database called **`lightningd.sqlite3`** within the **`/btc-server/core-lightning/bitcoin`** directory. If you have a backup of this database and are absolutely certain that all your channels have their latest states, you could just recover your node entirely with using the **`hsm_secret`** file and the **`lightningd.sqlite3`** database.

The problem is that you have to be absolutely certain the database contains the latest states of the channels. Per the revocation mechanism explained in *Chapter 16*, if you publish an old commitment transaction to the Block Chain in order to close a channel, your peer can take the whole balance of the channel. So you really should not recover your channel funds with this database, unless you are ***absolutely certain*** that you have the lastest commitment transactions for all the channels.

Another mechanism for recovery is through a file called **`emergency.recover`**, also located in the **`/btc-server/core-lightning/bitcoin`** directory. This file only keeps a record of all your open channels. With the information in this file, you can request all your channel peers to close existing channels. They cannot be 100% sure as to the reason why you are making the request. In addition, you may be leveraging watchtowers. Hence, they will be incentivized to close the latest state of the channel, so as not to risk losing all their funds in the channel. This approach prevents users from accidentally publishing old channel states and losing all their funds after a recovery process. The downside is that you will have to re-open channels to your node, and that can be quite the effort, as explained in the section *Obtaining Liquidity* below. 

Your best course of action is to have backups of both the **`lightningd.sqlite3`** database and the **`emergency.recover`** file. If you ever suffer a serious data loss on your server, you at least have flexibility with regards to how you restore your node. 

To start, obtain an empty USB drive. We will need to format it with the **`ext4`** file system. Easiest is to do this on another Linux system, but we will assume that you are using your server for formatting the USB drive. The instructions are offered below, but you can adapt them if you have another Linux system available. 

* Execute **`$ sudo systemctl set-default graphical.target`** and reboot your system to enter the graphical user environment.
* Plug in your empty USB drive. 
* Open a terminal windown and execute **`sudo gparted`**.
* Select your USB drive from the drop-down menu on the right. ***Be sure you select the right drive***
* If there are any visible partitions on your USB drive, right-click on each of them and select **`unmount`**. For each partition, click **`Partition`** from the menu and select **`delete`**. Click the green check mark at the top of the application window to execute each deletion operation. 
* On the **`Device`** menu, choose **`Create Partition Table`**.
* Select the **`gpt`** option and click **`Ok`**.
* Once the partition table has been created, select **`Partition`** and select **`New`**.
* On the right side of the menu, select **`primary`** partition for the type. Select the **`ext4`** file system. You can choose the label **`channel-backups`**. Once you have addressed all the options, click **`Add`**. Be sure to click the green check mark on the top of the screen to create the volume.
* After completion of the process, execute **`sudo systemctl set-default multi-user.target`** in the terminal window.
* Reboot your system back into the command line mode. 

Take out the USB drive from your server for a moment. To see the current state of the disks on your server, execute **`lsblk`**. If you now place the USB drive back into your server and execute **`lsblk`** again, you will see it listed, probably as **`sda`** or **`sdb`**. The USB drive may have automatically mounted in your **`/media`** directory, but more than likely this is not the case. If not, proceed as follows, assuming the partition is **`sdb1`**:

* Move into the **`/media`** directory and execute **`sudo mkdir usb`**.
* Move into the **`/etc`** directory and open the **`fstab`** file with a text editor.
* Insert a line at the bottom with the following elements: **`/dev/sdb1`**, **`/media/usb`**, **`ext4`**, **`defaults,noatime`**, **`0`**, **`0`**. It is important that each item is separated with a tab from the next item. 
* Reboot your system.

If you did everything correctly, the USB should now mount each time your server is started. As we will need the **`lightningd`** account to write to the USB, we will have to set the ownership and permissions for the **`/media/usb`** directory accordingly. Move into the **`/media`** directory and execute the following instructions:

* **`$ sudo -R chown lightningd usb`**
* **`$ sudo =R chgrp lightningd usb`**
* **`$ sudo =R chmod 750 usb`**

We will now configure Core Lightning to automatically backup the **`lightningd.sqlite3`** database to your USB stick. Move into the **`/etc/lightningd`** directory and open the **`config`** file with a text editor. After the **`Networking options`** section, insert the following text:

`## Backups`
`wallet=sqlite3:///btc-server/core-lightning/bitcoin/lightningd.sqlite3:/media/usb/lightningd.sqlite3`

You can now restart the Core Lightning service. Execute **`$ sudo systemctl restart lightningd.service`** and check that it's working properly with **`$ sudo systemctl status lightningd.service`**. If you are running into issues, it is likely because the configuration file is specified incorrectly or because the ownership and permission settings for the USB drive are not accurate. 

We will also automatically backup our **`emergency.recover`** file. To set it up for a daily backup, proceed as follows:

* Install **`rsync`** by executing **`sudo apt install rsync`**.
* Open up crontab by executing **`$ sudo crontab -e`**
* Insert the following line at the bottom: **`30 17 * * * sudo rsync -av --log-file=/media/usb/rsync_log /btc-server/core-lightning/bitcoin/emergency.recover /media/usb`**. 
* Exit and save the file.

Every day at 17:30, the rsync application will make a copy of the **`emergency.recover`** file and place it on your USB drive. The option **`a`** indicates archival mode, which is suitable for backups. The options also ensure a log file is stored on your USB drive with verbose output. That way you can verify whether rsync is working properly.  


## Obtaining initial liquidity

While you now have a Core Lightning node running on your system with all the proper backups in place, you don't have any liquidity. Importantly, there are many ways you can go about obtaining liquidity for your node. ***You should read the sections in this chapter about obtaining liquidity first and do some research on your own before taking any specific steps to obtain liquidity.*** 

Let's start with some background. We can make a distinction between **outbound** and **inbound** liquidity. The former refers to your capacity to make payments to others, while the latter refers to your capacity to receive payments from others. While obtaining outbound liquidity is easier, obtaining either type of liquidity presents some serious financial and user experience issues within the current Lightning ecosystem. In fact, while in some ways the Lightning experience has improved in recent years, there is at least one way in which it has arguably become worse, namely with regards to obtaining liquidity. 

Assuming your goal is to become just a Lightning user, not a routing operator, then you can technically use the Lightning network with only one channel. You should go that route if you have a limited budget. However, you should note that relying on one channel comes with various downsides. 

* First, if the channel partner temporarily or permanently goes offline, you will not be able to make any payments. 
* Second, you have no alternative routing options if the channel partner decides to raise their routing fees. 
* Third, if the channel becomes imbalanced, you risk issues with making payments. 
* Fourth, only having one channel partner is less optimal from a privacy perspective. Notably, you cannot make any multi-path payments. 

Given the above considerations and budget permitting, even an ordinary Lightning user would do well to have channels with various nodes. Ideally, you would have at least some channels with highly-connected nodes. But some channels with less prominent nodes supports liquidity and decentralization within the network. It can also benefit your privacy. Generally speaking, the higher the channel capacity, the better in terms of liquidity management. So you should probably aim for channels with at least 300K to 1M satoshis in size, depending on your budget. 

You may have heard that some node clients for the Lightning Network now support **dual funding channels**, where both parties can commit funds to the channel at the same time. While this is indeed supported by Core Lightning, it is still an experimental feature at the time of writing, one that actually has to be configured through building the application from the source code. So we will only share how to obtain liquidity with **one-way funded channels** here.  

As stated above, there are many ways you can attempt to obtain liquidity for a fresh node. But one good place to start is by finding an exchange that accepts lightning deposits and is well-connected. If you open a channel with the exchange and then deposit lightning into your account, you will end up with a channel that has both outbound and inbound liquidity. A good service provider you can try this approach with is Kraken. You would start as follows:

* Ensure that you have an account with Kraken.
* You should already have them as a connected peer from following the instructions in the previous chapter. But if not, you can find their node address on **Lightning explorers** such as Amboss ([https://Amboss.space](https://Amboss.space)), **LightningNetwork+** ([https://lightningnetwork.plus/nodes](https://lightningnetwork.plus/nodes)), and **1ML** ([https://1ml.com/](https://1ml.com/)), and make a connection as explained in the previous chapter. 
* Read the online description of the conditions for opening a channel to Kraken (most recently, described at [https://support.kraken.com/hc/en-us/articles/9829766858260-Connecting-to-Kraken-s-Lightning-node](https://support.kraken.com/hc/en-us/articles/9829766858260-Connecting-to-Kraken-s-Lightning-node)). 

At the time of writing, the minimum channel capacity required for a channel to Kraken was 1 Million satoshis. As already stated, bigger channels are typically better, as long as the channel peer is well-connected and stable. If you are new to Lightning, you can aim to open a channel close to the minimum of 1M satoshis. If you are comfortable with it, you should consider opening a larger channel. If you are on a very limited budget and only have the means for one channel, then opening a single channel to Kraken would be a good option for at least making Lightning useable to you (though you will have increased risks of not finding routes). 

To open the channel to Kraken, you will first need funds desposited into your Lightning node. To create a new deposit address, execute the following instruction:

* **`ln-cli newaddr`**

You can now send bitcoin in a standard manner to this on-chain address. To see your funds, including those locked up in channels, you can execute **`$ ln-cli listfunds`**. 

Once your funds have confirmed, you can open the channel to Kraken. You can either open a channel with a specific amount or instruct your node to use all available on-chain funds for the channel. Both examples are offered below. The first instruction creates a channel with 1.5 Million satoshis of outbound balance, while the second just uses all available funds for the outbound balance. Both channels are public by default. 

* **`ln-cli fundchannel 02f1a8c87607f415c8f22c00593002775941dea48869ce23096af27b0cfdcc0b69@52.13.118.208:9735 1500000`**
* **`ln-cli fundchannel 02f1a8c87607f415c8f22c00593002775941dea48869ce23096af27b0cfdcc0b69@52.13.118.208:9735 all`**

Importantly, you need to be careful with the transaction fees for the funding transaction of a channel. The instructions above will automatically set the fees and those could be quite high. You can check information about your feerates in satoshis per virtual byte using the **`ln-cli feerates perkb`** instruction. You can set a lower feerate with the **`fundchannel`** command if necessary. 

Once you have started the process of opening a channel to Kraken, you can explore the state of the channel using the **`ln-cli listpeerchannels`** instruction. Among the information you should see the transaction ID of the funding transaction. You can search for the transaction in your local block explorer to see the number of confirmations. The channel will require three confirmations to be useable and six confirmations before it is announced to others.

Once the channel has been confirmed, you can deposit some bitcoin into your Kraken account (and perhaps exchange it for fiat money). The deposit redistributes the balance in the channel, so that you will also have outbound capacity. To be meaningful, you should deposit a sizeable amount. But it does not have to be exactly down the middle. For example, if you expect to make payments more than receive them, you can deposit 500K satoshis of a 1.5M satoshi channel. 

To make the deposit, you will need to request Kraken for a Lightning invoice. The **`invoice`** will be a very long string that includes information about the receiving node, the amount, and so on. You can, then, make the payment using the **`$ ln-cli pay [INVOICE]`** instruction. 


## Testing the hsm_secret backup

At this point, you should test your paper backup of the **`hsm_secret`** file. In case something goes wrong, your losses will still be limited. 

To start, move the **`hsm_secret`** file from the **`/btc-server/core-lightning/bitcoin`** directory into the **`/btc-server/core-lightning`** directory by executing the following command from within the **`/btc-server/core-lightning/bitcoin`** directory:

* **`sudo mv hsm_secret /btc-server/core-lightning`**

Now enter into the **`root`** account and execute the following instruction:

* **`$ cat > hsm_secret_hex <<HEX`**

From your paper backup, you can enter all the relevant information. You should proceed as follows:

* Write **`00: `** and follow it with the eight four-digit strings of hexadecimal code. Press **`enter `**.
* Write **`10: `** and follow it with the eight four-digit strings of hexadecimal code. Press **`enter `**.
* Write **`HEX`** and press enter.

You should now have the secret store in hexadecimal format in the file. You can now transform the hexadecimal code into binary and output it to a new **`hsm_secret`** file. Execute the following instruction:

* **`xxd -r hsm_secret_hex > hsm_secret`**

The **`hsm_secret`** file should only have read permissions for the **`lightningd`** account. Otherwise, Core Lightning might not start. To set the right permissions, execute the following instructions:

* **`$ chown lightningd hsm_secret`**
* **`$ chgrp lightningd hsm_secret`**
* **`$ chmod 400 hsm_secret`**

You can now remove the **`hsm_secret_hex`** file and move back into the **`administrator`** account. You should restart the service and check that it's working properly. Execute the following commands:

* **`sudo systemctl restart lightningd.service`**
* **`sudo systemctl status lightningd.service`**

The service should be active and running. To make sure that you have really restored everything properly, you should make another small payment to Kraken. If you made a mistake, you will not be able to sign a new commitment transaction and cannot make the payment. If the payment goes through, your backup is in order.

If you made the payment to Kraken successfully, you can move into the **`btc-server/core-lightning`** directory and delete the old version of the **`hsm_secret`** file.  


## Watchtowers

You should only open channels with other nodes that are reliable and trustworthy. That said, you cannot rely on trust as an assurance against other nodes cheating by publishing revoked commitment transactions to the Block Chain. This is particularly a risk because your node can go offline. For this reason, obtaining the services of one or more **watchtowers** is important. These help monitor revoked commitment transactions on your channels to ensure that they are not published to the Block Chain by mischievous peers. 

Importantly, watchtowers do not know the details of your revoked commitment transactions. There is an encryption scheme which ensures that they cannot see the data. Only when one of your peers acts mischievously are watchtowers able to decrypt the data and claim the balance of a channel on your behalf. So, as long as all peers behave honestly, a watchtower will never know the details of a payment channel. 

To set up a watchtower on your Core Lightning node, we will need to install a plugin. While plugins is part of what makes Core Lightning powerful, they can sometimes prove a hassle to set up properly. To make our lives easier, we have created a home directory for the **`lightningd`** account. Typically most plugins require a home directory by default and circumventing these settings can sometimes prove difficult.  

The plugin we will install here is from the **Eye of Satoshi** project. You can find their project on Github: [https://github.com/talaia-labs/rust-teos/tree/master/watchtower-plugin](https://github.com/talaia-labs/rust-teos/tree/master/watchtower-plugin). 

Rather than changing the ownership and permissions of particular files and directories all the time to setup the plugin, we will often resort to executing instructions with the **`lightningd`** service account without actually logging in. For this purpose, we will need the **`runuser`** application. Proceed as follows:

* Execute **`sudo apt install util-linux`** to make sure **`runuser`** is installed.
* The application should be installed within the **`sbin`** directory. This directory is probably not within your **`PATH`** variable. To add it, move into the **`/home/administrator`** directory and open the **`.profile`** file with a text editor. Add **`:/sbin`** to the end of the export statement, so that it reads: **`export PATH=$PATH:/btc-server/bitcoin-core/bin:/btc-server/core-lightning/bin:/sbin`**. Save and exit the file. Execute **`source .profile`** to ensure the settings are loaded. 

Next, we will need to import the PGP key used to sign the commits in the Eye of Satoshi repository. The key is one dedicated to automatically signing commits on the web interface of a GitHub enterprise server. The fingerprint is 5DE3 E050 9C47 EA3C F04A 42D3 4AEE 18F8 3AFD EB23. Set up the PGP key as follows:

* Import the key by executing **`$ sudo gpg --keyserver keyserver.ubuntu.com --search 5DE3E0509C47EA3CF04A42D34AEE18F83AFDEB23`**.
* Validate the key by executing **`$ sudo gpg --sign-key 5DE3E0509C47EA3CF04A42D34AEE18F83AFDEB23`**

If you now execute **`sudo gpg --list-keys`**, you should see it added to your keyring with the validation level of **`full`**. 

We will need to build the watchtower plugin from the source code, as there are no binaries available. The source code is written in Rust, so we will need to have the Rust Compiler installed. Execute the following instruction:

* **`$ sudo apt install rustc`**

We are now ready to clone the Github repository. Move into **`/home/lightningd`** directory. We will not use the **`runuser`** application yet, as this will cause issues with Github. So we will instead execute all git instructions from our **`administrator`** account and change the ownership and permissions manually aftwerwards. Clone the repository with the following instruction:

* **`$ sudo git clone https://github.com/talaia-labs/rust-teos.git`**

Move into the **`rust-teos`** directory and execute the following instruction:

* **`$ sudo git tag -n | sort -V`**

You will see a list of tagged snapshots for the codebase. You should verify the latest snapshot which counts as an official release (so not a release candidate, as indicated by the **`rc`** at the end of the version). At the time of writing, this is version **`v0.2.0`**. As the commits of the release are signed and not the tags, you will first have to find the hash of the commit. For version **`v0.2.0`**, the commit hash was **`43f9971`**. To verify the commit, execute the following instruction: 

* **`sudo git verify-commit 43f9971`**

If the signature is valid, you can move into the state of the codebase for the latest release as follows:

* **`$ sudo git checkout v0.2.0`**

Now move back into the **`/home/lightningd`** directory. We are not finished with our git instructions, so should set the right owners and permissions for the **`rust-teos`** directory. Execute the following instructions:

* **`$ sudo chown -R lightningd rust-teos`**
* **`$ sudo chgrp -R lightningd rust-teos`**
* **`$ sudo chmod 750 rust-teos`**

You can now move into the **`rust-teos`** directory. One dependency that you may be missing for building the plugin is **`rustfmt`**. You can install it as follows:

**`$ sudo apt install rustfmt`**

At this point, you are ready to build the plugin. Execute the following instruction:

* **`$ sudo runuser -u lightningd -- cargo install --locked --path watchtower-plugin`**

Executing the installation instructions from the **`lightningd`** account ensures that everything is stored within that account's home directory. You may at this point still run into issues due to dependencies that are missing. The output should indicate what you are missing. Install any missing dependencies with your package manager and just run the original instruction again (that is, **`$ sudo runuser -u lightningd -- cargo install --locked --path watchtower-plugin`**). 

After the installation has finished, move into your **`/home/lightningd`** directory. If you execute **`$ ls -a -l`**, you will see a **`.cargo`** directory where your Rust binaries are stored. Move into the **`.cargo/bin`** directory. You should now see a binary called the **`watchtower-client`**.

To make the plugin work, we first need to create a directory for Core Lightning to load plugins. Proceed as follows:

* Move into the **`/btc-server/core-lightning`** directory.
* Execute **`sudo runuser -u lightningd -- mkdir plugins`**

We can now store the watchtower-client plugin directly inside this directory and Lightnind will load it on startup. Storing a symbolic link, however, is generally more convenient with updates. You just have to update the executable in the home directory of the **`lightningd`** account without additionally moving it to the **`plugins`** directory. To create a symbolic link, execute the following instruction:

* **`sudo runuser -u lightningd -- ln -s /home/lightningd/.cargo/bin/watchtower-client ./plugins/watchtower-client`**

You should now see the symobolic link placed within the **`plugins`** directory. You should set the permissions for executing the plugin as follows: **`$ sudo chmod 750 watchtower-client`**.

At this point, you can restart the **`lightningd`** service. Execute **`sudo systemctl restart lightningd.service`**. After restarting, you should see a **`.watchtower`** directory appear within your **`/home/lightningd`** directory. This will contain all the data for your watchtower client. You can also check that the plugin is running successfully with the **`$ ln-cli plugin list`** instruction. You should see your plugin described as **`"active": "true"`**. 

We can now start to leverage watchtowers for improving the security of your node. Preferably, you will add several watchtowers for redundancy. We will illustrate how to use a watchtower leveraging the watchtower service run by **frznode.com**. You can find the information in the appropriate section of their website ([ https://www.frznode.com/teos.html]( https://www.frznode.com/teos.html)). To subscribe to their free watchtower service run on an onion address, you merely need to execute the following command: 

* **`$ ln-cli registertower 032a4ff7dd37ae01de848c3029c91253e0f30ee9038dcbcd08d6338bbbcb11339a@devrzfhfi2xay3ur5fpiualkqq6sedkaykvwsaf4in5rprohsoxfqoad.onion:9814`**
 
If all goes well, you should receive an output with five data points: **`user_id`**, **`available_slots`**, **`subscription_start`**, **`subscription_expiry`**, and **`subscription_signature`**. The **`user_id`** refers to your own public key. The **`available_slots`** indicates how many appointments the client can send to the tower per month. The client will send an appointment each time a commitment transaction in one of their channels is updated. The **`subscription_start`** should be the current block height, while the **`subscription_expiry`** states when the subscription ends. It typically should be far into the future.

You can check that the service is running properly by executing **`ln-cli listtowers`**. You should see the tower output with a **`"status": "reachable"`**.

There are a number of other watchtowers around that offer free subscriptions. A good starting point is the list that is maintained within the repository ([https://github.com/talaia-labs/rust-teos/discussions/158](https://github.com/talaia-labs/rust-teos/discussions/158)). While some of these will be offline or refuse a connection, many of them work. You should at this point add several more, so that you have four or five watchtower services.

At the end of the day, these services are all for free. You can consider paying a service provider for a watchtower service, as this may give some more certainty. You could try the clearnet service from **frznode.com** ([https://microlancer.io/service/view/1413](https://microlancer.io/service/view/1413)) or some other service provider. 

You should also consider running the Eye of Satoshi server. You can, then, offer your services to others. Some good watchtower operators are likely also to be more keen on offering your their services if you offer the same service in return. LightningNetwork+ actually has a watchtower swap service that you can utilize, once you have your own server up and running. 


## Expanding liquidity

Your budget allowing, you should open further Lightning channels at this point. 

One option is to set up a few more channels with exchanges that support Lightning deposits as we did in the section *Obtaining Initial Liquidity* above. You can also attempt to open up channels via some well-connected **Lightning service providers** such as Bitrefill's **Thor** or Voltage's **Flow**. These will also obtain you inbound liquidity quickly, though these service providers are not always offering public channels. 

But there are also other great alternatives. With one or more public channels in place, your node will start to become known to others in the Lightning network. Given sufficient time, you should be able to find your node on platforms such as LightningNetwork+ and Amboss. And both of these platforms offer good opportunities for further liquidity.

We are particularly keen on the the Lightning Pool and Lightning Swap services offered by LightningNetwork+. To make use of these services, you should first claim your node on their website simply by offering a signature for your node's public key ID. The procedure is as follows: 

* Find your node by the public key ID or your alias in the search bar. (Again, it might take at least a few hours after you first opened a public channel before LightningNetwork+ knows about your node.)
* Click on your node's profile and select **`Claim your node`**.
* Copy the message and execute **`$ ln-cli signmessage [MESSAGE]`** within the terminal to your server.
* Copy the **`zbase`** signature, paste it into the empty box on the LightningNetwork+ website, and click **`Claim`**.

You should now see your node details as an administrator. You can come back to this website any time and log in using the signature method above.

The Swap service allows three, four, or five nodes to open channels of equal size in a ring to one another. Participating in a Swap, therefore, will obtain you and inbound and outbound channel of the same size, but to different peers. Most swaps come with conditions, primarily on the number of channels and the **node's capacity**(i.e., the sum of all inbound and outbound liquidity). But you can also start a swap on your own and set your own conditions. 

The LightningNetwork+ platform also offers their Pool service. If you move to page for the service, you can see a list of offers for opening channels. In return for opening channels with outbound liquidity to others, you will receive Pool Liquidity credits. These credits allow you to advertise your own node within the Liquidity Pool, so that others can open channels to you. While it takes a substantial amount of money and time to become significant with regards to credit accumulation, it may be a good avenue to inbound liquidity in the long-run with enough commitment on your side. 

As an alternative to these services from the LightningNetwork+, you can also utilize Amboss' **Magma** service. This is basically a marketplace where you can pay others to open a channel to you. You can utilize it once you have claimed your node in a similar fashion as on LightningNetwork+. Importantly, you should pay attention to the conditions of the offers, particularly with regards to the guarantees on how long the channel will stay open.

Besides connecting to well-known nodes and using professional services such as Pool and Magma, you can also reach out to the owners of other Lightning nodes and make direct agreements with them. Many people within the Bitcoin community run a lightning node and can be found within Telegram groups and other fora. As long as you can give them some assurance of stability, they will often be willing to open a channel with you. 

So there is really a constellations of options with regards to obtaining liquidity. And it really depends on your budget, purposes, and risk tolerance for how actively you want to pursue liquidity on your node. The above considerations should give you some ideas. If you're still struggling at this point, here are some ideas of what you might do with particular budgets that would seem reasonable in most circumstances: 

* **500K satoshis**: Open one or two channels to very well-connected nodes. You can check LightningNetwork+, Amboss, and other such websites for node rankings. At least one node should be a Lightning-supporting exchange to also obtain some inbound liquidity. In case of two channels, split the capacity equally between them. 
* **2M satoshis**: Open three channels to very well-connected nodes, at least one being an exchange for obtaining inbound liquidity. Participate in one swap on the LightningNetwork+ platform. Split the budget equally among all the channels.  
* **10M satoshis**: Open four or five channels to very well-connected nodes, at least two being exchanges for obtaining inbound liquidity. Participate in various swaps with less well-known nodes. Make all the channels roughly around 1M satoshis in capacity. 

However you proceed on the basis of these considerations, it is important to remember that your node has a hot wallet, both for the on-chain and channel funds. This comes with certain risks. However much of your funds you dedicate to the Lightning node wallet, you should in any case ***limit it to money that is needed for channels***. Any Bitcoin savings should be kept offline in a hardware wallet and managed with a wallet application such as Sparrow or Specter (see *Chapter 15*). 


## Python plugins

You can see a list of community-curated plugins for Core Lightning here: [https://github.com/lightningd/plugins](https://github.com/lightningd/plugins). Many of the plugins for Core Lightning are written in Python, rather than Rust. In this section, we will just for illustrative purposes install one of these Python plugins. The plugin is called **`Summary`** and it can show you some basic information about your node. 

To start, we will clone the entire repository of plugins and set the right ownership and permissions. Move into your **`/home/lightningd`** directory and execute:

* **`sudo git clone https://github.com/lightningd/plugins.git`**
* **`$ sudo chown -R lightningd plugins`**
* **`$ sudo chgrp -R lightningd plugins`**
* **`$ sudo chmod 750 plugins`**

Next, you will need to have the package manager for Python3 installed. It is called **Pip**. Execute the following instruction:

* **`sudo apt install python3-pip`**

You can now move into the **`plugins/summary`** directory. To install the requirements, execute the following instruction:

**`sudo runuser -u lightningd -- pip3 install --break-system-packages --user -r requirements.txt`**

This ensures any of the requirements are installed within your **`lightningd`** home directory with the right ownership and permissions. The **`break-system-packages`** option avoids an **`externally-managed-environment`** error. 

The required packages were installed in the **`/home/lightningd/.local/lib/python3.11/site-packages`** and **`/home/lightningd/.local/bin`** directories. Given that we log into our system with the **`administrator`** account, the **`PATH`** variable for the **`lightningd`** account has the administrator's settings. So we have to add these two paths for the requirements.  

* Move into the **`/home/administrator`** directory.
* Open the **`.profile`** file with a text editor.
* Add **`/home/lightningd/.local/lib/python3.11/site-packages`** and **`/home/lightningd/.local/bin`** to the **`PATH`** variable.
* Save and exit the file.
* Execute **`source .profile`**.

After your system has restarted, move back into the **`/home/lightningd/plugins/summary`** directory. You can create a symbolic link to the **`summary.py`** file in the **`plugins`** directory for Lightningd. Execute the following instructions:

**`sudo runuser -u lightningd -- cp summary.py /btc-server/core-lightning/plugins`**

You can now restart the Lightningd service and check on the status.

**`sudo systemctl restart lightningd.service`**
**`sudo systemctl status lightningd.service`**

If you have any issues, it is likely due ownership and permissions settings, or the setting of the **`PATH`** variable. Once you have the plugin working properly, you can use it by executing the following instruction:

* **`ln-cli summary`**