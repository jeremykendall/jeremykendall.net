---
layout: post
title: "Detecting and Converting File Encoding"
date: 2015-05-08 09:03:01 -0500
comments: true
categories: ["General", "cli", "OSX"]
---

I had a couple of files show up in a project that weren't utf-8 encoded and
needed to be converted. In the past, I found detecting encoding and converting
from one encoding to another to be an arcane and challenging task. This morning
it only took a few tries on Google and, a few [superuser.com](https://superuser.com)
answers later, I was good to go.

## Encoding Detection

I was quickly able to determine that the CSV file in question was encoded with `utf-16le` by using the following command:

```
$ file -I unknown-encoding.csv
unknown-encoding.csv: text/plain; charset=utf-16le
```


## Converting to UTF-8

Converting the file to a new encoding was just as easy:

```
iconv -f utf-16le -t utf-8 unknown-encoding.csv > new-encoding.csv
```

## References

The commands above were sourced from the following superuser questions and accepted answers:

* [Determining the encoding of a file on Mac OS X?](http://superuser.com/questions/151972/determining-the-encoding-of-a-file-on-mac-os-x)
* [Converting the encoding of a text file (Mac OS X)](http://superuser.com/questions/151981/converting-the-encoding-of-a-text-file-mac-os-x)
