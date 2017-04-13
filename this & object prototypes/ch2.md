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

### Implicit binding

Another rule to consider is: does the call-site have a context object?

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  foo: foo,
  a: 2
};

obj.foo(); //2
```
Firstly, notice the manner in which foo() is declared and then later added as a reference property onto obj. Regardless of whether foo() is initially declared on obj, or is added as a reference later (as this snippet shows), in neither case is the function really "owned" or "contained" by the obj object.

However, the call-site uses the obj context to reference the function, so you could say that the obj object "owns" or "contains" the function reference at the time the function is called.

Whatever you choose to call this pattern, at the point that foo() is called, it's preceded by an object reference to obj. When there is a context object for a function reference, the implicit binding rule says that it's that object which should be used for the function call's this binding.

Only the top level of an object property chain matters

```javascript
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

### Implicitly Lost
A common frustration is when an implicitly bound function loses that binding.

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // function reference/alias!

var a = "oops, global"; // `a` also property on global object

bar(); // "oops, global"
```

even though `bar` appears to be reference for `obj.foo`, but, in fact, it is just another reference to `foo`

another common way this happens is when you pass around a function as call-back function

```javascript
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` is just another reference to `foo`

	fn(); // <-- call-site!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

doFoo( obj.foo ); // "oops, global"
```

### Explicit binding

With implicit binding as we just saw, we had to mutate the object in question to include a reference on itself to the function, and use this property function reference to indirectly (implicitly) bind this to the object.

But, what if you want to force a function call to use a particular object for the this binding, without putting a property function reference on the object?

All functions have some utilities available to them which can be useful for this task.Specifically, functions have call(..) and apply(..) methods. Technically, JavaScript host environments sometimes provide functions which are special enough (a kind way of putting it!) that they do not have such functionality. But those are few. The vast majority of functions provided, and certainly all functions you will create, do have access to call(..) and apply(..)

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

Invoking foo with explicit binding by foo.call(..) allows us to force its this to be obj.

### hard binding

```javascript
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

var bar = function() {
	foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` hard binds `foo`'s `this` to `obj`
// so that it cannot be overriden
bar.call( window ); // 2
```

We create a function bar() which, internally, manually calls foo.call(obj), thereby forcibly invoking foo with obj binding for this. No matter how you later invoke the function bar, it will always manually invoke foo with obj.

```javascript
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}

var obj = {
  a: 2
};

var bar = function() {
  return foo.apply(obj, arguments);
}

var b = bar(3);
console.log(b);
```

another way to express this pattern is to use re-usable helper
```javascript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

// simple `bind` helper
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

var obj = {
	a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

ES5 provides `bind` function

```javascript
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

bind function returns a new function with `this` bound to whatever object you pass in the the argument.

### `new` binding

In javascript, constructors and just regular functions that are called with `new` keyword.

 there's really no such thing as "constructor functions", but rather construction calls of functions.

When a function is invoked with new in front of it, otherwise known as a constructor call, the following things are done automatically:

1. a brand new object is created (aka, constructed) out of thin air
2. the newly constructed object is [[Prototype]]-linked
3. the newly constructed object is set as the this binding for that function call
4. unless the function returns its own alternate object, the new-invoked function call will automatically return the newly constructed object.

```javascript
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

By calling foo(..) with new in front of it, we've constructed a new object and set that new object as the this for the call of foo(..). So new is the final way that a function call's this can be bound. We'll call this new binding.
