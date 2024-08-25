---
title: Not Giving-Up :&nbsp;Hacking Netherlands Government
categories:
- write-ups
excerpt: |
  My experience of hacking into the dutch government and get rewarded “Dutch Government lousy T-Shirt”. In this blog post, i’ll explain all the technical part and non-technical parts of it.
image: "https://picsum.photos/2560/600?image=733"
---

> I get rewarded “Dutch Government T-Shirt” for my contribution to their security infrastructure. I’ll explain all the technical part and non-technical parts of it. But before that, I want to tell you why am I proud of it or why the title of this post is “NOT GIVING-UP”.

### Why so much Proud?
- Well, there are mainly 2 reasons for it.

1. This is my small aim in 2022 to hack into any government, so I chose the Netherlands government (cuz they offer this cool T-Shirt), but I was even not able to make it in 2023, and finally made it possible in Feb-2024.

2. I reported around 11 reports to the Netherlands government and 10 of them closed. So it became very challenging.

<img src="/assets/logos/ncsc.jpeg" alt="ncsc" width="1000" height="auto">

- Now, Let’s come to the technical part of it ..e what was the Vulnerability, what was my approach ..etc

### What was my Mindset?
- For the first 4–5 reports, I was totally based on recon, there is no such manual hacking over there. It mainly included subdomain brute-forcing, directory brute-forcing, and exposure of secrets.

- 1st report → `phpinfo.php` disclosure. (includes server versions and system internal paths) → Closed as “N/A” (Dec-2022)
- 2nd report → `Cross Site Scripting` → non-authentication application.(simple blog webpage). (Jan-2023)
- 3rd report → `Blind SQL Injection` → Not the part of Netherlands government.(Feb-2023)
- 4th report → `Username Enumeration` → Duplicate (Apr-2023)
- 5th report → `Server Security Misconfigurations` → Closed as N/A. (July-2023)

This continues, So in December 2023, when I recovered from my mental health issues(lacking interest & motivation in everything) and came back to hacking, I decided again to hack on the Netherlands government through manual hacking only. 

I restarted in January 2024 and sent 5 reports to them which included critical issues, like Account Takeover, SSRF, XSS ..etc
I was pretty much shocked when they said N/A in every report, but at last, I reported SSRF and it got accepted.

### What was my initial Approach?

well, come back to the topic.

1 — At first, I was going like other hackers i..e just to pick up any one asset from the [GitHub scope](https://gist.github.com/R0X4R/81e6c50c091a20b060afe5c259b58cfa) list and start hacking on it.

2 — When I was not getting results, I started the shortcut method i..e picked up assets that had already been tested before, So I went through all the medium articles where others told their stories of “Hacking into the Dutch Government” and picked up assets from their. Still not getting accepted.

3 — When I came back to December 2023, I was focused on manual hacking, so that time I was focused more on account takeovers and authentication-based attacks, and luckily found an Account Takeover via CSRF attack in one of their servers, which they closed as N/A, I don’t know why ??

4 — Then, I created a nuclei template to find and filters the subdomains which includes login/signup features. I tested about 3–4 web-application but nothing interesting was found. Most of them are restricted.

```yaml
id: LOGIN-SIGNUP

info:
  name: Login/Signup Detection
  author: Prakhar0x01
  severity: info
  classification:
    cpe: cpe:2.3:a:application:application:*:*:*:*:*:*:*:*
  metadata:
    verified: true
    max-request: 4
    vendor: application
    product: application
    category: cms
  tags: tech,login,signup,forms

http:
  - method: GET
    path:
      - "{{BaseURL}}"

    redirects: false
    max-redirects: 0
    stop-at-first-match: true

    matchers-condition: or
    matchers:
      - type: word
        words:
          - 'Login'
          - 'login'
          - 'Sign In'
          - 'Sign Up'
          - 'Register'
          - 'login-form'
          - 'signup-form'
          - 'register-form'
          - 'signin'
          - 'signup'
          - 'create account'
          - 'create_account'
          - 'createaccount'
          - 'Create Account'
          - 'Create_Account'
        condition: or
```

- But this template is pretty much accurate.

### How I found [SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery) (Server Side Request Forgery)
- At last, I got a subdomain where I just noticed some random `xml` data, it was not like a typical `404` demo page. (I forgot to take a screenshot of it). I was about to move on, but then i noticed something like `GeoServer`, I bruteforce and found the `/wms` directory, I looked into it → found a version → researched about it, and found out that the version is vulnerable to [`CVE-2023–43795`](https://www.synacktiv.com/en/advisories/unauthenticated-server-side-request-forgery-crlf-injection-in-geoserver-wms)
- I quickly tested the exploit and made POC of it, wrote the explanation, and reported it.

- BTW, I can also achieve RCE through it, but it requires certain server conditions, hence I finalize to report SSRF only.

<img src="/assets/logos/ssrf.jpg" alt="ssrf" width="1000" height="auto">

### Conclusion
Yes, hacking becoming hard nowadays cuz there is so much competition globally from various other great hackers and developers aren’t fools like before. But at the same time, you’ll get more attack surfaces like APIs, and Mob. application, IOS ..etc

> “Sometimes you don’t need to be very smart or very hard-working, all you need is just not to Give Up”.

#### Timeline
- Reported -> 23/Jan/2024
- Fixed -> 31/Jan/2024
- Swag Recieved -> 27/Mar/2024