# Module 5: Methods and Traits - Part 1: Methods

## Core Problem and Key Assumptions
- **Problem**: Understanding how to add behavior to types in Rust through methods and associated functions
- **Key Assumptions**:
  - You understand basic Rust syntax, structs, and enums
  - You want to organize code by attaching functions to types
  - You need to implement clean, reusable APIs for your data structures
  - You aim to leverage Rust's ownership model in method design

## Essential Concepts to Cover
1. Defining Methods on Types
2. Self, &self, and &mut self Receiver Types
3. Associated Functions
4. Builder Pattern

---

## 1. Defining Methods on Types

### Concise Explanation
Methods are functions attached to a specific type, allowing you to organize code by grouping functionality with the data it operates on. In Rust, methods are defined within `impl` blocks associated with a type. Methods take a special first parameter, `self` (in various forms), which represents the instance they're called on.

Defining methods offers several advantages: they provide namespace management, enable method chaining, improve code readability, and help organize related functionality. Unlike standalone functions, methods have access to the data they're attached to through `self`.

### Where to Use
- When functionality is closely tied to a specific data type
- To encapsulate implementation details while exposing a clean interface
- When implementing object-oriented patterns in Rust
- For fluent interfaces with method chaining
- To implement traits on your types
- To provide type-specific utility operations

### Code Snippet
```rust
// Define a struct
struct Rectangle {
    width: f64,
    height: f64,
}

// Implement methods on the Rectangle type
impl Rectangle {
    // Method with &self (immutable reference to self)
    fn area(&self) -> f64 {
        self.width * self.height
    }
    
    // Method with &mut self (mutable reference to self)
    fn scale(&mut self, factor: f64) {
        self.width *= factor;
        self.height *= factor;
    }
    
    // Method that consumes self
    fn into_square(self) -> Square {
        let side = (self.width + self.height) / 2.0;
        Square { side }
    }
    
    // Associated function (constructor)
    fn new(width: f64, height: f64) -> Rectangle {
        Rectangle { width, height }
    }
    
    // Method with additional parameters
    fn can_contain(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
    
    // Method that returns &self for method chaining
    fn validate(&self) -> &Self {
        assert!(self.width > 0.0, "Width must be positive");
        assert!(self.height > 0.0, "Height must be positive");
        self
    }
}

// Define another struct for the example
struct Square {
    side: f64,
}

// Implement methods on an enum
enum Shape {
    Circle(f64),  // radius
    Rectangle(f64, f64),  // width, height
    Triangle(f64, f64, f64),  // sides
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
    
    fn describe(&self) -> String {
        match self {
            Shape::Circle(_) => "circle".to_string(),
            Shape::Rectangle(_, _) => "rectangle".to_string(),
            Shape::Triangle(_, _, _) => "triangle".to_string(),
        }
    }
}

// Implementing methods on generic types
struct Pair<T> {
    first: T,
    second: T,
}

impl<T> Pair<T> {
    fn new(first: T, second: T) -> Self {
        Pair { first, second }
    }
    
    fn get_first(&self) -> &T {
        &self.first
    }
    
    fn get_second(&self) -> &T {
        &self.second
    }
}

// Implementing methods conditional on trait bounds
impl<T: std::fmt::Display> Pair<T> {
    fn print(&self) {
        println!("({}, {})", self.first, self.second);
    }
}

fn main() {
    // Using methods
    let mut rect = Rectangle::new(10.0, 5.0);
    println!("Area: {}", rect.area());
    
    // Method chaining
    rect.validate();
    
    // Mutable method
    rect.scale(2.0);
    println!("Area after scaling: {}", rect.area());
    
    // Method with parameters
    let rect2 = Rectangle::new(3.0, 2.0);
    println!("rect can contain rect2: {}", rect.can_contain(&rect2));
    
    // Consuming method
    let square = rect.into_square();
    
    // Methods on enums
    let circle = Shape::Circle(5.0);
    println!("The {} has area {}", circle.describe(), circle.area());
    
    // Methods on generic types
    let numbers = Pair::new(1, 2);
    println!("First number: {}", numbers.get_first());
    
    // Methods with trait bounds
    let strings = Pair::new("hello", "world");
    strings.print();
}
```

### Real-World Example
A temperature conversion system with methods:

```rust
#[derive(Debug, Clone, Copy)]
enum TemperatureUnit {
    Celsius,
    Fahrenheit,
    Kelvin,
}

#[derive(Debug, Clone, Copy)]
struct Temperature {
    value: f64,
    unit: TemperatureUnit,
}

impl Temperature {
    // Constructor
    fn new(value: f64, unit: TemperatureUnit) -> Self {
        Temperature { value, unit }
    }
    
    // Constructors for specific units
    fn from_celsius(value: f64) -> Self {
        Temperature::new(value, TemperatureUnit::Celsius)
    }
    
    fn from_fahrenheit(value: f64) -> Self {
        Temperature::new(value, TemperatureUnit::Fahrenheit)
    }
    
    fn from_kelvin(value: f64) -> Self {
        Temperature::new(value, TemperatureUnit::Kelvin)
    }
    
    // Conversion methods
    fn to_celsius(&self) -> f64 {
        match self.unit {
            TemperatureUnit::Celsius => self.value,
            TemperatureUnit::Fahrenheit => (self.value - 32.0) * 5.0 / 9.0,
            TemperatureUnit::Kelvin => self.value - 273.15,
        }
    }
    
    fn to_fahrenheit(&self) -> f64 {
        match self.unit {
            TemperatureUnit::Celsius => self.value * 9.0 / 5.0 + 32.0,
            TemperatureUnit::Fahrenheit => self.value,
            TemperatureUnit::Kelvin => self.value * 9.0 / 5.0 - 459.67,
        }
    }
    
    fn to_kelvin(&self) -> f64 {
        match self.unit {
            TemperatureUnit::Celsius => self.value + 273.15,
            TemperatureUnit::Fahrenheit => (self.value + 459.67) * 5.0 / 9.0,
            TemperatureUnit::Kelvin => self.value,
        }
    }
    
    // Convert to another Temperature with different unit
    fn convert(&self, unit: TemperatureUnit) -> Self {
        let value = match unit {
            TemperatureUnit::Celsius => self.to_celsius(),
            TemperatureUnit::Fahrenheit => self.to_fahrenheit(),
            TemperatureUnit::Kelvin => self.to_kelvin(),
        };
        
        Temperature::new(value, unit)
    }
    
    // Temperature manipulation methods
    fn add(&self, other: &Temperature) -> Self {
        // Convert other to self's unit first
        let other_converted = other.convert(self.unit);
        Temperature::new(self.value + other_converted.value, self.unit)
    }
    
    fn subtract(&self, other: &Temperature) -> Self {
        // Convert other to self's unit first
        let other_converted = other.convert(self.unit);
        Temperature::new(self.value - other_converted.value, self.unit)
    }
    
    fn scale(&self, factor: f64) -> Self {
        // Scale absolute temperature (convert to Kelvin first)
        let kelvin_value = self.to_kelvin();
        let new_kelvin = kelvin_value * factor;
        
        // Convert back to original unit
        Temperature::from_kelvin(new_kelvin).convert(self.unit)
    }
    
    // Utility methods
    fn is_freezing(&self) -> bool {
        self.to_celsius() <= 0.0
    }
    
    fn is_boiling(&self) -> bool {
        self.to_celsius() >= 100.0
    }
    
    fn format(&self, precision: usize) -> String {
        let symbol = match self.unit {
            TemperatureUnit::Celsius => "°C",
            TemperatureUnit::Fahrenheit => "°F",
            TemperatureUnit::Kelvin => "K",
        };
        
        format!("{:.precision$}{}", self.value, symbol, precision = precision)
    }
}

// A temperature logger that uses the Temperature type
struct TemperatureLogger {
    temperatures: Vec<(Temperature, std::time::SystemTime)>,
    unit_preference: TemperatureUnit,
}

impl TemperatureLogger {
    fn new(unit_preference: TemperatureUnit) -> Self {
        TemperatureLogger {
            temperatures: Vec::new(),
            unit_preference,
        }
    }
    
    fn log(&mut self, temperature: Temperature) {
        self.temperatures.push((temperature, std::time::SystemTime::now()));
    }
    
    fn average(&self) -> Option<Temperature> {
        if self.temperatures.is_empty() {
            return None;
        }
        
        // Convert all temperatures to the preferred unit for averaging
        let sum: f64 = self.temperatures
            .iter()
            .map(|(temp, _)| temp.convert(self.unit_preference).value)
            .sum();
        
        let avg = sum / self.temperatures.len() as f64;
        Some(Temperature::new(avg, self.unit_preference))
    }
    
    fn min(&self) -> Option<Temperature> {
        if self.temperatures.is_empty() {
            return None;
        }
        
        // Find minimum temperature in the preferred unit
        let min_temp = self.temperatures
            .iter()
            .min_by(|(a, _), (b, _)| {
                let a_value = a.convert(self.unit_preference).value;
                let b_value = b.convert(self.unit_preference).value;
                a_value.partial_cmp(&b_value).unwrap()
            })
            .map(|(temp, _)| temp.convert(self.unit_preference));
        
        min_temp
    }
    
    fn max(&self) -> Option<Temperature> {
        if self.temperatures.is_empty() {
            return None;
        }
        
        // Find maximum temperature in the preferred unit
        let max_temp = self.temperatures
            .iter()
            .max_by(|(a, _), (b, _)| {
                let a_value = a.convert(self.unit_preference).value;
                let b_value = b.convert(self.unit_preference).value;
                a_value.partial_cmp(&b_value).unwrap()
            })
            .map(|(temp, _)| temp.convert(self.unit_preference));
        
        max_temp
    }
    
    fn change_unit_preference(&mut self, unit: TemperatureUnit) {
        self.unit_preference = unit;
    }
    
    fn print_summary(&self) {
        println!("Temperature Log Summary:");
        println!("Number of readings: {}", self.temperatures.len());
        
        if let Some(avg) = self.average() {
            println!("Average temperature: {}", avg.format(1));
        }
        
        if let Some(min) = self.min() {
            println!("Minimum temperature: {}", min.format(1));
        }
        
        if let Some(max) = self.max() {
            println!("Maximum temperature: {}", max.format(1));
        }
    }
}

fn main() {
    // Create temperatures in different units
    let room_temp = Temperature::from_celsius(22.5);
    let body_temp = Temperature::from_fahrenheit(98.6);
    let boiling = Temperature::from_celsius(100.0);
    let absolute_zero = Temperature::from_kelvin(0.0);
    
    // Convert temperatures
    println!("Room temperature: {} = {} = {}",
             room_temp.format(1),
             room_temp.convert(TemperatureUnit::Fahrenheit).format(1),
             room_temp.convert(TemperatureUnit::Kelvin).format(1));
    
    println!("Body temperature: {} = {}",
             body_temp.format(1),
             body_temp.convert(TemperatureUnit::Celsius).format(1));
    
    // Check states
    println!("Is boiling water boiling? {}", boiling.is_boiling());
    println!("Is absolute zero freezing? {}", absolute_zero.is_freezing());
    
    // Perform arithmetic
    let temp_difference = body_temp.subtract(&room_temp);
    println!("Temperature difference: {}", temp_difference.format(1));
    
    let double_room_temp = room_temp.scale(2.0);
    println!("Double room temperature (from absolute): {}", double_room_temp.format(1));
    
    // Use the temperature logger
    let mut logger = TemperatureLogger::new(TemperatureUnit::Celsius);
    
    // Log some temperatures
    logger.log(room_temp);
    logger.log(body_temp);
    logger.log(Temperature::from_celsius(18.5)); // Morning temperature
    logger.log(Temperature::from_celsius(24.8)); // Afternoon temperature
    
    // Print summary
    logger.print_summary();
    
    // Change unit preference
    logger.change_unit_preference(TemperatureUnit::Fahrenheit);
    println!("\nSummary in Fahrenheit:");
    logger.print_summary();
}
```

### Common Pitfalls
- **Pitfall**: Confusion about when to use `self`, `&self`, or `&mut self`.
  - **Solution**: Choose based on whether the method needs to consume, read, or modify the instance. Use `&self` for read-only access, `&mut self` for modification, and `self` when transferring ownership.
- **Pitfall**: Accidental ownership transfer with `self`.
  - **Solution**: Be deliberate about using `self` by consuming value only when creating a new one or transferring ownership is the intended behavior.
- **Pitfall**: Implementing methods on generic types without required trait bounds.
  - **Solution**: Add necessary trait bounds or use conditional implementation with `impl<T: SomeTrait>`.
- **Pitfall**: Method name conflicts with inherent methods from traits.
  - **Solution**: Choose distinct method names or use fully qualified syntax when needed.
- **Pitfall**: Overly complex method chains that sacrifice readability.
  - **Solution**: Balance method chaining for fluency with readability; break complex chains into intermediate steps when appropriate.

### Confusion Questions with Answers
1. **Q**: When should I implement a method versus a standalone function?
   **A**: Implement a method when functionality is closely tied to a specific type and needs access to its internals. Use standalone functions when:
   - The operation works with multiple types or doesn't conceptually belong to any type
   - The operation is very general or utility-like
   - You're implementing algorithms that operate on many different types

2. **Q**: How do methods affect Rust's borrow checker?
   **A**: Methods use clear receiver types that inform the borrow checker: `&self` borrows immutably, allowing multiple readers; `&mut self` borrows mutably, allowing only one writer and no readers; `self` takes ownership, consuming the value. This explicit ownership model helps prevent memory safety issues at compile time.

3. **Q**: Can I implement methods for types I didn't define, like standard library types?
   **A**: Yes, you can implement methods for any type in your crate or if either the type or trait is defined in your crate (the "orphan rule"). To add methods to external types, you need to create a custom trait and implement it for those types.

---

## 2. Self, &self, and &mut self Receiver Types

### Concise Explanation
Rust methods use receiver parameters that determine how the method interacts with the instance it's called on. The three primary receiver types are:

1. `self`: Takes ownership of the instance, consuming it. The method can do anything with the value, but the original instance cannot be used after the method call.

2. `&self`: Borrows the instance immutably. The method can read but not modify the instance. Multiple methods can borrow the same instance immutably at the same time.

3. `&mut self`: Borrows the instance mutably. The method can read and modify the instance, but requires exclusive access to the value.

Additionally, Rust provides `Self` (capital S) as a type alias for the type the `impl` block is for, which is useful in return types and associated functions.

### Where to Use
- **`self`**: For methods that consume the instance, usually transforming it into something else or when transferring ownership
- **`&self`**: For methods that only read data (getters, calculations, validations)
- **`&mut self`**: For methods that modify the instance (setters, state changes)
- **`Self`**: In return types, associated functions, and as type reference

### Code Snippet
```rust
struct Counter {
    count: usize,
    step: usize,
    name: String,
}

impl Counter {
    // Associated function (constructor)
    fn new(name: &str) -> Self {
        Counter {
            count: 0,
            step: 1,
            name: name.to_string(),
        }
    }
    
    // Takes &self - immutable reference
    // Used for methods that just read the instance
    fn current(&self) -> usize {
        self.count
    }
    
    fn name(&self) -> &str {
        &self.name
    }
    
    fn describe(&self) -> String {
        format!("Counter '{}' is at {} (step: {})", 
                self.name, self.count, self.step)
    }
    
    // Takes &mut self - mutable reference
    // Used for methods that modify the instance
    fn increment(&mut self) {
        self.count += self.step;
    }
    
    fn decrement(&mut self) -> Result<(), String> {
        if self.count < self.step {
            return Err("Cannot decrement below zero".to_string());
        }
        self.count -= self.step;
        Ok(())
    }
    
    fn set_count(&mut self, value: usize) {
        self.count = value;
    }
    
    fn set_step(&mut self, value: usize) -> &mut Self {
        self.step = value;
        self // Return self for method chaining
    }
    
    fn reset(&mut self) -> &mut Self {
        self.count = 0;
        self // Return self for method chaining
    }
    
    // Takes self - consumes the instance (ownership transfer)
    // Used when transforming into another type or consuming resources
    fn into_named_tuple(self) -> (String, usize, usize) {
        (self.name, self.count, self.step)
    }
    
    fn finalize(self) -> usize {
        println!("Counter '{}' finalized at {}", self.name, self.count);
        self.count
    }
    
    // Clone first if you need the original
    fn double(self) -> Self {
        let mut new_counter = self;
        new_counter.count *= 2;
        new_counter
    }
    
    // Advanced: Generic method with elided lifetime
    fn as_tuple(&self) -> (&str, usize) {
        (&self.name, self.count)
    }
    
    // Advanced: Generic method with explicit lifetime
    fn view<'a>(&'a self) -> CounterView<'a> {
        CounterView { counter: self }
    }
}

// A struct that borrows a Counter
struct CounterView<'a> {
    counter: &'a Counter,
}

impl<'a> CounterView<'a> {
    fn display(&self) {
        println!("View of counter '{}': {}", 
                 self.counter.name, self.counter.count);
    }
}

// Another example with a different type
struct StateMachine<T> {
    state: T,
    transitions: usize,
}

// Implementing generic methods with Self
impl<T> StateMachine<T> {
    // Using Self as return type (refers to StateMachine<T>)
    fn new(initial_state: T) -> Self {
        StateMachine {
            state: initial_state,
            transitions: 0,
        }
    }
    
    fn get_transitions(&self) -> usize {
        self.transitions
    }
    
    // Using &self for reading state
    fn get_state_ref(&self) -> &T {
        &self.state
    }
    
    // Using &mut self for updating state
    fn set_state(&mut self, new_state: T) -> &mut Self {
        self.state = new_state;
        self.transitions += 1;
        self
    }
    
    // Using self for consuming the machine
    fn into_state(self) -> T {
        self.state
    }
}

// Using specific types with receiver type requirements
impl StateMachine<String> {
    fn append_to_state(&mut self, suffix: &str) {
        self.state.push_str(suffix);
        self.transitions += 1;
    }
}

fn main() {
    // Using methods with different receiver types
    let mut counter = Counter::new("Main");
    
    // &self methods (immutable reference)
    println!("Counter name: {}", counter.name());
    println!("Current count: {}", counter.current());
    println!("Description: {}", counter.describe());
    
    // &mut self methods (mutable reference)
    counter.increment();
    counter.increment();
    println!("After incrementing: {}", counter.current());
    
    counter.set_step(5).reset();
    println!("After chained methods: {}", counter.describe());
    
    // Error handling with Result
    match counter.decrement() {
        Ok(()) => println!("Decremented successfully"),
        Err(e) => println!("Error: {}", e),
    }
    
    // Using the counter view with borrowed reference
    let view = counter.view();
    view.display();
    
    // Create a clone before consuming methods
    let counter_clone = counter.clone();
    
    // self methods (consumes the instance)
    let doubled = counter.double();
    println!("Doubled counter: {}", doubled.describe());
    
    // counter is no longer available here!
    // println!("Original: {}", counter.describe()); // This would not compile
    
    // But the clone is still available
    println!("Clone: {}", counter_clone.describe());
    
    // Consume the doubled counter
    let final_count = doubled.finalize();
    println!("Final count: {}", final_count);
    
    // Using generic StateMachine
    let mut machine = StateMachine::new("INIT".to_string());
    machine.set_state("RUNNING".to_string());
    machine.append_to_state("_FAST");
    
    println!("Machine state: {}", machine.get_state_ref());
    println!("Transitions: {}", machine.get_transitions());
    
    // Consume the machine to get the final state
    let final_state = machine.into_state();
    println!("Final state: {}", final_state);
}
```

### Real-World Example
A document editing system demonstrating different receiver types:

```rust
use std::collections::HashMap;
use std::time::{SystemTime, UNIX_EPOCH};

#[derive(Debug, Clone)]
enum ContentBlock {
    Text(String),
    Image { url: String, alt_text: String, width: u32, height: u32 },
    List { items: Vec<String>, ordered: bool },
    Code { content: String, language: String },
}

#[derive(Debug, Clone)]
struct Document {
    id: String,
    title: String,
    author: String,
    content: Vec<ContentBlock>,
    metadata: HashMap<String, String>,
    created_at: u64,
    updated_at: u64,
}

impl Document {
    // Associated function - creates a new Document instance
    fn new(id: &str, title: &str, author: &str) -> Self {
        let timestamp = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        Document {
            id: id.to_string(),
            title: title.to_string(),
            author: author.to_string(),
            content: Vec::new(),
            metadata: HashMap::new(),
            created_at: timestamp,
            updated_at: timestamp,
        }
    }
    
    // &self methods - read-only access to document
    fn word_count(&self) -> usize {
        self.content.iter().map(|block| match block {
            ContentBlock::Text(text) => text.split_whitespace().count(),
            ContentBlock::List { items, .. } => {
                items.iter().map(|item| item.split_whitespace().count()).sum()
            },
            ContentBlock::Code { content, .. } => content.split_whitespace().count(),
            ContentBlock::Image { alt_text, .. } => alt_text.split_whitespace().count(),
        }).sum()
    }
    
    fn get_title(&self) -> &str {
        &self.title
    }
    
    fn get_author(&self) -> &str {
        &self.author
    }
    
    fn get_metadata(&self, key: &str) -> Option<&String> {
        self.metadata.get(key)
    }
    
    fn content_length(&self) -> usize {
        self.content.len()
    }
    
    fn contains_text(&self, search_text: &str) -> bool {
        for block in &self.content {
            match block {
                ContentBlock::Text(text) => {
                    if text.contains(search_text) {
                        return true;
                    }
                },
                ContentBlock::List { items, .. } => {
                    for item in items {
                        if item.contains(search_text) {
                            return true;
                        }
                    }
                },
                ContentBlock::Code { content, .. } => {
                    if content.contains(search_text) {
                        return true;
                    }
                },
                ContentBlock::Image { alt_text, .. } => {
                    if alt_text.contains(search_text) {
                        return true;
                    }
                },
            }
        }
        false
    }
    
    fn preview(&self, length: usize) -> String {
        for block in &self.content {
            if let ContentBlock::Text(text) = block {
                if !text.is_empty() {
                    if text.len() <= length {
                        return text.clone();
                    } else {
                        return text[..length].to_string() + "...";
                    }
                }
            }
        }
        "".to_string()
    }
    
    fn time_since_update(&self) -> u64 {
        let now = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        now - self.updated_at
    }
    
    // &mut self methods - modify the document
    fn add_text(&mut self, text: &str) -> &mut Self {
        self.content.push(ContentBlock::Text(text.to_string()));
        self.update_timestamp();
        self
    }
    
    fn add_image(&mut self, url: &str, alt_text: &str, width: u32, height: u32) -> &mut Self {
        self.content.push(ContentBlock::Image {
            url: url.to_string(),
            alt_text: alt_text.to_string(),
            width,
            height,
        });
        self.update_timestamp();
        self
    }
    
    fn add_list(&mut self, items: Vec<String>, ordered: bool) -> &mut Self {
        self.content.push(ContentBlock::List { items, ordered });
        self.update_timestamp();
        self
    }
    
    fn add_code(&mut self, content: &str, language: &str) -> &mut Self {
        self.content.push(ContentBlock::Code {
            content: content.to_string(),
            language: language.to_string(),
        });
        self.update_timestamp();
        self
    }
    
    fn set_title(&mut self, title: &str) -> &mut Self {
        self.title = title.to_string();
        self.update_timestamp();
        self
    }
    
    fn set_metadata(&mut self, key: &str, value: &str) -> &mut Self {
        self.metadata.insert(key.to_string(), value.to_string());
        self.update_timestamp();
        self
    }
    
    fn remove_metadata(&mut self, key: &str) -> &mut Self {
        self.metadata.remove(key);
        self.update_timestamp();
        self
    }
    
    fn remove_block(&mut self, index: usize) -> Result<ContentBlock, String> {
        if index >= self.content.len() {
            return Err(format!("Index {} out of bounds", index));
        }
        
        let block = self.content.remove(index);
        self.update_timestamp();
        Ok(block)
    }
    
    fn update_timestamp(&mut self) {
        self.updated_at = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
    }
    
    fn clear_content(&mut self) -> &mut Self {
        self.content.clear();
        self.update_timestamp();
        self
    }
    
    // self methods - consume the document
    fn merge(self, other: Document) -> Document {
        let mut merged = self;
        
        // Combine content
        for block in other.content {
            merged.content.push(block);
        }
        
        // Combine metadata
        for (key, value) in other.metadata {
            merged.metadata.insert(key, value);
        }
        
        // Update timestamp
        merged.update_timestamp();
        
        merged
    }
    
    fn into_archive(self) -> ArchivedDocument {
        ArchivedDocument {
            original_id: self.id,
            title: self.title,
            author: self.author,
            word_count: self.word_count(),
            archived_at: SystemTime::now()
                .duration_since(UNIX_EPOCH)
                .unwrap()
                .as_secs(),
        }
    }
    
    fn transform(self, transformation: fn(Document) -> Document) -> Document {
        transformation(self)
    }
}

#[derive(Debug)]
struct ArchivedDocument {
    original_id: String,
    title: String,
    author: String,
    word_count: usize,
    archived_at: u64,
}

// DocumentRenderer borrows a Document and renders it
struct DocumentRenderer<'a> {
    document: &'a Document,
    format: RenderFormat,
}

enum RenderFormat {
    Plain,
    HTML,
    Markdown,
}

impl<'a> DocumentRenderer<'a> {
    fn new(document: &'a Document, format: RenderFormat) -> Self {
        DocumentRenderer { document, format }
    }
    
    fn render(&self) -> String {
        let mut output = String::new();
        
        match self.format {
            RenderFormat::Plain => {
                output.push_str(&format!("Title: {}\n", self.document.title));
                output.push_str(&format!("Author: {}\n\n", self.document.author));
                
                for block in &self.document.content {
                    match block {
                        ContentBlock::Text(text) => {
                            output.push_str(text);
                            output.push_str("\n\n");
                        },
                        ContentBlock::Image { alt_text, .. } => {
                            output.push_str(&format!("[Image: {}]\n\n", alt_text));
                        },
                        ContentBlock::List { items, ordered } => {
                            if *ordered {
                                for (i, item) in items.iter().enumerate() {
                                    output.push_str(&format!("{}. {}\n", i+1, item));
                                }
                            } else {
                                for item in items {
                                    output.push_str(&format!("- {}\n", item));
                                }
                            }
                            output.push_str("\n");
                        },
                        ContentBlock::Code { content, language } => {
                            output.push_str(&format!("Code ({})\n", language));
                            output.push_str(content);
                            output.push_str("\n\n");
                        },
                    }
                }
            },
            RenderFormat::HTML => {
                output.push_str(&format!("<h1>{}</h1>\n", self.document.title));
                output.push_str(&format!("<p>By: {}</p>\n", self.document.author));
                
                for block in &self.document.content {
                    match block {
                        ContentBlock::Text(text) => {
                            output.push_str(&format!("<p>{}</p>\n", text));
                        },
                        ContentBlock::Image { url, alt_text, width, height } => {
                            output.push_str(&format!(
                                "<img src=\"{}\" alt=\"{}\" width=\"{}\" height=\"{}\">\n",
                                url, alt_text, width, height
                            ));
                        },
                        ContentBlock::List { items, ordered } => {
                            if *ordered {
                                output.push_str("<ol>\n");
                                for item in items {
                                    output.push_str(&format!("  <li>{}</li>\n", item));
                                }
                                output.push_str("</ol>\n");
                            } else {
                                output.push_str("<ul>\n");
                                for item in items {
                                    output.push_str(&format!("  <li>{}</li>\n", item));
                                }
                                output.push_str("</ul>\n");
                            }
                        },
                        ContentBlock::Code { content, language } => {
                            output.push_str(&format!(
                                "<pre><code class=\"language-{}\">{}</code></pre>\n",
                                language, content
                            ));
                        },
                    }
                }
            },
            RenderFormat::Markdown => {
                output.push_str(&format!("# {}\n", self.document.title));
                output.push_str(&format!("By: {}\n\n", self.document.author));
                
                for block in &self.document.content {
                    match block {
                        ContentBlock::Text(text) => {
                            output.push_str(text);
                            output.push_str("\n\n");
                        },
                        ContentBlock::Image { url, alt_text, .. } => {
                            output.push_str(&format!("![{}]({})\n\n", alt_text, url));
                        },
                        ContentBlock::List { items, ordered } => {
                            for (i, item) in items.iter().enumerate() {
                                if *ordered {
                                    output.push_str(&format!("{}. {}\n", i+1, item));
                                } else {
                                    output.push_str(&format!("- {}\n", item));
                                }
                            }
                            output.push_str("\n");
                        },
                        ContentBlock::Code { content, language } => {
                            output.push_str(&format!("```{}\n", language));
                            output.push_str(content);
                            output.push_str("\n```\n\n");
                        },
                    }
                }
            },
        }
        
        output
    }
}

fn main() {
    // Create a new document
    let mut doc = Document::new("doc-001", "Rust Methods Guide", "Alice Smith");
    
    // Using &mut self methods with method chaining
    doc.add_text("# Understanding Rust Methods")
       .add_text("This guide explains the different types of method receivers in Rust.")
       .add_list(
           vec![
               "self - Takes ownership".to_string(),
               "&self - Borrows immutably".to_string(),
               "&mut self - Borrows mutably".to_string(),
           ],
           true
       )
       .set_metadata("category", "programming")
       .set_metadata("difficulty", "intermediate");
    
    // Add a code block
    doc.add_code(
        "impl Counter {\n    fn increment(&mut self) {\n        self.count += 1;\n    }\n}",
        "rust"
    );
    
    // Add an image
    doc.add_image(
        "https://example.com/rust-logo.png",
        "The Rust Programming Language Logo",
        200,
        100
    );
    
    // Using &self methods (immutable access)
    println!("Document: {} by {}", doc.get_title(), doc.get_author());
    println!("Word count: {}", doc.word_count());
    println!("Content blocks: {}", doc.content_length());
    println!("Preview: {}", doc.preview(30));
    println!("Contains 'Rust': {}", doc.contains_text("Rust"));
    println!("Category: {}", doc.get_metadata("category").unwrap_or(&"Uncategorized".to_string()));
    println!("Last updated: {} seconds ago", doc.time_since_update());
    
    // Create a document renderer (uses &self)
    let renderer = DocumentRenderer::new(&doc, RenderFormat::Markdown);
    let markdown = renderer.render();
    println!("\nRendered document (Markdown):\n{}", markdown);
    
    // HTML rendering
    let html_renderer = DocumentRenderer::new(&doc, RenderFormat::HTML);
    let html = html_renderer.render();
    println!("\nRendered document (HTML preview):\n{:.200}...", html);
    
    // Create another document for merging
    let mut appendix = Document::new("doc-002", "Appendix", "Bob Johnson");
    appendix.add_text("## Additional Resources")
           .add_text("Here are some helpful links for learning more about Rust methods:");
    
    appendix.add_list(
        vec![
            "The Rust Book".to_string(),
            "Rust by Example".to_string(),
            "Rust Reference".to_string(),
        ],
        false
    );
    
    // Using a consuming method (self)
    let merged_doc = doc.merge(appendix);
    println!("\nMerged document word count: {}", merged_doc.word_count());
    
    // Transforming the document (self)
    let archived = merged_doc.into_archive();
    println!("\nArchived document: {:?}", archived);
}
```

### Common Pitfalls
- **Pitfall**: Using `self` when `&self` or `&mut self` would suffice, unnecessarily transferring ownership.
  - **Solution**: Use the least restrictive receiver type that meets your needs; only use `self` when you need to transform or consume the instance.
- **Pitfall**: Using `&mut self` when the method only reads data.
  - **Solution**: Use `&self` for methods that don't modify the instance to allow more flexible usage.
- **Pitfall**: Forgetting that `self` methods consume the value, making it unavailable for further use.
  - **Solution**: Create a clone if you need to use the instance after calling a consuming method, or restructure code to avoid consuming until the end.
- **Pitfall**: Using chained methods with mixed receiver types.
  - **Solution**: Be aware that a chain of `&mut self` methods must end before a consuming method, and consuming methods must come last in a chain.
- **Pitfall**: Returning `self` instead of `&self` or `&mut self` in method chains.
  - **Solution**: Return `&Self` or `&mut Self` for chainable methods to avoid consuming the value.

### Confusion Questions with Answers
1. **Q**: Why does Rust have three different `self` parameter types instead of just one?
   **A**: The three receiver types give Rust fine-grained control over ownership and borrowing, allowing you to express intent clearly: `&self` for operations that only need to read data, `&mut self` for operations that modify data, and `self` for operations that consume the object. This explicit control helps prevent memory safety issues and enables the compiler to optimize code more effectively.

2. **Q**: When should I return `&mut self` from a method?
   **A**: Return `&mut self` when you want to enable method chaining for methods that modify the instance. This pattern, known as the "builder pattern" or "fluent interface," allows you to perform multiple operations on an object in a single expression, improving readability and reducing code verbosity.

3. **Q**: What's the difference between `Self` (capital S) and `self` (lowercase s) in Rust methods?
   **A**: `Self` (capital S) is a type alias referring to the type that the `impl` block is for, useful in return types and associated functions. `self` (lowercase s) is the name of the first parameter in a method, representing the instance the method is called on. `Self` is about the type itself, while `self` is about a specific instance of that type.

---

## 3. Associated Functions

### Concise Explanation
Associated functions are functions defined within an `impl` block but don't take `self` as a parameter. They're associated with a type rather than an instance of that type, similar to "static methods" in other languages.

Associated functions are called using the syntax `Type::function()` rather than `instance.method()`. They're commonly used for constructors (like `new()`) and utility functions related to a type but not requiring a specific instance.

The key difference between associated functions and methods is that associated functions don't have access to instance data (no `self` parameter), while methods do.

### Where to Use
- For constructors that create new instances (`new()`)
- For factory functions that create instances with specific configurations
- For utility functions related to a type that don't need instance access
- For conversions between related types
- For operations that relate to a type conceptually but don't require instance data
- For namespace organization, grouping related functions under a type

### Code Snippet
```rust
struct Point {
    x: f64,
    y: f64,
}

impl Point {
    // Associated function used as constructor
    fn new(x: f64, y: f64) -> Self {
        Point { x, y }
    }
    
    // Alternative constructors (factory functions)
    fn origin() -> Self {
        Point { x: 0.0, y: 0.0 }
    }
    
    fn from_tuple(coords: (f64, f64)) -> Self {
        Point { x: coords.0, y: coords.1 }
    }
    
    // Utility functions
    fn distance_between(a: &Point, b: &Point) -> f64 {
        ((b.x - a.x).powi(2) + (b.y - a.y).powi(2)).sqrt()
    }
    
    // Regular method for comparison
    fn distance_to(&self, other: &Point) -> f64 {
        Point::distance_between(self, other)
    }
    
    // Constants can be included in impl blocks too
    const DIMENSIONS: usize = 2;
}

// Generic type example
struct Vector<T> {
    elements: Vec<T>,
}

impl<T> Vector<T> {
    // Associated functions with generic types
    fn new() -> Self {
        Vector { elements: Vec::new() }
    }
    
    fn with_capacity(capacity: usize) -> Self {
        Vector { elements: Vec::with_capacity(capacity) }
    }
    
    fn from_slice(slice: &[T]) -> Self 
    where T: Clone {
        Vector { elements: slice.to_vec() }
    }
    
    // Regular methods
    fn push(&mut self, item: T) {
        self.elements.push(item);
    }
    
    fn len(&self) -> usize {
        self.elements.len()
    }
}

// Associated functions with trait bounds
impl<T: std::fmt::Debug> Vector<T> {
    fn debug_all(vectors: &[Vector<T>]) {
        for (i, vector) in vectors.iter().enumerate() {
            println!("Vector {}: {:?}", i, vector.elements);
        }
    }
}

// Enums can have associated functions too
enum Shape {
    Circle(f64),
    Rectangle(f64, f64),
    Triangle(f64, f64, f64),
}

impl Shape {
    // Associated functions for creating shapes
    fn circle(radius: f64) -> Self {
        if radius <= 0.0 {
            panic!("Radius must be positive");
        }
        Shape::Circle(radius)
    }
    
    fn rectangle(width: f64, height: f64) -> Self {
        if width <= 0.0 || height <= 0.0 {
            panic!("Dimensions must be positive");
        }
        Shape::Rectangle(width, height)
    }
    
    fn square(side: f64) -> Self {
        Shape::rectangle(side, side)
    }
    
    fn triangle(a: f64, b: f64, c: f64) -> Result<Self, String> {
        // Check triangle inequality theorem
        if a + b <= c || a + c <= b || b + c <= a {
            return Err("Invalid triangle dimensions".to_string());
        }
        Ok(Shape::Triangle(a, b, c))
    }
    
    // Regular methods
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(r) => std::f64::consts::PI * r * r,
            Shape::Rectangle(w, h) => w * h,
            Shape::Triangle(a, b, c) => {
                let s = (a + b + c) / 2.0;
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
        }
    }
}

fn main() {
    // Using constructors
    let p1 = Point::new(3.0, 4.0);
    let p2 = Point::origin();
    let p3 = Point::from_tuple((5.0, 6.0));
    
    // Using utility functions
    let distance = Point::distance_between(&p1, &p3);
    println!("Distance between points: {}", distance);
    
    // Using regular methods for comparison
    let distance2 = p1.distance_to(&p3);
    println!("Distance using method: {}", distance2);
    
    // Accessing associated constants
    println!("Point dimensions: {}", Point::DIMENSIONS);
    
    // Using generic associated functions
    let mut v1: Vector<i32> = Vector::new();
    let mut v2 = Vector::with_capacity(10);
    let v3 = Vector::from_slice(&[1, 2, 3, 4, 5]);
    
    v1.push(42);
    v2.push(100);
    
    println!("Vector lengths: {}, {}, {}", v1.len(), v2.len(), v3.len());
    
    // Using associated functions with trait bounds
    let vectors = [v1, v2, v3];
    Vector::debug_all(&vectors);
    
    // Using enum associated functions
    let circle = Shape::circle(5.0);
    let rect = Shape::rectangle(10.0, 5.0);
    let square = Shape::square(7.0);
    
    match Shape::triangle(3.0, 4.0, 5.0) {
        Ok(triangle) => {
            println!("Triangle area: {}", triangle.area());
        },
        Err(e) => {
            println!("Error creating triangle: {}", e);
        }
    }
    
    println!("Areas: Circle={}, Rectangle={}, Square={}",
             circle.area(), rect.area(), square.area());
}
```

### Real-World Example
A database connection manager with associated functions:

```rust
use std::collections::HashMap;
use std::fmt;
use std::sync::{Arc, Mutex};
use std::time::{Duration, Instant};

// Database connection configuration
#[derive(Debug, Clone)]
struct ConnectionConfig {
    host: String,
    port: u16,
    username: String,
    password: String,
    database: String,
    connection_timeout_ms: u64,
    query_timeout_ms: u64,
    max_connections: usize,
}

impl ConnectionConfig {
    // Associated function: standard constructor
    fn new(
        host: &str,
        port: u16,
        username: &str,
        password: &str,
        database: &str,
    ) -> Self {
        ConnectionConfig {
            host: host.to_string(),
            port,
            username: username.to_string(),
            password: password.to_string(),
            database: database.to_string(),
            connection_timeout_ms: 30000,  // 30 seconds default
            query_timeout_ms: 10000,      // 10 seconds default
            max_connections: 10,          // 10 connections default
        }
    }
    
    // Associated function: create from URL
    fn from_url(url: &str) -> Result<Self, String> {
        // Parse a URL like "postgres://username:password@host:port/database"
        if !url.contains("://") {
            return Err("Invalid URL format: missing scheme".to_string());
        }
        
        let parts: Vec<&str> = url.splitn(2, "://").collect();
        let scheme = parts[0];
        let rest = parts[1];
        
        // Check scheme
        match scheme {
            "postgres" | "mysql" | "sqlite" => {},
            _ => return Err(format!("Unsupported database type: {}", scheme)),
        }
        
        // Parse credentials and host
        let parts: Vec<&str> = rest.splitn(2, "@").collect();
        if parts.len() != 2 {
            return Err("Invalid URL: missing credentials or host".to_string());
        }
        
        let credentials = parts[0];
        let host_part = parts[1];
        
        // Parse username and password
        let cred_parts: Vec<&str> = credentials.splitn(2, ":").collect();
        if cred_parts.len() != 2 {
            return Err("Invalid credentials format".to_string());
        }
        
        let username = cred_parts[0];
        let password = cred_parts[1];
        
        // Parse host, port, and database
        let host_parts: Vec<&str> = host_part.splitn(2, "/").collect();
        if host_parts.len() != 2 {
            return Err("Invalid host format: missing database".to_string());
        }
        
        let host_port = host_parts[0];
        let database = host_parts[1];
        
        // Parse host and port
        let host_port_parts: Vec<&str> = host_port.splitn(2, ":").collect();
        let host = host_port_parts[0];
        let port = if host_port_parts.len() > 1 {
            host_port_parts[1].parse::<u16>().map_err(|_| "Invalid port".to_string())?
        } else {
            // Default ports based on scheme
            match scheme {
                "postgres" => 5432,
                "mysql" => 3306,
                "sqlite" => 0, // SQLite doesn't use ports
                _ => 0,
            }
        };
        
        // Create config with defaults
        Ok(ConnectionConfig {
            host: host.to_string(),
            port,
            username: username.to_string(),
            password: password.to_string(),
            database: database.to_string(),
            connection_timeout_ms: 30000,
            query_timeout_ms: 10000,
            max_connections: 10,
        })
    }
    
    // Associated function: create configuration for testing
    fn for_testing() -> Self {
        ConnectionConfig {
            host: "localhost".to_string(),
            port: 5432,
            username: "test_user".to_string(),
            password: "test_password".to_string(),
            database: "test_db".to_string(),
            connection_timeout_ms: 5000,   // Shorter timeout for tests
            query_timeout_ms: 3000,        // Shorter timeout for tests
            max_connections: 5,            // Fewer connections for tests
        }
    }
    
    // Associated function: create an in-memory configuration
    fn in_memory() -> Self {
        ConnectionConfig {
            host: ":memory:".to_string(),
            port: 0,
            username: "".to_string(),
            password: "".to_string(),
            database: "memory".to_string(),
            connection_timeout_ms: 1000,
            query_timeout_ms: 1000,
            max_connections: 1,
        }
    }
    
    // Regular methods
    fn get_connection_string(&self) -> String {
        format!(
            "{}:{}@{}:{}/{}",
            self.username, self.password, self.host, self.port, self.database
        )
    }
    
    fn with_timeout(mut self, connection_ms: u64, query_ms: u64) -> Self {
        self.connection_timeout_ms = connection_ms;
        self.query_timeout_ms = query_ms;
        self
    }
    
    fn with_max_connections(mut self, max: usize) -> Self {
        self.max_connections = max;
        self
    }
    
    fn validate(&self) -> Result<(), String> {
        if self.host.is_empty() {
            return Err("Host cannot be empty".to_string());
        }
        
        if self.host != ":memory:" && self.port == 0 {
            return Err("Port must be specified for non-memory databases".to_string());
        }
        
        if self.connection_timeout_ms == 0 {
            return Err("Connection timeout cannot be zero".to_string());
        }
        
        if self.query_timeout_ms == 0 {
            return Err("Query timeout cannot be zero".to_string());
        }
        
        if self.max_connections == 0 {
            return Err("Max connections cannot be zero".to_string());
        }
        
        Ok(())
    }
}

// A simulated database connection
struct Connection {
    id: usize,
    config: ConnectionConfig,
    created_at: Instant,
    last_used: Instant,
    is_busy: bool,
}

impl Connection {
    // Associated function: create a new connection
    fn new(id: usize, config: &ConnectionConfig) -> Self {
        println!("Creating connection #{} to {}", id, config.host);
        
        // In a real implementation, this would establish an actual database connection
        Connection {
            id,
            config: config.clone(),
            created_at: Instant::now(),
            last_used: Instant::now(),
            is_busy: false,
        }
    }
    
    // Regular methods
    fn execute(&mut self, query: &str) -> Result<(), String> {
        if self.is_busy {
            return Err("Connection is busy".to_string());
        }
        
        self.is_busy = true;
        self.last_used = Instant::now();
        
        println!(
            "Connection #{} executing query: {} on {}",
            self.id, query, self.config.database
        );
        
        // Simulate query execution
        std::thread::sleep(Duration::from_millis(100));
        
        self.is_busy = false;
        Ok(())
    }
    
    fn is_expired(&self, max_idle_time: Duration) -> bool {
        self.last_used.elapsed() > max_idle_time
    }
    
    fn age(&self) -> Duration {
        self.created_at.elapsed()
    }
}

impl fmt::Debug for Connection {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        f.debug_struct("Connection")
            .field("id", &self.id)
            .field("database", &self.config.database)
            .field("age", &format!("{:?}", self.age()))
            .field("last_used", &format!("{:?}", self.last_used.elapsed()))
            .field("is_busy", &self.is_busy)
            .finish()
    }
}

// Connection pool
struct ConnectionPool {
    config: ConnectionConfig,
    connections: Vec<Connection>,
    next_id: usize,
    max_idle_time: Duration,
}

impl ConnectionPool {
    // Associated function: create a new pool
    fn new(config: ConnectionConfig) -> Result<Self, String> {
        // Validate configuration
        config.validate()?;
        
        Ok(ConnectionPool {
            config,
            connections: Vec::new(),
            next_id: 1,
            max_idle_time: Duration::from_secs(300), // 5 minutes default
        })
    }
    
    // Associated function: create a pool with pre-filled connections
    fn with_connections(config: ConnectionConfig, count: usize) -> Result<Self, String> {
        let mut pool = Self::new(config)?;
        
        // Create initial connections
        for _ in 0..count {
            pool.create_connection();
        }
        
        Ok(pool)
    }
    
    // Regular methods
    fn create_connection(&mut self) -> &mut Connection {
        let id = self.next_id;
        self.next_id += 1;
        
        let connection = Connection::new(id, &self.config);
        self.connections.push(connection);
        
        self.connections.last_mut().unwrap()
    }
    
    fn get_connection(&mut self) -> Result<&mut Connection, String> {
        // First, try to find an idle connection
        for conn in &mut self.connections {
            if !conn.is_busy {
                conn.is_busy = true;
                conn.last_used = Instant::now();
                return Ok(conn);
            }
        }
        
        // If no idle connection and we're below max, create a new one
        if self.connections.len() < self.config.max_connections {
            let conn = self.create_connection();
            conn.is_busy = true;
            return Ok(conn);
        }
        
        // If we're at capacity, check for expired connections
        self.cleanup();
        
        // Try again after cleanup
        for conn in &mut self.connections {
            if !conn.is_busy {
                conn.is_busy = true;
                conn.last_used = Instant::now();
                return Ok(conn);
            }
        }
        
        Err("No available connections".to_string())
    }
    
    fn release_connection(&mut self, id: usize) -> Result<(), String> {
        for conn in &mut self.connections {
            if conn.id == id {
                if !conn.is_busy {
                    return Err(format!("Connection #{} is not marked as busy", id));
                }
                conn.is_busy = false;
                conn.last_used = Instant::now();
                return Ok(());
            }
        }
        
        Err(format!("Connection #{} not found", id))
    }
    
    fn cleanup(&mut self) {
        let max_idle_time = self.max_idle_time;
        
        // Remove expired connections
        self.connections.retain(|conn| {
            if conn.is_busy {
                true // Keep busy connections
            } else {
                !conn.is_expired(max_idle_time) // Keep non-expired idle connections
            }
        });
    }
    
    fn set_max_idle_time(&mut self, seconds: u64) {
        self.max_idle_time = Duration::from_secs(seconds);
    }
    
    fn execute_query(&mut self, query: &str) -> Result<(), String> {
        let conn = self.get_connection()?;
        let id = conn.id;
        
        match conn.execute(query) {
            Ok(()) => {
                self.release_connection(id)?;
                Ok(())
            },
            Err(e) => {
                self.release_connection(id)?;
                Err(e)
            }
        }
    }
    
    fn stats(&self) -> ConnectionPoolStats {
        let total = self.connections.len();
        let busy = self.connections.iter().filter(|c| c.is_busy).count();
        let idle = total - busy;
        
        ConnectionPoolStats {
            total,
            busy,
            idle,
            max_connections: self.config.max_connections,
        }
    }
}

struct ConnectionPoolStats {
    total: usize,
    busy: usize,
    idle: usize,
    max_connections: usize,
}

impl fmt::Display for ConnectionPoolStats {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(
            f, 
            "ConnectionPool: {}/{} connections ({} busy, {} idle)",
            self.total, self.max_connections, self.busy, self.idle
        )
    }
}

// Connection manager to handle multiple pools
struct ConnectionManager {
    pools: HashMap<String, Arc<Mutex<ConnectionPool>>>,
}

impl ConnectionManager {
    // Associated function: create a new manager
    fn new() -> Self {
        ConnectionManager {
            pools: HashMap::new(),
        }
    }
    
    // Methods
    fn add_pool(&mut self, name: &str, config: ConnectionConfig) -> Result<(), String> {
        if self.pools.contains_key(name) {
            return Err(format!("Pool '{}' already exists", name));
        }
        
        let pool = ConnectionPool::new(config)?;
        self.pools.insert(name.to_string(), Arc::new(Mutex::new(pool)));
        Ok(())
    }
    
    fn get_pool(&self, name: &str) -> Result<Arc<Mutex<ConnectionPool>>, String> {
        self.pools.get(name)
            .cloned()
            .ok_or_else(|| format!("Pool '{}' not found", name))
    }
    
    fn execute_query(&self, pool_name: &str, query: &str) -> Result<(), String> {
        let pool = self.get_pool(pool_name)?;
        let mut pool = pool.lock().unwrap();
        pool.execute_query(query)
    }
    
    fn remove_pool(&mut self, name: &str) -> Result<(), String> {
        if self.pools.remove(name).is_none() {
            return Err(format!("Pool '{}' not found", name));
        }
        Ok(())
    }
    
    fn list_pools(&self) -> Vec<String> {
        self.pools.keys().cloned().collect()
    }
    
    fn get_stats(&self) -> HashMap<String, String> {
        let mut stats = HashMap::new();
        
        for (name, pool) in &self.pools {
            if let Ok(pool) = pool.lock() {
                stats.insert(name.clone(), pool.stats().to_string());
            } else {
                stats.insert(name.clone(), "Failed to acquire lock".to_string());
            }
        }
        
        stats
    }
}

fn main() {
    // Using associated functions to create configurations
    let pg_config = ConnectionConfig::new(
        "localhost",
        5432,
        "postgres",
        "password123",
        "myapp"
    ).with_timeout(60000, 30000)
     .with_max_connections(20);
    
    // Using from_url associated function
    match ConnectionConfig::from_url("postgres://admin:secret@db.example.com:5432/production") {
        Ok(config) => println!("Created config from URL: {}", config.get_connection_string()),
        Err(e) => println!("Error parsing URL: {}", e),
    }
    
    // Using special factory functions
    let test_config = ConnectionConfig::for_testing();
    let memory_config = ConnectionConfig::in_memory();
    
    println!("Test config: {}", test_config.get_connection_string());
    println!("Memory config: {}", memory_config.get_connection_string());
    
    // Creating connection pools
    let mut manager = ConnectionManager::new();
    
    // Add pools using different configs
    if let Err(e) = manager.add_pool("main", pg_config) {
        println!("Error creating main pool: {}", e);
    }
    
    if let Err(e) = manager.add_pool("test", test_config) {
        println!("Error creating test pool: {}", e);
    }
    
    if let Err(e) = manager.add_pool("memory", memory_config) {
        println!("Error creating memory pool: {}", e);
    }
    
    // Execute some queries
    println!("\nExecuting queries:");
    
    for query in &[
        "SELECT * FROM users",
        "INSERT INTO orders (product_id, quantity) VALUES (42, 10)",
        "UPDATE inventory SET stock = stock - 10 WHERE product_id = 42",
    ] {
        match manager.execute_query("main", query) {
            Ok(_) => println!("Query executed successfully"),
            Err(e) => println!("Query error: {}", e),
        }
    }
    
    // Display stats
    println!("\nConnection pool stats:");
    for (name, stats) in manager.get_stats() {
        println!("{}: {}", name, stats);
    }
    
    // Remove a pool
    if let Err(e) = manager.remove_pool("test") {
        println!("Error removing pool: {}", e);
    }
    
    // List remaining pools
    println!("\nRemaining pools: {:?}", manager.list_pools());
}
```

### Common Pitfalls
- **Pitfall**: Confusion between associated functions and methods.
  - **Solution**: Remember that associated functions are called with `Type::function()` syntax and don't have access to instance data. Use associated functions for operations that don't require an instance.
- **Pitfall**: Creating too many specialized constructors.
  - **Solution**: Limit associated functions to common use cases; use builder patterns or configuration structs for complex initialization.
- **Pitfall**: Redundant utility functions that could live outside the type.
  - **Solution**: Only include associated functions that conceptually belong to the type and benefit from being in its namespace.
- **Pitfall**: Overly complex factory methods with too many parameters.
  - **Solution**: Consider using default arguments with builder pattern methods, or create specialized constructors for common configurations.
- **Pitfall**: Type name repetition in every function call.
  - **Solution**: Use type inference where possible, or create shorter aliases for verbose type names.

### Confusion Questions with Answers
1. **Q**: Why use an associated function instead of just a regular function outside the impl block?
   **A**: Associated functions provide namespace organization, making the function visibly related to its type and accessible through the type's namespace. They help group related functionality, improve discoverability, and avoid naming collisions. Additionally, associated functions have access to the type's private fields when creating new instances.

2. **Q**: What's the difference between `Self` (capital S) and the concrete type name in associated functions?
   **A**: `Self` is a type alias that refers to the type being implemented. Using `Self` instead of the concrete type name makes code more maintainable (only one place to change if the type is renamed) and clearer when working with generics or traits. It also reduces repetition, especially with long type names.

3. **Q**: Can associated functions access instance fields or methods?
   **A**: No, associated functions don't have a `self` parameter, so they cannot directly access instance fields or methods. However, they can access other associated functions, constants, and create or use instances of the type. If they need to work with instance data, they must be explicitly passed an instance as a parameter.

---

## 4. Builder Pattern

### Concise Explanation
The Builder pattern is a design pattern used to construct complex objects step by step, separating construction from representation. In Rust, it's commonly implemented using method chaining with `&mut self` or consuming `self` and returning a new instance.

The pattern is particularly useful when dealing with objects that have:
- Many optional parameters
- Complex validation rules
- Multiple configurations
- Dependencies between parameters

Instead of having constructors with many parameters or multiple factory methods, the Builder pattern provides a clean, fluent API for object construction.

### Where to Use
- For types with many optional parameters
- When constructing immutable objects
- When object creation involves multiple steps
- For constructions with complex validation rules
- When parameters have interdependencies
- For improving API ergonomics and readability

### Code Snippet
```rust
// Object to be built
#[derive(Debug)]
struct HttpRequest {
    method: HttpMethod,
    url: String,
    headers: Vec<(String, String)>,
    body: Option<String>,
    timeout_ms: u64,
    follow_redirects: bool,
    verify_ssl: bool,
    basic_auth: Option<(String, String)>,
}

#[derive(Debug, Clone)]
enum HttpMethod {
    GET,
    POST,
    PUT,
    DELETE,
    PATCH,
    HEAD,
    OPTIONS,
}

// Builder for the HttpRequest
#[derive(Default)]
struct HttpRequestBuilder {
    method: Option<HttpMethod>,
    url: Option<String>,
    headers: Vec<(String, String)>,
    body: Option<String>,
    timeout_ms: Option<u64>,
    follow_redirects: Option<bool>,
    verify_ssl: Option<bool>,
    basic_auth: Option<(String, String)>,
}

impl HttpRequestBuilder {
    // Create a new builder
    fn new() -> Self {
        HttpRequestBuilder::default()
    }
    
    // Builder methods for setting properties
    fn method(mut self, method: HttpMethod) -> Self {
        self.method = Some(method);
        self
    }
    
    fn url(mut self, url: impl Into<String>) -> Self {
        self.url = Some(url.into());
        self
    }
    
    fn header(mut self, name: impl Into<String>, value: impl Into<String>) -> Self {
        self.headers.push((name.into(), value.into()));
        self
    }
    
    fn body(mut self, body: impl Into<String>) -> Self {
        self.body = Some(body.into());
        self
    }
    
    fn timeout(mut self, timeout_ms: u64) -> Self {
        self.timeout_ms = Some(timeout_ms);
        self
    }
    
    fn follow_redirects(mut self, follow: bool) -> Self {
        self.follow_redirects = Some(follow);
        self
    }
    
    fn verify_ssl(mut self, verify: bool) -> Self {
        self.verify_ssl = Some(verify);
        self
    }
    
    fn basic_auth(mut self, username: impl Into<String>, password: impl Into<String>) -> Self {
        self.basic_auth = Some((username.into(), password.into()));
        self
    }
    
    // Method to build the final object
    fn build(self) -> Result<HttpRequest, String> {
        // Validate required fields
        let method = self.method.ok_or("HTTP method is required")?;
        let url = self.url.ok_or("URL is required")?;
        
        // Set defaults for optional fields
        let timeout_ms = self.timeout_ms.unwrap_or(30000); // 30 seconds default
        let follow_redirects = self.follow_redirects.unwrap_or(true);
        let verify_ssl = self.verify_ssl.unwrap_or(true);
        
        // Validate URL format
        if !url.starts_with("http://") && !url.starts_with("https://") {
            return Err("URL must start with http:// or https://".to_string());
        }
        
        // Build and return the HttpRequest
        Ok(HttpRequest {
            method,
            url,
            headers: self.headers,
            body: self.body,
            timeout_ms,
            follow_redirects,
            verify_ssl,
            basic_auth: self.basic_auth,
        })
    }
}

// Convenience methods on HttpMethod for common requests
impl HttpMethod {
    fn get() -> HttpMethod {
        HttpMethod::GET
    }
    
    fn post() -> HttpMethod {
        HttpMethod::POST
    }
    
    fn put() -> HttpMethod {
        HttpMethod::PUT
    }
    
    fn delete() -> HttpMethod {
        HttpMethod::DELETE
    }
}

// Implement the builder pattern directly on HttpRequest
impl HttpRequest {
    fn builder() -> HttpRequestBuilder {
        HttpRequestBuilder::new()
    }
    
    // Convenience methods for common request types
    fn get(url: impl Into<String>) -> HttpRequestBuilder {
        HttpRequestBuilder::new()
            .method(HttpMethod::GET)
            .url(url)
    }
    
    fn post(url: impl Into<String>) -> HttpRequestBuilder {
        HttpRequestBuilder::new()
            .method(HttpMethod::POST)
            .url(url)
    }
    
    // Execute the request (just a simulation for example)
    fn execute(&self) -> Result<HttpResponse, String> {
        println!("Executing {} request to {}", format!("{:?}", self.method), self.url);
        
        // In a real implementation, this would make the actual HTTP request
        // For this example, we'll simulate a response
        let status = 200;
        let response_body = format!("Response from {} to {}", format!("{:?}", self.method), self.url);
        
        Ok(HttpResponse {
            status,
            body: response_body,
            headers: vec![
                ("Content-Type".to_string(), "application/json".to_string()),
                ("Server".to_string(), "rust-example".to_string()),
            ],
        })
    }
}

// Simplified HTTP response for example purposes
#[derive(Debug)]
struct HttpResponse {
    status: u16,
    body: String,
    headers: Vec<(String, String)>,
}

// Advanced builder pattern example with typed validation
struct ServerConfig {
    host: String,
    port: u16,
    workers: usize,
    timeout: u64,
    log_level: LogLevel,
    features: Vec<String>,
}

enum LogLevel {
    Error,
    Warn,
    Info,
    Debug,
    Trace,
}

struct ServerConfigBuilder<H, P, W> {
    host: H,
    port: P,
    workers: W,
    timeout: Option<u64>,
    log_level: Option<LogLevel>,
    features: Vec<String>,
}

// Type states to track which fields have been set
struct Unset;
struct Set<T>(T);

// Initial builder with all required fields unset
impl ServerConfigBuilder<Unset, Unset, Unset> {
    fn new() -> Self {
        ServerConfigBuilder {
            host: Unset,
            port: Unset,
            workers: Unset,
            timeout: None,
            log_level: None,
            features: Vec::new(),
        }
    }
}

// Methods available for any state
impl<H, P, W> ServerConfigBuilder<H, P, W> {
    fn timeout(mut self, timeout: u64) -> Self {
        self.timeout = Some(timeout);
        self
    }
    
    fn log_level(mut self, level: LogLevel) -> Self {
        self.log_level = Some(level);
        self
    }
    
    fn add_feature(mut self, feature: impl Into<String>) -> Self {
        self.features.push(feature.into());
        self
    }
}

// Setting host
impl<P, W> ServerConfigBuilder<Unset, P, W> {
    fn host(self, host: impl Into<String>) -> ServerConfigBuilder<Set<String>, P, W> {
        ServerConfigBuilder {
            host: Set(host.into()),
            port: self.port,
            workers: self.workers,
            timeout: self.timeout,
            log_level: self.log_level,
            features: self.features,
        }
    }
}

// Setting port
impl<H, W> ServerConfigBuilder<H, Unset, W> {
    fn port(self, port: u16) -> ServerConfigBuilder<H, Set<u16>, W> {
        ServerConfigBuilder {
            host: self.host,
            port: Set(port),
            workers: self.workers,
            timeout: self.timeout,
            log_level: self.log_level,
            features: self.features,
        }
    }
}

// Setting workers
impl<H, P> ServerConfigBuilder<H, P, Unset> {
    fn workers(self, workers: usize) -> ServerConfigBuilder<H, P, Set<usize>> {
        ServerConfigBuilder {
            host: self.host,
            port: self.port,
            workers: Set(workers),
            timeout: self.timeout,
            log_level: self.log_level,
            features: self.features,
        }
    }
}

// Building the config, only allowed when all required fields are set
impl ServerConfigBuilder<Set<String>, Set<u16>, Set<usize>> {
    fn build(self) -> ServerConfig {
        ServerConfig {
            host: self.host.0,
            port: self.port.0,
            workers: self.workers.0,
            timeout: self.timeout.unwrap_or(30000),
            log_level: self.log_level.unwrap_or(LogLevel::Info),
            features: self.features,
        }
    }
}

fn main() {
    // Basic builder pattern example
    let request = HttpRequest::builder()
        .method(HttpMethod::GET)
        .url("https://api.example.com/users")
        .header("User-Agent", "Rust Example")
        .timeout(5000)
        .build();
    
    match request {
        Ok(req) => {
            println!("Created request: {:?}", req);
            
            // Execute the request
            match req.execute() {
                Ok(response) => println!("Response: {:?}", response),
                Err(e) => println!("Error executing request: {}", e),
            }
        },
        Err(e) => println!("Error building request: {}", e),
    }
    
    // Using convenience methods
    let post_request = HttpRequest::post("https://api.example.com/users")
        .header("Content-Type", "application/json")
        .body(r#"{"name": "John", "email": "john@example.com"}"#)
        .build();
    
    if let Ok(req) = post_request {
        println!("\nCreated POST request: {:?}", req);
    }
    
    // Example of error handling
    let invalid_request = HttpRequest::get("invalid-url")
        .build();
    
    if let Err(e) = invalid_request {
        println!("\nExpected error: {}", e);
    }
    
    // Advanced builder with compile-time validation
    let server_config = ServerConfigBuilder::new()
        .host("localhost")
        .port(8080)
        .workers(4)
        .timeout(60000)
        .add_feature("auth")
        .add_feature("logging")
        .build();
    
    println!("\nServer config created with {} workers and {} features", 
             server_config.workers, server_config.features.len());
    
    // This would not compile because required fields are missing:
    // let invalid_config = ServerConfigBuilder::new()
    //    .host("localhost")
    //    .timeout(1000)
    //    .build();
}
```

### Real-World Example
A configuration builder for a game engine:

```rust
use std::collections::HashMap;
use std::path::PathBuf;

// Game engine configuration
#[derive(Debug)]
struct GameEngineConfig {
    // Window settings
    window_title: String,
    window_width: u32,
    window_height: u32,
    fullscreen: bool,
    vsync: bool,
    
    // Graphics settings
    resolution_scale: f32,
    shadow_quality: ShadowQuality,
    texture_quality: TextureQuality,
    anti_aliasing: AntiAliasing,
    
    // System settings
    max_fps: Option<u32>,
    thread_count: usize,
    asset_path: PathBuf,
    save_path: PathBuf,
    
    // Audio settings
    master_volume: f32,
    music_volume: f32,
    sfx_volume: f32,
    surround_sound: bool,
    
    // Game settings
    difficulty: Difficulty,
    language: String,
    subtitles: bool,
    user_keybindings: HashMap<String, String>,
}

// Enums for configuration options
#[derive(Debug, Clone, Copy)]
enum ShadowQuality {
    Off,
    Low,
    Medium,
    High,
    Ultra,
}

#[derive(Debug, Clone, Copy)]
enum TextureQuality {
    Low,
    Medium,
    High,
    Ultra,
}

#[derive(Debug, Clone, Copy)]
enum AntiAliasing {
    None,
    FXAA,
    TAA,
    MSAA2x,
    MSAA4x,
    MSAA8x,
}

#[derive(Debug, Clone, Copy)]
enum Difficulty {
    Easy,
    Normal,
    Hard,
    Nightmare,
}

// The builder
#[derive(Default)]
struct GameEngineConfigBuilder {
    // Window settings
    window_title: Option<String>,
    window_width: Option<u32>,
    window_height: Option<u32>,
    fullscreen: Option<bool>,
    vsync: Option<bool>,
    
    // Graphics settings
    resolution_scale: Option<f32>,
    shadow_quality: Option<ShadowQuality>,
    texture_quality: Option<TextureQuality>,
    anti_aliasing: Option<AntiAliasing>,
    
    // System settings
    max_fps: Option<u32>,
    thread_count: Option<usize>,
    asset_path: Option<PathBuf>,
    save_path: Option<PathBuf>,
    
    // Audio settings
    master_volume: Option<f32>,
    music_volume: Option<f32>,
    sfx_volume: Option<f32>,
    surround_sound: Option<bool>,
    
    // Game settings
    difficulty: Option<Difficulty>,
    language: Option<String>,
    subtitles: Option<bool>,
    user_keybindings: HashMap<String, String>,
}

impl GameEngineConfigBuilder {
    // Create a new builder with default values
    fn new() -> Self {
        GameEngineConfigBuilder::default()
    }
    
    // Create a builder from an existing config
    fn from_config(config: &GameEngineConfig) -> Self {
        GameEngineConfigBuilder {
            window_title: Some(config.window_title.clone()),
            window_width: Some(config.window_width),
            window_height: Some(config.window_height),
            fullscreen: Some(config.fullscreen),
            vsync: Some(config.vsync),
            resolution_scale: Some(config.resolution_scale),
            shadow_quality: Some(config.shadow_quality),
            texture_quality: Some(config.texture_quality),
            anti_aliasing: Some(config.anti_aliasing),
            max_fps: config.max_fps,
            thread_count: Some(config.thread_count),
            asset_path: Some(config.asset_path.clone()),
            save_path: Some(config.save_path.clone()),
            master_volume: Some(config.master_volume),
            music_volume: Some(config.music_volume),
            sfx_volume: Some(config.sfx_volume),
            surround_sound: Some(config.surround_sound),
            difficulty: Some(config.difficulty),
            language: Some(config.language.clone()),
            subtitles: Some(config.subtitles),
            user_keybindings: config.user_keybindings.clone(),
        }
    }
    
    // Window settings
    fn window_title(mut self, title: impl Into<String>) -> Self {
        self.window_title = Some(title.into());
        self
    }
    
    fn window_size(mut self, width: u32, height: u32) -> Self {
        self.window_width = Some(width);
        self.window_height = Some(height);
        self
    }
    
    fn fullscreen(mut self, fullscreen: bool) -> Self {
        self.fullscreen = Some(fullscreen);
        self
    }
    
    fn vsync(mut self, vsync: bool) -> Self {
        self.vsync = Some(vsync);
        self
    }
    
    // Graphics settings
    fn resolution_scale(mut self, scale: f32) -> Self {
        self.resolution_scale = Some(scale.max(0.25).min(2.0)); // Clamp between 0.25 and 2.0
        self
    }
    
    fn shadow_quality(mut self, quality: ShadowQuality) -> Self {
        self.shadow_quality = Some(quality);
        self
    }
    
    fn texture_quality(mut self, quality: TextureQuality) -> Self {
        self.texture_quality = Some(quality);
        self
    }
    
    fn anti_aliasing(mut self, aa: AntiAliasing) -> Self {
        self.anti_aliasing = Some(aa);
        self
    }
    
    // System settings
    fn max_fps(mut self, fps: Option<u32>) -> Self {
        self.max_fps = fps;
        self
    }
    
    fn thread_count(mut self, count: usize) -> Self {
        self.thread_count = Some(count);
        self
    }
    
    fn asset_path(mut self, path: impl Into<PathBuf>) -> Self {
        self.asset_path = Some(path.into());
        self
    }
    
    fn save_path(mut self, path: impl Into<PathBuf>) -> Self {
        self.save_path = Some(path.into());
        self
    }
    
    // Audio settings
    fn master_volume(mut self, volume: f32) -> Self {
        self.master_volume = Some(volume.max(0.0).min(1.0)); // Clamp between 0.0 and 1.0
        self
    }
    
    fn music_volume(mut self, volume: f32) -> Self {
        self.music_volume = Some(volume.max(0.0).min(1.0)); // Clamp between 0.0 and 1.0
        self
    }
    
    fn sfx_volume(mut self, volume: f32) -> Self {
        self.sfx_volume = Some(volume.max(0.0).min(1.0)); // Clamp between 0.0 and 1.0
        self
    }
    
    fn surround_sound(mut self, enabled: bool) -> Self {
        self.surround_sound = Some(enabled);
        self
    }
    
    // Game settings
    fn difficulty(mut self, difficulty: Difficulty) -> Self {
        self.difficulty = Some(difficulty);
        self
    }
    
    fn language(mut self, language: impl Into<String>) -> Self {
        self.language = Some(language.into());
        self
    }
    
    fn subtitles(mut self, enabled: bool) -> Self {
        self.subtitles = Some(enabled);
        self
    }
    
    fn keybinding(mut self, action: impl Into<String>, key: impl Into<String>) -> Self {
        self.user_keybindings.insert(action.into(), key.into());
        self
    }
    
    // Preset configurations
    fn low_spec(mut self) -> Self {
        self.resolution_scale = Some(0.75);
        self.shadow_quality = Some(ShadowQuality::Low);
        self.texture_quality = Some(TextureQuality::Low);
        self.anti_aliasing = Some(AntiAliasing::None);
        self
    }
    
    fn mid_spec(mut self) -> Self {
        self.resolution_scale = Some(1.0);
        self.shadow_quality = Some(ShadowQuality::Medium);
        self.texture_quality = Some(TextureQuality::Medium);
        self.anti_aliasing = Some(AntiAliasing::FXAA);
        self
    }
    
    fn high_spec(mut self) -> Self {
        self.resolution_scale = Some(1.0);
        self.shadow_quality = Some(ShadowQuality::High);
        self.texture_quality = Some(TextureQuality::High);
        self.anti_aliasing = Some(AntiAliasing::TAA);
        self
    }
    
    fn ultra_spec(mut self) -> Self {
        self.resolution_scale = Some(1.5);
        self.shadow_quality = Some(ShadowQuality::Ultra);
        self.texture_quality = Some(TextureQuality::Ultra);
        self.anti_aliasing = Some(AntiAliasing::MSAA4x);
        self
    }
    
    // Build the configuration
    fn build(self) -> Result<GameEngineConfig, String> {
        // Validate the configuration
        let window_title = self.window_title.ok_or("Window title is required")?;
        let window_width = self.window_width.ok_or("Window width is required")?;
        let window_height = self.window_height.ok_or("Window height is required")?;
        
        if window_width < 640 || window_height < 480 {
            return Err("Window dimensions too small (minimum 640x480)".to_string());
        }
        
        let asset_path = self.asset_path.ok_or("Asset path is required")?;
        let save_path = self.save_path.ok_or("Save path is required")?;
        
        // Provide defaults for optional settings
        let fullscreen = self.fullscreen.unwrap_or(false);
        let vsync = self.vsync.unwrap_or(true);
        let resolution_scale = self.resolution_scale.unwrap_or(1.0);
        let shadow_quality = self.shadow_quality.unwrap_or(ShadowQuality::Medium);
        let texture_quality = self.texture_quality.unwrap_or(TextureQuality::Medium);
        let anti_aliasing = self.anti_aliasing.unwrap_or(AntiAliasing::FXAA);
        let thread_count = self.thread_count.unwrap_or_else(|| {
            std::thread::available_parallelism().map_or(2, |p| p.get())
        });
        let master_volume = self.master_volume.unwrap_or(0.7);
        let music_volume = self.music_volume.unwrap_or(0.5);
        let sfx_volume = self.sfx_volume.unwrap_or(0.7);
        let surround_sound = self.surround_sound.unwrap_or(false);
        let difficulty = self.difficulty.unwrap_or(Difficulty::Normal);
        let language = self.language.unwrap_or_else(|| "en".to_string());
        let subtitles = self.subtitles.unwrap_or(true);
        
        // Build and return the configuration
        Ok(GameEngineConfig {
            window_title,
            window_width,
            window_height,
            fullscreen,
            vsync,
            resolution_scale,
            shadow_quality,
            texture_quality,
            anti_aliasing,
            max_fps: self.max_fps,
            thread_count,
            asset_path,
            save_path,
            master_volume,
            music_volume,
            sfx_volume,
            surround_sound,
            difficulty,
            language,
            subtitles,
            user_keybindings: self.user_keybindings,
        })
    }
}

// Factory for GameEngineConfig
impl GameEngineConfig {
    fn builder() -> GameEngineConfigBuilder {
        GameEngineConfigBuilder::new()
    }
    
    // Methods to create common configurations
    fn for_development() -> GameEngineConfig {
        GameEngineConfigBuilder::new()
            .window_title("Game Engine - Development Build")
            .window_size(1280, 720)
            .fullscreen(false)
            .vsync(false)
            .max_fps(None) // Unlimited FPS
            .asset_path("./assets")
            .save_path("./save")
            .mid_spec()
            .build()
            .expect("Failed to create development config")
    }
    
    fn for_testing() -> GameEngineConfig {
        GameEngineConfigBuilder::new()
            .window_title("Game Engine - Test Build")
            .window_size(800, 600)
            .fullscreen(false)
            .vsync(false)
            .max_fps(Some(60))
            .asset_path("./test_assets")
            .save_path("./test_save")
            .low_spec()
            .build()
            .expect("Failed to create test config")
    }
    
    // Method to update an existing configuration
    fn update(&self, updates: impl FnOnce(GameEngineConfigBuilder) -> GameEngineConfigBuilder) -> Result<Self, String> {
        let builder = GameEngineConfigBuilder::from_config(self);
        updates(builder).build()
    }
    
    // Method to save configuration to a file (simplified)
    fn save_to_file(&self, path: &str) -> Result<(), String> {
        println!("Saving configuration to {}", path);
        // In a real implementation, serialize and save the config
        Ok(())
    }
    
    // Method to load configuration from a file (simplified)
    fn load_from_file(path: &str) -> Result<Self, String> {
        println!("Loading configuration from {}", path);
        // In a real implementation, deserialize from the file
        // For this example, return a default config
        Ok(Self::for_development())
    }
}

// Game class that uses the configuration
struct Game {
    config: GameEngineConfig,
    is_running: bool,
}

impl Game {
    fn new(config: GameEngineConfig) -> Self {
        Game {
            config,
            is_running: false,
        }
    }
    
    fn init(&mut self) {
        println!("Initializing game with config:");
        println!("  Window: {}x{}, '{}'", 
                 self.config.window_width, 
                 self.config.window_height,
                 self.config.window_title);
        println!("  Graphics: {:?} shadows, {:?} textures, {:?} AA, {}x scale", 
                 self.config.shadow_quality,
                 self.config.texture_quality,
                 self.config.anti_aliasing,
                 self.config.resolution_scale);
        println!("  System: {} threads, FPS cap: {:?}", 
                 self.config.thread_count,
                 self.config.max_fps);
        
        self.is_running = true;
    }
    
    fn update_config(&mut self, new_config: GameEngineConfig) {
        println!("Updating game configuration");
        self.config = new_config;
        
        // In a real game, we might need to apply changes to the engine
        println!("Applied new configuration");
    }
    
    fn run(&mut self) {
        if !self.is_running {
            self.init();
        }
        
        println!("Game is running...");
        // Game loop would go here
    }
}

fn main() {
    // Create a config using the builder pattern with method chaining
    let config_result = GameEngineConfig::builder()
        .window_title("Awesome Game")
        .window_size(1920, 1080)
        .fullscreen(true)
        .vsync(true)
        .high_spec()
        .asset_path("./assets")
        .save_path("./save")
        .difficulty(Difficulty::Hard)
        .language("en-US")
        .keybinding("jump", "SPACE")
        .keybinding("move_forward", "W")
        .keybinding("move_backward", "S")
        .build();
    
    match config_result {
        Ok(config) => {
            // Create and run the game
            let mut game = Game::new(config);
            game.run();
            
            // Update the configuration
            match game.config.update(|b| {
                b.fullscreen(false)
                 .vsync(false)
                 .max_fps(Some(144))
            }) {
                Ok(new_config) => {
                    game.update_config(new_config);
                },
                Err(e) => println!("Failed to update config: {}", e),
            }
        },
        Err(e) => println!("Configuration error: {}", e),
    }
    
    // Use preset configurations
    let dev_config = GameEngineConfig::for_development();
    let test_config = GameEngineConfig::for_testing();
    
    println!("\nDevelopment config: {}x{}, {}x scale, {:?} shadows", 
             dev_config.window_width, 
             dev_config.window_height,
             dev_config.resolution_scale,
             dev_config.shadow_quality);
    
    println!("Test config: {}x{}, {}x scale, {:?} shadows", 
             test_config.window_width, 
             test_config.window_height,
             test_config.resolution_scale,
             test_config.shadow_quality);
    
    // Save and load configuration
    if let Err(e) = dev_config.save_to_file("config.json") {
        println!("Failed to save config: {}", e);
    }
    
    match GameEngineConfig::load_from_file("config.json") {
        Ok(loaded_config) => {
            println!("\nLoaded configuration successfully");
            let mut game = Game::new(loaded_config);
            game.run();
        },
        Err(e) => println!("Failed to load config: {}", e),
    }
}
```

### Common Pitfalls
- **Pitfall**: Forgetting validation in the `build` method.
  - **Solution**: Always validate required fields and parameter interdependencies before creating the target object.
- **Pitfall**: Creating a mutable builder without method chaining.
  - **Solution**: Return `self` from builder methods to enable method chaining for a fluent interface.
- **Pitfall**: Excessive copying of large fields during builder construction.
  - **Solution**: Use references where appropriate or implement efficient Clone for large data.
- **Pitfall**: Redundant error handling in each builder method.
  - **Solution**: Defer validation until the final `build` method to avoid premature errors.
- **Pitfall**: Not providing sensible defaults in the `build` method.
  - **Solution**: Set reasonable defaults for optional parameters that weren't specified by the caller.

### Confusion Questions with Answers
1. **Q**: When should I use the builder pattern versus multiple constructors?
   **A**: Use the builder pattern when:
   - You have many optional parameters (more than 3-4)
   - Parameters are dependent on each other
   - Construction involves complex validation
   - You want a fluent, readable API
   - The object is immutable after creation
   
   Multiple constructors may be simpler when you have few parameters or only a few common configurations.

2. **Q**: Why return `self` in builder methods instead of `&mut self`?
   **A**: Returning `self` (consuming the builder) prevents issues with lifetimes and borrowing when chaining methods. It's simpler and more flexible than returning `&mut self`, especially when building with temporary values. However, returning `&mut self` can be appropriate when you need to reuse the builder for creating multiple related objects.

3. **Q**: How do I implement the builder pattern for types with lifetimes or generic parameters?
   **A**: For types with lifetimes or generics, the builder must include the same lifetime or generic parameters. You'll need to:
   1. Declare the same type/lifetime parameters on the builder struct
   2. Propagate them through the builder methods
   3. Apply them correctly when constructing the final object
   
   If dealing with references and complex lifetimes, consider using owned types in the builder and converting to references in the final `build` method.

---

## Next Actions: Exercises and Projects

### Exercises
1. **Method Implementation Practice**
   - Add methods to existing structs to transform their data
   - Create fluent interfaces for data processing
   - Implement both consuming and non-consuming methods for the same operations
   - Update a library to use method chaining

2. **Self Parameter Challenge**
   - Create a type with methods using all three `self` types
   - Implement a function that accepts any kind of `self` using generics
   - Convert functions to methods with appropriate receiver types
   - Design an API that uses `&self` efficiently

3. **Associated Functions Workshop**
   - Design factory functions for complex object creation
   - Implement utility functions as associated functions
   - Create specialized constructors for common use cases
   - Convert free functions to associated functions

4. **Builder Pattern Application**
   - Implement the builder pattern for a configuration system
   - Create a builder with validation and interdependent parameters
   - Add type-state techniques for compile-time validation
   - Refactor existing code to use the builder pattern

### Micro-Project: Command-Line Parser
Build a command-line argument parser that:
- Uses methods to define and parse arguments
- Implements the builder pattern for configuration
- Uses associated functions for factory methods
- Provides a clean, fluent API for defining commands
- Handles validation and type conversion

```rust
// Example starter code for command-line parser micro-project
struct Arg {
    name: String,
    short: Option<char>,
    long: Option<String>,
    help: String,
    required: bool,
    takes_value: bool,
    default_value: Option<String>,
    possible_values: Vec<String>,
}

struct Command {
    name: String,
    about: String,
    args: Vec<Arg>,
    subcommands: Vec<Command>,
}

struct ArgBuilder {
    name: String,
    short: Option<char>,
    long: Option<String>,
    help: String,
    required: bool,
    takes_value: bool,
    default_value: Option<String>,
    possible_values: Vec<String>,
}

impl ArgBuilder {
    fn new(name: impl Into<String>) -> Self {
        ArgBuilder {
            name: name.into(),
            short: None,
            long: None,
            help: String::new(),
            required: false,
            takes_value: false,
            default_value: None,
            possible_values: Vec::new(),
        }
    }
    
    fn short(mut self, short: char) -> Self {
        self.short = Some(short);
        self
    }
    
    fn long(mut self, long: impl Into<String>) -> Self {
        self.long = Some(long.into());
        self
    }
    
    fn help(mut self, help: impl Into<String>) -> Self {
        self.help = help.into();
        self
    }
    
    fn required(mut self, required: bool) -> Self {
        self.required = required;
        self
    }
    
    fn takes_value(mut self, takes_value: bool) -> Self {
        self.takes_value = takes_value;
        self
    }
    
    fn default_value(mut self, value: impl Into<String>) -> Self {
        self.default_value = Some(value.into());
        self
    }
    
    fn possible_value(mut self, value: impl Into<String>) -> Self {
        self.possible_values.push(value.into());
        self
    }
    
    fn build(self) -> Arg {
        Arg {
            name: self.name,
            short: self.short,
            long: self.long,
            help: self.help,
            required: self.required,
            takes_value: self.takes_value,
            default_value: self.default_value,
            possible_values: self.possible_values,
        }
    }
}

// Complete the implementation with Command builder, parsing logic, etc.
```

## Success Criteria
You've mastered methods in Rust when you can:
1. Choose the appropriate receiver type (`self`, `&self`, or `&mut self`) based on your needs
2. Implement associated functions for construction and utility operations
3. Create clear, readable APIs using method chaining
4. Design fluent interfaces that hide implementation complexity
5. Implement the builder pattern for complex object construction
6. Apply idiomatic Rust patterns to method implementations
7. Organize related functionality into cohesive method sets

## Troubleshooting Advice
1. **Method Receiver Issues**
   - If the compiler complains about moved values, you might need `&self` instead of `self`
   - If you can't modify a value, you might need `&mut self` instead of `&self`
   - If lifetime errors occur, consider if you really need references or could use owned values

2. **Method vs. Associated Function Confusion**
   - Remember that methods take `self` and can access instance data
   - Associated functions don't have `self` and can't access instance fields directly
   - If you need to work with instances, use a method; if working with types, use an associated function

3. **Builder Pattern Problems**
   - Ensure all required fields are validated before construction
   - Return `Result` from `build()` for runtime validation errors
   - Use type-state patterns for compile-time validation
   - Set sensible defaults to simplify API usage

4. **Method Chaining Issues**
   - Verify you're returning `self` or `&mut self` from methods to enable chaining
   - Check for incorrect self types in chainable methods
   - Make sure ownership is handled correctly in the chain
   - For option-returning methods, consider using combinators to maintain the chain

5. **API Design Considerations**
   - Balance between fluency and simplicity
   - Avoid too many methods that do similar things
   - Provide good documentation for complex method behaviors
   - Consider the end user experience when designing method signatures
