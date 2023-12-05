# Securing Electrs

Our Electrs application does not use a **TLS certificate** for connections. That means, any requests over our local Wifi network are only encrypted via the shared key offered by our router (probably via **WAP-2** or **WAP-3**). Anyone with the shared key can, thus, decipher traffic between another local computer and Electrs. Even without possessing the shared key, standard Wi-Fi encryption has known weaknesses. If your connection runs via ethernet cables, it is probably more secure in practice. Even though the traffic is not encrypted, intercepting packets over ethernet is typically more difficult, as it requires physically tapping the line. 

Even more importantly, if you were ever to connect to Electrs from outside your local network, all communications to it would be sent to your router unencrypted. While we will only utilize Electrs from our local network, it is still better just to create a TLS connection capability. 

We can do this by using NGINX. NGINX, pronounced as “engine X”, is a popular server application in the World Wide Web’s infrastructure. While the application can be utilized for a variety of purposes, we will employ it here as a simple reverse proxy. A **reverse proxy**, in this context, means a server application that acts as an intermediary to other server applications on your server. (Sometimes a “reverse proxy” can also refer to a distinct hardware server which focuses on performing this type of function.)

So, concretely, all traffic to Electrs from a remote machine will actually be sent to our NGINX server. It will be configured with a TLS certificate. The NGINX server will, then, forward the traffic on the local machine to Electrs. 

Let's start with installing NGINX and running it as a service.


## Running NGINX as a service

To start, you first need to install NGINX on your server. Proceed with the following instructions:

* **`$ sudo apt update`**
* **`$ sudo apt upgrade`**
* **`$ sudo apt install nginx`**

Upon installation, NGINX automatically creates a service file and enables the service. If you execute **`$ sudo systemctl status nginx`**, you should see that the NGINX service is running. By default, NGINX will act as a web server for your system. If you execute **`$ netstat -ntl`**, you should see that the application is listening on port 80, the standard port for http traffic. 

When activated, NGINX by default starts a master process with the **`root`** account. Most of the work by the application, however, is handed over to subprocesses running from a less-priveliged service account. By default, this account is called **`www-data`**. It is set up with similar kinds of protections as our **`bitcoind`** and **`electrs`** service accounts. You can see all this in action by executing **`$ htop`**. You should see various worker processes running under the **`www-data`** account. 

In most operating systems, you can only run services on priveliged ports (0 to 1023) from the root account. As you will not be running a public web server or any other service that requires these ports, you could change the account associated with the NGINX master process and stop using the root account. However, NGINX has been developed to run securely with the **`root`** account executing the master process. So we will not make any changes in this regard.  

We do suggest to change the service account name associated with the subprocesses for NGINX to **`nginx`** as it is a bit clearer than **`www-data`**. To start, we will need to create the service account in our system with the following instruction:

* **`$ sudo useradd --shell /usr/sbin/nologin --system -M nginx`**

Next, we will need to make the necessary changes to the NGINX global configuration file. Proceed as follows: 

- Enter into the **`root`** account by executing **`$ su root`** and entering your password. 
- Move into the **`/etc/nginx`** directory.
- Open the global NGINX  configuration file called **`nginx.conf`** with a text editor.
- On the top of the configuration file, change **`www-data`** to `**nginx`**. Save and exit the file.
- Incorporate the new configuration with the following instructions:
    - **`sudo systemctl stop nginx`**
    - **`systemctl daemon-reload`**
    - **`systemctl start nginx`**

Now executing **`$ htop`**, you should see the worker processes are running via the **`nginx`** service account. 


## Clean the default configurations

By default, NGINX comes with a rather elaborate configuration structure that includes multiple files. We will only work with the global configuration file. To start, we will want to open **`nginx.conf`** again with a text editor and change it to look as follows:

`user nginx;`  
`worker_processes auto;`  
`pid /run/nginx.pid;`
`error_log /var/log/nginx/error.log;`
`include /etc/nginx/modules-enabled/*.conf;`  

`events {` 
`worker_connections 768;`  
`}`  

This altered configuration file excludes all the **`http`** and **`mail`** settings. These **NGINX contexts** determine the behavior of http and mail traffic that is received. But as we will not need NGINX as a web or mail server, we can delete these settings. Instead, we will need to configure the behavior for incoming TCP packets, which is done with the **stream** context.

In order to check that you have not made any mistakes, you can restart the NGINX server and check on the status with the following commands:

* **`$ systemctl stop nginx`**
* **`$ systemctl restart nginx`**
* **`$ systemctl status nginx`**

It should show that NGINX is active. If not, you probably made a mistake in the configuration file syntax. The output from the **`status`** command and the systemd journal output (**`$ sudo journalctl -xeu electrs.service`**) should provide you with further information on the problem. 


## Configuring the reverse proxy

We can now configure the reverse proxy for Electrs using the stream context. 

To start, we need to ensure that NGINX has the library available for the stream context. Execute the following command:

* **`sudo apt -y install libnginx-mod-stream`**

Next, open the global configuration file again with a text editor. Below the **`events`** context, add the following configurations:

`stream {`

`##`  
`#Electrs proxy server`  
`##`  
`upstream electrs {`
`server 127.0.0.1:50001;`
`}`

`server {`
`listen 50002;`  
`proxy_pass electrs;`  
`}` 

`}`  

The configuration specifies how NGINX should handle TCP traffic. All the general settings for TCP traffic are left to their default values. In addition, there is one server. It listens on port 50002 and passes any data it receives to Electrs listening on port 50001 locally on your machine. Any data Electrs sends back is also passed through this server. 

We have labeled the server port as "50002", just to make it recognizable for Electrs traffic. Alternatively, you can also configure all NGINX ports in some recognizable way such as by using 12001, 12002, 12003, and so on, as they are needed for server applications. You can also apply any other system if it’s more convenient for you. Just avoid using any privileged ports between 0 and 1024, or any ports that you have dedicated to other applications. At this point, **write the reverse proxy port (i.e., "0.0.0.0:50002) down on your server record**.  

At this point, you are ready to test if everything has worked properly. Proceed as follows:

* Exit and save the nginx.conf file.
* Restart the NGINX service with the following command: **`$ sudo systemctl restart nginx`**. If you made any mistakes in the configuration file, NGINX will not restart. Execute **`$ sudo systemctl status nginx`** and **`$ sudo journalctl -xeu electrs.service`** to identify the mistake in your configuration file. Fix it and try starting NGINX again.
* Once NGINX has restarted successfully, execute **`$ netstat -ntl`**. You should now see NGINX listening on port 50002 for all interfaces (meaning Local Address = “0.0.0.0:50002”).

While NGINX is now listening on port 50002, your server firewall does not yet allow any traffic through. We will start by only allowing traffic from our managing computer. If need be, we can always make it more accessible at a later point. To allow traffic to port 50002, execute the following instruction:

* **`$ sudo ufw allow from 192.168.2.200 to any port 50002`**


## Create and configure a TLS certificate

TLS certificates are used for two main purposes: (1) they allow a client computer to authenticate a server; and (2), they allow a remote client to establish an https connection with a server. 

To illustrate how TLS certificates typically work, suppose that you are attempting to connect to a server for www.Amazon.com. Amazon will have an TLS certificate connected to your traffic that comes in for their web server application.  

The TLS certificate can first be used to authenticate the Amazon server. This is important because you would like to know that you are really dealing with a server owned by Amazon and not, say, a server owned by a criminal syndicate looking to steal your credit card details. The way this works is as follows:

* The TLS certificate has a public key and identification details (e.g., company name, location).
* In addition, the TLS certificate contains a digital signature over the public key and identification information. This signature is from one of various **certificate authorities**. Official certificate authorities only make these signatures if the legitimacy of the public key and identity information has been verified. These certificate authorities can either be **root authorities** or **intermediaries**. The former sign their own certificates, while the latter have a certificate signed by another intermediary or root authority. 
* Your system contains many public keys of well-known, trusted certificate authorities (both root authorities and intermediaries) and can, therefore, verify if the digital signature on Amazon's TLS certificate is valid by following a chain of signatures from any intermediaries to the root.  
* In case of a valid signature chain to a trusted root authority, you know that the server you are communicating with is really from Amazon. 

The https connection with the server can, then, be established using the public key on the TLS certificate. After verifying Amazon’s identity, your computer and Amazon’s server will, via a key exchange protocol, share a symmetric private key. This allows you to have a secure communication session. After the communication session is finished, the private key is discarded. 

While you have set up the proxy server for Electrs, nothing has changed about the nature of the traffic between the rest of the local network and your proxy server. As you want secure communication between your server and any other computers in the local network, you need to create a TLS certificate and configure your NGINX proxy server with it.

To create a TLS connection between the reverse proxy and any remote computers, people often recommend a utility called **certbot**. To the best of my knowledge, you cannot use this utility without an actual Web domain. So, instead, we will just create a certificate ourselves with **openssl**. To start, you will need to ensure that the latest version of the application is installed on your system. Proceed as follows:

* **`$ sudo apt update`**
* **`$ sudo apt upgrade`**
* **`$ sudo apt install openssl`**

The main directory for your ssl application is **`/etc/ssl`**. Here you can find a directory with any private keys you own (**`private`**) as well as a directory with various certificates from certificate autorities (**`certs`**). While these certificates are all from trusted authorities, you can also create the same type of certificate on your own to authenticate connections to your server. Before creating the certificate, we need to set a few configuration options. 

* Open the “openssl.cnf” file in the **`/etc/ssl`** directory with a text editor.
* Move down until you see the configuration options for the **`req`** command. You can identify the section by the `[ req ]` on the left-side of the screen. 
* Moving down a bit further, comment out the **`emailAddress`**, `**emailAddress_Max`**, and `**organizationUnitName`** lines under `**[ req_distinguished_name ]`** by placing a "#" in front of each item.
* Save and exit the file.

Now we can create the TLS certificate for NGINX with the following instruction:

* **`$ sudo openssl req -x509 -newkey rsa:4096 -days 3650 -nodes -keyout /etc/ssl/private/nginx.key -out /etc/ssl/certs/nginx.crt`**

The **`req`** command followed by the **`x509`** option instructs the application to create a new root certificate signed by yourself. The certificate type, **`x509`**, is just a common standard. It will sign the request with a new, 4096-bit RSA key (**`-newkey rsa:4096`**). The certificate is valid for ten years given the **`days`** option. The private key has no passphrase (**`-nodes`**). Finally, the **`-keyout`** and **`-out`** options indicate where to store the private key and certificate, respectively. 

After executing the instruction, the application will start asking your for identification information. While many of the details for your certificate do not matter too much, you should enter details that will help you remember the purpose of the certificate. Some example details are following:

* Country Name: AU
* State or Province Name: New South Wales
* Locality Name: Sydney
* Organization Name: Bitcoin Server
* Common Name: btc-server

In order to see the certificate you just created, execute the following instruction:

* **`sudo openssl x509 -in /etc/ssl/certs/nginx.crt`**

As a final step, you will have to incorporate the certificate and the private key into your NGINX reverse proxy server for Electrs. Importantly, the NGINX private key will only be read by the NGINX master process. Hence, we can leave the private key and certificate in their current directories and need not be concerned with adapting permissions. All you need to do is configure the NGINX global configuration file. Open **`nginx.conf`** in the **`/etc/nginx directory`**, and change your Electrs reverse proxy block to look as follows: 

**NGINX global configuration file**

`stream {`

`##`  
`#Electrs proxy server`  
`##`  

`upstream electrs {`
`server 127.0.0.1:50001;`
`}`

`server {`
`listen 50002 ssl;`  
`proxy_pass electrs;`

`ssl_certificate /etc/ssl/certs/nginx.crt;`  
`ssl_certificate_key /etc/ssl/private/nginx.key;`  

`ssl_session_cache shared:SSL:5m;`
`ssl_session_timeout 4h;`
`ssl_protocols TLSv1.2 TLSv1.3;`
`ssl_ciphers TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;`
`ssl_prefer_server_ciphers on;`
`}` 

`}`

After making the necessary changes, save and exit the configuration file.

These settings mostly resemble those offered in the Elctrs Github repository.<sup>[1](#footnote1)</sup> The **`ssl`** parameter for the **`listen`** directive indicates that this is an SSL connection to NGINX. The **`ssl_certificate`** and **`ssl_certificate_key`** directives specify the location of the ssl certificate and private key respectively. The **`ssl_session_cache`** option ensures a cache of session parameters is created to improve performance. The size of the cache is 5MB. The **`ssl_session_timeout`** specifies how long session details are stored in the cach. The **`ssl_procotols`** are set to **`TLSv1.2`** and **`TLSv1.3`** only for security purposes. The **`ssl_ciphers`** directive specifies the allowable standards for connections between the client and your reverse proxy. Both of these last two directives are recommended for a good tradeoff between compatibility and security by the Mozilla Foundation.<sup>[2](#footnote2)</sup> The final directive, **`ssl_prefer_server_ciphers on`** specifies that server ciphers are preferred over client ciphers. 

You can now easily test whether your configurations are working properly. Proceed as follows:

* Restart the NGINX server by executing **`$ sudo systemctl restart nginx`**
* In case it does not restart, you have probably made a mistake in importing the configuration options. You can troubleshoot using the **`$ sudo systemctl status nginx`** instruction. 

Once the NGINX service is running properly, you can test the SSL connection by executing the following instruction from powershell on your managing computer:

* **`Test-Netconnection 192.168.2.150 -Port 50002`**   

All your communications between your managing computer and Electrs are traveling via a TLS connection to NGNIX. Only the connection between NGINX and Electrs is now without encryption and authentication on your local machine. 


## Notes

<a name="footnote1">1</a>. See here: https://github.com/romanz/electrs/blob/master/doc/config.md#extra-configuration-suggestions. 

<a name="footnote2">2</a>. See Mozilla Foundation, “Security/Server side TLS”, “Intermediate compatability”, available at https://wiki.mozilla.org/Security/Server_Side_TLS. 