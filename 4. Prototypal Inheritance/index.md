# Module 4: Prototypes & Inheritance

## Why This Module Matters
JavaScript doesn't have traditional classes (even ES6 classes are syntactic sugar). Everything is based on prototypes. Understanding prototypes is crucial for debugging, performance optimization, understanding frameworks, and mastering JavaScript's object system. This builds on execution context (Module 1) since prototype lookup happens during property access.

## Core Concept: Prototype Chain

Every object in JavaScript has a hidden `[[Prototype]]` property (accessible via `__proto__` or `Object.getPrototypeOf()`) that points to another object. This creates a chain used for property lookup.

```javascript
var obj = {
    name: "Object"
};

console.log(obj.toString); // [Function: toString]
// obj doesn't have toString, but Object.prototype does!

// Prototype chain: obj → Object.prototype → null
console.log(obj.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null (end of chain)
```

## Function Constructors & Prototypes

### How Constructor Functions Work
```javascript
function Person(name, age) {
    // 'this' refers to newly created object (from 'new')
    this.name = name;
    this.age = age;
}

// Add methods to prototype (shared by all instances)
Person.prototype.greet = function() {
    return `Hi, I'm ${this.name}, ${this.age} years old`;
};

Person.prototype.isAdult = function() {
    return this.age >= 18;
};

var alice = new Person("Alice", 25);
var bob = new Person("Bob", 17);

console.log(alice.greet()); // "Hi, I'm Alice, 25 years old"
console.log(bob.isAdult()); // false

// Both instances share the same prototype methods
console.log(alice.greet === bob.greet); // true (same function reference)
```

### What 'new' Actually Does
```javascript
function Person(name) {
    this.name = name;
}

// When you call: var person = new Person("Alice");
// JavaScript does this:

function simulateNew(constructor, ...args) {
    // 1. Create new object with constructor's prototype
    var newObj = Object.create(constructor.prototype);
    
    // 2. Call constructor with 'this' bound to new object
    var result = constructor.apply(newObj, args);
    
    // 3. Return object (or constructor's return value if it's an object)
    return (typeof result === 'object' && result !== null) ? result : newObj;
}

var person1 = new Person("Alice");
var person2 = simulateNew(Person, "Bob");

console.log(person1.constructor === Person); // true
console.log(person2.constructor === Person); // true
```

## Prototype Chain Lookup

### Property Resolution Process
```javascript
function Animal(species) {
    this.species = species;
}

Animal.prototype.breathe = function() {
    return `${this.species} breathes`;
};

function Dog(name, breed) {
    Animal.call(this, "Canine"); // Call parent constructor
    this.name = name;
    this.breed = breed;
}

// Set up inheritance
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
    return `${this.name} barks!`;
};

var rex = new Dog("Rex", "German Shepherd");

// Property lookup chain:
console.log(rex.name);    // Own property: "Rex"
console.log(rex.bark());  // Dog.prototype: "Rex barks!"
console.log(rex.breathe()); // Animal.prototype: "Canine breathes"
console.log(rex.toString()); // Object.prototype: "[object Object]"

// Prototype chain: rex → Dog.prototype → Animal.prototype → Object.prototype → null
```

### hasOwnProperty vs in Operator
```javascript
function Parent() {
    this.ownProp = "own";
}

Parent.prototype.inheritedProp = "inherited";

var child = new Parent();

// Check own properties
console.log(child.hasOwnProperty('ownProp')); // true
console.log(child.hasOwnProperty('inheritedProp')); // false

// Check entire prototype chain
console.log('ownProp' in child); // true
console.log('inheritedProp' in child); // true

// Enumerate own properties only
for (var prop in child) {
    if (child.hasOwnProperty(prop)) {
        console.log("Own property:", prop);
    }
}
```

## Classical Inheritance Patterns

### Pattern 1: Constructor Stealing
```javascript
function Vehicle(make, model) {
    this.make = make;
    this.model = model;
    this.started = false;
}

Vehicle.prototype.start = function() {
    this.started = true;
    return `${this.make} ${this.model} started`;
};

function Car(make, model, doors) {
    // Call parent constructor
    Vehicle.call(this, make, model);
    this.doors = doors;
}

// Inherit prototype methods
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

// Add child-specific methods
Car.prototype.openDoors = function() {
    return `Opening ${this.doors} doors`;
};

var myCar = new Car("Toyota", "Camry", 4);
console.log(myCar.start()); // "Toyota Camry started"
console.log(myCar.openDoors()); // "Opening 4 doors"
```

### Pattern 2: Prototypal Inheritance (Object.create)
```javascript
var animal = {
    type: "Animal",
    eat: function(food) {
        return `${this.type} eats ${food}`;
    }
};

// Create object that inherits from animal
var dog = Object.create(animal);
dog.type = "Dog";
dog.bark = function() {
    return `${this.type} barks`;
};

console.log(dog.eat("meat")); // "Dog eats meat"
console.log(dog.bark()); // "Dog barks"

// Prototype chain: dog → animal → Object.prototype → null
console.log(Object.getPrototypeOf(dog) === animal); // true
```

### Pattern 3: Mixin Pattern
```javascript
// Define mixins
var CanFly = {
    fly: function() {
        return `${this.name} is flying`;
    },
    
    land: function() {
        return `${this.name} has landed`;
    }
};

var CanSwim = {
    swim: function() {
        return `${this.name} is swimming`;
    }
};

// Base constructor
function Bird(name) {
    this.name = name;
}

// Mix in abilities
Object.assign(Bird.prototype, CanFly);

function Duck(name) {
    Bird.call(this, name);
}

Duck.prototype = Object.create(Bird.prototype);
Duck.prototype.constructor = Duck;

// Duck can also swim
Object.assign(Duck.prototype, CanSwim);

var donald = new Duck("Donald");
console.log(donald.fly()); // "Donald is flying"
console.log(donald.swim()); // "Donald is swimming"
```

## ES6 Classes (Syntactic Sugar)

### Class Syntax
```javascript
class Animal {
    constructor(species) {
        this.species = species;
    }
    
    breathe() {
        return `${this.species} breathes`;
    }
    
    // Static method
    static getKingdom() {
        return "Animalia";
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super("Canine"); // Call parent constructor
        this.name = name;
        this.breed = breed;
    }
    
    bark() {
        return `${this.name} barks!`;
    }
    
    // Override parent method
    breathe() {
        return super.breathe() + " heavily";
    }
}

var rex = new Dog("Rex", "Labrador");
console.log(rex.bark()); // "Rex barks!"
console.log(rex.breathe()); // "Canine breathes heavily"
console.log(Dog.getKingdom()); // "Animalia"
```

### What Classes Really Are
```javascript
class MyClass {
    constructor(value) {
        this.value = value;
    }
    
    method() {
        return this.value;
    }
}

// Classes are just functions!
console.log(typeof MyClass); // "function"
console.log(MyClass.prototype.method); // [Function: method]

// Equivalent function constructor:
function MyFunction(value) {
    this.value = value;
}

MyFunction.prototype.method = function() {
    return this.value;
};

// Both create identical objects
var classInstance = new MyClass("test");
var functionInstance = new MyFunction("test");

console.log(classInstance.constructor === MyClass); // true
console.log(functionInstance.constructor === MyFunction); // true
```

## Advanced Prototype Concepts

### Prototype Pollution Attack
```javascript
// Dangerous: Modifying Object.prototype affects ALL objects
Object.prototype.toString = function() {
    return "HACKED!";
};

var innocent = {};
console.log(innocent.toString()); // "HACKED!"

// Prevention: Use Object.create(null) for dictionaries
var safeDict = Object.create(null);
safeDict.toString = "safe"; // Won't affect other objects

// Or use Map for key-value storage
var safeMap = new Map();
safeMap.set('toString', 'safe');
```

### Dynamic Prototype Extension
```javascript
function User(name) {
    this.name = name;
    
    // Add methods dynamically based on conditions
    if (typeof User.prototype.greet !== 'function') {
        User.prototype.greet = function() {
            return `Hello, I'm ${this.name}`;
        };
    }
}

var user1 = new User("Alice");
var user2 = new User("Bob");

// Both instances have the greet method
console.log(user1.greet()); // "Hello, I'm Alice"
console.log(user2.greet()); // "Hello, I'm Bob"
```

### Prototype Chain Modification
```javascript
function BaseClass() {
    this.base = true;
}

function ChildClass() {
    this.child = true;
}

// Set up inheritance
ChildClass.prototype = Object.create(BaseClass.prototype);
ChildClass.prototype.constructor = ChildClass;

var instance = new ChildClass();

// Dynamically change prototype chain
function NewBase() {
    this.newBase = true;
}

NewBase.prototype.newMethod = function() {
    return "New method";
};

// Change the prototype (affects all instances)
Object.setPrototypeOf(ChildClass.prototype, NewBase.prototype);

console.log(instance.newMethod()); // "New method"
```

## Performance Considerations

### Prototype vs Own Properties
```javascript
function SlowClass() {
    // Own properties - faster access but more memory
    this.method1 = function() { return "method1"; };
    this.method2 = function() { return "method2"; };
    this.method3 = function() { return "method3"; };
}

function FastClass() {
    // Prototype methods - slower access but shared memory
}

FastClass.prototype.method1 = function() { return "method1"; };
FastClass.prototype.method2 = function() { return "method2"; };
FastClass.prototype.method3 = function() { return "method3"; };

// Memory usage test
var slowInstances = [];
var fastInstances = [];

for (let i = 0; i < 1000; i++) {
    slowInstances.push(new SlowClass());
    fastInstances.push(new FastClass());
}

// SlowClass uses much more memory (each instance has own methods)
// FastClass shares methods via prototype (more memory efficient)
```

### Prototype Chain Length Impact
```javascript
// Long prototype chain - slower property lookup
function Level1() {}
function Level2() {}
function Level3() {}
function Level4() {}
function Level5() {}

Level2.prototype = Object.create(Level1.prototype);
Level3.prototype = Object.create(Level2.prototype);
Level4.prototype = Object.create(Level3.prototype);
Level5.prototype = Object.create(Level4.prototype);

Level1.prototype.method = function() {
    return "Found at level 1";
};

var instance = new Level5();

// Property lookup goes through 5 levels
console.time("Deep lookup");
for (let i = 0; i < 100000; i++) {
    instance.method();
}
console.timeEnd("Deep lookup");

// Shorter chain - faster lookup
function DirectClass() {}
DirectClass.prototype.method = function() {
    return "Found directly";
};

var directInstance = new DirectClass();

console.time("Direct lookup");
for (let i = 0; i < 100000; i++) {
    directInstance.method();
}
console.timeEnd("Direct lookup");
```

## Common Interview Patterns

### Pattern 1: instanceof vs typeof
```javascript
function MyClass() {}
var instance = new MyClass();

console.log(typeof instance); // "object"
console.log(instance instanceof MyClass); // true
console.log(instance instanceof Object); // true

// instanceof checks prototype chain
console.log(instance instanceof Array); // false

// Custom instanceof
function customInstanceof(obj, constructor) {
    var prototype = constructor.prototype;
    var proto = Object.getPrototypeOf(obj);
    
    while (proto !== null) {
        if (proto === prototype) {
            return true;
        }
        proto = Object.getPrototypeOf(proto);
    }
    
    return false;
}
```

### Pattern 2: Prototype Method Override
```javascript
function Parent() {
    this.type = "parent";
}

Parent.prototype.speak = function() {
    return `${this.type} speaks`;
};

function Child() {
    Parent.call(this);
    this.type = "child";
}

Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

// Override parent method
Child.prototype.speak = function() {
    var parentResult = Parent.prototype.speak.call(this);
    return parentResult + " loudly";
};

var child = new Child();
console.log(child.speak()); // "child speaks loudly"
```

### Pattern 3: Factory vs Constructor
```javascript
// Constructor pattern
function ConstructorPerson(name) {
    this.name = name;
}

ConstructorPerson.prototype.greet = function() {
    return `Hi, I'm ${this.name}`;
};

// Factory pattern
function createPerson(name) {
    return {
        name: name,
        greet: function() {
            return `Hi, I'm ${this.name}`;
        }
    };
}

var constructorPerson = new ConstructorPerson("Alice");
var factoryPerson = createPerson("Bob");

// Different prototype chains
console.log(constructorPerson instanceof ConstructorPerson); // true
console.log(factoryPerson instanceof ConstructorPerson); // false

// Factory objects don't share methods (memory inefficient)
var person1 = createPerson("Person1");
var person2 = createPerson("Person2");
console.log(person1.greet === person2.greet); // false (different functions)

// Constructor objects share methods via prototype
var constructor1 = new ConstructorPerson("Constructor1");
var constructor2 = new ConstructorPerson("Constructor2");
console.log(constructor1.greet === constructor2.greet); // true (same function)
```

## Debugging Prototype Issues

### Visualizing Prototype Chain
```javascript
function visualizePrototypeChain(obj) {
    var chain = [];
    var current = obj;
    
    while (current !== null) {
        chain.push({
            constructor: current.constructor ? current.constructor.name : 'Unknown',
            properties: Object.getOwnPropertyNames(current),
            prototype: current
        });
        current = Object.getPrototypeOf(current);
    }
    
    return chain;
}

function Animal() {}
Animal.prototype.breathe = function() {};

function Dog() {}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.bark = function() {};

var rex = new Dog();
console.log(visualizePrototypeChain(rex));
```

### Property Source Detection
```javascript
function findPropertySource(obj, propName) {
    var current = obj;
    var level = 0;
    
    while (current !== null) {
        if (current.hasOwnProperty(propName)) {
            return {
                level: level,
                source: current.constructor ? current.constructor.name : 'Object',
                value: current[propName]
            };
        }
        current = Object.getPrototypeOf(current);
        level++;
    }
    
    return null;
}

function Parent() {
    this.ownProp = "parent own";
}
Parent.prototype.protoProp = "parent proto";

function Child() {
    Parent.call(this);
}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.childProp = "child proto";

var instance = new Child();
console.log(findPropertySource(instance, 'ownProp')); // Level 0 (own)
console.log(findPropertySource(instance, 'protoProp')); // Level 2 (Parent.prototype)
```

## Interview Questions & Challenges

### Question 1: Predict the Output
```javascript
function A() {}
A.prototype.value = 1;

function B() {}
B.prototype = Object.create(A.prototype);
B.prototype.value = 2;

function C() {}
C.prototype = Object.create(B.prototype);
C.prototype.constructor = C;

var c = new C();
console.log(c.value); // ?
console.log(c.constructor); // ?
console.log(c instanceof A); // ?
console.log(c instanceof B); // ?
console.log(c instanceof C); // ?
```

### Question 2: Fix the Inheritance
```javascript
// This inheritance setup is broken - fix it
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    return `${this.name} makes a sound`;
};

function Dog(name, breed) {
    this.name = name;
    this.breed = breed;
}

Dog.prototype.speak = function() {
    return `${this.name} barks`;
};

var dog = new Dog("Rex", "Labrador");
console.log(dog instanceof Animal); // Should be true
console.log(dog.constructor); // Should be Dog
```

### Question 3: Create a Shape Hierarchy
```javascript
// Create a shape inheritance hierarchy with:
// - Shape (base class with area() method that throws error)
// - Rectangle (width, height, calculates area)
// - Square (extends Rectangle, only needs one side)
// - Circle (radius, calculates area)

// Requirements:
// - Use constructor functions (not ES6 classes)
// - Proper prototype chain setup
// - Override area() method appropriately
// - Include constructor property fixing
```

### Question 4: Implement Multiple Inheritance
```javascript
// JavaScript doesn't support multiple inheritance
// Create a mixin system that allows an object to inherit from multiple sources

function mixin(target, ...sources) {
    // Your implementation here
    // Should copy methods from sources to target.prototype
    // Handle method conflicts (later sources override earlier ones)
}

function CanFly() {}
CanFly.prototype.fly = function() { return "flying"; };

function CanSwim() {}
CanSwim.prototype.swim = function() { return "swimming"; };

function Duck() {}

mixin(Duck, CanFly, CanSwim);

var duck = new Duck();
console.log(duck.fly()); // "flying"
console.log(duck.swim()); // "swimming"
```

## Key Takeaways for Interviews

1. **Prototypes are the foundation** of JavaScript's object system
2. **Every object has [[Prototype]]** pointing to another object (prototype chain)
3. **Property lookup traverses the chain** until found or reaching null
4. **Constructor functions + prototype** = classical inheritance pattern
5. **ES6 classes are syntactic sugar** over constructor functions
6. **Object.create()** is the modern way to set up inheritance
7. **instanceof checks prototype chain** membership
8. **Performance: prototype methods** are memory efficient vs own properties
9. **Prototype pollution** can be a security issue
10. **hasOwnProperty()** vs **in operator** for property checking

## Practice Challenges

### Challenge 1: Custom Array
```javascript
// Create a custom array-like object that inherits from Array
// but adds custom methods like sum(), average(), etc.
function NumberArray(...numbers) {
    // Your implementation
}

var nums = new NumberArray(1, 2, 3, 4, 5);
console.log(nums.length); // 5
console.log(nums.sum()); // 15
console.log(nums.average()); // 3
nums.push(6);
console.log(nums.sum()); // 21
```

### Challenge 2: Deep Clone with Prototype
```javascript
// Create a deep clone function that preserves prototype chain
function deepCloneWithPrototype(obj) {
    // Your implementation
    // Should clone object and maintain prototype relationships
}

function Parent() { this.parentProp = "parent"; }
Parent.prototype.parentMethod = function() { return "parent method"; };

function Child() { Parent.call(this); this.childProp = "child"; }
Child.prototype = Object.create(Parent.prototype);
Child.prototype.childMethod = function() { return "child method"; };

var original = new Child();
var cloned = deepCloneWithPrototype(original);

console.log(cloned instanceof Child); // true
console.log(cloned.parentMethod()); // "parent method"
```

### Challenge 3: Prototype Chain Analyzer
```javascript
// Create a function that analyzes and visualizes prototype chains
function analyzePrototypeChain(obj) {
    // Your implementation
    // Should return detailed information about the prototype chain
    // Including: constructor names, own properties, inherited properties
}
```
