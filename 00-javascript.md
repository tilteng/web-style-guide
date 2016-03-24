# JavaScript

## The Basics

* 2 character space indents, no tabs
* Semicolons end a line
* "Best effort" 80 character line limit.  Set a ruler in your editor.  Small 
  deviations are okay for very long strings that can't be reasonably broken
  up, but most of your code should fall within the 80 character line limit.
* Use strict equality for comparing values: `===`, `!==`
* camelCased variables
* No global references (e.g. defined without `let`, `const`, `var`, `function`)
* Prefer single quotes to double quotes for strings

When joining the project you should look at the most recent pull requests that
are being produced in a repo and try to follow their general style conventions.
**The best way to learn a project's style is to read code.**

If you are just getting started with JavaScript (especially modern use of the
language), a great guide for general style is 
[Airbnb's style guide](https://github.com/airbnb/javascript).

If anything is unclear please ask!

## Babel and ES6

We use [Babel](https://babeljs.io/) for compiling our code into Javascript. It
polyfills most of the standard built-in objects to match the documentation on [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)
and compiles from ES6 into Javascript. Unless otherwise specified, an [ES6 feature](http://babeljs.io/docs/learn-es2015/) is considered good practice, specially if it makes the code easier to understand.

## References

Do not use the `var` keyword.  Variables declared with `var` live in the 
function scope, not block scope.  Block scope behaves more predictably and 
should be preferred in all cases.

```javascript
if (a > 3) {
    var i = 3;
}

console.log(i); // outputs 3
```

```javascript
if (a > 3) {
    let i = 3;
}

console.log(i); // reference error
```

Prefer to use `const` for all reference declarations.  Variables declared as
`const` cannot be reassigned - this will result in Babel throwing an error at
compile-time (not runtime).

```javascript
const a = 1;

console.log(a);
```

If you must reassign a reference (for example, keeping a running counter), use
a `let` declaration.

```javascript 
let a = 0;
if (b > 0) {
    a = a + 1;
}
```

https://github.com/airbnb/javascript#references for more information.

## Callbacks and the "this" keyword

Avoid rebinding the `this` keyword.  ES6 arrow functions make it so that in most
cases you can avoid using `bind`, `call`, `apply`, and so forth.

Don't do:

```
function() {
    this.setState({ foo: 'bar' });
}.bind(this);
```

Instead use ES6 arrow functions to automatically re-bind `this` in the scope of
the anonymous function:

```
() => this.setState({ foo: 'bar' });
```

If the function you need to bind is a 1-line function you can just inline the 
expression:

```
() => foo()
```

1-line functions will return their last value, making them very convenient to 
use in combination with iteration functions that expect a value like `.map()`.

Otherwise, you will have to use curly braces to create a new block.

```
() => {
    foo();
    bar();
}
```

Never use the `var self = this;` pattern.

## Modules

We have chosen to use the ES6 syntax (`import` and `export`) for
requiring and exporting modules.

Include all your requires at the top of the file using `const`, using single declaration only.

```javascript
// SomeComponent.js
const React = require('react');
const classnames = require('classnames');
const { TiltLogo, TiltAirplane } = require('tilt-images');
const SendEmailLightbox = require('../lightboxes/SendEmailLightbox');
const LightboxBase = require('root/lightbox/LightboxBase');
```

- Webpack generates a project that uses `root` as a base. It can be used as an
alternative to relative paths on code that does not run in the express server.
- Don't use default exports as they are not easy to include using CommonJS.
- Don't require modules dynamically as this makes them hard to validate on compile
time.

## Declaring objects

Take advantage of the shorthand functions for declaring objects. It's best to declare
an object in a single statement.

```javascript
const eventName = "mouseout";
const attr = {};
const obj = {
    attr,
    [eventName]: function() {},
    click() {},
};

// Avoid creating obj['undefined'] when this.props.mouseover is undefined
if (this.props.mouseover) {
    obj.mouseover = this.props.mouseover;
}
```

## Universal Javascript

Our code will run in the server and the browser. Some common assumptions of single
environment Javascript are not valid in an Isomorphic world.

### Window object.

Use `typeof window === "undefined"` to check for availability of the `window`
object. The common `window === undefined` will throw a `ReferenceError: window is not defined`
inside node.

```javascript
if (typeof window === "undefined") {
  window.addEventListener('resize', function (e) {});
}
```

## Asynchronous calls, callbacks and promises

Try to avoid callback hell by avoiding asynchronous calls ;) and delegate to the
dispatcher as much as possible. When it's necessary to handle a callback, consider
Promises first as they keep the interface predictable.

### Ajax requests

We use [superagent](http://visionmedia.github.io/superagent/) for http calls.
Ajax calls should live inside ActionCreators and ideally handle callbacks via Promises.

```javascript
const request = require('superagent');

UserActions.verifyEmail: function(dispatcher, email) {
    return new Promise(function(resolve, reject) {
        request
            .put('/email/verify')
            .csrf()
            .send({ email: email })
            .end(function(err, res) {
            // err will be set on any non-2xx response
            if(err) {
                reject(err);
            } else {
                dispatcher.dispatch('userResetInfo', res.body);
                resolve(res.body);
            }
      });
  });
};
```

Most of the time it's ok to ignore the resulting promise as the state transitions
will be handled by the dispatcher. However, many components benefit from using
the promise to track the loading state.

```javascript
// UserEmailVerify.js
//........
    handleVerifySubmit(email) {
        this.setState({
            isLoading: true
        });
        UserActions.verifyEmail(this.context.dispatcher, email)
            .then(() => {
                this.setState({
                    isLoading: false
                });
            })
            .catch(() => {
                // Don't forget to handle the error case!
                this.setState({
                    isLoading: false
                });
            });
    },
//.........
```

## Replacing library patterns

We have deprecated usage of jQuery and underscore within our codebase. Most of the utility functions they provide are readily available in ES6.

### Merging objects:

Don't do

```javascript
_.extend(object1, object2, object3, ...);
$.extend(object1, object2, object3, ...);
```

Do:

```javascript
Object.assign(object1, object2, object3, ...)
```

**[tip]** Object.assign has the side effect of modifying object1. If you want to
get a new copy of object then pass an empty object as the first argument. This is
also a good way to set default properties:

```javascript
position = Object.assign({
    position: "absolute"
}, position);
```

### Checking if an object is an array

Use `Array.isArray`

```javascript
Array.isArray(someObject);
```

### Checking if an object is a function

```javascript
typeof someObject === "function";
```

### Iterating over collections

Both jQuery and underscore provide shorthand functions for iterating over key-value
pairs in objects.


Don't do:

```javascript
$.each(someObject, function(key, value) {
});
_.each(someObject, function(value, key) {
});

for (let i = 0; i < someArray.length; i++) {
}

for (let i in someObject) {}
```

Yes do:

```javascript
// Iterating over values on an array
someArray.forEach(function(index, value) {});

// Iterating over key value pairs on an object.
Object.keys(someObject).forEach(function(key) {
    const value = someObject[key];
});
```

- `Array.prototype.forEach` returns undefined. This is different from the library
functions which return the called object.

### Testing values in array

Use the built-in array methods for testing if values are present in the array.

```javascript
_.every(someArray, function(value, index) {});
_.some(someArray, function(value, index) {});
```

```javascript
someArray.every(function(value, index) {});
someArray.some(function(value, index) {});
```

### Array filtering

Don't do

```javascript
$.grep(array, function() {});
_.filter(array, function() {});
```

Do

```javascript
array.filter(function() {});
```
