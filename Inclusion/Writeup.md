# Inclusion
## Abstract
Local file inclusion is a vulnerability caused by lack of user input validation that allows an attacker to include a file in a request to a website. This can most commonly be exploited via directory traversal. On this machine, a port scan shows us port 22 is open, allowing for ssh login. Through a bit of directory traversal, we find both a username and password. We're able to log in with these credentials and can gain root access through a binary with the SUID bit enabled.

