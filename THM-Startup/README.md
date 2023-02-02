###THM-Startup
https://tryhackme.com/room/startup

Last CTF (THM - mrrobot) took to much time, mainly because I oweranalyzed things.
This one i´m going to look for the easy solutions.

##Lets go!

##Starting with standard enumeration.
********
#Nmap (top 1000 ports with script scan)
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
*************
#Dirbuster

```
dirb http://10.10.96.196 -w /usr/share/wordlists/dirb/common.txt

http://10.10.96.196/index.html
http://10.10.96.196/files/ftp/ 
```
*******
#HTTP
![image](https://user-images.githubusercontent.com/93491173/216457959-4ac23f3e-f451-4385-82ff-23bda5fc37e2.png)
```

```
*******
#FTP
Since FTP directory is writable 
```
