# Chapter 2. Hardware selection

You can create a Bitcoin server on all types of computers, including desktops, laptops, single-board computers, phones, and dedicated hardware servers. Whether a computer is capable of handling your Bitcoin server depends on your choice of software applications and their configurations, as well as the hardware capabilities of your computer.

We will not prescribe any particular hardware system and leave you free to make your own choices. Instead, this chapter merely contains some general considerations and tips on selecting hardware that meets the requirements.


## Disk space

The key factor to consider in your hardware choices is storage space. The main consumer of storage space will be Bitcoin Core. We will set it up with an entire history of the Block Chain, which is currently just over 500 GB (November 2023). In addition, we will set it up with a transaction index. Our choice of Electrum server, Fulcrum, will also require a fair amount of diskspace. 

For convenience, we will attempt to set up the binaries, startup scripts, and associated data of Bitcoin services in a common base directory. We will call this the **Bitcoin services directory** throughout this guide, though not all data and applications are installed here (for reasons explained later). This Bitcoin services directory can be its own dedicated drive, or part of a more comprehensive drive that also supports your operating system and other applications. 

Given that the diskspace requirement for the Block Chain continuously grows—about 50GB to 100GB per year—and that we will also run additional services on your Bitcoin server, you should definitely ensure ample diskspace. We would recommend a system with a minimum of 1TB in diskspace for a dedicated Bitcoin drive, and would strongly recommend going higher if you plan to use this server for a few years. Anything over 2.5TB is probably overkill, unless you have really bold intentions for your Bitcoin server (such as running an ElectrumX server or Blockstream’s Esplora). 

These size indications will probably also work if you are using a comprehensive drive—that is one that also includes the operating system and other applications—to store your Bitcoin services directory. At least, as long as you do not do too much else with your Bitcoin server than what is covered in this guide.

Traditional hard drives (HDDs) are cheaper. But a system with only solid state drives (SSDs) has a number of advantages.   

1. SSDs make practically no noise, by contrast to HDDs. If you intend to keep your Bitcoin server in your living room, a home office, or a bedroom, then SSDs are a safer bet. 
2. SSDs generally have much higher read and write speeds than HDDs. Use an SSD for your Bitcoin services directory and your Block Chain data will definitely download much faster. 
3. SSDs are more power efficient than HDDs. This is particularly relevant when running a server 24/7.  

Ideally, you would use an SSD with a PCI express connection for your Bitcoin services directory. Such SSDs work with PCIe 3.0 or higher. They are known as **NVMe SSD drives**—that is, non-volatile memory express SSD drives—and have the **M.2 form factor**. An NVMe SSD will have by far the highest reading and writing speeds. 

That said, NVMe SSDs are very expensive and you may not have the option for a system that includes one. In that case, a standard **SATA SSD** of 1TB or 2TB for storing your Bitcoin services directory is a great option, and what most people typically would use. A SATA SSD will still give you very good performance, much better than a traditional HDD.

In case you use a dedicated drive for your Bitcoin services directory, then your will still need a drive for your operating system and other applications. An SSD is also the best option here, particularly for running your operating system. 

Many people run Bitcoin servers from **single-board computers** such as **Raspberry Pis**, **Odroids**, or **Rocks**. With these credit-card sized computers, the operating system is often installed on a (micro) SD card. That type of memory card is typically much less robust than a quality SSD or even a traditional HDD. If you are planning on using a single-board computer, you should check if you can also boot it from an external SSD. You will need that SSD for your Bitcoin services directory anyway (due to size limitations on SD cards), and also using it for your operating system will offer much better performance. We know that you can boot a Raspberry Pi 4 from an external SSD, but it seems somewhat complicated.    

Note that running a Bitcoin server does not necessarily require so much diskspace as indicated above. Notably, instead of maintaining the entire Block Chain, you could also choose just to run a **pruned node**. This makes almost no demands on diskspace. The space demands result from the fact that we want a Bitcoin server that is flexible and secure. 


## Backup drives

You should always make sure any bitcoin you have are properly backed up. For on-chain bitcoin this typically requires keeping a series of 12 to 24 mnemonic words and, optionally, a password on some offline medium. Sometimes the backup can be more complicated, as in case of a multisignature address. For bitcoin placed in the lightning network, you will also need to store information about channel states.   

You may also wish to have an backup system for your entire server, or at least some of the key data (e.g., Block Chain data). In case you have a system failure, such a backup system would help you get everything back up an running more quickly.

Without considering backups, you would probably opt for a single comprehensive drive or at most two drives in your Bitcoin server: one for the operating system and other applications, and one for your Bitcoin services directory. If you also want a backup system, then, depending on your preferences, you can consider adding more internal drives. To ensure availability of your server, you may considering working with mirroring **RAID configurations** (RAID 1).   

We will not concern ourselves with implementing a broader backup system in this guide. We will only concern ourselves with backing up bitcoin, the currency, when relevant. So you will have to figure general server backups out on your own. Just be aware that some data on your system will be sensitive, such as wallet profiles, Tor hidden service keys, and so on.   


## Energy efficiency

You will want to have your server running 24/7. Constantly turning on/off your server is a giant hassle. With regards to Lightning, turning off your server is additionally a security risk as you cannot monitor channel partners and have to rely fully on any watchtowers you have configured. 

Given the 24/7 availability need, energy efficiency is an important requirement of your system. Due to its energy efficiency as well as low component costs and ample online support, many people choose to buy a Raspberry Pi. Given that a Raspberry Pi is only a computer in the strict sense of the word,<sup>[1](#footnote1)</sup> various components are then added to make it into a useable system (e.g., external drive, heatsink, fans, micro SD card). 

As an example of its energy efficiency, the Raspberry Pi 4 was tested to use only 7.6 watts of power under load.<sup>[2](#footnote2)</sup> On the basis of that estimate, a Raspberry Pi 4 should only use about 67 kilowatt hours of energy per year under load. This is a very low level of energy consumption for a computer. A standard office laptop will typically run between 30 to 100 Watts under load. A standard home PC will typically consume between 200 to 500 Watts under load.  

There are many other types of single-board computers, such as those from the Odroid and Rock series, and smaller PCs, such as ***NUCs***, that you can choose for your Bitcoin server. Many of these weigh up quite well against the Raspberry Pi series in terms of energy efficiency and other considerations. If you are willing to spend more money, you can also find larger-sized, more robust computers that are still fairly energy efficient. These larger-sized systems are typically more convenient and future-proof. In addition, they have the advantage that they cannot easily be knocked over. For creating this guide, we used a Shuttle DS20U with 32GB RAM, a 500GB NVME SSD (for the operating system and general applications), and a 2TB SATA SSD for the Bitcoin services directory.<sup>[3](#footnote3)</sup> 

If you have an old laptop or PC that you can use for a Bitcoin server, then you might want to use it instead of purchasing a new system. However, it is likely somewhat energy inefficient. If you do become a serious noderunner, you will want to explore energy-efficient options at a later point.


## Other considerations

People manage to complete an **initial block chain download** (**IBD**) and to operate their Bitcoin Core nodes on systems with very minimal specifications. But unless you are facing serious financial constraints, we would recommend having a system with the following specifications (next to the 1TB minimal diskspace requirement):

* A processor with at least 1.5GHz speed, preferably with multiple cores
* At least 4GB RAM
* Sufficient cooling through system design or equipment like heat sinks and fans
* Proper casing for environmental protection and robustness

In addition, you should ensure that your Internet service plan offers a reasonable amount of download and upload capacity each month. Typically, your server will perform better if it is connected to your router with an ethernet cable, instead of via a Wireless access point. 


## Weighing up the options

If you are experienced in setting up servers, the above considerations and tips should be enough for you to make hardware choices.

If you are still struggling after reading these considerations and tips, you probably just want to use an old PC or laptop if you have it available. If you don't have anything available, we would recommend buying on old PC if you do not want to spend too much money and time tinkering right now. While laptops are typically more energy efficient, PCs are generally more powerful in the same price range. Try still to buy the PC with energy efficiency in mind. 

That advices assumes, however, that you do not have money to burn. If you do, we definitely recommend to instead buy a modern, energy-efficient, and robust system like the Shuttle DS20U we used for creating this guide. You can typically have such units assembled with RAM and drives of your choice. If you want something smaller, you could also look into Intel NUCs and other small PCs. 

If you enjoy tinkering, you might also try a single-board computer with various components. You could also take this approach if you really want the utmost in terms of energy efficiency. We must admit, however, that, even though they are fun, we think these single-board systems are not really that great for Bitcoin servers. They are much less robust and less secure than a typical PC. Often times they also turn out to be more costly than an old PC or laptop, at least initially.   


## Notes

<a name="footnote1">1</a>. In common-day language, we often use the term “computer” to refer to a computer case with all the necessary electronic components inside, such as the motherboard, processors, RAM, various drives, extension boards, power supply, and fans. Strictly speaking, however, a “computer” is merely a device capable of accepting inputs and generating outputs. That just requires processing power, RAM, and support for input and output peripherals. It’s similar to how we might use the term “aquarium” to refer to an empty aquarium as well as an aquarium stocked with a pump, heater, soil, plants, water, and fish. The latter is much more useful, at least if you like aquariums. A computer with all the relevant peripherals to serve a useful purpose including secondary storage, power supply, keyboard, mouse, screen, and so on, is more precisely a computer system.

<a name="footnote2">2</a>. See Tom’s Hardware, “Raspberry Pi 4 review: The gold standard for single-board computing”, June 2 (2020) (available at: https://www.tomshardware.com/reviews/raspberry-pi-4-b,6193.html).

<a name="footnote3">3</a>. See here: https://www.shuttle.eu/en/products/slim/ds20u. 