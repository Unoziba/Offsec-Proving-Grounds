**SCANNING**
The tool used for scanning is nmap, specifically the below command
sudo nmap -sC -sV -p- host address  you can use the following nmap scan for faster results; `nmap -T5 -A -p- --min-rate=500 <ip address>`

The scan revealed the following ports to be open:
![[Pasted image 20240102224840.png]]

Opening the host address on a webrowser takes us to a page containing a file named **save.zip**. We then proceed to downloading the zip file.

Attempting to unzip the file, it required a password as shown below:

![[Pasted image 20240102215941.png]]

The only option was to obtain the zip file hash and cracking that hash to obtain the password using a password cracking tool known as **John The Ripper**.

Command to obtain hash:
zip2john zipfile > hashfile, in our case: zip2john save.zip > save.hash

After obtaining the hash, we run command  "**sudo john --wordlist=/usr/share/wordlists/rockyou.txt save.hash**" to obtain the password:

![[Pasted image 20240102221209.png]]
Now it's time to extract the zip file using the password obtained above. The file contains a folder titled **etc**.

etc contents:

![[Pasted image 20240102223146.png]]

Available is the passwd and shadow files which contain user accounts and their password hashes. We can now use our previous tool **John The Ripper** to crack those hashes:

Step 1: Use the unhash command on both the passwd and shadow files and output the results into one file, eg **unshadow passwd shadow > crackit**, crackit being the output of the unshadow command.

Step 2: Use John to crack the above output to get a valid account and password

![[Pasted image 20240103195139.png]]

We then immediately get a password for the **296640a3b825115a47b68fc44501c828** account 

Password: **server**

Since we have the credentials we try to ssh into this unique account.

After logging in we use the command ls to list files in the current directory, we see that there is local.txt and user.txt. Trying to cat these text files gave the error in the screenshot below:

![[Pasted image 20240103234105.png]]
This is because the path variable is linked to the home directory of the current and as such we must make it point to the bin folder where all commands are stored. We do this by running the following command: **export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin**

Running the above command will still return an error and thus we must logout and re-run the ssh command with a -t "bash --noprofile" to break out of the current shell.

eg. ssh **296640a3b825115a47b68fc44501c828**@hostaddress -t "bash --noprofile"

After doing so we set the PATH as discussed before, run the usual cat command to get the local.txt flag:

![[Pasted image 20240103235606.png]]
Privilege Escalation:
From observation we see an interesting ELF file titled honeypot.decoy. Executing this file launches an interactive program with 8 options. 7 of these are basic commands and the one that stands out the most is number 5 (Launch an AV Scan). Observe screenshot below:

![[Pasted image 20240104012234.png]]

From here on out we can use a tool known as pspy64 to see background processes that take place. To use this tool we open a second terminal and ssh into it again. NB: Do not forget to add the -t "bash --noprofile" and change the PATH.

Run a simple http server on your local machine to download/upload the pspy64 tool to the target machine. Once it's downloaded to the target machine, run the command **chmod +x pspy64** to make it executable. To run it in a way that you see all processes, simply use the command **./pspy64 -pf -i 1000**.

Screenshot: Running pspy side by side with honeypot.decoy

![[Pasted image 20240104010358.png]]
From the above screenshot, we see that the root account runs a chkrootkit (highlighted in blue). We then tried to find out if there any vulnerabilities surrounding the chkrootkit version on the target machine and there actually is. check: https://www.exploit-db.com/exploits/33899

Screenshot of the exploitdb page:

![[Pasted image 20240104013535.png]]

We did as suggested in the screenshot however we had to take it further and insert a netcat revershell in the update file and made it executable using **chmod +x update**
. So now when the chkroot command updates it will run the update file with our reverse shell. NB: reverse shell we used was obtained from: https://www.revshells.com/

From our nc listener on our local machine we got a connection and running whoami establishes that we are root and thus we can get the root flag. Observe the screenshot below:

![[Pasted image 20240104010438.png]]

---- The End ---- 
