---
author: "Jeremy Kendall"
comments: true
date: 2008-12-24 15:41:06+00:00
layout: post
slug: conditional-form-validation-with-zend_form
title: Conditional Form Validation with Zend_Form
wordpress_id: 128
categories:
- Development
tags:
- how to
- zend framework
---

A question from 'ronny stalker' in the [Zend_Form_Element_Multi - Tips and Tricks](http://www.jeremykendall.net/2008/12/04/zend_form_element_multi-tips-and-tricks/) comments:


> I need to do different validations for field A depending on the value of field B and (possibly depending on a variable that is not in the form at all - C ).

in this kind of logic:

if (B ==1)
{
validator_B(A);
}
elseif (C)
{
validator_C(A);
}
else
{
validator_Default(A);
}

I understand that validators get a secondary argument called $context - which can be used to check values of other fields, but how can a validator get knowledge of other variables in the environment?


While this post may not answer ronny's question exactly, hopefully it will give him a good starting point to get over the hump.

**If other, please explain - Conditional Validation Using $context**

Many forms have a set of radio buttons, or sometimes a select element, where a user can choose from one of several options.  Sometimes "other"  will be one of those options, with a corresponding "If other, please explain" text field placed directly after.  If "other" is selected, then the accompanying text field is usually required.  Since there's not a standard [Zend Validate validator](http://framework.zend.com/manual/en/zend.validate.html) for this scenario, I've written a custom validator that seems to do the trick.


    
    
     'Parent field does not exist in form input',
        self::KEY_IS_EMPTY   => 'Based on your answer above, this field is required',
      );
      
      /**
       * Key to test against
       *
       * @var string|array
       */
      protected $_contextKey;
      
      /**
       * String to test for
       *
       * @var string
       */
      protected $_testValue;
      
      /**
       * FieldDepends constructor
       *
       * @param string $contextKey Name of parent field to test against
       * @param string $testValue Value of multi option that, if selected, child field required
       */
      public function __construct($contextKey, $testValue = null) {
        $this->setTestValue($testValue);
        $this->setContextKey($contextKey);
      }
      
      /**
       * Defined by Zend_Validate_Interface
       *
       * Wrapper around doValid()
       *
       * @param  string $value
       * @param  array  $context
       * @return boolean
       */
      public function isValid($value, $context = null) {
        
        $contextKey = $this->getContextKey();
        
        // If context key is an array, doValid for each context key
        if (is_array($contextKey)) {
          foreach ($contextKey as $ck) {
            $this->setContextKey($ck);
            if(!$this->doValid($value, $context)) {
              return false;
            }
          }
        } else {
          if(!$this->doValid($value, $context)) {
            return false;
          }
        }
        return true;
      }
      
      /**
       * Returns true if dependant field value is not empty when parent field value
       * indicates that the dependant field is required
       *
       * @param  string $value
       * @param  array  $context
       * @return boolean
       */
      public function doValid($value, $context = null) {
        $testValue  = $this->getTestValue();
        $contextKey = $this->getContextKey();
        $value      = (string) $value;
        $this->_setValue($value);
        
        if ((null === $context) || !is_array($context) || !array_key_exists($contextKey, $context)) {
          $this->_error(self::KEY_NOT_FOUND);
          return false;
        }
    
        if (is_array($context[$contextKey])) {
          $parentField = $context[$contextKey][0];
        } else {
          $parentField = $context[$contextKey];
        }
        
        if ($testValue) {
          if ($testValue == ($parentField) && empty($value)) {
            $this->_error(self::KEY_IS_EMPTY);
            return false;
          }
        } else {
          if (!empty($parentField) && empty($value)) {
            $this->_error(self::KEY_IS_EMPTY);
            return false;
          }
        }
        
        return true;
      }
      
      /**
       * @return string
       */
      protected function getContextKey() {
        return $this->_contextKey;
      }
      
      /**
       * @param string $contextKey
       */
      protected function setContextKey($contextKey) {
        $this->_contextKey = $contextKey;
      }
      
      /**
       * @return string
       */
      protected function getTestValue () {
        return $this->_testValue;
      }
      
      /**
       * @param string $testValue
       */
      protected function setTestValue ($testValue) {
        $this->_testValue = $testValue;
      }
    }  
    



The validator above is essentially a conditional NotEmpty validator.  It checks the value of a parent field to see if a child field should be required.  IMPORTANT:  allowEmpty _must_ be set to false on the child field.

Here's an example of how to use the validator.


    
    
    // Parent element
    $this->addElement('radio', 'flavor', array(
      'required'     => true,
      'label'        => 'Choose a flavor',
      'multiOptions' => array('Vanilla' => 'Vanilla', 'Chocolate' => 'Chocolate', 'Other' => 'Other')
    ));
    
    // Child element. IMPORTANT: allowEmpty must be set to false!
    $this->addElement('text', 'flavorOther', array(
      'allowEmpty' => false,
      'label'      => 'If Other, provide flavor here',
      'validators' => array(new Kendall_Validate_FieldDepends('flavor', 'Other')),
    ));
    



Again, please note that allowEmpty has been set to false on the child field.  This is necessary to run the FieldDepends validator even when the "If other . . ." element is empty.

While I'm sure there's plenty of room for refactoring, the above code has served me well.

**Adding Validators After Submission but Before Validation**

Expanding on the example above, what if it became necessary to add additional validators to the "If other . . ." field?  Because the "If other . . ." field has allowEmpty set to false, and because an empty value is sometimes a valid value, it is not possible to add additional validators that will run only if the field is not empty.  The additional validators will run regardless of the value of the "If other . . ." element, throwing errors when the element is empty.  Additional validators will have to be added somewhere else.

In order to work around this issue, I added a custom method called preValidation() to my form class.


    
    
    public function preValidation($data) {
      
      if (!empty($data['flavorOther'])) {
        $this->flavorOther->addValidator(new FlavorOther_Validator());
      }
      
      return $data;
    }
    



The preValidation() method is called after submission but before validation.


    
    
    $form = new Flavor_Form();
    
    if (!$this->getRequest()->isPost()) {
      // Display form
      $this->view->form = $form;
      return;
    } 
    
    $data = $form->preValidation($_POST);
    
    if (!$form->isValid($data)) {
      // Failed validation, redisplay form with values and errors
      $this->view->form = $form;
      return;
    }
    
    
    // Passed validation
    



While the preValidation() code above adds validation depending on the state of an element in the form, it would be trivial to add validation to the form based on any number of conditions, including conditions that exist as a result of business rules rather than the form's input.

**Wrapping Up**

Writing custom validators for the Zend Framework makes server side validation of unique validation scenarios a breeze.  I have yet to encounter a non-standard validation scenario where I haven't been able to address it by writing a custom validator.  With the ability to extend Zend Form with a couple of helpful custom methods, adding additional validation after form submission becomes trivial.

Have you ever had to write any custom validators?  Any suggestions on improving the code above?  Jump down to the comments and let us know!

**UPDATED** to add code comments to the validator implementation example.  Thanks to reader [Neil](http://www.11outof10.com/) for the suggestion.
