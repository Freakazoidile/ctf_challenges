# DPI 902 - Challenge VM 1
**Submitted By:** Thomas Zuk

**Challenge Machine IP**: 172.16.11.45


## Reconnaissance and Mapping
I started with a Syn scan of all TCP ports against the target machine to identify open ports. 
`nmap -sS -p- 172.16.11.45` which discovered ports `2222 , 31336` were open. I feed these ports into a Nmap Service scan to determine the services running on the ports. As suspected SSH was running on port 2222. Port 31336 was running Apache web server.

```bassh
nmap -p 22222,31336 -A 172.16.11.45
 
PORT      STATE SERVICE VERSION
22222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d2:4c:a3:17:47:64:4c:41:2d:fd:fc:0f:ff:4b:12:2d (RSA)
|   256 51:d2:b3:fd:5e:30:41:ec:19:89:ed:bd:fd:a8:7e:78 (ECDSA)
|_  256 95:82:06:46:8a:97:ac:f6:42:f3:90:7e:e4:ee:8c:41 (EdDSA)
31336/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.8

```

Navigating to `http://172.16.11.45:31336` in a web browser revealed the Apache default index.html. After looking at the source code I discovered a comment that `users in /sekret_users`, loading this directory revealed an image from the movie Hackers and the following text: "I use key based authentication so no one can hack me." This claim, paired with SSH likely means there is a SSH key to retrieve. I tried to log into SSH as user **acidburn** but it failed due to a key error. 

I ran the directory brute force tool `dirb` on `http://172.16.11.45:31336/sekret_users` and discovered the `.ssh` directory. Based on common SSH key names I tried:
	* http://172.16.11.45:31336/sekret_users/.ssh/id_rsa
	* http://172.16.11.45:31336/sekret_users/.ssh/id_rsa.pub
	* * http://172.16.11.45:31336/sekret_users/.ssh/known_hosts

The **id_rsa.pub** file existed! I tried to use the key to ssh to the server on port 2222 `ssh -i zerocool_id_rsa.pub -P 2222 zerocool@172.16.11.45` however it required a password to use the key.  I used `ssh2john` to format the SSH key so that the the password cracking tool `john` could understand the key and attmept to crack the password. Using the wordlist `rockyou.txt` the password associated with the key was `acidburn`. 

I SSH using with the credentials successfully. In Zerocool's home directory was a file aptly named `flag.txt`, the contents were:
`DPI902{HackThePlanet}`

