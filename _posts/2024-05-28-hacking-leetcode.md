---
title: From Coins to Chaos :&nbsp;Business Logic Exploits in LeetCode
categories:
- write-ups
excerpt: |
  While most people use LeetCode to sharpen their problem-solving skills, I took a different route—digging into its logic. In this blog, I’ll walk you through how I found and responsibly reported three impactful business logic vulnerabilities in LeetCode’s main application - [leetcode.com](https://leetcode.com/). Each case involved clever misuse of logic, and none required complex technical exploits. Just pure understanding of how things shouldn't work.
image: "https://picsum.photos/2560/600?image=733"
---

> While most people use LeetCode to sharpen their problem-solving skills, I took a different route—digging into its logic. In this blog, I’ll walk you through how I found and responsibly reported three impactful business logic vulnerabilities in LeetCode’s main application - [leetcode.com](https://leetcode.com/). Each case involved clever misuse of logic, and none required complex technical exploits. Just pure understanding of how things shouldn't work.

### 🧩 Case 1: Infinite Redemptions with Limited LeetCoins

##### 📌 Description

LeetCode offers a store where users can redeem items using "leetcoins." Logically, if you have 70 leetcoins, you should only be able to redeem one item worth that amount. But due to race condition vulnerabilities, I was able to redeem multiple items with the same coin balance.

##### 💥 Exploit Summary

By intercepting the redemption request and launching it concurrently using Turbo Intruder, I was able to bypass redemption limits.

##### 🛠 Steps to Reproduce

- Start with 70 leetcoins.
- Intercept the redemption request via any proxy tools.
- Send it to Turbo Intruder with this script:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=50,
                           requestsPerConnection=50,
                           pipeline=False)

    for i in range(50):
        engine.queue(target.req, target.baseInput, gate='race1')

    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```


- Execute and observe—multiple items get redeemed.


<style>
.video-container {
  position: relative;
  padding-bottom: 56.25%; /* 16:9 aspect ratio */
  height: 0;
  overflow: hidden;
  max-width: 100%;
}

.video-container iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
<div class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/UHyi9J3xmOY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>


##### 🎯 Impact

- Bypasses monthly limits (e.g. more than 3 time-travel tickets).
- Can be extended to perks or premium subscriptions.
- Financial implications for LeetCode.

---

### 🔓 Case 2: Free Access to Premium Content

##### 📌 Description

LeetCode offers premium problems with paid access. However, using a clever frontend trick, I was able to access premium descriptions, discussions, and solutions.

##### 🛠 Steps to Reproduce

- Use a non-premium account.
- Visit any premium problem (e.g. **[Word-Squares](https://leetcode.com/problems/word-squares/)**).
- Intercept and modify the response—set all parameters to true except **`"isPaidOnly": false`**.
- To reveal the problem description: Add a note to the problem.
- Go to your notes section and enable **"Show Description."**

<style>
.video-container {
  position: relative;
  padding-bottom: 56.25%; /* 16:9 aspect ratio */
  height: 0;
  overflow: hidden;
  max-width: 100%;
}

.video-container iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
<div class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/R-p11kuCin0?si=gZABuinMSMl8mqcm" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>


##### 🎯 Impact

- Unauthorized viewing of premium content.
- Could lead to public leaks and financial damage.
- Involves only frontend manipulation—no backend auth.

---

### 🪙 Case 3: Free LeetCoins via Profile Update Race Condition

##### 📌 Description

By editing basic profile info (like name or gender) concurrently, I triggered a condition where the application granted leetcoins multiple times for a single update.

##### 🛠 Steps to Reproduce

- Create a LeetCode account.
- Go to the Edit Profile section.
- Intercept the request and send it to Turbo Intruder.
- Use this script:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=50,
                           requestsPerConnection=50,
                           pipeline=False)

    for i in range(50):
        engine.queue(target.req, target.baseInput, gate='race1')

    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

- Set **`HTTP/1.1`** to **`HTTP/1.0`** and mark **`%s`** appropriately.
- Execute and refresh the profile page —> extra coins added.


<style>
.video-container {
  position: relative;
  padding-bottom: 56.25%; /* 16:9 aspect ratio */
  height: 0;
  overflow: hidden;
  max-width: 100%;
}

.video-container iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
<div class="video-container">
<iframe width="560" height="315" src="https://www.youtube.com/embed/q2Fp18B1-Hs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>


##### 🎯 Impact

- Generation of coins beyond permitted logic.
- Can be used to buy perks or affect user rankings.
- Again, financial damage for the platform.

---

#### 🔍 Takeaways
- Business logic flaws are powerful. They exploit what the app allows—not what it protects.
- Race conditions can occur in unexpected places. Backend validation is a must.
- Always check what a user is allowed to do logically, not just technically.

---

#### TimeLine

- Cases Submission - 23-May-2024
- Reports Triaged - 28-May-2024
- Resolution - 07-Oct-2024

---

##### My Personal thought to LeetCode and Security Researchers ...
1. **For Security Researchers :** Please do not try to perform security research on leetcode infrastructure. My overall experience with this team is horrible and they offer leetcoins as a return of your investment. So don't expect anything in return.

2. **For LeetCode :** Very unprofessional - Dev/Sec Team take huge time to resolve a case (~ 6 months) , Company did not offer any LoA, Swags or anything to security researcher who helped them from a major financial loss. If I want, I can mine millions of leetcoins easily and make a lot of money by unethically selling them or redeeming any swags/goodies, well it's leetcode...!! (Very Disappointing)
