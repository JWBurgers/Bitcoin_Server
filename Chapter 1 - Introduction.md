# Chapter 1. Introduction

Many Bitcoin users run a home server to support their Bitcoin activities. Typically, these servers are set up with the support of **node solutions** or **self-hosting solutions**. The former include software solutions, often with a hardware component, that focus on supporting your bitcoin activities specifically. The latter are software solutions, also often with a hardware component, that focus on self-hosting activities more generally (e.g., running your own cloud service, web server). There are many node and self-hosting solutions on the market, including from the following companies and projects: Fulmo, Start 9, Nodl, Umbrel, Lightninginabox, and MyNode.  

While node and self-hosting solutions can surely offer a nice and convenient Bitcoin server solution, building a Bitcoin server from the ground up—or at least, somewhat more from the ground up—does have various advantages. Specifically, we can identify at least six advantages to a ***homespun Bitcoin server***.      

1. It teaches you much more about how Bitcoin works and how to protect your financial sovereignty and security. 
2. It offers you more control in software and hardware choices.
3. Software problems are easier to troubleshoot with a homespun Bitcoin server. 
4. Node and self-hosting solutions inject another layer of trust into your Bitcoin server. You have to trust the creators, for instance, with implementing legitimate versions of a full node application and keeping it updated. Most node and self-hosting solution projects have a very limited number of dedicated contributors, so using them clearly introduces additional risks.
5. Your full node application is your voting right within the Bitcoin ecosystem. If Bitcoin is facing a constitutional moment—such as it did with the Segwit Civil War in 2017—you want to determine the version of the software you run for your transactions. There is no guarantee a node solution or self-hosting solution will offer support for your convictions.  
6. A homespun Bitcoin server is typically more cost efficient.    

This guide offers a recipe for making a homespun Bitcoin server. Your server will run 24/7 with core services that are configured to run automatically, including a Bitcoin node, an electrum server, a block explorer server, and a Lightning node. The applications we will use for these services, respectively, are **Bitcoin Core**, **Fulcrum**, **BTC RPC Explorer**, and **C-Lightning**. We will also use some supporting services for privacy and security next to these core services, including **Tor** and **NGINX**. Finally, you will have wallet applications on one or more remote devices to utilize your server. We will show how to use **Sparrow** from a local computer and **Phoenix** from a mobile phone. 


## The guide in more detail

This guide consists of three main parts. 

The first part is dedicated to setting up your server system. You have the freedom to choose whatever hardware you like for building your Bitcoin server: an old laptop or pc, a single-board computer such as a Raspberry Pi, and so on. *Chapter 2* offers some general guidelines and tips on choosing appropriate hardware. In **Chapter 3**, we will walk through the steps for installing the Debian operating system and performing some basic setup tasks. This chapter also includes instructions for mounting drives in case you want to use multiple drives within your system. *Chapter 4* sets up the remote connection from a managing computer to your Bitcoin server. In *Chapter 5*, we will implement some final general security and performance measures. 

At this point in the guide, you will have already seen quite some Linux command line instructions. If you are unfamiliar with the command line environment, then you can learn some of the basics in *Chapter 6*. In case you are struggling with the instructions in the first part of the guide, you may consider reading *Chapter 6* first and then moving back to part one.    

In the third part of this guide, we will turn to the core and supporting services for your Bitcoin server and the connection of wallets from other computers. In **Chapter 7**, we will examine the considerations and steps for installing Tor on your system. We will, then, spend several chapters on Bitcoin Core, the heart of your server. We build Bitcoin Core from source code in *Chapter 8*. In *Chapter 9*, we will set up **Bitcoind**—the node application that comes with the Bitcoin Core executables—as a Linux service. In *Chapter 10*, we configure Bitcoind and run the service. In *Chapter 11*, we will further explore Bitcoin Core. *Chapter 12* is dedicated to setting up Fulcrum as a service. In *Chapter 13*, we will show how to configure NGINX as a proxy for Fulcrum, so that all connections to it use transport layer security. *Chapter 14* offers instructions for setting up the BTC-RPC block explorer. In *Chapter 15*, we will connect the Sparrow wallet from a local computer to your server. We set up C-Lightning and connect to it from the Phoenix wallet on a mobile phone in *Chapters 16* and *17*, respectively. Chapter **18** explains how to keep your system updated. 


## Instruction assumptions

To help set up and manage your Bitcoin server, you will need a separate computer. I will refer to this computer as your ***managing computer*** or the ***remote computer*** throughout the guide. This can be your main personal computer, or any other convenient computer that is ***on the same local network*** as the Bitcoin server. You should use a computer from which you typically expect to make Bitcoin transactions. 

For the instructions, ***I assume that your managing computer has a Windows operating system***. It would be too complicated to also cater the instructions to Apple users. While the steps should be very similar, you have to fill in the details on your own.   

Your Bitcoin server will eventually just run on its own and be managed remotely. But during the initial set up of the server, you will need to have a screen, keyboard, and mouse available for it. 


## Target audience

This guide is really intended to be educational as much as practical. It caters primarily to students with some technical background, but with limited knowledge of the command-line environment, building applications from source code, and so on. For such students, the guide is probably a serious challenge. Nevertheless, they should be able to complete it with some good elbow grease. If you are a novice, please remember the following:

* Many of the challenges you will encounter can probably be resolved with a Google search. 
* Throughout the discussion, we will spend a lot of time on ***why to do things*** rather than just ***how to do things*** precisely to help you. 
* If you become stuck in part one of the guide because of the command line instructions, you might be helped by reading the background on the command line environment in *Chapter 6* first.   
* You have a lot of freedom with regards to how closely you follow these instructions. If you are a novice, however, we would highly recommend following them as closely as possible. 

More advanced readers such as seasoned developers can skip and skim much of the discussion in this guide, particularly in the first two parts. They are also welcome to make adaptations to the instructions as they see fit. While more advanced readers are certainly welcome, they should be aware that there are various guides in existence already that cater towards more advanced audiences. A great example is *Raspibolt: Beginner’s Guide to Lightning on a Raspberry Pi*.<sup>[1](#footnote1)</sup>


## After the guide

This guide is really intended to be an educational journey more than anything else. Once you have created your server, you have two options:

First, you can keep the server for your own practical use. We will really set up most of the functionality that you will need for your Bitcoin activities. You can also build it out with further services on your own. There are many further educational resources online that can help you with any expansions. As a start, you can check out the "Ministry of Nodes"<sup>[1](#footnote1)</sup> and "402 Payment Required"<sup>[2](#footnote2)</sup> Youtube channels. As mentioned above, we also highly recommend the guide by Stadicus called *Raspibolt: Beginner’s Guide to Lightning on a Raspberry Pi*. There really are many other great resources available, so definitely do some research on your own.  

Second, you can choose to utilize a node or self-hosting solution instead. Particularly if you want to experiment with many different applications, these solutions will offer you a much more practical experience, even though they come at some costs as mentioned at the beginning of this chapter. In any case, having completed this guide, you will be in a much better position to understand how everything in such a node or self-hosting solution is working. 


## Notes

<a name="footnote1">1</a>. Available at: https://stadicus.github.io/Raspibolt. 

<a name="footnote2">2</a>. Available at: https://www.youtube.com/c/MinistryofNodes.

<a name="footnote3">3</a>. Available at: https://www.youtube.com/channel/UC_62FowZPxGB6ysv4mcj20A. 