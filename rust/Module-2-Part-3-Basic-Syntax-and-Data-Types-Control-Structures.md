# Module 2: Basic Syntax and Data Types - Part 3: Control Structures

## Core Problem and Key Assumptions
- **Problem**: Understanding how to control program flow in Rust with its unique features and safety guarantees
- **Key Assumptions**:
  - You understand basic variable declarations and data types
  - You need to implement decision-making and repetition in your code
  - You want to leverage Rust's pattern matching capabilities
  - You aim to write concise, safe, and expressive code

## Essential Concepts to Cover
1. Conditional Statements (`if`, `else`, `if let`)
2. Match Expressions
3. Basic Loops (`loop`, `while`, `for`)
4. Loop Control (`break`, `continue`, labeled loops)

---

## 1. Conditional Statements (if, else, if let)

### Concise Explanation
Rust's conditional statements allow code execution based on boolean conditions. The `if` expression evaluates a condition and executes a block if the condition is true. It can be extended with `else if` for multiple conditions and `else` for a fallback. The `if let` syntax provides a concise way to handle a single pattern match, particularly useful with `Option` and `Result` types.

Key points:
- Conditions don't need parentheses, but the code block requires braces
- `if` is an expression, so it can return values
- Condition must be a boolean (no implicit conversion from integers)
- All branches of an `if` expression that returns a value must return the same type

### Where to Use
- Basic decision-making based on boolean conditions
- Conditional initialization of variables
- Simple pattern matching with `if let`
- Early returns in functions
- Implementing conditional logic without sacrificing readability

### Code Snippet
```rust
fn main() {
    // Basic if statement
    let number = 6;
    
    if number % 2 == 0 {
        println!("{} is even", number);
    } else {
        println!("{} is odd", number);
    }
    
    // if with multiple conditions
    if number < 5 {
        println!("less than five");
    } else if number < 10 {
        println!("between five and nine");
    } else {
        println!("ten or greater");
    }
    
    // if as an expression (returning a value)
    let message = if number % 2 == 0 {
        "even"
    } else {
        "odd"
    };
    println!("The number is {}", message);
    
    // if with let expressions to create scoped variables
    if let some_value = calculate_value() {
        println!("Calculated value: {}", some_value);
    }
    
    // if let for pattern matching
    let some_option = Some(42);
    
    // Verbose way
    if some_option.is_some() {
        let value = some_option.unwrap();
        println!("Got value: {}", value);
    }
    
    // Concise way with if let
    if let Some(value) = some_option {
        println!("Got value: {}", value);
    }
    
    // Multiple if let conditions
    let point = (3, -2);
    
    if let (x, y) if x > 0 && y > 0 {
        println!("Point is in the first quadrant");
    } else if let (x, y) if x < 0 && y > 0 {
        println!("Point is in the second quadrant");
    } else if let (x, y) if x < 0 && y < 0 {
        println!("Point is in the third quadrant");
    } else if let (x, y) if x > 0 && y < 0 {
        println!("Point is in the fourth quadrant");
    } else {
        println!("Point is on an axis");
    }
}

fn calculate_value() -> i32 {
    // Some calculation
    42
}
```

### Real-World Example
A function that validates and processes user input:

```rust
enum UserStatus {
    Active,
    Inactive,
    Pending,
}

struct User {
    id: u64,
    name: Option<String>,
    email: String,
    status: UserStatus,
}

fn process_user_data(user: User) -> Result<String, String> {
    // Ensure user has a valid name
    if user.name.is_none() {
        return Err("User must have a name".to_string());
    }
    
    // Check user status
    if let UserStatus::Inactive = user.status {
        return Err("Cannot process inactive user".to_string());
    }
    
    // Unwrap the name (we've already checked it's Some)
    let name = user.name.unwrap();
    
    // Validate email format (very simple check)
    if !user.email.contains('@') {
        return Err("Invalid email format".to_string());
    }
    
    // Process the user differently based on status
    let confirmation = if let UserStatus::Active = user.status {
        format!("User {} (ID: {}) processed with active status", name, user.id)
    } else {
        format!("User {} (ID: {}) processed with pending status", name, user.id)
    };
    
    Ok(confirmation)
}

fn main() {
    let users = vec![
        User {
            id: 1,
            name: Some("Alice".to_string()),
            email: "alice@example.com".to_string(),
            status: UserStatus::Active,
        },
        User {
            id: 2,
            name: None,
            email: "bob@example.com".to_string(),
            status: UserStatus::Active,
        },
        User {
            id: 3,
            name: Some("Charlie".to_string()),
            email: "charlie-example.com".to_string(), // Missing @
            status: UserStatus::Pending,
        },
        User {
            id: 4,
            name: Some("Dave".to_string()),
            email: "dave@example.com".to_string(),
            status: UserStatus::Inactive,
        },
    ];
    
    for user in users {
        match process_user_data(user) {
            Ok(message) => println!("Success: {}", message),
            Err(error) => println!("Error: {}", error),
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Forgetting that conditions must be boolean expressions.
  - **Solution**: Use explicit comparison (`if x != 0`) instead of relying on non-zero values.
- **Pitfall**: Missing braces around if blocks.
  - **Solution**: In Rust, braces are required even for single-statement blocks.
- **Pitfall**: Inconsistent return types from different branches of an if expression.
  - **Solution**: Ensure all branches return the same type or use appropriate `Option`/`Result` types.
- **Pitfall**: Overusing if-else chains instead of match expressions.
  - **Solution**: For complex pattern matching, especially with enums, use `match` instead.

### Confusion Questions with Answers
1. **Q**: What's the difference between `if let` and regular `if` statements?
   **A**: `if let` combines `if` and `let` to provide a concise way to match a single pattern. It's syntactic sugar for a `match` with one pattern and a catch-all case. Regular `if` just evaluates a boolean expression without the pattern matching capability.

2. **Q**: Can an `if` expression in Rust return different types in different branches?
   **A**: No. Since `if` is an expression that can yield a value, all branches must return the same type for the compiler to know the type of the overall expression. This ensures type safety.

3. **Q**: Why doesn't Rust support the ternary operator (`condition ? then : else`) like many other languages?
   **A**: Rust's `if` expressions already serve the purpose of a ternary operator. Since `if` can return values, you can write `let x = if condition { value1 } else { value2 };`, which is clearer and more consistent than introducing a separate ternary syntax.

---

## 2. Match Expressions

### Concise Explanation
The `match` expression is one of Rust's most powerful features, allowing you to compare a value against a series of patterns and execute code based on which pattern matches. Unlike switch statements in some languages, Rust's `match` is:
- Exhaustive (all possible cases must be handled)
- An expression (returns a value)
- Pattern-based (matches against patterns, not just values)
- Type-aware (uses Rust's type system for safety)

Match arms consist of a pattern and the code to run if the value matches that pattern, separated by `=>`.

### Where to Use
- When handling multiple possible conditions
- For destructuring enums and extracting their values
- When working with `Option` and `Result` types
- To replace complex if-else chains
- For exhaustive case analysis
- When pattern matching against complex data structures

### Code Snippet
```rust
fn main() {
    // Basic match with integers
    let number = 13;
    
    match number {
        // Match a single value
        1 => println!("One!"),
        // Match multiple values
        2 | 3 | 5 | 7 | 11 | 13 => println!("This is a prime"),
        // Match a range
        15..=19 => println!("A teen"),
        // Default case
        _ => println!("Another number"),
    }
    
    // Match with enums
    let opt = Some(42);
    
    match opt {
        Some(value) => println!("Got value: {}", value),
        None => println!("No value"),
    }
    
    // Match as an expression
    let description = match number % 2 {
        0 => "even",
        _ => "odd",
    };
    println!("{} is {}", number, description);
    
    // Destructuring tuples
    let point = (2, 3);
    
    match point {
        (0, 0) => println!("Origin"),
        (0, y) => println!("Y-axis at y={}", y),
        (x, 0) => println!("X-axis at x={}", x),
        (x, y) => println!("Point at ({}, {})", x, y),
    }
    
    // Destructuring structs
    struct Point {
        x: i32,
        y: i32,
    }
    
    let point = Point { x: 10, y: 20 };
    
    match point {
        Point { x: 0, y: 0 } => println!("Origin"),
        Point { x, y: 0 } => println!("X-axis at x={}", x),
        Point { x: 0, y } => println!("Y-axis at y={}", y),
        Point { x, y } => println!("Point at ({}, {})", x, y),
    }
    
    // Match guards
    let pair = (2, -2);
    
    match pair {
        (x, y) if x == y => println!("Equal"),
        (x, y) if x + y == 0 => println!("Sum to zero"),
        (x, _) if x % 2 == 0 => println!("First is even"),
        _ => println!("No match"),
    }
    
    // Binding parts of a value with @ operator
    let num = 5;
    
    match num {
        n @ 1..=10 => println!("{} is between 1 and 10", n),
        n @ 11..=20 => println!("{} is between 11 and 20", n),
        _ => println!("Number is out of range"),
    }
}
```

### Real-World Example
A function that processes network events with appropriate error handling:

```rust
enum NetworkEvent {
    Connection { ip: String, port: u16 },
    Disconnection { reason: String },
    Data { bytes: Vec<u8> },
    Error { code: u32, message: String },
}

fn process_network_event(event: NetworkEvent) -> Result<String, String> {
    match event {
        NetworkEvent::Connection { ip, port } => {
            if !ip.contains('.') && !ip.contains(':') {
                return Err("Invalid IP address format".to_string());
            }
            
            let protocol = match port {
                80 => "HTTP",
                443 => "HTTPS",
                21 => "FTP",
                22 => "SSH",
                _ => "Unknown",
            };
            
            Ok(format!("New {} connection from {} on port {}", protocol, ip, port))
        },
        
        NetworkEvent::Disconnection { reason } => {
            Ok(format!("Disconnected: {}", reason))
        },
        
        NetworkEvent::Data { bytes } if bytes.is_empty() => {
            Err("Received empty data packet".to_string())
        },
        
        NetworkEvent::Data { bytes } => {
            let size = bytes.len();
            let first_byte = bytes[0];
            
            // Match on the protocol identifier (first byte)
            let protocol_name = match first_byte {
                0x01 => "File Transfer",
                0x02 => "Message",
                0x03 => "Status Update",
                b @ 0x10..=0x1F => format!("Audio Stream {}", b - 0x10),
                _ => "Unknown Protocol",
            };
            
            Ok(format!("Received {} data: {} bytes", protocol_name, size))
        },
        
        NetworkEvent::Error { code, message } => {
            match code {
                400..=499 => Err(format!("Client error {}: {}", code, message)),
                500..=599 => Err(format!("Server error {}: {}", code, message)),
                _ => Err(format!("Unknown error {}: {}", code, message)),
            }
        }
    }
}

fn main() {
    let events = vec![
        NetworkEvent::Connection { ip: "192.168.1.1".to_string(), port: 80 },
        NetworkEvent::Connection { ip: "invalid".to_string(), port: 443 },
        NetworkEvent::Data { bytes: vec![0x02, 0x48, 0x65, 0x6C, 0x6C, 0x6F] }, // "Hello"
        NetworkEvent::Data { bytes: vec![] }, // Empty data
        NetworkEvent::Data { bytes: vec![0x12, 0x01, 0x02] }, // Audio stream
        NetworkEvent::Error { code: 404, message: "Not Found".to_string() },
        NetworkEvent::Disconnection { reason: "User initiated".to_string() },
    ];
    
    for event in events {
        match process_network_event(event) {
            Ok(info) => println!("✓ {}", info),
            Err(error) => println!("✗ {}", error),
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Forgetting to make match expressions exhaustive.
  - **Solution**: Use the `_` wildcard pattern to catch all remaining cases when not explicitly handled.
- **Pitfall**: Accidentally creating unreachable patterns.
  - **Solution**: Order patterns from most specific to most general, as matches are evaluated in order.
- **Pitfall**: Not using match when appropriate, leading to verbose code.
  - **Solution**: When working with enums or complex conditions, prefer `match` over nested `if` statements.
- **Pitfall**: Using too many match guards, making code hard to follow.
  - **Solution**: For complex conditions, consider breaking the logic into separate functions.

### Confusion Questions with Answers
1. **Q**: What happens if I don't cover all possible cases in a match expression?
   **A**: The Rust compiler will give an error. Match expressions must be exhaustive, meaning all possible values must be covered. You can use the `_` wildcard pattern to catch any remaining cases.

2. **Q**: How does match differ from a switch statement in languages like C or JavaScript?
   **A**: Rust's match is more powerful: it (1) is an expression that returns values, (2) ensures all cases are handled, (3) supports complex pattern matching including destructuring, (4) has no implicit fallthrough between cases, and (5) uses pattern matching rather than just comparing values.

3. **Q**: When should I use a match expression versus if/else chains?
   **A**: Use match when: (1) working with enums, (2) you need exhaustive pattern matching, (3) you're destructuring complex data, or (4) you have multiple conditions to check against a single value. Use if/else for simple boolean logic or when conditions check different values.

---

## 3. Basic Loops (loop, while, for)

### Concise Explanation
Rust offers three types of loops:
1. `loop`: An infinite loop that runs until explicitly broken out of using `break`
2. `while`: A conditional loop that continues as long as a condition is true
3. `for`: An iterator-based loop for iterating over collections or ranges

These loops provide different ways to repeat code based on your needs. Like other expressions in Rust, loops can return values when broken with `break`.

### Where to Use
- `loop`:
  - When you need an infinite loop
  - For retrying operations until success
  - When the termination condition is complex or in the middle of the loop
  - When you need to return a value from a loop
- `while`:
  - When a loop should continue as long as a condition remains true
  - For simple pre-condition loops
  - When the number of iterations is not known in advance
- `for`:
  - Iterating over collections (arrays, vectors, maps)
  - Looping a specific number of times with ranges
  - Working with iterators
  - When you want to avoid manual indexing and bounds checking

### Code Snippet
```rust
fn main() {
    // Basic loop (infinite until broken)
    let mut counter = 0;
    
    loop {
        counter += 1;
        println!("Loop iteration: {}", counter);
        
        if counter >= 5 {
            break; // Exit the loop
        }
    }
    
    // loop with a return value
    let result = loop {
        counter += 1;
        
        if counter >= 10 {
            break counter * 2; // Return this value
        }
    };
    
    println!("Loop result: {}", result);
    
    // while loop
    let mut number = 3;
    
    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }
    
    println!("Liftoff!");
    
    // while with a mutable collection
    let mut stack = vec![1, 2, 3, 4, 5];
    
    while let Some(top) = stack.pop() {
        println!("Popped: {}", top);
    }
    
    // for loop with a range
    for i in 1..=5 {
        println!("For range: {}", i);
    }
    
    // for loop with an iterator
    let animals = vec!["cat", "dog", "fish", "bird"];
    
    for animal in animals {
        println!("Animal: {}", animal);
    }
    
    // for loop with enumeration
    let fruits = vec!["apple", "banana", "cherry"];
    
    for (index, fruit) in fruits.iter().enumerate() {
        println!("Fruit {}: {}", index, fruit);
    }
    
    // for loop with references
    let mut numbers = [1, 2, 3, 4, 5];
    
    // Iterate over references to modify the original array
    for number in &mut numbers {
        *number *= 2;
    }
    
    // Iterate over references to read
    for number in &numbers {
        println!("Number: {}", number);
    }
    
    // Nested loops
    for x in 1..=3 {
        for y in 1..=3 {
            println!("({}, {})", x, y);
        }
    }
}
```

### Real-World Example
A simple file processing utility:

```rust
use std::fs::{self, File};
use std::io::{self, BufRead, BufReader, Write};
use std::path::Path;

fn process_log_files(directory: &str, search_term: &str) -> io::Result<()> {
    let mut matches = Vec::new();
    
    // Read all entries in the directory
    for entry_result in fs::read_dir(directory)? {
        let entry = entry_result?;
        let path = entry.path();
        
        // Skip if not a file or doesn't have .log extension
        if !path.is_file() || path.extension().and_then(|ext| ext.to_str()) != Some("log") {
            continue;
        }
        
        println!("Processing file: {}", path.display());
        
        // Open and read the file
        let file = File::open(&path)?;
        let reader = BufReader::new(file);
        let file_name = path.file_name().unwrap().to_string_lossy();
        
        // Process each line
        for (line_number, line_result) in reader.lines().enumerate() {
            let line = line_result?;
            
            // Check if line contains the search term
            if line.contains(search_term) {
                matches.push(format!(
                    "File: {}, Line: {}, Content: {}", 
                    file_name, 
                    line_number + 1, 
                    line
                ));
            }
        }
    }
    
    // Print results
    if matches.is_empty() {
        println!("No matches found for '{}'", search_term);
    } else {
        println!("\nMatches for '{}':", search_term);
        
        let output_path = format!("{}/search_results.txt", directory);
        let mut output_file = File::create(&output_path)?;
        
        writeln!(output_file, "Search results for '{}'\n", search_term)?;
        
        for (i, matched_line) in matches.iter().enumerate() {
            println!("{}. {}", i + 1, matched_line);
            writeln!(output_file, "{}. {}", i + 1, matched_line)?;
        }
        
        println!("\nResults saved to {}", output_path);
    }
    
    Ok(())
}

fn main() {
    match process_log_files("./logs", "ERROR") {
        Ok(_) => println!("Processing completed successfully."),
        Err(e) => eprintln!("Error: {}", e),
    }
}
```

### Common Pitfalls
- **Pitfall**: Creating unintentional infinite loops.
  - **Solution**: Ensure there's a clear exit condition and that it's reachable.
- **Pitfall**: Using `for` loop when you need to modify a collection while iterating.
  - **Solution**: Use `while let` with explicit removal or filtering methods instead.
- **Pitfall**: Forgetting to dereference mutable references in for loops.
  - **Solution**: Remember to use `*number += 1` when modifying values through a mutable reference.
- **Pitfall**: Collecting values in a loop without pre-allocating space.
  - **Solution**: Use `Vec::with_capacity()` when you know the approximate size.

### Confusion Questions with Answers
1. **Q**: Why does Rust have both `loop` and `while true` if they seem to do the same thing?
   **A**: `loop` makes the intention of an infinite loop clearer and optimizes better. The compiler knows it's meant to be infinite, so it can make better optimizations. Additionally, `loop` expressions can return values when broken.

2. **Q**: How do I iterate over a collection and modify it at the same time?
   **A**: You typically can't modify a collection while iterating over it directly. Instead, you can: (1) use indices with a while loop, (2) iterate over a clone and modify the original, (3) collect modifications separately and apply them after the loop, or (4) use specialized methods like `retain()`.

3. **Q**: What's the difference between `for x in collection` and `for x in &collection`?
   **A**: `for x in collection` takes ownership of the collection, moving values into `x`. After the loop, the collection is no longer usable. `for x in &collection` borrows the collection, giving you references to the items. The collection remains usable after the loop.

---

## 4. Loop Control (break, continue, labeled loops)

### Concise Explanation
Rust provides several ways to control the flow within loops:
- `break`: Exits the loop immediately
- `continue`: Skips to the next iteration of the loop
- Labeled loops: Allow `break` and `continue` to specify which loop they affect in nested loops
- `break` with a value: Returns a value from a loop

These mechanisms give you precise control over loop execution and enable complex flow patterns while maintaining readability.

### Where to Use
- `break`: When a termination condition is met
- `continue`: When you want to skip processing the rest of the current iteration
- Labeled loops: In nested loops when you need to break or continue an outer loop
- `break` with value: When computing a result inside a loop that needs to be returned

### Code Snippet
```rust
fn main() {
    // Basic break and continue
    let mut count = 0;
    
    loop {
        count += 1;
        
        if count % 2 == 0 {
            println!("Skipping even number {}", count);
            continue; // Skip the rest of this iteration
        }
        
        println!("Processing odd number {}", count);
        
        if count >= 5 {
            println!("Breaking the loop at count = {}", count);
            break;
        }
    }
    
    // Breaking with a value
    let result = loop {
        count += 1;
        
        if count >= 10 {
            // This value will be returned from the loop
            break count * count;
        }
    };
    
    println!("Loop result: {}", result);
    
    // Labeled loops for nested loops
    'outer: for x in 1..=5 {
        println!("Outer loop iteration x = {}", x);
        
        'inner: for y in 1..=5 {
            println!("  Inner loop iteration y = {}", y);
            
            if x == 3 && y == 2 {
                println!("  Breaking inner loop when x={}, y={}", x, y);
                break; // Breaks only the inner loop
            }
            
            if x == 4 && y == 3 {
                println!("Breaking outer loop when x={}, y={}", x, y);
                break 'outer; // Breaks the outer loop
            }
        }
    }
    
    // continue with labeled loops
    'rows: for i in 1..=5 {
        println!("Row {}", i);
        
        'cols: for j in 1..=5 {
            if j == 2 {
                println!("  Skipping column 2");
                continue 'cols; // Skip to the next iteration of cols
            }
            
            if i == 3 && j == 4 {
                println!("  Skipping to next row");
                continue 'rows; // Skip to the next iteration of rows
            }
            
            println!("  Cell ({}, {})", i, j);
        }
    }
    
    // break in while
    let mut counter = 0;
    
    while counter < 10 {
        counter += 1;
        
        if counter == 7 {
            println!("Breaking at 7");
            break;
        }
        
        println!("While loop: {}", counter);
    }
    
    // continue in for loops
    for n in 1..=10 {
        if n % 3 == 0 {
            continue; // Skip multiples of 3
        }
        
        println!("For loop with continue: {}", n);
    }
}
```

### Real-World Example
A prime number sieve algorithm with loop control:

```rust
fn sieve_of_eratosthenes(limit: usize) -> Vec<usize> {
    if limit < 2 {
        return Vec::new(); // No primes less than 2
    }
    
    // Create a vector to track if numbers are prime
    let mut is_prime = vec![true; limit + 1];
    is_prime[0] = false;
    is_prime[1] = false;
    
    let mut primes = Vec::new();
    
    // Main sieving loop
    'outer: for num in 2..=limit {
        // If already marked as not prime, skip
        if !is_prime[num] {
            continue;
        }
        
        // Found a prime, add to our result
        primes.push(num);
        
        // If we've already found enough primes, break
        if primes.len() >= 1000 {
            println!("Reached 1000 primes, breaking calculation");
            break;
        }
        
        // Special handling for even numbers
        if num == 2 {
            // Mark all even numbers greater than 2 as non-prime
            for i in (4..=limit).step_by(2) {
                is_prime[i] = false;
            }
            continue; // Skip to next iteration
        }
        
        // Mark all multiples of this prime as non-prime
        // Start from num²
        let mut multiple = num * num;
        
        while multiple <= limit {
            is_prime[multiple] = false;
            multiple += num;
            
            // Example of using labeled breaks (though not necessary here)
            if multiple > limit {
                continue 'outer;
            }
        }
    }
    
    primes
}

fn main() {
    let limit = 100;
    let primes = sieve_of_eratosthenes(limit);
    
    println!("Primes up to {}: ", limit);
    
    // Print primes in rows of 10
    for (i, prime) in primes.iter().enumerate() {
        print!("{:3} ", prime);
        
        if (i + 1) % 10 == 0 {
            println!();
        }
    }
    
    // Final newline if needed
    if primes.len() % 10 != 0 {
        println!();
    }
}
```

### Common Pitfalls
- **Pitfall**: Using `break` or `continue` without labels in nested loops when intending to affect an outer loop.
  - **Solution**: Use labeled loops to specify which loop should be affected.
- **Pitfall**: Forgetting that `break` with a value only works in `loop`, not in `for` or `while`.
  - **Solution**: If you need a return value from a `for` or `while` loop, use a separate variable.
- **Pitfall**: Overusing loop labels making code harder to read.
  - **Solution**: Consider refactoring complex nested loops into separate functions.
- **Pitfall**: Creating an infinite loop without a clear exit condition.
  - **Solution**: Always ensure there's at least one reachable `break` statement in every `loop`.

### Confusion Questions with Answers
1. **Q**: How do labeled loops work with `break` and `continue`?
   **A**: Labels identify specific loops in nested structures. By default, `break` and `continue` affect the innermost loop. With a label (`break 'label` or `continue 'label`), they affect the loop with that label. This gives precise control over which loop to exit or continue.

2. **Q**: Can I return a value from any type of loop?
   **A**: No. Only the `loop` construct can return a value using `break value;`. For `while` and `for` loops, you need to use a separate variable outside the loop to store the result.

3. **Q**: Is there a way to "continue" a loop but restart from the beginning rather than going to the next iteration?
   **A**: No, Rust doesn't have a direct "restart" mechanism. However, you can simulate this by using labeled continue with a nested loop or by using a boolean flag and a conditional continue at the start of the loop.

---

## Next Actions: Exercises and Projects

### Exercises
1. **Conditional Logic Practice**
   - Write a function that determines the grade (A, B, C, D, F) based on a numeric score
   - Use `if`, `else if`, and `else` for one implementation
   - Rewrite the same logic using `match` for comparison

2. **Pattern Matching Challenge**
   - Create a function that takes various geometric shapes (rectangle, circle, triangle) as enums
   - Use `match` and pattern matching to calculate area and perimeter for each shape
   - Add a special case for squares as a variant of rectangles

3. **Loop Control Mastery**
   - Write a program that finds all Armstrong numbers between 1 and 10,000
   - Use appropriate loops and breaking conditions
   - Optimize by using mathematical properties to reduce calculations

4. **Nested Loop Labeling**
   - Create a multiplication table using nested loops
   - Add special formatting for prime numbers
   - Use labeled breaks to terminate the program when a user-specified value is found

### Micro-Project: Text Pattern Finder
Build a tool that searches a text file for patterns:
- Uses `if let` and `match` for processing different pattern types
- Employs loop control to efficiently scan the text
- Implements pattern matching for various search criteria (exact match, word boundary, regex-lite)
- Uses labeled loops for complex navigation through the text

```rust
// Example starter code
enum SearchPattern {
    Exact(String),
    StartsWith(String),
    EndsWith(String),
    Contains(String),
    Regex(String), // Simple regex support
}

struct SearchResult {
    line_number: usize,
    line: String,
    start_pos: usize,
    end_pos: usize,
}

fn search_text(text: &str, pattern: SearchPattern) -> Vec<SearchResult> {
    let mut results = Vec::new();
    
    // Split the text into lines
    for (line_number, line) in text.lines().enumerate() {
        // Implement pattern matching based on the SearchPattern type
        match pattern {
            SearchPattern::Exact(ref p) => {
                // Add search logic
            },
            SearchPattern::StartsWith(ref p) => {
                // Add search logic
            },
            // Implement other pattern types
            _ => {}
        }
    }
    
    results
}

// Complete the implementation...
```

## Success Criteria
You've mastered control structures in Rust when you can:
1. Use conditional statements effectively, choosing between `if/else` and `if let` appropriately
2. Write exhaustive `match` expressions with pattern matching for various data types
3. Implement the right loop type (`loop`, `while`, or `for`) for different scenarios
4. Control loop execution with `break` and `continue`, including with labeled loops
5. Return values from loops when needed
6. Refactor nested conditionals and loops for better readability
7. Implement complex algorithms using Rust's control structures efficiently

## Troubleshooting Advice
1. **Match Expression Issues**
   - "non-exhaustive patterns" error: Ensure your match covers all possible values or add a `_` wildcard
   - "unreachable pattern" warning: You have a pattern that will never match, often due to a more general pattern earlier in the match
   - Pattern matching fails: Check the order of patterns; more specific patterns must come before more general ones

2. **Loop Problems**
   - Infinite loops: Ensure your break condition is reachable and logical
   - Loop terminates too early: Check your break condition logic and verify variable modifications
   - Loop terminates too late: Make sure your break condition is evaluated at the right time

3. **Conditional Logic Errors**
   - Unexpected behavior: Verify your boolean expressions and check for operator precedence issues
   - Type mismatches: Rust won't implicitly convert types in conditions; use explicit comparisons
   - Brace misplacement: Ensure your code blocks are properly delimited with braces

4. **Flow Control Confusion**
   - Breaks affect wrong loop: Use labeled loops to specify which loop to break from
   - Skip/continue logic incorrect: Review your continue statements, possibly add labeled continues
   - Complex conditions: Simplify by extracting logic into functions with meaningful names

5. **Performance Concerns**
   - Inefficient loops: Consider breaking early when conditions are met
   - Unnecessary iterations: Use `continue` to skip unnecessary work
   - Nested loops too slow: Look for algorithmic improvements that reduce the number of iterations
