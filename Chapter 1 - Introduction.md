# Chapter 1. Introduction

Many Bitcoin users run a server to support their Bitcoin activities. Typically, these servers are set up with the support of **node solutions** or **self-hosting solutions**. The former includes software solutions, often with a hardware component, that focus on supporting your bitcoin activities. The latter are software solutions, also often with a hardware component, that focus self-hosting activities more generally such as running your own cloud service, web server, and so on. There are many node and self-hsoting solutions on the market, including from the following companies and projects: Fulmo, Start 9, Nodl, Umbrel, Lightninginabox, and MyNode.  

While node and self-hosting solutions can surely offer a nice and convenient Bitcoin server solution for many people, building a Bitcoin server from the ground up—or at least, somewhat more from the ground up—does have various advantages. Specifically, we can identify at least six advantages to a **homespun Bitcoin server**.      

1. It teaches you much more about how Bitcoin works and how to protect your financial sovereignty and security. 
2. It offers you more control in software and hardware choices.
3. Software problems are easier to troubleshoot with a homespun Bitcoin server. 
4. Node and self-hosting solutions inject another layer of trust into your Bitcoin server. You have to trust the creators, for instance, with implementing legitimate versions of a full node application and keeping it updated. Most node and self-hosting solution projects have a very limited number of dedicated contributors, so using them clearly introduces additional risks.
5. Your full node application is your voting right within the Bitcoin ecosystem. If Bitcoin is facing a constitutional moment—such as it did with the Segwit Civil War in 2017—you want to determine the version of the software you run for your transactions. There is no guarantee a node solution or self-hosting solution will offer support for your convictions.  
6. A homespun Bitcoin server is typically more cost efficient.    

This guide offers a recipe for making a basic homespun Bitcoin server. Your server will run 24/7 with core services that are configured to run automatically (i.e., as Linux services). These core applications include a Bitcoin Core node, the Electrs electrum server, and the server for a local block explorer called BTC RPC Block Explorer. We will also provide you with the information on how to run a standard Tor node or a Tor hidden service if you require it. The Bitcoin server is connected to Sparrow, a popular desktop wallet, on a remote computer. It is also connected to a client application for the BTC RPC Block Explorer on the remote computer.

You can build out this basic Bitcoin server with further applications on your own. If you are keen to get onto Lightning, *Appendix A* will show how to install a Core Lightning node and connect it to external wallets. There are also many resources online that can help you further for including other applications. To start, we recommend checking out Ministry of Nodes<sup>[1](#footnote1)</sup> and 402 Payment Required<sup>[2](#footnote2)</sup> Youtube channels. We would also recommend consulting the guide by Stadicus called “Raspibolt: Beginner’s guide to Lightning on a Raspberry Pi”.<sup>[3](#footnote3)</sup> There really are many other great resources available, so definitely do some research on your own.  

## The guide in more detail

The first part of this guide is dedicated to setting up your server system. You have the freedom to choose whatever hardware you like for building your Bitcoin server: an old laptop or pc, a single-board computer such as a Raspberry Pi, and so on. *Chapter 2* offers some general guidelines and tips on choosing appropriate hardware. In **Chapter 3**, we will walk through the steps for installing the Debian operating system and performing some basic setup tasks. This chapter also includes instructions for mounting the drive for your bitcoin server in case you want to use multiple drives within your system. *Chapter 4* sets up the remote connection from a managing computer to your Bitcoin server. In *Chapter 5*, we will take some final security and performance measures. 

If you are unfamiliar with the Linux command line environment, then you can learn some of the basics in *Chapter 6*.   

In the third part of this guide, we will turn to the core services for your Bitcoin server and the relevant client applications on a remote computer. In **Chapter 7**, we will walk through the considerations and steps for configuring Tor on your system. We will build and install Bitcoin Core in *Chapter 8*. In *Chapter 9*, we will build and install Electrs. *Chapter 10* offers instructions for building and installing the BTC-RPC block explorer and connecting it to a client application on a remote computer. Finally, in *Chapter 11*, we will connect a Sparrow wallet on a remote computer to your Bitcoin server.


## Instruction assumptions

To help set up and manage your Bitcoin server, you will need a separate computer. I will refer to this computer as your **managing computer** or **remote computer** throughout the guide. This can be your main personal computer, or any other convenient computer that is ***on the same local network*** as the Bitcoin server. You should use a computer from which you typically expect to make Bitcoin transactions. 

For the instructions, ***I assume that your managing computer has a Windows operating system***. It would be too complicated to also cater the instructions to Apple users. While the steps should be very similar, you have to fill in the details on your own.   

Your Bitcoin server will eventually just run on its own and be managed remotely. But during the instructions, you will need to have a screen, keyboard, and mouse available for your Bitcoin server. 


## Target audience

If this is the first time you are setting up a server and are not familiar with the command-line environment, building applications from source code, and so on, then this guide will probably be a serious challenge. Nevertheless, you should be able to complete it with some elbow grease. If you are a novice, please remember the following:

* Many of the challenges you will encounter can probably be resolved with a Google search. 
* Throughout the discussion, we will spend a lot of time on ***why to do things*** rather than just ***how to do things*** precisely to help you. 
* If you become stuck in part one of the guide because of the command line instructions, you might be helped by reading the background on the command line environment in *Chapter 6* first.   
* You have a lot of freedom with regards to how closely you follow these instructions. If you are a novice, however, we would highly recommend following them as closely as possible. 

If you are a more advanced reader, then you can skip or skim much of the discussion in this guide, particularly in the first two parts. You are also welcome to make adaptations to the instructions as you see fit. You are welcome, for example, to try to create the Bitcoin server on Tails OS instead of Debian. Just remember that you are on your own with issues and troubleshooting. 


## Notes

<a name="footnote1">1</a>. Available at: https://www.youtube.com/c/MinistryofNodes.

<a name="footnote2">2</a>. Available at: https://www.youtube.com/channel/UC_62FowZPxGB6ysv4mcj20A. 

<a name="footnote3">3</a>. Available at: https://stadicus.github.io/Raspibolt. 