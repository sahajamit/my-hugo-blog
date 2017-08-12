+++
date = "2015-06-28T09:32:45-04:00"
draft = false
highlight = true
title = "Mobile Emulation In Chromedriver"
tags = ["blog", "selenium"]
+++

ChromeDriver 2.11 release has the mobile emulation support using Chrome Dev Tools. I have been waiting for this feature since long.

This feature is really handy to run your selenium tests against different mobile screen sizes with minimal setup. This does not provide you 100% emulation but atlest it will give you some fair idea about the compatibility of your application with different mobile screen size.

This is how we can emulate different devices using the Java code.

```java
import java.util.HashMap;
import java.util.Map;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.remote.DesiredCapabilities;

public class MobileEmulation {
 static DesiredCapabilities  capabilities;
 static String DeviceName;
 public static void main(String[] args) {
  //some Sample Devices. Complete list can be found here: https://code.google.com/p/chromium/codesearch#chromium/src/chrome/test/chromedriver/chrome/mobile_device_list.cc
  //pick any of the device

//  DeviceName = "Google Nexus 5";
//  DeviceName = "Samsung Galaxy S4";
//  DeviceName = "Samsung Galaxy Note 3";
//  DeviceName = "Samsung Galaxy Note II";
//  DeviceName = "Apple iPhone 4";
  DeviceName = "Apple iPhone 5";
//  DeviceName = "Apple iPad 3 / 4";

  String ChromeDriverPath = System.getProperty("user.dir") + "/lib/chromedriver.exe";
  System.setProperty("webdriver.chrome.driver", ChromeDriverPath);

  Map<String, String> mobileEmulation = new HashMap<String, String>();
  mobileEmulation.put("deviceName", DeviceName);

  Map<String, Object> chromeOptions = new HashMap<String, Object>();
  chromeOptions.put("mobileEmulation", mobileEmulation);

  capabilities = DesiredCapabilities.chrome();
  capabilities.setCapability(ChromeOptions.CAPABILITY, chromeOptions);
  WebDriver driver = new ChromeDriver(capabilities);

  driver.manage().window().maximize();
  driver.navigate().to("http://www.google.com");
 }
}
```
