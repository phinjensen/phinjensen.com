---
title: "Getting around a Linksys router error"
author: Phineas Jensen
date: 2025-11-29
description: "Buying a cheap router—Problems accessing the administration panel—Browser debugger fun"
tags:
  - networking
  - debugging
  - journal
---

TL;DR: If you're getting a "Your router was not successfully setup" error on your Linksys router, try navigating to `http://<router_ip>/ui/dynamic/setup/change_router_password.html`.

I recently picked up a Linksys EA7300 router at a thrift store for $8, hoping to use it to extend the wifi coverage in my apartment in the way that seemed most obvious to me--running ethernet from the main Google Fiber router to this new (used) router and putting it somewhere else where it will provide a better signal to weak spots. Also, the router has 4 ethernet ports so I can use it as an ethernet switch as well!

After plugging the router into the wall, I factory reset it, connected it to the Google Fiber router with ethernet, and connected to the wireless network using the default password on the bottom of the router. I had full internet access, so the wifi extending idea was sound! When I tried to connect to the router admin page at 192.168.1.1, I was taken to the Google Fiber router's configuration page instead, which surprised me at first but makes sense. Both routers are acting as gateways and assign local IPs in their own blocks. Checking my network info, I found that it had taken a `10.*` block (with the actual gateway IP being `10.192.97.193`).

I opened that IP address in Firefox and ran into my first issue: a page telling me "Your router was not successfully setup" with some troubleshooting steps that didn't help me. I figured this might be happening because it was being assigned an IP address on a network it didn't expect, or something else related to the fact that I was daisy-chaining routers. Because it was already working as a makeshift wifi extender I knew that wasn't a real problem, so I looked for a way to get around the issue.

First, I saw that there was an error in the JavaScript console saying `can't access property "Transaction", RAINIER.jnap is undefined`. A quick internet search didn't bring anything up for that, and after looking a bit further I realized that message was coming *after* the browser was redirected to the `generic_error.html` error page, so it couldn't be the source of the error. But speaking of `generic_error.html`, if I'm being redirected to that page, then there must be some JavaScript that loads that page, right? In fact, there is. I opened up the Firefox debugger, then `http://10.192.97.193`, and hit the pause button once the page loaded. It didn't matter where execution paused because I wanted time to search the JavaScript for `generic_error.html` before I was redirected to that page (which might not load the JS file I was interested in), which was easy using Firefox's code search tool in the Debugger tab of the developer tools panel. I found only one reference to it, in this (formatted[^1]) snippet of code in the `globals.js` file:

```javascript
if (aa) {
  if (aa === 'start') {
    RAINIER.shared.util.setCookie('page', window.location, null, '/');
    RAINIER.shared.util.setCookie('errCode', '2318', null, '/');
    window.location.replace('/ui/dynamic/setup/generic_error.html')
  } else {
    if (aa === 'wifiReconnect') {
      window.location.replace('/ui/dynamic/setup/change_router_password.html')
    }
  }
  return
}
```

I created a breakpoint at the `if (aa) {` line, then continued execution. When it stopped on that line, I traversed a few frames up the call stack and found that the `aa` variable is the `setupState` field of a network description. Why that field being "start" is an error at this point, I have no idea, but I found that if I changed it to `wifiReconnect` (by entering `aa = 'wifiReconnect'` in the normal JS console), and continued execution, I was instead taken to a set-the-router-password page at `http://<router_ip>/ui/dynamic/setup/change_router_password.html`. Once I set that password, the main configuration panel loaded without a problem and I haven't seen the "Your router was not successfully setup" page since!

Now it's time to try installing OpenWRT on this thing...

[^1]: The little `{ }` button at the bottom of the JS code panel in the Debugger tab will format the code for you, and it works with line-based breakpoints!
