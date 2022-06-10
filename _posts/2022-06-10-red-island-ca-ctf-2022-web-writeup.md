---
layout: post
title:  "CA CTF 2022: Exploiting Redis Lua Sandbox Escape RCE with SSRF - Red Island"
external_url: https://www.hackthebox.com/blog/red-island-ca-ctf-2022-web-writeup
external_site: https://www.hackthebox.com
feed:
  excerpt_only: true
date: 2022-06-10
categories: CTF
thumbnail: /img/red-island.webp#center
tags: HTB CA-CTF ctf web ssrf node-libcurl gopher redis cve-2022-0543 write-up
---

In this write-up, we'll go over the web challenge Red Island, rated as medium difficulty in the Cyber Apocalypse CTF 2022. The solution requires exploiting a Server-Side Request Forgery (SSRF) vulnerability to perform Redis Lua sandbox escape RCE (CVE-2022-0543) with Gopher protocol.
