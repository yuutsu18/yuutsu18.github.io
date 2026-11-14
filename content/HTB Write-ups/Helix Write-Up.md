
## Intro

Welcome to this unofficial write-up accessible only to OTERIA jury, respecting HTB rules about publishing a write up of an active machine publicly.

This small report was made to demonstrate my methodology when encountering a Linux machine in a HTB context.

### Enumeration

This is the most crucial step of a HTB CTF (as well as in a real situation). 
We need to be methodical in our approach:

- First, nmap, always (`nmap -Sv -sC -p- <machine-ip>`) -> 22(SSH) 80(HTTP)

- Then, we need to check for directories if there are any we can access and subdomains with `ffuf` -> flow.helix.htb -> apache Nifi v1.21.0

- Now that we have a vesion of a service, we should search for a high severity CVE that allow RCE (classic on HTB) -> luckily for us, with a quick internet search -> CVE 2023-34468 ! (CVSS 8.8)

- Basically, we understand quickly that we can execute code (while unauthenticated) on the server thanks to it.

- From here, we have 3 options:

	- Use metasploit and search for a module that exploits this CVE
	- Search for a PoC of this vuln on github and modifiy it if necessary
	- Write a script / Exploit with our hands

- For a HTB context, Metasploit / GitHub PoC are our best friends, so let's use one of them.

### Reverse-Shell into Horizontal Movement

Now that we exploited the CVE, we get access to the server as "nifi".
As of today, I would have continued the box by installing linpeas on the server somewhere we can have access to and execute the script to find something faster. 
At the time, I searched for suspicious file manually running commands and travelling the file system.

- Like I mentioned, I was searching manually, so I was looking for potential config files,... in the folder of the service that allowed us to come there.

- We find in `/opt/nifi-1.21.0/support-bundles` -> `operator_id_ed25519.bak`
  Perfect ! We got the bak config file, and it contains a huge gift for us: The SSH private key of operator (the user we find in `/user`).

- Generally, an SSH private key is protected by a password, that you would have to crack using tools such as hashcat. 
  Here, we find out this key doesn't have any pw ;)

- We can just login as operator with this private key that we just found -> user.txt 🚩

### Lateral Movement (PrivEsc)

Nice, we just got access to the user account. I did another enumeration of the machine and search for 2 things that are very recurrent in HTB machine for the root access 

-> Listening services on weird ports (`ss -tlnp`) that you usually have to port forward using chisel for example

-> Files with sudo permission that are specific to the box 


- By checking files with sudo permissions (with a simple `sudo -l`), we find `/usr/local/sbin/helix-maint-console NOPASSWD (root)` and identify it instantly as the main PrivEsc vector.

- We perform `ss -tlnp` to check for listening services -> 127.0.0.1:4840 -> "opc ua" -> definietly linked the helix maintenance console

- In `operator` files (`/user/operator`), we find a pdf that we have to crack, and then it explains the how the nuclear reactor service works.

- By understanding the pdf content, we can establish a plan:

	- We need to satisfy the conditions to make helix-maint-console spawn a root shell (make the temperature be >=295 Celcius degrees and open a maintenance window)
	- The most essential step to not skip: make the temperature respect the alarm threshold that (in real life will cause danger) blocks us away from getting the root shell

- So, our script needs to meet theses expectations. 

- Since AI is a scripting god, we need to prompt it so that the script will exactly make what we want it to execute and **understand every line of the script before executing it.**

- Finally, we execute the script (`chmod +x`) and spawn a root shell as expected ! root# 🏁
#### little experience 💚

In my run, i was stuck in the cooking of the script -> I fought my social anxiety and asked for a nudge on the HTB discord since this was the latest machine at the time (active, so no help). 
A kind person helped me a gave me the script and explained what it performed.
Then, the person told me that they used AI to make the script since it was way faster and efficient if you prompt it properly and understand what it does (so you can execute it safely, here this was important in this nuclear reactor situation).


## Amelioration

In this box, what i would have done completely differently is the enum when gaining access to the reverse shell and user.

-> In fact, using ==🟡linpeas== is faster and can show you things that you forgot to check.

-> The other advantage is that in most the cases you can figure out the path to user then root at the same time, because everything is shown directly.

-> You can establish plans and think about the path **before** trying to exploit something mindlessly.
