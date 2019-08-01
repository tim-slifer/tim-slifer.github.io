---
layout: post
title: Easy CSS Selectors by Injecting Sizzle
author: Tim Slifer
excerpt_separator: <!--more-->
---

Years back, I came across a very cool blog post that detailed the how and why behind injecting Sizzle in WebDriver 
tests. The whole idea came as a silver bullet solution for me at the time, and remains a prominent part of my UI 
modeling approach even today. While there are a number of similar approaches posted on the web, that specific post has
either been taken down or edited beyond my recognition. The lessons I've learned from it, however, have lived on.

<!--more-->

I was at a point where my dislike for XPath was already growing, having seen the versatility and the ease-of-use of CSS 
Selectors. Unfortunately, I had not yet discovered a simple way to parameterize my CSS Selectors to the point where I 
could leave XPath behind entirely.

> A quick word of caution before we get started. This approach will entail performing actions that will alter the DOM 
of the application under test. The extent of this change is limited to adding a single script, if it hasn't already 
been included by the application developers. I've employed this approach for over 7 years now on many different 
applications, and have yet to see an adverse reaction to system testability. Your mileage, of course, may vary.

Back in the Selenium-RC days, CSS Selectors were powered by Sizzle. When WebDriver (Selenium 2) came along, the 
decision was made to rely upon the native CSS engine in the browser under test. In order to restore the functionality 
and versatility of Sizzle selectors, we now must inject Sizzle during our test.

> Selenium 4 is currently in alpha, and may have some changes that could affect the approach I define below. Once the
dust settles and Selenium 4 is upon us as a final release, I will follow up to this post with the latest.

# Overview

We're going to be working with one primary component within the Selenium library, the `By` class. The `By` class, as 
you surely know, is how we tell WebDriver how we will be locating an element, be it by ID, CSS, XPath, etc. Since 
we're going to introduce a new means of element selection, we'll be extending `By` and adding our own class that will
do the heavy lifting for us.

In the end, we will be able to initialize a `WebElement` using our new selector type and Sizzle CSS expressions.

```java
driver.findElement(BySizzle.css(".btn:contains('selenium')"));
```

# Importing Sizzle

For this implementation, we'll be obtaining Sizzle by importing the Webjars/Sizzle project as a dependency.

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>sizzle</artifactId>
    <version>2.3.4</version>
</dependency>
```

> [Webjars](https://www.webjars.org/) is a super-useful project that packages client-side web libraries into JAR files.

# Extending the By Class to Enable Sizzle

The first task is to extend `By`, which will allow us to call our Sizzle implementation. Since we're working in the 
context of Sizzle selectors, we can call this class `BySizzle`.

```java
public abstract class BySizzle extends By {
    
    public static By css(final String selector) {
        if (selector == null) {
            throw new IllegalArgumentException("The selector cannot be null.");
        }
        return new BySizzleCssSelector(selector);
    }
}
```

> At this point, the approach I had learned would extend the `ByCssSelector` class in Selenium and override the 
existing CSS Selector logic to utilize a failover design. In short, the implementation would attempt to use a 
standard CSS Selector first. If this failed, then a Sizzle CSS Selector was attempted. As I recall, the reasoning 
behind this was a very nominal performance advantage of normal CSS Selectors to Sizzle. 

> Since Sizzle CSS is a superset of CSS Selectors, we can pass standard CSS Selectors to Sizzle and not have to worry 
about any failover logic. This, along with my tendency to stay consistent with existing patterns, led me to choose a 
different direction which follows directly the pattern in place for the various locator type methods. 

# Adding a New Static Class for Sizzle Selectors

In reviewing the `By` class, we can observe a pattern amongst the various locator types. First, are the methods with 
which we are already very familiar. These will include `By.id()`, `By.cssSelector()`, `By.xPath()`, etc. In these 
methods, a static class is then returned which handles generating the WebElements using their given approach. 

Our second task is to add our own static class for Sizzle CSS on our new `BySizzle` class.

```java
public static class BySizzleCssSelector extends By implements Serializable {
    
    private String selector;
    
    public BySizzleCssSelector(String selector) {
        this.selector = selector;
    }
}
```

Our next step is to build the functionality that we'll call from the `findElement()` and `findElements()` methods. 
Since these steps are common to both, we'll add `private` methods to call.

First, we need to access the `WebDriver` instance from the `SearchContext`. 

```java
private WebDriver getWebDriver(SearchContext context) {
    if (context instanceof WebDriver) {
        return (WebDriver) context;
    }
    
    if (context instanceof WrapsDriver) {
        return ((WrapsDriver) context).getWrappedDriver();
    }
    
    throw new IllegalStateException("WebDriver instance not found on SearchContext.");
}
```

Second, we need to poll the DOM and check whether or not Sizzle is present.

```java
private void injectSizzleIfNeeded(WebDriver driver) {
    if (!isSizzleLoaded(driver)) {
        injectSizzle(driver);
    }
}

private Boolean isSizzleLoaded(WebDriver driver) {
    try {
        return (Boolean) ((JavascriptExecutor) driver).executeScript("return Sizzle() != null");
    }
    catch (WebDriverException e) {
        return false;
    }
}
```

If Sizzle is not found, we'll then inject the script.

```java
private void injectSizzle(WebDriver driver) {
    InputStream path =
            getClass().getResourceAsStream(
                    "/META-INF/resources/webjars/sizzle/2.4.3/sizzle.js");
    String sizzleLoader = readFile(path);
    ((JavascriptExecutor) driver).executeScript(
            "if(typeof define !== 'undefined') {" +
                    "var oldDefine = define; " +
                    "define = undefined;" + sizzleLoader +
                    "window.define = oldDefine;" +
                    "}" +
                    "else{" + sizzleLoader + "}");
}

private String readFile(InputStream path) {
    try {
        Reader reader = new BufferedReader(new InputStreamReader(path));
        StringBuilder builder = new StringBuilder();
        char[] buffer = new char[8192];
        int read;
        while ((read = reader.read(buffer, 0, buffer.length)) > 0) {
            builder.append(buffer, 0, read);
        }
        reader.close();
        return builder.toString();
    }
    catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
```

> Since we have Sizzle as a dependency, this implementation will inject the full script onto the DOM. This approach 
is intended as a more portable solution that can work anywhere. In a more controlled environment, where the Sizzle.js
file can be hosted from a static location, the injection could be changed to simply append a `<script>` node onto the
DOM.

> The javascript statement above works with a function called `define`. This was a workaround to a constraint I've 
encountered with applications that include RequireJS. The existing `define` is "set aside" while the injection 
occurs, then is restored after.

Finally, we'll search the DOM using the selector expression.

```java
private List runSizzle(WebDriver driver) {
    return (List) ((JavascriptExecutor) driver).executeScript("return Sizzle(\"" + selector + "\")");
}
```

Now, we can override a few methods on the `By` class to complete our implementation.

To find the first instance of an element:

```java
@Override
public WebElement findElement(SearchContext context) {
    WebDriver driver = getWebDriver(context);
    injectSizzleIfNeeded(driver);
    List elements = runSizzle(driver);
    
    if (elements.size() > 0) {
        return (WebElement) elements.get(0);
    }
    else {
        throw new NoSuchElementException("Unable to locate element: [" + selector + "]");
    }
}
```

To find all instances of the matching elements:

```java
@Override
@SuppressWarnings ({"unchecked"})
public List<RemoteWebElement> findElements(SearchContext context) {
    WebDriver driver = getWebDriver(context);
    injectSizzleIfNeeded(driver);
    
    return runSizzle(driver);
}
```

> I'm no fan of suppressing warnings. This one is required since we're casting the return value of a Javascript 
expression into a Java data type.

And finally, `toString()`:

```java
@Override
public String toString() {
    return "BySizzle.css: [" + selector + "]";
}
```

# Get Started

Now that Sizzle can be injected onto the DOM, we are now able to make use of much more flexible selector expressions.
There are a number of excellent sources of information on crafting CSS Selectors both with and without Sizzle. I 
prefer the 
[Rosetta Stone](https://www.red-gate.com/simple-talk/dotnet/.net-framework/xpath,-css,-dom-and-selenium-the-rosetta-stone/) 
wall chart by [Michael Sorens](https://github.com/msorens). This cheat-sheet is a fantastic reference for the many 
various structures available to build both XPath (if that's your cup of tea) and CSS Selectors. What I found 
especially handy is how the author has labeled the CSS Selectors that are not natively supported by Selenium, but are
available via Sizzle.

# Need a Library?

I've open sourced this project on [GitHub](https://github.com/tim-slifer/webdriver-sizzle-injector), and published to 
Maven Central. The final implementation differs just a bit from this guide, but the overall intent and use case is
the same. The WebDriver-Sizzle-Injector project is released under the 
[MIT License](https://opensource.org/licenses/MIT).
