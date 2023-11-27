# Chapter 7. Using Tor 

In order to protect their privacy, many Bitcoin users employ the Tor service. In this chapter, we will first explain what the Tor service is and why many bitcoiners rely on it to protect their privacy. In the discussion, we will consider the role of Tor with regards to privacy before the implemention of BIP-324 in version 26.0 of Bitcoin Core and after. We will, then, install the Tor service on our system and close with discussion of a few privacy issues when using Bitcoin. 


## Bitcoin Core before v26.0

Before Bitcoin Core v26.0, there was no native support for the encryption of communications between nodes. So, if you just run a version of Bitcoind before v26.0 from your home server without additional precautions, your node will communicate with other nodes, such as about blocks and transactions, in plaintext. Additionally, all your data communications will be accompanied with your router’s public IP address when traveling over the Internet. 

The situation is summarized in *Figures 1* and *2*. Any Bitcoind messages by one node to the other are broken up into packets, sent across the Internet, and re-assembled on the other side according to the standards also typically used with communications over the World Wide Web (that is, according to the standards of the **transmission control protocol** or the **TCP**.) Albeit somewhat simplified, the figures show the basic format of your sent and received Bitcoind packets respectively as they travel over the Internet. The **payload** contains a part of the Bitcoind message, the IP address identifies the destination (typically a home router), and the logical port is used to associate the communication with a particular application, namely Bitcoind. (Note that these communications will look different as they travel in your local network, due to the network address translation service managed by your router.)  

*Figure 1: Sent communications*

![Figure 1: Sent communications.](/Images/Figure6-1.png "Figure 1: Sent Communications")

*Figure 2: Received communications*

![Figure 2: Received communications.](/Images/Figure6-2.png "Figure 2: Received Communications")

Plainly running Bitcoind before v26.0, thus, conveys information both at the data layer and the network layer of your communication packets. This introduces two risks. 

First, anyone observing your Internet traffic, such as your Internet service provider (ISP), will know that you are running a Bitcoin node and, hence, that you probably have bitcoin. They can obviously ascertain this information from your node’s data communications (about transactions, blocks, and so on). They can also ascertain it purely from the port numbers being used in your Internet communications (8333 is the common listening port for bitcoin nodes that accept inbound connections). 

In the best scenario, only your ISP will be able to precisely couple your router's public IP address to your identity. Others who intercept your communication packets will only know the geographical area from which they originated. But other parties such as governments, companies, and hackers could attempt to obtain this information from your ISP. In addition, anyone directly monitoring the traffic from your house would be able to tie your identity to the IP address. So with all the IP information on Bitcoind communications, you really should just assume anyone can learn that you are running a Bitcoin node.    

Second, plainly running Bitcoind limits your transaction privacy. Your full node will, by default, communicate about many transactions which were originally sent to you by other nodes. But this achieves very little by way of transaction privacy. 

To start, anyone who monitors your home Internet traffic on location can easily see which specific transactions originated from your IP address specifically. Your ISP, of course, does this standardly. But even without monitoring your home network locally, parties that monitor the Bitcoin network, such as chain analysis companies, can apply techniques to establish that particular transactions likely originated from your router's public IP address. Through various methods, they can try to couple those transactions to your identity. 

At least one common way that people address the concerns above is by using the Tor service to implement Tor circuits and/or Tor hidden services in their communications. Let's first walk through a description of these two concepts. 


## Tor circuits

Suppose that you wanted to make requests from a web server, but wanted to ensure that neither the web server nor anyone else would be able to know that you are making those requests. You cannot achieve such privacy with a regular http connection, as anyone who sees the communication packets can see the public IP address of the originating router. The same is true, however, if your communications to the server use the **transport layer security** standard (or its predecessor, the **secure sockets layer** or **SSL** standard). While that would encrypt the payload of the packets, the web server can still see the packets are coming from your IP address, as would anyone else who sees the packets en route.

A common way that people address the need for privacy in this situation is by utilizing the **onion routing network**, better known as the **Tor network**. It starts with a client who constructs what is known as a **Tor circuit** or **Tor path**. In broad lines, such a circuit is created as follows:  

* The client downloads information about a great number of **Tor relay nodes** from the Tor directory. The **Tor directory** is a network of servers that keep an updated list with information on all the Tor relay nodes available. The information includes an IP/Port address and a public key for each Tor relay. The directory is public. 
* The client selects three relay nodes to build a Tor circuit. The first node is known as the **entry** or **guard node**, the second as the **middle node**, and the third as the **exit** node.  
* The client makes a connection to the chosen entry node and they establish a symmetric cryptographic key through a Diffie-Hellman key exchange protocol. Let's refer to this key as k<sup>1</sup>. 
* The client then establishes a symmetric cryptographic key with both the middle node and the exit node. But it does not conduct these key exchanges through direct connections with these nodes. Instead, the client uses the entry node as a forwarding point for data about the key exchange with the middle node, and the entry and middle nodes as forwarding points for data about the key exchange with the exit node. Let's call the symmetric keys exchanged in this manner k<sup>2</sup> and k<sup>3</sup> respectively.

We will not delve into the details of the key exchange. The main point is that, if the key establishment protocol works as intended, the following is true:

* The client knows the IP/Port of all the relay nodes and the keys k<sup>1</sup>, k<sup>2</sup>, and k<sup>3</sup>.
* The entry node knows the IP/Port of the client and the middle node, and the key k<sup>1</sup>.
* The middle node knows the IP/Port of the entry node and the exit node, and the key k<sup>2</sup>.
* The exit node knows the IP/Port of the middle node, and the key k<sup>3</sup>. 

Each time that the client now wants to send communications to the web server, each packet is processed as follows:

* The client creates the payload and adds the IP/Port information of the web server as the destination. All of this is encrypted with the key k<sub>3</sub>. 
* The client then adds the IP/Port information of the exit node as the destination and encrypts all the data with k<sub>2</sub>.
* The client then adds the IP/Port information of the middle node as the desintation and encrypts all the data with k<sub>1</sub>.
* Finally, the client adds the IP/Port information of the entry node as the destination and her own IP/Port information as the source. 

Each packet is now sent to the web server in the following manner:

* The client sends the packet to the entry node, as indicated by the destination IP/Port. 
* The entry node strips the IP/Port information of the client's router from the packet source. She also strips her own IP/Port information from the packet destination. She, then, decrypts the payload using k<sub>1</sub>, exposing the IP/Port information of the middle node as the new destination. She then adds her own IP/Port information as the source address and sends the packet to the middle node.
* The middle node strips the IP/Port information of the entry node from the packet source. She also strips her own IP/Port information from the packet destination. She, then, decrypts the payload using k<sub>2</sub>, exposing the IP/Port information of the exit node as the new destination. She adds her own IP/Port information as the packet source and sends the packet to the exit node.
* The exit node strips the IP/Port information of the middle node from the packet source. She also strips her own IP/Port information from the packet destination. She, then, decrypts the payload using k<sub>3</sub>, exposing the IP/Port information of the exit node as the new destination. She adds her own IP/Port information as the packet source and sends the packet onto the web server.

The web server processes the packets from the exit node in a standard way to reply to the requests. The web server might know that the exit node is a Tor relay by checking the Tor directory, but it does not really matter for processing any requests to it. Any packets sent back out by the web server end up with the client as follows: 

* The web server creates a standard TCP/IP packet that is sent to the exit node.
* The exit node strips the IP/Port information of the web server from the packet source. She also strips her own IP/Port information from the packet destination. She, then, encrypts the payload using k<sub>3</sub>. She then adds the IP/Port information for the middle node as the destination and her own IP/Port information as the packet source, and sends the packet to the middle node.
* The middle node strips the IP/Port information of the exit node from the packet source. She also strips her own IP/Port information from the packet destination. She, then, encrypts the payload using k<sub>2</sub>. She then adds the IP/Port information for the entry node as the destination and her own IP/Port information as the packet source, and sends the packet to the entry node.
* The entry node strips the IP/Port information of the middle node from the packet source. She also strips her own IP/Port information from the packet destination. She, then, encrypts the payload using k<sub>1</sub>. She then adds the IP/Port information for the client as the destination and her own IP/Port information as the packet source, and sends the packet to the client.
* The client decrypts the packet using, in order, k<sub>1</sub>, k<sub>2</sub>, and k<sub>3</sub>. 

Routing packets with the type of encryption layering illustrated above is known as **onion routing**. The Tor protocol is by far the most widely used implementation of this privacy-enhancing approach to packet routing. It intends to ensure that the web server only knows the IP/Port of the exit node. She is not supposed to know the IP/Port of the middle node, the entry node, or the client. While the entry node does know the IP/Port of the client, she should not know the IP/Port of the exit node or the web server. If all works as intended, the client enjoys a much greater amount of privacy using a Tor circuit to communicate with the web server. If necessary, the client can also use TLS on all her packets, so that the payload is not exposed to the exit node.  

Importantly, there are many ways in which the privacy of a Tor circuit can be broken. One way is through the client's activities. If the client, for example, uses a Tor circuit to communicate with a web server, but logs into an account associated with her real world identity, then the web server will still know who is making the requests. There are also ways for malicious actors to carry out privacy-breaking attacks. A very common type is known as an **entry/exit attack**. The attack works as follows:

* An attacker invests resources into running many Tor relays.
* The attacker performs packet analytics on all her relay traffic, in order to correlate packets she sees across different relay roles.
* By chance, the attacker should sometimes be both the entry and exit node in a circuit. If her packet analytics are sophisticated enough, she will be able to establish reasonably well when she is the entry and exit node for a particular circuit.
* In such cases, the attacker knows the public IP/Port of the client's router from the information she has as the entry node and the public IP/Port of the web server from the information she has as the exit node. 

All ordinary Tor relays are published in the Tor directory. So if you just connect to an ordinary Tor circuit, it becomes pretty easy to establish that you are using Tor. In many places, this does not really matter. Originally created by the US Naval Research Laboratory in the 1990s, the Tor Project is now a perfectly legitimate American non-profit organization. In many countries, using Tor is perfectly legal. 

However, in some countries Tor is, in fact illegal, or at least much more risky to use. Such countries can also easily block all network traffic to the IP/Port addresses listed in the Tor directory. In such situations, one can also employ Tor relays which are not publicly listed called **bridge nodes**. Anyone at risk of censorship or that wants to better hide that they are using Tor circuits should connect to a Tor circuit via bridge node instead. 

 your ISP or anyone else knowing about your Tor usage hardly strikes me as a grave concern. In case you do want to protect your privacy even further, you can look into Tor Bridges or combining Tor with VPNs.


## Tor hidden services

A Tor circuit can greatly improve the privacy of the client, but it does little for the privacy of the server. Privacy on the server side can be ensured using what is known as a **Tor hidden service**. A server can establish a hidden service as follows:

* The server creates a private-public keypair. The server will encode the public key in what is known as an **onion address**, which ends with the ".onion" extension. (At least, this is the case in the latest version of the standard, v3. In previous versions, the onion address was encoded using the hash of the public key.)
* The hidden service selects several ordinary Tor relays from the Tor public directory, typically three, to serve as **introduction points**. It, then, creates a Tor circuit to each of these relays. The hidden service then requests each relay if they will serve as an introduction point. If the relay agrees, the hidden service shares it's onion address with the relay. 
* The hidden service, then, creates a description of the hidden service, which includes the onion address and the IP/Port information of its introduction points. The description is signed with the service's public key. 
* The description is, then, sent out over the Tor Network's **distributed hash table system** (DHT) using the circuits to the introduction points. Various nodes in the network will store the description of the hidden service, where a hash of the public key serves as the search key. 

Once the hidden service is set up, a client can make a connection to the hidden service as follows:

* The client needs to learn about the onion address of the hidden service. It is not published anywhere within the Tor network, so it will have to be obtained from a website, forum, personal correspondence, or some other method.  
* Once the client knows the onion address, they can calculate the indexation hash and make a query to the Tor DHT. They will be returned the IP/Port information of the introduction points.
* The client selects a random ordinary Tor relay from the directory, which will serve as the **rendezvous point**. The client establishes a Tor circuit to the rendezvous point, though, in contrast to standard circuits, it only has two intermediate hops. Using the circuit, the client sends the rendezvous point a one-time secret. 
* The client, then, creates an introduction message with the IP/Port of the rendezvous point and the same one-time secret. That message is encrypted with the hidden service's public key.
* The client sends the message to the rendezvous point for forwarding to one of the introduction points. The rendezvous point creates a Tor circuit to the introduction point, and sends the message to the introduction point with a request for forwarding to the hidden service.
* The hidden service decrypts the message to learn about the rendezvous point and the one-time secret.
* If the hidden service wants to establish a connection, they will then create a standard Tor circuit with three intermediary relays to the rendezvous point. Let's refer to the symmetric cryptographic keys for this circuit as k<sup>4</sup>, k<sup>5</sup>, and k<sup>6</sup>. The hidden service will, then, send the rendezvous point a message with the one-time secret. On the basis of the one-time secret, the rendezvous point, then, makes the connection between the two circuits, and notifies the client of the connection. 
* Once the circuit has been established, the client and hidden service establish a symmetric key ***k***, using Diffie-Hellman key exchange. 

A communications packet from the client to the server is processed as follows:

* The client encrypts the packet using the secret key ***k***. It, then, processes the packet according to the standard rules of a Tor circuit, using the keys k<sup>1</sup> (entry node), k<sup>2</sup> (intermediate node), and k<sup>3</sup> (rendezvous point). The rendezvous point receives the packet and decrypts it using k<sup>3</sup> to reveal the message encrypted with the key ***k***.
* The rendezvouspoint, then, forwards the packet through the Tor circuit with the server. The packet is processed using the corresponding encryption key at each hop. The server, then, has to decrypt the incoming packet using k<sup>4</sup>, k<sup>5</sup>, k<sup>6</sup>, and finally ***k***. 

The process works similar for any communications sent from the server to the client.

The total circuit between the server and client contains six intermediate hops. Any message is end-to-end encrypted due to the secret key ***k***. As long as the hidden service does not make any connections over **clearnet**, then the protocol can help assure that neither the client nor the web server will have each other's IP/Port information. While hidden service connections slow down communications, they can offer much more privacy for both clients and servers. 


## Bitcoin Core before v26.0 and Tor

A Bitcoin node can have either only outbound connections, or both outbound and inbound connections. An **outbound connection** is a connection which the node initiates from her side. An **inboud connection** to a node is one initiated by the other node. 

By default, Bitcoin Core only allows outbound connections. Specifically, the default number of outbound connections is 10, which ensures sufficient redundancy to protect yourself from dishonest nodes, specifically to protect yourself against **eclipse attacks**. A node that only accepts outbound connections is called a **non-listening node**.

However, Bitcoin Core can also be configured to accept inbound connections. Specifically, you can allow up to 115 inbound connections. The maximum of 125 connections that is set by default on Bitcoin Core is motivated by the typical system limitations for personal computers. A node that also allows inbound connections is known as a **listening node** (as it “listens” for incoming connection requests). 

Suppose for the moment that you only had the standard 10 outbound connections. There are various ways, you can arrange your outbound connections with the Tor service. To start, you can choose to connect to both clearnet and onion addresses using the Tor service. Any connections to clearnet addresses will then use ordinary Tor circuits, while connections to onion services will come with the lengthier circuits that are encrypted end-to-end. If desired, you can choose to disable connections to onion services or to clearnet addresses. Note that you do not have the option to use both clearnet and Tor connections (either via ordinary circuits or hidden services) with outbound connections. 

Suppose now that you also wanted to accept inbound connections. To start, you can advertise only a clearnet address on which you are listening using port forwarding. In this case others can connect to you directly over clearnet or via a Tor circuit, it depends on their preferences. You can, however, also use the Tor service to configure a hidden service address for inbound connections. Unlike with clearnet connections, you will not need to set up port forwarding, as the Tor service typically can automatically bypass your router settings. If you accept inbound connections via the hidden service, you can choose to still also accept clearnet connections or to disable them alltogether.

What are sensible choices for you in this array of possibilities? The answer is that it really depends on your situation.

If all your outbound connections run over standard Tor circuits and/or hidden service circuits and all your inbound transactions, if these are allowed, only via your own onion service, you potentially run certain risks that you will not have if you have at least some clearnet connections. In 2014, for example, Biryukov and Pustogarov showed that any node with only outbound connections over Tor can be attacked fairly easily. In sum, the attacker bans legitimate exit nodes and carries out a DoS attack on Bitcoind hidden services, so that a victim connects only to the attacker's exit nodes and hidden services. This type of attack is known as an **eclipse attack**, and she can use this type of attack to distort and exploit your view of the Block Chain and transactions.<sup>[1](#footnote1)</sup> 

Hence, only making connections over ordinary Tor circuits and/or onion service circuits should only be considered ***if you really need to hide that you are running a Bitcoin node***. In that case, you should do something like the following: 

* Stream all your outbound connections over ordinary and onion service circuits. It does not make so much sense to restrict your outbound connections to either one of these options. In fact, making connections to both clearnet and onion addresses ensures that you help avoid network partitions. To reduce the chance of censorhip by authorities, you can run a bridge node. 
* Try to allow inbound connections, if you can, as it is always a good idea to support the network. But in your case, you would have to restrict the inbound connections to only work via a Tor hidden service. Any clearnet connections would expose your Bitcoin node. 
* Be careful with any transactions you make or receive. Try to verify any transactions independently, for example, by using your bridge and a Tor browser to check it's status on a publicly available block explorer. 

While this approach is by no means a guarantee of your privacy and comes with some more security risks, it can help you if you really need to hide your bitcoin usage. 

In case you only care about transaction privacy, then the best policy is to have some connections over ordinary and hidden service circuits, and some connections plainly over clearnet. Given that your outbound connections must be either over clearnet or over Tor and cannot be a combination, there are probably two good ways in which you could configure the Tor service with Bitcoin Core to achieve a combination of Tor and clearnet connections. First, you could set things up as follows:

* Just leave all your outbound connections to work over clearnet, so you know that you are abiding by the assumptions of Bitcoin's security model.
* Set up a Tor hidden service for inbound connections.
* Set up port forwarding to also accept some inbound connections over clearnet. As many people will only use clearnet, you are offering wider support for the network by also allowing inbound connections. 

Alternatively, you could also just let all your outbound connections run over ordinary and hidden service Tor circuits, but only accept inbound connections over clearnet or a combination of inbound transactions over clearnet and a Tor hidden service. 

When your run a combination of clearnet and Tor connections, it will not be as obvious whether any transaction plainly sent over clearnet orginated from your node. After all, it could have been shared with you via a Tor connection and you are just forwarding it. 


## Bitcoin Core after v26.0 and Tor

Starting from Bitcoin Core v26.0, a node will attempt to establish encrypted connections with other nodes. As long as they also support a v26.0 or higher, an encrypted connection will be established, where the payload resembles a pseudorandom bytestream. The details of the standard are set out in ***BIP-324**.<sup>[2](#footnote2)</sup> 

Encrypted connections between Bitcoind nodes increase transaction privacy. Currently, an attacker can break transaction privacy by passively intercepting network traffic. In order to break transaction privacy with BIP-324 encrypted connections, an attacker would need to carry out an active attack of on of the following forms: as a persistent man-in-the-middle, by downgrading connections to be non-encrypted, or by spinning up their own nodes and making connections to honest nodes. Such active attacks are much costlier. In case the attacker is running their own nodes and deliberately making connections to certain nodes, the attack is also fairly detectable. In addition, the pseudrandom encrypted communications make identification of Bitcoin network traffic more difficult, especially if the bytestream can be shaped to mimic other protocols on the Internet.

While BIP-324 is important for the overall privacy and security of Bitcoind usage, using the Tor service still has substantial benefits if you want to hide that you are using Bitcoind. In that case, you will still want all your outbound and inbound connections to use the Tor service. Even refusing non-encrypted clearnet connections, your IP address will still be exposed on all communications. So the benefit of BIP-324 in this regard is limited.  

With regards to transaction privacy, the benefits of using the Tor service are less clear. But probably mixing Tor and clearnet connections is better assurance against privacy attacks, then only using clearnet connections with encryption. In any case, running a Tor hidden service supports those network nodes that only have outbound connections with Tor hidden services, and that, at most, accept inbound connections via a Tor hidden service. So in this case, Tor can still be a good idea even if it has some bandwith and latency costs.    


## The choice for Tor

For the rest of this chapter, we will install and set up the Tor service on your system. In the next chapter, we will provide instructions for how to create a Tor hidden service for Bitcoind. We will, then, set up Bitcoind to also accept inbound clearnet connections. All outbound connections will be left to the default settings. These settings help assure transaction privacy and ensure that you are widely supporting the network via your inbound connections. 

We made this choice because transaction privacy is probably the primary concern for most Bitcoin users. It is no one else’s business to be aware of your financial transactions and to make deductions on their basis (such as about your spending habits, how many bitcoins you might have, where you obtained them, and so on). In addition, transaction privacy is crucial to network security. It helps ensure Bitcoin fungibility and continued censorship resistance. 

By contrast, hiding your Bitcoin usage alltogether is probably less of a concern for many Bitcoin users. It is in any case much harder to achieve. But if you really need to hide your Bitcoin usage and are willing to accept some security risks (like eclipse attacks), you can also run all your connections with the Tor service. We will also show how to manage this in this next chapter. 


## Tor as a service

Importantly, you should manage Tor on your Linux system as a service, not as a user application. The same goes for your other Bitcoin server applications. We already encountered the concept of Linux services when installing Fail2ban (and possibly Log2ram if you installed it). So at this point it is a good moment to ask: What exactly is a Linux service?  

A **service** is any application that has been configured, so that it can, if desired, automatically run in the background of your Linux system. Typically you need to manage applications as services when they are server applications. Suppose, for example, that you are hosting a website at home. Then you clearly will want to run your web server application as a service. Managing it as a user application means you would have to manually restart the application each time it crashes, or each time you restart the server.  

Services in modern Linux distributions are typically managed by an application known as **systemd**.<sup>[3](#footnote3)</sup> This application is, in fact, itself a Linux service, the first one that starts up when your operating system boots. As a user, you can control the systemd application through a command line tool known as **systemctl**.  

You may also come across the **init application** in discussions about Linux services, the predecessor to systemd. Most distributions of Linux standardly include both the systemd and init applications. We will only use systemd. 

Managing an application as a service requires a properly configured **service file**. Your system will generally have three types of service files:

1.	Those that came with your operating system.   
2.	Those that are created automatically by certain applications upon installation. 
3.	Those that you create manually.   

The installation of Tor automatically creates a service file for you (so it’s of type 2). This makes enabling the Tor service very easy. To install Tor and enable it as a service, just follow the instructions below. 

* **`$ sudo apt update`**
* **`$ sudo apt upgrade`**
* **`$ sudo apt install tor`**
* **`$ sudo systemctl enable tor`**

After enabling Tor as a service, you can confirm that it is indeed running by executing the following instruction: **`$ sudo systemctl status tor`**. It should output that Tor is “active” (highlighted in green).

Note that the default repository for the APT only includes the latest long-term release of the Tor application, not the latest stable release. This should be sufficient for your purposes. But you can consult the Tor project’s website for instructions on how to install the latest stable release if you wish. Be warned, however, that this is a much more complicated installation process, and, to our mind, not really necessary. 

Linux service files can be stored in multiple locations. In Debian, service files that come with your system or that are created via APT installations, generally seem to be located within the /lib/systemd/system and /usr/lib/systemd/system directories. Any service file you manually create is best placed in the /etc/systemd/system directory.

To see the service files associated with Tor, proceed as follows:

* **`$ cd /lib/systemd/system`**
* **`$ cat tor@default.service`**
* **`$ cat tor.service`**
* **`$ cat tor@.service`**

As a general rule, it is best not to tinker with any of the service files that came with your Linux distribution, or that were automatically created with APT installations. In other words, you should probably stay clear from the service files in the /lib/systemd/system and /usr/lib/systemd/system directories.  

We will not worry about the exact contents of the service files associated with Tor. You will learn a little more about configuring service files when we turn to Bitcoin Core in the next chapter. 


## Further privacy and security issues

We will turn to making Bitcoind run over Tor in the next chapter. But before moving on, we just want to briefly mention two further points regarding privacy and security.

First, we will not be using the Tor browser. This just a browser application that automatically connects to the world wide web via a Tor circuit. While we do encourage the use of the Tor Browser generally, you really do need to take care with using it to connect to Bitcoin service providers such as exchanges. ***The Tor Browser has, namely, been subject to specific types of attacks with Bitcoin transactions.***<sup>[4](#footnote4)</sup>

Second, we cannot emphasize enough that privacy is a complicated issue when using Bitcoin. Tor helps, but it is not a silver bullet. 

Imagine, for instance, that you have bought all your bitcoin from a centralized exchange which utilizes know your customer controls. Your Tor usage will do nothing to stop the exchange from tracking your usage of those coins via the Block Chain. 

Hence, Tor does not offer you a one stop solution for privacy. Many other factors matter. If you are highly concerned about transaction privacy in particular, here are some of the main actions you can consider: 

* Running a local block explorer to prevent a centralized server from clustering interest in particular addresses and transactions to an IP address. If you do use a public block explorer, we would recommend at least using the Tor browser.
* Using a trustworthy coinjoin implementation to help make following transaction chains on the Block Chain more difficult. 
* Avoiding the re-use of your addresses, including change addresses. Typically wallets are configured not to re-use addresses, but not always.  
* If possible, source your bitcoins from more privacy-friendly sources such as peer to peer exchanges without know your customer controls. In any case, receiving new bitcoins and mixing them with your current bitcoins can introduce serious privacy risks, even if your current bitcoins are all well-protected in terms of privacy.
* You can also consider keeping separate bitcoin wallets with an eye on privacy. For instance, you can create a wallet to accept payments and a wallet for long-term storage. You can, then, properly coinjoin any payments you have received before moving them into long-term storage. 
* You can also consider chain hopping. In the past, a common approach has been to exchange your bitcoin for an altcoin which has more privacy protection built into its basic transaction protocol. After conducting some transactions in the more privacy-oriented coin, you can then exchange these coins back for bitcoin (preferably in separate batches over a period of time). We are skeptical that privacy-friendly coins are really that privacy-friendly. But you might try the same concept using Liquid, which supports confidential transactions. 
* Obviously, be careful with any sharing of Bitcoin addresses that you own.

Importantly, some of the measures above really matter in the details with regards to their effectiveness and some carry risks in their execution, particularly for users with less technical expertise. Again, privacy is a complicated issue when using Bitcoin. 


## Notes

<a name="footnote1">1</a>. Alex Biryukov and Ivan Pustogarov, "Bitcoin over Tor isn't a good idea", October 22 (2014), available at https://arxiv.org/abs/1410.6079. 

<a name="footnote2">2</a>. Dhruv Mehta, Tim Ruffing, Jonas Schnelli, and Pieter Wuille, "Version 2 P2P encrypted transport protocol", *BIP-324*, available at https://github.com/bitcoin/bips/blob/master/bip-0324.mediawiki. The original proposal was labeled as BIP-151.  

<a name="footnote3">3</a>. Systemd is also the name for a suite of tools to manage your operating system. The systemd application is the most important application in this suite. Systemctl, mentioned below, is also included in the sytemd suite.

<a name="footnote4">4</a>. See Nusenu, “How malicious Tor relays are exploiting users in 2020”, August 9 (2020), available at https://medium.com/@nusenu/how-malicious-tor-relays-are-exploiting-users-in-2020-part-i-1097575c0cac.