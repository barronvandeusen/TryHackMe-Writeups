## ABSTRACT

GhostCat is a tryhackme machine with an easy rating revolving around the exploitation of CVE-2020-1938. The approach to this machine was fairly straightforward - beginning with a port scan, we determined the Apache JServ protocol was running. A search of exploitdb (and searchsploit) yielded a metasploit module. Running this revealed plaintext credentials which we used to establish a foothold in the machine. Root was obtained by copying .asc and .pgp keys to local machine, running the .pgp file through gpg2john, cracking the ensuing hash with john, and using the obtained password to decrypt the pgp file. This revealed credentials that granted us access to a secondary account. From there, sudo privileges were leveraged to gain a root shell and obtaining root.txt

## ENUMERATION:

The first thing to do here is a trick I picked up from John Hammond - set the target IP address to an environment variable $IP. This saves an inordinate amount of copy/pasting and typing. 

export IP=[Target IP Address}

From there we can begin enumeration. To start, we'll run an nmap scap with my standard parameters:

sudo nmap -sC -sV -O -oN initial_scan.txt $IP

I will also run a second scan on all ports:

nmap -p- $IP

The first scan results indicate the machine is running Apache Tomcat
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/9.0.30

Armed with this information, I trotted over to exploitdb and ran a search for Apache 9. The first among the listed results that caught my eye was "Apache Tomcat - AJP 'Ghostcat' File Read/Inclusion (Metasploit)". One, AJP matched with the nmap results I'd already seen. Two, it was a metasploit module, and I'm not above taking the easy road if it gets me to the same place faster. I've since checked some other walkthroughs and there's a single line of python that will accomplish the same as what I"m fixing to do.

## EXPLOITATION

Let's fire up metasploit and find our module:

   0  auxiliary/admin/http/tomcat_ghostcat    2020-02-20       normal     No     Ghostcat
   1  exploit/linux/http/netgear_unauth_exec  2016-02-25       excellent  Yes    Netgear Devices Unauthenticated Remote Command Execution
   
Nothing too fancy to do here. Select the ghostcat module, set the required options (in this case we only need RHOSTS) and run the command. This yields a set of what look like credentials. We try to ssh into our machine with said credentials and we have success! We're in as user skyfuck. Let's take a peek around.

![image](https://user-images.githubusercontent.com/6416242/135782454-38268c57-675c-491f-ac0b-5cd59044aef3.png)

There's a little bit we can glean here. The first is that our user is holding on to a .pgp and a .asc file in their home directory. The second is that there's another user, merlin. The third is our first flag **user.txt** which we can still read as skyfuck! That's one down.

## ESCALATION

The first trick (realistically one of the only tricks I know so far) is to check sudo privileges with *sudo -l*. Our results?

>Sorry, user skyfuck may not run sudo on ubuntu.

Fantastic, no use there. Since there's another user on here, odds are good we'll have to pull some magic and turn into Merlin. Remember those files we saw in skyfuck's home directory? Let's scp those into our local machine and see what we can do with them.

I tried to cat out credential.pgp and received, to no one's surprise, an encrypted mess of noise. Catting out tryhackme.asc, though, showed a pgp key! Let's invite our good friend John the Ripper to the party by running gpg2john on tryhackme.asc, outputting that to a file, and then cracking our john-generated hash with john.

![image](https://user-images.githubusercontent.com/6416242/135783957-fb8fea9c-35aa-4137-9579-41aab7bc9bda.png)

Now that we've got our password, let's head back to the target machine. Add the .asc file to the keyring then decrypt.

> gpg --allow-secret-key-import --import tryhackme.asc 
> gpg --decrypt credential.pgp

And we have our Merlin credentials. It's a mighty password, so we'd have been there forever with just a brute force attack. Switch users to become merlin, plug in our new password, and feel the magic! 

Another check of our sudo rights now shows merlin can run **zip** as root. From here, we can head to GTFOBins, search for zip, and plug in the couple lines of code that nab us a root session. 

![image](https://user-images.githubusercontent.com/6416242/135784363-5b725118-0eaf-4f30-9ef9-7411ef0c42fe.png)

That's root.txt and that's the machine defeated!

