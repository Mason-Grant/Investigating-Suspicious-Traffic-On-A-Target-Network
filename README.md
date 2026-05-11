**1. Objective**

With this project, the aim is to capture and analyse the complete process of a web request. by observing a DNS lookup, TCP handshake, and finally HTTP GET request conversation, to demonstrate an understanding of the OSI model and the encapsulation of data across a network.


**2. Setup**

* Operating system: Parrot Security (running via Oracle Box on a Windows 11 machine)
  * I chose Parrot for this scenario as it comes pre-packaged with a litany of software that can be used(like Wireshark). Furthermore, Linux-based operating systems are the standard in the working world, making this scenario a relevant practice.
* Tools: Wireshark GUI and terminal CLI
  * I chose Wireshark for this task (specifically the version using a GUI) as it allows me to clearly document my findings, and there is no better tool when it comes to sniffing packets, with a whole host of additional information provided, allowing us to easily pivot from simple reconnaissance to network hardening. 
* Network config: bridged adapter with promiscuous mode enabled
  * This is a vital network configuration, as without it, our VM isn't properly integrated into my local network. The bridged adapter means that the Parrot VM shares the same network qualities as my Windows machine, allowing us to actually connect to the local network and sniff packets using Wireshark. This is because of my lack of a static IP address so the IP we use is the generic 192.168.1.1 and unless we are on the same network as my router, we won't get anything on Wireshark when accessing the target website on my Windows machine.
     * NOTE In a typical setting setting it would make much more sense to simply use the Parrot OS for the entire process instead of using the Windows machine. However, I wanted to make certain that this network configuration was working, and that my VM could properly sniff packets from other devices on my network. From my perspective, this exercise is incredibly low risk as I can monitor exactly who is connected to my router, and as you can see below, I have taken a multitude of measures to keep my Windows machine safe.
* Target: HTTP://neverssl.com (chosen as the site is unencrypted for good, clear-text analysis)
  * This website has been chosen as a target for this scenario as it is widely known as a website that is 'safe' despite not utilizing HTTPS. This is important, as while I might be using a VM for the packet sniffing(a VM that I can deprovision whenever and re-provision each time I boot the VM meaning it is safe from malware), I am accessing the website from my Windows machine. Neverssl.com is frequently used for similar exercises like the one we are using today, and therefore helps mitigate the risks commonly associated with accessing HTTP-only websites.
* Precautionary Measures: Windows Defender, backups, Windows Photo Editor(Redaction)
  * Windows Defender is integral here, as when accessing websites that do not leverage HTTPS, there is always a risk that a threat actor could access this clear-text information and potentially try to hack me in some way. Therefore, a good anti-virus is vital, as Windows Defender will make me aware of any unusual activity on my system(such as new admin accounts being made) or any malicious code being deployed on my system, helping me deploy effective countermeasures if a breach were to occur.
  * Backups are also integral. Allowing me to revert my machine to an earlier state, before the breach occurred. For example, if, while accessing the site, a hacker used the clear-text DNS to enact a DNS poisoning attack, in order to reroute my network traffic with faulty DNS replies, it could result in me being directed to malicious sites, causing my Windows machine to become infected with malware. With a backup once I am notified of this by Windows Defender, I have the option to completely roll back the system, keeping my computer safe from further infections as a result.
  * Throughout this scenario, I am gathering photographic evidence in the form of screenshots to help support the content laid out in this report. Within some of the raw screenshots was sensitive personal information that has been redacted to prevent a breach from occurring due to poorly managed information. 


**3. Process**
* ***Clear cache*** I cleared cache to help prevent my browser from disrupting the capture of a clean DNS query. As Parrot is Debian-based, DNS is not cached, and I have not installed a daemon for this so just clearing cache within the browser is sufficient.
* ***Wireshark setup*** Within the parrot CLI, I ran 'Sudo Wireshark' to start wireshark. Then I selected eth0 from the list of options as this is my primary interface. I applied a filter to the captured packets of 'HTTP || DNS' ,to Wireshark, to remove unwanted traffic from the list.
  * > This is vital as the moment you start running wireshark without a filter, the feed is filled almost instantly with 'noise' like arp requests, so in order to focus on the target traffic we need to apply this filter to work efficiently.
* ***Generation of target traffic*** using a browser, I navigated to HTTP://neverssl.com in order to generate a DNS lookup and HTTP get request. I had to use the 'nslookup' command as my browser hid the DNS lookup from wireshark.
* ***Analysis*** I discovered the DNS query(UDP port 53). Additionally, using the "Follow TCP stream" function I was able to locate the TCP handshake (SYN,SYN-ACK,ACK) and the HTTP get request conversation in clear-text.


**4. Findings**
* ***DNS Resolution*** I successfully captured the standard query for neverssl.com and the response, which linked the domain to the IPv6 address:" 2600:1f13:37c:1400:ba21:7165:5fc7:736e". Due to my Internet Service Provider, IPV4 addresses cannot be monitored using wireshark as all IPV4 requests go through my ISP and then my router, and so have a standard IP i.e "192.168.1.1". Thankfully, we can circumvent this issue with the use of IPV6 addresses. Using 'Nslookup' in Parrot, we can query the website and receive the correct IPv4/IPv6 address.
   * This Dns query was captured in clear-text highlighting a massive security risk as a hacker can sniff this packet and would instantly see what website I am trying to access. Additionally, my IPv6 address would also be leaked meaning the hacker would then have the means to begin attacking my network via my IPv6 address being leaked. For a business this would pose a massive risk as one exposed Ip addressed could lead to an attacker breaching the network then moving laterally.
* ***Transport*** Using Wireshark, I confirmed that a successful TCP handshake was occurring. I observed the correct incrementation of the sequence and acknowledgement numbers, proving a stable connection was present.
  * However, a major flaw to bring light to here is that there is a distinct lack of TLS(Transport Layer Security). As, after seeing the SYN, SYN-ACK, SYN TCP handshake, we should then expect to see the encrypted data of a TLS exchange.
    * These packets lose their confidentiality as there is no encryption preventing anyone who sniffs the packets can see the entire data stream with no real enxryption besides some simple hashing at best which can be easily deciphered with an online tool.
    * Futhermore, in lacking proper TLS, there is a complete loss of intergrity for this data. TLS provides a hash that can be used by the recipient system to asses the integity of any data received and allows the recipient to delineate whether or not the data has been tampered with by a threat actor.
       * > This means an attacker can intercept and edit data in a MITM (Man-In-The-Middle) attack. Wherein, the attacker can alter the packets in order to further their malicious objectives i.e injecting malicious links into the packet. As there is no TLS present there is nothing in place to prevent these types of MITM attacks from occuring.
* ***Transparency*** Due to the site utilizing HTTP(port 80) as opposed to the more secure HTTPS(Port 443) I was able to read all the raw data in clear-text, including HTTP headers, cookies and the server's 200 OK response.
  * Failing to utilize HTTPS is a massive security risk and loss of confidentiality, all data sent via port 80 is unencrypted it comes through as clear text. This means that if a threat actor was monitoring the network, they could sniff the HTTP request which would essentially be like reading an open book on the victim, Usernames, passwords and cookie session ID's would all be there in clear text. Its a major issue for a company as people often use the same passwords for different services, habitually. Consequently, if a business network was sniffed and the threat actor was to learn that an employee had DNS queries for a website that was not using HTTPS, that threat actor would easily be able to use software to create a custom word list based on this employees used passwords. Potentially allowing the threat actor to begin an attack on the business through the employee's account, then move laterally through the rest of the business. This highlights the massive risk that even accessing a website that only using HTTP can pose.

**5. Explanation**

* ***Outline of Security Posture***
   * This investigation has identified      that the target network is conducting operations whilst harboring multiple gaps in security that compromise the confidentiality and integrity of the data of users using this network. through a lack of cryptographic techniques we are able to read critical information, like session cookies and user data, in clear-text. which means a threat actor could also see this data.


* ***Impact Analysis***
   * The potential impacts caused by these security gaps are severe. As DNS queries are not encrypted any bad actor can perform passive reconnaissance in order to build  a profile of the users habits I.e the websites they frequent, in order to target a vulnerable website the user might frequent. Moreover, as there is no proper encryption any credentials used on this website can be easily deciphered. as people tend to reuse passwords (even for their company work accounts) it could lead to a credential stuffing attack against a company's infrastructure due to the leaked credentials. In turn, this could lead to a bad actor moving laterally and potentially compromising the company's entire system. whether they accessed the HTTP website on the company network or not.

* ***Recommended Remediations***

This project is substantial as it mirrors real-world network troubleshooting. Understanding the TCP handshake is vital to troubleshooting errors with unstable conections or blockages caused by firewalls.Moreover, seeing the stark contrast between unencrypted and encrypted traffic highlights the absolute importance of TLS/SSL in modern cybersecurity. This project demonstrated understanding of layers 3,4 and 7 of the OSI model, which is a critical foundation of networking.


**6. Screenshots/Evidence**

<p align="center">
  <img src="Trafficlist.png" alt="Trafficlist" width="100%">
</p>

The wireshark traffic list showing the green HTTP packets and blue DNS packets (we are focusing on 3763,4435 and 4443)


<p align="center">
  <img src="HTTPpayload.png" alt="HTTPpayload width="100%">
</p>

The "Follow TCP stream" of the HTTP get request where we can see some details about the firmware of the server hosting the website due to the fact HTTP is not encrypted and is clear-text


<p align="Center"?
  <img src="DNSquery.png" alt="Screenshot showing the DNS query in wireshark" width="100%">
</p>

A screenshot from wirehark showing an enhanced view of a captured DNS query packet. Here we can see DNS using UDP port 53 and the URL of the target website "neverssl.com"

<p align="center">
  <img src="IPV6.png" alt="IPv6 address of target" width="100%">
</p>

The IPv6 address of the target shown here. Due to my ISP IPv4 adresses are not used in the conventional way and are routed straight to my router as generic addresses i.e 192.168.1.1

<p align="Center">
  <img src="TCPhandshake.png" alt="the three way TCP handshake" width="100%">
</p>

Here we can clearly see the TCP handshake occuring syn, syn-ack and ack. ensuring a stable connection is being made. we can also see FIN,ACK packets showing that the data transfer finished and ended succinctly.

<p align="center">
  <img src="nslookup.png" alt="nslookup of target in parrot" width="800">
</p>

in order to ensure the correct packet was being analysed,by double checking the IPv6 address between wireshark and the CLI output, and to force DNS query packets to be detected by wireshark, due to the browser hiding this data from wireshark, nslookup was used on the target URL. here is the terminal output from an non-authoratative DNS server.




