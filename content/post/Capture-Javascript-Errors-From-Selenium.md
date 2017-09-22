+++
date = "2015-02-12T09:32:45-04:00"
draft = false
highlight = true
title = "HOW TO CAPTURE JAVASCRIPT ERRORS FROM YOUR WEB APPLICATION"
tags = ["selenium", "javascript"]
+++

This article is also available [here](https://www.3pillarglobal.com/insights/how-to-capture-javascript-errors-from-your-web-application)

Recently, in one of my projects I was asked to come up with an automated way to capture all the JavaScript exceptions from the browser for one of our Web Applications. It is a known fact that most of the modern web applications in use today involve an extensive use of JavaScript for the client side processing which makes it all the more important to capture all JavaScript exceptions and consider each Java Script exception as a bug that needs to be fixed, irrespective of whether it is currently affecting the website functionality or the user experience.

There are many browser based tools which developers rely on to find and debug JavaScript errors like [Chrome’s DevTool](https://developers.google.com/chrome-developer-tools/), [Firebug](http://getfirebug.com/) , [IE Developer toolbar](https://blogs.msdn.microsoft.com/ie/2007/01/09/ie-developer-toolbar-beta-3-now-available/) etc. but still there exists a need of automated logging of these errors to a centralized server. To meet this requirement there are few paid solutions available today like ([errorception](http://errorception.com/), jslogger and [muscular](http://www.muscula.com/)) which requires inserting a script tag on every web page and then it will start recording all the Java Script errors and also log them to a centralized location in real-time.

Till now we have addressed this problem from a developer’s perspective but being a tester I would also like to discuss few approaches to capture these JavaScript errors in automated tests using [Selenium WebDriver](http://code.google.com/p/selenium/). Webdriver does not support this feature and it is an [issue](https://code.google.com/p/selenium/issues/detail?id=148) open since 2009 and well documented on their Google group. Different people have come up with different approaches to tackle this problem and I would like to discuss each approach one by one.

# APPROACH 1: INJECTING A SCRIPT ON EACH PAGE TO CATCH ALL UNDETECTED JAVASCRIPT ERRORS. #

First we need to add a script like the following on every page

Now these errors can be extracted from your Webdriver code like this:
```java
((JavascriptExecutor) driver).executeScript("return window.javascript_errors ");
```

I believe this approach is good but there are still a few drawbacks worth mentioning here. First, the JavaScript error won’t necessarily occur in the currently active window so we need to go through all the iFrames and windows to check the errors. Second problem is that it is not easy to convince your developers and management team to add this extra script in your webpages to make it testable. To overcome the second issue we can use a proxy like [Browsermob](https://github.com/lightbody/browsermob-proxy) or [Fiddler Core](http://fiddler2.com/fiddlercore) to make sure our script gets injected in to the page on time to catch the onload event.

# APPROACH 2: USING A FIREFOX EXTENSION TO CAPTURE THE ERRORS. #

You can create your own Firefox extension as Firefox allows extensions to get notified of JavaScript errors, no matter where they occur, then you can extract these errors from your Webdriver code. Marc Guillemot has created a similar extension and more details can be found on [here](https://github.com/mguillem/JSErrorCollector).

Here is the simple Java example to use this extension to extract the errors.

```java
    FirefoxProfile ffProfile = new FirefoxProfile();
    JavaScriptError.addExtension(ffProfile);
    final WebDriver driver = new FirefoxDriver(ffProfile);
 
    driver.get("http://somesite");
 
    final List jsErrors = JavaScriptError.readErrors(driver);
    assertTrue(jsErrors.toString(), jsErrors.isEmpty());
```

# APPROACH 3: USING CHROMEDRIVER2 (WEBDRIVER FOR CHROME) #

In the GTAC 2013, Ken Kania from Google has explained how ChromeDriver2 is different from its previous version as it uses Chrome DevTools API and now it provides access to internals of Web App and we can do things like:

* Query and manipulate DOM tree
* Debug JavaScript and extracting JavaScript errors
* Memory and Performance Profiling
* Network Traffic Monitoring

Detailed slides of this presentation can be found [here](https://docs.google.com/presentation/d/14xU207jL1aepO_8-WrkEcSIPUscUjak5jtxMaUoMegg/pub?start=false&loop=false&delayms=3000#slide=id.gbe6ac7bc_0185).

I would like to recommend this approach as it is very simple, robust and does not require any tweaking or workaround like the earlier approaches. Here I am posting the Java code to extract the JavaScript errors using ChromeDriver2

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.logging.LogEntries;
import org.openqa.selenium.logging.LogEntry;
import org.openqa.selenium.logging.LogType;
import org.openqa.selenium.logging.LoggingPreferences;
import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
 
public class JSErrorsLogging {
private WebDriver driver;
    @BeforeMethod
    public void setUp() {
        System.setProperty("webdriver.chrome.driver", "{path to your chromedriver.exe}");        
        DesiredCapabilities capabilities = DesiredCapabilities.chrome();
        LoggingPreferences loggingprefs = new LoggingPreferences();
        loggingprefs.enable(LogType.BROWSER, Level.ALL);
        capabilities.setCapability(CapabilityType.LOGGING_PREFS, loggingprefs);
        driver = new ChromeDriver(capabilities);
    }
 
    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
    public void ExtractJSLogs() {
        LogEntries logEntries = driver.manage().logs().get(LogType.BROWSER);
        for (LogEntry entry : logEntries) {
            System.out.println(new Date(entry.getTimestamp()) + " " + entry.getLevel() + " " + entry.getMessage());
        }
    }
    @Test
    public void testMethod() {
        driver.get("http://yourwebsite.com");
        ExtractJSLogs();
    }
}
```

This approach can only be used in those languages where the Logger interface has been implemented for WebDriver. We cannot use C# for this approach as according to this link, Logger interface is not yet implemented in C#.

I hope what I have shared with you through this blog will help you address the issue of how to capture JavaScript errors from your web application(s). Please do leave your comments in the box below.



