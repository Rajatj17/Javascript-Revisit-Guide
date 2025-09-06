# JavaScript Interview Prep: Prerequisites & Reading Order

## 🎯 Essential Prerequisites (Read These First!)

### 1. Fundamental JavaScript Concepts
**Must understand before starting any module:**

#### Basic JavaScript Engine Concepts
- **How JavaScript is parsed and executed**
  - Compilation vs Interpretation
  - V8 engine basics (Chrome/Node.js)
  - Call stack fundamentals

#### Lexical Environment (CRITICAL!)
```javascript
// You need to understand this concept deeply
function outer() {
    var outerVar = "I'm outer";
    
    function inner() {
        var innerVar = "I'm inner";
        console.log(outerVar); // How does this work?
    }
    
    return inner;
}

// What happens here?
var myFunc = outer();
myFunc(); // How does inner() still access outerVar?
```

**Key concepts to understand:**
- Environment Record (where variables are stored)
- Outer Environment Reference (the scope chain)
- How environments are created and linked

#### Memory Management Basics
- **Stack vs Heap**
  - Primitives in stack
  - Objects in heap
  - References and garbage collection

---

## 📚 Module-by-Module Prerequisites

### Module 1: Execution Context & Hoisting
**Prerequisites to read first:**

1. **JavaScript Engine Execution Process**
   - Tokenization/Lexing
   - Parsing (AST creation)
   - Compilation phase
   - Execution phase

2. **Memory Allocation**
   - How variables are stored
   - Creation phase vs execution phase
   - Variable environment vs lexical environment

3. **Scope Fundamentals**
   - Global scope
   - Function scope
   - Block scope (ES6+)

**Key Resource:** 
- MDN: "JavaScript Execution Context"
- YDKJS: "Scope & Closures" (Chapter 1-2)

---

### Module 2: 'this' Binding System
**Prerequisites to read first:**

1. **Function Invocation Patterns**
   - Method invocation
   - Function invocation
   - Constructor invocation
   - apply/call invocation

2. **Object-Oriented JavaScript Basics**
   - Objects and properties
   - Methods vs functions
   - Constructor functions basics

3. **Arrow Functions vs Regular Functions**
   - Lexical vs dynamic binding
   - When arrow functions were introduced and why

**Key Resource:**
- YDKJS: "this & Object Prototypes" (Chapter 1-2)
- MDN: "this" keyword documentation

---

### Module 3: Closures & Scope Chain
**Prerequisites to read first:**

1. **Lexical Environment (DEEP DIVE)**
   - Environment Records
   - Outer Environment References
   - How scope chains are formed

2. **Function Lifecycle**
   - Function creation
   - Function execution
   - Function cleanup and GC

3. **First-Class Functions**
   - Functions as values
   - Functions as arguments
   - Functions as return values

**Key Resource:**
- YDKJS: "Scope & Closures" (Complete book)
- MDN: "Closures" documentation

---

### Module 4: Prototypes & Inheritance
**Prerequisites to read first:**

1. **Object Fundamentals**
   - Object creation patterns
   - Property descriptors
   - Object.defineProperty basics

2. **Constructor Functions**
   - How `new` operator works
   - Function.prototype property
   - Instance vs prototype properties

3. **JavaScript's Object Model**
   - Everything is an object (almost)
   - Prototype chain concept
   - Object.prototype and null

**Key Resource:**
- YDKJS: "this & Object Prototypes" (Chapter 3-6)
- MDN: "Object prototypes"

---

### Module 5: Asynchronous JavaScript
**Prerequisites to read first:**

1. **JavaScript Runtime Model**
   - Single-threaded nature
   - Call stack
   - Event loop basics
   - Task queue vs microtask queue

2. **Callback Pattern**
   - Higher-order functions
   - Error-first callbacks (Node.js style)
   - Callback hell problem

3. **Promise Fundamentals**
   - Promise states
   - Promise chaining
   - Error propagation

**Key Resource:**
- MDN: "Event Loop"
- YDKJS: "Async & Performance"
- JavaScript.info: "Promises, async/await"

---

### Module 6: Advanced Functions & Patterns
**Prerequisites to read first:**

1. **Functional Programming Concepts**
   - Pure functions
   - Immutability
   - Side effects

2. **Higher-Order Functions**
   - Functions that accept functions
   - Functions that return functions
   - Array methods (map, filter, reduce)

3. **Function Composition**
   - Combining functions
   - Data transformation pipelines

**Key Resource:**
- "Functional-Light JavaScript" by Kyle Simpson
- MDN: "Higher-order functions"

---

### Module 7: Error Handling & Debugging
**Prerequisites to read first:**

1. **Error Object and Types**
   - Built-in error types
   - Stack traces
   - Error propagation

2. **Browser DevTools Basics**
   - Console usage
   - Debugger statement
   - Breakpoints

3. **Testing Fundamentals**
   - Unit testing concepts
   - Assertion libraries
   - Mock vs stub

**Key Resource:**
- MDN: "Error" documentation
- Chrome DevTools documentation

---

### Module 8: Modern JavaScript & Best Practices
**Prerequisites to read first:**

1. **ES6+ Features Overview**
   - let/const vs var
   - Arrow functions
   - Template literals
   - Destructuring
   - Spread/rest operators

2. **Module Systems**
   - CommonJS vs ES modules
   - Import/export syntax
   - Module loading

3. **Build Tools Basics**
   - Webpack/bundling concepts
   - Transpilation (Babel)
   - Minification

**Key Resource:**
- "Exploring ES6" by Axel Rauschmayer
- MDN: "JavaScript modules"

---

## 🏃‍♂️ Recommended Learning Path

### Phase 1: Core Foundations (1-2 weeks)
1. **JavaScript Engine & Execution**
   - Read: "How JavaScript Works" series
   - Practice: Basic execution context examples

2. **Lexical Environment Deep Dive**
   - Read: YDKJS "Scope & Closures" Ch 1-2
   - Practice: Scope chain tracing exercises

3. **Basic Object Model**
   - Read: Objects and prototypes basics
   - Practice: Object creation patterns

### Phase 2: Advanced Concepts (2-3 weeks)
1. **Complete Modules 1-3** (Execution Context → this → Closures)
2. **Complete Module 4** (Prototypes)
3. **Complete Module 5** (Async JavaScript)

### Phase 3: Professional Skills (1-2 weeks)
1. **Complete Modules 6-7** (Advanced Functions → Error Handling)
2. **Complete Module 8** (Modern JavaScript)

---

## 📖 Essential Reading List (Priority Order)

### Must Read (Before Starting)
1. **"You Don't Know JS: Scope & Closures"** - Kyle Simpson
2. **MDN: "JavaScript Execution Context"**
3. **"How JavaScript Works" series** - Alexander Zlatkov (blog)

### Should Read (While Learning)
1. **"You Don't Know JS: this & Object Prototypes"** - Kyle Simpson
2. **"You Don't Know JS: Async & Performance"** - Kyle Simpson
3. **JavaScript.info** (complete tutorial)

### Nice to Read (For Depth)
1. **"Effective JavaScript"** - David Herman
2. **"JavaScript: The Good Parts"** - Douglas Crockford
3. **"Functional-Light JavaScript"** - Kyle Simpson

---

## 🔧 Practical Prerequisites Check

**Before Module 1, can you answer:**
- What happens during JavaScript's compilation phase?
- What is a lexical environment?
- How are variables stored in memory?

**Before Module 2, can you answer:**
- What are the 4 ways to call a function in JavaScript?
- How does method invocation differ from function invocation?

**Before Module 3, can you answer:**
- What is a scope chain?
- How do nested functions access outer variables?
- What happens when a function finishes executing?

**Before Module 4, can you answer:**
- What is the difference between `__proto__` and `prototype`?
- How does the `new` operator work?
- What is prototype chain lookup?

**Before Module 5, can you answer:**
- What is the event loop?
- How do microtasks differ from macrotasks?
- What are the three states of a Promise?

---

## ⚡ Quick Start Option

**If you're short on time, absolute minimum to understand:**

1. **Lexical Environment** (30 minutes)
   - Environment records
   - Scope chains
   - Closure mechanics

2. **this Binding Rules** (20 minutes)
   - Implicit, explicit, new, default binding
   - Arrow function lexical this

3. **Prototype Basics** (20 minutes)
   - prototype vs __proto__
   - Prototype chain lookup
   - Constructor functions

4. **Event Loop Basics** (15 minutes)
   - Call stack
   - Task queue
   - Microtask queue

**Total: ~1.5 hours of focused reading**

This foundation will let you understand 80% of the module content, though you'll want to go deeper for interviews.

---

## 🎯 Red Flags (Don't Skip These!)

**If you don't understand these, stop and learn them first:**

❌ **"I don't know what lexical environment means"**
❌ **"I'm confused about scope vs context"**  
❌ **"I don't understand how closures work"**
❌ **"I can't explain what 'this' refers to"**
❌ **"I don't know the difference between prototype and __proto__"**
❌ **"I'm unclear about the event loop"**

These concepts are fundamental to everything else!

The key is building a solid foundation. Rushing into advanced topics without understanding the basics will make everything much harder. Take time with the prerequisites - it's an investment that pays off massively!