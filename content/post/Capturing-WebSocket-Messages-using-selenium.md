+++
date = "2017-07-28T09:32:45-04:00"
draft = false
highlight = true
title = "Capturing WebSocket Messages Using Selenium WebDriver"
tags = ["java", "selenium", "webdriver"]
+++

Recently, I was testing a web application which was using **[WebSocket](https://www.websocket.org//)** as a networking layer to talk to the backend server. The messages are serialized using **[Avro](https://avro.apache.org/)** and being sent over the web socket layer to the backend jetty server. 

For those who are not familiar with WebSocket messages, please go to this link in chrome "https://web-demo.adaptivecluster.com/". Once the pages is loaded, open the chrome devtool and navigate to the WS sub-tab uunder the network tab. Refer the below screenshot:

![Example image](/img/ws-messages-1.png)

The avro messages are quite complex and there was lot of logic involved to construct these messages. The UI code was written in [**Reactjs**](https://facebook.github.io/react/) and [**Typescript**](https://www.typescriptlang.org/) to build these messages and it was essential to test the correctness of these messages.

As all our UI automation was developed using Selenium, so we decided to use the same selenium code to validate these messages.

The whole problem boils down to the fact that:
##### **"How to capture WebSocket messages from your Selenium code ?"**      
                                                 
We do not want to involve any external web proxy library like BrowserMob/Fiddler to address this problem so we decided to explore if ChromeDriver can do this task without any help of external libraries.

After searching few links on google, we found this [link](https://sites.google.com/a/chromium.org/chromedriver/logging/performance-log) where it is shown, how to capture the browser network data using chromedriver.

We tried this solution and it worked. Below is the final code where I have shown how we captured both inbound/outbound messages on one of the demo site "**https://web-demo.adaptivecluster.com/**"

*The code uses Java 8 and following maven dependencies*
```xml
    <dependencies>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>3.5.3</version>
        </dependency>
        <dependency>
            <groupId>org.json</groupId>
            <artifactId>json</artifactId>
            <version>20170516</version>
        </dependency>
    </dependencies>
```

**Code:**
{{< gist sahajamit c2d6827736f2b267a8d08c41e559ad24 >}}



