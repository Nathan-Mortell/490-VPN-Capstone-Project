# 490-VPN-Capstone-Project-Instructions

This Repo is to show the installation process I did to make my own Portable VPN, what to do if you have to work with an Apple Router, and an optional section on how to (maybe) install an Ad Blocker with the VPN.

## Technologies Used

- Raspberry Pi 5
- Wireguard
- OpenVPN
- Linux
- PiVPN

## Instructional Guide
The first step of this guide is pretty simple, you must first buy a Raspberry Pi. It's up to you if you want to get any of the additional upgrades (like a case or extra mini to HDMI cord). It's mandatory to have have 1 micro to HDMI cord, the power cord, and an SD card with around 16GB (the installation will not use all this space). I would recommend buying the complete bundle on Amazon for $100. Make sure the RAM is at least 8 GB. Make sure you also have a monitor, keyboard, mouse, and ethernet to set this up; it's essentially a micro-computer. And make sure you have easy access to your router, for this installation, I'll talk about what to do if you have an Apple router.

When first setting up your Pi, follow the default installation process on screen, I recommend installing the GUI so the installation process will be easier. Make sure to enable SSH and use password authentication. Set up your own username and password and do not connect the device over Wifi for the time being, if the Pi cannot get plugged into an ethernet, then do Wifi, and do local time setting. 

![test](https://imgur.com/7WSuqW2.png)

After the Pi is done booting up you'll be prompted to log in with your password, after that, you'll have a screen similar to mine shown below, since this is Ubuntu the overall GUI design is very similar to Windows 10.

![test](https://imgur.com/7eDt5cZ.png)

Now go to the bottom CMD terminal icon and once it's open were gonna run an update command to make sure the device is up to date before we being installing the VPN. Run the command "sudo apt update && sudo apt upgrade -y" sudo is for giving the command administration permission, and this will then update your Pi.

This is when the process can get tricky, and you'll need to understand some network concepts as the process goes along. Before installing the PiVPN we'll first be setting up a DHCP reservation on your router for the Pi. The reason for doing this is that normally, the IP address of each device on your home network will change every so often, and this will impact the Pi since it's going to be your VPN. Making a reservation in your router will fix this and will keep the IP address static. For this process, I had to do it on an Apple AirPort Extreme via a Macbook. Once you get your Macbook open the Finder App and type in AirPort Utility. You then see your Apple router and its connection to the internet, click on the router and then edit at the lower right. After that go to the network tab, then under DHCP reservations select the + icon. From there you'll be prompted to give a description, reserve address, Mac, and IPv4 address. For the description name it something along the lines of Pi Reservation, for reserver address to MAC Address, then for the Mac address we'll need to go back to your Pi and get both its MAC and IPv4. To do this type in "ifconfig" and from there we need the inet for the IPv4 and the ether for the MAC address, mine is shown below.

![test](https://imgur.com/Kc8cijS.png)

The next step is to now begin installing PiVPN. Open the CMD terminal icon again and type "curl -L https://install.pivpn.io | bash." This is taken from the PiVPN website and will begin the installation procedure and open the installation wizard shown below. 

![test](https://imgur.com/SDX7vcQ.png)

When you get to the step where you are asked that you want to used a DHCP Reservation, 
