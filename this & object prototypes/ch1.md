# `this` keyword

the keyword `this` is automatically defined in the scope of every function.

## Motivation and Utility of `this`

```javascript
function identify() {
  return this.name.toUpperCase();
}

function speak() {
  var greeting = "Hello, I am " + identify.call(this);
  console.log(greeting);
}

var me = {
  name: "Jon"
};

var you = {
  name: "Reader"
};

identify.call(me); // JON
identify.call(you); // READER

speak.call(me); // Hello, I am JON
speak.call(you); // Hello, I am READER
```

`identify()` and `speak()` can be re-used against multiple context objects (`me` and `you`), instead of making separate versions of the function for each object.

you can also explicitly pass in the context object.
```javascript
function identify(context) {
  return context.name.toUpperCase();
}

function speak(context) {
  var greeting = "Hello, I am " + identify(context);
  console.log(greeting);
}

identify(me); // JON
speak(you); // Hello, I am READER
```

`this` mechanism allows more elegant way of passing around object reference, which leads to cleaner API design and easier re-use. It becomes more apparent as the usage pattern gets more complex.

## Clarifying Misconceptions

The name `this` creates confusion when people take it too literally. There are two meaning often assumed mistakenly for `this`.

### itself

beginners often think `this` refers to the function object itself, which is incorrect.

In the following examples, we try to find how many times `foo()` was called.

```javascript
function foo(num) {
  console.log("foo: " + num);

  // keep track of how many times foo is called.
  this.count++;
}

foo.count = 0; // initialize count with 0.

var  i;

for (i=0; i<10; i++) {
  if (i > 5) {
    foo(i);
  }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was foo called?
console.log(foo.count) // 0 --- still
```

The code snippet shows `this.count` does not reference `foo.count`. Instead, there is a global variable `count` that is set to `Nan`, so `this` was referencing the window object.

At this point, people just try to get around the problem by hacking toward some other solution, such as:

```javascript
function foo(num) {
  console.log("foo: " + num);
  data.count++;
}

var data = {
  count: 0
};

var i;

for (i=0; i<10; i++) {
  if (i > 5) {
    foo(i);
  }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many time was `foo` called?
console.log(data.count); // 4
```

while this approach solves the problem, it does not help with out lack of understanding of `this`, and relies on lexical scope.

if you want to reference `foo` from inside the function itself, `this` won't be enough. you generally need a reference to the function object via variable that points at it.

```javascript
function foo() {
  foo.count = 4; // `foo` refers to itself
}

setTimeout(function() {
  // no way to refer to itself for anon. function
}, 100)
```

`arguments.callee` refers to the function object, and usually the only way to refer to itself in anonymous functions, but its use is frowned upon, so it is better to use named function when reference to itself is needed.

Other solution without using `this` and using the identifier `foo` to refence itself:

```javascript
function foo(num) {
  console.log("foo: " + num);
  foo.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
  if (i > 5) {
      foo(i);
  }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log(foo.count);
```

this example works, but it side-steps actual understanding of how `this` works

another solution is to force `this` to point to `foo` itself:

```javascript
function foo(num) {
  console.log("num: " + num);

  // keep track of how many times `foo` is called
  // `this` is actually `foo` now, base on
  // how `foo` is called
  this.count++;
}

foo.count = 0;

var i;

for (i=0; i < 10; i++) {
  if (i > 5) {
    foo.call(foo, i);
  }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log(foo.count); // 4
```

Instead of avoiding to use `this`, we embrace it!

###Its Scope

Another common misconception is to think `this` refers to the lexical scope.

```javascript
function foo() {
  var a = 2;
  this.bar();
}

function bar() {
  console.log(this.a);
}

foo(); //undefined
```

Trying to reference lexical scope with `this` won't work!

## What is `this` then?

`this` is not an author time binding, it is a runtime binding. It depends on the location of the function's invocation. It has nothing to do where the function was declared, but has everything to do with how it was called.

When a function is invoked, an active record, execution context is created. This record contains information about where the function was called from(the call-stack), how the function was invoked, what parameters were passed, etc. One of the properties of this record is the `this` binding which will be used for the duration of that function's execution.

## Review

`this` is not the function itself nor the lexical scope.

`this` is actually a binding that is made when a function is invoked, and what it references entirely depends on the location of function invocation.
