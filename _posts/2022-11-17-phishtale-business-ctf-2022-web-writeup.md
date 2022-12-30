---
layout: post
title:  "Business CTF 2022: H2 Request Smuggling and SSTI - Phishtale"
external_url: https://www.hackthebox.com/blog/business-ctf-2022-phishtale-writeup
external_site: https://www.hackthebox.com
feed:
  excerpt_only: true
date: 2022-11-17
categories: CTF
thumbnail: /img/phishtale-business-ctf-2022.png#center
tags: HTB business-ctf ctf web request-smuggling ssti http2 cve-2021-36740 cve-2022-23614 rce write-up
---

This blog post will cover the creator's perspective, challenge motives, and the write-up of the web challenge Phishtale from Business CTF 2022. The challenge involves exploiting an HTTP/2 Request Smuggling vulnerability and bypassing Twig Sandbox Policy for Server-Side Template Injection to gain RCE.