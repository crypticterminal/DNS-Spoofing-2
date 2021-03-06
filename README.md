Name: Rohan Vaish
SBU ID: 111447435
Network Security CSE 508
Homework 4: DNS Packet Injection

DNS Inject
----------------------------------------------------------

Format?
	dnsinject [-i interface] [-h hostnames] expression

	-i  Listen on network device <interface> (e.g., eth0). If not specified,
	    dnsinject should select a default interface to listen on. The same
	    interface should be used for packet injection.

	-h  Read a list of IP address and hostname pairs specifying the hostnames to
	    be hijacked. If '-h' is not specified, dnsinject should forge replies for
	    all observed requests with the local machine's IP address as an answer.
	    
	<expression> is a BPF filter that specifies a subset of the traffic to be
	monitored. This option is useful for targeting a single or a set of particular
	victims.

	The <hostnames> file contains one IP and hostname pair per line,
	separated by whitespace, in the following format:
	1.1.1.1      foo.example.com
	2.2.2.2      bar.example.com
	3.3.3.3  www.cs.stonybrook.edu
	8.8.8.8 www.facebook.com
	7.7.7.7 www.google.com

Testing Environment?
	Ubuntu 16.04.3 LTS
	Linux 4.10.0-35-generic x86_64

Language?
	Python 2.7.12 (default, Nov 20 2017, 18:23:56) 
	[GCC 5.4.0 20160609] on linux2



How to compile/run?
	sudo python dnsinject.py [-i interface] [-h hostnames] expression

Working examples?
	1. sudo python dnsinject.py --> This is spoof every DNS packet going from the default interface of the system with the attacker's local IP
		nslookup example.com

		Output->

		BPF Filter: udp port 53 
		Sniffing on default interface
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "172.25.80.1" 
		No file entered, using attacker's ip
		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "172.25.80.1" 
		No file entered, using attacker's ip
		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "172.25.80.1"
		This is spoof every DNS packet going from the default interface of the system with the attacker's local IP

	2. sudo python dnsinject.py -i wlp2s0 --> This is spoof every DNS packet going from the wlp2s0 interface of the system with the attacker's local IP
		nslookup google.

		Output->

		BPF Filter: udp port 53 
		Sniffing on wlp2s0
		No file entered, using attacker's ip
		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "172.25.80.1" 
		No file entered, using attacker's ip
		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "172.25.80.1" 
		No file entered, using attacker's ip
		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "172.25.80.1" 
		No file entered, using attacker's ip


	3. sudo python dnsinject.py -i wlp2s0 -h hostname --> This is spoof every DNS packet according to hostnames in hostname file going from the wlp2s0 interface of the system with the corresponding IP address in the file
		nslookup facebook.com

		Output->

		Hostnames to be spoofed are presend in file: hostname
		BPF Filter: udp port 53 
		Sniffing on wlp2s0

		Found! Spoofing using the file entry of www.facebook.com

		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "8.8.8.8" 

		Found! Spoofing using the file entry of www.facebook.com

		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "8.8.8.8" 



		
	4. sudo python dnsinject.py -i wlp2s0 -h hostname ip --> This is spoof every UDP filtered DNS packet according to hostnames in hostname file going from the wlp2s0 interface of the system with the corresponding IP address in the file
		nslookup google.com

		Output->

		Found! Spoofing using the file entry of www.google.com
		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "7.7.7.7" 

		Found! Spoofing using the file entry of www.google.com
		.
		Sent 1 packets.
		Packet Details:IP / UDP / DNS Ans "7.7.7.7"

Design?
	1. If the user does not enter any interface and file, each DNS reply is spoofed with attacker's local IP address
	2. If the user enters interface, each DNS reply on that interface is spoofed with attacker's local IP address
	3. If the user enters hostname file, each DNS reply on that interface is spoofed with the corresponding spoofed IP in the file


DNS Detect
----------------------------------------------------------

Format?

	dnsdetect [-i interface] [-r tracefile] expression

	-i  Listen on network device <interface> (e.g., eth0). If not specified,
	    the program should select a default interface to listen on.

	-r  Read packets from <tracefile> (tcpdump format). Useful for detecting
	    DNS poisoning attacks in existing network traces.

	<expression> is a BPF filter that specifies a subset of the traffic to be
	monitored.

Testing Environment?
	Ubuntu 16.04.3 LTS
	Linux 4.10.0-35-generic x86_64

Language?
	Python 2.7.12 (default, Nov 20 2017, 18:23:56) 
	[GCC 5.4.0 20160609] on linux2

How to compile/run?
	sudo python dnsdetect [-i interface] [-r tracefile] expression

Working examples?
	1. sudo python dnsdetect.py --> This is detect every DNS reply spoofing on the default interface
		nslookup facebook.com

		Output->

		Sniffing from default interface: wlp2s0
		2017-12-11 12:43:20 DNS poisoning attempt
		TXID 0xd3c2 Request facebook.com
		Answer1 [31.13.71.36]
		Answer2 [8.8.8.8]

		2017-12-11 12:43:20 DNS poisoning attempt
		TXID 0xd3c2 Request facebook.com
		Answer1 [31.13.71.36]
		Answer2 [8.8.8.8]

		2017-12-11 12:43:20 DNS poisoning attempt
		TXID 0xd3c2 Request facebook.com
		Answer1 [31.13.71.36]
		Answer2 [8.8.8.8]

		2017-12-11 12:43:20 DNS poisoning attempt
		TXID 0xd3c2 Request facebook.com
		Answer1 [31.13.71.36]
		Answer2 [8.8.8.8]

	2. sudo python dnsdetect.py -r hw4_1.pcap --> This is detect every DNS reply spoofing from the tracefile

		Output->

		Reading from tracefile: hw4_1.pcap
		reading from file hw4_1.pcap, link-type EN10MB (Ethernet)
		2017-12-10 00:26:53 DNS poisoning attempt
		TXID 0x73a7 Request www.iitb.ac.in
		Answer1 [172.25.82.240]
		Answer2 [103.21.127.114]

		2017-12-10 00:26:53 DNS poisoning attempt
		TXID 0x73a7 Request www.iitb.ac.in
		Answer1 [172.25.82.240]
		Answer2 [103.21.127.114]

		2017-12-10 00:26:55 DNS poisoning attempt
		TXID 0xa22f Request twitter.com
		Answer1 [172.25.82.240]
		Answer2 [104.244.42.193]


Detection Strategy/ Design?
	1. Reading from the interface/tracefile and accumulating all the packets read in a deque of size 10
	2. If the latest sniffed packet matches all the headers from any of the last 10 read packets except the DNSRR rdata field, then it's a spoofed response and  is printed as "DNS poisoning attempt"

Taking care of the false positive?
	1. By maintaining a deque of size 10, I ensure that the difference in the the original DNS reply and the spoofed DNS reply can not be more than of 10 	packets. So, if they differ by more than 10 packets, they are assumed to be replies from 2 different DNS requests which is genuine.
	2. Hence, the false positive are minimal with the implementation of deque of size 10
	3. However, there might be some false positive incase many DNS requests are made quickly, as the replies to both can be present in the deque which 		might flag them in DNS detect

Citation?
	1. https://stackoverflow.com/questions/12501780/dnsrr-iteration
	2. https://stackoverflow.com/questions/30698521/python-netifaces-how-to-get-currently-used-network-interface
	3. https://www.nginx.com/resources/glossary/dns-load-balancing/
	4. https://linuxexplore.com/2012/06/07/use-tcpdump-to-capture-in-a-pcap-file-wireshark-dump/
