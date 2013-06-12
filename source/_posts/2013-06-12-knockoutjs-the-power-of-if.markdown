---
layout: post
title: "KnockoutJS The Power of if"
date: 2013-06-12 14:11
comments: true
categories: 
---


## To If or IfNot That is the Question

Knockoutjs is a fantastic library for building single page applications and for making use of the MVVM in your javascript heavy web pages. Something that is over looked a lot when talking about knockout is the if and ifnot bindings.

### What Does if Do?

From the docs

> The if binding causes a section of markup to appear in your document (and to have its data-bind attributes applied), only if a specified expression evaluates to true (or a true-ish value such as a non-null object or nonempty string).

This is an extremely powerful feature of knockout, some reasons:

- It allows you to have conditional DOM elements without caring if the property is defined

- It allows you to have large collections and only render the current active one

- It allows you to speed up your page rendering time

Let's discuss:

Say you have the following ViewModel that has 1 Million child items

    var data = function() { 
      var self = this;
     
      self.items = ko.observableArray();
      self.selectedItem = ko.observable();
     
      //get old value
      self.selectedItem.subscribe(function(value) { 
        if(value) {
          value.Toggle();
        }
      }, self, "beforeChange");
     
      //get the new value
      self.selectedItem.subscribe(function(value) { 
        if(value) {
          value.Toggle();
        }
      });
     
      //populate some items
      for(var i = 0; i < 1000; i++)
      {
        self.items.push(new nested(i));
      }
    };
     
    var nested = function(name) { 
      var self = this;
     
      self.Name = name;
     
      self.IsActive = ko.observable(false);
     
      self.Toggle = function() { 
        self.IsActive(!self.IsActive());
      }
     
      self.nestedItems = ko.observableArray();
     
      //populate some propeties on the items
      self.msg = var msg = 'This is a really big collection ' + self.Name;
      for(var i = 0; i < 1000; i++)
      {
        self.nestedItems.push(self.msg);
      }
    }
     
Then you render this as follows

    <select data-bind="options: items, optionsText: 'Name', value: selectedItem"></select>
     
    <ul data-bind="foreach: items">
      <li data-bind="if: IsActive">
        <ul data-bind="foreach: nestedItems">
          <li>
            <span data-bind="text: $data"></span>
          </li>
        </ul>
      </li>
    </ul>
 
JsFiddle This uses a slightly smaller amount of items as 1 Million is really not the best idea to generate in JS.

What's difference between if and ifnot

Again from the docs

>The ifnot binding is exactly the same as the if binding, except that it inverts the result of whatever expression you pass to it. For more details, see documentation for the if binding.

It also says that ifnot is the same as an inverted if or

    data-bind="if: IsActive"

is the same as

    data-bind="if: !IsActive()"

One thing to note is that if you directly access an observable in a binding you don't need to evaluate it; however, if you need to invert it you must first access the value (e.g. IsActive()). ifnot avoids this requirement.
