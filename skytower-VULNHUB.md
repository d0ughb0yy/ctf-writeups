# Skytower

## Recon

### Nmap Scan
PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.2.22 ((Debian))
3128/tcp open  http-proxy Squid http proxy 3.1.20
MAC Address: 08:00:27:54:4A:37 (Oracle VirtualBox virtual NIC)
On this machine we have 2 ports open; 80 and 3128. Port 80 returns a login form and nothing else and port 3128 we can see is running a Squid http proxy.

### Web Exploitation
The login form was using a piece of custom code to filter out SQL Injection attacks and the usual payload ' OR '1'='1'-- was filtered into '11. To bypass this I used || instead of OR and # as a comment indicator; ' || 1=1#
Inside I got a clue from the dashboard, It pointed me to an SSH server and gave me the credentials for it, but there is no SSH port opened so that took me to port 3128 which was running an http proxy.
Since proxy was not protected I can use a tool called proxytunnel to tunnel the ssh server through the proxy on my designated port;
proxytunnel -p <PROXY_IP>:<PROXY_PORT> -d 127.0.0.1:22 -a 1234
This command will tunnel the SSH connection from the local loopback address through the proxy on my port 1234.

## Machine Exploitation

### Initial Foothold
When logging in through the SSH I would get logged out immediately. I suspected it had something with the .bashrc file so when logging in the ssh I passed the /bin/bash parameter to drop me directly into a bash shell then I deleted the .bashrc file and logged in normally through ssh again.
ssh user@IP 1234 /bin/bash --> This will drop you directly into a bash shell bypassing .bashrc commands

### Moving through the machine
As the user john I did not have any special permissions or ways to escalate to root, but when looking through the web directory I found a login.php file with the MySQL database credentials hard coded in. I used those to login into the database and dump the credentials of all users on the machine.

## Escalating to Root
I used the dumped credentials to access another user name sara, I repeated the process with the .bashrc file and logged in as sara. Again I ran sudo -l to see if I got any permissions and I got back the command /bin/cat that can be run as root on the /accounts/ directory, so using sudo cat /accounts/../root/flag.txt I had read permissions as root for all files, the flag under root contained the root password so we can login as root.