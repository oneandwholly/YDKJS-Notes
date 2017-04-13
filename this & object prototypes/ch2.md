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
