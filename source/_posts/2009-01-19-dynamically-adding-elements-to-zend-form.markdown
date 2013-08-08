---
author: admin
comments: true
date: 2009-01-19 17:44:56+00:00
layout: post
slug: dynamically-adding-elements-to-zend-form
title: Dynamically Adding Elements to Zend_Form
wordpress_id: 154
categories:
- Development
tags:
- jquery
- zend form
- zend framework
---

There have been some requests on the Zend Framework mailing lists for information on how to dynamically add elements to Zend_Form.  This is something that I've been looking into myself, and I'd like to share what I've come up with.

Please note that this code is a proof of concept / request for peer review detailing the work that I've done to date, and not an example of what I might consider the best way to address this use case.  Special thanks go to [Cory Wiles](http://www.corywiles.com/home/) who helped me think things through when I first started giving this a go.

First, let's do a high level walk through of what the code is going to do, then take a look at the code, and wrap up with a live example.

**High Level Overview**

The form in this example extends Zend_Form and consists of a hidden element that stores an ID, a single text element, buttons for adding and removing dynamic elements, and a submit button.  The add and remove buttons are used to trigger a jQuery script that adds and removes dynamic elements. The jQuery script uses the value of the hidden ID element to set element order and make the dynamic element names and IDs unique.

The form class consists of the standard init() method for building the form and two custom methods for dealing with dynamic elements.  There is a preValidation() method, called after the form is submitted but before it is validated, that searches the submitted form data for dynamically added fields.  If any new fields are found, the addNewField() method takes care of adding the new fields to the form.

jQuery is used to request the new form element from the form's Controller via Ajax, utilizing the AjaxContext action helper. jQuery is also used to find the most recently added dynamic element, allowing for easy removal of dynamically added elements from the form.

The action controller contains the action that displays the form, and it also has a newfieldAction() that utilizes the AjaxContext to return markup for new fields.

**The Zend_Form Subclass**

Let's start with the code for the form.  The most important item here is that each form element has its order property set.  You can see the huge jump in the order between the "name" element and the  "addElement" button.  This gap occurs so the dynamic elements can be placed exactly where I want them and so they'll maintain their position in the form once they've been added to the form object.


    
    
    public function init() {
    	
      $this->addElement('hidden', 'id', array(
        'value' => 1
      ));
      
      $this->addElement('text', 'name', array(
        'required' => true,
        'label'    => 'Name',
        'order'    => 2,
      ));
      
      $this->addElement('button', 'addElement', array(
        'label' => 'Add',
        'order' => 91
      ));
      
      $this->addElement('button', 'removeElement', array(
        'label' => 'Remove',
        'order' => 92
      ));
      
      // Submit
      $this->addElement('submit', 'submit', array(
        'label' => 'Submit',
        'order' => 93
      ));
    }
    



**Action Controller**

The action that displays the form is straightforward.  If you've ever done any work with Zend_Form, I'm sure you recognize what's going on here.  The only thing to note is the $form->preValidation() method.  That's where the magic happens.  We'll get to that in a bit.


    
    
    /**
     * Shows the dynamic form demonstration page
     */
    public function dynamicFormElementsAction() {
    	
      $form = new Code_Form_Dynamic();
      
      // Form has not been submitted - pass to view and return
      if (!$this->getRequest()->isPost()) {
        $this->view->form = $form;
        return;
      }
    
       // Form has been submitted - run data through preValidation()
      $form->preValidation($_POST);
      
       // If the form doesn't validate, pass to view and return
      if (!$form->isValid($_POST)) {
        $this->view->form = $form;
        return;
      }
      
       // Form is valid
      $this->view->form = $form;
    }
    



Next comes the controller's newfieldAction().  This action utilizes the AjaxContext action helper to pass the new field's markup back to the form view.


    
    
    /**
     * Ajax action that returns the dynamic form field
     */
    public function newfieldAction() {
      
      $ajaxContext = $this->_helper->getHelper('AjaxContext');
      $ajaxContext->addActionContext('newfield', 'html')->initContext();
      
      $id = $this->_getParam('id', null);
      
      $element = new Zend_Form_Element_Text("newName$id");
      $element->setRequired(true)->setLabel('Name');
      
      $this->view->field = $element->__toString();
    }
    



**jQuery**

The jQuery script is also fairly straightforward.  I attach event listeners to the "Add" and "Remove" buttons that call the ajaxAddField and removeField methods respectively.

The ajaxAddField method makes a post request to the newfieldAction using [jQuery's .ajax method](http://docs.jquery.com/Ajax/jQuery.ajax#options), passing in the current value of the hidden ID element.  On success, the new element's markup is added to the form, and the ID is incremented and stored in the hidden ID element.

The removeField method finds the last element in the page with the class dynamic, removes it, then decrements the current ID and stores the new value in the hidden ID element.


    
    
    <script type="text/javascript">
    
    $(document).ready(function() {
      
      $("#addElement").click( 
          function() { 
              ajaxAddField();
           }
        );
      
      $("#removeElement").click(
          function() {
              removeField();
          }
        );
      }
    );
    
    // Get value of id - integer appended to dynamic form field names and ids
    var id = $("#id").val();
    
    // Retrieve new element's html from controller
    function ajaxAddField() {
      $.ajax(
        {
          type: "POST",
          url: "<?=$this->url(array('action' => 'newfield', 'format' => 'html'));?>",
          data: "id=" + id,
          success: function(newElement) {
            
            // Insert new element before the Add button
            $("#addElement-label").before(newElement);
            
            // Increment and store id
            $("#id").val(++id);
          }
        }
      );
    }
    
    function removeField() {
    
      // Get the last used id
      var lastId = $("#id").val() - 1;
    
      // Build the attribute search string.  This will match the last added  dt and dd elements.  
      // Specifically, it matches any element where the id begins with 'newName<int>-'.
      searchString = '*[id^=newName' + lastId + '-]';
    
      // Remove the elements that match the search string.
      $(searchString).remove()
    
      // Decrement and store id
      $("#id").val(--id);
    }
    </script>
    



**Zend_Form: preValidation() and addNewField()**

Now on to the fun stuff.  All of the code up to this point is present to support what happens in the form's preValidation() method.  Remember that preValidation() is called after the form has been submitted but before the form is validated.  preValidation() searches through the submitted form's data for new fields.  If it finds any new fields, it calls addNewField() and adds the new fields to the form object.  By adding the new form fields to the form object before validation, any filters and validators attached to the new fields will be run as if those fields had always existed in the form object.


    
    
    /**
     * After post, pre validation hook
     * 
     * Finds all fields where name includes 'newName' and uses addNewField to add
     * them to the form object
     * 
     * @param array $data $_GET or $_POST
     */
    public function preValidation(array $data) {
    
      // array_filter callback
      function findFields($field) {
        // return field names that include 'newName'
        if (strpos($field, 'newName') !== false) {
          return $field;
        }
      }
      
      // Search $data for dynamically added fields using findFields callback
      $newFields = array_filter(array_keys($data), 'findFields');
      
      foreach ($newFields as $fieldName) {
        // strip the id number off of the field name and use it to set new order
        $order = ltrim($fieldName, 'newName') + 2;
        $this->addNewField($fieldName, $data[$fieldName], $order);
      }
    }
    
    /**
     * Adds new fields to form
     *
     * @param string $name
     * @param string $value
     * @param int    $order
     */
    public function addNewField($name, $value, $order) {
      
      $this->addElement('text', $name, array(
        'required'       => true,
        'label'          => 'Name',
        'value'          => $value,
        'order'          => $order
      ));
    }
    



**Live Example**

If you'd like to see working version of this proof of concept, please visit [the live example](http://code.jeremykendall.net/forms/dynamic-form-elements) at code.jeremykendall.net.

**Summary**

The ability to dynamically add form fields to Zend_Form is a feature I'd really like to see added to Zend_Form.  If I were talented enough, I might attempt to make a formal proposal myself.  In the meantime, what I've come up with can perhaps serve as a starting point for adding very simple elements to very simple forms.

Thanks again to [Cory Wiles](http://www.corywiles.com/home/) for helping me work out some of the kinks during the planning phase.  Any mistakes, bad practices, or egregious coding errors are the result of my implementation, not his insight and suggestions.

**Request for Comments / Peer Review**

If you've made it this far, I'm grateful to you for hanging in with me.  If you have suggestions for improvements to the code, an implementation of your own, or if you see mistakes I've made or poor practices that I've employed, I'd appreciate your input.  Thank you in advance for taking the time to discuss this concept with myself and with the ZF community at large.

**Full Controller, Form, and View Code**

If you're interested in the complete code for the controller, form, and views, I've posted them over at pastebin.  Follow the links below to view / grab the code.



	
  * [Action Controller (FormsController)
](http://jeremykendall.pastebin.com/f675a1ec1)

	
  * [Form (Code_Form_Dynamic)
](http://jeremykendall.pastebin.com/f5d408d7b)

	
  * [Form view (dynamic-form-elements.phtml)
](http://jeremykendall.pastebin.com/f56ccb1f7)

	
  * [Ajax view (newfield.ajax.phtml)
](http://jeremykendall.pastebin.com/f423d0ee3)


**Further reading:**



	
  * [Zend_Form](http://framework.zend.com/manual/en/zend.form.html)

	
  * [Zend_Form Decorators](http://framework.zend.com/manual/en/zend.form.elements.html#zend.form.elements.decorators)

	
  * [AjaxContext Action Helper](http://framework.zend.com/manual/en/zend.controller.actionhelpers.html#zend.controller.actionhelpers.contextswitch)

	
  * [jQuery JavaScript Library](http://jquery.com/)

	
  * [jQuery Events/click](http://docs.jquery.com/Events/click)

	
  * [jQuery Manipulation/append](http://docs.jquery.com/Manipulation/append)

	
  * [jQuery Manipulation/remove](http://docs.jquery.com/Manipulation/remove)


