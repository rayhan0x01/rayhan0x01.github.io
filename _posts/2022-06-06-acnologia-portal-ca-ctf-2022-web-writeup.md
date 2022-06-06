---
layout: post
title:  "CA CTF 2022: Exploiting Zip Slip and Pickle Deserialization - Acnologia Portal"
external_url: https://www.hackthebox.com/blog/acnologia-portal-ca-ctf-2022-web-writeup
external_site: https://www.hackthebox.com
feed:
  excerpt_only: true
date: 2022-06-06
categories: CTF
thumbnail: /img/acnologia-portal.webp#center
tags: HTB CA-CTF ctf web cookie-forgery blind-xss csrf flask-session pickle-deserialization zip-slip write-up
---

In this write-up, we'll go over the web challenge Acnologia Portal, rated as medium difficulty in the CyberApocalypse CTF 2022. The solution requires exploiting a blind-XSS vulnerability and performing CSRF to upload a zip file for arbitrary file injection, crafting Flask-Session cookie for deserialization to get remote code execution.
