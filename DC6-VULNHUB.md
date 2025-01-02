# DC: 6
## Recon

### Nmap Scan
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:52:ce:ce:01:b6:94:eb:7b:03:7d:be:08:7f:5f:fd (RSA)
|   256 3c:83:65:71:dd:73:d7:23:f8:83:0d:e3:46:bc:b5:6f (ECDSA)
|_  256 41:89:9e:85:ae:30:5b:e0:8f:a4:68:71:06:b4:15:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Did not follow redirect to http://wordy/
|_http-server-header: Apache/2.4.25 (Debian)
MAC Address: 08:00:27:3A:8C:A8 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
From the network scan I can see two ports opened: SSH port 22 and HTTP port 80.

### Web Recon
Visiting the target on port 80 we are unable to connect since it's pointing us to a domain called wordy which was not known. So by editing the /etc/hosts file I got access to http://wordy/. The target is a Wordpress site so I ran wpscan to enumerate users and plugins. This got me a list of users on the site and also the clue which pointed me to a filtered rockyou.txt file from the creator of the machine, this everything pointed me to a brute force attack on wp-login.php. I performed the attack using wpscan and got back credentials for the user mark.


## Exploitation

### Web Exploit
As the user mark I could not upload plugins or change themes or anything, but I noticed I had access to a plugin called Activity Monitor. Searching on searchsploit I found an Authenticated Remote Code Execution vulnerability for this plugin which I could use to get a reverse shell into the machine. The exploit from searchsploit gives me a limited shell so I used netcat to get myself a stable reverse shell in the machine.

### Privilege Escalation
In the machine I could read wp-config.php file which contains database credentials for that machine. In the database I dumped all the hashes of wordpress users so I could crack them. In further examination of the machine I found plain text credentials stored in a to-do list of a user, and I could use these to log in as a real user - graham.
Now as graham, using sudo -l I found I had read,write and execute permissions on a bash script from user jens and we can execute it as the user jens. I can exploit this simply by appending /bin/bash to the script and execute it as the user jens.

### Getting root
To get root was relatively simple, as user jens I again ran sudo -l and saw I can run nmap with sudo. A search on gtfobins returns a string of commands used to escalate to root. Basically I created a file in which I inserted "os.execute('/bin/bash')" and then ran nmap with the --script= flag with our file as a parameter. And when it executes it will give us a root shell inside nmap.