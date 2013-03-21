backbone-validator [![Build Status](https://travis-ci.org/fantactuka/backbone-validator.png?branch=master)](https://travis-ci.org/fantactuka/backbone-validator)
==================

Backbone model validator allows you to define validation rules for model and utilize it for model-standalone validation or bind its events to the view so you can display errors if needed. Inspired by @thedersen's backbone-validation

# Installation
Using [Bower](http://twitter.github.com/bower/) `bower install backbone-validator` or just copy [backbone-validator.js](https://raw.github.com/fantactuka/backbone-validator/master/backbone-validator.js)

# Usage
## Model

```js
var User = Backbone.Model.extend({
  validation: {
    name: {
      required: true,
      message: 'Name is required'
    },
    
    email: {
      required: true,
      format: 'email',
      message: 'Does not match format'
    },
    
    books: {
      collection: true
    },
    
    phone: [{
      format: 'number',
      message: 'Does not match format'
    }, {
      maxLength: 15,
      message: 'Too long'
    }]
  }
});

var user = new User();
```
Setting attributes
```js
user.set({ email: 'wrong_format_email', phone: 'wrong_format_and_also_very_long' }, { validate: true }); 
// Attributes won't be set, since validation failed. Validation errors are stored

user.set({ email: 'wrong_format_email', phone: 'wrong_format_and_also_very_long' }, { validate: true, suppress: true }); 
// Attributes will be set, but model will trigger its validation events, and store validation errors as for previous case

user.validationError; // => { email: ['Does not match format'], phone: ['Does not match format', 'Too long'] };
```
Saving model
```js
user.save(); 
// Validation triggered automatically. If nothing passed, it will validate entire model.

user.save({ email: 'user@example.com' }); 
// Validation triggered automatically. Validates only email.
```
Checking model validity
```js
// Model#isValidreturns boolean depending on model validity
user.isValid();                   // Will check all attributes
user.isValid(['email', 'name']);  // Will check specific attributes
user.isValid('email');            // Will check specific attribute

// Model#validate returns null if model is valid (no errors), or errors object if any validation failed
user.validate();                   // Will check all attributes
user.validate(['email', 'name']);  // Will check specific attributes
user.validate('email');            // Will check specific attribute
```

## View
```js
var UserView = Backbone.View.extend({
  initialize: function() {
    ...
    this.model = new User();
    this.bindValidation();
  },
  
  onValidField: function(attrName, attrValue, model) {
    // Triggered for each valid attribute
  },
  
  onInvalidField: function(attrName, attrValue, errors, model) {
    // Triggered for each invalid attribute.
  }
});
```

Note that `onValidField` and `onInvalidField` methods are optional for the view. By default it's taken from Backbone.Validator.ViewCallbacks. So you can override those defaults:
```js
Validator.ViewCallbacks = {
  onValidField: function(name /*, value, model*/) {
    var input = this.$('input[name="' + name + '"]');
    input.next('.error-text').remove();
  },

  onInvalidField: function(name, value, errors /*, model*/) {
    var input = this.$('input[name="' + name + '"]');
    input.next('.error-text').remove();
    input.after('<div class="error-text">' + errors.join(', ') + '</div>');
  }
};
```

These methods could be also passed as option to `bindValidation` method:
```js
bindValidation(this.model, {
  onValidField: function() { ... },
  onInvalidField: function() { ... }
});
```
## Built-in validators

* `required` - just checks value validity with `!!`
* `collection` - runs validation for models collection/array and returns indexed error object
* `minLength`
* `maxLength`
* `fn` - function that receives attribute value and returns true if it's valid, or false/error message if not
* `format` - pattern matching. **Please note:** format does not require field to exist. E.g. phone number could be optional, but should match format if it is not empty. So in case you need to check field existance as well - use `required` validator
  * `email`
  * `numeric`
  * `email`
  * `url`

Usage examples:
```js
var User = Backbone.Model.extend({
  validation: {
    name: {
      required: true,
      minLength: 2,
      maxLength: 20,
      fn: function(value) {
        return ~valie.indexOf('a') ? 'Name should have at least one "a" letter' : true;
      }
    },
    
    phone: {
      format: 'numeric'
    },
    
    documents: {
      collection: true
    }
  }
});

```


## Adding validator
```js
Backbone.Validator.add('myCubicValidator', function(value, expectation) {
  return value * value === expectation;
}, 'Default error message');
```
Validation method could return true/false as well as error message or array of messages which will be treated as validation vailure:
```js
Backbone.Validator.add('myCustomValidator', function(value, expectation) {
  return value === expectation ? true : 'Value does not match expectation. Should be ' + expectation;
});
```


## Standalone validator
In fact you can utilize validator for plain objects, so you can do something like this:
```js
var validations = {
  name: {
    required: true,
    message: 'Name is required'
  },
  
  email: {
    required: true,
    format: 'email',
    message: 'Does not match format'
  }
};

Backbone.Validator.validate({ name: '', email: '' }, validations); // -> { name: ['Name is required'], email: ['Does not match format'] }
```

