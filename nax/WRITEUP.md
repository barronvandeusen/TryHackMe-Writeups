# Abstract

Good morning and welcome to another machine! The description of this room/machine tells us there's a critical security flaw on this most powerful and trusted network monitoring software that allows an authenticated user to perform RCE. We know the shape of what we're going after, so let's get started. 

## Enumeration

As always, we'll begin with our nmap scan

> nmap -sC -sV -O -oN initial_scan.txt 10.10.181.219
![image](https://user-images.githubusercontent.com/6416242/166150845-07d0090d-9349-407d-8e25-056e1a4d2b18.png)

>Open ports:
22 - ssh
25 - smtp
80 - http
389 - ldap - OpenLDAP 2.2.x - 2.3.x
443 - https

Let's pull up the site and search around a bit. 
![image](https://user-images.githubusercontent.com/6416242/166150836-f35ef18f-f3b5-45d4-b24c-d21c900f94af.png)

Not a lot at first glance, although that list of elements might be some kind of hint or puzzle. We'll come back to that in a second. Viewing source shows this comment:

> <! --/nagiosxi/ -->

Nagios XI is the name of the software - we know a thing now! Very exciting update. At this point I'm guessing there's a login panel of some sort, so let's run gobuster on it.

> gobuster dir -w /usr/share/wordlists/dirb/common.txt -u 10.10.181.219

While that's running I want to come back to that list of elements. Might not be anything, might be something.

Ag - Hg - Ta - Sb - Po - Pd - Hg - Pt - Lr

Silver - Mercury - Tantalum - Antimony - Polonium - Palladium - Mercury - Platinum - Lawrencium

While that's running, dirbuster has come back with results. We have an index.html AND an index.php, plus a directory called /nagios as well as a /javascript. We don't have access to /javascript yet, so let's take a look at that index.php file.

We're brought to a login screen. Googling for default credentials gets us something, but trying to use them at the logins at /index.php and /nagios doesn't authenticate us. Let's go back to those elements. We know the element name, we know the annotation, so logically the next bit of information would be the element number. That gets us the following:

47 80 73 51 84 46 80 78 103

What does it mean? Let's ask our friend CyberChef! We plug those numbers in and get the result /PI3T.PNg which sounds like a hidden file and therefore the answer to question one! Navigating there brings up an image, which tells me we're in steganography land. Let's download the picture and get to work unraveling its secrets. I used binwalk and didn't get any useful information, then tried exiftool and got, among other things, the artist name, which is the answer to question 2.

## Art History

We're going to metagame a little bit here: we have the artist name, Piet Mondrian, and the next question, which talks about running into an error with the tool and exporting the image as a ppm file. Let's google "piet ppm" and see the results.

Lo and behold, piet is a programming language where the program is a picture. We have a picture. I like where this is going. There's a program called npiet that will decode and create piet program/pictures. Ran into some installation errors here, swore quite a bit, rage-nuked the download and directory. On the third time following the instructions I realized I didn't need the 'make' or 'make install' commands and running ./config had done it. I did, however, run into the expected PPM error, downloaded GIMP, and converted the image. We run npiet on our newly acquired .ppm and we get a repeating output containing the username and password!

User: nagiosadmin
Password: this_isnt_the_password

## Infiltration

With this in hand, we can log in at $IP/index.php. The first thing to take note of us the version number - 5.5.6

The next question wanted the CVE number this is vulnerable to. I cheated here, because none of the answers I returned are what tryhackme wanted. The answer is CVE-2019-15949. There you go. Saved you some frustrations and googles. You're welcome, feel free to buy me a beer next time you see me. 

Next step is to fire up metasploit. Let's do that and search for nagios XI 5 and we have a few solutions that apply to our version.
![image](https://user-images.githubusercontent.com/6416242/166150880-69270191-ab9a-4626-b6f0-84dbdaf56559.png)

I like the look of that first one. Tryhackme, however, says that's the wrong answer. So we play the metagame and plug in all the ones that COULD be it (anything with version prior to 5.6) until we find that tryhackme wants us to use exploit/linux/http/nagios_xi_plugins_check_plugin_authenticated_rce

Plug that in. There's your answer. That's two beers. Select the exploit, plug your options in, and...I'm in.
![image](https://user-images.githubusercontent.com/6416242/166150892-efcaa4e7-d470-44a8-954d-48d2819e51ca.png)

Not only am I in, I'm in as root, which means this machine is as good as solved. 

![image](https://user-images.githubusercontent.com/6416242/166150907-78830e4e-9eb8-426f-a863-d4513d80850d.png)

Check the usual spots, /home and /root. Get your flags. Wave 'em around. Celebrate a little, you just won this room.

## Final Thoughts

I may go back and try to root this box with the other CVE/metasploit results. If I do, I'll create an addendum to this writeup. In the meantime, congratulations, you've pwned another box AND learned about an incredibly influential Dutch artist. We're all about learning here. Go make some art and hack more stuff!
