+++
date = "2017-07-28T09:32:45-04:00"
draft = false
highlight = true
title = "Mystery of StaleElementReferenceException in Selenium WebDriver"
tags = ["blog", "selenium"]
+++

If you are a Selenium developer than you would have surely faced this mysterious exception called "**StaleElementReferenceException**"

Why exactly it occurs? This has been my favorite interview question since last many years and most of the time candidates gets confused it with NoSuchElementException. In case you have never worked on a dynamic ajax based application then there could be a chance that you have never faced it.

Let's go little deeper and unveils the mystery behind it. When we run a simple code like this:

```java
WebElement searchBox = driver.findElement(By.cssSelector("input[name='q'"));
searchBox.sendKeys("Selenium");
```

When WebDriver executes the above code then it assigns an internal id (refer the below image) to every web element and it refers this id to interact with that element.

![Example image](/img/stale-element-1.png)

Now, let's assume when you have fetched the element and before doing any action on it, something got refreshed on your page. It could be the entire page refresh or some internal ajax call which has refreshed a section of the dom where your element falls. In this scenario the internal id which webdriver was using has become stale, so now for every operation on this WebElement, we will get StaleElementReferenceException.

To overcome this problem, the only choice we have is to re-fetch the element from Dom and this time WebDriver will assign a different Id to this element.

So from the above example what we understood is that if we are working with an AJAX-heavy application where page's dom can get changed on every interaction then it is wise to fetch the web elements every time when we are operating on them. There are couple of ways to make sure, the element always gets refreshed before we use it:

Page Factory Design Pattern:
---------------------------



 Please refer the below code.

 ```java
GoogleSearchPage page = PageFactory.initElements(driver,GoogleSearchPage.class);
```

```java
public class GoogleSearchPage {
    @FindBy(how = How.NAME, using = "q")
    private WebElement searchBox;

    public void searchFor(String text) {
        searchBox.sendKeys(text);
    }
```
In the above example, a proxy would be configured for every Web Element when the page gets initialised. Every time we use a WebElement it will go and find it again so we shouldn't see StaleElementException. This approach would solve your stale element problem at most of the places except some corner cases which I will cover in the next approach.

Refreshing the Element whenever it gets stale:
----------------------------------------------

When you work on a modern, reactive, real-time application developed in technologies like Angularjs/Reactjs which has hell lot of data and there is a persistent web-socket connection which keep pushing data to your browser and which makes your dom to change.  Let's take an example of a stock exchange where there a data grid which displays real-time information and data keeps changing too frequently. In this case, whenever the data gets changed at server-side, the changes will be pushed automatically to your UI grid and depending on your data your respective rows or cells will get stale.

Here, Page factory can not help as most of your grid elements are dynamic and you cannot configure their locators while initiliazing your page. Also if you have created your own data model to prase data, than it is difficult to configure Page Factory accross all your data model classes.

To deal with this problem I decided to develop a generic method to refresh the element in case it gets stale. To refresh an element, we first need to figure out its By locator but Selenium API has not exposed anything to re-construct the locator from an existing web element. I was fortunate that they have exposed a toString method on WebElement which print all the locators being used to build that element. Let's see the below example where we are finding an element which is a child of another element:

```java
WebElement elem1 = driver.findElement(By.xpath("//div[@id='searchform']"));
WebElement elem2 = elem1.findElement(By.cssSelector("input[name='q'"));
System.out.println(elem2.toString());
```

**Output of the above code would be:**
```
[[[[ChromeDriver: chrome on XP (bd6a0d83229c67d5f7e6060b1bd768e9)] -> xpath: //div[@id='searchform']]] -> css selector: input[name='q']
```

Now we have to apply all the reverse-engineering to build the element again from this String. Thanks to [Reflection API in Java](https://docs.oracle.com/javase/tutorial/reflect/) which can help us to dynamically execute the code to build the element.

**Here is the final implementation:**

```java
WebElement refreshedElement = StaleElementUtils.refreshElement(elem2);
```

This refreshElement method will check if the element is stale then it will re-fetch the element from the dom. So for all the data grid elements which can get stale anytime, we can use this method as a precautionary measure to avoid stale element exception.

Please feel free to share your thoughts on my approach and would love to know, how you have handled this interesting exception.

Here is the complete [code](https://gist.github.com/sahajamit/f94d0094f35cd63deb21c45328ab5b47):



```java
package com.sahajamit.selenium.utils;

import com.sahajamit.selenium.driver.DriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.StaleElementReferenceException;
import org.openqa.selenium.WebElement;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;

public class StaleElementUtils {
    private static final Logger LOGGER = LoggerFactory.getLogger(UIUtils.class);
    public static WebElement refreshElement(WebElement elem){
        if(!isElementStale(elem))
            return elem;
        Object lastObject = null;
        try{
            String[] arr = elem.toString().split("->");
            List<String> newStr = new ArrayList<String>();
            for(String s:arr){
                String newstr = s.trim().replaceAll("^\\[+", "").replaceAll("\\]+$","");
                String[] parts = newstr.split(": ");
                String key = parts[0];
                String value = parts[1];
                int leftBracketsCount = value.length() - value.replace("[", "").length();
                int rightBracketscount = value.length() - value.replace("]", "").length();
                if(leftBracketsCount-rightBracketscount==1)
                    value = value + "]";
                if(lastObject==null){
                    lastObject = DriverManager.getDriver();
                }else{
                    lastObject = getWebElement(lastObject, key, value);
                }
            }
        }catch(Exception e){
            LOGGER.error("Error in Refreshing the stale Element.");
        }
        return (WebElement)lastObject;
    }

    public static boolean isElementStale(WebElement e){
        try{
            e.isDisplayed();
            return false;
        }catch(StaleElementReferenceException ex){
            return true;
        }
    }

    private static WebElement getWebElement(Object lastObject, String key, String value){
        WebElement element = null;
        try {
            By by = getBy(key,value);
            Method m = getCaseInsensitiveDeclaredMethod(lastObject,"findElement");
            element = (WebElement) m.invoke(lastObject,by);
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return element;
    }

    private static By getBy(String key, String value) throws InvocationTargetException, IllegalAccessException {
        By by = null;
        Class clazz = By.class;
        String methodName = key.replace(" ","");
        Method m = getCaseInsensitiveStaticDeclaredMethod(clazz,methodName);
        return (By) m.invoke(null,value);
    }
    private static Method getCaseInsensitiveDeclaredMethod(Object obj, String methodName) {
        Method[] methods = obj.getClass().getMethods();
        Method method = null;
        for (Method m : methods) {
            if (m.getName().equalsIgnoreCase(methodName)) {
                method = m;
                break;
            }
        }
        if (method == null) {
            throw new IllegalStateException(String.format("%s Method name is not found for this Class %s", methodName, obj.getClass().toString()));
        }
        return method;
    }
    private static Method getCaseInsensitiveStaticDeclaredMethod(Class clazz, String methodName) {
        Method[] methods = clazz.getMethods();
        Method method = null;
        for (Method m : methods) {
            if (m.getName().equalsIgnoreCase(methodName)) {
                method = m;
                break;
            }
        }
        if (method == null) {
            throw new IllegalStateException(String.format("%s Method name is not found for this Class %s", methodName, clazz.toString()));
        }
        return method;
    }
}
```
