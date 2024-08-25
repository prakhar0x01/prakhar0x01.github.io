---
title: The tale of IDOR's :&nbsp;Hacking US Dept. Of Defence
categories:
- write-ups
excerpt: |
  My experience of finding multiple IDOR's vulnerabilities in the US Army Systems sub-unit of US Dept. Of Defence. In this blog post, i’ll explain all the technical part and non-technical parts of it.
image: "https://picsum.photos/2560/600?image=733"
---

> I get acknowledged & appreciated by US Department Of Defence for finding multiple Security vulnerabilities in the US amry systems i..e `█████.army.mil`. In this blog post, I’ll explain all the technical part and non-technical parts of it.

- If you want to read original Reports :
1. [**#2586616** : Restrict any user from Login to their account](https://hackerone.com/reports/2586616)
2. [**#2586662** : IDOR leads to Modify any user Biographical Details](https://hackerone.com/reports/2586662)
3. [**#2587953** : Email Takeover leads to Permanent Account Deletion](https://hackerone.com/reports/2587953)
4. [**#2586641** : IDOR leads to view any user Biographical Details](https://hackerone.com/reports/2586641)
5. [**#2586584** : IDOR leads to PII Leak](https://hackerone.com/reports/2586584)

### My Initial Approach?

1 - At first, I picked up any one asset from the [GitHub scope](https://github.com/KingOfBugbounty/KingRecon_DOD) list and started hacking on it.

2 - But after a couple of minutes, I realized this is not my thing, I feel like wasting my time in filtering assets & running automated tools for subdomain enumeration.

3 - Hence, I decided to look for [Access Control](https://portswigger.net/web-security/access-control), [IDOR's](https://portswigger.net/web-security/access-control/idor), [Authentication](https://portswigger.net/web-security/authentication) Issues. So, for that, I need functional assets that have some functionalities that need to be broken.

4 - Hence, I used simple google dorks as <code style="color:red;">Site: *.army.mil "register" | "sign up" | "login"</code>. I have some interesting assets, but most are restricted only to federal staff.

5 - One is available for normal users like it was based on internships or contract-based serving in the US Army. Like as a normal user, I could apply in that portal by registering myself & submitting my application.

### [1 - IDOR leads to view any user Biographical Details](https://hackerone.com/reports/2586641)
- Before submitting your application, you have to complete your profile in which your biographical details are also included. The user's Identification is based on Numeric IDs (eg. `121312`).

- Yeah, so now it was pretty much very simple to change your user-id to someone else user-id.

<img src="/assets/logos/idor1.png" alt="ssrf" width="1000" height="auto">


### [2 - IDOR leads to modify any user Biographical Details](https://hackerone.com/reports/2586662)
- On the same endpoint (as above), there is a feature to update your biographical details. It was also based on numeric user IDs. so yes it also became very easy for me to exploit it.

##### Vulnerable Request

```r
POST /JOINOnline/Board/SubmitDoc HTTP/1.1
Host: www.█████.army.mil
Cookie: {YOUR-COOKIES}
...snip...
Connection: close

------WebKitFormBoundaryrQSrSuOi1l18BB2E
Content-Disposition: form-data; name="UserId"
268
------WebKitFormBoundaryrQSrSuOi1l18BB2E
Content-Disposition: form-data; name="Id"
1328
------WebKitFormBoundaryrQSrSuOi1l18BB2E
...snip...
Male
------WebKitFormBoundaryrQSrSuOi1l18BB2E
Content-Disposition: form-data; name="__RequestVerificationToken"
{VERIFICATION-TOKEN}
------WebKitFormBoundaryrQSrSuOi1l18BB2E--
```

- Yeah, so it was pretty much very simple to change your Doc-id (`1328`) to someone else doc-id & change other user's details.

### [3 - IDOR leads to PII Leak](https://hackerone.com/reports/2586584)
- This vulnerability happened in the Update Account section. In this section, there is no sensitive information other than email.

<img src="/assets/logos/idor2.png" alt="ssrf" width="1000" height="auto">


### [4 - Restrict any user from Login to their account](https://hackerone.com/reports/2586616)
- I found this vulnerability in the Update Account section. There is no verification when changing the email.

- so I can change my email to one that belongs to another user already registered on the portal. This would prevent that user from logging in and their account would be suspended.

### [5 - Email Takeover leads to Permanent Account Deletion](https://hackerone.com/reports/2587953)
- So after all the above findings, I'm looking for some more attack surface. Hence decided to look into web archives.

```
https://web.archive.org/cdx/search/cdx?url=█████.army.mil&fl=original&collapse=urlkey
```

- Eventually, discover a hidden portal :

```
https://www.█████.army.mil/852████3EBO25/CreateAccount.html
```

- Now the process is pretty much the same as above (Case-4). The only difference is that once I take over any user email. Now that the user account is no more & either the account is deleted permanently or the account information(including password) is changed to my(attacker's) information.

- Hence, I just created two accounts & perform the necessary steps to exploit the vulnerability.


### Conclusion
It took me about 2 hrs to find 7 vulnerabilities in one asset. I didn't perform so much recon, automation ..etc. Just picked up one asset from Google Dorks & Started hacking on it. And the vulnerabilities are pretty much very simple & straightforward.

> “Sometimes you don’t need to be very smart, just try simple things”.

#### Timeline
- Reported -> 06/July/2024
- Triaged & Appreciated -> 06/July/2024 - 07/July/2024
- Resolved -> 12/July/2024 - 18/July/2024
