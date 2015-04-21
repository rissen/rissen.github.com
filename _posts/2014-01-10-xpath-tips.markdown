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

## Contains a whole word, class attribute conatians a className1
```
//div[contains(concat(' ',normalize-space(@class), ' '),' className1 ')]
```

## References
[XPath Functions](http://www.w3schools.com/xpath/xpath_functions.asp)


