---
author: "Jeremy Kendall"
comments: true
date: 2009-10-21 15:28:14+00:00
layout: post
slug: zf-7984-zend_tool-exits-with-fatal-errors-after-installing-phpunit-3-4-0
title: ZF-7984 - Zend_Tool Exits with Fatal Errors after installing PHPUnit 3.4.0+
wordpress_id: 197
categories:
- Development
tags:
- bug
- phpunit
- zend framework
---

I'm lucky enough to have made it to [ZendCon](http://zendcon.com/) again this year, and I'm having a blast learning new stuff, hanging out with old friends, making new friends, and generally grabbing up as much schwag as possible.

One of the topics that I'm most interested in is unit testing, specifically unit testing [Zend Framework](http://framework.zend.com/) MVC apps.  While there's a lot I have yet to learn on that topic, I ran into a bug last night that I wanted to let you know about.

In preparation to dig into ZF unit testing, I updated my install of [PHPUnit](http://www.phpunit.de/) to the latest version (currently 3.4.1, installed via [PEAR](http://pear.php.net/)).  When I tried to create a new ZF project using [Zend_Tool](http://framework.zend.com/manual/en/zend.application.quick-start.html#zend.application.quick-start.zend-tool), I received the following error:

    
    
    jkendall@san-diego:~/dev/www$ zf create project asplode
    
    Fatal error: Cannot redeclare class phpunit_framework_testsuite_dataprovider in /usr/share/php/PHPUnit/Framework/TestSuite/DataProvider.php on line 64
    
    Call Stack:
        0.0020     111440   1. {main}() /usr/share/phplib/ZendFramework-1.9.3PL1/bin/zf.php:0
        0.0020     111560   2. zf_main() /usr/share/phplib/ZendFramework-1.9.3PL1/bin/zf.php:23
        0.0220     686832   3. zf_run($zfConfig = array ('HOME' => '/home/jkendall')) /usr/share/phplib/ZendFramework-1.9.3PL1/bin/zf.php:36
        0.0221     686952   4. Zend_Tool_Framework_Client_Console::main($options = array ()) /usr/share/phplib/ZendFramework-1.9.3PL1/bin/zf.php:214
        0.0221     687440   5. Zend_Tool_Framework_Client_Abstract->dispatch() /usr/share/phplib/ZendFramework-1.9.3PL1/library/Zend/Tool/Framework/Client/Console.php:96
        0.0222     687560   6. Zend_Tool_Framework_Client_Abstract->initialize() /usr/share/phplib/ZendFramework-1.9.3PL1/library/Zend/Tool/Framework/Client/Abstract.php:209
        0.0296     866600   7. Zend_Tool_Framework_Loader_Abstract->load() /usr/share/phplib/ZendFramework-1.9.3PL1/library/Zend/Tool/Framework/Client/Abstract.php:118
        0.4100    2729736   8. include_once('/usr/share/php/PHPUnit/Framework/TestSuite/DataProvider.php') /usr/share/phplib/ZendFramework-1.9.3PL1/library/Zend/Tool/Framework/Loader/Abstract.php:90
    
    jkendall@san-diego:~/dev/www$
    



As it turns out, this is a known bug in version 1.9.0+ of the Zend Framework.  See [ZF-7894](http://framework.zend.com/issues/browse/ZF-7894) in the ZF issue tracker for full details.  While this issue is not yet resolved in the tracker, [Raphael Stolt](http://raphaelstolt.blogspot.com/) has [provided a workaround](http://framework.zend.com/issues/browse/ZF-7894?focusedCommentId=34826&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#action_34826) in the form of a diff file attached to the issue.  While your mileage may vary, the patch worked perfectly for me.  I'm able to go ahead and dive into unit testing my Zend Framework applications.

**UPDATE**: [ZF-7894](http://framework.zend.com/issues/browse/ZF-7894) was resolved during Bug Hunt days this week.  Many thanks to Benjamin Eberlei!
