# Module 3: TypeScript Functions and Modules Cheatsheet

## Core Problem
JavaScript's loose approach to function parameters, return values, and module organization can lead to runtime errors and maintainability issues. TypeScript solves these problems by adding strong typing to functions and providing robust module systems for better code organization.

## Key Assumptions
- You have a basic understanding of TypeScript syntax and types
- You are familiar with JavaScript functions
- You have completed Modules 1 and 2 or have equivalent knowledge

## Essential Concepts

### 1. Functions

#### 1.1 Function Declarations and Expressions

**Concise Explanation:**  
TypeScript supports two main ways to define functions: function declarations (hoisted, named functions) and function expressions (assigned to variables). Both allow for type annotations on parameters and return values, enabling compile-time type checking.

**Where to Use:**
- Function declarations: For top-level functions that need to be hoisted or called before declaration
- Function expressions: For function assignments, callbacks, or when functions should not be hoisted

**Code Snippet:**
```typescript
// Function declaration with type annotations
function multiply(a: number, b: number): number {
  return a * b;
}

// Function expression with type annotations
const divide = function(a: number, b: number): number {
  if (b === 0) {
    throw new Error("Cannot divide by zero");
  }
  return a / b;
};

// Function expression with inferred return type
const add = function(a: number, b: number) {
  return a + b;  // Return type is inferred as number
};

// Anonymous function as callback
[1, 2, 3].map(function(num: number): string {
  return `Number: ${num}`;
});
```

**Real-World Example:**  
A utility library for form validation:

```typescript
// Form validation library
// Function declaration for a public API function
export function validateEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

// Function expression for a helper function
const formatErrorMessage = function(field: string, message: string): string {
  return `${field}: ${message}`;
};

// Usage in a form validation context
export function validateForm(form: { email: string; password: string }): string[] {
  const errors: string[] = [];
  
  if (!validateEmail(form.email)) {
    errors.push(formatErrorMessage("Email", "Invalid email format"));
  }
  
  if (form.password.length < 8) {
    errors.push(formatErrorMessage("Password", "Password must be at least 8 characters"));
  }
  
  return errors;
}
```

**Common Pitfalls:**
- Forgetting that function declarations are hoisted but function expressions are not
- Omitting return type annotations for complex functions, making it harder to identify incorrect return values
- Not being explicit about void returns when a function doesn't return anything
- Inconsistent use of declaration styles across a codebase, making patterns harder to recognize

**Three Common Questions:**

1. **Q: Should I always specify the return type for functions or let TypeScript infer it?**  
   **A:** It's a best practice to explicitly annotate return types for public API functions and complex functions. This serves as documentation and catches errors if implementation changes break the expected return type. For simple functions with obvious returns, inference can reduce verbosity.

2. **Q: What's the difference between `function foo() {}` and `const foo = function() {}`?**  
   **A:** The first is a function declaration that's hoisted (can be called before its definition in code). The second is a function expression assigned to a variable, which isn't hoisted. Function declarations are useful for top-level functions, while expressions are often used for callbacks, immediately-invoked functions, or where block-scoping matters.

3. **Q: Can I mix function declarations and expressions in my code?**  
   **A:** Yes, but be consistent based on your use case. Use declarations for stable, named functions in your API. Use expressions when treating functions as values, for callbacks, or when you need to leverage closures. Some teams adopt style guides that prefer one form over the other for consistency.

#### 1.2 Parameters and Return Types

**Concise Explanation:**  
TypeScript allows you to annotate function parameters and return types, ensuring inputs and outputs match expected types. The compiler checks every function call to make sure you're passing the right arguments and using the return values correctly.

**Where to Use:**
- Parameter types: Always, to ensure functions receive correct inputs
- Return types: For public APIs, complex functions, or when inference isn't clear
- Void return type: When a function doesn't return a usable value

**Code Snippet:**
```typescript
// Basic parameter and return types
function calculateArea(width: number, height: number): number {
  return width * height;
}

// Multiple parameter types
function formatName(first: string, last: string): string {
  return `${first} ${last}`;
}

// No return value (void)
function logMessage(message: string): void {
  console.log(message);
  // No return statement needed
}

// Return type with union
function findElement(id: string): HTMLElement | null {
  return document.getElementById(id);
}

// Object parameter with interface
interface User {
  id: number;
  name: string;
}

function getUserDisplayName(user: User): string {
  return `${user.name} (ID: ${user.id})`;
}

// Function with callbacks
function fetchData(
  url: string, 
  onSuccess: (data: object) => void, 
  onError: (error: Error) => void
): void {
  // Implementation omitted
}
```

**Real-World Example:**  
API client for a weather service:

```typescript
interface WeatherData {
  location: {
    city: string;
    country: string;
    coordinates: [number, number]; // [latitude, longitude]
  };
  currentConditions: {
    temperature: number;
    humidity: number;
    windSpeed: number;
    description: string;
  };
  forecast: Array<{
    date: string;
    minTemp: number;
    maxTemp: number;
    condition: string;
  }>;
}

interface ApiOptions {
  units: 'metric' | 'imperial';
  language: string;
  timeout: number;
}

// API client with strongly typed parameters and returns
class WeatherApiClient {
  private apiKey: string;
  private baseUrl: string;
  
  constructor(apiKey: string, baseUrl: string = 'https://api.weather.example') {
    this.apiKey = apiKey;
    this.baseUrl = baseUrl;
  }
  
  async getCurrentWeather(city: string, options: ApiOptions): Promise<WeatherData> {
    const url = this.buildUrl('/current', { city, ...options });
    
    try {
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`API error: ${response.status}`);
      }
      
      return await response.json() as WeatherData;
    } catch (error) {
      throw new Error(`Failed to fetch weather: ${error.message}`);
    }
  }
  
  async getForecasts(
    days: number, 
    cities: string[], 
    options: Partial<ApiOptions>
  ): Promise<Record<string, WeatherData>> {
    // Implementation omitted
    return {}; // Just for example
  }
  
  private buildUrl(endpoint: string, params: Record<string, any>): string {
    const queryString = new URLSearchParams({
      ...params,
      apiKey: this.apiKey
    }).toString();
    
    return `${this.baseUrl}${endpoint}?${queryString}`;
  }
}
```

**Common Pitfalls:**
- Not accounting for null or undefined values in parameter/return types
- Overly restrictive parameter types that make functions less reusable
- Forgetting to check for undefined parameters when making them optional
- Not using union types when a function can return different types based on inputs
- Using `any` for complex parameters instead of defining proper interfaces

**Three Common Questions:**

1. **Q: When should I use `void` versus `undefined` as a return type?**  
   **A:** Use `void` when the function doesn't return any value or when you want to explicitly ignore any returned value. Use `undefined` when the function intentionally returns the undefined value that might be used. For example: `function logMessage(): void` vs `function getValue(): string | undefined`.

2. **Q: How do TypeScript function types handle extra parameters?**  
   **A:** TypeScript uses structural typing for functions, so it only checks that the required parameters are provided. Additional parameters are allowed but ignored. This matches JavaScript's behavior where extra arguments don't cause errors.

3. **Q: Can I specify types for `this` inside a function?**  
   **A:** Yes, TypeScript allows specifying the `this` type as the first parameter of a function declaration (though it's removed in the compiled JavaScript). Example: `function handleClick(this: HTMLButton, event: Event) { console.log(this.id); }`. This ensures `this` has the right type when used.

#### 1.3 Optional and Default Parameters

**Concise Explanation:**  
TypeScript allows you to make parameters optional by adding a `?` after the parameter name, or provide default values that are used when the parameter is omitted. Optional parameters must appear after required ones, while default parameters can appear anywhere.

**Where to Use:**
- Optional parameters: When a parameter isn't always needed
- Default parameters: When a parameter has a sensible default value
- Both: In public APIs to make functions more flexible but still type-safe

**Code Snippet:**
```typescript
// Optional parameters (with ?)
function greet(name: string, title?: string): string {
  if (title) {
    return `Hello, ${title} ${name}!`;
  }
  return `Hello, ${name}!`;
}

// Usage
greet("John");         // Valid: "Hello, John!"
greet("John", "Mr.");  // Valid: "Hello, Mr. John!"

// Default parameters
function createUser(name: string, active: boolean = true, role: string = "user"): object {
  return { name, active, role };
}

// Usage
createUser("Alice");                   // { name: "Alice", active: true, role: "user" }
createUser("Bob", false);              // { name: "Bob", active: false, role: "user" }
createUser("Charlie", undefined, "admin"); // { name: "Charlie", active: true, role: "admin" }

// Combining optional and default parameters
function fetchData(
  url: string, 
  method: string = "GET",
  timeout?: number,  // Optional parameter
  headers: Record<string, string> = {}  // Default parameter
): Promise<Response> {
  const options = {
    method,
    headers,
    // Only add timeout if it exists
    ...(timeout !== undefined && { timeout })
  };
  
  return fetch(url, options);
}
```

**Real-World Example:**  
Configuration manager for an application:

```typescript
interface LoggerConfig {
  level: 'debug' | 'info' | 'warn' | 'error';
  format?: 'text' | 'json';
  destination?: 'console' | 'file';
  filepath?: string;
  maxSize?: number;
}

class ConfigManager {
  // Default configuration with some required and some optional parameters
  initializeLogger(
    appName: string,
    level: LoggerConfig['level'] = 'info',
    options?: Partial<Omit<LoggerConfig, 'level'>>
  ): LoggerConfig {
    // Start with defaults
    const config: LoggerConfig = {
      level,
      format: 'text',
      destination: 'console'
    };
    
    // Override with any provided options
    if (options) {
      Object.assign(config, options);
      
      // Add validation for specific combinations
      if (options.destination === 'file' && !options.filepath) {
        throw new Error('Filepath is required when destination is file');
      }
    }
    
    console.log(`Initializing logger for ${appName} with level ${level}`);
    return config;
  }
  
  // Function with multiple optional parameters
  createDatabaseConnection(
    host: string,
    user: string,
    password: string,
    database?: string,  // Optional
    port: number = 5432,  // Default
    ssl: boolean = false,  // Default
    timeout: number = 30000  // Default
  ) {
    const config = {
      host,
      user,
      password,
      port,
      ssl,
      timeout
    };
    
    if (database) {
      return this.connectToSpecificDB(config, database);
    } else {
      return this.connectToDefaultDB(config);
    }
  }
  
  private connectToSpecificDB(config: object, database: string) {
    // Implementation omitted
    return { connection: "established", database };
  }
  
  private connectToDefaultDB(config: object) {
    // Implementation omitted
    return { connection: "established", database: "default" };
  }
}
```

**Common Pitfalls:**
- Putting required parameters after optional parameters (TypeScript will raise an error)
- Not handling the case where an optional parameter is undefined
- Using default parameters in types/interfaces, which isn't supported
- Confusing the behavior of `null` vs `undefined` for optional parameters
- Not accounting for explicitly passed `undefined` which will trigger the default value

**Three Common Questions:**

1. **Q: What's the difference between `parameter?: type` and `parameter: type = defaultValue`?**  
   **A:** An optional parameter (`parameter?: type`) can be omitted but will be `undefined` if not provided. A parameter with a default value (`parameter: type = defaultValue`) can also be omitted but will use the default value instead of being undefined. Default parameters are more convenient when you have a common fallback value.

2. **Q: Can I make a parameter both optional and have a default value?**  
   **A:** This is redundant since a default value already makes the parameter optional. If you write `function(name?: string = "default")`, `name` will be "default" if omitted or explicitly set to undefined. Just use `function(name: string = "default")`.

3. **Q: How do optional parameters work with function type definitions?**  
   **A:** When defining function types or interfaces, optional parameters are declared the same way: `type Callback = (id: number, name?: string) => void`. A function that requires the optional parameter would not be assignable to this type, but a function that doesn't use the parameter would be compatible.

#### 1.4 Rest Parameters and Spread Operator

**Concise Explanation:**  
Rest parameters collect multiple arguments into a single array parameter, allowing functions to accept a variable number of arguments. The spread operator does the opposite, expanding an array into individual arguments when calling a function. Both use the `...` syntax but in different contexts.

**Where to Use:**
- Rest parameters: When accepting an unknown number of arguments of the same type
- Spread operator: When passing array elements as individual arguments to a function
- Both: When working with variable-length inputs or combining arrays

**Code Snippet:**
```typescript
// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, num) => total + num, 0);
}

// Usage
console.log(sum(1, 2));        // 3
console.log(sum(1, 2, 3, 4));  // 10

// Rest parameter with other parameters
function logMessage(message: string, ...tags: string[]): void {
  console.log(`${message} | Tags: ${tags.join(', ')}`);
}

// Usage
logMessage("User logged in", "auth", "user");  // "User logged in | Tags: auth, user"

// Typed rest parameters with union types
function processItems(...items: (string | number)[]): void {
  items.forEach(item => {
    if (typeof item === 'string') {
      console.log(`String: ${item.toUpperCase()}`);
    } else {
      console.log(`Number: ${item.toFixed(2)}`);
    }
  });
}

// Spread operator in function calls
const numbers = [1, 2, 3, 4];
console.log(sum(...numbers));  // 10

// Spread with arrays
const baseColors = ['red', 'green', 'blue'];
const extendedColors = [...baseColors, 'yellow', 'purple'];

// Spread with objects
const defaults = { timeout: 1000, retries: 3 };
const userConfig = { timeout: 2000 };
const config = { ...defaults, ...userConfig };  // { timeout: 2000, retries: 3 }
```

**Real-World Example:**  
Event system in a component library:

```typescript
type EventHandler = (...args: any[]) => void;

class EventEmitter {
  private events: Record<string, EventHandler[]> = {};
  
  // Register event handlers using rest parameters
  on(event: string, ...handlers: EventHandler[]): void {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    
    this.events[event].push(...handlers);  // Spread operators to add all handlers
  }
  
  // Trigger event with any number of arguments
  emit(event: string, ...args: any[]): boolean {
    const handlers = this.events[event];
    
    if (!handlers || handlers.length === 0) {
      return false;
    }
    
    handlers.forEach(handler => {
      handler(...args);  // Spread the args when calling the handler
    });
    
    return true;
  }
  
  // Remove specific handler
  off(event: string, handler: EventHandler): void {
    const handlers = this.events[event];
    
    if (!handlers) return;
    
    this.events[event] = handlers.filter(h => h !== handler);
  }
}

// Usage example
const events = new EventEmitter();

function userCreated(id: number, name: string) {
  console.log(`User created: ${name} (ID: ${id})`);
}

function userActivity(id: number, name: string) {
  console.log(`User activity: ${name}`);
}

// Register multiple handlers at once
events.on('user:create', userCreated, userActivity);

// Emit with multiple arguments
events.emit('user:create', 1, 'John Doe');  // Triggers both handlers
```

**Common Pitfalls:**
- Placing rest parameters before other parameters (they must be last)
- Confusing rest parameters in function definitions (`function f(...args)`) with spread in function calls (`f(...array)`)
- Not properly typing rest parameters, leading to type errors when processing the arguments
- Not handling empty arrays when using spread (can be a problem if the function expects at least one argument)
- Excessive use of rest parameters when specific argument types would be clearer

**Three Common Questions:**

1. **Q: Can I use multiple rest parameters in a single function?**  
   **A:** No, you can have only one rest parameter in a function, and it must be the last parameter. This is because rest parameters collect "all remaining arguments," so having another parameter after them wouldn't make sense.

2. **Q: What happens if I spread an empty array into a function call?**  
   **A:** It's equivalent to calling the function with no arguments for that spread position. For example, `f(...[])` is the same as `f()`. If the function requires arguments in that position, you'll need to handle this case separately.

3. **Q: How does TypeScript handle different types in rest parameters?**  
   **A:** You can use union types for rest parameters like `...args: (string | number)[]`, or tuple types for more specific patterns like `...args: [string, number, ...boolean[]]` which requires a string, then a number, followed by any number of booleans.

#### 1.5 Function Overloading

**Concise Explanation:**  
Function overloading allows you to define multiple function signatures for the same function, enabling it to handle different parameter types and return values. In TypeScript, you define overload signatures first, followed by an implementation signature that is compatible with all the overloads.

**Where to Use:**
- When a function behaves differently based on parameter types or counts
- API functions that need to handle different input formats
- When the return type depends on the input types
- When creating flexible utility functions

**Code Snippet:**
```typescript
// Function overloading - multiple signatures
// Overload signatures
function parseValue(value: string): string[];
function parseValue(value: number): string;
function parseValue(value: boolean): string;
// Implementation signature - must be compatible with all overloads
function parseValue(value: string | number | boolean): string | string[] {
  if (typeof value === 'string') {
    return value.split(',');
  } else if (typeof value === 'number') {
    return value.toString();
  } else {
    return value ? 'true' : 'false';
  }
}

// Usage
const result1 = parseValue('apple,orange,banana');  // string[]
const result2 = parseValue(42);                     // string
const result3 = parseValue(true);                   // string

// Overloads with different parameter counts
function createElement(tag: string): HTMLElement;
function createElement(tag: string, text: string): HTMLElement;
function createElement(tag: string, options: object): HTMLElement;
function createElement(tag: string, textOrOptions?: string | object): HTMLElement {
  const element = document.createElement(tag);
  
  if (textOrOptions) {
    if (typeof textOrOptions === 'string') {
      element.textContent = textOrOptions;
    } else {
      Object.assign(element, textOrOptions);
    }
  }
  
  return element;
}

// Usage
const div1 = createElement('div');
const div2 = createElement('div', 'Hello world');
const div3 = createElement('div', { className: 'container', id: 'main' });
```

**Real-World Example:**  
A utility function for handling different data fetching scenarios:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

interface Post {
  id: number;
  title: string;
  content: string;
  userId: number;
}

// API client with overloads for different entity types
// Overload for fetching a user
function fetchData(endpoint: 'users', id: number): Promise<User>;
// Overload for fetching all users
function fetchData(endpoint: 'users'): Promise<User[]>;
// Overload for fetching a post
function fetchData(endpoint: 'posts', id: number): Promise<Post>;
// Overload for fetching all posts
function fetchData(endpoint: 'posts'): Promise<Post[]>;
// Overload for fetching posts by user
function fetchData(endpoint: 'posts', userId: number, byUser: true): Promise<Post[]>;

// Implementation that handles all overloads
async function fetchData(
  endpoint: 'users' | 'posts',
  idOrUserId?: number,
  byUser?: boolean
): Promise<User | User[] | Post | Post[]> {
  const baseUrl = 'https://api.example.com';
  let url = `${baseUrl}/${endpoint}`;
  
  if (idOrUserId !== undefined) {
    if (byUser && endpoint === 'posts') {
      url += `?userId=${idOrUserId}`;
    } else {
      url += `/${idOrUserId}`;
    }
  }
  
  const response = await fetch(url);
  
  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }
  
  return response.json();
}

// Usage with proper type inference
async function loadDashboard() {
  // Gets typed as User[]
  const users = await fetchData('users');
  
  // Gets typed as User
  const currentUser = await fetchData('users', 1);
  
  // Gets typed as Post[]
  const allPosts = await fetchData('posts');
  
  // Gets typed as Post
  const featuredPost = await fetchData('posts', 42);
  
  // Gets typed as Post[]
  const userPosts = await fetchData('posts', currentUser.id, true);
  
  return {
    users,
    currentUser,
    featuredPost,
    userPosts
  };
}
```

**Common Pitfalls:**
- Implementation signature not compatible with all overload signatures
- Overcomplicating function implementations to handle too many overloads
- Not handling all edge cases in the implementation
- Overly broad implementation signature making type checking less effective
- Inconsistent return types between overloads causing confusion

**Three Common Questions:**

1. **Q: Why do I need an implementation signature after my overload signatures?**  
   **A:** In TypeScript, the overload signatures only exist for type checking during compilation. The implementation signature is what actually gets compiled to JavaScript and must be compatible with all overloads. The implementation signature isn't exposed to users of your function; they only see the overload signatures.

2. **Q: Can I use function overloading with arrow functions?**  
   **A:** Not directly. Function overloading only works with function declarations and function expressions. For arrow functions, you can use method overloading in a class or create a function declaration with overloads and assign it to a variable later.

3. **Q: How does TypeScript determine which overload to use?**  
   **A:** TypeScript uses the first matching overload signature based on the arguments provided. It tries each overload in order from top to bottom, so more specific overloads should come before more general ones. If none match exactly, it falls back to the implementation signature, which may lead to type errors.

#### 1.6 Arrow Functions and Lexical this

**Concise Explanation:**  
Arrow functions provide a concise syntax for creating functions and capture `this` from their surrounding context (lexical `this`). Unlike regular functions, arrow functions don't have their own `this` binding, making them ideal for callbacks where you want to preserve the outer `this` context.

**Where to Use:**
- Short, single-expression functions
- Callbacks where you need to access the outer `this` context
- Event handlers in classes
- Functional programming patterns like map, filter, reduce

**Code Snippet:**
```typescript
// Basic arrow function
const add = (a: number, b: number): number => a + b;

// Multi-line arrow function with explicit return
const multiply = (a: number, b: number): number => {
  const result = a * b;
  return result;
};

// Arrow function with no parameters
const getRandomNumber = (): number => Math.random();

// Arrow function with a single parameter (parentheses optional)
const double = (num: number): number => num * 2;

// Arrow functions with type annotations
const greet = (name: string): string => `Hello, ${name}!`;

// Arrow function as a callback
const numbers = [1, 2, 3, 4];
const squared = numbers.map((num: number): number => num * num);

// "this" in arrow functions vs regular functions
class Counter {
  count = 0;
  
  // Method using regular function - "this" will change based on caller
  incrementProblem() {
    setTimeout(function() {
      this.count++; // Error: "this" refers to the global object/undefined in strict mode
      console.log(this.count); // NaN
    }, 1000);
  }
  
  // Method using arrow function - "this" is lexically bound to Counter instance
  increment() {
    setTimeout(() => {
      this.count++; // "this" still refers to the Counter instance
      console.log(this.count); // Works as expected
    }, 1000);
  }
  
  // Alternative with explicit binding
  incrementWithBind() {
    setTimeout(function() {
      this.count++;
      console.log(this.count);
    }.bind(this), 1000);
  }
}
```

**Real-World Example:**  
Event handling in a React component:

```typescript
import React, { useState, useEffect } from 'react';

interface User {
  id: number;
  name: string;
}

// React component using arrow functions for event handlers
const UserListComponent: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [selectedId, setSelectedId] = useState<number | null>(null);
  
  // Arrow function for async operations
  const fetchUsers = async (): Promise<void> => {
    try {
      const response = await fetch('https://api.example.com/users');
      const data = await response.json();
      setUsers(data);
    } catch (error) {
      console.error('Failed to fetch users:', error);
    }
  };
  
  // Arrow function as effect callback
  useEffect(() => {
    fetchUsers();
    
    // Arrow function as cleanup
    return () => {
      console.log('Component unmounting');
    };
  }, []);
  
  // Arrow function as event handler
  const handleUserSelect = (id: number): void => {
    setSelectedId(id);
  };
  
  // Arrow function with conditional logic
  const renderUserList = (): JSX.Element => {
    if (users.length === 0) {
      return <p>No users found</p>;
    }
    
    return (
      <ul>
        {users.map((user) => (
          <li
            key={user.id}
            onClick={() => handleUserSelect(user.id)}
            className={user.id === selectedId ? 'selected' : ''}
          >
            {user.name}
          </li>
        ))}
      </ul>
    );
  };
  
  return (
    <div className="user-list-container">
      <h2>Users</h2>
      {renderUserList()}
    </div>
  );
};
```

**Common Pitfalls:**
- Trying to use arrow functions as constructors (they can't be used with `new`)
- Using arrow functions for class methods when you need to bind to the instance
- Creating arrow functions inside render methods/loops, causing unnecessary re-creations
- Forgetting that arrow functions don't have their own `arguments` object
- Using arrow functions when you need to access the function name through `this.name`

**Three Common Questions:**

1. **Q: What's the main difference between an arrow function and a regular function?**  
   **A:** The key differences are: 1) Arrow functions don't have their own `this` binding; they inherit `this` from the surrounding scope. 2) Arrow functions can't be used as constructors with `new`. 3) Arrow functions don't have an `arguments` object. 4) Arrow functions don't have their own `super` or `new.target` keywords.

2. **Q: When should I use a regular function instead of an arrow function?**  
   **A:** Use regular functions when: 1) You need the function to have its own `this` context (e.g., in object methods that use `this`), 2) You need to use the function as a constructor with `new`, 3) You need access to the `arguments` object, or 4) You need to use `yield` in a generator function.

3. **Q: How does TypeScript handle `this` typing in arrow functions?**  
   **A:** Since arrow functions capture `this` lexically from their surrounding context, TypeScript uses the `this` type from that outer scope. You don't need to explicitly declare the type of `this` for arrow functions like you sometimes do for regular functions. In cases where you want to access `this` properties in callbacks, arrow functions are safer as TypeScript can properly track the `this` type.

#### 1.7 Function Types and Signatures

**Concise Explanation:**  
Function types let you describe the shape of a function, including its parameter types and return type. They're used to type variables that hold functions, pass functions as arguments, or return functions from other functions. Function signatures establish contracts that implementations must follow.

**Where to Use:**
- When storing functions in variables or properties
- Defining callback parameter types
- Creating higher-order functions
- Implementing dependency injection patterns
- Defining function properties in interfaces

**Code Snippet:**
```typescript
// Basic function type
type MathOperation = (a: number, b: number) => number;

// Function type with parameter names (for readability)
type Formatter = (value: string, options: object) => string;

// Function type variable
const add: MathOperation = (a, b) => a + b;
const subtract: MathOperation = (a, b) => a - b;

// Function type as a parameter
function calculate(a: number, b: number, operation: MathOperation): number {
  return operation(a, b);
}

// Usage
const result = calculate(10, 5, subtract); // 5

// Function type with generic
type Mapper<T, U> = (item: T) => U;

const numberToString: Mapper<number, string> = (num) => num.toString();

// Function type as property in interface
interface APIClient {
  get: <T>(url: string) => Promise<T>;
  post: <T>(url: string, data: any) => Promise<T>;
}

// Call signature syntax (alternative to arrow syntax)
type HttpRequest = {
  (url: string, method: string): Promise<Response>;
  timeout?: number;
};

// Construct signature
interface DateConstructor {
  new (value: number): Date;
  new (value: string): Date;
  new (year: number, month: number, day?: number): Date;
}

// Function that returns a function
function createMultiplier(factor: number): (value: number) => number {
  return (value) => value * factor;
}

const double = createMultiplier(2);
console.log(double(5)); // 10
```

**Real-World Example:**  
A custom hook for form handling in React:

```typescript
// Form hook with function types
import { useState, ChangeEvent, FormEvent } from 'react';

// Function types for handlers
type FieldValidator<T> = (value: T) => string | null;
type SubmitHandler<T> = (values: T) => void | Promise<void>;
type FieldChangeHandler = (e: ChangeEvent<HTMLInputElement>) => void;
type FormSubmitHandler = (e: FormEvent<HTMLFormElement>) => void;

interface UseFormOptions<T> {
  initialValues: T;
  validators?: Partial<Record<keyof T, FieldValidator<any>>>;
  onSubmit: SubmitHandler<T>;
}

interface UseFormResult<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  handleChange: FieldChangeHandler;
  handleSubmit: FormSubmitHandler;
  isValid: boolean;
  reset: () => void;
}

// Custom form hook with strongly typed functions
function useForm<T extends Record<string, any>>(
  options: UseFormOptions<T>
): UseFormResult<T> {
  const { initialValues, validators = {}, onSubmit } = options;
  
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  
  // Field change handler
  const handleChange: FieldChangeHandler = (e) => {
    const { name, value, type, checked } = e.target;
    const fieldValue = type === 'checkbox' ? checked : value;
    
    // Update the field
    setValues(prevValues => ({
      ...prevValues,
      [name]: fieldValue
    }));
    
    // Validate if we have a validator
    if (validators[name as keyof T]) {
      const validator = validators[name as keyof T] as FieldValidator<any>;
      const error = validator(fieldValue);
      
      setErrors(prevErrors => ({
        ...prevErrors,
        [name]: error
      }));
    }
  };
  
  // Form submission handler
  const handleSubmit: FormSubmitHandler = (e) => {
    e.preventDefault();
    
    // Validate all fields
    const formErrors: Partial<Record<keyof T, string>> = {};
    let hasErrors = false;
    
    // Run all validators
    Object.keys(validators).forEach((key) => {
      const fieldKey = key as keyof T;
      const validator = validators[fieldKey] as FieldValidator<any>;
      const error = validator(values[fieldKey]);
      
      if (error) {
        formErrors[fieldKey] = error;
        hasErrors = true;
      }
    });
    
    setErrors(formErrors);
    
    // Submit if valid
    if (!hasErrors) {
      onSubmit(values);
    }
  };
  
  // Check if form is valid
  const isValid = Object.values(errors).every(error => error === null || error === undefined);
  
  // Reset form
  const reset = () => {
    setValues(initialValues);
    setErrors({});
  };
  
  return {
    values,
    errors,
    handleChange,
    handleSubmit,
    isValid,
    reset
  };
}

// Usage
interface LoginForm {
  email: string;
  password: string;
  rememberMe: boolean;
}

function LoginComponent() {
  const { values, errors, handleChange, handleSubmit, isValid } = useForm<LoginForm>({
    initialValues: {
      email: '',
      password: '',
      rememberMe: false
    },
    validators: {
      email: (value) => !value.includes('@') ? 'Invalid email' : null,
      password: (value) => value.length < 8 ? 'Password must be at least 8 characters' : null
    },
    onSubmit: async (formValues) => {
      // Handle submission
      console.log('Submitting', formValues);
    }
  });
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
}
```

**Common Pitfalls:**
- Forgetting to make function types compatible with their implementations
- Not accounting for optional parameters in function types
- Mixing arrow function syntax and call signature syntax inconsistently
- Overly rigid function types that make reuse difficult
- Using `Function` type instead of specific function signatures, losing type safety

**Three Common Questions:**

1. **Q: What's the difference between `interface` with call signature and `type` with arrow syntax for function types?**  
   **A:** Both can define function types. Arrow syntax (`type F = (a: string) => number`) is more concise. Call signatures (`interface F { (a: string): number }`) allow additional properties on the function object and can be extended or implemented. Choose based on your needs: arrow syntax for simple functions, call signatures for functions with properties.

2. **Q: How can I represent a function with both properties and callable behavior?**  
   **A:** Use an interface with a call signature plus additional properties:
   ```typescript
   interface FetchFunction {
     (url: string): Promise<Response>;
     timeout: number;
     retries: number;
   }
   ```
   This represents a function that can be called with a URL and also has timeout and retries properties.

3. **Q: How do TypeScript function types handle compatibility?**  
   **A:** TypeScript uses structural typing for functions. Function types are compatible if the target function has the same or fewer parameters (contravariance) and the return type is assignable (covariance). Parameter types must be assignable in reverse direction. For example, a function that accepts a more general type (like `Animal`) can be assigned to a function type expecting a more specific type (like `Dog`).

### 2. Modules

#### 2.1 ES Modules vs CommonJS Modules

**Concise Explanation:**  
TypeScript supports both ES Modules (ESM) and CommonJS (CJS) module systems. ES modules use `import`/`export` syntax and are the modern standard for JavaScript. CommonJS uses `require()`/`module.exports` and is traditionally used in Node.js. TypeScript allows you to write code using ESM syntax and compile to either format based on your target environment.

**Where to Use:**
- ES Modules: Modern browsers, newer Node.js versions, bundlers like webpack
- CommonJS: Older Node.js applications, libraries that need maximum compatibility
- Both: When building libraries that need to support multiple environments

**Code Snippet:**
```typescript
// ES Modules syntax (in TypeScript files)
// Exporting
export const PI = 3.14159;
export function square(x: number): number {
  return x * x;
}
export default class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

// Importing
import Calculator, { PI, square } from './math';
import * as utils from './utils';

// CommonJS syntax (Node.js style)
// Exporting
exports.PI = 3.14159;
exports.square = function(x: number): number {
  return x * x;
};
module.exports = {
  PI: 3.14159,
  square(x: number): number {
    return x * x;
  }
};

// Importing
const Calculator = require('./math');
const { PI, square } = require('./math');

// TypeScript configuration for different module systems
// tsconfig.json for ES Modules
{
  "compilerOptions": {
    "module": "esnext", // or "es2020", "es2022", etc.
    "target": "es2020"
  }
}

// tsconfig.json for CommonJS
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es2020"
  }
}
```

**Real-World Example:**  
Building a library that supports both module systems:

```typescript
// math-utils.ts
/**
 * Mathematical utilities
 * @packageDocumentation
 */

/**
 * Calculates the Greatest Common Divisor of two numbers
 */
export function gcd(a: number, b: number): number {
  while (b !== 0) {
    const temp = b;
    b = a % b;
    a = temp;
  }
  return a;
}

/**
 * Calculates the Least Common Multiple of two numbers
 */
export function lcm(a: number, b: number): number {
  return (a * b) / gcd(a, b);
}

/**
 * Checks if a number is prime
 */
export function isPrime(num: number): boolean {
  if (num <= 1) return false;
  if (num <= 3) return true;
  
  if (num % 2 === 0 || num % 3 === 0) return false;
  
  for (let i = 5; i * i <= num; i += 6) {
    if (num % i === 0 || num % (i + 2) === 0) {
      return false;
    }
  }
  
  return true;
}

// Default export
export default {
  gcd,
  lcm,
  isPrime
};

// Package configuration in package.json to support both module systems
{
  "name": "math-utils",
  "version": "1.0.0",
  "main": "dist/cjs/index.js",  // CommonJS entry point
  "module": "dist/esm/index.js", // ES Module entry point
  "types": "dist/types/index.d.ts",
  "exports": {
    ".": {
      "require": "./dist/cjs/index.js",
      "import": "./dist/esm/index.js",
      "types": "./dist/types/index.d.ts"
    }
  },
  "scripts": {
    "build": "npm run build:cjs && npm run build:esm && npm run build:types",
    "build:cjs": "tsc --module commonjs --outDir dist/cjs",
    "build:esm": "tsc --module esnext --outDir dist/esm",
    "build:types": "tsc --declaration --declarationDir dist/types --emitDeclarationOnly"
  }
}
```

**Common Pitfalls:**
- Mixing ES Module and CommonJS syntax in the same file
- Forgetting that `import` statements are hoisted but `require()` is not
- Not configuring the correct `module` option in tsconfig.json for your target environment
- Default exports behaving differently between ESM and CommonJS
- Not accounting for top-level await support differences between module systems

**Three Common Questions:**

1. **Q: How do I use a CommonJS module in a project using ES modules?**  
   **A:** TypeScript provides interoperability via the `esModuleInterop` flag in tsconfig.json. With this enabled, you can import CommonJS modules as if they were ES modules: `import fs from 'fs'` instead of `import * as fs from 'fs'`. This helps bridge the gap between the two module systems.

2. **Q: Which module system should I use for a new TypeScript project?**  
   **A:** For modern applications, prefer ES Modules (set `"module": "esnext"` or similar in tsconfig.json). They're the standard in modern JavaScript, support tree-shaking better, and offer features like top-level await. Use CommonJS if you need to support older Node.js versions or have significant dependencies on CommonJS-only packages.

3. **Q: How can I identify which module system a package uses?**  
   **A:** Check the package.json file. Modern packages often include both:
   - `"main"`: Points to CommonJS version
   - `"module"` or `"jsnext:main"`: Points to ES Module version
   - `"exports"`: Provides conditional exports based on environment
   Newer packages may have dropped CommonJS support and only provide ES modules.

#### 2.2 Import and Export Syntax

**Concise Explanation:**  
TypeScript's import and export syntax controls what parts of your code are accessible from other modules. Named exports make specific declarations available, while default exports provide a single main export. TypeScript verifies that imports match exports at compile time, ensuring type safety across module boundaries.

**Where to Use:**
- Named exports: For libraries exposing multiple functions/classes, utility modules
- Default exports: For modules with a primary class/function, React components
- Type exports: For sharing types and interfaces between modules
- Re-exports: For creating aggregated public APIs

**Code Snippet:**
```typescript
// Named exports
export const PI = 3.14159;
export function square(x: number): number {
  return x * x;
}
export class Circle {
  constructor(public radius: number) {}
  
  area(): number {
    return PI * square(this.radius);
  }
}

// Default export
export default class Rectangle {
  constructor(public width: number, public height: number) {}
  
  area(): number {
    return this.width * this.height;
  }
}

// Exporting types and interfaces
export type Shape = Circle | Rectangle;
export interface Drawable {
  draw(): void;
}

// Exporting after declaration
function cube(x: number): number {
  return x * x * x;
}
const E = 2.71828;
class Triangle {}

export { cube, E, Triangle };

// Renaming exports
export { cube as calculateCube, Triangle as ThreeSidedPolygon };

// Importing named exports
import { PI, square, Circle } from './shapes';

// Importing default export
import Rectangle from './shapes';

// Importing both default and named exports
import Rectangle, { PI, square } from './shapes';

// Importing with aliases
import { Circle as RoundShape, square as calculateSquare } from './shapes';

// Importing everything as a namespace
import * as Shapes from './shapes';
console.log(Shapes.PI);
const circle = new Shapes.Circle(5);

// Importing types
import { Shape, Drawable } from './shapes';

// Dynamic import (for code splitting)
async function loadShapes() {
  const shapes = await import('./shapes');
  return new shapes.Circle(10);
}
```

**Real-World Example:**  
A modular application structure for a dashboard:

```typescript
// src/models/user.ts
export interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
}

export type UserId = string;

export interface UserPreferences {
  theme: 'light' | 'dark';
  notifications: boolean;
  language: string;
}

// src/services/api.ts
import { User, UserId } from '../models/user';

export class ApiError extends Error {
  constructor(public statusCode: number, message: string) {
    super(message);
  }
}

export async function fetchUser(id: UserId): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  
  if (!response.ok) {
    throw new ApiError(response.status, 'Failed to fetch user');
  }
  
  return response.json();
}

// src/services/index.ts - Re-export pattern
export * from './api';
export * from './auth';
export * from './storage';

// src/components/UserProfile.tsx
import React, { useState, useEffect } from 'react';
import { User, UserPreferences } from '../models/user';
import { fetchUser, ApiError } from '../services';

interface UserProfileProps {
  userId: string;
}

// Default export for a React component
export default function UserProfile({ userId }: UserProfileProps) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    async function loadUser() {
      try {
        setLoading(true);
        const userData = await fetchUser(userId);
        setUser(userData);
        setError(null);
      } catch (err) {
        setError(err instanceof ApiError 
          ? `Error ${err.statusCode}: ${err.message}` 
          : 'An unknown error occurred'
        );
      } finally {
        setLoading(false);
      }
    }
    
    loadUser();
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div className="error">{error}</div>;
  if (!user) return <div>No user found</div>;
  
  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <div className="role-badge">{user.role}</div>
    </div>
  );
}

// src/index.ts
export { default as UserProfile } from './components/UserProfile';
export * from './models/user';
export * from './services';
```

**Common Pitfalls:**
- Circular dependencies between modules causing runtime issues
- Using default exports without consistent naming conventions
- Too many exports from a single file, making dependencies hard to track
- Forgetting to export types that are used in public API signatures
- Overusing barrel files (index.ts re-exports) which can impact tree-shaking

**Three Common Questions:**

1. **Q: When should I use named exports versus default exports?**  
   **A:** Use named exports when a module exports multiple items of equal importance, like utility functions. This makes imports explicit and helps with tree-shaking. Use default exports for a module's main class/function/component, especially for React components where each file typically contains one primary export. Many teams choose one style for consistency.

2. **Q: How can I import a type and a value with the same name?**  
   **A:** From TypeScript 4.5 onwards, you can use import type specifiers to avoid name conflicts:
   ```typescript
   import { User } from './models'; // Imports the value (class/function/variable)
   import type { User } from './models'; // Imports only the type
   // Or use a namespace import for one of them
   import * as Models from './models';
   import type { User } from './models';
   const user: User = new Models.User();
   ```

3. **Q: What happens to imports and exports when TypeScript compiles to JavaScript?**  
   **A:** Type-only imports and exports are completely removed from the JavaScript output. Value imports/exports are transformed according to the target module system specified in tsconfig.json (e.g., `import` becomes `require()` when targeting CommonJS). The `isolatedModules` and `importsNotUsedAsValues` options further control how TypeScript handles imports during compilation.

#### 2.3 Default and Named Exports

**Concise Explanation:**  
Default exports provide a single, main export from a module, while named exports allow multiple declarations to be exported individually. Understanding when to use each helps create cleaner module interfaces and improves code organization.

**Where to Use:**
- Default exports: Single-responsibility modules, main classes/components
- Named exports: Utility functions, libraries with multiple related exports
- Mixed approach: Component library with helpers, configuration modules

**Code Snippet:**
```typescript
// File: math.ts
// Named exports
export const PI = 3.14159;
export function square(x: number): number {
  return x * x;
}

// Default export
export default class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
  
  subtract(a: number, b: number): number {
    return a - b;
  }
}

// File: user.ts
// Interface with named export
export interface User {
  id: string;
  name: string;
}

// Default export function
export default function createUser(name: string): User {
  return {
    id: Date.now().toString(),
    name
  };
}

// File: app.ts
// Importing default export
import Calculator from './math';
const calc = new Calculator();
console.log(calc.add(2, 3)); // 5

// Importing named exports
import { PI, square } from './math';
console.log(PI); // 3.14159
console.log(square(4)); // 16

// Importing default and named exports together
import createUser, { User } from './user';
const user: User = createUser('Alice');

// Renaming default import
import MathCalculator from './math';
const calculator = new MathCalculator();

// Namespace import (includes default as .default)
import * as MathModule from './math';
const calc2 = new MathModule.default(); // Default export
console.log(MathModule.PI); // Named export
```

**Real-World Example:**  
Organizing a React application with Redux:

```typescript
// src/components/Button.tsx
import React from 'react';

interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary' | 'danger';
}

// Default export for the main component
export default function Button({ 
  label, 
  onClick, 
  disabled = false, 
  variant = 'primary' 
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {label}
    </button>
  );
}

// Named exports for related utilities
export function IconButton({ icon, ...props }: ButtonProps & { icon: string }) {
  return (
    <Button {...props}>
      <i className={`icon ${icon}`}></i>
      {props.label}
    </Button>
  );
}

export const ButtonSizes = {
  SMALL: 'sm',
  MEDIUM: 'md',
  LARGE: 'lg'
} as const;

// src/store/userSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface UserState {
  id: string | null;
  name: string | null;
  isAuthenticated: boolean;
  preferences: {
    theme: 'light' | 'dark';
  };
}

const initialState: UserState = {
  id: null,
  name: null,
  isAuthenticated: false,
  preferences: {
    theme: 'light'
  }
};

// Named export for the slice
export const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    login: (state, action: PayloadAction<{ id: string; name: string }>) => {
      state.id = action.payload.id;
      state.name = action.payload.name;
      state.isAuthenticated = true;
    },
    logout: (state) => {
      state.id = null;
      state.name = null;
      state.isAuthenticated = false;
    },
    setTheme: (state, action: PayloadAction<'light' | 'dark'>) => {
      state.preferences.theme = action.payload;
    }
  }
});

// Named exports for actions
export const { login, logout, setTheme } = userSlice.actions;

// Named export for selectors
export const selectUser = (state: { user: UserState }) => state.user;
export const selectTheme = (state: { user: UserState }) => state.user.preferences.theme;

// Default export for the reducer
export default userSlice.reducer;

// src/App.tsx - Demonstrating import usage
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import Button, { IconButton, ButtonSizes } from './components/Button';
import userReducer, { 
  login, 
  logout, 
  setTheme, 
  selectUser, 
  selectTheme 
} from './store/userSlice';

export default function App() {
  const dispatch = useDispatch();
  const user = useSelector(selectUser);
  const theme = useSelector(selectTheme);
  
  // Component logic...
  
  return (
    <div className={`app-container theme-${theme}`}>
      <h1>Welcome {user.name || 'Guest'}</h1>
      
      {!user.isAuthenticated ? (
        <Button 
          label="Log In" 
          onClick={() => dispatch(login({ id: '123', name: 'Alice' }))} 
          variant="primary" 
        />
      ) : (
        <IconButton 
          label="Log Out" 
          icon="logout" 
          onClick={() => dispatch(logout())} 
          variant="secondary" 
        />
      )}
    </div>
  );
}
```

**Common Pitfalls:**
- Inconsistent naming of default imports across files
- Renaming default imports, making it hard to trace back to the source module
- Mixing default and named exports without clear conventions
- Default exporting anonymous functions, making debugging harder
- Creating many small files each with a default export, increasing import complexity

**Three Common Questions:**

1. **Q: If a module has both default and named exports, how do I import them together?**  
   **A:** You can import both in a single statement: `import DefaultExport, { namedExport1, namedExport2 } from './module';`. Alternatively, use a namespace import: `import * as Module from './module';` and access as `Module.default` and `Module.namedExport`.

2. **Q: Can a module have multiple default exports?**  
   **A:** No, each module can have at most one default export. If you need multiple exports, use named exports. The default export is meant to represent the primary or most commonly used export from a module.

3. **Q: Should I prefer named exports or default exports for my code?**  
   **A:** Each has advantages. Named exports are self-documenting (import names must match export names), work better with autocompletion, and support better tree-shaking. Default exports are convenient for single-export modules like React components but can lead to inconsistent naming. Many teams adopt conventions like "use named exports for utilities and default exports for components."

#### 2.4 Module Augmentation

**Concise Explanation:**  
Module augmentation allows you to extend existing modules with new properties and methods without modifying their original source code. This is useful for adding functionality to third-party libraries or extending the TypeScript built-in modules.

**Where to Use:**
- Adding properties to third-party modules
- Extending built-in TypeScript declarations
- Adding custom methods to global objects
- Plugin systems that enhance existing modules

**Code Snippet:**
```typescript
// Original module definition in third-party library
// node_modules/some-library/index.d.ts
export interface User {
  id: number;
  name: string;
}

export function createUser(name: string): User;

// Augmenting the module with new functionality
// src/types/some-library.d.ts
import 'some-library';

// Declare module augmentation
declare module 'some-library' {
  // Add new interface
  export interface UserRole {
    role: 'admin' | 'user' | 'guest';
  }
  
  // Add properties to existing interface
  export interface User {
    email: string;
    role: UserRole['role'];
  }
  
  // Add new function
  export function getUserRole(user: User): UserRole['role'];
}

// Usage in your code
import { createUser, getUserRole } from 'some-library';

const user = createUser('Alice');
user.email = 'alice@example.com';
user.role = 'admin';
const role = getUserRole(user);

// Augmenting built-in modules
declare module 'express-serve-static-core' {
  interface Request {
    user?: {
      id: string;
      roles: string[];
    };
  }
}

// Global augmentation
declare global {
  interface String {
    toTitleCase(): string;
  }
  
  interface Window {
    analytics: {
      track(event: string, properties?: Record<string, any>): void;
    };
  }
}

// Implementation for the augmented String method
String.prototype.toTitleCase = function(): string {
  return this.replace(/\w\S*/g, (txt) => {
    return txt.charAt(0).toUpperCase() + txt.substring(1).toLowerCase();
  });
};
```

**Real-World Example:**  
Extending Express.js request type for authentication:

```typescript
// src/types/express.d.ts
import 'express';

declare module 'express-serve-static-core' {
  // Extend Express Request object with custom properties
  interface Request {
    user?: {
      id: string;
      email: string;
      roles: string[];
      permissions: string[];
      lastLogin: Date;
    };
    
    isAuthenticated(): boolean;
    hasRole(role: string): boolean;
  }
}

// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

interface JwtPayload {
  id: string;
  email: string;
  roles: string[];
  permissions: string[];
}

export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ message: 'Authentication required' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload;
    
    // Attach user to request object (using our augmented type)
    req.user = {
      id: decoded.id,
      email: decoded.email,
      roles: decoded.roles,
      permissions: decoded.permissions,
      lastLogin: new Date()
    };
    
    // Implement the methods we declared
    req.isAuthenticated = function() {
      return !!req.user;
    };
    
    req.hasRole = function(role: string) {
      return req.user?.roles.includes(role) || false;
    };
    
    next();
  } catch (error) {
    return res.status(401).json({ message: 'Invalid token' });
  }
}

// src/routes/users.ts
import { Router, Request, Response } from 'express';
import { authMiddleware } from '../middleware/auth';

const router = Router();

router.get('/profile', authMiddleware, (req: Request, res: Response) => {
  // We can use our augmented properties and methods
  if (!req.isAuthenticated()) {
    return res.status(401).send('Unauthorized');
  }
  
  if (!req.hasRole('user')) {
    return res.status(403).send('Forbidden');
  }
  
  // Access the user data
  const { id, email, lastLogin } = req.user!;
  
  res.json({
    id,
    email,
    lastLogin,
    roles: req.user!.roles
  });
});

export default router;
```

**Common Pitfalls:**
- Augmenting modules without understanding their internal structure
- Name conflicts between original and augmented declarations
- Not updating augmentations when the original module changes
- Overusing module augmentation, making code hard to maintain
- Not documenting augmentations, leading to confusion for other developers

**Three Common Questions:**

1. **Q: How is module augmentation different from simply extending a type?**  
   **A:** Module augmentation actually changes how TypeScript understands an existing module, so the augmentation applies anywhere the module is imported. Extending a type just creates a new type based on an existing one. Module augmentation affects the type system globally for that module, while type extension is limited to where the extended type is used.

2. **Q: Can I augment a module that doesn't have a declaration file?**  
   **A:** You must have a type declaration for the module first before you can augment it. If the module doesn't have declaration files, you can create a basic declaration file (`.d.ts`) for it first, and then augment it. For example: `declare module 'module-without-types' { export function originalFunc(): void; }`, and then use module augmentation to add more types.

3. **Q: Where should I place module augmentation files in my project structure?**  
   **A:** It's common to place module augmentation files in a `types` or `@types` directory within your project. You should also make sure these files are included in your `tsconfig.json` either directly or through the `include` patterns. Each augmentation should be in a dedicated file with a clear naming convention, such as `module-name.d.ts`.

#### 2.5 Dynamic Imports

**Concise Explanation:**  
Dynamic imports allow you to load modules on demand rather than up front, enabling code splitting, lazy loading, and conditional loading based on runtime conditions. This improves initial load time and reduces memory usage by only loading what's needed when it's needed.

**Where to Use:**
- Lazy-loading routes in single page applications
- Large feature modules that aren't always needed
- Loading modules based on user interaction or conditions
- Optimizing browser bundle size
- Server-side applications with conditional features

**Code Snippet:**
```typescript
// Basic dynamic import
async function loadModule() {
  // Module is loaded only when this function is called
  const module = await import('./heavy-module');
  return module.default;
}

// Conditional dynamic imports
async function loadFeature(featureName: string) {
  switch (featureName) {
    case 'charts':
      const { ChartComponent } = await import('./features/charts');
      return ChartComponent;
    case 'tables':
      const { TableComponent } = await import('./features/tables');
      return TableComponent;
    default:
      throw new Error(`Unknown feature: ${featureName}`);
  }
}

// Dynamic import with destructuring
async function loadUtils() {
  const { formatDate, formatCurrency } = await import('./utils/formatters');
  return {
    formatDate,
    formatCurrency
  };
}

// Type-safe dynamic imports
type Module = typeof import('./module-with-types');

async function loadTypedModule(): Promise<Module> {
  return import('./module-with-types');
}

// Error handling with dynamic imports
async function safeLoadModule() {
  try {
    return await import('./module-that-might-fail');
  } catch (error) {
    console.error('Failed to load module:', error);
    return null;
  }
}

// Dynamic imports with Promise.all
async function loadMultipleModules() {
  const [moduleA, moduleB] = await Promise.all([
    import('./module-a'),
    import('./module-b')
  ]);
  
  return {
    featureA: moduleA.default,
    featureB: moduleB.default
  };
}
```

**Real-World Example:**  
Lazy-loaded routes in a React application:

```typescript
// src/App.tsx
import React, { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// Static import for components always needed
import Header from './components/Header';
import Footer from './components/Footer';
import Loading from './components/Loading';

// Lazy loaded components using dynamic imports
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const UserProfile = lazy(() => import('./pages/UserProfile'));
const Settings = lazy(() => 
  // We can even add artificial delay for testing loading states
  new Promise<{ default: React.ComponentType }>(resolve => {
    setTimeout(() => {
      import('./pages/Settings').then(resolve);
    }, 1000);
  })
);

// Import specific exports from a module
const ProductCatalog = lazy(async () => {
  const module = await import('./pages/Products');
  // Return an object with a default property
  return { default: module.ProductCatalog };
});

interface FeatureToggle {
  enableAnalytics: boolean;
  enableExperimentalFeatures: boolean;
}

// Dynamic feature loading based on configuration
const App: React.FC = () => {
  const [featureToggles, setFeatureToggles] = React.useState<FeatureToggle | null>(null);
  const [analyticsLoaded, setAnalyticsLoaded] = React.useState(false);

  // Load feature toggles from API
  React.useEffect(() => {
    fetch('/api/features')
      .then(res => res.json())
      .then(data => setFeatureToggles(data));
  }, []);

  // Dynamically load analytics if enabled
  React.useEffect(() => {
    if (featureToggles?.enableAnalytics && !analyticsLoaded) {
      import('./services/analytics')
        .then(module => {
          module.initialize();
          setAnalyticsLoaded(true);
        })
        .catch(error => {
          console.error('Failed to load analytics:', error);
        });
    }
  }, [featureToggles, analyticsLoaded]);

  // Conditionally render experimental features
  const ExperimentalFeatures = React.useMemo(() => 
    featureToggles?.enableExperimentalFeatures
      ? lazy(() => import('./features/Experimental'))
      : () => null,
    [featureToggles?.enableExperimentalFeatures]
  );

  return (
    <BrowserRouter>
      <Header />
      <nav>
        <Link to="/">Home</Link>
        <Link to="/dashboard">Dashboard</Link>
        <Link to="/profile">Profile</Link>
        <Link to="/settings">Settings</Link>
        <Link to="/products">Products</Link>
        {featureToggles?.enableExperimentalFeatures && (
          <Link to="/experimental">Experimental</Link>
        )}
      </nav>
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<UserProfile />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/products" element={<ProductCatalog />} />
          {featureToggles?.enableExperimentalFeatures && (
            <Route path="/experimental" element={<ExperimentalFeatures />} />
          )}
        </Routes>
      </Suspense>
      <Footer />
    </BrowserRouter>
  );
};

export default App;
```

**Common Pitfalls:**
- Not handling errors that could occur during dynamic imports
- Forgetting that dynamic imports return a Promise, requiring await/then
- Excessive use of dynamic imports for small modules, increasing HTTP requests
- Not considering the impact on UX when modules take time to load
- Forgetting to implement loading states for dynamically imported components

**Three Common Questions:**

1. **Q: How does TypeScript handle type checking for dynamic imports?**  
   **A:** TypeScript provides type safety for dynamic imports by inferring the type from the imported module. You can also explicitly type the dynamic import using `typeof import('./module')`. The result of a dynamic import is always a Promise that resolves to the module's exports, so proper async handling is required.

2. **Q: Can dynamic imports work with tree shaking?**  
   **A:** Yes, dynamic imports actually enhance tree shaking by allowing the bundler to split your code into separate chunks. When a module is dynamically imported, the bundler creates a separate chunk containing only the code needed for that module and its dependencies. This can significantly reduce the initial bundle size.

3. **Q: How do dynamic imports differ between Node.js and browsers?**  
   **A:** In browsers, dynamic imports create separate chunks that are loaded on demand over HTTP. In Node.js, dynamic imports load modules from the filesystem when needed. Node.js dynamic imports can use variables in the import path (like `import(`./${moduleName}`)`) more freely, while in browsers this can complicate bundling. Also, Node.js has different caching behavior for dynamically imported modules compared to browsers.

#### 2.6 Namespaces (legacy)

**Concise Explanation:**  
Namespaces are TypeScript's original way of organizing code, allowing you to group related functionality under a single name to avoid naming conflicts. While considered legacy compared to ES modules, they're still useful for organizing code in specific scenarios, especially in global scripts and declaration files.

**Where to Use:**
- Legacy code that already uses namespaces
- Declaration files (.d.ts) for global libraries
- Browser scripts without module bundlers
- Organizing related types in declaration files
- Avoiding global namespace pollution

**Code Snippet:**
```typescript
// Basic namespace
namespace Geometry {
  export interface Point {
    x: number;
    y: number;
  }
  
  export class Circle {
    constructor(public center: Point, public radius: number) {}
    
    area(): number {
      return Math.PI * this.radius ** 2;
    }
  }
  
  // Non-exported items are private to the namespace
  function distance(a: Point, b: Point): number {
    return Math.sqrt((b.x - a.x) ** 2 + (b.y - a.y) ** 2);
  }
  
  export function isPointInCircle(point: Point, circle: Circle): boolean {
    return distance(point, circle.center) <= circle.radius;
  }
}

// Using the namespace
const point: Geometry.Point = { x: 1, y: 2 };
const circle = new Geometry.Circle({ x: 0, y: 0 }, 5);
const inCircle = Geometry.isPointInCircle(point, circle);

// Nested namespaces
namespace App {
  export namespace Models {
    export interface User {
      id: number;
      name: string;
    }
  }
  
  export namespace Services {
    export class UserService {
      getUser(id: number): Models.User {
        // Implementation omitted
        return { id, name: 'User ' + id };
      }
    }
  }
  
  export namespace Utils {
    export function formatUser(user: Models.User): string {
      return `${user.name} (ID: ${user.id})`;
    }
  }
}

// Using nested namespaces
const userService = new App.Services.UserService();
const user = userService.getUser(1);
const formattedUser = App.Utils.formatUser(user);

// Namespace spanning multiple files
// file1.ts
namespace Validation {
  export interface StringValidator {
    isValid(s: string): boolean;
  }
}

// file2.ts
/// <reference path="file1.ts" />
namespace Validation {
  export class EmailValidator implements StringValidator {
    isValid(s: string): boolean {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return emailRegex.test(s);
    }
  }
}
```

**Real-World Example:**  
Creating a library with global utilities:

```typescript
// declarations.d.ts
// Global namespace for utility library
declare namespace Utils {
  interface StringUtils {
    capitalize(str: string): string;
    truncate(str: string, length: number): string;
    slugify(str: string): string;
  }
  
  interface NumberUtils {
    format(num: number, decimals?: number): string;
    roundTo(num: number, places: number): number;
  }
  
  interface DateUtils {
    format(date: Date, format: string): string;
    addDays(date: Date, days: number): Date;
    diffInDays(a: Date, b: Date): number;
  }
}

// Extend global objects
interface Window {
  Utils: {
    string: Utils.StringUtils;
    number: Utils.NumberUtils;
    date: Utils.DateUtils;
  };
}

// Implementation in utils.ts
namespace Utils {
  // StringUtils implementation
  export class StringUtilsImpl implements StringUtils {
    capitalize(str: string): string {
      if (!str) return str;
      return str.charAt(0).toUpperCase() + str.slice(1);
    }
    
    truncate(str: string, length: number): string {
      if (!str || str.length <= length) return str;
      return str.slice(0, length) + '...';
    }
    
    slugify(str: string): string {
      return str
        .toLowerCase()
        .replace(/[\s\W-]+/g, '-')
        .replace(/^-|-$/g, '');
    }
  }
  
  // NumberUtils implementation
  export class NumberUtilsImpl implements NumberUtils {
    format(num: number, decimals: number = 2): string {
      return num.toFixed(decimals).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    }
    
    roundTo(num: number, places: number): number {
      const factor = Math.pow(10, places);
      return Math.round(num * factor) / factor;
    }
  }
  
  // DateUtils implementation
  export class DateUtilsImpl implements DateUtils {
    format(date: Date, format: string): string {
      // Simple format implementation
      return format
        .replace('YYYY', date.getFullYear().toString())
        .replace('MM', (date.getMonth() + 1).toString().padStart(2, '0'))
        .replace('DD', date.getDate().toString().padStart(2, '0'));
    }
    
    addDays(date: Date, days: number): Date {
      const result = new Date(date);
      result.setDate(result.getDate() + days);
      return result;
    }
    
    diffInDays(a: Date, b: Date): number {
      const msPerDay = 1000 * 60 * 60 * 24;
      const difference = Math.abs(a.getTime() - b.getTime());
      return Math.round(difference / msPerDay);
    }
  }
}

// Attach to window for global access
window.Utils = {
  string: new Utils.StringUtilsImpl(),
  number: new Utils.NumberUtilsImpl(),
  date: new Utils.DateUtilsImpl()
};

// Usage in application
document.getElementById('format-btn')?.addEventListener('click', () => {
  const price = 1234.56;
  const formattedPrice = window.Utils.number.format(price);
  
  const dateStr = window.Utils.date.format(new Date(), 'YYYY-MM-DD');
  
  const title = "this is a test title";
  const capitalizedTitle = window.Utils.string.capitalize(title);
  
  console.log(formattedPrice, dateStr, capitalizedTitle);
});
```

**Common Pitfalls:**
- Using namespaces instead of ES modules in modern applications
- Creating deeply nested namespaces that are hard to import and use
- Mixing namespaces and modules, leading to confusion
- Not understanding how namespaces are merged across files
- Forgetting that namespaces have their own visibility rules separate from ES modules

**Three Common Questions:**

1. **Q: When should I use namespaces instead of ES modules?**  
   **A:** In modern TypeScript code, ES modules are generally preferred. However, namespaces still make sense for: 1) Declaration files for libraries that expose global objects, 2) Scripts that run directly in browsers without bundling, 3) Organizing complex types in declaration files, or 4) Maintaining legacy TypeScript code that already uses namespaces.

2. **Q: How do namespaces work with module bundlers like webpack?**  
   **A:** Namespaces are a TypeScript-only feature that gets compiled away. When using a module bundler, namespaces are typically converted to IIFE (Immediately Invoked Function Expressions) that create objects with properties corresponding to the exported members. However, this can interfere with tree-shaking and code splitting, which is why ES modules are preferred for bundled applications.

3. **Q: What's the difference between `namespace` and `module` keywords in TypeScript?**  
   **A:** The `module` keyword is an outdated synonym for `namespace` from early TypeScript versions. They function identically, but `namespace` is the preferred keyword in modern TypeScript. The only time you might see `module` used is in ambient declarations for external modules, like `declare module 'some-module'`.

## Next Actions: Exercises and Projects

### Micro-Project 1: Type-Safe Event System
Build a flexible event emitter system:
1. Define function types for different event handlers
2. Create an EventEmitter class that uses generics for type-safe events
3. Implement on, off, and emit methods with proper typing
4. Add support for once() listeners that auto-remove after firing
5. Use type intersections or unions for handling different event data types

### Micro-Project 2: Module Organization Challenge
Convert a disorganized codebase:
1. Take a flat file structure and refactor it into proper modules
2. Implement both named and default exports where appropriate
3. Create barrel files (index.ts) that re-export from multiple modules
4. Add proper type exports for public API surfaces
5. Implement dynamic imports for lazy loading large feature modules

### Micro-Project 3: Function Utilities Library
Create a utility library for function operations:
1. Implement type-safe versions of common functional programming helpers (compose, curry, memoize)
2. Add proper typing for default parameters, rest parameters, and overloads
3. Create helper functions for working with async functions
4. Build decorators for logging, timing, and error handling
5. Add a test suite that validates type safety and functionality

## Success Criteria

You've mastered Module 3 concepts when you can:
1. Choose the appropriate function definition style based on the use case
2. Create well-typed functions with correct parameter and return types
3. Implement function overloads to provide a clear API for complex functions
4. Properly organize code into modules with appropriate imports and exports
5. Understand when to use named vs. default exports
6. Implement dynamic imports for code splitting
7. Augment existing modules to add functionality without modifying source code
8. Recognize when the legacy namespace approach might still be appropriate

## Troubleshooting Guide

### Function Issues
- **Problem**: Function parameter type errors
  **Solution**: Check if you're passing the correct types; consider using union types for flexibility or generics for related types

- **Problem**: "This" context lost in callbacks
  **Solution**: Use arrow functions to preserve lexical this, or explicitly bind the function

- **Problem**: Functions with many parameters becoming unwieldy
  **Solution**: Convert to using a configuration object pattern with an interface

### Module Issues
- **Problem**: Circular dependencies between modules
  **Solution**: Refactor to extract shared types/interfaces into a separate module, or use interface merging

- **Problem**: TypeScript not recognizing imports
  **Solution**: Check module resolution settings in tsconfig.json; verify file extensions and paths

- **Problem**: Difficulty with dynamic imports in bundled code
  **Solution**: Ensure your bundler (webpack, etc.) is configured for code splitting; use named exports inside dynamically imported modules
