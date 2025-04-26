# Module 3: Functions and Packages

## Core Problem
Understanding how to organize Go code through functions and packages to create reusable, maintainable, and well-structured programs.

## Key Assumptions
- You understand basic Go syntax and data types
- You want to write modular, reusable code
- You need to organize code into logical units
- You want to leverage Go's standard library effectively

## Essential Concepts

### 1. Defining and Calling Functions

**Concise Explanation:**
Functions in Go are defined with the `func` keyword, followed by the function name, parameters, return type, and body. Functions can be called by using their name followed by parentheses containing any required arguments. Go functions can return multiple values and can be defined as methods on types.

**Where to Use:**
- Encapsulating reusable logic
- Breaking complex operations into manageable steps
- Creating APIs for other code to use
- Implementing behavior for custom types

**Code Snippet:**
```go
// Basic function definition
func greet(name string) string {
    return "Hello, " + name + "!"
}

// Function with multiple parameters
func add(a, b int) int {
    return a + b
}

// Function with no return value
func printInfo(message string) {
    fmt.Println(message)
}

// Function as a method on a type
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Using the functions
func main() {
    // Calling a function
    message := greet("Gopher")
    fmt.Println(message) // Output: Hello, Gopher!
    
    // Calling a function with multiple arguments
    sum := add(5, 7)
    fmt.Println(sum) // Output: 12
    
    // Calling a method on a struct
    rect := Rectangle{Width: 10, Height: 5}
    area := rect.Area()
    fmt.Println(area) // Output: 50
}
```

**Real-World Example:**
A web server handling HTTP requests uses functions to route and process different endpoints:

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "time"
)

// Handler function for the home page
func homeHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Welcome to our website!")
}

// Handler function for the API endpoint
func apiHandler(w http.ResponseWriter, r *http.Request) {
    data := map[string]interface{}{
        "message": "API response",
        "time":    time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(data)
}

// Main function that sets up the server
func main() {
    // Register route handlers
    http.HandleFunc("/", homeHandler)
    http.HandleFunc("/api", apiHandler)
    
    // Start the server
    fmt.Println("Server starting on :8080")
    http.ListenAndServe(":8080", nil)
}
```

**Common Pitfalls:**
- Forgetting to return values from functions that require returns
- Not handling errors returned by called functions
- Creating overly complex functions instead of breaking them into smaller, focused ones
- Using global variables instead of passing parameters
- Accidentally modifying slice or map parameters (they're passed by reference)
- Ignoring return values from functions

**Confusion Questions:**

1. **Q: What's the difference between functions and methods in Go?**
   
   A: In Go, a function is a standalone piece of code that performs a specific task, while a method is a function that belongs to a specific type (the "receiver"). Methods have an extra receiver parameter before the method name that specifies the type to which the method belongs.
   
   ```go
   // Function
   func CalculateArea(width, height float64) float64 {
       return width * height
   }
   
   // Method
   func (r Rectangle) CalculateArea() float64 {
       return r.Width * r.Height
   }
   
   // Usage
   area1 := CalculateArea(10, 5)     // Calling a function
   rect := Rectangle{10, 5}
   area2 := rect.CalculateArea()     // Calling a method
   ```
   
   Methods provide a way to associate behavior with data, similar to object-oriented programming, while functions are more procedural in nature.

2. **Q: When should I use pointer receivers versus value receivers for methods?**
   
   A: Use pointer receivers when:
   - You need to modify the receiver (change its state)
   - The receiver is a large struct (to avoid copying)
   - Consistency is needed with other methods of the same type
   
   Use value receivers when:
   - The receiver is small and naturally copied (like primitives or small structs)
   - You don't need to modify the receiver
   - The receiver is meant to be immutable
   
   ```go
   // Value receiver - doesn't modify the original Rectangle
   func (r Rectangle) Area() float64 {
       return r.Width * r.Height
   }
   
   // Pointer receiver - modifies the original Rectangle
   func (r *Rectangle) Scale(factor float64) {
       r.Width *= factor
       r.Height *= factor
   }
   
   func main() {
       rect := Rectangle{10, 5}
       area := rect.Area()           // Value receiver method
       rect.Scale(2)                 // Pointer receiver method
       newArea := rect.Area()        // rect is now {20, 10}
   }
   ```
   
   The general rule of thumb: if in doubt, use a pointer receiver for consistency and to avoid unexpected behavior.

3. **Q: How does Go handle function overloading (defining multiple functions with the same name)?**
   
   A: Go doesn't support function overloading like C++ or Java. Each function name must be unique within its scope. Instead, Go offers several alternatives:
   
   1. Use different function names:
      ```go
      func AddInts(a, b int) int { return a + b }
      func AddFloats(a, b float64) float64 { return a + b }
      ```
   
   2. Use variadic functions or optional parameters with default values:
      ```go
      func Add(nums ...int) int {
          total := 0
          for _, n := range nums {
              total += n
          }
          return total
      }
      ```
   
   3. Use interfaces for polymorphic behavior:
      ```go
      type Adder interface {
          Add(interface{}) interface{}
      }
      ```
   
   4. Use type methods for specialized behavior:
      ```go
      func (i Int) Add(j Int) Int { return i + j }
      func (f Float) Add(g Float) Float { return f + g }
      ```
   
   This approach forces explicit naming and type specification, making code clearer and less ambiguous.

### 2. Parameters and Return Values

**Concise Explanation:**
Go functions specify parameters with name and type, and return values with their types. Parameters are passed by value, meaning functions receive a copy of the argument's value, though slices, maps, and channels effectively behave like references. Return types are specified after parameters, and a function can return multiple values.

**Where to Use:**
- Defining function inputs and outputs
- Processing data and returning results
- Handling optional inputs with default values
- Returning both results and error status

**Code Snippet:**
```go
// Basic parameters and return
func multiply(x, y int) int {
    return x * y
}

// Multiple parameters of different types
func formatName(firstName, lastName string, id int) string {
    return fmt.Sprintf("%s %s (ID: %d)", firstName, lastName, id)
}

// No parameters
func getCurrentTime() time.Time {
    return time.Now()
}

// No return value
func logMessage(message string) {
    log.Println(message)
}

// Default values using optional parameters
func greet(name string, language string) string {
    if language == "" {
        language = "en"
    }
    
    switch language {
    case "es":
        return "¡Hola, " + name + "!"
    case "fr":
        return "Bonjour, " + name + "!"
    default:
        return "Hello, " + name + "!"
    }
}

// Using the functions
func main() {
    product := multiply(4, 5)
    fullName := formatName("John", "Doe", 12345)
    now := getCurrentTime()
    logMessage("Application started")
    
    // Optional parameter
    greeting1 := greet("Maria", "")       // Uses default language
    greeting2 := greet("Pierre", "fr")    // Uses specified language
}
```

**Real-World Example:**
A user authentication system uses functions with appropriate parameters and return values:

```go
package auth

import (
    "errors"
    "time"
)

// User represents an authenticated user
type User struct {
    ID        string
    Email     string
    Name      string
    CreatedAt time.Time
}

// AuthResponse contains authentication results
type AuthResponse struct {
    User        *User
    Token       string
    ExpireTime  time.Time
}

// LoginUser authenticates a user and returns user details and token
func LoginUser(email, password string) (AuthResponse, error) {
    // Validate parameters
    if email == "" || password == "" {
        return AuthResponse{}, errors.New("email and password required")
    }
    
    // Check credentials against database
    user, err := findUserByEmail(email)
    if err != nil {
        return AuthResponse{}, err
    }
    
    // Verify password
    if !verifyPassword(user.ID, password) {
        return AuthResponse{}, errors.New("invalid credentials")
    }
    
    // Generate token
    token, expiry := generateToken(user.ID)
    
    // Return successful response
    return AuthResponse{
        User:       user,
        Token:      token,
        ExpireTime: expiry,
    }, nil
}

// Helper functions
func findUserByEmail(email string) (*User, error) {
    // Database lookup implementation
    // ...
    return &User{
        ID:        "user123",
        Email:     email,
        Name:      "John Doe",
        CreatedAt: time.Now().Add(-30 * 24 * time.Hour),
    }, nil
}

func verifyPassword(userID, password string) bool {
    // Password verification implementation
    // ...
    return true
}

func generateToken(userID string) (string, time.Time) {
    // Token generation implementation
    // ...
    expiry := time.Now().Add(24 * time.Hour)
    return "jwt-token-string", expiry
}
```

**Common Pitfalls:**
- Forgetting that parameters are passed by value (copies are created)
- Not checking slices/maps for nil before operating on them
- Returning pointers to local variables that go out of scope
- Accidentally modifying input parameters that should remain unchanged
- Returning or passing large structs by value when pointers would be more efficient
- Inconsistent error handling in return values

**Confusion Questions:**

1. **Q: How are slices and maps handled when passed as arguments to functions?**
   
   A: While Go uses pass by value for all arguments (creating copies), slices and maps contain internal pointers to their underlying data. When you pass a slice or map to a function:
   
   - The slice or map header (a small struct) is copied
   - The copied header still points to the original underlying data
   - Changes to the elements of a slice or entries in a map are visible to the caller
   - However, reassigning the entire slice or map within the function won't affect the caller
   
   ```go
   func modifySlice(s []int) {
       s[0] = 100            // This change is visible to the caller
       s = append(s, 200)    // This change is NOT visible to the caller
   }
   
   func modifyMap(m map[string]int) {
       m["key"] = 100        // This change is visible to the caller
       m = nil               // This change is NOT visible to the caller
   }
   
   func main() {
       slice := []int{1, 2, 3}
       modifySlice(slice)
       fmt.Println(slice)    // Output: [100 2 3] (first element changed)
       
       myMap := map[string]int{"key": 1}
       modifyMap(myMap)
       fmt.Println(myMap)    // Output: map[key:100] (value changed)
   }
   ```
   
   If you need to modify the slice itself (length, capacity) or reassign the map, pass a pointer to the slice or map.

2. **Q: When should I use named return values versus regular return values?**
   
   A: Named return values have several use cases:
   
   - When return values have a clear semantic meaning:
     ```go
     func divideAndRemainder(a, b int) (quotient, remainder int) {
         quotient = a / b
         remainder = a % b
         return  // Implicit return of named values
     }
     ```
   
   - To make complex logic with multiple return points clearer:
     ```go
     func processData(data []byte) (result string, err error) {
         if len(data) == 0 {
             err = errors.New("empty data")
             return  // Both result (empty) and err are returned
         }
         
         // Process data...
         if problemDetected {
             err = errors.New("processing failed")
             return  // Partial result and err are returned
         }
         
         result = string(data)
         return  // Both result and nil err are returned
     }
     ```
   
   - For documentation purposes in interfaces or public APIs
   
   However, named return values should be avoided when:
   - Returns are simple and obvious
   - Return values are used immediately and don't need names
   - Too many return values would make the function signature cluttered
   
   Remember that named return variables are initialized to their zero values and are visible throughout the function.

3. **Q: How can I make a function parameter optional in Go?**
   
   A: Go doesn't have built-in optional parameters or default values, but there are several idiomatic approaches:
   
   1. Function variants with different signatures:
      ```go
      func Connect() (*Connection, error) {
          return Connect("localhost", 8080)
      }
      
      func ConnectWithHost(host string) (*Connection, error) {
          return Connect(host, 8080)
      }
      
      func Connect(host string, port int) (*Connection, error) {
          // Implementation
      }
      ```
   
   2. Variadic parameters (for trailing options):
      ```go
      func CreateUser(name string, options ...UserOption) *User {
          // Default values
          user := &User{
              Name:  name,
              Role:  "user",
              Active: true,
          }
          
          // Apply options
          for _, option := range options {
              option(user)
          }
          
          return user
      }
      
      // Option functions
      func WithRole(role string) UserOption {
          return func(u *User) {
              u.Role = role
          }
      }
      
      func WithActive(active bool) UserOption {
          return func(u *User) {
              u.Active = active
          }
      }
      
      // Usage
      user1 := CreateUser("Alice")  // Default options
      user2 := CreateUser("Bob", WithRole("admin"), WithActive(false))
      ```
   
   3. Options struct (best for many options):
      ```go
      type ServerOptions struct {
          Port    int
          Timeout time.Duration
          TLS     bool
      }
      
      func NewServer(addr string, options ServerOptions) *Server {
          // Use options or defaults
          if options.Port == 0 {
              options.Port = 8080  // Default
          }
          if options.Timeout == 0 {
              options.Timeout = 30 * time.Second  // Default
          }
          
          // Create server
      }
      
      // Usage
      server1 := NewServer("localhost", ServerOptions{})  // All defaults
      server2 := NewServer("example.com", ServerOptions{
          Port: 443,
          TLS:  true,
      })  // Custom port and TLS, default timeout
      ```
   
   The approach you choose depends on the complexity of your function and the number of optional parameters.

### 3. Multiple Return Values

**Concise Explanation:**
Go functions can return multiple values, separated by commas. This is commonly used to return both a result and an error status, or to return multiple related values from a single operation. Multiple return values are a cleaner alternative to output parameters in other languages.

**Where to Use:**
- Returning both results and error information
- Functions that need to return related values
- Operations that have multiple outcomes
- Parsing or extraction functions that return a value and success status

**Code Snippet:**
```go
// Return both result and error
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Return multiple related values
func minMax(nums []int) (min, max int) {
    if len(nums) == 0 {
        return 0, 0
    }
    
    min, max = nums[0], nums[0]
    for _, num := range nums[1:] {
        if num < min {
            min = num
        }
        if num > max {
            max = num
        }
    }
    return min, max
}

// Parse a string into components
func parseUserInfo(info string) (name string, age int, active bool) {
    parts := strings.Split(info, ",")
    if len(parts) < 3 {
        return "", 0, false
    }
    
    name = strings.TrimSpace(parts[0])
    age, _ = strconv.Atoi(strings.TrimSpace(parts[1]))
    active = strings.TrimSpace(parts[2]) == "active"
    
    return name, age, active
}

// Using multiple return values
func main() {
    // Handle errors with multiple returns
    result, err := divide(10, 2)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Result:", result)
    
    // Get both values from single call
    smallest, largest := minMax([]int{3, 7, 1, 9, 4})
    fmt.Printf("Min: %d, Max: %d\n", smallest, largest)
    
    // Parse complex data
    name, age, active := parseUserInfo("John Doe, 35, active")
    fmt.Printf("Name: %s, Age: %d, Active: %t\n", name, age, active)
    
    // Ignore values you don't need with blank identifier
    _, err = divide(5, 0)  // Only care about the error
    justTheMax := func(nums []int) int {
        _, max := minMax(nums)
        return max
    }
}
```

**Real-World Example:**
A file processing utility that reads, parses, and processes configuration files:

```go
package config

import (
    "encoding/json"
    "io/ioutil"
    "os"
    "path/filepath"
)

// Config holds application configuration
type Config struct {
    DatabaseURL string   `json:"database_url"`
    ServerPort  int      `json:"server_port"`
    LogLevel    string   `json:"log_level"`
    AllowedIPs  []string `json:"allowed_ips"`
}

// FindConfigFile looks for a config file in multiple locations
func FindConfigFile(fileName string) (string, bool) {
    // Check current directory
    if _, err := os.Stat(fileName); err == nil {
        return fileName, true
    }
    
    // Check home directory
    homeDir, err := os.UserHomeDir()
    if err == nil {
        homePath := filepath.Join(homeDir, ".config", fileName)
        if _, err := os.Stat(homePath); err == nil {
            return homePath, true
        }
    }
    
    // Check system directory
    sysPath := filepath.Join("/etc", fileName)
    if _, err := os.Stat(sysPath); err == nil {
        return sysPath, true
    }
    
    return "", false
}

// LoadConfig loads and parses a JSON configuration file
func LoadConfig(path string) (Config, error) {
    var config Config
    
    // Set defaults
    config.ServerPort = 8080
    config.LogLevel = "info"
    
    // Read file
    data, err := ioutil.ReadFile(path)
    if err != nil {
        return config, err
    }
    
    // Parse JSON
    err = json.Unmarshal(data, &config)
    if err != nil {
        return config, err
    }
    
    return config, nil
}

// ValidateConfig checks if the configuration is valid
func ValidateConfig(cfg Config) (bool, []string) {
    var problems []string
    
    if cfg.DatabaseURL == "" {
        problems = append(problems, "database_url is required")
    }
    
    if cfg.ServerPort < 1024 || cfg.ServerPort > 65535 {
        problems = append(problems, "server_port must be between 1024 and 65535")
    }
    
    valid := len(problems) == 0
    return valid, problems
}

// Example usage
func SetupApplication() (Config, error) {
    // Find config file
    configPath, found := FindConfigFile("app.json")
    if !found {
        return Config{}, errors.New("configuration file not found")
    }
    
    // Load config
    config, err := LoadConfig(configPath)
    if err != nil {
        return Config{}, fmt.Errorf("error loading config: %w", err)
    }
    
    // Validate config
    valid, problems := ValidateConfig(config)
    if !valid {
        return Config{}, fmt.Errorf("invalid configuration: %s", strings.Join(problems, ", "))
    }
    
    return config, nil
}
```

**Common Pitfalls:**
- Not checking error returns from functions
- Ignoring important returned values
- Returning too many values from a single function (making it hard to use)
- Inconsistent error handling or return patterns
- Forgetting that each return statement must provide all return values
- Not using the blank identifier (`_`) for unused return values

**Confusion Questions:**

1. **Q: What's the best practice for handling multiple return values, especially with errors?**
   
   A: The most common Go idiom for handling multiple return values with errors is:
   
   ```go
   result, err := someFunction()
   if err != nil {
       // Handle error
       return nil, err  // Often propagate the error up
   }
   
   // Continue with successful result
   ```
   
   Additional best practices include:
   
   - Check errors immediately after the function call
   - Don't ignore errors unless you have a specific reason
   - Handle each error appropriately - don't just log and continue
   - Use the blank identifier (`_`) when you intentionally want to ignore a return value
   - When wrapping errors, use `fmt.Errorf("context: %w", err)` to preserve the error chain
   - Consider custom error types for complex error handling
   
   ```go
   // Good error handling
   user, err := db.GetUser(userID)
   if err != nil {
       if errors.Is(err, sql.ErrNoRows) {
           return nil, fmt.Errorf("user %s not found", userID)
       }
       return nil, fmt.Errorf("database error: %w", err)
   }
   ```
   
   This pattern makes error handling explicit and helps create robust, reliable code.

2. **Q: How do multiple return values in Go compare to approaches in other languages?**
   
   A: Go's multiple return values provide several advantages over approaches in other languages:
   
   - **Compared to output parameters** (C#, C++):
     Go's approach is clearer as outputs are explicit in the function signature, rather than being implicit in parameter annotations.
     ```go
     // Go - clear which values are returned
     func divide(a, b int) (int, int, error) { ... }
     
     // vs C# - less clear which parameters are outputs
     void Divide(int a, int b, out int quotient, out int remainder) { ... }
     ```
   
   - **Compared to tuples/pairs** (Python, Scala):
     Go's approach names the returned types and often uses named return values, making the API clearer.
     ```go
     // Go - explicit types for each return value
     func processFile(path string) (data []byte, lines int, err error) { ... }
     
     // vs Python - tuple of unspecified types
     def process_file(path): # Returns (data, line_count)
     ```
   
   - **Compared to single returns with exceptions** (Java, C#):
     Go's explicit error returns lead to clearer error handling flow rather than invisible exception paths.
     ```go
     // Go - error handling is visible in the code
     data, err := LoadConfig("config.json")
     if err != nil {
         // Handle error
     }
     
     // vs Java - exception paths are invisible
     try {
         Config data = loadConfig("config.json");
         // Use data
     } catch (Exception e) {
         // Handle exception
     }
     ```
   
   The Go approach leads to more explicit code with clearer intent and more predictable error handling.

3. **Q: How do I deal with situations where a function needs to return many values?**
   
   A: If a function needs to return more than 2-3 values, consider these alternatives:
   
   1. **Use a struct to group related return values**:
      ```go
      // Too many return values
      func getFileInfo(path string) (string, int64, time.Time, os.FileMode, error)
      
      // Better: Group related values in a struct
      type FileInfo struct {
          Name    string
          Size    int64
          ModTime time.Time
          Mode    os.FileMode
      }
      
      func getFileInfo(path string) (FileInfo, error)
      ```
   
   2. **Split into multiple functions with focused responsibilities**:
      ```go
      // Too broad
      func processUserData(userID string) (user User, orders []Order, stats UserStats, err error)
      
      // Better: Split responsibilities
      func GetUser(userID string) (User, error)
      func GetUserOrders(userID string) ([]Order, error)
      func GetUserStats(userID string) (UserStats, error)
      ```
   
   3. **Use builder pattern or step-by-step processing**:
      ```go
      // Instead of one complex function
      result, metadata, logs, err := processComplexData(input)
      
      // Use a builder or processor
      processor := NewDataProcessor(input)
      if err := processor.Validate(); err != nil {
          return err
      }
      result := processor.GetResult()
      metadata := processor.GetMetadata()
      logs := processor.GetLogs()
      ```
   
   4. **Consider if all returns are truly necessary**:
      Sometimes callers don't need all the returned information. Design your API based on actual caller needs.
      
   Having more than three return values often indicates the function is doing too much or the returns should be grouped logically.

### 4. Named Return Values

**Concise Explanation:**
Named return values in Go allow you to give names to the return values in a function declaration. These named variables are initialized to their zero values at the start of the function and can be returned implicitly using a "naked" return statement. Named returns enhance readability and make complex return logic cleaner.

**Where to Use:**
- When return values have clear semantic meanings
- Functions with complex logic and multiple return points
- Documenting function return values
- Simplifying error handling with early returns

**Code Snippet:**
```go
// Basic named return values
func divide(a, b int) (quotient int, remainder int, err error) {
    if b == 0 {
        err = errors.New("division by zero")
        return // Returns initialized quotient, remainder and the error
    }
    
    quotient = a / b
    remainder = a % b
    return // Returns quotient, remainder, and nil error
}

// Compact form for same types
func rectDimensions(area float64, aspectRatio float64) (width, height float64) {
    width = math.Sqrt(area * aspectRatio)
    height = area / width
    return // Returns calculated width and height
}

// Early returns with named values
func processTransaction(txID string) (success bool, result string, err error) {
    // Default to failure unless explicitly successful
    success = false
    
    tx, err := database.GetTransaction(txID)
    if err != nil {
        result = "Transaction not found"
        return // Early return with zero success, error message and err
    }
    
    if tx.IsProcessed {
        result = "Transaction already processed"
        return // Another early return
    }
    
    err = tx.Process()
    if err != nil {
        result = "Processing failed"
        return // Another early return with error
    }
    
    // Success case
    success = true
    result = "Transaction processed successfully"
    return
}

// Using named return values
func main() {
    q, r, err := divide(10, 3)
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Printf("10 ÷ 3 = %d remainder %d\n", q, r)
    }
    
    width, height := rectDimensions(100, 16.0/9.0)
    fmt.Printf("Rectangle dimensions: %.2f x %.2f\n", width, height)
    
    success, result, err := processTransaction("tx_123456")
    if !success {
        fmt.Printf("Failed: %s, Error: %v\n", result, err)
    } else {
        fmt.Printf("Success: %s\n", result)
    }
}
```

**Real-World Example:**
A file parsing utility with named return values for clear error handling:

```go
package parser

import (
    "bufio"
    "errors"
    "fmt"
    "os"
    "strings"
)

// Config represents parsed configuration values
type Config map[string]string

// ParseConfigFile reads a configuration file and returns a map of settings
func ParseConfigFile(path string) (config Config, lineCount int, err error) {
    // Initialize return value
    config = make(Config)
    
    // Open file
    file, err := os.Open(path)
    if err != nil {
        err = fmt.Errorf("failed to open config file: %w", err)
        return // Early return with zero config, zero lineCount, and error
    }
    defer file.Close()
    
    // Read line by line
    scanner := bufio.NewScanner(file)
    lineNum := 0
    
    for scanner.Scan() {
        lineNum++
        line := strings.TrimSpace(scanner.Text())
        
        // Skip empty lines and comments
        if line == "" || strings.HasPrefix(line, "#") {
            continue
        }
        
        // Parse key=value pairs
        parts := strings.SplitN(line, "=", 2)
        if len(parts) != 2 {
            err = fmt.Errorf("invalid format at line %d: %s", lineNum, line)
            return // Return with partial config, lineCount, and error
        }
        
        key := strings.TrimSpace(parts[0])
        value := strings.TrimSpace(parts[1])
        
        // Validate key
        if key == "" {
            err = fmt.Errorf("empty key at line %d", lineNum)
            return // Early return
        }
        
        // Save to config map
        config[key] = value
    }
    
    // Check for scanner errors
    if err = scanner.Err(); err != nil {
        err = fmt.Errorf("error reading config file: %w", err)
        return // Return with partial config and error
    }
    
    // Update line count for return
    lineCount = lineNum
    return // Return success
}

// Using the parser
func LoadApplicationConfig(configPath string) Config {
    config, lines, err := ParseConfigFile(configPath)
    if err != nil {
        fmt.Printf("Configuration error: %v\n", err)
        return nil
    }
    
    fmt.Printf("Loaded %d configuration settings from %d lines\n", 
               len(config), lines)
    return config
}
```

**Common Pitfalls:**
- Using naked returns in large functions where it's hard to track the return values
- Modifying named return values but forgetting they'll be returned by a naked return
- Assuming named return variables override the need to specify values in return statements
- Inconsistent use of naked returns across functions
- Forgetting that named returns are initialized to zero values

**Confusion Questions:**

1. **Q: When should I use a naked return versus explicitly returning values?**
   
   A: Use naked returns (`return` without values) when:
   
   - The function is short and the flow is simple
   - You're using early returns for error handling
   - The named return values clearly communicate intent
   
   Use explicit returns (`return x, y, z`) when:
   
   - The function is long or complex
   - You want to be explicit about what's being returned
   - The return statement is far from the function declaration
   
   ```go
   // Good use of naked return: short function with clear intent
   func split(sum int) (x, y int) {
       x = sum * 4 / 9
       y = sum - x
       return
   }
   
   // Better with explicit return: longer function
   func processData(data []byte) (result string, err error) {
       // Many lines of processing...
       // ...
       // Many more lines...
       
       // Explicit return for clarity in a long function
       return result, err
   }
   ```
   
   The key is consistency and clarity - if using named returns, be consistent about whether you use naked returns or explicit returns within a function.

2. **Q: Are named return values always initialized to their zero values?**
   
   A: Yes, named return values are always initialized to their zero values at the beginning of the function execution:
   
   - `int`, `float64`, etc.: initialized to `0`
   - `string`: initialized to `""`
   - `bool`: initialized to `false`
   - pointers, slices, maps, channels, functions: initialized to `nil`
   - structs: initialized with zero values for all fields
   
   ```go
   func getData() (count int, names []string, err error) {
       // count is already 0
       // names is already nil
       // err is already nil
       
       if loadFailed {
           err = errors.New("loading failed")
           return // Returns 0, nil, and the error
       }
       
       // Must explicitly set values if you want non-zero returns
       count = 42
       names = []string{"Alice", "Bob"}
       return // Returns 42, ["Alice", "Bob"], nil
   }
   ```
   
   This initialization behavior can be helpful for error handling patterns but can lead to subtle bugs if you forget it. Always be explicit when you want to return specific non-zero values.

3. **Q: How do named return values affect function documentation and readability?**
   
   A: Named return values serve as documentation by describing the semantic meaning of what's being returned:
   
   ```go
   // Without named returns - less clear what each return represents
   func parseVersion(s string) (string, int, int, int, error)
   
   // With named returns - clearer what each value represents
   func parseVersion(s string) (version string, major, minor, patch int, err error)
   ```
   
   However, there are readability trade-offs:
   
   **Benefits:**
   - Self-documents the purpose of each return value
   - Makes error handling with early returns cleaner
   - Clarifies the meaning of returned values in function signatures
   
   **Drawbacks:**
   - Can lead to confusing code if the named variables are modified throughout a large function
   - Naked returns in long functions make it harder to see what's being returned
   - Can create longer function signatures that wrap in editors
   
   Best practices:
   - Use named returns for public API functions to enhance documentation
   - Keep functions with naked returns relatively short
   - Be consistent with return style within a function
   - Consider using named returns primarily for error handling patterns
   
   Tools like `godoc` will show the named return values in the generated documentation, improving API clarity.

### 5. Variadic Functions

**Concise Explanation:**
Variadic functions in Go can accept a variable number of arguments of the same type. These are specified using an ellipsis (`...`) before the type in the function signature. Inside the function, the variadic parameter is treated as a slice. This enables flexible function calls with any number of arguments, from zero to many.

**Where to Use:**
- Functions that need to accept a varying number of inputs
- Helper functions like string formatting
- Collection or aggregation operations
- Wrappers around existing functions
- Builder patterns and configuration options

**Code Snippet:**
```go
// Basic variadic function
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}

// Variadic function with regular parameters
func join(separator string, parts ...string) string {
    return strings.Join(parts, separator)
}

// Variadic function that processes each argument differently
func process(items ...interface{}) []string {
    result := make([]string, 0, len(items))
    
    for _, item := range items {
        switch v := item.(type) {
        case string:
            result = append(result, v)
        case int:
            result = append(result, fmt.Sprintf("int: %d", v))
        case bool:
            result = append(result, fmt.Sprintf("bool: %t", v))
        default:
            result = append(result, fmt.Sprintf("unknown: %v", v))
        }
    }
    
    return result
}

// Using variadic functions
func main() {
    // Variable number of arguments
    fmt.Println(sum())         // 0 arguments
    fmt.Println(sum(1))        // 1 argument
    fmt.Println(sum(1, 2, 3))  // 3 arguments
    
    // With regular parameter
    fmt.Println(join("-", "a", "b", "c"))  // "a-b-c"
    
    // Passing a slice to a variadic function
    numbers := []int{4, 5, 6}
    fmt.Println(sum(numbers...))  // Spread operator
    
    // Variadic function with mixed types
    results := process("hello", 42, true, 3.14)
    for _, r := range results {
        fmt.Println(r)
    }
}
```

**Real-World Example:**
A logging system with severity levels and variable formatting:

```go
package logger

import (
    "fmt"
    "os"
    "time"
)

// LogLevel represents logging severity
type LogLevel int

const (
    DEBUG LogLevel = iota
    INFO
    WARNING
    ERROR
    FATAL
)

// Logger handles application logging
type Logger struct {
    MinLevel LogLevel
    Output   *os.File
    Prefix   string
}

// NewLogger creates a configured logger
func NewLogger(minLevel LogLevel) *Logger {
    return &Logger{
        MinLevel: minLevel,
        Output:   os.Stdout,
        Prefix:   "",
    }
}

// Log formats and outputs a log message if level is sufficient
func (l *Logger) Log(level LogLevel, format string, args ...interface{}) {
    // Skip if below minimum level
    if level < l.MinLevel {
        return
    }
    
    // Get level name
    levelName := "UNKNOWN"
    switch level {
    case DEBUG:
        levelName = "DEBUG"
    case INFO:
        levelName = "INFO"
    case WARNING:
        levelName = "WARNING"
    case ERROR:
        levelName = "ERROR"
    case FATAL:
        levelName = "FATAL"
    }
    
    // Format message with variable args
    message := format
    if len(args) > 0 {
        message = fmt.Sprintf(format, args...)
    }
    
    // Construct log line
    timestamp := time.Now().Format("2006-01-02 15:04:05")
    logLine := fmt.Sprintf("%s [%s] %s%s\n", 
                         timestamp, levelName, l.Prefix, message)
    
    // Output log
    fmt.Fprint(l.Output, logLine)
    
    // Exit on fatal errors
    if level == FATAL {
        os.Exit(1)
    }
}

// Convenience methods for different log levels
func (l *Logger) Debug(format string, args ...interface{}) {
    l.Log(DEBUG, format, args...)
}

func (l *Logger) Info(format string, args ...interface{}) {
    l.Log(INFO, format, args...)
}

func (l *Logger) Warning(format string, args ...interface{}) {
    l.Log(WARNING, format, args...)
}

func (l *Logger) Error(format string, args ...interface{}) {
    l.Log(ERROR, format, args...)
}

func (l *Logger) Fatal(format string, args ...interface{}) {
    l.Log(FATAL, format, args...)
}

// Using the logger
func Example() {
    log := NewLogger(INFO)
    log.Prefix = "[MyApp] "
    
    // These won't print (below minimum level)
    log.Debug("Debug message: %s", "detailed info")
    
    // These will print
    log.Info("Application started")
    log.Info("Processing %d items", 42)
    log.Warning("System resources low: %d%% memory available", 15)
    
    if err := processImportantTask(); err != nil {
        log.Error("Failed to process task: %v", err)
    }
    
    // This would terminate the program
    // log.Fatal("Unrecoverable error: database connection lost")
}
```

**Common Pitfalls:**
- Forgetting that the variadic parameter must be the last parameter
- Not using the spread operator (`...`) when passing a slice to a variadic function
- Creating variadic functions that are hard to use due to ambiguous behavior
- Performance issues when creating large temporary slices
- Type safety issues with `interface{}` variadic functions
- Not considering zero-argument cases

**Confusion Questions:**

1. **Q: How do I pass a slice to a variadic function?**
   
   A: Use the spread operator (`...`) after the slice to expand it into individual arguments:
   
   ```go
   func sum(nums ...int) int {
       total := 0
       for _, n := range nums {
           total += n
       }
       return total
   }
   
   func main() {
       // Direct arguments
       sum(1, 2, 3)  // Passing individual arguments
       
       // Passing a slice
       numbers := []int{4, 5, 6}
       sum(numbers...) // Spread operator expands the slice
       
       // Won't compile - type mismatch
       // sum(numbers)  // Error: cannot use numbers (type []int) as type int
   }
   ```
   
   Without the spread operator, you would be trying to pass a slice as a single int argument, which causes a type error. The spread operator tells the compiler to expand the slice into individual arguments matching the variadic parameter.

2. **Q: When should I use variadic functions versus accepting a slice parameter?**
   
   A: Choose variadic functions when:
   
   - You want to provide a more convenient API for callers
   - The function makes sense with any number of arguments, including zero
   - You're creating a wrapper or utility function
   - The number of arguments is typically small
   
   ```go
   // Variadic - more convenient for common cases
   func PrintNames(names ...string) {
       for _, name := range names {
           fmt.Println(name)
       }
   }
   
   // Usage
   PrintNames("Alice")
   PrintNames("Bob", "Charlie", "Dave")
   ```
   
   Choose a slice parameter when:
   
   - The arguments naturally form a collection
   - The number of arguments is typically large
   - You need to pre-process the collection as a whole
   - The collection is generated or processed elsewhere
   
   ```go
   // Slice parameter - better for collections
   func CalculateStatistics(values []float64) (min, max, avg float64) {
       // Process collection
   }
   
   // Usage
   measurements := loadMeasurements()  // Returns []float64
   min, max, avg := CalculateStatistics(measurements)
   ```
   
   In many cases, you can provide both options for flexibility:
   
   ```go
   // Slice version
   func Join(elems []string, sep string) string {
       return strings.Join(elems, sep)
   }
   
   // Variadic convenience wrapper
   func JoinArgs(sep string, elems ...string) string {
       return Join(elems, sep)
   }
   ```
   
   This approach is used in the standard library, like `strings.Join` and `fmt.Sprint`.

3. **Q: How do variadic functions work with different types or empty argument lists?**
   
   A: Variadic functions handle different scenarios as follows:
   
   - **Empty argument lists**: The variadic parameter becomes an empty slice
     ```go
     func example(args ...int) {
         fmt.Println(len(args)) // 0 if no arguments provided
         fmt.Println(args == nil) // false - it's an empty slice, not nil
     }
     ```
   
   - **Different types**: Use `interface{}` for mixed types, then type assert or switch
     ```go
     func printAny(args ...interface{}) {
         for _, arg := range args {
             switch v := arg.(type) {
             case int:
                 fmt.Println("Int:", v)
             case string:
                 fmt.Println("String:", v)
             default:
                 fmt.Printf("Unknown type: %T\n", v)
             }
         }
     }
     
     // Usage
     printAny(1, "hello", true, 3.14)
     ```
   
   - **Strongly-typed variadics**: For type safety, you can define type constraints
     ```go
     type Number interface {
         int | int64 | float64
     }
     
     // Go 1.18+ with generics
     func Sum[T Number](nums ...T) T {
         var total T
         for _, n := range nums {
             total += n
         }
         return total
     }
     ```
   
   A common pattern in Go is the "options" pattern, where a variadic parameter accepts option functions:
   
   ```go
   type Server struct {
       port int
       timeout time.Duration
   }
   
   type ServerOption func(*Server)
   
   func WithPort(port int) ServerOption {
       return func(s *Server) {
           s.port = port
       }
   }
   
   func WithTimeout(timeout time.Duration) ServerOption {
       return func(s *Server) {
           s.timeout = timeout
       }
   }
   
   func NewServer(options ...ServerOption) *Server {
       // Default values
       s := &Server{
           port:    8080,
           timeout: time.Minute,
       }
       
       // Apply all options
       for _, option := range options {
           option(s)
       }
       
       return s
   }
   
   // Usage
   server := NewServer() // Use defaults
   server = NewServer(WithPort(3000), WithTimeout(30*time.Second))
   ```
   
   This pattern provides a flexible and type-safe way to handle optional configuration.

### 6. Anonymous Functions and Closures

**Concise Explanation:**
Anonymous functions in Go are functions defined without a name, often used inline where a function is needed briefly. Closures are anonymous functions that can access and modify variables from the surrounding scope. These features enable powerful functional programming patterns like callbacks, decorators, and deferred operations.

**Where to Use:**
- One-off operations that don't need a named function
- Callback functions for APIs
- Concurrent operations using goroutines
- Event handlers
- Deferred cleanup operations
- Function factories that generate specialized functions

**Code Snippet:**
```go
// Basic anonymous function
func main() {
    // Anonymous function defined and called immediately
    func() {
        fmt.Println("Hello from anonymous function")
    }()
    
    // Anonymous function with parameters
    func(name string) {
        fmt.Println("Hello,", name)
    }("Gopher")
    
    // Assigned to a variable
    greet := func(name string) string {
        return "Welcome, " + name
    }
    
    // Use the function variable
    message := greet("Alice")
    fmt.Println(message)
    
    // Closure accessing outer variables
    counter := 0
    increment := func() int {
        counter++  // Accessing counter from outer scope
        return counter
    }
    
    fmt.Println(increment()) // 1
    fmt.Println(increment()) // 2
    fmt.Println(counter)     // 2 (modified by closure)
    
    // Function returning a closure
    adder := createAdder(10)
    fmt.Println(adder(5))  // 15
    fmt.Println(adder(7))  // 17
}

// Function that returns a closure
func createAdder(base int) func(int) int {
    return func(x int) int {
        return base + x  // base is captured from outer scope
    }
}
```

**Real-World Example:**
A middleware pattern for HTTP request handling:

```go
package middleware

import (
    "log"
    "net/http"
    "time"
)

// Middleware represents an HTTP handler middleware
type Middleware func(http.HandlerFunc) http.HandlerFunc

// Chain combines multiple middleware into a single middleware
func Chain(middlewares ...Middleware) Middleware {
    return func(final http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            // Chain middleware in order, with final handler at the end
            last := final
            for i := len(middlewares) - 1; i >= 0; i-- {
                last = middlewares[i](last)
            }
            last(w, r)
        }
    }
}

// Logger is middleware that logs request details
func Logger() Middleware {
    return func(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            // Call the next handler
            next(w, r)
            
            // Log after request is processed
            log.Printf(
                "%s %s %s",
                r.Method,
                r.RequestURI,
                time.Since(start),
            )
        }
    }
}

// Auth checks if a user is authenticated
func Auth(authToken string) Middleware {
    return func(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            // Check auth token from request
            token := r.Header.Get("Authorization")
            
            if token != authToken {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }
            
            // If authenticated, proceed to next handler
            next(w, r)
        }
    }
}

// Recover middleware catches panics and returns 500 error
func Recover() Middleware {
    return func(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            defer func() {
                if err := recover(); err != nil {
                    log.Printf("Panic: %v", err)
                    http.Error(w, "Internal Server Error", http.StatusInternalServerError)
                }
            }()
            
            next(w, r)
        }
    }
}

// Example usage
func SetupAPI() {
    // Create middleware chain
    middleware := Chain(
        Logger(),
        Recover(),
        Auth("secret-token"),
    )
    
    // Apply middleware to handlers
    http.HandleFunc("/api/data", middleware(handleGetData))
    http.HandleFunc("/api/users", middleware(handleGetUsers))
    
    // Start server
    log.Println("Starting server on :8080")
    http.ListenAndServe(":8080", nil)
}

// Handler functions
func handleGetData(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("This is the data"))
}

func handleGetUsers(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("User list"))
}
```

**Common Pitfalls:**
- Accidentally capturing loop variables in goroutines
- Memory leaks from closures that capture unnecessary large objects
- Modifying shared variables from multiple goroutines without synchronization
- Creating complex nested anonymous functions that are hard to read
- Too much indentation from many nested closures
- Forgetting that goroutines with closures may outlive their caller's scope

**Confusion Questions:**

1. **Q: Why does capturing loop variables in goroutines or closures often lead to unexpected behavior?**
   
   A: This is one of the most common closure-related bugs in Go. Loop variables in Go are reused across iterations, so closures capture the variable itself, not its value at the time of capture:
   
   ```go
   // Bug: All goroutines will likely print the same number
   for i := 0; i < 5; i++ {
       go func() {
           fmt.Println(i) // Captures i, which changes with each iteration
       }()
   }
   // By the time goroutines execute, i is likely 5
   
   // Solution 1: Pass the current value as a parameter
   for i := 0; i < 5; i++ {
       go func(val int) {
           fmt.Println(val) // Uses the value passed in
       }(i) // Pass current value of i
   }
   
   // Solution 2: Create a new variable in each iteration
   for i := 0; i < 5; i++ {
       i := i // Create a new i variable scoped to this iteration
       go func() {
           fmt.Println(i) // Captures the iteration-scoped variable
       }()
   }
   ```
   
   This issue is particularly important with goroutines because the loop may finish before goroutines execute, leading to all goroutines seeing the final loop variable value.

2. **Q: How are closures different from regular functions, and what's happening under the hood?**
   
   A: Closures are special because they capture and "close over" variables from their surrounding lexical scope:
   
   - **Regular functions**: Can only access their parameters and local variables.
   - **Closures**: Can access parameters, local variables, AND variables from the enclosing scope.
   
   Under the hood, Go implements closures by creating a hidden struct that stores references to all captured variables. The function portion of the closure becomes a method on this struct:
   
   ```go
   // This code:
   counter := 0
   increment := func() int {
       counter++
       return counter
   }
   
   // Conceptually becomes something like:
   type closureEnvironment struct {
       counter *int
   }
   
   func (env *closureEnvironment) incrementFn() int {
       *env.counter++
       return *env.counter
   }
   
   // Creating:
   counterVal := 0
   env := &closureEnvironment{counter: &counterVal}
   increment := env.incrementFn
   ```
   
   This is why:
   - Multiple closures capturing the same variable all see each other's changes
   - Changes to captured variables persist between function calls
   - Closures extend the lifetime of captured variables until all referencing closures are garbage-collected

3. **Q: What are practical use cases for function literals (anonymous functions) in Go?**
   
   A: Anonymous functions have many practical applications in Go:
   
   1. **Goroutines for concurrent operations**:
      ```go
      // Start a background operation
      go func() {
          // Process data asynchronously
          processData(largeDataSet)
          log.Println("Processing complete")
      }()
      ```
   
   2. **Deferred cleanup operations**:
      ```go
      file, err := os.Open(filename)
      if err != nil {
          return err
      }
      defer func() {
          file.Close()
          log.Printf("Closed file: %s", filename)
      }()
      ```
   
   3. **Function factories (creating specialized functions)**:
      ```go
      func newRateLimiter(maxRequests int, interval time.Duration) func() bool {
          lastRequest := time.Now()
          requestCount := 0
          
          return func() bool {
              now := time.Now()
              // Reset counter after interval
              if now.Sub(lastRequest) > interval {
                  lastRequest = now
                  requestCount = 0
              }
              
              // Check if under limit
              if requestCount < maxRequests {
                  requestCount++
                  return true
              }
              return false
          }
      }
      
      // Usage
      apiLimiter := newRateLimiter(100, time.Minute)
      if apiLimiter() {
          // Make API call
      }
      ```
   
   4. **Sorting with custom comparators**:
      ```go
      sort.Slice(people, func(i, j int) bool {
          return people[i].Age < people[j].Age
      })
      ```
   
   5. **Event handlers and callbacks**:
      ```go
      button.OnClick(func() {
          log.Println("Button clicked")
          updateUI()
      })
      ```
   
   6. **Middleware patterns** (as shown in the real-world example)
   
   7. **One-time initialization logic**:
      ```go
      var config = func() Configuration {
          // Complex initialization
          c := loadConfigFile()
          c.Normalize()
          c.Validate()
          return c
      }()
      ```
   
   These patterns leverage the conciseness of anonymous functions and the power of closures to create elegant, functional solutions.

### 7. Defer, Panic, and Recover

**Concise Explanation:**
Go provides three mechanisms for handling exceptional situations: `defer` schedules functions to be executed when the current function returns, `panic` disrupts normal execution when something impossible has happened, and `recover` regains control after a panic. Together, they provide a more controlled alternative to traditional exception handling.

**Where to Use:**
- **Defer**: Resource cleanup, file closures, mutex unlocks, timing operations
- **Panic**: Unrecoverable errors, impossible conditions, initialization failures
- **Recover**: Converting panics into normal error returns, protecting program stability

**Code Snippet:**
```go
// Basic defer usage
func readFile(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // Will be called when function returns
    
    data, err := io.ReadAll(f)
    if err != nil {
        return "", err
    }
    
    return string(data), nil
}

// Multiple defers (executed in LIFO order)
func processData() {
    defer fmt.Println("First defer")
    defer fmt.Println("Second defer")
    defer fmt.Println("Third defer")  // Executed first
    
    fmt.Println("Processing data...")
}

// Defer capturing variables
func example() {
    i := 1
    defer fmt.Println("Deferred print:", i)  // Captures i=1
    i = 2
    fmt.Println("Regular print:", i)  // Prints i=2
}

// Basic panic example
func divide(a, b int) int {
    if b == 0 {
        panic("division by zero")
    }
    return a / b
}

// Recover example
func safeOperation() (err error) {
    defer func() {
        if r := recover(); r != nil {
            // Convert panic to error
            err = fmt.Errorf("recovered from panic: %v", r)
        }
    }()
    
    // Operation that might panic
    performRiskyOperation()
    return nil
}

// Combining all three
func complexOperation() (result int, err error) {
    // Setup recovery
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("operation failed: %v", r)
        }
    }()
    
    // Resource acquisition and cleanup
    resource, err := acquireResource()
    if err != nil {
        return 0, err
    }
    defer releaseResource(resource)
    
    // Another resource with cleanup
    conn, err := openConnection()
    if err != nil {
        return 0, err
    }
    defer conn.Close()
    
    // Operations that might panic
    result = performCalculation(resource, conn)
    return result, nil
}
```

**Real-World Example:**
A database transaction manager using defer, panic, and recover:

```go
package db

import (
    "database/sql"
    "fmt"
    "time"
)

// DB represents a database connection pool
type DB struct {
    *sql.DB
}

// Transaction represents a database transaction
type Transaction struct {
    tx     *sql.Tx
    db     *DB
    done   bool
    start  time.Time
}

// Begin starts a new database transaction
func (db *DB) Begin() (*Transaction, error) {
    tx, err := db.DB.Begin()
    if err != nil {
        return nil, fmt.Errorf("failed to begin transaction: %w", err)
    }
    
    return &Transaction{
        tx:    tx,
        db:    db,
        start: time.Now(),
    }, nil
}

// Exec executes a query within the transaction
func (t *Transaction) Exec(query string, args ...interface{}) (sql.Result, error) {
    if t.done {
        return nil, fmt.Errorf("transaction already committed or rolled back")
    }
    return t.tx.Exec(query, args...)
}

// Query executes a query that returns rows
func (t *Transaction) Query(query string, args ...interface{}) (*sql.Rows, error) {
    if t.done {
        return nil, fmt.Errorf("transaction already committed or rolled back")
    }
    return t.tx.Query(query, args...)
}

// Commit commits the transaction
func (t *Transaction) Commit() error {
    if t.done {
        return fmt.Errorf("transaction already committed or rolled back")
    }
    err := t.tx.Commit()
    t.done = true
    duration := time.Since(t.start)
    
    if err != nil {
        return fmt.Errorf("commit failed: %w", err)
    }
    
    if duration > 500*time.Millisecond {
        // Log slow transaction
        fmt.Printf("SLOW TRANSACTION: %v\n", duration)
    }
    
    return nil
}

// Rollback rolls back the transaction
func (t *Transaction) Rollback() error {
    if t.done {
        return nil // Already committed or rolled back
    }
    err := t.tx.Rollback()
    t.done = true
    if err != nil {
        return fmt.Errorf("rollback failed: %w", err)
    }
    return nil
}

// WithTransaction executes a function within a transaction
// Handles commit/rollback automatically
func (db *DB) WithTransaction(fn func(*Transaction) error) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    
    // Ensure rollback happens if panic or error
    defer func() {
        if !tx.done {
            tx.Rollback()
        }
    }()
    
    // Setup panic recovery
    panicked := true
    defer func() {
        if panicked {
            // Convert panic to error
            r := recover()
            if r != nil {
                err = fmt.Errorf("panic in transaction: %v", r)
            }
        }
    }()
    
    // Run the user function
    err = fn(tx)
    
    // No panic occurred
    panicked = false
    
    // Handle result
    if err != nil {
        tx.Rollback()
        return err
    }
    
    return tx.Commit()
}

// Example usage
func main() {
    db := &DB{/* initialized database connection */}
    
    err := db.WithTransaction(func(tx *Transaction) error {
        // First operation
        _, err := tx.Exec("INSERT INTO users (name, email) VALUES (?, ?)", 
                         "Alice", "alice@example.com")
        if err != nil {
            return err
        }
        
        // Second operation that might panic
        _, err = tx.Exec("UPDATE accounts SET balance = balance + ? WHERE user_id = ?", 
                        100, getUserID("Alice"))
        if err != nil {
            return err
        }
        
        return nil
    })
    
    if err != nil {
        fmt.Printf("Transaction failed: %v\n", err)
    } else {
        fmt.Println("Transaction committed successfully")
    }
}
```

**Common Pitfalls:**
- Forgetting to defer cleanup operations immediately after resource acquisition
- Assuming deferred functions execute in the same order they're defined (they execute in reverse order)
- Capturing loop variables in deferred function literals
- Changing return values after a defer statement has been executed
- Using recover outside of a deferred function (it only works in deferred functions)
- Not handling the returned value from recover()
- Overusing panic for normal error conditions instead of returning errors
- Forgetting that deferred functions run after return values are evaluated but before the function returns

**Confusion Questions:**

1. **Q: In what order are multiple deferred functions executed, and why does this matter?**
   
   A: Deferred functions are executed in Last In, First Out (LIFO) order - the opposite order in which they were deferred. This is crucial when dealing with related resources that need to be cleaned up in a specific order:
   
   ```go
   func processNestedResources() error {
       // Open outer resource
       outer, err := openOuterResource()
       if err != nil {
           return err
       }
       defer outer.Close()  // Executed second (last)
       
       // Open inner resource
       inner, err := outer.OpenInnerResource()
       if err != nil {
           return err
       }
       defer inner.Close()  // Executed first
       
       // Work with resources
       return nil
   }
   ```
   
   This LIFO ordering ensures that dependent resources are cleaned up in the correct order: inner resources are closed before their outer containers. This matters when:
   
   - Working with nested resources (files within archives, sub-transactions)
   - Resources have dependencies (database connections and their prepared statements)
   - Multiple locks need to be released in the reverse order of acquisition to prevent deadlocks
   
   If defers executed in the order defined, many resource cleanup patterns would be impossible to implement correctly.

2. **Q: When should I use panic/recover versus regular error handling in Go?**
   
   A: In Go, there's a clear distinction between when to use each approach:
   
   **Use regular error handling (returning errors) when:**
   - The error is expected or can be reasonably anticipated
   - The caller might want to handle different error cases differently
   - The function can't complete its task but the program can still continue
   - You're writing a library or package for others to use
   
   ```go
   func readConfig(path string) (*Config, error) {
       file, err := os.Open(path)
       if err != nil {
           return nil, fmt.Errorf("failed to open config: %w", err)
       }
       defer file.Close()
       // ...
   }
   ```
   
   **Use panic/recover when:**
   - Something impossible has happened (invariant violation)
   - An unrecoverable error occurs during initialization
   - Programming errors that should never happen (like array bounds violations)
   - Simplifying code in rare cases where all error paths would lead to program termination
   - Converting a third-party library's panics into errors at API boundaries
   
   ```go
   func MustCompileRegex(pattern string) *regexp.Regexp {
       regex, err := regexp.Compile(pattern)
       if err != nil {
           panic(fmt.Sprintf("invalid regex pattern %q: %v", pattern, err))
       }
       return regex
   }
   
   // For API boundaries, convert panics to errors
   func SafeHandler(fn http.HandlerFunc) http.HandlerFunc {
       return func(w http.ResponseWriter, r *http.Request) {
           defer func() {
               if err := recover(); err != nil {
                   log.Printf("panic in handler: %v", err)
                   http.Error(w, "Internal Server Error", 500)
               }
           }()
           fn(w, r)
       }
   }
   ```
   
   In general, favor explicit error returns over panic/recover in most situations, as it makes code flow more obvious and error handling more consistent.

3. **Q: How do deferred function arguments get evaluated, and what are the implications?**
   
   A: Arguments to deferred function calls are evaluated immediately when the `defer` statement is executed, not when the deferred function is actually run:
   
   ```go
   func example() {
       i := 1
       defer fmt.Println("Value:", i)  // Captures i=1 now
       i = 2
   }  // Prints "Value: 1" when exiting
   ```
   
   This behavior has important implications:
   
   1. **Value capturing**: If you want to use the final value of a variable, use a closure instead:
      ```go
      i := 1
      defer func() { fmt.Println("Value:", i) }()  // Uses i's value at execution time
      i = 2
      // Prints "Value: 2" when executed
      ```
   
   2. **Function return values**: If you need to access or modify the function's return values, use named return values and a closure:
      ```go
      func readFile() (err error) {
          // This will modify the returned err value if a panic occurs
          defer func() {
              if r := recover(); r != nil {
                  err = fmt.Errorf("panic while reading: %v", r)
              }
          }()
          // ...
      }
      ```
   
   3. **Method receivers**: When deferring a method call, the receiver is also evaluated immediately:
      ```go
      p := &Person{Name: "Alice"}
      defer p.PrintName()  // Will use "Alice" even if p changes
      p.Name = "Bob"
      // Will still print "Alice" when deferred function executes
      ```
   
   4. **Loop variables**: Be careful with loop variables in defers, especially in loops:
      ```go
      // INCORRECT - all defers see the same file variable
      for _, filename := range filenames {
          file, err := os.Open(filename)
          defer file.Close()  // Will close the last opened file multiple times!
          // ...
      }
      
      // CORRECT - use a closure to capture the current file
      for _, filename := range filenames {
          file, err := os.Open(filename)
          defer func(f *os.File) {
              f.Close()
          }(file)  // Pass current file as argument
          // ...
      }
      ```
   
   Understanding this evaluation timing is crucial for writing correct deferred operations, particularly for resource cleanup and error handling.

### 8. Creating and Organizing Packages

**Concise Explanation:**
Packages in Go are the basic unit of code organization and reuse. Each Go source file begins with a package declaration that identifies which package it belongs to. Packages provide a namespace for identifiers and control visibility through capitalization. Well-organized packages have a clear, focused purpose and follow Go conventions for structure and naming.

**Where to Use:**
- Organizing related functionality into cohesive units
- Creating reusable libraries
- Controlling visibility and encapsulation of code
- Managing dependencies between different parts of a program
- Separating concerns in a large application

**Code Snippet:**
```go
// File: math/geometry.go
package math

import "math"

// Circle represents a geometric circle
type Circle struct {
    Radius float64    // Exported field (capitalized)
    area   float64    // Unexported field (lowercase)
}

// Area calculates the area of a circle (exported method)
func (c *Circle) Area() float64 {
    if c.area == 0 && c.Radius > 0 {
        c.area = math.Pi * c.Radius * c.Radius
    }
    return c.area
}

// Circumference calculates the circumference (exported)
func (c *Circle) Circumference() float64 {
    return 2 * math.Pi * c.Radius
}

// helper functions (unexported)
func radiansToDegrees(rad float64) float64 {
    return rad * (180 / math.Pi)
}

// NewCircle creates a new circle instance (exported constructor)
func NewCircle(radius float64) *Circle {
    return &Circle{
        Radius: radius,
    }
}

// File: main.go
package main

import (
    "fmt"
    "myapp/math"    // Import custom package
)

func main() {
    circle := math.NewCircle(5)
    
    // Access exported methods
    area := circle.Area()
    circumference := circle.Circumference()
    
    fmt.Printf("Circle with radius %.1f: Area=%.2f, Circumference=%.2f\n",
              circle.Radius, area, circumference)
    
    // Can't access unexported properties or functions
    // fmt.Println(circle.area)  // Compilation error
    // fmt.Println(math.radiansToDegrees(1.0))  // Compilation error
}
```

**Real-World Example:**
A well-structured logging package with levels and formatters:

```
myapp/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── config/
│   │   ├── config.go
│   │   └── loader.go
│   ├── handler/
│   │   ├── auth.go
│   │   └── user.go
│   └── storage/
│       ├── database.go
│       └── models.go
├── pkg/
│   └── logger/
│       ├── logger.go      // Main package interface
│       ├── level.go       // Log levels definition
│       ├── formatter.go   // Log formatting interface
│       ├── json.go        // JSON formatter implementation
│       └── console.go     // Console formatter implementation
└── go.mod
```

File contents for the logger package:

```go
// pkg/logger/logger.go
package logger

import (
    "io"
    "os"
    "sync"
)

// Logger represents a logging object with configurable output and format
type Logger struct {
    level     Level
    formatter Formatter
    output    io.Writer
    mu        sync.Mutex
}

// New creates a new logger with specified options
func New(options ...Option) *Logger {
    // Default configuration
    logger := &Logger{
        level:     InfoLevel,
        formatter: new(ConsoleFormatter),
        output:    os.Stdout,
    }
    
    // Apply provided options
    for _, option := range options {
        option(logger)
    }
    
    return logger
}

// Option defines a logger configuration option
type Option func(*Logger)

// WithLevel sets the logger's minimum level
func WithLevel(level Level) Option {
    return func(l *Logger) {
        l.level = level
    }
}

// WithFormatter sets the logger's formatter
func WithFormatter(formatter Formatter) Option {
    return func(l *Logger) {
        l.formatter = formatter
    }
}

// WithOutput sets the logger's output destination
func WithOutput(output io.Writer) Option {
    return func(l *Logger) {
        l.output = output
    }
}

// Log logs a message at the specified level
func (l *Logger) Log(level Level, message string, fields ...Field) {
    if level < l.level {
        return
    }
    
    entry := &Entry{
        Level:     level,
        Message:   message,
        Fields:    make(map[string]interface{}),
        Timestamp: time.Now(),
    }
    
    // Add fields to entry
    for _, field := range fields {
        field.AddTo(entry.Fields)
    }
    
    // Format and write log entry
    l.mu.Lock()
    defer l.mu.Unlock()
    
    formatted := l.formatter.Format(entry)
    l.output.Write(formatted)
}

// Convenience methods
func (l *Logger) Debug(message string, fields ...Field) {
    l.Log(DebugLevel, message, fields...)
}

func (l *Logger) Info(message string, fields ...Field) {
    l.Log(InfoLevel, message, fields...)
}

func (l *Logger) Warn(message string, fields ...Field) {
    l.Log(WarnLevel, message, fields...)
}

func (l *Logger) Error(message string, fields ...Field) {
    l.Log(ErrorLevel, message, fields...)
}

// pkg/logger/level.go
package logger

// Level represents a logging level
type Level int

const (
    // Debug level for detailed troubleshooting
    DebugLevel Level = iota
    // Info level for general operational information
    InfoLevel
    // Warn level for warning conditions
    WarnLevel
    // Error level for error conditions
    ErrorLevel
    // Fatal level for critical errors that prevent operation
    FatalLevel
)

// String returns the string representation of the level
func (l Level) String() string {
    switch l {
    case DebugLevel:
        return "DEBUG"
    case InfoLevel:
        return "INFO"
    case WarnLevel:
        return "WARN"
    case ErrorLevel:
        return "ERROR"
    case FatalLevel:
        return "FATAL"
    default:
        return "UNKNOWN"
    }
}

// pkg/logger/formatter.go
package logger

import "time"

// Entry represents a log entry with metadata
type Entry struct {
    Level     Level
    Message   string
    Fields    map[string]interface{}
    Timestamp time.Time
}

// Formatter defines the interface for log formatters
type Formatter interface {
    Format(entry *Entry) []byte
}

// pkg/logger/console.go
package logger

import (
    "fmt"
    "strings"
    "time"
)

// ConsoleFormatter formats log entries for console output
type ConsoleFormatter struct {
    // Configuration options could be added here
    TimeFormat string
    Colors     bool
}

// Format implements the Formatter interface
func (f *ConsoleFormatter) Format(entry *Entry) []byte {
    // Set default time format if empty
    timeFormat := f.TimeFormat
    if timeFormat == "" {
        timeFormat = time.RFC3339
    }
    
    // Format timestamp
    timestamp := entry.Timestamp.Format(timeFormat)
    
    // Format level
    level := entry.Level.String()
    
    // Format fields
    var fields string
    if len(entry.Fields) > 0 {
        parts := make([]string, 0, len(entry.Fields))
        for k, v := range entry.Fields {
            parts = append(parts, fmt.Sprintf("%s=%v", k, v))
        }
        fields = " " + strings.Join(parts, " ")
    }
    
    // Build log line
    logLine := fmt.Sprintf("[%s] %s: %s%s\n",
        timestamp, level, entry.Message, fields)
    
    return []byte(logLine)
}

// Example usage in application:
func main() {
    // Create a logger
    log := logger.New(
        logger.WithLevel(logger.DebugLevel),
        logger.WithFormatter(&logger.JSONFormatter{}),
    )
    
    // Log messages
    log.Info("Application started", logger.String("version", "1.0.0"))
    log.Debug("Debug information", logger.Int("connections", 5))
    
    // Create a child logger for a specific component
    dbLogger := logger.New(
        logger.WithLevel(log.GetLevel()),
        logger.WithFormatter(log.GetFormatter()),
        logger.WithOutput(log.GetOutput()),
        logger.WithPrefix("database"),
    )
    
    dbLogger.Info("Connected to database", logger.String("host", "localhost"))
}
```

**Common Pitfalls:**
- Circular dependencies between packages
- Packages that are too large with unrelated functionality
- Packages that are too small and granular, causing import complexity
- Incorrect package naming (not using lowercase, using underscores)
- Missing package documentation and examples
- Exposing internal implementation details that should be unexported
- Not using internal/ directory for private packages

**Confusion Questions:**

1. **Q: What's the difference between internal packages and unexported identifiers in Go?**
   
   A: Go provides two distinct mechanisms for controlling visibility:
   
   **Unexported Identifiers** (lowercase names):
   - Control visibility at the identifier level (functions, variables, fields)
   - Hidden from other packages but visible within the same package
   - Based purely on capitalization (lowercase = unexported, uppercase = exported)
   
   ```go
   package mypackage
   
   var PublicVar = "visible to other packages"  // Exported
   var privateVar = "visible only in this package"  // Unexported
   
   type User struct {
       Name string  // Exported field
       password string  // Unexported field
   }
   ```
   
   **Internal Packages** (using the `internal/` directory):
   - Control visibility at the package level
   - Only importable by the parent of the internal directory and its children
   - Based on filesystem structure
   
   ```
   myapp/
   ├── main.go             // Can import myapp/internal/database
   ├── api/
   │   └── handlers.go     // Can import myapp/internal/database
   └── internal/
       └── database/       // This package can only be imported by code within myapp
           └── db.go
   ```
   
   A package outside of `myapp/` cannot import `myapp/internal/database`.
   
   Use them together strategically:
   - Use unexported identifiers for implementation details within a package
   - Use internal packages for code that should be shared between multiple packages in your application but not exposed to external users
   - Public packages with exported identifiers form your public API

2. **Q: How should I organize and name packages in a Go project?**
   
   A: Go has several conventions for package organization and naming:
   
   **Package Naming:**
   - Use short, lowercase, single-word names without underscores or mixedCaps
   - Name should be descriptive and reflect the package's purpose
   - Avoid generic names like "util", "common", or "misc"
   - Package name should match the last element of the import path
   
   ```go
   package database   // Good: clear purpose
   package db         // Good: common abbreviation
   package sqldb      // Good: qualified purpose
   
   package SQL        // Bad: uppercase
   package data_base  // Bad: underscore
   package databaseUtil  // Bad: mixedCaps
   ```
   
   **Directory Structure:**
   - Standard layout for applications:
     ```
     myapp/
     ├── cmd/                  // Main applications
     │   ├── myapp/            // Main application binary
     │   │   └── main.go
     │   └── tools/            // Support tools
     │       └── migration/
     │           └── main.go
     ├── internal/             // Private packages
     │   ├── auth/             // Authentication logic
     │   └── storage/          // Storage logic
     ├── pkg/                  // Public library code
     │   └── validator/        // Can be imported by others
     └── api/                  // API definitions
         └── openapi.yaml
     ```
   
   - Standard layout for libraries:
     ```
     mylibrary/
     ├── library.go            // Core functionality
     ├── errors.go             // Error definitions
     ├── internal/             // Private implementation
     │   └── parser/
     ├── example_test.go       // Examples
     └── library_test.go       // Tests
     ```
   
   **Key Principles:**
   - Organize by dependency relationships, not by technical layers
   - Group by functional purpose (what, not how)
   - Keep related code close together
   - Use the internal directory for private shared code
   - Use small, focused packages with clear responsibilities

3. **Q: How can I manage common dependencies across packages without causing import cycles?**
   
   A: Import cycles are a common challenge when organizing Go code. Here are strategies to avoid them:
   
   1. **Extract shared types to a separate package**:
      ```
      myapp/
      ├── models/            // Shared data structures
      │   └── user.go        // User model used by multiple packages
      ├── auth/
      │   └── auth.go        // Imports models
      └── database/
          └── database.go    // Imports models, not auth
      ```
   
   2. **Use interfaces to break dependencies**:
      ```go
      // storage/storage.go
      package storage
      
      type UserStore interface {
          GetUser(id string) (*models.User, error)
          // ...
      }
      
      // auth/auth.go
      package auth
      
      type Service struct {
          store storage.UserStore  // Interface, not concrete type
      }
      ```
   
   3. **Use dependency injection**:
      ```go
      // auth/auth.go
      package auth
      
      func Authenticate(username, password string, store UserStore) (*User, error) {
          // No direct import of database package
      }
      
      // main.go
      package main
      
      func main() {
          db := database.New()
          user, err := auth.Authenticate("user", "pass", db)
      }
      ```
   
   4. **Follow directional dependencies** (domain-driven design):
      - Core domain models should not depend on infrastructure
      - Infrastructure depends on domain models
      - Use interfaces in the domain layer that are implemented by infrastructure
   
   5. **Use the internal/pkg pattern** for shared internal code:
      ```
      myapp/
      ├── internal/
      │   ├── auth/          // Uses internal/pkg/validation
      │   ├── api/           // Uses internal/pkg/validation
      │   └── pkg/           // Shared internal code
      │       └── validation/
      ```
   
   The key is to maintain a clear direction of dependencies and not let them become circular. Interfaces and dependency injection are particularly powerful tools for managing these relationships.

### 9. Import Statements and Visibility

**Concise Explanation:**
Go uses the `import` statement to access code from other packages. Names beginning with an uppercase letter (exported) are accessible outside the package, while lowercase names (unexported) are only accessible within the package. Import statements support different forms including aliasing and blank imports to satisfy different needs.

**Where to Use:**
- Accessing functionality from other packages
- Managing name conflicts with import aliases
- Triggering package initialization with blank imports
- Organizing imports for readability
- Encapsulating implementation details with unexported names

**Code Snippet:**
```go
// Basic imports
package main

import (
    "fmt"           // Standard library import
    "io/ioutil"     // Hierarchical package path
    "net/http"      // Another standard package
    
    "github.com/user/project/package"  // External package
    "myapp/internal/config"            // Internal package
)

// Import with alias (to avoid name conflicts)
import (
    "fmt"
    crand "crypto/rand"   // Aliased import
    "math/rand"           // Would conflict with crypto/rand without alias
)

// Blank import (runs init() but doesn't import names)
import (
    "fmt"
    _ "image/png"   // Register PNG format without importing names
    _ "image/jpeg"  // Register JPEG format without importing names
)

// Import with dot (avoid in most cases)
import (
    "fmt"
    . "math"  // Imports all exported names into current namespace
)

// Visibility examples
func main() {
    // Accessing exported names from packages
    fmt.Println("Hello, world!")           // Exported function
    client := &http.Client{}               // Exported type
    page, _ := ioutil.ReadFile("file.txt") // Exported function
    
    // Access exported and unexported items
    config := config.Load()            // Exported function from imported package
    fmt.Println(config.DatabaseURL)    // Exported field
    // fmt.Println(config.internal)    // Error: unexported field
    
    // Using aliased imports
    regularRandom := rand.Intn(100)
    cryptoRandom, _ := crand.Prime(crand.Reader, 64)
    
    // Using dot import
    x := Sqrt(64)  // Instead of math.Sqrt(64)
}
```

**Real-World Example:**
A web service with properly organized imports and visibility control:

```go
// cmd/api/main.go
package main

import (
    "context"
    "flag"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
    
    "myapp/config"
    "myapp/internal/auth"
    "myapp/internal/database"
    "myapp/internal/handler"
    "myapp/pkg/logger"
    
    "github.com/gorilla/mux"
    _ "github.com/lib/pq" // PostgreSQL driver registration
)

func main() {
    // Parse command line flags
    configPath := flag.String("config", "config.json", "Path to configuration file")
    flag.Parse()
    
    // Initialize logger
    log := logger.New(logger.InfoLevel)
    log.Info("Starting API server")
    
    // Load configuration
    cfg, err := config.Load(*configPath)
    if err != nil {
        log.Error("Failed to load configuration", logger.Error(err))
        os.Exit(1)
    }
    
    // Connect to database
    db, err := database.Connect(cfg.Database)
    if err != nil {
        log.Error("Failed to connect to database", logger.Error(err))
        os.Exit(1)
    }
    defer db.Close()
    
    // Initialize authentication
    authService := auth.NewService(
        auth.WithUserStore(db),
        auth.WithTokenDuration(cfg.Auth.TokenDuration),
        auth.WithSecretKey(cfg.Auth.SecretKey),
    )
    
    // Create router
    router := mux.NewRouter()
    
    // Register handlers
    handler.RegisterHealthcheck(router)
    handler.RegisterAuth(router, authService, log)
    handler.RegisterUsers(router, db, authService, log)
    
    // Create HTTP server
    server := &http.Server{
        Addr:    fmt.Sprintf(":%d", cfg.Server.Port),
        Handler: router,
    }
    
    // Start server in goroutine
    go func() {
        log.Info("Server listening", logger.Int("port", cfg.Server.Port))
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Error("Server error", logger.Error(err))
            os.Exit(1)
        }
    }()
    
    // Wait for interrupt signal
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
    <-sigChan
    
    log.Info("Shutting down server")
    
    // Create shutdown context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    // Gracefully shutdown the server
    if err := server.Shutdown(ctx); err != nil {
        log.Error("Server shutdown failed", logger.Error(err))
    }
    
    log.Info("Server stopped")
}

// internal/auth/service.go
package auth

import (
    "errors"
    "time"
    
    "github.com/golang-jwt/jwt/v4"
    "golang.org/x/crypto/bcrypt"
    
    "myapp/internal/database"
    "myapp/pkg/models"
)

// Unexported errors
var (
    errInvalidCredentials = errors.New("invalid credentials")
    errExpiredToken      = errors.New("token expired")
)

// Service handles authentication operations
type Service struct {
    userStore   UserStore
    secretKey   []byte
    tokenExpiry time.Duration
}

// UserStore defines the interface for user operations
type UserStore interface {
    GetUserByEmail(email string) (*models.User, error)
    // Other methods...
}

// Option configures the auth service
type Option func(*Service)

// WithUserStore sets the user store
func WithUserStore(store UserStore) Option {
    return func(s *Service) {
        s.userStore = store
    }
}

// WithSecretKey sets the JWT secret key
func WithSecretKey(key string) Option {
    return func(s *Service) {
        s.secretKey = []byte(key)
    }
}

// WithTokenDuration sets the token expiry duration
func WithTokenDuration(duration time.Duration) Option {
    return func(s *Service) {
        s.tokenExpiry = duration
    }
}

// NewService creates a new auth service with options
func NewService(options ...Option) *Service {
    // Default configuration
    service := &Service{
        tokenExpiry: 24 * time.Hour,
    }
    
    // Apply options
    for _, option := range options {
        option(service)
    }
    
    return service
}

// Login authenticates a user and returns a JWT token
func (s *Service) Login(email, password string) (string, error) {
    // Get user from store
    user, err := s.userStore.GetUserByEmail(email)
    if err != nil {
        if errors.Is(err, database.ErrNotFound) {
            return "", errInvalidCredentials
        }
        return "", err
    }
    
    // Verify password
    if err := bcrypt.CompareHashAndPassword(
        []byte(user.PasswordHash), []byte(password)); err != nil {
        return "", errInvalidCredentials
    }
    
    // Create token
    return s.createToken(user)
}

// Unexported helper method
func (s *Service) createToken(user *models.User) (string, error) {
    claims := jwt.MapClaims{
        "sub": user.ID,
        "email": user.Email,
        "role": user.Role,
        "exp": time.Now().Add(s.tokenExpiry).Unix(),
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(s.secretKey)
}

// Exported error check functions
func IsInvalidCredentials(err error) bool {
    return errors.Is(err, errInvalidCredentials)
}

func IsExpiredToken(err error) bool {
    return errors.Is(err, errExpiredToken)
}
```

**Common Pitfalls:**
- Circular dependencies between packages
- Importing unused packages (Go compiler rejects this)
- Overusing dot imports, making it unclear which functions come from which package
- Not grouping and organizing imports consistently
- Forgetting to use an import alias when necessary
- Exposing implementation details that should be unexported
- Importing a package just for its init() effects without using the blank identifier

**Confusion Questions:**

1. **Q: What's the difference between dot imports, blank imports, and regular imports?**
   
   A: Go offers three main import styles, each with specific uses:
   
   **Regular Import**:
   ```go
   import "fmt"
   ```
   - Imports the package with its package name as qualifier
   - Usage: `fmt.Println("Hello")`
   - This is the standard, recommended style for most imports
   
   **Blank Import** (with underscore):
   ```go
   import _ "image/png"
   ```
   - Executes the package's init() function but doesn't import any names
   - Usage: No direct usage of the package's exported identifiers
   - Used for side effects like driver registration, image format registration
   - Common in database drivers: `import _ "github.com/lib/pq"`
   
   **Dot Import**:
   ```go
   import . "math"
   ```
   - Imports all exported identifiers directly into current namespace
   - Usage: `Sqrt(4)` instead of `math.Sqrt(4)`
   - Generally avoided in production code because it:
     - Makes it unclear where functions come from
     - Can cause name conflicts
     - Reduces code clarity
   - May be acceptable in tests for conciseness
   
   **Import with Alias**:
   ```go
   import m "math"
   ```
   - Imports the package with a custom qualifier name
   - Usage: `m.Sqrt(4)` instead of `math.Sqrt(4)`
   - Used to:
     - Avoid name conflicts between packages
     - Use shorter names for frequently used packages
     - Clarify package purpose when the package name is ambiguous

2. **Q: How does Go determine what's exported and what's not?**
   
   A: Go uses a refreshingly simple rule to determine visibility: identifier capitalization.
   
   **Exported (Public) Names**:
   - Begin with an uppercase letter
   - Accessible from other packages
   - Examples: `fmt.Println`, `http.Request`, `json.Marshal`
   
   **Unexported (Private) Names**:
   - Begin with a lowercase letter
   - Only accessible within the declaring package
   - Examples: `fmt.newPrinter`, `http.serverHandler`, `json.typeFields`
   
   This applies to all identifiers:
   
   ```go
   package user
   
   // Exported types (accessible from other packages)
   type User struct {
       ID        string  // Exported field
       Email     string  // Exported field
       Name      string  // Exported field
       password  string  // Unexported field (private)
   }
   
   // Exported function (accessible from other packages)
   func NewUser(name, email string) *User {
       return &User{
           Name:  name,
           Email: email,
       }
   }
   
   // Unexported function (private to this package)
   func validateEmail(email string) bool {
       // Implementation
       return true
   }
   
   // Exported constants
   const (
       RoleAdmin  = "admin"
       RoleUser   = "user"
       maxRetries = 3  // Unexported constant
   )
   ```
   
   This convention is enforced by the compiler, not just a style guideline. Code attempting to access unexported names from another package will not compile.

3. **Q: What is the recommended way to manage and organize import statements in Go?**
   
   A: Go has well-established conventions for organizing imports:
   
   **Standard Format**:
   - Group imports logically
   - Standard library imports first
   - External packages second
   - Your own packages third
   - Groups separated by blank lines
   
   ```go
   import (
       // Standard library
       "fmt"
       "io"
       "time"
       
       // Third-party packages
       "github.com/aws/aws-sdk-go/aws"
       "github.com/gorilla/mux"
       
       // Local packages
       "myapp/internal/auth"
       "myapp/internal/config"
   )
   ```
   
   **Automatic Formatting**:
   - `goimports` tool automatically formats and organizes imports
   - VS Code and other editors can do this automatically on save
   - The `go fmt` command also sorts imports but doesn't add missing ones
   
   **Alias Use Guidelines**:
   - Use aliases to avoid name conflicts
   - Use aliases when the package name doesn't match the last element of the import path
   - Use aliases for clarity when two packages have the same name
   
   ```go
   import (
       "crypto/rand"
       mrand "math/rand"   // Alias to distinguish from crypto/rand
       
       corev1 "k8s.io/api/core/v1"  // Version clarification
       metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
   )
   ```
   
   **Import Path Style**:
   - For local application packages, use relative paths: `"myapp/pkg/logger"`
   - For published modules, use the full module path: `"github.com/username/module/pkg/logger"`
   
   Following these conventions makes your code more readable and consistent with the Go ecosystem.

### 10. Package Initialization

**Concise Explanation:**
Go packages initialize in a well-defined order. When a package is imported, all its imported packages are initialized first, then global variables are initialized, and finally `init()` functions are executed in the order they're defined. This happens before the `main()` function runs, allowing packages to set up their state before use.

**Where to Use:**
- One-time initialization of package-level state
- Driver registration (database, image formats)
- Setting up default configurations
- Validating package configurations
- Registering implementations with a registry
- Populating lookup tables or caches

**Code Snippet:**
```go
package database

import (
    "database/sql"
    "fmt"
    "log"
    "sync"
    
    _ "github.com/lib/pq" // Registers PostgreSQL driver via init()
)

// Package-level variables (initialized first)
var (
    defaultDB  *sql.DB
    defaultURL = "postgres://localhost:5432/mydb"
    initOnce   sync.Once
    dbMutex    sync.RWMutex
    errorLog   = log.New(os.Stderr, "DB ERROR: ", log.Ldate|log.Ltime)
)

// init function (called after variable initialization)
func init() {
    fmt.Println("Database package initializing...")
    
    // Try to connect with default config
    if _, err := sql.Open("postgres", defaultURL); err != nil {
        errorLog.Printf("Warning: Could not connect to default database: %v", err)
    }
}

// Another init function in the same package (called after the first init)
func init() {
    fmt.Println("Setting up connection pool defaults...")
    sql.SetMaxOpenConns(25)
    sql.SetMaxIdleConns(5)
}

// Connect establishes a database connection
func Connect(url string) (*sql.DB, error) {
    if url == "" {
        url = defaultURL
    }
    
    return sql.Open("postgres", url)
}

// Default returns a shared database connection
func Default() (*sql.DB, error) {
    var err error
    
    initOnce.Do(func() {
        defaultDB, err = Connect(defaultURL)
    })
    
    return defaultDB, err
}

// Example usage in main package
package main

import (
    "fmt"
    "log"
    
    "myapp/database"
)

func main() {
    // By now, database package is already initialized
    
    db, err := database.Default()
    if err != nil {
        log.Fatalf("Failed to get default database: %v", err)
    }
    
    // Use the database
    err = db.Ping()
    if err != nil {
        log.Fatalf("Database ping failed: %v", err)
    }
    
    fmt.Println("Connected to database successfully")
}
```

**Real-World Example:**
A configuration manager using package initialization:

```go
// config/config.go
package config

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "os"
    "path/filepath"
    "sync"
)

// Config holds application configuration
type Config struct {
    Environment string `json:"environment"`
    Server      struct {
        Host string `json:"host"`
        Port int    `json:"port"`
    } `json:"server"`
    Database struct {
        URL      string `json:"url"`
        MaxConns int    `json:"max_conns"`
    } `json:"database"`
    Logging struct {
        Level  string `json:"level"`
        Output string `json:"output"`
    } `json:"logging"`
}

var (
    // Singleton config instance
    config     *Config
    configOnce sync.Once
    configLock sync.RWMutex
    
    // Default paths to look for config files
    searchPaths []string
)

// Package initialization
func init() {
    // Setup default search paths for config files
    fmt.Println("Initializing configuration package...")
    
    // Start with current directory
    searchPaths = append(searchPaths, ".")
    
    // Add config directory if exists
    if _, err := os.Stat("./config"); err == nil {
        searchPaths = append(searchPaths, "./config")
    }
    
    // Add home directory config
    if homeDir, err := os.UserHomeDir(); err == nil {
        searchPaths = append(searchPaths, filepath.Join(homeDir, ".myapp"))
    }
    
    // Add system config directory
    searchPaths = append(searchPaths, "/etc/myapp")
    
    // Look for environment-specific config path
    if envPath := os.Getenv("MYAPP_CONFIG_PATH"); envPath != "" {
        searchPaths = append([]string{envPath}, searchPaths...)
    }
    
    // Set default config if MYAPP_USE_DEFAULTS is set
    if os.Getenv("MYAPP_USE_DEFAULTS") != "" {
        configOnce.Do(func() {
            config = defaultConfig()
        })
    }
}

// Default configuration
func defaultConfig() *Config {
    cfg := &Config{
        Environment: "development",
    }
    
    cfg.Server.Host = "localhost"
    cfg.Server.Port = 8080
    
    cfg.Database.URL = "postgres://localhost:5432/myapp"
    cfg.Database.MaxConns = 10
    
    cfg.Logging.Level = "info"
    cfg.Logging.Output = "stdout"
    
    return cfg
}

// Load loads configuration from file
func Load(filename string) (*Config, error) {
    // If filename is empty, search in default paths
    if filename == "" {
        for _, path := range searchPaths {
            // Try both config.json and config.development.json (etc)
            for _, name := range []string{"config.json", fmt.Sprintf("config.%s.json", 
                                         os.Getenv("MYAPP_ENV"))} {
                fullPath := filepath.Join(path, name)
                if _, err := os.Stat(fullPath); err == nil {
                    filename = fullPath
                    break
                }
            }
            if filename != "" {
                break
            }
        }
    }
    
    // If still no config file found
    if filename == "" {
        return nil, fmt.Errorf("no configuration file found in search paths")
    }
    
    // Read and parse the config file
    data, err := ioutil.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("failed to read config file: %w", err)
    }
    
    var cfg Config
    if err := json.Unmarshal(data, &cfg); err != nil {
        return nil, fmt.Errorf("failed to parse config file: %w", err)
    }
    
    return &cfg, nil
}

// Get returns the singleton config instance, loading it if necessary
func Get() (*Config, error) {
    configLock.RLock()
    if config != nil {
        defer configLock.RUnlock()
        return config, nil
    }
    configLock.RUnlock()
    
    // Need to initialize config
    configLock.Lock()
    defer configLock.Unlock()
    
    var err error
    configOnce.Do(func() {
        // Try to load from file
        config, err = Load("")
        if err != nil {
            // Fall back to default config
            config = defaultConfig()
        }
    })
    
    return config, err
}

// Set replaces the current configuration
func Set(cfg *Config) {
    configLock.Lock()
    defer configLock.Unlock()
    config = cfg
}

// usage in main.go
package main

import (
    "log"
    
    "myapp/config"
    "myapp/database"
    "myapp/server"
)

func main() {
    // Config package already initialized
    
    // Get singleton config
    cfg, err := config.Get()
    if err != nil {
        log.Fatalf("Failed to get config: %v", err)
    }
    
    // Initialize other components with config
    db, err := database.Connect(cfg.Database.URL)
    if err != nil {
        log.Fatalf("Database connection failed: %v", err)
    }
    
    // Start server
    server := server.New(cfg.Server.Host, cfg.Server.Port)
    server.Start()
}
```

**Common Pitfalls:**
- Overusing `init()` functions for complex logic instead of explicit initialization
- Relying on initialization order across packages, which can be fragile
- Side effects in `init()` functions that make testing difficult
- Circular dependencies between packages causing initialization issues
- Long-running operations in `init()` that delay program startup
- Global state that makes the code hard to test and reason about
- Not handling initialization failures gracefully

**Confusion Questions:**

1. **Q: In what order are package-level variables and init() functions executed?**
   
   A: Go follows a strict initialization order:
   
   1. **Package import order**: Packages are initialized in dependency order. If package A imports package B, B is fully initialized before A starts initialization.
   
   2. **Within each package**:
      - Package-level variable declarations are initialized first, in the order they appear in the source files (but dependencies between them are resolved first)
      - `init()` functions are called in the order they appear in the source files after all variables are initialized
   
   For example:
   
   ```go
   // file1.go
   package example
   
   import "fmt"
   
   var a = b + 1  // a depends on b, so b is initialized first despite appearing later
   var b = 1      // b has no dependencies
   
   func init() {
       fmt.Println("First init in file1")
       fmt.Printf("a = %d, b = %d\n", a, b)
   }
   
   // file2.go (same package)
   package example
   
   import "fmt"
   
   var c = 3
   
   func init() {
       fmt.Println("First init in file2")
       fmt.Printf("c = %d\n", c)
   }
   
   func init() {
       fmt.Println("Second init in file2")
   }
   ```
   
   Output would be:
   ```
   First init in file1
   a = 2, b = 1
   First init in file2
   c = 3
   Second init in file2
   ```
   
   This ordering is deterministic but can be complex in large programs, so it's best not to rely on intricate initialization ordering.

2. **Q: When should I use init() functions and when should I avoid them?**
   
   A: `init()` functions are powerful but should be used judiciously:
   
   **Good uses for init()**:
   - Registering implementations (database drivers, image codecs)
   - Setting up package-level caches or lookup tables
   - Validating that package-level variables are properly configured
   - One-time computations needed by package functions
   - Setting reasonable defaults for package behavior
   
   ```go
   func init() {
       // Good: Registering a database driver
       sql.Register("postgres", &PostgresDriver{})
       
       // Good: Populating a lookup table
       for i, name := range dayNames {
           dayMap[name] = i
       }
   }
   ```
   
   **Avoid init() for**:
   - Complex logic that should be explicit
   - Code with side effects (network calls, file I/O)
   - Functionality that should be configurable or testable
   - Anything that might fail and needs error handling
   
   ```go
   func init() {
       // Bad: Complex logic with side effects
       db, err := sql.Open("postgres", os.Getenv("DATABASE_URL"))
       if err != nil {
           log.Fatal("Failed to connect to database:", err)
       }
       defaultDB = db
       
       // Better approach: Provide an explicit initialization function
       // func InitDB() (*sql.DB, error) { ... }
   }
   ```
   
   As a rule of thumb, prefer explicit initialization that can be controlled and tested over implicit initialization in `init()`.

3. **Q: How can I test packages that use init() functions and package-level variables?**
   
   A: Testing code with `init()` functions and package-level variables can be challenging because they run before your tests. Here are strategies to make such code more testable:
   
   1. **Extract init logic to exported functions**:
      ```go
      // Instead of complex init():
      func init() {
          complexInitialization()
      }
      
      // Better: Allow explicit initialization
      var initialized bool
      var initOnce sync.Once
      
      func Initialize() {
          initOnce.Do(complexInitialization)
      }
      
      func complexInitialization() {
          // Implementation
          initialized = true
      }
      
      // Functions can check and call Initialize()
      func PublicFunction() {
          if !initialized {
              Initialize()
          }
          // ...
      }
      ```
   
   2. **Make dependencies injectable**:
      ```go
      // Instead of:
      var defaultClient *http.Client
      
      func init() {
          defaultClient = &http.Client{Timeout: 10 * time.Second}
      }
      
      func FetchData(url string) ([]byte, error) {
          return defaultClient.Get(url)
      }
      
      // Better:
      var defaultClient = &http.Client{Timeout: 10 * time.Second}
      
      func FetchData(url string, client *http.Client) ([]byte, error) {
          if client == nil {
              client = defaultClient
          }
          return client.Get(url)
      }
      ```
   
   3. **Use build tags or flags for testing**:
      ```go
      // file: database.go
      package db
      
      var Production = true
      
      // file: database_test.go
      // +build test
      
      package db
      
      func init() {
          Production = false
      }
      ```
   
   4. **Mock external dependencies**:
      ```go
      // Original code
      var timeNow = time.Now
      
      func isExpired(timestamp time.Time) bool {
          return timestamp.Before(timeNow())
      }
      
      // In tests
      func TestIsExpired(t *testing.T) {
          // Mock time
          oldTimeNow := timeNow
          defer func() { timeNow = oldTimeNow }()
          
          timeNow = func() time.Time {
              return time.Date(2023, 1, 1, 0, 0, 0, 0, time.UTC)
          }
          
          // Run tests with mocked time
      }
      ```
   
   These approaches make code more modular and testable while still providing the convenience of defaults for normal usage.

### 11. The init() Function

**Concise Explanation:**
The `init()` function in Go is a special function that's automatically executed when a package is initialized. A package can have multiple `init()` functions, even across multiple files, and they execute in the order they're defined after all package-level variables are initialized. This is used for one-time setup that must happen before the package is used.

**Where to Use:**
- One-time package setup
- Database driver registration
- Default configuration setup
- Field validation for package-level variables
- Registering codecs or handlers
- Pre-computing lookup tables or caches
- Setting up global state (with caution)

**Code Snippet:**
```go
package handler

import (
    "database/sql"
    "fmt"
    "log"
    "net/http"
    "os"
    "time"
)

// Package-level variables
var (
    db           *sql.DB
    logger       *log.Logger
    startupTime  time.Time
    allowedHosts map[string]bool
)

// First init function - setup logger
func init() {
    fmt.Println("Handler init: setting up logger")
    logger = log.New(os.Stdout, "HANDLER: ", log.Ldate|log.Ltime|log.Lshortfile)
}

// Second init function - parse allowed hosts
func init() {
    fmt.Println("Handler init: configuring allowed hosts")
    
    // Initialize map
    allowedHosts = make(map[string]bool)
    
    // Add default allowed hosts
    allowedHosts["localhost"] = true
    allowedHosts["127.0.0.1"] = true
    
    // Add hosts from environment variable
    if hosts := os.Getenv("ALLOWED_HOSTS"); hosts != "" {
        for _, host := range strings.Split(hosts, ",") {
            allowedHosts[strings.TrimSpace(host)] = true
        }
    }
    
    // Log configured hosts
    logger.Println("Configured allowed hosts:", allowedHosts)
}

// Third init function - record startup time
func init() {
    fmt.Println("Handler init: recording startup time")
    startupTime = time.Now()
}

// Connect initializes the database connection
func Connect(dsn string) error {
    var err error
    db, err = sql.Open("postgres", dsn)
    if err != nil {
        return fmt.Errorf("database connection failed: %w", err)
    }
    
    // Test connection
    err = db.Ping()
    if err != nil {
        return fmt.Errorf("database ping failed: %w", err)
    }
    
    logger.Println("Database connected successfully")
    return nil
}

// IsHostAllowed checks if a host is in the allowed list
func IsHostAllowed(host string) bool {
    return allowedHosts[host]
}

// HomeHandler handles the home page request
func HomeHandler(w http.ResponseWriter, r *http.Request) {
    // Check host is allowed
    if !IsHostAllowed(r.Host) {
        http.Error(w, "Host not allowed", http.StatusForbidden)
        return
    }
    
    // Serve request
    fmt.Fprintf(w, "Welcome! Server has been up for %s", 
               time.Since(startupTime).Round(time.Second))
}
```

**Real-World Example:**
A plugin registration system using init functions:

```go
// pkg/plugin/plugin.go
package plugin

import (
    "fmt"
    "sync"
)

// Plugin defines the interface all plugins must implement
type Plugin interface {
    Name() string
    Execute(args map[string]interface{}) (interface{}, error)
}

// Global registry
var (
    registry      = make(map[string]Plugin)
    registryMutex sync.RWMutex
)

// Register adds a plugin to the registry
func Register(p Plugin) {
    registryMutex.Lock()
    defer registryMutex.Unlock()
    
    name := p.Name()
    if _, exists := registry[name]; exists {
        fmt.Printf("Warning: Plugin %q already registered, overwriting\n", name)
    }
    
    registry[name] = p
    fmt.Printf("Plugin %q registered\n", name)
}

// Get retrieves a plugin by name
func Get(name string) (Plugin, bool) {
    registryMutex.RLock()
    defer registryMutex.RUnlock()
    
    plugin, exists := registry[name]
    return plugin, exists
}

// ListPlugins returns all registered plugins
func ListPlugins() []string {
    registryMutex.RLock()
    defer registryMutex.RUnlock()
    
    plugins := make([]string, 0, len(registry))
    for name := range registry {
        plugins = append(plugins, name)
    }
    
    return plugins
}

// plugins/logger/logger.go
package logger

import (
    "fmt"
    "time"
    
    "myapp/pkg/plugin"
)

// LoggerPlugin implements the Plugin interface for logging
type LoggerPlugin struct{}

// init registers the plugin
func init() {
    fmt.Println("Registering logger plugin")
    plugin.Register(&LoggerPlugin{})
}

// Name returns the plugin name
func (p *LoggerPlugin) Name() string {
    return "logger"
}

// Execute runs the plugin
func (p *LoggerPlugin) Execute(args map[string]interface{}) (interface{}, error) {
    message, ok := args["message"].(string)
    if !ok {
        return nil, fmt.Errorf("message argument required")
    }
    
    level := "info"
    if lvl, ok := args["level"].(string); ok {
        level = lvl
    }
    
    timestamp := time.Now().Format(time.RFC3339)
    logEntry := fmt.Sprintf("[%s] %s: %s", timestamp, level, message)
    
    fmt.Println(logEntry)
    return logEntry, nil
}

// plugins/counter/counter.go
package counter

import (
    "fmt"
    "sync/atomic"
    
    "myapp/pkg/plugin"
)

// CounterPlugin implements the Plugin interface for counting
type CounterPlugin struct {
    count int64
}

// Global counter instance
var globalCounter = &CounterPlugin{}

// init registers the plugin
func init() {
    fmt.Println("Registering counter plugin")
    plugin.Register(globalCounter)
}

// Name returns the plugin name
func (p *CounterPlugin) Name() string {
    return "counter"
}

// Execute runs the plugin
func (p *CounterPlugin) Execute(args map[string]interface{}) (interface{}, error) {
    // Default increment
    increment := int64(1)
    
    // Check if increment was provided
    if inc, ok := args["increment"].(float64); ok {
        increment = int64(inc)
    }
    
    // Atomically add to counter
    newValue := atomic.AddInt64(&p.count, increment)
    return newValue, nil
}

// main.go
package main

import (
    "fmt"
    
    // Import plugins for side effects (init registration)
    _ "myapp/plugins/logger"
    _ "myapp/plugins/counter"
    
    "myapp/pkg/plugin"
)

func main() {
    // List all registered plugins
    plugins := plugin.ListPlugins()
    fmt.Println("Available plugins:", plugins)
    
    // Use the logger plugin
    loggerPlugin, _ := plugin.Get("logger")
    loggerPlugin.Execute(map[string]interface{}{
        "message": "Application started",
        "level":   "info",
    })
    
    // Use the counter plugin
    counterPlugin, _ := plugin.Get("counter")
    result, _ := counterPlugin.Execute(map[string]interface{}{
        "increment": 1.0,
    })
    fmt.Println("Counter value:", result)
    
    result, _ = counterPlugin.Execute(map[string]interface{}{
        "increment": 5.0,
    })
    fmt.Println("Counter value:", result)
}

// Output:
// Registering logger plugin
// Registering counter plugin
// Available plugins: [logger counter]
// [2023-05-15T14:32:05Z] info: Application started
// Counter value: 1
// Counter value: 6
```

**Common Pitfalls:**
- Overreliance on `init()` functions for complex setup that should be explicit
- Performing operations in `init()` that might fail without proper error handling
- Creating hidden dependencies between packages through initialization order
- Making code hard to test due to side effects in `init()`
- Using `init()` for operations that should be configurable
- Circular dependencies causing initialization deadlocks
- Performance issues from heavy initialization before `main()`

**Confusion Questions:**

1. **Q: Can init() functions return errors or take parameters?**
   
   A: No, `init()` functions cannot return errors or take parameters. They must have the exact signature `func init()` with no parameters and no return values:
   
   ```go
   // Correct init function
   func init() {
       // Initialization code
   }
   
   // These would cause compilation errors:
   func init(config string) { }            // Error: parameters not allowed
   func init() error { return nil }        // Error: return values not allowed
   func init() (success bool) { return }   // Error: return values not allowed
   ```
   
   This limitation means you need alternative strategies for handling initialization errors:
   
   1. **Log errors and continue with defaults**:
      ```go
      func init() {
          config, err := loadConfig()
          if err != nil {
              log.Printf("Warning: Failed to load config: %v", err)
              config = defaultConfig()
          }
          setupWithConfig(config)
      }
      ```
   
   2. **Panic in truly unrecoverable situations**:
      ```go
      func init() {
          if err := criticalSetup(); err != nil {
              panic(fmt.Sprintf("Fatal initialization error: %v", err))
          }
      }
      ```
   
   3. **Defer initialization to an exported function**:
      ```go
      var initialized bool
      var initErr error
      
      func init() {
          // Light setup that can't fail
          prepareForInitialization()
      }
      
      func Initialize() error {
          if initialized {
              return initErr
          }
          
          // Perform initialization that might fail
          initErr = heavyInitialization()
          initialized = true
          return initErr
      }
      ```
   
   The third approach is generally preferable as it allows for proper error handling and testing.

2. **Q: What happens if there's a panic in an init() function?**
   
   A: If a panic occurs in an `init()` function and is not recovered within that function, the program will terminate immediately with an error message showing the panic and a stack trace. This happens before `main()` is called:
   
   ```go
   func init() {
       // This will cause the program to terminate during initialization
       panic("initialization failed")
   }
   ```
   
   To make initialization more robust, you can recover from panics within init:
   
   ```go
   func init() {
       defer func() {
           if r := recover(); r != nil {
               fmt.Printf("Recovered from panic in init: %v\n", r)
               // Set a default state or flag the initialization problem
               initializationFailed = true
           }
       }()
       
       // Risky initialization that might panic
       riskyInitialization()
   }
   ```
   
   However, recovering from panics in `init()` should be used with caution. If the initialization is truly critical, it might be better to let the program terminate rather than continue in a partially initialized state.
   
   A better approach is to avoid operations that might panic in `init()` functions altogether and instead use explicit initialization functions that return errors.

3. **Q: How can I control the order of init() functions across packages?**
   
   A: You cannot directly control the order of `init()` functions across different packages. Go guarantees:
   
   1. A package's `init()` functions run only after all its imported packages have been initialized
   2. Within a package, `init()` functions run in the order they appear in the source files
   
   If you need specific initialization order between packages, you have a few strategies:
   
   **1. Use import dependencies**:
   ```go
   // a/a.go
   package a
   
   func init() {
       fmt.Println("Package A initialized")
   }
   
   // b/b.go
   package b
   
   import "myapp/a"  // Ensures package a is initialized before b
   
   func init() {
       fmt.Println("Package B initialized")
   }
   
   // main.go
   package main
   
   import "myapp/b"  // Ensures package b (and transitively a) are initialized
   
   func main() {
       // By now, both package a and b are initialized
   }
   ```
   
   **2. Use explicit initialization functions instead**:
   ```go
   // a/a.go
   package a
   
   func Initialize() {
       fmt.Println("Package A initialized")
   }
   
   // b/b.go
   package b
   
   import "myapp/a"
   
   func Initialize() {
       a.Initialize()  // Explicitly initialize package A first
       fmt.Println("Package B initialized")
   }
   
   // main.go
   package main
   
   import (
       "myapp/b"
   )
   
   func main() {
       b.Initialize()  // This also initializes a
   }
   ```
   
   **3. Use dependency injection and configuration**:
   ```go
   // Define interfaces and dependencies explicitly
   // Pass initialized dependencies to components that need them
   ```
   
   The explicit initialization approach gives you the most control and makes the dependencies clearer. It's also much easier to test and handle errors properly.

### 12. Standard Library Overview

**Concise Explanation:**
Go's standard library is a comprehensive collection of packages that provide essential functionality for building applications. It includes packages for I/O, networking, web servers, cryptography, encoding/decoding, time handling, and more. The standard library is known for its consistency, quality, and efficiency, and it follows Go's design principles of simplicity and practicality.

**Where to Use:**
- Building network services and APIs
- File and data processing
- Encoding and decoding structured data
- Working with time, dates, and durations
- Handling concurrency and synchronization
- Creating HTTP clients and servers
- Performing cryptography operations
- Implementing common algorithms and data structures

**Code Snippet:**
```go
// A small program demonstrating various standard library packages
package main

import (
    "encoding/json"       // JSON encoding/decoding
    "flag"                // Command-line flag parsing
    "fmt"                 // Formatted I/O
    "io"                  // Basic I/O interfaces
    "log"                 // Logging functionality
    "net/http"            // HTTP client and server
    "os"                  // OS functionality
    "path/filepath"       // File path manipulation
    "strings"             // String manipulation
    "sync"                // Synchronization primitives
    "time"                // Time handling
)

// User represents a user record
type User struct {
    ID        int       `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

func main() {
    // Parse command line flags
    port := flag.Int("port", 8080, "Server port")
    dataDir := flag.String("data", "./data", "Data directory")
    verbose := flag.Bool("v", false, "Enable verbose output")
    flag.Parse()
    
    // Configure logging
    if *verbose {
        log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
    } else {
        log.SetFlags(log.Ldate | log.Ltime)
    }
    
    // Create data directory if it doesn't exist
    err := os.MkdirAll(*dataDir, 0755)
    if err != nil {
        log.Fatalf("Failed to create data directory: %v", err)
    }
    
    // Initialize users
    users := []User{
        {1, "Alice", "alice@example.com", time.Now().Add(-24 * time.Hour)},
        {2, "Bob", "bob@example.com", time.Now()},
    }
    
    // Create a waitgroup for server shutdown
    var wg sync.WaitGroup
    
    // Define HTTP handlers
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Welcome to the Go Standard Library Demo!")
    })
    
    http.HandleFunc("/users", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(users)
    })
    
    http.HandleFunc("/user", func(w http.ResponseWriter, r *http.Request) {
        // Get ID from query parameter
        idStr := r.URL.Query().Get("id")
        if idStr == "" {
            http.Error(w, "Missing id parameter", http.StatusBadRequest)
            return
        }
        
        var id int
        if _, err := fmt.Sscanf(idStr, "%d", &id); err != nil {
            http.Error(w, "Invalid id format", http.StatusBadRequest)
            return
        }
        
        // Find user
        for _, user := range users {
            if user.ID == id {
                w.Header().Set("Content-Type", "application/json")
                json.NewEncoder(w).Encode(user)
                return
            }
        }
        
        http.Error(w, "User not found", http.StatusNotFound)
    })
    
    http.HandleFunc("/save", func(w http.ResponseWriter, r *http.Request) {
        if r.Method != http.MethodPost {
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
            return
        }
        
        // Read request body
        var user User
        err := json.NewDecoder(r.Body).Decode(&user)
        if err != nil {
            http.Error(w, "Invalid JSON", http.StatusBadRequest)
            return
        }
        
        // Validate user data
        if user.Name == "" || !strings.Contains(user.Email, "@") {
            http.Error(w, "Invalid user data", http.StatusBadRequest)
            return
        }
        
        // Set creation time if not provided
        if user.CreatedAt.IsZero() {
            user.CreatedAt = time.Now()
        }
        
        // Generate ID if not provided
        if user.ID == 0 {
            // Find max ID and increment
            maxID := 0
            for _, u := range users {
                if u.ID > maxID {
                    maxID = u.ID
                }
            }
            user.ID = maxID + 1
        }
        
        // Save user to data file
        filename := filepath.Join(*dataDir, fmt.Sprintf("user_%d.json", user.ID))
        file, err := os.Create(filename)
        if err != nil {
            log.Printf("Error creating file: %v", err)
            http.Error(w, "Internal server error", http.StatusInternalServerError)
            return
        }
        defer file.Close()
        
        // Write pretty JSON
        encoder := json.NewEncoder(file)
        encoder.SetIndent("", "  ")
        if err := encoder.Encode(user); err != nil {
            log.Printf("Error encoding user: %v", err)
            http.Error(w, "Internal server error", http.StatusInternalServerError)
            return
        }
        
        // Add to users slice if new
        found := false
        for i, u := range users {
            if u.ID == user.ID {
                users[i] = user
                found = true
                break
            }
        }
        if !found {
            users = append(users, user)
        }
        
        // Return success
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusCreated)
        json.NewEncoder(w).Encode(map[string]interface{}{
            "success": true,
            "user_id": user.ID,
        })
    })
    
    // Start server
    serverAddr := fmt.Sprintf(":%d", *port)
    server := &http.Server{
        Addr:         serverAddr,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
    }
    
    wg.Add(1)
    go func() {
        defer wg.Done()
        log.Printf("Server starting on %s", serverAddr)
        if err := server.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatalf("HTTP server error: %v", err)
        }
        log.Println("Server stopped")
    }()
    
    // Wait for shutdown signal
    waitForInterrupt()
    
    // Graceful shutdown
    log.Println("Shutting down server...")
    shutdownCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    if err := server.Shutdown(shutdownCtx); err != nil {
        log.Fatalf("Server shutdown error: %v", err)
    }
    
    // Wait for server goroutine to exit
    wg.Wait()
    log.Println("Server gracefully stopped")
}

// Helper function to wait for interrupt signal
func waitForInterrupt() {
    c := make(chan os.Signal, 1)
    signal.Notify(c, os.Interrupt, syscall.SIGTERM)
    <-c
}
```

**Real-World Example:**
A file processing utility that demonstrates various standard library packages working together:

```go
package main

import (
    "archive/zip"
    "bufio"
    "compress/gzip"
    "crypto/md5"
    "encoding/csv"
    "encoding/hex"
    "encoding/json"
    "flag"
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "path/filepath"
    "regexp"
    "strconv"
    "strings"
    "sync"
    "time"
)

// FileInfo represents metadata about a processed file
type FileInfo struct {
    Path         string    `json:"path"`
    Size         int64     `json:"size"`
    ModTime      time.Time `json:"mod_time"`
    MD5Hash      string    `json:"md5_hash"`
    LineCount    int       `json:"line_count,omitempty"`
    WordCount    int       `json:"word_count,omitempty"`
    ProcessedAt  time.Time `json:"processed_at"`
}

// Stats represents aggregate statistics
type Stats struct {
    FileCount     int       `json:"file_count"`
    TotalSize     int64     `json:"total_size"`
    TotalLines    int       `json:"total_lines"`
    TotalWords    int       `json:"total_words"`
    OldestFile    string    `json:"oldest_file"`
    OldestTime    time.Time `json:"oldest_time"`
    NewestFile    string    `json:"newest_file"`
    NewestTime    time.Time `json:"newest_time"`
    ProcessedAt   time.Time `json:"processed_at"`
}

// Command-line flags
var (
    inputDir  = flag.String("in", ".", "Input directory to process")
    outputDir = flag.String("out", "./output", "Output directory for results")
    pattern   = flag.String("pattern", ".*", "Regex pattern for filenames to include")
    recursive = flag.Bool("recursive", false, "Process directories recursively")
    workers   = flag.Int("workers", 4, "Number of worker goroutines")
    httpAddr  = flag.String("http", ":8080", "HTTP server address (empty to disable)")
    compress  = flag.Bool("compress", false, "Compress output files with gzip")
    format    = flag.String("format", "json", "Output format (json, csv)")
)

func main() {
    // Parse command line flags
    flag.Parse()
    
    // Configure logging
    log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
    
    // Validate flags
    if *inputDir == "" {
        log.Fatal("Input directory cannot be empty")
    }
    
    // Create output directory if it doesn't exist
    if err := os.MkdirAll(*outputDir, 0755); err != nil {
        log.Fatalf("Failed to create output directory: %v", err)
    }
    
    // Compile the regex pattern
    regex, err := regexp.Compile(*pattern)
    if err != nil {
        log.Fatalf("Invalid regex pattern: %v", err)
    }
    
    // Start HTTP server if enabled
    if *httpAddr != "" {
        go startHTTPServer(*httpAddr, *inputDir, *outputDir)
    }
    
    // Process files
    results, err := processFiles(*inputDir, *outputDir, regex, *recursive, *workers)
    if err != nil {
        log.Fatalf("Processing failed: %v", err)
    }
    
    // Generate statistics
    stats := calculateStats(results)
    
    // Save results based on format
    if err := saveResults(results, stats, *outputDir, *format, *compress); err != nil {
        log.Fatalf("Failed to save results: %v", err)
    }
    
    log.Printf("Processing complete: %d files processed", stats.FileCount)
}

func processFiles(inputDir, outputDir string, pattern *regexp.Regexp, recursive bool, workerCount int) ([]FileInfo, error) {
    // Find all files to process
    var filesToProcess []string
    
    walkFunc := func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        
        // Skip directories unless in recursive mode
        if info.IsDir() {
            if path != inputDir && !recursive {
                return filepath.SkipDir
            }
            return nil
        }
        
        // Skip non-regular files
        if !info.Mode().IsRegular() {
            return nil
        }
        
        // Check if filename matches pattern
        if pattern.MatchString(info.Name()) {
            filesToProcess = append(filesToProcess, path)
        }
        
        return nil
    }
    
    if err := filepath.Walk(inputDir, walkFunc); err != nil {
        return nil, fmt.Errorf("failed to walk directory: %w", err)
    }
    
    // Process files concurrently with worker pool
    resultsCh := make(chan FileInfo, len(filesToProcess))
    errorsCh := make(chan error, len(filesToProcess))
    jobs := make(chan string, len(filesToProcess))
    
    // Start workers
    var wg sync.WaitGroup
    for i := 0; i < workerCount; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for path := range jobs {
                info, err := processFile(path)
                if err != nil {
                    errorsCh <- fmt.Errorf("failed to process %s: %w", path, err)
                    continue
                }
                resultsCh <- info
            }
        }()
    }
    
    // Send jobs to workers
    for _, path := range filesToProcess {
        jobs <- path
    }
    close(jobs)
    
    // Wait for all workers to finish
    wg.Wait()
    close(resultsCh)
    close(errorsCh)
    
    // Check for errors
    select {
    case err := <-errorsCh:
        return nil, err
    default:
        // No errors
    }
    
    // Collect results
    var results []FileInfo
    for result := range resultsCh {
        results = append(results, result)
    }
    
    return results, nil
}

func processFile(path string) (FileInfo, error) {
    // Open the file
    file, err := os.Open(path)
    if err != nil {
        return FileInfo{}, err
    }
    defer file.Close()
    
    // Get file info
    fileInfo, err := file.Stat()
    if err != nil {
        return FileInfo{}, err
    }
    
    // Create MD5 hash
    hasher := md5.New()
    if _, err := io.Copy(hasher, file); err != nil {
        return FileInfo{}, err
    }
    hash := hex.EncodeToString(hasher.Sum(nil))
    
    // Reset file pointer
    if _, err := file.Seek(0, 0); err != nil {
        return FileInfo{}, err
    }
    
    // Count lines and words
    scanner := bufio.NewScanner(file)
    lineCount := 0
    wordCount := 0
    
    for scanner.Scan() {
        lineCount++
        words := strings.Fields(scanner.Text())
        wordCount += len(words)
    }
    
    if err := scanner.Err(); err != nil {
        return FileInfo{}, err
    }
    
    return FileInfo{
        Path:        path,
        Size:        fileInfo.Size(),
        ModTime:     fileInfo.ModTime(),
        MD5Hash:     hash,
        LineCount:   lineCount,
        WordCount:   wordCount,
        ProcessedAt: time.Now(),
    }, nil
}

func calculateStats(files []FileInfo) Stats {
    if len(files) == 0 {
        return Stats{ProcessedAt: time.Now()}
    }
    
    stats := Stats{
        FileCount:   len(files),
        OldestTime:  files[0].ModTime,
        OldestFile:  files[0].Path,
        NewestTime:  files[0].ModTime,
        NewestFile:  files[0].Path,
        ProcessedAt: time.Now(),
    }
    
    for _, file := range files {
        stats.TotalSize += file.Size
        stats.TotalLines += file.LineCount
        stats.TotalWords += file.WordCount
        
        if file.ModTime.Before(stats.OldestTime) {
            stats.OldestTime = file.ModTime
            stats.OldestFile = file.Path
        }
        
        if file.ModTime.After(stats.NewestTime) {
            stats.NewestTime = file.ModTime
            stats.NewestFile = file.Path
        }
    }
    
    return stats
}

func saveResults(results []FileInfo, stats Stats, outputDir, format string, compress bool) error {
    // Create result filenames
    detailsFile := filepath.Join(outputDir, "file_details."+format)
    statsFile := filepath.Join(outputDir, "statistics."+format)
    
    if compress {
        detailsFile += ".gz"
        statsFile += ".gz"
    }
    
    // Save detailed results
    if err := saveData(results, detailsFile, format, compress); err != nil {
        return fmt.Errorf("failed to save details: %w", err)
    }
    
    // Save statistics
    if err := saveData(stats, statsFile, format, compress); err != nil {
        return fmt.Errorf("failed to save statistics: %w", err)
    }
    
    return nil
}

func saveData(data interface{}, filename, format string, compress bool) error {
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    var writer io.Writer = file
    
    // Add compression if enabled
    if compress {
        gz := gzip.NewWriter(file)
        defer gz.Close()
        writer = gz
    }
    
    // Write in requested format
    switch format {
    case "json":
        encoder := json.NewEncoder(writer)
        encoder.SetIndent("", "  ")
        return encoder.Encode(data)
        
    case "csv":
        csvWriter := csv.NewWriter(writer)
        defer csvWriter.Flush()
        
        switch v := data.(type) {
        case []FileInfo:
            // Write header
            header := []string{"Path", "Size", "ModTime", "MD5Hash", "LineCount", "WordCount", "ProcessedAt"}
            if err := csvWriter.Write(header); err != nil {
                return err
            }
            
            // Write rows
            for _, info := range v {
                row := []string{
                    info.Path,
                    strconv.FormatInt(info.Size, 10),
                    info.ModTime.Format(time.RFC3339),
                    info.MD5Hash,
                    strconv.Itoa(info.LineCount),
                    strconv.Itoa(info.WordCount),
                    info.ProcessedAt.Format(time.RFC3339),
                }
                if err := csvWriter.Write(row); err != nil {
                    return err
                }
            }
            
        case Stats:
            // Write header
            header := []string{"FileCount", "TotalSize", "TotalLines", "TotalWords", "OldestFile", "OldestTime", "NewestFile", "NewestTime", "ProcessedAt"}
            if err := csvWriter.Write(header); err != nil {
                return err
            }
            
            // Write row
            row := []string{
                strconv.Itoa(v.FileCount),
                strconv.FormatInt(v.TotalSize, 10),
                strconv.Itoa(v.TotalLines),
                strconv.Itoa(v.TotalWords),
                v.OldestFile,
                v.OldestTime.Format(time.RFC3339),
                v.NewestFile,
                v.NewestTime.Format(time.RFC3339),
                v.ProcessedAt.Format(time.RFC3339),
            }
            if err := csvWriter.Write(row); err != nil {
                return err
            }
            
        default:
            return fmt.Errorf("unsupported data type for CSV export")
        }
        
        return nil
        
    default:
        return fmt.Errorf("unsupported format: %s", format)
    }
}

func startHTTPServer(addr, inputDir, outputDir string) {
    // Status handler
    http.HandleFunc("/status", func(w http.ResponseWriter, r *http.Request) {
        status := map[string]interface{}{
            "status":      "running",
            "input_dir":   inputDir,
            "output_dir":  outputDir,
            "server_time": time.Now(),
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(status)
    })
    
    // Results handler
    http.HandleFunc("/results", func(w http.ResponseWriter, r *http.Request) {
        // Serve the statistics file
        statsFile := filepath.Join(outputDir, "statistics.json")
        if _, err := os.Stat(statsFile); os.IsNotExist(err) {
            statsFile = filepath.Join(outputDir, "statistics.json.gz")
            if _, err := os.Stat(statsFile); os.IsNotExist(err) {
                http.Error(w, "Results not yet available", http.StatusNotFound)
                return
            }
        }
        
        http.ServeFile(w, r, statsFile)
    })
    
    // Download handler
    http.HandleFunc("/download/", func(w http.ResponseWriter, r *http.Request) {
        fileName := strings.TrimPrefix(r.URL.Path, "/download/")
        if fileName == "" {
            http.Error(w, "File not specified", http.StatusBadRequest)
            return
        }
        
        filePath := filepath.Join(outputDir, fileName)
        if _, err := os.Stat(filePath); os.IsNotExist(err) {
            http.Error(w, "File not found", http.StatusNotFound)
            return
        }
        
        http.ServeFile(w, r, filePath)
    })
    
    // Start HTTP server
    log.Printf("HTTP server listening on %s", addr)
    if err := http.ListenAndServe(addr, nil); err != nil {
        log.Printf("HTTP server error: %v", err)
    }
}

// Helper function for creating a ZIP archive of results
func createArchive(outputDir, archiveFile string) error {
    // Create output file
    zipFile, err := os.Create(archiveFile)
    if err != nil {
        return err
    }
    defer zipFile.Close()
    
    // Create ZIP writer
    archive := zip.NewWriter(zipFile)
    defer archive.Close()
    
    // Walk output directory
    return filepath.Walk(outputDir, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        
        // Skip directories and the archive itself
        if info.IsDir() || path == archiveFile {
            return nil
        }
        
        // Create ZIP header
        header, err := zip.FileInfoHeader(info)
        if err != nil {
            return err
        }
        
        // Make paths relative to output directory
        relPath, err := filepath.Rel(outputDir, path)
        if err != nil {
            return err
        }
        header.Name = relPath
        
        // Create ZIP entry
        writer, err := archive.CreateHeader(header)
        if err != nil {
            return err
        }
        
        // Copy file contents
        file, err := os.Open(path)
        if err != nil {
            return err
        }
        defer file.Close()
        
        _, err = io.Copy(writer, file)
        return err
    })
}
```

**Common Pitfalls:**
- Reinventing functionality already provided by the standard library
- Not checking for errors returned by standard library functions
- Not fully understanding the behavior of functions (e.g., time.Parse format)
- Using deprecated functions or approaches
- Not using context for cancelable operations
- Missing opportunities to use more efficient standard library functions
- Incorrect use of synchronization primitives leading to deadlocks or race conditions

**Confusion Questions:**

1. **Q: How do I decide whether to use the standard library or a third-party package?**
   
   A: This decision involves several considerations:
   
   **Use the standard library when**:
   - It provides the functionality you need with acceptable performance
   - Long-term stability is essential (the standard library has a strong compatibility guarantee)
   - You want to minimize dependencies in your project
   - You need functionality that's part of Go's core concepts (HTTP, JSON, basic I/O)
   
   ```go
   // Standard library approach
   import (
       "encoding/json"
       "net/http"
   )
   
   func handler(w http.ResponseWriter, r *http.Request) {
       data := map[string]string{"message": "Hello, World!"}
       w.Header().Set("Content-Type", "application/json")
       json.NewEncoder(w).Encode(data)
   }
   ```
   
   **Consider third-party packages when**:
   - You need more advanced features not available in the standard library
   - The package significantly improves productivity for complex tasks
   - Performance requirements exceed what the standard library can provide
   - The package has a strong community and maintenance record
   
   ```go
   // Third-party library approach with gin
   import "github.com/gin-gonic/gin"
   
   func handler(c *gin.Context) {
       c.JSON(200, gin.H{
           "message": "Hello, World!",
       })
   }
   ```
   
   **Best Practice**:
   Start with the standard library and only add third-party dependencies when there's a clear benefit. For each dependency, consider:
   
   1. How mature and well-maintained is it?
   2. Does it follow Go idioms and conventions?
   3. Will it create dependency headaches later?
   4. How much functionality do you actually need?
   
   The Go community generally values simplicity and minimal dependencies, so don't overlook what the standard library can do.

2. **Q: What are the most important packages in the standard library that every Go developer should know?**
   
   A: Here's a curated list of essential standard library packages:
   
   **Core Functionality**:
   - `fmt`: Formatted I/O with functions like `Println` and `Sprintf`
   - `errors`: Error handling primitives and wrapping
   - `context`: For cancellation, deadlines, and request-scoped values
   - `time`: Date, time, and duration operations
   
   **Collections and Data Structures**:
   - `strings`: String manipulation functions
   - `strconv`: String conversions to/from other types
   - `sort`: Sorting slices and user-defined collections
   - `container/*`: Specialized containers like heaps and lists
   
   **I/O and Filesystem**:
   - `io`: Basic I/O interfaces
   - `bufio`: Buffered I/O
   - `os`: OS functionality and file operations
   - `path/filepath`: File path manipulation
   
   **Encoding/Decoding**:
   - `encoding/json`: JSON encoding and decoding
   - `encoding/xml`: XML processing
   - `encoding/csv`: CSV file reading and writing
   - `encoding/base64`: Base64 encoding
   
   **Networking**:
   - `net/http`: HTTP client and server implementations
   - `net`: Basic networking interfaces
   - `net/url`: URL parsing and manipulation
   
   **Concurrency**:
   - `sync`: Synchronization primitives (Mutex, WaitGroup, etc.)
   - `sync/atomic`: Atomic operations for low-level synchronization
   
   **Reflection and Metaprogramming**:
   - `reflect`: Runtime reflection capabilities
   
   **Development**:
   - `testing`: Support for automated testing
   - `log`: Simple logging capability
   - `flag`: Command-line flag parsing
   
   Mastering these packages provides a solid foundation for most Go development tasks. When you're familiar with them, you'll often find you can solve problems with just the standard library that might otherwise require third-party packages.

3. **Q: What are some less obvious but powerful features in the standard library?**
   
   A: The Go standard library has many hidden gems that can save you time and effort:
   
   1. **`text/template` and `html/template`**:
      Powerful templating engines for text and HTML with context-aware escaping:
      ```go
      tmpl := template.Must(template.New("example").Parse("Hello {{.Name}}!"))
      tmpl.Execute(os.Stdout, struct{ Name string }{"Gopher"})
      ```
   
   2. **`net/http/httputil`**:
      Includes a reverse proxy that can forward requests to another server:
      ```go
      proxy := httputil.NewSingleHostReverseProxy(targetURL)
      http.Handle("/api/", proxy)
      ```
   
   3. **`encoding/csv`**:
      Not just for reading CSV files, but also writing them with proper escaping:
      ```go
      writer := csv.NewWriter(os.Stdout)
      writer.Write([]string{"Name", "Age"})
      writer.Write([]string{"Alice", "30"})
      writer.Flush()
      ```
   
   4. **`regexp`**:
      Full-featured regular expression engine with capturing groups and replacements:
      ```go
      r := regexp.MustCompile(`(\w+):(\d+)`)
      matches := r.FindAllStringSubmatch("alice:42,bob:25", -1)
      // matches[0] is ["alice:42", "alice", "42"]
      ```
   
   5. **`database/sql`**:
      A generic interface to SQL databases that works with many drivers:
      ```go
      db, _ := sql.Open("postgres", "connection_string")
      rows, _ := db.Query("SELECT name FROM users WHERE age > $1", 18)
      ```
   
   6. **`net/http/pprof`**:
      Profile your running web application with a single import:
      ```go
      import _ "net/http/pprof"  // Registers handlers under /debug/pprof/
      ```
   
   7. **`time.Ticker` and `time.Timer`**:
      For recurring and one-time events:
      ```go
      ticker := time.NewTicker(1 * time.Second)
      for t := range ticker.C {
          fmt.Println("Tick at", t)
      }
      ```
   
   8. **`io.MultiWriter` and `io.MultiReader`**:
      Combine multiple writers or readers:
      ```go
      file, _ := os.Create("log.txt")
      writer := io.MultiWriter(os.Stdout, file)
      fmt.Fprintln(writer, "Log this to both console and file")
      ```
   
   9. **`sort.Interface`**:
      Create custom sorting for any data structure:
      ```go
      type ByLength []string
      func (s ByLength) Len() int { return len(s) }
      func (s ByLength) Less(i, j int) bool { return len(s[i]) < len(s[j]) }
      func (s ByLength) Swap(i, j int) { s[i], s[j] = s[j], s[i] }
      
      sorted := []string{"hello", "a", "world"}
      sort.Sort(ByLength(sorted))
      ```
   
   10. **`bytes.Buffer`**:
       Efficient way to build strings:
       ```go
       var buf bytes.Buffer
       for i := 0; i < 1000; i++ {
           fmt.Fprintf(&buf, "%d", i)
       }
       result := buf.String()
       ```
   
   These features often eliminate the need for third-party libraries and can make your code more efficient and maintainable.

## Next Actions

### Exercises and Micro-Projects

1. **Function Library**
   - Create utility functions for common operations (string manipulation, numeric calculations)
   - Implement functions with various parameter and return types
   - Practice using named return values and multiple returns
   - Build a set of variadic helper functions

2. **Custom Package Structure**
   - Create a multi-package application with proper organization
   - Define clear package boundaries and responsibilities
   - Use internal packages for implementation details
   - Practice exporting only necessary identifiers

3. **Command-Line Tool**
   - Create a CLI tool using functions and packages
   - Implement subcommands as separate packages
   - Use defer for resource cleanup
   - Handle errors properly with custom error types

4. **Middleware Chain**
   - Implement an HTTP middleware system using closures
   - Create various middleware components (logging, authentication, rate limiting)
   - Practice function composition patterns
   - Use context package for request scoping

5. **Plugin System**
   - Design a plugin architecture with interfaces
   - Use init() functions for registration
   - Support dynamic loading of functionality
   - Practice dependency injection patterns

### Real-World Project: Task Management API

Build a task management REST API that demonstrates various function and package concepts:

1. **Project Structure**:
   ```
   taskmanager/
   ├── cmd/
   │   └── server/
   │       └── main.go          # Entry point
   ├── internal/
   │   ├── auth/                # Authentication package
   │   │   ├── auth.go
   │   │   └── middleware.go
   │   ├── handler/             # HTTP handlers
   │   │   ├── tasks.go
   │   │   └── users.go
   │   ├── model/               # Domain models
   │   │   ├── task.go
   │   │   └── user.go
   │   └── storage/             # Data persistence
   │       ├── memory.go
   │       └── postgres.go
   ├── pkg/
   │   ├── config/              # Configuration
   │   │   └── config.go
   │   └── logger/              # Logging package
   │       └── logger.go
   └── go.mod
   ```

2. **Features**:
   - Create, read, update, delete tasks
   - User authentication and authorization
   - Task filtering and sorting
   - Task assignment and status tracking

3. **Implementation Focus**:
   - Use interfaces for storage to allow different backends
   - Implement middleware using closures
   - Use context for request cancellation and timeouts
   - Practice error handling with multiple return values
   - Organize code into packages with clear responsibilities
   - Use defer for resource cleanup
   - Implement proper initialization with init() where appropriate

This project will exercise your understanding of functions, closures, packages, and the standard library, while building something useful.

## Success Criteria

You've mastered Functions and Packages when you can:

1. **Function Design**
   - Create functions with appropriate parameters and return values
   - Use multiple return values for error handling
   - Implement variadic functions when appropriate
   - Understand when to use named return values
   - Create and use closures effectively

2. **Package Organization**
   - Design a clear package structure for applications
   - Correctly use visibility rules (exported vs unexported)
   - Create packages with focused responsibilities
   - Avoid circular dependencies between packages
   - Properly initialize packages with variables and init()

3. **Error Handling**
   - Return errors appropriately from functions
   - Use multiple return values for success/failure states
   - Implement appropriate defer, panic, and recover patterns
   - Create custom error types when needed

4. **Standard Library**
   - Know which packages to use for common tasks
   - Combine multiple standard library packages effectively
   - Use the standard library before reaching for third-party packages
   - Understand common patterns in the standard library

5. **Code Quality**
   - Write code with clear boundaries between packages
   - Use functions with appropriate scope and size
   - Implement consistent error handling patterns
   - Follow Go conventions for function and package design

## Troubleshooting

### Common Issues and Solutions

1. **"Imported and not used" errors**
   - Remove unused imports
   - If needed for initialization, use blank import (`import _ "package"`)
   - Consider if the package is actually necessary

2. **"Undefined: X" errors**
   - Check if the identifier is exported (capitalized) in its package
   - Verify the package is imported correctly
   - Ensure there are no typos in the identifier name

3. **Circular package dependencies**
   - Extract common types to a separate package
   - Use interfaces to break direct dependencies
   - Reorganize your package structure to follow natural dependency flow
   - Consider dependency injection patterns

4. **"Multiple packages in directory" error**
   - Each directory must contain only one package
   - Test files can use `package name_test` to create a separate package
   - Move different packages to different directories

5. **Missing returns in functions**
   - Add return statements for all code paths
   - Use named return values for complex functions
   - Consider early returns for error cases

6. **defer not working as expected**
   - Remember defer runs in LIFO order
   - Check that deferred functions are declared before they're needed
   - Remember that arguments to deferred functions are evaluated immediately

### Debugging Tips

1. **Debugging functions**
   ```go
   func debugWrapper(name string, f func(int) int) func(int) int {
       return func(x int) int {
           fmt.Printf("Calling %s with %d\n", name, x)
           result := f(x)
           fmt.Printf("%s returned %d\n", name, result)
           return result
       }
   }
   
   // Usage
   square := debugWrapper("square", func(x int) int { return x * x })
   square(4) // Logs the input and output
   ```

2. **Finding package initialization issues**
   ```go
   func init() {
       fmt.Printf("Package %s initializing\n", "mypackage")
       // Put log statements in each init function
   }
   ```

3. **Inspecting return values**
   ```go
   result, err := someFunction()
   fmt.Printf("Result: %#v, Error: %v\n", result, err)
   ```

4. **Tracing defer execution**
   ```go
   defer func() {
       fmt.Println("Defer executed at:", time.Now())
   }()
   ```

5. **Debugging closures**
   ```go
   counter := 0
   increment := func() int {
       counter++
       fmt.Printf("Counter is now %d\n", counter)
       return counter
   }
   ```

### Package Design Tips

1. **Package Naming**
   - Use simple, lower-case, single-word names
   - Name should describe the package's purpose, not contents
   - Avoid generic names like "util" or "common"
   - Avoid package name conflicts with standard library

2. **Directory Structure**
   - Keep similar functionality together
   - Use `/cmd` for executables
   - Use `/internal` for non-exported code
   - Use `/pkg` for exported library code
   - Use `/api` for API definitions

3. **Visibility Guidelines**
   - Export only what other packages need
   - Keep implementation details unexported
   - Use interfaces to expose behavior, not implementation
   - Document exported functions and types well

4. **Avoiding Circular Dependencies**
   - Draw dependency diagrams to visualize relationships
   - Create a core domain package with no outward dependencies
   - Push dependencies outward, not inward
   - Use interfaces defined in the "client" package

5. **Initialization Order**
   - Don't rely on complex initialization order
   - Use explicit initialization functions for logic that might fail
   - Keep init() functions simple and idempotent
   - Use sync.Once for expensive one-time initialization

Remember that good package design is about managing complexity and dependencies. Strive for packages that are focused on a single responsibility, with clear boundaries and interfaces.
