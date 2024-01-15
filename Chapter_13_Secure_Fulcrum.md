# Secure Fulcrum

Our Fulcrum application does not currently use a **TLS certificate** for connections. That means, any requests over our local Wifi network would only be encrypted via the shared key offered by our router (probably via **WAP-2** or **WAP-3**). Anyone with the shared key can, thus, decipher traffic between another local computer and Fulcrum. Even without possessing the shared key, standard Wi-Fi encryption has known weaknesses. If your connection runs via ethernet cables, it is probably more secure in practice. Even though the traffic is not encrypted, intercepting packets over ethernet is typically more difficult, as it requires physically tapping the line. 

Even more importantly, if you were ever to connect to Fulcrum from outside your local network, all communications to it would be sent to your router unencrypted. While we will only utilize Fulctrum from our local network, you should set up a TLS connection. 

While Fulcrum has built-in TLS capabilities (as we saw in the configuration file in the previous chapter), we will not utilize them. Instead, we will use **NGINX**, pronounced as “engine X”. This is a popular server application in the World Wide Web’s infrastructure. While the application can be utilized for a variety of purposes, we will employ it here as a simple reverse proxy. A **reverse proxy**, in this context, means a server application that acts as an intermediary to other server applications on your server. (Sometimes a “reverse proxy” can also refer to a distinct hardware server which focuses on performing this type of function.)

So, concretely, all traffic to Fulcrum from remote machines will actually be sent to our NGINX server. It will be configured with a TLS certificate. The NGINX server will, then, forward the traffic on the local machine to Fulcrum.

While NGINX is strictly not necessary for Fulcrum, it is convenient to have NGINX available as a reverse proxy and run our outside communications through it. Some services, such as BTC Explorer, do not have built-in TLS capabilities, so we would need a reverse proxy anyway. Running our applications through NGINX when possible also offers an easy overview and means we generally have to spend less time configuring TLS certificates. 

Let's start with installing NGINX and running it as a service.


## NGINX installation

To start, you first need to install NGINX on your server. Proceed with the following instructions:

* **`$ sudo apt update`**
* **`$ sudo apt upgrade`**
* **`$ sudo apt install nginx`**

Upon installation, NGINX automatically creates a service file and enables the service. If you execute **`$ sudo systemctl status nginx`**, you should see that the NGINX service is running. By default, NGINX will act as a web server for your system. If you execute **`$ netstat -ntl`**, you should see that the application is listening on port 80, the standard port for http traffic. 

When activated, NGINX by default starts a master process with the **`root`** account. Most of the work by the application, however, is handed over to subprocesses running from a less-priveliged service account. By default, this account is called **`www-data`**. It is set up with similar kinds of protections as our **`bitcoind`** and **`fulcrum`** service accounts. You can see the service by executing **`$ htop`**. It should show various worker processes running under the **`www-data`** account. 

In most operating systems, you can only run services on priveliged ports (0 to 1023) from the **`root`** account. As you will not be running a public web server or any other service that requires these ports, you could change the account associated with the NGINX master process and stop using the **`root`** account. However, we will not do this here. NGINX has been developed to run securely with the **`root`** account executing the master process. We are more likely to break things than the increase the security by making changes. In addition, making these changes would be a lot of extra work.   

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

`user nginx;  
worker_processes auto;  
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
worker_connections 768;
}`  

This altered configuration file excludes all the **`http`** and **`mail`** settings. These **NGINX contexts** determine the behavior of http and mail traffic that is received by your server. But as we will not need NGINX as a web or mail server, we can delete these settings. Instead, we will need to configure the behavior for incoming TCP packets, which is done with the **stream** context.

In order to check that you have not made any mistakes, you can restart the NGINX server and check on the status with the following commands:

* **`$ systemctl restart nginx`**
* **`$ systemctl status nginx`**

It should show that NGINX is active. If not, you probably made a mistake in the configuration file syntax. The output from the **`status`** command and the systemd journal output (**`$ sudo journalctl -xeu fulcrum.service`**) should provide you with further information on the problem. 


## Configuring the reverse proxy

We can now configure the reverse proxy for Fulcrum using the stream context. 

To start, we need to ensure that NGINX has the library available for the stream context. Execute the following command:

* **`sudo apt -y install libnginx-mod-stream`**

Next, open the global configuration file again with a text editor. Below the **`events`** context, add the following configurations:

`stream {`

`##`
`#Fulcrum proxy server`
`##`
`upstream fulcrum {
server 127.0.0.1:50001;
}

server {
listen 50002;
proxy_pass fulcrum;
} 

}`  

The configuration specifies how NGINX should handle certain TCP traffic. It will listen on port 50002 for outside connections and pass any data it receives to Fulcrum listening on port 50001 locally on your machine. Any data Fulcrum sends back is also passed through this server. 

We have labeled the server port as 50002, just to make it recognizable for Fulcrum TLS traffic. Alternatively, you can also configure all NGINX ports in some recognizable way such as by using 12001, 12002, 12003, and so on, as they are needed for server applications. You can also apply any other system if it’s more convenient for you. Just avoid using any privileged ports from 0 through 1023, or any ports that you have dedicated to other applications. At this point, ***write the reverse proxy port (i.e., "0.0.0.0:50002) and internal listening port (i.e., "127.0.0.1:50001") down on your server record***.  

You are now ready to test if everything has worked properly. Proceed as follows:

* Exit and save the nginx.conf file.
* Restart the NGINX service with the following command: **`$ sudo systemctl restart nginx`**. If you made any mistakes in the configuration file, NGINX will not restart. Execute **`$ sudo systemctl status nginx`** and **`$ sudo journalctl -xeu fulcrum.service`** to identify the mistake in your configuration file. Fix it and try starting NGINX again.
* Once NGINX has restarted successfully, execute **`$ netstat -ntl`**. You should now see NGINX listening on port 50002 for all interfaces (meaning Local Address = **`0.0.0.0:50002`**).

While NGINX is now listening on port 50002, your server firewall does not yet allow any traffic through. We will start by only allowing traffic from our managing computer. If need be, we can always make it more accessible at a later point. To allow traffic to port 50002, execute the following instruction:

* **`$ sudo ufw allow from 192.168.2.200 to any port 50002`**


## A self-signed TLS certificate

To enable a TLS connection to Fulcrum from our managing computer, we will need a **self-signed certificate**. Let's start with some background.

TLS certificates are used for two main purposes: (1) they allow a client computer to authenticate a server; and (2), they allow a remote client to establish an https connection with a server. 

To illustrate how TLS certificates typically work, suppose that you are attempting to connect to a server for www.Ebay.com. Ebay will have an TLS certificate connected to your traffic that comes in for their web server application. 

The TLS certificate can first be used to authenticate the Ebay server. This is important because you would like to know that you are really dealing with a server owned by Ebay and not, say, a server owned by a criminal syndicate looking to steal your credit card details. The way this works is as follows:

* The TLS certificate has a public key and identification details (e.g., company name, location).
* In addition, the TLS certificate contains a digital signature over the public key and identification information. This signature is from one of various **certificate authorities**. Official certificate authorities only make these signatures if the legitimacy of the public key and identity information has been verified. These certificate authorities can either be **root authorities** or **intermediaries**. The former sign their own certificates, while the latter have a certificate signed by another intermediary or root authority. 
* Your system contains many public keys of well-known, trusted certificate authorities (both root authorities and intermediaries) and can, therefore, verify if the digital signature on Ebay's TLS certificate is valid by following a chain of signatures from any intermediaries to the root.  
* In case of a valid signature chain to a trusted root authority, you know that the server you are communicating with is really from Ebay. 

The https connection with the server can, then, be established using the public key on the TLS certificate. After verifying Ebay’s identity, your computer and Ebay’s server will, via a key exchange protocol, share a symmetric key. This allows you to have a secure communication session. After the communication session is finished, the symmetric key is discarded. 

While you have set up the proxy server for Fulcrum, nothing has changed about the nature of the traffic between the rest of the local network and your proxy server. As you want secure communication between your server and any other computers in the local network, you need to create a TLS certificate and configure your NGINX proxy server with it.

In this case, we will need to create a self-signed certificate: that is, a TLS certificate for our server that is verified by our own root authority. This is actually somewhat of a complicated business. Browsers and other clients applications typically don't like home servers with self-signed certificates. And how much they complain will depend on how exactly you create the certificates. To help us, we will use the instructions found here: [https://github.com/ChristianLempa/cheat-sheets/blob/main/misc/ssl-certs.md](https://github.com/ChristianLempa/cheat-sheets/blob/main/misc/ssl-certs.md).

To start, you will need to ensure that the latest version of the application is installed on your system. Proceed as follows:

* **`$ sudo apt update`**
* **`$ sudo apt upgrade`**
* **`$ sudo apt install openssl`**

The main directory for your ssl application is **`/etc/ssl`**. Here you can find a directory with any private keys you own (**`private`**) as well as a directory with various certificates from certificate autorities (**`certs`**). While these certificates are all from trusted authorities, you can also create the same type of certificate on your own to authenticate connections to your server. 

To create a root CA, you will first need a private key for it. Execute the following command:

* **`$ sudo openssl genrsa -aes256 -out /etc/ssl/private/ca-key.pem 4096`**

This key is created with a passphrase. You can choose the same password as for your GPG signing key without too much security cost. ***Record the password for your CA private key on your server record.***

As a next step, you will create a root certificate for the CA. Essentially, the certificate declares a public key with identity details, signed by the same public key. It follows a standard format called **`x509`**. Execute the following instruction:

**`$ sudo openssl req -new -x509 -sha256 -days 3650 -key /etc/ssl/private/ca-key.pem -out /etc/ssl/certs/ca.pem`**

You will be asked several identification details for the certificate. The details are not too important. You can leave the **`Common Name`** blank en enter the remaining details, mutatis mutandis, as per the following example:

* Country Name: AU
* State or Province Name: New South Wales
* Locality Name: Sydney
* Organization Name: Home Network CA

Let's now move to creating a TLS certificate for your server. We need to set a few configuration options. 

* Open the **`openssl.cnf`** file in the **`/etc/ssl`** directory with a text editor.
* Move down until you see the configuration options for the **`req`** command. You can identify the section by the `[ req ]` on the left-side of the screen. 
* Moving down a bit further, comment out the **`emailAddress`**, `**emailAddress_Max`**, and `**organizationUnitName`** lines under `**[ req_distinguished_name ]`** by placing a "#" in front of each item.
* Save and exit the file.

Next, let's create the private key for the TLS certificate. In this case, we will not encrypt it, so that the server always has access to it. Execute the following instruction:

* **`$ sudo openssl genrsa -out /etc/ssl/private/nginx-key.pem 4096`**

To create a certificate signing request for your local root CA, execute the following instruction from within the **`/etc/ssl`** directory:

* **`$ sudo openssl req -new -sha256 -subj "/CN=btc-server.home" -key /etc/ssl/private/nginx-key.pem -out nginx.csr`**

A certificate signing request is now within your **`/etc/ssl`** directory. It should now be signed by the local root CA. Before doing so, however, we need to ensure that your certificate also includes a **`subjectAltName`** field. This is the field that will generally be checked by browsers and applications to verify that you are connecting to the right server. In our case, we will use **`*.btc-server.home`**, which also includes any subdomains, as well as the local IP address of the server. To save the right details to a file, execute the instruction below. You probably need to execute the instruction with the root profile. 

* **`$ echo "subjectAltName=DNS:*.btc-server.home,IP:192.168.2.150" >> extfile.cnf`**

In the final step, you can create the TLS certificate for your NGINX server. Execute the following instruction from within the **`/etc/ssl`** directory:

* **`$ sudo openssl x509 -req -sha256 -days 3650 -in nginx.csr -CA /etc/ssl/certs/ca.pem -CAkey /etc/ssl/private/ca-key.pem -out /etc/ssl/certs/nginx.crt -extfile extfile.cnf -CAcreateserial`**

This instruction tells openssl to create and sign an **`x509`** certificate with the local root CA. The main details are included within **`nginx.csr`** and should be extended with the data in the **`extfile.cnf`** file. It is typical to create a serial number with each locally signed certificate. While both **`.pem`** and **`.crt`** are acceptable extensions for certificates, Windows and other platforms typically prefer the **`.crt`** extensions. Hence, why our file is called **`nginx.crt`**. 

You can find the certificate in the **`/etc/ssl/certs`** directory. To view it in standard format and in text format respectively, you can use the following instructions:

* **`sudo openssl x509 -in /etc/ssl/certs/nginx.crt`**
* **`sudo openssl x509 -in /etc/ssl/certs/nginx.crt -text`**

At this point, our server is identified by the host-name **`btc-server`**. However, nowhere have we specified that it is within the **`.home`** domain and that **`btc-server.home`** should be coupled to the **`192.168.2.150`** IP address. To do so, proceed as follows:

* Log into your home network's router.
* Find the local network DNS settings.
* Define the network domain as **`home`**. The option is probably called **`Domain`** or something similar.
* Make sure your host name, **`btc-server`** is coupled to your IP address **`192.168.2.150`**. The option is probably called **`Host Name`** or something similar. 
* Save all your settings.
* **Make sure you note the domain of your local network as **`home`** on your server record**.

As a final step, you will have to incorporate the certificate and the private key into your NGINX reverse proxy server for Fulcrum. Importantly, the NGINX private key will only be read by the NGINX master process. Hence, we can leave the private key and certificate in their current directories and need not be concerned with adapting permissions. All you need to do is configure the NGINX global configuration file. Open **`nginx.conf`** in the **`/etc/nginx directory`**, and change your Fulcrum reverse proxy block to look as follows: 

**NGINX global configuration file**

`stream {`

`##`  
`#Fulcrum proxy server`  
`##`  

`upstream fulcrum {`
`server 127.0.0.1:50001;`
`}`

`server {`
`listen 50002 ssl;`  
`proxy_pass fulcrum;`

`ssl_certificate /etc/ssl/certs/nginx.crt;`  
`ssl_certificate_key /etc/ssl/private/nginx-key.pem;`  

`ssl_session_cache shared:SSL:5m;`
`ssl_session_timeout 4h;`
`ssl_protocols TLSv1.2 TLSv1.3;`
`ssl_prefer_server_ciphers on;`
`}` 

`}`

After making the necessary changes, save and exit the configuration file.

The **`ssl`** parameter for the **`listen`** directive indicates that this is an SSL connection to NGINX. The **`ssl_certificate`** and **`ssl_certificate_key`** directives specify the location of the ssl certificate and private key respectively. The **`ssl_session_cache`** option ensures a cache of session parameters is created to improve performance. The size of the cache is 5MB. The **`ssl_session_timeout`** specifies how long session details are stored in the cache. The **`ssl_procotols`** are limited to **`TLSv1.2`** and **`TLSv1.3`** for security purposes. The final directive, **`ssl_prefer_server_ciphers on`** specifies that server ciphers are preferred over client ciphers. 

You can now easily test whether your configurations are working properly. Proceed as follows:

* Restart the NGINX server by executing **`$ sudo systemctl restart nginx`**
* In case it does not restart, you have probably made a mistake in importing the configuration options. You can troubleshoot in the standard ways.  

Once the NGINX service is running properly, you can test the SSL connection by executing the following instruction from powershell on your managing computer:

* **`Test-Netconnection 192.168.2.150 -Port 50002`**   

All your communications between your managing computer and Fulcrum are traveling via a TLS connection to NGNIX. Only the connection between NGINX and Fulcrum is now without encryption and authentication on your local machine. 