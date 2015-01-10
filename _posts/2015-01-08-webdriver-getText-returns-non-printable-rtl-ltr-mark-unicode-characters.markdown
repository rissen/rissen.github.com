---
layout: posts
title: "Workaround for WebElement.getText() returns non-printable(RTL/LTR mark) Unicode characters"
category: testng
tags: [webdriver]
---

# Workaround for WebElement.getText() returns non-printable(RTL/LTR mark) Unicode characters

This issue occurs on Chrome webdriver, Firefox webdriver works well.

## workaround
Replace all RTL/LTR characters with an empty character("").

Java code snippet:

```java
//&lrm; Java source code
String lrm = "\u200e";
//&rlm; Java source code
String rlm = "\u200f" ;
wedriver.get( "http://127.0.0.1:8080/" );
String rtlText = wedriver.findElement(By.id("rtl")).getText();
System.out.println("text:" + rtlText + ":");
System.out.println("length:" + rtlText.length());
System.out.println("length:" + rtlText.replaceAll(lrm, "").replaceAll(rlm, "").length());
```

html:

```html
<span id="rtl">Hello&rlm; World&lrm;</span>
```

output:
```
text:Hello‏ World‎:
length:13
length:11
```

## References
[W3C H34](http://www.w3.org/TR/WCAG20-TECHS/H34.html)
[LRM unicode character](http://www.fileformat.info/info/unicode/char/200e/index.htm)
[RLM unicode character](http://www.fileformat.info/info/unicode/char/200f/index.htm)
[webdriver issue link](https://code.google.com/p/selenium/issues/detail?id=4473)

