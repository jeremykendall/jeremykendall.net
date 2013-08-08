---
author: "Jeremy Kendall"
comments: true
date: 2008-02-24 16:50:49+00:00
layout: post
slug: image-upload-issues-with-internet-explorer-and-imagepjpeg-2
title: Image Upload Issues with Internet Explorer and image/pjpeg
wordpress_id: 4
categories:
- Development
tags:
- ie
- jpeg
- upload
- zend framework
---

I'm working on a project that uses [Zend_Gdata](http://framework.zend.com/manual/en/zend.gdata.html), specifically [Zend_Gdata_Photos](http://framework.zend.com/manual/en/zend.gdata.photos.html), to manage photos in Google's [Picasa Web Albums](http://picasaweb.google.com/).  When the application went into its testing phase, the image upload functionality began to throw exceptions.  While I could upload photos to Picasa without issue, my business partner [John](http://www.jandbchildress.com/) couldn't upload any photos at all.  The application kept throwing the following error:
    
    Error Message: Exception Message: Expected response code 200, got 400



Since the app worked fine for me, the issue didn't seem to be with the code.  After some frustrating, unproductive troubleshooting, I finally built a file upload mockup and ran the $_FILES array through [Zend_Debug::dump()](http://framework.zend.com/manual/en/zend.debug.html).  I discovered that while Firefox reports the MIME type of a jpg as "image/jpeg", Internet Explorer reports the MIME type as "image/pjpeg."  Thanks a lot, Redmond.

As it turned out, the problem occurred when I tried to set the upload content type to "image/pjpeg":
    
    $fd = $this->_service->newMediaFileSource($photo["tmp_name"]);
    $fd->setContentType($photo["type"]);



I resolved the issue by adding a test for the pjpeg MIME type before setting the content type.  The new code looks like this:
    
    /** Resolves issue where IE changes jpg MIME type to image/pjpeg, breaking upload */
    $photo["type"] = ($photo["type"] === "image/pjpeg") ? "image/jpeg" : $photo["type"];
    $fd = $this->_service->newMediaFileSource($photo["tmp_name"]);
    $fd->setContentType($photo["type"]);



The code snippets above come from an image upload helper method that I built (borrowing heavily from [Cory Wiles](http://www.corywiles.com/)) to make working with Zend_Gdata_Photos a little easier.

If you're having similar problems uploading images, regardless of the method that you're using, check that MIME type and see if that's not the issue.
