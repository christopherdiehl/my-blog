+++
date = "1/16/2020"
draft = true
title = "Browser Fingerprinting"

+++

Browser fingerprinting is most often done via the userAgent string inside the browser's navigator object. To view your navigator object, open the browser console and type in **_navigator_**

    Navigator {vendorSub: "", productSub: "20030107", vendor: "Google Inc.", maxTouchPoints: 0, hardwareConcurrency: 4, …}
    vendorSub: ""
    productSub: "20030107"
    vendor: "Google Inc."
    maxTouchPoints: 0
    hardwareConcurrency: 4
    cookieEnabled: true
    appCodeName: "Mozilla"
    appName: "Netscape"
    appVersion: "5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36"
    platform: "MacIntel"
    product: "Gecko"
    userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36"
    language: "en-US"
    languages: (3) ["en-US", "en", "la"]
    ...