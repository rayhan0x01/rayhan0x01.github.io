---
layout: post
title:  "Uni CTF 2022: UNIX socket injection to custom RCE POP chain - Spell Orsterra"
external_url: https://www.hackthebox.com/blog/uni-ctf-2022-spell-orsterra-writeup
external_site: https://www.hackthebox.com
feed:
  excerpt_only: true
date: 2022-12-30
categories: CTF
thumbnail: /img/spell-orsterra-uni-ctf-2022.png#center
tags: HTB uni-ctf ctf web nginx unix-socket-injection redis php-messenger deserialization pop-chain php-gd idat-chunks rce write-up
---

This blog post will cover the creator's perspective, challenge motives, and the write-up of the web challenge Spell Orsterra from UNI CTF 2022. The challenge portrays a fictional application with a heavy tech stack and involves exploiting Nginx UNIX socket injection, queued message handling deserialization, and custom POP chain to export PHP backdoor with PHP-GD image compression bypass.