---
author: admin
comments: true
date: 2008-12-11 13:45:33+00:00
layout: post
slug: zebra-tables-with-jquery
title: Zebra Tables with jQuery
wordpress_id: 120
categories:
- Development
tags:
- javascript
- jquery
---

I've used the classic A List Apart [_Zebra Tables_](http://www.alistapart.com/articles/zebratables) technique to stripe my tables for years.  It's always worked well, and I never really considered updating the technique until last week.  I've been making heavy use of the [jQuery](http://jquery.com/) library lately, and I really disliked including another external js file whenever I wanted to stripe a table, so I thought I'd see if someone had come up with a jQuery friendly table striping technique.  It took about 10 seconds on Google to find what I was looking for, and the solution was so simple and elegant that I wanted to kick myself for not thinking of it, er, myself.

For the whole scoop, head over and [read the tutorial](http://15daysofjquery.com/examples/zebra/).  If you're like me, you want to get right to the point, so here goes.

The idea is to use jQuery to select alternate rows from your table and apply a css class to them. In the example below, the table has a class of 'striped' and jQuery adds the class 'alt' to the even rows.


    
    
    $(document).ready(function(){
        $(".striped tr:even").addClass("alt");
    });
    



Whip up a little css that adds a background color to .alt and you're done.  Not bad, huh?

The tutorial author also included an example of jQuery code that allows for a nice hover effect when you mouse over the table rows.  I wasn't as interested in that, so you'll have to [head over there](http://15daysofjquery.com/examples/zebra/) for the scoop.
