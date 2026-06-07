# LAB-6---LIAN-YU
# TryHackMe-Lian_Yu WalkThrough

A complete TryHackMe "Lian_Yu" CTF walkthrough focusing on web enumeration, repairing corrupted magic bytes, steganography, and Linux privilege escalation.

## Machine Details

| Item | Description |                       
|------|-------------|
| Machine Name | Lian_Yu |
| Platform | TryHackMe |
| Difficulty | Easy |

## Target Information
* **Target IP:** 10.48.174.186
* **Operating System:** Linux


## Phase 1: Reconnaissance & Footprinting
The assessment commenced with a passive analysis of the target's web presence to gather initial intelligence before launching aggressive scans.

**Web Surface Analysis (Port 80)**
Navigating to the target's IP address `(http://10.48.174.186)` revealed an "ARROWVERSE" themed landing page detailing Oliver Queen's backstory.

**Key Discovery:**
While the page lacked interactive elements, a deeper inspection of the HTML source code yielded a critical footprinting clue. A hidden comment contained a specific code word:

```bash
vigilante
```

## Phase 2: Scanning & Enumeration

With basic context established, active scanning was utilized to map the attack surface, identify open ports, and locate hidden web directories.

**Network Vulnerability Scanning**

An initial `nmap` scan was executed to probe for running services and their specific versions:

```bash
nmap -sC -sV 10.48.174.186
```

![Nmap Scan Result](images/01-recon/nmap/nmap-scan.png)

**Open Ports Identified:**

`21/tcp` - FTP (vsftpd 3.0.2)

`22/tcp` - SSH (OpenSSH 6.7p1)

`80/tcp` - HTTP (Apache httpd)

`111/tcp` - RPC (rpcbind)

**Web Directory Bruteforcing**

Visiting the web server on `port 80` :

```bash
http://10.48.174.186
```

![Website Arrowverse](images/01-recon/enumeration/web-enumeration.png)

The landing page features an 'ARROWVERSE' theme detailing Oliver Queen's backstory. Since there are no visible links or interactive elements on the surface, the next logical step is to perform directory brute-forcing to uncover hidden paths.

Gobuster was deployed to brute-force hidden paths:

```bash
gobuster dir -u http://10.48.174.186/ --wordlist /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
```

This enumeration successfully uncovered a hidden directory: `/island`. A subsequent scan targeting `http://10.48.174.186/island`.

![Website island](images/02-scanning/gobuster-01-ip/user-vigilante.png)

We found out the Code Word by highlighting the `page text` or `viewing the page source`.

  Code Word : 
  
  ```bash
  vigilante - (This is our FTP user)
  ```

To dig deeper into the web structure, a secondary `gobuster` scan was initiated, this time specifically targeting the `/island` directory:

```bash
gobuster dir -u [http://10.48.174.186/island](http://10.48.174.186/island) --wordlist /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
```

It revealed a nested directory: `/2100`.

![source-code](images/02-scanning/gobuster-02-island/page-source.png)

Upon investigating the source code of the `/2100` page, a comment hinted at files utilizing a .ticket extension. A targeted gobuster scan was initiated using the `-x` flag:

```bash
gobuster dir -u http://10.48.174.186/island/2100 --wordlist /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -x .ticket
```

This precise scan located `green_arrow.ticket`. Viewing this file in the browser exposed a Base58 encoded token: `RTy8yhBQdscX`.

![Website-green](images/02-scanning/gobuster-03-2100-ticket/web-encrypt-code.png)

Utilizing CyberChef to decode the Base58 string produced the following plaintext password:

```bash
!#th3h00d
```

![Website CyberChef](images/02-scanning/gobuster-03-2100-ticket/ftp-password.png)

## Phase 3: Gaining Access (Initial Foothold)

Armed with the intelligence gathered from the scanning phase, the next objective was to breach the system.

**FTP Exploitation & Data Exfiltration**

The credentials obtained (`vigilante` / `!#th3h00d`) were used to successfully authenticate into the FTP service on port 21.

Directory listing (ls -al) revealed three images, which were downloaded for local analysis. Additionally, navigating to the parent directory exposed the existence of a second user on the system: `slade`.

**Steganography & Magic Byte Repair**

While `aa.jpg` and `Queen's_Gambit.png` functioned normally, Leave_me_alone.png returned a format error.
Analyzing the file with a hex editor exposed corrupted magic bytes in the file header `(58 45 6F AE)`.

These were manually corrected to the standard PNG signature (`89 50 4E 47`). The repaired image rendered correctly and displayed a hidden word: `password`.

This keyword was immediately utilized as a passphrase for `steghide` against `aa.jpg`:

```bash
steghide extract -sf aa.jpg
```

This extracted a compressed archive (`ss.zip`). Decompressing the archive yielded two files. Reading the `shado` file provided a new credential:

```bash
M3tahuman
```

## Phase 3.1: Privilege Escalation

The final stage involved leveraging the newly discovered credentials to escalate privileges to a root administrator.

**SSH Authentication**

The credentials for the newly discovered user (slade / M3tahuman) provided successful SSH access to the target environment.

**User Flag Captured:**

```bash
THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
```

**System Enumeration for Privilege Escalation**

A hidden file named `.Important` suggested searching for a "Secret_Mission". Checking the user's sudo privileges (`sudo -l`) revealed a critical misconfiguration: `slade` was permitted to run `/usr/bin/pkexec` as `root` without password authentication.

This vulnerability was exploited to instantly spawn a root shell:

```bash
sudo pkexec /bin/sh
```

With absolute control over the system, the final objective was secured in the root directory.

**Root Flag Captured:**

```bash
THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
```






   

