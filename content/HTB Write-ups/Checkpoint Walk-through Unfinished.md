## Intro

Welcome to this unofficial write-up accessible only to OTERIA jury, respecting HTB rules about publishing a write up of an active machine publicly.

This report was made to demonstrate my methodology when encountering an AD in a HTB context.

## Enumeration

- To begin with, Windows AD are still new to me, so this will focus on how i learn and deal with struggles.
- First we start with the classic nmap scan `nmap -sV -sC -p- 10.129.176.249`
![[Pasted image 20261020205500.png|428]]

- We are given credentials: alex.turner / Checkpoint2024!
	- From what i can remember of an AD enum, let's start to use netexec to enumerate users by making the LDAP speak a bit![[Pasted image 20261020210455.png|633]]
	- It gave us additional piece of information such as the domain name
	- netexec (nxc) can also enumerate smb file shares, let's try![[Pasted image 20261020212655.png]]
	- We have `READ` access to these files, which means it is a potential way to progress
	- We found a single interesting file across the shares: `GptTmpl.inf`![[Pasted image 20261020221201.png|633]]
	- It seems to talk in terms of RID instead of users (according to AI and research), let's check RID![[Pasted image 20261020222443.png]]
	- To be transparent, i had no clues of how should I exploit all of this, and i couldn't understand how it work atm. Even though I asked AI to explain me the concepts, it's late and i'm lost.
- A common step to enumerate AD is to use bloodhound, it is a graphical representation of the system, more visual and "thinking friendly" 
