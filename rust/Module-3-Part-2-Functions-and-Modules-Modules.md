# Module 3: Functions and Modules - Part 2: Modules

## Core Problem and Key Assumptions
- **Problem**: Understanding Rust's module system for organizing code into logical units and managing visibility
- **Key Assumptions**:
  - You understand basic Rust syntax and functions
  - You need to organize code into maintainable, reusable components
  - You want to control encapsulation and visibility of your code
  - You aim to create clear API boundaries for your libraries and applications

## Essential Concepts to Cover
1. Creating and Organizing Modules
2. Visibility Rules and Access Control
3. Module Structure and Hierarchies
4. The mod.rs Pattern
5. Standard Library Module Organization

---

## 1. Creating and Organizing Modules

### Concise Explanation
Modules in Rust provide a way to organize code into logical units and control item visibility. They help avoid name conflicts and create a hierarchical structure for your code. Modules can be defined in-line in a single file, or split across multiple files.

There are two main ways to declare modules:
1. Using the `mod` keyword within a file to create nested modules
2. Creating separate files or directories for modules with the appropriate file structure

Modules follow a tree-like structure starting from the crate root, which is either `src/main.rs` for binaries or `src/lib.rs` for libraries.

### Where to Use
- To organize related functionality together
- To create logical API boundaries for your code
- When developing libraries to expose a clear public interface
- To group related types, functions, and traits
- To separate concerns in larger applications

### Code Snippet
```rust
// Main file (src/main.rs or src/lib.rs)

// Inline module declaration
mod math {
    // Module contents go here
    pub fn add(a: i32, b: i32) -> i32 {
        a + b
    }
    
    pub fn subtract(a: i32, b: i32) -> i32 {
        a - b
    }
    
    // Private function (not accessible outside this module)
    fn internal_helper() {
        println!("This is only visible within the math module");
    }
    
    // Nested module
    pub mod advanced {
        pub fn multiply(a: i32, b: i32) -> i32 {
            a * b
        }
        
        pub fn divide(a: i32, b: i32) -> Option<i32> {
            if b == 0 {
                None
            } else {
                Some(a / b)
            }
        }
    }
}

// Using modules
fn main() {
    // Accessing module items with path syntax
    let sum = math::add(5, 3);
    println!("Sum: {}", sum);
    
    // Accessing nested module
    let product = math::advanced::multiply(4, 6);
    println!("Product: {}", product);
    
    // Private functions cannot be accessed
    // math::internal_helper(); // ERROR: function is private
}
```

**Module in a separate file:**

File structure:
```
src/
├── main.rs
├── utils.rs
```

```rust
// src/main.rs
mod utils; // Declare that we're using a module from utils.rs

fn main() {
    utils::helper::print_message("Hello from main!");
    let formatted = utils::format_text("important message");
    println!("{}", formatted);
}
```

```rust
// src/utils.rs
// Items are private by default
pub fn format_text(text: &str) -> String {
    format!(":: {} ::", text.to_uppercase())
}

// Nested module in the same file
pub mod helper {
    pub fn print_message(msg: &str) {
        println!("MESSAGE: {}", msg);
    }
    
    // Private function
    fn internal_format(text: &str) -> String {
        format!(">> {} <<", text)
    }
}
```

**Module in a directory:**

File structure:
```
src/
├── main.rs
├── data/
│   ├── mod.rs
│   ├── models.rs
│   └── validation.rs
```

```rust
// src/main.rs
mod data; // Loads the module from data/mod.rs

fn main() {
    // Create a user and validate it
    let user = data::models::User::new("alice", "alice@example.com");
    
    match data::validation::validate_user(&user) {
        Ok(_) => println!("User is valid: {:?}", user),
        Err(e) => println!("Invalid user: {}", e),
    }
}
```

```rust
// src/data/mod.rs
pub mod models;
pub mod validation;

// You can also have code directly in mod.rs
pub fn initialize_data() {
    println!("Initializing data module");
}
```

```rust
// src/data/models.rs
#[derive(Debug)]
pub struct User {
    pub username: String,
    pub email: String,
}

impl User {
    pub fn new(username: &str, email: &str) -> Self {
        User {
            username: username.to_string(),
            email: email.to_string(),
        }
    }
}
```

```rust
// src/data/validation.rs
use super::models::User;

pub fn validate_user(user: &User) -> Result<(), String> {
    if user.username.is_empty() {
        return Err("Username cannot be empty".to_string());
    }
    
    if !user.email.contains('@') {
        return Err("Invalid email format".to_string());
    }
    
    Ok(())
}
```

### Real-World Example
A small application using modules to organize a command-line todo application:

File structure:
```
src/
├── main.rs
├── tasks/
│   ├── mod.rs
│   ├── task.rs
│   └── task_list.rs
├── storage/
│   ├── mod.rs
│   ├── file_storage.rs
│   └── memory_storage.rs
└── ui/
    ├── mod.rs
    └── cli.rs
```

```rust
// src/main.rs
mod tasks;
mod storage;
mod ui;

use tasks::task_list::TaskList;
use tasks::task::Task;
use storage::file_storage::FileStorage;
use ui::cli;

fn main() {
    // Initialize storage
    let storage = FileStorage::new("tasks.json");
    
    // Load tasks or create a new task list
    let mut task_list = match storage.load() {
        Ok(list) => {
            println!("Loaded {} tasks from storage", list.tasks().len());
            list
        },
        Err(e) => {
            println!("Error loading tasks: {}. Creating a new task list.", e);
            TaskList::new()
        }
    };
    
    // Run the CLI interface
    cli::run(&mut task_list, &storage);
}
```

```rust
// src/tasks/mod.rs
pub mod task;
pub mod task_list;
```

```rust
// src/tasks/task.rs
use std::time::{SystemTime, UNIX_EPOCH};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Task {
    id: u64,
    title: String,
    completed: bool,
    created_at: u64,
}

impl Task {
    pub fn new(title: &str) -> Self {
        let now = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        Task {
            id: now, // Use timestamp as a simple ID
            title: title.to_string(),
            completed: false,
            created_at: now,
        }
    }
    
    pub fn id(&self) -> u64 {
        self.id
    }
    
    pub fn title(&self) -> &str {
        &self.title
    }
    
    pub fn is_completed(&self) -> bool {
        self.completed
    }
    
    pub fn mark_completed(&mut self) {
        self.completed = true;
    }
    
    pub fn mark_incomplete(&mut self) {
        self.completed = false;
    }
}
```

```rust
// src/tasks/task_list.rs
use super::task::Task;
use std::collections::HashMap;

#[derive(Serialize, Deserialize)]
pub struct TaskList {
    tasks: HashMap<u64, Task>,
}

impl TaskList {
    pub fn new() -> Self {
        TaskList {
            tasks: HashMap::new(),
        }
    }
    
    pub fn add_task(&mut self, title: &str) -> &Task {
        let task = Task::new(title);
        let id = task.id();
        self.tasks.insert(id, task);
        &self.tasks[&id]
    }
    
    pub fn remove_task(&mut self, id: u64) -> Option<Task> {
        self.tasks.remove(&id)
    }
    
    pub fn get_task(&self, id: u64) -> Option<&Task> {
        self.tasks.get(&id)
    }
    
    pub fn get_task_mut(&mut self, id: u64) -> Option<&mut Task> {
        self.tasks.get_mut(&id)
    }
    
    pub fn tasks(&self) -> Vec<&Task> {
        self.tasks.values().collect()
    }
}
```

```rust
// src/storage/mod.rs
pub mod file_storage;
pub mod memory_storage;

use crate::tasks::task_list::TaskList;

pub trait Storage {
    fn save(&self, task_list: &TaskList) -> Result<(), String>;
    fn load(&self) -> Result<TaskList, String>;
}
```

```rust
// src/storage/file_storage.rs
use std::fs::{self, File};
use std::io::{Read, Write};
use std::path::Path;

use super::Storage;
use crate::tasks::task_list::TaskList;

pub struct FileStorage {
    filename: String,
}

impl FileStorage {
    pub fn new(filename: &str) -> Self {
        FileStorage {
            filename: filename.to_string(),
        }
    }
}

impl Storage for FileStorage {
    fn save(&self, task_list: &TaskList) -> Result<(), String> {
        let json = serde_json::to_string_pretty(task_list)
            .map_err(|e| format!("Serialization error: {}", e))?;
        
        File::create(&self.filename)
            .map_err(|e| format!("Failed to create file: {}", e))?
            .write_all(json.as_bytes())
            .map_err(|e| format!("Failed to write to file: {}", e))?;
        
        Ok(())
    }
    
    fn load(&self) -> Result<TaskList, String> {
        if !Path::new(&self.filename).exists() {
            return Err("Storage file does not exist".to_string());
        }
        
        let mut file = File::open(&self.filename)
            .map_err(|e| format!("Failed to open file: {}", e))?;
        
        let mut contents = String::new();
        file.read_to_string(&mut contents)
            .map_err(|e| format!("Failed to read file: {}", e))?;
        
        serde_json::from_str(&contents)
            .map_err(|e| format!("Deserialization error: {}", e))
    }
}
```

```rust
// src/ui/mod.rs
pub mod cli;
```

```rust
// src/ui/cli.rs
use std::io::{self, Write};

use crate::tasks::task_list::TaskList;
use crate::storage::Storage;

pub fn run<S: Storage>(task_list: &mut TaskList, storage: &S) {
    loop {
        print_menu();
        let choice = get_input("Enter your choice: ");
        
        match choice.trim() {
            "1" => list_tasks(task_list),
            "2" => add_task(task_list, storage),
            "3" => complete_task(task_list, storage),
            "4" => remove_task(task_list, storage),
            "5" => {
                println!("Goodbye!");
                break;
            }
            _ => println!("Invalid choice, please try again."),
        }
    }
}

fn print_menu() {
    println!("\n===== TODO APP =====");
    println!("1. List all tasks");
    println!("2. Add a new task");
    println!("3. Mark a task as completed");
    println!("4. Remove a task");
    println!("5. Exit");
    println!("===================");
}

fn get_input(prompt: &str) -> String {
    print!("{}", prompt);
    io::stdout().flush().unwrap();
    
    let mut input = String::new();
    io::stdin().read_line(&mut input).unwrap();
    input
}

fn list_tasks(task_list: &TaskList) {
    let tasks = task_list.tasks();
    
    if tasks.is_empty() {
        println!("No tasks found.");
        return;
    }
    
    println!("\nTasks:");
    for task in tasks {
        let status = if task.is_completed() { "✓" } else { " " };
        println!("[{}] {} - {}", status, task.id(), task.title());
    }
}

fn add_task(task_list: &mut TaskList, storage: &impl Storage) {
    let title = get_input("Enter task title: ");
    let task = task_list.add_task(title.trim());
    
    println!("Task added: {}", task.title());
    save_tasks(task_list, storage);
}

fn complete_task(task_list: &mut TaskList, storage: &impl Storage) {
    let id_str = get_input("Enter task ID to mark as completed: ");
    let id = match id_str.trim().parse::<u64>() {
        Ok(id) => id,
        Err(_) => {
            println!("Invalid ID format.");
            return;
        }
    };
    
    match task_list.get_task_mut(id) {
        Some(task) => {
            task.mark_completed();
            println!("Task marked as completed: {}", task.title());
            save_tasks(task_list, storage);
        }
        None => println!("Task with ID {} not found.", id),
    }
}

fn remove_task(task_list: &mut TaskList, storage: &impl Storage) {
    let id_str = get_input("Enter task ID to remove: ");
    let id = match id_str.trim().parse::<u64>() {
        Ok(id) => id,
        Err(_) => {
            println!("Invalid ID format.");
            return;
        }
    };
    
    match task_list.remove_task(id) {
        Some(task) => {
            println!("Task removed: {}", task.title());
            save_tasks(task_list, storage);
        }
        None => println!("Task with ID {} not found.", id),
    }
}

fn save_tasks(task_list: &TaskList, storage: &impl Storage) {
    match storage.save(task_list) {
        Ok(_) => println!("Tasks saved successfully."),
        Err(e) => println!("Error saving tasks: {}", e),
    }
}
```

### Common Pitfalls
- **Pitfall**: Forgetting to make items public with `pub`.
  - **Solution**: Remember that all items in Rust are private by default. Use `pub` for anything that needs to be accessed outside its module.
- **Pitfall**: Confusion between module declarations and use statements.
  - **Solution**: `mod math;` declares/imports a module, while `use math::add;` brings a specific item into scope.
- **Pitfall**: Incorrect file structure for multi-file modules.
  - **Solution**: Follow Rust's conventions: `foo.rs` or `foo/mod.rs` for a module named `foo`.
- **Pitfall**: Circular dependencies between modules.
  - **Solution**: Refactor code to eliminate cyclical references or use forward declarations.
- **Pitfall**: Importing too many items individually.
  - **Solution**: Use grouped imports `use std::{fs, io, path};` or glob imports `use module::*;` (but use the latter sparingly).

### Confusion Questions with Answers
1. **Q**: What's the difference between `mod` and `use` keywords?
   **A**: `mod` declares or imports a module, making its contents available through a path (e.g., `mod math;` lets you access `math::add()`). `use` brings specific items from modules into the current scope to avoid writing the full path each time (e.g., `use math::add;` allows writing just `add()`).

2. **Q**: Why can't I access a public function in a submodule when I've imported the parent?
   **A**: You need to make both the module and the item public. For example, in `mod parent { mod child { pub fn foo() {} } }`, `foo` is public within `child`, but `child` itself is private. Use `pub mod child` to make the submodule accessible.

3. **Q**: What's the difference between using a separate file and using `mod` inside another file?
   **A**: They're functionally equivalent; it's an organizational choice. Inline modules with `mod {}` keep related code together in one file, while separate files make larger codebases more manageable. The module hierarchy is the same either way.

---

## 2. Visibility Rules and Access Control

### Concise Explanation
Rust provides fine-grained control over visibility of items (functions, types, modules, etc.) through several visibility modifiers:

- Default (no modifier): Private to the current module and its child modules
- `pub`: Public to anyone who can access the parent module
- `pub(crate)`: Public within the current crate only
- `pub(super)`: Public to the parent module only
- `pub(in path)`: Public only to a specific path

These visibility rules apply to all items in Rust (functions, structs, enums, modules, etc.) and help enforce encapsulation and create clean API boundaries.

### Where to Use
- `pub`: For APIs you want to expose to users of your crate
- `pub(crate)`: For items you want to share across your crate but not expose externally
- `pub(super)`: For items that should only be accessible to the parent module
- `pub(in path)`: For items that should only be accessible to a specific module path
- Default (private): For implementation details that shouldn't be accessed directly

### Code Snippet
```rust
// src/lib.rs
pub mod shapes; // Public module that users can access
mod internal; // Private module, only accessible within this crate

pub use shapes::Rectangle; // Re-export Rectangle for a cleaner API

// Private function, only accessible in this file
fn helper() {
    println!("This is a private helper function");
}

// src/shapes.rs
// This is a public struct in a public module - fully accessible
pub struct Rectangle {
    pub width: f64,  // Public field
    pub height: f64, // Public field
}

// This struct has restricted visibility
pub(crate) struct Circle {
    pub radius: f64,
}

// This enum is only visible to the parent module
pub(super) enum ShapeType {
    Rect,
    Circ,
}

impl Rectangle {
    // Public constructor
    pub fn new(width: f64, height: f64) -> Self {
        Rectangle { width, height }
    }
    
    // Public method
    pub fn area(&self) -> f64 {
        self.width * self.height
    }
    
    // Private method, only accessible within Rectangle impl blocks
    fn validate(&self) -> bool {
        self.width > 0.0 && self.height > 0.0
    }
}

impl Circle {
    // Public within the crate only
    pub(crate) fn new(radius: f64) -> Self {
        Circle { radius }
    }
    
    // Public within the crate only
    pub(crate) fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
}

// src/internal.rs
use super::shapes::{Rectangle, Circle, ShapeType}; // Can access crate-visible and super-visible items

pub(crate) struct ShapeFactory {
    default_size: f64,
}

impl ShapeFactory {
    pub(crate) fn new(default_size: f64) -> Self {
        ShapeFactory { default_size }
    }
    
    pub(crate) fn create_rectangle(&self) -> Rectangle {
        Rectangle::new(self.default_size, self.default_size)
    }
    
    pub(crate) fn create_circle(&self) -> Circle {
        Circle::new(self.default_size / 2.0)
    }
    
    pub(crate) fn get_shape_type(&self, is_circle: bool) -> ShapeType {
        if is_circle {
            ShapeType::Circ
        } else {
            ShapeType::Rect
        }
    }
}
```

**Usage from outside the crate:**

```rust
// In an external crate using the above library
use shapes_lib::Rectangle;

fn main() {
    // Can use Rectangle because it's public and re-exported
    let rect = Rectangle::new(5.0, 10.0);
    println!("Rectangle area: {}", rect.area());
    
    // Can access public fields
    println!("Width: {}, Height: {}", rect.width, rect.height);
    
    // These would fail:
    // rect.validate(); // Error: private method
    // let circle = shapes_lib::shapes::Circle::new(5.0); // Error: Circle is pub(crate)
    // let factory = shapes_lib::internal::ShapeFactory::new(10.0); // Error: internal is private
}
```

### Real-World Example
A database connection library with visibility controls:

```rust
// src/lib.rs
pub mod connection;
pub mod query;
mod config;
mod utils;

pub use connection::Connection;
pub use query::QueryBuilder;

// Re-export error types for users
mod error;
pub use error::{Error, Result};

// src/error.rs
// Public error type for users of the library
#[derive(Debug)]
pub enum Error {
    ConnectionFailed(String),
    QueryFailed(String),
    InvalidConfig(String),
}

// Type alias for convenience
pub type Result<T> = std::result::Result<T, Error>;

// src/config.rs
// Configuration is only visible within the crate
pub(crate) struct Config {
    pub(crate) host: String,
    pub(crate) port: u16,
    pub(crate) username: String,
    pub(crate) password: String,
    pub(crate) database: String,
    pub(crate) pool_size: u32,
    pub(crate) connection_timeout: u64,
}

impl Config {
    pub(crate) fn from_env() -> Result<Self> {
        use std::env;
        
        Ok(Config {
            host: env::var("DB_HOST").unwrap_or_else(|_| "localhost".to_string()),
            port: env::var("DB_PORT")
                .ok()
                .and_then(|s| s.parse().ok())
                .unwrap_or(5432),
            username: env::var("DB_USER").unwrap_or_else(|_| "postgres".to_string()),
            password: env::var("DB_PASSWORD").map_err(|_| {
                Error::InvalidConfig("DB_PASSWORD environment variable not set".to_string())
            })?,
            database: env::var("DB_NAME").unwrap_or_else(|_| "postgres".to_string()),
            pool_size: env::var("DB_POOL_SIZE")
                .ok()
                .and_then(|s| s.parse().ok())
                .unwrap_or(10),
            connection_timeout: env::var("DB_TIMEOUT")
                .ok()
                .and_then(|s| s.parse().ok())
                .unwrap_or(30),
        })
    }
    
    pub(crate) fn connection_string(&self) -> String {
        format!(
            "postgres://{}:{}@{}:{}/{}",
            self.username, self.password, self.host, self.port, self.database
        )
    }
}

// src/utils.rs
// Utilities only used internally
pub(crate) fn format_query(query: &str, params: &[&str]) -> String {
    let mut result = query.to_string();
    
    for (i, param) in params.iter().enumerate() {
        let placeholder = format!("${}", i + 1);
        result = result.replace(&placeholder, param);
    }
    
    result
}

// Only visible to the super module (lib.rs)
pub(super) fn log_query(query: &str) {
    println!("QUERY: {}", query);
}

// src/connection.rs
use crate::config::Config;
use crate::error::{Error, Result};
use crate::utils;

#[derive(Debug)]
pub struct Connection {
    config: Config,
    // Other connection fields would go here
    connected: bool,
}

impl Connection {
    pub fn new() -> Result<Self> {
        let config = Config::from_env()?;
        Ok(Connection {
            config,
            connected: false,
        })
    }
    
    pub fn connect(&mut self) -> Result<()> {
        // In a real implementation, this would actually connect to a database
        println!("Connecting to {}", self.config.connection_string());
        
        // Log query using util function (only visible within the crate)
        utils::log_query("SELECT 1");
        
        self.connected = true;
        Ok(())
    }
    
    pub fn execute(&self, query: &str) -> Result<u64> {
        if !self.connected {
            return Err(Error::ConnectionFailed("Not connected".to_string()));
        }
        
        // Use internal utility
        let formatted = utils::format_query(query, &["value1", "value2"]);
        println!("Executing: {}", formatted);
        
        // Simulate query execution
        Ok(42)
    }
    
    // Method only visible within the crate
    pub(crate) fn get_config(&self) -> &Config {
        &self.config
    }
}

// src/query.rs
use crate::connection::Connection;
use crate::error::{Error, Result};

pub struct QueryBuilder {
    query: String,
    params: Vec<String>,
}

impl QueryBuilder {
    pub fn new() -> Self {
        QueryBuilder {
            query: String::new(),
            params: Vec::new(),
        }
    }
    
    pub fn select(mut self, fields: &[&str]) -> Self {
        self.query = format!("SELECT {} ", fields.join(", "));
        self
    }
    
    pub fn from(mut self, table: &str) -> Self {
        self.query.push_str(&format!("FROM {} ", table));
        self
    }
    
    pub fn where_clause(mut self, condition: &str) -> Self {
        self.query.push_str(&format!("WHERE {} ", condition));
        self
    }
    
    pub fn param(mut self, value: &str) -> Self {
        self.params.push(value.to_string());
        self
    }
    
    pub fn build(self) -> String {
        self.query
    }
    
    pub fn execute(self, conn: &Connection) -> Result<u64> {
        // Here we're using the Connection's public interface
        conn.execute(&self.build())
    }
    
    // This could be used within the crate to inspect the config
    // Not accessible to users of the library
    pub(crate) fn debug_with_config(self, conn: &Connection) {
        let config = conn.get_config(); // Using crate-visible method
        println!("Query: {} on database {}", self.query, config.database);
    }
}
```

### Common Pitfalls
- **Pitfall**: Exposing implementation details by making too many items public.
  - **Solution**: Start with the minimum visibility needed and only increase it when necessary.
- **Pitfall**: Not making struct fields public when you want them to be read/written directly.
  - **Solution**: Use `pub` on struct fields individually; structs can be public but have private fields.
- **Pitfall**: Trying to access a public item in a private module from outside.
  - **Solution**: Both the module and the item must be public for external access.
- **Pitfall**: Using `pub(crate)` when you actually want to expose the API to users.
  - **Solution**: Use `pub` for any API element users of your crate should be able to access.
- **Pitfall**: Making implementation details public and then regretting it when you want to change them.
  - **Solution**: Consider using getter/setter methods and keeping fields private for better encapsulation.

### Confusion Questions with Answers
1. **Q**: What's the difference between `pub` and `pub(crate)`?
   **A**: `pub` makes an item accessible to any code that can access the module (including external crates), while `pub(crate)` restricts access to only within the current crate. Use `pub(crate)` for items that should be shared across your crate but not exposed as part of your public API.

2. **Q**: Can I have a private field in a public struct?
   **A**: Yes. Each field has its own visibility control. A common pattern is a public struct with private fields and public methods to access or modify those fields (getters and setters), which gives you more control over how the struct is used.

3. **Q**: If I make a module public, are all its contents automatically public too?
   **A**: No. Making a module public only means that other modules can access it, but the visibility of each item inside the module is controlled separately. You need to mark each item with the appropriate visibility modifier.

---

## 3. Module Structure and Hierarchies

### Concise Explanation
Rust modules form a hierarchical structure starting from the crate root (`src/main.rs` or `src/lib.rs`). Modules can be nested within other modules, creating a tree-like structure. This hierarchy determines the path you use to refer to items within modules.

There are two primary approaches to structuring modules:
1. **In-line modules**: Defining modules directly within files using `mod` blocks
2. **File-based modules**: Separating modules into their own files or directories

For larger projects, a well-designed module hierarchy improves code organization and maintainability.

### Where to Use
- When organizing large codebases with many components
- To separate distinct functional areas of your application
- For creating logical boundaries around related functionality
- When designing library APIs with clear structure
- To manage public/private interfaces effectively

### Code Snippet
**Basic module hierarchy:**

```rust
// src/lib.rs - The crate root
pub mod app;            // Module in src/app.rs or src/app/mod.rs
pub mod utils;          // Module in src/utils.rs or src/utils/mod.rs

// src/app.rs or src/app/mod.rs
pub mod models;         // submodule in src/app/models.rs or src/app/models/mod.rs
pub mod controllers;    // submodule in src/app/controllers.rs or src/app/controllers/mod.rs

// src/app/models.rs or src/app/models/mod.rs
pub struct User {
    pub id: u64,
    pub name: String,
}

// src/app/controllers.rs or src/app/controllers/mod.rs
pub mod auth;           // submodule in src/app/controllers/auth.rs
pub mod users;          // submodule in src/app/controllers/users.rs

// src/app/controllers/users.rs
use crate::app::models::User;

pub fn get_user(id: u64) -> Option<User> {
    // Implementation...
    Some(User { id, name: "Test User".to_string() })
}
```

**Access patterns for nested modules:**

```rust
// src/main.rs
use my_crate::app::controllers::users::get_user;
use my_crate::app::models::User;
use my_crate::utils;

fn main() {
    // Full path
    let user1 = my_crate::app::controllers::users::get_user(1);
    
    // Using imports
    let user2 = get_user(2);
    
    if let Some(user) = user1 {
        println!("Found user: {}", user.name);
    }
    
    // Creating new user
    let new_user = User {
        id: 3,
        name: String::from("New User"),
    };
    
    utils::log(&format!("Created user: {}", new_user.name));
}
```

**Module with submodules in a single file:**

```rust
// src/config.rs
pub mod database {
    pub struct DatabaseConfig {
        pub url: String,
        pub max_connections: u32,
    }
    
    impl DatabaseConfig {
        pub fn new(url: &str) -> Self {
            DatabaseConfig {
                url: url.to_string(),
                max_connections: 10,
            }
        }
    }
    
    // Private helper
    fn validate_url(url: &str) -> bool {
        url.starts_with("postgres://") || url.starts_with("mysql://")
    }
}

pub mod logging {
    pub enum LogLevel {
        Debug,
        Info,
        Warning,
        Error,
    }
    
    pub struct LogConfig {
        pub level: LogLevel,
        pub file: Option<String>,
    }
    
    impl LogConfig {
        pub fn new(level: LogLevel) -> Self {
            LogConfig {
                level,
                file: None,
            }
        }
        
        pub fn with_file(mut self, path: &str) -> Self {
            self.file = Some(path.to_string());
            self
        }
    }
}

// Usage within the same crate
pub fn load_config() -> (database::DatabaseConfig, logging::LogConfig) {
    let db_config = database::DatabaseConfig::new("postgres://localhost/myapp");
    let log_config = logging::LogConfig::new(logging::LogLevel::Info)
        .with_file("app.log");
    
    (db_config, log_config)
}
```

### Real-World Example
A more complex web API project structure:

```
src/
├── main.rs           # Application entry point
├── lib.rs            # Library entry point
├── config/           # Configuration module
│   ├── mod.rs
│   ├── app.rs
│   └── database.rs
├── models/           # Data models
│   ├── mod.rs
│   ├── user.rs
│   └── post.rs
├── services/         # Business logic
│   ├── mod.rs
│   ├── auth.rs
│   └── posts.rs
├── repositories/     # Data access
│   ├── mod.rs
│   ├── user_repo.rs
│   └── post_repo.rs
├── api/              # API routes and handlers
│   ├── mod.rs
│   ├── middleware.rs
│   ├── auth.rs
│   └── posts.rs
└── utils/            # Utility functions
    ├── mod.rs
    ├── validation.rs
    └── logging.rs
```

```rust
// src/lib.rs
pub mod config;
pub mod models;
pub mod services;
pub mod repositories;
pub mod api;
pub mod utils;

// Convenience re-exports for commonly used items
pub use api::create_server;
pub use config::AppConfig;
pub use models::{User, Post};

// Public error type for the entire crate
pub type Result<T> = std::result::Result<T, Box<dyn std::error::Error>>;

// src/config/mod.rs
mod app;
mod database;

pub use app::AppConfig;
pub use database::DatabaseConfig;

// src/models/mod.rs
mod user;
mod post;

pub use user::User;
pub use post::Post;

// src/models/user.rs
use serde::{Deserialize, Serialize};
use uuid::Uuid;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct User {
    pub id: Uuid,
    pub username: String,
    pub email: String,
    #[serde(skip_serializing)]
    password_hash: String,
    pub created_at: chrono::DateTime<chrono::Utc>,
}

impl User {
    pub fn new(username: &str, email: &str, password: &str) -> Self {
        User {
            id: Uuid::new_v4(),
            username: username.to_string(),
            email: email.to_string(),
            password_hash: hash_password(password),
            created_at: chrono::Utc::now(),
        }
    }
    
    pub fn verify_password(&self, password: &str) -> bool {
        verify_password(password, &self.password_hash)
    }
}

// Private helper functions
fn hash_password(password: &str) -> String {
    // In a real app, use a proper password hashing library
    format!("hashed_{}", password)
}

fn verify_password(password: &str, hash: &str) -> bool {
    hash == format!("hashed_{}", password)
}

// src/repositories/mod.rs
mod user_repo;
mod post_repo;

pub use user_repo::UserRepository;
pub use post_repo::PostRepository;

// src/repositories/user_repo.rs
use crate::models::User;
use crate::config::DatabaseConfig;
use crate::Result;
use uuid::Uuid;
use std::collections::HashMap;
use std::sync::{Arc, Mutex};

pub struct UserRepository {
    db_config: DatabaseConfig,
    // This would be a database connection in a real app
    // Using a simple in-memory store for the example
    users: Arc<Mutex<HashMap<Uuid, User>>>,
}

impl UserRepository {
    pub fn new(db_config: DatabaseConfig) -> Self {
        UserRepository {
            db_config,
            users: Arc::new(Mutex::new(HashMap::new())),
        }
    }
    
    pub async fn find_by_id(&self, id: Uuid) -> Result<Option<User>> {
        // In a real app, this would query a database
        let users = self.users.lock().unwrap();
        Ok(users.get(&id).cloned())
    }
    
    pub async fn find_by_username(&self, username: &str) -> Result<Option<User>> {
        let users = self.users.lock().unwrap();
        Ok(users.values()
            .find(|u| u.username == username)
            .cloned())
    }
    
    pub async fn create(&self, user: User) -> Result<User> {
        let mut users = self.users.lock().unwrap();
        let user_clone = user.clone();
        users.insert(user.id, user);
        Ok(user_clone)
    }
}

// src/services/mod.rs
mod auth;
mod posts;

pub use auth::AuthService;
pub use posts::PostService;

// src/services/auth.rs
use crate::models::User;
use crate::repositories::UserRepository;
use crate::utils::validation;
use crate::Result;
use jsonwebtoken::{encode, Header, EncodingKey};
use serde::{Serialize, Deserialize};
use std::sync::Arc;

#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    sub: String,
    exp: usize,
}

pub struct AuthService {
    user_repo: Arc<UserRepository>,
    jwt_secret: String,
}

impl AuthService {
    pub fn new(user_repo: Arc<UserRepository>, jwt_secret: &str) -> Self {
        AuthService {
            user_repo,
            jwt_secret: jwt_secret.to_string(),
        }
    }
    
    pub async fn register(
        &self,
        username: &str,
        email: &str,
        password: &str
    ) -> Result<User> {
        // Validate input
        validation::validate_username(username)?;
        validation::validate_email(email)?;
        validation::validate_password(password)?;
        
        // Check if username already exists
        if let Some(_) = self.user_repo.find_by_username(username).await? {
            return Err("Username already taken".into());
        }
        
        // Create user
        let user = User::new(username, email, password);
        let created_user = self.user_repo.create(user).await?;
        
        Ok(created_user)
    }
    
    pub async fn login(&self, username: &str, password: &str) -> Result<String> {
        // Find user
        let user = match self.user_repo.find_by_username(username).await? {
            Some(user) => user,
            None => return Err("Invalid username or password".into()),
        };
        
        // Verify password
        if !user.verify_password(password) {
            return Err("Invalid username or password".into());
        }
        
        // Generate JWT
        let claims = Claims {
            sub: user.id.to_string(),
            exp: (chrono::Utc::now() + chrono::Duration::hours(24)).timestamp() as usize,
        };
        
        let token = encode(
            &Header::default(),
            &claims,
            &EncodingKey::from_secret(self.jwt_secret.as_bytes()),
        )?;
        
        Ok(token)
    }
}

// src/api/mod.rs
mod middleware;
mod auth;
mod posts;

use crate::config::AppConfig;
use crate::services::{AuthService, PostService};
use std::sync::Arc;

// Re-export the function to create the server
pub use self::middleware::create_server;

// src/api/middleware.rs
use crate::config::AppConfig;
use crate::services::{AuthService, PostService};
use crate::repositories::{UserRepository, PostRepository};
use std::sync::Arc;

pub fn create_server(config: AppConfig) -> Server {
    // Initialize repositories
    let user_repo = Arc::new(UserRepository::new(config.database.clone()));
    let post_repo = Arc::new(PostRepository::new(config.database.clone()));
    
    // Initialize services
    let auth_service = Arc::new(AuthService::new(
        Arc::clone(&user_repo),
        &config.jwt_secret,
    ));
    let post_service = Arc::new(PostService::new(
        Arc::clone(&post_repo),
        Arc::clone(&user_repo),
    ));
    
    // Create the server (this would use a web framework in a real app)
    let server = Server::new(
        auth_service,
        post_service,
    );
    
    server
}

// Placeholder for a web server
pub struct Server {
    auth_service: Arc<AuthService>,
    post_service: Arc<PostService>,
}

impl Server {
    fn new(
        auth_service: Arc<AuthService>,
        post_service: Arc<PostService>,
    ) -> Self {
        Server {
            auth_service,
            post_service,
        }
    }
    
    pub fn start(&self, host: &str, port: u16) {
        println!("Server starting on {}:{}", host, port);
        // In a real app, this would start the web server
    }
}
```

### Common Pitfalls
- **Pitfall**: Creating overly complex module hierarchies that are hard to navigate.
  - **Solution**: Keep your module hierarchy relatively flat and focused on logical separation of concerns.
- **Pitfall**: Inconsistent module naming or organization patterns.
  - **Solution**: Establish and follow consistent naming conventions for your modules.
- **Pitfall**: Circular dependencies between modules.
  - **Solution**: Refactor to break dependency cycles, possibly introducing a new module both depend on.
- **Pitfall**: Mixing in-line and file-based modules inconsistently.
  - **Solution**: Choose one approach for most of your code, using the other only when it makes sense.
- **Pitfall**: Forgetting to update module declarations when moving files.
  - **Solution**: Start with a clear structure and be careful when refactoring module organization.

### Confusion Questions with Answers
1. **Q**: How do I reference an item in a parent or sibling module?
   **A**: You can use:
   - `super::` to access the parent module
   - `crate::` to access from the crate root
   - Absolute paths from the crate root: `crate::path::to::item`
   - Relative paths: `super::sibling_module::item`

2. **Q**: What's the best way to structure a large Rust project with many modules?
   **A**: Generally:
   - Organize by feature/domain area rather than by type
   - Keep related functionality together
   - Use a relatively flat hierarchy with 2-3 levels of nesting
   - Create clear boundaries between major subsystems
   - Re-export commonly used items at higher levels for convenience

3. **Q**: Is there a way to import all items from a module without listing each one?
   **A**: Yes, you can use a glob import with `use module_name::*;`. However, this is generally discouraged except in tests or when you know exactly what you're importing, as it can make it unclear which items are in scope.

---

## 4. The mod.rs Pattern

### Concise Explanation
The "mod.rs pattern" refers to a convention for organizing Rust modules where a directory contains a special file named `mod.rs` that serves as the root of that module, along with other files that contain submodules.

There are two main approaches to file-based modules in Rust:
1. **The mod.rs pattern**: Using `mod.rs` files within directories
   ```
   src/
   ├── models/
   │   ├── mod.rs         # models module
   │   ├── user.rs        # models::user module
   │   └── post.rs        # models::post module
   ```

2. **The newer "module file" pattern** (introduced in Rust 2018): Using files with the same name as the module
   ```
   src/
   ├── models.rs          # models module
   ├── models/
   │   ├── user.rs        # models::user module
   │   └── post.rs        # models::post module
   ```

Both approaches work and have their own advantages and disadvantages.

### Where to Use
- Use the mod.rs pattern when:
  - You have a clear conceptual separation of modules
  - Each module includes several related submodules
  - You prefer having all module declarations in a single place
- Use the newer file-based pattern when:
  - You want to avoid the name `mod.rs` appearing multiple times in editor tabs
  - You prefer seeing the module name in file explorers
  - You want to follow more recent Rust conventions

### Code Snippet
**Example using the mod.rs pattern:**

```
src/
├── lib.rs
├── models/
│   ├── mod.rs
│   ├── user.rs
│   └── post.rs
```

```rust
// src/lib.rs
pub mod models;

// src/models/mod.rs
// This file defines what's in the models module and imports submodules
pub mod user;
pub mod post;

// Re-exports for convenience
pub use user::User;
pub use post::Post;

// Common functionality for all models
pub trait Model {
    fn validate(&self) -> bool;
}

// src/models/user.rs
use super::Model;

pub struct User {
    pub id: u64,
    pub name: String,
    pub email: String,
}

impl User {
    pub fn new(name: &str, email: &str) -> Self {
        User {
            id: 0, // Would be generated in a real app
            name: name.to_string(),
            email: email.to_string(),
        }
    }
}

impl Model for User {
    fn validate(&self) -> bool {
        !self.name.is_empty() && self.email.contains('@')
    }
}

// src/models/post.rs
use super::Model;
use super::user::User;

pub struct Post {
    pub id: u64,
    pub title: String,
    pub content: String,
    pub author: u64, // User ID
}

impl Post {
    pub fn new(title: &str, content: &str, author: &User) -> Self {
        Post {
            id: 0,
            title: title.to_string(),
            content: content.to_string(),
            author: author.id,
        }
    }
}

impl Model for Post {
    fn validate(&self) -> bool {
        !self.title.is_empty() && !self.content.is_empty()
    }
}
```

**Example using the newer file-based pattern:**

```
src/
├── lib.rs
├── models.rs
├── models/
│   ├── user.rs
│   └── post.rs
```

```rust
// src/lib.rs
pub mod models;

// src/models.rs
// This file defines what's in the models module and imports submodules
pub mod user;
pub mod post;

// Re-exports for convenience
pub use self::user::User;
pub use self::post::Post;

// Common functionality for all models
pub trait Model {
    fn validate(&self) -> bool;
}

// src/models/user.rs
use crate::models::Model;

pub struct User {
    pub id: u64,
    pub name: String,
    pub email: String,
}

impl User {
    pub fn new(name: &str, email: &str) -> Self {
        User {
            id: 0,
            name: name.to_string(),
            email: email.to_string(),
        }
    }
}

impl Model for User {
    fn validate(&self) -> bool {
        !self.name.is_empty() && self.email.contains('@')
    }
}

// src/models/post.rs
use crate::models::Model;
use crate::models::user::User;

pub struct Post {
    pub id: u64,
    pub title: String,
    pub content: String,
    pub author: u64,
}

impl Post {
    pub fn new(title: &str, content: &str, author: &User) -> Self {
        Post {
            id: 0,
            title: title.to_string(),
            content: content.to_string(),
            author: author.id,
        }
    }
}

impl Model for Post {
    fn validate(&self) -> bool {
        !self.title.is_empty() && !self.content.is_empty()
    }
}
```

### Real-World Example
A configuration module using the mod.rs pattern:

```
src/
├── lib.rs
├── config/
│   ├── mod.rs
│   ├── database.rs
│   ├── server.rs
│   ├── logging.rs
│   └── auth.rs
```

```rust
// src/lib.rs
pub mod config;

// Re-export the Config struct for convenience
pub use config::Config;

// src/config/mod.rs
mod database;
mod server;
mod logging;
mod auth;

// Re-export specific types
pub use database::DatabaseConfig;
pub use server::ServerConfig;
pub use logging::LogConfig;
pub use auth::AuthConfig;

use std::fs;
use std::path::Path;
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct Config {
    pub database: DatabaseConfig,
    pub server: ServerConfig,
    pub logging: LogConfig,
    pub auth: AuthConfig,
}

impl Config {
    pub fn from_file(path: &str) -> Result<Self, Box<dyn std::error::Error>> {
        let config_str = fs::read_to_string(path)?;
        let config = serde_json::from_str(&config_str)?;
        Ok(config)
    }
    
    pub fn default() -> Self {
        Config {
            database: DatabaseConfig::default(),
            server: ServerConfig::default(),
            logging: LogConfig::default(),
            auth: AuthConfig::default(),
        }
    }
    
    pub fn save_to_file(&self, path: &str) -> Result<(), Box<dyn std::error::Error>> {
        let config_str = serde_json::to_string_pretty(self)?;
        fs::write(path, config_str)?;
        Ok(())
    }
}

// src/config/database.rs
use serde::{Serialize, Deserialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct DatabaseConfig {
    pub host: String,
    pub port: u16,
    pub username: String,
    pub password: String,
    pub database_name: String,
    pub max_connections: u32,
}

impl Default for DatabaseConfig {
    fn default() -> Self {
        DatabaseConfig {
            host: "localhost".to_string(),
            port: 5432,
            username: "postgres".to_string(),
            password: "".to_string(),
            database_name: "app".to_string(),
            max_connections: 10,
        }
    }
}

impl DatabaseConfig {
    pub fn connection_string(&self) -> String {
        format!(
            "postgres://{}:{}@{}:{}/{}",
            self.username, self.password, self.host, self.port, self.database_name
        )
    }
}

// src/config/server.rs
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct ServerConfig {
    pub host: String,
    pub port: u16,
    pub workers: usize,
    pub timeout: u64,
}

impl Default for ServerConfig {
    fn default() -> Self {
        ServerConfig {
            host: "127.0.0.1".to_string(),
            port: 8080,
            workers: 4,
            timeout: 30,
        }
    }
}

// src/config/logging.rs
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
pub enum LogLevel {
    Debug,
    Info,
    Warning,
    Error,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct LogConfig {
    pub level: LogLevel,
    pub file: Option<String>,
    pub console: bool,
}

impl Default for LogConfig {
    fn default() -> Self {
        LogConfig {
            level: LogLevel::Info,
            file: Some("app.log".to_string()),
            console: true,
        }
    }
}

// src/config/auth.rs
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct AuthConfig {
    pub jwt_secret: String,
    pub token_expiry_hours: u32,
    pub refresh_token_expiry_days: u32,
}

impl Default for AuthConfig {
    fn default() -> Self {
        AuthConfig {
            jwt_secret: "change_me_in_production".to_string(),
            token_expiry_hours: 24,
            refresh_token_expiry_days: 30,
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Confusion when switching between projects using different patterns.
  - **Solution**: Maintain consistency within a single project and document your module approach.
- **Pitfall**: Many files named `mod.rs` can be confusing in editors.
  - **Solution**: Consider using the newer file-based pattern, especially for larger projects.
- **Pitfall**: Forgetting to declare submodules in the parent module.
  - **Solution**: Always add `mod submodule_name;` in the parent module (mod.rs or parent.rs).
- **Pitfall**: Incorrect `use` paths when refactoring between patterns.
  - **Solution**: Double-check imports after refactoring and use the compiler's help to fix errors.
- **Pitfall**: Confusion about relative paths with `super::` in different module patterns.
  - **Solution**: Consider using more absolute paths (`crate::`) when module hierarchy is complex.

### Confusion Questions with Answers
1. **Q**: Which pattern should I use, mod.rs or the newer file-based pattern?
   **A**: Both are valid and widely used. The newer file-based pattern (`module.rs` with a `module/` directory) is generally preferred in new projects because it avoids multiple files with the same name and shows module names in file explorers. However, the mod.rs pattern is still common and works well for conceptually encapsulated modules.

2. **Q**: Do I need to create a mod.rs file for every directory of Rust code?
   **A**: In the mod.rs pattern, yes - each directory that represents a module needs a mod.rs file. In the newer pattern, you'd have a `module.rs` file and a `module/` directory instead. Either way, you need to declare the module in its parent with `mod module_name;`.

3. **Q**: How do I convert from the mod.rs pattern to the newer file-based pattern?
   **A**: For each `module/mod.rs` file:
   1. Create a new file named `module.rs` at the same level as the `module/` directory
   2. Move the contents of `module/mod.rs` to `module.rs`
   3. Update any relative paths using `super::` or `self::` as needed
   4. Delete the original `mod.rs` file
   5. Keep the submodule files in the `module/` directory

---

## 5. Standard Library Module Organization

### Concise Explanation
Rust's standard library (`std`) is organized into modules that group related functionality. Understanding this organization helps you find the right types and functions when you need them, and also serves as a good example of module design.

Key modules in the standard library include:
- `std::collections`: Data structures like HashMap, Vec, etc.
- `std::fs`, `std::io`: File system and I/O operations
- `std::path`: Path manipulation
- `std::sync`, `std::thread`: Concurrency primitives
- `std::time`: Time-related functionality
- `std::net`: Networking
- `std::error`: Error handling
- `std::fmt`: Formatting and display

The standard library makes extensive use of re-exports to create a convenient API surface while maintaining logical organization internally.

### Where to Use
- When you need standard types and functions for common tasks
- As a reference for your own module organization
- To learn idiomatic Rust patterns
- When implementing traits from the standard library
- For understanding how to design clear, well-structured APIs

### Code Snippet
```rust
// Importing from various std modules
use std::collections::HashMap;
use std::fs::File;
use std::io::{self, Read, Write};
use std::path::{Path, PathBuf};
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::{Duration, Instant};

fn main() -> io::Result<()> {
    // Using collections
    let mut map = HashMap::new();
    map.insert("key1", "value1");
    map.insert("key2", "value2");
    println!("Map: {:?}", map);
    
    // File I/O
    let mut file = File::create("example.txt")?;
    file.write_all(b"Hello, world!")?;
    
    // Path manipulation
    let path = Path::new("example.txt");
    println!("File name: {:?}", path.file_name());
    
    // Time measurement
    let start = Instant::now();
    thread::sleep(Duration::from_millis(100));
    let elapsed = start.elapsed();
    println!("Operation took: {:?}", elapsed);
    
    // Threading with shared state
    let counter = Arc::new(Mutex::new(0));
    
    let mut handles = vec![];
    
    for _ in 0..5 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter_clone.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final count: {}", *counter.lock().unwrap());
    
    Ok(())
}
```

**Exploring std modules:**

```rust
// std::collections
use std::collections::{HashMap, HashSet, BTreeMap, VecDeque, LinkedList};

fn collections_examples() {
    // HashMap: Key-value pairs with O(1) average lookups
    let mut map = HashMap::new();
    map.insert("apple", 5);
    map.insert("banana", 8);
    println!("HashMap: {:?}", map);
    
    // HashSet: Unique values with O(1) average lookups
    let mut set = HashSet::new();
    set.insert("apple");
    set.insert("banana");
    set.insert("apple"); // Duplicate, won't be added
    println!("HashSet: {:?}", set);
    
    // BTreeMap: Sorted key-value pairs
    let mut btree = BTreeMap::new();
    btree.insert("banana", 8);
    btree.insert("apple", 5);
    btree.insert("cherry", 3);
    println!("BTreeMap (sorted by key): {:?}", btree);
    
    // VecDeque: Double-ended queue
    let mut queue = VecDeque::new();
    queue.push_back(1);
    queue.push_back(2);
    queue.push_front(0);
    println!("VecDeque: {:?}", queue);
    
    // LinkedList
    let mut list = LinkedList::new();
    list.push_back(1);
    list.push_back(2);
    list.push_front(0);
    println!("LinkedList: {:?}", list);
}

// std::fs and std::io
use std::fs::{self, File};
use std::io::{self, Read, Write, BufReader, BufWriter};

fn file_io_examples() -> io::Result<()> {
    // Create a directory
    fs::create_dir_all("example/nested")?;
    
    // Write to a file
    let mut file = File::create("example/test.txt")?;
    file.write_all(b"Hello, world!\n")?;
    
    // Append to a file
    let mut file = fs::OpenOptions::new()
        .append(true)
        .open("example/test.txt")?;
    file.write_all(b"Another line.\n")?;
    
    // Read a file to a string
    let content = fs::read_to_string("example/test.txt")?;
    println!("File content: {}", content);
    
    // Buffered reading
    let file = File::open("example/test.txt")?;
    let reader = BufReader::new(file);
    
    for line in reader.lines() {
        println!("Line: {}", line?);
    }
    
    // Buffered writing (more efficient for many small writes)
    let file = File::create("example/buffered.txt")?;
    let mut writer = BufWriter::new(file);
    
    for i in 0..10 {
        writeln!(writer, "Line {}", i)?;
    }
    writer.flush()?;
    
    // List directory contents
    for entry in fs::read_dir("example")? {
        let entry = entry?;
        println!("Found: {:?}", entry.path());
    }
    
    Ok(())
}

// std::path
use std::path::{Path, PathBuf};

fn path_examples() {
    // Create paths
    let path = Path::new("/usr/local/bin/program");
    
    // Path components
    println!("File name: {:?}", path.file_name());
    println!("Directory: {:?}", path.parent());
    println!("Extension: {:?}", path.extension());
    
    // PathBuf (owned, mutable path)
    let mut path_buf = PathBuf::from("/usr/local");
    path_buf.push("bin");
    path_buf.push("program");
    
    println!("Built path: {}", path_buf.display());
    
    // Path operations
    if path_buf.is_absolute() {
        println!("This is an absolute path");
    }
    
    // Path joining
    let config_path = Path::new("/etc").join("config").join("app.conf");
    println!("Config path: {}", config_path.display());
    
    // Windows vs. Unix paths
    #[cfg(windows)]
    {
        let windows_path = Path::new("C:\\Users\\username\\Documents");
        println!("Windows path: {}", windows_path.display());
    }
    
    #[cfg(unix)]
    {
        let unix_path = Path::new("/home/username/documents");
        println!("Unix path: {}", unix_path.display());
    }
    
    // Canonical paths (resolves symbolic links)
    match std::fs::canonicalize(".") {
        Ok(canonical) => println!("Current directory (canonical): {}", canonical.display()),
        Err(e) => println!("Error getting canonical path: {}", e),
    }
}

// std::sync and std::thread
use std::sync::{Arc, Mutex, RwLock, mpsc};
use std::thread;
use std::time::Duration;

fn concurrency_examples() {
    // Basic thread spawning
    let handle = thread::spawn(|| {
        println!("Hello from a thread!");
        thread::sleep(Duration::from_millis(100));
        println!("Thread finished");
    });
    
    // Wait for the thread to finish
    handle.join().unwrap();
    
    // Shared data with Arc (Atomic Reference Count) and Mutex
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for i in 0..5 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter_clone.lock().unwrap();
            *num += 1;
            println!("Thread {} increased counter to {}", i, *num);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final counter: {}", *counter.lock().unwrap());
    
    // Reader-writer lock for multiple readers OR one writer
    let data = Arc::new(RwLock::new(vec![1, 2, 3]));
    let mut reader_handles = vec![];
    let mut writer_handles = vec![];
    
    // Spawn readers
    for i in 0..3 {
        let data_clone = Arc::clone(&data);
        let handle = thread::spawn(move || {
            // Multiple readers can read simultaneously
            let data = data_clone.read().unwrap();
            println!("Reader {} sees: {:?}", i, *data);
            thread::sleep(Duration::from_millis(100));
        });
        reader_handles.push(handle);
    }
    
    // Spawn a writer
    let data_clone = Arc::clone(&data);
    let writer = thread::spawn(move || {
        thread::sleep(Duration::from_millis(50));
        // Only one writer can write at a time, and no readers can read while writing
        let mut data = data_clone.write().unwrap();
        data.push(4);
        println!("Writer updated data to: {:?}", *data);
    });
    
    // Join all threads
    for handle in reader_handles {
        handle.join().unwrap();
    }
    writer.join().unwrap();
    
    // Channels for message passing
    let (tx, rx) = mpsc::channel();
    
    // Spawn a sender thread
    thread::spawn(move || {
        for i in 1..5 {
            tx.send(format!("Message {}", i)).unwrap();
            thread::sleep(Duration::from_millis(50));
        }
    });
    
    // Receive messages
    for _ in 0..4 {
        match rx.recv() {
            Ok(msg) => println!("Received: {}", msg),
            Err(e) => println!("Error receiving: {}", e),
        }
    }
}

// std::time
use std::time::{Duration, Instant, SystemTime, UNIX_EPOCH};

fn time_examples() {
    // Duration: represents a span of time
    let duration = Duration::from_secs(2) + Duration::from_millis(500);
    println!("Duration: {:?}", duration);
    
    // Instant: for high-precision time measurement
    let start = Instant::now();
    // Simulate work
    thread::sleep(Duration::from_millis(100));
    let elapsed = start.elapsed();
    println!("Operation took: {:?}", elapsed);
    
    // SystemTime: for wall-clock time
    match SystemTime::now().duration_since(UNIX_EPOCH) {
        Ok(n) => println!("Seconds since UNIX_EPOCH: {}", n.as_secs()),
        Err(_) => println!("SystemTime before UNIX_EPOCH!"),
    }
    
    // One day in the future
    let tomorrow = SystemTime::now() + Duration::from_secs(86400);
    println!("Tomorrow: {:?}", tomorrow);
}

// std::fmt
use std::fmt;

struct Point {
    x: i32,
    y: i32,
}

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

impl fmt::Debug for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        f.debug_struct("Point")
            .field("x", &self.x)
            .field("y", &self.y)
            .finish()
    }
}

fn formatting_examples() {
    let point = Point { x: 1, y: 2 };
    
    // Different formatting options
    println!("Display: {}", point);         // Uses fmt::Display
    println!("Debug: {:?}", point);         // Uses fmt::Debug
    println!("Pretty debug: {:#?}", point); // Uses pretty-printed fmt::Debug
    
    // Format specifiers
    let pi = 3.14159265359;
    println!("Default: {}", pi);
    println!("Two decimal places: {:.2}", pi);
    println!("Width of 10: {:10}", pi);
    println!("Width of 10, 3 decimals: {:10.3}", pi);
    println!("Left-aligned, width 10: {:<10}", pi);
    println!("Right-aligned, width 10: {:>10}", pi);
    println!("Center-aligned, width 10: {:^10}", pi);
    println!("With sign: {:+}", pi);
    
    // Number formatting
    let number = 42;
    println!("Decimal: {}", number);
    println!("Hex: {:x}", number);
    println!("Octal: {:o}", number);
    println!("Binary: {:b}", number);
    println!("Hex with 0x: {:#x}", number);
    
    // Formatted strings
    let formatted = format!("A formatted {}: {}", "string", 123);
    println!("{}", formatted);
}

// std::error
use std::error::Error;
use std::fmt;

// Custom error type
#[derive(Debug)]
enum AppError {
    IoError(std::io::Error),
    ParseError(String),
    ValidationError(String),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            AppError::IoError(err) => write!(f, "I/O error: {}", err),
            AppError::ParseError(msg) => write!(f, "Parse error: {}", msg),
            AppError::ValidationError(msg) => write!(f, "Validation error: {}", msg),
        }
    }
}

impl Error for AppError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match self {
            AppError::IoError(err) => Some(err),
            _ => None,
        }
    }
}

// For automatic conversion from io::Error to our AppError
impl From<std::io::Error> for AppError {
    fn from(err: std::io::Error) -> Self {
        AppError::IoError(err)
    }
}

fn error_handling_examples() -> Result<(), AppError> {
    // Error propagation with ?
    let content = std::fs::read_to_string("nonexistent.txt")?;
    
    // Parse content
    let number = match content.trim().parse::<i32>() {
        Ok(n) => n,
        Err(_) => return Err(AppError::ParseError("Could not parse as integer".to_string())),
    };
    
    // Validate result
    if number < 0 {
        return Err(AppError::ValidationError("Number cannot be negative".to_string()));
    }
    
    println!("Successfully read and parsed: {}", number);
    Ok(())
}

// std::net
use std::net::{IpAddr, Ipv4Addr, Ipv6Addr, TcpListener, TcpStream};
use std::io::prelude::*;

fn networking_examples() -> io::Result<()> {
    // IP addresses
    let localhost_v4 = IpAddr::V4(Ipv4Addr::new(127, 0, 0, 1));
    let localhost_v6 = IpAddr::V6(Ipv6Addr::new(0, 0, 0, 0, 0, 0, 0, 1));
    
    println!("IPv4 localhost: {}", localhost_v4);
    println!("IPv6 localhost: {}", localhost_v6);
    
    // TCP listener (server)
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("Listening on port 8080...");
    
    // Spawn a client thread
    thread::spawn(|| {
        thread::sleep(Duration::from_millis(100));
        match TcpStream::connect("127.0.0.1:8080") {
            Ok(mut stream) => {
                println!("Client connected to server");
                stream.write_all(b"Hello from client").unwrap();
            },
            Err(e) => println!("Failed to connect: {}", e),
        }
    });
    
    // Accept one connection
    match listener.accept() {
        Ok((mut stream, addr)) => {
            println!("Connection established with: {}", addr);
            
            let mut buffer = [0; 1024];
            match stream.read(&mut buffer) {
                Ok(n) => {
                    let message = String::from_utf8_lossy(&buffer[0..n]);
                    println!("Received: {}", message);
                    
                    // Echo back
                    stream.write_all(b"Hello from server").unwrap();
                },
                Err(e) => println!("Error reading from connection: {}", e),
            }
        },
        Err(e) => println!("Error accepting connection: {}", e),
    }
    
    Ok(())
}

// Main function to demonstrate standard library modules
fn main() -> io::Result<()> {
    println!("=== Collections Examples ===");
    collections_examples();
    
    println!("\n=== File I/O Examples ===");
    file_io_examples()?;
    
    println!("\n=== Path Examples ===");
    path_examples();
    
    println!("\n=== Concurrency Examples ===");
    concurrency_examples();
    
    println!("\n=== Time Examples ===");
    time_examples();
    
    println!("\n=== Formatting Examples ===");
    formatting_examples();
    
    println!("\n=== Error Handling Examples ===");
    if let Err(e) = error_handling_examples() {
        println!("Error occurred: {}", e);
        if let Some(source) = e.source() {
            println!("Caused by: {}", source);
        }
    }
    
    println!("\n=== Networking Examples ===");
    networking_examples()?;
    
    Ok(())
}
```

### Real-World Example
A logging library making use of multiple standard library modules:

```rust
use std::collections::HashMap;
use std::fmt;
use std::fs::{self, File, OpenOptions};
use std::io::{self, Write};
use std::path::PathBuf;
use std::sync::{Arc, Mutex};
use std::time::{SystemTime, UNIX_EPOCH};

// Log level enum
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord)]
pub enum LogLevel {
    Trace,
    Debug,
    Info,
    Warning,
    Error,
    Critical,
}

impl fmt::Display for LogLevel {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            LogLevel::Trace => write!(f, "TRACE"),
            LogLevel::Debug => write!(f, "DEBUG"),
            LogLevel::Info => write!(f, "INFO"),
            LogLevel::Warning => write!(f, "WARNING"),
            LogLevel::Error => write!(f, "ERROR"),
            LogLevel::Critical => write!(f, "CRITICAL"),
        }
    }
}

// Target for log messages
pub trait LogTarget: Send + Sync {
    fn log(&self, timestamp: u64, level: LogLevel, message: &str) -> io::Result<()>;
}

// Console log target
pub struct ConsoleLogger {
    min_level: LogLevel,
    colored_output: bool,
}

impl ConsoleLogger {
    pub fn new(min_level: LogLevel) -> Self {
        ConsoleLogger {
            min_level,
            colored_output: true,
        }
    }
    
    pub fn with_color(mut self, colored: bool) -> Self {
        self.colored_output = colored;
        self
    }
    
    fn get_color(&self, level: LogLevel) -> &'static str {
        if !self.colored_output {
            return "";
        }
        
        match level {
            LogLevel::Trace => "\x1b[37m", // Light gray
            LogLevel::Debug => "\x1b[36m", // Cyan
            LogLevel::Info => "\x1b[32m",  // Green
            LogLevel::Warning => "\x1b[33m", // Yellow
            LogLevel::Error => "\x1b[31m", // Red
            LogLevel::Critical => "\x1b[35m", // Magenta
        }
    }
}

impl LogTarget for ConsoleLogger {
    fn log(&self, timestamp: u64, level: LogLevel, message: &str) -> io::Result<()> {
        if level < self.min_level {
            return Ok(());
        }
        
        let reset = if self.colored_output { "\x1b[0m" } else { "" };
        let color = self.get_color(level);
        
        let dt = SystemTime::UNIX_EPOCH + std::time::Duration::from_secs(timestamp);
        let dt_string = format!("{:?}", dt);
        
        println!(
            "{}{} [{}] {}{}",
            color, dt_string, level, message, reset
        );
        
        Ok(())
    }
}

// File log target
pub struct FileLogger {
    file: Mutex<File>,
    min_level: LogLevel,
}

impl FileLogger {
    pub fn new(path: &str, min_level: LogLevel) -> io::Result<Self> {
        // Create directory if it doesn't exist
        if let Some(parent) = PathBuf::from(path).parent() {
            fs::create_dir_all(parent)?;
        }
        
        let file = OpenOptions::new()
            .create(true)
            .append(true)
            .open(path)?;
        
        Ok(FileLogger {
            file: Mutex::new(file),
            min_level,
        })
    }
}

impl LogTarget for FileLogger {
    fn log(&self, timestamp: u64, level: LogLevel, message: &str) -> io::Result<()> {
        if level < self.min_level {
            return Ok(());
        }
        
        let dt = SystemTime::UNIX_EPOCH + std::time::Duration::from_secs(timestamp);
        let dt_string = format!("{:?}", dt);
        
        let mut file = self.file.lock().unwrap();
        writeln!(file, "{} [{}] {}", dt_string, level, message)?;
        file.flush()?;
        
        Ok(())
    }
}

// Logger that can send to multiple targets
pub struct Logger {
    targets: Vec<Arc<dyn LogTarget>>,
    context: Mutex<HashMap<String, String>>,
}

impl Logger {
    pub fn new() -> Self {
        Logger {
            targets: Vec::new(),
            context: Mutex::new(HashMap::new()),
        }
    }
    
    pub fn add_target(&mut self, target: Arc<dyn LogTarget>) {
        self.targets.push(target);
    }
    
    pub fn set_context(&self, key: &str, value: &str) {
        let mut context = self.context.lock().unwrap();
        context.insert(key.to_string(), value.to_string());
    }
    
    pub fn clear_context(&self, key: &str) {
        let mut context = self.context.lock().unwrap();
        context.remove(key);
    }
    
    fn format_with_context(&self, message: &str) -> String {
        let context = self.context.lock().unwrap();
        if context.is_empty() {
            return message.to_string();
        }
        
        let context_str = context
            .iter()
            .map(|(k, v)| format!("{}={}", k, v))
            .collect::<Vec<_>>()
            .join(", ");
        
        format!("{} [{}]", message, context_str)
    }
    
    pub fn log(&self, level: LogLevel, message: &str) -> io::Result<()> {
        let timestamp = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        let formatted = self.format_with_context(message);
        
        for target in &self.targets {
            if let Err(e) = target.log(timestamp, level, &formatted) {
                eprintln!("Error writing to log target: {}", e);
            }
        }
        
        Ok(())
    }
    
    pub fn trace(&self, message: &str) -> io::Result<()> {
        self.log(LogLevel::Trace, message)
    }
    
    pub fn debug(&self, message: &str) -> io::Result<()> {
        self.log(LogLevel::Debug, message)
    }
    
    pub fn info(&self, message: &str) -> io::Result<()> {
        self.log(LogLevel::Info, message)
    }
    
    pub fn warning(&self, message: &str) -> io::Result<()> {
        self.log(LogLevel::Warning, message)
    }
    
    pub fn error(&self, message: &str) -> io::Result<()> {
        self.log(LogLevel::Error, message)
    }
    
    pub fn critical(&self, message: &str) -> io::Result<()> {
        self.log(LogLevel::Critical, message)
    }
}

// Usage example
fn main() -> io::Result<()> {
    // Create logger with multiple targets
    let mut logger = Logger::new();
    
    // Add console logger with colored output
    let console_target = Arc::new(
        ConsoleLogger::new(LogLevel::Debug)
            .with_color(true)
    );
    logger.add_target(console_target);
    
    // Add file logger
    let file_target = Arc::new(
        FileLogger::new("logs/application.log", LogLevel::Info)?
    );
    logger.add_target(file_target);
    
    // Set context values
    logger.set_context("app", "example");
    logger.set_context("version", "1.0.0");
    
    // Log at different levels
    logger.trace("This is a trace message")?;
    logger.debug("This is a debug message")?;
    logger.info("This is an info message")?;
    logger.warning("This is a warning message")?;
    logger.error("This is an error message")?;
    
    // Change context for a specific operation
    logger.set_context("operation", "file_processing");
    logger.info("Starting file processing")?;
    
    // Simulate error
    logger.error("Failed to process file: permission denied")?;
    
    // Clear operation context
    logger.clear_context("operation");
    
    // Log a critical error
    logger.critical("Application shutdown due to critical error")?;
    
    Ok(())
}
```

### Common Pitfalls
- **Pitfall**: Reinventing functionality that already exists in the standard library.
  - **Solution**: Familiarize yourself with what's available in `std` before implementing your own version.
- **Pitfall**: Using direct items without importing, leading to verbose code.
  - **Solution**: Use appropriate `use` statements to bring commonly used items into scope.
- **Pitfall**: Not knowing which standard library module contains a needed function.
  - **Solution**: Use the Rust documentation, which is organized by module, or search tools like `docs.rs`.
- **Pitfall**: Using deprecated APIs in the standard library.
  - **Solution**: Check for deprecation warnings and alternatives in the documentation.
- **Pitfall**: Not utilizing re-exports when they would simplify imports.
  - **Solution**: Check for re-exports at higher module levels (e.g., `std::io` re-exports many types from its submodules).

### Confusion Questions with Answers
1. **Q**: How do I find which module contains a particular type or function?
   **A**: The official Rust docs at `docs.rust-lang.org` have a search function. You can also use the `cargo doc --open` command to view documentation locally. Standard types often have a module path shown in the documentation (e.g., `std::collections::HashMap`).

2. **Q**: How can I use a standard library feature that's behind a feature flag?
   **A**: Some std features require enabling specific features in your `Cargo.toml`. Add a line like:
   ```toml
   [dependencies]
   # Enable std experimental features
   std = { version = "1.0.0", features = ["feature_name"] }
   ```
   However, many experimental features are only available on nightly Rust.

3. **Q**: How does module organization in my project relate to the standard library?
   **A**: The standard library provides a good model for organizing your own code. Notice how it groups related functionality into modules, manages visibility, and uses re-exports for convenience. Consider using similar patterns in your projects, especially as they grow larger.

---

## Next Actions: Exercises and Projects

### Exercises
1. **Module Organization Practice**
   - Take an existing flat Rust project and reorganize it into a proper module hierarchy
   - Ensure public interfaces are clean and well-defined
   - Use visibility modifiers appropriately

2. **Visibility Control Exercise**
   - Create a library with a clear public API
   - Implement private implementation details that aren't exposed
   - Use different visibility modifiers for various items

3. **Standard Library Explorer**
   - Write a program that uses at least five different standard library modules
   - Document what each module provides and why you chose it
   - Explore alternative approaches using different parts of the standard library

4. **Module Pattern Comparison**
   - Implement the same module structure using both the mod.rs pattern and the file-based pattern
   - Compare and contrast the benefits and drawbacks of each approach
   - Document your findings and preferences

### Micro-Project: Configuration Library
Build a configuration management library with proper module organization:

```rust
// Structure:
// src/
// ├── lib.rs
// ├── config/
// │   ├── mod.rs
// │   ├── source.rs
// │   └── value.rs
// ├── formats/
// │   ├── mod.rs
// │   ├── json.rs
// │   ├── toml.rs
// │   └── yaml.rs
// └── utils/
//     ├── mod.rs
//     ├── conversion.rs
//     └── validation.rs

// Example implementation for config/mod.rs
pub mod source;
pub mod value;

use std::collections::HashMap;
use std::path::Path;
use source::ConfigSource;
use value::ConfigValue;
use crate::formats;

pub struct Config {
    values: HashMap<String, ConfigValue>,
}

impl Config {
    pub fn new() -> Self {
        Config {
            values: HashMap::new(),
        }
    }
    
    pub fn from_file<P: AsRef<Path>>(path: P) -> Result<Self, String> {
        // Implementation would detect file type and load accordingly
        let path = path.as_ref();
        
        if let Some(ext) = path.extension() {
            match ext.to_str() {
                Some("json") => formats::json::load_file(path),
                Some("toml") => formats::toml::load_file(path),
                Some("yaml") | Some("yml") => formats::yaml::load_file(path),
                _ => Err(format!("Unsupported file extension: {:?}", ext)),
            }
        } else {
            Err("File has no extension".to_string())
        }
    }
    
    pub fn get(&self, key: &str) -> Option<&ConfigValue> {
        self.values.get(key)
    }
    
    pub fn set(&mut self, key: &str, value: ConfigValue) {
        self.values.insert(key.to_string(), value);
    }
    
    // ... other methods ...
}

// Complete the implementation based on above structure
```

## Success Criteria
You've mastered modules in Rust when you can:
1. Organize code into a logical module hierarchy
2. Apply appropriate visibility controls to create clean APIs
3. Navigate and understand the organization of the standard library
4. Choose appropriate module organization patterns for your projects
5. Use imports and re-exports effectively
6. Manage dependencies between modules
7. Design modules that group related functionality

## Troubleshooting Advice
1. **Module Not Found Errors**
   - Ensure you've declared modules with `mod module_name;` in the parent module
   - Check file paths match module names (`mod foo;` should match `foo.rs` or `foo/mod.rs`)
   - Verify casing (modules should be snake_case)
   - Remember that `mod` statements are relative to the current file

2. **Visibility Issues**
   - Remember that making a module public doesn't make its contents public
   - Use `pub` on both the module and the items you want to expose
   - For nested items, each level in the path needs to be public
   - Use `pub(crate)`, `pub(super)`, or `pub(in path)` for fine-grained control

3. **Import Problems**
   - For items in the same crate, start paths with `crate::`, `super::`, or `self::`
   - Check that you're not trying to import private items
   - Consider using "use" statements at the top of the file
   - Group imports from the same module with braces: `use std::{fs, io, path};`

4. **File Organization Conflicts**
   - Don't mix the mod.rs pattern and the file pattern for the same module
   - If you have both `foo.rs` and `foo/mod.rs`, the compiler will be confused
   - Stick to one approach consistently within a project

5. **Circular Dependencies**
   - Move shared functionality to a common parent module
   - Use forward declarations or type references instead of direct dependencies
   - Consider if your design could be improved to eliminate circular references
   - Use trait objects to break tight coupling between modules
