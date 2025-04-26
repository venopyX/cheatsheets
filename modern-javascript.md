# Modern JavaScript - Complete Practical Course

## Core Problem
JavaScript is an essential programming language for web development that allows you to create dynamic, interactive websites. The core challenge is learning how to effectively use JavaScript to manipulate web page content, respond to user actions, and handle asynchronous operations while understanding its unique characteristics.

## Key Concepts to Cover
1. Variables and Data Types
2. Operators and Expressions
3. Control Flow (Conditionals and Loops)
4. Functions and Scope
5. DOM Manipulation
6. Events and Event Handling
7. Arrays and Array Methods
8. Objects and Object Methods
9. ES6+ Features (Arrow Functions, Destructuring, Spread/Rest)
10. Promises and Asynchronous JavaScript
11. Async/Await

## 1. Variables and Data Types

### Concise Explanation
Variables in JavaScript are containers for storing data values. JavaScript has three ways to declare variables: `var`, `let`, and `const`, each with different scoping rules and reassignment capabilities.

### Where to Use
- Use `const` by default (for values that won't change)
- Use `let` when you need to reassign values
- Avoid `var` in modern code (legacy syntax with function scope)

### Code Snippet
```javascript
// Variable declarations
const username = "John"; // Cannot be reassigned
let score = 75;          // Can be reassigned
score = 80;              // Works fine

// Data types
const numberValue = 42;             // Number
const textValue = "Hello";          // String
const isActive = true;              // Boolean
const userSettings = null;          // Null
let futureData;                     // Undefined
const uniqueId = Symbol("id");      // Symbol
const bigNumber = 9007199254740991n; // BigInt

// Object and array (reference types)
const user = { name: "Alice", age: 25 };  // Object
const colors = ["red", "green", "blue"];  // Array
```

### Real-World Example
When building a shopping cart:
```javascript
// Items in cart won't be reassigned to a new array, but its contents change
const shoppingCart = [];

// Price might change during the session
let totalPrice = 0;

function addToCart(item) {
  shoppingCart.push(item);
  totalPrice += item.price;
  
  // Update the UI with current total
  document.getElementById("total").textContent = `$${totalPrice.toFixed(2)}`;
}
```

### Common Pitfalls
1. **Confusion about `const`**: While `const` prevents reassignment, it doesn't make objects or arrays immutable.
   ```javascript
   const user = { name: "John" };
   user.name = "Jane"; // This works! Only reassignment is prohibited
   user = {}; // Error: Assignment to constant variable
   ```

2. **Scope issues**: `let` and `const` are block-scoped while `var` is function-scoped.
   ```javascript
   if (true) {
     var x = 10; // Available outside the block
     let y = 20; // Only available inside the block
   }
   console.log(x); // 10
   console.log(y); // ReferenceError: y is not defined
   ```

3. **Temporal Dead Zone**: Trying to access `let` and `const` variables before declaration.
   ```javascript
   console.log(a); // undefined - hoisted but not initialized
   console.log(b); // ReferenceError - in the "temporal dead zone"
   
   var a = 1;
   let b = 2;
   ```

### Common Questions

**Q1: What's the difference between `null` and `undefined`?**
A1: `undefined` means a variable has been declared but not assigned a value, while `null` is an explicit assignment representing "no value." `typeof undefined` is "undefined" but `typeof null` is "object" (a historical JavaScript bug).

**Q2: When should I use `let` vs `const`?**
A2: Use `const` by default for any variable whose value will not be reassigned. Use `let` when you need to reassign values, like counters, accumulators, or values that change over time. Avoid `var` in modern code.

**Q3: Why does `const array = [1, 2, 3]; array.push(4);` work but `array = [1, 2]` doesn't?**
A3: `const` prevents reassignment of the variable (changing what the variable points to), but it doesn't make the contents immutable. Array methods modify the existing array rather than creating a new assignment.

## 2. Operators and Expressions

### Concise Explanation
Operators in JavaScript perform operations on variables and values. They include arithmetic operators, assignment operators, comparison operators, logical operators, and more specialized operators like the ternary operator.

### Where to Use
- Arithmetic operators for mathematical calculations
- Comparison operators in conditionals
- Logical operators for combining conditions
- Assignment operators for concise value assignment
- Ternary operators for simple conditional expressions

### Code Snippet
```javascript
// Arithmetic operators
const sum = 5 + 3;       // Addition: 8
const difference = 10 - 4; // Subtraction: 6
const product = 3 * 4;   // Multiplication: 12
const quotient = 20 / 5; // Division: 4
const remainder = 10 % 3; // Modulus (remainder): 1
const power = 2 ** 3;    // Exponentiation: 8

// Comparison operators
const isEqual = 5 === 5;        // Strict equality: true
const isNotEqual = 5 !== '5';   // Strict inequality: true
const isGreater = 10 > 5;       // Greater than: true
const isLessOrEqual = 5 <= 5;   // Less than or equal: true

// Logical operators
const andOperation = true && false; // Logical AND: false
const orOperation = true || false;  // Logical OR: true
const notOperation = !true;         // Logical NOT: false

// Assignment operators
let x = 10;
x += 5;  // Same as x = x + 5: 15
x *= 2;  // Same as x = x * 2: 30

// Ternary operator
const userType = age >= 18 ? "adult" : "minor";
```

### Real-World Example
Form validation for a signup page:
```javascript
function validateForm(username, password, confirmPassword) {
  // Check if fields are empty
  const isFormComplete = username && password && confirmPassword;
  
  // Check if passwords match
  const passwordsMatch = password === confirmPassword;
  
  // Check password length
  const isPasswordValid = password.length >= 8;
  
  // Return validation message
  return isFormComplete && passwordsMatch && isPasswordValid
    ? { valid: true, message: "Signup successful!" }
    : { 
        valid: false, 
        message: !isFormComplete ? "All fields are required" :
                 !passwordsMatch ? "Passwords don't match" :
                 "Password must be at least 8 characters"
      };
}

const result = validateForm("john_doe", "pass123", "pass123");
document.getElementById("message").textContent = result.message;
```

### Common Pitfalls
1. **Using `==` instead of `===`**: The loose equality operator performs type coercion.
   ```javascript
   "5" == 5;   // true - types are coerced
   "5" === 5;  // false - different types
   ```

2. **Short-circuit evaluation surprises**: Logical operators return values, not just boolean.
   ```javascript
   const username = user && user.name; // If user exists, get user.name, else user (which is falsy)
   const defaultName = username || "Guest"; // Use username if truthy, otherwise "Guest"
   ```

3. **Unary plus/minus confusion**: Using `+` to convert strings to numbers.
   ```javascript
   const strNum = "42";
   const num = +strNum; // 42 as a number, not a string
   const negative = -num; // -42
   ```

### Common Questions

**Q1: What's the difference between `==` and `===`?**
A1: `==` is loose equality that performs type coercion, comparing values after converting them to a common type. `===` is strict equality that requires both value and type to be identical. Always prefer `===` to avoid unexpected type conversion bugs.

**Q2: Why does `0 == false` evaluate to `true`?**
A2: The loose equality `==` performs type coercion. When comparing a boolean with a non-boolean, JavaScript converts the boolean to a number (`false` becomes `0`). Then it compares `0 == 0`, which is `true`.

**Q3: What does the `??` operator do and when should I use it?**
A3: The nullish coalescing operator (`??`) returns the right-hand operand when the left-hand operand is `null` or `undefined`. Unlike `||`, it doesn't check for falsiness, which makes it useful for values like `0` or empty strings that might be valid but evaluate as falsy.

## 3. Control Flow (Conditionals and Loops)

### Concise Explanation
Control flow statements determine the order in which code executes. Conditionals (`if`, `else`, `switch`) execute code based on conditions, while loops (`for`, `while`, `do-while`) execute code repeatedly based on a condition.

### Where to Use
- `if/else` for simple branching logic
- `switch` for multiple condition branches with the same variable
- `for` loops when you know the number of iterations
- `while/do-while` loops when iterations depend on a condition
- Array methods like `forEach`, `map`, `filter` for iterating over arrays

### Code Snippet
```javascript
// If-else statement
const hour = new Date().getHours();

if (hour < 12) {
  console.log("Good morning");
} else if (hour < 18) {
  console.log("Good afternoon");
} else {
  console.log("Good evening");
}

// Switch statement
const day = new Date().getDay();

switch (day) {
  case 0:
    console.log("Sunday");
    break;
  case 6:
    console.log("Saturday");
    break;
  default:
    console.log("Weekday");
    break;
}

// For loop
for (let i = 0; i < 5; i++) {
  console.log(`Iteration ${i}`);
}

// For...of loop (iterates over array values)
const fruits = ["apple", "banana", "orange"];
for (const fruit of fruits) {
  console.log(fruit);
}

// For...in loop (iterates over object keys)
const user = { name: "John", age: 30 };
for (const key in user) {
  console.log(`${key}: ${user[key]}`);
}

// While loop
let count = 0;
while (count < 5) {
  console.log(`Count: ${count}`);
  count++;
}

// Do-while loop (always executes at least once)
let i = 0;
do {
  console.log(`i is ${i}`);
  i++;
} while (i < 3);
```

### Real-World Example
Building a simple quiz application:
```javascript
function runQuiz(questions) {
  let score = 0;
  
  // Loop through each question
  for (let i = 0; i < questions.length; i++) {
    const question = questions[i];
    const userAnswer = prompt(question.text);
    
    // Check if answer is correct
    if (userAnswer.toLowerCase() === question.answer.toLowerCase()) {
      score++;
      alert("Correct!");
    } else {
      alert(`Sorry, the correct answer was: ${question.answer}`);
    }
  }
  
  // Determine result message based on score
  let message;
  switch (true) {
    case (score === questions.length):
      message = "Perfect score! Excellent!";
      break;
    case (score >= questions.length * 0.7):
      message = "Great job! You did well!";
      break;
    case (score >= questions.length * 0.5):
      message = "Good effort. Room for improvement.";
      break;
    default:
      message = "Keep studying and try again.";
  }
  
  return `You scored ${score} out of ${questions.length}. ${message}`;
}

const quizQuestions = [
  { text: "What is the capital of France?", answer: "Paris" },
  { text: "What is 2 + 2?", answer: "4" },
  { text: "Who wrote Hamlet?", answer: "Shakespeare" }
];

document.getElementById("result").textContent = runQuiz(quizQuestions);
```

### Common Pitfalls
1. **Forgetting `break` in `switch` statements**: Without `break`, execution "falls through" to the next case.
   ```javascript
   switch (value) {
     case 1:
       console.log("One");
       // Missing break - execution will fall through!
     case 2:
       console.log("Two");
       break;
   }
   // If value is 1, both "One" and "Two" will be logged
   ```

2. **Infinite loops**: Forgetting to update the loop condition.
   ```javascript
   let i = 0;
   while (i < 10) {
     console.log(i);
     // Infinite loop! i never changes
   }
   ```

3. **Off-by-one errors**: Incorrect loop boundaries.
   ```javascript
   const arr = [1, 2, 3];
   // Incorrect: Accessing index out of bounds (arr[3] is undefined)
   for (let i = 0; i <= arr.length; i++) {
     console.log(arr[i]); 
   }
   ```

### Common Questions

**Q1: When should I use a `for...of` loop versus a traditional `for` loop?**
A1: Use `for...of` when you need to iterate over all values in an array or iterable and don't need the index. Use a traditional `for` loop when you need the index, need to skip elements, or want more control over iteration boundaries.

**Q2: What's the difference between `for...in` and `for...of` loops?**
A2: `for...in` iterates over all enumerable property keys of an object (including inherited ones) and is primarily for objects. `for...of` iterates over values of iterable objects (Arrays, Strings, Maps, Sets, etc.) and is not for regular objects.

**Q3: Why would I use a `switch` statement instead of multiple `if/else` statements?**
A3: Use `switch` when comparing a single variable against multiple possible values for better readability and potentially better performance. Use `if/else` chains when conditions are more complex or involve different variables.

## 4. Functions and Scope

### Concise Explanation
Functions are reusable blocks of code that perform specific tasks. JavaScript has multiple ways to define functions, each with different characteristics. Scope defines where variables are accessible within your code.

### Where to Use
- Function declarations for general-purpose, hoisted functions
- Function expressions for callbacks or non-hoisted scenarios
- Arrow functions for concise syntax and lexical `this` binding
- Higher-order functions for functional programming patterns

### Code Snippet
```javascript
// Function declaration (hoisted)
function greet(name) {
  return `Hello, ${name}!`;
}

// Function expression (not hoisted)
const multiply = function(a, b) {
  return a * b;
};

// Arrow function (concise, lexical this)
const add = (a, b) => a + b;

// Default parameters
function createUser(name, role = "user") {
  return { name, role };
}

// Rest parameters
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

// Higher-order function (returns a function)
function createMultiplier(factor) {
  return function(number) {
    return number * factor;
  };
}
const double = createMultiplier(2);
console.log(double(5)); // 10
```

### Real-World Example
Building a simple calculator with different operations:
```javascript
// Create a calculator object with different operations
const calculator = {
  // Store the value
  value: 0,
  
  // Method to reset the value
  clear() {
    this.value = 0;
    return this;
  },
  
  // Methods for basic operations
  add(num) {
    this.value += num;
    return this;
  },
  
  subtract(num) {
    this.value -= num;
    return this;
  },
  
  multiply(num) {
    this.value *= num;
    return this;
  },
  
  divide(num) {
    if (num === 0) {
      throw new Error("Cannot divide by zero");
    }
    this.value /= num;
    return this;
  },
  
  // Method to get the current value
  getValue() {
    return this.value;
  }
};

// Use the calculator with method chaining
const result = calculator
  .clear()
  .add(10)
  .multiply(2)
  .subtract(5)
  .divide(3)
  .getValue();

console.log(result); // 5
```

### Common Pitfalls
1. **Forgetting `return` statements**: Functions without explicit returns will return `undefined`.
   ```javascript
   function add(a, b) {
     a + b; // No return! Function returns undefined
   }
   ```

2. **The `this` keyword in different contexts**:
   ```javascript
   const user = {
     name: "John",
     // Method uses 'this' to refer to the object
     greet() {
       console.log(`Hello, I'm ${this.name}`);
     },
     // Regular function loses 'this' context
     delayedGreet() {
       setTimeout(function() {
         console.log(`Hello, I'm ${this.name}`); // 'this' is window or undefined
       }, 1000);
     },
     // Arrow function preserves 'this' context
     delayedGreetArrow() {
       setTimeout(() => {
         console.log(`Hello, I'm ${this.name}`); // 'this' is still the user object
       }, 1000);
     }
   };
   ```

3. **Variable scope issues**:
   ```javascript
   let globalVar = "I'm global";
   
   function outerFunc() {
     let outerVar = "I'm from outer";
     
     function innerFunc() {
       let innerVar = "I'm inner";
       console.log(globalVar); // I'm global
       console.log(outerVar);  // I'm from outer
       console.log(innerVar);  // I'm inner
     }
     
     innerFunc();
     console.log(innerVar); // ReferenceError: innerVar is not defined
   }
   ```

### Common Questions

**Q1: What's the difference between function declarations and function expressions?**
A1: Function declarations are hoisted (available before their actual declaration in code) and can be called anywhere in their scope. Function expressions are not hoisted and must be defined before they're called. Function declarations start with the `function` keyword at the beginning of a statement, while function expressions assign a function to a variable.

**Q2: When should I use arrow functions versus regular functions?**
A2: Use arrow functions for short callbacks, when you want to preserve the parent `this` context, or for concise one-line functions. Use regular functions when you need to use `this` to refer to the function's own context, when you need the `arguments` object, or when you need to use the `new` keyword to create instances.

**Q3: What is closure in JavaScript and why is it useful?**
A3: A closure is a function that remembers its outer variables and can access them even when executed outside its original scope. Closures are useful for data encapsulation, creating private variables, and for preserving state in asynchronous operations. They're the foundation for many JavaScript patterns like the module pattern.

## 5. DOM Manipulation

### Concise Explanation
DOM (Document Object Model) manipulation allows JavaScript to interact with HTML elements on a page. It enables you to access, modify, create, and delete elements and their content, attributes, and styles.

### Where to Use
- Responding to user interactions
- Updating page content dynamically
- Creating interactive UI components
- Form validation and handling
- Animations and visual effects

### Code Snippet
```javascript
// Selecting elements
const heading = document.getElementById("main-heading");
const paragraphs = document.querySelectorAll("p");
const navItems = document.getElementsByClassName("nav-item");
const form = document.querySelector("form#contact");

// Modifying content
heading.textContent = "Updated Heading"; // Text only
heading.innerHTML = "Updated <em>Heading</em>"; // HTML content

// Modifying attributes
const link = document.querySelector("a");
link.href = "https://example.com";
link.setAttribute("target", "_blank");
console.log(link.getAttribute("href"));
link.removeAttribute("title");

// Modifying styles
const box = document.querySelector(".box");
box.style.backgroundColor = "blue";
box.style.padding = "20px";
box.style.width = "100px";

// Modifying classes
box.classList.add("highlight");
box.classList.remove("old-class");
box.classList.toggle("visible");
box.classList.replace("old", "new");

// Creating elements
const newParagraph = document.createElement("p");
newParagraph.textContent = "This is a new paragraph";
document.body.appendChild(newParagraph);

// Inserting elements
const container = document.querySelector(".container");
container.prepend(newParagraph); // Insert at beginning
container.append(newParagraph); // Insert at end
container.before(newParagraph); // Insert before
container.after(newParagraph); // Insert after

// Removing elements
const oldElement = document.querySelector(".old-element");
oldElement.remove();

// Event handling (simple)
const button = document.querySelector("button");
button.addEventListener("click", function() {
  alert("Button was clicked!");
});
```

### Real-World Example
Building a dynamic to-do list:
```javascript
// Get the elements we need
const todoForm = document.getElementById("todo-form");
const todoInput = document.getElementById("todo-input");
const todoList = document.getElementById("todo-list");

// Function to create a new todo item
function createTodoItem(text) {
  // Create the list item
  const li = document.createElement("li");
  li.className = "todo-item";
  
  // Add the text
  const span = document.createElement("span");
  span.textContent = text;
  li.appendChild(span);
  
  // Add a delete button
  const deleteBtn = document.createElement("button");
  deleteBtn.textContent = "Delete";
  deleteBtn.className = "delete-btn";
  li.appendChild(deleteBtn);
  
  // Add a complete button
  const completeBtn = document.createElement("button");
  completeBtn.textContent = "Complete";
  completeBtn.className = "complete-btn";
  li.appendChild(completeBtn);
  
  return li;
}

// Event listener for form submission
todoForm.addEventListener("submit", function(e) {
  // Prevent the default form submission
  e.preventDefault();
  
  // Get the input value and trim whitespace
  const todoText = todoInput.value.trim();
  
  // Only add non-empty todos
  if (todoText !== "") {
    // Create and add the todo item
    const todoItem = createTodoItem(todoText);
    todoList.appendChild(todoItem);
    
    // Clear the input
    todoInput.value = "";
  }
});

// Event delegation for handling clicks on todo list items
todoList.addEventListener("click", function(e) {
  const target = e.target;
  
  // Handle delete button
  if (target.classList.contains("delete-btn")) {
    const todoItem = target.parentElement;
    todoItem.remove();
  }
  
  // Handle complete button
  if (target.classList.contains("complete-btn")) {
    const todoItem = target.parentElement;
    todoItem.classList.toggle("completed");
  }
});
```

### Common Pitfalls
1. **Direct HTML manipulation with `innerHTML` can be a security risk**:
   ```javascript
   // Vulnerable to XSS attacks if userInput contains malicious code
   element.innerHTML = userInput;
   
   // Safer alternatives:
   element.textContent = userInput; // For text
   // Or for complex HTML, create elements and append them
   ```

2. **Inefficient DOM updates**:
   ```javascript
   // Inefficient: modifies the DOM in each iteration
   for (let i = 0; i < 100; i++) {
     container.innerHTML += `<div>${i}</div>`;
   }
   
   // Better: build up content and update once
   let content = '';
   for (let i = 0; i < 100; i++) {
     content += `<div>${i}</div>`;
   }
   container.innerHTML = content;
   
   // Or best: use DocumentFragment
   const fragment = document.createDocumentFragment();
   for (let i = 0; i < 100; i++) {
     const div = document.createElement('div');
     div.textContent = i;
     fragment.appendChild(div);
   }
   container.appendChild(fragment);
   ```

3. **Not checking if elements exist before manipulating them**:
   ```javascript
   // This might throw an error if element doesn't exist
   document.getElementById("non-existent").style.color = "red";
   
   // Better approach
   const element = document.getElementById("possibly-exists");
   if (element) {
     element.style.color = "red";
   }
   ```

### Common Questions

**Q1: What's the difference between `textContent`, `innerText`, and `innerHTML`?**
A1: `textContent` gets/sets the text content of all elements, including script and style elements, and ignores CSS styling. `innerText` is aware of CSS styling and only returns visible text. `innerHTML` gets/sets the HTML content, parsing strings as HTML (which can pose security risks with user input).

**Q2: How do I efficiently update multiple elements in the DOM?**
A2: Use document fragments to build up changes off-DOM before applying them once, use CSS classes for style changes instead of manipulating inline styles, and batch your DOM operations together to minimize reflows and repaints.

**Q3: What's the best way to remove all child elements from a parent element?**
A3: The most efficient way is to use `parentElement.innerHTML = ''`, but this has event handler implications. For more control, you can use a loop: `while (parentElement.firstChild) { parentElement.removeChild(parentElement.firstChild); }`. Modern browsers also support `parentElement.replaceChildren()`.

## 6. Events and Event Handling

### Concise Explanation
Events in JavaScript are actions or occurrences (like clicks, key presses, or form submissions) that can be detected and responded to with code. Event handling is the process of writing code that responds to these events.

### Where to Use
- User interactions (clicks, key presses, form submissions)
- Browser events (page load, resize, scroll)
- Custom events for component communication
- API responses and asynchronous operations

### Code Snippet
```javascript
// Basic event handling with addEventListener
const button = document.querySelector("#myButton");
button.addEventListener("click", function(event) {
  console.log("Button was clicked!");
  console.log(event); // The event object contains useful information
});

// Common event types
document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM fully loaded");
});

window.addEventListener("load", () => {
  console.log("Page fully loaded, including images");
});

document.addEventListener("keydown", (e) => {
  console.log(`Key pressed: ${e.key}`);
});

const input = document.querySelector("input");
input.addEventListener("focus", () => {
  console.log("Input focused");
});
input.addEventListener("blur", () => {
  console.log("Input lost focus");
});

// Event delegation (handling events for multiple elements)
const list = document.querySelector("#myList");
list.addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log(`Clicked on: ${e.target.textContent}`);
  }
});

// Preventing default behavior
const form = document.querySelector("form");
form.addEventListener("submit", (e) => {
  e.preventDefault(); // Stop form from submitting
  console.log("Form submission prevented");
});

// Stopping event propagation
const inner = document.querySelector(".inner");
inner.addEventListener("click", (e) => {
  e.stopPropagation(); // Stop event from bubbling up
  console.log("Inner clicked");
});

// Event once option
button.addEventListener("click", handler, { once: true }); // Only triggers once

// Custom events
const customEvent = new CustomEvent("userAction", {
  detail: { name: "John", action: "login" }
});
document.dispatchEvent(customEvent);

// Listen for custom event
document.addEventListener("userAction", (e) => {
  console.log(`User ${e.detail.name} performed ${e.detail.action}`);
});
```

### Real-World Example
Creating an interactive image gallery with event handling:
```javascript
const gallery = document.querySelector(".gallery");
const mainImage = document.querySelector(".main-image");
const caption = document.querySelector(".caption");
const thumbnails = document.querySelector(".thumbnails");

// Image data
const images = [
  { src: "image1.jpg", alt: "Nature landscape", title: "Beautiful mountains" },
  { src: "image2.jpg", alt: "City skyline", title: "New York at night" },
  { src: "image3.jpg", alt: "Beach sunset", title: "Tropical paradise" }
];

// Initialize gallery
function initGallery() {
  // Create thumbnails
  images.forEach((image, index) => {
    const thumb = document.createElement("img");
    thumb.src = `thumbnails/${image.src}`;
    thumb.alt = image.alt;
    thumb.dataset.index = index;
    thumb.className = "thumbnail";
    thumbnails.appendChild(thumb);
  });
  
  // Set initial main image
  setMainImage(0);
  
  // Add event listeners
  thumbnails.addEventListener("click", handleThumbnailClick);
  mainImage.addEventListener("click", openFullscreen);
  
  // Keyboard navigation
  document.addEventListener("keydown", handleKeyNavigation);
}

// Handle thumbnail clicks using event delegation
function handleThumbnailClick(e) {
  if (e.target.classList.contains("thumbnail")) {
    const index = parseInt(e.target.dataset.index);
    setMainImage(index);
    
    // Update active thumbnail
    document.querySelectorAll(".thumbnail").forEach(thumb => {
      thumb.classList.remove("active");
    });
    e.target.classList.add("active");
  }
}

// Set the main image and caption
function setMainImage(index) {
  const image = images[index];
  mainImage.src = image.src;
  mainImage.alt = image.alt;
  caption.textContent = image.title;
  mainImage.dataset.index = index;
}

// Open image in fullscreen mode
function openFullscreen() {
  if (mainImage.requestFullscreen) {
    mainImage.requestFullscreen();
  } else if (mainImage.mozRequestFullScreen) { /* Firefox */
    mainImage.mozRequestFullScreen();
  } else if (mainImage.webkitRequestFullscreen) { /* Chrome, Safari */
    mainImage.webkitRequestFullscreen();
  } else if (mainImage.msRequestFullscreen) { /* IE/Edge */
    mainImage.msRequestFullscreen();
  }
}

// Handle keyboard navigation
function handleKeyNavigation(e) {
  const currentIndex = parseInt(mainImage.dataset.index);
  let newIndex;
  
  if (e.key === "ArrowRight") {
    newIndex = (currentIndex + 1) % images.length;
    setMainImage(newIndex);
  } else if (e.key === "ArrowLeft") {
    newIndex = (currentIndex - 1 + images.length) % images.length;
    setMainImage(newIndex);
  }
  
  // Update active thumbnail if changed
  if (newIndex !== undefined) {
    document.querySelectorAll(".thumbnail").forEach(thumb => {
      thumb.classList.remove("active");
    });
    document.querySelector(`.thumbnail[data-index="${newIndex}"]`).classList.add("active");
  }
}

// Initialize the gallery when the DOM is loaded
document.addEventListener("DOMContentLoaded", initGallery);
```

### Common Pitfalls
1. **Memory leaks from unremoved event listeners**:
   ```javascript
   // Example of a potential memory leak
   function addHandlers() {
     const button = document.querySelector("button");
     // Creating a new function reference each time
     button.addEventListener("click", function() {
       console.log("Clicked");
     });
   }
   
   // Called repeatedly without removing previous listeners
   setInterval(addHandlers, 1000);
   
   // Better approach: store and remove reference
   function setupHandlers() {
     const button = document.querySelector("button");
     const handler = function() { console.log("Clicked"); };
     
     button.addEventListener("click", handler);
     
     // When needed:
     // button.removeEventListener("click", handler);
   }
   ```

2. **Event bubbling/capturing confusion**:
   ```javascript
   // Bubbling (default): events propagate from target up to parents
   parent.addEventListener("click", () => console.log("Parent clicked"));
   child.addEventListener("click", () => console.log("Child clicked"));
   // Clicking child prints: "Child clicked" then "Parent clicked"
   
   // Capturing: events propagate from top down to target
   parent.addEventListener("click", () => console.log("Parent clicked"), true);
   child.addEventListener("click", () => console.log("Child clicked"), true);
   // Clicking child prints: "Parent clicked" then "Child clicked"
   ```

3. **Overusing direct event handlers instead of delegation**:
   ```javascript
   // Inefficient: adding handlers to each item
   document.querySelectorAll("li").forEach(item => {
     item.addEventListener("click", handleClick);
   });
   
   // Better: use event delegation
   document.querySelector("ul").addEventListener("click", (e) => {
     if (e.target.tagName === "LI") {
       handleClick(e);
     }
   });
   ```

### Common Questions

**Q1: What is event delegation and why is it useful?**
A1: Event delegation is a technique where you attach a single event listener to a parent element to handle events for multiple child elements (even future ones). It's useful because it reduces the number of event listeners (improving performance), works for dynamically added elements, and reduces memory usage.

**Q2: What is the difference between event bubbling and event capturing?**
A2: Event bubbling (the default) is when an event starts at the target element and bubbles up through its ancestors. Event capturing is the opposite—events start at the top ancestor and move down to the target. You can set the third parameter of `addEventListener` to `true` to use the capturing phase instead of bubbling.

**Q3: How can I remove an event listener?**
A3: Use `removeEventListener()` with the same event type and function reference that was used with `addEventListener()`. This is why it's important to use named functions or store function references when adding event listeners that you'll need to remove later.

## 7. Arrays and Array Methods

### Concise Explanation
Arrays are ordered collections of values in JavaScript. JavaScript provides powerful built-in methods to manipulate, transform, and query arrays without writing custom loops.

### Where to Use
- Storing collections of related items
- Managing lists of data (users, products, etc.)
- Data processing and transformation
- Functional programming patterns

### Code Snippet
```javascript
// Creating arrays
const fruits = ["apple", "banana", "orange"];
const numbers = [1, 2, 3, 4, 5];
const mixed = [1, "two", { three: 3 }, [4]];
const newArray = new Array(3); // Creates array with 3 empty slots

// Accessing elements
const firstFruit = fruits[0]; // "apple"
const lastNumber = numbers[numbers.length - 1]; // 5

// Modifying arrays
fruits.push("grape"); // Add to end: ["apple", "banana", "orange", "grape"]
fruits.pop(); // Remove from end: ["apple", "banana", "orange"]
fruits.unshift("kiwi"); // Add to beginning: ["kiwi", "apple", "banana", "orange"]
fruits.shift(); // Remove from beginning: ["apple", "banana", "orange"]
fruits.splice(1, 1, "mango"); // Replace: ["apple", "mango", "orange"]

// Finding elements
const bananaIndex = fruits.indexOf("banana"); // 1
const hasApple = fruits.includes("apple"); // true

// Transformation methods
// map - transform each element
const doubled = numbers.map(num => num * 2); // [2, 4, 6, 8, 10]

// filter - create a new array with elements passing a test
const evenNumbers = numbers.filter(num => num % 2 === 0); // [2, 4]

// reduce - accumulate values
const sum = numbers.reduce((total, num) => total + num, 0); // 15

// find - get first element matching a condition
const firstEven = numbers.find(num => num % 2 === 0); // 2

// findIndex - get index of first element matching a condition
const firstEvenIndex = numbers.findIndex(num => num % 2 === 0); // 1

// Iteration methods
// forEach - execute function for each element
numbers.forEach(num => console.log(num));

// some - check if at least one element passes test
const hasEven = numbers.some(num => num % 2 === 0); // true

// every - check if all elements pass test
const allEven = numbers.every(num => num % 2 === 0); // false

// Array modifications
// sort - sort elements (caution: converts to strings by default)
fruits.sort(); // ["apple", "mango", "orange"]
numbers.sort((a, b) => a - b); // Proper numeric sort: [1, 2, 3, 4, 5]

// reverse - reverse array order
numbers.reverse(); // [5, 4, 3, 2, 1]

// join - create string from array
const fruitString = fruits.join(", "); // "apple, mango, orange"

// slice - get section of array
const subset = numbers.slice(1, 4); // [4, 3, 2]

// flat - flatten nested arrays
const nested = [1, [2, [3, 4]]];
const flattened = nested.flat(2); // [1, 2, 3, 4]
```

### Real-World Example
Building a product filtering and sorting system:
```javascript
// Product data
const products = [
  { id: 1, name: "Laptop", price: 999.99, category: "electronics", inStock: true },
  { id: 2, name: "Headphones", price: 119.99, category: "electronics", inStock: false },
  { id: 3, name: "Coffee Maker", price: 89.99, category: "appliances", inStock: true },
  { id: 4, name: "Running Shoes", price: 79.99, category: "clothing", inStock: true },
  { id: 5, name: "Blender", price: 49.99, category: "appliances", inStock: true }
];

// Filtering and sorting products based on user selection
function filterAndSortProducts(products, filters, sortBy, sortOrder) {
  // First apply all filters
  let filteredProducts = products.filter(product => {
    // Check each filter
    if (filters.category && product.category !== filters.category) {
      return false;
    }
    
    if (filters.inStock === true && !product.inStock) {
      return false;
    }
    
    if (filters.minPrice && product.price < filters.minPrice) {
      return false;
    }
    
    if (filters.maxPrice && product.price > filters.maxPrice) {
      return false;
    }
    
    if (filters.searchTerm && !product.name.toLowerCase().includes(filters.searchTerm.toLowerCase())) {
      return false;
    }
    
    // All filters passed
    return true;
  });
  
  // Then sort the results
  if (sortBy) {
    filteredProducts.sort((a, b) => {
      // Handle numeric or string sorting
      const aValue = a[sortBy];
      const bValue = b[sortBy];
      
      // Sort differently based on data type
      if (typeof aValue === 'string') {
        const comparison = aValue.localeCompare(bValue);
        return sortOrder === 'desc' ? -comparison : comparison;
      } else {
        // Numeric sort
        return sortOrder === 'desc' ? bValue - aValue : aValue - bValue;
      }
    });
  }
  
  return filteredProducts;
}

// Usage examples
// 1. Get in-stock electronics sorted by price (lowest first)
const inStockElectronics = filterAndSortProducts(
  products,
  { category: "electronics", inStock: true },
  "price",
  "asc"
);

// 2. Search for products containing "e" under $100
const searchResults = filterAndSortProducts(
  products,
  { searchTerm: "e", maxPrice: 100 },
  "price",
  "desc"
);

// 3. Get all products sorted alphabetically
const allSortedByName = filterAndSortProducts(
  products,
  {},
  "name",
  "asc"
);

// Display results
function displayProducts(products) {
  // Map to HTML elements
  const productElements = products.map(product => {
    return `
      <div class="product ${!product.inStock ? 'out-of-stock' : ''}">
        <h3>${product.name}</h3>
        <p>$${product.price.toFixed(2)}</p>
        <p>Category: ${product.category}</p>
        <p>${product.inStock ? 'In Stock' : 'Out of Stock'}</p>
      </div>
    `;
  });
  
  // Join and insert into DOM
  document.querySelector('.product-list').innerHTML = 
    productElements.length ? productElements.join('') : '<p>No products found</p>';
}

// Display initial results
displayProducts(allSortedByName);
```

### Common Pitfalls
1. **Mutation vs. non-mutation confusion**:
   ```javascript
   // These methods MODIFY the original array:
   array.push(), array.pop(), array.shift(), array.unshift(), 
   array.splice(), array.sort(), array.reverse()
   
   // These methods DO NOT modify the original array:
   array.map(), array.filter(), array.reduce(), array.concat(), 
   array.slice(), array.join()
   ```

2. **Default sort behavior converts to strings**:
   ```javascript
   // Wrong way to sort numbers
   [10, 5, 1, 20].sort(); // [1, 10, 20, 5] - sorted as strings!
   
   // Correct way
   [10, 5, 1, 20].sort((a, b) => a - b); // [1, 5, 10, 20]
   ```

3. **Array-like objects vs. true arrays**:
   ```javascript
   // Array-like object (e.g., DOM NodeList)
   const divs = document.querySelectorAll('div');
   
   // Won't work - NodeList doesn't have map method
   divs.map(div => div.textContent);
   
   // Solution 1: Convert to array first
   Array.from(divs).map(div => div.textContent);
   
   // Solution 2: Use Array.from with mapping function
   Array.from(divs, div => div.textContent);
   
   // Solution 3: Use spread syntax
   [...divs].map(div => div.textContent);
   ```

### Common Questions

**Q1: What's the difference between `map()` and `forEach()`?**
A1: `map()` creates and returns a new array with transformed elements, while `forEach()` simply executes a function on each element and returns `undefined`. Use `map()` when you need a new array based on transformations, and `forEach()` when you just need to execute some code for each element without creating a new array.

**Q2: What's the best way to remove duplicates from an array?**
A2: The most modern and concise way is using a Set: `const uniqueArray = [...new Set(originalArray)]`. This works well for primitive values. For objects or more complex comparisons, you might need `filter()` with a custom comparison function or use a map to track seen values.

**Q3: How do I find and remove an item from an array by its value?**
A3: To remove a single occurrence: `const index = array.indexOf(value); if (index > -1) array.splice(index, 1);` 
To remove all occurrences: `array = array.filter(item => item !== value);`
For objects, you would use a more specific comparison in `filter()` or `findIndex()`.

## 8. Objects and Object Methods

### Concise Explanation
Objects in JavaScript are collections of key-value pairs where values can be of any type, including functions (methods). They are the foundation of JavaScript's object-oriented capabilities and are used to store and organize related data and functionality.

### Where to Use
- Grouping related data and behaviors
- Creating reusable components and modules
- Implementing data models
- Organizing configuration settings
- Namespace management

### Code Snippet
```javascript
// Object literals
const user = {
  // Properties
  firstName: "John",
  lastName: "Doe",
  age: 30,
  
  // Methods
  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  
  // Nested object
  address: {
    street: "123 Main St",
    city: "Boston",
    zipCode: "02101"
  }
};

// Accessing properties
console.log(user.firstName); // "John"
console.log(user["lastName"]); // "Doe"

// Using methods
console.log(user.getFullName()); // "John Doe"

// Adding properties and methods
user.email = "john@example.com";
user.greet = function() {
  return `Hello, my name is ${this.firstName}`;
};

// Removing properties
delete user.age;

// Checking if property exists
console.log("email" in user); // true
console.log(user.hasOwnProperty("phone")); // false

// Object destructuring
const { firstName, lastName, address: { city } } = user;
console.log(`${firstName} lives in ${city}`); // "John lives in Boston"

// Object.keys, values, entries
console.log(Object.keys(user)); // ["firstName", "lastName", "address", "email", "greet"]
console.log(Object.values(user)); // [values of all properties]
console.log(Object.entries(user)); // [["firstName", "John"], ["lastName", "Doe"], ...]

// Copying objects
// Shallow copy
const userCopy = { ...user };
const anotherCopy = Object.assign({}, user);

// Deep copy (simple, but not perfect for all cases)
const deepCopy = JSON.parse(JSON.stringify(user));

// Object methods
// Object.freeze - make object immutable
const frozenObj = Object.freeze({ x: 1, y: 2 });
frozenObj.x = 10; // No effect in strict mode, silently fails otherwise

// Object.seal - prevents adding/removing properties, but allows modifying
const sealedObj = Object.seal({ x: 1, y: 2 });
sealedObj.x = 10; // Works
sealedObj.z = 3; // No effect in strict mode

// Object merging
const defaults = { theme: "light", notifications: true };
const userPrefs = { theme: "dark" };
const mergedPrefs = { ...defaults, ...userPrefs }; // { theme: "dark", notifications: true }
```

### Real-World Example
Building a simple expense tracker object:
```javascript
const expenseTracker = {
  // Private data (by convention)
  _expenses: [],
  _categories: ["Food", "Housing", "Transportation", "Entertainment", "Other"],
  _budget: 1000,
  
  // Add an expense
  addExpense(description, amount, category = "Other", date = new Date()) {
    // Validate inputs
    if (!description || description.trim() === "") {
      throw new Error("Description is required");
    }
    
    if (typeof amount !== "number" || amount <= 0) {
      throw new Error("Amount must be a positive number");
    }
    
    if (!this._categories.includes(category)) {
      throw new Error(`Invalid category. Choose from: ${this._categories.join(", ")}`);
    }
    
    // Create expense object with ID
    const expense = {
      id: Date.now() + Math.random().toString(16).slice(2),
      description,
      amount,
      category,
      date: new Date(date),
      createdAt: new Date()
    };
    
    // Add to expenses array
    this._expenses.push(expense);
    
    return expense.id;
  },
  
  // Remove an expense by ID
  removeExpense(id) {
    const initialLength = this._expenses.length;
    this._expenses = this._expenses.filter(expense => expense.id !== id);
    
    return this._expenses.length !== initialLength;
  },
  
  // Edit an expense
  editExpense(id, updates) {
    const expenseIndex = this._expenses.findIndex(expense => expense.id === id);
    
    if (expenseIndex === -1) {
      return false;
    }
    
    // Create a new object with the updates
    this._expenses[expenseIndex] = {
      ...this._expenses[expenseIndex],
      ...updates,
      // Prevent updating certain fields
      id: this._expenses[expenseIndex].id,
      createdAt: this._expenses[expenseIndex].createdAt,
      // Ensure date is a Date object
      date: updates.date ? new Date(updates.date) : this._expenses[expenseIndex].date
    };
    
    return true;
  },
  
  // Get all expenses
  getAllExpenses() {
    // Return a copy to prevent modification
    return [...this._expenses];
  },
  
  // Get expenses by category
  getExpensesByCategory(category) {
    return this._expenses.filter(expense => expense.category === category);
  },
  
  // Calculate total expenses
  getTotalExpenses() {
    return this._expenses.reduce((total, expense) => total + expense.amount, 0);
  },
  
  // Calculate remaining budget
  getRemainingBudget() {
    return this._budget - this.getTotalExpenses();
  },
  
  // Set budget
  setBudget(amount) {
    if (typeof amount !== "number" || amount < 0) {
      throw new Error("Budget must be a non-negative number");
    }
    
    this._budget = amount;
  },
  
  // Get expense summary by category
  getExpenseSummary() {
    const summary = {};
    
    // Initialize all categories to zero
    this._categories.forEach(category => {
      summary[category] = 0;
    });
    
    // Sum expenses by category
    this._expenses.forEach(expense => {
      summary[expense.category] += expense.amount;
    });
    
    return summary;
  }
};

// Usage example
expenseTracker.setBudget(2000);

expenseTracker.addExpense("Groceries", 75.50, "Food");
expenseTracker.addExpense("Movie tickets", 24.99, "Entertainment");
expenseTracker.addExpense("Gas", 45.00, "Transportation");

console.log("Total expenses:", expenseTracker.getTotalExpenses());
console.log("Remaining budget:", expenseTracker.getRemainingBudget());
console.log("Expense summary:", expenseTracker.getExpenseSummary());

// Edit an expense
const expenses = expenseTracker.getAllExpenses();
expenseTracker.editExpense(expenses[0].id, { amount: 85.75 });

// View updated total
console.log("Updated total:", expenseTracker.getTotalExpenses());
```

### Common Pitfalls
1. **The `this` keyword context confusion**:
   ```javascript
   const user = {
     name: "John",
     greet() {
       console.log(`Hello, ${this.name}`);
     },
     delayedGreet() {
       setTimeout(function() {
         // 'this' here refers to the global object, not the user object
         console.log(`Hello, ${this.name}`); // "Hello, undefined"
       }, 1000);
     },
     // Solutions:
     // 1. Use arrow function
     correctDelayedGreet1() {
       setTimeout(() => {
         console.log(`Hello, ${this.name}`); // "Hello, John"
       }, 1000);
     },
     // 2. Store this reference
     correctDelayedGreet2() {
       const self = this;
       setTimeout(function() {
         console.log(`Hello, ${self.name}`); // "Hello, John"
       }, 1000);
     },
     // 3. Use bind
     correctDelayedGreet3() {
       setTimeout(function() {
         console.log(`Hello, ${this.name}`); // "Hello, John"
       }.bind(this), 1000);
     }
   };
   ```

2. **Shallow vs. deep copying objects**:
   ```javascript
   const original = { 
     name: "Original", 
     details: { 
       id: 123, 
       settings: { color: "blue" } 
     } 
   };
   
   // Shallow copy - nested objects are still shared
   const shallowCopy = { ...original };
   shallowCopy.name = "Copy";
   shallowCopy.details.id = 456;
   
   console.log(original.name); // "Original"
   console.log(original.details.id); // 456 - changed because details is shared!
   
   // Deep copy options:
   // 1. JSON method (limitations: doesn't copy functions, Date objects properly)
   const jsonCopy = JSON.parse(JSON.stringify(original));
   
   // 2. Third-party libraries like lodash _.cloneDeep()
   
   // 3. Custom recursive function
   function deepClone(obj) {
     if (obj === null || typeof obj !== "object") return obj;
     
     const copy = Array.isArray(obj) ? [] : {};
     
     Object.keys(obj).forEach(key => {
       copy[key] = deepClone(obj[key]);
     });
     
     return copy;
   }
   ```

3. **Property access vs bracket notation confusion**:
   ```javascript
   const user = {
     name: "John",
     "user-id": 123  // Property with special characters
   };
   
   console.log(user.name);    // "John" - works
   console.log(user.user-id); // NaN - interpreted as user minus id!
   console.log(user["user-id"]); // 123 - correct way
   
   // Dynamic property access
   const propName = "name";
   console.log(user.propName); // undefined - looking for literal "propName"
   console.log(user[propName]); // "John" - using the value of propName variable
   ```

### Common Questions

**Q1: What's the difference between objects created with object literals and those created with constructors or classes?**
A1: Objects created with object literals are standalone instances. Objects created with constructors or classes can share methods through the prototype chain, which is more memory-efficient. Class/constructor patterns also provide a consistent way to create multiple objects with the same structure and behavior, while literals are typically used for one-off objects.

**Q2: How do I prevent an object from being modified?**
A2: Use `Object.freeze(obj)` to make all properties non-writable and non-configurable (though nested objects are not frozen). Use `Object.seal(obj)` to prevent adding/removing properties but allow modifying existing ones. For deep immutability, use recursive freezing or consider immutability libraries. Note that these are "shallow" by default and don't affect nested objects.

**Q3: What's the best way to check if a property exists on an object vs. its prototype chain?**
A3: Use `obj.hasOwnProperty('propName')` to check if a property exists directly on the object and not in its prototype chain. Use `'propName' in obj` to check if a property exists either on the object or its prototype chain. To check if a property exists and is not undefined, use `obj.propName !== undefined` or `Object.prototype.hasOwnProperty.call(obj, 'propName')` for safer checks.

## 9. ES6+ Features (Arrow Functions, Destructuring, Spread/Rest)

### Concise Explanation
ES6+ (ECMAScript 2015 and later) introduced many powerful features to JavaScript, making code more concise, readable, and expressive. These features include arrow functions, destructuring, template literals, spread/rest operators, default parameters, and more.

### Where to Use
- Arrow functions for concise callbacks and lexical `this`
- Destructuring for extracting data from objects and arrays
- Spread operator for array/object manipulation
- Template literals for string interpolation
- Classes for cleaner object-oriented code

### Code Snippet
```javascript
// Arrow functions
const add = (a, b) => a + b; // Implicit return
const square = x => x * x; // Single parameter doesn't need parentheses
const complex = (x, y) => {
  const sum = x + y;
  return sum * 2;
}; // Multiline with explicit return

// Arrow functions and this
const counter = {
  count: 0,
  // Traditional function loses 'this' context in callbacks
  incrementBad: function() {
    setTimeout(function() {
      this.count++; // 'this' refers to window/global, not counter
      console.log(this.count); // NaN or error
    }, 1000);
  },
  // Arrow function preserves 'this' context
  incrementGood: function() {
    setTimeout(() => {
      this.count++; // 'this' refers to counter object
      console.log(this.count); // 1
    }, 1000);
  }
};

// Destructuring objects
const user = {
  name: "John Doe",
  age: 30,
  address: {
    city: "New York",
    zip: "10001"
  }
};

// Basic destructuring
const { name, age } = user;
console.log(name); // "John Doe"

// Renaming variables
const { name: userName, age: userAge } = user;
console.log(userName); // "John Doe"

// Default values
const { country = "USA" } = user;
console.log(country); // "USA" (default value since it wasn't in user)

// Nested destructuring
const { address: { city, zip } } = user;
console.log(city); // "New York"

// Destructuring arrays
const colors = ["red", "green", "blue"];

// Basic array destructuring
const [firstColor, secondColor] = colors;
console.log(firstColor); // "red"

// Skip elements
const [, , thirdColor] = colors;
console.log(thirdColor); // "blue"

// Rest pattern
const [primary, ...secondaryColors] = colors;
console.log(secondaryColors); // ["green", "blue"]

// Default values
const [main = "black", accent = "white"] = [];
console.log(main, accent); // "black" "white"

// Spread operator for arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// Combine arrays
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Clone array
const clonedArr = [...arr1]; // [1, 2, 3]

// String to array
const chars = [..."hello"]; // ["h", "e", "l", "l", "o"]

// Spread operator for objects
const defaults = { theme: "light", fontSize: 12 };
const userPrefs = { theme: "dark" };

// Merge objects (last one overrides previous)
const settings = { ...defaults, ...userPrefs }; // { theme: "dark", fontSize: 12 }

// Clone object (shallow)
const clonedObj = { ...user };

// Rest parameter in functions
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15

// Template literals
const name2 = "Alice";
const greeting = `Hello, ${name2}!
Welcome to our website.`; // Multiline with interpolation

// Tagged template literals
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return `${result}${str}${values[i] ? `<mark>${values[i]}</mark>` : ''}`;
  }, '');
}

const name3 = "Alice";
const age2 = 30;
const highlighted = highlight`My name is ${name3} and I am ${age2} years old.`;
// "My name is <mark>Alice</mark> and I am <mark>30</mark> years old."
```

### Real-World Example
Building a flexible data processing utility:
```javascript
const dataProcessor = {
  // Process different types of data
  process({ data, filters = [], transformations = [], options = {} }) {
    // Apply default options
    const settings = {
      skipNulls: true,
      logProcess: false,
      maxResults: 1000,
      ...options
    };
    
    if (settings.logProcess) {
      console.log(`Processing ${data.length} records`);
    }
    
    // Apply filters
    let processed = [...data]; // Clone the array
    
    // Apply each filter
    filters.forEach(filter => {
      processed = processed.filter(item => this._applyFilter(item, filter));
    });
    
    if (settings.logProcess) {
      console.log(`After filtering: ${processed.length} records`);
    }
    
    // Apply transformations
    processed = processed.map(item => {
      // Create a copy of the item
      const result = { ...item };
      
      // Apply each transformation
      transformations.forEach(transformation => {
        this._applyTransformation(result, transformation);
      });
      
      return result;
    });
    
    // Handle null/undefined values if needed
    if (settings.skipNulls) {
      processed = processed.filter(item => item !== null && item !== undefined);
    }
    
    // Limit results
    if (processed.length > settings.maxResults) {
      processed = processed.slice(0, settings.maxResults);
    }
    
    // Return results with metadata
    return {
      results: processed,
      count: processed.length,
      timestamp: new Date(),
      appliedFilters: filters.length,
      appliedTransformations: transformations.length
    };
  },
  
  // Helper methods
  _applyFilter(item, filter) {
    const { field, operator, value } = filter;
    
    // Get field value (support nested fields with dot notation)
    const fieldValue = field.split('.').reduce((obj, key) => 
      obj && obj[key] !== undefined ? obj[key] : null, item);
      
    // Apply operator
    switch (operator) {
      case 'eq': return fieldValue === value;
      case 'neq': return fieldValue !== value;
      case 'gt': return fieldValue > value;
      case 'lt': return fieldValue < value;
      case 'contains': return String(fieldValue).includes(value);
      case 'startsWith': return String(fieldValue).startsWith(value);
      case 'endsWith': return String(fieldValue).endsWith(value);
      case 'in': return Array.isArray(value) && value.includes(fieldValue);
      default: return true;
    }
  },
  
  _applyTransformation(item, transformation) {
    const { field, action, args = [] } = transformation;
    
    // Handle nested fields
    const fieldParts = field.split('.');
    const lastPart = fieldParts.pop();
    
    // Navigate to the parent object
    const parent = fieldParts.reduce((obj, part) => {
      if (!obj[part]) obj[part] = {};
      return obj[part];
    }, item);
    
    // Get current value
    const currentValue = parent[lastPart];
    
    // Apply transformation
    switch (action) {
      case 'uppercase':
        parent[lastPart] = String(currentValue).toUpperCase();
        break;
      case 'lowercase':
        parent[lastPart] = String(currentValue).toLowerCase();
        break;
      case 'multiply':
        parent[lastPart] = Number(currentValue) * (args[0] || 1);
        break;
      case 'format':
        if (args[0] === 'currency') {
          parent[lastPart] = new Intl.NumberFormat('en-US', {
            style: 'currency',
            currency: args[1] || 'USD'
          }).format(currentValue);
        }
        break;
      default:
        // Custom transformation if function is provided
        if (typeof action === 'function') {
          parent[lastPart] = action(currentValue, ...args);
        }
    }
  }
};

// Usage example
const userData = [
  { name: "John", salary: 50000, department: { id: 1, name: "IT" } },
  { name: "Jane", salary: 60000, department: { id: 2, name: "HR" } },
  { name: "Bob", salary: 55000, department: { id: 1, name: "IT" } }
];

const result = dataProcessor.process({
  data: userData,
  filters: [
    { field: 'salary', operator: 'gt', value: 52000 }
  ],
  transformations: [
    { field: 'name', action: 'uppercase' },
    { field: 'salary', action: 'format', args: ['currency', 'USD'] },
    { field: 'department.name', action: 'lowercase' }
  ],
  options: {
    logProcess: true
  }
});

console.log(result.results);
// [
//   { name: "JANE", salary: "$60,000.00", department: { id: 2, name: "hr" } },
//   { name: "BOB", salary: "$55,000.00", department: { id: 1, name: "it" } }
// ]
```

### Common Pitfalls
1. **Arrow functions can't be used as constructors**:
   ```javascript
   const Person = (name) => {
     this.name = name; // 'this' is not bound correctly
   };
   
   const john = new Person("John"); // TypeError: Person is not a constructor
   
   // Solution: Use a regular function
   function Person(name) {
     this.name = name;
   }
   ```

2. **Destructuring undefined objects**:
   ```javascript
   // This will throw an error
   const { name } = undefined;
   
   // Safer approach with default empty object
   const { name } = data || {};
   
   // Or with optional chaining (ES2020)
   const name = data?.name;
   ```

3. **Spread operator only does shallow copying**:
   ```javascript
   const user = {
     name: "John",
     settings: {
       theme: "dark",
       notifications: true
     }
   };
   
   const clone = { ...user };
   clone.settings.theme = "light";
   
   console.log(user.settings.theme); // "light" - nested object was not deeply cloned!
   
   // For deep cloning, use alternatives like:
   // 1. JSON.parse(JSON.stringify(user)) - with limitations
   // 2. Structured clone (modern browsers)
   // 3. Libraries like lodash _.cloneDeep()
   ```

### Common Questions

**Q1: When should I use an arrow function versus a regular function?**
A1: Use arrow functions when you want to preserve the lexical `this` binding (like in callbacks or event handlers), for short one-line functions, or for functional programming with array methods. Use regular functions when you need to use `this` to refer to the function's context, when you need the function as a constructor, when you need access to `arguments` object, or when you need to use `yield`.

**Q2: What's the difference between the spread and rest operators? They look the same.**
A2: They use the same syntax (`...`) but in different contexts. The spread operator "expands" an array or object into individual elements (used in array literals, function calls, or object literals). The rest operator "collects" multiple elements into a single array (used in function parameters or destructuring assignments). A quick way to remember: spread "spreads out" values, rest "collects the rest" of values.

**Q3: Why doesn't destructuring work for deeply nested properties the same way spread doesn't do deep copies?**
A3: Destructuring, like the spread operator, only works at a single level of nesting. When you destructure nested objects, you're extracting references to those objects, not creating copies of their values. To destructure deeply nested properties, you need to use nested destructuring patterns: `const { user: { name, profile: { age } } } = data;`. For deep copies, you need special techniques like JSON methods or recursive copying.

## 10. Promises and Asynchronous JavaScript

### Concise Explanation
Promises are objects that represent the eventual completion or failure of an asynchronous operation and its resulting value. They allow you to write asynchronous code in a more manageable way, avoiding the "callback hell" of nested callbacks. Promises have three states: pending, fulfilled, or rejected.

### Where to Use
- API calls (fetch)
- File operations
- Database queries
- Any operation that takes time and shouldn't block the main thread
- Managing multiple concurrent operations
- Error handling in asynchronous code

### Code Snippet
```javascript
// Creating a promise
const myPromise = new Promise((resolve, reject) => {
  // Asynchronous operation
  setTimeout(() => {
    const success = true;
    
    if (success) {
      resolve("Operation completed successfully");
    } else {
      reject(new Error("Operation failed"));
    }
  }, 1000);
});

// Using a promise with .then() and .catch()
myPromise
  .then(result => {
    console.log(result); // "Operation completed successfully"
    return result.toUpperCase();
  })
  .then(upperResult => {
    console.log(upperResult); // "OPERATION COMPLETED SUCCESSFULLY"
  })
  .catch(error => {
    console.error(error.message);
  })
  .finally(() => {
    console.log("Promise settled (fulfilled or rejected)");
  });

// Promise.all() - wait for all promises to resolve
const promise1 = Promise.resolve(1);
const promise2 = Promise.resolve(2);
const promise3 = new Promise(resolve => setTimeout(() => resolve(3), 1000));

Promise.all([promise1, promise2, promise3])
  .then(values => {
    console.log(values); // [1, 2, 3]
  })
  .catch(error => console.error(error));

// Promise.race() - resolve or reject as soon as one promise settles
const fast = new Promise(resolve => setTimeout(() => resolve("Fast!"), 100));
const slow = new Promise(resolve => setTimeout(() => resolve("Slow!"), 500));

Promise.race([fast, slow])
  .then(result => console.log(result)); // "Fast!"

// Promise.allSettled() - wait for all promises to settle
const resolved = Promise.resolve(42);
const rejected = Promise.reject(new Error("Failed"));

Promise.allSettled([resolved, rejected])
  .then(results => {
    console.log(results);
    // [
    //   { status: "fulfilled", value: 42 },
    //   { status: "rejected", reason: Error("Failed") }
    // ]
  });

// Promise chaining
function getUser(userId) {
  return fetch(`https://api.example.com/users/${userId}`)
    .then(response => {
      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }
      return response.json();
    });
}

function getUserPosts(user) {
  return fetch(`https://api.example.com/users/${user.id}/posts`)
    .then(response => response.json());
}

// Chain promises
getUser(1)
  .then(user => {
    console.log("User:", user);
    return getUserPosts(user); // Return a new promise
  })
  .then(posts => {
    console.log("Posts:", posts);
  })
  .catch(error => {
    console.error("Error in promise chain:", error);
  });
```

### Real-World Example
Building an image loading utility with promises:
```javascript
// Image loader utility with promise-based API
const imageLoader = {
  // Load a single image
  load(url) {
    return new Promise((resolve, reject) => {
      const img = new Image();
      
      img.onload = () => resolve(img);
      img.onerror = () => reject(new Error(`Failed to load image: ${url}`));
      
      img.src = url;
    });
  },
  
  // Load multiple images in parallel
  loadAll(urls) {
    const promises = urls.map(url => this.load(url));
    return Promise.all(promises);
  },
  
  // Load images one after another
  loadSequentially(urls) {
    // Start with an empty array and a resolved promise
    return urls.reduce((promise, url) => {
      return promise.then(images => {
        return this.load(url).then(img => [...images, img]);
      });
    }, Promise.resolve([]));
  },
  
  // Load with timeout
  loadWithTimeout(url, timeoutMs = 5000) {
    return Promise.race([
      this.load(url),
      new Promise((_, reject) => 
        setTimeout(() => reject(new Error(`Timeout loading: ${url}`)), timeoutMs)
      )
    ]);
  },
  
  // Preload images but don't wait for them all
  preload(urls) {
    urls.forEach(url => {
      this.load(url)
        .then(() => console.log(`Preloaded: ${url}`))
        .catch(() => console.warn(`Failed to preload: ${url}`));
    });
  }
};

// Usage example: Load images for a carousel
function initializeCarousel(urls) {
  const carousel = document.querySelector('.carousel');
  const status = document.querySelector('.status');
  
  // Show loading indicator
  status.textContent = 'Loading images...';
  
  // Load all carousel images
  imageLoader.loadAll(urls)
    .then(images => {
      // Clear loading indicator
      status.textContent = '';
      
      // Add images to the carousel
      images.forEach(img => {
        const slide = document.createElement('div');
        slide.className = 'carousel-slide';
        slide.appendChild(img);
        carousel.appendChild(slide);
      });
      
      // Initialize the carousel functionality
      initializeCarouselControls();
    })
    .catch(error => {
      status.textContent = 'Failed to load some images. Please try again.';
      console.error('Carousel image loading failed:', error);
    });
}

// Start preloading the next set of images in the background
function preloadNextGalleryPage() {
  const nextPageUrls = [
    'https://example.com/gallery/next/image1.jpg',
    'https://example.com/gallery/next/image2.jpg',
    'https://example.com/gallery/next/image3.jpg'
  ];
  
  imageLoader.preload(nextPageUrls);
}

// Initialize the carousel with image URLs
initializeCarousel([
  'https://example.com/gallery/image1.jpg',
  'https://example.com/gallery/image2.jpg',
  'https://example.com/gallery/image3.jpg'
]);

// Start preloading the next page of images
preloadNextGalleryPage();
```

### Common Pitfalls
1. **Not handling promise rejections**:
   ```javascript
   // This will cause an unhandled promise rejection
   fetch('https://api.example.com/data')
     .then(response => response.json())
     .then(data => console.log(data));
     // No .catch() to handle errors!
     
   // Proper error handling
   fetch('https://api.example.com/data')
     .then(response => {
       if (!response.ok) {
         throw new Error(`HTTP error! Status: ${response.status}`);
       }
       return response.json();
     })
     .then(data => console.log(data))
     .catch(error => console.error('Fetch error:', error));
   ```

2. **Forgetting that .then() returns a new promise**:
   ```javascript
   // Not using the return value
   const promise = getUser(1);
   
   promise.then(user => {
     const postsPromise = getUserPosts(user); // This creates a new promise
     // But we don't return it, so the next .then() doesn't wait for it!
   });
   
   // .then() continues with the result of the original promise
   promise.then(user => {
     console.log("This runs in parallel with getUserPosts!");
   });
   
   // Correct chaining
   getUser(1)
     .then(user => {
       return getUserPosts(user); // Return the promise for proper chaining
     })
     .then(posts => {
       console.log("Posts:", posts); // This waits for getUserPosts to finish
     });
   ```

3. **Nested promises instead of chaining**:
   ```javascript
   // Promise nesting (callback hell-like)
   getUser(1)
     .then(user => {
       getUserPosts(user)
         .then(posts => {
           getUserComments(posts[0].id)
             .then(comments => {
               // Deeply nested, harder to read and manage
               console.log(comments);
             });
         });
     });
     
   // Proper promise chaining
   getUser(1)
     .then(user => getUserPosts(user))
     .then(posts => getUserComments(posts[0].id))
     .then(comments => console.log(comments))
     .catch(error => console.error(error));
   ```

### Common Questions

**Q1: What's the difference between `Promise.all()`, `Promise.race()`, and `Promise.allSettled()`?**
A1: `Promise.all()` waits for all promises to fulfill or for any one to reject, and returns an array of all fulfilled values. `Promise.race()` returns the result of the first promise that settles (either fulfills or rejects). `Promise.allSettled()` waits for all promises to settle regardless of whether they fulfill or reject, and returns an array of objects describing each outcome.

**Q2: How do I properly handle errors in a promise chain?**
A2: Add a `.catch()` at the end of your promise chain to catch any errors that occurred in any of the previous promises. For more granular error handling, you can add multiple `.catch()` blocks at different points in the chain. Remember that a `.catch()` also returns a promise, so you can continue the chain after catching an error if needed.

**Q3: Can I convert callback-based functions to promise-based functions?**
A3: Yes, you can "promisify" callback-based functions by wrapping them in a promise. For example:

```javascript
// Callback-based function
function getDataCallback(id, callback) {
  setTimeout(() => {
    if (id > 0) {
      callback(null, { id, name: "Item " + id });
    } else {
      callback(new Error("Invalid ID"));
    }
  }, 1000);
}

// Promisified version
function getDataPromise(id) {
  return new Promise((resolve, reject) => {
    getDataCallback(id, (error, data) => {
      if (error) {
        reject(error);
      } else {
        resolve(data);
      }
    });
  });
}

// Now you can use it with promise syntax
getDataPromise(123)
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

Node.js also provides a built-in utility called `util.promisify` for this purpose.

## 11. Async/Await

### Concise Explanation
Async/await is syntactic sugar built on top of promises, making asynchronous code look and behave more like synchronous code. It makes promise-based code more readable and easier to debug. An `async` function always returns a promise, and the `await` keyword can only be used inside an `async` function.

### Where to Use
- Any situation where you would use promises
- Complex sequences of asynchronous operations
- When you need to handle errors in a try/catch pattern
- When readability is important and you want to avoid promise chain nesting

### Code Snippet
```javascript
// Basic async/await
async function fetchUserData(userId) {
  try {
    // await suspends execution until promise resolves
    const response = await fetch(`https://api.example.com/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    
    const userData = await response.json();
    return userData;
  } catch (error) {
    console.error("Failed to fetch user:", error);
    throw error; // Re-throw to allow caller to handle
  }
}

// Using an async function
fetchUserData(1)
  .then(userData => console.log(userData))
  .catch(error => console.error("Error in main:", error));

// Sequential async operations
async function loadUserAndPosts(userId) {
  // Each await waits for the previous operation to complete
  const user = await fetchUserData(userId);
  const posts = await fetch(`https://api.example.com/users/${user.id}/posts`)
    .then(response => response.json());
  
  return { user, posts };
}

// Parallel async operations
async function loadUserAndPostsParallel(userId) {
  // Start both fetch operations in parallel
  const userPromise = fetchUserData(userId);
  const postsPromise = fetch(`https://api.example.com/users/${userId}/posts`)
    .then(response => response.json());
  
  // Wait for both to complete
  const user = await userPromise;
  const posts = await postsPromise;
  
  return { user, posts };
}

// Alternative parallel approach using Promise.all
async function loadUserAndPostsWithPromiseAll(userId) {
  // Wait for all promises to resolve in parallel
  const [user, posts] = await Promise.all([
    fetchUserData(userId),
    fetch(`https://api.example.com/users/${userId}/posts`).then(response => response.json())
  ]);
  
  return { user, posts };
}

// Async with array methods
async function processUsers(userIds) {
  // Use map to create an array of promises
  const userPromises = userIds.map(id => fetchUserData(id));
  
  // Wait for all promises to resolve
  const users = await Promise.all(userPromises);
  
  return users;
}

// Async with error handling
async function fetchWithRetry(url, retries = 3) {
  let lastError;
  
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      console.warn(`Attempt ${i + 1} failed:`, error);
      lastError = error;
      
      // Wait before retrying (exponential backoff)
      if (i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
      }
    }
  }
  
  throw lastError;
}
```

### Real-World Example
Building a data fetching and caching service:
```javascript
// Data service with caching and error handling
class DataService {
  constructor() {
    this.cache = new Map();
    this.pendingRequests = new Map();
    this.cacheExpiry = 5 * 60 * 1000; // 5 minutes
  }
  
  // Get data with caching
  async getData(url, options = {}) {
    const cacheKey = this._getCacheKey(url, options);
    
    // Check if we have a valid cached response
    if (this._isCacheValid(cacheKey)) {
      console.log(`[Cache hit] ${url}`);
      return this.cache.get(cacheKey).data;
    }
    
    // Check if this request is already in progress
    if (this.pendingRequests.has(cacheKey)) {
      console.log(`[Pending request] ${url}`);
      return this.pendingRequests.get(cacheKey);
    }
    
    // Create a new request promise
    const requestPromise = this._fetchData(url, options);
    
    // Store the pending request
    this.pendingRequests.set(cacheKey, requestPromise);
    
    try {
      // Wait for the data
      const data = await requestPromise;
      
      // Cache the result if caching is enabled
      if (!options.noCache) {
        this._cacheResponse(cacheKey, data);
      }
      
      return data;
    } finally {
      // Remove from pending regardless of outcome
      this.pendingRequests.delete(cacheKey);
    }
  }
  
  // Fetch data from API
  async _fetchData(url, options = {}) {
    try {
      const { retries = 2, timeout = 10000, ...fetchOptions } = options;
      
      // Create a timeout promise
      const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error(`Request timeout for: ${url}`)), timeout);
      });
      
      // Try the request with retries
      let lastError;
      
      for (let attempt = 0; attempt <= retries; attempt++) {
        try {
          // Race between fetch and timeout
          const response = await Promise.race([
            fetch(url, fetchOptions),
            timeoutPromise
          ]);
          
          if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
          }
          
          const data = await response.json();
          
          // Log successful request
          console.log(`[Fetch success] ${url}`);
          
          return data;
        } catch (error) {
          lastError = error;
          
          // Log retry attempts
          if (attempt < retries) {
            console.warn(`[Retry ${attempt + 1}/${retries}] ${url}: ${error.message}`);
            // Wait before retrying (exponential backoff)
            await new Promise(r => setTimeout(r, 1000 * Math.pow(2, attempt)));
          }
        }
      }
      
      // If we get here, all retries failed
      console.error(`[Fetch failed] ${url}: ${lastError.message}`);
      throw lastError;
    } catch (error) {
      console.error(`[Fetch error] ${url}: ${error.message}`);
      throw error;
    }
  }
  
  // Generate a cache key
  _getCacheKey(url, options) {
    // Include relevant options in the cache key
    const keyParts = [url];
    
    if (options.body) {
      keyParts.push(options.body);
    }
    
    if (options.headers) {
      keyParts.push(JSON.stringify(options.headers));
    }
    
    return keyParts.join('|');
  }
  
  // Check if cached data is valid
  _isCacheValid(cacheKey) {
    if (!this.cache.has(cacheKey)) {
      return false;
    }
    
    const { timestamp } = this.cache.get(cacheKey);
    return Date.now() - timestamp < this.cacheExpiry;
  }
  
  // Store response in cache
  _cacheResponse(cacheKey, data) {
    this.cache.set(cacheKey, {
      timestamp: Date.now(),
      data
    });
  }
  
  // Clear cache
  clearCache() {
    this.cache.clear();
    console.log('Cache cleared');
  }
  
  // Clear specific cache entry
  invalidateCache(url, options = {}) {
    const cacheKey = this._getCacheKey(url, options);
    const deleted = this.cache.delete(cacheKey);
    console.log(`Cache ${deleted ? 'invalidated' : 'not found'}: ${url}`);
    return deleted;
  }
}

// Usage example
async function loadDashboard() {
  try {
    const dataService = new DataService();
    
    // Start all requests in parallel
    const [userData, postsData, analyticsData] = await Promise.all([
      dataService.getData('https://api.example.com/user/profile'),
      dataService.getData('https://api.example.com/posts/recent'),
      dataService.getData('https://api.example.com/analytics/summary', { 
        noCache: true, // Always get fresh analytics
        retries: 3
      })
    ]);
    
    // Process and display the data
    updateUserProfile(userData);
    displayRecentPosts(postsData);
    renderAnalytics(analyticsData);
    
    console.log('Dashboard loaded successfully');
  } catch (error) {
    console.error('Failed to load dashboard:', error);
    showErrorMessage('Some dashboard components failed to load. Please try again later.');
  }
}

// Load the dashboard
loadDashboard();
```

### Common Pitfalls
1. **Forgetting that `async` functions always return promises**:
   ```javascript
   // This doesn't work as expected
   function getData() {
     const data = fetchUserData(1); // fetchUserData is async
     console.log(data); // This logs a Promise, not the data!
   }
   
   // Correct approach
   async function getData() {
     try {
       const data = await fetchUserData(1);
       console.log(data); // Now this logs the actual data
     } catch (error) {
       console.error(error);
     }
   }
   ```

2. **Sequential vs parallel async operations**:
   ```javascript
   // Sequential - slower
   async function getMultipleData() {
     const data1 = await fetchData1(); // Wait for first request
     const data2 = await fetchData2(); // Only starts after first completes
     return [data1, data2]; 
   }
   
   // Parallel - faster
   async function getMultipleDataParallel() {
     // Start both requests immediately
     const data1Promise = fetchData1();
     const data2Promise = fetchData2();
     
     // Now wait for their results
     const data1 = await data1Promise;
     const data2 = await data2Promise;
     
     return [data1, data2];
   }
   
   // Or more concisely with Promise.all
   async function getMultipleDataParallelConcise() {
     return Promise.all([fetchData1(), fetchData2()]);
   }
   ```

3. **Not using try/catch for error handling**:
   ```javascript
   // Missing error handling
   async function getUserData(userId) {
     const response = await fetch(`/api/users/${userId}`);
     const data = await response.json();
     return data; // If fetch fails, error propagates to caller
   }
   
   // Proper error handling
   async function getUserData(userId) {
     try {
       const response = await fetch(`/api/users/${userId}`);
       
       if (!response.ok) {
         throw new Error(`HTTP error! Status: ${response.status}`);
       }
       
       const data = await response.json();
       return data;
     } catch (error) {
       console.error(`Failed to get user ${userId}:`, error);
       // Either handle the error or re-throw
       throw error;
     }
   }
   ```

### Common Questions

**Q1: When should I use async/await versus promise chains with .then()?**
A1: Use async/await when you want more readable, sequential-looking code, especially with complex logic and error handling. It's particularly valuable for conditional logic around promises. Use promise chains when you're doing simpler transformations in a functional style, or when working with existing code that uses promise chains. Generally, async/await is more readable and maintainable for most use cases.

**Q2: Can I use await outside of an async function?**
A2: Traditionally, await could only be used inside an async function. However, modern JavaScript (since ES2022) supports "top-level await" in ECMAScript modules, allowing await outside of async functions at the module's top level. This is useful for dynamic imports and initialization. In regular scripts and older environments, you still need to wrap await calls in an async function.

**Q3: How do I handle multiple promises running in parallel with async/await?**
A3: There are three main approaches:
1. Start all promises and then await them individually:
   ```javascript
   const promise1 = fetch('/api/data1');
   const promise2 = fetch('/api/data2');
   const data1 = await promise1;
   const data2 = await promise2;
   ```

2. Use Promise.all for when all promises must succeed:
   ```javascript
   const [data1, data2] = await Promise.all([fetch('/api/data1'), fetch('/api/data2')]);
   ```

3. Use Promise.allSettled when you need to handle mixed results:
   ```javascript
   const results = await Promise.allSettled([fetch('/api/data1'), fetch('/api/data2')]);
   // Process results checking the status of each
   ```

## Next Actions: Practice Projects

### Project 1: Interactive Todo List
**Goal**: Create a complete todo list application with CRUD operations.

**Features to Implement**:
- Add new tasks
- Mark tasks as complete
- Edit task text
- Delete tasks
- Filter tasks (all, active, completed)
- Save tasks to localStorage

**Skills Practiced**:
- DOM manipulation
- Event handling
- Array methods
- localStorage usage
- Object manipulation

### Project 2: Weather Dashboard
**Goal**: Build a weather dashboard that fetches real-time data from a weather API.

**Features to Implement**:
- Search for cities
- Display current weather
- Show a 5-day forecast
- Save favorite locations
- Toggle between Celsius/Fahrenheit
- Display weather icons

**Skills Practiced**:
- Fetch API
- Promises or Async/Await
- DOM manipulation
- Error handling
- Working with external APIs

### Project 3: Interactive Image Gallery
**Goal**: Create an image gallery with filtering and lightbox functionality.

**Features to Implement**:
- Grid display of images
- Filter images by category
- Search by image name/tags
- Lightbox for larger image view
- Image preloading
- Navigation between images

**Skills Practiced**:
- Array methods for filtering/sorting
- Event delegation
- Promise-based image loading
- DOM manipulation
- CSS class toggling

### Project 4: E-commerce Shopping Cart
**Goal**: Implement a shopping cart for an online store.

**Features to Implement**:
- Display product list
- Add/remove from cart
- Update quantity
- Calculate totals
- Apply discount codes
- Persist cart in localStorage

**Skills Practiced**:
- Object manipulation
- Array methods
- DOM updates
- Event handling
- Data storage

### Project 5: Github Profile Viewer
**Goal**: Create an app that fetches and displays GitHub user information.

**Features to Implement**:
- Search for GitHub users
- Display profile information
- Show repositories list
- Show commit history for selected repo
- Sort/filter repositories

**Skills Practiced**:
- Fetch API with async/await
- Promise handling
- Data transformation
- Error handling
- DOM manipulation

## Success Criteria

You can consider yourself successful in learning JavaScript when you can:

1. **Write Code Independently**:
   - Create projects from scratch without heavy reliance on tutorials
   - Debug your own code effectively
   - Refactor code to improve readability and performance

2. **Understand Core Concepts**:
   - Explain scope and closures
   - Describe how asynchronous JavaScript works
   - Understand the event loop
   - Know when to use different array methods

3. **Demonstrate Technical Skills**:
   - Manipulate the DOM confidently
   - Handle events and user interactions
   - Work with APIs and asynchronous code
   - Use modern ES6+ features appropriately

4. **Apply Best Practices**:
   - Write clean, maintainable code
   - Structure application logic effectively
   - Implement proper error handling
   - Create reusable functions and components

5. **Solve Real Problems**:
   - Create practical applications that solve real needs
   - Optimize code for performance
   - Implement complex features without tutorial guidance
   - Read and understand documentation to implement new features

## Troubleshooting Guide

### Debugging JavaScript

**Console Isn't Displaying Expected Output**:
- Check for syntax errors (look for red in your console)
- Verify that your script is properly linked in HTML
- Use `console.log()` at different points to trace execution flow
- Make sure your selectors match the elements you're targeting

**Uncaught TypeError: Cannot Read Property 'X' of Undefined/Null**:
- Use optional chaining: `object?.property`
- Add null checks before accessing properties
- Trace where the value should be defined
- Check that API responses are in the expected format

**Event Listeners Not Working**:
- Verify elements exist when listeners are added (DOM content loaded?)
- Check element selectors (are they correct?)
- Verify event type (click, submit, etc.)
- Check if event bubbling/capturing is causing issues

**Asynchronous Code Problems**:
- Use `async/await` with proper `try/catch` blocks
- Check if you're properly waiting for promises to resolve
- Look for unhandled promise rejections
- Verify API endpoints and network responses

**Scope and Context Issues**:
- Watch out for `this` context changing in functions
- Use arrow functions to preserve `this` in callbacks
- Check for variable scope issues (block scope vs function scope)
- Look for name collisions between global and local variables

**Performance Problems**:
- Minimize DOM manipulations (use document fragments)
- Avoid layout thrashing (batch your reads and writes)
- Use debounce/throttle for frequent events (scroll, resize)
- Be careful with memory leaks (remove event listeners when needed)

**Testing Your Knowledge**:
- Try explaining concepts to someone else
- Build projects without following tutorials
- Modify existing code to add new features
- Refactor old code using newer JavaScript features

Remember: Learn by doing. The best way to master JavaScript is to build projects that challenge you and solve real problems. When you encounter bugs (and you will!), treat them as learning opportunities rather than obstacles.

---


## Author
**Gemechis Chala (@venopyX)**

GitHub: [venopyX](https://github.com/venopyX)
LinkedIn: [in/venopyx](https://linkedin.com/in/venopyx)
