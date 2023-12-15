# Connecting Bitcoin wallet applications

Now that you have a Bitcoin Core node, electrum server, and block explorer set up locally on your server, you can connect Bitcoin wallet applications to your server. While many wallets will work directly with your Bitcoin Core node, you will typically want to connect any wallets to an electrum server. This offers better performance. Some wallets can also make use of your local block explorer.

Connecting your Bitcoin wallet applications to your own infrastructure greatly improves your privacy. Connecting your Bitcoin wallets to public infrastructure means those who run the infrastructure can see all your transactions. In fact, many public electrum servers are known to be honeypots for chain analysis companies and other actors attempt to collect transaction data.

In this chapter, we will show how to connect the **Sparrow** and **Specter** wallet applications to your server, assuming these are installed on your managing computer. There are many other wallet software options available. Not all of these, however, are necessarily good options. When selecting wallet software for a desktop, you will typically want at least the following minimal features: 

* The ability to connect to your own Bitcoin Core node or Electrum server.
* Support for the major hardware wallets such as Trezor and Cold Card.  
* Easy ways for organizing and logically separating your funds.
* Support for multisignature addresses. 
* Coin selection ability for transactions. 
* And, of course, open source code that has been reviewed by others.  

The Sparrow and Specter desktop wallet applications at least meet these minimal standards. They are generally popular choices, though both cater to more advanced users. 

In principle, you can also connect to your Electrum server with wallets that are outside of your local network. One of the more secure and straightforward ways would be to run a hidden service for your Fulcrum server. Many wallet applications, both desktop and mobile, will, then, allow you to connect via a Tor hidden service circuit. 

But unless you have a very acute reason for connecting a non-local on-chain wallet in this manner, you will probably just want to forego this option. Typically, the better choice is just to use a lightning wallet that is connected to your server infrastructure for payments on the go and leave your Electrum server completely locked down locally. So we will not show how to connect non-local wallets here.

Let's start with the Sparrow Wallet.


## Sparrow Wallet

The maintainer of the Sparrow Wallet project is Craig Raw. His PGP public key fingerprint is as follows:

* D4D0 D320 2FC0 6849 A257 B38D E946 1833 4C67 4B40

You will first need to import this public key into Kleopetra on your managing computer. Proceed as follows:

* Verify the fingerprint of the public key against other sources such as Keybase and Ubuntu Keyserver.
* Create a text file on your desktop and name it **`craw.asc`**. 
* Copy the full PGP key from Keybase or Ubuntu Keyserver (so not just the fingerprint) and paste it into the **`.asc`** file.  
* On Kleopetra, import the public key and then certify it using your private signing key.
* Once the key has been imported and certified, you can delete the public key file from your desktop.

Next, travel to the Sparrow Github repository on your web browser (located at [https://github.com/sparrowwallet/sparrow](https://github.com/sparrowwallet/sparrow)). Move into the latest releases and download onto your desktop the Windows installer, manifest file, and file with the signature over the manifest. Note that a **manifest file** is simply a file that contains the hashes of all the released packages. To verify the executable, proceed as follows:

* Open the manifest file with a text editor and copy the hash for the Windows installer. 
* Open your MD5 and SHA checksum utility. Paste the hash you found for the Windows installer at the bottom. Then browse for the Windows installer on your desktop. Once all the hash values are calculated, select **`Verify`**. There should be a match.
* If the previous step was successful, open Kleopetra, select **`decrypt/verify`**, and select your manifest file. Kleopetra will automatically recognize the accompanying signature file and validate the signature against the manifest file. 

You should receive an output that the signature if from Craig Raw and valid. ***If not, you must first figure out why you are not obtaining a valid signature. You never want to install wallet software that you were not able to verify.*** 

Assuming the verification process succeeded, you can now delete the manifest file and the signature file from your desktop. 

Next, you can launch the installer. You should ignore the warnings from Windows, as you have verified the software manually. A shortcut for Sparrow should now appear on your desktop. You can see where the binary is located on your system by hovering over the shortcut. At this point, you no longer need the installer and can delete it.

You can now open Sparrow. After opening the application, you need to make some configuration changes. 

* Go to **`File`** and click on **`Preferences`**
* Select the **`Server`** option
* Select **`Private Electrum`**
* For the URL, you need to offer the IP address for your server: **`192.168.2.150`**. You also need to fill in the NGINX proxy port, namely **`50002`**
* Toggle on the **`ssl`** option
* At the end of the previous chapter, you left the **`NGINX.crt`** on your desktop. You should now cut and paste it to the following directory: **`C://Users/[UserName]/Appdata/Local/Sparrow`**. (In case you don't have the certificate anymore, just recreate it as explained in *Chapter 14*.)
* For the certificate to use with the SSL connection, you can now select the **`NGINX.crt`** from the Sparrow directory above.
* * Push the **`Test connection`** button

You should now see that you are connected to your Fulcrum server. It will make the same connection each time the application is started. If you need to change the connection details, you can come back at a later time and use the **`Edit Existing Connection`** option. 

Any address and transaction data you need is now retrieved from your own Electrum server. Any requests are over a secure SSL connection via your NGINX proxy. Alternatively, you could have also connected to Bitcoin Core, instead of your Electrum server. But that would have required setting up RPC API credentials and configuring Bitcoin Core to also operate over the NGINX proxy. In addition, any information about addresses on an imported wallet would be retrieved much more slowly. Hence, it makes more sense to connect to your Electrum server. 

As a final step, move back into the **`Preferences`** section, but this time select the **`General`** configuration options. You should also change the settings for your block explorer as follows:

* Select **`Custom`**
* Enter **`https://192.168.2.150:3003`**

For any transaction into a wallet, you can right click on it and select **`View Transaction`**. A separate tab with transaction information will open. You should see a button in the tab that will allow you to explore your transaction in the locally set up block explorer. 

Any public information you now use in your Sparrow wallet such as fee rate estimation is not really privacy-sensitive. You can now further explore the Sparrow wallet on your own. There are many good resources online. One source we generally recommend for walkthroughs of wallet application is the *BTC Sessions* youtube channel.<sup>[1](#footnote1)</sup> 


## Specter

Another good wallet application you can try out is Specter. 

To start, you should import the PGP public key for Specter releases into Kleopetra. The key fingerprint is as follows:

** 785A 2269 EE3A 9736 AC1A 4F4C 864B 7CF9 A811 FEF7

To import this public key into Kleopetra on your managing computer, proceed as follows:

* Verify the fingerprint of the public key against other sources such as Keybase and Ubuntu Keyserver.
* Download the full key from Ubuntu Keyserver (so not just the fingerprint) onto your desktop.   
* On Kleopetra, import the public key and then certify it using your private signing key.
* Once the key has been imported and certified, you can delete the public key file from your desktop.

Next, travel to the Specter Github repository on your browswer (located at [https://github.com/cryptoadvance/specter-desktop](https://github.com/cryptoadvance/specter-desktop)). Move into the latest releases section of the repository. You can download the Windows installer, and the **`SHA256SUMS`** and **`SHA256SUMS.asc`** files onto the desktop of your managing computer. To verify the installer, proceed as follows:

* Open the **`SHA256SUMS`** file and copy the hash for the Windows installer.
* Open your MD5 and SHA checksum utility to verify the hash within the **`SHA256SUMS`** file against the calculated hash of the Windows installer. 
* If the hashes match, open Kleopetra, select **`decrypt/verify`**, and select the **`SHA256SUMS.asc`** file. Kleopetra will just verify the signature against the hash data file. 

You should receive an output that the signature is a valid one from the Specter Signer. ***If not, you must first figure out why you are not obtaining a valid signature. You never want to install wallet software that you were not able to verify.*** 

Assuming the verification process succeeded, you can now delete the hashes and signature files from your desktop. 

Next, you can launch the installer. Once the setup is completed, delete the installer and open the Specter application. Proceed as follows: 

* Your welcome screen should allow you to connect to either your own Electrum server or your own Bitcoin node. Select the **`Electrum Connection`** option. 
* Select **`Enter my own`**.
* For the **`Host`** fill in **`192.168.2.150`**.
* For the **`Port`** fill in **`50002`**.
* Make sure you select the **`Use SSL`** option
* Click **`Connect`**

You should shortly see a successful connection to your Electrum server via the NGINX proxy. Anytime, you can check the status of your connection at the top left corner of your application. 

Next, move into the **`Settings`** section and select the **`General tab`**. Under **`Data sources`**, you will see that you can select a **`Custom`** block explorer. Having set this option, offer the following details for the URL: **`https://192.168.2.150:3003`**. 

With these settings, all your transactions will have a high degree of privacy. You can now further explore the Specter wallet on your own. Again, there are many good resources online to support you. 


## Notes

<a name="footnote1">1</a>. Available here: https://www.youtube.com/@BTCSessions/featured. 