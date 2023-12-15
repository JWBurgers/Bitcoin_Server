# Further exploring Bitcoin Core

In this chapter, we will offer a few more clarifications about Bitcoin Core data storage and the source code. 


## Exploring Bitcoin Core data

It is good to have a basic understanding of where and how data is stored within the Bitcoin Core data directory.

To start, move into the **`bitcoin-data`** directory and find the **`blocks`** directory. As Bitcoin Core automatically sets permissions for these directories, you will not be able to access it with the **`administrator`** profile. Therefore, first switch to the **`root`** profile before entering the directory. 

If you enter into the **`blocks`** directory, you will see the following: 

* Many **`blkxxxxx.dat`** files
* Many **`revxxxxx.dat`** files
* An index directory

The **`blkxxxx.dat`** files contain blocks in a raw data format. These include blocks that are part of the Block Chain, as well as **stale blocks**: that is, blocks that form a side-branch to your version of the Block Chain. 

Peers only share blocks with you that are included in their version of the Block Chain. So when you initially start synchronizing with the network, you will not receive any stale blocks. As you come closer to the tip of the Block Chain, however, the more likely your peers are going to have different views on the Block Chain. At this point, you might also begin receiving stale blocks that you store in the **`blkxxxxx.dat`** files. Once synchronized, you will continue to receive stale blocks once in a while when miners discover a new valid block at roughly the same time. 

As the **`blkxxxxx.dat`** files are in a binary format, you cannot just open them with a text editor. But you can view one of the files in hexadecimal format with the following instruction: **`$ od -x An blkxxxxx.dat`**.

For each **`blkxxxxx.dat`** file, there is a corresponding **`revxxxxx.dat`** file. For each block in a **`blkxxxxx.dat`** file, a record of the unspent transaction outputs is included in the corresponding **`revxxxxx.dat`** file.

Finally, the index directory contains a LevelDB database with metadata about where to find particular blocks in the **`blkxxxx.dat`** files. Without this database, searching for any block on your system would be very slow. The index does not just index blocks for the Block Chain, but for the entire Block Tree stored within your **`blkxxxx.dat`** files (that includes your actual version of the Block Chain as well as branches populated by stale blocks). 

Your node will have a block tree that is different from most other active nodes. Consensus does not require a consistent view on the block tree. Instead, however, your node will generally have the same view of the Block Chain as other active nodes. Any discrepancies are usually resolved rather quickly by the consensus rules on the network.   

Next, move to the **`chainstate`** directory in the **`bitcoin-data`** directory. The contents are a LevelDB database which stores all the unspent transaction outputs (UTXOs). Each time new valid blocks are added to the Block Chain, this database is updated. As UTXOs are sometimes referred to as coins, sometimes this database is also referred to as the **coins database**. 

With the unspent transaction outputs database, your node can easily verify the validity of a new transaction it receives from its peers. If the transaction references a UTXO which is not in the database, your node will reject it. If it references a UTXO which is in the database, your node will continue performing other validity checks. The same principle applies to the transactions in new blocks your node receives from peers. 

Without the unspent transaction outputs database, your node would have to search through the **`blkxxxxx.dat`** files directly to verify the validity of new blocks and transactions. This would take an extremely long time.

If your node receives a valid new block that builds on a different history than your current version of the Block Chain and this alternate history has more proof of work behind it, Bitcoin Core will have to perform what is known as a Block Chain reorganization. A key element in that process is updating the unspent transaction outputs database. This is done with the information from the **`revxxxxx.dat`** files. 

Next, move into the **`indexes`** directory within your **`bitcoin-data`** directory, and then subsequently into the **`txindex`** directory. This contains a levelDB database which has metadata about where to find particular transactions in your **`blkxxxxx.dat`** files. This transaction index will allow you to easily query transactions on the basis of their ID with a local block explorer. It also often required for running an electrum server, including **Fulcrum**. 

Within the **`bitcoin-data`** directory, there are also various files of interest. The most important ones are as follows: 

* The **`mempool.dat`** file contains all the valid transactions your node has processed, which have not yet been included into a valid block. When a new valid block is found, all transactions from your mempool that are also in the block are removed. 
* The **`onion_v3_private_key`** file, which contains the private key for your onion service configured in the last chapter.
* The **`peers.dat`** file stores information on your active connections as well as other peers on the network.
* The **`debug.log`** we have already explored and contains the log for Bitcoind.


## Source code

Bitcoin Core is a free and open source project that relies on contributors from all around the world. Anyone interested in freedom money is highly encouraged to join the project. There are three ways to contribute directly to the codebase for Bitcoin Core.

1.	Code review
2.	Code testing
3.	Suggesting code changes

Suggested code changes can cover refactoring, debugging, and new features, though any refactoring should always be done only out of necessity for facilitating debugging or new features. Code change suggestions must, without exception, be made via **pull requests**. Any pull requests are subjected to code review and testing by peers. Many pull requests are rejected or withdrawn due to feedback from peers. Successful pull requests often require substantial discussion and revisions, particularly when introducing new features. 

We will not enter into the details of this governance process here. But do be aware that there are, in fact, two repositories, not one, for Bitcoin Core source code: at **`[github.com/bitcoin-core/gui](github.com/bitcoin-core/gui)`** and **`[github.com/bitcoin/bitcoin](github.com/bitcoin-core/gui)`**. We have been using only the latter in this guide. 

The former should be utilized for creating and updating pull requests, as well as discussion with regards to the GUI application within the Bitcoin Core package. The latter should be used for any other proposed changes to the Bitcoin code. Any finalized pull request is merged within the **`github.com/bitcoin/bitcoin`** repository of the codebase and mirrored within the **`github.com/bitcoin-core/gui`** repository. 

Why are there two repositories and why do they have this workload separation?

In 2013, the Bitcoin-Qt client was rebranded as Bitcoin Core. The purpose of this change was to more clearly separate the Bitcoin Network from the reference implementation. <sup>[1](#footnote1)</sup>

At this time, the Bitcoin Core organization was also created on Github. It seems originally to have been intended for storing Bitcoin Core project-specific materials. But over time, it has become the default location for new repositories related to Bitcoin Core.<sup>[1](#footnote2)</sup> The original bitcoin organization was maintained, however, for fear of breaking links and general familiarity with the location of the code base. 

In 2020, the decision was made to subdivide the workload between the two repositories as described above. This had several reasons. It is difficult to pro-actively ignore conversations and pull requests for certain parts of the codebase, and issues can only be hidden by filtering from the default view. So practically having two code bases was thought to be easier. In addition, separating them out thematically was to though to make for more focused review. <sup>[3](#footnote3)</sup> 


## Notes

<a name="footnote1">1</a>. You can find the main discussion in the following issue: see https://github.com/bitcoin/bitcoin/issues/3203. 

<a name="footnote2">2</a>. See https://bitcoin.stackexchange.com/questions/119054/why-do-we-need-github-com-bitcoin-core-when-we-already-have-github-com-bitcoin. 

<a name="footnote3">3</a>. You can find the main discussion within this pull request: https://github.com/bitcoin/bitcoin/pull/19071. 