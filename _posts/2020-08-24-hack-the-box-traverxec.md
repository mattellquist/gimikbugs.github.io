---
layout: post
title: "Traverxec"
subtitle: Hack The Box
date: 2020-08-24
image: '/images/traverxec/traverxec_bio.png'
share-img: '/images/traverxec/traverxec_bio.png'
published: false
categories: writeups
author: Gimik
tags:
  - Writeup
  - kali
  
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