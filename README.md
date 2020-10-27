This repo provides some basic info on Ubuntu 16 Xenial Xerus server version, some of which applies to linux in general.

In this repo, you'll sometimes see duplicate files, one of which ends in "_USB". The one **without** "_USB" is meant for a ubuntu running on a virtual machine, the other for a usb drive version of the same system (directly cloned from virtual machine to usb drive). The files are different, because on Windows, my linux relies on VMWare for NAT-ting, plus the virtual adapters are bridged to real network adapters, while, when booted from usb drive, it has direct access to the real network adapters.

So you decided to install linux and are unsure which distro to use. Now, I can't speak for all linux distros out there, but, judging Ubuntu by itself, I have reservations about it. By this, I don't mean to imply that other distros are faring better, I can only judge Ubuntu by itself. So first, it's distributed as a server version, so its primary purpose is networking. However, if you install it, you'll notice it ships tons of crap tools / system components completely irrelevant to a network admin, while important network programmes are missing completely. They may be in the image, but the basic installation doesn't install them, while I expect them to be part of a minimal installation of a **server** edition. The installer is not exactly user-friendly, so you end up with either an empty system containing nothing of importance in regard to networking and have to install everything afterwards, or with a system full of crap (actually, you end up with a system full of crap in either case).

So, yes, you can uninstall bullshit stuff and install the missing stuff afterwards, but then the vanilla version obviously must not be advertised as network-capable, but rather as a distro capable of **attaining** network capability, if you invest time (very MUCH time) into adjusting it.

Besides, installing missing programmes is problematic with Ubuntu, because the packages provided by

	packages.ubuntu.com

are always outdated, but if you search the debian repository:

https://www.debian.org/distrib/packages

They provide packages which are much up-to-date. So if a particular version of a programme is not working, you either have to compile a newer version yourself, or download the debian package, because Ubuntu distributors take much time to update their packages. 'cause they are lazy. Now, you may think, debian, ubuntu, who cares, it's pretty much the same stuff, packages should work on both systems. I am not certain about that. The packages I downloaded off debian did work on Ubuntu, but there's obviously no guarantee ALL of them work across both systems.

Here's a very informative and clean man page, not just for Ubuntu users, but for linux in general:

	"XXX" site:manpages.debian.org

So basically, search on google, replacing "XXX" by your search term.

Now, about auto-login as root. Unfortunately, most people, when asked about this, will just answer with lamentations on how important security is and won't tell you how to do this. How here's a short instruction:

/etc/shadow : remove root password by deleting everything ("XXX") between the first and second colon:

	root:XXX:

/lib/systemd/system/getty@.service (root auto-login):

	ExecStart=-/sbin/agetty -a root --noclear %I $TERM

remove the user account XXX that was set up when installing Ubuntu (no need since we'll run as root):

	userdel --remove XXX

If you are running the server in a virtual machine, then you may prefer to edit plain text files in your favourite text editor on Windows, then push them back to Linux over the folder the virtual guest shares with the host (using VMWare Workstation here). In that case, don't forget the carriage return problem, as some programmes on linux, like radvd, are sensitive to this and may refuse to run. So format line breaks as "\n" (Windows: "\r\n" !) before pushing the files to linux.

Now, the most important thing, the services. You may want to disable services to save memory/cpu load, primarily, if you are running the virtual machine in the background as a router and want to keep it small and unobtrusive. The directories you want to look at are:

	/lib/systemd/system

plus:

	/etc/init.d/

These are the primary folders where services are kept, but there are other folders, like:

	/etc/systemd/system

	/etc/rcXXX.d/

	/run/systemd/generator

	/run/systemd/generator.late

The whole service structure is extremely obscure, which gets even more problematic due to the fact that different linux distros use different folders, so it is difficult to find information on the internet for your particular distro. Personally, I haven't messed with the folders mentioned last. What I can tell you, is that if you disable a service in:

	/lib/systemd/system

Ubuntu will switch to:

	/etc/init.d/

and STILL execute the service from there. So if you want to disable the service completely, you need to disable the service at:

	/lib/systemd/system

and delete the service file from:

	/etc/init.d/

However, not all services can be disabled, since some of them are static. Now, people will tell you to use:

	systemctl mask ...

But, personally, I had problems with this. It's much better to delete such a static service file directly from "/lib/systemd/system", delete the corresponding service from "/etc/init.d/" and reboot the system.

The commands that are useful for working with services are "systemctl" and "service":

	service --status-all

	systemctl list-unit-files

	systemctl list-unit-files --type=service

	systemctl list-unit-files --state=enabled

	systemctl list-unit-files --state=static

	systemctl status XXX

	service XXX status

	systemctl is-enabled XXX

	systemctl show XXX

	systemctl enable XXX

	systemctl disable XXX

	systemctl start XXX

	systemctl stop XXX

To check your system performance, which is primarily affected by services, use:

	systemd-analyze time

	systemd-analyze blame

	systemd-analyze critical-chain

Running Ubuntu in VMWare Workstation can be problematic due to several problems:

VMWare Tools by VMWare themselves are not working properly. Tools provided by Ubuntu:

https://help.ubuntu.com/community/VMware/Tools

are working, but have limited functionality.

Running non-English versions of Ubuntu in Workstation has a very weird bug - if you set your linux keyboard layout to, let's say, German, then paste something to the virtual machine using VMWare GUI, then keys will be interpreted as English strokes, so the characters will be adjusted based on the English keyboard setup. This problem has been reported to VMWare, but they (surprise!) don't do anything about this. 'cause they are lazy. So you are obviously better off using ssh to interact with the machine, primarily, if you have it run in the background without UI. This is not a problem with guest systems that support the VMWare version of VM Tools, but with Ubuntu, VMWare just virtually executes the key strokes, "typing" them into the guest, and since it weirdly is "typing" on an English "keyboard", it breaks the string pasted into it.

There's a repo on github that claims to fix the bugs that prevent the compilation of the native VMware Tools (released by VMWare themselves) into linux, but that repo is not working, maybe it's not meant for the CLI version of linux.

Now, getting the ubuntu version of the VMware Tools to add a shared folder into the guest system is a damn science in itself in that there's little documentation, and syntax posted on the net is mostly not working. It's also important to init /usr/bin/vmhgfs-fuse at a proper moment, lest you like a crash on boot. So I added it pretty much at the end of the boot process:

	rm /lib/systemd/system/rc-local.service

Now "/etc/init.d/rc.local" will be executed ("/lib/systemd/system/rc-local.service" was causing problems) and will execute the following script:

https://github.com/WRFan/ubuntu/blob/main/etc/rc.local

adding:

	/mnt/shared/

shared folder, which you can use to copy files from your host harddisk to directories on the guest system and vice versa.

Another problem is that the VMWare virtual harddisk file of a linux guest gets larger and larger, because it' not compacted properly (due to ext4 format?), so when checking used harddisk space in the guest system itself, the size will be much smaller than the size of the actual harddisk file on your real harddisk, although harddisk space is not pre-allocated. You basically need to delete all your snapshots, run the virtual machine, fill all empty space with zeroes:

	dd if=/dev/zero of=/wipefile bs=1024x1024; rm wipefile

then shutdown the machine and compact the virtual harddisk through VMWare GUI (Edit Virtual Machine Settings).

Now about networking. Windows (non-server versions) are not exactly good at NAT-ting or ipv6 prefix delegation, so the purpose of my virtual machine running Ubuntu is to serve as a router (cable modem). Now I am not going into detail about Microsoft ICS, suffice to say it's total crap and ridden with bugs, which makes it unusable. So I am using ubuntu for networking tasks. This poses two problems.

ipv4 ip address is requested by the Windows in-built dhcp server, so the PC itself has access to ipv4, but, obviously, the other devices connected to the network do not.

An ipv6 prefix must be requested from the ISP and then delegated to all ipv6 capable devices on the network, something MS ICS attempts to achieve, but fails pathetically, for many reasons, but especially, because it doesn't adhere to standards and misinterprets the replies of the ISP server when requesting the prefix:

Windows:

	DHCPv6 Solicit XID

	Option: Identity Association for Prefix Delegation (25)

	Enterprise ID: Microsoft

	vendor-class-data: MSFT 5.0

ISP first reply:

	DHCPv6 Advertise XID

	Status Message: No prefix available on Link 'XXX'

ISP second reply:

	DHCPv6 Advertise XID

	Prefix length: 59

	Prefix address: 2a02:XXX:XXX:XXX::

So this pathetic Windows dhcp client gets confused by the first reply, while the consortium clearly states the dhcp client is to ignore such a reply and consider any other replies it gets. If you look at the output by dhclient on linux, it will properly state something like "Ignoring reply not supplying a prefix, checking other replies for prefix presence". Windows dchp client instead just pathetically re-solicits, obviously getting the same reply from the ISP. So no prefix for you. There are other reasons too why the Windows client will fail to request a prefix, so you are ipv6-less most of the time if you rely on this MS crap client.

The whole problem is, nobody knows what MS actually compiled into their system. Go assault Redmond to acquire their code! The question is, what makes the Windows dhcp client request a prefix in the first place. It's actually ICS, when enabled that is making it do it. Anything concerning ICS and dhcp, their actual job and their interrelationship are completely unclear.

Also, this morose ICS is setting on SLAAC-ed systems:

	Connection-specific DNS Suffix: mshome.net

Bullshit! Check out the shit that this causes. Open a permission settings dialog in any programme you want, Windows Explorer (Right click on drive -> Settings -> tab Security, regedit (registry permissions dialog), whatever. Now try editing permissions. Programme that loaded the permissions dialog freezes. Freezes? Why freezes? Why, the damn system tries to connect to DNS to resolve this nonsense set by ICS. The firewall blocks, and what exception could you ever add considering the fact the permissions component can be loaded by ANY programme on Windows? So, Russinovich, HEEEELP! Yeah, you got it, his Process Monitor. So after 3 hours I stumble upon:

	HKLM\SOFTWARE\Policies\Microsoft\Netlogon\Parameters

		AllowDnsSuffixSearch (REG_DWORD) -> 0

Of course, google returns nothing on this issue, but once you actually find the answer, voila! suddenly, google knows everything... of course, you actually need to type the answer into the search box.

Now look at:

	%systemroot%\System32\drivers\etc\hosts.ics

Microsoft can't even get their own damn extensions right, since ".ics" actually points to "Outlook.File.ics.15" in registry. is this file an Outlook file? Damn stupidity all over!

So I am using dhclient to request the prefix. Now, there's been many lamentations about dhclient, and in fact, probably any other dhcp client out there is better suited for requesting IPs (except for the MS dhcp client, lol), but it all depends on your ISP, and as far as I am concerned, dhclient gets it job done satisfactorily.

Now, there's no communication whatsoever between dhclient and the actual SLAAC-er (using radvd), so you have to make sure yourself the prefix attained by dhclient is actually passed to radvd. Depending on linux distro used, this whole process varies, so finding info on the net is pretty much impossible. So again, I am talking about Ubuntu 16 server edition! Keeping this in mind, let's go over the individual steps:

https://github.com/WRFan/ubuntu/blob/main/etc/network/interfaces

https://github.com/WRFan/ubuntu/blob/main/etc/network/interfaces_USB

https://manpages.debian.org/unstable/bridge-utils/bridge-utils-interfaces.5.en.html

The interfaces_USB is the USB drive version (for linux boot on your real computer).

The "interfaces" file sets up your network, but it's obscure, because the commands passed to it are not the real commands that are executed later, rather, the system interprets these commands and executes the real stuff in the background. So to make this less obscure, I am using the actual commands in the interfaces file.

As you can see, the loopback is initialised first. Nothing extraordinary with this in the virtual machine file, but if you take a look at the USB version of the file, you'll notice I am loading two network modules. This is due to the problem that linux may load your network adapters in the wrong order, so you can never be sure what your eth0 or eth1 will be. So to make sure the order stays the way we want it, we first disable the auto-load of network modules through:

https://github.com/WRFan/ubuntu/blob/main/etc/modprobe.d/blacklist.conf

then load them manually through the "interfaces" file in the right order.

Next, the NAT bridge. Now, this is unnecessary for the USB drive version, because there linux has direct access to the network adapters, so dhclient is responsible for requesting both ipv4 and ipv6, NAT is realised through iptables, but on our virtual machine, the virtual linux guest gets its ipv4 through the VMWare Workstation. Now, of course, it could be possible to bridge the ipv4 interface of the virtual linux to the real adapter and request ipv4 through dhclient there too, but then your Windows machine is losing control over ipv4 and has to rely on the virtual machine, something that is unnecessary, since the Microsoft dhcp client requests the ipv4 IP from the ISP without problems. So by adding the VMWare NAT interface to the virtual guest the virtual machine gets access to ipv4, but that's irrelevant, what we want is to NAT ipv4 to our real devices like SIP ATA boxes or our mobile phones. So we need to make the VMWare internal NAT gateway ip visible to the entire network.

"But why rely on VMWare NAT? Aren't there any Windows native programmes that realise NAT?", you may ask.

Such as? "ICS", you'll say. Again, ICS causes problems all over, plus, it's completely and utterly impossible to get it working on WinPE (now haven't I tried for 7 days and nights, failing pathetically!). Now I don't want to miss phone calls when I am working on WinPE, but my SIP ATA relies on ipv4 (thank you, Cisco, for NOT adding ipv6 support!), so requires NAT to connect to the SIP server.

Apart from ICS, there are a couple of NAT programmes that claim to be capable to provide NAT on Windows, but they all failed miserably when tested. I'll just name Wingate, now that one is simply pathetic, eats up your CPU by hammering senseless requests to the system and doesn't NAT anything.

Other than that, there's WinPkFilter, which looked pretty good to me first, so I actually compiled a binary, which was supposed to auto-run on Windows boot initialising NAT through the WinPkFilter driver. It seemed to be working all right, but when accessing certain ipv4 addresses (don't remember exactly which ones, pinging local network addresses or something), the driver was getting into an eternal loop. I was first thinking I made a mistake while writing the code, but the small NAT programme supplied with WinPkFilter exhibits the same behaviour. All these nonsense made the Windows tcpip stack completely unstable, it was the first time since the installation of Win10 I was actually forced to use the Reset button on my computer. Now killing Win10, which is incredibly stable, is a great achievement, heads up, WinPkFilter! But personally, I must say, I don't like the idea of instability, so WinPkFilter flew right into the Recycle Bin. As did all other NAT programmes I tested... including ICS (great work, dism!). So back to VMWare NAT.

As you can see, "interfaces" bridges "eth0" (bridged to the second real network adapter, which is connected to the network, but is NOT the one talking to the cable modem) and "eth2" (the VMWare NAT adapter). This makes the VMWare NAT gateway accessible to the entire network, not just the virtual guest systems.

Next is "eth1". That's a virtual adapter bridged to the first real network adapter (directly communicating with the cable modem). Because a cable modem can only ever communicate with a single MAC address, both the real network adapter and the virtual adapter bridged to it share the same MAC address. Conflict? No, as long as you obviously disable the ipv6 interface on your real network adapter in Windows.

The USB "interfaces" file is less complicated - there "eth0" is the real network adapter talking to the cable modem, the USB version of dhclient requests both ipv4 AND ipv6. NAT realised through iptables.

Now talking about iptables, this can be confusing to a Windows user. If you enable NAT through ICS on Windows, then do:

	ping 8.8.8.8 -S 192.168.0.13

where "192.168.0.13" is the IP of the interface that relies on NAT, the network adapter will successfully ping the IP. This is different for iptables with

	net.ipv4.ip_forward

set to 1. The NAT router ITSELF will not ping global ipv4 addresses, so a Windows user will conclude NAT is not working, while it actually does, you need to test it differently: Set the ipv4 gateway address of your NAT router in your mobile phone settings and try to ping ipv4 global addresses on your phone. If it's working, congratulations, NAT's working.

There's something else to keep in mind. Any Windows native NAT programmes that I mentioned above (including MS ICS) require forwarding set on the interface serving as a NAT router:

	netsh interface ipv4 set interface "Ethernet 2" forwarding=enabled

Now it took me some time to realise, why ICS NAT-ting was working, while WinPkFilter NAT was returning a "general error" for pings. That's because ICS enables forwarding automatically when the SharedAccess service is enabled and disables forwarding when it's disabled, but the other programmes like Wingate/WinPkFilter do not, they also don't mention this in their forums/documentation, so I can only assume they are not even aware of this problem. I found it out completely by accident, no single page on the entire internet mentions it, unless Google misunderstood my desperate question.

Note that my VMWare solution is not affected by any of this, nor does it require forwarding or net.ipv4.ip_forward being enabled:

VMWare NAT:

https://github.com/WRFan/ubuntu/blob/main/etc/sysctl.conf

iptables NAT:

https://github.com/WRFan/ubuntu/blob/main/etc/sysctl_USB.conf

Now talking about "sysctl.conf", and kernel parameters in general, Ubuntu spreads all parameters over dozens of files on the system, and if there are duplicate values, there's a precedence order between those conf files so the kernel can decide which value to use. This is incredibly confusing, and you do best to delete all those files and keep ALL your kernel settings in just one file, I am using just this "sysctl.conf" file.

So much for ipv4, but now consider further the ipv6 process. dhclient configuration is handled by:

https://github.com/WRFan/ubuntu/blob/main/etc/dhcp/dhclient.conf

yet there's hardly any information on how to configure the actual IP acquisition process from the ISP. For example, you want dhclient to request a prefix of a specific length, specific "Preferred lifetime", or "Valid lifetime". Now, dhclient man pages will tell you this and that, other pages will tell you to do that and this, but NOTHING of what they claim is of any value.

The greatest problem is those people are not actually telling what version of dhclient they use, maybe they compile their own version, or it's a different linux distro, I don't know. The only ones who know how to setup the ubuntu version of dhclient is ubuntu themselves, and, man, they sure as hell are NOT talking. Yet it's primarily the pathetic nature of dhclient itself that makes any adjustments difficult or even impossible; again, this is very dependent on your ISP whether dhclient will be of ANY use to you. Now, fortunately, my ISP recognises the stupidity of dhclient and actually fixes its pathetic requests:

dhclient:

	DHCPv6 Solicit XID

	Option: Identity Association for Non-temporary Address (3)

ISP:

	DHCPv6 Advertise XID

	Option: IA Address (5)

	Preferred lifetime: 604800

	Valid lifetime: 1209600

dhclient:

	DHCPv6 Request XID

	Option: IA Address (5)

	Preferred lifetime: 7200

	Valid lifetime: 7500

ISP:

	DHCPv6 Reply XID

	IPv6 address: 2a02:XXX:XXX:XXX:XXX:XXX:XXX:XXX

	Preferred lifetime: 604800

	Valid lifetime: 1209600

dhclient:

	DHCPv6 Solicit XID

	Option: IA Prefix (26)

	Preferred lifetime: 0

	Valid lifetime: 0

	Prefix length: 59

	Prefix address: ::

ISP:

	DHCPv6 Advertise XID

	Preferred lifetime: 604800

	Valid lifetime: 1209600

	Prefix length: 59

	Prefix address: 2a02:XXX:XXX:XXX::

dhclient:

	DHCPv6 Request XID

	Option: IA Prefix (26)

	Preferred lifetime: 7200

	Valid lifetime: 7500

	Prefix length: 59

	Prefix address: 2a02:XXX:XXX:XXX::

ISP:

	DHCPv6 Reply XID

	Option: IA Prefix (26)

	Preferred lifetime: 604800

	Valid lifetime: 1209600

	Prefix length: 59

	Prefix address: 2a02:XXX:XXX:XXX::

Look at this pathetic dhclient! It's told Preferred lifetime until deprecation is a **week**, yet it stubbornly requests 2 damn minutes! Thank you, dear ISP, for not listening to this idiotic client! Now if you figure out how to set "Preferred lifetime" on the UBUNTU (Ubuntu! Not Fedora or something!) version of this programme, add a topic to:

https://github.com/WRFan/ubuntu/issues

Personally, I've given up on communicating with this programme, it doesn't listen to me and just does what it wants. Fortunately, my needs are satisfied despite all these problems, but I've heard people whining about rfc3442-classless-static-routes (what's that?) not being handled correctly by dhclient, overall everybody agrees this client is shitty.

Talking about "IA Address", why the hell am I requesting it, if all my needs are covered by the prefix? Why the hell is ANYBODY requesting it? Because if you look on the internet, people will tell you to set the /128 address on the router and delegate the prefix to everything else. Why, what's wrong with using the prefix for the router itself? Has everyone got nuts out there? So I am like, no "$new_ip6_address" for me, thank you, so I requested a prefix from the ISP right away. But since everyone is insane out there, my morose ISP will not give me a prefix until I've actually requested this useless "IA Address". So great, I request it, there you go, ISP, satisfied? Then I throw it away and use the prefix for everything including the router:

https://github.com/WRFan/ubuntu/blob/main/sbin/dhclient-script

	if [ -z $new_ip6_prefix ]; then

		exit 0

	fi

Another problem with dhclient is that it considers a prefix a valid one when re-binding, even if the ISP specifically tells it it's been deprecated. So here we go, I have to waste my time to fix this pathetic programme's bugs:

	if [ $old_preferred_life ] && [ $old_preferred_life -eq "0" ]; then

		rm /var/lib/dhcp/dhclient6.leases

		exit 0

	fi

For the actual syntax dhclient uses check out the "interfaces" files on this repo.

So great, we've got our ips, now what? dhclient is not talking to radvd, so it's (again) up to us. radvd uses:

https://github.com/WRFan/ubuntu/blob/main/etc/radvd.conf

	prefix 2a02:XXX:XXX:XXX::/64

So we create a template on Ubuntu, then tell dhclient to update the prefix inside "radvd.conf" whenever it acquires one. Again, check:

https://github.com/WRFan/ubuntu/blob/main/sbin/dhclient-script

Btw, my "radvd.conf" states:

	AdvPreferredLifetime 30

So I set the "preferred lifetime" of the SLAAC-ed prefix to 30 seconds, because if I disable the router (for whatever reason), Windows must realise it as quickly as possible and deprecate the prefix. Sure, there's an option in radvd to send a "bye bye" on shutdown, but I sometimes just shut down the virtual machine itself, so radvd is just purged from memory, so sends nothing.

Now we kill the dhclient (no need to have it sitting in memory, we've got our prefix for a week) and run radvd through /etc/network/interfaces .

For radvd, check:

https://manpages.debian.org/unstable/radvd/radvd.8.en.html

https://manpages.debian.org/unstable/radvd/radvd.conf.5.en.html

https://manpages.debian.org/unstable/radvdump/radvdump.8.en.html

Again, check your syntax for "radvd.conf" if you edit it on Windows! radvd has problems with the Windows way of line breaking, this carriage return problem is still haunting us after 60 years!

	radvd --configtest

I am running radvd directly, check

	systemctl status radvd.service

to ensure it's disabled. It must not be set to "Always run", since we need to acquire the ip and update "radvd.conf" before running it.

As you can see, I am calling radvd with the parameter:

	radvd --logmethod none

otherwise it whines about some irrelevant crap and fills your harddisk with useless bullshit log messages. Now judging by itself, I'd say radvd is crappy, but hey, I learnt to expect the worst, so compared to bullshit I sometimes witness on Windows, Linux, Darwin, this one fares ok in relation to other drek.

Now the question is, what network adapter are we going to use as our actual router? Pages on the internet tell you to use a network adapter that by itself does NOT have access to the cable modem and use

	net.ipv6.conf.all.forwarding = 1

to forward all requests to the network interface talking to the cable modem. Yet the question is, why? Why not use radvd to advertise the interface talking to your cable modem as the actual ipv6 gateway? So let's consider a real linux, not a virtual machine, for simplicity reasons:

https://github.com/WRFan/ubuntu/blob/main/etc/network/interfaces_USB

https://github.com/WRFan/ubuntu/blob/main/sbin/dhclient-script_USB

https://github.com/WRFan/ubuntu/blob/main/etc/radvd_USB.conf

eth0 -> interface talking to cable modem

eth1 -> actual router. not talking to cable modem. forwards all requests to eth0

radvd -> advertises eth1

Why? Why not advertise eth0?

Set ("sysctl.conf"):

	net.ipv6.conf.eth0.accept_ra = 2

At first, it looks good. "eth0" gets RAs from the ISP to set the route gateway set by the ISP. radvd advertises eth0, eth0 ignores radvd RAs, since it's smart enough to recognise it's its own link-local address that is being advertised. So no conflict between RAs. But now check out the incredible INCREDIBLE havoc this is causing! Unless you enable forwarding on an interface that is trying to use the router, 3/4 of all pings will go down! So you won't be able to access a single page out there. Enabling forwarding on every interface? Ridiculous! How do you enable forwarding on iOS that gets the ipv6 prefix SLAAC-ed to it? Go ask that damn Apple!

Now what causes ip requests to fail if forwarding is disabled? I have no idea, but let's just wireshark iPhone net requests. iOS forwarding is disabled, so we take a look at Wireshark output and see that each ipv6 request by iOS will loop itself into hundred thousands dupes! It fails to send tcp RST to reset the connection, instead, the requests repeat themselves indefinitely. So your wifi Access Point will be flashing continuously. I am absolutely at a loss what causes this, but obviously, all this comes down to forwarding, there MUST be forwarding somewhere, and so the obvious thing to do it to set forwarding on the router itself by using a different interface as your actual router and forwarding all ipv6 requests arriving at it to the interface talking to the cable modem.

Now great, we have that, but now radvd is advertising, and the ISP is advertising either! So the router will not pick up its own link-local address advertised by radvd, but the interface talking to the cable modem ("eth0" in this example) WILL! So it now has TWO routes, one assigned by ISP, the second by radvd. Bad bad bad! If it picks up the radvd route, it will forward all requests forwarded to it by "eth1" back to "eth1", DUPE!

Ok, why not disable RA listening on "eth0" by setting ("sysctl.conf")

	net.ipv6.conf.eth0.accept_ra = 0

and set the ipv6 manually on "eth0" based on the prefix (there's actually code for this - check "dhclient-script" on this repo and look for "ipv6calc")? All right, we disable RA listening, we've got a global ipv6 address onto the "eth0" interface, but how is Ubuntu supposed to set the default gateway, i.e., route, to the ISP? Set it manually? ISP gateway is not static, at least this is the case with my ISP. So I am setting

	AdvDefaultPreference low

in radvd.conf, since my ISP advertises its own gateway/route at

	Prf (Default Router Preference): Medium (0)

Of course, now every other interface that can't communicate with the cable modem will still try to use the ISP gateway first, since they see the ISP RAs too. But since "eth0" is the only one getting a response from the cable modem (only one MAC can talk to it), they will drop the ISP gateway and use the one advertised by radvd.

Concerning the network settings in "sysctl.conf", you can check them yourself on the internet. Still, I should mention a weirdness of linux. Concerning "net.ipv6.conf.XXX.forwarding" there's:

	sysctl net.ipv6.conf.all.forwarding

	sysctl net.ipv6.conf.default.forwarding

	sysctl net.ipv6.conf.XXX.forwarding

where XXX is the actual interface name, like eth0. These three variants are actually available for pretty much all system variables in "sysctl.conf", and nobody out there actually cares which are needed and which are not, instead, everybody just stupidly sets a variable to its value first on "all", then on "default", then on every single interface. From my experience I can tell you the "default" does nothing, so forget about it completely.

Now setting a variable to a specific value on a particular interface only instead of "all" makes sense, but check this out: in our example (usb drive), it should suffice to set forwarding on eth1 (actual router) only:

	net.ipv6.conf.eth1.forwarding = 1

Now reboot, BAMM! not working. Now set it like this:

	net.ipv6.conf.all.forwarding = 1

	net.ipv6.conf.eth0.forwarding = 0

Working. What the hell is the difference? It's exactly the same:

	eth0 (cable modem talker) -> forwarding disabled

	eth1 (router) -> forwarding enabled.

But no, Ubuntu makes it complicated and wants you to specifically set the "net.ipv6.conf.all.forwarding" ("ALL") to 1. Why make it easy if you can confuse people, right, Ubuntu? Don't know if this nonsense is specific to Ubuntu only. Maybe insanity lives in the kernel.

Now some words on the graphical support in Ubuntu CLI. It's a terminal version, so such basic support should be available, shouldn't it? It's just a damn cursor and white characters on a black background. Unfortunately, Linux isn't capable of providing even such basic support. Now, take care, if you have graphical problems, don't even attempt to use google. People will talk your ears off about installing nvidia drivers, zenity, xserver-xorg-video-nouveau, x11-xserver-utils, xrandr, nvidia-settings, xorg, etc. etc. None of this applies to Ubuntu server edition, but there's some kind of miscommunication between me and google, maybe you fare better. So the problem is this. As I said, I booted Ubuntu from an usb drive, and I got 16:9 ratio, plus the left and right edges of the screen were cut off. I am used to 4:3 ratio, that's what I am using on all my Windows systems, isn't linux even capable of setting ratio? So I searched for like 5 days, read everything I could find... nothing. I'm like, "google, I am using tty, HEEELP!". Google replies, "sure, x11 version? no problem!" NO, DAMN IT, NOT THE GUI VERSION!

Then they tell you to go edit "/etc/default/grub". Now am not even going to talk about that crap file. The actual file used is:

https://github.com/WRFan/ubuntu/blob/main/boot/grub/grub.cfg

and that's what grub creates when you execute "update-grub" and what it uses on boot. So I am translating what people said to grub.cfg syntax. So I am telling google, "I wanna change my ratio! Ratio, you get it, you stupid search engine?!" Google replies, "all right, so you want to change your resolution? No problem!" Sigh. So people tell you to add (follow my "grub.cfg"):

	video=vga16fb:640x480-60

	video=uvesafb:mode_option=1024x768-8

	vga=normal

	video=vesafb:ywrap

	video=vesafb:off

	vesa:off

	vga:off

	video=VGA-1:1152x864M-32@75e

	vga=0x318

	vga=792

	nox2apic

	acpi=off

	noapic

	nolapic

	agp=off

	video=1024x768

Enough bullshit? Or do you want me to find more crap for you? 'cause the internet is full of this useless crap.

So what actually sets the resolution is:

	set gfxmode=1024x768x8

	terminal_output gfxterm

but that's actually not what I was asking. I asked about ratio, but all right, if everybody is talking about resolution, let's talk resolution. Where's 1152x864, please? Nothing? Linux is too stupid to support it? All right, we've talked about resolution and found out linux is pathetic. Can we go back to ratio? No? You still want to know how to check resolution? Well... don't remember. Forgotten. Wasn't it something like:

	hwinfo --framebuffer

	hwinfo --gfxcard

	lshw -c video

GRUB ONLY:

	set pager=1

	vbeinfo

People also mentioned tons of other programmes, but they are meant for the X11 version, because nobody understands the word "tty". Now back to my poor ratio! So this CLI version of ubuntu uses some basic video driver called "vesafb". Shoot me if I know what it is or how to control it. There are only a few pages and nothing what they say works. Now look, this page mentions ratio!

https://www.kernel.org/doc/html/latest/fb/modedb.html

Except what the hell is this page saying? How to pass aspect ratio to the kernel through grub? Grub, answer! Total silence. Grub's not talking. Grub is dead!

Ok, so I can live with 16:9. But what about those cut-off edges? Google? Look, Google found something!

https://unix.stackexchange.com/questions/407021

https://tldp.org/HOWTO/html_single/Framebuffer-HOWTO

Oh, wait, we have the usual problem: Ignorant people opening their mouths, yet only hot air coming out. So "overscan". Affects cheap TVs. Yet I don't have a TV connected to my computer, it's a Fujitsu SL3220W monitor? So five days and just bullshit on Google. So finally, I am thinking, there are buttons on my monitor. Do they have any purpose? So I am pushing them like mad, suddenly, the screen adjusts itself and the edges of the screen are normalised. Stays normalised on linux reboot. Of course, nobody mentioned anything like this (I read like a 1000 pages in 5 days!). What's the purpose of Google again?

Also, this is not a real solution, the actual problem is the pathetic nature of the linux driver. Like, for example, why isn't there 1152x864 support on ubuntu cli? It sure is supported by my monitor and Windows. My monitor has two options concerning ratio - enforce 16:9 or listen to software. If I enforce 16:9, it will ignore what Windows tells it, but if I set "listen to software", it will set 4:3, as specified in Windows. Now it's not my poor monitor's fault if linux is incapable of supporting a 60 year old ratio.

Now, since we've been talking about Grub, let's talk some more about it. So, when running on my virtual machine, it's not a problem, since there's just one OS and one harddisk. But now imagine a real computer, I have Win11, WinPE, plus I still had WinXP at that time, so BOOTMGR. So I want to run ubuntu from usb flash drive and I don't want to change my boot order in the bios every time I want to boot linux. So somehow need to add ubuntu to BootMGR.

Yeah, but wait... Not so fast. Let's talk about adding the MBR to the flash drive. Now, apart from people talking my ears off about that new MBR replacement GPT (which is a pathetic crap from my point of view, which causes problems all over), all I got from google is "use grub-install". Use "grub-install"? All right:

	mount /dev/sdb2 /work

Now executing the all cool grub-install!

	grub-install: warning: File system `ext2' doesn't support embedding.

	grub-install: error: embedding is not possible, but this is required for cross-disk install.

	/usr/sbin/grub-install: warning: Embedding is not possible. GRUB can only be installed in this setup by using blocklists. However, blocklists are UNRELIABLE and their use is discouraged.

	/usr/sbin/grub-install: error: will not proceed with blocklists.

"ext2"? Well, the partition has been formatted as ext4 (fdisk 83), but whatever you say, Grubby. Google, help!

Google:

	Force it.

All right, Googy, forcing:

	/usr/sbin/grub-install --force --boot-directory=/work/boot /dev/sdb2

Look, Grubby did it! Well, it still whined my ears off, but at least it wrote the damn MBR.

Now, how to add ubuntu flash drive to BootMGR? Google?

	Add a Real-mode Boot Sector

But Google, this is nonsense, the morons tell you to "dd" the bootsector from the flash drive and create a new BCD Store boot entry pointing to the file containing the boot sector. "Just put the file on c:", they say, "and point to it". Exactly how's the BOOTMGR supposed to know what drive this boot sector belongs to? It gotta know which drive it must boot when "ubuntu" is selected on the BootMGR menu!

	diskpart

	list disk

		Datenträger 2 Online 3824 MB

	select disk 2

	list partition

		Partition 1 Primär 1777 MB 1024 KB

		Partition 0 Primär 2046 MB 1778 MB

So hd 2 (0-based), 2nd partition (why the hell does diskpart list it as "Partition 0"?!). So the extracted mbr contains this info? hd2, partition 2? docs.microsoft.com , some help, please?

docs.microsoft.com :

	...

Total silence? All right, let's check the built-in bcdedit help. Checking... what these morons on the net say, doesn't make ANY sense. Google, output all pages dealing with adding Grub to BootMGR!

Google:

	There's no need.

Why not?

Google:

	My database contains 10.000 identical webpages providing the same info that you already received. It seems morose webmasters copy paste info from each other without actually considering what they are copying.

Google, why do humans do this?

Google:

	For this very reason: They are human.

So while messing around with Fedora and other bullshit I stumbled upon Syslinux. This one I kind of liked... first. Well, anything better than Grub, right? So Syslinux allows you to point to a specific hd/partition combination you want to boot. Syslinux passes control to my flash drive (just as if it were set first in the BIOS), Grub on the second partition (ext4) takes over. All right, let's do it:

	syslinux –f c: c:\ubuntu.bin

	bcdedit /create /d Ubuntu /application bootsector

	bcdedit /displayorder {XXX-XXX-XXX-XXX-XXX} /addlast

	bcdedit /set {XXX-XXX-XXX-XXX-XXX} device partition=c:

	bcdedit /set {XXX-XXX-XXX-XXX-XXX} path \ubuntu.bin

Wait, you stupid syslinux, you messed with the damn file attributes!

	attrib -r -h -s "c:\ldlinux.sys"

	attrib -r -h -s "c:\ldlinux.c32"

Copy to c:\:

	chain.c32

	libcom32.c32

	libutil.c32

	vesamenu.c32

Yeah, but why do I have to keep all that shit in the root directory? Can't I copy it to some other directory and point to it? Syslinux?

Syslinux:

	Of course! You just need to bla bla bla bla bla

Syslinux, it's not working! I did exactly what you said: "bla bla bla bla bla"! But I get a boot error!

Syslinux:

	bla bla bla bla bla! BLA!

Aaal... All right... I guess, I keep all those shit files in the root directory. So, the conf file:

https://github.com/WRFan/ubuntu/blob/main/syslinux/syslinux.cfg

I don't require a timeout (since BootMGR is already counting, so no timeout for Syslinux/Grub). So Syslinux:

	TOTALTIMEOUT 0

Grub:

	set timeout=0

Booted, Syslinux menu is just sitting there. Syslinux? I set the timeout to 0. Why's the menu there?

Syslinux:

	Bla! bla bla bla bla bla! BLAAAA!

Stackoverflow, superuser, anybody, HEEEEELP!

	set TOTALTIMEOUT to 1, otherwise "ONTIMEOUT" won't trigger.

But this is stupid! Well, still better than Grub. That's some consolation.

So all right, it booted. Two days later, "boot sector failure". Re-wrote MBR to usb drive, rebooted, working again. A week later, boot sector failure. And so on. No idea what causes this. Do I have to re-write the MBR every time I modify the ext4 partition of the usb drive? Or is it the changes to the partition syslinux resides on that causes this? Syslinux?

	BLAAAAAAAAAAAAAAAAA!

I get what I deserve.

Another word about that pathetic Grub: If you have an error in grub.cfg and grub doesn't boot, and you then hit reset, reboot to Windows, edit your grub.cfg (I do it by mounting the flash drive into my virtual linux in VMWare), reboot back to ubuntu, grub may not reflect your changes immediately, it sometimes caches a previous version (hell knows where) and loads that one, so you may need to reboot ubuntu several times before grub decides to actually re-parse "grub.cfg". Grub, what's up with this damn caching?

Grub:

	BLAAAAAAAAAAAAAAAAA! BLAAAAAAAAAAAAAAAAA!

Sigh. I think we're done here. Unless you think killing Grub is a good idea? Count me in!

Now let's talk some more about Ubuntu server edition wonderful networking support. It's a server edition, right? So here we've got a Mac running Apache + Webdav, and it's up to you, dear Ubuntu, to mount the webdav as a network drive. Ubuntu?

Ubuntu:

	No idea what you are talking about.

All right, let's look at the Ubuntu packages... Ah, here - davfs2. Coool. Let's install

	davfs2_1.5.5-1: Segmentation fault

Segment-what? Let's check what the developer has to say on this. Developer?

	Problem has been fixed in the newest version. compile davfs2 with -std=c89

Ubuntu, a compiled package of the newest version, please?

	I am waaay too lazy to compile anything. Who cares? Damn users, always demanding something from me. Incredibly annoying creatures! Let me sleep, I'm tired...

So I checked with Debian, and they are actually caring about their users. As I said above, they update their packages more frequently than the Ubuntu lazybones.

Now some words on hardware support in linux. I already spoke about the wonderful graphics support on Linux. Now let's talk about Realtek drivers. But first, recall the last decade of the 20th century. The dinosaurs were still walking the earth. Dos, Windows 3.1, then Windows 95! Win 98! Wonderful systems lacking hardware support for anything! No drivers! You need drivers? Push a floppy into the floppy drive! Install the driver! Enjoy a bluescreen! Then! Revelation! WinXP! System providing drivers for everything! No need for floppies! No need to download drivers! Everything provided by Microsoft! Then! Windows 10! Even better driver support! Well, my Adaptec SCSI controller not supported, but that's irrelevant! Who cares about harddisks? Maybe Microsoft hates Adaptec. Yet otherwise, everything's there.

Now imagine - going from Win10 to Linux! Linux! Realtec r8168 driver? Not provided. NOT PROVIDED? What the hell? 21st century out there, damn it! Download off Realtek. Realtek pages are constantly down, damn it! Get it from Github. Compile. Or get from Ubuntu. It still needs compilation. So compiling. Freezes. Constant disk access, harddisk indicator constantly flashing, yet nothing happens. What the hell is going on? Not even a cryptic error? Nothing? Turns out, not enough memory (was compiling on virtual machine, for compilation purposes, increase RAM to like 320 MB minumum). All right, working.

Now I got this wonderfull all-cheap Realtek network card from Amazon, r8169 chip. Ubuntu?

Ubuntu:

	Hahahahaha!

Not funny. Besides, your stupid r8168 install package attempts to disable the r8169 driver if found. Github version does it too. What's that good for? Where am I going to find a r8169 driver? Github?

Github:

	Hahahahaha!

Why is everybody laughing? The card does what it's supposed to do. It's supported on Win10 AND WinXP! Where's the damn linux driver? Well, finally got it off Realtek. 20 minutes to download a 100 kb file. What's up with this damn Taiwanese company?

So I finally got it working, but then ubuntu messed up the module load order, I wrote about it above.

Now let me point out some other stuff as it occurs to me. dmesg output is insufficient. I want to read everything that occurs during boot. That stuff is not written anywhere on Ubuntu. They will tell you to pause the boot sequence. Wonderful! By the time Ubuntu notices you hit the pause key, half the text is already gone. So you have to reboot all over, feverishly hitting the pause key, hoping the boot process will pause at the right moment. Is this some kind of "Hit the keyboard as fast as you can" contest?

Disabled apparmor in Grub. Checking:

	cat /sys/module/apparmor/parameters/enabled

Disabled.

Dmesg?

	AppArmor: AppArmor disabled by boot time parameter

/proc/cmdline ?

	BOOT_IMAGE=/boot/vmlinuz-4.4.0-142-generic root=UUID=XXX-XXX-XXX-XXX-XXX ro fbcon=scrollback:2048k consoleblank=0 net.ifnames=0 random.trust_cpu=on apparmor=0 noapm

All right, so I can "dpkg --purge" any AppArmor crap packages, right? Well, suffice to say, it's a good idea to try such stuff in a virtual machine. You do have a snapshot, do you? What's up with this security bullshit, anyway? Why is AppArmor still loaded, if it's disabled? Why can't I purge this crap?

HFS+ support on Ubuntu is increeedible! You have a feeling you are using a mac! Great support for this file system!

	unknown filesystem type 'hfsplus'

	This is not a HFS+ volume (Unknown error -1)

Isn't it cool? But look, ubuntu's provides utilities for HFS: hfsplus, hfsprogs. There's just a tiny problem. The actual kernel support is not there. But then again, Windows isn't faring better. Neither do any third-party programmes for Windows. They promise total support for mac systems, and all you get is a harddisk filled with crap. Fortunately, just a virtual harddisk. Snapshot loaded, puff! crap gone. But I still got no support for .dmg files, especially journalled stuff. All I hear is "transmac". But transmac is opening most files read-only. Ubuntu doesn't provide even such basic support. So there's dmg2img. Wonderful. So

	dmg2img -v -i /mnt/shared/Work/packs/058-74917-062.dmg -o /mnt/shared/Work/packs/058-74917-062.iso

Wait, not so wonderful. There's still HFS inside. no mount on linux. Well, maybe MS and linux are right. After all, who wants Apple crap attached to their asses?

What's up with that pathetic resolv package on Ubuntu? Everybody complains about it, nobody needs it, everybody is annoyed by it, yet it's still there? purge it as soon as possible, add manually:

	/etc/resolv.conf

Add your "nameserver"-s to it for DNS. One file needed, yet look at all the "resolv" bullshit that is going on:

	/usr/lib/resolvconf/

	/etc/resolvconf/

	run-parts: executing /etc/network/if-up.d/000resolvconf

	run-parts: executing /etc/network/if-down.d/resolvconf

	/sbin/resolvconf

	/run/resolvconf

	/etc/dhcp/dhclient-enter-hooks.d/resolvconf

	/lib/systemd/system/resolvconf.service

Symlink:

	/etc/resolv.conf -> ../run/resolvconf/resolv.conf

Stop service:

	systemctl disable resolvconf

	dangling symlink

Look, we can't resolve askubuntu.com any longer!

This is just a small list. Github doesn't have enough space to list all the resolv crap going on on your system. Again, get rid of it.

What's up with the damn time sync across all systems? I even got one on my SIP ATA! Anybody's fond of a system accessing internet by itself? A good system is a system that stays offline until the user actually requests a server!

	systemctl status systemd-timesyncd

	/lib/systemd/system/systemd-timesyncd.service; disabled; vendor preset: **enabled**

What's up with bash across all systems? Microsoft's cmd is very comfortable, you type something, make a spelling mistake, hit Escape, entire line deleted. Want to copy stuff to clipboard? Ctrl+a, then right mouse button click. Now you've heard about the problems by which VMWare Workstation is ridden. Putty connecting to the ssh server running on Ubuntu isn't faring any better. But that's putty's fault. However, the inability to delete the current line by hitting Escape is bash's fault. So people will tell you to add

	bind '"\e": kill-whole-line'

to:
	/root/.bashrc

but there's a delay between hitting Escape and the actual line removal!

While working with bash, I always type "cls" when switching from Windows to linux or darwin, then I switch back to Windows and type "clear" into cmd and powershell! That's it! Linux talks Micro-speak now (again, ".bashrc")!

	alias clear='clear && echo -en "\e[3J"'

	alias cls='clear'

Now some final words about the wonderful *in-built* network support of this **server** version. Now mind it, I said, I expect even a minimal installation to favour network programmes and to install them. **Server** edition, right? So I already mentioned davfs2.

radvdump? nada

apache2? nada

dnsmasq? nada

openssh-server? nada

make and gcc support to compile network adapter drivers? nada

snmp? nada

traceroute, iputils-ping, iproute2? either not there, or outdated, don't remember

ethtool? nada

geoiplookup? nada

dnstracer? nada

dnsutils? missing or outdated, don't remember

bridge-utils? nada. well, actually, this was a good decision. this programme is crap. use "ip link"

wget? nada

iptables? nada

ifupdown, ifup? Outdated

isc-dhcp-client? nada

ipv6calc? nada

radvd? nada

Now, I wish I had written down all the crap that I actually purged from the system. There were like 100 packages (minimal installation!), and NONE had anything to do with networking WHATSOEVER.