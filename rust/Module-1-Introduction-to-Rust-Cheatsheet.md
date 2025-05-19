# Module 1: Introduction to Rust Cheatsheet

## Core Problem and Key Assumptions
- **Problem**: Learning a systems programming language with memory safety guarantees without a garbage collector
- **Key Assumptions**:
  - You have basic programming experience in another language
  - You're looking for a language that is both powerful and safe
  - You need a language suitable for systems programming, web development, CLI tools, or other performance-critical applications

## Essential Concepts to Cover
1. Rust Philosophy and Design Principles
2. Setting Up the Rust Development Environment
3. Understanding the Rust Toolchain
4. Working with Cargo and Dependency Management
5. Writing Your First Rust Program
6. Rust Project Structure and Organization

---

## 1. Rust Philosophy and Design Principles

### Concise Explanation
Rust is a systems programming language focused on three goals: safety, speed, and concurrency. It prevents memory-related bugs at compile time through its ownership system while maintaining C/C++-like performance. Rust is designed for developers who need low-level control without sacrificing safety.

### Where to Use
- Systems programming (operating systems, device drivers)
- Performance-critical applications
- Applications requiring memory safety guarantees
- Concurrent and parallel programming
- WebAssembly applications
- CLI tools and utilities

### Code Snippet
```rust
// Rust's safety principles in action:
fn main() {
    // Variables are immutable by default
    let name = "Rust";
    
    // Explicit mutability
    let mut counter = 0;
    counter += 1;
    
    // Ownership example
    let s1 = String::from("hello");
    let s2 = s1; // Value moved here, s1 no longer valid
    
    // This would cause a compile error:
    // println!("{}", s1); // ERROR: value used after move
    
    println!("{}", s2); // Works fine
}
```

### Real-World Example
Mozilla developed Rust to solve memory safety issues in Firefox. Today, critical Firefox components like the CSS engine (Stylo) are written in Rust, resulting in fewer crashes and security vulnerabilities while maintaining or improving performance.

### Common Pitfalls
- **Misconception**: Rust is just another systems programming language like C++.
  - **Reality**: Rust's ownership system is a fundamentally different approach to memory management.
- **Misconception**: Rust's safety features make it slow or inefficient.
  - **Reality**: Most safety checks happen at compile time with minimal runtime cost.
- **Misconception**: Rust is too difficult for beginners.
  - **Reality**: While Rust has a steep learning curve, its strict compiler provides excellent guidance.

### Confusion Questions with Answers
1. **Q**: Is Rust's ownership system the same as garbage collection?
   **A**: No. Garbage collection happens at runtime with performance costs. Rust's ownership system enforces memory safety at compile time with no runtime overhead.

2. **Q**: Does Rust completely eliminate all bugs?
   **A**: No. Rust eliminates memory safety bugs like null pointer dereferences, buffer overflows, and data races. Logic errors, API misuse, and other bugs are still possible.

3. **Q**: Why doesn't Rust use garbage collection like other modern languages?
   **A**: Rust aims to provide memory safety without sacrificing performance or predictability. Garbage collection can introduce unpredictable pauses and overhead that aren't acceptable for some use cases.

---

## 2. Setting Up the Rust Development Environment

### Concise Explanation
Setting up Rust involves installing the Rust toolchain using rustup, the official Rust installer. Rustup installs the compiler (rustc), package manager (Cargo), and other essential tools. It also enables easy switching between stable, beta, and nightly Rust versions.

### Where to Use
- Initial setup for any Rust development
- Switching between different Rust versions for specific project needs
- Installing Rust on different operating systems (Windows, macOS, Linux)

### Code Snippet
```bash
# Installing Rust on Unix-like systems (macOS, Linux)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# On Windows, download and run rustup-init.exe from https://rustup.rs

# Verify installation
rustc --version
cargo --version

# Update Rust
rustup update

# Install a specific toolchain
rustup install nightly
rustup default nightly  # Switch to nightly

# Switch back to stable
rustup default stable
```

### Real-World Example
A web development team needs to use a Rust feature only available in the nightly compiler for a critical performance optimization. They use rustup to maintain both stable and nightly Rust installations, switching between them as needed without disrupting their development environment.

### Common Pitfalls
- **Pitfall**: Installing Rust without rustup (e.g., from OS package managers).
  - **Solution**: Always use rustup for better version control and cross-platform consistency.
- **Pitfall**: Not updating the toolchain regularly.
  - **Solution**: Run `rustup update` periodically to get bug fixes and improvements.
- **Pitfall**: Using nightly features without understanding the stability risks.
  - **Solution**: Prefer stable Rust unless you specifically need nightly features.

### Confusion Questions with Answers
1. **Q**: What's the difference between rustc and Cargo?
   **A**: rustc is the compiler that turns Rust code into executable binaries. Cargo is the package manager and build system that handles dependencies, building, testing, and more.

2. **Q**: Do I need to reinstall Rust when a new version is released?
   **A**: No. Simply run `rustup update` to get the latest stable version. Rustup manages updates for you.

3. **Q**: Can I have multiple Rust versions installed simultaneously?
   **A**: Yes. Rustup allows installing multiple toolchains (stable, beta, nightly, or specific versions) and switching between them with `rustup default <toolchain>`.

---

## 3. Rust Toolchain: Components and Usage

### Concise Explanation
The Rust toolchain consists of several integrated tools: rustc (compiler), cargo (package manager/build tool), rustfmt (code formatter), clippy (linter), rustdoc (documentation generator), and cargo test (testing framework). Together, these tools provide a comprehensive development environment.

### Where to Use
- rustc: Directly compiling Rust files (rarely used directly)
- cargo: Daily development, managing projects and dependencies
- rustfmt: Ensuring code style consistency
- clippy: Catching common mistakes and improving code quality
- rustdoc: Generating documentation from code comments
- cargo test: Writing and running tests

### Code Snippet
```bash
# Compile a single file directly (uncommon)
rustc main.rs

# More commonly, use Cargo
cargo new hello_world    # Create new project
cargo build              # Compile project
cargo run                # Compile and run
cargo check              # Check for errors without producing binary
cargo test               # Run tests
cargo fmt                # Format code with rustfmt
cargo clippy             # Run the Clippy linter
cargo doc --open         # Generate and open documentation
```

### Real-World Example
A developer starting on an existing Rust project clones the repository, runs `cargo build` to compile it, `cargo test` to verify everything works, and `cargo clippy` to check for any code issues. After making changes, they use `cargo fmt` to ensure their code follows project style guidelines before committing.

### Common Pitfalls
- **Pitfall**: Running `rustc` directly on files in a Cargo project.
  - **Solution**: Almost always use `cargo build` instead for proper dependency management.
- **Pitfall**: Not using clippy or rustfmt during development.
  - **Solution**: Integrate these tools into your workflow for higher code quality.
- **Pitfall**: Writing code without documentation comments.
  - **Solution**: Document public APIs with `///` comments that rustdoc can process.

### Confusion Questions with Answers
1. **Q**: When should I use `cargo check` vs. `cargo build`?
   **A**: Use `cargo check` during development to quickly verify your code compiles without producing an executable. It's faster than `cargo build`, which creates the actual binary.

2. **Q**: What's the difference between `cargo build` and `cargo build --release`?
   **A**: `cargo build` creates a debug build with no optimizations for faster compilation. `cargo build --release` creates an optimized build that runs much faster but takes longer to compile. Use release mode for benchmarks and production.

3. **Q**: Do I need to install rustfmt and clippy separately?
   **A**: No, they're included with rustup, though you might need to run `rustup component add clippy rustfmt` if they weren't installed by default.

---

## 4. Working with Cargo and Dependency Management

### Concise Explanation
Cargo is Rust's build system and package manager. It handles building code, downloading dependencies (called "crates"), and managing project configuration. Dependencies are specified in the `Cargo.toml` file, which follows the TOML format and contains project metadata and configuration.

### Where to Use
- Creating new Rust projects
- Managing dependencies from crates.io (Rust's package registry)
- Configuring build settings and features
- Publishing your own crates
- Managing workspaces with multiple related packages

### Code Snippet
```toml
# Cargo.toml example
[package]
name = "my_app"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <your.email@example.com>"]
description = "A short description of my application"

[dependencies]
# From crates.io with semantic versioning
serde = "1.0"           # Any 1.x.y version
rand = "^0.8.5"         # At least 0.8.5, but below 0.9.0
log = "~0.4.17"         # At least 0.4.17, but below 0.5.0

# Specific version
time = "=0.3.17"        # Exactly 0.3.17

# Features
tokio = { version = "1.0", features = ["full"] }

# Git repository
rocket = { git = "https://github.com/SergioBenitez/Rocket" }

# Local path
my_library = { path = "../my_library" }

[dev-dependencies]    # Used only for tests
pretty_assertions = "1.3"

[build-dependencies]  # Used only at build time
cc = "1.0"
```

### Real-World Example
A web service needs to parse JSON, handle HTTP requests, and connect to a database. The developer adds dependencies in `Cargo.toml` for serde_json (serialization), rocket (web framework), and sqlx (database). When running `cargo build`, Cargo automatically downloads these crates and their dependencies from crates.io.

### Common Pitfalls
- **Pitfall**: Using `*` for version numbers.
  - **Solution**: Specify version constraints (`1.0`, `^1.0`, etc.) to avoid unexpected breaking changes.
- **Pitfall**: Not understanding semver (semantic versioning).
  - **Solution**: Learn the meaning of major, minor, and patch versions (x.y.z) in Rust's ecosystem.
- **Pitfall**: Adding too many dependencies for simple tasks.
  - **Solution**: Consider if a dependency is worth it; more dependencies mean larger binaries and potential security issues.

### Confusion Questions with Answers
1. **Q**: What's the difference between `Cargo.toml` and `Cargo.lock`?
   **A**: `Cargo.toml` is the manifest you edit to specify version requirements. `Cargo.lock` is generated by Cargo and records the exact versions used. Commit `Cargo.lock` for applications, but not for libraries.

2. **Q**: How do I use a feature from a crate?
   **A**: Specify features using the syntax: `crate_name = { version = "1.0", features = ["feature1", "feature2"] }`. Features enable optional functionality.

3. **Q**: How do I use a local crate I'm developing?
   **A**: Use the path dependency: `my_crate = { path = "../path/to/my_crate" }`. Cargo will link to your local code instead of downloading from crates.io.

---

## 5. Writing and Running Your First Rust Program

### Concise Explanation
Creating your first Rust program involves generating a project with Cargo, understanding basic syntax for the main function, and using Rust's standard input/output functions. The entry point of a Rust program is the `main()` function, which returns no value by default.

### Where to Use
- Learning Rust basics
- Getting familiar with the syntax and tooling
- Creating small utilities or test programs
- Understanding the compilation and execution process

### Code Snippet
```rust
// Create a new project:
// $ cargo new hello_world
// $ cd hello_world

// This is src/main.rs:
fn main() {
    // Printing to console
    println!("Hello, world!");
    
    // Variables (immutable by default)
    let name = "Rust";
    println!("Hello, {}!", name);
    
    // Mutable variables
    let mut counter = 5;
    counter += 1;
    println!("Counter: {}", counter);
    
    // Basic user input
    println!("What's your name?");
    
    let mut input = String::new();
    
    std::io::stdin()
        .read_line(&mut input)
        .expect("Failed to read line");
    
    // Removing newline character
    let input = input.trim();
    
    println!("Hello, {}!", input);
}
```

### Real-World Example
A developer wants to create a simple CLI tool to convert temperatures between Celsius and Fahrenheit. They create a new Cargo project, implement the conversion logic in the main function, add input validation, and run it with `cargo run`. This introduces them to basic Rust syntax and I/O operations.

### Common Pitfalls
- **Pitfall**: Forgetting that variables are immutable by default.
  - **Solution**: Use `let mut` for variables that need to change.
- **Pitfall**: Not handling potential errors from I/O operations.
  - **Solution**: Use proper error handling like `.expect()` or `?` operator.
- **Pitfall**: Confusing `println!` (macro) with a regular function.
  - **Solution**: Remember that macros in Rust end with `!` and have different rules than functions.

### Confusion Questions with Answers
1. **Q**: Why do we need the `!` after `println`?
   **A**: The `!` indicates that `println` is a macro, not a function. Macros are expanded at compile time and can take a variable number of arguments with different types.

2. **Q**: What does `expect()` do in the input reading code?
   **A**: `expect()` handles the `Result` returned by `read_line()`. If an error occurs, the program will panic with the provided message. In production code, you'd typically use proper error handling instead.

3. **Q**: How do I run my Rust program with command-line arguments?
   **A**: Use `std::env::args()` to access arguments, and run with `cargo run -- arg1 arg2` (the `--` separates Cargo's arguments from your program's arguments).

---

## 6. Rust Project Structure and Organization

### Concise Explanation
Rust projects follow a conventional structure where code is organized in modules and crates. A standard Cargo project has a src directory containing source files, with `main.rs` as the binary entry point or `lib.rs` for a library. Multiple modules can be used to organize code logically.

### Where to Use
- Any non-trivial Rust project
- Sharing code between multiple binaries
- Creating reusable libraries
- Organizing large codebases

### Code Snippet
```
# Standard Rust project structure
my_project/
├── Cargo.toml         # Project configuration and dependencies
├── Cargo.lock         # Exact versions of dependencies (auto-generated)
├── src/               # Source directory
│   ├── main.rs        # Binary entry point
│   ├── lib.rs         # Library entry point (optional)
│   └── utils/         # Module directory
│       ├── mod.rs     # Module declaration
│       └── math.rs    # Submodule implementation
├── tests/             # Integration tests
│   └── integration_test.rs
├── benches/           # Benchmarks
│   └── benchmark.rs
├── examples/          # Example code
│   └── example.rs
└── target/            # Build output (auto-generated)
```

```rust
// src/main.rs
mod utils; // Declare module

fn main() {
    println!("2 + 2 = {}", utils::math::add(2, 2));
}

// src/utils/mod.rs
pub mod math; // Make math module public

// src/utils/math.rs
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

// Alternative module declaration without mod.rs (Rust 2018+)
// src/main.rs
mod utils;

// src/utils.rs
pub mod math;

// src/utils/math.rs
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

### Real-World Example
A CLI tool for processing image files is structured with a main.rs for the entry point and command parsing, a lib.rs containing the core functionality, and separate modules for image processing, file I/O, and utilities. This organization makes the code easier to test and maintain as the project grows.

### Common Pitfalls
- **Pitfall**: Putting too much code in a single file.
  - **Solution**: Use modules to logically separate functionality.
- **Pitfall**: Not understanding the module visibility rules.
  - **Solution**: Remember that items are private by default; use `pub` to expose them.
- **Pitfall**: Confusion between the old (mod.rs) and new (filename.rs) module systems.
  - **Solution**: Choose one approach and be consistent; the newer approach is generally preferred.

### Confusion Questions with Answers
1. **Q**: What's the difference between a crate and a module?
   **A**: A crate is a compilation unit that produces a library or executable. A module is a code organization unit within a crate that controls the privacy boundary and namespace.

2. **Q**: How do I make items from one module available in another?
   **A**: Make the items public with `pub` and then use the module path to access them. You may also use `use` statements to bring items into scope.

3. **Q**: Should I use `mod.rs` or the new module system?
   **A**: For new projects, the new system (without `mod.rs`) is generally preferred for cleaner file organization. Use `src/module_name.rs` for module declarations and `src/module_name/submodule.rs` for submodules.

---

## Next Actions: Exercises and Projects

### Exercises
1. **Environment Setup**
   - Install Rust using rustup
   - Set up Visual Studio Code or another editor with Rust extensions
   - Configure clippy and rustfmt

2. **Hello World Plus**
   - Create a "Hello World" program that takes a name as a command-line argument
   - Enhance it to support multiple names and customize the greeting

3. **Simple Calculator**
   - Build a command-line calculator that can perform basic operations
   - Implement proper error handling for invalid inputs

4. **File Word Counter**
   - Create a program that counts words, lines, and characters in a text file
   - Use modules to separate file I/O and counting logic

### Micro-Projects
1. **Temperature Converter**
   - Create a CLI tool that converts between Celsius, Fahrenheit, and Kelvin
   - Implement a clean module structure with separate conversion logic
   - Add proper error handling and user-friendly messages

2. **To-Do List Manager**
   - Build a simple command-line to-do list application
   - Implement commands to add, list, complete, and delete tasks
   - Save tasks to a file and load them when the program starts
   - Use modules to organize code for commands, file operations, and task management

3. **Mini Web Server**
   - Create a simple HTTP server that serves static files
   - Implement basic routing for different paths
   - Organize the code into modules for server logic, routing, and file handling

## Success Criteria
You've mastered the basics of Rust when you can:
1. Create new Rust projects using Cargo
2. Understand and explain Rust's basic syntax and safety principles
3. Use the Rust toolchain effectively (cargo, rustfmt, clippy)
4. Manage dependencies in Cargo.toml
5. Structure a multi-file Rust project with proper visibility and module organization
6. Write, compile, and run a Rust program that handles user input and potential errors
7. Read and understand error messages from the Rust compiler

## Troubleshooting Advice
1. **Compiler Errors**
   - Read the error message carefully; Rust's compiler provides detailed explanations
   - Focus on the first error first, as subsequent errors may be side effects
   - Look for suggestions in the error message, which often tell you exactly how to fix the issue

2. **Ownership and Borrowing Issues**
   - When seeing "value moved here" or "borrow of moved value" errors, review Rust's ownership rules
   - Consider using references (`&`) when you don't need ownership
   - For complex cases, try using `clone()` to create copies (though this has performance implications)

3. **Module and Path Problems**
   - Check your module declarations (`mod`) and use statements
   - Verify that items you're trying to access are marked as `pub`
   - Make sure module hierarchies match your file structure

4. **Build Issues**
   - Run `cargo clean` followed by `cargo build` to rebuild from scratch
   - Check that your Rust version is compatible with your dependencies
   - Use `cargo update` to update dependencies while respecting your version constraints

5. **Runtime Errors**
   - Use `println!` debugging or the `dbg!` macro to inspect values
   - Wrap risky operations in `Result` handling code
   - For panics, run with `RUST_BACKTRACE=1` to see the full stack trace: `RUST_BACKTRACE=1 cargo run`
