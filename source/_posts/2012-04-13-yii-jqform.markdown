---
layout: post
title: "Yii and jQuery Form plugin"
date: 2012-04-13 11:14
comments: true
sharing: true
footer: true
categories: [yii jquery jqform]
---

[Yii](http://www.yiiframework.com) is a *"Fast, Secure and Professional PHP framework"*.

I abandoned CodeIgniter in favour of Yii a few months ago and I have already delivered two successful projects using the latter one.

Among the interesting features offered by Yii, there is one that has been a real headache: the form widget and the client-side validation.
Yii framework allows you to define validation rules in the model classes and then it generates the javascript code to perform validation also on the client. The js code is generated on [CActiveForm](http://www.yiiframework.com/doc/api/1.1/CActiveForm) widget rendering. When the user submits the form, the validatio code is executed on the browser and if everything is fine, the data are sent to the server.

## Problem ##
Till now, everything looks awesome, but at some point I needed to let [jqForm](http://jquery.malsup.com/form/) jQuery plugin to handle the Ajax form submission. **BUT** I would love to keep the Yii generated javascript validation. ** AND ** I don't want to change the Yii javascript library code.

## Solution ##
** EVENTS!! ** Yes, the Yii form widget allows you to specify a javascript function that is called after client-side validation. 
<!-- more -->
This is what should happend once the Yii submit listener intercepts the submit button click event:

* prevents default form submission
* validate user input  
* executes the "afterValidate" function, if any
* if validation is ok, it submits the form to the server


The "afterValidate" function should be designed to perform at least three tasks:

- check that the validation was successful
- if user inputs have been validated, submit the form using jqForm plugin
- prevent the form submission by the Yii javascript library

### Import jqForm library ###
In the form view file:

``` php
Yii::app()->clientScript->registerScriptFile('/js/jquery.form.js');
```
This code tells Yii to include the *jquery.form.js* file in the html page header.

### Define afterValidate function ###
In the form view file:

``` php
Yii::app()->clientScript->registerScript('yiijqform', "
   function myAfterValidateFunction(form, data, hasError)
   {
        if(!hasError)
        {
            $(form).ajaxSubmit({
                dataType: 'json',
                timeout: 10000,                
                success: function(data)
                {
                	// do something
                },
                error: function(xhr)
                {
                	alert('ops...');
                }
            });
        }
        return false;
   }
       
", CClientScript::POS_HEAD);
```

When the myAfterValidateFunction is invoked, the form is Ajax submitted through the ajaxSubmit function defined in the jqForm plugin. Please be sure to check for the <code>hasError</code> paramter that contains the result of the client-side validation. If it is <code>false</code>, the form shouldn't be submitted to the server.

The <code>return false</code> at the end of the function body prevents the flow of Yii javascript form submission handler to be resumed. Otherwise, the form would be submitted twice!

### Attach function to afterValidate event ###
In the form view file:

``` php
$form = $this->beginWidget('CActiveForm', array(
    'id'=>'myform_id',
    'action'=>array('/my/action'),
    'enableClientValidation'=>true,
    'clientOptions'=>array(
        'validateOnSubmit'=>true,
        'validateOnChange'=>false,
        'afterValidate'=>'js:myAfterValidateFunction'
    )
));
```
As you can see, attaching a javascript function to afterValidate event is pretty simple using the CActiveForm widget options. 

### Final thoughts ###
As you can see, integrating jqForm with Yii javascript libraries is quite easy once you know how to hook your code without touching framework core files. Fortunately Yii framework is very well documented and the source code can be understood by experienced Php developers. If you know more elegant ways to accomplish this task, please drop me a line via e-mail or comment below.