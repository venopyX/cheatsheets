# Module 2: Basic Syntax and Data Types - Part 2: Primitive Data Types

## Core Problem and Key Assumptions
- **Problem**: Understanding Rust's type system fundamentals and how primitive data types work
- **Key Assumptions**:
  - You understand basic variable declaration and scope
  - You need to know how to work with different data types effectively
  - You want to understand how Rust's type system prevents common errors
  - You're interested in efficient memory usage and performance implications

## Essential Concepts to Cover
1. Integer Types
2. Floating-Point Numbers
3. Booleans
4. Characters
5. Strings (`&str` and `String`)
6. Type Conversions

---

## 1. Integer Types

### Concise Explanation
Rust provides signed and unsigned integer types with explicit bit widths. Signed integers can represent negative values, while unsigned integers can only be positive. Each type has a specific memory size and range:
- Signed: `i8`, `i16`, `i32`, `i64`, `i128`, `isize` (based on architecture, 32 or 64 bits)
- Unsigned: `u8`, `u16`, `u32`, `u64`, `u128`, `usize` (based on architecture, 32 or 64 bits)

By default, integers are `i32`, which is generally the fastest type even on 64-bit systems.

### Where to Use
- `u8`: For byte values (0 to 255), e.g., ASCII characters, color components
- `i32`: For most integer arithmetic (default integer type)
- `u32`/`i32`: For medium-sized non-negative/negative numbers
- `u64`/`i64`: For larger numbers (e.g., timestamps, file sizes)
- `u128`/`i128`: For very large numbers (e.g., cryptography, scientific computing)
- `usize`: For memory sizes, indices, and collection lengths
- `isize`: Rarely used, for pointer arithmetic or when negative indices might be needed

### Code Snippet
```rust
fn main() {
    // Integer literals with type suffixes
    let a: i32 = 42;          // Decimal
    let b = 0xff_u8;          // Hexadecimal with type suffix
    let c = 0o70_i16;         // Octal with type suffix
    let d = 0b1010_1010_u16;  // Binary with type suffix
    
    // Integer literals can use _ as a visual separator
    let million = 1_000_000;  // Easier to read than 1000000
    
    // Type inference
    let x = 42;  // Defaults to i32
    
    // Size in memory
    println!("Size of i8: {} bytes", std::mem::size_of::<i8>());      // 1
    println!("Size of i32: {} bytes", std::mem::size_of::<i32>());    // 4
    println!("Size of usize: {} bytes", std::mem::size_of::<usize>()); // 8 on 64-bit
    
    // Range limits
    println!("i8 range: {} to {}", i8::MIN, i8::MAX);    // -128 to 127
    println!("u8 range: {} to {}", u8::MIN, u8::MAX);    // 0 to 255
    println!("i32 range: {} to {}", i32::MIN, i32::MAX); // -2^31 to 2^31-1
    
    // Integer operations
    let sum = 5 + 10;
    let difference = 95.5 - 4.3;
    let product = 4 * 30;
    let quotient = 56.7 / 32.2;
    let remainder = 43 % 5;
    
    // Overflow handling in debug mode (panic)
    // let will_panic = 255_u8 + 1; // Uncomment to see panic
    
    // Explicit overflow handling
    let wrapped = 255_u8.wrapping_add(1);  // 0
    let saturated = 255_u8.saturating_add(1);  // 255
    let (result, overflowed) = 255_u8.overflowing_add(1);  // (0, true)
    let checked = 255_u8.checked_add(1);  // None
}
```

### Real-World Example
A program that calculates file sizes and selects the appropriate unit:

```rust
fn format_file_size(size_in_bytes: u64) -> String {
    const KB: u64 = 1024;
    const MB: u64 = KB * 1024;
    const GB: u64 = MB * 1024;
    const TB: u64 = GB * 1024;
    
    if size_in_bytes < KB {
        format!("{} B", size_in_bytes)
    } else if size_in_bytes < MB {
        format!("{:.2} KB", size_in_bytes as f64 / KB as f64)
    } else if size_in_bytes < GB {
        format!("{:.2} MB", size_in_bytes as f64 / MB as f64)
    } else if size_in_bytes < TB {
        format!("{:.2} GB", size_in_bytes as f64 / GB as f64)
    } else {
        format!("{:.2} TB", size_in_bytes as f64 / TB as f64)
    }
}

fn main() {
    let sizes = [
        135,                // 135 B
        5_400,              // 5.27 KB
        3_500_000,          // 3.34 MB
        2_500_000_000,      // 2.33 GB
        9_000_000_000_000,  // 8.18 TB
    ];
    
    for size in sizes {
        println!("{}", format_file_size(size));
    }
}
```

### Common Pitfalls
- **Pitfall**: Integer overflow/underflow in debug builds causes panic, but wraps in release builds.
  - **Solution**: Use explicit methods like `wrapping_add`, `saturating_add`, or `checked_add` for intentional handling.
- **Pitfall**: Choosing an integer type that's too small for the range of values.
  - **Solution**: Consider future growth; when in doubt, choose a larger type like `i64` or `u64`.
- **Pitfall**: Mixing signed and unsigned integers leading to unexpected behavior.
  - **Solution**: Be consistent with integer types in calculations, or use explicit casts when mixing is necessary.

### Confusion Questions with Answers
1. **Q**: What's the difference between `usize` and `u64`?
   **A**: `usize` matches the architecture's pointer size (32-bit or 64-bit), making it ideal for collection indices and sizes. `u64` is always 64 bits regardless of architecture. Use `usize` for anything related to memory sizes or indexing, and `u64` for fixed-size 64-bit values.

2. **Q**: Why does Rust panic on integer overflow in debug mode but not in release mode?
   **A**: This is a design decision to catch potential bugs during development (debug builds) while allowing optimized code in production (release builds). In production, checking for overflow on every operation would be too costly for performance. If you need consistent behavior, use the explicit overflow-handling methods.

3. **Q**: When should I use explicit type annotations for integers?
   **A**: Use explicit types when: (1) you need a specific size for memory efficiency, (2) you're working with types that must match a specific width (like file formats or FFI), (3) the default `i32` isn't appropriate for your use case, or (4) when you want to make your intentions clear to future readers of your code.

---

## 2. Floating-Point Numbers

### Concise Explanation
Rust has two floating-point types: `f32` (32-bit, single precision) and `f64` (64-bit, double precision). These follow the IEEE-754 standard. The default type is `f64`, which provides more precision but uses more memory.

Floating-point numbers in Rust can represent decimal values but have limited precision and are subject to rounding errors, which is a fundamental characteristic of floating-point arithmetic in all programming languages.

### Where to Use
- `f32`: When memory usage is critical, for graphics programming (e.g., OpenGL), or when working with large arrays of floating-point numbers
- `f64`: For default floating-point calculations, scientific computing, or when higher precision is needed

### Code Snippet
```rust
fn main() {
    // Floating-point literals
    let x = 2.0;      // f64 by default
    let y: f32 = 3.0; // Explicitly f32
    
    // Scientific notation
    let billion = 1e9;       // 1,000,000,000 as f64
    let small = 1.23e-5_f32; // 0.0000123 as f32
    
    // Basic operations
    let sum = 10.5 + 5.3;
    let difference = 95.6 - 81.3;
    let product = 4.0 * 30.0;
    let quotient = 56.7 / 32.2;
    
    // No remainder (%) operator for floats, use fmod instead
    let remainder = 43.5_f64.rem_euclid(5.0);
    
    // Special values
    let infinity = f64::INFINITY;
    let neg_infinity = f64::NEG_INFINITY;
    let nan = f64::NAN;
    
    // NaN behavior
    println!("NaN == NaN: {}", nan == nan); // false
    println!("Is NaN: {}", nan.is_nan());   // true
    
    // Comparisons
    println!("1.0 > 2.0: {}", 1.0 > 2.0);   // false
    
    // Epsilon for approximate equality
    let a = 0.1 + 0.2;
    let b = 0.3;
    println!("a == b: {}", a == b);  // false due to floating-point precision
    println!("a ≈ b: {}", (a - b).abs() < f64::EPSILON);  // true with epsilon
    
    // Formatting for display
    println!("Pi: {:.5}", std::f64::consts::PI);  // 3.14159
}
```

### Real-World Example
A simple physics simulation calculating projectile motion:

```rust
fn calculate_projectile_position(
    initial_velocity: f64,
    angle_degrees: f64,
    time: f64
) -> (f64, f64) {
    // Constants
    const GRAVITY: f64 = 9.81; // m/s²
    
    // Convert angle to radians
    let angle_radians = angle_degrees.to_radians();
    
    // Decompose velocity into components
    let velocity_x = initial_velocity * angle_radians.cos();
    let velocity_y = initial_velocity * angle_radians.sin();
    
    // Calculate position at time t
    let x = velocity_x * time;
    let y = velocity_y * time - 0.5 * GRAVITY * time * time;
    
    (x, y)
}

fn main() {
    // Calculate projectile positions at different times
    let initial_velocity = 20.0; // m/s
    let angle = 45.0; // degrees
    
    for t in 0..10 {
        let time = t as f64 * 0.5; // Every half second
        let (x, y) = calculate_projectile_position(initial_velocity, angle, time);
        
        // Only print if projectile is above ground
        if y >= 0.0 {
            println!("At t={:.1}s: position=({:.2}m, {:.2}m)", time, x, y);
        } else {
            println!("At t={:.1}s: projectile has hit the ground", time);
            break;
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Comparing floating-point numbers for exact equality.
  - **Solution**: Use an epsilon value for approximate equality: `(a - b).abs() < f64::EPSILON`.
- **Pitfall**: Not handling NaN and infinity values.
  - **Solution**: Use `is_nan()`, `is_infinite()`, and `is_finite()` to check for special values.
- **Pitfall**: Loss of precision in calculations with values of very different magnitudes.
  - **Solution**: Rearrange calculations or use libraries designed for high-precision arithmetic when needed.

### Confusion Questions with Answers
1. **Q**: Why doesn't `0.1 + 0.2 == 0.3` evaluate to `true` in Rust?
   **A**: This is due to the inherent imprecision of binary floating-point representation. The decimal 0.1 and 0.2 can't be exactly represented in binary floating-point, so their sum isn't exactly 0.3. This behavior is consistent across most programming languages.

2. **Q**: When should I use `f32` instead of `f64`?
   **A**: Use `f32` when: (1) memory usage is critical, (2) you're working with hardware that's optimized for 32-bit operations, (3) the increased precision of `f64` isn't necessary, or (4) you're interfacing with APIs that expect 32-bit floats (like many graphics libraries).

3. **Q**: How do I handle currency or financial calculations in Rust?
   **A**: Never use floating-point types for currency or financial calculations due to precision issues. Instead, use the `rust_decimal` crate or represent money as integer cents/pennies. For example, $10.25 would be stored as 1025 cents.

---

## 3. Booleans

### Concise Explanation
The boolean type in Rust is `bool` and has two possible values: `true` and `false`. It occupies one byte of memory. Booleans are used for conditional logic, flags, and as the result of comparison operations.

### Where to Use
- Conditional statements (`if`, `while`)
- Representing binary states (on/off, yes/no, valid/invalid)
- Function return values indicating success/failure
- Boolean algebra and logical operations

### Code Snippet
```rust
fn main() {
    // Boolean literals
    let t = true;
    let f: bool = false; // With explicit type annotation
    
    // From comparison expressions
    let equal = 10 == 10;
    let not_equal = 10 != 5;
    let greater = 10 > 5;
    let less = 5 < 10;
    let greater_or_equal = 10 >= 10;
    let less_or_equal = 5 <= 10;
    
    // Logical operators
    let conjunction = true && false; // Logical AND (false)
    let disjunction = true || false; // Logical OR (true)
    let negation = !true;            // Logical NOT (false)
    
    // Short-circuit evaluation
    let x = false && expensive_function(); // expensive_function() is not called
    let y = true || expensive_function();  // expensive_function() is not called
    
    // Used in control flow
    if t {
        println!("Condition is true");
    } else {
        println!("Condition is false");
    }
    
    // Ternary-like expression using if-else
    let value = if f { 10 } else { 20 };
    println!("Value: {}", value); // 20
    
    // Booleans with match expressions
    match greater {
        true => println!("It is greater"),
        false => println!("It is not greater"),
    }
    
    // Size of a boolean
    println!("Size of bool: {} bytes", std::mem::size_of::<bool>()); // 1 byte
}

fn expensive_function() -> bool {
    println!("Expensive function called");
    true
}
```

### Real-World Example
A validation function for a user registration form:

```rust
struct UserRegistration {
    username: String,
    email: String,
    password: String,
    age: u8,
}

fn is_valid_email(email: &str) -> bool {
    // Simplified email validation
    email.contains('@') && email.contains('.')
}

fn is_valid_password(password: &str) -> bool {
    // Password should be at least 8 characters long
    // and contain at least one digit
    password.len() >= 8 && password.chars().any(|c| c.is_digit(10))
}

fn validate_registration(registration: &UserRegistration) -> (bool, Vec<String>) {
    let mut is_valid = true;
    let mut errors = Vec::new();
    
    // Validate username
    if registration.username.len() < 3 {
        is_valid = false;
        errors.push("Username must be at least 3 characters long".to_string());
    }
    
    // Validate email
    if !is_valid_email(&registration.email) {
        is_valid = false;
        errors.push("Email address is not valid".to_string());
    }
    
    // Validate password
    if !is_valid_password(&registration.password) {
        is_valid = false;
        errors.push("Password must be at least 8 characters long and contain a digit".to_string());
    }
    
    // Validate age
    if registration.age < 18 {
        is_valid = false;
        errors.push("You must be at least 18 years old".to_string());
    }
    
    (is_valid, errors)
}

fn main() {
    let user = UserRegistration {
        username: "bob".to_string(),
        email: "bob@example".to_string(), // Missing top-level domain
        password: "password".to_string(), // No digits
        age: 16,                          // Under 18
    };
    
    let (is_valid, errors) = validate_registration(&user);
    
    if is_valid {
        println!("Registration is valid!");
    } else {
        println!("Registration is invalid. Errors:");
        for error in errors {
            println!("- {}", error);
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Using `==` with booleans when unnecessary.
  - **Solution**: Simplify `if x == true` to just `if x` and `if x == false` to `if !x`.
- **Pitfall**: Not taking advantage of short-circuit evaluation.
  - **Solution**: Order conditions to put faster/more likely conditions first in `&&` chains, and faster/less likely conditions first in `||` chains.
- **Pitfall**: Using booleans when an enum would be more appropriate for multi-state logic.
  - **Solution**: If you find yourself with many related boolean flags, consider using an enum instead.

### Confusion Questions with Answers
1. **Q**: Is there a performance cost to using short-circuit evaluation in Rust?
   **A**: No, short-circuit evaluation is an optimization. In expressions like `a && b`, if `a` is `false`, `b` won't be evaluated at all, which improves performance if `b` is expensive to compute.

2. **Q**: Why does a `bool` take up a full byte when it only needs one bit?
   **A**: Memory is typically addressed in bytes, not bits. While a boolean conceptually needs only one bit, using a full byte makes memory access more efficient. Rust does optimize space when multiple booleans are stored in arrays or structs.

3. **Q**: Can I convert integers to booleans in Rust like in C/C++?
   **A**: No, Rust does not implicitly convert integers to booleans. You must explicitly compare: `if x != 0` rather than `if x`. This is a safety feature to prevent accidents. To convert, use explicit comparison like `x != 0` or `x > 0`.

---

## 4. Characters

### Concise Explanation
The `char` type in Rust represents a Unicode Scalar Value, which means it can represent much more than just ASCII characters. Each `char` is a 21-bit value (though stored in 4 bytes for alignment), capable of representing any Unicode character including emoji, accented letters, and characters from non-Latin alphabets.

### Where to Use
- When working with individual Unicode characters
- Text processing at the character level
- Unicode manipulation
- When you need to iterate through text by Unicode code points

### Code Snippet
```rust
fn main() {
    // Character literals use single quotes
    let c = 'z';
    let z: char = 'ℤ'; // Unicode character for the set of integers
    let heart_eyed_cat = '😻'; // Even emoji are valid chars
    
    // Escape sequences
    let tab = '\t';
    let newline = '\n';
    let single_quote = '\'';
    let backslash = '\\';
    
    // Unicode escape sequences
    let unicode_escape = '\u{211D}'; // ℝ (real number symbol)
    
    // Size of a char (always 4 bytes)
    println!("Size of char: {} bytes", std::mem::size_of::<char>()); // 4
    
    // Converting between char and integer types
    let ascii_code = 'A' as u32;  // 65
    let character = char::from_u32(ascii_code).unwrap();  // 'A'
    
    // Some character methods
    println!("Is alphabetic: {}", 'a'.is_alphabetic());  // true
    println!("Is numeric: {}", '7'.is_numeric());        // true
    println!("Is whitespace: {}", ' '.is_whitespace());  // true
    println!("Is uppercase: {}", 'A'.is_uppercase());    // true
    println!("To uppercase: {}", 'a'.to_uppercase());    // 'A'
    println!("To lowercase: {}", 'A'.to_lowercase());    // 'a'
    
    // Iterating through the characters of a string
    for c in "hello".chars() {
        println!("{}", c);
    }
}
```

### Real-World Example
A character counter that distinguishes between different types of characters:

```rust
fn analyze_text(text: &str) -> (usize, usize, usize, usize, usize) {
    let mut letters = 0;
    let mut digits = 0;
    let mut whitespace = 0;
    let mut punctuation = 0;
    let mut other = 0;
    
    for c in text.chars() {
        if c.is_alphabetic() {
            letters += 1;
        } else if c.is_digit(10) {
            digits += 1;
        } else if c.is_whitespace() {
            whitespace += 1;
        } else if c.is_ascii_punctuation() {
            punctuation += 1;
        } else {
            other += 1;
        }
    }
    
    (letters, digits, whitespace, punctuation, other)
}

fn main() {
    let text = "Hello, world! 123 😊";
    let (letters, digits, whitespace, punctuation, other) = analyze_text(text);
    
    println!("Text analysis for: '{}'", text);
    println!("Letters: {}", letters);          // 10
    println!("Digits: {}", digits);            // 3
    println!("Whitespace: {}", whitespace);    // 3
    println!("Punctuation: {}", punctuation);  // 2
    println!("Other (incl. emoji): {}", other); // 1
}
```

### Common Pitfalls
- **Pitfall**: Confusing a `char` with a single-character string.
  - **Solution**: Remember that `'a'` is a char, while `"a"` is a string. They are different types.
- **Pitfall**: Assuming a `char` only supports ASCII.
  - **Solution**: Rust's `char` type fully supports Unicode, allowing for emoji, accented characters, etc.
- **Pitfall**: Thinking that one character always takes one byte of storage.
  - **Solution**: Remember that a Rust `char` is 4 bytes, and even in UTF-8 encoding a single character can take up to 4 bytes.

### Confusion Questions with Answers
1. **Q**: Why does a `char` use 4 bytes when Unicode only needs up to 21 bits?
   **A**: For memory alignment and efficiency. Most systems access memory more efficiently in 4-byte (32-bit) chunks. The extra bits are essentially padding to improve performance.

2. **Q**: Is a `char` the same as a single letter or character in a string?
   **A**: Almost, but not quite. A `char` is a single Unicode Scalar Value. Some visible characters (like some emoji or combined characters) may be represented as multiple `char` values when decomposed. What appears as a single character to the user might be multiple `char`s.

3. **Q**: How do I convert between characters and their numeric Unicode values?
   **A**: Convert a `char` to its numeric value with `char as u32`. Convert back with `char::from_u32(n)`, which returns an `Option<char>` because not all 32-bit values are valid Unicode code points.

---

## 5. Strings (`&str` and `String`)

### Concise Explanation
Rust has two main string types:
1. `&str` (string slice): An immutable reference to UTF-8 encoded string data. It's often seen as a string literal (`"hello"`) or as a view into a `String`.
2. `String`: A growable, mutable, owned UTF-8 encoded string. Think of it like a `Vec<u8>` that is guaranteed to be valid UTF-8.

Both types ensure strings are always valid UTF-8, unlike some other languages which allow invalid Unicode sequences.

### Where to Use
- `&str`:
  - Function parameters when you only need to read the string
  - String literals in your code (`"hello"`)
  - Slices of other strings
  - When you need a lightweight reference to string data
- `String`:
  - When you need to own or modify the string
  - Building strings dynamically
  - Storing strings in a struct
  - Reading text from external sources (files, network)

### Code Snippet
```rust
fn main() {
    // String literals are &str
    let greeting = "Hello, world!";
    
    // Creating a String from a string literal
    let s1 = String::from("Hello");
    let s2 = "World".to_string();
    
    // Building a String
    let mut s = String::new();
    s.push_str("Hello, ");
    s.push_str("world!");
    s.push('!');  // Add a single character
    
    // Concatenation
    let s3 = s1 + " " + &s2; // Note: s1 is moved here and can't be used again
    
    // Format macro (preferred for complex concatenation)
    let s4 = format!("{}, {}!", "Hello", "world");
    
    // String slices
    let slice = &s4[0..5]; // "Hello"
    
    // Iterating over a string's chars
    for c in "Здравствуйте".chars() {
        print!("{} ", c);
    }
    println!();
    
    // Iterating over bytes (careful: not valid for non-ASCII!)
    for b in "Hello".bytes() {
        print!("{} ", b);
    }
    println!();
    
    // Converting between &str and String
    let owned: String = "borrowed".to_string();
    let borrowed: &str = &owned;
    let also_borrowed: &str = owned.as_str();
    
    // String length vs char count
    let russian = "Здравствуйте";
    println!("Bytes: {}", russian.len());                // 24
    println!("Chars: {}", russian.chars().count());      // 12
    
    // Comparing strings
    println!("Equal: {}", "hello" == "hello");           // true
    println!("Compare: {}", "apple".cmp("banana"));      // Less
}
```

### Real-World Example
A simple username validator that ensures usernames meet certain criteria:

```rust
fn is_valid_username(username: &str) -> bool {
    // Check length (3-20 characters)
    if username.chars().count() < 3 || username.chars().count() > 20 {
        return false;
    }
    
    // Check first character is a letter
    if !username.chars().next().unwrap().is_alphabetic() {
        return false;
    }
    
    // Check all characters are alphanumeric or underscores
    for c in username.chars() {
        if !c.is_alphanumeric() && c != '_' {
            return false;
        }
    }
    
    // Check username doesn't contain double underscores
    if username.contains("__") {
        return false;
    }
    
    true
}

fn main() {
    let usernames = vec![
        "alice",            // Valid
        "bob_smith",        // Valid
        "a1",               // Too short
        "9lives",           // Starts with a number
        "john.doe",         // Contains invalid character
        "very_long_username_exceeds_limit",  // Too long
        "user__name",       // Contains double underscore
    ];
    
    for username in usernames {
        println!(
            "Username '{}' is {}",
            username,
            if is_valid_username(username) { "valid" } else { "invalid" }
        );
    }
}
```

### Common Pitfalls
- **Pitfall**: Indexing strings directly with `[]`.
  - **Solution**: Rust doesn't allow indexing strings directly because UTF-8 characters vary in byte length. Use slices (`&s[0..3]`) with caution or better yet, iterate with `.chars()`.
- **Pitfall**: Mixing up `&str` and `String` types.
  - **Solution**: Remember `&str` is borrowed and immutable, while `String` is owned and mutable. Use `.to_string()` or `String::from()` to convert from `&str` to `String`.
- **Pitfall**: Using `len()` to count characters.
  - **Solution**: `len()` returns the number of bytes, not characters. Use `chars().count()` to count Unicode characters.

### Confusion Questions with Answers
1. **Q**: What's the difference between `&str` and `String`, and when should I use each?
   **A**: `&str` is a view into string data (borrowed, immutable), while `String` is owned and mutable. Use `&str` for function parameters when you don't need ownership, and `String` when you need to modify the string or store it in a struct.

2. **Q**: Why can't I index into a string using bracket notation like `my_string[0]`?
   **A**: Unlike some languages, Rust strings are UTF-8 encoded, so characters can take 1-4 bytes. This means random access by character index would require scanning the string, which isn't O(1). Indexing by byte position with slices is possible but can lead to panics if you split a multi-byte character.

3. **Q**: What's the most efficient way to build a complex string with many parts?
   **A**: Use the `format!` macro or a `String` with multiple `push_str` calls. Avoid repeatedly using `+` operator concatenation, which creates new `String` instances. For very large strings or repeated operations, consider using `std::fmt::Write` or a string builder pattern.

---

## 6. Type Conversions

### Concise Explanation
Rust requires explicit type conversions in most cases, which helps prevent subtle bugs. There are several ways to convert between types:
1. Primitive numeric type casting using `as`
2. `From` and `Into` traits for clean conversions
3. `TryFrom` and `TryInto` for conversions that might fail
4. Custom conversion functions and methods

Explicit conversions help catch errors at compile time rather than causing unexpected behavior at runtime.

### Where to Use
- Converting between numeric types
- Converting between string types
- Converting user input to appropriate data types
- Working with APIs that expect specific types
- Implementing conversions for custom types

### Code Snippet
```rust
fn main() {
    // Numeric type casting with as
    let integer = 5;
    let float = integer as f64;
    println!("integer: {}, float: {}", integer, float);
    
    // Lossy conversions
    let large_number = 1000;
    let small_number = large_number as u8; // 232 (1000 % 256)
    println!("large: {}, small: {}", large_number, small_number);
    
    // From/Into traits
    let s1 = String::from("hello"); // From
    let s2: String = "world".into(); // Into
    
    // String to number
    let num_str = "42";
    let num: i32 = num_str.parse().expect("Not a number");
    
    // Using turbofish syntax
    let num = "42".parse::<i32>().unwrap();
    
    // TryFrom/TryInto (for fallible conversions)
    use std::convert::TryFrom;
    let big = 1000i32;
    let small = u8::try_from(big);
    
    match small {
        Ok(n) => println!("Converted to: {}", n),
        Err(e) => println!("Conversion failed: {}", e),
    }
    
    // Converting Option<T> to Option<U>
    let some_string = Some("123");
    let some_number: Option<i32> = some_string.map(|s| s.parse().unwrap());
    println!("Mapped option: {:?}", some_number);
    
    // Using match for custom conversion logic
    let weather_code = 500;
    let weather = match weather_code {
        200..=299 => "Thunderstorm",
        300..=399 => "Drizzle",
        400..=499 => "Rain",
        500..=599 => "Heavy Rain",
        600..=699 => "Snow",
        _ => "Unknown",
    };
    println!("Weather: {}", weather);
}

// Custom conversion implementations
struct Celsius(f64);
struct Fahrenheit(f64);

impl From<Celsius> for Fahrenheit {
    fn from(celsius: Celsius) -> Self {
        Fahrenheit(celsius.0 * 9.0 / 5.0 + 32.0)
    }
}

impl From<Fahrenheit> for Celsius {
    fn from(fahrenheit: Fahrenheit) -> Self {
        Celsius((fahrenheit.0 - 32.0) * 5.0 / 9.0)
    }
}

// Example usage
fn temperature_conversions() {
    let body_temp = Celsius(37.0);
    let fahrenheit: Fahrenheit = body_temp.into();
    println!("Body temperature: {:.1}°F", fahrenheit.0);
    
    let boiling = Fahrenheit(212.0);
    let celsius: Celsius = boiling.into();
    println!("Boiling point: {:.1}°C", celsius.0);
}
```

### Real-World Example
A function that parses different data formats from strings:

```rust
enum UserRole {
    Admin,
    Moderator,
    User,
    Guest,
}

impl std::str::FromStr for UserRole {
    type Err = String;
    
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        match s.to_lowercase().as_str() {
            "admin" => Ok(UserRole::Admin),
            "moderator" | "mod" => Ok(UserRole::Moderator),
            "user" => Ok(UserRole::User),
            "guest" => Ok(UserRole::Guest),
            _ => Err(format!("'{}' is not a valid role", s)),
        }
    }
}

fn parse_user_data(data: &str) -> Result<(String, u32, UserRole), String> {
    let parts: Vec<&str> = data.split(',').map(|s| s.trim()).collect();
    
    if parts.len() != 3 {
        return Err("Data must have exactly 3 parts: name, age, role".to_string());
    }
    
    let name = parts[0].to_string();
    
    let age: u32 = match parts[1].parse() {
        Ok(age) => age,
        Err(_) => return Err(format!("Invalid age: '{}'", parts[1])),
    };
    
    let role: UserRole = match parts[2].parse() {
        Ok(role) => role,
        Err(e) => return Err(e),
    };
    
    Ok((name, age, role))
}

fn main() {
    let user_data = vec![
        "Alice, 29, Admin",
        "Bob, 35, Moderator",
        "Charlie, twenty, User",  // Invalid age
        "Dave, 42, SuperUser",    // Invalid role
    ];
    
    for data in user_data {
        match parse_user_data(data) {
            Ok((name, age, role)) => {
                let role_str = match role {
                    UserRole::Admin => "Administrator",
                    UserRole::Moderator => "Moderator",
                    UserRole::User => "Regular User",
                    UserRole::Guest => "Guest User",
                };
                println!("Successfully parsed: {} ({}), role: {}", name, age, role_str);
            },
            Err(e) => println!("Error parsing '{}': {}", data, e),
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Using `as` for conversions that can lose data without checking.
  - **Solution**: Use `TryFrom`/`TryInto` for potentially lossy conversions to handle errors.
- **Pitfall**: Neglecting to handle errors when parsing strings.
  - **Solution**: Always handle error cases with `match`, `unwrap_or_default`, or proper error propagation.
- **Pitfall**: Inefficient repeated conversions.
  - **Solution**: Convert once and then operate on the converted value, rather than converting multiple times.

### Confusion Questions with Answers
1. **Q**: What's the difference between `as` and the `From`/`Into` traits?
   **A**: `as` is a primitive cast operator that works only for certain types (mainly numeric) and can lead to data loss. `From`/`Into` are traits that provide more controlled conversions, work with custom types, and often preserve semantics better. `TryFrom`/`TryInto` are preferred when conversion might fail.

2. **Q**: How do I choose between `parse()`, `from_str()`, and other conversion methods?
   **A**: `parse()` uses the `FromStr` trait under the hood and is convenient for string-to-type conversions. Use `parse()` for simple string parsing, `FromStr` for implementing custom string parsing, and `From`/`Into` for general type conversions that don't involve strings.

3. **Q**: Why do some conversions return `Result` while others don't?
   **A**: Conversions that can fail (like parsing a string to a number) return `Result` to allow error handling. Infallible conversions (like `i32` to `i64` where no data can be lost) use `From`/`Into` and return the value directly. When data loss is possible, use `TryFrom`/`TryInto` which return `Result`.

---

## Next Actions: Exercises and Projects

### Exercises
1. **Number Type Explorer**
   - Create a program that demonstrates the limits of different integer types
   - Show what happens with overflow in debug vs. release mode
   - Experiment with the different overflow handling methods

2. **Floating-Point Precision**
   - Create examples showing floating-point precision issues
   - Implement a safe approximate equality function with epsilon
   - Compare results between `f32` and `f64`

3. **String Manipulation Lab**
   - Build functions that manipulate strings in various ways (uppercase, reverse, count words)
   - Practice with both `String` and `&str`
   - Handle Unicode characters correctly

4. **Type Conversion Playground**
   - Create a program that safely converts between various types
   - Implement both `From` and `TryFrom` for custom types
   - Handle potential conversion errors gracefully

### Micro-Project: Text Analyzer
Build a text analyzer that:
- Counts words, sentences, paragraphs
- Calculates average word length
- Identifies most common words
- Analyzes character distributions
- Reports statistics about the text

```rust
// Example starter code
struct TextStats {
    character_count: usize,
    word_count: usize,
    sentence_count: usize,
    paragraph_count: usize,
    average_word_length: f64,
    most_common_words: Vec<(String, usize)>,
    character_distribution: std::collections::HashMap<char, usize>,
}

impl TextStats {
    fn new() -> Self {
        TextStats {
            character_count: 0,
            word_count: 0,
            sentence_count: 0,
            paragraph_count: 0,
            average_word_length: 0.0,
            most_common_words: Vec::new(),
            character_distribution: std::collections::HashMap::new(),
        }
    }
    
    fn from_text(text: &str) -> Self {
        let mut stats = TextStats::new();
        // Implement analysis logic...
        stats
    }
    
    fn display(&self) {
        println!("Text Statistics:");
        println!("-----------------");
        println!("Character count: {}", self.character_count);
        println!("Word count: {}", self.word_count);
        println!("Sentence count: {}", self.sentence_count);
        println!("Paragraph count: {}", self.paragraph_count);
        println!("Average word length: {:.2} characters", self.average_word_length);
        println!("\nMost common words:");
        for (word, count) in &self.most_common_words[0..std::cmp::min(5, self.most_common_words.len())] {
            println!("- {}: {} occurrences", word, count);
        }
        // Display character distribution...
    }
}

// Complete the implementation...
```

## Success Criteria
You've mastered primitive data types in Rust when you can:
1. Choose the appropriate integer and floating-point types for different scenarios
2. Handle numeric operations safely, including potential overflows
3. Understand when to use `String` vs `&str` and convert between them appropriately
4. Correctly work with Unicode characters and strings
5. Perform type conversions safely, handling potential failures
6. Implement conversions for your own types using the appropriate traits
7. Understand the memory layout and performance implications of different types

## Troubleshooting Advice
1. **Integer Overflow Issues**
   - Debug build: Check for panics caused by overflow
   - Release build: Look for unexpected results due to wrapping
   - Use explicit methods like `checked_add`, `saturating_sub`, or `wrapping_mul` to handle overflow intentionally
   - Consider using larger integer types if values might exceed the range

2. **Floating-Point Precision Problems**
   - Never test floating-point numbers for exact equality
   - Use an epsilon for approximate equality: `(a - b).abs() < 1e-10`
   - Consider fixed-point arithmetic or the `rust_decimal` crate for financial calculations
   - Use `f64` instead of `f32` when extra precision is needed

3. **String Handling Errors**
   - "Index out of bounds" when slicing: UTF-8 characters have variable width
   - Use `.chars().nth(n)` to get the nth character instead of indexing
   - String literals are `&str`, not `String` - use `.to_string()` or `String::from()` to convert
   - Remember that `.len()` gives bytes, not character count

4. **Type Conversion Problems**
   - Compiler errors about mismatched types: use explicit conversion with `as`, `into()`, or `from()`
   - Unexpected values after conversion: check for potential data loss
   - Use `TryFrom` instead of `as` when conversion might fail
   - Handle the result of `.parse()` with `match` or `?` operator, don't just `unwrap()`

5. **Character Encoding Issues**
   - Unicode-related bugs: ensure you're handling Unicode properly (don't assume 1 char = 1 byte)
   - String data from external sources: validate UTF-8 correctness
   - String serialization/deserialization: specify encoding format
   - Use the `unicode-segmentation` crate for advanced text processing
