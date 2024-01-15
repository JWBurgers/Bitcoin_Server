# Chapter 1. Introduction

Many Bitcoin users run a home server to support their Bitcoin activities. Typically, these servers are set up with the support of **node solutions** or **self-hosting solutions**. The former include software solutions, often with a hardware component, that focus on supporting your Bitcoin activities specifically. The latter are solutions that focus on self-hosting activities more generally (e.g., running your own cloud service, web server). There are many node and self-hosting solutions on the market, including from the following companies and projects: Fulmo, Start 9, Nodl, Umbrel, Lightninginabox, and MyNode.  

While node and self-hosting solutions can surely offer a nice and convenient Bitcoin server solution, building a Bitcoin server from the ground up—or at least, somewhat more from the ground up—does have various advantages. Specifically, we can identify at least six advantages to a ***homespun Bitcoin server***.      

1. It teaches you much more about how Bitcoin works and it's limitations.
2. It offers you more control in software and hardware choices.
3. Software problems are easier to troubleshoot with a homespun Bitcoin server. 
4. Node and self-hosting solutions inject another layer of trust into your Bitcoin server. You have to trust the creators, for instance, with implementing legitimate versions of a full node application and keeping it updated. Most node and self-hosting solution projects have a very limited number of dedicated contributors, so using them clearly introduces additional risks.
5. Your full node application is your voting right within the Bitcoin ecosystem. If Bitcoin is facing a constitutional moment—such as it did with the Segwit Civil War in 2017—you want to determine the version of the software you run for your transactions. There is no guarantee a node solution or self-hosting solution will offer support for your convictions.  
6. A homespun Bitcoin server is typically more cost efficient.    

This guide offers a recipe for making a homespun Bitcoin server. Your server will run 24/7 with core services that are configured to run automatically, including a Bitcoin node, an Electrum server, and a Lightning node. 


## The guide in more detail

*Chapters 1* through *5* guide you in setting up your server system. We guide you through hardware choices, installing the operating system, establishing a remote connection from a managing computer, and implementing various functional, security, and performance measures. In case you are unfamiliar with the command line environment, then you can learn some of the basics in *Chapter 6*. In case you are struggling with command line instructions in the first five chapters, you may also consider reading *Chapter 6* first and then moving back to the set up of your server system.    

*Chapters 7* through *19* turn your general server into a Bitcoin server. We will set up five core services, including **Bitcoin Core**, **Fulcrum**, **BTC Explorer**, **C-Lightning**, and **Ride the Lightning**. We will use two main supporting services for privacy and security, **Tor** and **NGINX**. We will additionally set up the following user interfaces to our server:

* The **Sparrow** and **Specter** wallets on a local computer. These allow you to manage on-chain bitcoin funds.
* The web interface to our Bitcoin Explorer on a local computer. This allows you to explore on-chain bitcoin transactions and addresses. 
* The web interface to our Ride the Lightning server on a local computer. This allows you to manage your lightning node, as well as make and receive Lightning payments. 
* The **Zeus** wallet on your mobile phone. This allows you to make and receive Lightning payments on the go. 


## Instruction assumptions

To set up and manage your Bitcoin server, you will need a separate computer. I will refer to this computer as your ***managing computer*** or the ***remote computer*** throughout the guide. This can be your main personal computer, or any other convenient computer that is ***on the same local network*** as the Bitcoin server. You should use a computer from which you typically expect to make Bitcoin transactions. 

For the instructions, ***I assume that your managing computer has a Windows operating system***. It would be too complicated to also cater the instructions to Apple users. While the steps should be very similar, you have to fill in the details on your own.   

Your Bitcoin server will eventually just run on its own and be managed remotely. But during the initial set up of the server, you will need to have a screen, keyboard, and mouse available for it. 


## Target audience

This guide is really intended to be educational as much as practical. It caters primarily to students with a limited technical background. These can be undergraduate students in IT-related fields, as well as students from other fields. Throughout the discussion, we will spend a lot of time on ***why to do things*** rather than just ***how to do things*** precisely to help them. Particularly in the early chapters, we will share a lot of general background information to ensure accessiblity for people that are not so familiar with managing servers, Linux, and the command line environment. 

While intended for students with limited technical backgrounds, anyone is, of course, free to profit from the contents of this guide. In case you are an IT specialist that wants to become more familiar with Bitcoin, you can skip and skim much of the discussion, particularly in the early chapters. You are also welcome to make tweaks to the instructions as you see fit. In case you are a more advanced reader, you should, however, be aware of various content that caters towards more advanced audiences. Some great examples are *Raspibolt: Beginner’s Guide to Lightning on a Raspberry Pi*.<sup>[1](#footnote1)</sup> and the *402 Payment Required*<sup>[2](#footnote2)</sup> Youtube channel. 


## After the guide

This guide is really intended to be an educational journey more than anything else. Once you have created your homespun Bitcoin server, you have two options:

First, you can keep the server for your own practical use. We will really set up most of the functionality that you will need for your Bitcoin activities. You can also build it out with further services on your own. There are many further educational resources online that can help you with any expansions, such as the *Raspibolt* guide and *402 Payment Required* youtube channel mentioned above. There really are many other great resources available, so definitely do some research on your own.  

Second, you can choose to utilize a node or self-hosting solution instead. Particularly if you want to experiment with many different applications, these solutions will offer you a much more practical experience, even though they come at some costs as mentioned at the beginning of this chapter. In any case, having completed this guide, you will be in a much better position to understand how everything in such a node or self-hosting solution is working. 


## Expectations

We would like to level-set on two key expectations.

First, if you are on a tight bugdget, you can set up the server with practically zero costs as long as you can obtain some suitable hardware. That said, you will need to make some investments into setting up Liquidity for your Lightning node. Opening a channel for $10 is not recommended. What you should do very much depends on the level of on-chain fees in the network. But if opening a channel generally costs around a few dollars, you should probably invest at least $100 to $200 to opening a single channel to a well-connected node with low routing fees.  

Second, while managing your own money and running your own financial infrastructure is great for financial sovereignty, your Bitcoin server is inevitably going to be a less practical experience than using centralized solutions, even after it is up and running. While technical, product, and user experience developments promise improvements in this regard, a decentralized approach will likely always come at some costs in practicality. 

This is particularly true for Lightning. While we can set up Bitcoin Core with wallets and block explorer that will offer a fairly good user experience, this is more difficult to achieve with Lighting: instant payments is the goal, not quite the current reality. 


## Notes

<a name="footnote1">1</a>. Available at: https://stadicus.github.io/Raspibolt. 

<a name="footnote2">2</a>. Available at: https://www.youtube.com/channel/UC_62FowZPxGB6ysv4mcj20A. 