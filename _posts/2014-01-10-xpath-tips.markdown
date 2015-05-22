---
layout: posts
title: "XPath Tips"
description: "XPath Tips"
category: testng
tags: [xpath]
---

# XPath Tips

## Remove whitespace, normalize-space()

```
//span[normalize-space(.)='text a']
```

## Locate table cell by column header name, count()

```
//table/tbody/tr/td[count(//table/thead/tr/th[.="column header"]/preceding-sibling::th)+1]
```

## Contains a whole word, class attribute conatians className1
```
//div[contains(concat(' ',normalize-space(@class), ' '),' className1 ')]
```

## Selecting siblings between two nodes using XPath
```
Selecting siblings between <li>B</li> and <li>F</li>
//li[.='B']/following-sibling::li[.='F']/preceding-sibling::li[preceding-sibling::li[.='B']]
```
```
<ul>
<li>A</li>
<li>B</li>
<li>C</li>
<li>D</li>
<li>E</li>
<li>F</li>
</ul>
```
## References
[XPath Functions](http://www.w3schools.com/xpath/xpath_functions.asp)


