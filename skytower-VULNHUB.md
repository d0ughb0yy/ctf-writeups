# Skytower

## Recon

### Nmap Scan
PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.2.22 ((Debian))
3128/tcp open  http-proxy Squid http proxy 3.1.20
MAC Address: 08:00:27:54:4A:37 (Oracle VirtualBox virtual NIC)
On this machine we have 2 ports open; 80 and 3128. Port 80 returns a login form and nothing else and port 3128 we can see is running a Squid http proxy.