# [TryHackMe:Internal](https://tryhackme.com/room/internal)
## Enumeration
Scanning target via *namp*
```bash
 sudo nmap -p- -Pn -sV -sC 10.130.144.228 -oN 10.130.144.228
```
Not too much founded but we know that server run in Linux environment  and SSH port  open.
```
Nmap scan report for internal.thm (10.130.144.228)
Host is up (0.00039s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.04 seconds
root@ip-10-130-83-170:~# 

```
**Content Directory**
enumerating web directories.
```bash
gobuster dir -u "http://internal.thm/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 64
```
we found this result
```
/blog                 (Status: 301) [Size: 311] [--> http://internal.thm/blog/]
/wordpress            (Status: 301) [Size: 316] [--> http://internal.thm/wordpress/]
/phpmyadmin           (Status: 301) [Size: 317] [--> http://internal.thm/phpmyadmin/]
/server-status        (Status: 403) [Size: 277]
```
login to WordPress page

![image](../../images/int-1_clean.png)

Bad login error message handling lead us to username enumeration vulnerability because if we tried another username we will find this **message**. 

![image](../../images/int-16_clean.png)


At this point I will use *wpscan* to do password attack to *admin* user with rockyou.txt 

```bash
wpscan --url http://internal.thm/blog --usernames admin --passwords /usr/share/wordlists/rockyou.txt --max-threads 50
```
Bingo! we have password 

![image](../../images/int-2_clean.png)

## Initial access
After login in inside admin page and searching a way to execute command on server through WordPress I found this way.
1) Go to appearance 
2) Theme Editor
3) Go simple page (or any page you can GET it)
4) Edit code to [PentestMonkey php reverse shell](https://github.com/pentestmonkey/php-reverse-shell)

![image](../../images/int-3_clean.png)

5) Establish on attacking machine an *nc* listener 
6) Request page `http://internal.thm/blog/index.php/sample-page/`
Now we have shell  !

![image](../../images/int-4_clean.png)

## Privilege escalation
### Enumeration
Installing [LinEnum.sh](https://raw.githubusercontent.com/rebootuser/LinEnum/refs/heads/master/LinEnum.sh) on target machine to search for anything lead us to privilege escalation.

![image](../../images/int-5_clean.png)

After running the script we found that file `/opt/wp-save.txt`contain password for user Aubreanna which we found during listing users through enumeration phase . 

![image](../../images/int-6_clean.png)

It is repeated pattern in CTF machine when find this type of credentials it is SSH cred.  

**login to SSH...**

Now we have session with user aubreanna 

![image](../../images/int-7_clean.png)
### Enumeration 
**User Flag**

![image](../../images/int-8_clean.png)

Also there is file *jenkins.txt*

![image](../../images/int-9_clean.png)

We have jenkins which is automation server to handle (CI/CD) . Let's investigate it. Using magic of tunneling we will tunnel traffic from server to our machine using SSH tunneling. 
We will open our machine terminal and establish open port then forward it to port 8080 on target server.

```bash
root@:ssh -L 9000:172.17.0.2:8080 aubreanna@10.130.148.115
aubreanna@10.130.148.115's password: 
# Enter password
```

Browse `localhost:9000` inside browser and bingo! 

![image](../../images/int-10_clean.png)

Doing simple google search about default credentials user name is *admin* but we do not know the password so we will brute force it using hydra or fuff and rockyou.txt use any tool but the password is inside rockyou.txt 

![image](../../images/int-11_clean.png)

We logged in with admin user maybe this user have higher privileges on system . Anyways jenkins can execute command line inside target machine. So we will establish reverse shell via Jenkins.

![image](../../images/int-12_clean.png)

And we got shell with **Jenkins** user 

![image](../../images/int-13_clean.png)

After some enumeration we got this note file inside `/opt`

![image](../../images/int-14_clean.png)

Auberanna which the user we have access on asks for root privileges for a reason. Going to shell we established before for this user and enter this password

![image](../../images/int-15_clean.png)

Bingo! we got root.

