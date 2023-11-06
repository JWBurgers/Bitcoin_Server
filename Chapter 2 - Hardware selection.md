# Chapter 2. Hardware selection

You can create a Bitcoin server on various types of computers, including desktops, laptops, phones, and hardware servers. Whether a computer is capable of handling your Bitcoin server depends on your choice of software applications and their configurations, as well as the capabilities of your computer.

We will not prescribe any particular hardware system and leave you free to make your own choices. So this chapter merely contains some general considerations and tips on selecting hardware that meets the requirements.

In case you would like more specific guidelines, we have added instructions for setting up a Raspberry Pi system in Appendix A. 


## Disk space

We will set up Bitcoin Core with an entire history of the Block Chain, which is currently just over 500 GB (November 2023). In addition, we will set up Bitcoin Core with a transaction index. A full Block Chain with a transaction index is really a prerequisite for many applications that you might eventually run on a Bitcoin server, including an electrum server and a local block explorer. Having the entire Block Chain stored also provides additional benefits. 

Given that the diskspace requirement for the Block Chain continuously grows—about 50 to 100GB per year—and that we will also create additional services on your Bitcoin server (electrum server and local block explorer), you will definitely need to have ample diskspace for this exercise. We would recommend a system with a bare minimum of 1TB in diskspace, and would strongly recommend going higher if you plan to use this server for a few years. Anything over 2.5TB is probably overkill, unless you have really bold intentions for your Bitcoin server (such as running an ElectrumX server or Blockstream’s Esplora). 

Although traditional hard drives (HDD) are cheaper, using solid state drives (SSDs) for storing the Block Chain has a number of advantages.   

1. SSDs make practically no noise. HDDs do, though how much depends on the particular HDD. If you intend to keep your Bitcoin server in your living room, a home office, or a bedroom, then an SSD is a safer bet. 
2. SSDs generally have much higher read and write speeds than HDDs. Although many relatively new HDDs will probably work for Bitcoin Core, a good SSD has higher speeds than you will ever need. The higher speeds ensure that your node initially syncs with the Block Chain must faster. 
3. Although the subject of some controversy, generally SSDs seem more power efficient than HDDs. 

Running Bitcoin Core does not necessarily require so much diskspace. If you are facing severe diskspace constraints and just want to route your own transactions onto the network, then just running a pruned node can be a viable option. This makes almost no demands on diskspace. As stated above, however, this will severely limit the flexibility of your server. You cannot run a pruned node for this guide. 


## Energy efficiency

You will want to have your server on 24/7. This allows you to receive and send mainnet transactions on the fly, use your block explorer, and so on, without needing to turn on/off the server all the time. Particularly if you plan to eventually run your own Lightning node, you will need to have your system on all the time to prevent forced channel closings and to support others in routing transactions.  

Given the 24/7 availability need, energy efficiency is, therefore, an important requirement. Due to its energy efficiency as well as low component costs and ample online support, many people choose to buy a ***Raspberry Pi***—a low cost, credit-card sized computer. Given that a Raspberry Pi is only a computer in the strict sense of the word,<sup>[2](#footnote2)</sup> various components are then added to make it into a useable system (e.g., external hard drive, heatsink, fans, micro SD card). 

As an example of its energy efficiency, the Raspberry Pi 4 was tested to use only 7.6 watts of power under load.<sup>[1](#footnote1)</sup> On the basis of that estimate, a Raspberry Pi 4 should only use about 67 kilowatt hours of energy per year under load. This is a very low level of energy consumption. A typical laptop, for instance, is probably consuming around five times that amount of energy. Typical desktops certainly even more. 

There are many other types of single-board computers, such as those from the ***Odroid*** and ***Rock series***, and smaller PCs, such as ***NUCs***, that you can choose from. Many of these weigh up quite well against the Raspberry Pi series in terms of energy efficiency and other considerations. If you are willing to spend more money, you can also find larger-sized, more robust computers that are still fairly energy efficient. They typically are more future-proof in their specifications than these smaller systems and cannot easily be knocked over.<sup>[3](#footnote3)</sup> 

If you have a laptop or old PC that you can use for this guide, then you might not want to use it instead of purchasing a new system. However, it is likely very energy inefficient. If you do become a serious noderunner, you will want to consider switching at a later point.


## Other considerations

People manage to complete an initial block chain download (IBD) and to operate their Bitcoin Core nodes on systems with very minimal specifications. But unless you are facing serious financial constraints, we would recommend having a system with the following specifications (next to the 1TB minimal diskspace requirement):

* A processor with at least 1.5 GHz, preferably with multiple cores
* At least 4 GB RAM
* Sufficient cooling equipment like heat sinks and fans
* Proper casing for environmental protection and robustness

In addition, you should ensure that your Internet service plan offers a reasonable amount of download and upload capacity each month. Typically, your server will perform better if it is connected to your router with an ethernet cable, instead of via a Wireless access point. 


## Notes

<a name="footnote1">1</a>. In common-day language, we often use the term “computer” to refer to a computer case with all the necessary electronic components inside, such as the motherboard, processors, RAM, various drives, extension boards, power supply, and fans. Strictly speaking, however, a “computer” is merely a device capable of accepting inputs and generating outputs. That just requires processing power, RAM, and support for input and output peripherals. It’s similar to how we might use the term “aquarium” to refer to an empty aquarium as well as an aquarium stocked with a pump, heater, soil, plants, water, and fish. The latter is much more useful, at least if you like aquariums. A computer with all the relevant peripherals to serve a useful purpose including secondary storage, power supply, keyboard, mouse, screen, and so on, is more precisely a computer system.

<a name="footnote2">2</a>. See Tom’s Hardware, “Raspberry Pi 4 review: The gold standard for single-board computing”, June 2 (2020) (available at: https://www.tomshardware.com/reviews/raspberry-pi-4-b,6193.html).

<a name="footnote3">3</a>. For creating this guide, we used a Shuttle DS20U with 32GB RAM, a 500GB SSD (for the operating system), and a 2TB SSD for the Bitcoin Server applications and data. See here: https://www.shuttle.eu/en/products/slim/ds20u. 
