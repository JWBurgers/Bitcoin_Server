# RTL and the Zeus wallet

So far, we have been working with our Lightning node from the command line. A graphical interface is, of course, much more convenient. Here we will first set up Ride the Lightning (RTL) on your managing computer, a popular web-based application. It is comprehensive and has an intuitive user interface. You will be able to view key data analytics about your ow node and the network. You will also be able to perform some common tasks such as making and receiving payments, signing messages, and opening and closing channels.  

Furthermore, we will set up the Zeus wallet on your mobile phone. This will allow you to make payments on the go with Lightning through your own server. 

Let's start by installing a REST plugin for Core Lightning, necessary for running RTL.


## REST server

To start, we will need to set up a REST server for Core Lightning, so that other applications can make API calls to our Lightning server. We will not have to start from scratch. If you move into the **`/btc-server/core-lightning/libexec/c-lightning/plugins`** directory, you will see a directory called **`clnrest`**. It contains all the files for setting up the plugin. 

To install the requirements for the plugin, move into the **`clnrest`** directory and execute the following instruction:

* **`sudo runuser -u lightningd -- pip3 install --break-system-packages --user -r requirements.txt`**

The necessary requirements will be installed into the **`/home/lightningd/.local/bin`** and **`/home/lightningd/.local/lib/python3.11/site-packages`** directories. These are already included within your **`PATH`** variable, despite any warnings you might receive to the contrary. 

To make sure the plugin loads properly when starting Core Lightning, you will have to set a listening address for the REST server. Move into the **`/etc/lightningd`** directory and open **`config`**. At the bottom of your Core Lightning configuration file, enter the following details:

`## REST API`
`# Sets the listening port for the REST server. This option must be set for the REST server plugin to load. The default port is 3010.`
`clnrest-port=3010`
`# Sets the listening address for the REST server. The default address is 127.0.0.1.`
`clnrest-host=127.0.0.1`

At this point the plugin should be able to load. Move into the **`/btc-server/core-lightning`** directory. Execute the following instructions:

**`sudo systemctl restart lightningd.service`**
**`sudo systemctl status lightningd.service`**

In case the service fails to start, you can troubleshoot in the standard manner.

To check that the REST server is working properly, you can execute **`sudo tail -n 500 log | grep rest`**. It may take some time. But at some point you should see a message **`plugin-clnrest.py: REST server running at https://127.0.0.1:3010`**. If you execute **`$ sudo netstat -ntl`**, you should also see the server listening on port **`3010`**. Finally, if you execute **`ln-cli plugin list`**, you should see the plugin described as **`"active": true`**. 


## RTL installation

The RTL application is a NodeJS application. Just as with the BTC Explorer application, all the source code needs to be available at execution time for the startup script. So we will clone all the source code directly within our **`/btc-server`** directory. 

To begin, let's import the PGP key for Suheb Saubyk, the project maintainer, to your GPG key ring. His key should have the following fingerprint:

* 3E9B D443 6C28 8039 CA82 7A92 00C9 E2BC 2E45 666F

You can verify that this public key indeed is coupled to his identity via the Ubuntu keyserver, Keybase, and other sources. Once you have verified the public key, add it to your GPG key ring and validate it with the following instructions:

* **`$ sudo gpg --keyserver keyserver.ubuntu.com --search 3E9BD4436C288039CA827A9200C9E2BC2E45666F`**
* **`$ sudo gpg --sign-key 3E9BD4436C288039CA827A9200C9E2BC2E45666F`**

If you now execute **`$ sudo gpg --list-keys`**, you should see the keypair listed and validated. 

Now that you have imported the maintainer's public key, you can clone the Github repository and verify the latest release. Proceed as follows:

* Change into the **`/btc-server`** directory.
* Execute **`sudo git clone https://github.com/Ride-The-Lightning/RTL.git`**.
* Move into the **`RTL`** directory.
* To see all the releases, execute **`$ sudo git tag -n | sort -V`**.
* Find the latest release. In our case, this was **`v0.15.0`**. 
* The tag has been signed. You can verify it with the following command: **`sudo git verify-tag v0.15.0`**.

If the signature checks out, you can move into the state of the codebase for **`v0.15.0`** by executing **`$ sudo git checkout v0.15.0`**.

We are now ready to install the dependencies for RTL. As with installing Core Lightning plugins, it is not so obvious how to bypass default settings that require access to a home directory with RTL. In addition, we noticed that if you make the installation from the **`administrator`** account and then change the ownership and permissions of the relevant directories and files, RTL was failing because it was attempting to access the home directory for the administrator. 

So before installation, we first need to create an RTL service account with a home directory, and implement all the necessary ownership and permissions settings. Proceed as follows:

* Create the service account by executing **`$ sudo useradd --shell /usr/sbin/nologin --system -m rtl`**. You should see a home directory appear for the account **`rtl`**.
* Move into the **`/btc-server`** directory.
* To adapt the ownership settings for the **`RTL`** directory, execute **`$ sudo chown -R rtl RTL`** and **`$ sudo chgrp -R rtl RTL`**. 
* To set the right permission levels, execute **`$ sudo chmod -R 750 RTL`**.
* To ensure that our **`administrator`** account can navigate the **`RTL`** directory properly, execute **`$ sudo usermod -a -G rtl administrator`**.

For the last setting to take effect, you should log into the **`root`** account and then back into the **`administrator`** account. Once you have done this, you can execute the installation instruction from within the **`/btc-server/RTL`** directory. 

* **`$ sudo runuser -u rtl -- npm install --omit-dev`**

You may receive some warnings, but the installation should be successful. When finished, you should see all the dependencies installed within the **`node_modules`** directory. 


## The service file

We will now create the service file for RTL. To start, proceed as follows:

* Enter into the **`/etc/systemd/system`** directory
* Execute the following instruction: **`$ sudo touch rtl.service`**

There is a service file template available in the project's Github repository, but we will deviate from it somewhat to include some standard hardening options. The service file content below has largely been adapted from the service file for your BTC Explorer. Copy the contents exactly as below on your system.  

`[Unit]
Description=Ride the Lightning Daemon
After=lightningd.service

[Service]
ExecStart=/usr/bin/node /btc-server/RTL/rtl.js
Type=simple

Restart=always
TimeoutSec=60
RestartSec=60

PermissionsStartOnly=true

User=rtl
Group=rtl

RuntimeDirectory=rtl
RuntimeDirectoryMode=0710

StateDirectory=rtl
StateDirectoryMode=0710

PrivateTmp=true

ProtectSystem=full

ProtectHome=true

NoNewPrivileges=true

PrivateDevices=true

[Install]
WantedBy=multi-user.target`

A few matters of note on the service file. The **`MemoryDenyWriteExecute=true`** hardening option produces an error with the RTL service, so is not included. We also could not find a way to easily set a custom path for the RTL configuration file. So we will use the default location, namely **`/btc-server/RTL`**. Hence, you do not need to ensure access to a configuration file within the **`/etc`** directory.  


## Configurations

We can now create the configuration file for RTL. We could not find a way to set a custom path, so will use the default **`/btc-server/RTL`** directory. 

To start, we will need to ensure that the RTL server can make API calls to Core Lightning's REST API. The API cannot be accessed by a username and password as with Bitcoin Core's RPC API. Instead, Core Lighting utilizes **`runes`** for access. These are basically files with access credentials and permission settings. To create a rune for RTL, execute the following instruction:

* **`ln-cli createrune`**

Make sure you copy the contents of the **`rune`** field in the output. It should be a long string that looks similar to the strings for Bitcoin RPC API passwords. 

Next, move into the **`/btc-server/RTL`** directory. Here we will make a file that makes the rune available to the **`rtl`** account. Proceed as follows:

* Create a rune file for the **`rtl`** account by executing **`$ sudo runuser -u rtl -- touch lightning_rune`**
* Open the file with a text editor and enter the following content: **`LIGHTNING_RUNE="<rune>"`**. In case you lost the rune, you can always reveal it again by executing **`ln-cli showrunes`**. Save and exit the file.
* Ensure the right permissions by executing **`$ sudo chmod 710 lightning_rune`**.

Within the **`/btc-server/RTL`** directory, there is a sample configuration file. Renaming it, we can ensure that the RTL application will read it on startup. Execute the following instruction:

* **`$ sudo mv Sample-RTL-Config.json RTL-Config.json`**

Open the file with a text editor. Ensure that it has the content see below. You will need to fill in the **`"multiPass"`** and **`"lnNode"`** fields, as explained below.   

`{
  "multiPass": "<password>",
  "port": "3000",
  "host": "127.0.0.1",
  "defaultNodeIndex": 1,
  "dbDirectoryPath": "/btc-server/RTL",
  "SSO": {
    "rtlSSO": 0,
    "rtlCookiePath": "",
    "logoutRedirectLink": ""
  },
  "nodes": [
    {
      "index": 1,
      "lnNode": "<your node name>",
      "lnImplementation": "CLN",
      "Authentication": {
        "runePath": "/btc-server/RTL/lightning_rune",
        "configPath": "/etc/lightningd"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "PURPLE",
        "bitcoindConfigPath": "/etc/bitcoind",
        "logLevel": "DEBUG",
        "fiatConversion": false,
        "unannouncedChannels": false,
        "lnServerUrl": "https://127.0.0.1:3010"
        }
    }
  ]
}`

You can find an explanation of all these configuration options in the repository. We will explain some of the key options below. 

* **`"multiPass"`**: From a managing computer, you will be required to log in with the password set here. The password stores as a hash, not a plaintext. In this case, you can choose the same password as for your **`administrator`** account without too much added security risks. **You should note this password on your server record**.
* **`"port"`**: This sets the port that the RTL server will listen on. The default port is **`3000`**. **You should note this port on your server record**.
* **`"host"`**: This sets the host that the RTL server will listen on. The default is all network interfaces. This should be on **`"127.0.0.1"`** in your case.  
* **`"SS"`**: These settings are for configuring single sign-ins. By setting "rtlSSO" to **`0'**`, this feature is turned off.
* **`"index"`**: Identifies the node via a number.
* **`"lnNode"`**: A name that uniquely describes the Lightning node for the interface. It makes sense to insert your Core Lightning alias here.
* **`"lnImplementation"`**: Specifies the node implementation. The options are **`"CLN"`** for Core Lightning, **`"LND"`** for LND, and **`"ECL"`** for Eclair.
* **`"Authentication"`**: Specifies the absolute path of the rune file that allows access to Core Lightning, as well as the path of Core Lightning's configuration file.
* **`"userPersona"`**: This tailors the data offered on the interface. Options are **`"MERCHANT"`** or **`"OPERATOR"`**.
* **`"logLevel"`**: This sets the information level in the log. **`"DEBUG"`** is most verbose. Allowed options are **`"ERROR"`**, **`"WARN"`**, **`"INFO"`**, and **`"DEBUG"`**.
* **`"unannouncedChannels"`**: This sets whether private channels are default with channel openings. The options are **`true`** or **`false`**.  
* **`"lnServerUrl"`**: This sets the address on which the Core Lightning Node's REST API is listening for requests. The default is port **`3010`**. 


## NGINX reverse proxy

We currently only have access to RTL from our local system. To make it securely accessible to our managing computer, we can have NGINX act as a reverse proxy (hence, why the **`"host"`** option above is set to **`"127.0.0.1"`** and not left to the default of all network interfaces). Any connection to NGINX from our managing computer will use the TLS standard. 

To set it up, move into the **`/etc/nginx`** directory and open the **`nginx.conf`** file. Within the **`stream`** context, enter the following proxy server details after those for BTC Explorer.

`##`
`# Ride the Lightning proxy server`
`##`
`
upstream rtl {
server 127.0.0.1:3000;
}
server {
listen 3001 ssl;
proxy_pass rtl;

ssl_certificate /etc/ssl/certs/nginx.crt;
ssl_certificate_key /etc/ssl/private/nginx-key.pem;

ssl_session_cache shared:SSL:5m;
ssl_session_timeout 4h;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
}
`
We need to allow traffic to the proxy port 3001 from your managing computer. Execute the following instruction:

* **`$ sudo ufw allow from 192.168.2.200 to any port 3001`**

Make sure you **write down the public listening address for the RTL application (i.e., **`0.0.0.0:3001`**).

We can now restart the service and check the status. Execute the following instructions:

* **`sudo systemctl restart nginx`**
* **`sudo systemctl status nginx`**

In case there any issues, you can troubleshoot in the standard ways. If all was set correctly, you should now also see your computer listening on **`0.0.0.0:3001`** if you execute **`netstat -ntl`**. 


## Start the service

At this point, we are ready to start the RTL service and access it from our managing computer. To start, execute the following instructions:

* **`$ sudo systemctl start rtl.service`**
* **`$ sudo systemctl enable rtl.service`**

The server will take a while to start up. After a short break, you should check on the status by executing **`sudo systemctl status rtl.service`**. You may have to check on it several times before seeing the following line:

* **`INFO: RTL => Server is up and running, please open the UI at http://127.0.0.1:3000 or your proxy configured url`**

If you are not obtaining this output, even after 5 or 10 minutes, something is probably wrong. You should troubleshoot in the standard ways to address the issue. Depending on what the issue is, you may already have a log file that you can consult in the **`/home/rtl/.npm/_logs`** directory. 

You should now be able to use the application. On your managing computer, open a web browser and enter **`https://192.168.2.150:3001`** or **`https://btc-server.local:3001`**in the address bar. Ignore the warnings if there are any and continue to the site. You should now be able to log in with the password you set. It will take some time for the web application to retrieve all the data about your node. Once logged in, you can view data and analytics about your node and the network, and perform standard tasks such as making and receiving payments, opening and closing channels, and changing configurations to Core Lightning.  

Just as for the BTC Explorer, you can try to circumvent the warnings in your browser. For Firefox, proceed as follows:

* Open the Firefox browser and locate the **`Settings`** option from the menu button on the right side of your window. 
* Select the **`Privacy and Security`** option on the left side of the window.
* Towards the bottom, you should see the **`Certificates`** section. Click on the **`View certificates`** button.
* On the **`Servers`** tab, click **`Add exception`**.
* Enter **`https://btc-server.local:3001`** or **`https://192.168.2.150:3001`**. You should probably use the same format as for the BTC Explorer.  
* Click **`Get certificate`**.
* Click on **`Confirm security exception`**.

The exception should now be permanently stored within your Firefox browser.  

While you can use RTL for payments on your managing computer, it is a somewhat slow experience. That is mainly because the application has to retrieve all of your node's information each time you use the service. One alternative option is to use the Zeus mobile wallet, which we will set up next, for your payments. 

Alternatively, you can search for a dedicated Lightning wallet on your managing computer. The main issue here is that you might not find any great options. While the **Electrum** wallet also supports connections to a custodial Core Lightning node, it seems impossible to connect Electrum over TLS to your Fulcrum server with a self-signed certificate. While **Zap** has generally been a solid wallet, it only works with **LND** and the project is now no longer maintained. This lack of options is illustrative of the limitations that Lightning still faces.  


## The Zeus wallet

We will now set up Lightning payments on-the-go on your mobile phone. As with desktop lightning wallets, your options are very limited if you want to connect to your own financial infrastructure. We will here work with probably the most popular choice in this regard: the **Zeus** wallet. It is available for Android and iPhone. In addition, it is also available on F-Droid, a repository for FOSS applications. So you can also install Zeus on more privacy-friendly Android alternatives such as Calyx OS. 

To make a connection between Zeus and our Core Lightning server, we will need to have a REST API in place. Unfortunately, we currently cannot use the Core REST server plugin we set up for RTL. Instead, Zeus only works with an older version of a REST API written for Core Lightning in NodeJS. This version of the API is also a project from the creators of Ride the Lightning. Whats also a hassle is that we will have to run this API as a service. Running it as a plugin would cause conflicts with the Core Lightning REST API we set up for RTL. 

We will set up the API first. We will call it the NodeJS REST API for clarity. We will, then, install Zeus and connect it in various ways to this API.


## The NodeJS REST API installation

To set up the NodeJS REST API, let's first clone the repository and verify the latest release. Proceed as follows:

* Change into the **`/btc-server`** directory
* Execute **`sudo git clone https://github.com/Ride-The-Lightning/c-lightning-REST.git`**
* Execute **`sudo mv c-lightning-REST cln-nodejs-rest`**
* Move into the **`cln-nodejs-rest`** directory
* To see all the releases, execute **`$ sudo git tag -n | sort -V`**
* Find the latest release. In our case, this was **`v0.10.7`**. 
* The tag has been signed. You can verify it with the following command: **`sudo git verify-tag v0.10.7`**

If the signature checks out, you can move the codebase state to the latest release by executing **`$ sudo git checkout v0.10.7`**.

The NodeJS REST server will require access to certain parts of the **`core-lightning`** directory. Rather than create a dedicated service account, we can just run this as a service with the **`lightningd`** account. So we just have to set the right ownership and permission settings to the main directory. Proceed as follows:

* Move into the **`/btc-server`** directory.
* To adapt the ownership settings, execute **`$ sudo chown -R lightningd cln-nodejs-rest`** and **`$ sudo chgrp -R lightningd cln-nodejs-rest`**. 
* To set the right permission levels, execute **`$ sudo chmod -R 750 cln-nodejs-rest`**.

With these settings in place, we can install the dependencies for the NodeJS REST server. Execute the following instruction from within the **`cln-nodejs-rest`** directory:

* **`$ sudo runuser -u lightningd -- npm install`**

You may receive some warnings, but the installation should be successful. When finished, you should see all the dependencies installed in the **`node_modules`** directory. 


## The service file

We will now create the service file. To start, proceed as follows:

* Enter into the **`/etc/systemd/system`** directory
* Execute the following instruction: **`$ sudo touch cln-nodejs-rest.service`**

There is a service file template available in the project's repository, but we will deviate from it somewhat to include some standard hardening options. Please copy the contents exactly as below on your system.  

`[Unit]
Description=Core Lightning NodeJS REST server
After=lightningd.service

[Service]
ExecStart=/usr/bin/node /btc-server/cln-nodejs-rest/cl-rest.js
Type=simple

Restart=always
TimeoutSec=60
RestartSec=60

PermissionsStartOnly=true

User=lightningd
Group=lightningd

RuntimeDirectory=cln-nodejs-rest
RuntimeDirectoryMode=0710

StateDirectory=cln-nodejs-rest
StateDirectoryMode=0710

PrivateTmp=true

ProtectSystem=full

NoNewPrivileges=true

PrivateDevices=true

[Install]
WantedBy=multi-user.target`

A few matters of note. The **`MemoryDenyWriteExecute=true`** hardening option produces an error with the service, so is not included in the service file. We also could not find a way to easily set a custom path for the configuration file, so decided to use the default location.   


## Tor connection

The easiest way to connect your Zeus wallet to Core Lightning is by setting up a Tor hidden service. 

To start, we need to set the configuration options for the NodeJS REST server. There is a sample configuration file within the **`/btc-server/cln-nodejs-rest`** directory. Rename the sample file as follows:

* **`$ sudo mv sample-cl-rest-config.json cl-rest-config.json`**

Open the configuration file and make sure the settings below are configured.  

`{
    "PORT": 3020,
    "DOCPORT": 4020,
    "PROTOCOL": "https",
    "EXECMODE": "production",
    "LNRPCPATH": "/btc-server/core-lightning/bitcoin/lightning-rpc",
    "RPCCOMMANDS": ["*"],
    "DOMAIN": "[your_domain]",
    "BIND": "127.0.0.1"
}`

The **`"PORT"`** option configures the listening port for the server. We deviate from the recommended listening port settings, namely **`3002`**, to avoid conflict with the BTC Explorer server. As our other REST API runs on port **`3010`**, port **`3020`** seems a sensible choice. For consistency, we also set the Swagger documentation port at **`4020`** instead of **`4001`**.

For the rest, it is important to note the custom path to the **`"lightning-rpc`** file and that the server will only listen to the loopback address. As we will configure outside communications over a Tor hidden service, it is not necessary (and only an additional risk) to listen on all network interfaces. The communication protocol still needs to be set to **`"https`"**, even though there is only internal communication to our server. Our understanding is that the Zeus wallet expects an **`https`** connection, even if Tor is configured. Strictly speaking, this would be unnecessary. 

You also need to set the domain option here, though it becomes important in the next section. Specifically, you should set the domain name associated with the public IP address of your home router. If you are not sure what that is, proceed as follows:

* Find out your public IP address. You can use [https://whatismyipaddress.com/](https://whatismyipaddress.com/). 
* Open the Command Prompt application on your managing computer ***as an administrator***.
* Execute **`$ nslookup [your_IP_address]`**.

The output should offer you a name for your public IP address. This should be set as your **`"DOMAIN"`** in the configuration options above. 

To view the certificate, enter into the **`certs`** directory and execute the following instruction:

* **`$ sudo openssl x509 -in certificate.pem -text`**

For now, let's try to incorporate all these settings to see if the server runs.

* **`sudo systemctl start cln-nodejs-rest.service`**
* **`sudo systemctl status cln-nodejs-rest.service`**

If all has been set in the right way, then you should see the following messages:

* **`cl-rest api server is ready and listening on 127.0.0.1:3020`**
* **`cl-rest doc server is ready and listening on 127.0.0.1:4020`**

As a last step, we will need to configure a Tor hidden service. First move into the **`/etc/tor`** directory and open the **`torcc`** file with a text editor. Go to the end of the file and add the following lines:

`## Hidden service for the Core Lightning NodeJS REST API
HiddenServiceDir /var/lib/tor/cln-nodejs-rest
HiddenServicePort 3020 127.0.0.1:3020`

The first configuration specifies a directory to store information about the hidden service. It is important that you set it within the **`tor`** directory, as the service account for Tor will require access. The second configuration specifies the public port for the hidden service, as well as the internal listening address of the NodeJS REST API. Let's relaunch the service and ensure it is still operating normally:

**`$ sudo systemctl restart tor.service`**
**`$ sudo systemctl status tor.service`**

To check that the hidden service is running, execute the following instruction:

* **`$ sudo cat /var/lib/tor/cln-nodejs-rest/hostname`**

If everything was configured correctly, you should now see the onion address for the hidden service.

Next, move into the **`/btc-server/cln-nodejs-rest/certs`** directory. You should see a file called **`access.macaroon`**. A **macaroon** is like an authentication cookie, but offers more advanced features. You will need the macaroon in hexadecimal format to obtain access with Zeus. To output the content in hexadecimal format, execute the following instruction:

* **`$ xxd -ps -u -c 1000 access.macaroon`**

As a final step, we now need to install and configure the Zeus wallet onto your mobile phone. Proceed as follows:

* Download the application from your app store and open it. Opening it for the first time, you will see a start screen. 
* Click on **`Advanced Set-Up`**. 
* Select **`Connect a node`**.
* Click on the **`+`** sign.
* For your **`Nickname`**, it makes sense to choose your Core Lightning alias and a Tor indication (i.e., **`[Alias] Tor`**). 
* Choose the **`Core Lightning (c-lightning-REST)`** for the node interface.
* For the **`Host`** field enter your hidden service address.
* Input the access macaroon in hexadecimal format.
* Specify the port for your hidden service, **`3020`**.

After some moments, you will see the Zeus interface load. There are three tabs at the bottom. The first tab will show you the on-chain and lightning funds available on your node. Note that it might take some time to pull in this data. The second tab allows you to make and receive payments. The final tab will display all your channels. 

The next time you connect with the Zeus wallet to your node, the data will be retrieved more quickly. Do note that it always does take some moments to establish a connection given that you are making the connection over a hiddden service.


# TLS connection

Connecting to your server via a Tor hidden service is very slow. It can also lead to instability in the connection. While it works, it does not really offer the experience of instant payments we are hoping for with Lightning.

You can connect directly over TLS with your node. Start as follows:

* Move into the **`/btc-server/cln-nodejs-rest`** directory and open the configuration file.
* Change the **`"BIND`"** setting to **`"0.0.0.0"`**.
* Restart the server by executing **`$ sudo systemctl restart cln-nodejs-rest`**.
* Log into your router and set up port forwarding for port **`3020`**. 

The TLS certificate for the application and the configuration settings ensure that port **`3020`** demands a TLS connection. There are various ways you can verify this is the case. Here are two ways:

* You can check with the **`openssl`** application. Execute **`$ openssl s_client --connect 127.0.0.1:3020`** on your server. You should see an output that you are connected followed by a lot of information about the connection. If the port were not SSL enabled, you would receive an error message.
* You can also use a port scanner. As an example, you can use this service: [https://pentest-tools.com/network-vulnerability-scanning/tcp-port-scanner-online-nmap](https://pentest-tools.com/network-vulnerability-scanning/tcp-port-scanner-online-nmap). It will list your connection as **`https`**.

You should now move back into your Zeus settings and click on the settings for your profiles. Using the **`+`** button, you can add an additional profile. Proceed as follows:

* Use **`[Alias] TLS`** as a nickname.
* Select the Core Lightning REST for the **`Node interface`**.
* For the host, use your router's public IP address.
* Fill in the same macaroon as for your Tor connection.
* For the rest port, fill in **`3020`**.

***From here on, read first before making any changes.*** If you leave the **`Use Tor`** and **`Certificate Verification`** toggles off, you will receive a warning. You can, however, bypass the warning. Were you to do this, you would see that you have a much faster and more stable connection to your node. 

The main problem, however, with connecting to your node in this manner is that you have ***no way to verify that you are indeed connecting to your own server***. Some bad actor may spoof your server address and offer you a fake TLS certificate. They can, then, decrypt all the API calls you make and intercept your macaroon. Once they have your macaroon, they can essentially close all your channels and steal all your funds. While we could probably do something like this quite safely within our local network, we cannot do it safely connecting over the Internet. 

If you want to test out this option, you can proceed as just discussed. But you should be aware that your funds are at risk. In any case, if you have experimented with this type of unverified connection at any time, you should go into your **`/btc-server/cln-nodejs-rest/certs`** directory; delete your **`access.macaroon`**, **`certificate.pem`**, **`key.pem`**, and **`rootKey.key`**; and then restart the service for your NodeJS REST server. This will generate a new macaroon that has not yet been dangerously exposed.

Given the risks of an unverified TLS connection, the proper way to make the connection would be as follows:

* Output the TLS certificate on your terminal window by executing **`$ sudo openssl x509 -in /btc-server/cln-nodejs-rest/certs/certificate.pem -text`**.
* Create a new text file on managing computer called **`certificate.pem`**. Copy the contents from your terminal window into it and save the file.
* Send the file to your own mobile phone (e.g., using Whatsapp or Signal).
* Download the file onto your mobile phone.
* Under your Settings, move into **`Security & privacy`** and **`More security & privacy`**. Then move into **`Encryption & credentials`**. 
* Now select **`Install certificate`** and choose **`CA certificate`**. 
* Ignore the warning and choose **`Install anyway`**.
* Select your certificate and it should install.
* If you move back to **`Encryption & credentials`**, you should see the certificate under **`Trusted credentials`** installed by the **`User`**.

You can now open Zeus again. Move into your TLS profile and turn on the **`Certificate Verification`** toggle. Press **`Save node config`**. 

If you now try to load the profile and are using an Android, you should see an error that partially reads **`Trust anchor for certification path not found`**. This is essentially Android complaining that the CA on the certificate is not known. The problem with platforms like Android is that they try to lock down more and more what users can do. 

At this point, you have three options. 

1. There seem to be ways to hack Android, so that it will accept a self-signed certificate. You can research those ways online and attempt them on your own. 
2. You may just leave the TLS profile alone and use your Tor profile instead. You should, however, then eliminate the forwarding for port 3020 on your router and make the NodeJS REST API only listen locally again: that is, you should set **`"BIND"="127.0.0.1"`** in the configuration file. 
3. You can find other ways to bypass the Android restrictions. You might, for instance, try to connect using Wireguard as a VPN to your server.