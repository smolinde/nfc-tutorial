# NFC Tutorial
<i>A full practical guide for classic NFC tags</i>

## 1 Introduction

NFC (Near-Field Communication) is a variety of communication protocols between two electronic devices over a short distance, usually less than 4cm.<sup>[[1]](https://en.wikipedia.org/wiki/Near-field_communication)</sup> We usually know NFC as a helpful technology when it comes to access control or contactless payment like [Google Pay](https://pay.google.com/about/) or [Apple Pay](https://www.apple.com/apple-pay/). There are a lot of different usecases where NFC is applied, and there is also a large variety of protocol standards. A very famous and wide-spread standard is the [MIFARE Classic](https://www.mifare.net/en/products/chip-card-ics/mifare-classic/). An advantage of MIFARE cards is that they are incredibly cheap and can be read by all possible devices. A drawback is that this protocol is very well examined and is no longer secure. In case you want to use NFC tags for storing confidential information, it is a very bad choice to use MIFARE tags. There are better and more secure tags available, such as [NTAG215](https://www.nxp.com/products/rfid-nfc/nfc-hf/ntag-for-tags-and-labels/ntag-213-215-216-nfc-forum-type-2-tag-compliant-ic-with-144-504-888-bytes-user-memory:NTAG213_215_216) cards. Although newer and better standards already exist, MIFARE tags are still used by a large number of companies and facilities.<br><br>
In this tutorial I will explain how to work practically with MIFARE cards. We will learn how to set up a DIY card reader and writer device, what MIFARE cards to use, how to manipulate contents of a card, and how to create fancy contact cards that can be read by all kinds of smartphones, including [Apple's iPhones](https://www.apple.com/iphone/).

## 2 DIY NFC Reader/Writer
### 2.1 Required Hardware and Materials
Let us start with some important physical requirements. Here is a list of hardware that we will use in this tutorial:
 * A Raspberry Pi Zero 2W with case and GPIO pins soldered
 * A case for your Raspberry Pi
 * A smartphone of your choice with mobile internet access
 * A computer/laptop of your choice
 * A 32GB nano SD card with suitable adapter for your computer
 * A power adapter with micro-USB (male)
 * A PN532 reader/writer module with pins soldered
 * Six female-to-female jumper wires (>10cm recommended)
 * A paper clip
 * A USB-A (female) to micro-USB (male) adapter
 * A smartphone wire that has a USB-A (male) connector
 * A wireless home network (2.4GHz)
 * New MIFARE cards of your choice
 * At least one new MIFARE chinese clone card with changeable UID
 * A soldering kit

### 2.2 Hardware Setup 
With everything at hand, let's start with the Raspberry Pi. Such set costs about 40 EUR (May 2024). I recommend [this seller on Amazon](https://www.amazon.de/stores/page/DDE68FC9-D2C7-464C-86A6-40A35B195E1B). The PN532 module comes usually with pins, MIFARE cards, and jumper wires. You can order it from [AliExpress](https://www.aliexpress.com/w/wholesale-pn532.html?spm=a2g0o.detail.search.0). There is a wide range of offers, find one that you like. One important thing to mention is that most times the seller includes only 4 jumper wires. If you use the [I2C protocol](https://en.wikipedia.org/wiki/I%C2%B2C) it works with only 4. However, using 6 together with the [SPI protocol](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface) singificantly increases operating speed which will be extremely important when running key recovery later. Therefore I recommend to find a seller that will send you 6 or buy jumper wires on [AliExpress](https://www.aliexpress.com/w/wholesale-jumper-wire.html?spm=a2g0o.productlist.search.0). If you have no soldering kit at home, consider buying [this one](https://aliexpress.com/item/1005006842091809.html).<br>

Now that we have everything, let us begin with the hardware preparation. Solder the GPIO pins to the Raspberry Pi, install the case, and solder the pins on the PN532 chip. Now grab your jumper wires and connect the PN532 module to the Raspberry Pi as shown below:<br>
<a name="wiring"><p align="center"><img src="./images/pn532_wiring.png" height="500px"></img></p></a>

Note that you have to change the position of the onboard switches to set the PN532 module to SPI. The black arrow in the image points on the switch block. The light yellow color in the diagram switch block is the switch position. First, remove the foil that is usually above the switches on new modules, then, with help of a paper clip, change the position of the switch. <b>Be careful, don't apply too much force!</b> Now check if you unplugged your soldering iron and continue with the software part.

### 2.3 Operating System Setup

Now to install the right operating system (OS) we have to download the [Raspberry Pi Imager](https://www.raspberrypi.com/software/). Open it on your PC and plug in your SD card. Now, select the device, in our case Raspberry Pi Zero 2W. For operating system, go to "Raspberry Pi OS (other)" > "Raspberry Pi OS Lite (32bit)". In storage part, select your SD card that you have plugged in before. Hit "NEXT". Now choose "EDIT SETTINGS". Here you can select a name for your Raspberry, your username and password, and set up your WiFi home network. Make sure you fill in the correct SSID and PSK (WiFi name and password), else you will have to do everything all over again. It should look something like this:<br>
<a name="configurationOS"><p align="center"><img src="./images/configuration_raspbian.png" height="500px"></img></p></a>

Hit "SAVE" and then "YES", and again "YES". At this point, say goodbye to everything that was stored previously on the SD card and wait until the program finishes the flash process. After termination, plug out the SD card and plug it into the Raspberry Pi. Now if you recall the [wiring diagram](#wiring), plug in your power supply into the <b>micro-USB port on the right!</b> The left port is for data transfer. Wait a few minutes. A green LED should be flashing in your Raspberry Pi and a red LED should be shining on the PN532 module. Make sure your computer is connected to the same network as your Raspberry Pi. Open a command line of your choice (I prefer using [Microsoft PowerShell](https://learn.microsoft.com/en-us/powershell/)). Now, let's take the credentials that I've chosen in [my configuration](#configurationOS) and try to connect the very first time via [SSH](https://en.wikipedia.org/wiki/Secure_Shell):

<a name="loginCommand">

		ssh smolinde@nfctutorial.local

</a>

The shell will prompt some security information. Ignore it, type in "yes" and hit <kbd>ENTER</kbd>. Now type in your password. Don't worry, although no letters appear on the screen, you are typing. After entering the password, hit <kbd>ENTER</kbd> again. Your screen should look something like this:<br>
<a name="firstShellLogin"><p align="center"><img src="./images/first_ssh_login.png" width="700px"></img></p></a>

Congratulations! You have now a running omnipotent computer.

### 2.4 Raspberry Pi Configuration
Before installing all required components to work with NFC cards we first have to do some steps to ensure that the Raspberry Pi is set up correctly to work as expected. A golden rule is to run the following command right after the first login:

		sudo apt update -y && sudo apt full-upgrade -y

Wait until the updates are installed. Now we have to do the following things:
* Enable SPI
* Expand file system
* Create a Hotspot
* Add 4GB of SWAP memory

Type the following command in your command line and hit <kbd>ENTER</kbd>:

		sudo raspi-config

With arrow keys, navigate to "3 Interface Options", hit <kbd>ENTER</kbd>, navigate to "I3 SPI", and hit <kbd>ENTER</kbd> again. Select "\<Yes\>", hit <kbd>ENTER</kbd> two times. Now navigate to "6 Advanced Options", hit <kbd>ENTER</kbd>, now you should select "A1 Expand Filesystem" with <kbd>ENTER</kbd> again. Wait a little bit, hit <kbd>ENTER</kbd> after the file system was resized. Now hit the <kbd>TAB</kbd> key two times and <kbd>ENTER</kbd> to finish, save, and close Raspberry Pi configuration GUI. It will ask you whether you would like to reboot, hit <kbd>ENTER</kbd> to do so. Now wait a few moments until the Raspberry Pi reboots. Repeat the [login command](#loginCommand). Now we are going to set up the wireless hotspot that will be activated when your home network is out of reach. This will allow you to connect to the device anywhere from any device (later more). To do so, run the following command:

		sudo nmcli dev wifi hotspot ifname wlan0 ssid "MyNetworkName" password "MySecurePassword"

The shell will disconnect immediately. Go to your computers WiFi settings, find the new network, and connect with your password. After this, run the [login command](#loginCommand) in your command line again. Now you can run:

		sudo nmcli con edit Hotspot

Inside of here, run the following commands:

		set connection.autoconnect yes
		set connection.autoconnect-priority -10
		set 802-11-wireless.hidden yes

Now type and run "save", type "yes" if asked, and "quit" to leave editing window. Now run:

		sudo nmcli con mod "preconfigured network (CHECK!!!)" connection.autoconnect-priority 10

This ensures that your home WiFi will be preferred and the Hotspot will be activated if and only if there is no other network in reach. You have to do this only once unless you decide to change your WiFi network properties. In that case you can always edit the existing connection (e.g. changing the password). I also recommend to hide the hotspot so that if you are using it somewhere else, people will not really see the additional existing network that is broadcasted by your Raspberry Pi. Now for key recovery we have to add a bit of RAM to our system, which is possible with [SWAP memory paging](https://en.wikipedia.org/wiki/Memory_paging). Run the following commands:

		sudo fallocate -l 4G /swapfile
		sudo chmod 600 /swapfile
		sudo mkswap /swapfile
		sudo swapon /swapfile

Now also edit the following configurato file with:

		sudo nano /etc/fstab

At the bottom of this file, paste the following line:

		/swapfile swap swap defaults 0 0

Press <kbd>CTRL</kbd>+<kbd>O</kbd>, <kbd>ENTER</kbd>, and then <kbd>CTRL</kbd>+<kbd>X</kbd> to save and close the file. Now also open the following file:

		sudo nano /etc/sysctl.conf

At the bottom of this file, paste the following line:

		vm.swappiness=100

Press <kbd>CTRL</kbd>+<kbd>O</kbd>, <kbd>ENTER</kbd>, and then <kbd>CTRL</kbd>+<kbd>X</kbd> to save and close the file. Reboot the Raspberry Pi via:

		sudo reboot now

Wait until the process is finished and continue with the software setup section.

### 2.5 Software Setup
For now, we have successfully prepared the hardware and Raspberry Pi configurations. Now we have to install and configure software components that we will use throughout this tutorial. The following command will do most of the work:

		sudo apt install screen git libnfc-dev libnfc-bin hexcurse autotools-dev autoconf libtool libusb-dev liblzma-dev ipheth-utils -y

After installing all new components, edit the following file with:

		sudo nano /etc/nfc/libnfc.conf

At the bottom of this file, paste the following two lines:

		device.name = "PN532 Reader (SPI)"
		device.connstring = "pn532_spi:/dev/spidev0.0:50000"

Press <kbd>CTRL</kbd>+<kbd>O</kbd>, <kbd>ENTER</kbd>, and then <kbd>CTRL</kbd>+<kbd>X</kbd> to save and close the file. At this point it makes sense to check whether we can actually access and use the PN532 module. To do so, run the following command:

		sudo nfc-list

The output should look something like this:

		nfc-list uses libnfc 1.8.0
		NFC device: PN532 Reader (SPI) opened

The version might be newer. If you see this output, then everything works as expected and soon you will be able to run your very first read/write commands! If the output yields errors or something else, go through all previous steps I have described so far or open !HERE! an issue if you are really stuck. Else continue with the installation of the key recovery tool. Run the following commands:

		git clone https://github.com/nfc-tools/mfoc-hardnested.git && cd mfoc-hardnested
		autoreconf -vis
		./configure
		make && sudo make install
		cd .. && rm -r mfoc-hardnested

These commands will install the desired recovery tool called [mfoc-hardnested](https://github.com/nfc-tools/mfoc-hardnested). At this point it is time for a legal disclaimer:


<b>THE METHODS PRESENTED BELOW ARE ONLY TO BE USED WITH OWN TAGS AND SHALL NEVER BE USED TO GAIN ILLEGAL ACCESS TO INFORMATION STORED ON TAGS THAT BELONG TO OTHERS THAN YOU. THE AUTHOR OF THIS TUTORIAL DOES NOT CARRY ANY RESPONSIBILITY IF THE BELOW LISTED METHODS ARE APPLIED IN UNLAWFUL WAYS. USE WITH CAUTION. USE AT YOUR OWN RISK AND INFORM YOURSELF ABOUT LOCAL LEGAL REGULATIONS.</b>

With this being said, we can now have a deeper look into the methods of how to read, write, format, and recover MIFARE classic cards.

## 3 Basic NFC Operations

## 4 Key Recovery

## 5 Smartphone Cards

## 6 Conclusion

## 7 Credits

