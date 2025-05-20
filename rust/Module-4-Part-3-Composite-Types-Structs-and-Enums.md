# Module 4: Composite Types - Part 3: Structs and Enums

## Core Problem and Key Assumptions
- **Problem**: Understanding how to define and use custom data types in Rust through structs and enums
- **Key Assumptions**:
  - You understand basic Rust syntax and primitive types
  - You need to create custom types to model your domain
  - You want to organize related data and behavior in a type-safe manner
  - You aim to leverage Rust's pattern matching and type system

## Essential Concepts to Cover
1. Defining Structs
2. Creating Instances
3. Accessing and Modifying Fields
4. Field Init Shorthand
5. Tuple Structs
6. Enums and Pattern Matching
7. Option and Result Enums

---

## 1. Defining Structs

### Concise Explanation
Structs in Rust are custom data types that allow you to group related values together and name them. They're similar to classes in other languages but without inheritance. Rust offers three types of structs:
- **Named-field structs**: Most common, with named fields
- **Tuple structs**: Unnamed fields, identified by their position
- **Unit structs**: No fields, useful for implementing traits without data

Structs provide the foundation for organizing data in a type-safe manner and serve as the basis for implementing object-oriented patterns in Rust.

### Where to Use
- When you need to group related data together
- For modeling domain concepts and entities
- When implementing data structures
- As a way to enforce type safety for related values
- For implementing methods on your data through impl blocks

### Code Snippet
```rust
// Basic struct definition with named fields
struct Person {
    name: String,
    age: u32,
    email: Option<String>,
}

// Struct with lifetime parameters for references
struct BorrowedData<'a> {
    text: &'a str,
    count: &'a u32,
}

// Struct with generic type parameters
struct Pair<T, U> {
    first: T,
    second: U,
}

// Struct with type constraints
struct SortableCollection<T: Ord> {
    items: Vec<T>,
}

// Documentation comments for struct and fields
/// Represents a point in 2D space
struct Point {
    /// X-coordinate
    x: f64,
    /// Y-coordinate
    y: f64,
}

// Private fields with public struct
pub struct Counter {
    // This field is private and cannot be accessed directly outside this module
    count: u32,
    // This field is public and can be accessed directly
    pub label: String,
}

// Unit struct (no fields)
struct UnitStruct;

// Adding methods to a struct
impl Point {
    // Associated function (static method)
    fn new(x: f64, y: f64) -> Self {
        Point { x, y }
    }
    
    // Method that takes &self (instance method)
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
    
    // Method that takes &mut self (mutable instance method)
    fn move_by(&mut self, dx: f64, dy: f64) {
        self.x += dx;
        self.y += dy;
    }
    
    // Method that takes self (consuming instance method)
    fn into_tuple(self) -> (f64, f64) {
        (self.x, self.y)
    }
}

// Impl with type parameters
impl<T, U> Pair<T, U> {
    fn new(first: T, second: U) -> Self {
        Pair { first, second }
    }
    
    fn swap(self) -> Pair<U, T> {
        Pair {
            first: self.second,
            second: self.first,
        }
    }
}

// Multiple impl blocks are allowed
impl Point {
    fn reflect_x(&mut self) {
        self.x = -self.x;
    }
}

fn main() {
    // Create instances of the structs
    let person = Person {
        name: String::from("Alice"),
        age: 30,
        email: Some(String::from("alice@example.com")),
    };
    
    let origin = Point::new(0.0, 0.0);
    
    let pair = Pair::new("hello", 42);
    
    // Using methods
    println!("Distance from origin: {}", origin.distance_from_origin());
    
    // Unit struct instantiation
    let _unit = UnitStruct;
}
```

### Real-World Example
A bank account system with multiple account types:

```rust
use std::fmt;

// A common trait for all account types
trait Account {
    fn balance(&self) -> f64;
    fn deposit(&mut self, amount: f64) -> Result<(), String>;
    fn withdraw(&mut self, amount: f64) -> Result<(), String>;
    fn account_number(&self) -> &str;
}

// Basic account information shared by all account types
struct BasicAccountInfo {
    account_number: String,
    holder_name: String,
    balance: f64,
}

// Checking account struct
struct CheckingAccount {
    info: BasicAccountInfo,
    overdraft_limit: f64,
}

// Savings account struct
struct SavingsAccount {
    info: BasicAccountInfo,
    interest_rate: f64,
}

// Implementation for basic account info
impl BasicAccountInfo {
    fn new(account_number: &str, holder_name: &str, initial_balance: f64) -> Self {
        BasicAccountInfo {
            account_number: account_number.to_string(),
            holder_name: holder_name.to_string(),
            balance: initial_balance,
        }
    }
    
    fn deposit(&mut self, amount: f64) -> Result<(), String> {
        if amount <= 0.0 {
            return Err("Deposit amount must be positive".to_string());
        }
        self.balance += amount;
        Ok(())
    }
    
    fn withdraw(&mut self, amount: f64, limit: f64) -> Result<(), String> {
        if amount <= 0.0 {
            return Err("Withdrawal amount must be positive".to_string());
        }
        
        let new_balance = self.balance - amount;
        if new_balance < -limit {
            return Err(format!(
                "Withdrawal would exceed limit. Available: ${:.2}",
                self.balance + limit
            ));
        }
        
        self.balance = new_balance;
        Ok(())
    }
}

// Implementation for checking account
impl CheckingAccount {
    fn new(account_number: &str, holder_name: &str, initial_balance: f64, overdraft_limit: f64) -> Self {
        CheckingAccount {
            info: BasicAccountInfo::new(account_number, holder_name, initial_balance),
            overdraft_limit,
        }
    }
}

// Implementation for savings account
impl SavingsAccount {
    fn new(account_number: &str, holder_name: &str, initial_balance: f64, interest_rate: f64) -> Self {
        SavingsAccount {
            info: BasicAccountInfo::new(account_number, holder_name, initial_balance),
            interest_rate,
        }
    }
    
    // Apply interest to the balance
    fn apply_interest(&mut self) {
        let interest = self.info.balance * self.interest_rate;
        self.info.balance += interest;
    }
}

// Implement Account trait for CheckingAccount
impl Account for CheckingAccount {
    fn balance(&self) -> f64 {
        self.info.balance
    }
    
    fn deposit(&mut self, amount: f64) -> Result<(), String> {
        self.info.deposit(amount)
    }
    
    fn withdraw(&mut self, amount: f64) -> Result<(), String> {
        self.info.withdraw(amount, self.overdraft_limit)
    }
    
    fn account_number(&self) -> &str {
        &self.info.account_number
    }
}

// Implement Account trait for SavingsAccount
impl Account for SavingsAccount {
    fn balance(&self) -> f64 {
        self.info.balance
    }
    
    fn deposit(&mut self, amount: f64) -> Result<(), String> {
        self.info.deposit(amount)
    }
    
    fn withdraw(&mut self, amount: f64) -> Result<(), String> {
        // Savings accounts have no overdraft
        self.info.withdraw(amount, 0.0)
    }
    
    fn account_number(&self) -> &str {
        &self.info.account_number
    }
}

// Implement display formatting for accounts
impl fmt::Display for CheckingAccount {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(
            f,
            "Checking Account #{} - Owner: {} - Balance: ${:.2} - Overdraft Limit: ${:.2}",
            self.info.account_number, self.info.holder_name, 
            self.info.balance, self.overdraft_limit
        )
    }
}

impl fmt::Display for SavingsAccount {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(
            f,
            "Savings Account #{} - Owner: {} - Balance: ${:.2} - Interest Rate: {:.2}%",
            self.info.account_number, self.info.holder_name, 
            self.info.balance, self.interest_rate * 100.0
        )
    }
}

// Bank struct to manage multiple accounts
struct Bank {
    checking_accounts: Vec<CheckingAccount>,
    savings_accounts: Vec<SavingsAccount>,
}

impl Bank {
    fn new() -> Self {
        Bank {
            checking_accounts: Vec::new(),
            savings_accounts: Vec::new(),
        }
    }
    
    fn add_checking_account(&mut self, account: CheckingAccount) {
        self.checking_accounts.push(account);
    }
    
    fn add_savings_account(&mut self, account: SavingsAccount) {
        self.savings_accounts.push(account);
    }
    
    fn find_checking_account(&mut self, account_number: &str) -> Option<&mut CheckingAccount> {
        self.checking_accounts.iter_mut().find(|a| a.account_number() == account_number)
    }
    
    fn find_savings_account(&mut self, account_number: &str) -> Option<&mut SavingsAccount> {
        self.savings_accounts.iter_mut().find(|a| a.account_number() == account_number)
    }
    
    fn apply_interest_to_all_savings(&mut self) {
        for account in &mut self.savings_accounts {
            account.apply_interest();
        }
    }
    
    fn total_deposits(&self) -> f64 {
        let checking_total: f64 = self.checking_accounts.iter().map(|a| a.balance()).sum();
        let savings_total: f64 = self.savings_accounts.iter().map(|a| a.balance()).sum();
        checking_total + savings_total
    }
}

fn main() {
    // Create a bank
    let mut bank = Bank::new();
    
    // Add accounts
    bank.add_checking_account(CheckingAccount::new("C-12345", "John Doe", 1500.0, 500.0));
    bank.add_checking_account(CheckingAccount::new("C-67890", "Jane Smith", 2500.0, 1000.0));
    bank.add_savings_account(SavingsAccount::new("S-12345", "John Doe", 5000.0, 0.025));
    
    // Process transactions
    if let Some(account) = bank.find_checking_account("C-12345") {
        println!("Found account: {}", account);
        
        if let Err(e) = account.withdraw(2000.0) {
            println!("Error: {}", e);
        } else {
            println!("Withdrawal successful. New balance: ${:.2}", account.balance());
        }
    }
    
    if let Some(account) = bank.find_savings_account("S-12345") {
        println!("Found account: {}", account);
        
        if let Err(e) = account.deposit(1000.0) {
            println!("Error: {}", e);
        } else {
            println!("Deposit successful. New balance: ${:.2}", account.balance());
            
            // Apply interest
            account.apply_interest();
            println!("After interest: {}", account);
        }
    }
    
    // Apply interest to all savings accounts
    bank.apply_interest_to_all_savings();
    
    // Print total deposits
    println!("Total deposits in bank: ${:.2}", bank.total_deposits());
}
```

### Common Pitfalls
- **Pitfall**: Trying to use a struct without initializing all fields.
  - **Solution**: Initialize all fields or use sensible defaults with `Default` trait.
- **Pitfall**: Confusing field ownership with struct ownership.
  - **Solution**: Remember that fields containing owned types like `String` will be owned by the struct.
- **Pitfall**: Using references in structs without proper lifetime annotations.
  - **Solution**: Add lifetime parameters when using references in struct fields.
- **Pitfall**: Not making fields and methods public when needed.
  - **Solution**: Use `pub` keyword appropriately for fields and impl blocks.
- **Pitfall**: Creating deep nesting of structs that becomes unwieldy.
  - **Solution**: Consider composition patterns, traits, or splitting into smaller modules.

### Confusion Questions with Answers
1. **Q**: What's the difference between methods and associated functions in a struct?
   **A**: Methods take `self`, `&self`, or `&mut self` as their first parameter and are called on instances of the struct (e.g., `point.distance()`). Associated functions don't take a self parameter and are called on the struct type itself using the `::` operator (e.g., `Point::new()`). They're similar to static methods in other languages.

2. **Q**: When should I use private fields in a struct?
   **A**: Use private fields to enforce invariants, control access to internal state, or hide implementation details. This is part of the encapsulation principle. For example, if a field must always be positive, make it private and provide methods to modify it that enforce this constraint.

3. **Q**: Can I have a struct with no fields?
   **A**: Yes, that's called a unit struct, written as `struct UnitStruct;`. Unit structs are useful when you need a unique type to implement traits on but don't need to store any data.

---

## 2. Creating Instances

### Concise Explanation
Creating instances of structs in Rust involves specifying the struct name followed by curly braces containing the field values. There are several ways to instantiate structs:
- Field-by-field initialization
- Creating from other structs with struct update syntax
- Using constructor functions (associated functions)
- Deriving or implementing `Default` for default values
- Pattern matching to destructure struct instances

The approach you choose depends on your specific needs, such as the complexity of initialization logic or the need for validation.

### Where to Use
- When you need to create objects to represent domain entities
- For initializing data structures with specific values
- When deserializing data into typed structures
- For creating configuration objects
- As function return values to group related results

### Code Snippet
```rust
// Define some example structs
struct Point {
    x: f64,
    y: f64,
}

struct Rectangle {
    top_left: Point,
    width: f64,
    height: f64,
}

// Struct implementing the Default trait
#[derive(Default)]
struct Configuration {
    timeout_seconds: u32,
    max_retries: u32,
    debug_mode: bool,
}

// Struct with a private field
pub struct Counter {
    count: u32,
}

impl Counter {
    // Constructor associated function
    pub fn new() -> Self {
        Counter { count: 0 }
    }
    
    // Constructor with initial value
    pub fn with_initial(count: u32) -> Self {
        Counter { count }
    }
}

// Struct with lifetime and generic parameters
struct Wrapper<'a, T> {
    name: &'a str,
    value: T,
}

fn main() {
    // Basic struct initialization (field by field)
    let origin = Point { x: 0.0, y: 0.0 };
    
    // Initializing a struct with complex fields
    let rect = Rectangle {
        top_left: Point { x: 5.0, y: 10.0 },
        width: 20.0,
        height: 30.0,
    };
    
    // Using struct update syntax
    let another_point = Point { x: 5.0, ..origin };
    
    // Creating through an associated function (constructor)
    let counter = Counter::new();
    let counter_10 = Counter::with_initial(10);
    
    // Using the Default trait
    let default_config = Configuration::default();
    
    // Partial initialization with defaults
    let custom_config = Configuration {
        timeout_seconds: 60,
        ..Configuration::default()
    };
    
    // Initializing with variables that match field names
    let x = 10.0;
    let y = 20.0;
    let point = Point { x, y }; // Field init shorthand
    
    // Creating a tuple struct
    struct Color(u8, u8, u8);
    let red = Color(255, 0, 0);
    
    // Creating a struct with references (requires lifetimes)
    let name = "example";
    let wrapper = Wrapper { name, value: 42 };
    
    // Creating an instance from a function
    fn create_point(x: f64, y: f64) -> Point {
        Point { x, y }
    }
    
    let function_point = create_point(15.0, 25.0);
    
    // Destructuring a struct with a let statement
    let Point { x, y } = function_point;
    println!("Destructured point: x={}, y={}", x, y);
    
    // Partially destructuring a struct
    let Point { x, .. } = another_point;
    println!("Partially destructured point: x={}", x);
}
```

### Real-World Example
A configuration system for a web application:

```rust
use std::collections::HashMap;
use std::fs::File;
use std::io::{self, Read};
use std::path::Path;

// Application settings struct
#[derive(Debug, Clone)]
struct AppSettings {
    server: ServerConfig,
    database: DatabaseConfig,
    logging: LoggingConfig,
    features: FeatureFlags,
}

#[derive(Debug, Clone)]
struct ServerConfig {
    host: String,
    port: u16,
    workers: usize,
    timeout_seconds: u64,
}

#[derive(Debug, Clone)]
struct DatabaseConfig {
    url: String,
    max_connections: u32,
    timeout_seconds: u64,
}

#[derive(Debug, Clone)]
struct LoggingConfig {
    level: LogLevel,
    file_path: Option<String>,
    format: LogFormat,
}

#[derive(Debug, Clone, PartialEq)]
enum LogLevel {
    Debug,
    Info,
    Warning,
    Error,
}

#[derive(Debug, Clone, PartialEq)]
enum LogFormat {
    Plain,
    Json,
    Custom(String),
}

#[derive(Debug, Clone)]
struct FeatureFlags {
    enable_authentication: bool,
    enable_api_rate_limit: bool,
    experimental_features: HashMap<String, bool>,
}

// Implement Default for all configurations
impl Default for ServerConfig {
    fn default() -> Self {
        ServerConfig {
            host: "127.0.0.1".to_string(),
            port: 8080,
            workers: 4,
            timeout_seconds: 30,
        }
    }
}

impl Default for DatabaseConfig {
    fn default() -> Self {
        DatabaseConfig {
            url: "postgres://localhost/app".to_string(),
            max_connections: 10,
            timeout_seconds: 5,
        }
    }
}

impl Default for LoggingConfig {
    fn default() -> Self {
        LoggingConfig {
            level: LogLevel::Info,
            file_path: None,
            format: LogFormat::Plain,
        }
    }
}

impl Default for FeatureFlags {
    fn default() -> Self {
        FeatureFlags {
            enable_authentication: true,
            enable_api_rate_limit: false,
            experimental_features: HashMap::new(),
        }
    }
}

impl Default for AppSettings {
    fn default() -> Self {
        AppSettings {
            server: ServerConfig::default(),
            database: DatabaseConfig::default(),
            logging: LoggingConfig::default(),
            features: FeatureFlags::default(),
        }
    }
}

// Implementation for parsing configs from different sources
impl AppSettings {
    // Create new settings with defaults
    fn new() -> Self {
        AppSettings::default()
    }
    
    // Create settings from environment variables
    fn from_env() -> Self {
        use std::env;
        
        // Start with defaults
        let mut settings = AppSettings::default();
        
        // Update from environment variables if they exist
        if let Ok(host) = env::var("APP_SERVER_HOST") {
            settings.server.host = host;
        }
        
        if let Ok(port_str) = env::var("APP_SERVER_PORT") {
            if let Ok(port) = port_str.parse() {
                settings.server.port = port;
            }
        }
        
        if let Ok(url) = env::var("APP_DATABASE_URL") {
            settings.database.url = url;
        }
        
        // And so on for other settings...
        
        settings
    }
    
    // Create settings from a configuration file
    fn from_file<P: AsRef<Path>>(path: P) -> io::Result<Self> {
        let mut file = File::open(path)?;
        let mut contents = String::new();
        file.read_to_string(&mut contents)?;
        
        // In a real application, you'd parse a configuration format
        // like JSON, TOML, or YAML here. For this example, we'll just
        // return the default settings.
        Ok(AppSettings::default())
    }
    
    // Create settings for development environment
    fn development() -> Self {
        AppSettings {
            server: ServerConfig {
                host: "localhost".to_string(),
                port: 3000,
                ..ServerConfig::default()
            },
            logging: LoggingConfig {
                level: LogLevel::Debug,
                ..LoggingConfig::default()
            },
            features: FeatureFlags {
                experimental_features: {
                    let mut map = HashMap::new();
                    map.insert("new_ui".to_string(), true);
                    map
                },
                ..FeatureFlags::default()
            },
            ..AppSettings::default()
        }
    }
    
    // Create settings for production environment
    fn production() -> Self {
        AppSettings {
            server: ServerConfig {
                host: "0.0.0.0".to_string(),
                port: 80,
                workers: 8,
                timeout_seconds: 60,
            },
            database: DatabaseConfig {
                max_connections: 100,
                ..DatabaseConfig::default()
            },
            logging: LoggingConfig {
                level: LogLevel::Warning,
                file_path: Some("/var/log/app.log".to_string()),
                format: LogFormat::Json,
            },
            features: FeatureFlags {
                enable_authentication: true,
                enable_api_rate_limit: true,
                experimental_features: HashMap::new(),
            },
        }
    }
    
    // Merge with another configuration (overwrite this config with other's values)
    fn merge(&mut self, other: AppSettings) {
        self.server = other.server;
        self.database = other.database;
        self.logging = other.logging;
        self.features = other.features;
    }
    
    // Override specific settings
    fn with_server_host(mut self, host: &str) -> Self {
        self.server.host = host.to_string();
        self
    }
    
    fn with_database_url(mut self, url: &str) -> Self {
        self.database.url = url.to_string();
        self
    }
    
    fn with_log_level(mut self, level: LogLevel) -> Self {
        self.logging.level = level;
        self
    }
    
    fn enable_feature(mut self, feature_name: &str) -> Self {
        self.features.experimental_features.insert(feature_name.to_string(), true);
        self
    }
}

fn main() {
    // Basic usage with defaults
    let default_settings = AppSettings::new();
    println!("Default server port: {}", default_settings.server.port);
    
    // Environment-specific configurations
    let dev_settings = AppSettings::development();
    println!("Development log level: {:?}", dev_settings.logging.level);
    
    let prod_settings = AppSettings::production();
    println!("Production server: {}:{}", prod_settings.server.host, prod_settings.server.port);
    
    // Custom configuration with builder pattern
    let custom_settings = AppSettings::default()
        .with_server_host("api.example.com")
        .with_database_url("postgres://user:pass@db.example.com/app")
        .with_log_level(LogLevel::Error)
        .enable_feature("dark_mode");
    
    println!("Custom server host: {}", custom_settings.server.host);
    println!("Dark mode enabled: {:?}", 
             custom_settings.features.experimental_features.get("dark_mode"));
    
    // Loading from environment (would use actual env vars in a real app)
    let env_settings = AppSettings::from_env();
    
    // Loading from a file (would parse actual config in a real app)
    if let Ok(file_settings) = AppSettings::from_file("config.toml") {
        println!("Loaded settings from file");
    }
    
    // Combining configurations
    let mut final_settings = AppSettings::default();
    
    // Layer configurations with priority order
    if let Ok(file_settings) = AppSettings::from_file("config.toml") {
        final_settings.merge(file_settings);
    }
    
    final_settings.merge(AppSettings::from_env());
    
    println!("Final DB URL: {}", final_settings.database.url);
}
```

### Common Pitfalls
- **Pitfall**: Forgetting to initialize all fields in a struct literal.
  - **Solution**: Use struct update syntax (`..default`) or implement `Default`.
- **Pitfall**: Using struct update syntax (`..base`) with a base struct that has moved.
  - **Solution**: Clone the base struct if you need to use it again after the update.
- **Pitfall**: Creating structs with references without understanding lifetimes.
  - **Solution**: Learn how lifetimes work and annotate structs properly with lifetime parameters.
- **Pitfall**: Writing verbose constructor code for simple structs.
  - **Solution**: Use Rust's field init shorthand when variable names match field names.
- **Pitfall**: Creating deep copies of structs unintentionally.
  - **Solution**: Be explicit about whether you're copying (with `Clone`) or taking ownership.

### Confusion Questions with Answers
1. **Q**: How do I create a struct with some fields from another struct?
   **A**: Use the struct update syntax with the `..` operator: `Point { x: 5.0, ..other_point }`. This will copy the remaining fields from `other_point`.

2. **Q**: Why can't I initialize only some fields of a struct?
   **A**: In Rust, all fields must be initialized when creating a struct. This enforces type safety and prevents "partially initialized" objects. For optional fields, use `Option` types or implement `Default` and use struct update syntax.

3. **Q**: What's the best way to create constructor functions for complex structs?
   **A**: Use associated functions named `new` or with descriptive names like `with_capacity`. For structs with many optional fields, consider using the builder pattern or providing methods that return a modified copy like `with_field(value)`.

---

## 3. Accessing and Modifying Fields

### Concise Explanation
Accessing and modifying struct fields in Rust is straightforward but governed by ownership and mutability rules. Fields are accessed using dot notation, and modification requires mutable access to the struct. Field access respects privacy rules—public fields can be accessed from outside the module, while private fields can only be accessed within the module where the struct is defined.

Key points:
- Field access uses dot notation (`instance.field`)
- Modifying fields requires a mutable reference to the struct
- Privacy modifiers control access to fields from outside the module
- Methods often provide controlled access to private fields
- Destructuring can be used to extract multiple fields at once

### Where to Use
- When retrieving data from struct instances
- For updating state in mutable structs
- In methods that need to read or modify struct state
- When implementing data transformations
- For serializing or deserializing structs
- When implementing validation or business logic

### Code Snippet
```rust
// Define a structure with different field visibility
pub struct Person {
    pub name: String,     // Public field, accessible everywhere
    pub age: u32,         // Public field
    height: f64,          // Private field, only accessible in this module
}

impl Person {
    // Constructor
    pub fn new(name: &str, age: u32, height: f64) -> Self {
        Person {
            name: name.to_string(),
            age,
            height,
        }
    }
    
    // Getter for private field
    pub fn height(&self) -> f64 {
        self.height
    }
    
    // Setter for private field with validation
    pub fn set_height(&mut self, height: f64) -> Result<(), &'static str> {
        if height <= 0.0 {
            return Err("Height must be positive");
        }
        self.height = height;
        Ok(())
    }
    
    // Method that uses multiple fields
    pub fn is_adult(&self) -> bool {
        self.age >= 18
    }
    
    // Method modifying multiple fields
    pub fn have_birthday(&mut self) {
        self.age += 1;
        println!("{} is now {} years old!", self.name, self.age);
    }
}

// A struct with nested fields
struct Company {
    name: String,
    address: Address,
    employees: Vec<Person>,
}

struct Address {
    street: String,
    city: String,
    country: String,
    postal_code: String,
}

impl Company {
    // Accessing nested fields
    fn is_international(&self) -> bool {
        self.address.country != "USA"
    }
    
    // Modifying nested fields
    fn relocate(&mut self, street: &str, city: &str, country: &str, postal_code: &str) {
        self.address.street = street.to_string();
        self.address.city = city.to_string();
        self.address.country = country.to_string();
        self.address.postal_code = postal_code.to_string();
    }
}

fn main() {
    // Create a mutable Person instance
    let mut person = Person::new("Alice", 30, 5.6);
    
    // Accessing public fields
    println!("Name: {}", person.name);
    println!("Age: {}", person.age);
    
    // Accessing private field through getter
    println!("Height: {} feet", person.height());
    
    // Modifying public fields
    person.name = "Alicia".to_string();
    person.age += 1;
    
    // Modifying private field through setter
    if let Err(e) = person.set_height(-1.0) {
        println!("Error: {}", e);
    }
    
    if let Ok(()) = person.set_height(5.7) {
        println!("New height: {} feet", person.height());
    }
    
    // Using a method that modifies fields
    person.have_birthday();
    
    // Create a company with nested structure
    let mut company = Company {
        name: "Acme Inc.".to_string(),
        address: Address {
            street: "123 Main St".to_string(),
            city: "Springfield".to_string(),
            country: "USA".to_string(),
            postal_code: "12345".to_string(),
        },
        employees: vec![
            Person::new("Bob", 25, 6.0),
            Person::new("Charlie", 35, 5.9),
        ],
    };
    
    // Accessing nested fields
    println!("Company: {}", company.name);
    println!("Location: {}, {}", company.address.city, company.address.country);
    println!("Employees: {}", company.employees.len());
    
    // Accessing fields from elements in a collection
    for (i, employee) in company.employees.iter().enumerate() {
        println!("Employee {}: {} ({})", i + 1, employee.name, employee.age);
    }
    
    // Modifying fields in collection elements
    if let Some(first_employee) = company.employees.get_mut(0) {
        first_employee.age += 1;
    }
    
    // Using destructuring to access multiple fields
    let Person { name, age, .. } = person;
    println!("Destructured: {} is {} years old", name, age);
    
    // Updating nested structures
    company.relocate("456 Broad St", "New York", "USA", "10001");
    
    // Checking if fields meet a condition
    if company.is_international() {
        println!("{} is an international company", company.name);
    } else {
        println!("{} is a domestic company", company.name);
    }
}
```

### Real-World Example
A document editor that manipulates nested structures:

```rust
use std::fmt;
use std::time::{SystemTime, UNIX_EPOCH};

// Types for a simple document editor

#[derive(Clone)]
struct Document {
    metadata: Metadata,
    content: Content,
    permissions: Permissions,
}

#[derive(Clone)]
struct Metadata {
    title: String,
    author: String,
    created_at: u64,
    last_modified: u64,
    tags: Vec<String>,
}

#[derive(Clone)]
struct Content {
    sections: Vec<Section>,
}

#[derive(Clone)]
struct Section {
    heading: String,
    paragraphs: Vec<Paragraph>,
}

#[derive(Clone)]
struct Paragraph {
    text: String,
    style: TextStyle,
}

#[derive(Clone)]
struct TextStyle {
    bold: bool,
    italic: bool,
    underline: bool,
    font_size: u8,
}

#[derive(Clone)]
struct Permissions {
    is_public: bool,
    allow_comments: bool,
    editors: Vec<String>,
    viewers: Vec<String>,
}

impl Document {
    fn new(title: &str, author: &str) -> Self {
        let now = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        Document {
            metadata: Metadata {
                title: title.to_string(),
                author: author.to_string(),
                created_at: now,
                last_modified: now,
                tags: Vec::new(),
            },
            content: Content {
                sections: Vec::new(),
            },
            permissions: Permissions {
                is_public: false,
                allow_comments: true,
                editors: vec![author.to_string()],
                viewers: Vec::new(),
            },
        }
    }
    
    // Accessing and modifying metadata
    fn title(&self) -> &str {
        &self.metadata.title
    }
    
    fn set_title(&mut self, title: &str) {
        self.metadata.title = title.to_string();
        self.update_modified_time();
    }
    
    fn author(&self) -> &str {
        &self.metadata.author
    }
    
    fn tags(&self) -> &[String] {
        &self.metadata.tags
    }
    
    fn add_tag(&mut self, tag: &str) {
        if !self.metadata.tags.contains(&tag.to_string()) {
            self.metadata.tags.push(tag.to_string());
            self.update_modified_time();
        }
    }
    
    fn remove_tag(&mut self, tag: &str) {
        self.metadata.tags.retain(|t| t != tag);
        self.update_modified_time();
    }
    
    // Accessing and modifying content
    fn add_section(&mut self, heading: &str) {
        self.content.sections.push(Section {
            heading: heading.to_string(),
            paragraphs: Vec::new(),
        });
        self.update_modified_time();
    }
    
    fn add_paragraph(&mut self, section_index: usize, text: &str, style: TextStyle) -> Result<(), &'static str> {
        if section_index >= self.content.sections.len() {
            return Err("Section index out of bounds");
        }
        
        self.content.sections[section_index].paragraphs.push(Paragraph {
            text: text.to_string(),
            style,
        });
        
        self.update_modified_time();
        Ok(())
    }
    
    fn get_section(&self, index: usize) -> Option<&Section> {
        self.content.sections.get(index)
    }
    
    fn get_section_mut(&mut self, index: usize) -> Option<&mut Section> {
        self.content.sections.get_mut(index)
    }
    
    // Accessing and modifying permissions
    fn make_public(&mut self, public: bool) {
        self.permissions.is_public = public;
    }
    
    fn allow_comments(&mut self, allow: bool) {
        self.permissions.allow_comments = allow;
    }
    
    fn add_editor(&mut self, username: &str) {
        if !self.permissions.editors.contains(&username.to_string()) {
            self.permissions.editors.push(username.to_string());
        }
    }
    
    fn add_viewer(&mut self, username: &str) {
        if !self.permissions.viewers.contains(&username.to_string()) {
            self.permissions.viewers.push(username.to_string());
        }
    }
    
    fn can_edit(&self, username: &str) -> bool {
        self.permissions.editors.contains(&username.to_string())
    }
    
    fn can_view(&self, username: &str) -> bool {
        self.permissions.is_public ||
        self.permissions.editors.contains(&username.to_string()) ||
        self.permissions.viewers.contains(&username.to_string())
    }
    
    // Helper methods
    fn update_modified_time(&mut self) {
        self.metadata.last_modified = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
    }
    
    fn word_count(&self) -> usize {
        let mut count = 0;
        for section in &self.content.sections {
            for paragraph in &section.paragraphs {
                count += paragraph.text.split_whitespace().count();
            }
        }
        count
    }
}

impl Section {
    fn edit_heading(&mut self, new_heading: &str) {
        self.heading = new_heading.to_string();
    }
    
    fn paragraph_count(&self) -> usize {
        self.paragraphs.len()
    }
}

impl fmt::Display for Document {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        writeln!(f, "# {}", self.metadata.title)?;
        writeln!(f, "By: {} | Tags: {}", 
                 self.metadata.author,
                 self.metadata.tags.join(", "))?;
        
        for section in &self.content.sections {
            writeln!(f, "\n## {}", section.heading)?;
            
            for paragraph in &section.paragraphs {
                let style = &paragraph.style;
                let mut text = paragraph.text.clone();
                
                if style.bold {
                    text = format!("**{}**", text);
                }
                if style.italic {
                    text = format!("*{}*", text);
                }
                
                writeln!(f, "\n{}", text)?;
            }
        }
        
        Ok(())
    }
}

fn main() {
    // Create a new document
    let mut doc = Document::new("Rust Programming Guide", "Alice");
    
    // Add metadata
    doc.add_tag("programming");
    doc.add_tag("rust");
    doc.add_tag("guide");
    
    // Add content
    doc.add_section("Introduction");
    doc.add_paragraph(0, "Rust is a systems programming language focused on safety, speed, and concurrency.", 
                     TextStyle { bold: false, italic: false, underline: false, font_size: 12 }).unwrap();
    
    doc.add_paragraph(0, "This guide will help you get started with Rust programming.", 
                     TextStyle { bold: true, italic: false, underline: false, font_size: 12 }).unwrap();
    
    doc.add_section("Getting Started");
    doc.add_paragraph(1, "To install Rust, you can use rustup, the Rust installer and version management tool.", 
                     TextStyle { bold: false, italic: false, underline: false, font_size: 12 }).unwrap();
    
    // Modify existing content
    if let Some(section) = doc.get_section_mut(1) {
        section.edit_heading("Installation & Setup");
    }
    
    // Update permissions
    doc.make_public(true);
    doc.add_editor("Bob");
    doc.add_viewer("Charlie");
    
    // Access fields and generate statistics
    println!("Document: \"{}\" by {}", doc.title(), doc.author());
    println!("Tags: {}", doc.tags().join(", "));
    println!("Word count: {}", doc.word_count());
    
    // Check permissions
    println!("Public: {}", doc.permissions.is_public);
    println!("Can Bob edit? {}", doc.can_edit("Bob"));
    println!("Can Dave view? {}", doc.can_view("Dave"));
    
    // Print the document
    println!("\n{}", doc);
    
    // Update title and check that modified time changes
    let original_modified = doc.metadata.last_modified;
    doc.set_title("Comprehensive Rust Programming Guide");
    
    if doc.metadata.last_modified > original_modified {
        println!("\nDocument was successfully updated.");
    }
    
    // Create a copy of the document (clone)
    let doc_copy = doc.clone();
    println!("Original and copy have the same title: {}", 
             doc.title() == doc_copy.title());
}
```

### Common Pitfalls
- **Pitfall**: Trying to modify a field of an immutable struct.
  - **Solution**: Make the struct binding mutable with `let mut`.
- **Pitfall**: Accessing a private field from outside its module.
  - **Solution**: Provide public getter and setter methods for private fields.
- **Pitfall**: Borrowing mutable references to multiple fields of the same struct simultaneously.
  - **Solution**: Either borrow the whole struct mutably or restructure your code to avoid multiple mutable borrows.
- **Pitfall**: Expecting a struct's fields to be private by default in other languages.
  - **Solution**: Remember that in Rust, fields are private by default within a module, not private to the struct itself.
- **Pitfall**: Making all fields public without considering encapsulation.
  - **Solution**: Use private fields with public getters/setters when you need to enforce invariants.

### Confusion Questions with Answers
1. **Q**: How do I access fields in nested structs?
   **A**: Use dot notation to traverse the structure: `company.address.city`. Each step in the chain accesses the next level of nesting.

2. **Q**: Can I make some fields in a struct immutable while others are mutable?
   **A**: No, mutability in Rust applies to the entire value, not individual fields. However, you can use interior mutability patterns like `RefCell` to achieve field-level mutability, or use methods to control which fields can be modified.

3. **Q**: How do I update multiple fields of a struct at once?
   **A**: You can either update each field individually, use struct update syntax to create a new instance, or provide a method that updates multiple fields. There's no direct "update multiple fields in place" syntax.

---

## 4. Field Init Shorthand

### Concise Explanation
Field init shorthand is a Rust syntax feature that allows you to initialize struct fields with variables that have the same name as the fields. Instead of writing `field: field`, you can simply write `field`. This shorthand makes struct initialization more concise and reduces repetition, especially when constructing structs from function parameters or local variables.

### Where to Use
- When initializing struct fields from variables with matching names
- In constructor functions or factory methods
- When creating structs from function parameters
- In methods that return new struct instances
- For cleaner, more readable struct initialization code

### Code Snippet
```rust
struct Point {
    x: f64,
    y: f64,
}

struct Rectangle {
    width: f64,
    height: f64,
    position: Point,
}

struct User {
    username: String,
    email: String,
    active: bool,
    sign_in_count: u64,
}

fn main() {
    // Without field init shorthand
    let x_coord = 10.0;
    let y_coord = 20.0;
    
    let point1 = Point {
        x: x_coord,
        y: y_coord,
    };
    
    // With field init shorthand
    let x = 10.0;
    let y = 20.0;
    
    let point2 = Point { x, y };
    
    // Mixing shorthand with regular initialization
    let width = 100.0;
    let position = Point { x: 0.0, y: 0.0 };
    
    let rect = Rectangle {
        width,                // Shorthand
        height: 50.0,         // Regular initialization
        position,             // Shorthand
    };
    
    // Using with functions
    fn build_user(username: String, email: String) -> User {
        User {
            username,         // Shorthand for username: username
            email,            // Shorthand for email: email
            active: true,
            sign_in_count: 1,
        }
    }
    
    let user1 = build_user(
        String::from("john_doe"),
        String::from("john@example.com"),
    );
    
    // Combining with struct update syntax
    let user2 = User {
        email: String::from("jane@example.com"),
        ..user1              // Copy remaining fields from user1
    };
    
    // Using in methods that create new instances
    impl Point {
        fn new(x: f64, y: f64) -> Self {
            Point { x, y }    // Shorthand initialization
        }
        
        fn translate(&self, dx: f64, dy: f64) -> Self {
            let x = self.x + dx;
            let y = self.y + dy;
            Point { x, y }    // Shorthand initialization
        }
    }
    
    let origin = Point::new(0.0, 0.0);
    let moved = origin.translate(5.0, 10.0);
    
    // Using in closures that return structs
    let points: Vec<Point> = (0..5)
        .map(|i| {
            let x = i as f64;
            let y = x * 2.0;
            Point { x, y }    // Shorthand in closure
        })
        .collect();
    
    // Using with destructuring
    let Point { x, y } = points[0];
    println!("First point: ({}, {})", x, y);
}
```

### Real-World Example
An e-commerce order processing system:

```rust
use std::time::{SystemTime, UNIX_EPOCH};
use std::collections::HashMap;

// Basic types for our e-commerce system
type ProductId = String;
type UserId = String;
type OrderId = String;
type AddressId = String;

#[derive(Debug, Clone)]
struct Product {
    id: ProductId,
    name: String,
    price: f64,
    weight: f64,  // in kg
    in_stock: bool,
}

#[derive(Debug, Clone)]
struct Address {
    id: AddressId,
    street: String,
    city: String,
    state: String,
    postal_code: String,
    country: String,
}

#[derive(Debug, Clone)]
struct User {
    id: UserId,
    name: String,
    email: String,
    default_address_id: Option<AddressId>,
    addresses: Vec<Address>,
}

#[derive(Debug, Clone)]
struct OrderItem {
    product_id: ProductId,
    quantity: u32,
    price_per_unit: f64,
}

#[derive(Debug, Clone)]
struct Order {
    id: OrderId,
    user_id: UserId,
    items: Vec<OrderItem>,
    shipping_address: Address,
    order_date: u64,
    status: OrderStatus,
    total: f64,
}

#[derive(Debug, Clone, PartialEq)]
enum OrderStatus {
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled,
}

// Our e-commerce service
struct EcommerceService {
    products: HashMap<ProductId, Product>,
    users: HashMap<UserId, User>,
    orders: HashMap<OrderId, Order>,
    order_counter: u64,
}

impl EcommerceService {
    fn new() -> Self {
        EcommerceService {
            products: HashMap::new(),
            users: HashMap::new(),
            orders: HashMap::new(),
            order_counter: 1000,
        }
    }
    
    fn add_product(&mut self, name: String, price: f64, weight: f64) -> ProductId {
        let id = format!("PROD-{}", self.products.len() + 1);
        
        // Using field init shorthand
        let product = Product {
            id: id.clone(),
            name,
            price,
            weight,
            in_stock: true,
        };
        
        self.products.insert(id.clone(), product);
        id
    }
    
    fn create_user(&mut self, name: String, email: String) -> UserId {
        let id = format!("USER-{}", self.users.len() + 1);
        
        // Using field init shorthand
        let user = User {
            id: id.clone(),
            name,
            email,
            default_address_id: None,
            addresses: Vec::new(),
        };
        
        self.users.insert(id.clone(), user);
        id
    }
    
    fn add_address(&mut self, user_id: &UserId, street: String, city: String, 
                   state: String, postal_code: String, country: String) -> Result<AddressId, String> {
        let user = self.users.get_mut(user_id)
            .ok_or_else(|| format!("User {} not found", user_id))?;
        
        let id = format!("ADDR-{}-{}", user_id, user.addresses.len() + 1);
        
        // Using field init shorthand
        let address = Address {
            id: id.clone(),
            street,
            city,
            state,
            postal_code,
            country,
        };
        
        // Set as default if it's the first address
        if user.default_address_id.is_none() {
            user.default_address_id = Some(id.clone());
        }
        
        user.addresses.push(address);
        Ok(id)
    }
    
    fn create_order(&mut self, user_id: UserId, items: Vec<(ProductId, u32)>, 
                    address_id: Option<AddressId>) -> Result<OrderId, String> {
        // Verify user exists
        let user = self.users.get(&user_id)
            .ok_or_else(|| format!("User {} not found", user_id))?;
        
        // Get shipping address
        let address_id = match address_id {
            Some(addr_id) => addr_id,
            None => user.default_address_id
                .clone()
                .ok_or_else(|| "No shipping address specified or set as default")?
        };
        
        // Find the address
        let address = user.addresses
            .iter()
            .find(|addr| addr.id == address_id)
            .ok_or_else(|| format!("Address {} not found", address_id))?
            .clone();
        
        // Process order items
        let mut order_items = Vec::new();
        let mut total = 0.0;
        
        for (product_id, quantity) in items {
            let product = self.products.get(&product_id)
                .ok_or_else(|| format!("Product {} not found", product_id))?;
            
            if !product.in_stock {
                return Err(format!("Product {} is out of stock", product.name));
            }
            
            let price_per_unit = product.price;
            
            // Using field init shorthand
            let item = OrderItem {
                product_id,
                quantity,
                price_per_unit,
            };
            
            total += price_per_unit * quantity as f64;
            order_items.push(item);
        }
        
        // Generate order ID
        self.order_counter += 1;
        let id = format!("ORDER-{}", self.order_counter);
        
        // Get current timestamp
        let order_date = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        // Create the order - using field init shorthand
        let order = Order {
            id: id.clone(),
            user_id,
            items: order_items,
            shipping_address: address,
            order_date,
            status: OrderStatus::Pending,
            total,
        };
        
        self.orders.insert(id.clone(), order);
        Ok(id)
    }
    
    fn update_order_status(&mut self, order_id: &OrderId, status: OrderStatus) -> Result<(), String> {
        let order = self.orders.get_mut(order_id)
            .ok_or_else(|| format!("Order {} not found", order_id))?;
        
        order.status = status;
        Ok(())
    }
    
    fn get_order(&self, order_id: &OrderId) -> Option<&Order> {
        self.orders.get(order_id)
    }
    
    fn get_user_orders(&self, user_id: &UserId) -> Vec<&Order> {
        self.orders
            .values()
            .filter(|order| order.user_id == *user_id)
            .collect()
    }
}

fn main() {
    let mut service = EcommerceService::new();
    
    // Add products - using field init shorthand in the service methods
    let laptop_id = service.add_product(
        "Laptop Pro 13".to_string(), 
        1299.99, 
        1.3
    );
    
    let phone_id = service.add_product(
        "Smartphone X".to_string(), 
        799.99, 
        0.2
    );
    
    let headphones_id = service.add_product(
        "Wireless Headphones".to_string(), 
        199.99, 
        0.3
    );
    
    // Create a user
    let user_id = service.create_user(
        "John Doe".to_string(),
        "john@example.com".to_string()
    );
    
    // Add an address
    let address_id = service.add_address(
        &user_id,
        "123 Main St".to_string(),
        "Anytown".to_string(),
        "CA".to_string(),
        "12345".to_string(),
        "USA".to_string()
    ).unwrap();
    
    // Create an order
    let order_items = vec![
        (laptop_id.clone(), 1),
        (headphones_id.clone(), 1),
    ];
    
    let order_id = service.create_order(
        user_id.clone(), 
        order_items, 
        Some(address_id)
    ).unwrap();
    
    // Update order status
    service.update_order_status(&order_id, OrderStatus::Processing).unwrap();
    
    // Get and display the order
    if let Some(order) = service.get_order(&order_id) {
        println!("Order #{}", order.id);
        println!("Status: {:?}", order.status);
        println!("Total: ${:.2}", order.total);
        println!("Shipping to: {} {}, {}", 
                 order.shipping_address.street,
                 order.shipping_address.city,
                 order.shipping_address.country);
        
        println!("\nItems:");
        for item in &order.items {
            let product = &service.products[&item.product_id];
            println!("  - {} x {} (${:.2} each)", 
                     item.quantity, product.name, item.price_per_unit);
        }
    }
    
    // Get all orders for a user
    let user_orders = service.get_user_orders(&user_id);
    println!("\nUser has {} orders in total", user_orders.len());
}
```

### Common Pitfalls
- **Pitfall**: Using field init shorthand when variable names don't match field names.
  - **Solution**: Only use shorthand for matching names; use regular initialization for others.
- **Pitfall**: Creating unnecessary temporary variables just to use the shorthand.
  - **Solution**: Don't force the use of shorthand; it's meant to reduce redundancy, not create it.
- **Pitfall**: Forgetting that shorthand is just syntactic sugar and doesn't change ownership rules.
  - **Solution**: Remember that field init shorthand still moves or copies values according to normal Rust rules.
- **Pitfall**: Assuming field init shorthand works for expressions.
  - **Solution**: Use regular field initialization for expressions or function calls.
- **Pitfall**: Getting confused between field init shorthand and struct update syntax.
  - **Solution**: Remember that field init shorthand is for individual fields (`field`), while struct update syntax is for copying remaining fields from another instance (`..other`).

### Confusion Questions with Answers
1. **Q**: Can I use field init shorthand and struct update syntax together?
   **A**: Yes, you can use both in the same struct initialization. For example: `User { name, email, ..default_user }`.

2. **Q**: Does field init shorthand work with expressions or function calls?
   **A**: No, it only works with simple variable names. For expressions or function calls, you must use regular field initialization: `field: get_value()`.

3. **Q**: Does field init shorthand change how ownership works for the variables?
   **A**: No, the ownership rules are the same as with regular initialization. The value is moved if it's not Copy, or copied if it is.

---

## 5. Tuple Structs

### Concise Explanation
Tuple structs are a hybrid between tuples and structs, combining the unnamed fields of tuples with the named type of structs. They're useful when you want to create a distinct type but don't need to name the fields. Tuple structs are defined with the `struct` keyword followed by the name and a tuple-like list of types in parentheses. They're particularly useful for simple wrapper types and newtypes.

### Where to Use
- For creating simple wrapper types around primitive values
- When implementing the newtype pattern for type safety
- For creating distinct types with the same field structure
- When field names would be redundant or unnecessary
- As lightweight data carriers for function returns
- For grouping related values into a meaningful type

### Code Snippet
```rust
// Basic tuple struct definition
struct Point(f64, f64);

// Tuple struct with a single field (newtype pattern)
struct Meters(f64);
struct Kilometers(f64);

// Tuple struct with multiple fields
struct RGB(u8, u8, u8);

// Tuple struct with generic type
struct Pair<T>(T, T);

// Implementing methods for tuple structs
impl Point {
    fn new(x: f64, y: f64) -> Self {
        Point(x, y)
    }
    
    fn x(&self) -> f64 {
        self.0  // Access first field with .0
    }
    
    fn y(&self) -> f64 {
        self.1  // Access second field with .1
    }
    
    fn distance_from_origin(&self) -> f64 {
        (self.0.powi(2) + self.1.powi(2)).sqrt()
    }
}

// Implementing methods for the newtype pattern
impl Meters {
    fn new(value: f64) -> Self {
        if value < 0.0 {
            panic!("Meters cannot be negative");
        }
        Meters(value)
    }
    
    fn value(&self) -> f64 {
        self.0
    }
    
    fn to_kilometers(&self) -> Kilometers {
        Kilometers(self.0 / 1000.0)
    }
}

impl Kilometers {
    fn new(value: f64) -> Self {
        if value < 0.0 {
            panic!("Kilometers cannot be negative");
        }
        Kilometers(value)
    }
    
    fn value(&self) -> f64 {
        self.0
    }
    
    fn to_meters(&self) -> Meters {
        Meters(self.0 * 1000.0)
    }
}

// Implementing Display for a tuple struct
impl std::fmt::Display for RGB {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "RGB({}, {}, {})", self.0, self.1, self.2)
    }
}

fn main() {
    // Creating tuple struct instances
    let origin = Point(0.0, 0.0);
    let red = RGB(255, 0, 0);
    let pair = Pair(42, 24);
    
    // Accessing tuple struct fields
    println!("Point: ({}, {})", origin.0, origin.1);
    println!("Red: r={}, g={}, b={}", red.0, red.1, red.2);
    println!("Pair: {} and {}", pair.0, pair.1);
    
    // Using methods
    let point = Point::new(3.0, 4.0);
    println!("Distance from origin: {}", point.distance_from_origin());
    
    // Using newtype pattern for type safety
    let distance = Meters::new(1500.0);
    println!("Distance in meters: {}", distance.value());
    
    let distance_km = distance.to_kilometers();
    println!("Distance in kilometers: {}", distance_km.value());
    
    // This would fail to compile - type system prevents mixing up units:
    // let wrong_sum = distance.value() + distance_km.value();
    
    // But this works fine:
    let meters_sum = distance.value() + distance_km.to_meters().value();
    println!("Sum of distances: {} meters", meters_sum);
    
    // Destructuring tuple structs
    let Point(x, y) = point;
    println!("Destructured point: x={}, y={}", x, y);
    
    // Tuple structs with the unit type for type-checking
    struct UnitStruct;
    
    // Creating a unit struct
    let unit = UnitStruct;
    
    // Tuple structs as function return values
    fn get_color_components(color: RGB) -> (u8, u8, u8) {
        (color.0, color.1, color.2)
    }
    
    let (r, g, b) = get_color_components(red);
    println!("Color components: r={}, g={}, b={}", r, g, b);
    
    // Using tuple structs for pattern matching
    fn describe_color(color: RGB) -> &'static str {
        match color {
            RGB(255, 0, 0) => "pure red",
            RGB(0, 255, 0) => "pure green",
            RGB(0, 0, 255) => "pure blue",
            RGB(r, g, b) if r == g && g == b => "grayscale",
            RGB(_, _, _) => "mixed color",
        }
    }
    
    println!("Red is a {}", describe_color(red));
    println!("Gray is a {}", describe_color(RGB(128, 128, 128)));
    println!("Yellow is a {}", describe_color(RGB(255, 255, 0)));
}
```

### Real-World Example
A physics simulation using tuple structs for vectors and forces:

```rust
use std::ops::{Add, Sub, Mul};

// Tuple structs for physical quantities
struct Vector2D(f64, f64);
struct Force(Vector2D);
struct Velocity(Vector2D);
struct Position(Vector2D);
struct Mass(f64);

// Implementation for 2D vector operations
impl Vector2D {
    fn new(x: f64, y: f64) -> Self {
        Vector2D(x, y)
    }
    
    fn zero() -> Self {
        Vector2D(0.0, 0.0)
    }
    
    fn magnitude(&self) -> f64 {
        (self.0 * self.0 + self.1 * self.1).sqrt()
    }
    
    fn normalize(&self) -> Self {
        let mag = self.magnitude();
        if mag > 0.0 {
            Vector2D(self.0 / mag, self.1 / mag)
        } else {
            Vector2D::zero()
        }
    }
}

// Implement addition for Vector2D
impl Add for Vector2D {
    type Output = Vector2D;
    
    fn add(self, other: Vector2D) -> Vector2D {
        Vector2D(self.0 + other.0, self.1 + other.1)
    }
}

// Implement subtraction for Vector2D
impl Sub for Vector2D {
    type Output = Vector2D;
    
    fn sub(self, other: Vector2D) -> Vector2D {
        Vector2D(self.0 - other.0, self.1 - other.1)
    }
}

// Implement scalar multiplication for Vector2D
impl Mul<f64> for Vector2D {
    type Output = Vector2D;
    
    fn mul(self, scalar: f64) -> Vector2D {
        Vector2D(self.0 * scalar, self.1 * scalar)
    }
}

// Implementations for physical quantities
impl Force {
    fn zero() -> Self {
        Force(Vector2D::zero())
    }
    
    fn from_components(x: f64, y: f64) -> Self {
        Force(Vector2D(x, y))
    }
    
    fn magnitude(&self) -> f64 {
        self.0.magnitude()
    }
}

impl Add for Force {
    type Output = Force;
    
    fn add(self, other: Force) -> Force {
        Force(self.0 + other.0)
    }
}

impl Velocity {
    fn zero() -> Self {
        Velocity(Vector2D::zero())
    }
    
    fn from_components(x: f64, y: f64) -> Self {
        Velocity(Vector2D(x, y))
    }
    
    fn magnitude(&self) -> f64 {
        self.0.magnitude()
    }
}

impl Position {
    fn origin() -> Self {
        Position(Vector2D::zero())
    }
    
    fn from_components(x: f64, y: f64) -> Self {
        Position(Vector2D(x, y))
    }
    
    fn distance_to(&self, other: &Position) -> f64 {
        (self.0 - other.0).magnitude()
    }
}

impl Mass {
    fn new(mass: f64) -> Self {
        if mass <= 0.0 {
            panic!("Mass must be positive");
        }
        Mass(mass)
    }
}

// A physics object in our simulation
struct PhysicsObject {
    name: String,
    position: Position,
    velocity: Velocity,
    mass: Mass,
}

impl PhysicsObject {
    fn new(name: &str, position: Position, velocity: Velocity, mass: Mass) -> Self {
        PhysicsObject {
            name: name.to_string(),
            position,
            velocity,
            mass,
        }
    }
    
    // Update position based on velocity
    fn update_position(&mut self, dt: f64) {
        // p = p + v * dt
        let displacement = self.velocity.0 * dt;
        self.position = Position(self.position.0 + displacement);
    }
    
    // Update velocity based on applied force
    fn apply_force(&mut self, force: Force, dt: f64) {
        // v = v + (F / m) * dt
        let acceleration = force.0 * (1.0 / self.mass.0);
        self.velocity = Velocity(self.velocity.0 + acceleration * dt);
    }
    
    // Calculate gravitational force between two objects
    fn gravitational_force(&self, other: &PhysicsObject) -> Force {
        const G: f64 = 6.67430e-11; // gravitational constant
        
        let delta = other.position.0 - self.position.0;
        let distance = delta.magnitude();
        
        if distance < 1e-10 {
            return Force::zero(); // Avoid division by zero
        }
        
        let force_magnitude = G * self.mass.0 * other.mass.0 / (distance * distance);
        let direction = delta.normalize();
        
        Force(direction * force_magnitude)
    }
}

// The simulation environment
struct PhysicsSimulation {
    objects: Vec<PhysicsObject>,
    time: f64,
    dt: f64,
}

impl PhysicsSimulation {
    fn new(dt: f64) -> Self {
        PhysicsSimulation {
            objects: Vec::new(),
            time: 0.0,
            dt,
        }
    }
    
    fn add_object(&mut self, object: PhysicsObject) {
        self.objects.push(object);
    }
    
    fn step(&mut self) {
        // Calculate forces
        let mut forces = vec![Force::zero(); self.objects.len()];
        
        // Calculate gravitational forces between all pairs
        for i in 0..self.objects.len() {
            for j in i+1..self.objects.len() {
                let force = self.objects[i].gravitational_force(&self.objects[j]);
                
                // Apply Newton's third law: equal and opposite forces
                forces[i] = forces[i] + force.clone();
                forces[j] = forces[j] + Force(force.0 * -1.0);
            }
        }
        
        // Apply forces and update positions
        for (i, object) in self.objects.iter_mut().enumerate() {
            object.apply_force(forces[i].clone(), self.dt);
            object.update_position(self.dt);
        }
        
        self.time += self.dt;
    }
    
    fn print_state(&self) {
        println!("Simulation time: {:.2} s", self.time);
        for object in &self.objects {
            println!("  {}: position=({:.2}, {:.2}), velocity=({:.2}, {:.2}), speed={:.2}",
                     object.name,
                     object.position.0.0, object.position.0.1,
                     object.velocity.0.0, object.velocity.0.1,
                     object.velocity.magnitude());
        }
    }
}

fn main() {
    // Create a simple 2-body simulation
    let mut sim = PhysicsSimulation::new(0.1); // 0.1 second time step
    
    // Add objects (scaled for easier visualization)
    sim.add_object(PhysicsObject::new(
        "Sun",
        Position::from_components(0.0, 0.0),
        Velocity::zero(),
        Mass::new(1.0e20) // Not actual solar mass, scaled for simulation
    ));
    
    sim.add_object(PhysicsObject::new(
        "Earth",
        Position::from_components(1000.0, 0.0),
        Velocity::from_components(0.0, 100.0),
        Mass::new(1.0e16) // Not actual Earth mass, scaled for simulation
    ));
    
    // Run simulation for a few steps
    println!("Initial state:");
    sim.print_state();
    
    for i in 1..=10 {
        sim.step();
        if i % 5 == 0 {
            println!("\nAfter {} steps:", i);
            sim.print_state();
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Confusing tuple structs with regular tuples.
  - **Solution**: Remember that tuple structs have a name and create a distinct type, while regular tuples are anonymous types.
- **Pitfall**: Forgetting that field access uses numeric indices (`.0`, `.1`) instead of named fields.
  - **Solution**: If field access clarity is important, consider using a regular struct with named fields instead.
- **Pitfall**: Not using tuple structs when they would be appropriate for the use case.
  - **Solution**: Consider tuple structs for simple wrapper types, the newtype pattern, or when field names are unnecessary.
- **Pitfall**: Overusing tuple structs for complex data that would benefit from named fields.
  - **Solution**: If you find yourself struggling to remember what each field represents, use a regular struct with named fields.
- **Pitfall**: Not implementing appropriate traits for tuple structs.
  - **Solution**: Consider implementing traits like `Debug`, `Display`, `PartialEq`, etc., to make your tuple structs more usable.

### Confusion Questions with Answers
1. **Q**: What's the difference between a tuple struct and a regular tuple?
   **A**: A tuple struct has a name and creates a distinct type, while a regular tuple is an anonymous type. This means tuple structs with identical field types are still considered different types by the compiler, which provides additional type safety.

2. **Q**: When should I use a tuple struct instead of a regular struct?
   **A**: Use tuple structs when you need a distinct type but field names would be redundant or unnecessary. They're particularly useful for implementing the newtype pattern, creating simple wrapper types, or when the meaning of the fields is obvious from context.

3. **Q**: How do I access fields in a tuple struct?
   **A**: Use numeric indices with dot notation: `.0` for the first field, `.1` for the second, and so on. You can also destructure tuple structs in patterns, like `let Point(x, y) = point;`.

---

## 6. Enums and Pattern Matching

### Concise Explanation
Enums (short for "enumerations") in Rust are custom data types that can have a fixed set of variants. Unlike enums in many other languages, Rust's enums are more powerful because each variant can optionally hold data of different types. This makes them ideal for representing data that can be one of several possible variants. 

Pattern matching is a powerful feature that works hand-in-hand with enums, allowing you to match against enum variants and extract their data in a concise and type-safe way. Rust's match expression ensures that all possible cases are handled, making your code more robust.

### Where to Use
- For representing data that can be in one of several distinct states
- When modeling concepts with a fixed set of possible values
- For error handling with the `Result` enum
- For representing optional values with the `Option` enum
- In state machines where entities transition between well-defined states
- When implementing parsers, interpreters, or compilers
- For representing complex data structures with variant parts

### Code Snippet
```rust
// Basic enum definition
enum Direction {
    North,
    East,
    South,
    West,
}

// Enum with data associated with each variant
enum Shape {
    Circle(f64),                     // Radius
    Rectangle(f64, f64),             // Width, Height
    Triangle(f64, f64, f64),         // Three sides
}

// Enum with struct-like variants
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(u8, u8, u8),
}

// Enum with generic types
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// Implementing methods on enums
impl Direction {
    fn to_angle(&self) -> f64 {
        match self {
            Direction::North => 0.0,
            Direction::East => 90.0,
            Direction::South => 180.0,
            Direction::West => 270.0,
        }
    }
    
    fn turn_clockwise(&self) -> Direction {
        match self {
            Direction::North => Direction::East,
            Direction::East => Direction::South,
            Direction::South => Direction::West,
            Direction::West => Direction::North,
        }
    }
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(radius) => std::f64::consts::PI * radius * radius,
            Shape::Rectangle(width, height) => width * height,
            Shape::Triangle(a, b, c) => {
                // Heron's formula
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        }
    }
}

fn main() {
    // Creating enum instances
    let direction = Direction::North;
    let shape = Shape::Rectangle(10.0, 20.0);
    let message = Message::Write(String::from("Hello"));
    
    // Using methods on enums
    println!("Direction angle: {} degrees", direction.to_angle());
    println!("Shape area: {}", shape.area());
    
    // Basic pattern matching with match expression
    match direction {
        Direction::North => println!("Going north!"),
        Direction::East => println!("Going east!"),
        Direction::South => println!("Going south!"),
        Direction::West => println!("Going west!"),
    }
    
    // Pattern matching with data extraction
    match shape {
        Shape::Circle(radius) => {
            println!("Circle with radius: {}", radius);
        },
        Shape::Rectangle(width, height) => {
            println!("Rectangle with width: {} and height: {}", width, height);
        },
        Shape::Triangle(a, b, c) => {
            println!("Triangle with sides: {}, {}, {}", a, b, c);
        },
    }
    
    // Matching with if-let for a single case
    if let Message::Write(text) = message {
        println!("Message content: {}", text);
    }
    
    // Matching with struct-like variants
    let move_message = Message::Move { x: 10, y: 20 };
    
    if let Message::Move { x, y } = move_message {
        println!("Moving to position: ({}, {})", x, y);
    }
    
    // Matching with wildcard and multiple patterns
    let direction = Direction::South;
    
    match direction {
        Direction::North | Direction::South => println!("Going along meridian!"),
        _ => println!("Going along parallel!"),
    }
    
    // Match with guards
    let shape = Shape::Circle(5.0);
    
    match shape {
        Shape::Circle(radius) if radius > 10.0 => println!("A large circle"),
        Shape::Circle(radius) => println!("A circle with radius {}", radius),
        Shape::Rectangle(w, h) if w == h => println!("A square"),
        Shape::Rectangle(_, _) => println!("A rectangle"),
        Shape::Triangle(_, _, _) => println!("A triangle"),
    }
    
    // Binding with @ pattern
    match shape {
        Shape::Circle(r @ 1.0...5.0) => println!("Circle with radius {} (small)", r),
        Shape::Circle(r @ 5.0...10.0) => println!("Circle with radius {} (medium)", r),
        Shape::Circle(r) => println!("Circle with radius {} (other size)", r),
        _ => println!("Not a circle"),
    }
    
    // Return values from match
    let description = match shape {
        Shape::Circle(_) => "a round shape",
        Shape::Rectangle(_, _) => "a four-sided shape",
        Shape::Triangle(_, _, _) => "a three-sided shape",
    };
    
    println!("The shape is {}", description);
    
    // Enums for error handling
    fn divide(a: f64, b: f64) -> Result<f64, String> {
        if b == 0.0 {
            Result::Err("Division by zero".to_string())
        } else {
            Result::Ok(a / b)
        }
    }
    
    // Using the Result enum
    match divide(10.0, 2.0) {
        Result::Ok(result) => println!("Division result: {}", result),
        Result::Err(error) => println!("Error: {}", error),
    }
    
    match divide(10.0, 0.0) {
        Result::Ok(result) => println!("Division result: {}", result),
        Result::Err(error) => println!("Error: {}", error),
    }
}
```

### Real-World Example
A command-line todo application using enums for commands and task states:

```rust
use std::fmt;
use std::time::{SystemTime, UNIX_EPOCH};

// Enum for task state
#[derive(Debug, Clone, PartialEq)]
enum TaskState {
    Pending,
    InProgress { started_at: u64 },
    Completed { completed_at: u64 },
    Cancelled { reason: String },
}

// Struct for a task
#[derive(Debug, Clone)]
struct Task {
    id: usize,
    title: String,
    description: Option<String>,
    created_at: u64,
    state: TaskState,
    tags: Vec<String>,
}

// Enum for commands that can be performed
enum Command {
    Add { title: String, description: Option<String>, tags: Vec<String> },
    List { filter: Option<TaskFilter> },
    Start { id: usize },
    Complete { id: usize },
    Cancel { id: usize, reason: String },
    Remove { id: usize },
    Tag { id: usize, tag: String },
    Help,
    Quit,
}

// Enum for filtering tasks in list command
enum TaskFilter {
    All,
    Pending,
    InProgress,
    Completed,
    Cancelled,
    ByTag(String),
}

// Implementation for TaskState
impl TaskState {
    fn is_active(&self) -> bool {
        matches!(self, TaskState::Pending | TaskState::InProgress { .. })
    }
    
    fn is_terminal(&self) -> bool {
        matches!(self, TaskState::Completed { .. } | TaskState::Cancelled { .. })
    }
    
    fn description(&self) -> &str {
        match self {
            TaskState::Pending => "Pending",
            TaskState::InProgress { .. } => "In Progress",
            TaskState::Completed { .. } => "Completed",
            TaskState::Cancelled { .. } => "Cancelled",
        }
    }
}

// Implementation for Task
impl Task {
    fn new(id: usize, title: &str, description: Option<String>, tags: Vec<String>) -> Self {
        let now = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        Task {
            id,
            title: title.to_string(),
            description,
            created_at: now,
            state: TaskState::Pending,
            tags,
        }
    }
    
    fn start(&mut self) -> Result<(), String> {
        match self.state {
            TaskState::Pending => {
                let now = SystemTime::now()
                    .duration_since(UNIX_EPOCH)
                    .unwrap()
                    .as_secs();
                
                self.state = TaskState::InProgress { started_at: now };
                Ok(())
            },
            TaskState::InProgress { .. } => {
                Err("Task is already in progress".to_string())
            },
            TaskState::Completed { .. } => {
                Err("Cannot start a completed task".to_string())
            },
            TaskState::Cancelled { .. } => {
                Err("Cannot start a cancelled task".to_string())
            },
        }
    }
    
    fn complete(&mut self) -> Result<(), String> {
        match self.state {
            TaskState::Pending | TaskState::InProgress { .. } => {
                let now = SystemTime::now()
                    .duration_since(UNIX_EPOCH)
                    .unwrap()
                    .as_secs();
                
                self.state = TaskState::Completed { completed_at: now };
                Ok(())
            },
            TaskState::Completed { .. } => {
                Err("Task is already completed".to_string())
            },
            TaskState::Cancelled { .. } => {
                Err("Cannot complete a cancelled task".to_string())
            },
        }
    }
    
    fn cancel(&mut self, reason: &str) -> Result<(), String> {
        match self.state {
            TaskState::Pending | TaskState::InProgress { .. } => {
                self.state = TaskState::Cancelled { reason: reason.to_string() };
                Ok(())
            },
            TaskState::Completed { .. } => {
                Err("Cannot cancel a completed task".to_string())
            },
            TaskState::Cancelled { .. } => {
                Err("Task is already cancelled".to_string())
            },
        }
    }
    
    fn add_tag(&mut self, tag: &str) {
        if !self.tags.contains(&tag.to_string()) {
            self.tags.push(tag.to_string());
        }
    }
    
    fn has_tag(&self, tag: &str) -> bool {
        self.tags.contains(&tag.to_string())
    }
    
    fn format_time(timestamp: u64) -> String {
        // Very simple time formatting for example purposes
        format!("{}", timestamp)
    }
}

impl fmt::Display for Task {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}] {} - {}", self.id, self.title, self.state.description())?;
        
        if let Some(desc) = &self.description {
            write!(f, "\n    Description: {}", desc)?;
        }
        
        match &self.state {
            TaskState::Pending => {},
            TaskState::InProgress { started_at } => {
                write!(f, "\n    Started at: {}", Task::format_time(*started_at))?;
            },
            TaskState::Completed { completed_at } => {
                write!(f, "\n    Completed at: {}", Task::format_time(*completed_at))?;
            },
            TaskState::Cancelled { reason } => {
                write!(f, "\n    Cancelled: {}", reason)?;
            },
        }
        
        if !self.tags.is_empty() {
            write!(f, "\n    Tags: {}", self.tags.join(", "))?;
        }
        
        Ok(())
    }
}

// TodoList manages the tasks
struct TodoList {
    tasks: Vec<Task>,
    next_id: usize,
}

impl TodoList {
    fn new() -> Self {
        TodoList {
            tasks: Vec::new(),
            next_id: 1,
        }
    }
    
    fn add_task(&mut self, title: &str, description: Option<String>, tags: Vec<String>) -> usize {
        let id = self.next_id;
        self.next_id += 1;
        
        let task = Task::new(id, title, description, tags);
        self.tasks.push(task);
        
        id
    }
    
    fn get_task(&self, id: usize) -> Option<&Task> {
        self.tasks.iter().find(|task| task.id == id)
    }
    
    fn get_task_mut(&mut self, id: usize) -> Option<&mut Task> {
        self.tasks.iter_mut().find(|task| task.id == id)
    }
    
    fn remove_task(&mut self, id: usize) -> Result<Task, String> {
        let position = self.tasks.iter().position(|task| task.id == id)
            .ok_or_else(|| format!("Task with ID {} not found", id))?;
        
        Ok(self.tasks.remove(position))
    }
    
    fn list_tasks(&self, filter: &Option<TaskFilter>) -> Vec<&Task> {
        match filter {
            None => self.tasks.iter().collect(),
            Some(TaskFilter::All) => self.tasks.iter().collect(),
            Some(TaskFilter::Pending) => self.tasks.iter()
                .filter(|task| matches!(task.state, TaskState::Pending))
                .collect(),
            Some(TaskFilter::InProgress) => self.tasks.iter()
                .filter(|task| matches!(task.state, TaskState::InProgress { .. }))
                .collect(),
            Some(TaskFilter::Completed) => self.tasks.iter()
                .filter(|task| matches!(task.state, TaskState::Completed { .. }))
                .collect(),
            Some(TaskFilter::Cancelled) => self.tasks.iter()
                .filter(|task| matches!(task.state, TaskState::Cancelled { .. }))
                .collect(),
            Some(TaskFilter::ByTag(tag)) => self.tasks.iter()
                .filter(|task| task.has_tag(tag))
                .collect(),
        }
    }
}

// Function to execute a command
fn execute_command(todo_list: &mut TodoList, command: Command) -> bool {
    match command {
        Command::Add { title, description, tags } => {
            let id = todo_list.add_task(&title, description, tags);
            println!("Added task with ID: {}", id);
            true
        },
        
        Command::List { filter } => {
            let tasks = todo_list.list_tasks(&filter);
            
            println!("Tasks ({})", tasks.len());
            if tasks.is_empty() {
                println!("  No tasks found.");
            } else {
                for task in tasks {
                    println!("\n{}", task);
                }
            }
            true
        },
        
        Command::Start { id } => {
            match todo_list.get_task_mut(id) {
                Some(task) => {
                    match task.start() {
                        Ok(_) => println!("Task started: {}", id),
                        Err(e) => println!("Error: {}", e),
                    }
                },
                None => println!("Task not found: {}", id),
            }
            true
        },
        
        Command::Complete { id } => {
            match todo_list.get_task_mut(id) {
                Some(task) => {
                    match task.complete() {
                        Ok(_) => println!("Task completed: {}", id),
                        Err(e) => println!("Error: {}", e),
                    }
                },
                None => println!("Task not found: {}", id),
            }
            true
        },
        
        Command::Cancel { id, reason } => {
            match todo_list.get_task_mut(id) {
                Some(task) => {
                    match task.cancel(&reason) {
                        Ok(_) => println!("Task cancelled: {}", id),
                        Err(e) => println!("Error: {}", e),
                    }
                },
                None => println!("Task not found: {}", id),
            }
            true
        },
        
        Command::Remove { id } => {
            match todo_list.remove_task(id) {
                Ok(_) => println!("Task removed: {}", id),
                Err(e) => println!("Error: {}", e),
            }
            true
        },
        
        Command::Tag { id, tag } => {
            match todo_list.get_task_mut(id) {
                Some(task) => {
                    task.add_tag(&tag);
                    println!("Added tag '{}' to task {}", tag, id);
                },
                None => println!("Task not found: {}", id),
            }
            true
        },
        
        Command::Help => {
            println!("Available commands:");
            println!("  add <title> [description] [tags...] - Add a new task");
            println!("  list [all|pending|in-progress|completed|cancelled|tag:<tag>] - List tasks");
            println!("  start <id> - Start a task");
            println!("  complete <id> - Complete a task");
            println!("  cancel <id> <reason> - Cancel a task");
            println!("  remove <id> - Remove a task");
            println!("  tag <id> <tag> - Add a tag to a task");
            println!("  help - Show this help message");
            println!("  quit - Exit the application");
            true
        },
        
        Command::Quit => {
            println!("Goodbye!");
            false
        },
    }
}

// Simple parser for commands (in a real application, use a proper CLI parser)
fn parse_command(input: &str) -> Result<Command, String> {
    let parts: Vec<&str> = input.trim().split_whitespace().collect();
    
    if parts.is_empty() {
        return Err("Empty command".to_string());
    }
    
    match parts[0] {
        "add" => {
            if parts.len() < 2 {
                return Err("Usage: add <title> [description] [tags...]".to_string());
            }
            
            let title = parts[1].to_string();
            let description = if parts.len() > 2 { Some(parts[2].to_string()) } else { None };
            
            let tags = if parts.len() > 3 {
                parts[3..].iter().map(|s| s.to_string()).collect()
            } else {
                Vec::new()
            };
            
            Ok(Command::Add { title, description, tags })
        },
        
        "list" => {
            let filter = if parts.len() > 1 {
                match parts[1] {
                    "all" => Some(TaskFilter::All),
                    "pending" => Some(TaskFilter::Pending),
                    "in-progress" => Some(TaskFilter::InProgress),
                    "completed" => Some(TaskFilter::Completed),
                    "cancelled" => Some(TaskFilter::Cancelled),
                    tag if tag.starts_with("tag:") => {
                        let tag_name = tag.strip_prefix("tag:").unwrap();
                        Some(TaskFilter::ByTag(tag_name.to_string()))
                    },
                    _ => return Err(format!("Unknown filter: {}", parts[1])),
                }
            } else {
                None
            };
            
            Ok(Command::List { filter })
        },
        
        "start" => {
            if parts.len() < 2 {
                return Err("Usage: start <id>".to_string());
            }
            
            match parts[1].parse::<usize>() {
                Ok(id) => Ok(Command::Start { id }),
                Err(_) => Err("Invalid task ID".to_string()),
            }
        },
        
        "complete" => {
            if parts.len() < 2 {
                return Err("Usage: complete <id>".to_string());
            }
            
            match parts[1].parse::<usize>() {
                Ok(id) => Ok(Command::Complete { id }),
                Err(_) => Err("Invalid task ID".to_string()),
            }
        },
        
        "cancel" => {
            if parts.len() < 3 {
                return Err("Usage: cancel <id> <reason>".to_string());
            }
            
            match parts[1].parse::<usize>() {
                Ok(id) => {
                    let reason = parts[2..].join(" ");
                    Ok(Command::Cancel { id, reason })
                },
                Err(_) => Err("Invalid task ID".to_string()),
            }
        },
        
        "remove" => {
            if parts.len() < 2 {
                return Err("Usage: remove <id>".to_string());
            }
            
            match parts[1].parse::<usize>() {
                Ok(id) => Ok(Command::Remove { id }),
                Err(_) => Err("Invalid task ID".to_string()),
            }
        },
        
        "tag" => {
            if parts.len() < 3 {
                return Err("Usage: tag <id> <tag>".to_string());
            }
            
            match parts[1].parse::<usize>() {
                Ok(id) => {
                    let tag = parts[2].to_string();
                    Ok(Command::Tag { id, tag })
                },
                Err(_) => Err("Invalid task ID".to_string()),
            }
        },
        
        "help" => Ok(Command::Help),
        
        "quit" | "exit" => Ok(Command::Quit),
        
        _ => Err(format!("Unknown command: {}", parts[0])),
    }
}

fn main() {
    let mut todo_list = TodoList::new();
    
    // Add some sample tasks
    todo_list.add_task(
        "Implement todo app", 
        Some("Create a simple CLI todo application".to_string()), 
        vec!["rust".to_string(), "project".to_string()]
    );
    
    todo_list.add_task(
        "Learn Rust enums", 
        None, 
        vec!["rust".to_string(), "learning".to_string()]
    );
    
    // In a real application, you would read user input here
    // For this example, we'll just simulate some commands
    let commands = [
        "list",
        "start 1",
        "tag 2 important",
        "list pending",
        "complete 1",
        "list in-progress",
        "list completed",
        "list tag:important",
        "cancel 2 No longer needed",
        "list",
        "quit",
    ];
    
    for command_str in commands.iter() {
        println!("\n> {}", command_str);
        
        match parse_command(command_str) {
            Ok(command) => {
                if !execute_command(&mut todo_list, command) {
                    break;
                }
            },
            Err(error) => println!("Error: {}", error),
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Forgetting to handle all possible variants in a match expression.
  - **Solution**: The compiler will warn you about non-exhaustive patterns; use wildcards (`_`) or the `..` pattern to handle remaining cases.
- **Pitfall**: Using if-let when you need to handle multiple cases.
  - **Solution**: Use a match expression when you need to handle multiple cases, and if-let for simpler, single-case scenarios.
- **Pitfall**: Not taking advantage of pattern matching for data extraction.
  - **Solution**: Use pattern matching to simultaneously check the variant and extract its data.
- **Pitfall**: Overcomplicating enums with too many variants or data types.
  - **Solution**: Consider splitting complex enums into multiple types or using composition.
- **Pitfall**: Not using match guards or the @ pattern for more complex conditions.
  - **Solution**: Learn advanced pattern matching features to write more concise, expressive code.

### Confusion Questions with Answers
1. **Q**: How do enums in Rust differ from enums in other languages?
   **A**: Rust's enums are more powerful than enums in many other languages because each variant can hold different types of data. This makes them similar to algebraic data types or tagged unions in functional programming languages. In contrast, enums in languages like C, Java, or C# are often limited to being simple sets of named constants.

2. **Q**: When should I use a match expression versus if-let?
   **A**: Use a match expression when you need to handle multiple variants or want exhaustiveness checking. Use if-let when you're only interested in one specific pattern and want more concise code for that single case. If-let is essentially syntactic sugar for a match with a single pattern and a catch-all `_ => {}` arm.

3. **Q**: How can I ensure I don't forget to handle a new enum variant if I add one later?
   **A**: The Rust compiler performs exhaustiveness checking on match expressions, so if you add a new variant to an enum, any match expressions that don't handle the new variant will generate a compilation error. This is a powerful feature that helps prevent bugs when refactoring.

---

## 7. Option and Result Enums

### Concise Explanation
`Option` and `Result` are two fundamental enum types in Rust's standard library that help manage uncertainty and error handling in a type-safe way.

The `Option<T>` enum represents a value that might be present (`Some(T)`) or absent (`None`). It's Rust's way of avoiding null references, forcing developers to explicitly handle the case where a value might not exist.

The `Result<T, E>` enum represents either success (`Ok(T)`) or failure (`Err(E)`). It's a core part of Rust's error handling strategy, enabling functions to return either a successful value or an error, with the compiler ensuring both cases are handled.

These enums, combined with pattern matching, make code more robust by eliminating whole classes of bugs related to null references and unhandled errors.

### Where to Use
- **Option**
  - For functions that might not return a valid value
  - When looking up values in collections
  - For optional struct fields
  - When parsing might fail
  - For functions where "no result" is a valid outcome

- **Result**
  - For functions that can fail
  - When performing I/O operations
  - For validations and conversions
  - When interacting with external systems
  - For propagating errors up the call stack

### Code Snippet
```rust
// Basic usage of Option
fn main() {
    // Creating Options
    let some_value = Some(42);
    let no_value: Option<i32> = None;
    
    // Pattern matching on Option
    match some_value {
        Some(value) => println!("Got a value: {}", value),
        None => println!("No value"),
    }
    
    // Using if-let for concise handling of Some
    if let Some(value) = some_value {
        println!("Got a value: {}", value);
    }
    
    // Using Option methods
    
    // is_some() and is_none() for checking variant
    println!("some_value.is_some(): {}", some_value.is_some());
    println!("no_value.is_none(): {}", no_value.is_none());
    
    // unwrap() extracts the value (panics if None)
    println!("Unwrapped value: {}", some_value.unwrap());
    
    // expect() is like unwrap() but with a custom error message
    println!("Expected value: {}", some_value.expect("Should have a value"));
    
    // unwrap_or() provides a default value if None
    println!("Unwrapped or default: {}", no_value.unwrap_or(0));
    
    // unwrap_or_else() computes a default if None
    println!("Unwrapped or computed: {}", no_value.unwrap_or_else(|| 42 * 2));
    
    // map() transforms the contained value
    let doubled = some_value.map(|x| x * 2);
    println!("Doubled: {:?}", doubled);
    
    // and_then() chains operations that return Option
    let option_string = Some("42");
    let parsed = option_string.and_then(|s| s.parse::<i32>().ok());
    println!("Parsed: {:?}", parsed);
    
    // filter() keeps Some values that satisfy a predicate
    let even = some_value.filter(|&x| x % 2 == 0);
    let odd = some_value.filter(|&x| x % 2 != 0);
    println!("Even: {:?}, Odd: {:?}", even, odd);
    
    // Using Result for error handling
    
    // Creating Results
    let success: Result<i32, &str> = Ok(42);
    let failure: Result<i32, &str> = Err("something went wrong");
    
    // Pattern matching on Result
    match success {
        Ok(value) => println!("Success: {}", value),
        Err(error) => println!("Error: {}", error),
    }
    
    // Using if-let for concise handling
    if let Ok(value) = success {
        println!("Got success: {}", value);
    }
    
    if let Err(error) = failure {
        println!("Got error: {}", error);
    }
    
    // Using Result methods
    
    // is_ok() and is_err() for checking variant
    println!("success.is_ok(): {}", success.is_ok());
    println!("failure.is_err(): {}", failure.is_err());
    
    // unwrap() extracts the success value (panics on error)
    println!("Unwrapped success: {}", success.unwrap());
    
    // expect() is like unwrap() but with a custom error message
    println!("Expected success: {}", success.expect("Should be successful"));
    
    // unwrap_or() provides a default value if Err
    println!("Unwrapped or default: {}", failure.unwrap_or(0));
    
    // unwrap_or_else() computes a default if Err
    println!("Unwrapped or computed: {}", failure.unwrap_or_else(|_| 42 * 2));
    
    // map() transforms the contained success value
    let doubled = success.map(|x| x * 2);
    println!("Doubled success: {:?}", doubled);
    
    // map_err() transforms the contained error value
    let new_error = failure.map_err(|e| format!("Error occurred: {}", e));
    println!("New error: {:?}", new_error);
    
    // and_then() chains operations that return Result
    let chained = success.and_then(|x| Ok(x * 2));
    println!("Chained: {:?}", chained);
    
    // Functions that return Option
    fn find_even(numbers: &[i32]) -> Option<i32> {
        for &num in numbers {
            if num % 2 == 0 {
                return Some(num);
            }
        }
        None
    }
    
    let numbers = [1, 3, 5, 7, 8, 9];
    match find_even(&numbers) {
        Some(num) => println!("Found even number: {}", num),
        None => println!("No even numbers found"),
    }
    
    // Functions that return Result
    fn divide(a: i32, b: i32) -> Result<i32, String> {
        if b == 0 {
            return Err("Division by zero".to_string());
        }
        Ok(a / b)
    }
    
    // Using the ? operator with Result for early returns
    fn calculate(a: i32, b: i32) -> Result<i32, String> {
        let div = divide(a, b)?; // Returns early if divide returns Err
        let result = div * 2;
        Ok(result)
    }
    
    match calculate(10, 2) {
        Ok(result) => println!("Calculation result: {}", result),
        Err(error) => println!("Calculation error: {}", error),
    }
    
    match calculate(10, 0) {
        Ok(result) => println!("Calculation result: {}", result),
        Err(error) => println!("Calculation error: {}", error),
    }
    
    // The ? operator also works with Option
    fn first_even_doubled(numbers: &[i32]) -> Option<i32> {
        let even = find_even(numbers)?; // Returns None early if find_even returns None
        Some(even * 2)
    }
    
    match first_even_doubled(&numbers) {
        Some(result) => println!("First even doubled: {}", result),
        None => println!("Couldn't find an even number"),
    }
    
    // Converting between Option and Result
    let opt_value = Some(42);
    let result_value: Result<i32, &str> = opt_value.ok_or("No value");
    println!("Option to Result: {:?}", result_value);
    
    let err_value: Result<i32, &str> = Err("Error message");
    let opt_value: Option<i32> = err_value.ok();
    println!("Result to Option: {:?}", opt_value);
    
    // Combining multiple Options
    let x = Some(2);
    let y = Some(4);
    let sum = x.and_then(|a| y.map(|b| a + b));
    println!("Sum of options: {:?}", sum);
    
    // Collecting Options and Results
    let numbers = vec![Ok(1), Err("bad"), Ok(3)];
    
    // Using collect to filter out errors
    let successful: Option<Vec<i32>> = numbers.iter().filter_map(|x| x.ok()).collect();
    println!("Successful results: {:?}", successful);
    
    // Using partition to separate Ok and Err values
    let strings = ["42", "not a number", "64"];
    let (numbers, errors): (Vec<_>, Vec<_>) = strings
        .iter()
        .map(|s| s.parse::<i32>())
        .partition(Result::is_ok);
    
    let numbers: Vec<i32> = numbers.into_iter().map(Result::unwrap).collect();
    let errors: Vec<_> = errors.into_iter().map(Result::unwrap_err).collect();
    
    println!("Parsed numbers: {:?}", numbers);
    println!("Parse errors: {:?}", errors);
}
```

### Real-World Example
A configuration file parser that handles errors gracefully:

```rust
use std::collections::HashMap;
use std::fs::File;
use std::io::{self, BufRead, BufReader};
use std::path::Path;
use std::str::FromStr;

#[derive(Debug)]
enum ConfigError {
    IoError(io::Error),
    MissingSection,
    InvalidFormat(String),
    MissingKey(String),
    ParseError { key: String, error: String },
}

impl From<io::Error> for ConfigError {
    fn from(error: io::Error) -> Self {
        ConfigError::IoError(error)
    }
}

#[derive(Debug)]
struct Config {
    sections: HashMap<String, Section>,
}

#[derive(Debug)]
struct Section {
    name: String,
    properties: HashMap<String, String>,
}

impl Config {
    fn load(path: &Path) -> Result<Self, ConfigError> {
        let file = File::open(path)?;
        let reader = BufReader::new(file);
        
        let mut config = Config {
            sections: HashMap::new(),
        };
        
        let mut current_section: Option<String> = None;
        
        for (line_num, line_result) in reader.lines().enumerate() {
            let line = line_result?;
            let line = line.trim();
            
            // Skip empty lines and comments
            if line.is_empty() || line.starts_with('#') {
                continue;
            }
            
            // Check for section header: [section]
            if line.starts_with('[') && line.ends_with(']') {
                let section_name = line[1..line.len()-1].trim().to_string();
                if section_name.is_empty() {
                    return Err(ConfigError::InvalidFormat(
                        format!("Empty section name at line {}", line_num + 1)
                    ));
                }
                
                current_section = Some(section_name.clone());
                config.sections.insert(section_name.clone(), Section {
                    name: section_name,
                    properties: HashMap::new(),
                });
                continue;
            }
            
            // Parse key-value pairs: key = value
            if let Some(pos) = line.find('=') {
                let section_name = current_section.as_ref().ok_or(ConfigError::MissingSection)?;
                
                let key = line[..pos].trim().to_string();
                let value = line[pos+1..].trim().to_string();
                
                if key.is_empty() {
                    return Err(ConfigError::InvalidFormat(
                        format!("Empty key at line {}", line_num + 1)
                    ));
                }
                
                if let Some(section) = config.sections.get_mut(section_name) {
                    section.properties.insert(key, value);
                }
                
                continue;
            }
            
            // If we get here, the line format is invalid
            return Err(ConfigError::InvalidFormat(
                format!("Invalid line format at line {}: '{}'", line_num + 1, line)
            ));
        }
        
        Ok(config)
    }
    
    fn get_section(&self, name: &str) -> Option<&Section> {
        self.sections.get(name)
    }
    
    fn get_string(&self, section: &str, key: &str) -> Result<String, ConfigError> {
        let section = self.sections.get(section)
            .ok_or_else(|| ConfigError::MissingKey(format!("Section '{}' not found", section)))?;
        
        let value = section.properties.get(key)
            .ok_or_else(|| ConfigError::MissingKey(format!("Key '{}' not found in section '{}'", key, section.name)))?;
        
        Ok(value.clone())
    }
    
    fn get_int(&self, section: &str, key: &str) -> Result<i64, ConfigError> {
        let value = self.get_string(section, key)?;
        
        value.parse::<i64>().map_err(|e| ConfigError::ParseError {
            key: format!("{}.{}", section, key),
            error: format!("Failed to parse as integer: {}", e),
        })
    }
    
    fn get_float(&self, section: &str, key: &str) -> Result<f64, ConfigError> {
        let value = self.get_string(section, key)?;
        
        value.parse::<f64>().map_err(|e| ConfigError::ParseError {
            key: format!("{}.{}", section, key),
            error: format!("Failed to parse as float: {}", e),
        })
    }
    
    fn get_bool(&self, section: &str, key: &str) -> Result<bool, ConfigError> {
        let value = self.get_string(section, key)?;
        
        match value.to_lowercase().as_str() {
            "true" | "yes" | "1" | "on" => Ok(true),
            "false" | "no" | "0" | "off" => Ok(false),
            _ => Err(ConfigError::ParseError {
                key: format!("{}.{}", section, key),
                error: format!("Failed to parse '{}' as boolean", value),
            }),
        }
    }
    
    fn get_array<T: FromStr>(&self, section: &str, key: &str) -> Result<Vec<T>, ConfigError> {
        let value = self.get_string(section, key)?;
        
        let mut result = Vec::new();
        for item in value.split(',') {
            let item = item.trim();
            match item.parse::<T>() {
                Ok(parsed) => result.push(parsed),
                Err(_) => return Err(ConfigError::ParseError {
                    key: format!("{}.{}", section, key),
                    error: format!("Failed to parse array item '{}' to requested type", item),
                }),
            }
        }
        
        Ok(result)
    }
}

impl Section {
    fn get(&self, key: &str) -> Option<&String> {
        self.properties.get(key)
    }
}

fn main() {
    // In a real program, this would be read from an actual file
    let temp_dir = std::env::temp_dir();
    let config_path = temp_dir.join("config.ini");
    
    // Create a sample config file for demonstration
    let create_sample_config = || -> Result<(), io::Error> {
        use std::io::Write;
        let mut file = File::create(&config_path)?;
        
        write!(file, "# Sample configuration file\n\n")?;
        write!(file, "[server]\n")?;
        write!(file, "host = 127.0.0.1\n")?;
        write!(file, "port = 8080\n")?;
        write!(file, "max_connections = 100\n")?;
        write!(file, "timeout = 30\n")?;
        write!(file, "debug = true\n\n")?;
        
        write!(file, "[database]\n")?;
        write!(file, "url = postgres://user:password@localhost/mydb\n")?;
        write!(file, "pool_size = 20\n")?;
        write!(file, "timeout = 5.5\n\n")?;
        
        write!(file, "[features]\n")?;
        write!(file, "enabled = true\n")?;
        write!(file, "modules = auth,users,products,orders\n")?;
        write!(file, "rate_limits = 10,20,50,100\n")?;
        
        Ok(())
    };
    
    // Try to create the sample config file
    if let Err(e) = create_sample_config() {
        eprintln!("Failed to create sample config: {}", e);
        return;
    }
    
    println!("Created sample config at: {:?}", config_path);
    
    // Load and parse the config file
    match Config::load(&config_path) {
        Ok(config) => {
            // Successfully loaded the config
            println!("Config loaded successfully!");
            
            // Access server configuration
            if let Ok(host) = config.get_string("server", "host") {
                println!("Server host: {}", host);
            }
            
            match config.get_int("server", "port") {
                Ok(port) => println!("Server port: {}", port),
                Err(e) => println!("Error getting port: {:?}", e),
            }
            
            // Use default value if not present
            let max_conn = config.get_int("server", "max_connections").unwrap_or(50);
            println!("Max connections: {}", max_conn);
            
            // Chain operations with the ? operator in a function
            let get_server_info = || -> Result<String, ConfigError> {
                let host = config.get_string("server", "host")?;
                let port = config.get_int("server", "port")?;
                let debug = config.get_bool("server", "debug")?;
                
                Ok(format!("Server info: {}:{} (debug: {})", host, port, debug))
            };
            
            match get_server_info() {
                Ok(info) => println!("{}", info),
                Err(e) => println!("Error getting server info: {:?}", e),
            }
            
            // Access database configuration
            if let Ok(db_url) = config.get_string("database", "url") {
                println!("Database URL: {}", db_url);
            }
            
            // Parse array values
            match config.get_array::<String>("features", "modules") {
                Ok(modules) => println!("Enabled modules: {:?}", modules),
                Err(e) => println!("Error getting modules: {:?}", e),
            }
            
            match config.get_array::<i32>("features", "rate_limits") {
                Ok(limits) => println!("Rate limits: {:?}", limits),
                Err(e) => println!("Error getting rate limits: {:?}", e),
            }
            
            // Access a non-existent section/key
            match config.get_string("logging", "level") {
                Ok(level) => println!("Log level: {}", level),
                Err(e) => println!("Error getting log level: {:?}", e),
            }
            
            // Print all config
            println!("\nFull configuration:");
            for (section_name, section) in &config.sections {
                println!("[{}]", section_name);
                for (key, value) in &section.properties {
                    println!("  {} = {}", key, value);
                }
            }
        },
        Err(e) => {
            // Failed to load the config
            eprintln!("Error loading config: {:?}", e);
        }
    }
    
    // Clean up
    if let Err(e) = std::fs::remove_file(&config_path) {
        eprintln!("Failed to remove sample config: {}", e);
    }
}
```

### Common Pitfalls
- **Pitfall**: Using `unwrap()` or `expect()` in production code.
  - **Solution**: Handle both variants explicitly or use combinator methods like `unwrap_or`, `ok_or`, etc.
- **Pitfall**: Not propagating errors with `?` operator when appropriate.
  - **Solution**: Use the `?` operator in functions that return `Result` or `Option` to simplify error propagation.
- **Pitfall**: Excessive nesting when working with multiple `Option` or `Result` values.
  - **Solution**: Use combinator methods like `and_then`, `map`, or `?` to flatten code.
- **Pitfall**: Ignoring the `None` or `Err` case in critical paths.
  - **Solution**: Always handle all variants, even if just logging or providing a default.
- **Pitfall**: Converting between `Option` and `Result` incorrectly.
  - **Solution**: Use helper methods like `ok_or` for `Option` to `Result` and `ok` for `Result` to `Option`.

### Confusion Questions with Answers
1. **Q**: When should I use `Option` versus `Result`?
   **A**: Use `Option` when a value might or might not exist, but absence isn't an error (e.g., finding an element in a collection). Use `Result` when an operation might fail and you need to communicate the reason for failure (e.g., opening a file, parsing a string).

2. **Q**: What's the difference between `unwrap()`, `expect()`, and the `?` operator?
   **A**: `unwrap()` and `expect()` extract the value from an `Option` or `Result` and panic if it's `None` or `Err` (with `expect()` allowing a custom error message). The `?` operator also extracts the value but returns the `None` or `Err` from the enclosing function instead of panicking, making it ideal for error propagation.

3. **Q**: How do I combine multiple `Option` or `Result` values?
   **A**: For multiple `Option` values, use methods like `and_then` or `map` to chain operations, or use the `?` operator in a function returning `Option`. For multiple `Result` values, use similar methods or the `?` operator in a function returning `Result`. You can also use the `combine` pattern with tuples or the `collect` method to convert collections of `Result` or `Option` values.

---

## Next Actions: Exercises and Projects

### Exercises
1. **Struct Composition Practice**
   - Design a struct hierarchy for a bookstore inventory system
   - Implement methods for adding, removing, and querying books
   - Create different types of books (physical, electronic, audio) using structs
   - Add support for tracking stock levels and sales

2. **Enum Modeling Challenge**
   - Model a state machine for an order processing system
   - Create an enum for HTTP responses with different status codes and data
   - Implement a command interpreter using enums for different commands
   - Design a configuration system with type-safe settings

3. **Option and Result Workshop**
   - Implement a safe division function using Option
   - Create a function to find an element in a vector safely
   - Build a parser for a simple data format using Result for errors
   - Implement a function chain that works with both Option and Result

4. **Composite Type Conversion**
   - Implement conversions between different struct types
   - Create a flexible data structure that can be serialized to multiple formats
   - Design types that can be converted between different representations
   - Build a system for handling different units of measurement with conversions

### Micro-Project: Task Management System
Build a task management system that:
- Uses structs to represent tasks with fields like title, description, due date, etc.
- Implements an enum for task status (pending, in-progress, completed, cancelled)
- Uses Option for optional fields like assignee or completion date
- Handles errors with Result when operations can fail (e.g., loading/saving data)
- Provides methods for typical operations (create, update, delete, assign, etc.)

```rust
use std::collections::HashMap;
use std::fmt;
use std::time::{Duration, SystemTime};

// Define the task status enum
#[derive(Debug, Clone, PartialEq)]
enum TaskStatus {
    Pending,
    InProgress,
    Completed { completed_at: SystemTime },
    Cancelled { reason: String },
}

// Define the task priority enum
#[derive(Debug, Clone, PartialEq, Eq, PartialOrd, Ord)]
enum Priority {
    Low,
    Medium,
    High,
    Critical,
}

// Define the task struct
#[derive(Debug, Clone)]
struct Task {
    id: u64,
    title: String,
    description: Option<String>,
    status: TaskStatus,
    created_at: SystemTime,
    due_date: Option<SystemTime>,
    priority: Priority,
    tags: Vec<String>,
    assignee: Option<String>,
}

// Define the error enum
#[derive(Debug)]
enum TaskError {
    NotFound(u64),
    InvalidOperation(String),
    InvalidStatus(String),
    DuplicateId(u64),
}

// Implement the Task struct
impl Task {
    fn new(id: u64, title: &str, priority: Priority) -> Self {
        Task {
            id,
            title: title.to_string(),
            description: None,
            status: TaskStatus::Pending,
            created_at: SystemTime::now(),
            due_date: None,
            priority,
            tags: Vec::new(),
            assignee: None,
        }
    }
    
    fn with_description(mut self, description: &str) -> Self {
        self.description = Some(description.to_string());
        self
    }
    
    fn with_due_date(mut self, due_date: SystemTime) -> Self {
        self.due_date = Some(due_date);
        self
    }
    
    fn with_assignee(mut self, assignee: &str) -> Self {
        self.assignee = Some(assignee.to_string());
        self
    }
    
    fn with_tags(mut self, tags: Vec<String>) -> Self {
        self.tags = tags;
        self
    }
    
    fn start(&mut self) -> Result<(), TaskError> {
        match self.status {
            TaskStatus::Pending => {
                self.status = TaskStatus::InProgress;
                Ok(())
            }
            TaskStatus::InProgress => {
                Err(TaskError::InvalidOperation("Task is already in progress".to_string()))
            }
            TaskStatus::Completed { .. } => {
                Err(TaskError::InvalidOperation("Cannot start a completed task".to_string()))
            }
            TaskStatus::Cancelled { .. } => {
                Err(TaskError::InvalidOperation("Cannot start a cancelled task".to_string()))
            }
        }
    }
    
    fn complete(&mut self) -> Result<(), TaskError> {
        match self.status {
            TaskStatus::Pending | TaskStatus::InProgress => {
                self.status = TaskStatus::Completed { completed_at: SystemTime::now() };
                Ok(())
            }
            TaskStatus::Completed { .. } => {
                Err(TaskError::InvalidOperation("Task is already completed".to_string()))
            }
            TaskStatus::Cancelled { .. } => {
                Err(TaskError::InvalidOperation("Cannot complete a cancelled task".to_string()))
            }
        }
    }
    
    fn cancel(&mut self, reason: &str) -> Result<(), TaskError> {
        match self.status {
            TaskStatus::Pending | TaskStatus::InProgress => {
                self.status = TaskStatus::Cancelled { reason: reason.to_string() };
                Ok(())
            }
            TaskStatus::Completed { .. } => {
                Err(TaskError::InvalidOperation("Cannot cancel a completed task".to_string()))
            }
            TaskStatus::Cancelled { .. } => {
                Err(TaskError::InvalidOperation("Task is already cancelled".to_string()))
            }
        }
    }
    
    fn add_tag(&mut self, tag: &str) {
        if !self.tags.contains(&tag.to_string()) {
            self.tags.push(tag.to_string());
        }
    }
    
    fn remove_tag(&mut self, tag: &str) {
        self.tags.retain(|t| t != tag);
    }
    
    fn is_overdue(&self) -> bool {
        match self.due_date {
            Some(due) => {
                match self.status {
                    TaskStatus::Pending | TaskStatus::InProgress => {
                        match SystemTime::now().duration_since(due) {
                            Ok(_) => true,
                            Err(_) => false,
                        }
                    },
                    _ => false,
                }
            },
            None => false,
        }
    }
    
    fn time_until_due(&self) -> Option<Duration> {
        match self.due_date {
            Some(due) => {
                match due.duration_since(SystemTime::now()) {
                    Ok(duration) => Some(duration),
                    Err(_) => None, // Already overdue
                }
            },
            None => None,
        }
    }
}

impl fmt::Display for Task {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "[{}] {} (Priority: {:?})", self.id, self.title, self.priority)?;
        
        if let Some(description) = &self.description {
            write!(f, "\n  Description: {}", description)?;
        }
        
        match &self.status {
            TaskStatus::Pending => write!(f, "\n  Status: Pending")?,
            TaskStatus::InProgress => write!(f, "\n  Status: In Progress")?,
            TaskStatus::Completed { completed_at } => {
                let elapsed = completed_at.duration_since(self.created_at)
                    .unwrap_or(Duration::from_secs(0));
                write!(f, "\n  Status: Completed (took {:?})", elapsed)?;
            },
            TaskStatus::Cancelled { reason } => {
                write!(f, "\n  Status: Cancelled (Reason: {})", reason)?;
            }
        }
        
        if let Some(assignee) = &self.assignee {
            write!(f, "\n  Assigned to: {}", assignee)?;
        }
        
        if let Some(due_date) = &self.due_date {
            match due_date.duration_since(self.created_at) {
                Ok(duration) => {
                    write!(f, "\n  Due in: {:?}", duration)?;
                },
                Err(_) => {
                    write!(f, "\n  Due date is in the past")?;
                }
            }
        }
        
        if !self.tags.is_empty() {
            write!(f, "\n  Tags: {}", self.tags.join(", "))?;
        }
        
        Ok(())
    }
}

// Task management system
struct TaskManager {
    tasks: HashMap<u64, Task>,
    next_id: u64,
}

impl TaskManager {
    fn new() -> Self {
        TaskManager {
            tasks: HashMap::new(),
            next_id: 1,
        }
    }
    
    fn create_task(&mut self, title: &str, priority: Priority) -> Task {
        let id = self.next_id;
        self.next_id += 1;
        
        let task = Task::new(id, title, priority);
        self.tasks.insert(id, task.clone());
        task
    }
    
    fn get_task(&self, id: u64) -> Option<&Task> {
        self.tasks.get(&id)
    }
    
    fn get_task_mut(&mut self, id: u64) -> Option<&mut Task> {
        self.tasks.get_mut(&id)
    }
    
    fn update_task(&mut self, id: u64, update_fn: fn(&mut Task) -> Result<(), TaskError>) -> Result<(), TaskError> {
        match self.tasks.get_mut(&id) {
            Some(task) => update_fn(task),
            None => Err(TaskError::NotFound(id)),
        }
    }
    
    fn delete_task(&mut self, id: u64) -> Result<Task, TaskError> {
        match self.tasks.remove(&id) {
            Some(task) => Ok(task),
            None => Err(TaskError::NotFound(id)),
        }
    }
    
    fn list_all_tasks(&self) -> Vec<&Task> {
        self.tasks.values().collect()
    }
    
    fn list_tasks_by_status(&self, status: fn(&TaskStatus) -> bool) -> Vec<&Task> {
        self.tasks
            .values()
            .filter(|task| status(&task.status))
            .collect()
    }
    
    fn list_pending_tasks(&self) -> Vec<&Task> {
        self.list_tasks_by_status(|status| matches!(status, TaskStatus::Pending))
    }
    
    fn list_in_progress_tasks(&self) -> Vec<&Task> {
        self.list_tasks_by_status(|status| matches!(status, TaskStatus::InProgress))
    }
    
    fn list_completed_tasks(&self) -> Vec<&Task> {
        self.list_tasks_by_status(|status| matches!(status, TaskStatus::Completed { .. }))
    }
    
    fn list_cancelled_tasks(&self) -> Vec<&Task> {
        self.list_tasks_by_status(|status| matches!(status, TaskStatus::Cancelled { .. }))
    }
    
    fn list_tasks_by_assignee(&self, assignee: &str) -> Vec<&Task> {
        self.tasks
            .values()
            .filter(|task| task.assignee.as_deref() == Some(assignee))
            .collect()
    }
    
    fn list_tasks_by_tag(&self, tag: &str) -> Vec<&Task> {
        self.tasks
            .values()
            .filter(|task| task.tags.contains(&tag.to_string()))
            .collect()
    }
    
    fn list_overdue_tasks(&self) -> Vec<&Task> {
        self.tasks
            .values()
            .filter(|task| task.is_overdue())
            .collect()
    }
    
    fn start_task(&mut self, id: u64) -> Result<(), TaskError> {
        self.update_task(id, |task| task.start())
    }
    
    fn complete_task(&mut self, id: u64) -> Result<(), TaskError> {
        self.update_task(id, |task| task.complete())
    }
    
    fn cancel_task(&mut self, id: u64, reason: &str) -> Result<(), TaskError> {
        let reason_clone = reason.to_string();
        self.update_task(id, move |task| task.cancel(&reason_clone))
    }
    
    fn assign_task(&mut self, id: u64, assignee: &str) -> Result<(), TaskError> {
        let assignee_clone = assignee.to_string();
        self.update_task(id, move |task| {
            task.assignee = Some(assignee_clone);
            Ok(())
        })
    }
    
    fn add_tag_to_task(&mut self, id: u64, tag: &str) -> Result<(), TaskError> {
        let tag_clone = tag.to_string();
        self.update_task(id, move |task| {
            task.add_tag(&tag_clone);
            Ok(())
        })
    }
}

fn main() {
    let mut manager = TaskManager::new();
    
    // Create some tasks
    let task1 = manager.create_task("Implement task manager", Priority::High)
        .with_description("Create a Rust program to manage tasks")
        .with_assignee("Alice");
    
    let task2 = manager.create_task("Write documentation", Priority::Medium)
        .with_description("Document the API for the task manager")
        .with_due_date(SystemTime::now() + Duration::from_secs(60 * 60 * 24 * 7)); // Due in a week
    
    let task3 = manager.create_task("Fix bugs", Priority::Critical)
        .with_description("Fix all reported bugs")
        .with_assignee("Bob")
        .with_tags(vec!["bug".to_string(), "urgent".to_string()]);
    
    // Start a task
    if let Err(e) = manager.start_task(task1.id) {
        println!("Error starting task: {:?}", e);
    } else {
        println!("Started task: {}", task1.id);
    }
    
    // Complete a task
    if let Err(e) = manager.complete_task(task1.id) {
        println!("Error completing task: {:?}", e);
    } else {
        println!("Completed task: {}", task1.id);
    }
    
    // Cancel a task
    if let Err(e) = manager.cancel_task(task2.id, "Postponed to next sprint") {
        println!("Error cancelling task: {:?}", e);
    } else {
        println!("Cancelled task: {}", task2.id);
    }
    
    // List all tasks
    println!("\nAll tasks:");
    for task in manager.list_all_tasks() {
        println!("{}", task);
    }
    
    // List pending tasks
    println!("\nPending tasks:");
    for task in manager.list_pending_tasks() {
        println!("{}", task);
    }
    
    // List completed tasks
    println!("\nCompleted tasks:");
    for task in manager.list_completed_tasks() {
        println!("{}", task);
    }
    
    // List cancelled tasks
    println!("\nCancelled tasks:");
    for task in manager.list_cancelled_tasks() {
        println!("{}", task);
    }
    
    // List tasks by assignee
    println!("\nTasks assigned to Bob:");
    for task in manager.list_tasks_by_assignee("Bob") {
        println!("{}", task);
    }
    
    // List tasks by tag
    println!("\nTasks tagged as 'urgent':");
    for task in manager.list_tasks_by_tag("urgent") {
        println!("{}", task);
    }
    
    // Add a tag to a task
    if let Err(e) = manager.add_tag_to_task(task3.id, "high-priority") {
        println!("Error adding tag: {:?}", e);
    } else {
        println!("Added tag to task: {}", task3.id);
    }
    
    // Try to start a completed task (should fail)
    match manager.start_task(task1.id) {
        Ok(_) => println!("Successfully started task"),
        Err(e) => println!("Expected error: {:?}", e),
    }
    
    // Get specific task
    if let Some(task) = manager.get_task(task3.id) {
        println!("\nTask details:\n{}", task);
        
        if let Some(time_until_due) = task.time_until_due() {
            println!("Time until due: {:?}", time_until_due);
        } else if task.is_overdue() {
            println!("Task is overdue!");
        }
    }
    
    // Delete a task
    match manager.delete_task(task3.id) {
        Ok(task) => println!("\nDeleted task: {}", task.title),
        Err(e) => println!("Error deleting task: {:?}", e),
    }
    
    // Try to get a deleted task (should return None)
    if manager.get_task(task3.id).is_none() {
        println!("Task {} was successfully deleted", task3.id);
    }
}
```

## Success Criteria
You've mastered structs and enums in Rust when you can:
1. Design appropriate type hierarchies for your domain using structs and enums
2. Implement methods and associated functions for your types
3. Use pattern matching effectively with enum variants
4. Apply the Option and Result types appropriately for error handling
5. Create clean, ergonomic APIs for your types
6. Choose the right composite type for a given problem
7. Implement the builder pattern and other initialization patterns
8. Handle complex state transitions using enums and pattern matching

## Troubleshooting Advice
1. **Struct Field Visibility Issues**
   - Remember fields are private by default
   - Use `pub` for fields that need to be accessed outside the module
   - Consider providing getter and setter methods instead of making fields public
   - Be consistent with your visibility choices across a module

2. **Pattern Matching Problems**
   - Ensure your `match` expressions are exhaustive or use `_` for catch-all
   - Use if-let for simpler, single-pattern matches
   - Remember to destructure enum variants to access their data
   - Check for refutable patterns in `let` bindings (which won't compile)

3. **Enum Design Concerns**
   - Avoid creating enums with too many variants; consider splitting into multiple types
   - Use enums for representing states or variants, not for polymorphism
   - Consider using structs with trait implementations for code reuse
   - Remember enums can have methods just like structs

4. **Option and Result Handling**
   - Avoid excessive unwrapping with `unwrap()` or `expect()`
   - Use the `?` operator for clean error propagation
   - Chain combinator methods instead of excessive nesting
   - Consider using the `anyhow` or `thiserror` crates for more complex error handling

5. **Initialization Complexity**
   - Use builder patterns for types with many optional fields
   - Implement the `Default` trait for common default values
   - Consider method chaining with `self` returns for fluent interfaces
   - Use field init shorthand when variable names match field names

Remember that the choice of struct vs. enum depends on your use case: use structs for entities with a fixed set of fields that all exist at once, and enums for representing values that can be in one of several possible states or variants.
