# BTC RPC Explorer

In this chapter, we will setup the BTC RPC Explorer server. It will connect to both Bitcoind and Fulcrum for obtaining the data to be served. We will also set up NGINX as a reverse proxy, so that any connections from outside our managing computer are made safely with TLS.  

Let's start with the installation of the application.


# Installation

Unlike Bitcoind and Electrs, BTC RPC explorer is written in an interpreted language, namely **JavaScript**. This means that there is no binary for running the application, only a startup script. To start, we will need to have both **nodejs** and **npm** installed. The former is an execution engine for back-end JavaScript applications. The latter is a package manager for nodejs applications. To install both applications with the APT and confirm the installations, execute the following instructions:

* **`sudo apt update`**
* **`sudo apt upgrade`**
* **`sudo apt install nodejs`**
* **`node -v`**
* **`sudo apt install npm`**
* **`npm -v`**

We will clone the latest release from the project's Github repository. To verify the latest release, we will need to add the PGP key for Dan Janosik, the project maintainer, to our GPG key ring. His public key is as follows:

* 4D84 1E6E 6B1B 68EB FAB4 A9E6 70C0 B166 321C 0AF8

You can verify that this public key indeed is coupled to his identity via Keybase, the Ubuntu keyserver, and perhaps elsewhere. Once you have established that the public key and identity indeed belong together, you can add it to your GPG key ring as follows:

* **`$ sudo gpg --keyserver keyserver.ubuntu.com --search 4D841E6E6B1B68EBFAB4A9E670C0B166321C0AF8`**

If you now execute **`$ sudo gpg --list-keys`**, you should see the public key listed. To change the validity level, you need to sign it with the private signing key you created in *Chapter 8*. Proceed as follows: 

* Execute **`$ sudo gpg --sign-key 4D841E6E6B1B68EBFAB4A9E670C0B166321C0AF8`**
* Click “yes” to confirm
* To finalize the signing process, enter the secure password for your signing key

Now that you have imported Dan's public key, you can clone the Github repository and verify the latest release. Proceed as follows:

* Change into the **`/btc-server`** directory
* Execute **`git clone https://github.com/janoside/btc-rpc-explorer`
* Move into the **`btc-rpc-explorer`** directory
* To see all the releases, execute **`$ sudo git tag -n | sort -V`**
* Find the latest release. In our case, this was **`v3.4.0`**. 
* For Bitcoind and Electrs we verified the releases by checking the signatures on the tags. However, in this case, a signature was only made on the commit for the release, not the actual release tag. If you click on the latest release in the Github repository, you will see the commit hash listed below the tag. For **`v3.4.0`**, the commit hash is **`bfc9f97`**. You can verify the commit with the following instruction: * **`$ sudo git verify-commit bfc9f97`** 

If the signature checks out, you can move the state of the repository to that of the latest release.

* **`$ sudo git checkout v3.4.0`**

As BTC RPC Explorer is written in an interpreted language, all the source code needs to be available at execution time to the startup script. Hence the decision to install all of the repository directly to our dedicated drive for Bitcoin-related applications and data. 

In order to run the application, we will still need to install all the dependencies. These are specified in the **`package.json`** file. To install them, execute the following instruction from within the **`btc-rpc-explorer`** directory: 

* **`$ sudo npm install`**

This should take some time. When finished, you should see all the dependencies install in the **`node_modules`** directory. 


## Service account and permissions

To run BTC RPC Explorer as a service, we will first need to create the service account and configure the right permissions for the relevant directories and files. To start, let's create the service account. We will set the name as **`btc-rpc-explorer`**. Execute the following instruction: 

* **`$ sudo useradd --shell /usr/sbin/nologin --system -M btc-rpc-explorer`**

The account owner of the **`btc-rpc-explorer`** directory should be the **`btc-rpc-explorer`** account and the group owner should be the **`btc-rpc-explorer`** group. We also need to set the read and write permissions. Within the **`btc-server`** directory, execute the following instructions:

* **`$ sudo chown -R btc-rpc-explorer btc-rpc-explorer`**
* **`$ sudo chgrp -R btc-rpc-explorer btc-rpc-explorer`**
* **`$ sudo chmod -R 750 btc-rpc-explorer`**

The 750 instruction sets the privileges for the account owner, group owner, and everyone else respectively. To verify all the instructions you just passed were incorporated successfully, you can execute **`$ ls -l`** within the **`btc-server`** directory. 

To ensure that our administrator account can navigate all the directories that have the electrs group as the owner, we need to add it to the group. Proceed as follows:

* **`$ sudo usermod -a -G btc-rpc-explorer administrator`**

You can verify that this instruction worked by executing **`$ groups administrator`**. You may have to reboot for these settings to take effect.


## Creating the service file

We will now create the service file for the **`btc-rpc-explorer`** service. To start, proceed as follows:

* Enter into the **`/etc/systemd/system`** directory
* Execute the following instruction: **`$ sudo touch explorer.service>`**

To write the service file, we will need to know the location of **`npm`**, as using **`$ npm start`** is a convenient way to run the application. To see the location, execute **`$ whereis npm`**. The first offered location is what you need and probably **`/usr/bin/npm`**.

The full specifications we would recommend for the service file are offered below. There was no sample service file in the respository, so we created one on our own. Go ahead an copy the specifications below into the explorer.service file. As we mentioned before, these service files are sensitive with regards to formatting (e.g., capitalization, spacing, tabs). Make sure that everything is exactly as below on your system.  

`[Unit]`
`Description=BTC RPC Explorer`
`After=bitcoind.service fulcrum.service`

`[Service]`
`WorkingDirectory=/btc-server/btc-rpc-explorer`
`ExecStart=/usr/bin/npm start` 
`Type=simple`

`Restart=always`
`TimeoutSec=60`
`RestartSec=60`

`PermissionsStartOnly=true`
`ExecStartPre=/bin/chgrp btc-rpc-explorer /etc/btc-rpc-explorer`

`User=btc-rpc-explorer`
`Group=btc-rpc-explorer`

`RuntimeDirectory=btc-rpc-explorer`
`RuntimeDirectoryMode=0710`

`StateDirectory=btc-rpc-explorer`
`StateDirectoryMode=0710`

`PrivateTmp=true`

`ProtectSystem=full`

`ProtectHome=true`

`NoNewPrivileges=true`

`PrivateDevices=true`

`[Install]`
`WantedBy=multi-user.target`

The settings should be familiar from service files we have created in previous chapters. Two main settings to note. First, in this service file setting the **`WorkingDirectory`** is paramount. The **`$ npm start`** command only works within the **`/btc-server/btc-rpc-explorer`** directory. Second, from the standard hardening options, the **`MemoryDenyWriteExecute=true`** has been removed. The explorer service would otherwise probably shut down immediately due to memory errors this option produces. 


## Configurations

We now need to create the configuration file in the usual manner. Proceed as follows:

* Move into the **`etc`** directory and execute **`$ sudo mkdir /btc-rpc-explorer`**
* Move into the **`btc-rpc-explorer`** directory and execute **`sudo touch .env`**
* Move into the **`etc`** directory and execute **`$ sudo chown -R btc-rpc-explorer btc-rpc-explorer`**, **`$ sudo chgrp -R btc-rpc-explorer btc-rpc-explorer`**, and **`$ sudo chmod -R 750 btc-rpc-explorer`**

As with Bitcoin-cli and Fulcrum, we will allow the explorer service to communicate with Bitcoind via a set of credentials. To create them, proceed as follows: 

* Travel to the **`bitcoin`** directory in the administrator account's home directory
* Move to the **`/share/rpauth`** directory
* Execute the following instruction: **`$ sudo python3 ./rpcauth.py btc-rpc-explorer`**.  
* After executing the instruction, the first line in the output will have the following format: `rpcauth=btc-rpc-explorer:[hex string]`. 
* At the bottom of the output is your password.    

Place the credentials in your Bitcoind configuration file. Proceed as follows: 

- Open your **`bitcoin.conf`** file in the **`/etc/bitcoind`** directory. 
- Below the RPC credentials for your administrator account, place those for your electrs account. Make sure you are entering the hash of your password and not your actual password. It will not work otherwise. The relevant section of your Bitcoind configuration file should look as follows:
    -`# [rpc]`
    -`# Username and hashed password combinations for JSON-RPC connections.`  
    - `rpcauth=administrator:[hex string]`
    - `rpcauth=electrs:[hex string]`
    - `rpctauth=btc-rpc-explorer:[hex string]`
- Save and exit the file.

Copy your password temporarily into a text document so that you don't lose it. 

We are now ready to populate the BTC RPC Explorer configuration file. Move into the **`/etc/btc-rpc-explorer`** directory and open the **`.env`** file. The project's Github repository offers a sample configuration file. You can view it on your own here: [https://github.com/janoside/btc-rpc-explorer/blob/master/.env-sample](https://github.com/janoside/btc-rpc-explorer/blob/master/.env-sample). You can consult this file if you ever need to use more advanced options in the future. For now, you can just enter the following simplified and customized configurations:  

`# Listening IP address and port. Port 3002 is the default port for the block explorer.`
`BTCEXP_HOST=127.0.0.1`
`BTCEXP_PORT=3002`

`# Bitcoin RPC Credentials.`
`BTCEXP_BITCOIND_HOST=127.0.0.1`
`BTCEXP_BITCOIND_PORT=8332`
`BTCEXP_BITCOIND_USER=btc-rpc-explorer`
`BTCEXP_BITCOIND_PASS=[password]`

`# Sets the type of server and address for displaying transaction lists and balances`
`BTCEXP_ADDRESS_API=electrum`
`BTCEXP_ELECTRUM_SERVERS=tcp://127.0.0.1:50001`

`# Setting to false ensures the availability of resource-intensive features such as UTXO set querying.`
`BTCEXP_SLOW_DEVICE_MODE=false`

The default port for the block explorer is 3002, which we will just leave in place. Importantly, the block explorer utilizes both data from Bitcoind and Fulcrum to be fully functional. Importantly, the connection is over the same machine so does not need to run over NGINX. It can just connect to them directly. Hence, be sure that the **`BTCEXP_ELECTRUM_SERVERS`** option uses the **`tcp`** label, and not the **`tls`** label as in the Github project's sample configuration file.


## NGINX reverse proxy

Just as Fulcrum, communications to the BTC RPC block explorer are not encrypted and unauthenticated by default. To make access from a remote system secure, we will also have NGINX act as a reverse proxy (hence, why the **`BTCEXP_HOST`** option above is set to "127.0.0.1" and not "0.0.0.0"). 

Move into the **`/etc/nginx`** directory and open the **`nginx.conf`** file. Within the **`stream`** context, enter the following proxy server details after those for Fulcrum.

`##`
`#BTC-RPC-Explorer proxy server`
`##`

`upstream btc-rpc-explorer {`
`server 127.0.0.1:3002;`
`}`
`server {`
`listen 3003 ssl;`
`proxy_pass btc-rpc-explorer;`

`ssl_certificate /etc/ssl/certs/nginx.crt;`
`ssl_certificate_key /etc/ssl/private/nginx.key;`

`ssl_session_cache shared:SSL:5m;`
`ssl_session_timeout 4h;`
`ssl_protocols TLSv1.2 TLSv1.3;`
`ssl_prefer_server_ciphers on;`
`}`

We need to allow traffic to the proxy port 3003. Execute the following instruction:

* **`$ sudo ufw allow from 192.168.2.200 to any port 3003`**

At this point, make sure you **write down the public listening address for the explorer (i.e., "0.0.0.0:3003"), as well as the internal listening address (i.e., "127.0.0.1:3002").


## Launching the block explorer service

At this point, we have changed quite a bit of information in various services. Easiest is probably just to enable the explorer service and then reboot our system. Execute the following instructions:

* **`$ sudo systemctl enable explorer`**
* **`$ sudo reboot`**

Check with **`$ sudo systemctl status nginx`** to see the NGINX service is running properly and troubleshoot if necessary. Do the same for the **`explorer`** service. If you have any issues, you will need to use the status output and the **`$ sudo journalctl -xeu [service]`** command to help you.  

Once all the services are running smoothly, you can as a final check now also execute **`sudo netstat -n -t -l`**. You should see both the internal and public addresses for the explorer service listed.


## Using the block explorer

You should now be able to use the block explorer. On your managing computer, open a web browser and enter **`https://192.168.2.150:3003`** or **`https://btc-server.local:3003`**in the address bar. Depending on your browser, you may receive a warning and have to click "continue" or something similar. But once you continue, you should now see the interface for the block explorer.

You can try import your self-signed root certificate for NGINX into a browser to get rid off these security warnings. To start, let’s copy the certificate onto your managing computer. 

* On your managing computer, create a new textfile by right-clicking on your desktop, selecting “New”, and selecting “Text document”. (This is a text document in a simple format.)
* Rename the text document **`nginx.crt`**. While **`.pem`** and `**.crt`** are both acceptable standards for root certificates, Windows seems to have a preference for **`.crt`** certificates. You should see the icon for the file change into that for a root certificate.
* Open **`nginx.crt`** with a text editor.
* On your server terminal, open the **`nginx.crt`** file with a text editor from the **`/etc/ssl/certs/`** directory. 
* Copy and paste the contents of the file into the **`nginx.crt`** file you have open on your managing computer. 
* Save and exit the **`nginx.crt`** file opn your managing computer. 

Next, you can import the certificate into one of your browsers. For Firefox, for example, proceed as follows:

* Open the Firefox browser and locate the **`Settings`** option from the menu button on the right side of your window. 
* Select the **`Privacy and Security`** option on the left side of the window.
* Towards the bottom, you should see a section called **`Certificates`**. Click on the **`View certificates`** button.
* Under the **`Authorities`** tab, click on the **`Import…`** button and select your **`nginx.crt`** on the managing computer. 
* On the dialog box, make sure the **`Trust this CA to identify websites`** option is checked.
* Press **`Ok`**.

If you now scroll down to the **`Bitcoin Server`** section, you should see your root certificate listed. At this point, you will still receive the warnings about traveling to your explorer page. To add a permanent exception, proceed as follows:

* On the **`Servers`** tab, click **`Add exception`**.
* Enter **`https://btc-server.local:3003`** or **`https://192.168.2.150:3003`**. You should enter the address in whatever format you find the most convenient way to connect to your server, as you can only make an exception for one or the other. 
* Click **`Get certificate`**.
* Click on **`Confirm security exception`**.

The exception should now be permanently stored within your Firefox browser. You will no longer receive any warnings for visiting the explorer page, as long as you use the address format of the exception. 

For Google, adding exceptions seems more complicated. You can import a certificate as follows:

** Move to **`Settings`**
** Select **`Privacy and security`**
** Select **`Security`**
** Select **`Manage certificates`**
** Move to the **`Trusted root certificates`** tab and click on **`Import`**. Select your root certificate and move through the Wizard. Ensure that you select **`Place all certificates in the following store`** and write out **`Trusted Root Certification Authorities`**. 
** After moving through the wizard, fill in the following address in the address bar: **`chrome://flags/#allow-insecure-localhost`**. 
** Make sure you enable **`Allow invalid certificates for resources loaded from localhost.`**.

If you now relaunch Chrome, you should be able to connect to the explorer without warnings if you use your server's IP address and port. Using the **`btc-server.local`** identifier will unfortunately still produce the warnings.

After importing the certificate into one or more of your browsers, just leave it on the desktop for now. 