# Execution Context and Object Context - Clarifying `window` , `global` and `this` in JavaScript.

## Introduction - This article is for clarity 

This article is created to give you a *grassroots* explanation of what execution context is in JS how it works. What is the relationship between `this`, JS Objects and an **Execution Context**. 
If you are a JavaScript developer , or study/know JS , it is not surprising to be confused by `this`. That confusion may stem from a hazy understanding of execution context and how it interacts with `this`.

Reading this post should give you the clarity you need to understand about what execution context, call stack, functions, objects, object context,  `this` keyword, `window` and `global` are, how they *relate* and *work* with each other, and their *differences*.
We will also ask a fundamental question - why objects don't have an execution context of their own and why `this` points to them *anyway* in execution context.

**Note** that this is **not** a complete deep-dive into the ECMAScript spec. But rather a basic overview meant to give you an idea and think with *clarity* about `this` in JavaScript.

If you are a JavaScript developer, or like learning things from grassroot basics you may enjoy this article.

---

## What is an Execution Context? 

To explain simply, understand Execution Context as a **metadata** *wrapper/container* for a block of code that contains all the necessary data to execute that block of code.
It holds:
 - Variables, arguments, and functions inside that function.
 - Rules for how to look things up (scope).

We call execution context metadata because:

- It describes and governs how the other code behaves (variables, `this`, scoping rules).
- This is the internal, behind-the-scenes mechanism of the engine.
- It exists for the engine's bookkeeping, not for direct programmer manipulation.


There are three main types of execution context: 

1. Global Execution Context: The default, base context. It is created when a script first starts running. It creates a global object (`window` in browsers, `global` in Node.js) and sets `this` to that global object.

2. Function Execution Context: A brand new context is created every time a function is called. Each function call get their own execution context, completely separate from other contexts.

3. Eval Execution Context: Created when code is executed inside the eval() function. (Generally rare and not recommended for security/performance reasons).
*We are going to ignore eval execution context in this article.*

```javascript
// This is how the JavaScript engine sees an Execution Context (conceptually)

ExecutionContext = {
    // Internal fields (you never see these directly):
    VariableEnvironment: { ... },    // Where local variables live
    LexicalEnvironment: { ... },     // For scope chain
    ThisBinding: ???,                // ← This stores the current 'this' value
    
    // Other internal metadata...
}

// When your code uses 'this', the engine looks up the ThisBinding field

```

### Call Stack : Where and How are the Execution Contexts Stored ? 
 

Everytime we call a function, a new execution context is created, but execution context is a data(metadata), creating and storing it consumes *memory* - **RAM (Random Access Memory)** to be specific. It is created when you <u>run the code</u>. The amount used by a single context memory is small (relatively), but it's <u>very real</u>. The contexts are organized in RAM through a list called **Call Stack**. 
When the program runs, call stack organises the execution context in a LIFO (Last In First Out) fashion. Each function that are called have their metadata pushed to the top of the stack and are removed after the function executes.

```javascript
let globalVar = "I'm global";

function outer() {
  let outerVar = "I'm outer";
  
  function inner() {
    let innerVar = "I'm inner";
    console.log(innerVar); // inner's Execution Context is on top of stack
  }
  
  inner(); // inner's context is created and pushed
  // inner's context is popped off the stack after it finishes
}

outer(); // outer's context is created and pushed
// outer's context is popped off the stack after it finishes

// Global Execution Context remains until the program ends.

```

### Do Objects have an Execution Context of their Own? 


Reading all this, you might wish to ask , or perhaps I should clarify : **Do objects have a execution context of their own? Just like functions do?**

The answer is No. Execution contexts track the **Flow of Control**, not the structure of data. Objects are data types that simply exist, they don't do anything.

| Entity | Creates Execution Context? | Why? |
| :--- | :---: | :--- |
| **Function** | **YES** | Functions **do** something when called; calling them triggers a new context to manage their execution. |
| **Object** | **NO** | Objects **are** something (data storage); they exist in memory but do not execute code on their own. |
| **Constructor** | **YES** | A constructor is a specific type of function. Using `new` invokes it, which creates a new execution context to initialize the instance. |
| **Prototype** | **NO** | Prototypes **are** something (inheritance blueprints); they define shared behavior but are not executed directly. |

```javascript
// An object literal - NO execution context created
const car = {
    brand: "Toyota",
    year: 2024,
    start: function() {  // The function INSIDE will get a context when CALLED
        console.log(`${this.brand} starting`);
    }
};

// The object 'car' just sits in memory as data.
// No "execution" happens when you merely access it:
console.log(car.brand);  // Just data lookup, no new context

// But when you CALL the function:
car.start();  // YES! NEW execution context created for this function call
```


## The `This` Keyword. 

`this` is a special keyword in JavaScript, that points to the object that is currently the context of execution. This is an implicit parameter. That's all.

So what does that mean? As we already discussed, an object does not have an execution context, it does not do anything.
Here, the statement "Object that is currently the context of execution" *means* The object that the current function is acting or operating upon. Or it can be said as ' `this` refers to the object that is calling the function. '

```javascript
const car = {
  brand: "Toyota",
  showBrand() {
    console.log(this.brand);
  }
};

car.showBrand(); // Output: "Toyota"
// 'this' refers to 'car' because 'car' called the function.
```
> Context Object (semantic) -->	"The object that the function is currently working on behalf of".

We know functions have their own context as metadata , now every executional context has a `thisBinding` property that lists the object that is calling the running function.

```javascript
// The internal Execution Context (in the JavaScript engine) looks like this:       
// (You never see this directly)                                                   

ExecutionContext = {
    VariableEnvironment: { /* local variables */ },
    LexicalEnvironment: { /* scope chain */ },
    ThisBinding: window,  // ← 'this' is a PROPERTY of the execution context
    // ↑ 'this' does NOT point TO the context, 
    //   'this' is a VALUE INSIDE the context
}

// In your code:
console.log(this);  // This gives you the ThisBinding value
                    
```
Anoter example, 
```javascript
// Think of it this way:

const executionContext = {           // Internal (you can't access this)
    variableEnv: { x: 5 },
    scopeChain: [...],
    thisBinding: myObject            // ← 'this' is just a property
};

// When you write 'this' in your code:
this    // You get executionContext.thisBinding
        
```

**So, Why is this stored as `thisBinding` in Execution Context Metadata?**
Because this can to be **different** for each function call, even when calling the *same* function.


```javascript
const obj1 = { name: "A", greet() { console.log(this.name); } };
const obj2 = { name: "B", greet: obj1.greet };

obj1.greet(); // "A" (this = obj1)
obj2.greet(); // "B" (this = obj2) — same function, different this!
```
> `this` is determined at call time, not definition time.

### Why is `this` important ? 


Importance 1: Method Reusability
```javascript
// Without 'this', you'd need separate functions for each object
const alice = { name: "Alice", greet: function() { return "Hi from Alice"; } };
const bob = { name: "Bob", greet: function() { return "Hi from Bob"; } };
// Duplicate code everywhere!

// With 'this', one function works for ALL objects
function reusableGreet() {
    return `Hi from ${this.name}`;
}

alice.greet = reusableGreet;
bob.greet = reusableGreet;
// One function, infinite objects!
```

Importance 2: Object-Oriented Programming
```javascript

// Without 'this', constructors couldn't work
function Person(name, age) {
    // 'this' allows us to refer to "the object being created"
    this.name = name;
    this.age = age;
    this.birthday = function() {
        this.age++;  // Updates the specific instance
    };
}

const alice = new Person("Alice", 30);
const bob = new Person("Bob", 25);

alice.birthday();  // Only alice's age increases to 31
console.log(alice.age);  // 31
console.log(bob.age);    // 25 (unchanged)

```
Importance 3: Dynamic Context

```javascript
// 'this' allows functions to work with different objects dynamically
function updateProperty(value) {
    this.property = value;
}

const obj1 = {};
const obj2 = {};
const obj3 = {};

// Same function, three different targets
updateProperty.call(obj1, "first");
updateProperty.call(obj2, "second");
updateProperty.call(obj3, "third");

console.log(obj1.property);  // "first"
console.log(obj2.property);  // "second"
console.log(obj3.property);  // "third"
```

Importance 4: Event Handling (Browser)

```javascript
// 'this' lets event handlers know WHICH element was clicked
document.querySelectorAll('button').forEach(button => {
    button.addEventListener('click', function() {
        // 'this' is the specific button that was clicked
        this.style.backgroundColor = 'red';
        this.disabled = true;
        console.log(`${this.id} was clicked`);
    });
});
// Without 'this', you'd need to pass the element manually every time
```

---
### Important:

>  The **arrow function** is an exception to the rule of `this` referring to the calling object.
> Arrow functions, *inherit* `this` from the surrounding (parent) execution context — at *definition time*, not call time.
  
  ```javascript
  const obj = {
  name: "My Object",
  regularFunc: function() {
    console.log(this.name);  // `this` = obj (because of dot rule)
  },
  arrowFunc: () => {
    console.log(this.name);  // `this` = whatever it was OUTSIDE obj
  }
};

obj.regularFunc(); // "My Object" ✅
obj.arrowFunc();   // undefined (or global name) — not "My Object"
  ```

---

## Trivia :


**List => `global` vs `window` Available APIs:** 

```javascript
// BROWSER window contains:
window.alert()      // Show popup
window.document     // DOM API
window.localStorage // Client storage
window.fetch()      // Network requests
window.requestAnimationFrame()
window.history
window.location

// NODE.JS global contains:
global.require()    // Module loading
global.module       // Current module
global.exports      // Module exports
global.process      // Process info
global.Buffer       // Binary data
global.setImmediate()
global.__dirname    // Current directory path
global.__filename   // Current file path
```
---

**Objects exist in heap memory, don't confuse Heap with Call Stack:**

| Memory Area | Stores | Execution Context? |
| :--- | :--- | :--- |
| **Call Stack** | Execution contexts (function calls), variables, primitives | **YES** - Grows/shrinks as functions run |
| **Heap** | Objects, arrays, prototypes, closures | **NO** - Persistent data storage |
| **Call Stack** | Fast, fixed-size, LIFO (Last-In, First-Out) | Small, organized |
| **Heap** | Slower, flexible, no fixed order | Large, unstructured |

---

**Here's another nice table/graph:**

```javascript

/*
┌─────────────────────────────────────────────┐
│         Execution Context (internal)        │
├─────────────────────────────────────────────┤
│ • VariableEnvironment                       │
│   - local variables                         │
│   - function declarations                   │
│                                             │
│ • LexicalEnvironment                        │
│   - scope chain references                  │
│                                             │
│ • ThisBinding ───────┐                      │
└──────────────────────│──────────────────────┘
                       │
                       │ (points to)
                       ↓
              ┌─────────────────┐
              │  The value of   │
              │    'this'       │
              │  in your code   │
              └─────────────────┘
*/

// In your code:
this  // Accesses the ThisBinding from the current execution context

```
---

**If you take a `this.xyz` here are some things it can refer to:**

```javascript
// 1. A regular object (most common)
const person = { name: "Alice", age: 30 };
person.describe = function() {
    console.log(this.name);     // 'this' = person object
    this.age++;                 // Modifies person.age
};
person.describe();  // 'this' points to person

// 2. The global object (window/global)
function showGlobal() {
    console.log(this.xyz);  // If xyz exists on global
}
showGlobal();  // 'this' = global object (window in browser)

// 3. undefined (in strict mode)
function showStrict() {
    "use strict";
    console.log(this);  // undefined
    // this.xyz would throw TypeError!
}
showStrict();

// 4. A primitive value (string, number, boolean)
function showPrimitive() {
    console.log(this);  // Could be "hello", 42, or true
    console.log(this.length);  // If string, logs its length
}
showPrimitive.call("hello");  // 'this' = "hello" (auto-boxed to String object)

// 5. null or undefined (intentionally)
function showNull() {
    console.log(this);  // null
}
showNull.call(null);  // 'this' = null

// 6. A new empty object (with constructor)
function Person(name) {
    this.name = name;   // 'this' = brand new empty object
    this.xyz = "custom"; // Adds property to new object
}
const alice = new Person("Alice");  // 'this' points to new object

// 7. Whatever you explicitly bind (using bind, call, apply)
const boundFunction = showGlobal.bind({ xyz: "custom value" });
boundFunction();  // 'this' = { xyz: "custom value" }
```
---

**A complete picture:**

```javascript
/*
1. OBJECTS exist in heap memory
   - They just hold data (properties)
   - They don't "do" anything
   - They have NO execution context

2. FUNCTIONS also exist in heap memory
   - They contain code
   - They are inert until called

3. EXECUTION CONTEXTS exist temporarily on call stack
   - Created when functions are CALLED
   - Contain metadata for that specific call
   - Include thisBinding field

4. thisBinding stores a REFERENCE to an object
   - Connects the function call to data
   - Allows the function to work with different objects
   - The referenced object remains just an object (no context)
*/
```

---
**Credits to: AI assistant - DeepSeek.**

**<u>If you actually read this far Thank You Very Much! This is my second article written, leave a comment if you enjoyed it or found anything problematic.</u>**
