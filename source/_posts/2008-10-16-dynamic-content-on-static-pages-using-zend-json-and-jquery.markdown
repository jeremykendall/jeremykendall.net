---
author: admin
comments: true
date: 2008-10-16 20:48:17+00:00
layout: post
slug: dynamic-content-on-static-pages-using-zend-json-and-jquery
title: Dynamic Content on Static Pages using Zend Json and jQuery
wordpress_id: 59
categories:
- Development
tags:
- ajax
- javascript
- jquery
- json
- xml
- zend framework
---

**The Problem**

One of the most highly trafficked sites on the intranet where I work is the cafeteria's web site.  Employees from all across the enterprise visit daily to see what's on the menu.  Maintaining the menu pages on the cafeteria site has been a constant challenge for our content folks.

As the bulk of our intranet is still mostly static (PHP isn't even installed on our main intranet box), our content people have to work really hard to keep the menu up-to-date.  The process consists of the cafeteria folks emailing an updated menu over to our content people and our content folks manually updating the menu's html.  Since the cafeteria is open seven days a week, that means our content folks have to manage eight separate pages: the cafeteria homepage featuring the current day's menu in a side bar and separate menu pages for each day of the week.

**The Solution, Almost**

The solution to this issue began with a simple Java CRUD app, allowing the cafeteria employees to manage the menu themselves.  The "Menu Builder" application spits out an XML document for consumption on the cafeteria website.  During the development process, I extended [Zend_Http_Client](http://framework.zend.com/manual/en/zend.http.html) and wrote a simple index page to parse the XML for display.  While the PHP solution for displaying the menu was nice, clean, and simple, it turned out to be completely useless.

As I mentioned above, the majority of our intranet is still running on a server without PHP installed.  We're working to move away from that server, but the process is slow and painful.  Without the ability to use PHP to serve the daily menu, we had to come up with something else.  AJAX was the obvious choice, but how to deal with the fact that we'd have to make cross-domain requests in order to retrieve the application's XML output?

**Cross-Domain JSON with jQuery**

After a little digging around, one of my co-workers turned me on to [jQuery](http://jquery.com/) and [JSONP](http://bob.pythonmac.org/archives/2005/12/05/remote-json-jsonp/).  As of version 1.2, jQuery provides native support for JSONP, a method of retrieving JSON data across domains.  The implementation can be found in [jQuery.getJSON](http://docs.jquery.com/Ajax/jQuery.getJSON).

Once the decision was made to utilize jQuery's getJSON, I had to come up with a way to convert the menu's XML output to JSON.  Since the Menu Builder app was already spitting out XML, there was no way I was going to add a module to the application to support the new JSON requirement.  I needed a simple way to convert XML to JSON, and I didn't want to spend any time rolling my own solution.

**Zend Framework to the Rescue**

Since we're already using [Zend Framework](http://framework.zend.com/) on our intranet, I decided to look to [Zend Json](http://framework.zend.com/manual/en/zend.json.html) and see if it could do the job.  Sure enough, there's a static [fromXml()](http://framework.zend.com/manual/en/zend.json.xml2json.html) method that takes care of the conversion beautifully.

After that, it was fairly quick work to create some JavaScript functions to parse the returned JSON and display it on the cafeteria website.

[Side note: As of ZF 1.6, the [Dojo](http://dojotoolkit.org/) JavaScript library is [integrated with Zend Framework](http://framework.zend.com/manual/en/zend.dojo.html).  Well before that integration occurred, we made the decision that jQuery would be our library of choice.  That's why I'm using jQuery for this solution rather than Dojo.]

**The Code**

First is an example of the XML returned from the Menu Builder application.

    
    
    
    <menu>
      <timestamp>Thu Oct 16 13:18:50 CDT 2008</timestamp>
      <day>Thursday</day>
      <date>October 16, 2008</date>
      <meal descr="Breakfast Feature">
        <category descr="None (will not display to users)">
          <item>
            <itemdescr>French Toast</itemdescr>
            <itemprice>0.45</itemprice>
          </item>
          <item>
            <itemdescr>Biscuit</itemdescr>
            <itemprice>0.25</itemprice>
          </item>
        </category>
      </meal>
      <meal descr="Lunch">
        <category descr="Grab & Go">
          <item>
            <itemdescr>Garden Vegetables</itemdescr>
            <itemprice>0.89</itemprice>
          </item>
        </category>
        <category descr="Papa J's Pizza by the Slice">
          <item>
            <itemdescr>Sausage Egg & Cheese Biscuit</itemdescr>
            <itemprice>2.09</itemprice>
          </item>
        </category>
      </meal>
    </menu>
    



Running the above XML through Zend_Json::fromXml() provided me with the JSON representation I needed.


    
    {
        "menu": {
            "timeStamp": "Thu Oct 16 13:18:50 CDT 2008",
            "day": "Thursday",
            "date": "October 16, 2008",
            "meal": [
                {
                    "@attributes": {
                        "descr": "Breakfast Feature"
                    },
                    "category": {
                        "@attributes": {
                            "descr": "None (will not display to users)"
                        },
                        "item": [
                            {
                                "itemdescr": "French Toast",
                                "itemprice": "0.45"
                            },
                            {
                                "itemdescr": "Biscuit",
                                "itemprice": "0.25"
                            }
                        ]
                    }
                },
                {
                    "@attributes": {
                        "descr": "Lunch"
                    },
                    "category": [
                        {
                            "@attributes": {
                                "descr": "Grab & Go"
                            },
                            "item": {
                                "itemdescr": "Garden Vegetables",
                                "itemprice": "0.89"
                            }
                        },
                        {
                            "@attributes": {
                                "descr": "Papa J's Pizza by the Slice"
                            },
                            "item": {
                                "itemdescr": "Sausage Egg & Cheese Biscuit",
                                "itemprice": "2.09"
                            }
                        }
                    ]
                }
            ]
        }
    }
    



As you can see in the samples above, menu items belong to categories and categories belong to meals.  Both meals and categories are described in XML attributes rather than in a description node.

Next is the PHP script that I wrote to get the menu XML, convert it to JSON, and echo it out to the jQuery.getJSON call.


    
    // Include required files
    require_once dirname(dirname(dirname(__FILE__))).'/cafeteria/lib/base.php';
    
    // Set service uri
    $uri  = $cafeteriaConfig->menu->serviceuri;
    
    // Valid days of the week
    $validDays = array('monday', 'tuesday', 'wednesday', 'thursday', 'friday', 'saturday', 'sunday');
    
    // Get day from querystring, validate
    if (isset($_GET['day'])) {
    
      $getDay = $_GET['day'];
    
      if (in_array($getDay, $validDays)) {
        $day = $getDay;
        $uri = $uri . '?day=' . $day;
      }
    
    }
    
    // Get xml from cafeteria service
    $xml = file_get_contents($uri);
    
    // Encode returned xml as JSON, do not ignore XML attributes
    $json = Zend_Json::fromXml(trim($xml), false);
    
    // Return response, wrapping response in the requested callback
    // @see http://remysharp.com/2007/10/08/what-is-jsonp/
    // @see http://docs.jquery.com/Release:jQuery_1.2/Ajax
    echo $_GET['callback'] . '(' . $json . ')';
    



There are a couple of important things to note in the script above.

First, we keep a lot of code in a base.php file.  This includes [Zend_Loader::registerAutoload()](http://framework.zend.com/manual/en/zend.loader.html#zend.loader.load.autoload) and the creation of a [Zend_Config](http://framework.zend.com/manual/en/zend.config.html) object, among other things, allowing for much [DRY](http://en.wikipedia.org/wiki/DRY_code)er code.

Second, you'll notice that I'm using the second, optional parameter for fromXml().  Set to false, fromXml() will return a representation of the XML attributes present in the input.  fromXml() ignores XML attributes by default.

Finally, you can see how the callback name and parentheses are wrapped around the returned JSON before outputting anything back to jQuery.getJSON.  This is required.  Without it, your cross-domain request won't return anything at all.

The next bit of code is the JavaScript that parses the returned JSON and outputs the full menu to the web.  Notice that there are two output functions.  This is to allow for two different styles of menu tables: one for the cafeteria homepage (sidebar), and one for the daily menu page (full sized).

    
    // Parses JSON representation of cafeteria menu and returns Menu table
    // Requires jQuery
    
    var menu = '';
    
    // Builds sidebar menu for cafeteria home page
    function outputSidebarMenu(json) {
    	var day = json.menu.day;
    	var heading = '<h4><a href="' + menuLink + '?requestedDay=' + day.toLowerCase() + '">' + day + "'s Menu</a></h4>\n";
    	menu = '<table id="side_datatable">' + getMeals(json.menu.meal) + '</table>';
    	return timeStampComment(json) + heading + menu;
    }
    
    // Builds table for daily menu page
    function outputDailyMenu(json) {
    	var day = json.menu.day;
    	var date = json.menu.date;
    	var meal = json.menu.meal;
    	var heading = '<h4 style="text-align:right; font-weight:normal;"><span style="float:left; font-weight:bold;">' + day + '\'s Menu</span>' + date + "</h4>\n";
    	menu = '<table width="400" class="datatable">' + "\n" + getMeals(meal) + "</table>\n";
    	return timeStampComment(json) + heading + menu;
    }
    
    // Get menu meals
    function getMeals(meals) {
    	var mealList = '';
    	if (meals != null) {
    		if (meals instanceof Array) {
    			$.each(meals, function(i,meal) {
    				mealList += buildMealRow(meal);
    				mealList += getCategories(meal.category);
    			});
    		} else {
    			mealList += buildMealRow(meals);
    			mealList += getCategories(meals.category);
    		}
    	}
    	return mealList;
    }
    
    // Get meal's categories
    function getCategories(categories) {
    	var catList = '';
    	if (categories != null) {
    		// Is categories an array?
    		if (categories instanceof Array) {
    			$.each(categories, function(i, category) {
    				catList += buildCategoryRow(category);
    			    catList += getItems(category.item);
    			});  
    		} else {
    			catList += buildCategoryRow(categories);
    			catList += getItems(categories.item);
    		}
    	}
    	return catList;
    }
    
    // Get category's items
    function getItems(items) {
    	var itemList = '';
    	// Is items an array?
    	if (items instanceof Array) {
    		$.each(items, function(i, item) {
    			itemList += buildItemRow(item);
    		});  
    	} else {
    		itemList += buildItemRow(items);
    	}
    	return itemList;
    }
    
    // Build individual meal's table row
    function buildMealRow(meal) {
    	var descr = meal['@attributes'].descr;
    	var mealRow = '<tr><th colspan="2">' + descr + "</th></tr>\n";
    	return mealRow;
    }
    
    // Build individual category's table row
    function buildCategoryRow(category) {
    	var catRow = '';
    	if (category != null) {
    		var descr = category['@attributes'].descr;
    		// Do not display the 'None' category
    		if (descr.search(/None/) == -1) {
    			catRow = '<tr class="title"><td colspan="2">' + descr + "</td></tr>\n";
    		}
    	}
    	return catRow;
    }
    
    // Build individual item's table row
    function buildItemRow(item) {
    	var itemRow = '';
    	if (item != null) { 
    		var itemDescr = item.itemdescr;
    		var itemPrice = item.itemprice;
    		itemRow = '<tr><td>' + itemDescr + '</td><td>$' + itemPrice + "</td></tr>\n";
    	}
    	return itemRow;
    }
    
    // Update Cafeteria Daily Menu page title with "[day]'s Menu"
    function updatePageTitle(json) {
        // Get day to use in page title
        var titleDay = json.menu.day;
        // Append day to page title
        document.title = document.title + ' | ' + titleDay + "'s Menu";
    }
    
    // Output html comment with note on when menu was last cached for troubleshooting
    // and debugging
    function timeStampComment(json) {
    	  var day = json.menu.day;
    	  var cacheTimeStamp = json.menu.timeStamp;
    	  var timeStampComment = "\n\n";
    	  return timeStampComment;
    }


There's one big gotcha in the code above.  Notice how I'm setting the descr variable in both the buildMenuRow() and the buildCategoryRow() functions.  If you don't use array notation with @attributes (['@attributes']), this code will fail in IE.

Finally, let's take a look at how I displayed the menu on the cafeteria home page and on the daily menu page.

This first snippet displays the sidebar menu on the cafeteria homepage.  Since the homepage always displays the current day's menu, the only parameter that I'm passing with the querystring is the required callback.


    
    // URL to menu display page
        var menuLink = 'http://www.example.com/cafeteria/dailyMenu.shtml';
        
        $(document).ready(function() {
            
           // Get JSON representation of cafeteria menu for today
            $.getJSON("http://www.cross-domain-example.com/service/cafeteria/json.php?callback=cafeteriaMenu", function(json) {
                // Build html menu table
                var content = outputSidebarMenu(json);
                // Add table to .sidebarMenu div
                $(".sidebarMenu").html(content);
            });
        });



The last snippet displays the full sized menu on the daily menu page.  Since I only want a single daily menu page, rather than a page for each day of the week, I had to have some way to grab a "requestedDay" parameter from the querystring.  The jQuery [Query String Object plugin](http://plugins.jquery.com/project/query-object) solved that problem perfectly.


    
    // Menu display requires jQuery and jQuery querystring plugin (jQuery.query.js)
        $(document).ready(function() {
            
              // JSON service URL
            var jsonUrl = 'http://www.cross-domain-example.com/service/cafeteria/json.php?callback=?';
               
            // Get requested day from QueryString
            // @see http://plugins.jquery.com/project/query-object
            var reqDay = $.query.get('requestedDay');
            jsonUrl = jsonUrl + '&day;=' + reqDay.toLowerCase();
    
            // Get JSON representation of cafeteria menu for today            
            $.getJSON(jsonUrl, function(json) {
    
                // Add "[day]'s Menu" to page title
                updatePageTitle(json);
    
                // Build html menu table
                var content = outputDailyMenu(json);
                
                // Add table to #menuBuilder div
                $("#menuBuilder").html(content);
            });
              
        });



**Wrapping Up**

When you're in a situation where you can't use PHP (or your programming language of choice) to display dynamic content, AJAX can be an excellent solution.  With the right tools, good documentation, and a little time spent on Google, it's relatively simple to overcome the limitations of your environment and deliver rich, up-to-date content to your users.  Combining Zend Framework, the jQuery JavaScript Library, and JSONP made for a nice, lightweight solution to the cafeteria menu problem, saving my clients and co-workers a lot of time and effort they can now expend elsewhere.
