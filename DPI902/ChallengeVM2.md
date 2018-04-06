# DPI 902 - Challenge VM 1
**Submitted By:** Thomas Zuk

**Challenge Machine IP**: 172.16.11.66


## Solution
I started with a Syn scan of all TCP ports against the target machine to identify open ports. 
`nmap -sS -p- 172.16.11.45` which discovered ports `22, 80` were open. I feed these ports into a Nmap Service scan to determine the services running on the ports. These are common ports for SSH and HTTP web server respectively.

Next I ran a TCP Service scan against the open ports to verify the services and version numbers.

```bash
nmap -p 22,80 -A 172.16.11.66


PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e9:d1:3a:13:a5:1a:cc:82:d5:61:31:b0:5d:54:a2:c8 (RSA)
|   256 c9:c4:08:f3:cf:9b:98:15:49:46:34:be:58:44:3b:bf (ECDSA)
|_  256 82:eb:f0:09:63:5d:a7:fd:39:28:08:53:09:36:14:9a (EdDSA)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title:  Megacorp Development Server 

OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.8
```

Visit the web server the index page said that there was a new user `acidburn` and that passwords are no longer just **Welcome** but Welcome with the users birth year appended to the end (E.G. Welcome1983). The author of the webpage is `bigdawg`.
Now that we have 2 possible users and the default password scheme we can attempt to brute force the accounts. First off was `acidburn` as it was the new user and most likely still default.

I create a list of passwords to try, then used `hydra` to brute force the login

```bash
for i in {1900..2018}; do echo "Welcome"$i >> passwords.txt; done

hydra -l acidburn -P passwords ssh://172.16.11.66
```

Hydra identified the valid password for the account as `Welcome1977`. The credentials `acidburn:Welcome1977` were successfully used to SSH into the target machine.

The First flag was revealed: `dpi902{Predictable_Default_Passwords_Make_Me_A_Sad_Panda} `

Knowing that the next goal was to pivot to a webserver I printed the interfaces on the local machine using `ifconfig` which showed another interface connected to the network `10.99.99.0/24`. 

From here I used the `ping` command to perform a ping sweep and find any other online hosts.

```bash
for i in {1..255}; do ping -c 2 10.99.99.$i; done

```

This revealed that the host `10.99.99.2` was online. Using `nc` I check for ports 80, 8080, 443, 8443, since we were looking for a web server. Port 80 was open!

I used SSH from my local kali machine to port forward my local port 8080 to port 80 on the new target machine with the following command:

`ssh -L 8080:10.99.99.2:80 acidburn@172.16.11.66`

I entered `http://127.0.0.1:8080` in my local browser which would use the SSH connection to connect to `http://10.99.99.2:80`. A login prompt appeared.


During my initial enumeration of the 127.16.11.66 webserver I discovered the directory  **/scripts** using dirb, which had a message that said these files are no longer accessible on the web server and have been moved to "/opt" for local access. On 172.16.11.66 the file `/opt/scripts/backup/backup.py` exists, the contents are in the screenshot below:

![alt text]( https://raw.githubusercontent.com/Freakazoidile/ctf_challenges/master/DPI902/screenshot%202.png "Challenge VM2 - Backup.py")


I used these credentials on the login prompt and obtained the second flag:

`secret_flag: dpi902{pivot_all_the_things}`







