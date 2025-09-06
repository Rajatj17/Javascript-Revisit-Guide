# Module 8: Modern JavaScript & Best Practices

## Why This Module Matters
Modern JavaScript includes powerful features that change how we write code. Understanding ES6+, module systems, performance optimization, and security best practices is essential for senior-level interviews and production development. This module ties together all previous concepts with real-world professional practices.

## Core Concept: Modern JavaScript Evolution

JavaScript has evolved rapidly since ES6 (2015). Each annual release adds features that improve developer experience and code quality.

```javascript
// ES5 vs Modern JavaScript comparison

// ES5 approach
function UserService(apiUrl) {
    this.apiUrl = apiUrl;
    this.cache = {};
}

UserService.prototype.getUser = function(id) {
    var self = this;
    
    if (this.cache[id]) {
        return Promise.resolve(this.cache[id]);
    }
    
    return fetch(this.apiUrl + '/users/' + id)
        .then(function(response) {
            return response.json();
        })
        .then(function(user) {
            self.cache[id] = user;
            return user;
        });
};

// Modern JavaScript approach
class UserService {
    #cache = new Map(); // Private field
    
    constructor(apiUrl) {
        this.apiUrl = apiUrl;
    }
    
    async getUser(id) {
        if (this.#cache.has(id)) {
            return this.#cache.get(id);
        }
        
        const response = await fetch(`${this.apiUrl}/users/${id}`);
        const user = await response.json();
        
        this.#cache.set(id, user);
        return user;
    }
    
    // Static method
    static createWithDefaults() {
        return new UserService('/api');
    }
}
```

## ES6+ Features Deep Dive

### Destructuring and Spread Operator
```javascript
// Advanced destructuring patterns
const user = {
    id: 1,
    name: "John Doe",
    address: {
        street: "123 Main St",
        city: "Boston",
        country: "USA"
    },
    hobbies: ["reading", "swimming", "coding"]
};

// Nested destructuring with renaming
const {
    name: userName,
    address: { city, country },
    hobbies: [firstHobby, ...otherHobbies]
} = user;

// Default values and computed properties
const { email = "no-email@example.com", age = 0 } = user;
const { [getPropertyName()]: dynamicValue } = user;

// Function parameter destructuring
function processUser({ name, address: { city } = {}, hobbies = [] }) {
    return {
        displayName: name,
        location: city,
        hobbyCount: hobbies.length
    };
}

// Advanced spread patterns
const baseConfig = { timeout: 5000, retries: 3 };
const apiConfig = { ...baseConfig, baseURL: '/api', timeout: 10000 }; // Override timeout

// Object merging with nested spreads
const mergeConfigs = (base, override) => ({
    ...base,
    ...override,
    headers: {
        ...base.headers,
        ...override.headers
    }
});

// Array operations with spread
const numbers = [1, 2, 3];
const moreNumbers = [0, ...numbers, 4, 5]; // [0, 1, 2, 3, 4, 5]
const copied = [...numbers]; // Shallow copy
const combined = [...numbers, ...moreNumbers];

// Function calls with spread
function sum(a, b, c) {
    return a + b + c;
}

const args = [1, 2, 3];
const result = sum(...args); // Equivalent to sum(1, 2, 3)
```

### Template Literals and Tagged Templates
```javascript
// Advanced template literal usage
const createQuery = (table, conditions) => {
    const whereClause = Object.entries(conditions)
        .map(([key, value]) => `${key} = '${value}'`)
        .join(' AND ');
    
    return `
        SELECT *
        FROM ${table}
        WHERE ${whereClause}
        ORDER BY created_at DESC
    `;
};

// Tagged template literals
function highlight(strings, ...values) {
    return strings.reduce((result, string, i) => {
        const value = values[i] ? `<mark>${values[i]}</mark>` : '';
        return result + string + value;
    }, '');
}

const searchTerm = "JavaScript";
const message = highlight`Welcome to ${searchTerm} development!`;
// "Welcome to <mark>JavaScript</mark> development!"

// SQL template with safety
function sql(strings, ...values) {
    // Escape SQL values to prevent injection
    const escapedValues = values.map(value => {
        if (typeof value === 'string') {
            return value.replace(/'/g, "''");
        }
        return value;
    });
    
    return strings.reduce((query, string, i) => {
        const value = escapedValues[i] || '';
        return query + string + value;
    }, '');
}

const userId = "'; DROP TABLE users; --";
const query = sql`SELECT * FROM users WHERE id = '${userId}'`;
// Safe: SELECT * FROM users WHERE id = '''; DROP TABLE users; --'

// Internationalization with templates
function i18n(strings, ...values) {
    const key = strings.join('{}');
    const translation = getTranslation(key);
    
    return translation.replace(/{}/g, () => values.shift());
}

const userName = "Alice";
const greeting = i18n`Hello, ${userName}! You have ${5} messages.`;
```

### Symbols and Well-Known Symbols
```javascript
// Creating and using symbols
const SECRET_PROPERTY = Symbol('secret');
const ITERATOR_METHOD = Symbol('customIterator');

class SecureContainer {
    constructor(data) {
        this.data = data;
        this[SECRET_PROPERTY] = "This is private";
    }
    
    // Symbol properties are not enumerable
    getSecret() {
        return this[SECRET_PROPERTY];
    }
    
    // Custom iterator
    *[Symbol.iterator]() {
        for (const item of this.data) {
            yield item;
        }
    }
    
    // Custom string representation
    [Symbol.toStringTag] = 'SecureContainer';
    
    // Custom primitive conversion
    [Symbol.toPrimitive](hint) {
        if (hint === 'number') {
            return this.data.length;
        }
        return `SecureContainer(${this.data.length} items)`;
    }
}

const container = new SecureContainer(['a', 'b', 'c']);

// Symbol properties don't show in normal enumeration
console.log(Object.keys(container)); // ['data']
console.log(Object.getOwnPropertySymbols(container)); // [Symbol(secret), Symbol.iterator, ...]

// Using well-known symbols
for (const item of container) {
    console.log(item); // Uses Symbol.iterator
}

console.log(String(container)); // "SecureContainer(3 items)" - Uses Symbol.toPrimitive
console.log(Object.prototype.toString.call(container)); // "[object SecureContainer]" - Uses Symbol.toStringTag

// Global symbol registry
const globalSymbol = Symbol.for('myGlobalSymbol');
const sameSymbol = Symbol.for('myGlobalSymbol');
console.log(globalSymbol === sameSymbol); // true
```

### Generators and Iterators
```javascript
// Advanced generator patterns
function* fibonacci() {
    let [prev, curr] = [0, 1];
    
    while (true) {
        yield curr;
        [prev, curr] = [curr, prev + curr];
    }
}

// Generator with return value and delegation
function* numbersWithSum() {
    let sum = 0;
    
    yield* [1, 2, 3].map(function*(n) {
        sum += n;
        yield n;
    });
    
    return sum; // Return value (not yielded)
}

const gen = numbersWithSum();
console.log(gen.next()); // {value: 1, done: false}
console.log(gen.next()); // {value: 2, done: false}
console.log(gen.next()); // {value: 3, done: false}
console.log(gen.next()); // {value: 6, done: true} - return value

// Async generators
async function* fetchUserPages(apiUrl) {
    let page = 1;
    
    while (true) {
        const response = await fetch(`${apiUrl}?page=${page}`);
        const data = await response.json();
        
        if (data.users.length === 0) {
            break;
        }
        
        yield data.users;
        page++;
    }
}

// Consuming async generator
async function processAllUsers() {
    for await (const userPage of fetchUserPages('/api/users')) {
        for (const user of userPage) {
            await processUser(user);
        }
    }
}

// Custom iterable class
class Range {
    constructor(start, end, step = 1) {
        this.start = start;
        this.end = end;
        this.step = step;
    }
    
    *[Symbol.iterator]() {
        let current = this.start;
        
        while (current < this.end) {
            yield current;
            current += this.step;
        }
    }
    
    // Async iterator
    async *[Symbol.asyncIterator]() {
        for (const value of this) {
            await new Promise(resolve => setTimeout(resolve, 100)); // Simulate async
            yield value;
        }
    }
}

// Usage
const range = new Range(0, 10, 2);
console.log([...range]); // [0, 2, 4, 6, 8]

async function demo() {
    for await (const value of range) {
        console.log(value); // Async iteration with delays
    }
}
```

## Module Systems

### ES Modules Deep Dive
```javascript
// math.js - Named exports
export const PI = 3.14159;
export const E = 2.71828;

export function add(a, b) {
    return a + b;
}

export function multiply(a, b) {
    return a * b;
}

// Default export
export default function calculate(expression) {
    // Complex calculation logic
    return eval(expression); // Don't do this in production!
}

// Re-exports
export { formatNumber } from './formatters.js';
export * as validators from './validators.js';

// Dynamic exports
const operations = ['add', 'subtract', 'multiply', 'divide'];
operations.forEach(op => {
    export const [op] = (a, b) => {
        // Implementation
    };
});
```

```javascript
// main.js - Import patterns
import calculate, { PI, add, multiply } from './math.js';
import * as math from './math.js';
import { add as addition, multiply as multiplication } from './math.js';

// Dynamic imports
async function loadMathModule() {
    const mathModule = await import('./math.js');
    return mathModule.default;
}

// Conditional loading
const loadCalculator = () => {
    if (window.Worker) {
        return import('./worker-calculator.js');
    } else {
        return import('./simple-calculator.js');
    }
};

// Import maps (in HTML)
/*
<script type="importmap">
{
  "imports": {
    "lodash": "/node_modules/lodash/lodash.js",
    "react": "/node_modules/react/index.js"
  }
}
</script>
*/

// Top-level await (in modules)
const config = await fetch('/api/config').then(r => r.json());
const userPreferences = await loadUserPreferences();

// Module state is singleton
let callCount = 0;
export function incrementCallCount() {
    return ++callCount;
}
```

### Module Patterns and Architecture
```javascript
// Module factory pattern
function createModule(dependencies) {
    const { logger, apiClient, cache } = dependencies;
    
    // Private state
    let isInitialized = false;
    const privateData = new Map();
    
    // Private functions
    function validateInput(data) {
        if (!data || typeof data !== 'object') {
            throw new Error('Invalid input data');
        }
    }
    
    function logOperation(operation, data) {
        logger.info(`Module operation: ${operation}`, data);
    }
    
    // Public API
    return {
        async initialize() {
            if (isInitialized) return;
            
            const config = await apiClient.get('/config');
            privateData.set('config', config);
            isInitialized = true;
            logOperation('initialize', { config });
        },
        
        async processData(input) {
            if (!isInitialized) {
                throw new Error('Module not initialized');
            }
            
            validateInput(input);
            
            const cacheKey = JSON.stringify(input);
            let result = cache.get(cacheKey);
            
            if (!result) {
                result = await this.performComplexOperation(input);
                cache.set(cacheKey, result);
            }
            
            logOperation('processData', { input, result });
            return result;
        },
        
        async performComplexOperation(data) {
            // Complex business logic
            return { processed: true, data };
        }
    };
}

// Dependency injection container
class DIContainer {
    constructor() {
        this.dependencies = new Map();
        this.singletons = new Map();
    }
    
    register(name, factory, options = {}) {
        this.dependencies.set(name, { factory, options });
    }
    
    resolve(name) {
        const dependency = this.dependencies.get(name);
        
        if (!dependency) {
            throw new Error(`Dependency '${name}' not found`);
        }
        
        if (dependency.options.singleton) {
            if (!this.singletons.has(name)) {
                this.singletons.set(name, dependency.factory(this));
            }
            return this.singletons.get(name);
        }
        
        return dependency.factory(this);
    }
}

// Usage
const container = new DIContainer();

container.register('logger', () => console, { singleton: true });
container.register('cache', () => new Map(), { singleton: true });
container.register('apiClient', (di) => createApiClient(di.resolve('logger')));
container.register('dataModule', (di) => createModule({
    logger: di.resolve('logger'),
    apiClient: di.resolve('apiClient'),
    cache: di.resolve('cache')
}));

const dataModule = container.resolve('dataModule');
```

## Performance Optimization

### Memory Management
```javascript
// Memory-efficient patterns
class MemoryEfficientCache {
    constructor(maxSize = 1000) {
        this.maxSize = maxSize;
        this.cache = new Map();
        this.accessOrder = [];
    }
    
    get(key) {
        if (this.cache.has(key)) {
            // Move to end (most recently used)
            this.updateAccessOrder(key);
            return this.cache.get(key);
        }
        return null;
    }
    
    set(key, value) {
        if (this.cache.size >= this.maxSize && !this.cache.has(key)) {
            // Remove least recently used
            const lru = this.accessOrder.shift();
            this.cache.delete(lru);
        }
        
        this.cache.set(key, value);
        this.updateAccessOrder(key);
    }
    
    updateAccessOrder(key) {
        const index = this.accessOrder.indexOf(key);
        if (index > -1) {
            this.accessOrder.splice(index, 1);
        }
        this.accessOrder.push(key);
    }
    
    // Memory cleanup
    clear() {
        this.cache.clear();
        this.accessOrder.length = 0;
    }
    
    // Memory usage info
    getMemoryInfo() {
        return {
            size: this.cache.size,
            maxSize: this.maxSize,
            utilizationPercent: (this.cache.size / this.maxSize) * 100
        };
    }
}

// WeakMap for memory-safe metadata
const elementMetadata = new WeakMap();

function attachMetadata(element, data) {
    elementMetadata.set(element, data);
}

function getMetadata(element) {
    return elementMetadata.get(element);
}

// When element is removed from DOM, metadata is automatically garbage collected

// Object pooling for frequently created objects
class ObjectPool {
    constructor(createFn, resetFn, maxSize = 50) {
        this.createFn = createFn;
        this.resetFn = resetFn;
        this.maxSize = maxSize;
        this.pool = [];
        this.created = 0;
        this.reused = 0;
    }
    
    acquire() {
        if (this.pool.length > 0) {
            this.reused++;
            return this.pool.pop();
        }
        
        this.created++;
        return this.createFn();
    }
    
    release(obj) {
        if (this.pool.length < this.maxSize) {
            this.resetFn(obj);
            this.pool.push(obj);
        }
    }
    
    getStats() {
        return {
            poolSize: this.pool.length,
            created: this.created,
            reused: this.reused,
            reuseRatio: this.reused / (this.created + this.reused)
        };
    }
}

// Usage
const vectorPool = new ObjectPool(
    () => ({ x: 0, y: 0, z: 0 }),
    (vector) => { vector.x = vector.y = vector.z = 0; }
);

function calculatePhysics() {
    const velocity = vectorPool.acquire();
    const acceleration = vectorPool.acquire();
    
    // Use vectors for calculations
    
    vectorPool.release(velocity);
    vectorPool.release(acceleration);
}
```

### Lazy Loading and Code Splitting
```javascript
// Lazy component loading
class LazyLoader {
    constructor() {
        this.loadedModules = new Map();
        this.loadingPromises = new Map();
    }
    
    async loadComponent(name) {
        if (this.loadedModules.has(name)) {
            return this.loadedModules.get(name);
        }
        
        if (this.loadingPromises.has(name)) {
            return this.loadingPromises.get(name);
        }
        
        const loadPromise = this.loadComponentModule(name);
        this.loadingPromises.set(name, loadPromise);
        
        try {
            const module = await loadPromise;
            this.loadedModules.set(name, module);
            this.loadingPromises.delete(name);
            return module;
        } catch (error) {
            this.loadingPromises.delete(name);
            throw error;
        }
    }
    
    async loadComponentModule(name) {
        const moduleMap = {
            'UserProfile': () => import('./components/UserProfile.js'),
            'Dashboard': () => import('./components/Dashboard.js'),
            'Settings': () => import('./components/Settings.js')
        };
        
        const loader = moduleMap[name];
        if (!loader) {
            throw new Error(`Component '${name}' not found`);
        }
        
        const module = await loader();
        return module.default || module;
    }
}

// Intersection Observer for lazy loading
class LazyImageLoader {
    constructor(options = {}) {
        this.observer = new IntersectionObserver(
            this.handleIntersection.bind(this),
            {
                rootMargin: options.rootMargin || '50px',
                threshold: options.threshold || 0.1
            }
        );
        this.loadedImages = new Set();
    }
    
    observe(element) {
        if (!this.loadedImages.has(element)) {
            this.observer.observe(element);
        }
    }
    
    handleIntersection(entries) {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                this.loadImage(entry.target);
            }
        });
    }
    
    loadImage(img) {
        const src = img.dataset.src;
        if (src && !this.loadedImages.has(img)) {
            img.src = src;
            img.removeAttribute('data-src');
            this.loadedImages.add(img);
            this.observer.unobserve(img);
            
            img.addEventListener('load', () => {
                img.classList.add('loaded');
            });
        }
    }
    
    disconnect() {
        this.observer.disconnect();
        this.loadedImages.clear();
    }
}

// Usage
const imageLoader = new LazyImageLoader({ rootMargin: '100px' });

document.querySelectorAll('img[data-src]').forEach(img => {
    imageLoader.observe(img);
});
```

### Performance Monitoring
```javascript
// Performance metrics collection
class PerformanceMonitor {
    constructor() {
        this.metrics = [];
        this.observers = [];
        this.setupObservers();
    }
    
    setupObservers() {
        // Performance Observer for various metrics
        if ('PerformanceObserver' in window) {
            // Long tasks
            const longTaskObserver = new PerformanceObserver(list => {
                list.getEntries().forEach(entry => {
                    this.recordMetric('longTask', {
                        duration: entry.duration,
                        startTime: entry.startTime
                    });
                });
            });
            longTaskObserver.observe({ entryTypes: ['longtask'] });
            
            // Layout shifts
            const layoutShiftObserver = new PerformanceObserver(list => {
                list.getEntries().forEach(entry => {
                    this.recordMetric('layoutShift', {
                        value: entry.value,
                        startTime: entry.startTime,
                        hadRecentInput: entry.hadRecentInput
                    });
                });
            });
            layoutShiftObserver.observe({ entryTypes: ['layout-shift'] });
            
            // Largest contentful paint
            const lcpObserver = new PerformanceObserver(list => {
                const entries = list.getEntries();
                const lastEntry = entries[entries.length - 1];
                
                this.recordMetric('largestContentfulPaint', {
                    startTime: lastEntry.startTime,
                    size: lastEntry.size,
                    element: lastEntry.element?.tagName
                });
            });
            lcpObserver.observe({ entryTypes: ['largest-contentful-paint'] });
        }
    }
    
    recordMetric(type, data) {
        this.metrics.push({
            type,
            timestamp: performance.now(),
            ...data
        });
        
        // Keep only recent metrics
        const cutoff = performance.now() - 300000; // 5 minutes
        this.metrics = this.metrics.filter(m => m.timestamp > cutoff);
    }
    
    // Manual timing
    startTiming(name) {
        performance.mark(`${name}-start`);
    }
    
    endTiming(name) {
        performance.mark(`${name}-end`);
        performance.measure(name, `${name}-start`, `${name}-end`);
        
        const measure = performance.getEntriesByName(name)[0];
        this.recordMetric('customTiming', {
            name,
            duration: measure.duration
        });
    }
    
    // Memory usage
    getMemoryUsage() {
        if (performance.memory) {
            return {
                used: Math.round(performance.memory.usedJSHeapSize / 1048576),
                total: Math.round(performance.memory.totalJSHeapSize / 1048576),
                limit: Math.round(performance.memory.jsHeapSizeLimit / 1048576)
            };
        }
        return null;
    }
    
    // Generate report
    generateReport() {
        const report = {
            timestamp: new Date().toISOString(),
            memoryUsage: this.getMemoryUsage(),
            metrics: {
                longTasks: this.metrics.filter(m => m.type === 'longTask'),
                layoutShifts: this.metrics.filter(m => m.type === 'layoutShift'),
                customTimings: this.metrics.filter(m => m.type === 'customTiming')
            }
        };
        
        // Calculate Core Web Vitals
        const lcpEntries = this.metrics.filter(m => m.type === 'largestContentfulPaint');
        if (lcpEntries.length > 0) {
            report.coreWebVitals = {
                LCP: lcpEntries[lcpEntries.length - 1].startTime,
                CLS: this.calculateCLS(),
                FID: this.calculateFID()
            };
        }
        
        return report;
    }
    
    calculateCLS() {
        const layoutShifts = this.metrics.filter(m => 
            m.type === 'layoutShift' && !m.hadRecentInput
        );
        
        return layoutShifts.reduce((total, shift) => total + shift.value, 0);
    }
    
    calculateFID() {
        // Implementation depends on first input delay measurement
        return null;
    }
}

// Usage
const performanceMonitor = new PerformanceMonitor();

// Manual timing
performanceMonitor.startTiming('dataProcessing');
await processLargeDataset();
performanceMonitor.endTiming('dataProcessing');

// Generate and send report
setInterval(() => {
    const report = performanceMonitor.generateReport();
    fetch('/api/performance', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(report)
    });
}, 60000); // Every minute
```

## Security Best Practices

### Input Validation and Sanitization
```javascript
// Comprehensive input validation
class InputValidator {
    static rules = {
        email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
        phone: /^\+?[\d\s\-\(\)]+$/,
        alphanumeric: /^[a-zA-Z0-9]+$/,
        safeString: /^[a-zA-Z0-9\s\-_.,!?]+$/
    };
    
    static validate(input, rules) {
        const errors = [];
        
        if (rules.required && !input) {
            errors.push('Field is required');
            return { isValid: false, errors };
        }
        
        if (!input) return { isValid: true, errors: [] };
        
        if (rules.type) {
            switch (rules.type) {
                case 'string':
                    if (typeof input !== 'string') {
                        errors.push('Must be a string');
                    }
                    break;
                case 'number':
                    if (typeof input !== 'number' || isNaN(input)) {
                        errors.push('Must be a valid number');
                    }
                    break;
                case 'email':
                    if (!this.rules.email.test(input)) {
                        errors.push('Must be a valid email address');
                    }
                    break;
            }
        }
        
        if (rules.minLength && input.length < rules.minLength) {
            errors.push(`Minimum length is ${rules.minLength}`);
        }
        
        if (rules.maxLength && input.length > rules.maxLength) {
            errors.push(`Maximum length is ${rules.maxLength}`);
        }
        
        if (rules.pattern && !rules.pattern.test(input)) {
            errors.push('Invalid format');
        }
        
        if (rules.custom) {
            const customResult = rules.custom(input);
            if (customResult !== true) {
                errors.push(customResult);
            }
        }
        
        return {
            isValid: errors.length === 0,
            errors
        };
    }
    
    static sanitize(input, options = {}) {
        if (typeof input !== 'string') return input;
        
        let sanitized = input;
        
        // HTML encoding
        if (options.escapeHtml !== false) {
            sanitized = sanitized
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;')
                .replace(/"/g, '&quot;')
                .replace(/'/g, '&#x27;');
        }
        
        // SQL injection prevention
        if (options.escapeSql) {
            sanitized = sanitized.replace(/'/g, "''");
        }
        
        // Remove dangerous characters
        if (options.removeDangerous) {
            sanitized = sanitized.replace(/[<>\"'%;()&+]/g, '');
        }
        
        // Trim whitespace
        if (options.trim !== false) {
            sanitized = sanitized.trim();
        }
        
        return sanitized;
    }
}

// Secure form handler
class SecureFormHandler {
    constructor(formElement) {
        this.form = formElement;
        this.validationRules = new Map();
        this.setupEventListeners();
    }
    
    addValidation(fieldName, rules) {
        this.validationRules.set(fieldName, rules);
    }
    
    setupEventListeners() {
        this.form.addEventListener('submit', this.handleSubmit.bind(this));
        
        // Real-time validation
        this.form.addEventListener('input', (e) => {
            if (this.validationRules.has(e.target.name)) {
                this.validateField(e.target);
            }
        });
    }
    
    validateField(field) {
        const rules = this.validationRules.get(field.name);
        if (!rules) return true;
        
        const result = InputValidator.validate(field.value, rules);
        
        // Update UI
        const errorElement = this.form.querySelector(`[data-error-for="${field.name}"]`);
        if (errorElement) {
            errorElement.textContent = result.errors.join(', ');
            errorElement.style.display = result.isValid ? 'none' : 'block';
        }
        
        field.classList.toggle('invalid', !result.isValid);
        
        return result.isValid;
    }
    
    async handleSubmit(e) {
        e.preventDefault();
        
        const formData = new FormData(this.form);
        const data = {};
        let allValid = true;
        
        // Validate all fields
        for (const [name, value] of formData) {
            if (this.validationRules.has(name)) {
                const field = this.form.querySelector(`[name="${name}"]`);
                if (!this.validateField(field)) {
                    allValid = false;
                }
                
                // Sanitize data
                const rules = this.validationRules.get(name);
                data[name] = InputValidator.sanitize(value, rules.sanitize || {});
            } else {
                data[name] = InputValidator.sanitize(value);
            }
        }
        
        if (allValid) {
            await this.submitData(data);
        }
    }
    
    async submitData(data) {
        try {
            // Add CSRF token
            const csrfToken = document.querySelector('meta[name="csrf-token"]')?.content;
            
            const response = await fetch(this.form.action, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'X-CSRF-Token': csrfToken
                },
                body: JSON.stringify(data)
            });
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            
            const result = await response.json();
            this.onSuccess(result);
            
        } catch (error) {
            this.onError(error);
        }
    }
    
    onSuccess(result) {
        // Handle successful submission
        console.log('Form submitted successfully:', result);
    }
    
    onError(error) {
        // Handle submission error
        console.error('Form submission failed:', error);
    }
}
```

### Content Security Policy and XSS Prevention
```javascript
// CSP violation reporter
class CSPViolationReporter {
    constructor() {
        this.setupViolationHandler();
    }
    
    setupViolationHandler() {
        document.addEventListener('securitypolicyviolation', (e) => {
            this.reportViolation({
                directive: e.violatedDirective,
                blockedURI: e.blockedURI,
                documentURI: e.documentURI,
                effectiveDirective: e.effectiveDirective,
                originalPolicy: e.originalPolicy,
                referrer: e.referrer,
                statusCode: e.statusCode,
                lineNumber: e.lineNumber,
                columnNumber: e.columnNumber
            });
        });
    }
    
    reportViolation(violation) {
        // Send to security monitoring service
        fetch('/api/security/csp-violation', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                violation,
                timestamp: new Date().toISOString(),
                userAgent: navigator.userAgent,
                url: window.location.href
            })
        }).catch(error => {
            console.error('Failed to report CSP violation:', error);
        });
    }
}

// Safe DOM manipulation
class SafeDOM {
    static createElement(tagName, attributes = {}, textContent = '') {
        const element = document.createElement(tagName);
        
        // Safely set attributes
        Object.entries(attributes).forEach(([key, value]) => {
            if (key === 'href' && !this.isValidURL(value)) {
                throw new Error('Invalid URL for href attribute');
            }
            
            if (key.startsWith('on')) {
                throw new Error('Event handlers not allowed in attributes');
            }
            
            element.setAttribute(key, value);
        });
        
        // Safely set text content
        if (textContent) {
            element.textContent = textContent;
        }
        
        return element;
    }
    
    static isValidURL(url) {
        try {
            const parsed = new URL(url);
            return ['http:', 'https:', 'mailto:', 'tel:'].includes(parsed.protocol);
        } catch {
            return false;
        }
    }
    
    static setInnerHTML(element, html) {
        // Use DOMPurify if available, otherwise basic sanitization
        if (window.DOMPurify) {
            element.innerHTML = DOMPurify.sanitize(html);
        } else {
            // Basic sanitization
            const sanitized = html
                .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
                .replace(/<iframe\b[^<]*(?:(?!<\/iframe>)<[^<]*)*<\/iframe>/gi, '')
                .replace(/on\w+\s*=\s*["'][^"']*["']/gi, '');
            
            element.innerHTML = sanitized;
        }
    }
    
    static addEventHandler(element, event, handler) {
        // Validate event name
        const validEvents = [
            'click', 'change', 'input', 'submit', 'load', 'resize',
            'scroll', 'focus', 'blur', 'keydown', 'keyup', 'mouseenter', 'mouseleave'
        ];
        
        if (!validEvents.includes(event)) {
            throw new Error(`Event '${event}' not in allowed list`);
        }
        
        element.addEventListener(event, handler);
    }
}

// Secure storage
class SecureStorage {
    constructor(storageType = 'localStorage') {
        this.storage = window[storageType];
        this.encryptionKey = this.getOrCreateKey();
    }
    
    getOrCreateKey() {
        // In production, derive from user session
        return 'secure-key-' + Date.now();
    }
    
    encrypt(data) {
        // Simple XOR encryption (use proper crypto in production)
        return btoa(JSON.stringify(data));
    }
    
    decrypt(encryptedData) {
        try {
            return JSON.parse(atob(encryptedData));
        } catch {
            return null;
        }
    }
    
    setItem(key, value) {
        try {
            const encrypted = this.encrypt(value);
            this.storage.setItem(key, encrypted);
        } catch (error) {
            console.error('Failed to store data:', error);
        }
    }
    
    getItem(key) {
        try {
            const encrypted = this.storage.getItem(key);
            return encrypted ? this.decrypt(encrypted) : null;
        } catch (error) {
            console.error('Failed to retrieve data:', error);
            return null;
        }
    }
    
    removeItem(key) {
        this.storage.removeItem(key);
    }
    
    clear() {
        this.storage.clear();
    }
}
```

## Testing Strategies

### Unit Testing Patterns
```javascript
// Test utilities
class TestFramework {
    constructor() {
        this.tests = [];
        this.beforeEachCallbacks = [];
        this.afterEachCallbacks = [];
    }
    
    beforeEach(callback) {
        this.beforeEachCallbacks.push(callback);
    }
    
    afterEach(callback) {
        this.afterEachCallbacks.push(callback);
    }
    
    test(description, testFunction) {
        this.tests.push({ description, testFunction });
    }
    
    async run() {
        const results = {
            passed: 0,
            failed: 0,
            errors: []
        };
        
        for (const test of this.tests) {
            try {
                // Run beforeEach callbacks
                for (const callback of this.beforeEachCallbacks) {
                    await callback();
                }
                
                // Run the test
                await test.testFunction();
                
                results.passed++;
                console.log(`✅ ${test.description}`);
                
            } catch (error) {
                results.failed++;
                results.errors.push({
                    test: test.description,
                    error: error.message
                });
                console.error(`❌ ${test.description}: ${error.message}`);
                
            } finally {
                // Run afterEach callbacks
                for (const callback of this.afterEachCallbacks) {
                    try {
                        await callback();
                    } catch (cleanupError) {
                        console.warn('Cleanup error:', cleanupError);
                    }
                }
            }
        }
        
        console.log(`\nTest Results: ${results.passed} passed, ${results.failed} failed`);
        return results;
    }
}

// Assertion library
class Assert {
    static equal(actual, expected, message = '') {
        if (actual !== expected) {
            throw new Error(`${message} Expected: ${expected}, Actual: ${actual}`);
        }
    }
    
    static deepEqual(actual, expected, message = '') {
        if (JSON.stringify(actual) !== JSON.stringify(expected)) {
            throw new Error(`${message} Objects not deeply equal`);
        }
    }
    
    static throws(fn, expectedError, message = '') {
        try {
            fn();
            throw new Error(`${message} Expected function to throw`);
        } catch (error) {
            if (expectedError && !(error instanceof expectedError)) {
                throw new Error(`${message} Expected ${expectedError.name}, got ${error.constructor.name}`);
            }
        }
    }
    
    static async rejects(promise, expectedError, message = '') {
        try {
            await promise;
            throw new Error(`${message} Expected promise to reject`);
        } catch (error) {
            if (expectedError && !(error instanceof expectedError)) {
                throw new Error(`${message} Expected ${expectedError.name}, got ${error.constructor.name}`);
            }
        }
    }
    
    static isTrue(value, message = '') {
        if (value !== true) {
            throw new Error(`${message} Expected true, got ${value}`);
        }
    }
    
    static isFalse(value, message = '') {
        if (value !== false) {
            throw new Error(`${message} Expected false, got ${value}`);
        }
    }
}

// Mock utilities
class MockFactory {
    static createMock(original) {
        const mock = {};
        const calls = new Map();
        
        if (typeof original === 'function') {
            return this.createFunctionMock(original);
        }
        
        // Mock object methods
        Object.getOwnPropertyNames(original).forEach(prop => {
            if (typeof original[prop] === 'function') {
                mock[prop] = this.createFunctionMock(original[prop]);
                calls.set(prop, []);
            } else {
                mock[prop] = original[prop];
            }
        });
        
        mock._getCalls = (methodName) => calls.get(methodName) || [];
        mock._getCallCount = (methodName) => mock._getCalls(methodName).length;
        mock._wasCalledWith = (methodName, ...args) => {
            return mock._getCalls(methodName).some(call => 
                JSON.stringify(call) === JSON.stringify(args)
            );
        };
        
        return mock;
    }
    
    static createFunctionMock(originalFn) {
        const calls = [];
        let returnValue = undefined;
        let throwError = null;
        
        const mockFn = function(...args) {
            calls.push(args);
            
            if (throwError) {
                throw throwError;
            }
            
            return returnValue;
        };
        
        mockFn.mockReturnValue = (value) => {
            returnValue = value;
            return mockFn;
        };
        
        mockFn.mockThrow = (error) => {
            throwError = error;
            return mockFn;
        };
        
        mockFn.getCalls = () => calls;
        mockFn.getCallCount = () => calls.length;
        mockFn.wasCalledWith = (...args) => {
            return calls.some(call => 
                JSON.stringify(call) === JSON.stringify(args)
            );
        };
        
        return mockFn;
    }
}

// Example test suite
const framework = new TestFramework();

// Setup and teardown
framework.beforeEach(() => {
    // Reset global state
    document.body.innerHTML = '';
});

framework.afterEach(() => {
    // Cleanup
});

// Tests
framework.test('Calculator adds numbers correctly', () => {
    const calc = new Calculator();
    const result = calc.add(2, 3);
    Assert.equal(result, 5, 'Addition should work');
});

framework.test('API client handles errors', async () => {
    const mockFetch = MockFactory.createFunctionMock(fetch);
    mockFetch.mockThrow(new Error('Network error'));
    
    global.fetch = mockFetch;
    
    const apiClient = new ApiClient();
    
    await Assert.rejects(
        apiClient.getData('/test'),
        Error,
        'Should throw network error'
    );
    
    Assert.isTrue(mockFetch.wasCalledWith('/test'), 'Should call fetch with correct URL');
});

framework.test('UserService caches data', async () => {
    const mockApiClient = MockFactory.createMock(ApiClient);
    mockApiClient.get.mockReturnValue(Promise.resolve({ id: 1, name: 'Test' }));
    
    const userService = new UserService(mockApiClient);
    
    await userService.getUser(1);
    await userService.getUser(1); // Second call should use cache
    
    Assert.equal(mockApiClient._getCallCount('get'), 1, 'Should only call API once');
});

// Run tests
framework.run();
```

### Integration Testing
```javascript
// Integration test utilities
class IntegrationTestRunner {
    constructor() {
        this.testEnvironment = null;
    }
    
    async setupTestEnvironment() {
        // Create isolated test environment
        this.testEnvironment = {
            dom: this.createTestDOM(),
            localStorage: this.createMockStorage(),
            fetch: this.createMockFetch(),
            services: new Map()
        };
        
        // Override globals for testing
        global.localStorage = this.testEnvironment.localStorage;
        global.fetch = this.testEnvironment.fetch;
    }
    
    createTestDOM() {
        // Create a minimal DOM environment
        return {
            createElement: (tag) => ({ tagName: tag, children: [] }),
            querySelector: (selector) => null,
            querySelectorAll: (selector) => [],
            addEventListener: () => {},
            removeEventListener: () => {}
        };
    }
    
    createMockStorage() {
        const storage = new Map();
        return {
            getItem: (key) => storage.get(key) || null,
            setItem: (key, value) => storage.set(key, value),
            removeItem: (key) => storage.delete(key),
            clear: () => storage.clear()
        };
    }
    
    createMockFetch() {
        const responses = new Map();
        
        const mockFetch = async (url, options = {}) => {
            const key = `${options.method || 'GET'} ${url}`;
            const response = responses.get(key);
            
            if (!response) {
                throw new Error(`No mock response for ${key}`);
            }
            
            return {
                ok: response.status >= 200 && response.status < 300,
                status: response.status,
                json: async () => response.data,
                text: async () => JSON.stringify(response.data)
            };
        };
        
        mockFetch.mockResponse = (method, url, data, status = 200) => {
            responses.set(`${method} ${url}`, { data, status });
        };
        
        return mockFetch;
    }
    
    async teardownTestEnvironment() {
        // Cleanup test environment
        this.testEnvironment = null;
    }
    
    async runIntegrationTest(testName, testFunction) {
        try {
            await this.setupTestEnvironment();
            console.log(`🧪 Running integration test: ${testName}`);
            
            await testFunction(this.testEnvironment);
            
            console.log(`✅ Integration test passed: ${testName}`);
            
        } catch (error) {
            console.error(`❌ Integration test failed: ${testName}`, error);
            throw error;
            
        } finally {
            await this.teardownTestEnvironment();
        }
    }
}

// Example integration tests
const integrationRunner = new IntegrationTestRunner();

integrationRunner.runIntegrationTest('User login flow', async (env) => {
    // Mock API responses
    env.fetch.mockResponse('POST', '/api/login', { 
        token: 'test-token',
        user: { id: 1, name: 'Test User' }
    });
    
    env.fetch.mockResponse('GET', '/api/user/profile', {
        id: 1,
        name: 'Test User',
        preferences: { theme: 'dark' }
    });
    
    // Test the complete login flow
    const authService = new AuthService();
    const userService = new UserService();
    
    // Login
    const loginResult = await authService.login('test@example.com', 'password');
    Assert.equal(loginResult.user.name, 'Test User');
    
    // Verify token is stored
    const storedToken = env.localStorage.getItem('authToken');
    Assert.equal(storedToken, 'test-token');
    
    // Load user profile
    const profile = await userService.getProfile();
    Assert.equal(profile.preferences.theme, 'dark');
});
```

## Interview Questions & Challenges

### Question 1: Modern JavaScript Features
```javascript
// Rewrite this ES5 code using modern JavaScript features
function UserManager(apiUrl) {
    var self = this;
    this.apiUrl = apiUrl;
    this.users = [];
    this.callbacks = [];
    
    this.loadUsers = function() {
        return fetch(this.apiUrl + '/users')
            .then(function(response) {
                return response.json();
            })
            .then(function(users) {
                self.users = users;
                self.callbacks.forEach(function(callback) {
                    callback(users);
                });
                return users;
            });
    };
    
    this.addUser = function(user) {
        return fetch(this.apiUrl + '/users', {
            method: 'POST',
            body: JSON.stringify(user)
        }).then(function(response) {
            return response.json();
        }).then(function(newUser) {
            self.users.push(newUser);
            return newUser;
        });
    };
    
    this.subscribe = function(callback) {
        this.callbacks.push(callback);
        return function() {
            var index = self.callbacks.indexOf(callback);
            if (index > -1) {
                self.callbacks.splice(index, 1);
            }
        };
    };
}

// Rewrite using: class syntax, async/await, private fields, destructuring, etc.
```

### Question 2: Performance Optimization
```javascript
// Optimize this code for better performance
function processLargeDataset(data) {
    var results = [];
    
    for (var i = 0; i < data.length; i++) {
        var item = data[i];
        
        // Expensive operation
        var processed = complexCalculation(item);
        
        // DOM manipulation
        var element = document.createElement('div');
        element.innerHTML = '<span>' + processed.value + '</span>';
        element.style.background = processed.color;
        document.body.appendChild(element);
        
        results.push(processed);
    }
    
    return results;
}

// Issues to address:
// 1. Blocking main thread
// 2. Inefficient DOM manipulation
// 3. Memory usage
// 4. No caching
```

### Question 3: Secure API Client
```javascript
// Create a secure API client that:
// - Validates all inputs
// - Prevents XSS in responses
// - Handles authentication tokens securely
// - Implements rate limiting
// - Logs security events

class SecureApiClient {
    constructor(baseUrl, options = {}) {
        // Your implementation
    }
    
    async request(endpoint, options = {}) {
        // Implement secure request handling
    }
}
```

### Question 4: Module Architecture
```javascript
// Design a modular application architecture using:
// - ES modules
// - Dependency injection
// - Event-driven communication
// - Lazy loading
// - Error boundaries

// Create the core architecture and show how modules would interact
```

### Question 5: Testing Strategy
```javascript
// Design a comprehensive testing strategy for a complex application
// Include: unit tests, integration tests, mocks, test utilities

// Example application: E-commerce checkout flow
// - User authentication
// - Cart management  
// - Payment processing
// - Order confirmation
```

## Production Deployment Best Practices

### Build Process and Optimization
```javascript
// Build configuration example
const buildConfig = {
    // Code splitting strategy
    splitChunks: {
        chunks: 'all',
        cacheGroups: {
            vendor: {
                test: /[\\/]node_modules[\\/]/,
                name: 'vendors',
                chunks: 'all'
            },
            common: {
                minChunks: 2,
                chunks: 'all',
                enforce: true
            }
        }
    },
    
    // Tree shaking configuration
    optimization: {
        usedExports: true,
        sideEffects: false
    },
    
    // Minification settings
    minimizer: {
        removeComments: true,
        collapseWhitespace: true,
        minifyJS: true,
        minifyCSS: true
    }
};

// Bundle analysis
function analyzeBundleSize() {
    const bundleAnalyzer = require('webpack-bundle-analyzer');
    
    return {
        generateReport: () => {
            bundleAnalyzer.analyzeBundle('./dist/bundle.js', {
                mode: 'static',
                reportFilename: 'bundle-report.html'
            });
        },
        
        checkSizeThresholds: (thresholds) => {
            const stats = getBundleStats();
            const warnings = [];
            
            if (stats.totalSize > thresholds.maxTotal) {
                warnings.push(`Total bundle size ${stats.totalSize} exceeds threshold ${thresholds.maxTotal}`);
            }
            
            if (stats.chunks.some(chunk => chunk.size > thresholds.maxChunk)) {
                warnings.push('Some chunks exceed size threshold');
            }
            
            return warnings;
        }
    };
}
```

### Monitoring and Observability
```javascript
// Application monitoring setup
class ApplicationMonitor {
    constructor() {
        this.setupRealUserMonitoring();
        this.setupErrorTracking();
        this.setupPerformanceTracking();
    }
    
    setupRealUserMonitoring() {
        // Track real user interactions
        const observer = new PerformanceObserver((list) => {
            list.getEntries().forEach(entry => {
                if (entry.entryType === 'navigation') {
                    this.trackPageLoad(entry);
                } else if (entry.entryType === 'paint') {
                    this.trackPaintTiming(entry);
                }
            });
        });
        
        observer.observe({ entryTypes: ['navigation', 'paint'] });
    }
    
    setupErrorTracking() {
        window.addEventListener('error', (event) => {
            this.reportError({
                type: 'javascript',
                message: event.message,
                filename: event.filename,
                lineno: event.lineno,
                colno: event.colno,
                stack: event.error?.stack
            });
        });
        
        window.addEventListener('unhandledrejection', (event) => {
            this.reportError({
                type: 'unhandled_promise',
                reason: event.reason
            });
        });
    }
    
    setupPerformanceTracking() {
        // Custom performance marks
        this.markStartup = () => performance.mark('app-startup');
        this.markInteractive = () => performance.mark('app-interactive');
        
        // Measure time to interactive
        setTimeout(() => {
            performance.measure('startup-time', 'app-startup', 'app-interactive');
            const measure = performance.getEntriesByName('startup-time')[0];
            this.reportMetric('startup_time', measure.duration);
        }, 0);
    }
    
    trackPageLoad(entry) {
        this.reportMetric('page_load_time', entry.loadEventEnd - entry.loadEventStart);
        this.reportMetric('dom_content_loaded', entry.domContentLoadedEventEnd - entry.domContentLoadedEventStart);
    }
    
    trackPaintTiming(entry) {
        this.reportMetric(entry.name.replace('-', '_'), entry.startTime);
    }
    
    reportError(error) {
        fetch('/api/errors', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                ...error,
                timestamp: Date.now(),
                url: window.location.href,
                userAgent: navigator.userAgent
            })
        });
    }
    
    reportMetric(name, value) {
        fetch('/api/metrics', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                name,
                value,
                timestamp: Date.now()
            })
        });
    }
}
```

## Key Takeaways for Interviews

1. **Modern JavaScript features** enhance code quality and developer experience
2. **ES modules** are the standard for modular JavaScript applications
3. **Performance optimization** requires measurement, analysis, and targeted improvements
4. **Security** must be built into every layer of the application
5. **Testing strategies** should cover unit, integration, and end-to-end scenarios
6. **Memory management** is crucial for long-running applications
7. **Code splitting and lazy loading** improve initial load performance
8. **Monitoring and observability** are essential for production applications
9. **Build processes** should optimize for performance and maintainability
10. **Professional practices** separate good developers from great ones

This completes our comprehensive **8-module JavaScript interview preparation guide**! Each module builds upon the previous ones, creating a complete picture of modern JavaScript development from fundamentals to production best practices.

You're now equipped with:
- Deep understanding of JavaScript's core mechanics
- Modern development patterns and practices
- Error handling and debugging skills
- Performance optimization techniques
- Security considerations
- Testing strategies
- Production deployment knowledge

Ready to tackle any JavaScript interview with confidence! 🚀