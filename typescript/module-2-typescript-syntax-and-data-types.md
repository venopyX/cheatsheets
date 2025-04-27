# Module 2: TypeScript Basic Syntax and Data Types

## Core Problem
JavaScript's dynamic typing can lead to unexpected runtime errors and make code harder to maintain at scale. TypeScript solves this by providing static typing that helps catch errors during development while maintaining JavaScript's flexibility.

## Key Assumptions
- You understand basic programming concepts
- You have completed Module 1 (Introduction to TypeScript)
- You're familiar with basic JavaScript syntax

## Essential Concepts

### 1. Variables and Constants

#### 1.1 Variable Declarations (var, let, const)

**Concise Explanation:**  
TypeScript offers three ways to declare variables: `var` (function-scoped, hoisted), `let` (block-scoped), and `const` (block-scoped, cannot be reassigned). Always prefer `const` when the value won't change, and `let` otherwise. Avoid `var` in modern TypeScript.

**Where to Use:**
- `const`: For values that shouldn't change after initialization
- `let`: For variables whose values need to change
- `var`: Legacy code only (avoid in new projects)

**Code Snippet:**
```typescript
// const - cannot be reassigned
const PI = 3.14159;
// PI = 3; // Error: Cannot assign to 'PI' because it is a constant

// let - block-scoped, can be reassigned
let count = 0;
count = count + 1; // Valid

// Block scoping demonstration
if (true) {
  let blockScoped = 'only visible in this block';
  const alsoBlockScoped = 'also only in this block';
  var notBlockScoped = 'visible outside block'; // Avoid
}
// console.log(blockScoped); // Error: Cannot find name 'blockScoped'
// console.log(alsoBlockScoped); // Error: Cannot find name 'alsoBlockScoped'
console.log(notBlockScoped); // Works, but not recommended
```

**Real-World Example:**  
In React applications, you commonly use `const` for components, configurations, and immutable state, while `let` is used for counters, accumulators, or variables that change during calculations:

```typescript
// React component using hooks
const UserProfile = ({ userId }: UserProfileProps) => {
  // State hook uses const for the state accessor
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  // let for variables that change during processing
  let retryCount = 0;
  
  const fetchUserData = async () => {
    while (retryCount < 3) {
      try {
        const response = await api.getUser(userId);
        setUser(response.data);
        break;
      } catch (error) {
        retryCount++;
        await new Promise(r => setTimeout(r, 1000));
      } finally {
        setLoading(false);
      }
    }
  };
  
  // Rest of component...
};
```

**Common Pitfalls:**
- Using `var` instead of `let` or `const`, causing unexpected behavior due to hoisting
- Forgetting that `const` only prevents reassignment, not mutation of objects/arrays
- Not considering block scope when declaring variables in loops or conditions
- Redeclaring variables with the same name in the same scope

**Three Common Questions:**

1. **Q: If I declare an object with `const`, why can I still modify its properties?**  
   **A:** `const` only prevents reassignment of the variable itself, not mutation of the object it references. The variable must always reference the same object, but that object's contents can change. To make the object immutable, use `Object.freeze()`.

2. **Q: When should I use `let` vs `const`?**  
   **A:** Use `const` by default for all variables. Only use `let` when you know the variable will need to be reassigned. This practice makes code clearer and prevents accidental reassignment.

3. **Q: What happens if I don't initialize a variable when declaring it with TypeScript?**  
   **A:** If you declare a variable without initializing it, you must specify its type, or TypeScript will infer it as `any`. Additionally, you must assign a value before using it, or you'll get a "variable is used before being assigned" error if `strictNullChecks` is enabled.

#### 1.2 Type Annotations and Inference

**Concise Explanation:**  
Type annotations explicitly define a variable's type using the colon syntax (`variable: type`). Type inference automatically determines a variable's type from its initialization value. TypeScript's inference is powerful, so explicit annotations are only needed when types cannot be inferred correctly.

**Where to Use:**
- Type annotations: Function parameters, function returns, variables without initialization, or to be more explicit
- Type inference: When initializing variables with values that clearly indicate their type

**Code Snippet:**
```typescript
// Type annotations - explicitly specifying types
let age: number = 30;
let name: string = "Alice";
let isActive: boolean = true;
let nullableValue: string | null = null;

// Type inference - TypeScript infers the types
let inferred = "This is a string"; // inferred as string
let inferredNumber = 42; // inferred as number
let inferredBoolean = true; // inferred as boolean

// Arrays and objects with annotations
let numbers: number[] = [1, 2, 3];
let mixed: (string | number)[] = [1, "two", 3];
let userData: { id: number; name: string } = { id: 1, name: "Bob" };

// Function with type annotations
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

// Type inference with functions
const double = (n: number) => n * 2; // Return type inferred as number
```

**Real-World Example:**  
An e-commerce application uses type annotations and inference to manage product data:

```typescript
// Product inventory management
interface Product {
  id: string;
  name: string;
  price: number;
  inStock: boolean;
  categories: string[];
}

// TypeScript infers the type based on the initialization
const products = [
  { id: "p1", name: "Phone", price: 699.99, inStock: true, categories: ["electronics", "gadgets"] },
  { id: "p2", name: "Laptop", price: 1299.99, inStock: false, categories: ["electronics", "computers"] }
]; // Inferred as Product[]

// Function with explicit types for parameters and return value
function findProductById(products: Product[], id: string): Product | undefined {
  return products.find(product => product.id === id);
}

// No need for type annotation here, as TypeScript infers it from findProductById's return type
const laptop = findProductById(products, "p2");
```

**Common Pitfalls:**
- Over-annotating when TypeScript can easily infer types
- Not annotating function parameters, leading to implicit `any`
- Forgetting to specify union types when a variable can be multiple types
- Using `any` when more specific types like `unknown` or generics would be better

**Three Common Questions:**

1. **Q: When should I explicitly add type annotations vs. rely on inference?**  
   **A:** Add explicit type annotations for function parameters, return types, and variables declared without initialization. Rely on inference when initializing variables with clear types. Explicit annotations also help document intent and make complex types clearer.

2. **Q: Why does TypeScript sometimes infer a more specific type than I want?**  
   **A:** TypeScript performs literal type inference, so constants are often inferred as their literal value rather than their general type. For example, `const x = 1` infers `x` as the literal type `1`, not just `number`. Use type annotations to broaden the type when needed: `const x: number = 1`.

3. **Q: How do I handle cases where I don't know the type in advance?**  
   **A:** Use union types (`string | number`), generics (`Array<T>`), or as a last resort, `unknown` (type-safe alternative to `any`). Avoid using `any` as it bypasses type checking.

#### 1.3 Basic Naming Conventions and Scope

**Concise Explanation:**  
TypeScript follows JavaScript naming conventions: camelCase for variables and functions, PascalCase for classes and interfaces, and UPPER_SNAKE_CASE for constants. Scope defines where variables can be accessed, with block-scoped variables (`let`/`const`) only accessible within the block where they're defined.

**Where to Use:**
- camelCase: For variables, functions, methods, properties
- PascalCase: For classes, interfaces, types, enums, type parameters
- UPPER_SNAKE_CASE: For true constants (values that represent unchanging values)
- Scope consideration: When designing nested functions or blocks of code

**Code Snippet:**
```typescript
// Naming conventions
const MAX_RETRY_ATTEMPTS = 3; // Constant
let userName = "Alice"; // Variable
function calculateTax(amount: number): number { // Function
  return amount * 0.2;
}

// Classes, interfaces and types use PascalCase
class UserAccount {
  private accountNumber: string;
  
  constructor(accountNumber: string) {
    this.accountNumber = accountNumber;
  }
}

interface ShippingAddress {
  street: string;
  city: string;
  zipCode: string;
}

// Scope example
function outerFunction() {
  const outerVar = "I'm in the outer function";
  
  function innerFunction() {
    const innerVar = "I'm in the inner function";
    console.log(outerVar); // Can access outer function's variables
  }
  
  innerFunction();
  // console.log(innerVar); // Error: Cannot find name 'innerVar'
}

// Block scope
if (true) {
  const blockVar = "I'm in a block";
  // Accessible only within this block
}
// console.log(blockVar); // Error: Cannot find name 'blockVar'
```

**Real-World Example:**  
A TypeScript React application demonstrates naming conventions and scope:

```typescript
// Constants
const API_BASE_URL = "https://api.example.com";
const MAX_ITEMS_PER_PAGE = 20;

// Interfaces use PascalCase
interface UserData {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
}

// Component uses PascalCase
const UserProfile: React.FC<{userId: string}> = ({userId}) => {
  // Variables use camelCase
  const [userData, setUserData] = useState<UserData | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  
  // Function uses camelCase
  const fetchUserData = async () => {
    try {
      // Local variable scoped to this function
      const response = await fetch(`${API_BASE_URL}/users/${userId}`);
      const data = await response.json();
      setUserData(data);
    } finally {
      setIsLoading(false);
    }
  };
  
  useEffect(() => {
    fetchUserData();
    // fetchUserData is accessible here because it's in the same scope
  }, [userId]);
  
  // JSX returned by component...
};
```

**Common Pitfalls:**
- Inconsistent naming conventions, making code harder to read and maintain
- Using reserved keywords as variable names
- Not respecting block scope, causing unexpected behavior
- Accessing variables before they're declared (temporal dead zone for `let`/`const`)
- Shadow variables by reusing names in nested scopes, leading to confusion

**Three Common Questions:**

1. **Q: Is there any automatic enforcement of naming conventions in TypeScript?**  
   **A:** TypeScript itself doesn't enforce naming conventions, but linters like ESLint with TypeScript plugins can enforce them through rules like `@typescript-eslint/naming-convention`. Configure these tools in your project for consistent style.

2. **Q: How does variable shadowing work in TypeScript?**  
   **A:** Variable shadowing occurs when a variable in an inner scope has the same name as one in an outer scope. The inner variable "shadows" the outer one, hiding it within that scope. While legal, this can cause confusion and bugs, so it's generally best avoided.

3. **Q: What's the difference between namespace scope and module scope?**  
   **A:** Namespace scope refers to variables declared inside a TypeScript namespace, accessible only within that namespace unless exported. Module scope refers to variables in an ES module (a file), which are only accessible within that module unless exported. Modern TypeScript favors ES modules over namespaces.

#### 1.4 Literal Types and Type Narrowing

**Concise Explanation:**  
Literal types are specific values rather than general types, like `"error"` instead of just `string`. Type narrowing is the process where TypeScript recognizes when a variable has been checked, thereby restricting its type to be more specific within a certain code block.

**Where to Use:**
- Literal types: For constants with specific known values, enum-like variables, or strict API responses
- Type narrowing: When handling union types, validating input, or implementing conditional logic based on types

**Code Snippet:**
```typescript
// Literal types
let status: "loading" | "success" | "error";
status = "loading"; // Valid
// status = "pending"; // Error: Type '"pending"' is not assignable to type '"loading" | "success" | "error"'

// Numeric literal types
type DiceValue = 1 | 2 | 3 | 4 | 5 | 6;
let roll: DiceValue = 4; // Valid
// let roll: DiceValue = 7; // Error: Type '7' is not assignable to type 'DiceValue'

// Boolean literal types
type True = true;
const isEnabled: True = true; // Valid
// const isEnabled: True = false; // Error

// Type narrowing with type guards
function process(value: string | number) {
  if (typeof value === "string") {
    // In this block, TypeScript knows that value is a string
    return value.toUpperCase();
  } else {
    // In this block, TypeScript knows that value is a number
    return value.toFixed(2);
  }
}

// Custom type guard
function isProduct(item: any): item is Product {
  return item && item.id && item.price !== undefined;
}

function displayItem(item: any) {
  if (isProduct(item)) {
    // TypeScript now knows item is a Product
    console.log(`Product: ${item.name}, Price: $${item.price}`);
  }
}
```

**Real-World Example:**  
A payment processing application uses literal types and type narrowing:

```typescript
// Payment processing application
type PaymentStatus = "pending" | "processing" | "completed" | "failed" | "refunded";
type PaymentMethod = "credit_card" | "paypal" | "bank_transfer" | "crypto";

interface Payment {
  id: string;
  amount: number;
  status: PaymentStatus;
  method: PaymentMethod;
  timestamp: Date;
}

function processPayment(payment: Payment) {
  // Type narrowing with switch
  switch (payment.status) {
    case "pending":
      return startProcessing(payment);
    case "processing":
      return checkProcessingStatus(payment);
    case "completed":
      return sendReceipt(payment);
    case "failed":
      return handleFailure(payment);
    case "refunded":
      return confirmRefund(payment);
  }
}

function displayPaymentDetails(payment: Payment | null) {
  // Type narrowing with null check
  if (!payment) {
    return "No payment information available";
  }
  
  // After the check, TypeScript knows payment is not null
  return `Payment of $${payment.amount} via ${payment.method} is ${payment.status}`;
}
```

**Common Pitfalls:**
- Not handling all possible cases in unions when narrowing types
- Forgetting that type narrowing only applies within the block where the check occurs
- Using type assertions (`as`) instead of proper type narrowing
- Not realizing literal types are much more restrictive than their base types

**Three Common Questions:**

1. **Q: How do literal types differ from enums?**  
   **A:** Literal types are specific values of basic types (like string or number) and can form unions. Enums are a TypeScript feature that creates a named set of constants. Literal types are often preferred in modern TypeScript as they're more flexible, have better type safety, and result in smaller compiled JavaScript.

2. **Q: Why doesn't TypeScript narrow the type outside my if-statement?**  
   **A:** Type narrowing is block-scoped. Once you exit the block (if statement, loop, etc.), TypeScript reverts to the original type. For cross-block narrowing, assign the narrowed value to a new variable or use a type assertion.

3. **Q: Can I use functions for type narrowing?**  
   **A:** Yes, with "user-defined type guards" by adding a type predicate (`is` keyword) to your function's return type. For example: `function isString(value: any): value is string { return typeof value === 'string'; }`. After calling this function in a condition, TypeScript will know the correct type.

### 2. Primitive Data Types

#### 2.1 Boolean, Number, String

**Concise Explanation:**  
The most common primitive types in TypeScript are `boolean` (true/false), `number` (integers and floating-point), and `string` (text). TypeScript enforces type safety, preventing operations between incompatible types.

**Where to Use:**
- `boolean`: For flags, toggles, condition checks
- `number`: For quantities, measurements, indices, mathematical operations
- `string`: For text, names, IDs, messages

**Code Snippet:**
```typescript
// Boolean
let isActive: boolean = true;
let hasPermission: boolean = false;
let isValid = Boolean("true"); // Constructs a boolean object
let isParsed = !!0; // Common pattern to convert to boolean (false in this case)

// Number
let integer: number = 42;
let float: number = 3.14;
let hex: number = 0xf00d; // Hexadecimal: 61453
let binary: number = 0b1010; // Binary: 10
let octal: number = 0o744; // Octal: 484
let infinity: number = Infinity;
let notANumber: number = NaN;

// String
let firstName: string = "John";
let lastName: string = 'Doe';
let greeting = `Hello, ${firstName} ${lastName}`; // Template string
let multiline = `This is a
multiline string`;

// String methods
let upperName = firstName.toUpperCase(); // "JOHN"
let hasD = lastName.includes("D"); // true
let replaced = greeting.replace("Hello", "Hi"); // "Hi, John Doe"
```

**Real-World Example:**  
A user registration form validation shows how these types work together:

```typescript
interface UserRegistration {
  username: string;
  password: string;
  age: number;
  agreeToTerms: boolean;
}

function validateRegistration(user: UserRegistration): string[] {
  const errors: string[] = [];
  
  // String validation
  if (user.username.length < 5) {
    errors.push("Username must be at least 5 characters long");
  }
  
  if (!/[A-Z]/.test(user.password) || !/[0-9]/.test(user.password)) {
    errors.push("Password must contain at least one uppercase letter and one number");
  }
  
  // Number validation
  if (user.age < 18) {
    errors.push("You must be at least 18 years old");
  }
  
  // Boolean validation
  if (!user.agreeToTerms) {
    errors.push("You must agree to the terms and conditions");
  }
  
  return errors;
}

const formData: UserRegistration = {
  username: "john123",
  password: "Secret1",
  age: 25,
  agreeToTerms: true
};

const validationErrors = validateRegistration(formData);
if (validationErrors.length === 0) {
  console.log("Registration valid!");
} else {
  console.log("Please fix the following errors:", validationErrors);
}
```

**Common Pitfalls:**
- Forgetting that `Number("123")` returns a primitive number, but `new Number("123")` returns a Number object
- Not accounting for NaN in calculations, which propagates through operations
- Using `==` instead of `===` for comparisons, which can lead to unexpected type coercions
- Forgetting that empty strings are falsy in conditional checks
- Not handling potential undefined or null values before performing operations

**Three Common Questions:**

1. **Q: What's the difference between `string` and `String` types in TypeScript?**  
   **A:** `string` (lowercase) is the primitive type, which you should use most of the time. `String` (uppercase) is the constructor object type. TypeScript best practice is to use lowercase primitive types (`string`, `number`, `boolean`) instead of their constructor counterparts.

2. **Q: How do I convert between different primitive types?**  
   **A:** For string to number: `Number(str)`, `parseInt(str)`, or `parseFloat(str)`. For number to string: `String(num)` or `num.toString()`. For any type to boolean: `Boolean(value)` or the double-negation shorthand `!!value`.

3. **Q: Why doesn't TypeScript catch errors with NaN or division by zero?**  
   **A:** These are runtime behaviors of JavaScript that aren't caught by TypeScript's type system. NaN is still of type `number`, and division by zero results in `Infinity`, which is also a valid `number`. Use runtime checks to handle these cases.

#### 2.2 Null and Undefined

**Concise Explanation:**  
In TypeScript, `null` represents an intentional absence of value, while `undefined` represents an uninitialized variable or missing property. The `strictNullChecks` compiler option enforces explicit handling of these values to prevent runtime errors.

**Where to Use:**
- `null`: When you want to explicitly indicate that a variable has no value
- `undefined`: For variables not yet assigned, optional parameters, or properties that may not exist
- Union types with `null`/`undefined`: For values that might be missing or not set

**Code Snippet:**
```typescript
// With strictNullChecks enabled
let name: string; // Type is string, but uninitialized (undefined)
// console.log(name); // Error: Variable 'name' is used before being assigned

let middleName: string | undefined; // Can be string or explicitly undefined
middleName = "Marie";
middleName = undefined; // Valid

let lastName: string | null = null; // Can be string or explicitly null
lastName = "Smith";

// Optional parameters and properties
function greet(name: string, title?: string) {
  // title is of type string | undefined
  if (title) {
    return `Hello, ${title} ${name}`;
  }
  return `Hello, ${name}`;
}

// Type guard for null and undefined
function getLength(text: string | null | undefined): number {
  // Non-null assertion operator (!) - use with caution
  // return text!.length; // Dangerous if text is null/undefined

  // Safe approach with type guard
  if (text) {
    return text.length;
  }
  return 0;
}

// Nullish coalescing operator (??) - TypeScript 3.7+
let userInput: string | null | undefined;
let message = userInput ?? "Default message"; // Use default if null/undefined
```

**Real-World Example:**  
A user profile management system that handles missing data:

```typescript
interface UserProfile {
  id: string;
  name: string;
  email: string;
  phoneNumber?: string; // Optional property
  address: Address | null; // Might explicitly have no address
  preferences?: UserPreferences; // Might not have preferences set
}

interface Address {
  street: string;
  city: string;
  zipCode: string;
}

interface UserPreferences {
  theme: 'light' | 'dark';
  notifications: boolean;
}

function renderUserContact(user: UserProfile): string {
  let contact = `Name: ${user.name}, Email: ${user.email}`;
  
  // Safe handling of optional phoneNumber
  if (user.phoneNumber) {
    contact += `, Phone: ${user.phoneNumber}`;
  }
  
  // Safe handling of nullable address
  if (user.address) {
    contact += `, Address: ${user.address.street}, ${user.address.city}`;
  } else {
    contact += ', No address provided';
  }
  
  // Safe handling of undefined preferences using optional chaining
  const theme = user.preferences?.theme ?? 'system default';
  contact += `, Theme: ${theme}`;
  
  return contact;
}
```

**Common Pitfalls:**
- Not enabling `strictNullChecks` in tsconfig.json, which allows null assignment to any type
- Using non-null assertion (`!`) when you're not 100% certain a value isn't null/undefined
- Forgetting to handle null/undefined cases in functions that might receive them
- Not distinguishing between intentional null values and undefined values
- Performing operations directly on potentially null/undefined values without checks

**Three Common Questions:**

1. **Q: When should I use null versus undefined?**  
   **A:** Use `undefined` for variables not yet assigned a value or for optional parameters/properties that weren't provided. Use `null` when you want to explicitly indicate "no value" for something that normally would have a value. However, many teams standardize on using just one of them throughout a codebase for consistency.

2. **Q: What's the difference between optional chaining (`?.`) and nullish coalescing (`??`)?**  
   **A:** Optional chaining (`?.`) safely accesses properties or methods on potentially null/undefined objects, returning undefined if the object is null/undefined. Nullish coalescing (`??`) provides a default value only when the left operand is null or undefined (but not for other falsy values like empty string or 0).

3. **Q: How do I tell TypeScript that I'm certain a value is not null when it thinks it might be?**  
   **A:** Use the non-null assertion operator (`!`), but only when you're absolutely certain: `const name = user!.name;`. Better options are to use a type guard check first (`if (user) { ... }`) or optional chaining (`user?.name`). The non-null assertion should be used sparingly, as it can lead to runtime errors.

#### 2.3 Symbol and BigInt

**Concise Explanation:**  
`Symbol` creates unique, immutable identifiers, useful for object keys where uniqueness is important. `BigInt` represents integers of arbitrary precision, beyond the safe integer limit of regular `number` types.

**Where to Use:**
- `Symbol`: For unique property keys, avoiding name collisions in objects or as unique identifiers
- `BigInt`: For working with very large integers (beyond ±2^53), such as in financial calculations or cryptography

**Code Snippet:**
```typescript
// Symbol
const id = Symbol("id"); // Description is for debugging only
const anotherId = Symbol("id");

console.log(id === anotherId); // false, symbols are always unique

// Using Symbol as property key
const user = {
  name: "Alice",
  [id]: 123456 // Symbol as property key
};

console.log(user[id]); // 123456
// Object.keys(user) won't include the symbol key

// Shared symbols via Symbol.for
const sharedSymbol1 = Symbol.for("shared");
const sharedSymbol2 = Symbol.for("shared");

console.log(sharedSymbol1 === sharedSymbol2); // true, same symbol from registry

// BigInt
const bigNumber = 9007199254740991n; // 'n' suffix denotes BigInt literal
const alsoHuge = BigInt("9007199254740991"); // Constructor approach

// BigInt operations
const result = bigNumber + 1n;
console.log(result); // 9007199254740992n

// Can't mix BigInt and number directly
// console.log(bigNumber + 1); // Error: Cannot mix BigInt and other types
console.log(Number(bigNumber) + 1); // Convert to number if value fits
```

**Real-World Example:**  
A unique ID system and financial calculation using Symbol and BigInt:

```typescript
// Unique property identifier with Symbol
class Database {
  private static readonly TABLE_NAME = Symbol('tableName');
  private static readonly PRIMARY_KEY = Symbol('primaryKey');
  
  constructor(tableName: string, primaryKey: string) {
    this[Database.TABLE_NAME] = tableName;
    this[Database.PRIMARY_KEY] = primaryKey;
  }
  
  query() {
    // Symbols can't be accessed from outside without reference to symbol
    console.log(`Querying ${this[Database.TABLE_NAME]} by ${this[Database.PRIMARY_KEY]}`);
  }
}

// BigInt for financial calculations
class FinancialCalculator {
  // Using BigInt for precise calculations with large numbers
  static calculateCompoundInterest(
    principal: bigint,
    rate: number,
    years: number,
    compoundingPerYear: number
  ): bigint {
    const ratePerPeriod = rate / compoundingPerYear;
    const totalPeriods = BigInt(Math.round(years * compoundingPerYear));
    
    // Convert rate to basis points for BigInt calculation (move decimal 4 places)
    const rateInBasisPoints = BigInt(Math.round(ratePerPeriod * 10000));
    
    let result = principal;
    for (let i = 0n; i < totalPeriods; i++) {
      // Calculate interest (principal * rate / 10000)
      const interest = (result * rateInBasisPoints) / 10000n;
      result += interest;
    }
    
    return result;
  }
}

// Usage
const investment = 1000000000000000n; // 1 quadrillion
const finalAmount = FinancialCalculator.calculateCompoundInterest(
  investment,
  0.05, // 5% annual interest
  30,   // 30 years
  12    // Monthly compounding
);
```

**Common Pitfalls:**
- Trying to convert a Symbol to a string or using it in JSON serialization
- Forgetting that Symbols as object keys won't show up in `Object.keys()` or `for...in` loops
- Attempting to mix BigInt and regular numbers in calculations without explicit conversion
- Not accounting for BigInt's absence in older JavaScript environments
- Overestimating when BigInt is needed (regular number works fine for most applications)

**Three Common Questions:**

1. **Q: Why would I use a Symbol instead of a string as an object key?**  
   **A:** Symbols guarantee uniqueness, preventing property name conflicts. This is especially useful for library authors adding properties to objects they don't own, or for creating "private" properties that won't be enumerated in loops. Symbols are also excluded from JSON serialization.

2. **Q: When should I use BigInt instead of number?**  
   **A:** Use BigInt when you need to represent integers larger than 2^53-1 (Number.MAX_SAFE_INTEGER) or smaller than -(2^53-1). Common use cases include working with very large IDs, financial calculations requiring precision for large values, cryptography, or when bit manipulation of large integers is needed.

3. **Q: How do I convert between BigInt and regular number types?**  
   **A:** Convert number to BigInt: `BigInt(123)`. Convert BigInt to number: `Number(123n)`. Be careful with the latter, as it can lose precision if the BigInt value exceeds the safe integer range of number. Check with `Number.isSafeInteger(Number(bigIntValue))` if precision matters.

#### 2.4 Arrays and Tuples

**Concise Explanation:**  
Arrays in TypeScript are ordered collections of elements of the same type or union of types. Tuples are arrays with fixed length and specific types for each position, useful when each element has a distinct meaning.

**Where to Use:**
- Arrays: For collections of elements of the same type with variable length
- Tuples: For fixed-length arrays where each position has a specific meaning and possibly different type
- Read-only arrays: When the array contents shouldn't be modified

**Code Snippet:**
```typescript
// Array types
let numbers: number[] = [1, 2, 3, 4, 5];
let strings: Array<string> = ["a", "b", "c"]; // Generic syntax

// Union type arrays
let mixed: (string | number)[] = [1, "two", 3, "four"];

// Multidimensional arrays
let matrix: number[][] = [
  [1, 2, 3],
  [4, 5, 6]
];

// Readonly arrays
const readonlyNumbers: readonly number[] = [1, 2, 3];
// readonlyNumbers.push(4); // Error: Property 'push' does not exist on type 'readonly number[]'

// Array methods (type-aware)
let first = numbers[0]; // Type is number
let popped = numbers.pop(); // Type is number | undefined
numbers.push(6); // Only accepts numbers
let doubled = numbers.map(n => n * 2); // Inferred as number[]
let found = numbers.find(n => n > 3); // Inferred as number | undefined

// Tuples
let tuple: [string, number, boolean] = ["hello", 42, true];
let first = tuple[0]; // Type is string
let second = tuple[1]; // Type is number
// tuple[3] = "error"; // Error: Index '3' is out of bounds

// Named tuples (for clarity)
let person: [name: string, age: number, active: boolean] = ["Alice", 30, true];

// Optional tuple elements
let optionalTuple: [string, number?] = ["hello"]; // Second element is optional

// Rest elements in tuples
let restTuple: [number, ...string[]] = [1, "a", "b", "c"];

// Readonly tuples
let readonlyTuple: readonly [string, number] = ["hello", 42];
// readonlyTuple[0] = "hi"; // Error: Cannot assign to '0' because it is a read-only property
```

**Real-World Example:**  
A data processing application using arrays and tuples:

```typescript
// Representing CSV data using tuples
type CsvRow = [string, string, number, Date]; // [name, category, amount, date]

function processTransactions(csvData: CsvRow[]): number {
  let total = 0;
  
  for (const [name, category, amount, date] of csvData) {
    // Each element has a known type based on its position
    if (category === "income") {
      total += amount;
    } else if (category === "expense") {
      total -= amount;
    }
    
    console.log(`${date.toLocaleDateString()}: ${name} - $${amount}`);
  }
  
  return total;
}

// Using arrays for data collection
function analyzeSpending(transactions: CsvRow[]): Record<string, number> {
  const categoryTotals: Record<string, number> = {};
  
  transactions.forEach(([name, category, amount, date]) => {
    if (!categoryTotals[category]) {
      categoryTotals[category] = 0;
    }
    categoryTotals[category] += amount;
  });
  
  return categoryTotals;
}

// Using array methods for data transformation
function getTransactionsByMonth(transactions: CsvRow[]): Map<string, CsvRow[]> {
  return transactions.reduce((map, transaction) => {
    const [, , , date] = transaction;
    const monthKey = `${date.getFullYear()}-${date.getMonth() + 1}`;
    
    if (!map.has(monthKey)) {
      map.set(monthKey, []);
    }
    
    map.get(monthKey)!.push(transaction);
    return map;
  }, new Map<string, CsvRow[]>());
}
```

**Common Pitfalls:**
- Confusing arrays and tuples when both use square bracket notation
- Not guarding against out-of-bounds array access
- Forgetting that JavaScript arrays are mutable by default
- Not handling potentially undefined values when accessing array elements
- Ignoring TypeScript's warnings about tuple length mismatches

**Three Common Questions:**

1. **Q: When should I use a tuple instead of an array or an object?**  
   **A:** Use tuples when you have a fixed number of elements where each position has a specific meaning and potentially different type. For example, a coordinate pair `[x, y]` or a key-value pair `[key, value]`. If you need named properties or the structure may change, use an object. If you have a collection of the same type with variable length, use an array.

2. **Q: How do I enforce that an array's length doesn't change?**  
   **A:** Use a readonly tuple with a specific length: `const pair: readonly [number, number] = [1, 2];`. For arrays where you know the exact length but all elements are the same type, you can use: `const fiveNumbers: readonly [number, number, number, number, number] = [1, 2, 3, 4, 5];`. For variable-length arrays that shouldn't be modified, use `readonly number[]`.

3. **Q: How can I access tuple elements by name rather than position?**  
   **A:** TypeScript doesn't provide direct named access for tuples. You have three options: 1) Use destructuring: `const [name, age] = person;`, 2) Use named tuple elements for documentation: `const person: [name: string, age: number] = ["Alice", 30];`, or 3) Use an object instead of a tuple if named access is important.

#### 2.5 Type Conversions and Assertions

**Concise Explanation:**  
Type conversions change a value from one type to another, either explicitly (like `Number(x)`) or implicitly through JavaScript's type coercion. Type assertions tell TypeScript to treat a value as a specific type when the compiler can't infer it correctly, without changing the runtime value.

**Where to Use:**
- Type conversions: When you need to actually change a value's type (like string to number)
- Type assertions: When you know more about a type than TypeScript can infer
- Const assertions: When you want TypeScript to infer the most specific literal type

**Code Snippet:**
```typescript
// Type conversions (changing runtime value)
let str = "42";
let num = Number(str); // Converts string to number
let bool = Boolean(str); // Converts string to boolean
let back = String(num); // Converts number to string

// Common conversion patterns
let intValue = parseInt(str); // String to integer
let floatValue = parseFloat("3.14"); // String to float
let hexValue = parseInt("FF", 16); // Hex string to number

// Type assertions (no runtime conversion)
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length; // as syntax
let strLength2: number = (<string>someValue).length; // angle bracket syntax (not used in JSX)

// Type assertions for objects
interface User {
  name: string;
  email: string;
}

let userData = JSON.parse('{"name":"John","email":"john@example.com"}');
let user = userData as User; // Assert that userData conforms to User interface

// Const assertions (TypeScript 3.4+)
// Without const assertion
let color = { name: "red" }; // Inferred as { name: string }
color.name = "blue"; // Valid

// With const assertion
let color2 = { name: "red" } as const; // Inferred as { readonly name: "red" }
// color2.name = "blue"; // Error: Cannot assign to 'name' because it is a read-only property

// Array with const assertion
let numbers = [1, 2, 3] as const; // readonly [1, 2, 3]
// numbers.push(4); // Error: Property 'push' does not exist on type 'readonly [1, 2, 3]'
```

**Real-World Example:**  
An API client handling JSON responses:

```typescript
interface ApiResponse {
  data: {
    users: User[];
    lastPage: number;
  };
  status: "success" | "error";
  message?: string;
}

interface User {
  id: number;
  name: string;
  email: string;
  active: boolean;
}

class ApiClient {
  async getUsers(): Promise<User[]> {
    const response = await fetch("https://api.example.com/users");
    
    // Parse JSON and assert type
    const jsonData = await response.json();
    const apiResponse = jsonData as ApiResponse;
    
    // Type guard for safer type assertion
    if (apiResponse.status === "error") {
      throw new Error(apiResponse.message || "Failed to fetch users");
    }
    
    return apiResponse.data.users;
  }
  
  formatUserData(user: unknown): User {
    // Type guard instead of direct assertion
    if (typeof user !== "object" || user === null) {
      throw new TypeError("Expected user to be an object");
    }
    
    const userObj = user as Record<string, unknown>;
    
    // Convert and validate fields
    const formattedUser: User = {
      id: Number(userObj.id || 0),
      name: String(userObj.name || ""),
      email: String(userObj.email || ""),
      active: Boolean(userObj.active)
    };
    
    return formattedUser;
  }
}

// Const assertions for configuration
const API_CONFIG = {
  baseUrl: "https://api.example.com",
  version: "v1",
  endpoints: {
    users: "/users",
    posts: "/posts"
  },
  rateLimit: 100
} as const; // All properties become readonly, and string values become literal types
```

**Common Pitfalls:**
- Using type assertions (`as`) too liberally, bypassing TypeScript's type safety
- Confusing type assertions (which only affect TypeScript) with type conversions (which change runtime values)
- Not handling potential failure cases in type conversions (e.g., `parseInt` failing)
- Using type assertions when a type guard would be safer
- Not understanding that type assertions don't perform runtime validation

**Three Common Questions:**

1. **Q: What's the difference between type assertions and type casting?**  
   **A:** In TypeScript, type assertions (`as` or `<>` syntax) only tell the compiler to treat a value as a certain type without runtime changes. Type casting (like `Number(x)`) actually converts a value from one type to another at runtime. Type assertions are compile-time constructs, while type casting affects the runtime value.

2. **Q: Is it safe to use type assertions with `any`?**  
   **A:** No. Type assertions like `value as TargetType` don't perform runtime checks, so asserting `any` to a specific type bypasses TypeScript's safety. Instead, use a type guard (`if (typeof value === 'string')`) or safer approaches like `unknown` with proper checking or the newer `satisfies` operator.

3. **Q: When would I use `as const` assertion?**  
   **A:** Use `as const` when you want TypeScript to infer the most specific literal type instead of a general type. It's useful for object literals that should be treated as read-only constants, arrays that shouldn't change, or when you want string literals to be treated as specific literal types rather than general strings.

### 3. Control Structures

#### 3.1 Conditional Statements (if-else)

**Concise Explanation:**  
Conditional statements in TypeScript follow JavaScript's syntax but benefit from TypeScript's type checking. The `if-else` statement executes different code blocks based on conditions, and TypeScript narrows types within each block based on the conditions.

**Where to Use:**
- Simple binary logic (if-else)
- Multiple related conditions (if-else-if-else)
- Type narrowing with type guards
- Conditional return values

**Code Snippet:**
```typescript
// Basic if-else
const age = 25;

if (age >= 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}

// Multiple conditions with else-if
function getAgeCategory(age: number): string {
  if (age < 13) {
    return "Child";
  } else if (age < 20) {
    return "Teenager";
  } else if (age < 65) {
    return "Adult";
  } else {
    return "Senior";
  }
}

// Type narrowing with conditionals
function processValue(value: string | number) {
  if (typeof value === "string") {
    // TypeScript knows value is a string here
    return value.toUpperCase();
  } else {
    // TypeScript knows value is a number here
    return value.toFixed(2);
  }
}

// Truthiness checks
function printName(name: string | null | undefined) {
  if (name) {
    // TypeScript knows name is a non-empty string here
    console.log(`Name: ${name}`);
  } else {
    // TypeScript knows name is either null, undefined, or ""
    console.log("Name not provided");
  }
}

// Using optional chaining with conditionals
function getUserCity(user: { address?: { city?: string } }) {
  if (user.address?.city) {
    // Safe access with definite assignment
    return user.address.city;
  }
  return "Unknown";
}
```

**Real-World Example:**  
Form validation logic in a TypeScript React application:

```typescript
interface FormData {
  email: string;
  password: string;
  confirmPassword: string;
  age?: number;
  terms: boolean;
}

function validateForm(formData: FormData): string[] {
  const errors: string[] = [];
  
  // Simple condition
  if (!formData.terms) {
    errors.push("You must accept the terms and conditions");
  }
  
  // Email validation with regex
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(formData.email)) {
    errors.push("Please enter a valid email address");
  }
  
  // Password validation with multiple conditions
  if (formData.password.length < 8) {
    errors.push("Password must be at least 8 characters");
  } else if (!/[A-Z]/.test(formData.password)) {
    errors.push("Password must contain at least one uppercase letter");
  } else if (!/[0-9]/.test(formData.password)) {
    errors.push("Password must contain at least one number");
  }
  
  // Comparing two fields
  if (formData.password !== formData.confirmPassword) {
    errors.push("Passwords do not match");
  }
  
  // Optional field with type narrowing
  if (formData.age !== undefined) {
    if (formData.age < 18) {
      errors.push("You must be at least 18 years old");
    } else if (formData.age > 120) {
      errors.push("Please enter a valid age");
    }
  }
  
  return errors;
}
```

**Common Pitfalls:**
- Not accounting for all possible types when using conditionals for type narrowing
- Using `==` instead of `===` (equality with type coercion vs strict equality)
- Forgetting that falsy values include `0`, `""`, `null`, `undefined`, `NaN`, and `false`
- Not considering edge cases in conditionals, especially for user input
- Overusing nested if statements, leading to "arrow code" or "pyramid of doom"

**Three Common Questions:**

1. **Q: How does TypeScript know my variable's type changes after a condition?**  
   **A:** TypeScript uses flow analysis called "control flow based type analysis" to track how a variable's type changes based on conditions. When you check a variable's type or value with `typeof`, `instanceof`, property checks, or other type guards, TypeScript narrows the variable's type within that scope.

2. **Q: Why doesn't TypeScript narrow the type after my custom validation function?**  
   **A:** Regular functions don't automatically narrow types after they return. To make TypeScript recognize custom validation as type narrowing, you need to define a "user-defined type guard" function that returns a type predicate, like: `function isString(value: any): value is string { return typeof value === 'string'; }`

3. **Q: Is there a conditional expression alternative to if-else statements?**  
   **A:** Yes, the ternary conditional operator: `const result = condition ? valueIfTrue : valueIfFalse;`. It's useful for simple conditions, especially when assigning values or returning from functions. For more complex conditions, if-else statements are often more readable.

#### 3.2 Switch Statements and Type Checking

**Concise Explanation:**  
Switch statements provide a cleaner way to handle multiple conditions based on a single value. In TypeScript, they work with type checking and contribute to type narrowing, making them particularly useful with union types, string literals, and enums.

**Where to Use:**
- Multiple related conditions with the same variable
- Pattern matching with string or number literals
- Exhaustive checks with union types
- Implementation of state machines

**Code Snippet:**
```typescript
// Basic switch with numbers
function getDayName(day: number): string {
  switch (day) {
    case 0:
      return "Sunday";
    case 1:
      return "Monday";
    case 2:
      return "Tuesday";
    case 3:
      return "Wednesday";
    case 4:
      return "Thursday";
    case 5:
      return "Friday";
    case 6:
      return "Saturday";
    default:
      return "Invalid day";
  }
}

// Switch with string literals
type Direction = "north" | "south" | "east" | "west";

function getVector(direction: Direction): [number, number] {
  switch (direction) {
    case "north":
      return [0, 1];
    case "south":
      return [0, -1];
    case "east":
      return [1, 0];
    case "west":
      return [-1, 0];
  }
}

// Exhaustive switch with union types
type Shape = 
  | { kind: "circle"; radius: number } 
  | { kind: "rectangle"; width: number; height: number }
  | { kind: "triangle"; base: number; height: number };

function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      // TypeScript can detect if we're missing any cases
      const _exhaustiveCheck: never = shape;
      throw new Error(`Unhandled shape kind: ${_exhaustiveCheck}`);
  }
}

// Grouped cases
function getDiscountRate(customerType: string): number {
  switch (customerType) {
    case "premium":
      return 0.2;
    case "gold":
    case "platinum":
      return 0.15;
    case "silver":
    case "bronze":
      return 0.1;
    default:
      return 0.05;
  }
}
```

**Real-World Example:**  
A state machine for processing user account status transitions:

```typescript
// Define possible account statuses
type AccountStatus = 
  | "pending" 
  | "active" 
  | "suspended" 
  | "deactivated" 
  | "closed";

// Define the allowed status transitions
type AccountAction = 
  | { type: "approve" }
  | { type: "suspend"; reason: string }
  | { type: "reactivate" }
  | { type: "deactivate"; byUser: boolean }
  | { type: "close"; reason: string };

class AccountProcessor {
  processStatusChange(
    currentStatus: AccountStatus, 
    action: AccountAction
  ): { newStatus: AccountStatus; message: string } {
    
    switch (action.type) {
      case "approve":
        if (currentStatus !== "pending") {
          throw new Error("Only pending accounts can be approved");
        }
        return {
          newStatus: "active",
          message: "Account has been approved and activated"
        };
        
      case "suspend":
        if (currentStatus !== "active") {
          throw new Error("Only active accounts can be suspended");
        }
        return {
          newStatus: "suspended",
          message: `Account suspended. Reason: ${action.reason}`
        };
        
      case "reactivate":
        switch (currentStatus) {
          case "suspended":
          case "deactivated":
            return {
              newStatus: "active",
              message: "Account has been reactivated"
            };
          default:
            throw new Error(`Cannot reactivate account with status: ${currentStatus}`);
        }
        
      case "deactivate":
        if (currentStatus !== "active") {
          throw new Error("Only active accounts can be deactivated");
        }
        return {
          newStatus: "deactivated",
          message: action.byUser 
            ? "Account deactivated by user" 
            : "Account deactivated by system"
        };
        
      case "close":
        if (currentStatus === "closed") {
          throw new Error("Account is already closed");
        }
        return {
          newStatus: "closed",
          message: `Account has been closed. Reason: ${action.reason}`
        };
        
      default:
        // Exhaustiveness check
        const _exhaustiveCheck: never = action;
        throw new Error(`Unhandled action type: ${JSON.stringify(action)}`);
    }
  }
}
```

**Common Pitfalls:**
- Forgetting `break` statements, causing fall-through (this is sometimes intentional, but often a bug)
- Not handling the `default` case when it's needed
- Using expressions as cases rather than constants
- Attempting to `switch` on complex conditions that would be better served by `if-else`
- Not utilizing TypeScript's exhaustiveness checking with discriminated unions and `never` type

**Three Common Questions:**

1. **Q: How can I ensure I've handled all possible values in a switch statement?**  
   **A:** Use a discriminated union type with a type property (like `kind` or `type`), then add a default case with an exhaustiveness check: `const _exhaustiveCheck: never = remainingValue;`. If you add a new type case but forget to add it to the switch, TypeScript will generate a compile-time error.

2. **Q: Why do I need `break` in each case, and what is fall-through?**  
   **A:** Without `break`, execution continues to the next case after matching one, which is called "fall-through". This is sometimes used intentionally to share code between cases, but it's often accidental and can cause bugs. TypeScript doesn't protect against fall-through, so you need to remember your `break` statements.

3. **Q: Can I use switch statements with complex objects or arrays?**  
   **A:** The switch expression is compared with case values using strict equality (`===`), which works best with primitive values. For complex objects, either use a discriminated union with a literal property to switch on, or use `if-else` with more complex conditions. You can't directly switch on object shapes or array structures.

#### 3.3 Loops (for, while, do-while, for...of, for...in)

**Concise Explanation:**  
TypeScript supports all JavaScript loop types with added type safety: `for` loops for counting, `while` and `do-while` for condition-based iteration, `for...of` for iterating values in arrays and iterables, and `for...in` for object properties.

**Where to Use:**
- `for`: When you need an index counter, or want precise control over iteration
- `while`: When you need to iterate until a condition becomes false
- `do-while`: When you need to execute code at least once, then continue based on a condition
- `for...of`: For iterating through arrays, strings, or other iterable values
- `for...in`: For iterating through object properties (keys)

**Code Snippet:**
```typescript
// Standard for loop
const numbers: number[] = [1, 2, 3, 4, 5];
for (let i = 0; i < numbers.length; i++) {
  console.log(numbers[i]);
}

// for loop with multiple variables
for (let i = 0, j = 10; i < 5; i++, j--) {
  console.log(i, j); // 0,10  1,9  2,8  3,7  4,6
}

// while loop
let count = 0;
while (count < 5) {
  console.log(count);
  count++;
}

// do-while loop - executes at least once
let answer: string;
do {
  answer = prompt("Please type 'yes'") || "";
} while (answer !== "yes");

// for...of loop (iterating values) - preferred for arrays
const fruits: string[] = ["apple", "banana", "cherry"];
for (const fruit of fruits) {
  console.log(fruit); // Iterates over values: "apple", "banana", "cherry"
}

// for...of with index using entries()
for (const [index, fruit] of fruits.entries()) {
  console.log(`${index}: ${fruit}`);
}

// for...of with iterating Map
const map = new Map<string, number>([
  ["apple", 1],
  ["banana", 2]
]);
for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}

// for...in loop (iterating keys/indices) - for objects
const person = { name: "Alice", age: 30, city: "Wonderland" };
for (const key in person) {
  // TypeScript doesn't know the exact type without a check
  if (Object.prototype.hasOwnProperty.call(person, key)) {
    const value = person[key as keyof typeof person];
    console.log(`${key}: ${value}`);
  }
}

// Loop control statements
for (let i = 0; i < 10; i++) {
  if (i === 3) continue; // Skip this iteration
  if (i === 7) break;    // Exit the loop
  console.log(i); // 0, 1, 2, 4, 5, 6
}
```

**Real-World Example:**  
Data processing with different loop types:

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  categories: string[];
  inStock: boolean;
}

class ProductProcessor {
  // Using for loop with indices
  findProductsInPriceRange(products: Product[], min: number, max: number): Product[] {
    const result: Product[] = [];
    for (let i = 0; i < products.length; i++) {
      const product = products[i];
      if (product.price >= min && product.price <= max) {
        result.push(product);
      }
    }
    return result;
  }
  
  // Using while loop with condition
  searchProductsByName(products: Product[], searchTerm: string): Product[] {
    const results: Product[] = [];
    let i = 0;
    
    while (i < products.length && results.length < 5) { // Find up to 5 matches
      if (products[i].name.toLowerCase().includes(searchTerm.toLowerCase())) {
        results.push(products[i]);
      }
      i++;
    }
    
    return results;
  }
  
  // Using for...of for values
  calculateTotalPrice(products: Product[]): number {
    let total = 0;
    for (const product of products) {
      if (product.inStock) {
        total += product.price;
      }
    }
    return total;
  }
  
  // Using for...of with destructuring and filtering
  getCategoryCounts(products: Product[]): Map<string, number> {
    const categoryCounts = new Map<string, number>();
    
    for (const { categories } of products) {
      for (const category of categories) {
        categoryCounts.set(
          category, 
          (categoryCounts.get(category) || 0) + 1
        );
      }
    }
    
    return categoryCounts;
  }
  
  // Using for...in for object manipulation
  convertToRecords(products: Product[]): Record<number, Product> {
    const productRecord: Record<number, Product> = {};
    
    for (const product of products) {
      productRecord[product.id] = product;
    }
    
    // Demo of for...in loop to access the record
    for (const idString in productRecord) {
      // Type guard because for...in always gives string keys
      const id = Number(idString);
      console.log(`Product ${id}: ${productRecord[id].name}`);
    }
    
    return productRecord;
  }
}
```

**Common Pitfalls:**
- Using `for...in` on arrays instead of `for...of` (for...in iterates indices as strings)
- Not handling the string-to-number conversion when using `for...in` with numeric keys
- Not checking `hasOwnProperty` when iterating with `for...in`, potentially accessing prototype properties
- Using `var` in loop declarations, which doesn't have block scope like `let`
- Creating closures inside loops without proper variable scoping

**Three Common Questions:**

1. **Q: When should I use for...of versus for...in?**  
   **A:** Use `for...of` to iterate over the values in an array or other iterable (strings, Maps, Sets). Use `for...in` only when you need to iterate over the enumerable property names (keys) of an object. Never use `for...in` with arrays unless you specifically need the indices as strings and understand its drawbacks.

2. **Q: Why doesn't TypeScript fully type check properties accessed with for...in?**  
   **A:** `for...in` loops iterate over strings (property names), but TypeScript can't know which property exists at compile time. You need to use a type assertion (`key as keyof typeof obj`) or an index signature (`{ [key: string]: any }`) to access properties safely.

3. **Q: How can I break out of nested loops efficiently?**  
   **A:** JavaScript doesn't have a built-in way to break from multiple loops directly. You can: 1) Use labeled statements with `break labelName;`, 2) Extract the nested loops into a function and use `return` to exit both loops, or 3) Use a boolean flag to signal when to exit both loops.

#### 3.4 Control Flow Analysis

**Concise Explanation:**  
Control flow analysis is how TypeScript tracks a variable's type as it changes through different code paths. This analysis enables type narrowing in conditional branches, eliminates impossible code paths, and helps catch potential runtime errors during compilation.

**Where to Use:**
- Type narrowing in conditional blocks
- Exhaustiveness checking in switches
- Definite assignment checking
- Unreachable code detection
- Complex type guard scenarios

**Code Snippet:**
```typescript
// Type narrowing - TypeScript tracks types through code paths
function process(value: string | number | null) {
  // Type guard narrows from string | number | null to string | number
  if (value === null) {
    return "No value provided";
  }
  
  // Type guard further narrows from string | number to string
  if (typeof value === "string") {
    return value.toUpperCase(); // TypeScript knows value is a string here
  }
  
  // TypeScript knows value must be a number here
  return value.toFixed(2);
}

// Multiple conditions with narrowing
function formatValue(value: unknown): string {
  if (typeof value === "string") {
    return value.trim();
  } else if (typeof value === "number") {
    return value.toFixed(2);
  } else if (value === null || value === undefined) {
    return "N/A";
  } else if (Array.isArray(value)) {
    return value.join(", ");
  } else if (typeof value === "object") {
    return JSON.stringify(value);
  } else {
    return String(value);
  }
}

// Definite assignment checking
function getUpperCase(condition: boolean) {
  let message: string; // Variable declared but not initialized
  
  if (condition) {
    message = "True";
  } else {
    message = "False";
  }
  
  // TypeScript knows message is definitely assigned here
  return message.toUpperCase();
}

// Exhaustiveness checking with discriminated unions
type Shape = 
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number }
  | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    case "rectangle":
      return shape.width * shape.height;
    default:
      // If we add a new shape type but forget to handle it,
      // TypeScript will catch it with this exhaustiveness check
      const _exhaustiveCheck: never = shape;
      throw new Error(`Unhandled shape: ${_exhaustiveCheck}`);
  }
}

// Control flow with early returns
function getDiscount(user: { isPremium?: boolean; points: number }): number {
  // Early return for premium users
  if (user.isPremium) {
    return 0.2; // 20% discount
  }
  
  // TypeScript knows user.isPremium is false or undefined here
  if (user.points > 1000) {
    return 0.1; // 10% discount
  }
  
  // TypeScript knows user.isPremium is false/undefined and points <= 1000
  return 0.05; // 5% default discount
}

// Narrowing with instanceof
class Customer {
  name: string;
  loyaltyYears: number;
  
  constructor(name: string, loyaltyYears: number) {
    this.name = name;
    this.loyaltyYears = loyaltyYears;
  }
  
  getDiscount() {
    return this.loyaltyYears * 0.01;
  }
}

class Employee {
  name: string;
  department: string;
  
  constructor(name: string, department: string) {
    this.name = name;
    this.department = department;
  }
  
  getDiscount() {
    return 0.25;
  }
}

function handlePerson(person: Customer | Employee) {
  console.log(`Processing ${person.name}`);
  
  if (person instanceof Customer) {
    // TypeScript knows person is Customer here
    console.log(`Loyalty years: ${person.loyaltyYears}`);
  } else {
    // TypeScript knows person is Employee here
    console.log(`Department: ${person.department}`);
  }
  
  // Both classes have getDiscount, so this is always safe
  return person.getDiscount();
}

// Control flow analysis with type predicates
interface Fish {
  swim(): void;
  name: string;
}

interface Bird {
  fly(): void;
  name: string;
}

// Type predicate function
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function takeCareOfPet(pet: Fish | Bird) {
  if (isFish(pet)) {
    // TypeScript knows pet is Fish here
    pet.swim();
  } else {
    // TypeScript knows pet is Bird here
    pet.fly();
  }
  
  console.log(`Feeding ${pet.name}`);
}
```

**Real-World Example:**  
A payment processing system that validates, processes, and analyzes different payment methods:

```typescript
// Define payment types with discriminated union
type PaymentMethod = 
  | { type: "credit_card"; cardNumber: string; expiry: string; cvv: string }
  | { type: "paypal"; email: string }
  | { type: "bank_transfer"; accountNumber: string; routingNumber: string }
  | { type: "crypto"; currency: string; walletAddress: string };

// Define possible payment statuses
type PaymentStatus = "pending" | "processing" | "completed" | "failed" | "refunded";

// Payment record
interface Payment {
  id: string;
  amount: number;
  method: PaymentMethod;
  status: PaymentStatus;
  timestamp: Date;
}

class PaymentProcessor {
  validatePayment(payment: Payment): boolean {
    // Validate common fields
    if (payment.amount <= 0) {
      console.error("Payment amount must be positive");
      return false;
    }
    
    // Type-specific validation with narrowing
    switch (payment.method.type) {
      case "credit_card":
        return this.validateCreditCard(payment.method);
      
      case "paypal":
        return this.validatePaypal(payment.method);
      
      case "bank_transfer":
        return this.validateBankTransfer(payment.method);
      
      case "crypto":
        return this.validateCrypto(payment.method);
      
      default:
        // Exhaustiveness check - TypeScript will error if we add a new payment type
        // but forget to handle it here
        const _exhaustiveCheck: never = payment.method;
        console.error(`Unhandled payment method: ${JSON.stringify(_exhaustiveCheck)}`);
        return false;
    }
  }
  
  processPayment(payment: Payment): PaymentStatus {
    // Validate first
    if (!this.validatePayment(payment)) {
      return "failed";
    }
    
    // Process based on payment method
    try {
      switch (payment.method.type) {
        case "credit_card":
          // Process credit card payment
          return this.processCreditCardPayment(payment) ? "completed" : "failed";
        
        case "paypal":
          // Early return pattern for async operations (simplified)
          if (this.checkPaypalFunds(payment.method.email, payment.amount)) {
            return "completed";
          }
          return "failed";
        
        case "bank_transfer":
          // Bank transfers are not immediate
          this.initiateBankTransfer(payment.method);
          return "processing";
        
        case "crypto":
          // Crypto payments need confirmation
          this.requestCryptoPayment(payment.method, payment.amount);
          return "pending";
      }
    } catch (error) {
      console.error("Payment processing error:", error);
      return "failed";
    }
  }
  
  // Type-specific validation methods with appropriate narrowing
  private validateCreditCard(method: { cardNumber: string; expiry: string; cvv: string }): boolean {
    return method.cardNumber.length >= 13 && 
           /^\d{2}\/\d{2}$/.test(method.expiry) && 
           /^\d{3,4}$/.test(method.cvv);
  }
  
  private validatePaypal(method: { email: string }): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(method.email);
  }
  
  private validateBankTransfer(method: { accountNumber: string; routingNumber: string }): boolean {
    return method.accountNumber.length > 5 && method.routingNumber.length > 5;
  }
  
  private validateCrypto(method: { currency: string; walletAddress: string }): boolean {
    const supportedCurrencies = ["BTC", "ETH", "SOL", "USDC"];
    return supportedCurrencies.includes(method.currency) && method.walletAddress.length > 10;
  }
  
  // Payment processing methods (implementation simplified)
  private processCreditCardPayment(payment: Payment): boolean {
    // Implementation omitted
    return true;
  }
  
  private checkPaypalFunds(email: string, amount: number): boolean {
    // Implementation omitted
    return true;
  }
  
  private initiateBankTransfer(method: { accountNumber: string; routingNumber: string }): void {
    // Implementation omitted
  }
  
  private requestCryptoPayment(method: { currency: string; walletAddress: string }, amount: number): void {
    // Implementation omitted
  }
}
```

**Common Pitfalls:**
- Not handling all possible branches in discriminated unions, causing TypeScript to miss narrowing
- Over-relying on runtime checks that TypeScript can't understand for type narrowing
- Reassigning variables after they've been narrowed, causing TypeScript to widen the type again
- Using non-type-safe operations that bypass TypeScript's control flow analysis
- Not understanding that control flow analysis is limited to the current function scope

**Three Common Questions:**

1. **Q: Why does TypeScript sometimes forget the narrowed type after a function call?**  
   **A:** TypeScript's control flow analysis is limited to the current function scope. When you call a function, TypeScript assumes the function might modify variables or have side effects that could invalidate the narrowing. To preserve narrowing across function calls, store the narrowed value in a new variable before the call.

2. **Q: How can I ensure TypeScript checks that I've handled all possible types in a union?**  
   **A:** Use a discriminated union (a union with a common "tag" property like `kind` or `type`) and exhaustiveness checking. In the default case of a switch statement, assign the remaining value to a variable of type `never`: `const _exhaustiveCheck: never = remainingValue;`. If you add a new type to the union but forget to handle it, TypeScript will generate a compile-time error.

3. **Q: How do custom type guards actually work with TypeScript's control flow?**  
   **A:** Custom type guards are functions with a special return type called a "type predicate" using the `is` keyword: `function isString(val: any): val is string`. When you use this function in a condition, TypeScript trusts that your function correctly identifies the type and applies the narrowing accordingly. The type predicate tells TypeScript what type to narrow to after a successful check.

## Next Actions: Exercises and Projects

### Micro-Project 1: Type-Safe Form Validator
Build a simple form validation library:
1. Define interfaces for different form field types (text, number, email, etc.)
2. Create validation rules using literal types and type guards
3. Implement validation functions that properly handle and narrow types
4. Build a form submission function that validates all fields and returns appropriate errors
5. Add custom error messages for each validation rule

### Micro-Project 2: Smart Data Transformer
Create a utility that processes data with various types:
1. Build functions that can handle multiple data types (string, number, boolean, arrays)
2. Use type narrowing to process each type appropriately
3. Implement transformation functions (e.g., converting formats, filtering, etc.)
4. Add exhaustive type checking to ensure all possible inputs are handled
5. Create immutable data structures using readonly modifiers and const assertions

### Micro-Project 3: Mini State Machine
Build a simple state machine that:
1. Uses literal types to define possible states and transitions
2. Validates state transitions using control flow analysis
3. Implements proper type narrowing for each state
4. Uses discriminated unions for different state data
5. Provides type-safe actions that work with the current state

## Success Criteria

You've mastered Module 2 concepts when you can:
1. Correctly choose between variable declaration types and understand their scoping rules
2. Apply type annotations only where necessary, leveraging TypeScript's inference
3. Use literal types and unions to create precise, type-safe APIs
4. Confidently handle nullable values with proper checks and optional chaining
5. Implement exhaustive type checking in switch statements and conditionals
6. Select the most appropriate loop construct for different scenarios
7. Create custom type guards that properly narrow types
8. Write code that eliminates dead code paths and unreachable states

## Troubleshooting Guide

### Type Issues
- **Problem**: TypeScript complains about possibly null/undefined values
  **Solution**: Use optional chaining (`?.`), nullish coalescing (`??`), or proper type guards (`if (value)`)

- **Problem**: Type narrowing doesn't persist after function calls
  **Solution**: Store narrowed values in a new variable before calling functions

### Variable Declaration Issues
- **Problem**: Access before declaration errors
  **Solution**: Initialize variables when declaring them or use proper type annotations with definite assignment

- **Problem**: Can't reassign const variables
  **Solution**: Use let for variables that need to change, const only for constants

### Control Flow Issues
- **Problem**: TypeScript doesn't recognize a custom type guard
  **Solution**: Use a type predicate with the `is` keyword in your return type

- **Problem**: Missing cases in a switch statement for a union type
  **Solution**: Add an exhaustiveness check with `never` type in the default case
