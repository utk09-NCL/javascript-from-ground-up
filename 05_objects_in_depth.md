# Module 5: Objects In-Depth

## 5.1 The this Keyword

### Why this Exists

`this` is JavaScript's way of letting a function know which object it should operate on. Without `this`, every function would need you to pass the object as an argument.

```javascript
// Without this - tedious and repetitive
const event1 = { title: "Workshop", capacity: 30 };
const event2 = { title: "Hackathon", capacity: 50 };

function showDetailsWithoutThis(eventObject) {
  console.log(eventObject.title + " (" + eventObject.capacity + " spots)");
}

showDetailsWithoutThis(event1);  // Have to pass the object every time
showDetailsWithoutThis(event2);

// With this - cleaner and reusable
function showDetails() {
  console.log(this.title + " (" + this.capacity + " spots)");
}

const event3 = { title: "Workshop", capacity: 30, show: showDetails };
const event4 = { title: "Hackathon", capacity: 50, show: showDetails };

event3.show();  // this automatically refers to event3
event4.show();  // this automatically refers to event4
```

`this` lets you write one function that works with any object. The same function can access different objects' properties depending on how you call it.

### How this Gets Determined

Here's the key: `this` is determined by how you call the function, not where you write it. JavaScript looks at the call site and decides what `this` should be.

### Default Binding

```javascript
function showThis() {
  console.log(this);
}

showThis();  // In browser: window, in Node: global, in strict mode: undefined
```

### What is Strict Mode?

Strict mode is a way to opt in to a restricted variant of JavaScript. It helps catch common coding mistakes and "unsafe" actions such as gaining access to the global object.

It is invoked by adding `"use strict";` at the beginning of a script or function.

Read more: [Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)

### Implicit Binding (Object Method)

When you call a function as a method of an object (with a dot), `this` points to that object. The object before the dot becomes `this`.

```javascript
const event = {
  title: "JavaScript Workshop",
  showTitle: function() {
    console.log(this.title);
  }
};

event.showTitle();  // "JavaScript Workshop"
// this = the object before the dot (event)

// But watch what happens when you detach the function!
const detached = event.showTitle;
detached();  // undefined (or error in strict mode)
// Why? Because there's no object before the dot anymore.
// You're calling detached() not event.detached()
// JavaScript can't find an object, so this falls back to default binding
```

This is a common gotcha when passing methods as callbacks:

```javascript
const event = {
  title: "React Workshop",
  showTitle: function() {
    console.log(this.title);
  }
};

// This breaks because setTimeout calls the function without an object context
setTimeout(event.showTitle, 1000);  // undefined

// Fix 1: Wrap it in an arrow function
setTimeout(() => event.showTitle(), 1000);  // "React Workshop"

// Fix 2: Use bind (explained below)
setTimeout(event.showTitle.bind(event), 1000);  // "React Workshop"
```

### Explicit Binding (call, apply, bind)

Sometimes you want to force a function to use a specific object as `this`. JavaScript gives you three methods to do this explicitly.

```javascript
function introduce(greeting, punctuation) {
  console.log(greeting + ", I'm " + this.name + punctuation);
}

const participant1 = { name: "Alice" };
const participant2 = { name: "Bob" };

// call - invoke immediately with this set to the first argument
// Remaining arguments are passed to the function
introduce.call(participant1, "Hello", "!");  // "Hello, I'm Alice!"
introduce.call(participant2, "Hi", ".");     // "Hi, I'm Bob."

// apply - same as call, but arguments as an array
// Useful when you already have arguments in an array
const args = ["Hey", "!!"];
introduce.apply(participant1, args);  // "Hey, I'm Alice!!"

// bind - creates a NEW function with this permanently set
// Doesn't invoke immediately, returns a new function
const aliceIntro = introduce.bind(participant1);
aliceIntro("Hello", "!");  // "Hello, I'm Alice!"
aliceIntro("Hi", ".");     // "Hi, I'm Alice."
// No matter how you call aliceIntro, this is always participant1

// bind is perfect for callbacks
const event = {
  title: "Workshop",
  showTitle: function() {
    console.log(this.title);
  }
};

setTimeout(event.showTitle.bind(event), 1000);  // "Workshop"
```

### Arrow Functions and this

Arrow functions are fundamentally different. They don't have their own `this` at all. Instead, they capture `this` from the surrounding scope when they're created. This is called "lexical this".

#### Why Arrow Functions Were Created

Before arrow functions, using `this` inside callbacks was a nightmare:

```javascript
const event = {
  title: "Workshop",
  attendees: ["Alice", "Bob"],

  // Regular function has this problem
  showAttendeesOld: function() {
    console.log("Event:", this.title);  // "Workshop" - this works here

    this.attendees.forEach(function(name) {
      // Problem! This callback is a new function with its own this
      // When forEach calls it, there's no object before the dot
      // So this falls back to undefined (strict mode) or global
      console.log(name + " attending " + this.title);  // this.title is undefined!
    });
  }
};

event.showAttendeesOld();
// Event: Workshop
// Alice attending undefined
// Bob attending undefined
```

The old workaround was to save `this` in a variable:

```javascript
const event = {
  title: "Workshop",
  attendees: ["Alice", "Bob"],

  showAttendees: function() {
    const self = this;  // Save this in a variable

    this.attendees.forEach(function(name) {
      console.log(name + " attending " + self.title);  // Use self instead
    });
  }
};

event.showAttendees();
// Alice attending Workshop
// Bob attending Workshop
```

#### Arrow Functions to the Rescue

Arrow functions don't create their own `this`. They use whatever `this` is in the surrounding code:

```javascript
const event = {
  title: "Workshop",
  attendees: ["Alice", "Bob"],

  showAttendees: function() {
    // Arrow function doesn't create new this, uses this from showAttendees
    this.attendees.forEach(name => {
      console.log(name + " attending " + this.title);  // this is still event!
    });
  }
};

event.showAttendees();
// Alice attending Workshop
// Bob attending Workshop
```

#### Important: Don't Use Arrow Functions as Methods

Because arrow functions capture `this` from surrounding scope, don't use them as object methods:

```javascript
const event = {
  title: "Workshop",

  // WRONG! Arrow function as method
  showTitle: () => {
    console.log(this.title);  // this is NOT event, it's whatever this is outside the object
  }
};

event.showTitle();  // undefined (or whatever global this is)
```

Arrow functions capture `this` from where the object is created (often global scope), not from the object itself.

#### When to Use Each

```javascript
const event = {
  title: "Workshop",
  attendees: ["Alice", "Bob"],

  // Use regular function for methods (you want this to be the object)
  showAttendees: function() {
    console.log("Event:", this.title);

    // Use arrow function for callbacks (you want to keep the method's this)
    this.attendees.forEach(name => {
      console.log(name + " is attending " + this.title);
    });
  }
};
```

**Rule of thumb:**

- Object methods: Use regular functions
- Callbacks inside methods: Use arrow functions
- Need to preserve `this` from outer scope: Use arrow functions
- Need `this` to change based on how it's called: Use regular functions

---

[Previous Module: 04_scope_hoisting_and_closures.md](04_scope_hoisting_and_closures.md) | [Next Module: 06_prototypes_and_classes.md](06_prototypes_and_classes.md)
