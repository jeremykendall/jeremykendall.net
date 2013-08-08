---
author: admin
comments: true
date: 2008-12-04 13:46:54+00:00
layout: post
slug: zend_form_element_multi-tips-and-tricks
title: Zend_Form_Element_Multi - Tips and Tricks
wordpress_id: 87
categories:
- Development
tags:
- zend framework
---

I'm responsible for creating a lot of forms at my day job.  It seems that any project I get involved in requires at least one form.  The [Zend Form](http://framework.zend.com/manual/en/zend.form.html) component has made my life a lot easier.  After putting together more forms than I can count, I've picked up a couple of tricks that I'd like to share.  Here are some for the Zend_Form_Element_Multi elements.

As noted in the [API documentation](http://framework.zend.com/apidoc/core/), Zend_Form_Element_Multi is the base class for multi-option form elements.  Its direct descendants are the Zend Form [Select](http://framework.zend.com/manual/en/zend.form.standardElements.html#zend.form.standardElements.select), [Radio](http://framework.zend.com/manual/en/zend.form.standardElements.html#zend.form.standardElements.radio), and [MultiCheckbox](http://framework.zend.com/manual/en/zend.form.standardElements.html#zend.form.standardElements.multiCheckbox) elements.  Adding options to these elements is possible using the addMultiOptions method.  Most of what I want to cover is about retrieving, creating, and adding options, with a short detour into validation.

**Using array_combine**

Sometimes you want the displayed element options to be the same options returned by the form (as opposed to displaying a string while returning an id).  Perhaps you'll be sending the value(s) along in an email or storing them as strings in a database.  While you can create an associative array with matching keys and values, the process quickly becomes tedious with an array of any appreciable size.  Why not use [array_combine](http://us.php.net/array_combine) to make life easier?

    
    $options = array('Vanilla', 'Chocolate', 'Strawberry', 'Cookies and Cream', 'Chocolate Chip');
    $options = array_combine($options, $options);


**Using array_merge**

[array_merge](http://us.php.net/array_merge) is helpful when you'd like to add an item to your options array that isn't already a part of the options array.  For example, I frequently add a "Please make a selection" option to my select elements.  Extending the above example, I might choose to add the new option like this:

    
    $options = array_merge(array('Please select a flavor'), $options);


One word of caution: array_merge will reindex numerically indexed arrays.  array_merge should never be used in a situation where the original array needs to be preserved, such as a numerically indexed array of id and value pairs pulled from a database.  In those cases, I use the + operator.

    
    // $options is an array of database ids and flavor descriptions
    $options = array('Select a flavor') + $options;


**Retrieving options using Zend_Db**

I frequently retrieve options from a database, using the record id as the array's index and a related string as the array's value.  There are a lot of ways retrieve options using [Zend_Db](http://framework.zend.com/manual/en/zend.db.html), but my favorite is the [fetchPairs](http://framework.zend.com/manual/en/zend.db.html#zend.db.adapter.select.fetchpairs) method.


> The `fetchPairs()` method returns data in an array                 of key-value pairs, as an associative array with a single entry                 per row.  The key of this associative array is taken from the                 first column returned by the SELECT query.  The value is taken                 from the second column returned by the SELECT query.  Any other                 columns returned by the query are discarded.


Here's what that might look like.

    
    $select = 'SELECT flavor_id, flavor FROM flavors';
    $options = $db->fetchPairs($select);


I especially enjoy using this method with [Zend_Db_Table](http://framework.zend.com/manual/en/zend.db.table.html), using custom table class methods to retrieve my options.  My table class usually looks like this:

    
    class Flavors extends Zend_Db_Table_Abstract {
    
      protected $_name = 'flavors';
    
      public function getFlavorOptions() {
    
        $select = $this->select()->from($this, array('flavor_id', 'flavor'));
        $result = $this->getAdapter()->fetchPairs($select);
    
        return $result;
      }
    }


Grabbing your options now becomes ridiculously simple.

    
    $flavors = new Flavors();
    $flavorOptions = $flavors->getFlavorOptions();


**Validation with Zend_Validate_InArray**

[Zend_Validate_InArray](http://framework.zend.com/manual/en/zend.validate.set.html#zend.validate.set.in_array) is the default validator for Multi elements, but the InArray validator can be a little tricky to implement properly.  Below are the two gotchas that I've run into.

Let's say you've got a select element in your form.  In order to force the user to select an option, you've added a "Please Select" option to the beginning of your options array.  If you use the default InArray validation against the full list of options, "Please Select" becomes a valid selection, and you may end up stuck with a lot of bad form submissions.  In order to get around this, I make sure to pass the original array of options to the validator, and use array_merge or the + operator to add the "Please select" option before adding the options to the element.

    
    // Create list of flavor options
    $flavorOptions = array('Vanilla', 'Chocolate', 'Strawberry', 'Cookies and Cream', 'Chocolate Chip');
    $flavorOptions = array_combine($flavorOptions, $flavorOptions);
    
    // Add "Select a flavor" option
    $flavorMultiOptions = array_merge(array('Select a flavor'), $flavorOptions);
    
    // Add flavor options to flavor select element
    $form->flavor->addMultiOptions($flavorMultiOptions);
    
    // Add validation, validating against original $flavorOptions array
    $form->flavor->addValidator(new Zend_Validate_InArray($flavorOptions));


The second gotcha has to do with option arrays where the keys and values don't match.  InArray tests the element's selected value against the values of the options array, but what you really want to do is test the element's selected value against the keys of the options array.  The trick is to use PHP's [array_keys](http://us.php.net/array_keys) function.

    
    // Get flavor ids and descriptions from database
    $flavors = new Flavors();
    $flavorOptions = $flavors->getFlavorOptions();
    
    // Add "Select a flavor" option, preserving original array with the + operator
    $flavorMultiOptions = array('Select a flavor') + $flavorOptions;
    
    // Add flavor options to flavor select element
    $form->flavor->addMultiOptions($flavorMultiOptions);
    
    // Add validation, validating against array keys of the original $flavorOptions array
    $form->flavor->addValidator(new Zend_Validate_InArray(array_keys($flavorOptions)));


Do you have any Zend_Form tips or tricks that you'd like to share?  Have I made any egregious errors above that need to be corrected?  Hit the comments and let me know.
