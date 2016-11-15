---
layout: post
title: Encode URL Data
tags: 
- 编码



---

# 编码URL数据

URL编码字符串，使用Core Foundation 函数的`CFURLCreateStringByAddingPercentEscapes`和`CFURLCreateStringByReplacingPercentEscapesUsingEncoding`。这些函数允许编码您指定的字符列表，除了高ASCII编码（`0x80`- `0xff`）和非打印字符。

根据[RFC 3986](http://tools.ietf.org/html/rfc3986)，在URL中的保留字符有：

```
reserved    = gen-delims / sub-delims
 
gen-delims  = ":" / "/" / "?" / "#" / "[" / "]" / "@"
 
sub-delims  = "!" / "$" / "&" / "'" / "(" / ")"
                  / "*" / "+" / "," / ";" / "="
```



因此，要正确URL编码一个UTF-8字符串时，你应该做到以下几点：

```
CFStringRef originalString = ...
 
CFStringRef encodedString = CFURLCreateStringByAddingPercentEscapes(
    kCFAllocatorDefault,
    originalString,
    NULL,
    CFSTR(":/?#[]@!$&'()*+,;="),
    kCFStringEncodingUTF8);
```



如果要解码一个URL片段，您必须首先将URL字符串分割成几个组成部分（领域和路径的部分）。如果不对其进行解码，您将无法分清其中差异，例如，一个编码了的&符号（原来是一个字段的内容的一部分）和一个光秃秃的＆符号（表明领域的结束）。

你已经打破了URL成各个片段，您可以按以下各部分进行解码：

```
CFStringRef decodedString = CFURLCreateStringByReplacingPercentEscapesUsingEncoding(
    kCFAllocatorDefault,
    encodedString,
    CFSTR(""),
    kCFStringEncodingUTF8);
```

要点：虽然NSString类提供了用于添加百分比编码方法，但不应该常常使用它们。 这些方法假定您传递的字符串包含一系列＆字符分隔的值，因此，您不能对包含＆符的字符串进行正确的URL编码。 如果你尝试使用这些方法做某事，你的代码可能容易受到URL字符串注入攻击（安全漏洞），这取决于另一端的软件如何处理格式不正确的URL。