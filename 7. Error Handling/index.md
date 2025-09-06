# Module 7: Error Handling & Debugging

## Why This Module Matters
Robust error handling and debugging skills are crucial for production applications. Understanding error types, debugging techniques, and testing strategies separates junior from senior developers. This builds on async patterns (Module 5) since error handling in async code is more complex.

## Core Concept: Error Types and the Error Object

JavaScript has several built-in error types, and understanding them helps with proper error handling and debugging.

```javascript
// Basic Error object
var error = new Error("Something went wrong");
console.log(error.name);    // "Error"
console.log(error.message); // "Something went wrong"
console.log(error.stack);   // Stack trace

// Built-in error types
try {
    // ReferenceError - accessing undefined variable
    console.log(undefinedVariable);
} catch (error) {
    console.log(error.name); // "ReferenceError"
}

try {
    // TypeError - wrong type operation
    null.someMethod();
} catch (error) {
    console.log(error.name); // "TypeError"
}

try {
    // SyntaxError - invalid syntax (usually at parse time)
    eval("var a = ;");
} catch (error) {
    console.log(error.name); // "SyntaxError"
}

try {
    // RangeError - number out of range
    var arr = new Array(-1);
} catch (error) {
    console.log(error.name); // "RangeError"
}
```

## Custom Error Classes

### Creating Meaningful Error Types
```javascript
// Custom error class
class ValidationError extends Error {
    constructor(message, field) {
        super(message);
        this.name = "ValidationError";
        this.field = field;
        
        // Maintains proper stack trace (V8 only)
        if (Error.captureStackTrace) {
            Error.captureStackTrace(this, ValidationError);
        }
    }
}

class NetworkError extends Error {
    constructor(message, status, url) {
        super(message);
        this.name = "NetworkError";
        this.status = status;
        this.url = url;
    }
}

class BusinessLogicError extends Error {
    constructor(message, code, details) {
        super(message);
        this.name = "BusinessLogicError";
        this.code = code;
        this.details = details;
    }
}

// Usage
function validateUser(user) {
    if (!user.email) {
        throw new ValidationError("Email is required", "email");
    }
    
    if (!user.email.includes('@')) {
        throw new ValidationError("Invalid email format", "email");
    }
    
    if (user.age < 0) {
        throw new ValidationError("Age cannot be negative", "age");
    }
}

// Error handling with specific types
try {
    validateUser({ email: "invalid-email", age: -5 });
} catch (error) {
    if (error instanceof ValidationError) {
        console.log(`Validation failed for ${error.field}: ${error.message}`);
    } else {
        console.log("Unexpected error:", error);
    }
}
```

### Error Factory Pattern
```javascript
class ErrorFactory {
    static createValidationError(field, value, rule) {
        return new ValidationError(
            `Field '${field}' with value '${value}' failed validation: ${rule}`,
            field
        );
    }
    
    static createNetworkError(response) {
        return new NetworkError(
            `HTTP ${response.status}: ${response.statusText}`,
            response.status,
            response.url
        );
    }
    
    static createBusinessError(code, context) {
        var messages = {
            'INSUFFICIENT_FUNDS': 'Insufficient funds for this transaction',
            'USER_NOT_FOUND': 'User does not exist',
            'PERMISSION_DENIED': 'You do not have permission for this action'
        };
        
        return new BusinessLogicError(
            messages[code] || 'Unknown business logic error',
            code,
            context
        );
    }
}

// Usage
try {
    throw ErrorFactory.createBusinessError('INSUFFICIENT_FUNDS', {
        userId: 123,
        requestedAmount: 1000,
        availableBalance: 500
    });
} catch (error) {
    console.log(error.code);    // "INSUFFICIENT_FUNDS"
    console.log(error.details); // { userId: 123, ... }
}
```

## Synchronous Error Handling

### Try-Catch-Finally Patterns
```javascript
function robustFileProcessor(filename) {
    var file = null;
    var result = null;
    
    try {
        // Simulated file operations
        file = openFile(filename);
        
        if (!file) {
            throw new Error(`Could not open file: ${filename}`);
        }
        
        result = processFile(file);
        
        if (!result.valid) {
            throw new ValidationError("File content is invalid", "content");
        }
        
        return result.data;
        
    } catch (error) {
        if (error instanceof ValidationError) {
            console.error(`Validation error: ${error.message}`);
            return getDefaultData();
        } else {
            console.error(`Processing error: ${error.message}`);
            throw error; // Re-throw for caller to handle
        }
    } finally {
        // Cleanup always runs
        if (file) {
            closeFile(file);
            console.log("File closed");
        }
    }
}

// Nested try-catch for granular error handling
function complexOperation() {
    try {
        var userData = null;
        
        try {
            userData = fetchUserData();
        } catch (fetchError) {
            console.warn("Failed to fetch user data, using cached");
            userData = getCachedUserData();
            
            if (!userData) {
                throw new Error("No user data available");
            }
        }
        
        try {
            return processUserData(userData);
        } catch (processError) {
            throw new Error(`Processing failed: ${processError.message}`);
        }
        
    } catch (error) {
        logError(error);
        return getDefaultResult();
    }
}
```

### Error Propagation Strategies
```javascript
// Option 1: Fail fast (let errors bubble up)
function strictFunction(data) {
    validateInput(data);      // Throws on invalid input
    var result = process(data); // Throws on processing error
    validateOutput(result);    // Throws on invalid output
    return result;
}

// Option 2: Graceful degradation
function resilientFunction(data) {
    try {
        validateInput(data);
    } catch (error) {
        console.warn("Invalid input, using defaults:", error.message);
        data = getDefaultInput();
    }
    
    var result;
    try {
        result = process(data);
    } catch (error) {
        console.warn("Processing failed, using fallback:", error.message);
        result = fallbackProcess(data);
    }
    
    try {
        validateOutput(result);
        return result;
    } catch (error) {
        console.error("Output validation failed:", error.message);
        return getDefaultOutput();
    }
}

// Option 3: Result pattern (no exceptions)
function functionalApproach(data) {
    var validationResult = safeValidateInput(data);
    if (!validationResult.success) {
        return { success: false, error: validationResult.error };
    }
    
    var processResult = safeProcess(validationResult.data);
    if (!processResult.success) {
        return { success: false, error: processResult.error };
    }
    
    return { success: true, data: processResult.data };
}
```

## Asynchronous Error Handling

### Promise Error Handling
```javascript
// Basic promise error handling
fetch('/api/data')
    .then(response => {
        if (!response.ok) {
            throw new NetworkError(
                `HTTP ${response.status}`,
                response.status,
                response.url
            );
        }
        return response.json();
    })
    .then(data => {
        if (!data.valid) {
            throw new ValidationError("Invalid data received", "data");
        }
        return processData(data);
    })
    .catch(error => {
        if (error instanceof NetworkError) {
            console.error("Network issue:", error.message);
            return getCachedData();
        } else if (error instanceof ValidationError) {
            console.error("Data validation failed:", error.message);
            return getDefaultData();
        } else {
            console.error("Unexpected error:", error);
            throw error; // Re-throw unknown errors
        }
    })
    .finally(() => {
        console.log("Request completed");
        hideLoadingSpinner();
    });

// Multiple catch blocks for different error types
Promise.resolve()
    .then(() => riskyOperation1())
    .catch(NetworkError, error => {
        console.log("Network error in operation 1");
        return fallbackData();
    })
    .then(data => riskyOperation2(data))
    .catch(ValidationError, error => {
        console.log("Validation error in operation 2");
        return sanitizeData(data);
    })
    .catch(error => {
        console.log("Any other error:", error);
        throw error;
    });
```

### Async/Await Error Handling
```javascript
async function robustAsyncOperation() {
    var result = null;
    var retryCount = 0;
    var maxRetries = 3;
    
    while (retryCount < maxRetries) {
        try {
            // Primary operation
            result = await primaryDataSource();
            break; // Success, exit retry loop
            
        } catch (error) {
            retryCount++;
            
            if (error instanceof NetworkError) {
                console.warn(`Network error (attempt ${retryCount}):`, error.message);
                
                if (retryCount < maxRetries) {
                    await delay(1000 * retryCount); // Exponential backoff
                    continue;
                }
                
                // Max retries reached, try fallback
                try {
                    result = await fallbackDataSource();
                    break;
                } catch (fallbackError) {
                    throw new Error("Both primary and fallback sources failed");
                }
                
            } else if (error instanceof ValidationError) {
                // Don't retry validation errors
                throw error;
                
            } else {
                // Unknown error, don't retry
                throw new Error(`Unexpected error: ${error.message}`);
            }
        }
    }
    
    return result;
}

// Error handling with specific recovery strategies
async function complexAsyncFlow() {
    try {
        var user = await fetchUser();
        
        try {
            var profile = await fetchProfile(user.id);
        } catch (profileError) {
            console.warn("Profile fetch failed, using basic profile");
            profile = createBasicProfile(user);
        }
        
        try {
            var preferences = await fetchPreferences(user.id);
        } catch (prefError) {
            console.warn("Preferences fetch failed, using defaults");
            preferences = getDefaultPreferences();
        }
        
        return { user, profile, preferences };
        
    } catch (userError) {
        if (userError instanceof NetworkError && userError.status === 401) {
            // Unauthorized - redirect to login
            redirectToLogin();
            throw new Error("Authentication required");
        } else {
            // Other errors - show error page
            throw new Error(`Failed to load user data: ${userError.message}`);
        }
    }
}
```

### Error Boundaries for Async Operations
```javascript
class AsyncErrorBoundary {
    constructor() {
        this.errorHandlers = new Map();
        this.globalHandler = null;
    }
    
    // Register error handler for specific error types
    catch(errorType, handler) {
        this.errorHandlers.set(errorType, handler);
        return this;
    }
    
    // Register global error handler
    catchAll(handler) {
        this.globalHandler = handler;
        return this;
    }
    
    // Execute async operation with error handling
    async execute(asyncOperation) {
        try {
            return await asyncOperation();
        } catch (error) {
            // Check for specific error handlers
            for (var [errorType, handler] of this.errorHandlers) {
                if (error instanceof errorType) {
                    return await handler(error);
                }
            }
            
            // Use global handler if available
            if (this.globalHandler) {
                return await this.globalHandler(error);
            }
            
            // Re-throw if no handler found
            throw error;
        }
    }
}

// Usage
var errorBoundary = new AsyncErrorBoundary()
    .catch(NetworkError, async (error) => {
        console.log("Network error handled");
        return await getCachedData();
    })
    .catch(ValidationError, async (error) => {
        console.log("Validation error handled");
        return getDefaultData();
    })
    .catchAll(async (error) => {
        console.error("Unexpected error:", error);
        await reportError(error);
        throw error;
    });

var result = await errorBoundary.execute(async () => {
    return await riskyAsyncOperation();
});
```

## Debugging Techniques

### Console Debugging Strategies
```javascript
// Structured logging
class Logger {
    static levels = {
        ERROR: 0,
        WARN: 1,
        INFO: 2,
        DEBUG: 3
    };
    
    static currentLevel = Logger.levels.INFO;
    
    static log(level, message, data = null) {
        if (level <= this.currentLevel) {
            var timestamp = new Date().toISOString();
            var levelName = Object.keys(this.levels)[level];
            
            console.log(`[${timestamp}] ${levelName}: ${message}`);
            
            if (data) {
                console.log(data);
            }
        }
    }
    
    static error(message, data) {
        this.log(this.levels.ERROR, message, data);
        console.trace(); // Show stack trace
    }
    
    static warn(message, data) {
        this.log(this.levels.WARN, message, data);
    }
    
    static info(message, data) {
        this.log(this.levels.INFO, message, data);
    }
    
    static debug(message, data) {
        this.log(this.levels.DEBUG, message, data);
    }
}

// Function execution tracing
function trace(target, propertyKey, descriptor) {
    var originalMethod = descriptor.value;
    
    descriptor.value = function(...args) {
        Logger.debug(`Entering ${propertyKey}`, { args });
        
        try {
            var result = originalMethod.apply(this, args);
            Logger.debug(`Exiting ${propertyKey}`, { result });
            return result;
        } catch (error) {
            Logger.error(`Error in ${propertyKey}`, { error, args });
            throw error;
        }
    };
    
    return descriptor;
}

// Performance monitoring
function monitor(name, fn) {
    return function(...args) {
        var start = performance.now();
        
        try {
            var result = fn.apply(this, args);
            var duration = performance.now() - start;
            
            Logger.info(`${name} completed in ${duration.toFixed(2)}ms`);
            return result;
            
        } catch (error) {
            var duration = performance.now() - start;
            Logger.error(`${name} failed after ${duration.toFixed(2)}ms`, error);
            throw error;
        }
    };
}
```

### Browser DevTools Integration
```javascript
// Enhanced debugging utilities
var Debug = {
    // Create debug groups
    group(name, collapsed = false) {
        collapsed ? console.groupCollapsed(name) : console.group(name);
    },
    
    groupEnd() {
        console.groupEnd();
    },
    
    // Conditional breakpoints in code
    breakIf(condition, message = "Debug breakpoint") {
        if (condition) {
            console.log(message);
            debugger; // Triggers breakpoint if DevTools open
        }
    },
    
    // Object inspection
    inspect(obj, depth = 2) {
        console.log(JSON.stringify(obj, null, 2));
    },
    
    // Function call counting
    createCounter(name) {
        var count = 0;
        return function() {
            count++;
            console.log(`${name} called ${count} times`);
        };
    },
    
    // Memory usage tracking
    memorySnapshot() {
        if (performance.memory) {
            console.table({
                used: `${Math.round(performance.memory.usedJSHeapSize / 1048576)} MB`,
                total: `${Math.round(performance.memory.totalJSHeapSize / 1048576)} MB`,
                limit: `${Math.round(performance.memory.jsHeapSizeLimit / 1048576)} MB`
            });
        }
    },
    
    // Network request monitoring
    monitorFetch() {
        var originalFetch = window.fetch;
        
        window.fetch = function(...args) {
            var url = args[0];
            var options = args[1] || {};
            
            console.log(`🌐 Fetch: ${options.method || 'GET'} ${url}`);
            
            return originalFetch.apply(this, args)
                .then(response => {
                    console.log(`✅ Response: ${response.status} ${url}`);
                    return response;
                })
                .catch(error => {
                    console.error(`❌ Fetch error: ${url}`, error);
                    throw error;
                });
        };
    }
};

// Usage
Debug.group("User Loading Process");
Debug.monitorFetch();

var loadUser = Debug.createCounter("loadUser");
loadUser();

Debug.breakIf(user.id === 123, "Debugging specific user");
Debug.inspect(user);
Debug.memorySnapshot();

Debug.groupEnd();
```

### Source Maps and Stack Traces
```javascript
// Stack trace analysis
function analyzeError(error) {
    var stackLines = error.stack.split('\n');
    
    var analysis = {
        errorType: error.name,
        message: error.message,
        stackFrames: []
    };
    
    stackLines.forEach((line, index) => {
        if (index === 0) return; // Skip error message line
        
        var match = line.match(/at\s+(.+?)\s+\((.+?):(\d+):(\d+)\)/);
        if (match) {
            analysis.stackFrames.push({
                function: match[1],
                file: match[2],
                line: parseInt(match[3]),
                column: parseInt(match[4])
            });
        }
    });
    
    return analysis;
}

// Error reporting with context
function reportError(error, context = {}) {
    var errorAnalysis = analyzeError(error);
    
    var report = {
        timestamp: new Date().toISOString(),
        userAgent: navigator.userAgent,
        url: window.location.href,
        userId: context.userId,
        action: context.action,
        error: errorAnalysis,
        browserState: {
            viewport: {
                width: window.innerWidth,
                height: window.innerHeight
            },
            memory: performance.memory ? {
                used: performance.memory.usedJSHeapSize,
                total: performance.memory.totalJSHeapSize
            } : null
        }
    };
    
    // Send to error tracking service
    fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(report)
    }).catch(reportingError => {
        console.error("Failed to report error:", reportingError);
        // Fallback to local storage
        localStorage.setItem(`error_${Date.now()}`, JSON.stringify(report));
    });
}
```

## Testing and Validation

### Unit Testing Error Scenarios
```javascript
// Testing error conditions
function divide(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
        throw new TypeError("Both arguments must be numbers");
    }
    
    if (b === 0) {
        throw new Error("Division by zero is not allowed");
    }
    
    return a / b;
}

// Test suite
function testDivideFunction() {
    var tests = [
        {
            name: "Should divide two positive numbers",
            test: () => {
                var result = divide(10, 2);
                console.assert(result === 5, "Expected 5, got " + result);
            }
        },
        {
            name: "Should throw TypeError for non-numbers",
            test: () => {
                try {
                    divide("10", 2);
                    console.assert(false, "Expected TypeError");
                } catch (error) {
                    console.assert(error instanceof TypeError, "Expected TypeError");
                }
            }
        },
        {
            name: "Should throw Error for division by zero",
            test: () => {
                try {
                    divide(10, 0);
                    console.assert(false, "Expected Error");
                } catch (error) {
                    console.assert(error.message.includes("Division by zero"), 
                                 "Expected division by zero message");
                }
            }
        }
    ];
    
    tests.forEach(({ name, test }) => {
        try {
            test();
            console.log(`✅ ${name}`);
        } catch (error) {
            console.error(`❌ ${name}: ${error.message}`);
        }
    });
}

testDivideFunction();
```

### Property-Based Testing
```javascript
// Generate test data
function generateRandomInput() {
    return {
        numbers: Array.from({ length: 10 }, () => Math.random() * 100),
        strings: Array.from({ length: 5 }, () => Math.random().toString(36)),
        objects: Array.from({ length: 3 }, () => ({
            id: Math.floor(Math.random() * 1000),
            value: Math.random()
        }))
    };
}

// Property-based test runner
function runPropertyTests(fn, property, iterations = 100) {
    for (let i = 0; i < iterations; i++) {
        var input = generateRandomInput();
        
        try {
            var result = fn(input);
            
            if (!property(input, result)) {
                throw new Error(`Property violation with input: ${JSON.stringify(input)}`);
            }
        } catch (error) {
            console.error(`Test failed on iteration ${i}:`, error);
            console.log("Input:", input);
            break;
        }
    }
    
    console.log(`✅ Property test passed ${iterations} iterations`);
}

// Example: Test that array sorting maintains length
runPropertyTests(
    (input) => input.numbers.sort((a, b) => a - b),
    (input, result) => input.numbers.length === result.length
);
```

## Production Error Handling

### Global Error Handlers
```javascript
// Global error handling setup
class GlobalErrorHandler {
    constructor() {
        this.setupHandlers();
        this.errorQueue = [];
        this.isReporting = false;
    }
    
    setupHandlers() {
        // Uncaught JavaScript errors
        window.addEventListener('error', (event) => {
            this.handleError({
                type: 'javascript',
                message: event.message,
                filename: event.filename,
                lineNumber: event.lineno,
                columnNumber: event.colno,
                error: event.error
            });
        });
        
        // Unhandled promise rejections
        window.addEventListener('unhandledrejection', (event) => {
            this.handleError({
                type: 'promise',
                reason: event.reason,
                promise: event.promise
            });
        });
        
        // Resource loading errors
        window.addEventListener('error', (event) => {
            if (event.target !== window) {
                this.handleError({
                    type: 'resource',
                    element: event.target.tagName,
                    source: event.target.src || event.target.href,
                    message: 'Resource failed to load'
                });
            }
        }, true);
    }
    
    handleError(errorInfo) {
        // Add context
        var enrichedError = {
            ...errorInfo,
            timestamp: new Date().toISOString(),
            url: window.location.href,
            userAgent: navigator.userAgent,
            userId: this.getCurrentUserId(),
            sessionId: this.getSessionId()
        };
        
        // Queue for batch reporting
        this.errorQueue.push(enrichedError);
        
        // Report immediately for critical errors
        if (this.isCritical(errorInfo)) {
            this.reportErrors();
        } else {
            // Batch report after delay
            setTimeout(() => this.reportErrors(), 5000);
        }
        
        // Log locally for debugging
        console.error('Global error captured:', enrichedError);
    }
    
    isCritical(errorInfo) {
        return errorInfo.type === 'javascript' && 
               errorInfo.message.includes('ReferenceError');
    }
    
    async reportErrors() {
        if (this.isReporting || this.errorQueue.length === 0) {
            return;
        }
        
        this.isReporting = true;
        var errors = [...this.errorQueue];
        this.errorQueue = [];
        
        try {
            await fetch('/api/errors/batch', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ errors })
            });
        } catch (reportingError) {
            // Fallback: store in localStorage
            var stored = JSON.parse(localStorage.getItem('pendingErrors') || '[]');
            localStorage.setItem('pendingErrors', JSON.stringify([...stored, ...errors]));
        } finally {
            this.isReporting = false;
        }
    }
    
    getCurrentUserId() {
        // Implementation depends on your auth system
        return localStorage.getItem('userId') || 'anonymous';
    }
    
    getSessionId() {
        // Implementation depends on your session management
        return sessionStorage.getItem('sessionId') || 'unknown';
    }
}

// Initialize global error handling
var globalErrorHandler = new GlobalErrorHandler();
```

### Circuit Breaker Pattern
```javascript
class CircuitBreaker {
    constructor(options = {}) {
        this.failureThreshold = options.failureThreshold || 5;
        this.resetTimeout = options.resetTimeout || 60000;
        this.monitoringPeriod = options.monitoringPeriod || 10000;
        
        this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
        this.failureCount = 0;
        this.lastFailureTime = null;
        this.successCount = 0;
    }
    
    async execute(operation) {
        if (this.state === 'OPEN') {
            if (Date.now() - this.lastFailureTime > this.resetTimeout) {
                this.state = 'HALF_OPEN';
                this.successCount = 0;
            } else {
                throw new Error('Circuit breaker is OPEN');
            }
        }
        
        try {
            var result = await operation();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }
    
    onSuccess() {
        this.failureCount = 0;
        
        if (this.state === 'HALF_OPEN') {
            this.successCount++;
            if (this.successCount >= 3) { // Require 3 successes to close
                this.state = 'CLOSED';
            }
        }
    }
    
    onFailure() {
        this.failureCount++;
        this.lastFailureTime = Date.now();
        
        if (this.failureCount >= this.failureThreshold) {
            this.state = 'OPEN';
        }
    }
    
    getState() {
        return {
            state: this.state,
            failureCount: this.failureCount,
            lastFailureTime: this.lastFailureTime
        };
    }
}

// Usage
var apiCircuitBreaker = new CircuitBreaker({
    failureThreshold: 3,
    resetTimeout: 30000
});

async function makeApiCall(url) {
    return apiCircuitBreaker.execute(async () => {
        var response = await fetch(url);
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        return response.json();
    });
}
```

## Interview Questions & Challenges

### Question 1: Error Handling Chain
```javascript
// What's wrong with this error handling? Fix it.
async function processUserOrder(orderId) {
    var order = await fetchOrder(orderId)
        .catch(error => {
            console.log("Order not found, creating new one");
            return createNewOrder(orderId);
        });
    
    var payment = await processPayment(order.amount)
        .catch(error => {
            console.log("Payment failed, trying backup processor");
            return backupPaymentProcessor(order.amount);
        });
    
    var shipping = await arrangeShipping(order)
        .catch(error => {
            console.log("Shipping failed, using default option");
            return defaultShipping(order);
        });
    
    return { order, payment, shipping };
}

// Issues:
// 1. Swallowing all error types equally
// 2. No error propagation for critical failures
// 3. No cleanup on partial failures
// 4. No transaction rollback
```

### Question 2: Implement Error Retry Logic
```javascript
// Create a retry function that:
// - Retries failed operations with exponential backoff
// - Only retries specific error types
// - Has maximum retry limit
// - Calls different handlers for different errors

async function retryWithBackoff(operation, options = {}) {
    // Your implementation here
}

// Usage
var result = await retryWithBackoff(
    () => unreliableApiCall(),
    {
        maxRetries: 3,
        initialDelay: 1000,
        backoffMultiplier: 2,
        retryOn: [NetworkError, TimeoutError],
        onRetry: (error, attempt) => console.log(`Retry ${attempt}: ${error.message}`)
    }
);
```

### Question 3: Debug Function Wrapper
```javascript
// Create a debugging wrapper that:
// - Logs function calls with arguments
// - Measures execution time
// - Catches and logs errors
// - Optionally replays function calls

function createDebugWrapper(fn, options = {}) {
    // Your implementation here
    // Should return wrapped function with debug capabilities
}

var debuggedFunction = createDebugWrapper(expensiveFunction, {
    logCalls: true,
    measureTime: true,
    catchErrors: true,
    replayEnabled: true
});
```

### Question 4: Error Aggregation System
```javascript
// Create an error aggregation system that:
// - Collects similar errors
// - Deduplicates based on stack trace
// - Provides error statistics
// - Triggers alerts for error spikes

class ErrorAggregator {
    constructor(options = {}) {
        // Your implementation here
        // Should support: collect(error), getStats(), getTopErrors()
    }
    
    collect(error) {
        // Group similar errors and count occurrences
    }
    
    getStats() {
        // Return error statistics
    }
    
    getTopErrors(limit = 10) {
        // Return most frequent errors
    }
}

var aggregator = new ErrorAggregator();
aggregator.collect(new Error("Database connection failed"));
console.log(aggregator.getStats());
```

### Question 5: Production Error Recovery
```javascript
// Design an error recovery system for a critical user action
function createRobustUserAction(primaryAction, options = {}) {
    // Your implementation should handle:
    // - Multiple fallback strategies
    // - Partial operation rollback
    // - User notification
    // - Error reporting
    // - Graceful degradation
    
    return async function(userData) {
        // Implementation here
    };
}

// Usage
var saveUserProfile = createRobustUserAction(
    async (userData) => await api.saveProfile(userData),
    {
        fallbacks: [
            async (userData) => await localStorage.saveProfile(userData),
            async (userData) => await indexedDB.saveProfile(userData)
        ],
        rollbackOn: [DatabaseError, ValidationError],
        notifyUser: true,
        reportErrors: true
    }
);
```

## Error Monitoring and Analytics

### Real-time Error Tracking
```javascript
class ErrorTracker {
    constructor() {
        this.errors = new Map(); // errorHash -> errorData
        this.timeWindows = new Map(); // timeWindow -> errorCounts
        this.listeners = [];
        this.startMonitoring();
    }
    
    track(error, context = {}) {
        var errorHash = this.generateErrorHash(error);
        var now = Date.now();
        var timeWindow = Math.floor(now / 60000) * 60000; // 1-minute windows
        
        // Update error data
        if (!this.errors.has(errorHash)) {
            this.errors.set(errorHash, {
                error: error,
                firstSeen: now,
                lastSeen: now,
                count: 0,
                contexts: []
            });
        }
        
        var errorData = this.errors.get(errorHash);
        errorData.count++;
        errorData.lastSeen = now;
        errorData.contexts.push(context);
        
        // Update time window counts
        if (!this.timeWindows.has(timeWindow)) {
            this.timeWindows.set(timeWindow, new Map());
        }
        
        var windowData = this.timeWindows.get(timeWindow);
        windowData.set(errorHash, (windowData.get(errorHash) || 0) + 1);
        
        // Check for error spikes
        this.checkForSpikes(errorHash, timeWindow);
        
        // Notify listeners
        this.notifyListeners('error', { errorHash, errorData, context });
    }
    
    generateErrorHash(error) {
        // Create hash based on error type, message, and stack location
        var stackLine = error.stack ? error.stack.split('\n')[1] : '';
        return btoa(`${error.name}:${error.message}:${stackLine}`);
    }
    
    checkForSpikes(errorHash, currentWindow) {
        var previousWindow = currentWindow - 60000;
        var currentCount = this.timeWindows.get(currentWindow)?.get(errorHash) || 0;
        var previousCount = this.timeWindows.get(previousWindow)?.get(errorHash) || 0;
        
        // Alert if error count increased by 300% or more
        if (currentCount >= 5 && currentCount > previousCount * 3) {
            this.notifyListeners('spike', {
                errorHash,
                currentCount,
                previousCount,
                increaseRatio: currentCount / Math.max(previousCount, 1)
            });
        }
    }
    
    startMonitoring() {
        // Clean up old time windows every 5 minutes
        setInterval(() => {
            var cutoff = Date.now() - 3600000; // Keep 1 hour of data
            for (var [timeWindow] of this.timeWindows) {
                if (timeWindow < cutoff) {
                    this.timeWindows.delete(timeWindow);
                }
            }
        }, 300000);
    }
    
    subscribe(listener) {
        this.listeners.push(listener);
        return () => {
            var index = this.listeners.indexOf(listener);
            if (index > -1) this.listeners.splice(index, 1);
        };
    }
    
    notifyListeners(type, data) {
        this.listeners.forEach(listener => {
            try {
                listener(type, data);
            } catch (error) {
                console.error('Error in tracker listener:', error);
            }
        });
    }
    
    getErrorReport() {
        var errors = Array.from(this.errors.entries()).map(([hash, data]) => ({
            hash,
            ...data,
            frequency: data.count / ((data.lastSeen - data.firstSeen) / 1000 || 1) // errors per second
        }));
        
        return {
            totalErrors: errors.reduce((sum, error) => sum + error.count, 0),
            uniqueErrors: errors.length,
            topErrors: errors.sort((a, b) => b.count - a.count).slice(0, 10),
            recentSpikes: this.getRecentSpikes()
        };
    }
    
    getRecentSpikes() {
        // Implementation for recent spike detection
        return [];
    }
}

// Global error tracker instance
var errorTracker = new ErrorTracker();

// Subscribe to alerts
errorTracker.subscribe((type, data) => {
    if (type === 'spike') {
        console.warn(`🚨 Error spike detected: ${data.errorHash}`);
        console.warn(`Count increased from ${data.previousCount} to ${data.currentCount}`);
        
        // Send alert to monitoring service
        fetch('/api/alerts', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                type: 'error_spike',
                data: data,
                timestamp: new Date().toISOString()
            })
        });
    }
});
```

### Performance Impact Analysis
```javascript
class PerformanceErrorAnalyzer {
    constructor() {
        this.performanceData = [];
        this.errorData = [];
        this.correlations = new Map();
    }
    
    recordPerformance(metric, value, timestamp = Date.now()) {
        this.performanceData.push({ metric, value, timestamp });
        
        // Keep only last hour of data
        var cutoff = timestamp - 3600000;
        this.performanceData = this.performanceData.filter(p => p.timestamp > cutoff);
    }
    
    recordError(error, timestamp = Date.now()) {
        this.errorData.push({ error, timestamp });
        
        // Keep only last hour of data
        var cutoff = timestamp - 3600000;
        this.errorData = this.errorData.filter(e => e.timestamp > cutoff);
        
        // Analyze correlation with performance
        this.analyzeCorrelation(error, timestamp);
    }
    
    analyzeCorrelation(error, errorTimestamp) {
        var timeWindow = 300000; // 5 minute window
        var windowStart = errorTimestamp - timeWindow;
        var windowEnd = errorTimestamp + timeWindow;
        
        // Get performance data around error time
        var nearbyPerformance = this.performanceData.filter(p => 
            p.timestamp >= windowStart && p.timestamp <= windowEnd
        );
        
        // Calculate average performance metrics
        var metrics = {};
        nearbyPerformance.forEach(p => {
            if (!metrics[p.metric]) {
                metrics[p.metric] = [];
            }
            metrics[p.metric].push(p.value);
        });
        
        // Store correlation data
        var errorHash = this.generateErrorHash(error);
        if (!this.correlations.has(errorHash)) {
            this.correlations.set(errorHash, {
                errorType: error.name,
                performanceImpact: {},
                occurrences: 0
            });
        }
        
        var correlation = this.correlations.get(errorHash);
        correlation.occurrences++;
        
        Object.keys(metrics).forEach(metric => {
            var values = metrics[metric];
            var average = values.reduce((a, b) => a + b, 0) / values.length;
            
            if (!correlation.performanceImpact[metric]) {
                correlation.performanceImpact[metric] = [];
            }
            correlation.performanceImpact[metric].push(average);
        });
    }
    
    generateErrorHash(error) {
        return btoa(`${error.name}:${error.message}`);
    }
    
    getPerformanceReport() {
        var report = {
            correlations: [],
            summary: {
                totalErrors: this.errorData.length,
                avgMemoryUsage: 0,
                avgResponseTime: 0
            }
        };
        
        // Calculate correlations
        for (var [errorHash, data] of this.correlations) {
            var correlationData = {
                errorHash,
                errorType: data.errorType,
                occurrences: data.occurrences,
                performanceImpact: {}
            };
            
            Object.keys(data.performanceImpact).forEach(metric => {
                var values = data.performanceImpact[metric];
                correlationData.performanceImpact[metric] = {
                    average: values.reduce((a, b) => a + b, 0) / values.length,
                    min: Math.min(...values),
                    max: Math.max(...values)
                };
            });
            
            report.correlations.push(correlationData);
        }
        
        return report;
    }
}

// Usage
var performanceAnalyzer = new PerformanceErrorAnalyzer();

// Record performance metrics
setInterval(() => {
    if (performance.memory) {
        performanceAnalyzer.recordPerformance('memoryUsage', performance.memory.usedJSHeapSize);
    }
    
    // Record response times from navigation timing
    if (performance.timing) {
        var responseTime = performance.timing.responseEnd - performance.timing.requestStart;
        performanceAnalyzer.recordPerformance('responseTime', responseTime);
    }
}, 1000);

// Hook into error tracking
window.addEventListener('error', (event) => {
    performanceAnalyzer.recordError(event.error);
});
```

## Key Takeaways for Interviews

1. **Error types matter** - use specific error classes for better handling
2. **Fail fast vs graceful degradation** - choose strategy based on criticality
3. **Async error handling** requires try-catch with async/await or .catch() with promises
4. **Global error handlers** are essential for production applications
5. **Stack traces provide debugging context** - preserve them in custom errors
6. **Circuit breakers** prevent cascading failures in distributed systems
7. **Error monitoring** helps identify patterns and performance impacts
8. **Testing error conditions** is as important as testing happy paths
9. **Error boundaries** isolate failures and enable recovery
10. **Structured logging** makes debugging in production much easier

## Practice Challenges

### Challenge 1: Robust API Client
```javascript
// Create an API client with comprehensive error handling
class RobustApiClient {
    constructor(baseUrl, options = {}) {
        // Your implementation
        // Should support:
        // - Automatic retries with backoff
        // - Request/response interceptors
        // - Circuit breaker pattern
        // - Request deduplication
        // - Error categorization
    }
    
    async get(endpoint, options = {}) {
        // Implementation here
    }
    
    async post(endpoint, data, options = {}) {
        // Implementation here
    }
}

var apiClient = new RobustApiClient('/api', {
    retries: 3,
    timeout: 5000,
    circuitBreaker: true
});
```

### Challenge 2: Error Recovery Middleware
```javascript
// Create middleware that automatically recovers from errors
function createErrorRecoveryMiddleware(strategies) {
    // Your implementation
    // Should try multiple recovery strategies in order
    // Log recovery attempts and results
    // Support both sync and async operations
    
    return function(operation) {
        // Return wrapped operation with error recovery
    };
}

var robustOperation = createErrorRecoveryMiddleware([
    'retry',
    'fallback',
    'cache',
    'default'
])(riskyOperation);
```

### Challenge 3: Development vs Production Error Handling
```javascript
// Create an error handling system that behaves differently in dev vs prod
class EnvironmentAwareErrorHandler {
    constructor(environment) {
        // Your implementation
        // Development: detailed errors, stack traces, debugging info
        // Production: sanitized errors, error reporting, user-friendly messages
    }
    
    handle(error, context = {}) {
        // Handle based on environment
    }
}

var errorHandler = new EnvironmentAwareErrorHandler(process.env.NODE_ENV);
```

This completes **Module 7: Error Handling & Debugging** with comprehensive coverage of error types, handling strategies, debugging techniques, and production-ready error management systems. The module provides both theoretical knowledge and practical implementation patterns that are essential for building robust JavaScript applications.

Ready for the next module when you are!