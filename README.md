# Vulnfire-Writeup
Writeup for Vulnfire CTF machine

# Vulnfire 
This is a writeup for a CTF machine leveled as easy created by @hellfire and @xploit.
Download link - [Google Drive Link](https://drive.google.com/file/d/1U6tLn-VgHD4dcU2sAhCL5V1mKykympKm/view?usp=sharing "https://drive.google.com/file/d/1U6tLn-VgHD4dcU2sAhCL5V1mKykympKm/view?usp=sharing")
Note from creators - 
**NOTE : The mentioned CTF is uploaded in google drive with a name of the file, VULNFIRE.ova. Download the file and start the hunt. It might be best if you use VirtualBox because it works fine on it rather than VMWare.**
Complete the installation of box and the networking part.
Find the ip of box using nmap for the range of the network.
for example - nmap 10.10.10.2/24

---
## Recon
Using nmap to find open ports on the target 
```
Nmap 7.92 scan initiated Mon Jul  4 15:13:03 2022 as: nmap -sC -sV -vv -oN nmap.txt 10.10.10.2
Nmap scan report for 10.10.10.2
Host is up, received syn-ack (0.0033s latency).
Scanned at 2022-07-04 15:13:03 +05 for 42s
Not shown: 996 filtered tcp ports (no-response)
PORT   STATE  SERVICE  REASON       VERSION
20/tcp closed ftp-data conn-refused
21/tcp open   ftp      syn-ack      vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.10.3
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open   ssh      syn-ack      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 5a:a7:bb:e0:71:4d:a6:de:b1:4a:33:99:08:64:8c:e1 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDYmcbIoWceZJx34jGvukqBS3Gr8UxJjGhEhB+Q9c/gAp6bf5H57tbw9oWCY+GExTTjgMJhbL8ExK+oYpybkwdfNNBz6MZtMpY/xfNpNWe8i4z0gOHJ8fEJ2R5Xe06lFjfEhXivoSib/hKE7N2LS3CzAnUsBaKc03p7VitHVtXljsHFaIXhZ2LWOgINMw6MuI7DbZ0faaE7/hZ6DSF3+uLh/Y3DBrBFJXVFuS8jh0wmj/1zNV+ws6550rk8ipUpEoIFS5QKZ7MmfpkRrDEMVDV5tIGRuWqAgVoT8C4UMNBHgSlFXAlfLOd4+nkfBg6KmB5ZoVtezIpmehomWbiAfUvCt4h4SX3RqVEFNnpmasOGRj2jtAfbzU0o+LukdhsxlmovSreGZ6KvX3LnC1kxsCBjeq0TetIi4gIm0xj9KLNtPzA5hCb9CtkzD8YpHFPWh1ZkCDm6fPIOlxOrPS7OwRBh1UiXsrC3dzkDqPZLW5mq0Lqr7bfZg4/k2JpQ7ZiJ49k=
|   256 1d:01:1f:df:79:84:47:f3:20:fb:72:c3:d6:20:71:03 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBO6A0y/3ftcML11jU+hGgTQXGCNG+QkhMUifnGnlgcMMHkU+qOJrisLvfxW9o978/qzwEyEhsPbMWHbRmT0j2G8=
|   256 ef:10:bb:ea:d7:44:88:5b:ef:57:03:8c:45:20:1a:8e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAkzcehwqJhTcPbmAVLt9YUMN/1qREuPFroeEuNHbnbr
80/tcp open   http     syn-ack      Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done at Mon Jul  4 15:13:45 2022 -- 1 IP address (1 host up) scanned in 42.52 seconds
```

According to the scan , there are 3 ports open. I only scanned 1000 ports since i knew this is a easy box and it wouldn't be configured on ports not default to services.
Let's enumerate these services one by one.

### Enumerating Anonymous FTP
By nmap script scan, it tells us there is anonymous login enabled on the ftp server. Provide the username as Anonymous and leave password empty.
You will get into the server. There are two interesting files there.
![[ftp_login.png]]
I got those two files on my machine using `get filename`.

Contents of note.txt 
![[note_txt_ftp.png]]
It gives 3 hints:
- webserver has a file which contain some kind of key
- there are php files on the webserver
- an image have steganographic data in them

Unzipping confidential.zip
It gives two files , fire.jpg and secret.txt 
![[fire_jpg_ftp.png]]
This image may contain steganographic data.
Tried different steg tools but couldn't find something for now.
Maybe we need a key to extract the data

Checking the other file
![[secret_txt_ftp.png]]
It tells us that the webserver shows default ubuntu page as nothing is configured for it. It also points that we have to perform fuzzing with certain file extensions to find files or directories which are not supposed to be exposed to regular user.

### Enumerating HTTP
Visited the http server. Found default page as stated above.
Tried opening robots.txt and viewing source code , not much to be found. 
Since we don't have any other data to look at and also the hint pointed, we fuzz for files and directories.
Used Gobuster with extensions .php as mentioned in ftp files
![[gobuster_log.png]]
found a file which have 200 response code.
It redirects to op_security.php?cmd= 
Since the file had a parameter , tried injecting commands and lfi nothing worked 
checked source code and found list of all parameters present
![[source_code 1.png]]
copied them into a dict file.
used ffuf for fuzzing the parameter with id value
`ffuf -c -u 'http://$ip/op_security.php?FUZZ=id' -w dic.txt`
![[ffuf_results.png]]
All the parameters have same output but one stands out as it have different number of output lines.
used that and tried command injection and it was successfull
![[mal_id_cmd.png]]
![[mal_ls_cmd.png]]

read the file using cat

![[http_file_ouput.png]]
Since it talks about steganography , tried that password for extracting files from the image.

### Getting access to system
used steghide on fire.jpg with above password
it extracted creds.txt
![[creds_txt_steg.png]]
It have the credentials for user but is encoded. The note tells us about decoding it as 'Vignere cipher' as it says 'Vi..' also it have provided the key to the cipher by highlighting the word **Think** with quotes.
using 'Think' as key and decoding string using online sites.
found creds for the user for ssh
`hellfire:xplo1t&hellfirearenoobs`

### Lateral Privesc
found user.txt and a message.txt

![[osint_hint.png]]

Quickly searched for user on twitter cause that's the most used osint social media account. Maybe would have used sherlock if haven't found it.
Found a paste-bin for leaked credentials , maybe there is password for another user that is on the machine i.e xploit.

copied hashes in a file and used john to crack them all
`john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=RAW-SHA256`
`john hashes.txt --format=RAW-SHA256 --show`
found 171 passwords , saved in a passwords.txt file

used hydra to bruteforce ssh pass for xploit

![[hydra_brute.png]]
`xploit:vcdkqhhqdtvxbc3c4mj3238gg`

### Privesc to root
Login as xploit using above creds.
found evil.txt which says:
`#TryHarder! Root password is unusual among many...`
suggested that there is a password leak somewhere and it contains many passwords , one of them is root's.

tried some manual recon and found interesting file in /var/backups/.bak
used `tar -xvf secret.tar`
After extracting , it gave out 4 pcap files
transferred them to my machine
Since they are pcaps, thought of using wireshark but basic forensics comes first.
run strings on first , nothing useful
run strings on second, found it contains ftp login packet capture ( as ftp is in plaintext)

![[root_ftp_info.png]]

there are passwords supplied too.
found different passwords for root
grep for only lines which contains passwords
one password stands out and hint pointed the root's password to be unusual among many.

![[root_pass.png]]
`Fv67i_+28WRT0mY7x`  
### Getting the root flag
`su root` with this password and now we are root
found confidential.txt which says to read about.me file
![[root_flag_task.png]]

about.me is encoded 

![[root_aboutme_enc.png]]
Looks like simple base64 
base64 decode it and found the flag.
![[root_flag_dcod.png]]