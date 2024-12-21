# Gallery CTF

## Enumeration
First things I ran nmap to see what ports we are working with and got back:
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
So this leaves me with only web enumeration to perform. Visiting on port 8080 I encounter a login panel with the name
of the CMS above it ; Simple Image Gallery CMS, I performed basic SQLi tests and the payload admin' OR '1'='1'# worked
and I was able to log into the admin account.

## Getting a shell
While logged in as admin, getting a shell was relatively easy, I saw that I can upload pictures so I tested if there
is some kind of extension check or file type check present but there were none. I simply uploaded a php reverse shell
and when visited on the site it would connect to my listener and I had a reverse shell into the machine as the www-data user.

## Privilege Escalation
So I really like linpeas when doing CTF's sometimes it feels like cheating but it saves a lot of time when enumerating inside the box.
In this case it helped me find a real user's credentials written in plain text under the /var directory where it saved a backup of the
user's directory along with the .bash_history file where the password was written.

## Getting the root user
As the user mike I noticed that I can run a rootkit.sh script, which when ran gave us a few options and "read" was one of them which opened
nano text editor to read a file. Looking for nano on GTFObins I found that we can escape the nano shell and stay the root user using the
command: reset; sh 1>&0 2>&0. So I run sudo -u root /opt/rootkit.sh, select "read" and then Control+R Control+X and input the command
from GTFObins.
There we go root user able to read root.txt and have root level permissions.
