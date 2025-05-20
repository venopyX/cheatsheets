# Module 4: Composite Types - Part 1: Arrays and Vectors

## Core Problem and Key Assumptions
- **Problem**: Understanding how to work with fixed-size arrays and dynamic vectors in Rust
- **Key Assumptions**:
  - You understand basic Rust syntax and primitive types
  - You need to store and manipulate collections of similar data elements
  - You want to leverage Rust's memory safety features while working with collections
  - You aim to write efficient code that properly manages memory

## Essential Concepts to Cover
1. Array Declaration and Initialization
2. Multidimensional Arrays
3. Vector Basics: Creation, Indexing, Appending
4. Vector Capacity and Growth
5. Iteration and Manipulation
6. Common Vector Operations

---

## 1. Array Declaration and Initialization

### Concise Explanation
Arrays in Rust are fixed-size collections of elements of the same type, stored in contiguous memory. The size of an array is part of its type and must be known at compile time. Arrays are denoted as `[T; N]` where `T` is the type of elements and `N` is the length (number of elements).

Key characteristics:
- Fixed size determined at compile time
- Elements are stored on the stack (unless the array itself is stored in a heap-allocated structure)
- Zero-indexed (first element is at index 0)
- Length is part of the array's type
- All elements must have the same type
- Arrays are passed by value (copying all elements) unless referenced

### Where to Use
- When you need a fixed number of elements of the same type
- For small to medium-sized collections that don't need to grow or shrink
- When performance is critical and you want to avoid heap allocations
- For data that logically forms a complete unit (e.g., RGB values, coordinates)
- In embedded systems or performance-critical code

### Code Snippet
```rust
fn main() {
    // Basic array declaration with type annotation
    let numbers: [i32; 5] = [1, 2, 3, 4, 5];
    
    // Type inference (compiler determines the type)
    let colors = ["red", "green", "blue"];
    
    // Initialization with default values
    let zeros = [0; 10]; // Creates [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    
    // Initialization with a specific value
    let ones: [i32; 4] = [1; 4]; // Creates [1, 1, 1, 1]
    
    // Arrays of custom types
    #[derive(Debug, Copy, Clone)]
    struct Point {
        x: f32,
        y: f32,
    }
    
    let origin = Point { x: 0.0, y: 0.0 };
    let points = [origin; 3]; // Creates an array with 3 copies of origin
    
    // Accessing array elements
    let first = numbers[0]; // First element
    let last = numbers[numbers.len() - 1]; // Last element
    
    // Getting array length
    let length = numbers.len();
    println!("Array length: {}", length);
    
    // Arrays have a fixed size
    // numbers.push(6); // Error: no method named `push` found for array
    
    // Creating a slice (reference to a section of an array)
    let slice = &numbers[1..4]; // References elements 1, 2, and 3
    println!("Slice: {:?}", slice);
    
    // Arrays implement Debug if their elements do
    println!("Numbers: {:?}", numbers);
    println!("Points: {:?}", points);
    
    // Arrays can be compared if their elements can be compared
    let array1 = [1, 2, 3];
    let array2 = [1, 2, 3];
    println!("Arrays are equal: {}", array1 == array2); // true
    
    // Initializing arrays with calculated values (using an iterator)
    let squares: [i32; 5] = std::array::from_fn(|i| (i as i32 + 1).pow(2));
    println!("Squares: {:?}", squares); // [1, 4, 9, 16, 25]
    
    // Destructuring arrays
    let [r, g, b] = colors;
    println!("Red: {}, Green: {}, Blue: {}", r, g, b);
}
```

### Real-World Example
A temperature logging system:

```rust
#[derive(Debug, Copy, Clone)]
struct TemperatureReading {
    celsius: f32,
    timestamp: u64,
}

impl TemperatureReading {
    fn new(celsius: f32, timestamp: u64) -> Self {
        TemperatureReading { celsius, timestamp }
    }
    
    fn to_fahrenheit(&self) -> f32 {
        (self.celsius * 9.0 / 5.0) + 32.0
    }
}

fn main() {
    // Initialize an array to hold hourly temperature readings for a day
    let mut daily_readings: [TemperatureReading; 24] = [
        TemperatureReading::new(0.0, 0); 24
    ];
    
    // Record some temperature readings (simulated data)
    daily_readings[0] = TemperatureReading::new(12.5, 1652313600); // 12 AM
    daily_readings[6] = TemperatureReading::new(10.2, 1652335200); // 6 AM
    daily_readings[12] = TemperatureReading::new(22.8, 1652356800); // 12 PM
    daily_readings[18] = TemperatureReading::new(18.5, 1652378400); // 6 PM
    
    // Calculate the average temperature for the recorded readings
    let mut sum = 0.0;
    let mut count = 0;
    
    for reading in daily_readings {
        if reading.timestamp > 0 { // Only consider initialized readings
            sum += reading.celsius;
            count += 1;
        }
    }
    
    let average = if count > 0 { sum / count as f32 } else { 0.0 };
    println!("Average temperature: {:.1}°C", average);
    
    // Find the highest and lowest temperatures
    let mut highest = TemperatureReading::new(f32::MIN, 0);
    let mut lowest = TemperatureReading::new(f32::MAX, 0);
    
    for reading in daily_readings {
        if reading.timestamp > 0 {
            if reading.celsius > highest.celsius {
                highest = reading;
            }
            if reading.celsius < lowest.celsius {
                lowest = reading;
            }
        }
    }
    
    println!("Highest temperature: {:.1}°C ({:.1}°F) at hour {}", 
             highest.celsius, 
             highest.to_fahrenheit(), 
             (highest.timestamp - 1652313600) / 3600);
    
    println!("Lowest temperature: {:.1}°C ({:.1}°F) at hour {}", 
             lowest.celsius, 
             lowest.to_fahrenheit(), 
             (lowest.timestamp - 1652313600) / 3600);
    
    // Use array methods
    // Find readings within a specific range
    let afternoon_readings = &daily_readings[12..18];
    println!("Afternoon readings: {:?}", afternoon_readings);
    
    // Using an array as a buffer
    let mut buffer = [0u8; 1024];
    // In a real application, this might be filled with data from a file or network
    buffer[0] = 72; // ASCII 'H'
    buffer[1] = 101; // ASCII 'e'
    buffer[2] = 108; // ASCII 'l'
    buffer[3] = 108; // ASCII 'l'
    buffer[4] = 111; // ASCII 'o'
    
    // Convert first 5 bytes to a string
    let message = std::str::from_utf8(&buffer[..5]).unwrap();
    println!("Message from buffer: {}", message);
}
```

### Common Pitfalls
- **Pitfall**: Trying to change the size of an array after it's been created.
  - **Solution**: Use a `Vec<T>` (vector) instead if you need a dynamically sized collection.
- **Pitfall**: Accessing array elements with an index that might be out of bounds.
  - **Solution**: Use `get` method which returns an `Option`, or combine with bounds checking.
- **Pitfall**: Attempting to create large arrays on the stack, causing stack overflow.
  - **Solution**: Use heap-allocated collections like `Vec` for large datasets.
- **Pitfall**: Forgetting that arrays are Copy types (if their elements are Copy).
  - **Solution**: Be aware that passing an array to a function will copy the entire array unless you pass a reference.
- **Pitfall**: Difficulty initializing arrays with non-Copy types.
  - **Solution**: Use `std::array::from_fn` or initialize with default values and then modify.

### Confusion Questions with Answers
1. **Q**: What's the difference between arrays and slices in Rust?
   **A**: Arrays (`[T; N]`) have a fixed size known at compile time and own their data. Slices (`&[T]`) are views into arrays (or other slices) with a dynamic size known at runtime. Slices don't own their data—they borrow it.

2. **Q**: How can I create an array with values that aren't known until runtime?
   **A**: You can't create a true array with a size determined at runtime. If you need a dynamically-sized collection, use `Vec<T>`. If you know the maximum size but not the actual size, you can create an array with that maximum size and keep track of how much is actually used.

3. **Q**: Why can't I append elements to an array?
   **A**: Arrays in Rust have a fixed size that's part of their type. Their size can't change after creation. If you need to add elements, you should use a `Vec<T>` (vector), which can grow as needed.

---

## 2. Multidimensional Arrays

### Concise Explanation
Multidimensional arrays in Rust are arrays of arrays. Since arrays are first-class values, you can nest them to create multi-dimensional structures. A two-dimensional array is denoted as `[[T; N]; M]` where `T` is the element type, `N` is the inner array length, and `M` is the outer array length.

Key points:
- A multidimensional array is really an array of arrays
- All inner arrays must have the same size (uniform dimensions)
- Accessing elements requires multiple indices
- Memory layout is contiguous and row-major (inner arrays are stored next to each other)
- Useful for mathematical matrices, grids, tables, or image data

### Where to Use
- For grid-based data structures like game boards, maps, or matrices
- When working with tabular data with a fixed number of rows and columns
- For image processing (pixels in a 2D or 3D space)
- In numerical computing with fixed-size matrices
- When you need efficient cache locality for nested structured data

### Code Snippet
```rust
fn main() {
    // 2D array (3 rows, 4 columns) with explicit type
    let grid: [[i32; 4]; 3] = [
        [1, 2, 3, 4],    // Row 0
        [5, 6, 7, 8],    // Row 1
        [9, 10, 11, 12], // Row 2
    ];
    
    // 2D array with type inference
    let matrix = [
        [1.0, 2.0, 3.0],
        [4.0, 5.0, 6.0],
    ];
    
    // Initialize a 3x3 grid with zeros
    let zeros = [[0; 3]; 3];
    
    // Initialize a multiplication table (10x10)
    let mut mult_table = [[0; 10]; 10];
    for i in 0..10 {
        for j in 0..10 {
            mult_table[i][j] = (i+1) * (j+1);
        }
    }
    
    // Accessing elements in a 2D array
    let element = grid[1][2]; // Row 1, Column 2 (value: 7)
    println!("Element at [1][2]: {}", element);
    
    // Getting dimensions of a 2D array
    let rows = grid.len();
    let columns = grid[0].len();
    println!("Grid dimensions: {} rows x {} columns", rows, columns);
    
    // Iterating over all elements in a 2D array
    println!("All grid elements:");
    for row in &grid {
        for &element in row {
            print!("{:4}", element);
        }
        println!();
    }
    
    // Iterating with indices
    println!("\nMultiplication table:");
    for i in 0..mult_table.len() {
        for j in 0..mult_table[i].len() {
            print!("{:4}", mult_table[i][j]);
        }
        println!();
    }
    
    // Creating a slice of a row
    let row_slice = &matrix[1][..]; // Slice of the second row
    println!("\nSecond row: {:?}", row_slice);
    
    // 3D array (2x3x4)
    let cube: [[[i32; 4]; 3]; 2] = [
        // First 2D array (index 0)
        [
            [1, 2, 3, 4],
            [5, 6, 7, 8],
            [9, 10, 11, 12],
        ],
        // Second 2D array (index 1)
        [
            [13, 14, 15, 16],
            [17, 18, 19, 20],
            [21, 22, 23, 24],
        ],
    ];
    
    // Accessing an element in a 3D array
    let value = cube[1][2][3]; // Value at [1][2][3]: 24
    println!("\nValue at [1][2][3]: {}", value);
    
    // Flattening a 2D array into a 1D array
    let flat: Vec<i32> = grid.iter().flatten().cloned().collect();
    println!("\nFlattened grid: {:?}", flat);
}
```

### Real-World Example
A tic-tac-toe game using a 2D array:

```rust
#[derive(Debug, Copy, Clone, PartialEq)]
enum Cell {
    Empty,
    X,
    O,
}

impl Cell {
    fn to_char(&self) -> char {
        match self {
            Cell::Empty => ' ',
            Cell::X => 'X',
            Cell::O => 'O',
        }
    }
}

struct TicTacToe {
    board: [[Cell; 3]; 3],
    current_player: Cell,
    moves: u8,
}

impl TicTacToe {
    fn new() -> Self {
        TicTacToe {
            board: [[Cell::Empty; 3]; 3],
            current_player: Cell::X, // X goes first
            moves: 0,
        }
    }
    
    fn print_board(&self) {
        println!("\n  0 1 2");
        for (i, row) in self.board.iter().enumerate() {
            print!("{} ", i);
            for cell in row {
                print!("{} ", cell.to_char());
            }
            println!();
        }
        println!();
    }
    
    fn make_move(&mut self, row: usize, col: usize) -> Result<(), &'static str> {
        if row >= 3 || col >= 3 {
            return Err("Position out of bounds");
        }
        
        if self.board[row][col] != Cell::Empty {
            return Err("Cell already occupied");
        }
        
        self.board[row][col] = self.current_player;
        self.moves += 1;
        
        // Switch player
        self.current_player = match self.current_player {
            Cell::X => Cell::O,
            Cell::O => Cell::X,
            Cell::Empty => unreachable!(),
        };
        
        Ok(())
    }
    
    fn check_winner(&self) -> Option<Cell> {
        // Check rows
        for row in &self.board {
            if row[0] != Cell::Empty && row[0] == row[1] && row[1] == row[2] {
                return Some(row[0]);
            }
        }
        
        // Check columns
        for col in 0..3 {
            if self.board[0][col] != Cell::Empty &&
               self.board[0][col] == self.board[1][col] &&
               self.board[1][col] == self.board[2][col] {
                return Some(self.board[0][col]);
            }
        }
        
        // Check diagonals
        if self.board[0][0] != Cell::Empty &&
           self.board[0][0] == self.board[1][1] &&
           self.board[1][1] == self.board[2][2] {
            return Some(self.board[0][0]);
        }
        
        if self.board[0][2] != Cell::Empty &&
           self.board[0][2] == self.board[1][1] &&
           self.board[1][1] == self.board[2][0] {
            return Some(self.board[0][2]);
        }
        
        None
    }
    
    fn is_full(&self) -> bool {
        self.moves == 9
    }
    
    fn game_over(&self) -> bool {
        self.check_winner().is_some() || self.is_full()
    }
}

fn main() {
    use std::io::{self, Write};

    let mut game = TicTacToe::new();
    
    println!("Welcome to Tic-Tac-Toe!");
    println!("Enter moves as 'row col' (e.g., '1 2')");
    
    while !game.game_over() {
        game.print_board();
        
        let player_char = match game.current_player {
            Cell::X => 'X',
            Cell::O => 'O',
            Cell::Empty => unreachable!(),
        };
        
        println!("Player {}'s turn.", player_char);
        
        // Get player move
        print!("Enter row and column: ");
        io::stdout().flush().unwrap();
        
        let mut input = String::new();
        io::stdin().read_line(&mut input).unwrap();
        
        let coordinates: Vec<usize> = input
            .trim()
            .split_whitespace()
            .map(|s| s.parse::<usize>())
            .filter_map(Result::ok)
            .collect();
        
        if coordinates.len() != 2 {
            println!("Invalid input. Please enter row and column numbers.");
            continue;
        }
        
        let (row, col) = (coordinates[0], coordinates[1]);
        
        match game.make_move(row, col) {
            Ok(_) => {}, // Valid move
            Err(msg) => {
                println!("Error: {}. Try again.", msg);
                continue;
            }
        }
    }
    
    // Game is over
    game.print_board();
    
    match game.check_winner() {
        Some(Cell::X) => println!("Player X wins!"),
        Some(Cell::O) => println!("Player O wins!"),
        _ => println!("It's a draw!"),
    }
}
```

### Common Pitfalls
- **Pitfall**: Confusing the order of indices in the array type definition.
  - **Solution**: Remember that Rust uses row-major order: `[[T; columns]; rows]`.
- **Pitfall**: Forgetting that all inner arrays must have the same size.
  - **Solution**: If you need jagged arrays (rows of different lengths), use a vector of vectors: `Vec<Vec<T>>`.
- **Pitfall**: Inefficient iteration patterns over multidimensional arrays.
  - **Solution**: Use cache-friendly iteration patterns (row by row) for better performance.
- **Pitfall**: Stack overflow when creating large multidimensional arrays.
  - **Solution**: Use `Vec<Vec<T>>` for large or dynamically sized 2D collections.
- **Pitfall**: Accidentally copying large arrays when passing to functions.
  - **Solution**: Pass references to arrays (`&[[T; N]; M]`) to avoid unnecessary copying.

### Confusion Questions with Answers
1. **Q**: How do I create a 2D array with dimensions known only at runtime?
   **A**: You can't create a true multidimensional array with dimensions known only at runtime. Use `Vec<Vec<T>>` instead, which allows for dynamically sized dimensions, though it may be less cache-efficient than a true 2D array.

2. **Q**: Is a 2D array in Rust stored contiguously in memory?
   **A**: Yes, Rust's multidimensional arrays are stored contiguously in row-major order. This is good for cache locality when iterating row by row, but be aware that iterating column by column will be less cache-efficient.

3. **Q**: How can I implement a matrix with non-uniform dimensions?
   **A**: For non-uniform dimensions (jagged arrays), use `Vec<Vec<T>>`. Each inner vector can have a different length. However, this representation is less memory-efficient than true rectangular arrays because each inner vector has its own separate allocation.

---

## 3. Vector Basics: Creation, Indexing, Appending

### Concise Explanation
Vectors (`Vec<T>`) are dynamic, resizable arrays that store elements of the same type. Unlike arrays, vectors are allocated on the heap and can grow or shrink at runtime. A vector keeps track of three properties: a pointer to the data on the heap, its length (number of elements), and its capacity (amount of memory reserved).

Key points:
- Vectors automatically grow as needed when elements are added
- Elements are stored contiguously in memory for efficient access
- Zero-indexed like arrays (first element is at index 0)
- Provides bounds checking when accessing elements
- Much more flexible than arrays, but with slight runtime overhead
- Heap-allocated, enabling dynamic sizing and larger collections

### Where to Use
- When you need a collection that can grow or shrink
- For storing an arbitrary number of elements
- When collection size is not known at compile time
- For building a collection by pushing elements one by one
- When you need flexibility and convenient methods for manipulation

### Code Snippet
```rust
fn main() {
    // Creating an empty vector with type annotation
    let mut numbers: Vec<i32> = Vec::new();
    
    // Creating a vector with initial values using vec! macro
    let mut colors = vec!["red", "green", "blue"];
    
    // Creating a vector with a specific capacity
    let mut with_capacity = Vec::with_capacity(10);
    
    // Creating a vector with repeated values
    let zeros = vec![0; 5]; // Creates [0, 0, 0, 0, 0]
    
    // Adding elements
    numbers.push(1);
    numbers.push(2);
    numbers.push(3);
    
    // Removing the last element
    let last = numbers.pop(); // Returns Some(3)
    println!("Popped value: {:?}", last);
    
    // Accessing elements with indexing (panics if out of bounds)
    let second = colors[1]; // "green"
    println!("Second color: {}", second);
    
    // Accessing elements safely with get (returns Option)
    match colors.get(5) {
        Some(value) => println!("Found: {}", value),
        None => println!("Index out of bounds"),
    }
    
    // Updating values
    if !colors.is_empty() {
        colors[0] = "crimson";
    }
    
    // Checking length
    println!("Number of colors: {}", colors.len());
    
    // Checking capacity
    println!("Vector capacity: {}", colors.capacity());
    
    // Check if vector is empty
    println!("Is the vector empty? {}", colors.is_empty());
    
    // Clear a vector (remove all elements)
    numbers.clear();
    println!("After clear, length: {}", numbers.len());
    
    // Inserting at specific positions
    let mut letters = vec!['a', 'c', 'd'];
    letters.insert(1, 'b'); // Insert 'b' at index 1
    println!("Letters: {:?}", letters); // ['a', 'b', 'c', 'd']
    
    // Removing from specific positions
    letters.remove(2); // Remove element at index 2 ('c')
    println!("After remove: {:?}", letters); // ['a', 'b', 'd']
    
    // Extending a vector with another collection
    let more_letters = vec!['e', 'f'];
    letters.extend(more_letters);
    println!("Extended: {:?}", letters); // ['a', 'b', 'd', 'e', 'f']
    
    // Creating a vector from an iterator
    let squares: Vec<i32> = (1..=5).map(|x| x * x).collect();
    println!("Squares: {:?}", squares); // [1, 4, 9, 16, 25]
    
    // Slicing a vector
    let slice = &squares[1..4]; // [4, 9, 16]
    println!("Slice: {:?}", slice);
    
    // Converting a vector to/from an array (if size is known)
    let array: [i32; 5] = [1, 2, 3, 4, 5];
    let vec_from_array: Vec<i32> = array.to_vec();
    
    // Converting between arrays and vectors with slices
    let small_vec = vec![1, 2, 3];
    let array_from_vec: [i32; 3] = small_vec.as_slice().try_into().unwrap();
    
    // Draining elements (remove and iterate over them)
    let mut nums = vec![1, 2, 3, 4, 5, 6];
    let drained: Vec<i32> = nums.drain(1..4).collect();
    println!("Drained: {:?}", drained); // [2, 3, 4]
    println!("Remaining: {:?}", nums);  // [1, 5, 6]
    
    // Vectors can hold any type, including other vectors
    let nested = vec![vec![1, 2], vec![3, 4, 5]];
    println!("Nested vector: {:?}", nested);
}
```

### Real-World Example
A simple task management system:

```rust
#[derive(Debug, Clone)]
struct Task {
    id: usize,
    description: String,
    completed: bool,
}

impl Task {
    fn new(id: usize, description: &str) -> Self {
        Task {
            id,
            description: description.to_string(),
            completed: false,
        }
    }
    
    fn complete(&mut self) {
        self.completed = true;
    }
    
    fn display(&self) -> String {
        let status = if self.completed { "✓" } else { " " };
        format!("[{}] {}: {}", status, self.id, self.description)
    }
}

struct TaskManager {
    tasks: Vec<Task>,
    next_id: usize,
}

impl TaskManager {
    fn new() -> Self {
        TaskManager {
            tasks: Vec::new(),
            next_id: 1,
        }
    }
    
    fn add_task(&mut self, description: &str) -> usize {
        let id = self.next_id;
        self.next_id += 1;
        
        let task = Task::new(id, description);
        self.tasks.push(task);
        id
    }
    
    fn complete_task(&mut self, id: usize) -> bool {
        for task in &mut self.tasks {
            if task.id == id {
                task.complete();
                return true;
            }
        }
        false
    }
    
    fn remove_task(&mut self, id: usize) -> bool {
        let initial_len = self.tasks.len();
        self.tasks.retain(|task| task.id != id);
        self.tasks.len() < initial_len
    }
    
    fn list_all(&self) {
        println!("Tasks:");
        if self.tasks.is_empty() {
            println!("  No tasks found.");
            return;
        }
        
        for task in &self.tasks {
            println!("  {}", task.display());
        }
    }
    
    fn list_completed(&self) {
        let completed: Vec<&Task> = self.tasks
            .iter()
            .filter(|task| task.completed)
            .collect();
        
        println!("Completed tasks:");
        if completed.is_empty() {
            println!("  No completed tasks.");
            return;
        }
        
        for task in completed {
            println!("  {}", task.display());
        }
    }
    
    fn list_pending(&self) {
        let pending: Vec<&Task> = self.tasks
            .iter()
            .filter(|task| !task.completed)
            .collect();
        
        println!("Pending tasks:");
        if pending.is_empty() {
            println!("  No pending tasks.");
            return;
        }
        
        for task in pending {
            println!("  {}", task.display());
        }
    }
    
    fn find_by_keywords(&self, keywords: &[&str]) -> Vec<&Task> {
        self.tasks
            .iter()
            .filter(|task| {
                keywords.iter().any(|&keyword| 
                    task.description.to_lowercase().contains(&keyword.to_lowercase())
                )
            })
            .collect()
    }
    
    fn save_to_file(&self, filename: &str) -> std::io::Result<()> {
        use std::fs::File;
        use std::io::Write;
        
        let mut file = File::create(filename)?;
        
        for task in &self.tasks {
            writeln!(
                file, 
                "{},{},{}",
                task.id,
                task.completed,
                task.description
            )?;
        }
        
        Ok(())
    }
    
    fn load_from_file(&mut self, filename: &str) -> std::io::Result<()> {
        use std::fs::File;
        use std::io::{BufRead, BufReader};
        
        let file = File::open(filename)?;
        let reader = BufReader::new(file);
        
        self.tasks.clear();
        self.next_id = 1;
        
        for line in reader.lines() {
            let line = line?;
            let parts: Vec<&str> = line.split(',').collect();
            
            if parts.len() >= 3 {
                if let Ok(id) = parts[0].parse::<usize>() {
                    let completed = parts[1] == "true";
                    let description = parts[2..].join(",");
                    
                    let task = Task {
                        id,
                        description,
                        completed,
                    };
                    
                    self.tasks.push(task);
                    if id >= self.next_id {
                        self.next_id = id + 1;
                    }
                }
            }
        }
        
        Ok(())
    }
}

fn main() {
    let mut manager = TaskManager::new();
    
    // Add some tasks
    manager.add_task("Buy groceries");
    manager.add_task("Finish Rust assignment");
    manager.add_task("Call dentist");
    manager.add_task("Schedule meeting with team");
    
    // Display all tasks
    manager.list_all();
    
    // Complete some tasks
    manager.complete_task(2);
    manager.complete_task(3);
    
    println!("\nAfter completing tasks 2 and 3:");
    manager.list_all();
    
    // Show only completed tasks
    println!();
    manager.list_completed();
    
    // Show only pending tasks
    println!();
    manager.list_pending();
    
    // Find tasks by keywords
    println!("\nTasks containing 'meeting':");
    let results = manager.find_by_keywords(&["meeting"]);
    for task in results {
        println!("  {}", task.display());
    }
    
    // Remove a task
    manager.remove_task(1);
    println!("\nAfter removing task 1:");
    manager.list_all();
    
    // Save tasks to file
    if let Err(e) = manager.save_to_file("tasks.csv") {
        println!("Error saving tasks: {}", e);
    } else {
        println!("\nTasks saved successfully.");
    }
    
    // Create a new manager and load tasks
    let mut new_manager = TaskManager::new();
    if let Err(e) = new_manager.load_from_file("tasks.csv") {
        println!("Error loading tasks: {}", e);
    } else {
        println!("\nTasks loaded successfully:");
        new_manager.list_all();
    }
}
```

### Common Pitfalls
- **Pitfall**: Forgetting that accessing elements by index (`vec[i]`) can panic if out of bounds.
  - **Solution**: Use `get` or `get_mut` methods which return `Option` types, or check bounds before indexing.
- **Pitfall**: Inefficient growing of vectors with frequent `push` operations.
  - **Solution**: Preallocate with `Vec::with_capacity` when you know the approximate size.
- **Pitfall**: Creating multiple mutable borrows of the same vector elements.
  - **Solution**: Use patterns like `split_at_mut` or indices to avoid simultaneous mutable borrows.
- **Pitfall**: Frequent resizing of vectors causing performance issues.
  - **Solution**: Reserve capacity upfront or in larger chunks, and consider specialized data structures for specific use cases.
- **Pitfall**: Using `remove` for multiple elements which causes O(n²) complexity.
  - **Solution**: Use `retain`, `drain`, or `swap_remove` for more efficient removal of multiple elements.

### Confusion Questions with Answers
1. **Q**: What's the difference between `Vec::new()` and `Vec::with_capacity()`?
   **A**: `Vec::new()` creates an empty vector with no allocated memory, while `Vec::with_capacity(n)` preallocates memory for `n` elements. Both create empty vectors, but using `with_capacity` is more efficient when you know you'll be adding a specific number of elements, as it avoids multiple reallocations.

2. **Q**: When should I use `pop` versus `remove`?
   **A**: Use `pop()` when you want to remove and retrieve the last element (O(1) operation). Use `remove(index)` when you need to remove an element at a specific position, but be aware it's O(n) because all elements after the removed one must be shifted.

3. **Q**: What happens if I access a vector element beyond its length?
   **A**: If you use direct indexing (`vec[i]`), your program will panic at runtime. If you use the `get` method (`vec.get(i)`), it returns an `Option<&T>` which will be `None` if the index is out of bounds, allowing you to handle the case safely.

---

## 4. Vector Capacity and Growth

### Concise Explanation
Vector capacity is the amount of memory allocated to store elements, which may be greater than the current length (number of elements). When adding elements, if the length would exceed capacity, the vector automatically reallocates with more capacity. Understanding how capacity works is essential for optimizing performance.

Key points:
- Capacity is the amount of memory reserved for future growth
- Vectors grow geometrically when needed (typically by doubling capacity)
- You can preallocate capacity to avoid frequent reallocations
- Capacity never shrinks automatically, even when elements are removed
- You can manually shrink capacity using `shrink_to_fit()`
- Efficient capacity management can significantly improve performance

### Where to Use
- When performance is critical, especially with large vectors
- In tight loops where elements are frequently added
- When memory usage needs to be optimized
- For collections with a known or estimated maximum size
- When implementing custom data structures with vectors

### Code Snippet
```rust
fn main() {
    // Creating vectors with different capacities
    let mut v1 = Vec::new();
    let mut v2 = Vec::with_capacity(10);
    
    println!("v1 - Length: {}, Capacity: {}", v1.len(), v1.capacity());
    println!("v2 - Length: {}, Capacity: {}", v2.len(), v2.capacity());
    
    // Adding elements to observe capacity growth
    println!("\nAdding elements to v1:");
    for i in 0..10 {
        v1.push(i);
        println!("After pushing {} - Length: {}, Capacity: {}", 
                 i, v1.len(), v1.capacity());
    }
    
    // Adding elements to a vector with preallocated capacity
    println!("\nAdding elements to v2:");
    for i in 0..10 {
        v2.push(i);
        println!("After pushing {} - Length: {}, Capacity: {}", 
                 i, v2.len(), v2.capacity());
    }
    
    // Demonstrating capacity growth beyond preallocated size
    println!("\nPushing beyond preallocated capacity:");
    for i in 10..15 {
        v2.push(i);
        println!("After pushing {} - Length: {}, Capacity: {}", 
                 i, v2.len(), v2.capacity());
    }
    
    // Capacity remains the same after removing elements
    println!("\nRemoving elements:");
    for _ in 0..5 {
        v2.pop();
        println!("After pop - Length: {}, Capacity: {}", 
                 v2.len(), v2.capacity());
    }
    
    // Manually reserving more capacity
    println!("\nReserving more capacity:");
    v1.reserve(10); // Reserve 10 more slots
    println!("v1 - Length: {}, Capacity: {}", v1.len(), v1.capacity());
    
    // Reserving exact additional capacity
    v2.reserve_exact(5); // Reserve exactly 5 more slots
    println!("v2 - Length: {}, Capacity: {}", v2.len(), v2.capacity());
    
    // Shrinking capacity to fit current length
    println!("\nShrinking to fit:");
    v1.shrink_to_fit();
    println!("v1 after shrink_to_fit - Length: {}, Capacity: {}", 
             v1.len(), v1.capacity());
    
    // Clearing a vector (removes all elements but keeps capacity)
    v1.clear();
    println!("\nv1 after clear - Length: {}, Capacity: {}", 
             v1.len(), v1.capacity());
    
    // Shrinking an empty vector to minimum capacity
    v1.shrink_to_fit();
    println!("v1 after shrink_to_fit - Length: {}, Capacity: {}", 
             v1.len(), v1.capacity());
    
    // Capacity optimization using Vec::with_capacity + extend
    let data = [1, 2, 3, 4, 5];
    let mut efficient = Vec::with_capacity(data.len());
    efficient.extend_from_slice(&data);
    println!("\nEfficient vector - Length: {}, Capacity: {}", 
             efficient.len(), efficient.capacity());
    
    // Demonstrating capacity management with strings
    let mut names = Vec::with_capacity(3);
    names.push("Alice".to_string());
    names.push("Bob".to_string());
    names.push("Charlie".to_string());
    
    println!("\nNames vector - Length: {}, Capacity: {}", 
             names.len(), names.capacity());
    
    // The truncate method reduces length but not capacity
    names.truncate(1);
    println!("After truncate - Length: {}, Capacity: {}", 
             names.len(), names.capacity());
}
```

### Real-World Example
A string buffer implementation with optimized capacity management:

```rust
struct StringBuffer {
    buffer: Vec<String>,
    current_size: usize,
    max_size: usize,
}

impl StringBuffer {
    // Create a new StringBuffer with initial capacity
    fn new(initial_capacity: usize, max_size: usize) -> Self {
        StringBuffer {
            buffer: Vec::with_capacity(initial_capacity),
            current_size: 0,
            max_size,
        }
    }
    
    // Add a string to the buffer, managing capacity efficiently
    fn append(&mut self, s: &str) -> Result<(), &'static str> {
        let new_len = self.current_size + s.len();
        
        if new_len > self.max_size {
            return Err("Buffer would exceed maximum size");
        }
        
        // Check if we need to flush before adding this string
        if self.buffer.is_empty() || s.len() > 1024 {
            // For large strings, store them directly
            self.buffer.push(s.to_string());
        } else {
            // For smaller strings, try to append to the last string to reduce allocations
            if let Some(last) = self.buffer.last_mut() {
                if last.len() + s.len() <= 1024 {
                    last.push_str(s);
                } else {
                    self.buffer.push(s.to_string());
                }
            } else {
                self.buffer.push(s.to_string());
            }
        }
        
        self.current_size = new_len;
        Ok(())
    }
    
    // Optimize the buffer by concatenating small strings
    fn optimize(&mut self) {
        if self.buffer.len() <= 1 {
            return; // Nothing to optimize
        }
        
        // Collect small strings into chunks
        let mut new_buffer = Vec::new();
        let mut current_chunk = String::new();
        
        for s in std::mem::take(&mut self.buffer) {
            if current_chunk.len() + s.len() <= 1024 {
                current_chunk.push_str(&s);
            } else {
                if !current_chunk.is_empty() {
                    new_buffer.push(current_chunk);
                    current_chunk = String::new();
                }
                
                if s.len() > 1024 {
                    // Keep large strings separate
                    new_buffer.push(s);
                } else {
                    current_chunk = s;
                }
            }
        }
        
        // Don't forget the last chunk
        if !current_chunk.is_empty() {
            new_buffer.push(current_chunk);
        }
        
        // Replace the buffer with our optimized version
        self.buffer = new_buffer;
    }
    
    // Get all content as a single string
    fn to_string(&self) -> String {
        self.buffer.join("")
    }
    
    // Clear the buffer
    fn clear(&mut self) {
        self.buffer.clear();
        self.current_size = 0;
        
        // Keep a minimal capacity
        if self.buffer.capacity() > 1024 {
            self.buffer.shrink_to_fit();
        }
    }
    
    // Get buffer statistics
    fn stats(&self) -> (usize, usize, usize, usize) {
        (
            self.current_size,                // Total content size
            self.buffer.len(),                // Number of chunks
            self.buffer.capacity(),           // Current capacity
            self.buffer.capacity() * 24 + self.current_size  // Approximate memory usage
        )
    }
}

fn main() {
    // Create a string buffer
    let mut buffer = StringBuffer::new(5, 1024 * 1024); // 1MB max size
    
    // Append some strings
    buffer.append("Hello, ").unwrap();
    buffer.append("world! ").unwrap();
    
    // Add a large string
    let large_string = "A".repeat(2000);
    buffer.append(&large_string).unwrap();
    
    // Add more small strings
    for i in 1..100 {
        buffer.append(&format!("Line {}: Some text here.\n", i)).unwrap();
    }
    
    // Print statistics before optimization
    let (size, chunks, capacity, memory) = buffer.stats();
    println!("Before optimization:");
    println!("  Size: {} bytes", size);
    println!("  Chunks: {}", chunks);
    println!("  Capacity: {} chunks", capacity);
    println!("  Approximate memory: {} bytes", memory);
    
    // Optimize the buffer
    buffer.optimize();
    
    // Print statistics after optimization
    let (size, chunks, capacity, memory) = buffer.stats();
    println!("\nAfter optimization:");
    println!("  Size: {} bytes", size);
    println!("  Chunks: {}", chunks);
    println!("  Capacity: {} chunks", capacity);
    println!("  Approximate memory: {} bytes", memory);
    
    // Get the full content
    let content = buffer.to_string();
    println!("\nContent length: {} bytes", content.len());
    println!("First 100 characters: {}", &content[..100.min(content.len())]);
    
    // Clear the buffer
    buffer.clear();
    println!("\nAfter clearing:");
    let (size, chunks, capacity, memory) = buffer.stats();
    println!("  Size: {} bytes", size);
    println!("  Chunks: {}", chunks);
    println!("  Capacity: {} chunks", capacity);
    println!("  Approximate memory: {} bytes", memory);
}
```

### Common Pitfalls
- **Pitfall**: Frequent reallocation due to small, incremental capacity increases.
  - **Solution**: Use `Vec::with_capacity` when you have an estimate of the final size, or use `reserve` to grow capacity in larger chunks.
- **Pitfall**: Wasting memory by not shrinking vectors after removing many elements.
  - **Solution**: Call `shrink_to_fit` after significant downsizing, but be aware this causes reallocation.
- **Pitfall**: Using `reserve_exact` when `reserve` would be more efficient.
  - **Solution**: Prefer `reserve` for most cases, as it may allocate extra space for future growth. Use `reserve_exact` only when precise memory control is needed.
- **Pitfall**: Not accounting for capacity growth behavior in memory-sensitive applications.
  - **Solution**: Be aware that vectors typically double in capacity when they grow, which can lead to more memory usage than strictly necessary.
- **Pitfall**: Constantly shrinking and growing vectors in a loop.
  - **Solution**: Resize once before the loop or use `clear()` to keep capacity but remove elements.

### Confusion Questions with Answers
1. **Q**: Why doesn't vector capacity decrease automatically when I remove elements?
   **A**: This is an optimization to avoid frequent reallocations. If elements are removed but might be added back later, keeping the existing capacity avoids the cost of reallocating memory. You can manually reduce capacity with `shrink_to_fit()`.

2. **Q**: How much extra capacity does a vector allocate when it grows?
   **A**: The exact growth factor depends on the implementation, but in Rust, vectors typically double their capacity when they need to grow. This amortizes the cost of reallocation over many push operations.

3. **Q**: Is there a performance difference between `Vec::new()` and `vec![]`?
   **A**: In terms of runtime performance, they're equivalent. `vec![]` is a macro that expands to code similar to `Vec::new()`. However, if you're initializing with values, `vec![1, 2, 3]` will create a vector with appropriate capacity and elements, whereas `Vec::new()` followed by pushes would need to grow.

---

## 5. Iteration and Manipulation

### Concise Explanation
Iterating over vectors (or other collections) is a fundamental operation in Rust. The language provides powerful tools for iteration, including various types of iterators, methods for transforming data during iteration, and ergonomic syntax for common operations. Understanding these patterns enables efficient and expressive manipulation of collections.

Key points:
- Iterators can be immutable, mutable, or consuming
- Many methods exist for transforming and filtering items during iteration
- Iterator methods are often more idiomatic than traditional loops
- Rust's iterator design enables zero-cost abstractions with performance comparable to hand-written loops
- Iterator chaining allows for concise, composable operations

### Where to Use
- For processing each element in a collection
- When performing data transformations
- For filtering elements based on conditions
- When aggregating data into a single result
- For composing multiple operations on a collection
- As an alternative to explicit indexing and loops

### Code Snippet
```rust
fn main() {
    // Create a vector to iterate over
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // Basic iteration (immutable)
    println!("Basic iteration:");
    for num in &numbers {
        print!("{} ", num);
    }
    println!();
    
    // Mutable iteration
    let mut mutable_numbers = numbers.clone();
    for num in &mut mutable_numbers {
        *num *= 2; // Double each number
    }
    println!("After doubling: {:?}", mutable_numbers);
    
    // Iterator methods
    
    // map: Transform each element
    let squared: Vec<i32> = numbers.iter().map(|x| x * x).collect();
    println!("Squared: {:?}", squared);
    
    // filter: Keep elements that satisfy a predicate
    let evens: Vec<&i32> = numbers.iter().filter(|&&x| x % 2 == 0).collect();
    println!("Even numbers: {:?}", evens);
    
    // find: Find first element matching a condition
    if let Some(&num) = numbers.iter().find(|&&x| x > 5) {
        println!("First number > 5: {}", num);
    }
    
    // any: Check if any element satisfies a condition
    let has_even = numbers.iter().any(|&x| x % 2 == 0);
    println!("Has even numbers: {}", has_even);
    
    // all: Check if all elements satisfy a condition
    let all_positive = numbers.iter().all(|&x| x > 0);
    println!("All positive: {}", all_positive);
    
    // fold: Accumulate a result (like reduce in other languages)
    let sum = numbers.iter().fold(0, |acc, &x| acc + x);
    println!("Sum using fold: {}", sum);
    
    // Chaining multiple operations
    let sum_of_even_squares: i32 = numbers.iter()
        .filter(|&&x| x % 2 == 0)  // Keep only even numbers
        .map(|&x| x * x)           // Square each one
        .sum();                    // Sum them up
    println!("Sum of even squares: {}", sum_of_even_squares);
    
    // Consuming the vector with into_iter
    let strings: Vec<String> = vec!["hello".to_string(), "world".to_string()];
    
    // This consumes the vector, transferring ownership of each element
    for s in strings {
        println!("String: {}", s);
    }
    // strings is no longer usable here
    
    // enumerate: Get indices along with elements
    for (i, num) in numbers.iter().enumerate() {
        println!("numbers[{}] = {}", i, num);
    }
    
    // zip: Combine multiple iterators
    let a = [1, 2, 3];
    let b = [4, 5, 6];
    
    let pairs: Vec<(i32, i32)> = a.iter()
        .zip(b.iter())
        .map(|(&x, &y)| (x, y))
        .collect();
    println!("Zipped pairs: {:?}", pairs);
    
    // take and skip
    let first_three: Vec<&i32> = numbers.iter().take(3).collect();
    let skipped_two: Vec<&i32> = numbers.iter().skip(2).collect();
    println!("First three: {:?}", first_three);
    println!("After skipping two: {:?}", skipped_two);
    
    // Mutable operations with iterators
    let mut vec = vec![1, 2, 3, 4, 5];
    
    // Using iter_mut for in-place modifications
    vec.iter_mut().for_each(|x| *x *= 3);
    println!("After multiplying by 3: {:?}", vec);
    
    // partition: Split a collection based on a condition
    let (even, odd): (Vec<i32>, Vec<i32>) = 
        numbers.into_iter().partition(|&x| x % 2 == 0);
    println!("Evens: {:?}, Odds: {:?}", even, odd);
}
```

### Real-World Example
Data analysis on a collection of products:

```rust
#[derive(Debug, Clone)]
struct Product {
    id: u32,
    name: String,
    price: f64,
    category: String,
    stock: u32,
    ratings: Vec<u8>, // Ratings from 1-5
}

impl Product {
    fn new(id: u32, name: &str, price: f64, category: &str, stock: u32) -> Self {
        Product {
            id,
            name: name.to_string(),
            price,
            category: category.to_string(),
            stock,
            ratings: Vec::new(),
        }
    }
    
    fn add_rating(&mut self, rating: u8) {
        if rating >= 1 && rating <= 5 {
            self.ratings.push(rating);
        }
    }
    
    fn average_rating(&self) -> Option<f64> {
        if self.ratings.is_empty() {
            None
        } else {
            let sum: u32 = self.ratings.iter().map(|&r| r as u32).sum();
            Some(sum as f64 / self.ratings.len() as f64)
        }
    }
}

struct Inventory {
    products: Vec<Product>,
}

impl Inventory {
    fn new() -> Self {
        Inventory {
            products: Vec::new(),
        }
    }
    
    fn add_product(&mut self, product: Product) {
        self.products.push(product);
    }
    
    // Find products by category
    fn find_by_category(&self, category: &str) -> Vec<&Product> {
        self.products
            .iter()
            .filter(|p| p.category == category)
            .collect()
    }
    
    // List all product categories
    fn list_categories(&self) -> Vec<String> {
        self.products
            .iter()
            .map(|p| p.category.clone())
            .collect::<std::collections::HashSet<_>>() // Remove duplicates
            .into_iter()
            .collect()
    }
    
    // Get products that need restocking
    fn need_restocking(&self, threshold: u32) -> Vec<&Product> {
        self.products
            .iter()
            .filter(|p| p.stock < threshold)
            .collect()
    }
    
    // Calculate total inventory value
    fn total_value(&self) -> f64 {
        self.products
            .iter()
            .map(|p| p.price * p.stock as f64)
            .sum()
    }
    
    // Get average price by category
    fn average_price_by_category(&self) -> std::collections::HashMap<String, f64> {
        let mut result = std::collections::HashMap::new();
        let mut counts = std::collections::HashMap::new();
        
        // Sum up prices by category
        for product in &self.products {
            let sum = result.entry(product.category.clone()).or_insert(0.0);
            *sum += product.price;
            
            let count = counts.entry(product.category.clone()).or_insert(0);
            *count += 1;
        }
        
        // Calculate averages
        for (category, sum) in &mut result {
            if let Some(&count) = counts.get(category) {
                *sum /= count as f64;
            }
        }
        
        result
    }
    
    // Find the most expensive products in each category
    fn most_expensive_by_category(&self) -> std::collections::HashMap<String, &Product> {
        let mut result = std::collections::HashMap::new();
        
        for product in &self.products {
            result
                .entry(product.category.clone())
                .and_modify(|p: &mut &Product| {
                    if product.price > p.price {
                        *p = product;
                    }
                })
                .or_insert(product);
        }
        
        result
    }
    
    // Find products with ratings above a threshold
    fn products_with_high_ratings(&self, min_rating: f64, min_reviews: usize) -> Vec<&Product> {
        self.products
            .iter()
            .filter(|p| {
                if let Some(avg) = p.average_rating() {
                    avg >= min_rating && p.ratings.len() >= min_reviews
                } else {
                    false
                }
            })
            .collect()
    }
}

fn main() {
    // Create an inventory and add products
    let mut inventory = Inventory::new();
    
    // Add electronics
    let mut laptop = Product::new(1, "Laptop", 999.99, "Electronics", 10);
    laptop.add_rating(4);
    laptop.add_rating(5);
    laptop.add_rating(4);
    inventory.add_product(laptop);
    
    let mut smartphone = Product::new(2, "Smartphone", 699.99, "Electronics", 15);
    smartphone.add_rating(5);
    smartphone.add_rating(5);
    smartphone.add_rating(4);
    smartphone.add_rating(5);
    inventory.add_product(smartphone);
    
    let mut headphones = Product::new(3, "Wireless Headphones", 149.99, "Electronics", 5);
    headphones.add_rating(3);
    headphones.add_rating(2);
    inventory.add_product(headphones);
    
    // Add books
    let mut book1 = Product::new(4, "Programming Rust", 45.99, "Books", 20);
    book1.add_rating(5);
    book1.add_rating(5);
    book1.add_rating(4);
    book1.add_rating(5);
    inventory.add_product(book1);
    
    let mut book2 = Product::new(5, "The Rust Programming Language", 39.99, "Books", 8);
    book2.add_rating(5);
    book2.add_rating(4);
    book2.add_rating(5);
    inventory.add_product(book2);
    
    // Add clothing
    inventory.add_product(Product::new(6, "T-Shirt", 19.99, "Clothing", 50));
    inventory.add_product(Product::new(7, "Jeans", 49.99, "Clothing", 30));
    
    // Show all available categories
    println!("Available categories: {:?}", inventory.list_categories());
    
    // Show products by category
    println!("\nBooks:");
    for product in inventory.find_by_category("Books") {
        println!("  - {} (${:.2})", product.name, product.price);
    }
    
    // Show products that need restocking
    println!("\nProducts to restock (below 10 units):");
    for product in inventory.need_restocking(10) {
        println!("  - {} (Current stock: {})", product.name, product.stock);
    }
    
    // Calculate total inventory value
    println!("\nTotal inventory value: ${:.2}", inventory.total_value());
    
    // Show average price by category
    println!("\nAverage price by category:");
    for (category, avg_price) in inventory.average_price_by_category() {
        println!("  - {}: ${:.2}", category, avg_price);
    }
    
    // Show most expensive product in each category
    println!("\nMost expensive product in each category:");
    for (category, product) in inventory.most_expensive_by_category() {
        println!("  - {}: {} (${:.2})", category, product.name, product.price);
    }
    
    // Show highly rated products
    println!("\nHighly rated products (at least 4.0 average with 3+ reviews):");
    for product in inventory.products_with_high_ratings(4.0, 3) {
        println!(
            "  - {} - {:.1}/5.0 ({} reviews)",
            product.name,
            product.average_rating().unwrap(),
            product.ratings.len()
        );
    }
}
```

### Common Pitfalls
- **Pitfall**: Trying to use a vector after moving it into a consuming iterator.
  - **Solution**: Use `iter()` instead of `into_iter()` if you need to keep the original vector.
- **Pitfall**: Multiple mutable borrows through iterators.
  - **Solution**: Avoid nested mutable iterations on the same collection; restructure your code or use indices.
- **Pitfall**: Unnecessary collection creation in iterator chains.
  - **Solution**: Only call `collect()` at the end of the chain when you need a concrete collection.
- **Pitfall**: Inefficient iteration patterns that could be solved with specialized methods.
  - **Solution**: Familiarize yourself with the Iterator trait methods like `find`, `any`, `all`, etc.
- **Pitfall**: Forgetting that `.iter()` provides references, not values.
  - **Solution**: Use appropriate patterns (e.g., `iter()` for `&T`, `iter_mut()` for `&mut T`, and `into_iter()` for `T`).

### Confusion Questions with Answers
1. **Q**: What's the difference between `iter()`, `iter_mut()`, and `into_iter()`?
   **A**: 
   - `iter()` creates an iterator that yields immutable references (`&T`) to the elements and doesn't consume the collection.
   - `iter_mut()` creates an iterator that yields mutable references (`&mut T`) to the elements and doesn't consume the collection.
   - `into_iter()` consumes the collection and yields owned values (or references, depending on the context).

2. **Q**: Why doesn't my iterator do anything unless I call `collect()` or similar?
   **A**: Iterators in Rust are lazy—they don't do any work until their elements are requested. Methods like `collect()`, `count()`, `sum()`, or explicit iteration with `for` trigger the actual computation. This is a performance feature called "lazy evaluation."

3. **Q**: How can I iterate over a vector and modify it at the same time?
   **A**: You generally can't iterate and modify a vector simultaneously due to Rust's borrowing rules. Options include:
   - Collect changes and apply them after iteration
   - Use indices or drain filters instead of iterators
   - Create a new vector with the changes
   - Use `split_at_mut()` to get separate mutable views of different parts of the vector

---

## 6. Common Vector Operations

### Concise Explanation
Vectors in Rust support a wide range of operations beyond basic manipulation. These include sorting, searching, slicing, concatenation, and various transformations. Understanding these common operations helps you use vectors effectively and idiomatically in your code.

Key operations:
- Sorting and comparison operations
- Searching and finding elements
- Slicing and subvector extraction
- Joining and splitting vectors
- Deduplication and uniqueness checks
- Advanced transformations

### Where to Use
- When you need to manipulate collections beyond basic add/remove operations
- For data processing and analysis tasks
- When implementing algorithms that require specific vector manipulations
- For efficient data transformation and aggregation
- When working with collections that need to be sorted, filtered, or combined

### Code Snippet
```rust
fn main() {
    let mut numbers = vec![5, 2, 8, 1, 9, 3, 4, 7, 6];
    
    // Sorting
    numbers.sort();
    println!("Sorted: {:?}", numbers);
    
    // Sort in descending order
    numbers.sort_by(|a, b| b.cmp(a));
    println!("Sorted descending: {:?}", numbers);
    
    // Sort by custom criteria
    let mut pairs = vec![(1, 'a'), (3, 'b'), (2, 'c')];
    pairs.sort_by_key(|&(num, _)| num);
    println!("Sorted by first element: {:?}", pairs);
    
    // Stable sort (preserves order of equal elements)
    let mut strings = vec!["hello", "world", "rust", "programming"];
    strings.sort_by_key(|s| s.len());
    println!("Sorted by length: {:?}", strings);
    
    // Partial sorting - find the smallest n elements
    let mut nums = vec![5, 2, 8, 1, 9, 3, 4, 7, 6];
    nums.sort_unstable_by(|a, b| a.partial_cmp(b).unwrap());
    println!("Sorted unstable: {:?}", nums);
    
    // Binary search (only works on sorted arrays)
    let sorted_nums = vec![1, 2, 3, 4, 5, 6, 7, 8, 9];
    match sorted_nums.binary_search(&6) {
        Ok(pos) => println!("Found 6 at position: {}", pos),
        Err(pos) => println!("6 not found, would be inserted at: {}", pos),
    }
    
    // Deduplication
    let mut with_duplicates = vec![1, 2, 2, 3, 3, 3, 4, 5, 5];
    with_duplicates.sort(); // Must be sorted first
    with_duplicates.dedup(); // Removes consecutive duplicates
    println!("Deduplicated: {:?}", with_duplicates);
    
    // Splitting and joining vectors
    let v1 = vec![1, 2, 3];
    let v2 = vec![4, 5, 6];
    
    // Concatenating vectors
    let mut combined = v1.clone();
    combined.extend(v2.clone());
    println!("Combined: {:?}", combined);
    
    // Concatenating with iterators
    let combined2: Vec<i32> = v1.iter().chain(v2.iter()).cloned().collect();
    println!("Combined with chain: {:?}", combined2);
    
    // Split a vector at an index
    let (left, right) = combined.split_at(3);
    println!("Split at index 3: Left: {:?}, Right: {:?}", left, right);
    
    // Mutable split (allows modifying both parts)
    let (left, right) = combined.split_at_mut(3);
    // Now you can modify both halves independently
    
    // Slicing operations
    let slice = &combined[2..5];
    println!("Slice from index 2 to 5: {:?}", slice);
    
    // Creating a reversed vector
    let reversed: Vec<i32> = combined.iter().rev().cloned().collect();
    println!("Reversed: {:?}", reversed);
    
    // Rotating elements left (moves the first n elements to the end)
    let mut to_rotate = vec![1, 2, 3, 4, 5];
    to_rotate.rotate_left(2);
    println!("Rotated left by 2: {:?}", to_rotate); // [3, 4, 5, 1, 2]
    
    // Rotating elements right (moves the last n elements to the beginning)
    to_rotate.rotate_right(3);
    println!("Rotated right by 3: {:?}", to_rotate); // [5, 1, 2, 3, 4]
    
    // Resizing a vector (with default value for new elements)
    let mut resizable = vec![1, 2, 3];
    resizable.resize(5, 0); // Extend to length 5, fill with 0s
    println!("Resized to 5: {:?}", resizable);
    
    resizable.resize(2, 0); // Shrink to length 2
    println!("Resized to 2: {:?}", resizable);
    
    // Retain elements matching a predicate
    let mut nums = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    nums.retain(|&x| x % 2 == 0); // Keep only even numbers
    println!("Only even numbers: {:?}", nums);
    
    // Swapping elements
    let mut swap_vec = vec!['a', 'b', 'c', 'd'];
    swap_vec.swap(0, 3); // Swap first and last elements
    println!("After swap: {:?}", swap_vec);
    
    // Efficient removal when order doesn't matter
    let mut to_remove = vec![1, 2, 3, 4, 5];
    let removed = to_remove.swap_remove(1); // Removes element at index 1
    println!("Removed element: {}, Vector: {:?}", removed, to_remove);
    
    // Joining vector elements into a string
    let words = vec!["hello", "world", "of", "rust"];
    let sentence = words.join(" ");
    println!("Joined: {}", sentence);
    
    // Converting to and from arrays (if size is known)
    let array_from_vec: [i32; 3] = [1, 2, 3];
    let vec_from_array = Vec::from(array_from_vec);
    println!("Vector from array: {:?}", vec_from_array);
    
    // Converting iterator to vector
    let squares: Vec<i32> = (1..=5).map(|x| x * x).collect();
    println!("Squares: {:?}", squares);
    
    // Window iterator (sliding window of elements)
    let data = vec![1, 2, 3, 4, 5];
    for window in data.windows(3) {
        println!("Window of 3: {:?}", window);
    }
    
    // Chunks iterator (non-overlapping chunks of elements)
    for chunk in data.chunks(2) {
        println!("Chunk of 2: {:?}", chunk);
    }
    
    // Flattening nested vectors
    let nested = vec![vec![1, 2], vec![3, 4, 5]];
    let flattened: Vec<i32> = nested.iter().flatten().cloned().collect();
    println!("Flattened: {:?}", flattened);
}
```

### Real-World Example
A text analysis tool that processes a document and provides various statistics:

```rust
use std::collections::{HashMap, HashSet};

struct TextAnalyzer {
    text: String,
    words: Vec<String>,
    word_count: HashMap<String, usize>,
}

impl TextAnalyzer {
    fn new(text: &str) -> Self {
        // Split the text into words
        let words: Vec<String> = text
            .split_whitespace()
            .map(|word| {
                word.trim_matches(|c: char| !c.is_alphanumeric())
                    .to_lowercase()
            })
            .filter(|word| !word.is_empty())
            .collect();
        
        // Count occurrences of each word
        let mut word_count = HashMap::new();
        for word in &words {
            *word_count.entry(word.clone()).or_insert(0) += 1;
        }
        
        TextAnalyzer {
            text: text.to_string(),
            words,
            word_count,
        }
    }
    
    // Get total number of words
    fn word_count(&self) -> usize {
        self.words.len()
    }
    
    // Get unique word count
    fn unique_word_count(&self) -> usize {
        self.word_count.len()
    }
    
    // Get average word length
    fn average_word_length(&self) -> f64 {
        if self.words.is_empty() {
            return 0.0;
        }
        
        let total_chars: usize = self.words.iter().map(|word| word.len()).sum();
        total_chars as f64 / self.words.len() as f64
    }
    
    // Get most common words
    fn most_common_words(&self, limit: usize) -> Vec<(String, usize)> {
        let mut word_counts: Vec<(String, usize)> = self.word_count
            .iter()
            .map(|(word, count)| (word.clone(), *count))
            .collect();
        
        // Sort by count in descending order
        word_counts.sort_by(|a, b| b.1.cmp(&a.1).then_with(|| a.0.cmp(&b.0)));
        
        // Take only the requested number of items
        word_counts.truncate(limit);
        word_counts
    }
    
    // Get longest words
    fn longest_words(&self, limit: usize) -> Vec<String> {
        let mut unique_words: Vec<String> = self.word_count
            .keys()
            .cloned()
            .collect();
        
        // Sort by length in descending order
        unique_words.sort_by(|a, b| b.len().cmp(&a.len()).then_with(|| a.cmp(b)));
        
        // Take only the requested number of items
        unique_words.truncate(limit);
        unique_words
    }
    
    // Find sentences containing a specific word
    fn sentences_with_word(&self, word: &str) -> Vec<String> {
        let lowercase_word = word.to_lowercase();
        let sentences: Vec<&str> = self.text
            .split(&['.', '!', '?'][..])
            .map(|s| s.trim())
            .filter(|s| !s.is_empty())
            .collect();
        
        sentences
            .iter()
            .filter(|&sentence| {
                sentence
                    .split_whitespace()
                    .any(|w| w.trim_matches(|c: char| !c.is_alphanumeric())
                        .to_lowercase() == lowercase_word)
            })
            .map(|&s| s.to_string())
            .collect()
    }
    
    // Get word frequency distribution
    fn word_frequency(&self) -> HashMap<usize, usize> {
        let mut frequency_count = HashMap::new();
        
        for &count in self.word_count.values() {
            *frequency_count.entry(count).or_insert(0) += 1;
        }
        
        frequency_count
    }
    
    // Find word pairs that appear together frequently
    fn common_word_pairs(&self, limit: usize) -> Vec<(String, String, usize)> {
        if self.words.len() < 2 {
            return Vec::new();
        }
        
        let mut pair_counts = HashMap::new();
        
        // Count adjacent word pairs
        for window in self.words.windows(2) {
            let pair = (window[0].clone(), window[1].clone());
            *pair_counts.entry(pair).or_insert(0) += 1;
        }
        
        // Convert to vector and sort
        let mut pairs: Vec<((String, String), usize)> = pair_counts
            .into_iter()
            .collect();
        
        pairs.sort_by(|a, b| b.1.cmp(&a.1));
        
        // Take top N and format
        pairs
            .into_iter()
            .take(limit)
            .map(|((word1, word2), count)| (word1, word2, count))
            .collect()
    }
    
    // Find keywords (excluding common words)
    fn keywords(&self, limit: usize) -> Vec<String> {
        // Common words to exclude
        let common_words: HashSet<&str> = [
            "the", "a", "an", "and", "or", "but", "in", "on", "at", 
            "to", "for", "with", "by", "of", "is", "are", "was", "were",
            "from", "that", "this", "it", "as", "be", "have", "has"
        ].iter().cloned().collect();
        
        let mut filtered_words: Vec<(String, usize)> = self.word_count
            .iter()
            .filter(|(word, _)| !common_words.contains(word.as_str()) && word.len() > 2)
            .map(|(word, count)| (word.clone(), *count))
            .collect();
        
        filtered_words.sort_by(|a, b| b.1.cmp(&a.1));
        
        filtered_words
            .into_iter()
            .take(limit)
            .map(|(word, _)| word)
            .collect()
    }
}

fn main() {
    let sample_text = "Rust is a multi-paradigm, general-purpose programming language. \
        Rust emphasizes performance, type safety, and concurrency. Rust enforces memory \
        safety—that is, that all references point to valid memory—without requiring the \
        use of a garbage collector or reference counting present in other memory-safe \
        languages. To simultaneously enforce memory safety and prevent concurrent data \
        races, Rust's borrow checker tracks the object lifetime and variable scope of all \
        references in a program during compilation. Rust is popular for systems programming \
        but also offers high-level features including functional programming constructs.";
    
    let analyzer = TextAnalyzer::new(sample_text);
    
    // Basic statistics
    println!("Text Analysis Results:");
    println!("Word count: {}", analyzer.word_count());
    println!("Unique words: {}", analyzer.unique_word_count());
    println!("Average word length: {:.2} characters", analyzer.average_word_length());
    
    // Most common words
    println!("\nMost common words:");
    for (word, count) in analyzer.most_common_words(5) {
        println!("  '{}': {} occurrences", word, count);
    }
    
    // Longest words
    println!("\nLongest words:");
    for word in analyzer.longest_words(5) {
        println!("  '{}' ({} characters)", word, word.len());
    }
    
    // Word frequency distribution
    println!("\nWord frequency distribution:");
    let mut frequencies: Vec<(usize, usize)> = analyzer.word_frequency().into_iter().collect();
    frequencies.sort_by_key(|&(count, _)| count);
    for (count, words) in frequencies {
        println!("  {} words appear {} time(s)", words, count);
    }
    
    // Find sentences with a specific word
    let target_word = "memory";
    println!("\nSentences containing '{}':", target_word);
    for (i, sentence) in analyzer.sentences_with_word(target_word).iter().enumerate() {
        println!("  {}. {}", i + 1, sentence);
    }
    
    // Common word pairs
    println!("\nCommon word pairs:");
    for (word1, word2, count) in analyzer.common_word_pairs(5) {
        println!("  '{}' + '{}': {} occurrences", word1, word2, count);
    }
    
    // Keywords
    println!("\nKeywords:");
    for keyword in analyzer.keywords(10) {
        println!("  {}", keyword);
    }
}
```

### Common Pitfalls
- **Pitfall**: Using `sort` unnecessarily when only the smallest/largest elements are needed.
  - **Solution**: Consider using `partial_sort` or maintaining a heap for better performance.
- **Pitfall**: Repeatedly searching through vectors linearly when a HashMap would be more efficient.
  - **Solution**: Choose the right data structure; use HashMaps for frequent lookups by value.
- **Pitfall**: Inefficient removal of multiple elements by repeatedly calling `remove`.
  - **Solution**: Use `retain` or `drain_filter` (when available) for batch removals.
- **Pitfall**: Creating unnecessary temporary vectors during transformations.
  - **Solution**: Chain iterator methods and only collect once at the end.
- **Pitfall**: Performing binary search on unsorted vectors.
  - **Solution**: Remember to sort before using `binary_search` methods.

### Confusion Questions with Answers
1. **Q**: What's the difference between `sort` and `sort_unstable`?
   **A**: `sort` is a stable sort that preserves the relative order of equal elements, while `sort_unstable` is potentially faster but doesn't guarantee preserving the order of equal elements. Use `sort` when you need to maintain relative ordering, and `sort_unstable` when you only care about the final sorted order.

2. **Q**: How can I efficiently remove multiple elements from a vector?
   **A**: The most efficient way is to use the `retain` method with a predicate function that returns `true` for elements you want to keep and `false` for those you want to remove. This avoids repeated shifting of elements that would occur with individual `remove` calls.

3. **Q**: When should I use `swap_remove` instead of `remove`?
   **A**: Use `swap_remove` when you don't care about preserving the order of elements in the vector. It's O(1) because it replaces the removed element with the last element in the vector rather than shifting all subsequent elements. Use `remove` when order matters, despite its O(n) time complexity.

---

## Next Actions: Exercises and Projects

### Exercises
1. **Array Manipulation Practice**
   - Create a function to reverse an array in-place
   - Implement a function to find the median value in an array
   - Write a function to check if an array is a palindrome
   - Create a function that rotates an array by n positions

2. **Vector Operations Challenge**
   - Implement a function to merge two sorted vectors while maintaining order
   - Create a function to remove all duplicates from a vector (not just consecutive ones)
   - Write a function that finds the "mode" (most common element) in a vector
   - Implement a moving average calculator using vectors

3. **Multidimensional Array Workshop**
   - Create a function to transpose a 2D matrix
   - Implement a flood fill algorithm on a 2D grid
   - Write a function to calculate the determinant of a 3x3 matrix
   - Implement Conway's Game of Life using 2D arrays

4. **Vector Performance Optimization**
   - Benchmark different methods of growing a vector (one by one vs. with_capacity)
   - Analyze the performance of different sorting strategies on vectors
   - Measure the impact of using `reserve` before multiple push operations
   - Compare the performance of vector, array, and linked list for different operations

### Micro-Project: Simple Image Processing
Build a simple image processing library that:
- Represents an image as a 2D array or vector of vectors
- Implements basic operations: rotate, flip, invert colors, adjust brightness
- Supports simple filters like blur, sharpen, or grayscale conversion
- Provides a way to convert between formats or display the output

```rust
// Example starter code for image processing micro-project
struct Image {
    width: usize,
    height: usize,
    // RGB pixels (each pixel is [R, G, B] where each color is 0-255)
    pixels: Vec<Vec<[u8; 3]>>,
}

impl Image {
    // Create a new blank image
    fn new(width: usize, height: usize, background: [u8; 3]) -> Self {
        let pixels = vec![vec![background; width]; height];
        Image { width, height, pixels }
    }
    
    // Rotate the image 90 degrees clockwise
    fn rotate_90_clockwise(&self) -> Self {
        let mut rotated = Image::new(self.height, self.width, [0, 0, 0]);
        
        for y in 0..self.height {
            for x in 0..self.width {
                rotated.pixels[x][self.height - 1 - y] = self.pixels[y][x];
            }
        }
        
        rotated
    }
    
    // Flip the image horizontally
    fn flip_horizontal(&self) -> Self {
        let mut flipped = Image::new(self.width, self.height, [0, 0, 0]);
        
        for y in 0..self.height {
            for x in 0..self.width {
                flipped.pixels[y][self.width - 1 - x] = self.pixels[y][x];
            }
        }
        
        flipped
    }
    
    // Apply a grayscale filter
    fn grayscale(&self) -> Self {
        let mut gray = Image::new(self.width, self.height, [0, 0, 0]);
        
        for y in 0..self.height {
            for x in 0..self.width {
                let pixel = self.pixels[y][x];
                // Convert to grayscale using luminosity method
                let gray_value = (0.299 * pixel[0] as f32 + 
                                 0.587 * pixel[1] as f32 + 
                                 0.114 * pixel[2] as f32) as u8;
                gray.pixels[y][x] = [gray_value, gray_value, gray_value];
            }
        }
        
        gray
    }
    
    // Implement more image processing functions...
}

// Complete the implementation with more filters and functionality
```

## Success Criteria
You've mastered arrays and vectors in Rust when you can:
1. Choose the appropriate data structure (array vs. vector) based on needs
2. Efficiently manipulate vectors with appropriate methods and patterns
3. Optimize vector operations for performance using capacity management
4. Work with multidimensional arrays and vectors for grid-based data
5. Leverage iterators and functional-style operations for transformations
6. Implement common algorithms efficiently using vector operations
7. Apply best practices to avoid common pitfalls with arrays and vectors

## Troubleshooting Advice
1. **Array Index Out of Bounds**
   - Use `get` or `get_mut` which return `Option` instead of direct indexing
   - Add explicit bounds checks before indexing
   - Consider using iterator methods instead of index-based access

2. **Vector Performance Issues**
   - Preallocate capacity when the approximate size is known
   - Use `with_capacity` when creating vectors from iterators with known size
   - Avoid unnecessary cloning of vector elements
   - Consider specialized data structures for specific use cases

3. **Borrowing and Mutability Errors**
   - Use `split_at_mut` to get multiple mutable slices
   - Restructure code to avoid simultaneous mutable borrows
   - Consider using indices instead of references when dealing with complex mutation patterns
   - Use scopes to limit the lifetime of borrows

4. **Memory Efficiency Concerns**
   - Call `shrink_to_fit` after removing many elements
   - Use appropriate types (e.g., `u8` instead of `i32`) when storing many small values
   - Consider using arrays for small, fixed-size collections
   - Use specialized collections like `SmallVec` for small vectors with stack allocation

5. **Unexpected Clone or Copy Behavior**
   - Be aware that arrays and vectors of Copy types implement Copy and Clone
   - Use references when passing large arrays or vectors to functions
   - Be explicit about cloning when needed
   - Understand ownership semantics for each operation

