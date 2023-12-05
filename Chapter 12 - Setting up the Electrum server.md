# Setting up the Electrum server

The Bitcoind configuration option "txindex=1" ensures that all Bitcoin transactions are indexed by their transaction ID. This means that you can quickly retrieve any transactions in the Block Chain and memory pool using their ID. To test it out, execute the following instruction:

* **`$ bitcoin-cli getrawtransaction b18cc4b8510b4b26fa8c7e6ec9b44f8c795d803800eacd26023021e81def0827`**

Without the transaction index option, you cannot retrieve this result. By default, you can only search for transactions currently in the memory pool on the basis of their ID, not those in the Block Chain. 

The **Electrs** electrum server will additionally index Bitcoin addresses.<sup>[1](#footnote1)</sup> Using this server application will, therefore, allow you to quickly retrieve the balances and other information of certain addresses. Many wallets have better performance if connected to this electrum server instead of Bitcoind. You will also need this electrum server to obtain a fully functional local Block Chain explorer. 

Electrs is intended for small-scale, private usage. You should not run this electrum server publicly. If that is your goal, you can look into more heavyweight electrum server applications such as the "spesmilo" fork of **ElectrumX**, **Fulcrum**, or **Electrs-esplora**. Note that particularly the Electrs-esplora application is very heavy duty and may far exceed the capacity of the hardware you selected for your system in *Chapter 2*. While Electrs technically does not require a transaction index, these more heavy-duty server applications definitely do.

Alternatively, there is also a more lightweight application, namely the **Electrum personal server**. However, this application only allows one connected wallet at a time. In addition, these server applications store the data for any wallet provided to them, which is an additional privacy concern. Electrs allows you to connect multiple wallets and indexes all the data. 

Let's first install the required building tools and dependencies. 


## Building tools and dependencies

Electrs is a Rust application. To install Rust, execute the following instruction:

* **`sudo apt install cargo`**

The Electrs application requires various build tools. Some of these would have been installed already during our installation of Bitcoin Core. But to be certain you have all the tools, execute the following instruction:

* **`sudo apt install clang cmake build-essential`**

The Github project's installation instructions offers two ways of building the application, namely via static linking to **`librocksdb`** or dynamic linking.<sup>[2](#footnote2)</sup> As we will likely not need to move the application to another system and prefer easy updates, we will use dynamic linking. In that case, we can just regularly install librocksdb, a C++ library, using our APT.

* **`sudo apt install librocksdb-dev=7.8.3-2`**


## Build Electrs

Now move into the administrator account's home directory and download the Electrs codebase with the following instruction:

* **`git clone https://github.com/romanz/electrs`**

To verify the latest release, you will need to add the PGP key for Roman Zeyde, the project maintainer, to your GPG key ring. His key should be as follows:

* 15C8 C357 4AE4 F1E2 5F3F 35C5 87CA E5FA 4691 7CBB

You can verify that this public key indeed is coupled to his identity via Roman's Github profile page, Twitter page, and Ubuntu keyserver. Once you have established that the public key and identity indeed belong together, you can add it to your GPG key ring as follows:

* **`$ sudo gpg --keyserver keyserver.ubuntu.com --search 15C8C3574AE4F1E25F3F35C587CAE5FA46917CBB`**

If you now execute **`$ sudo gpg --list-keys`**, you should see the keypair listed. To change the validity level of the public key, you need to sign it with the signing key you created in *Chapter 8*. Proceed as follows: 

•	Execute **`$ sudo gpg --sign-key 15C8C3574AE4F1E25F3F35C587CAE5FA46917CBB`**
•	Click “yes” to confirm
•	To finalize the signing process, enter the secure password for your signing key

To see all the versions of the Git repository, execute the following instruction:

* **`$ sudo git tag -n | sort -V`**

Find the latest release of the codebase that was not specifically labeled as a candidate release ("rc"). In our case, this was v0.10.1. You can verify that this release has a legitimate signature as follows:

* **`$ sudo git tag -v v0.10.1`** 

You should see that the release commit has a valid signature from Roman Zeyde. Your current view of the Electrs repository will be in its latest state, so we should now move the state of the repository into v0.10.1. Execute the following command:

* **`$ sudo git checkout v0.10.1`**

You can now build Electrs with the following command from within the Electrs directory:

* **`$ sudo ROCKSDB_INCLUDE_DIR=/usr/include ROCKSDB_LIB_DIR=/usr/lib cargo build --locked --release`**

Let's move the Electrs binary to our **`btc-server`** directory and create a directory for our data, just as we did for our Bitcoin Core binaries. Proceed as follows:

* In the *`btc-server`* directory, execute **`$ sudo mkdir electrs`**
* Move into the electrs directory, execute **`$ sudo mkdir bin electrs-data`**
* To move the binary into the new **`bin`** directory, execute `**$ sudo cp ~/electrs/target/release/electrs /btc-server/electrs/bin/`**

At a later point, we will be sure to also point all the storage of the Electrs data into the **`eletrs-data directory`**. In this way, all key binaries and data will be stored within the dedicated directory for our Bitcoin server.


## Service account and permissions

To run Electrs as a service, we will need to create the server account and configure the right permissions for the relevant directories and files. Many of the instructions mimic those for Bitcoind in *Chapter 9*, so we will be much more brief here. 

To start, let's create the service account. We will set the name as "electrs". Execute the following instruction: 

* **`$ sudo useradd --shell /usr/sbin/nologin --system -M electrs`**

All these options have the same meaning as in *Chapter 9*. The **`shell`** option will prevent logging into the account. The **`system`** option ensures that the account recieves both a user and a group ID that is lower than 1000. The last option specifies that the new account has no home directory. 

The account owner of the "electrs" directory should be the electrs account and the group owner should be the electrs group. We also need to set the read and write permissions. Within the **`btc-server`** directory, execute the following instructions:

* **`$ sudo chown -R electrs electrs`**
* **`$ sudo chgrp -R electrs electrs`**
* **`$ sudo chmod -R 750 electrs`**

The 750 instruction sets the privileges for the account owner, group owner, and everyone else respectively. To verify all the instructions you just passed were incorporated successfully, you can execute **`$ ls -l`** within the **`btc-server`** directory. 

To ensure that our administrator account can navigate all the directories that have the electrs group as the owner, we need to add it to the group. Proceed as follows:

* **`$ sudo usermod -a -G electrs administrator`**

You can verify that this instruction worked by executing **`$ groups administrator`**. 


## Creating the service file

We will now create the service file for the electrs service. To start, proceed as follows:

* Enter into the **`/etc/systemd/system`** directory
* Execute the following instruction: **`$ sudo touch electrs.service>`**

The specifications we would recommend for the service file are offered below. There is a sample service file in the respository.<sup>[3](#footnote3)</sup> It's typically good practice to follow templates that are included with the software. But as our context is quite different, in particular because Electrs will be run from a dedicated service account, our specifications look quite different.  

Go ahead an copy the specifications below into the electrs.service file. As we mentioned before, these service files are sensitive with regards to formatting (e.g., capitalization, spacing, tabs). Make sure that everything is exactly as below on your system.  

`[Unit]`
`Description=Electrs`
`After=bitcoind.service`

`[Service]`
`ExecStart=/btc-server/electrs/bin/electrs --conf /etc/electrs/config.toml --log-filters INFO`
`Type=simple`

`KillMode=process`

`Restart=always`
`TimeoutSec=60`
`RestartSec=60`

`PermissionsStartOnly=true`
`ExecStartPre=/bin/chgrp electrs /etc/electrs`

`User=electrs`
`Group=electrs`

`RuntimeDirectory=electrs`
`RuntimeDirectoryMode=0710`

`ConfigurationDirectory=electrs`
`ConfigurationDirectoryMode=0710`

`StateDirectory=electrs`
`StateDirectoryMode=0710`

`Environment="RUST_BACKTRACE=1"`

`PrivateTmp=true`

`ProtectSystem=full`

`NoNewPrivileges=true`

`PrivateDevices=true`

`MemoryDenyWriteExecute=true`

`[Install]`
`WantedBy=multi-user.target`

Many of these options you should recognize from the service file for Bitcoind. A few key differences are as follows:

* The electrs.service should not start until bitcoind has successfully started up. 
* The default mode of the server is as a daemon. We must specify the location of the configuration file in the **`/etc/electrs`** directory. 
* The Electrs server does not have child processes, so has a type of **`simple`**.
* The option **`TimeoutSec`** specifies how long to wait when starting or stopping the service before marking it as failed or forcefully killing it. This combines the **`TimeoutStartSec`** and **`TimeoutStopSec`** options we used for Bitcoind. 
* Setting **`Restart=always`** ensures that the service will restart regardless of the circumstances under which it exited. Setting it to **`on-failure`** as we did for Bitcoind is more restrictive. 
* The **`ProtectHome=True`** setting in the service file for Bitcoind has been removed. Electrs will not start with this setting and throw a permission error with regards to accessing a default configuration file location.    


## RPC credentials

We need access credentials for Electrs, so that it can make requests of Bitcoind's RPC API. Proceed as follows:

* Travel to the **`bitcoin`** directory in the adiministrator account's home directory
* Move to the **`/share/rpauth`** directory
* Execute the following instruction: **`$ sudo python3 ./rpcauth.py electrs`**.  
* After executing the instruction, the first line in the output will have the following format: `rpcauth=electrs:[hex string]`. 
* At the bottom of the output is your password.    

Place the credentials in your Bitcoind configuration file. Proceed as follows: 

- Open your **`bitcoin.conf`** file in the **`/etc/bitcoind`** directory. 
- Below the RPC credentials for your administrator account, place those for your electrs account. Make sure you are entering the hash of your password and not your actual password. It will not work otherwise. The relevant section of your Bitcoind configuration file should look as follows:
    -`# [rpc]`
    -`# Username and hashed password combinations for JSON-RPC connections.`  
    - `rpcauth=administrator:[hex string]`
    - `rpcauth=electrs:[hex string]`
- Save and exit the file.
- Copy your password temporarily into a text document so that you don't lose it. We will need it in the next section.


## Creating the configuration file

Now we need to create the configuration file in the **`/etc/electrs`** directory. Proceed as follows:

* Move into the **`/etc`** directory
* Execute **`$ sudo mkdir electrs`**
* Move into the **`/electrs`** directory and execute **`sudo touch config.toml`**

You can enter the following configuration options into the file. A description is given for each of the configuration options. 

`# This indicates the network for which electrs is working. By default, the network is "bitcoin", which refers to the Bitcoin mainnet.`
`network="bitcoin"`

`# This specificies the location of your bitcoin data directory.`
`daemon_dir="/btc-server/bitcoin-core/bitcoin-data"`

`# The address on which Bitcoind's RPC API listens for requests. The port is standardly 8332. In order for Bitcoind to listen on this address, you will need to set "server=1" in the Bitcoind configuration file.`
`daemon_rpc_addr = "127.0.0.1:8332"`

`# This is the listening address of Bitcoind with regards to inbound connections. The port is standardly 8333.`
`daemon_p2p_addr = "0.0.0.0:8333"`

`# The Bitcoind RPC credentials for Electrs.`
`auth="electrs:[PASSWORD]"`

`# The address on which the electrs server API should listen for requests. By default, it will only accept requests from the machine locally via 127.0.0.1. Connecting wallets and other applications not on the machine is best done through a proxy, and letting the API only listen on the local machine is the most secure.`
`electrum_rpc_addr = "127.0.0.1:50001"`

`# Directory where the electrs index should be stored. It should have at least 70GB of free space.`
`db_dir = "/btc-server/electrs/electrs-data"`

As the Electrs server is set to only accept requests from the local machine, we cannot connect any wallets and applications from our managing computer directly. Instead, we will connect those through a proxy server, namely NGINX. This allows any local connections to occur over an encrypted connection, which is much more secure. As we will not connect directly to Electrs from other computers, we do not need to make any changes to our firewall.  

You can check that the Electrs service is listening properly by executing the following commands:

* **`sudo apt install net-tools`**
* **`sudo netstat -n -t -l`**

To help keep track of our the port numbers of our services, **write down the address and port on which Electrs is listening (i.e., "127.0.0.1:50001) on your server record**. 

## Start the Electrs service

We are now ready to start the Electrs service. Execute the following instruction:

* **`sudo systemctl enable electrs`**
* **`sudo systemctl start electrs`**

If you now execute **`sudo systemctl status electrs`**, you should see that the service is enabled and running. In case you run into any issues, you can execute **`$ sudo journalctl -xeu electrs.service`**. Once the service is running, it will take some time to synchronize with the Block Chain and index all the address data. To see just the relevant log data from the systemd journal, execute the following instruction:

* **`sudo journalctl -f -u electrs`**


## Notes

<a name="footnote1">1</a>. The Github project is located here: https://github.com/romanz/electrs. 

<a name="footnote2">2</a>. The build documentation is available here: https://github.com/romanz/electrs/blob/master/doc/install.md. 

<a name="footnote3">3</a>. The configuration documentation is available here: https://github.com/romanz/electrs/blob/master/doc/config.md. 