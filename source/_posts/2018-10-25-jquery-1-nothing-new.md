---
layout: post
title: "JQuery 1: Nothing New"
permalink: jquery-1-nothing-new
date: 2018-10-25 11:02:48
comments: true
description: "JQuery 1: Nothing New"
keywords: "JQuery"
categories: Study

tags:
- Javascript
- Source code read
---

## Background
JQuery has the no `new` implementation, so whenever we use it, just need to do `$('something')`, instead of doing `new JQuery('something')`.

Also, JQuery supports method chaining, something like `$('body').css('something').prop('something..')`.

In this note, we will reveal how JQuery does this, and why JQuery need to do in this way.

## Implementation
### Intuitive
```js
(function() {
  let jquery = function(selector) {
    return new jquery(selector);
  };
  window.$ = window.jQuery = jquery;
})();
```
This will recursively call the `jquery` constructor and raise the stack full error, obviously.

### Return `this`
```js
(function() {
  let jquery = function(selector) {
    this.selector = selector;
    return this;
  };
  jquery.prototype = {
    selector: 'html',
    getDom: function() {
      return document.querySelector(this.selector);
    }
  };
  window.$ = window.jQuery = jquery;
})();
```
This doesn't work too. The `this` is `window`, not an instance of jQuery.

### Put `init` in `prototype`
```js
(function() {
  let jquery = function(selector) {
    return jquery.prototype.init(selector);
  };
  jquery.prototype = {
    init: function(selector) {
      this.selector = selector;
      return this;
    },
    selector: 'html',
    getDom: function() {
      return document.querySelector(this.selector);
    }
  };
  window.$ = window.jQuery = jquery;
})();
x = $('abc');
y = $('def');
x.selector = 'xxxxxxxxxx';
y.selector;
```
All jQuery object refers to the same `jquery.prototype`, whenever you change one of them, you change all of them.

This is not expected by us.

### Use `new`
```js
(function() {
  let jquery = function(selector) {
    return new jquery.prototype.init(selector);
  };
  jquery.prototype = {
    init: function(selector) {
      this.selector = selector;
      return this;
    },
    selector: 'html',
    getDom: function() {
      return document.querySelector(this.selector);
    }
  };
  window.$ = window.jQuery = jquery;
})();
x = $('abc');
y = $('def');
x.selector = 'xxxxxxxxxx';
y.selector;
x.getDom();
```
In order to avoid the same reference thing, we use `new` to create a new object to avoid global changes to all objects.

However, `x.getDom()` will raise the no method error, as the `new` will make a new object instead of using the current `jquery.prototype` object.

### Link `prototype`
```js
(function() {
  let jquery = function(selector) {
    return new jquery.prototype.init(selector);
  };
  jquery.prototype = {
    init: function(selector) {
      this.selector = selector || this.selector;
      return this;
    },
    selector: 'html',
    getDom: function() {
      return document.querySelector(this.selector);
    }
  };
  // this is tricky
  jquery.prototype.init.prototype = jquery.prototype;
  window.$ = window.jQuery = jquery;
})();
x = $('abc');
y = $('def');
x.selector = 'xxxxxxxxxx';
y.selector;
x.getDom();
```
We manually link the prototype to share the common methods. This is one of the most tricky part in its source code. The idea is to copy the prototype from `jquery.prototype` to the actual `init` method. So when we use `new` in `jquery`, we will have access to all properties in the prototype as expected.

### Chain method calls
```js
(function() {
  let jquery = function(selector) {
    return new jquery.prototype.init(selector);
  };
  jquery.prototype = {
    init: function(selector) {
      this.selector = selector || this.selector;
      return this;
    },
    selector: 'html',
    getDom: function() {
      return document.querySelector(this.selector);
    },
    css: function(css) {
      this._css = css;
      return this;
    }
  };
  jquery.prototype.init.prototype = jquery.prototype;
  window.$ = window.jQuery = jquery;
})();
```
The `return this` is the solution. It returns the object itself with all methods available to use again.

### Why put `init` in `prototype`
```js
(function() {
  let JQuery = function(selector) {
    this.selector = selector || this.selector;
  };
  JQuery.prototype = {
    selector: 'html',
    getDom: function() {
      return document.querySelector(this.selector);
    },
    css: function(css) {
      this._css = css;
      return this;
    }
  };
  let jquery = function(selector) {
    return new JQuery(selector);
  };
  jquery.prototype = JQuery.prototype;
  window.$ = window.jQuery = jquery;
})();
```
As a factory method `$()`, the above implementation works too. So why jquery puts `init` in its `prototype` specifically, which makes the code much more difficult to read.

The commit was in [jquery's githut repo](https://github.com/jquery/jquery/commit/f97f77c034dc62001a687c728bdfdc71a23bf6b8). However, it doesn't provide enough information for us. I found a similar question [here](https://stackoverflow.com/questions/18782973/why-is-the-init-function-in-jquery-prototype-and-not-in-jquerys-closure). I think the answer is, the `init` way exposes the method to outside, so other code can **override** when needed. Also, it is the pattern the author thinks good to implement inheritance.

### Override the `init`
```js
(function() {
  let jquery = function(selector) {
    return new jquery.prototype.init(selector);
  };
  jquery.prototype = {
    init: function(selector) {
      this.selector = selector || this.selector;
      return this;
    },
    selector: 'html',
    getDom: function() {
      return document.querySelector(this.selector);
    },
    css: function(css) {
      this._css = css;
      return this;
    }
  };
  jquery.prototype.init.prototype = jquery.prototype;
  window.$ = window.jQuery = jquery;
})();

oldInit = $.prototype.init;

$.prototype.init = function(selector) {
  console.log(new Date());
  return oldInit.call($.prototype, selector);
};
```
This is a way to override the `init`. Now every `$()` will output the current timestamp first.
