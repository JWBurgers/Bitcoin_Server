# Fulcrum

For Bitcoind, we configured the option **`txindex=1`**. This ensures that all Bitcoin transactions are indexed by their transaction ID. It allows you to quickly retrieve any transactions in the Block Chain and memory pool. To test it out, execute the following instruction:

* **`$ bitcoin-cli getrawtransaction b18cc4b8510b4b26fa8c7e6ec9b44f8c795d803800eacd26023021e81def0827`**

Without the transaction index option, you cannot retrieve this result. By default, you can only search for transactions currently in the memory pool on the basis of their ID, not those in the Block Chain. 

The **Fulcrum** electrum server will additionally index Bitcoin addresses on the Block Chain.<sup>[1](#footnote1)</sup> Using this server application will, therefore, allow you to retrieve the balances and other information from known addresses on the Block Chain. Many wallets have better performance if connected to this type of electrum server instead of Bitcoind. You will also need this electrum server to obtain a fully functional local Block Chain explorer. 

We tested two common Electrum implementations for our purposes: **Fulcrum** and **Electrs**. While Fulcrum does require more setup time, it far outperformed Electrs. Hence why we adopt it here.<sup>[2](#footnote2)</sup> Other options are the "spesmilo" fork of **ElectrumX** and **Electrs-esplora**. You could also try and install the former if you want to test a different option than Fulcrum. The Electrs-esplora application is very heavy duty and may far exceed the capacity of the hardware you selected for your system in *Chapter 2*. 

Alternatively, there is also a more lightweight application, namely the **Electrum personal server**. However, this application only allows one connected wallet at a time. In addition, it only stores the data for any wallet provided to them, which is an additional privacy concern. Fulcrum allows you to connect multiple wallets and indexes all the address data on the Block Chain. 


## Installation

The Fulcrum application has binary releases, so we will install the latest binary. If you wish, you can also go through the steps of building the source code on the project's Github page [https://github.com/cculianu/Fulcrum](https://github.com/cculianu/Fulcrum). Your experiences in *Chapter 8* should be sufficient to help you this time around. 

To start, let's import the PGP key for Calin Culianu, the project maintainer, to your GPG key ring. His public key should be as follows:

* D465 135F 97D0 047E 18E9 9DC3 2181 0A54 2031 C02C

You can verify that this public key indeed is coupled to his identity via the Ubuntu keyserver and potentially other sources. Once you have established that the public key and identity indeed belong together, you can add it to your GPG key ring as follows:

* **`$ sudo gpg --keyserver keyserver.ubuntu.com --search D465135F97D0047E18E99DC321810A542031C02C`**

If you now execute **`$ sudo gpg --list-keys`**, you should see the public key listed. To change the validity level, you need to sign it with the signing private key you created in *Chapter 8*. Proceed as follows: 

•	Execute **`$ sudo gpg --sign-key D465135F97D0047E18E99DC321810A542031C02C`**.
•	Click **`yes`** to confirm.
•	To finalize the signing process, enter the secure password for your signing key.

Next, navigate to the releases section of the Github project page. Find the latest release, which in our case was version **`1.9.7`**. There are three types of Linux packages, all are in the **`tar.gz`** archive format. Such packages are also known as **tarballs**. The three packages are used as follows:

* **`Fulcrum-1.9.7-x86_64-linux.tar.gz`**: This contains a Fulcrum package for any Linux operating system which works with a processor that has an x86 architecture. 
* **`Fulcrum-1.9.7-x86_64-linux-ub16.tar.gz`**: This is a package for any older Linux operating system that has an x86 architecture.
* **`Fulcrum-1.9.7-arm64-linux.tar.gz`**: This contains a Fulcrum package for any Linux operating system which works with a processor that has an ARM architecture. 

You will probably need either the first or third package, depending on the architecture of your processor. We will assume you are using the first package for the instructions, but adapt them accordingly if necessary. 

To download the x86_64 Linux package, proceed as follows:

* Right-click on the link to the package. Select **`Copy Link Location`** from the menu. This is the exact url from which we will download the tarball.  
* Travel to the **`/btc-server`** directory on your server.   
* You can download the tarball into the directory with the wget application. It should be installed on your system, but download it with the APT if necessary (i.e., **`$ sudo apt install wget`**).
* Right-click on the terminal window to your Bitcoin server with your mouse. This should paste the url we found for the tarball. 
* Add the necessary text to obtain the following instruction: **`$ sudo wget https://github.com/cculianu/Fulcrum/releases/download/v1.9.7/Fulcrum-1.9.7-x86_64-linux.tar.gz`**. Execute the instruction.

You should now see the tarball within the directory. At this point, we can verify the authenticity. Here we will not do that with Git as we are only installing a binary. Instead, the general approach with software verification in theses cases is as follows:

1. Obtain the official package hash and a signature over that hash. 
2. Make a hash of the package you downloaded and compare the result to the official package hash. They should be the same.
3. Verify that the digital signature over the official package hash indeed was made with the maintainer’s public key.   

We will now make these steps specifically for Fulcrum. Among the latest release download links on the Github page, you should see a file that contains a hash of every released package. In our case, this was **`Fulcrum-1.9.7-shasums.txt`**. In addition, you should see a file that contains a signature over this shasums file. In our case, this was **`Fulcrum-1.9.7-shasums.txt.asc`**. Download both into your **`/btc-server`** directory.

* **`$ sudo wget https://github.com/cculianu/Fulcrum/releases/download/v1.9.7/Fulcrum-1.9.7-shasums.txt`**
* **`$ sudo wget https://github.com/cculianu/Fulcrum/releases/download/v1.9.7/Fulcrum-1.9.7-shasums.txt.asc`**

Output the content of the **`shasums.txt`** file with the **`sudo tail Fulcrum-1.9.7-shasums.txt`** instruction. You can view the hash of the tarball you downloaded with the following instruction: **`$ sudo sha256sum Fulcrum-1.9.7-x86_64-linux.tar.gz`**. The two should be the same. ***If not, there is a discrepancy between your downloaded version and the official version of the package. In that case, do not use the tarball and find out why there is a discrepancy.***

The hash on its own does not mean much. An attacker could have compromised your Fulcrum package and altered the official hash in the release file you downloaded accordingly. What an attacker practically cannot do, however, is spoof a valid signature with Calin's private key over the shasums file. So as a last step, you need to verify that the signature over the shasums is indeed valid. Execute the following instruction:  

* **`$ sudo gpg --verify Fulcrum-1.9.7-shasums.txt.asc`**

You should receive a confirmation message. ***If this is not the case, then you cannot trust the package you downloaded and need to figure out what is going on.*** After the verification process, you can delete both the shasums and signature files. 

We can now unpack the tarball. Execute the following instruction:

* **`$ sudo tar -x -z -f Fulcrum-1.9.7-x86_64-linux.tar.gz`**

The option **`x`** indicates to the tar application that it should extract files. The option **`z`** indicates that it needs to decompress the archive first. The option **`f`** indicates that it should extract the files from a file, not a tape drive.  

After executing the instruction above, Fulcrum should install. When finished, you should have a directory dedicated to Fulcrum. Let's organize a bit with the following instructions:

* **`$ sudo mv Fulcrum-1.9.7-x86_64-linux fulcrum**`**
* **`$ sudo rm -R Fulcrum-1.9.7-x86_64-linux.tar.gz`**

If you enter into the **`fulcrum`** directory, you should see two binaries: **`Fulcrum`** and **`FulcrumAdmin`**. Change the names to be lower case, so that we have consistency with our other applications as follows:

* **`$ sudo mv Fulcrum fulcrum**`**
* **`$ sudo mv FulcrumAdmin fulcrum-admin**`**

The **`fulcrum`** binary is the server application, while **`fulcrum-admin`** is a command line tool to query the server. We will not really use this command line tool. As a last step, we will need to create a data directory for fulcrum. Create one in the **`fulcrum`** directory by executing the following instruction:

* **`$ sudo mkdir fulcrum-data`**


## Service account and permissions

To run Fulcrum as a service, we will need to create the server account and configure the right permissions for the relevant directories and files. Many of the instructions mimic those for Bitcoind in *Chapter 9*, so we will be much more brief here. 

To start, let's create the service account. We will set the name as "fulcrum". Execute the following instruction: 

* **`$ sudo useradd --shell /usr/sbin/nologin --system -M fulcrum`**

All these options have the same meaning as in *Chapter 9*. 

The user owner of the **`fulcrum`** directory should be the **`fulcrum`** account and the group owner should be the **`fulcrum`** group. We also need to set the correct permissions to the directory. Within the **`/btc-server`** directory, execute the following instructions:

* **`$ sudo chown -R fulcrum fulcrum`**
* **`$ sudo chgrp -R fulcrum fulcrum`**
* **`$ sudo chmod -R 750 fulcrum`**

All these options also have the same meaning as in *Chapter 9*. To verify all the instructions you just passed were incorporated successfully, you can execute **`$ ls -l`** within the **`btc-server`** directory. 

Finally, to ensure that our **`administrator`** account can navigate all the directories and files that have the **`fulcrum`** group as the group owner, we need to add the account to the group. Proceed as follows:

* **`$ sudo usermod -a -G fulcrum administrator`**

You can verify that this instruction worked by executing **`$ groups administrator`**. To ensure the new group settings are loaded for your **`administrator`**, just switch to the **`root`** account and then back into the **`administrator`** account.


## The service file

We will now create the service file for the fulcrum service. To start, proceed as follows:

* Enter into the **`/etc/systemd/system`** directory
* Execute the following instruction: **`$ sudo touch fulcrum.service`**

The specifications we recommend for the service file are offered below. There is no sample service file in the respository, so we created one. Go ahead an copy the specifications below into the **`fulcrum.service`** file. As we mentioned before, these service files are sensitive with regards to formatting (e.g., capitalization, spacing, tabs). Make sure that everything is exactly as below on your system.  

`[Unit]
Description=Fulcrum
After=bitcoind.service

[Service]
ExecStart=/btc-server/fulcrum/fulcrum /etc/fulcrum/fulcrum.conf
Type=simple

Restart=always
TimeoutSec=60
RestartSec=60

PermissionsStartOnly=true
ExecStartPre=/bin/chgrp fulcrum /etc/fulcrum

User=fulcrum
Group=fulcrum

RuntimeDirectory=fulcrum
RuntimeDirectoryMode=0710

ConfigurationDirectory=fulcrum
ConfigurationDirectoryMode=0710

StateDirectory=fulcrum
StateDirectoryMode=0710

PrivateTmp=true

ProtectSystem=full

ProtectHome=true

NoNewPrivileges=true

PrivateDevices=true

MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target`

Many of these options you should recognize from the service file for Bitcoind, so we will not explain them again here. A few key differences are as follows:

* The fulcrum service should not start until Bitcoind has successfully started up. 
* The configuration file is an argument when executing the binary, not an option. We will place it in the **`/etc/fulcrum`** directory later. 
* The Fulcrum server does not have child processes, so has a type of **`simple`**.
* The option **`TimeoutSec`** specifies how long to wait when starting or stopping the service before marking it as failed or forcefully killing it. This combines the **`TimeoutStartSec`** and **`TimeoutStopSec`** options we used for Bitcoind. 
* Setting **`Restart=always`** ensures that the service will restart regardless of the circumstances under which it exited. Setting it to **`on-failure`** as we did for Bitcoind is more restrictive. 


## RPC credentials

We need access credentials for fulcrum, so that it can make requests of Bitcoind's RPC API. Proceed as follows:

* Travel to the **`bitcoin`** directory in the **`adiministrator`** account's home directory.
* Move into the **`/share/rpauth`** directory.
* Execute the following instruction: **`$ sudo python3 ./rpcauth.py fulcrum`**.  
* After executing the instruction, the first line in the output will have the following format: `rpcauth=fulcrum:[hex string]`. 
* At the bottom of the output is your password.    

Place the credentials in your Bitcoind configuration file as follows: 

- Open your **`bitcoin.conf`** file in the **`/etc/bitcoind`** directory. 
- Below the RPC credentials for your **`administrator`** account, place those for your **`fulcrum`** account. Make sure you are entering the hash of your password and not your actual password. It will not work otherwise. The relevant section of your Bitcoind configuration file should look as follows:
    -`# [rpc]`
    -`# Username and hashed password combinations for JSON-RPC connections.`  
    - `rpcauth=administrator:[hex string]`
    - `rpcauth=fulcrum:[hex string]`
- Save and exit the file.
- Copy your password temporarily into a text document so that you don't lose it. We will need it in the next section.


## Creating the configuration file

We now need to create the configuration file in the **`/etc/fulcrum`** directory. Proceed as follows:

* Move into the **`/etc`** directory
* Execute **`$ sudo mkdir fulcrum`**
* Move into the **`/fulcrum`** directory and execute **`sudo touch fulcrum.conf`**

If you look in the **`/btc-server/fulcrum`** directory, you will see a sample configuration file. You can copy it into your configuration file directory as follows:

* **`$ sudo mv /btc-server/fulcrum/fulcrum-example-config.conf /etc/fulcrum/fulcrum.conf`**

The descriptions offered in this configuration file are good. So we suggest to leave the template as it is and just adapt those options which are necessary. Proceed as follows:

* Under the **`Basic`** options ensure that the data directory is set as follows: **`datadir = /btc-server/fulcrum/fulcrum-data`**.
* Under the **`Basic`** options, the Bitcoind RPA API IP address and listening port should already be set correctly: **`bitcoind = 127.0.0.1:8332`**.
* Under the **`Basic`** options, ensure you set the RPC user as follows: **`rpcuser = fulcrum`**.
* Under the **`Basic`** options, ensure you set the RPC password to the result from the previous section as follows: **`rpcpassword = [password]`**.
* Under the **`Basic`** options, ensure you uncomment the listening port option and change the defaults to the following: **`tcp = 127.0.0.1:50001`**. We will only listen internally for communications to fulcrum for now. 
* We will not set any of the SSL port settings. Also we will not generate any SSL private key or certificate with the service. We will address the security issues in the next chapter.
* Under the **`Basic`** options, ensure you configure the **`Admin RPC bind settings`**. You should uncomment the last line, so that you see **`admin = 127.0.0.1:8000`**. This will allow you to use the command line tool. 
* Under the **`Peer discovery and public server options`**, change the **`peer discovery`** setting. It should be set as follows: **`peering = false`**. This will avoid that fulcrum will attempt to learn about other servers. 

As the Fulcrum server is set to only accept requests from the local machine, we cannot connect any wallets and applications from our managing computer directly. Instead, we will connect those through a proxy server, namely NGINX, in the next chapter. This allows any local connections to occur over an encrypted connection, which is much more secure. As we will not connect directly to Fulcrum from other computers, we do not need to make any changes to our firewall.  


## Start the Fulcrum service

We are now ready to start the Fulcrum service. Execute * **`sudo systemctl restart bitcoind`** as we have changed the configurations to Bitcoind. After a minute or so, execute the following instructions:

* **`sudo systemctl enable fulcrum`**
* **`sudo systemctl start fulcrum`**

If you now execute **`sudo systemctl status fulcrum`**, you should see that the service is enabled and running. In case you run into any issues, you can execute **`$ sudo journalctl -xeu fulcrum.service`**. Once the service is running, it will start building the index for the address data. To see the relevant log data from the systemd journal on a running basis, you can also execute the following instruction:

* **`sudo journalctl -f -u fulcrum`**

The indexation process will take a long time. In our case, it took about two days with fairly solid hardware. Just leave the output running until you see lines that express an up to date block height. The wait is well worth it in terms of performance. 

Once the building of the index has been completed, you can check that the Fulcrum service is listening properly by executing the following commands:

* **`sudo apt install net-tools`**
* **`netstat -n -t -l`**

To help keep track of the port numbers of our services, **write down the address and port on which Fulcrum is listening (i.e., "127.0.0.1:50001") on your server record**. 

To use the **`fulcrum-admin`** tool, just enter into the **`btc-server/fulcrum`** directory and execute **`$ ./fulcrum-admin -p 8000 [command]`**. You can check the available commands by executing **`$ ./fulcrum-admin --help`**. This tool is useful in case you are running a public Fulcrum server. As we are not, you will probably not use this tool. 


## Notes

<a name="footnote1">1</a>. The Github project is located here:https: //github.com/cculianu/Fulcrum. 

<a name="footnote2">2</a>. For a good blog post on the comparison, see Sparrow Wallet, "Server performance", February 1 (2022), available at: https://sparrowwallet.com/docs/server-performance.html. Jameson Lopp had good results also with ElectrumX in a review. See Jameson Lopp, "Electrum server performance report", November 19 (2022), available at https://blog.keys.casa/electrum-server-performance-report-2022/.  


The build documentation is available here: https://github.com/romanz/electrs/blob/master/doc/install.md. 

<a name="footnote3">3</a>. The configuration documentation is available here: https://github.com/romanz/electrs/blob/master/doc/config.md. 