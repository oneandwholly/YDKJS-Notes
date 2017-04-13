# What is `this`?

`this` is a binding made for each function invocation, based entirely on **call-site**.

## Call-site

To understand `this`, we have to understand call-site: the location in code where a function is called.(**not where it's declared**).

It is not as easy as 'go locate where the function is called'. We have to think about the **call stack**(stack of functions executed to get the current execution context).
```javascript
function baz() {
  // call-stack is: `baz`
  // so, our call-site is in the global scope

  console.log("baz");
  bar(); // <- call-site for `bar`
}

function bar() {
  // call-stack is: `baz` -> `bar`
  // so, our call-site in in 'baz'

  console.log("bar");
  foo(); // <- call-site for `foo`
}

function foo() {
  // call-stack is: `baz` -> `bar` -> `foo`
  // so, our call-site is in `bar`

  console.log("foo");
  debugger;
}

baz(); // <- call-site for `baz`
```

call-site is the only thing that matters for `this` binding.

you can use chrome's debugger tool to visualize call-stack.

## Rules for determining `this` binding

you must determine which of the 4 rules apply

### Default binding

```javascript
function foo() {
	console.log( this.a );
}

var a = 2;

foo(); // 2
```

`this.a` resolves to the global variable `a`, because the default binding applies to the function call.
`foo` is called with a plain, un-decorated function reference. None other rules apply, so default binding applies.

```javascript
function foo() {
  'use strict';

  console.log(this.a);
}
var a = 2;

foo(); // TypeError
```

In `strict mode`, `this` points to `undefined`.

```javascript
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```

even though `this` binding is entirely based on the call-site, the global object is eligible for default binding only if the content in `foo` is executed in non-strict mode.

 
