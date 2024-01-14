# Appendix: Server Record Template

This overview contains all the details regarding my Bitcoin Server.

## General

Host name: 
Static IPv4 address server: 
Router and DNS address: 
Static IPv4 address managing computer: 
Bitcoin services directory: 

Notes. The Bitcoin services directory contains all the main data, binaries, and scripts associated with Bitcoin-specific applications. Configuration files are typically placed in the **`/etc`** directory and service files in the /etc/systemd/system directory. Some source code for server applications is also placed in the **`/home/administrator`** directory. Various Lightning-related data, binaries, scripts, and dependencies are also placed within the **`/home/lightningd directory`**. 

## Administrative accounts

Root password: 
Administrator user name: 
Administrator user account: 
Administrator user account password: 
Sudo validation key password: 
SSH server port: 

Notes. The administrator account is the main account for server configuration and maintenance. All the other accounts are used as service accounts. They typically do not have a home directory or a login option.   

## Bitcoind

Bitcoind JSON-RPC API: 
BitcoindP2Pinbound: 
Router mapping port: 
Hidden service: 
Configuration file: 
System service file: 

## NGINX 

Local CA private key:  
Private key password: 
Self-signed certificate directory: 

## Fulcrum

Fulcrum listening address: 
Fulcrum NGINX reverse proxy: 
Fulcrum-admin listening address: 

Notes. The Fulcrum server listens internally for connections. Any external data requests are run through the NGINX proxy and secured by TLS.  

## BTC Explorer

BTC Explorer listening address: 
BTC Explorer NGINX reverse proxy: 

Notes. The BTC Explorer server listens internally for connections. Any external data requests are run through the NGINX proxy and secured by TLS.  

## Core Lightning

Inbound peer connections listening address: 
Local Router mapping port: 
CLN-REST API listening address: 
CLN-NODEJS-REST API listening address:

## Ride the Lightning

Access password: 
RTL listening address: 127.0.0.1:3000
RTL NGINX reverse proxy: 0.0.0.0:3001

Notes. The RTL server listens internally for connections. Any external data requests are run through the NGINX proxy and secured by TLS.  