# `this`

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
