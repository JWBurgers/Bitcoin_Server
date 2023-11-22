# Chapter 2. Hardware selection

You can create a Bitcoin server on all types of computers, including desktops, laptops, single-board computers, phones, and dedicated hardware servers. Whether a computer is capable of handling your Bitcoin server depends on your choice of software applications and their configurations, as well as the hardware capabilities of your computer.

We will not prescribe any particular hardware system and leave you free to make your own choices. Instead, this chapter merely contains some general considerations and tips on selecting hardware that meets the requirements.


## Disk space

You should definitely set up Bitcoin Core, Electrs, and other such purely Bitcoin-related services and store all their associated data on a single drive. You can do this on a dedicated Bitcoin drive on your server, or on a more comprehensive drive that also includes your operating system and other applications and data (e.g., Tor, Fail2Ban, SSH). 

Whatever approach you take, the key variable you will need to consider in your hardware choices is storage space. We will set up Bitcoin Core with an entire history of the Block Chain, which is currently just over 500 GB (November 2023). In addition, we will set up Bitcoin Core with a transaction index. A full Block Chain with a transaction index is really a prerequisite for many applications that you might eventually run on a Bitcoin server, including an electrum server and a local block explorer. Having the entire Block Chain stored on your system also provides additional benefits. 

Given that the diskspace requirement for the Block Chain continuously grows—about 50 to 100GB per year—and that we will also create additional services on your Bitcoin server (electrum server and local block explorer), you will definitely need to have ample diskspace for this exercise. We would recommend a system with a minimum of 1TB in diskspace just for a dedicated Bitcoin drive, and would strongly recommend going higher if you plan to use this server for a few years. Anything over 2.5TB is probably overkill, unless you have really bold intentions for your Bitcoin server (such as running an ElectrumX server or Blockstream’s Esplora). 

These size indications will probably also work for you if you are using a more comprehensive drive to store your Bitcoin applications and associated data (i.e., one that also includes, for example, the operating system and other applications and data). At least, as long as you do not do too much else with your Bitcoin server than what is covered in this guide.

Although traditional hard drives (HDDs) are cheaper, using solid state drives (SSDs) to store your Bitcoin applications and data has a number of advantages.   

1. SSDs make practically no noise. HDDs do, though how much depends on the particular HDD. If you intend to keep your Bitcoin server in your living room, a home office, or a bedroom, then an SSD is a safer bet. 
2. SSDs generally have much higher read and write speeds than HDDs. Although many relatively new HDDs will probably work for Bitcoin Core, a good SSD has higher speeds than you will ever need. The higher speeds ensure that your node initially syncs with the Block Chain must faster. 
3. Although the subject of some controversy, generally SSDs seem more power efficient than HDDs. 

Ideally, you would use an SSD with a PCI express connection—that is, a "peripheral component interconnect express" connection—for your Bitcoin server applications and data (assuming your system can support it). Such an SSD works with PCIe 3.0 or higher. These types of SSDs are known as **NVMe SSD drives**—that is, non-volatile memory express SSD drives—and have the **M.2 form factor**. An NVMe SSD will have by far the highest reading and writing speeds. 

That said, NVMe SSD drives are very expensive and you may not have the option for a system that supports it. In that case, a standard **SATA SSD** of 1 or 2TB for your Bitcoin server applications and data would be a great option, and what most people typically would use. (SATA stands for "serial advanced technology attachment".) A SATA SSD will still give you very good performance, much better than a traditional HDD.

In case you use a dedicated drive for your Bitcoin applications and data, then your will still need a separate drive for your operating system and other applications and data. An SSD is also the best option here, particularly for running your operating system. 

Many people run Bitcoin servers from **single-board computers** such as **Raspberry Pis**, **Odroids**, or **Rocks**. With these credit-card sized computers the operating system is often installed on a (micro) SD card. That type of memory card is typically much less robust than a quality SSD or even a traditional HDD. If you are planning on using a single-board computer, you should check if you can also boot it from an external SSD. You will need that SSD for your Bitcoin applications and data anyway, and also using it for your operating system will offer much better performance. We know that you can boot a Raspberry Pi 4 from an external SSD, but it seems somewhat complicated.    

Note that running Bitcoin Core does not necessarily require so much diskspace as indicated above. If you are facing severe diskspace constraints and just want to route your own transactions onto the network, then just running a **pruned node** can be a viable option. This makes almost no demands on diskspace. As already indicated, however, not maintaining the full Block Chain will severely limit the flexibility of your server. And you cannot, in any case, complete this guide with a pruned node. 


## Backup drives

You should always make sure any bitcoin you have are properly backed up. For on-chain Bitcoin this typically requires keeping a series of 12 to 24 mnemonic words and, optionally, a password. Sometimes the backup can be more complicated, as in case of a multisignature address. For bitcoin placed in the lightning network, you will also need to make backups about channel states. We will show you the basics of backing up your bitcoin at a later stage in the guide.  

You may also believe it advantageous to have an automatic backup system for all your other Bitcoin server data, or at least some key parts of that data like your version of the Block Chain. In case you have a system failure, such a backup would help you get everything back up an running more quickly.

Without considering backups, you would probably opt for a single comprehensive drive or at most two drives in your Bitcoin server: one for the operating system and other applications and data, and one drive for your Bitcoin applications and data. If you want to have a backup system to an internal drive or implement a mirroring RAID configuration, you will need to account for the additional drives in your system requirements. 

We will not concern ourselves with implementing such a backup system in this guide. We will only concern ourselves with backing up bitcoin. So you will have to figure it out on your own. Just be aware that some data on your system will be sensitive, such as wallet profiles, Tor hidden service keys, and so on.   


## Energy efficiency

You will want to have your server on 24/7. This allows you to receive and send mainnet transactions, use your block explorer, and so on, on the fly. Constantly turning on/off your server is a hassle. Particularly if you plan to eventually run your own Lightning node, you will need to have your system available all the time to prevent forced channel closings and to support others in routing transactions.  

Given the 24/7 availability need, energy efficiency is an important requirement on your system. Due to its energy efficiency as well as low component costs and ample online support, many people choose to buy a Raspberry Pi. Given that a Raspberry Pi is only a computer in the strict sense of the word,<sup>[1](#footnote1)</sup> various components are then added to make it into a useable system (e.g., external drive, heatsink, fans, micro SD card). 

As an example of its energy efficiency, the Raspberry Pi 4 was tested to use only 7.6 watts of power under load.<sup>[2](#footnote2)</sup> On the basis of that estimate, a Raspberry Pi 4 should only use about 67 kilowatt hours of energy per year under load. This is a very low level of energy consumption. A standard office laptop will typically run between 30 to 100 Watts under load. A standard home PC will typically consume between 200 to 500 Watts under load.  

There are many other types of single-board computers, such as those from the Odroid and Rock series, and smaller PCs, such as ***NUCs***, that you can choose from for your Bitcoin server. Many of these weigh up quite well against the Raspberry Pi series in terms of energy efficiency and other considerations. If you are willing to spend more money, you can also find larger-sized, more robust computers that are still fairly energy efficient. These larger-sized systems are typically more powerful and more future-proof. In addition, they have the advantage that they cannot easily be knocked over. For creating this guide, we used, for instance, a Shuttle DS20U with 32GB RAM, a 500GB NVME SSD (for the operating system and general applications and data), and a 2TB SATA SSD for the Bitcoin-related applications and data.<sup>[3](#footnote3)</sup> 

If you have a laptop or old PC that you can use for this guide, then you might want to use it instead of purchasing a new system. However, it is likely somewhat energy inefficient. If you do become a serious noderunner, you will want to explore energy-efficient options at a later point.


## Other considerations

People manage to complete an **initial block chain download** (**IBD**) and to operate their Bitcoin Core nodes on systems with very minimal specifications. But unless you are facing serious financial constraints, we would recommend having a system with the following specifications (next to the 1TB minimal diskspace requirement):

* A processor with at least 1.5GHz, preferably with multiple cores
* At least 4GB RAM
* Sufficient cooling through design or equipment like heat sinks and fans
* Proper casing for environmental protection and robustness

In addition, you should ensure that your Internet service plan offers a reasonable amount of download and upload capacity each month. Typically, your server will perform better if it is connected to your router with an ethernet cable, instead of via a Wireless access point. 


## Weighing up the options

If you are experienced in setting up servers, the above considerations and tips should be enough for you to make some hardware choices.

If you are a novice and still struggling after reading these considerations and tips, you probably just want to use an old PC or laptop if you have it available. If you don't have anything available, we would probably recommend buying on old PC if you do not want to spend too much money and time tinkering right now. While laptops are typically more energy efficient, PCs tend to be much more powerful than laptops in the same price range. Try still to buy the PC with energy efficiency in mind. 

If you have money to burn, we recommend buying a modern, energy-efficient, and robust system like the Shuttle DS20U we used for creating this guide. You can typically have such units assembled with RAM and drives of your choice. If you want something smaller, you could also looking into Intel NUCs and other such smaller PCs. 

For novices that enjoy tinkering, you might try a single-board computer with various parts. You could also try this approach if you really want the utmost in terms of energy efficiency. We must admit, however, that, even though they are fun, we think these single-board systems are not really that great for Bitcoin servers. They are much less robust and less secure than a typical PC. Often times they also turn out to be more costly than an old PC or laptop.   


## Notes

<a name="footnote1">1</a>. In common-day language, we often use the term “computer” to refer to a computer case with all the necessary electronic components inside, such as the motherboard, processors, RAM, various drives, extension boards, power supply, and fans. Strictly speaking, however, a “computer” is merely a device capable of accepting inputs and generating outputs. That just requires processing power, RAM, and support for input and output peripherals. It’s similar to how we might use the term “aquarium” to refer to an empty aquarium as well as an aquarium stocked with a pump, heater, soil, plants, water, and fish. The latter is much more useful, at least if you like aquariums. A computer with all the relevant peripherals to serve a useful purpose including secondary storage, power supply, keyboard, mouse, screen, and so on, is more precisely a computer system.

<a name="footnote2">2</a>. See Tom’s Hardware, “Raspberry Pi 4 review: The gold standard for single-board computing”, June 2 (2020) (available at: https://www.tomshardware.com/reviews/raspberry-pi-4-b,6193.html).

<a name="footnote3">3</a>. See here: https://www.shuttle.eu/en/products/slim/ds20u. 
