## Reconnaissance and Enumeration

Let's start with an **nmap** scan to discover the open ports and the services running on the machine.

```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP>
```

```
nmap -p22,5000 -sCV <IP> -oN targeted
```

![nmap](./assets/nmap.png)

## Gaining access 

On the web service running on **port 5000**, we can register a new user.

![register](./assets/Captura_de_pantalla_de_2025-12-03_17-56-07.png)

After registering, we can log in.

![login](./assets/Captura_de_pantalla_de_2025-12-03_17-57-32.png)

Inside the **/feed** directory, we can see another user named **admin**. We also notice that we can upload a profile image. This is a good opportunity to try an **XSS attack** to steal the admin's cookie.

To do this, we need to upload an `.svg` file that contains a small script, because the admin’s browser will execute it.

Here’s a simple script:

![script1](./assets/1script.png)

Go to the button named **"Mi perfil"** to access your profile and upload the script.

Before uploading, listen with **netcat** on port `9999`.

![perfil](./assets/mi_perfil.png)

After uploading the script, we receive a cookie.

![fstcookie](./assets/fstcookie.png)

But this cookie is **our cookie**, so I generated a better script (with DeepSeek) to steal the **admin** cookie.

![script2](./assets/script2.png)

Place your IP in the script, upload it again, and listen with netcat.

![scncookie](./assets/scncookie.png)

Copy the admin cookie into your browser's developer tools (**Storage** section), refresh, and you’re in.

![putcookie](./assets/putthecookie.png)

Once inside, we get credentials for SSH. Let’s use them.

![sshcreds](./assets/enhorabuena.png)


## Scalling privilages

Inside SSH as the user **hijacking**, I checked for root-executable programs with:

```
sudo -l
```

Then I looked for SUID binaries:

```
find / -perm -4000 2>/dev/null
```

![find](./assets/find.png)

Among them, `/usr/bin/env` stands out — it can be abused to escalate privileges easily.

Just run:

```
/usr/bin/env /bin/bash -p
```

![root](./assets/root.png)

And that’s it. We are **root**. So easy >:3

