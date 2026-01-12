# Module 4: Scope, Hoisting and Closures

## 4.1 Lexical Scope

"Scope is where JavaScript looks for variables. Lexical scope means the code structure determines what you can access, not when or where it runs."

Lexical scope (or static scope) means a variable's accessibility is determined by its physical location in the source code, set during coding/compiling, not by where the function is called at runtime.

### The Three Scopes: var vs let vs const

JavaScript has three ways to declare variables: `var`, `let`, and `const`. You should use `let` and `const` in modern code.

#### How Scope Works

```txt
SCOPE HIERARCHY:
1. Global Scope - accessible everywhere
2. Function Scope - created by functions (ALL three types)
3. Block Scope - created by {} (ONLY let and const)

var = Function-scoped (old, quirky)
let = Block-scoped (modern, preferred)
const = Block-scoped (modern, preferred)
```

#### The var Problem: Function Scope Leakage

Variables declared with `var` ignore block boundaries. They leak out of `if` statements and loops into the entire function.

```javascript
function demonstrateVar() {
  if (true) {
    var leaked = "I escape blocks!";  // var ignores block boundaries
  }
  console.log(leaked);  // ✓ Works - var is function-scoped
}

demonstrateVar();  // "I escape blocks!"

// Same variable exists in the entire function
function counterExample() {
  for (var i = 0; i < 3; i++) {
    console.log("Loop:", i);
  }
  console.log("After loop:", i);  // ✓ i = 3 (var escaped the loop!)
}

counterExample();
// Loop: 0
// Loop: 1
// Loop: 2
// After loop: 3
```

#### The let and const Solution: Block Scope

`let` and `const` respect block boundaries. Declare them in a block, they stay in that block.

```javascript
function demonstrateLetConst() {
  if (true) {
    let safe = "I stay in my block";
    const alsoSafe = "Me too!";
  }
  // console.log(safe);      // ✗ ReferenceError
  // console.log(alsoSafe);  // ✗ ReferenceError
}

demonstrateLetConst();

// Variables are properly contained
function counterExampleFixed() {
  for (let i = 0; i < 3; i++) {
    console.log("Loop:", i);
  }
  // console.log(i);  // ✗ ReferenceError - i doesn't exist here
}

counterExampleFixed();
// Loop: 0
// Loop: 1
// Loop: 2
// Error! i is not defined
```

#### Practical Example

```javascript
function processRegistrations() {
  const eventId = 101;  // Function scope (const)

  if (eventId === 101) {
    // BLOCK SCOPE (let and const stay here)
    const message = "Processing JavaScript event";  // Block scope
    let count = 0;                                  // Block scope
    var legacy = "I'm function-scoped!";            // Function scope (var)

    for (let i = 0; i < 3; i++) {
      count++;  // count is accessible because it's in parent block
      console.log("Iteration " + i);
    }

    // At end of if block:
    console.log("count:", count);      // ✓ 3 (still in block)
    console.log("message:", message);  // ✓ accessible (still in block)
    // console.log(i);                 // ✗ ReferenceError (i is block-scoped to for)
  }

  // At end of if block:
  console.log("legacy:", legacy);  // ✓ "I'm function-scoped!" (var escaped!)
  // console.log(count);           // ✗ ReferenceError (count is block-scoped)
  // console.log(message);         // ✗ ReferenceError (message is block-scoped)
}

processRegistrations();
```

```txt
GLOBAL SCOPE
│
└─ processRegistrations() FUNCTION SCOPE
   │
   └─ eventId (const) - accessible here and nowhere else
   │
   └─ if block SCOPE
      │
      ├─ message (const) - only in this block
      ├─ count (let) - only in this block
      ├─ legacy (var) - hoisted to function scope! ⚠️
      │
      └─ for loop BLOCK SCOPE
         │
         └─ i (let) - only in this loop!
```

### Scope Chain

When JavaScript needs a variable, it starts searching in the current scope. If it doesn't find it, it checks the parent scope. Then the grandparent. It keeps going outward until it reaches global scope. If it still hasn't found the variable, you get a ReferenceError.

```javascript
const globalVar = "global";  // 🔴 Global scope

function outer() {
  const outerVar = "outer";  // 🔵 Outer function scope

  function inner() {
    const innerVar = "inner";  // 🟢 Inner function scope

    // When JavaScript looks for a variable, it searches:
    // 1. Current scope (green) first
    // 2. If not found, then Parent scope (blue)
    // 3. If still not found, then Grandparent scope (red)
    // 4. If not found anywhere: ReferenceError!

    console.log(innerVar);   // 🟢 Found in own scope
    console.log(outerVar);   // 🔵 Found in parent scope
    console.log(globalVar);  // 🔴 Found in global scope
  }

  inner();

  // outer() can only see blue and red, not green!
  console.log(outerVar);    // 🔵 Found
  console.log(globalVar);   // 🔴 Found
  // console.log(innerVar); // ❌ ReferenceError - not in outer or global
}

outer();
```

### Event Management System

```javascript
// Think of scopes as colored buckets, and variables as colored marbles
// Each variable (marble) belongs to exactly one scope (bucket)

const APP_NAME = "GHW Events";  // Global - available everywhere

function createEventManager() {
  // This scope has access to: APP_NAME (global)
  let totalEvents = 0;

  function addEvent(eventTitle) {
    // This scope has access to: APP_NAME, totalEvents, and own variables
    const timestamp = Date.now();  // Local to addEvent
    totalEvents++;

    console.log(`[${APP_NAME}] Added: ${eventTitle} at ${timestamp}`);
    return { title: eventTitle, timestamp };
  }

  function getStats() {
    // This scope has access to: APP_NAME, totalEvents
    return {
      appName: APP_NAME,      // From global
      eventCount: totalEvents  // From parent function
    };
  }

  // Return both functions (they remember their parent scope!)
  return { addEvent, getStats };
}

const manager = createEventManager();

manager.addEvent("JavaScript Workshop");
manager.addEvent("React Fundamentals");

console.log(manager.getStats());

// Variables from inner scopes are NOT accessible:
// console.log(totalEvents);  // ✗ ReferenceError

// When JavaScript looks for a variable:
// 1. Check current scope
// 2. Check parent scope
// 3. Check grandparent scope
// ... continue until global scope
// If not found anywhere: ReferenceError!
```

## 4.2 Hoisting & TDZ

JavaScript processes your code in two passes. First pass: it finds all declarations. Second pass: it executes the code. This is why you can sometimes use things before they're declared. But there are rules.

### Understanding Hoisting

With `var`, the declaration gets processed first, but the assignment stays where you wrote it. Function declarations get fully processed in the first pass.

```javascript
// JavaScript "hoists" declarations to the top of their scope
// But initialization stays where you wrote it

// What you write:
console.log(eventName);  // undefined (not an error!)
var eventName = "Workshop";

// What JavaScript sees:
// var eventName;           // Declaration hoisted
// console.log(eventName);  // undefined
// eventName = "Workshop";  // Assignment stays here

// Function declarations are fully hoisted
sayHello();  // Works!

function sayHello() {
  console.log("Hello!");
}

// Function expressions are NOT hoisted
// greet();  // TypeError: greet is not a function
var greet = function() {
  console.log("Hi!");
};
```

### Temporal Dead Zone (TDZ)

`let` and `const` are also hoisted, but you can't use them before their declaration line. The time between the start of the block and the declaration is called the Temporal Dead Zone. Try to access the variable during this time and you get an error.

```javascript
// let and const ARE hoisted, but into a "temporal dead zone"
// You can't access them before their declaration

// console.log(title);  // ReferenceError: Cannot access 'title' before initialization
let title = "JavaScript Workshop";
console.log(title);     // "JavaScript Workshop"

// The variable EXISTS but is in the TDZ until the declaration
{
  // TDZ starts for 'workshop'
  // console.log(workshop);  // ReferenceError!
  let workshop = "React";     // TDZ ends here
  console.log(workshop);      // "React"
}

// This is why let/const are safer than var
// They fail loudly instead of silently returning undefined
```

## 4.3 Closures

A closure is when a function remembers variables from its parent scope, even after the parent function finishes executing.

### How Closures Work

When you create a function inside another function, the inner function gets access to the outer function's variables. That inner function "closes over" those variables. Even if the outer function returns and is done executing, the inner function still has access to those variables.

```javascript
function createCounter() {
  let count = 0;  // This variable lives in createCounter's scope

  return function increment() {
    count++;  // This inner function can still access count
    console.log(count);
  };
}

const counter1 = createCounter();
counter1();  // 1
counter1();  // 2
counter1();  // 3

const counter2 = createCounter();
counter2();  // 1 (separate closure, separate count variable)
```

Each time you call `createCounter()`, you create a new closure with its own `count` variable. The returned function remembers its specific `count`.

### Why Closures Matter

Closures let you create `private` variables in JavaScript.

```javascript
function createBankAccount(initialBalance) {
  let balance = initialBalance;  // Private variable, since let is block-scoped

  return {
    deposit(amount) {
      balance += amount; // Inner function can access balance
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) {
        console.log("Insufficient funds");
        return balance;
      }
      balance -= amount;
      return balance;
    },
    getBalance() {
      return balance;
    }
  };
}

const myAccount = createBankAccount(100); // balance is private, not accessible from outside, but methods can access it
myAccount.deposit(50);    // 150
myAccount.withdraw(30);   // 120
console.log(myAccount.getBalance());  // 120

// You can't access balance directly
console.log(myAccount.balance);  // undefined
```

The `balance` variable is truly private. The only way to interact with it is through the methods that have access via closure.

### The Classic var Loop Problem

Here's where understanding closures becomes critical. This is a famous gotcha:

```javascript
// BROKEN: Using var
function createHandlers() {
  const handlers = [];

  for (var i = 0; i < 3; i++) {
    handlers.push(() => console.log(i));
  }

  return handlers;
}

const broken = createHandlers();
broken[0]();  // 3 (not 0!)
broken[1]();  // 3 (not 1!)
broken[2]();  // 3 (not 2!)
```

Why do they all print 3? Because `var` is function-scoped. There's only ONE `i` variable shared by all the closures. By the time you call the handlers, the loop finished and `i` is 3.

```javascript
// FIXED: Using let
function createHandlers() {
  const handlers = [];

  for (let i = 0; i < 3; i++) {  // let creates a new binding each iteration
    handlers.push(() => console.log(i));
  }

  return handlers;
}

const fixed = createHandlers();
fixed[0]();  // 0
fixed[1]();  // 1
fixed[2]();  // 2
```

With `let`, each loop iteration gets its own `i` variable. Each closure captures its own separate `i`.

### Practical Example: Event Registration System

```javascript
function findAvailableEvents(events, registrations) {
  const available = [];

  for (let i = 0; i < events.length; i++) {
    const event = events[i];  // New binding for each iteration

    const currentRegistrations = registrations.filter(r => r.eventId === event.id);
    const spotsLeft = event.capacity - currentRegistrations.length;

    if (spotsLeft > 0) {
      available.push({
        ...event,
        spotsLeft,
        // This closure captures the correct `event` because of let
        register: () => {
          console.log(`Registering for ${event.title}`);
          return event.id;
        },
        // Another closure capturing the same event
        checkAvailability: () => {
          return `${event.title} has ${spotsLeft} spots left`;
        }
      });
    }
  }

  return available;
}

// Usage
const events = [
  { id: 1, title: "JavaScript Workshop", capacity: 30 },
  { id: 2, title: "React Fundamentals", capacity: 25 }
];
const registrations = [
  { eventId: 1, userId: 101 },
  { eventId: 1, userId: 102 }
];

const availableEvents = findAvailableEvents(events, registrations);
availableEvents[0].register();  // "Registering for JavaScript Workshop"
console.log(availableEvents[0].checkAvailability());  // "JavaScript Workshop has 28 spots left"
```

Each returned object has methods that are closures. They remember their specific `event` and `spotsLeft` values, even though the loop has moved on or finished.

---

[Previous Module: 03_coercion_and_equality.md](03_coercion_and_equality.md) | [Next Module: 05_objects_in_depth.md](05_objects_in_depth.md)
