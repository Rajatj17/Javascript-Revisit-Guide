# Module 1: Execution Context & Hoisting Foundation

## Why This Module Matters
Understanding execution context is fundamental to JavaScript mastery. It explains why hoisting works, how scope chains are formed, and sets the foundation for understanding closures, `this` binding, and async behavior.

## Core Concept: How JavaScript Really Executes

JavaScript doesn't execute line by line. It has **two distinct phases**:

### Phase 1: Creation Phase (Compilation)
- Creates execution context
- Sets up memory space (hoisting happens here)
- Establishes scope chains
- Determines `this` value

### Phase 2: Execution Phase
- Executes code line by line
- Assigns values to variables
- Calls functions

## Types of Execution Contexts

### 1. Global Execution Context (GEC)
```javascript
console.log("This runs in Global Execution Context");

// Creation phase creates:
// - Global object (window in browser, global in Node.js)
// - this = global object
// - Hoists var declarations and function declarations
```

### 2. Function Execution Context (FEC)
```javascript
function myFunction() {
    console.log("This creates a new Function Execution Context");
    
    // Each function call creates its own context:
    // - arguments object
    // - this binding
    // - Local scope
}

myFunction(); // Creates new FEC
myFunction(); // Creates another new FEC
```

### 3. Eval Execution Context (rarely used)
```javascript
// Created by eval() - avoid in production
eval("var x = 10;");
```

## Hoisting Deep Dive

### What Actually Gets Hoisted?

#### Variable Declarations (var)
```javascript
// What you write:
console.log(myVar); // undefined
var myVar = 5;
console.log(myVar); // 5

// How JavaScript sees it:
var myVar; // Declaration hoisted, initialized with undefined
console.log(myVar); // undefined
myVar = 5; // Assignment stays in place
console.log(myVar); // 5
```

#### Function Declarations
```javascript
// What you write:
console.log(getName()); // "Alice" - works!

function getName() {
    return "Alice";
}

// How JavaScript sees it:
function getName() {    // Entire function hoisted
    return "Alice";
}
console.log(getName()); // "Alice"
```

#### Function Expressions (NOT hoisted)
```javascript
// What you write:
console.log(getAge()); // TypeError: getAge is not a function

var getAge = function() {
    return 30;
};

// How JavaScript sees it:
var getAge; // Only declaration hoisted, value is undefined
console.log(getAge()); // Trying to call undefined()
getAge = function() {
    return 30;
};
```

### Let and Const Hoisting

#### The Temporal Dead Zone (TDZ)
```javascript
console.log(a); // ReferenceError: Cannot access 'a' before initialization
console.log(b); // ReferenceError: Cannot access 'b' before initialization

let a = 1;
const b = 2;

// They ARE hoisted, but not initialized
// TDZ = from start of scope until declaration line
```

#### Block Scope vs Function Scope
```javascript
function testScoping() {
    // TDZ for 'letVar' starts here
    
    if (true) {
        // TDZ for 'letVar' continues here
        console.log(letVar); // ReferenceError
        let letVar = "block scoped";
    }
    
    // letVar doesn't exist here
    console.log(letVar); // ReferenceError
}

function testVarScoping() {
    if (true) {
        var varVar = "function scoped";
    }
    
    console.log(varVar); // "function scoped" - accessible!
}
```

## Complex Hoisting Scenarios

### Scenario 1: Variable vs Function Name Collision
```javascript
console.log(foo); // [Function: foo]

var foo = "I'm a variable";
function foo() {
    return "I'm a function";
}

console.log(foo); // "I'm a variable"

// Explanation:
// Creation phase:
// 1. var foo = undefined (hoisted)
// 2. function foo() {...} (hoisted and overwrites undefined)
// 
// Execution phase:
// 1. foo = "I'm a variable" (assignment overwrites function)
```

### Scenario 2: Functions Inside Blocks
```javascript
console.log(typeof bar); // "undefined" in strict mode

if (true) {
    function bar() {
        return "block function";
    }
}

// In non-strict mode: function might be hoisted
// In strict mode: function is block-scoped
```

### Scenario 3: Nested Function Hoisting
```javascript
function outer() {
    console.log(inner); // [Function: inner]
    
    function inner() {
        return "I'm inner";
    }
    
    console.log(inner()); // "I'm inner"
}

outer();
```

## Execution Context Stack (Call Stack)

### How the Stack Works
```javascript
function first() {
    console.log("Inside first");
    second();
    console.log("Back to first");
}

function second() {
    console.log("Inside second");
    third();
    console.log("Back to second");
}

function third() {
    console.log("Inside third");
}

first();

// Call Stack visualization:
// 1. [Global Context]
// 2. [Global Context, first()]
// 3. [Global Context, first(), second()]
// 4. [Global Context, first(), second(), third()]
// 5. [Global Context, first(), second()] // third() popped
// 6. [Global Context, first()] // second() popped  
// 7. [Global Context] // first() popped
```

## Variable Environment vs Lexical Environment

### Variable Environment
- Where var declarations and function declarations live
- Created during creation phase

### Lexical Environment
- Where let/const declarations live
- Has reference to outer lexical environment (scope chain)

```javascript
function outer() {
    var outerVar = "outer";
    
    function inner() {
        var innerVar = "inner";
        console.log(outerVar); // Can access through scope chain
    }
    
    inner();
}

// Scope chain: inner's lexical env → outer's lexical env → global lexical env
```

## Common Interview Traps

### Trap 1: Loop Closures
```javascript
// Problem:
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 3, 3, 3 (not 0, 1, 2)
    }, 100);
}

// Why? All callbacks share the same lexical environment
// where i = 3 after loop completes

// Solutions:
// 1. Use let (block scoped)
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 0, 1, 2
    }, 100);
}

// 2. Create new scope with IIFE
for (var i = 0; i < 3; i++) {
    (function(j) {
        setTimeout(function() {
            console.log(j); // 0, 1, 2
        }, 100);
    })(i);
}
```

### Trap 2: Function Redeclaration
```javascript
// What happens here?
foo(); // "Second"

function foo() {
    console.log("First");
}

function foo() {
    console.log("Second");
}

foo(); // "Second"

// Both declarations are hoisted, second one overwrites first
```

### Trap 3: Mixed var and function
```javascript
console.log(typeof a); // "function"
console.log(typeof b); // "undefined"

var a = 1;
var b = 2;

function a() {
    console.log("function a");
}

// Creation phase:
// var a = undefined
// var b = undefined  
// function a() {...} (overwrites var a)
```

## Practical Debugging Techniques

### 1. Visualizing Execution Context
```javascript
function debugContext() {
    console.log("=== Creation Phase ===");
    console.log("Variables hoisted:", typeof myVar); // undefined
    console.log("Functions hoisted:", typeof myFunc); // function
    
    console.log("=== Execution Phase ===");
    var myVar = "Now assigned";
    
    function myFunc() {
        return "I was hoisted";
    }
    
    console.log("Variable after assignment:", myVar);
}
```

### 2. Checking Scope Chain
```javascript
function checkScope() {
    var level1 = "outer";
    
    function inner() {
        var level2 = "inner";
        console.log("From inner:", level1); // Can access outer
        
        function deepInner() {
            var level3 = "deep";
            console.log("From deep:", level1, level2); // Can access both
        }
        
        deepInner();
    }
    
    inner();
}
```

## Practice Challenges

### Challenge 1: Predict the Output
```javascript
var x = 1;
var y = 2;

function test() {
    console.log(x, y); // What gets logged?
    var x = 3;
    let y = 4;
    console.log(x, y); // What gets logged?
}

test();
console.log(x, y); // What gets logged?
```

### Challenge 2: Fix the Code
```javascript
// This doesn't work as expected - fix it
function createButtons() {
    var buttons = [];
    
    for (var i = 0; i < 3; i++) {
        buttons[i] = function() {
            console.log("Button " + i + " clicked");
        };
    }
    
    return buttons;
}

var myButtons = createButtons();
myButtons[0](); // Should log "Button 0 clicked"
myButtons[1](); // Should log "Button 1 clicked"
```

### Challenge 3: Execution Context Analysis
```javascript
// Trace through execution contexts and predict output
function a() {
    console.log("a1");
    b();
    console.log("a2");
}

function b() {
    console.log("b1");
    c();
    console.log("b2");
}

function c() {
    console.log("c1");
}

a();
```

## Key Takeaways for Interviews

1. **Hoisting is NOT physically moving code** - it's about creation phase memory allocation
2. **var creates undefined placeholders**, let/const create uninitialized bindings (TDZ)
3. **Function declarations are fully hoisted**, function expressions are not
4. **Each function call creates new execution context** with its own scope
5. **Scope chain is determined lexically** (where code is written, not where it's called)
6. **Understanding execution context explains closures, this binding, and variable access**

