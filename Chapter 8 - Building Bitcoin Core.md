# Chapter 8. Build Bitcoin Core

We will now build Bitcoin Core from the source code. The further configuration and set up of Bitcoin Core will be completed in *Chapters 9* and *10*. At the time of writing, the latest release for Bitcoin Core was 25.1. If you have a newer release available, you should adapt the instructions below accordingly. 

Let's start with the question of why we should build Bitcoin Core in the first place. 

## Why build Bitcoin Core?

The main executables or binaries offered by Bitcoin Core are **Bitcoind** (the “Bitcoin daemon”), **Bitcoin-cli** (the “Bitcoin command line interface”), **Bitcoin-wallet**, and **Bitcoin-Qt**. 

* Bitcoind is a command-line application that allows you to run a node. 
* Bitcoin-cli is a command line application via which you can make requests of Bitcoind.
* Bitcoin-wallet is a command-line wallet application. 
* Finally, Bitcoin-Qt combines the functionality of these three applications into an application with a graphical user interface.

One option for obtaining these binaries on your system is to install them directly from the Bitcoin Core official website (https://bitcoincore.org/en/download/). For version 25.1, there are eight installation options: one installer for Windows, two installers for Mac operating systems (for x86 CPU architecture and ARM architecture), and five options for Linux systems.

* Linux (tgz): This contains a Bitcoin Core package for any Linux operating system which works with a processor that has an x86 architecture. 
* ARM Linux: This contains a Bitcoin Core package for any Linux operating system which works with a processor that has an ARM architecture. It comes in both a 32-bit and a 64-bit version. 
* RISC-V Linux: This contains a Bitcoin Core package for any Linux operating system which works with a processor that has a RISC-V architecture. 
* PPC64 Linux: This contains a Bitcoin Core package for any Linux operating system which works with a 64 bit big-endian PowerPC and Power ISA processors.
* Linux (Snapstore): This is a Snap file that will work on any Linux distribution. You download the Bitcoin Core Snap to your system, and just run it straight away.  

There are many guides online that will show you how to install these executables directly. Importantly, after downloading any of these, you will want to ensure that your download—whether a tarball, zip file, or snap file—is genuine by verifying the file through digital signatures made over a hash of the file. This verification process helps ensure that no errors were incurred in the download process and, more importantly, that no attackers tricked you into downloading a malicious file. 

You can also download a Bitcoin Core package with binaries from your APT repository. But we would recommend against this approach for two reasons. 

1. Any failure by the repository maintainers to remain up to date can pose issues for your Bitcoin server, particularly when the updates concern security issues. Watching the Bitcoin Core web site and their twitter account for new releases is a better practice (see https://twitter.com/bitcoincoreorg). 
2. Whenever Bitcoin has political moments—such as was the case during the Segwit Civil Wars in 2017—you can vote on your preferred protocols with your node. In this case, you do not want to rely on your APT for determining which version of Bitcoin Core you run. 

Another approach to obtaining the main binaries from Bitcoin Core is to build them from source code.<sup>[1](#footnote1)</sup> There are several potential advantages to building Bitcoin Core from the source code. 

* Building from the source code will allow you to make customizations to the software. 
* Building from the source code will allow you to experiment with the latest version of Bitcoin Core, not only the latest release.
* Building from the source code will allow you to run tests on the codebase.

So particularly for developers, building from source code has various advantages. But even if you are not a developer building from source code can be educational, so that is the path we will follow here. As everywhere else in this guide, we will assume that you are running Debian for the instructions. But if you are running a different Linux distribution, you can consult the build instructions in the documentation section of the code base for support [https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md](https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md).


## Preliminaries

It is always good to ensure that your system is up to date before making any changes. So if you have not updated your system in a while, be sure to first do that as follows:

* **`$ sudo apt update`**
* **`$ sudo apt upgrade`**

Next, you will need to ensure that you have all the relevant building tools installed on your system. From the terminal window, proceed as follows:

* **`$ sudo apt install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3`**

Once we have downloaded the source code from the Bitcoin Core Github repository, we will want to verify its authenticity. To do so, we will need to have **Git**, a version control system, and **GPG** (“GNU Privacy Guard”), an application for key management and signature validation, installed. Execute the following commands to ensure these applications are installed on your system:

* **`$ sudo apt install git`**
* **`$ sudo apt install gpg`**

To clone the Bitcoin Core repository from Github onto your system, proceed as follows: 

•	Move into your home directory.
•	From the main page of the Bitcoin Core repository, click on the “Local” tab. 
•	Copy the link for HTTPS, which should be https://github.com/bitcoin/bitcoin.git). 
•	Execute **`$ sudo git clone https://github.com/bitcoin/bitcoin.git`**

You should now see in your home directory a new directory called **`bitcoin`**. If you enter it, you will see the source code for Bitcoin Core. 

While anyone can contribute to Bitcoin Core, only a few of these contributors have commit access—that is, the ability to merge code changes into the master branch. This is nec-essary, of course, otherwise the repository would turn into complete chaos. 

All or some of the contributors with commit access will be active maintainers of the repository. Maintainers ensure that reviewed and approved code changes are indeed merged into the master branch. They also eliminate obvious spam pull requests. Not everyone with commit access is necessarily very active as a maintainer. 

You should always take steps to verify that any source code you download is trustworthy. This is fairly straightforward with Bitcoin Core as all commits onto the master branch are signed by the maintainer that was responsible for the merge. Currently, a tool is being developed to verify the entire commit history (https://github.com/bitcoin/bitcoin/tree/master/contrib/verify-commits). You can explore this tool on your own. Here we will merely verify the signature on the latest release which should be sufficient. 

You can view the fingerprints of the current people with commit access in the “trusted-keys” file in the “contrib” directory (https://github.com/bitcoin/bitcoin/blob/master/contrib/verify-commits/trusted-keys). A fingerprint is used to conveniently verify the legitimacy of a public key from multiple sources. As of the current date of writing, there are only five fingerprints listed in the file:

* Michael Ford: E777 299F C265 DD04 7930 70EB 944D 35F9 AC3D B76A
* Hennadii Stepanov: D1DB F2C4 B96F 2DEB F4C1 6654 4101 0811 2E7E A81F
* Andrew Chow: 1528 1230 0785 C964 44D3 334D 1756 5732 E08E 5E41
* Gloria Zhao: 6B00 2C6E A3F9 1B1B 0DF0 C9BC 8F61 7F12 00A6 D25C
* Ryan Ofsky: 4D1B 3D5E CBA1 A7E0 5371 EEBE 4680 0E30 FC74 8A66

You should first verify that the fingerprints listed in the “trusted-keys” file indeed belong to the purported identity. You can do this by matching the information in the file against other sources. 

To start, you can compare the fingerprints in the file against the fingerprints provided above. They should match. Additionally, you can consult other sources such as personal websites, Twitter accounts, and so on; often these will list people’s PGP fingerprints. You can also verify the accuracy of the fingerprints listed in the file against online PGP key databases such as the Ubuntu keyserver (https://keyserver.ubuntu.com).

This first verification process just is to confirm that a particular identity is coupled to a particular PGP key. It is a way to protect against, for example, an attacker that may have compromised one of the maintainer accounts and replaced one of the fingerprints with one of their own. If you find different fingerprints for the same identity, there is a problem and you should not trust the current Bitcoin Core repository. You should first figure out what is going on, and if any of the keys have been compromised. 

To verify source code releases, it is easiest just to have the PGP keys for all the people with committ access on your GPG keychain. To add the five public keys belonging to the fingerprints above, you can follow the process, mutatis mutandis, as done for Michael Ford’s public key below:

•	Move into the ~bitcoin/contrib/verify-commits directory
•	Execute $ sudo cat trusted-keys
•	Copy the key for Michael Ford and paste it into the following command: **`$ sudo gpg --keyserver keyserver.ubuntu.com --search E777299FC265DD04793070EB944D35F9AC3DB76A`**
•	You should see the key that belongs to Michael Ford
•	Enter “1” into the terminal and it will add the public key to your GPG key ring

If you now execute **`$ sudo gpg --list-keys`**, you should see all five keypairs as well as your own listed. By default, GPG assigns newly imported public keys a validity level of “unknown”. The validity level of a public key indicates your conviction about the trustworthiness of the key's owner. This is, of course a more thorny question: Can you actually trust these people not to commit erroneous or malicious code to the master branch? 

While you have probably never met any of the Bitcoin Core maintainers, you will see that each of them has a long history of contributing to the Bitcoin Github project. In addition, all pull requests are extensively reviewed and monitored by other contributors. Thus, any untoward behavior by maintainers typically would come to light quickly. So unless you see that fingerprints for the same identity from different sources do not match, you typically have little to worry about. 

If you want to be conservative in trusting code, one strategy you can apply is to use older releases of Bitcoin Core. This is, of course, only a good strategy if you know the release does not have known security issues (seccurity patches are only pulled into the two major releases before the current).    

Leaving the default validity level of "unknown" in place will produce warnings each time you use that public key for the verification of Bitcoin Core releases. To change this validity attribute, you will first need your own public-private key pair. Proceed as follows:

* Execute **`$ sudo gpg --full-generate-key`**
* Select an “RSA pair” (option 1)
* Set the security level at “4096 bits”
* Select an expiration date (a good option is two years, i.e., $ 2y)
* Provide your name and an e-mail (you can use public_key_verification@GPG.com for clarity). In addition, provide a comment of "Validity-level keypair" to indicate that this key pair is only for managing your server GPG public key ring and not for personal communication. You will not share any of this information publicly.
* Confirm with “O” for “okay”
* Choose a very secure password. Preferably this password is different from the one for your admin profile or your root profile. You need this password to access your private key and make digital signatures.
* Record the password for your private key.
* Type randomly on the keyboard until an RSA keypair has been generated.

For any private-public key pair you create, the validity levels are automatically set to ultimate. To change the validity level of the other public keys, you need to sign them with your own key you created earlier. You can follow the process, mutatis mutandis, as done for Michael Ford’s public key below:

•	Execute **`$ sudo gpg --sign-key E777299FC265DD04793070EB944D35F9AC3DB76A`**
•	Click “yes” to confirm
•	To finalize the signing process, enter the secure password to use the private key you created earlier

Once you have signed all the public keys, you can execute **`$ sudo gpg --list-keys`** again. You will see that the validity level for all the public keys has been changed to “full”. This has roughly the same meaning as “ultimate”. The only difference is that for the former type of public key you do not own the private key yourself.

Importantly, the list of active maintainers and those with commit access more generally has changed throughout Bitcoin’s history. So you may encounter a list of keys for commit access that does not match exactly what has been displayed above. In that case, you should adapt the instructions accordingly. 

As a final step, you should now verify the commit of the latest release. Move into the **`bitcoin`** directory. Executing **`$ sudo git tag -n | sort -V`**, you can see a list of tagged snapshots of the codebase. You should verify the latest snapshot which counts as an official release (so not a release candidate snapshot, as indicated by the “rc” at the end of the version). At the time of writing that is version 25.1, so the verification process would be done with the following command: 

* **`$ sudo git tag -v v25.1`** 

With regards to this particular version, you should see that the commit has a valid signature from Michael Ford. The valid signature may be different for other versions.  

Your current view of the bitcoin repository will be in its latest state, which may be unstable. You should now move the state of the repository into v25.1. Execute the following command:

* **`$ sudo git checkout v25.1`**


## Navigating the source code

To view the whole Bitcoin Core repository (for v25.1), move into the **`bitcoin directory`** and execute **`$ ls -a`**. This will show all files and directories, including both hidden and unhidden. The Bitcoin Core codebase is involved and it takes even seasoned developers a long time to become intimately familiar with it. But let’s at least obtain a high-level impression of how the codebase is organized.

The base directory includes several pieces of key documentation as well as configuration files and scripts required to build Bitcoin Core. You should familiarize yourself with the **`README`** and **`CONTRIBUTING`** documents to obtain a feel for the codebase and how changes are made. There are also various subdirectories within the base directory.

The **`build-aux`** directory contains all the macro files that help create the configuration file for the makefile (a file which provides the instructions for how to build the source code) during the build process.  

The **`build_msvc`** directory contains files required to build Bitcoin Core with msbuild or Visual Studio on a Windows system. One can also build Bitcoin Core for a Windows system on a Linux system using the Mingw-w64 cross-compiler toolchain. Alternatively, one can build Bitcoin core on the Windows Linux subsystem.

The **`ci`** directory contains scripts to automatically build and test the main branch of Bitcoin Core with a pull request before the integration of that pull request. The workflow configuration file is in the **`.Github`** directory. 

The **`contrib`** directory is one common in free and open source software projects. It includes various tools, particularly for developers, that are not a part of the core software project. There is a separate repository for tools that maintainers use in the Bitcoin Core Github organization (https://github.com/bitcoin-core/bitcoin-maintainer-tools). 

Bitcoin Core has numerous dependencies. You can build these dependencies in a straightforward manner by relying on your system’s package manager during the Bitcoin Core build process. However, you can also build these dependencies yourself. The **`depends`** directory contains all the required information. 

The **`doc`** directory houses many documents with information about the codebase. The subdirectories include **`design`** (several documents about the design of the source code), **`man`** (place-holder files that will be populated after building Bitcoin Core), **`policies`** (a collection of documentation about the policies that nodes maintain in relaying unconfirmed transactions), release notes (every major and minor release has release notes that summarize the main information about the release). Some key documents are outside of this directory, such as **`CONTRIBUTING`** and **`SECURITY`** which are in the main directory. 
 
The **`.github`** directory has various standard files that can be utilized within Github projects. It includes templates for issues and pull requests, as well as a workflow configuration which ensures Bitcoin Core is built and undergoes various tests before any pull request is merged. Various files in the root directory are also automatically processed for the layout of the project on Github (e.g., **`README.md`** and **`SECURITY.md`**).  

The **`share`** directory contains generally helpful files that are distributed with the source code as well as the binaries. After building Bitcoin Core, for example, the directory includes a sample configuration file. 

The **`src`** directory contains all the C++ source code for the project. The source code is used to create the main binaries such as bitcoind, bitcoin-wallet, and bitcoin-cli. It also contains the source code for the unit tests, as well as source code for the fuzz and util tests.

The **`test`** directory contains all the code for testing, not included in the **`src`** directory. This includes functional tests that examine the binary applications in their entirety (written in python and can be executed with the test-runner), the test-runners for the fuzz and util tests, and lint tests which can check your code for programmatic and stylistic errors before compilation. 

Bitcoin Core translations and localizations are managed using the Transifex platform. Anyone can create an account and make contributions to the translations/localizations via the platform. The configuration file (**`config`**) in the **`.tx`** directory maps the Bitcoin Core codebase into translatable components on the Transifex platform.  


## Build dependencies

Building Bitcoin Core requires various software dependencies. By relying on your system’s package manager during the build process, you cannot be sure that the versions it fetches for all these dependencies are fully compatible with the versions assumed by Bitcoin Core. So, there are advantages to building these dependencies yourself. That is what we will do here.

Move into the **`bitcoin/depends`** directory. Building the software dependencies assumes some further dependencies that may not all be installed on your system. You should at least ensure that **`curl`** and **`bison`** are installed by executing the following commands:

* **`$ sudo apt install curl`**
* **`$ sudo apt install bison`**

To start building the software dependencies for Bitcoin Core, now execute the following command:

	$ sudo make

The build process should last quite a while. In case you run into issues during execution, it is likely because certain commands are unrecognized due to missing packages. By reading the error messages, you can figure out which packages are missing and install them. Once installed, it is better to clean up the binaries by executing **`$sudo make clean`** first. Afterwards, you can re-execute **`$sudo make`** to build the dependencies successfully from scratch. 

Once the build process has finished, you should see a new subdirectory within the **`bitcoin/depends`** directory called **`x86_64-pc-linux-gnu`** (with different operating systems and architectures, you will see a differently named directory). This directory houses all the dependency binaries. These binaries should stay in this directory. 


## Building Bitcoin Core 

We are now ready to build Bitcoin Core. 

Move into the base directory of the bitcoin repository. Here you will see a file called **`autogen.sh`**. It contains a bash script that performs a number of actions. Most importantly, it will create a configuration script in the base directory called **`configure`**. This configuration script is, in turn, used to create a makefile. Execute the bash script **`autogen.sh`** as follows:

* **`$ sudo ./autogen.sh`**

After completion, you should see the **`configure`** file for your makefile in the base directory. It allows for various configuration options. To view them, execute the following command:

* **`$ sudo ./configure --help`**

By default, the configuration script ignores the dependencies you built in the previous section and fetches them with the package manager. So we need to ensure that the configuration script knows about their location and points to them in the makefile. In addition, all the main user binaries for Bitcoin Core will by default be stored in the **`/usr/bin directory`** at the end of the build process. For convenience, we will instead create these binaries in a **`bitcoin-core`** directory within our **`btc-server`** server directory. Proceed as follows:

* Move into the **`btc-server`** directory and execute **`$ mkdir bitcoin-core`**. You should not require the "sudo" modifier to make changes in this directory. 
* Move back into the **`bitcoin`** directory within your home directory. 
* Now execute **`$ sudo CONFIG_SITE=$PWD/depends/x86_64-pc-linux-gnu/share/config.site ./configure --bindir /btc-server/bitcoin-core/bin`**

After this process finishes, you should see a file called **`Makefile`** within the base directory. The next step is to build the source code. Execute the following command:

* **`$ sudo make`**

The make application will read the Makefile to (1) compile all the specified object code from the source code, (2) link all these object files together in multiple executables, and (3) create various data and configuration files. Subsequent executions of **`$ sudo make`** will only compile source files for which the source code has changed, and then re-create executables as necessary. The build process should last quite a while. 

Once finished, the executables are included in various places within the **`src`** directory. As the final step in the build process, you should execute the following command:

* **`$ sudo make install`**

If you now look in the **`/btc-server/bitcoin-core`** directory, you should see a **`bin`** directory that includes the following executables: **`bench_bitcoin`**, **`bitcoind`**, **`bitcoin-cli`**, **`bitcoin-tx`, **`bitcoin-util`**, **`bitcoin-wallet`**, **`test_bitcoin`**, and **`test_bitcoin-qt`**. 


## Notes

<a name="footnote1">1</a>. You will also find numerous other guides that walk you through building Bitcoin Core from source. Particularly if you get stuck at any point, you can consider utilizing these other guides. A good resource to also consult is Andreas Antonopoulos, Mastering Bitcoin, O’Reilly (Sepastopol, CA: 2017), pp. 32–37. 