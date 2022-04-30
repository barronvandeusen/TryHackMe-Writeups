Target IP = 10.10.47.244


Let's start with the nmap scan

Open ports are 22, 80,139, 145. We've got Samba!
Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-04-19T23:05:12
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: techsupport
|   NetBIOS computer name: TECHSUPPORT\x00
|   Domain name: \x00
|   FQDN: techsupport
|_  System time: 2022-04-20T04:35:12+05:30
|_clock-skew: mean: -1h49m58s, deviation: 3h10m30s, median: 0s

Let's enumerate some shares: smbmap -H 10.10.47.244
Results: a read-only share called websvr

smbclient \\\\C:\\websvr -I 10.10.47.244

We find a file, enter.txt

GOALS
=====
1)Make fake popup and host it online on Digital Ocean server
2)Fix subrion site, /subrion doesn't work, edit from panel
3)Edit wordpress website

IMP
===
Subrion creds
|->admin:7sKvntXdPEJaxazce9PXi24zaFrLiKWCk [cooked with magical formula]
Wordpress creds
|->

Okay, let's sort that password out first. "Magical formula" means we're heading over to CyberChef to unmagic it.
This gives us our password for Suberion. Now we just have to find where to leverage it.

Okay, let's google around about Subrion. The default install is under $Machine_IP/subrion/admin. Try that and we get redirected to:

https://10.0.2.15/subrion/subrion/admin/

Where the original IP is 10.10.47.244

No joy there, but on Subrion's github page, there's a robots.txt file. 
One of the disallowed entries is /panel, which sounds like what the note might be referencing. 
Checking the url $IP/subrion/panel gives us a login panel! We enter the credentials we found earlier and we're in!
