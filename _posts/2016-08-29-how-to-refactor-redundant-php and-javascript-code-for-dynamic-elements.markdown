---
layout: post
title:  "How to refactor redundant PHP and JavaScript code for dynamic elements"
date:   2016-08-29
categories: php javascript refactoring codeigniter
---

The implementation of dynamic UI elements, specifically in forms, is one of the most memorable features I have implemented. Commonly it is used in managing objects that are variable and can be multiple such as entries in an address book. It was a requirement in my undergraduate software engineering class for certain models and I remember scouring the Internet for ways to implement it. Fortunately I was able to deliver the feature with my team then.

![Concept for Daylog create view with dynamic Task - Set](/assets/images/posts/2017-03-20-day-log-app/improvements/create-daylog-with-task-set.png){:class="img-responsive"}

They are pretty easy in theory: assign an event to a button that creates or clones elements we want to be added, validate and send to the server; however these are much more than that. There are attributes that need to be tracked for each element, and since they are only added in the DOM after the whole page has been loaded, they will not be present after a redirect. A lot of work is needed in both the front and back-end.

## Problem

Recently I made a site that needs this feature and I had to revisit my design pattern from the past. My current set-up for validating such elements are OK but I have a ton of redundant HTML element code in both the front-end (JS) and back-end (PHP) process to make this work.

Basically this is the process:

1. Default form has the initial dynamic element (in most cases, depending on your business case)
2. User chooses to add another, JavaScript appends this to DOM
3. User submits via POST
4. If there are errors, then return to view with submitted values and show errors

Pretty straightforward, but now I have the following redundant code:

1.  Original form structure in `form.php` with the initial dynamic element:

    ~~~ html
    <form>
        <input type="text" name="names[1]" value="<?php echo (isset('names[1]') ? $_POST['names[1]'] : ''); ?>">
        <?php echo (form_error('names[1]')) ? form_error('names[1]') : ''; ?>
    </form>
    ~~~

2.  Dynamic element creation

    ~~~ javascript
    let form = document.getElementById('formId');
    let newInput = document.createElement('input');
    /* Steps to customize input element */
    form.appendChild(newInput);
    ~~~

3.  Recreate submitted elements

    After the form with dynamic elements is submitted, if there are errors, we should show all of them. The following code will loop all if there are any in `POST` otherwise just show the initial element.

    ~~~ ruby
    <?php
        if(isset($_POST['names'])) {
            foreach($_POST['names'] as $key => $value) {
                echo '<input type="text" name="names['.$key.']" value="<?php setvalue('names['.$key.']'); ?>">';

                echo (form_error('names['.$key.']') ? form_error('names['.$key.']') : '');
            }
        } else {
    ?>
        <input type="text" name="name[1]" value="<?php setvalue('names['.$key.']'); ?>">
        <?php echo (form_error('name[1]')) ? form_error('name[1]') : ''; ?>
    <?php } ?>
    ~~~~

I gave up on trying to tie in item 2 with 1 and 3 because JavaScript is needed to handle dynamic DOM creation (which PHP cannot do). However we can cleanup the latter two.

## Solution

> **NOTE**: I am using CodeIgniter 3.x with a helper method to "load" view files from another PHP file.

A basic design pattern was used to refactor 1 and 3 - move the redundant code to another file.

~~~ ruby
<?php
    if(!isset($_POST['names'])) {
        $this->load->view('path/to/complex/form', array('key', 1));
    } else {
        foreach($_POST['names'] as $key => $value) {
            $this->load->view('partial_form', array('key', $key));
        }
    }
?>
~~~

Here is `partial_form.php`:

~~~ ruby
<input type="text" name='<?php echo 'names['.$key.']'; ?>' value='<?php setvalue('names['.$key.']'); ?>'>

<?php echo (form_error('names['.$key.']') ? form_error('names['.$key.']') : ''); ?>
~~~

This is useful in other CRUD cases because updated and deleted `names` will be handled by the loop.

> **NOTE**: This is my current personal solution for this issue. I am most willing to receive feedback on how to further improve on this concept.

## References

* Sandip. "Best method of including views within views in CodeIgniter." *Php - Best method of including views within views in CodeIgniter - Stack Overflow*. N.p., 5 Mar. 2013. Web. Aug. 2016. \<[`http://stackoverflow.com/a/15221642`](http://stackoverflow.com/a/15221642)\>.