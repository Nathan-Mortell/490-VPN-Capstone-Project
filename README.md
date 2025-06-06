# 490-VPN-Capstone-Project-Instructions

This Readme is to show the installation process I did to turn my Raspberry Pi 5 into a Portable VPN, what to do if you have to work with an Apple Router, and an optional section on how to (maybe) install an Ad Blocker with the VPN.

## Technologies Used

- Raspberry Pi 5
- Wireguard
- OpenDNS
- Linux Ubuntu
- PiVPN
- Apple Airport Extreme
- Tailscale (sort of)

## Instructional Guide
The first step of this guide is pretty simple, you must first buy a Raspberry Pi. It's up to you if you want to get any of the additional upgrades (like a case or extra mini to HDMI cord). It's mandatory to have have 1 micro to HDMI cord, the power cord, and an SD card with around 16GB (the installation will not use all this space). I would recommend buying the complete bundle on Amazon for $100. Make sure the RAM is at least 8 GB. Make sure you also have a monitor, keyboard, mouse, and ethernet to set this up; it's essentially a micro-computer. And make sure you have easy access to your router, for this installation, I'll talk about what to do if you have an Apple router.

When first setting up your Pi, follow the default installation process on screen, I recommend installing the GUI so the installation process will be easier. Make sure to enable SSH and use password authentication. Set up your own username and password and do not connect the device over Wifi for the time being, if the Pi cannot get plugged into an ethernet, then do Wifi, and do local time setting. 

![test](https://imgur.com/7WSuqW2.png)

After the Pi is done booting up you'll be prompted to log in with your password, after that, you'll have a screen similar to mine shown below, since this is Ubuntu the overall GUI design is very similar to Windows 10.

![test](https://imgur.com/7eDt5cZ.png)

Now go to the bottom CMD terminal icon and once it's open were gonna run an update command to make sure the device is up to date before we being installing the VPN. Run the command 
```sh
sudo apt update && sudo apt upgrade -y
```
sudo is for giving the command administration permission, and this will then update your Pi.

## Installing and AD blocker on your IP VPN (Optional and Maybe works)

This step is something I intended on doing. When I finished my capstone project I wanted to go back and make it so I could make a block list of ad domains. But I soon realized that this step should've been done even before installing the PiVPN on my Pi. What I found to be the best way of blocking ads is through a WireGuard based VPN protocol called Tailscale. It allows you to connect all of your device like WireGuard and lets you pick a DNS server to use, and its free. The first step to install it on your Pi, open the CMD terminal and type: 
```sh
curl -fsSL https://tailscale.com/install.sh | sh
```

After the process is complete, run the following command: 
```sh
tailscale up --accept-dns=false
```
The reason for running this command is since Pi-Hole uses DNS servers configured within Linux as its upstream servers, where it will send DNS queries that it cannot answer on its own. Since we're going to make the Pi-Hole be our DNS server, we don't want Pi-Hole trying to use itself as its own upstream. After that, install tailscale on your other devices with the link here https://tailscale.com/download . After that you will set up your Pi as your DNS server via Tailscales admin console. You can find your Pi's Tailscale IP address from the machines page of the admin console. Go to the Nameservers section of Tailscale and select add nameserver, type in the IP address and then enable Overide DNS servers. After that you'll need to disable the key expiry. Go the machines page of the admin console again and then click disable key expiry on your Pi.

##

This is when the process can get tricky, and you'll need to understand some network concepts as the process goes along. Before installing the PiVPN we'll first be setting up a DHCP reservation on your router for the Pi. The reason for doing this is that normally, the IP address of each device on your home network will change every so often, and this will impact the Pi since it's going to be your VPN. Making a reservation in your router will fix this and will keep the IP address static. For this process, I had to do it on an Apple AirPort Extreme via a Macbook. 

## DHCP Reservation on Apple AirPort Extreme

Once you get your Macbook open the Finder App and type in AirPort Utility. You then see your Apple router and its connection to the internet, click on the router and then edit at the lower right. After that go to the network tab, then under DHCP reservations select the + icon. From there you'll be prompted to give a description, reserve address, Mac, and IPv4 address. For the description name it something along the lines of Pi Reservation, for reserver address to MAC Address, then for the Mac address we'll need to go back to your Pi and get both its MAC and IPv4. To do this type in "ifconfig" and from there we need the inet for the IPv4 and the ether for the MAC address, mine is shown below. Once you get yours, enter them into the reservation and click save, then update. You'll be prompted to restard your router and it'll take around 7 minutes to restart and enable the reservation. Now you have a dedicated DHCP reservation for your Pi.

![test](https://imgur.com/Kc8cijS.png)

##

The next step is to now begin installing PiVPN. Open the CMD terminal icon again and type:
```sh
curl -L https://install.pivpn.io | bash
```
This is taken from the PiVPN website and will begin the installation procedure and open the installation wizard shown below. 

![test](https://imgur.com/SDX7vcQ.png)

When you get to the step where you are asked if you want to use a DHCP Reservation, click yes since you already set it up beforehand. For the default user select the account you made on the Pi and then some more packages will be installed. Once those are done you'll be prompted to select between OpenVPN or WireGuard. These are not VPN providers but VPN protocols; a VPN protocol uses a set of instructions to establish a secure and encrypted connection between your device and the VPN server for the movement of data. After looking at multiple sources online, WireGuard is the preferred protocol over OpenVPN (the installation itself even recommends it). 

![test](https://imgur.com/V3V1f1y.png)


## WireGuard over OpenVPN

I'll be quickly going over WireGuard and why its as popular as it is. First, WireGuard has a simple network interface, it works by adding and removing routes with route(8) or ip-route(8) and works with all networking utilities. Specific aspects of the interface are configured using the wg(8) tool which acts as a tunnel interface. WireGuard provides an example of how it associates tunnel IP addresses with public keys and remote endpoints. When the interface sends a packet to a peer, it does the following:

1. This packet is meant for 192.168.30.8. Which peer is that? Let me look... Okay, it's for peer ABCDEFGH. (Or if it's not for any configured peer, drop the packet.)
2. Encrypt entire IP packet using peer ABCDEFGH's public key.
3. What is the remote endpoint of peer ABCDEFGH? Let me look... Okay, the endpoint is UDP port 53133 on host 216.58.211.110.
4. Send encrypted bytes from step 2 over the Internet to 216.58.211.110:53133 using UDP.

When the interface receives a packet, this happens:

1. I just got a packet from UDP port 7361 on host 98.139.183.24. Let's decrypt it!
2. It decrypted and authenticated properly for peer LMNOPQRS. Okay, let's remember that peer LMNOPQRS's most recent Internet endpoint is 98.139.183.24:7361 using UDP.
3. Once decrypted, the plain-text packet is from 192.168.43.89. Is peer LMNOPQRS allowed to be sending us packets as 192.168.43.89?
4. If so, accept the packet on the interface. If not, drop it.

The other aspect of WireGuard that makes it popular is the concept called Cryptokey Routing. This works by associating public keys with a list of tunnel IP addresses that are allowed inside the tunnel. Each network has a private key and a list of peers. Each peer has a public key. Public keys are used by peers to authenticate each other and can be passed around for use in configuration files by any out-of-band method. It's similar to how one might send their SSH public key to a friend for access to a shell server. In the server configuration, each peer (a client) will be able to send packets to the network interface with a source IP matching the corresponding list of allowed IPs. For example, when a packet is received by the server from peer gN65BkIK..., after being decrypted and authenticated, if its source IP is 10.10.10.230, then it's allowed onto the interface; otherwise, it's dropped.

These built-in components and more are why WireGuard is preferred over other protocols like OpenVPN, back to the installation guide.

When you select WireGuard, you'll be shown the default WireGuard port, make sure it says its UDP number "51820", if it doesn't, change it to it. This number will be important later on when giving the VPN proper internet access.

![test](https://imgur.com/6XWQwiw.png)

##

After that, you'll be asked what DNS provider will the VPN be using. For this I selected OpenDNS since it's a free, trusted DNS provider, but you can select any of the other options if you already have a preferred provider. Then you'll be asked if clients will be using a public IP or DNS name to connect to your server. Since we already made a static IP address for our device, we'll be selecting the top option, but if your IP address was dynamic, then you would use the DNS entry and set up a dynamic DNS. 

![test](https://imgur.com/6eJHlR8.png)

You'll be asked if you want to enable unattended upgrades of security patches to your server. Do select yes since this will keep your PiVPN secure. After this, the VPN server keys will be generated and after that your Pi will restart multiple times. Once its done restarting, it's time to make some PiVPN accounts. Open the CMD termianal and type:
```sh
pivpn -a
```
If you're prompted to enter the client IP from a range, skip this. You'll be prompted to enter a name, I recommend just giving it the name of the device. Do this process for every device you want to connect; do not use the same account on multiple devices. 

![test](https://imgur.com/d3N7kFc.png)

If you want to see all current accounts, do 
```sh
pivpn -c
``` 
and if you want to remove any, do 
```sh
pivpn -r accountnamehere
```
If you want to make a backup of all your accounts in case of a crash, type: 
```sh
pivpn -bk
```
From there, it'll tell you where the backup has been saved and an instructional guide on how to migrate it.

![test](https://imgur.com/1hyf8M1.png)

## Enabling internet connection on Apple AirPort Extreme

Open your Macbook again and go to the Finder App, and type in AirPort Utility. Click on the router again and then edit at the lower right. After that, go to the network tab, then under Port Settings, and select the + icon. From there type "Personal Web Sharing" in the description, this will autofill the Public and Private TCP Ports. After that, reuse the WireGuard UDP number 51820 for both the private and public UDP ports. And last enter the PI's IP address for the Private IP. Click save and update, this will once again restart your router and enable the changes. 

##

When it comes to connecting mobile devices to your PiVPN we'll need to use the WireGuard app. Once you have it installed, go back to your PiVPN and in the CMD terminal we'll generate a qr code to scan, type: 
```sh
pivpn -qr
```
and you'll be prompted to enter the name of the client to show, enter the name for the mobile account you made and then a qr code will be generated. Back to your phone, select add a tunnel and create from QR code (make sure to enable camera access to the WireGuard app). Once you scan the qr code with the WireGuard app, name it and it'll appear as a toggable option in the app. To make sure the connection is stable, download an app that allows you to ping IP addresses, I found one simply called Ping. From there ping the IP address of the Pi and the gateway IP, an example of a successful device ping is below.

![test](https://imgur.com/ypt4Nxe.png)

For connecting the PiVPN to your desktop, I found this to be the simplest method. Download the WireGuard client for Windows and or Apple. Once it's installed, open it, and you'll see the option to import tunnels from a file. Since the files are generated on the Pi, we'll be using Google Drive to move the files from the Pi to your device. There will be a folder in your Pi called configs, and in there will be a .conf file for each account made. Open Google Drive on your Pi VPN by going to the web browser in the bottom left. Once in Google Drive select new and File upload. From there select the .conf file and it'll be uploaded to the drive. Go back to your device and open Google Drive on there, and download the .conf file and use that as the import tunnel from the file. A good way to check to see if the VPN is enable is to go to the website What Is My IP? Before you enable your VPN run the website to see what your current IP is. Now connect to the VPN and refresh the page and do it again, if the IP address is changed, then it's successfully connected. 

## Troubleshooting

If you're having issues with connecting to the internet when on the VPN, the first thing to check is your Apple router. Go through the DHCP reservation and enabling internet connection steps again to make sure you filled out each section correctly. If you are still having internet issues, go to your Pi CMD terminal and type 
```sh
pivpn -d
```
to begin the debugging procedure. If the process ever stops and prompts you to fix something, type Y. After the debugging is complete, type 
```sh
sudo reboot
```
to reboot the Pi so the changes take effect.

##
