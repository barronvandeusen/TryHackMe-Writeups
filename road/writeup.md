# Abstract

Happy Labor Day! Today we're tackling Road, a medium-rated tryhackme machine that, according to the blurb, is based on a real-world penetration test. No hints outside of that, so let's get to it. Shoutout to my friend and fellow hacker XAngryChairX who hacked this one with me.

## Enumeration

Starting with our nmap scan, we see that we only have two open ports, ssh and http:
![nmapscan](https://user-images.githubusercontent.com/6416242/188501266-eb5c36dc-bb70-4d74-912c-d84ee54daa22.PNG)

We know it's a website, so let's enumerate it a bit before we pay it a visit. We run a gobuster scan and see a few interesting directories:
![gobuster_scan1](https://user-images.githubusercontent.com/6416242/188502188-611ad06f-61c6-4c22-a49d-fd886ae4d21b.PNG)

I chased up on the PHPMyAdmin, hoping to find some default credentials. No such luck, but at least I know we that the site is leveraging php. The other interesting directory, v2, has an admin login page and an index.html page. For now, let's take a look at the site. I was able to register an account using a fake email, which gave me access to additional features.
![webpage1](https://user-images.githubusercontent.com/6416242/188502696-d7d67b0f-011c-4768-b7ea-5fba4a7bc801.PNG)

What caught my eye was the option to reset the password - "ResetUser" on the left side of the screen. A password reset feature might give us the option to change parameters and give us access to another account, but we'd need that account name. Fortunately, scrolling down to the bottom of the page, we see two things that look promising. One, a field to upload a profile picture, and two, a note with the admin email account. 

![admin_and_upload](https://user-images.githubusercontent.com/6416242/188515084-177ffcb6-5c83-4d49-88c1-5e50af68dfcf.PNG)

Let's see if we can become admin@sky.thm. I fired up BurpSuite, turned intercept on, and went through the password change process, swapping out my bogus email for admin@sky.thm

![change_email](https://user-images.githubusercontent.com/6416242/188515582-0c23c96b-c09b-4083-a893-0b02c513017e.PNG)

From here, I checked that picture upload option to see if I could upload, say, a .php file. Nothing seemed to stop it, but there was a catch - I couldn't find where the images were stored. I poked around gobuster a bit more to no avail, but turns out the answer was hiding in plain sight. Or at least, in plain page source.

![directory_in_comments](https://user-images.githubusercontent.com/6416242/188524136-6c1614b6-5edb-45bc-a70e-8e9b1cdfd8f4.PNG)

## Infiltration

That's our huckleberry. We upload a standard php reverse shell (this can be found in /usr/shell/webshells/php if you're running the new version of kali), fire up a netcat listener, navigate to the .php file we uploaded in the URL bar, and...

![got_shell](https://user-images.githubusercontent.com/6416242/188525026-37e59eaa-46e9-4e54-bef0-b27675a69621.PNG)

We're in. A quick whois show's we're www-data. A limited account, but the permissions are loose enough that we can find user.txt in /home/webdeveloper. Sudo -l tells us that we can't do anything without a password on this account, so the next logical option is to pivot. We know that there's a user called 'webdeveloper' on this machine due to the directory in /home, but let's see if there's another user. The /etc/passwd file shows us a useful surprise.

![hello_mongo](https://user-images.githubusercontent.com/6416242/188539018-405ab7dc-91c0-429c-b978-09213d264e23.PNG)

There's a mongo database running on this machine, and it looks like there's no login required. Sure enough, we can just jump in with the mongo command. Poking around gives us a backup database, and in this backup database is, lo and behold, webdeveloper's credentials! We can use those to ssh in as webdeveloper.

## Escalation

We see right off the bat that webdeveloper can run sky_utility_backup with elevated privilges, but the LD_PRELOAD stuck out at me - I don't recall seeing that before.

![sudo_dash_l](https://user-images.githubusercontent.com/6416242/188539500-cb8ce826-968a-48e6-8562-3de18a1a6914.PNG)

A quick google search led me to this excellent write-up by Raj Chandel: https://www.hackingarticles.in/linux-privilege-escalation-using-ld_preload/ 

This includes a bit of code we can run and tie to a command with sudo privileges. Follow Raj's advice, remember what webdeveloper can do, and we have root, and thus the root flag.

![iamroot](https://user-images.githubusercontent.com/6416242/188539865-c21729e1-e764-4334-8a90-bdc5b43fc9ba.PNG)

## Final Thoughts

The biggest takeaway I had from this box is to alter my enumeration methods. There are a couple of red herrings on this machine - the PHPMyAdmin page, an out-of-date web service - that sent me on deep dives. Enumerating broadly rather than deep will net you more possibilities. This box also reinforced an axiom of tech support that's carried into hacking: TRY THE SIMPLE STUFF FIRST. About a quarter of all help desk calls get resovled with "reboot it," "plug it in," or "turn it on." In this case, the web app exploits were as straightforward as can be - a quick request edit via BurpSuite, an upload form letting us put in whatever we please. This one was a thrill to work on, especially because I got to do so with my buddy. XAngryChairX, you're fantastic. 




