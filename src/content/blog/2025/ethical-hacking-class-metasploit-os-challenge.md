---
title: "Ethical Hacking class -  Metasploit OS challenge."
description: ""
pubDate: "Mar 18 2025"
categories: ["Uncategorized"]
tags: ["hacking", "john", "john the ripper", "kali", "password cracking", "Ubuntu"]
heroImage: "/images/posts/metasploit.jpg"
---
A combination of the final steps of my college class, a new job and a Pwnagotchi experiment run wild - I have been quite absent. The class I am taking threw us into CTF groups with 10 different challenges to accomplish. I took on Challenge 9:

Use the Category\_09\_Scanning\_Exploitation-Challenge09-Group1-6VM for this question. The VM is set to the host-only network set to 192.168.1.200. The root password is not given, and the default password has been changed. You will need to configure another VM like Kali on the host-only network to scan it. A good scan will get you the root password. After that, you can break into the system and then get the hash for elmo. The password for cookiemonster is the flag.

Seems simple enough right. Well, the configuration of Virtual Box and the provided OVA file was annoying to say the least. I forfeited on the host-only mode and set Virtual Box to "Bridged Adapter" - I wanted to attack the mini-PC from other devices on my network, vice locally within the OS. I did configure my network to secure this preference.

I started off with an NMAP scan of the target: 192.168.1.200

```
sudo -A -sV -O 192.168.1.200 > metasploit.txt

See nmap.org for modifiers and usage.
```

```
Nmap scan report for 192.168.1.200
Host is up (0.0082s latency).
Not shown: 980 closed tcp ports (reset)
PORT     STATE    SERVICE     VERSION
21/tcp   open     ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.2.242
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              39 Oct 21  2020 readthis.txt
22/tcp   open     ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|_  XXXXXXXXX (DSA)
23/tcp   open     telnet?
25/tcp   open     smtp        Postfix smtpd
|_smtp-commands: metasploitable.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
53/tcp   open     domain      ISC BIND 9.4.2
| dns-nsid: 
|_  bind.version: 9.4.2
80/tcp   open     http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
|_http-title: Metasploitable2 - Linux
111/tcp  open     rpcbind
139/tcp  open     netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open     netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
512/tcp  open     exec        netkit-rsh rexecd
513/tcp  open     login?
514/tcp  open     shell       Netkit rshd
1099/tcp open     java-rmi    GNU Classpath grmiregistry
1524/tcp open     bindshell   Metasploitable root shell
2049/tcp open     nfs         2-4 (RPC #100003)
3306/tcp open     mysql?
5432/tcp filtered postgresql
6667/tcp open     irc         UnrealIRCd
8009/tcp open     ajp13       Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8180/tcp open     http        Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/5.5
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 2 hops
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-02-14T01:32:58-05:00
|_clock-skew: mean: -8d18h40m32s, deviation: 3h32m08s, median: -8d21h10m33s
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

TRACEROUTE (using port 110/tcp)
HOP RTT      ADDRESS
1   0.33 ms  192.168.2.1
2   15.78 ms 192.168.1.200

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1293.50 seconds
```

Quite fruitful! I see a lot of opportunities were built into this Metasploit OS. I am going to focus on the FTP protocol since it has revealed a file with a potential hint: readthis.txt.

* * *

## Grabbing the readthis.txt file via FTP protocol

According to our NMAP scan the FTP service allows for "Anonymous" logins. After a quick search and a hint of information from [stackoverflow.com](https://stackoverflow.com/questions/3936911/how-can-i-login-anonymously-with-ftp-usr-bin-ftp) - it seems we can simply use the following credentials for access:

```
Name: anonymous
Password: anonymous@domain.com
```

After a successful login. I use very simple commands to grab the readthis.txt file.  

ftp > **get readthis.txt**

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/02/Screenshot-2025-02-22-at-7.14.38 PM-1-1024x526.png)

Upon display of the contents of the file I found the file contains login information. This has potential to be the first flag.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/02/Screenshot-2025-02-22-at-7.16.01 PM-1024x85.png)

Opening up the Metasploit VM of Ubuntu we use the login information above which allowed us to gain root access to the  Ubuntu OS. 

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/03/image.png)

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/03/image-1.png)

Now we need to look for the hash information on the other user accounts. To find this we must explore two files in the /**etc** folder:

```
passwd
shadow
```

Since John the Ripper likes certain [formatting](https://www.mohammedalani.com/tutorials/cracking-a-shadow-password-using-john-the-ripper/) in the password.txt file - we have to use the unshadow linux command to combine the "shadow" and "passwd" files.

```
unshadow passwd shadow > unshadow.txt
```

The output of unshadow.txt below reveals our second flag - the hash for elmo.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/03/image-2-1024x284.png)

_**unshadow.txt file output**_

Next we move our unshadow.txt file over to our Kali VM and run it against the "[rockyou.txt](https://github.com/dw0rsec/rockyou.txt)" wordlist in John the Ripper aka "John." As you can see it grabbed passwords for most of the hashes we fed it below.

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/02/Screenshot-2025-02-13-at-9.48.26 PM-1024x518.png)

**_results of John running on our unshadow.txt file_**

If you observe the output from above versus the unshadow.txt file you will notice John did not spit out a password for our cookiemonster hash:

```
cookiemonster:$1$Lq4lHJ0b$evOHD.itBRq2EupoAw.j..:1005:1005:,,,:/home/cookiemonster:/bin/bash
```

This of course was not initially clear to me why - but I was certain it was not in the "rockyou.txt" wordlist. So I pulled the string out of the "unshadow.txt" file and ran it separately in a file called "cookiemonster.txt" with an "incremental" modifier for a brute force attack. John took roughly six minutes on my M1 Macbook to find the password:

![](https://wbb.afh.mybluehost.me/wp-content/uploads/2025/03/image-3.png)

```
cookiemonster:eye:1005:1005:,,,:/home/cookiemonster:/bin/bash
```

You can see in the string, the password is **eye**.

I chose to run John locally to speed up the process. See [Homebrew](https://brew.sh/) for information on installing packages in MacOS if interested.

If this password was more complex it could have taken much longer. See this example of some serious hardware for password cracking on [#\_shellntel Blog](https://blog.shellntel.com/).

So as a byproduct of an incorrectly configured FTP protocol and the very fortunate find of the readthis.txt file under "/home/FTP" we were able to login, grab the passwd and shadow files, then utilize John the Ripper to find the flag. Granted this is likely never to happen on a moderately configured system. But still a fun challenge to test beginner Black Hat skills.

**Flags found  
Root password: CMIT321  
\# Hash for elmo: $1$M7qExhYD$r0/n6WTwbZDoFBCT5.30o  
Password for cookiemonster: eye**
