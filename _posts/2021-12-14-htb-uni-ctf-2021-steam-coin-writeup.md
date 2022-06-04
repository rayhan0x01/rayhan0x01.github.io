---
layout: post
title:  "Gears of web exploits that sync in harmony; SteamCoin write-up from HTB University CTF 2021"
external_url: https://www.hackthebox.com/blog/UNI-CTF-2021-Steam-Coin
external_site: https://www.hackthebox.com
feed:
  excerpt_only: true
date: 2021-12-14
categories: CTF
thumbnail: /img/steam-coin.webp#center
tags: HTB uni-ctf ctf web cve-2021-40346 scripting request-smuggling blind-xss jwt csrf write-up
---

In this write-up, we'll go over the solution for the challenge SteamCoin that requires the exploitation of multiple server-side and client-side vulnerabilities. The solution involves a JWT authentication bypass through JKU claim misuse using unrestricted file upload, HTTP request smuggling for ACL bypass, and XSS to CSRF on an automated UI testing service to exfiltrate the flag from CouchDB.

