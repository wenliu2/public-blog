---
title: "Yaml Syntax"
date: 2019-03-25T09:11:07+08:00
categories:
- Technology
- Yaml
tags:
- yaml
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

### Scalars
---
Scalars are a pretty basic concept. They are the strings and numbers that make up the data on the page. A scalar could be a boolean property, like Yes, integer (number) such as 5, or a string of text, like a sentence or the title of your website.

Scalars are often called variables in programming. If you were making a list of types of animals, they would be the names given to those animals.

Most scalars are unquoted, but if you are typing a string that uses punctuation and other elements that can be confused with YAML syntax (dashes, colons, etc.) you may want to quote this data using single ' or double " quotation marks. Double quotation marks allow you to use escapings to represent ASCII and Unicode characters.

```
integer: 25
string: "25"
float: 25.0
boolean: Yes
```

### Sequences
---
Here is a simple sequence. It is a basic list with each item in the list placed in its own line with an opening dash.
```
- Cat
- Dog
- Goldfish
```

This sequence places each item in the list at the same level. If you want to create a nested sequence with items and sub-items, you can do so by placing a single space before each dash in the sub-items. YAML uses spaces, NOT tabs, for indentation. You can see an example of this below.
```
-
 - Cat
 - Dog
 - Goldfish
-
 - Python
 - Lion
 - Tiger
```

### Mappings
---