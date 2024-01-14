# A Homespun Bitcoin Server

## Project description

This repository contains a guide to building a Bitcoin server from the ground up—that is, without the support of a prepackaged node solution such as myNode, Nodl One, Umbrel, and Raspiblitz. The server will include five core services, including a Bitcoin node (Bitcoin Core), an electrum server (Fulcrum), a local block explorer (BTC Explorer), a Lightning node (Core Lightning), and a server for an interface to your Lightning node (Ride the Lightning). We will additionally set up the following user interfaces to our server:

* Wallets on a local computer that allow you to manage on-chain bitcoin funds (Sparrow and Specter)
* A web interface to our Bitcoin Explorer on a local computer. This allows you to explore on-chain bitcoin transactions and addresses. 
* A web interface to our Lightning node interface server. This interface allows you to manage your lightning node, as well as make and receive Lightning payments. 
* A wallet on your mobile phone, so that you can make and receive Lightning payments on the go. 

If desired, you can build out this Bitcoin server with further applications on your own.  


## Target audience

Node solutions such as Fulmo, Start 9, Nodl, Umbrel, Lightninginabox, and MyNode, offer a nice and convenient product for many people. Nevertheless, building your own Bitcoin server from the ground up—or at least, somewhat more from the ground up—does have various advantages. This guide offers a recipe for making a homespun Bitcoin server that caters to less technical audiences. We spend a lot of time on ***why to do things*** rather than just ***how to do things***. Particularly in the early chapters, we will share a lot of general background information to ensure accessiblity for people that are not so familiar with managing servers, Linux, and the command line environment. 

While intended for students with limited technical backgrounds, anyone is, of course, free to profit from the contents of this guide. In case you are an IT specialist that wants to become more familiar with Bitcoin, you can skip and skim much of the discussion, particularly in the early chapters.


## Contributions

Please have a look at the contributions file in the repository for some guidelines on how to support the project.