# The Linux command line

If you have followed the instructions in *Chapters 3* through *5* and are new to the command line environment, you probably felt a little bit like a hacker. But what we did is hardly rocket science. Now that you have some experience, we will remove the mystery by giving some more background in this chapter. If you are already quite familiar with command line environments, you can just continue with the instructions in *Chapter 7*.

While we could have started our guide with an introduction to the command line environment, it is probably more effective to learn the background after some practical experience. Hence, why we placed this background chapter in the middle of the guide. 


## Command line interfaces

A **command line interface** is any interface that allows a user to interact with one or more programs through text-based commands. Whereas we generally interact with our computers and programs nowadays through graphical user interfaces, command line interfaces still serve an important role in computing. Particularly if you want to do things like set up your own server, you will need to use a command line interface. You can see an example of a command line interface in *Figure 1*. This is the default terminal application for a Raspberry Pi computer.

*Figure 1: Raspberry Pi terminal window*

![Figure 1: Raspberry Pi terminal window](/Images/Figure2-1.png "Raspberry Pi terminal window")


A command line interface is sometimes integrated into a particular application. If you launch the graphical application within the Bitcoin Core software package known as Bitcoin-Qt, for example, there is a small command line interface called the **Bitcoin Console**. Certain types of operations not available on the rest of the graphical interface are available through the Bitcoin Console via text-based commands. You can see a picture of the Bitcoin Console in *Figure 2*. 

*Figure 2: The Bitcoin console*

![Figure 2: The Bitcoin console](/Images/Figure2-2.jpg "The Bitcoin console")


In contrast to the Bitcoin Console, command line interfaces are typically used to interact with an entire computer, rather than just one specific application. With Linux systems, you can generally use command line interfaces in two ways.  

First, you can specify the system to boot to the **main command line interface**, instead of the desktop environment (which is really a graphical user interface into your entire computer). In fact, you can install a **headless version** of most Linux distributions, meaning that they only come with the main command line interface and no desktop environment. A headless version can be particularly desirable for increasing performance on a server. 

Second, you can access your Linux system via a command line interface from within the desktop environment by using a **terminal application** or a **terminal emulator**. In *Figure 2* above, you can see the default terminal application for Raspberry Pis called **LXTerminal**. There are, however, many different terminal programs available for Linux distributions, such as **Gnome terminal**, **Konsole**, and **Terminator**. By default, Debian utilizes the Gnome terminal application.  

The command line interface from a terminal application is sometimes specifically called a **terminal window** to indicate its presence within a graphical environment. 

The name terminal application is a reference to the physical terminals that were once popular in computing. 

During World War II, the efforts to break German encryption systems centered on building early computers at Bletchley Park. The BOMBE (probably named after a polish ice cream called a “bomba”) was entirely mechanical and used to break the encryption from German enigma machines. The **Colossus** was instead an electro-mechanical computer that supported breaking the encryption from more sophisticated Lorentz machines. It was, unlike the BOMBE, also programmable and, thus, adaptable to different problems. It was, therefore, really the world’s first computer in the modern sense of the word. 

In the early post-war period, the focus remained on conducting scientific research into building programmable computers. This led to the first electrical computers such as Manchester Baby and ENIAC. 

By the 1960s, however, computers were not just an object of research, they were an actual tool used by scientists and defense agencies. These early computers were very large and expensive machines. Usually they were locked safely in a room somewhere at a university or office building. 

Physical terminals allowed users to connect to these computers. Sometimes that would be through a cabled connection when the end users were nearby. More commonly, however, user terminals connected to these large computers remotely via phone lines. As computers were often far away, having to be near a computer for connecting was often impractical. 

The terminal’s job was to accept commands from the user, to send these to the computer for processing, and then to display the outputs that the computer returned. Originally, terminals relied on punch cards for accepting user inputs, while the returned outputs by the computer were printed (such devices were known as **teletypes**, **teleprinters**, or **teletypewriters**). Over time, however, terminals primarily became text-screen displays with keyboards. 

Terminal applications are, thus, so called because the command-line interfaces they provide such as in *Figure 1* resemble those of traditional physical terminals. You can see an example of an old-style physical terminal in *Figure 3*.

*Figure 3: A DEC VT100 terminal*

![Figure 3: A DEC VT100 terminal](/Images/Figure2-3.png "A DEC VT100 terminal")


Importantly, a terminal application only provides a command line interface to your computer. Just as the traditional physical terminals, these types of programs do not process any inputs directly. They only accept inputs from the user, send those inputs to particular programs, and then retrieve and display the outputs textually. The same is true for the main command line interface on a Linux system.   

You are likely to come across a great variety of terms that people use to refer to command line interfaces or specific types of command line interfaces, including **virtual terminal**, **command line terminal**, **console**, and **virtual console**. Sometimes people might also refer to a (specific type of) command line interface simply as a terminal. 

We will not run through all the common ways in which these terms are used in practice, nor on the merits of particular usages. Simply be aware that there is a lot of liberal language usage when it comes to command line interfaces.  

In the rest of the text, we will only use the terms command line interface, main command line interface, terminal application, and terminal window, with their meanings as described in this section. 


## Command line interpreters

If you launch a terminal application in a Linux system or boot into the main command line interface, the interface does not process commands directly. So in order for such a command line interface to be useful, you need to load a program into it that can interpret your commands for processing. 

The standard program that you would work with is called a **command line interpreter** or a **shell**, and such a program is automatically loaded when you launch a terminal application on Linux or boot into the main command line interface. A command line interpreter allows you to interact with your entire computer and all its programs. 

The term "interpreter" refers to the particular way in which a command line interpreter converts your commands, written in a high-level language, into **machine code**: that is, the code that can be directly understood by your central processing unit (machine code is also known as **binary code**). An **interpreter** takes source code and directly translates it statement by statement into machine code to be executed by the central processing unit. 

Suppose that your source code was located in a file. Each time you, then, execute this source code, the interpreter will translate it into machine code anew. It doesn’t store the machine code somewhere on your machine for reusage. 

An interpreter is commonly contrasted with a compiler. A **compiler** takes the source code of a particular program and converts it in its entirety to machine code. That machine code is, then, all **linked** by the compiler in an executable file. Once the executable file has been created, you can ask your computer to run that executable file each time you need the program. Programming languages like **C** and **C++** standardly work with compilers (though in principle they could also work with an interpreter). 

In the context of a command line interpreter, the application is just taking each command you execute, and directly translates it into machine code. It’s not storing the machine code of any of your commands to draw on for the next time. 

Your Linux operating system will load a default command line interpreter when you launch a terminal application or boot into the main command line interface. There are a wide variety of interpreters for Linux systems. A popular choice that comes with many distributions is called **GNU Bash**. Bash here stands for “Bourne again shell.” It is also the standard command line interpeter for Debina.  

The Bourne shell was originally developed by Stephen Bourne at Bell Labs to be used with Unix operating systems. GNU Bash was developed by Brian Fox for the GNU project as a free alternative to the Bourne shell. There are a number of alternative command line interpreters you could also use with a Linux system, such as **C Shell**, **Z Shell**, and **Korn Shell**. 

These command line interpreters are all very similar. They each work with terse and basic syntax that is well-suited to being entered in a terminal window or the main command line interface. 
  
The program running in your Linux terminal window or the main command line interface does not have to be the default command line interpreter. For example, you could use the command line interpreter to launch the **Python Shell** (also known as the **Python Interactive Shell**). 

Despite the name, this is not really a shell in the sense that we have been discussing. Instead it is a small program that allows you to act interactively with the **Python Interpreter**, a program that can interpret the Python programming language. 

You may want to open the Python Shell, for example, to quickly check small snippets of code. Until you proactively close the Python interpreter, it will be the default application running in your Linux terminal window or main command line interface, not your command line interpreter. 

As in the previous section, you need to be aware of the conceptual liberty that surrounds terms like command line interpreter and shell. I will not run through the myriads of ways these terms are used, nor their merits. 

Do again note, however, that I am employing the terms command line interpreter and shell only to refer to the interpreters of small, terse languages that are specifically intended for controlling a computer via a command line interface. The Python Interactive Shell is, thus, for instance, not really a shell or command line interpreter in this sense.


## The structure of Bash commands

In this section, we will give you some more information about the structure of commands. As the Bash command line interpreter is most popular for Linux systems and used by our Debian operating system, we will tailor the discussion to that application. But in general, all we say here is probably applicable to any command line interpreter for Linux. (It may also apply to command line interpreters on other operating systems, but we are less familiar with those applications.)  

When you enter a command line interface with GNU Bash, you will see a line of text that precedes all commands. It standardly shows the user account, the name of the host system, the working directory location, and a dollar sign. The dollar sign indicates that Bash is ready for commands. This sign is generally known as the **command prompt**.

The line of text above precedes all commands. It is produced by a configuration file. You can edit this configuration file if you wish. It should be visible from your home directory if you execute **`$ ls -a`**. The configuration file is called ".bashrc", where the dot indicates the file is hidden (hence, why you need to show all files in the directory to see it). 

You are always interacting with Bash from within a particular directory. This is no different than within a desktop environment. On startup of Bash, this is by default the home directory. To understand where you are located in your system with Bash, two symbols are important to understand:  

* The “/” sign, or forward slash, symbolizes the **root directory**, if it is at the very start of the string of text after the colon. This root directory is the top of the directory tree, and you cannot move any higher. So if you see, for example, “[user]@[host name]: / $” in Bash, then you are currently working from the root directory. 
* The “\~” sign, or tilde, symbolizes the **home directory** for the current user of the system. The home directory is by default in the following directory location: /home. So if your username is Satoshi_Nakamoto, your user directory would be in the following location: /home/Satoshi_Nakamato. This is equivalent to just seeing the symbol “~”. 

Commands for GNU Bash always have the following sequential structure:

1. A command name (always required)
2. Options
3. Arguments

The **command name** always comes first, then the **options** you want to specify (also called **flags** or **switches**), and finally any **arguments** you want to give with the command. 

Command names in Bash are always lowercase abbreviations or words. Usually the abbreviations are reasonably intuitive. For instance, the **`$ mkdir`** command name is used for making a new directory, while the **`$ ls`** command lists all non-hidden files and directories within the current working directory. 

All commands have default options for their execution, which can be changed by adding different execution options. For instance, as said above, just giving the command **`$ ls`** lists all non-hidden files and directories within the current directory. But you can change the options while executing this command to generate a different result. As already mentioned, if you wanted to also list all the hidden files and directories in the current directory, you can execute the following command **`$ ls -a`**. The “a” here stands for “all.” 

You can add more than one option to a command. If you also wanted to display basic data about all hidden and non-hidden files and directories, you could execute the following command **`$ ls -a -l`**, where the “l” stands for “long.” The order of the options does not matter. You could have also given the command **`$ ls -l -a`**. 

In fact, you can even combine options in your notation, like in this command **`$ ls -al`**. Some people prefer to keep the options separate for legibility, which is the practice we will adopt in this guide. Combining them, however, is more efficient. Importantly, there are many different commands for the terminal (you can even create them yourself), and the options do not always have the same meaning across different commands. 

Options are always specified with a single or double hyphen sign (- or --), followed by the option indication. Single hyphens are for options with one letter, double hyphens for options that are words. 

Finally, you may enter arguments in your commands. Some commands do not require any arguments in order for Bash to understand them. For instance, the command **`cal`** just outputs the calendar. You can add arguments to the command name, but they are not required. 

For some commands, however, your will need to always provide an argument. For instance, you cannot use the **`$ mkdir`** command to make a new directory, without specifying the name of the directory you wish to make. Hence, you would need something like the following: **`$ mkdir Invoices`**. In this case, Bash will create a new directory called “Invoices.”

It is also possible to execute multiple commands on the terminal at the same time. You can separate commands in multiple ways, such as using semicolons in between them. Suppose, for instance, that you wanted to create a new directory called “NewDirectory” with three new files, and then list the contents. One way to do this is with the following series of commands: **`$ mkdir NewDirectory; touch file1 file2 file3; ls NewDirectory`**.

A key point to understand in using Bash or any other command line interpreter successfully, is that there are two types of user modes: **root user mode** and **normal user mode**. The root user has the ability to perform a wide variety of operations on the system, while normal users are much more restricted in what they can do. 

For instance, while a normal user can create new files and directories in their home folder, she cannot usually install and remove system programs or change system settings. That can typically only be done via root user mode. 


## Four types of commands

There are four types of commands within Bash. 

First, you can provide a type of command known as a **shell built-in**. Bash understands these commands directly. They include commands such as **`$ cd`** ("change directory"), **`$ mkdir`**, and **`$ pwd`** ("print working directory"). If you search your computer for a file called “cd” or “pwd” you will not find it, as these commands are built into the shell program directly. 

Second, you can provide a command to launch an executable program. These programs can be written in C, C++, Perl, Python, or any other type of higher-level language. Executable programs can include anything from text editors, to computer games, to smaller executable programs such as **`$ mv`** ("move"), which is used to move files, and **`$ ls`**. 

Most smaller executable programs typically only execute once and then close automatically. Sometimes programs may remain open until you instruct the shell to close them. This is the case, for instance, with the Nano text editor and the Python interpreter.   

Your current working directory is irrelevant to launching an executable program successfully, if the location of the program is specified in the **PATH variable**, or **$PATH**. By default, there are a number of different locations stored in the PATH variable that the shell will look to find the executable program. If you have installed the program in an unusual location, you will need to include that location as an option within the PATH variable in order for the shell to find it. 

If an executable program is not included in your PATH variable, you can still execute it, but you will have to specify its location in the execution instructions. Suppose, for example, that you had a bitcoind application in the following directory: /bitcoin-server/bitcoin-core/bin. Just executing **`$ bitcoind`** will not do anything. However, you could execute the program if you indicated the path. Following are two examples:

* From the root directory, you would have to execute **`$ /bitcoin-server/bitcoin-core/bin/bitcoind`**.
* From the program's directory, you would have to execute **`$ ./bitcoind`**, where the "." indicates the current directory. 

Third, you can ask the interpreter to execute what is known as a **shell script** or a **shell function**. A shell script is really just a collection of commands stored within a file. Within a Bash environment, would typically call it a **Bash script**. Often Bash scripts have the extension ".sh" or ".bash".  

If you are faced with repetitive tasks that require the same set of commands, then it makes much more sense to store them in a shell script than to enter them into the command line interface each time. Importantly, a shell script does not have to run the exact same code each time. There are ways you can change the values of certain variables in their execution. 

The final type of command which you can give to the shell is known as an **alias**. An alias is nothing more than a shortcut to another command or series of commands, specified by the user. 

For instance, suppose that you wanted to display the branches of your Git repository in a graphical form. Rather than typing in the command **`$ git log --al --decorate --oneline --graph`** each time, you could just create an alias called "graph”. 


## Some basic commands

Bash is not too difficult to learn (though in my personal experience, easy to forget). You already have some experience with Bash from the previous chapters. You now also have some detailed background from the previous sections in this chapter. The easiest now would be to find a good course or tutorial online. There are many. Any good course should not be more than a few hours; any more is probably overkill, unless you are venturing into very advanced topics. After such a brief course, you should be able to master the command line reasonably well. 

If you prefer to practice on your own, then you can have a look at the list of commands given below. In case you ever accidentally fill in a command that is taking a really long time to execute, or if you want to return to the command line from an ongoing process, you can kill the process by pressing the “Control button” and the “C button” at the same time on your keyboard.  

**`$ echo $0`** This command will tell you what shell program you are using; it should be GNU Bash. 

**`$ cal`** This command gives you a calendar; executing **`$ cal 2000`** will give the calendar for the year 2000, and similarly for other years.

**`$ date`** Gives the current date.

**`$ watch`** This will repeatedly execute a command. If you, for instance, use the command **`$ watch date`**, you will see the date constantly being updated in your terminal. You can exit by pressing the Control button and the C button at the same time.

**`$ pwd`** Prints the absolute path of the current working directory.

**`$ cd`** Changes the directory either by specifying an **absolute path** (i.e., relative to your root directory) or a **relative path** (i.e., relative to your current directory). 

The way to use the **`$ cd`** command and the meanings of absolute and relative path specifications are best illustrated by some examples:  

- **`$cd ~`**: Moves your location in the system to your user directory; this is an absolute path specification.
- **`$ cd /`**: Moves your location in the system to your root directory; this is an absolute path specification.
- **`$ cd business`**: If your current working directory indeed has a directory called business, then your system will now move into business (otherwise, it will give you an error); this is a relative path specification.
- **`$ cd business/Invoices`**: If your current working directory indeed has a directory called business, and the business directory has a directory called Invoices, then your system will now move into the Invoices directory (otherwise, it will give you an error); this is a relative path specification.
- **`$ cd /opt/Google/chrome`**: This brings you to the chrome directory and is an absolute path specification. 
- **`$ cd ..`**: This is a shorthand that brings you one directory level up; it is a relative path specification as it is relative to the current working directory. 
- **`$ cd -`**: Takes you to the last directory you were in, before the current working directory; it is an absolute path specification, because it is specified independently of the current working directory. 

**`$ ls`** Lists all non-hidden files and folders in the current directory. 

You can also specify listing another directory using absolute or relative path specifications. The following are examples: 
- **`$ ls business`**: Lists all the non-hidden files and folders in the business directory, if that directory is contained within the current directory.
- **`$ ls /home/Don_Johnson/business`**: Lists all the non-hidden files and folders in the business directory, if that directory exists in the absolute path specified.

**`$ mkdir`** Make a new directory in the current directory (e.g., **`$ mkdir Logos`**), or elsewhere in the system by specifying the right path (e.g., **`$ mkdir ~/Logos`** or **`$ mkdir marketing/Logos`**. 

**`$ touch`** Entering touch with a name creates a new file if it did not already exist. If the file name already existed, then the command will update the timestamp on the file indicating when it was last accessed. 

**`$ rm`** Removes a file or directory (e.g., **`$ rm Logos`**). In case of a directory, it is only removed if it is empty. 

In order to remove a directory with directories and/or files in it, you can add the options “R” and “i,” which are recursive and interactive respectively. For the Logos directory, it would be the following command: **`$ rm -R -i Logos`**.

The recursive option goes down into the directory to make everything removable. The interactive option checks to make sure if you really want to delete each file or subdirectory (you can answer either "Y" or "N" to each request for deletion). You can also leave the interactive option out, in which case your system will just delete everything. 

You want to be careful with this remove command. The command **`$ rm -R -f /`**> will forcibly delete all the files and directories on your computer without prompting any approvals. 

**`$ cp`** This command can do several types of operations, best illustrated via examples.

- The command **`$ cp file1 file2`** will copy the contents of file1 to file2. If file2 does not yet exist, it will be created. If file2 already exists, its current contents will be overwritten. The original file1 will also remain. 
- The command **`$ cp file1 file2 directory1`** will copy file1 and file2 into directory1. If directory1 does not yet exist, it will be created. If directory1 already exists, the contents of any files in the directory already named either file1 or file2 will be overwritten. The original file1 and file2 will also remain.  
- The command **`$ cp -R file1 file2 directory1 directory2`** will also move directory1 into directory2 in the same type of way as for files in the previous example. Note that you need to specify the recursive option to copy directories. 

**`$ mv`** This command can do several types of operations, best illustrated via examples.

- The command **`$ mv file1 file2`** will rename file1 to file2. 
- The command **`$ mv directory1 directory2`** will rename directory1 to directory2. 
- The command can also move files and/or directories to a directory by the following types of commands: **`$ file1 file2 directory1 directory2`**. In order for this operation to work, directory2 already has to already exist. 

**$` type`** Followed by a command name, this will tell you the type of command. For instance, **`$ type cd`** will tell you that this command is a shell built-in. For executable programs and shell scripts, this command will just tell you the absolute path where it is located. To distinguish between executable programs and shell scripts, you can use the command below ("file"). 

**`$ file`** Gives information about a file or directory when used with a file or directory name. Note that in Linux, a directory is also a type of file, hence why you can use this command on a directory name. 

**`$ history`** Gives the entire history of commands in a Bash session.

**`$ clear`** Clears the entire screen of previous commands.

**`$ help`** Followed by the name of a shell built-in will provide you with information on the command. 

**`$ man`** Followed by the name of a standard executable program or shell script will provide you further information about it. 


## Managing Linux applications

While the graphical interfaces on Linux distributions are getting much better, it is still fairly common to manage your applications via a command line interface. If you are a Windows or Mac user, then managing applications this way will probably seem somewhat confusing at first. So I will briefly provide an overview of how Linux applications are managed via a command line interface here. 

The primary way in which most linux users obtain applications is through **Linux packages**. Each package typically corresponds to a single user application, though sometimes it may correspond to multiple user applications. Linux packages come in two main formats with some differences in their download and installation process: **traditional Linux package formats** and **distribution independent package formats**. Typically, anytime you read about “linux packages”, the former type of package is meant, unless otherwise specified. Let’s discuss traditional linux packages first. 

Any traditional linux package includes, barring some exceptions, three main items. First, it includes the **binaries** (or **executable files**) which pertain to the application(s). These binaries provide the main functionality of the application(s). Note that even a single application may have multiple associated binaries. Second, a package includes a set of installation instructions for your Linux system such as in what directories to install the binaries, where to place configuration files, and so on. Finally, a package includes information on which other packages are required on your system for the application to work. These additional packages needed to successfully run your application are called **dependencies**. 

There are two main standards for traditional Linux packages: those in the **.deb file format** and those in the **.rpm file format**. The former are intended for Debian and its derrivatives. The latter is another popular standard used, among others, by **Red Hat Enteprise Linux** and **Fedora**. 

Typically, if a linux application is offered through the traditional package format, it will be available in both the .deb and .rpm varieties. If not, programs exist that can convert between the two standards. 

The main way traditional linux packages are distributed is through **Linux repositories**. These are servers which store many different packages that you can install on your system. Though not typical, traditional linux packages can in principle also be distributed via websites and other media.  

Applications that manage these traditional linux packages on your system are known as **package managers**. These management tasks include primarily the downloading, installation, removal, and upgrading of the packages on your system. 

There are various package managers that you can use for .deb and .rpm prackages. In order to have a sense of what these can do, lets have a look a the **Advanced Packaging Tool (APT)**, which is probably the most common package manager for .deb packages. Following are the main capabilities of the APT:

* It can search the packages available from linux repositories. By default, APT searches packages located within the one or more default repositories for your distribution. However, you can also add and remove repositories you want APT to track. 
* APT can install the applications in these packages on your system. During this process, it will automatically inform you of any dependencies and give you the option to install them as well.
* APT records package installation details. In particular, it tracks the locations of the binary files installed for each package. This means that it knows exactly which file on your system belongs to which package. This is in stark contrast to Windows. A Windows system has no idea which files belong to which applications, and installers manage the installation and removal of them.   
* APT can search for updates of any linux packages that you are managing with the tool, and allow you to upgrade them all simultaneously. (Note, in Linux terminology, we generally call the installation of updates for a packages as **upgrading** the package.)
* APT allows you to remove packages, and provides various options during the removal process. 

There are two main ways to provide commands to the APT application via a command line interface: via apt commands and apt-get/apt-cache commands. 

These two sets of commands are very similar. However, the latter is generally meant for more advanced use, while the former is generally for ordinary users. In fact, the apt commands are just simplified apt-get/apt-cache commands. So in most situations, you would invoke the APT application via apt commands.<sup>[1](#footnote1)</sup> As apt commands were only created in 2014, however, you can also frequently come across instructions that use apt-get/apt-cache commands.  

Below are the main commands that you would have to know if using the APT. When executing these commands, you should remember that you typically would have to be in root user mode in order to execute most of them. You can do this by either switching to the root user profile, or by preceding any commands with “sudo” if in the profile of a regular user. 

**`$ apt search`** Searches the packages in the linux repositories you have registered with the package manager for any search terms you provide. For example, the command **`$ apt search chess game`** will search and retrieve any packages that are consistent with the search terms “chess” and “game”. 

**`$ apt install`** Installs one or more packages available from your registered repositories. For instance, **`$ apt install chromium-browser`** will install the package for the chromium browser on your computer. Beforehand, it will tell you which dependencies are needed and give you the option to install them. 	

**`$ apt remove`** This command will remove the binaries associated with one or more packages. For instance, **`$ apt remove chromium-browser`** will remove the chromium browser binaries from your computer. 

**`$ apt purge`** This removes not just the binaries of one or more packages, but also most of the associated configuration and data files. It does not remove any dependencies you originally installed with the package.  

**`$ apt update`** This command does not take any arguments. The APT just downloads the basic information for all the packages it manages on your system from repositories. It, then, compares this new overview of your packages to its previous overview, and checks for updates. It will, then, output output a list of all the packages on your system which have updates available. Note, this command does not actually upgrade your packages, it only checks for the availability of updates. 

**`$ apt upgrade`** Without any arguments, this command will upgrade all the packages the APT manages on your system which have updates available. You can also use this command to upgrade only one or a limited set of specific packages. 

While the traditional linux packages are still the most popular, we have seen a significant rise in the use of distribution independent packages in recent years. The main formats in this regard are **AppImage**, **Snap**, and **Flatpack**. 

As the name implies, these package formats can be understood by almost any Linux distribution. In contrast to the traditional linux packages, distribution independent packages contain all the relevant dependencies within the package (except for very basic dependencies). Instead of distributing binaries across your system, the binaries for these packages are all included in one directory.  

As with traditional linux packages, Flatpacks and Snaps are also distributed through online repositories, and you can manage the downloading and installation of them through package managers. Many managers for traditional packages, however, will not work. For example, the APT does not offer support for these newer package formats. 

For handling the Flatpack and Snap formats, there are, among other options, package managers by the same name. Their commands display some similarities to the commands for the APT. We will not review them here. You can find plenty of resources online that discuss different commands for the Flatpack and Snap package managers.

AppImages work a little differently. Unlike with Flatpacks, Snaps, or traditional linux packages, the application and all its dependencies are stored in a single file. Hence, there is no real installation process for AppImages.  

Instead, you can just download an AppImage from a website, allow it to run, and you are ready to use it. You can remove the file from your system just by deleting it. As with other types of packages, however, there are tools available for downloading and tracking your AppImages.   

Newer package formats such as Flatpack, Snap, and AppImage have, despite some differences between them, several common advantages. For instance, with these newer formats developers only need to create one package, which can be used across all major Linux distributions. Another potential benefit lies in that fact that the application does not have to rely on system-wide versions of particular dependencies. This can be particularly beneficial for applications that are really sensitive to updates for their dependencies. 

There are also a number of disadvantages these distribution independent packages have compared to traditional linux packages. They are larger because they include all the necessary dependencies. This means that you might also have the same dependency installed multiple times for different packages. Another potential downside is that the developers of the package have to maintain the updates for all its dependencies. For traditional packages, this is done by the Linux developers. 

Finally, instead of using packages, you can also build the source code of applications directly. This, however, is a much more involved process. We will build several applications from source code in this guide, including Bitcoin Core. 


## Notes

<a name="footnote1">1</a>. The APT is really just a useful tool that interacts with the Debian Package Manager. It is also possible to communicate with the Debian Package Manager directly, rather than through the APT. You can interact directly with the package manager using “dpkg” commands. 