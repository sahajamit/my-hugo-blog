+++
date = "2017-11-10T09:32:45-04:00"
draft = false
highlight = true
title = "Adding the power of Chrome Devtools in your Selenium tests."
tags = ["java", "selenium", "webdriver", "websocket", "chromedriver", "chrome"]
+++

I was just browsing the Twitter and came accross a tweet mentioning that Chrome 63 is coming with  **[Multi-client remote debugging support](https://developers.google.com/web/updates/2017/10/devtools-release-notes#multi-client)**

[comment]: <> ({{< tweet 922874617424879616 >}})


This news made me little excited as it opens up hell lot of possibilities for people like me who works in Test Automation. Just to give you all some back ground on how chromedriver server controls the chrome when we send our WebDriver commands in our Selenium tests. 

Till chrome 57 version, Chromedriver used to use an automation extension which gets injected in to your chrome and it helps to facilitate the communication between the chromedriver and the chrome.

But from Chrome 58 onwards they have removed the use of this extension and now everything is controlled by Chrome's dev tool api which uses WebSocket communication and also supports remote debugging. With this change chromedriver team has adviced to use the following chrome option to avoid any errors.

```java
options.setExperimentalOption("useAutomationExtension",automationExtension);
```

Let's go little deeper in to this Chrome's [remote debugging protocol](https://chromedevtools.github.io/devtools-protocol/)