---
layout: post
title:  "CA CTF 2022: Exploiting LFR and forging Cookies - Mutation Lab"
external_url: https://www.hackthebox.com/blog/mutation-lab-ca-ctf-2022-web-writeup
external_site: https://www.hackthebox.com
feed:
  excerpt_only: true
date: 2022-06-03
categories: CTF
thumbnail: /img/mutation-lab.webp#center
tags: HTB CA-CTF ctf web cookie-forgery lfr cookie-session directory-traversal cve-2021-23631 write-up
---

In this writeup, we'll go over the web challenge Mutation Lab, rated as medium difficulty in the CyberApocalypse CTF 2022. The solution requires exploiting a local file read vulnerability to steal the cookie signing key and crafting a session cookie for the admin.

