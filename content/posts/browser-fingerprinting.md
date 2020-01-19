+++
date = "2020-01-19T14:16:20+00:00"
title = "Browser Fingerprinting"
tags = ["browser fingerprinting", "security", "privacy"]
keywords = ["browser fingerprinting", "privacy", "digital fingerprint"]
+++
#### _With great power comes great responsibility._

Browser fingerprinting is a great power, but unfortunately is not often used responsibly. In fact, companies routinely use the technique to identify with 99% accuracy who the individual browsing their site is. This allows [tailored ads](https://support.google.com/google-ads/answer/2580383?hl=en "Google Ads Demographic ") to a specific demographic. Perhaps the best use case for browser fingerprinting comes in the form of analytics for site developers. If the developers know what devices and browsers are frequenting the site, they can improve and test for those specific browsers and screen sizes.

##### _How is browser fingerprinting done?_

The  fingerprinting is most often accomplished by viewing the userAgent string inside the browser's navigator object, however the navigator object contains other useful properties such as the device's current geolocation and number of CPU cores. To view your navigator object, open the browser console and type in **_navigator_** and you should see something like what's below:

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

From the above a few things are obvious:

1. I'm using a Macbook with v10.14.2
2. I'm running Chrome
3. I'm in the US
4. Since my hardwareConcurrency is 4, I have a quad core CPU. Therefore, I cannot be on a Macbook Air.

#####  _Burn the navigator!_

Calm down, we can't actually burn the navigator. Like I said previously, although the navigator can be used maliciously, it is still very important for developers to enable the development of the platforms we know and love. That being said, at least one team at Google is attempting to [do away with the user-agent string](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/-2JIRNMWJ7s/yHe4tQNLCgAJ "Proposal to remove user-agent"), a feature that according to Google's own [metrics](https://www.chromestatus.com/metrics/feature/timeline/popularity/2663 "Google Chrome User Agent Metrics") is used in approximately 90% of web pages.

While the feature is still a proposal, the team is looking to freeze and remove the user agent string starting in March 2020. The reasoning behind the decision being:

> The User-Agent string is an abundant source of passive fingerprinting information about our users. It contains many details about the user’s browser and device as well as many lies ("Mozilla/5.0", anyone?) that were or are needed for compatibility purposes, as servers grew reliant on bad User Agent sniffing.
> 
> 
> On top of those privacy issues, User-Agent sniffing is an abundant source of compatibility issues, in particular for minority browsers, resulting in browsers lying about themselves (generally or to specific sites), and sites (including Google properties) being broken in some browsers for no good reason.

It is important to note that the proposal would still have allow **active** browser fingerprinting. The migration to [User Agent Client Hints](https://www.google.com/url?q=https%3A%2F%2Fwicg.github.io%2Fua-client-hints%2F&sa=D&sntz=1&usg=AFQjCNGWrUbAKuA-kIHu0bnNrEN-n_BPAw "User Agent Client Hints") would result in all the information being sent to a server if only if the server requests the information, whereas now the site can retrieve the information without the browser/ user being aware.

#####  _What can I do?_

Unfortunately you cannot do terribly much at this point of time to avoid device fingerprinting other than disabling JavaScript or start using [Firefox](https://blog.mozilla.org/firefox/how-to-block-fingerprinting-with-firefox/ "Firefox avoids browser fingerprinting") . While I would highly recommend using Firefox, I would not recommend disabling JavaScript unless you want half of the internet to be unusable in any meaningful way.

##### _Alternative Methods_

It's also important to note that there are other ways to determine a user's browser and device. For example, you could guess the device type by checking the _window.height_ and _window.width_ to determine if the windows is larger than a phone or a tablet and categorize accordingly. Additionally, you can determine the user's browser by checking for implementation differences in various browser APIs. Without going into too much detail, I've pasted a rough example of how to do just that below the Further Reading section.

##### Nothing is perfect

While I'm not overly thrilled with the lack of privacy options provided by default by browsers other than Firefox, I understand why things are the way they are. Moving forward, I'd love to see the Chromium project and Safari continue to provide a more private browsing experience.

###### Further Reading:

* [https://blog.mozilla.org/internetcitizen/2018/07/26/this-is-your-digital-fingerprint/](https://blog.mozilla.org/internetcitizen/2018/07/26/this-is-your-digital-fingerprint/ "Your Digital Fingerprint")

###### Code Snippet

    let isPrivate = false;
    
    /**
     * Gets the browser name or returns an empty string if unknown.
     * This function also caches the result to provide for any
     * future calls this function has.
     *
     * @returns {string}
     */
    var browser = function() {
      // Return cached result if avalible, else get result then cache it.
      if (browser.prototype._cachedResult) return browser.prototype._cachedResult;
    
      // Opera 8.0+
      var isOpera =
        (!!window.opr && !!opr.addons) ||
        !!window.opera ||
        navigator.userAgent.indexOf(" OPR/") >= 0;
    
      // Firefox 1.0+
      var isFirefox = typeof InstallTrigger !== "undefined";
    
      // Safari 3.0+ "[object HTMLElementConstructor]"
      var isSafari =
        /constructor/i.test(window.HTMLElement) ||
        (function(p) {
          return p.toString() === "[object SafariRemoteNotification]";
        })(!window["safari"] || safari.pushNotification);
    
      // Internet Explorer 6-11
      var isIE = /*@cc_on!@*/ false || !!document.documentMode;
    
      // Edge 20+
      var isEdge = !isIE && !!window.StyleMedia;
    
      // Chrome 1+
      var isChrome = !!window.chrome && !!window.chrome.webstore;
    
      // Blink engine detection
      var isBlink = (isChrome || isOpera) && !!window.CSS;
    
      return (browser.prototype._cachedResult = isOpera
        ? "Opera"
        : isFirefox
        ? "Firefox"
        : isSafari
        ? "Safari"
        : isChrome
        ? "Chrome"
        : isIE
        ? "IE"
        : isEdge
        ? "Edge"
        : isBlink
        ? "Blink"
        : "Don't know");
    };
    
    const replacePrivateText = () => {
      document.body.innerHTML = document.body.innerHTML.replace(
        "hello",
        "hello private"
      );
    };
    
    console.log(browser());
    switch (browser()) {
      case "Firefox":
        var db = indexedDB.open("test");
        db.onerror = function() {
          replacePrivateText();
        };
        break;
      case "IE":
        if (!window.indexedDB && (window.PointerEvent || window.MSPointerEvent)) {
          replacePrivateText();
        }
        break;
      case "Edge": // not chromium based edge
        if (!window.indexedDB && (window.PointerEvent || window.MSPointerEvent)) {
          replacePrivateText();
        }
      case "Don't know":
      case "Chrome":
        const fs = window.RequestFileSystem || window.webkitRequestFileSystem;
        if (!fs) {
          replacePrivateText();
          break;
        } // old way to check assuming
        // chrome 76 + https://ww.9to5google.com/2019/08/09/new-york-times-detect-incognito-chrome-76/
        //https://arstechnica.com/information-technology/2019/07/chrome-76-prevents-nyt-and-other-news-sites-from-detecting-incognito-mode/
        //https://mishravikas.com/articles/2019-07/bypassing-anti-incognito-detection-google-chrome.html
        if ("storage" in navigator && "estimate" in navigator.storage) {
          navigator.storage.estimate().then(({ usage, quota }) => {
            console.log(`Using ${usage} out of ${quota} bytes.`);
            if (quota < 120000000) {
              replacePrivateText();
            } else {
              console.log("Not Incognito");
            }
          });
        } else {
          console.log("Can not detect");
        }
    }
    
    // honorable mention
    // Jesse Li timing attack: https://blog.jse.li/posts/chrome-76-incognito-filesystem-timing/