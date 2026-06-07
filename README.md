**TryHackMe - Lian Yu (walkthrough)**

**Step 1 Reconnaissance and Port Scanning**

The first phase involves identifying open ports and services running on the target machine. An Nmap scan was performed to gather this information.
**Command: nmap -sV 10.48.128.239**

<img width="1675" height="824" alt="1Nmap" src="https://github.com/user-attachments/assets/a5f79fbf-2f8f-42ad-a4b5-c465da8b6805" />

**Step 2 Web Application Analysis**

Following the discovery of an open web service (Port 80), the target IP was accessed via a web browser to inspect the application front-end.

**URL Accessed: http://10.48.128.239**

<img width="1671" height="1199" alt="2HTTP" src="https://github.com/user-attachments/assets/5a5023ff-d00e-4de5-942d-ffa1da43b7bd" />

**Step 3 Web Directory Enumeration**

To identify hidden directories and potential entry points within the web application, a directory brute-force attack was conducted. This process helps uncover paths that are not linked on the main landing page.

**Command Execution: gobuster dir -u http://10.48.128.239 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt**

Key Discovery: The scan successfully identified a hidden directory: /island (Status: 301).

<img width="1145" height="412" alt="3Gobuster" src="https://github.com/user-attachments/assets/f57f858a-692e-4973-bf11-06c7ede90c18" />

**Step 4 Manual Investigation of Hidden Directories**

**URL Accessed: http://10.48.128.239/island/**

<img width="1277" height="411" alt="4Vigilante" src="https://github.com/user-attachments/assets/191bb987-bbbc-4e5a-8b97-08d7b0ed6f1d" />

**Step 5 Recursive Directory Enumeration**

With the code word in hand, a more targeted enumeration was performed on the newly discovered /island directory to find further sub-directories.

**Command Execution: gobuster dir -u http://10.48.128.239/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt**

Result: The scan identified another hidden sub-directory named /2100 with a Status 301 (Moved Permanently).

<img width="736" height="436" alt="5Island" src="https://github.com/user-attachments/assets/d59c54f9-e3fb-482d-95dd-0c916be3b135" />

**Step 6 Source Code Inspection of /2100**

Upon discovering the /2100 directory, a manual inspection of the page's source code was performed to identify hidden hints or comments left by the developer.

**URL Accessed: view-source:http://10.48.128.239/island/2100/**

<img width="1274" height="498" alt="6Ticket2100" src="https://github.com/user-attachments/assets/342390f0-9c24-470a-b963-d919943bcab7" />

**Step 7 Identifying the Access Token**

Guided by the hint discovered in the source code of the /2100 directory, a specialized Gobuster scan was executed to locate the specific file mentioned.

**Command Execution: gobuster dir -u http://10.48.128.239/island/2100/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x .ticket -t 50**

Key Discovery: The scan successfully identified a file named green_arrow.ticket (Status: 200).

<img width="736" height="435" alt="7GreenArrowTicket" src="https://github.com/user-attachments/assets/01104771-4b03-49af-9d2d-961343b62223" />

**Step 8 Accessing the Queen’s Gambit Token**

Once the hidden file was identified through enumeration, it was accessed to retrieve the potential credential required for the next stage of the attack

**URL Accessed: http://10.48.128.239/island/2100/green_arrow.ticket**

Data Extraction: Directly below the text, a unique string was identified: **RTy8yhBQdscX**

<img width="1283" height="327" alt="8Gambit" src="https://github.com/user-attachments/assets/809f5fb1-ad28-408b-a654-ea8688fa7bc8" />

**Step 9 Decoding the Access Token**

The encoded string retrieved from the green_arrow.ticket file required decryption to obtain the actual credential. Based on the character set, a Base-58 decoding algorithm was identified as the likely method.

Input Data: The encoded string **RTy8yhBQdscX** was entered into an online Base-58 decoder.

Decoding Result: The tool successfully converted the ciphertext into the plaintext password: **!#th3h00d**

<img width="664" height="619" alt="9Base58" src="https://github.com/user-attachments/assets/10cd2c77-a184-421a-8f30-969766480b2f" />


**Step 10 FTP Exploitation and Data Exfiltration**

With the password **!#th3h00d** successfully decoded, the next phase involved accessing the FTP service identified during the initial scan. This service was used to harvest sensitive files from the target machine.

Key Files Identified:

.other_user: A hidden file potentially containing information about other system accounts.

aa.jpg: A themed image file.

Leave_me_alone.png: An image file.

Queen's_Gambit.png: An image file related to the "Arrow" theme.

Exfiltration Process: The **"get"** command was utilized to download all identified files to the local attacker machine for offline analysis.

<img width="792" height="787" alt="10ftp" src="https://github.com/user-attachments/assets/31462c23-d88c-4d42-9cf2-a2f8970e342c" />

<img width="639" height="135" alt="11other_user" src="https://github.com/user-attachments/assets/4ed52bcd-d35d-4220-957b-ffc95e407828" />

**Step 11 Local File Management and Analysis**

After successfully downloading the files from the FTP server, the contents were organized within the local machine's file system to prepare for further forensic and steganographic analysis.

Directory Location: The exfiltrated data was stored in the **"/root"** directory of the attacking machine.

<img width="803" height="570" alt="12Folder" src="https://github.com/user-attachments/assets/eac98a51-7fca-4ab6-af44-5e30dcf037f1" />

**Step 12 File Header Analysis and Repair**

Upon attempting to open the exfiltrated image Leave_me_alone.png, it was discovered that the file was corrupted or unrecognizable by standard image viewers. A hexadecimal investigation was conducted to verify the file signature.

Technical Action: To repair the file, the corrupted bytes were manually edited and replaced with the standard PNG file signature: **89 50 4E 47 0D 0A 1A 0A**

<img width="736" height="513" alt="13Hexedit" src="https://github.com/user-attachments/assets/ee059303-fc0d-4835-abb5-86d59250e1ae" />

<img width="555" height="405" alt="Leave_me_alone" src="https://github.com/user-attachments/assets/1daa4d0f-b5e0-4fc8-a8b8-2e88f7ebf181" />

**Step 13 Steganographic Extraction and Data Decryption**

After repairing the file headers, a steganographic analysis was performed on the image files. This led to the discovery of an encrypted archive containing sensitive system files.

Extraction Process: The tool **"steghide"** was used to extract hidden data from aa.jpg using the passphrase.

You can find the passphrase from the Leave_me_alone.jpg file.

Command: **steghide extract -sf aa.jpg**

Archive Contents: Upon unzipping the archive, two critical files were recovered: 

passwd.txt: Containing a potential password. 
shado: Containing a second credential or hint.

<img width="491" height="95" alt="14Steghide" src="https://github.com/user-attachments/assets/7b4e859c-cc75-4bea-b05a-bb3141975734" />

<img width="788" height="553" alt="16passwd" src="https://github.com/user-attachments/assets/608c8e59-e855-4eee-82e9-c870c14158ce" />

<img width="643" height="193" alt="15Shado" src="https://github.com/user-attachments/assets/90c0ed36-c6bc-4194-b05b-7220580e1bdb" />

**Step 14 Identifying Secondary User Credentials**

With the files from the encrypted archive extracted, the investigation shifted to identifying valid system users and their corresponding credentials for remote access.

Command Execution: **cat .other_user**

Discovery: The file revealed a specific username: **slade**

<img width="1673" height="415" alt="17CatOther user" src="https://github.com/user-attachments/assets/d9733d94-78b1-4aaf-90e4-af1963c836d6" />


**Step 15 Establishing Remote Access via SSH**

With the valid username and password identified from the previous forensic steps, the next phase was to establish a remote session on the target machine.

Command Execution: **ssh slade@10.48.128.239**

The password can be found in the **shado** file.

<img width="699" height="399" alt="18sshSlade" src="https://github.com/user-attachments/assets/ebbbff5d-b67a-4db3-b2a3-38e334d0f4a7" />

**Step 16 Local Enumeration and Flag Capture**

Once the SSH session was established, the user's home directory was explored to locate the first objective of the challenge.

Command Execution: **ls -al**

Flag Retrieval: The **"cat user.txt"** command was used to read the file.

<img width="548" height="278" alt="19Flag1" src="https://github.com/user-attachments/assets/50ec2329-1648-4baf-a19e-871615b4ee44" />


**Step 17 Privilege Escalation**

After obtaining the initial user shell, the final objective was to escalate privileges to the root user. This was achieved by exploiting a misconfiguration or utilizing sudo permissions granted to the current user.

Command Execution: **sudo pkexec /bin/bash**

The password can be found in the **shado** file.

<img width="368" height="72" alt="20RootLianYu" src="https://github.com/user-attachments/assets/44dccbf5-6f34-4f3e-b59a-c9ff3cbfb7d2" />

**Step 18 Final Flag Retrieval (Root Flag)**

Command Execution: **"ls" followed by "cat root.txt"**

<img width="857" height="501" alt="21Flag" src="https://github.com/user-attachments/assets/0e83f067-9010-4a42-beaa-6544ea656888" />


























