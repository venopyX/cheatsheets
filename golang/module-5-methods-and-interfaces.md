# Module 5: Methods and Interfaces

## Core Problem
Understanding Go's approach to object-oriented programming through methods and interfaces, allowing for code reuse, polymorphism, and abstraction without traditional inheritance-based class systems.

## Key Assumptions
- You understand basic Go syntax, data types, and functions
- You need to create reusable and modular code
- You want to achieve polymorphism and abstraction in Go
- You want to create flexible APIs that can work with multiple types

## Essential Concepts

### 1. Defining Methods on Types

**Concise Explanation:**
A method is a function with a special receiver argument that appears before the function name. The receiver connects the function to a specific type, allowing you to call the function using dot notation on values of that type. Methods can be defined for any named type that's declared in the same package, except for pointer or interface types.

**Where to Use:**
- To associate behavior with data
- To implement object-oriented patterns in Go
- To enable method chaining for fluent APIs
- To implement interfaces
- To encapsulate operations on complex types
- To add behaviors to custom types

**Code Snippet:**
```go
// Define a type
type Rectangle struct {
    Width  float64
    Height float64
}

// Method with a value receiver
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Method with a pointer receiver
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

// Method on a non-struct type
type Distance float64

func (d Distance) ToMeters() float64 {
    return float64(d) * 0.3048 // Convert feet to meters
}

func main() {
    // Create a Rectangle
    rect := Rectangle{Width: 10, Height: 5}
    
    // Call the Area method
    area := rect.Area()
    fmt.Printf("Area: %.2f\n", area) // Output: Area: 50.00
    
    // Call the Scale method
    rect.Scale(2)
    fmt.Printf("Scaled dimensions: %.1f x %.1f\n", rect.Width, rect.Height) // Output: Scaled dimensions: 20.0 x 10.0
    fmt.Printf("New area: %.2f\n", rect.Area()) // Output: New area: 200.00
    
    // Method on a non-struct type
    distance := Distance(100)
    meters := distance.ToMeters()
    fmt.Printf("%.2f feet = %.2f meters\n", distance, meters) // Output: 100.00 feet = 30.48 meters
}
```

**Real-World Example:**
Building a user management system with methods:

```go
package user

import (
    "errors"
    "regexp"
    "strings"
    "time"
    "golang.org/x/crypto/bcrypt"
)

// User represents a system user
type User struct {
    ID        string
    Email     string
    Password  string // Hashed
    FirstName string
    LastName  string
    Active    bool
    CreatedAt time.Time
    UpdatedAt time.Time
}

// New creates a new User with sensible defaults
func New(email, password, firstName, lastName string) (*User, error) {
    id := generateID()
    now := time.Now()
    
    user := &User{
        ID:        id,
        Email:     strings.ToLower(strings.TrimSpace(email)),
        FirstName: firstName,
        LastName:  lastName,
        Active:    true,
        CreatedAt: now,
        UpdatedAt: now,
    }
    
    if err := user.SetPassword(password); err != nil {
        return nil, err
    }
    
    if err := user.Validate(); err != nil {
        return nil, err
    }
    
    return user, nil
}

// FullName returns the user's full name
func (u User) FullName() string {
    return u.FirstName + " " + u.LastName
}

// SetPassword hashes and sets the user's password
func (u *User) SetPassword(password string) error {
    if len(password) < 8 {
        return errors.New("password must be at least 8 characters")
    }
    
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        return err
    }
    
    u.Password = string(hash)
    u.UpdatedAt = time.Now()
    return nil
}

// CheckPassword verifies if the provided password is correct
func (u User) CheckPassword(password string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(u.Password), []byte(password))
    return err == nil
}

// Validate checks if the user data is valid
func (u User) Validate() error {
    if u.Email == "" {
        return errors.New("email is required")
    }
    
    emailRegex := regexp.MustCompile(`^[a-z0-9._%+\-]+@[a-z0-9.\-]+\.[a-z]{2,}$`)
    if !emailRegex.MatchString(u.Email) {
        return errors.New("invalid email format")
    }
    
    if u.FirstName == "" {
        return errors.New("first name is required")
    }
    
    if u.LastName == "" {
        return errors.New("last name is required")
    }
    
    return nil
}

// Deactivate sets the user as inactive
func (u *User) Deactivate() {
    u.Active = false
    u.UpdatedAt = time.Now()
}

// Activate sets the user as active
func (u *User) Activate() {
    u.Active = true
    u.UpdatedAt = time.Now()
}

// IsActive returns whether the user is active
func (u User) IsActive() bool {
    return u.Active
}

// Age returns the duration since the user was created
func (u User) Age() time.Duration {
    return time.Since(u.CreatedAt)
}

// Usage example
func Example() {
    user, err := New("alice@example.com", "securepass", "Alice", "Smith")
    if err != nil {
        panic(err)
    }
    
    // Use methods
    fmt.Println("Welcome,", user.FullName())
    
    // Check password
    if user.CheckPassword("wrongpass") {
        fmt.Println("Password is correct") // Won't print
    }
    
    if user.CheckPassword("securepass") {
        fmt.Println("Password is correct") // Will print
    }
    
    // Deactivate the user
    user.Deactivate()
    fmt.Printf("User active: %v\n", user.IsActive()) // false
    
    // Activate the user
    user.Activate()
    fmt.Printf("User active: %v\n", user.IsActive()) // true
    
    // Print user account age
    fmt.Printf("Account age: %v\n", user.Age())
}
```

**Common Pitfalls:**
- Forgetting that value receivers operate on a copy (modifications won't affect the original)
- Using pointer receivers when unnecessary (for small structs with no mutations)
- Not being consistent with receiver types (mixing pointer and value receivers for the same type)
- Adding methods to types from other packages (not allowed in Go)
- Forgetting to check nil pointers in pointer receiver methods
- Not understanding when pointer receiver methods can be called on values (automatic dereferencing)
- Over-complicating method names when the receiver type already provides context

**Confusion Questions:**

1. **Q: When should I use a value receiver versus a pointer receiver?**
   
   A: The choice between value receivers and pointer receivers depends on several factors:
   
   **Use pointer receivers when:**
   
   1. **You need to modify the receiver**:
      ```go
      func (u *User) SetName(name string) {
          u.Name = name  // Modifies the original User
      }
      ```
   
   2. **The receiver is large** (to avoid copying):
      ```go
      func (img *LargeImage) Process() {
          // Avoids copying the entire image data
      }
      ```
   
   3. **You need to maintain a consistent receiver type** when other methods must use pointers:
      ```go
      // If some methods need pointers, use pointers for all methods
      func (u *User) Save() { /* ... */ }       // Must be pointer
      func (u *User) Validate() bool { /* ... */ }  // Use pointer for consistency
      ```
   
   4. **To handle nil receivers** properly:
      ```go
      func (l *Logger) Log(msg string) {
          if l == nil {
              // Handle nil logger case
              return
          }
          // Normal logging
      }
      ```
   
   **Use value receivers when:**
   
   1. **You don't need to modify the receiver**:
      ```go
      func (r Rectangle) Area() float64 {
          return r.Width * r.Height  // No modification needed
      }
      ```
   
   2. **The receiver is a small, built-in type** or a small struct:
      ```go
      func (p Point) Distance(q Point) float64 {
          // Point is small, copying is cheap
      }
      ```
   
   3. **You want to enforce immutability**:
      ```go
      func (d Date) AddDays(days int) Date {
          // Returns a new Date without modifying the original
          newDate := d  // Copy
          // Modify newDate...
          return newDate
      }
      ```
   
   4. **The receiver is naturally a value type** (like a small struct representing a value):
      ```go
      func (m Money) Format() string {
          return fmt.Sprintf("$%.2f", m.Amount)
      }
      ```
   
   The general rule: if in doubt, use a pointer receiver. But be consistent—if some methods of a type must use pointer receivers, use pointer receivers for all methods of that type.

2. **Q: Can I add methods to built-in types like int or string?**
   
   A: No, you cannot add methods directly to built-in types like `int` or `string`. However, you can create your own type based on a built-in type (called a "type definition") and then add methods to that new type:
   
   ```go
   // This won't compile
   func (s string) Reverse() string {
       // Error: cannot define new methods on non-local type string
   }
   
   // This works: define a new type first
   type MyString string
   
   // Then add a method to your new type
   func (s MyString) Reverse() string {
       runes := []rune(string(s))
       for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
           runes[i], runes[j] = runes[j], runes[i]
       }
       return string(runes)
   }
   
   // Usage
   func main() {
       str := MyString("Hello, World!")
       reversed := str.Reverse()
       fmt.Println(reversed) // "!dlroW ,olleH"
       
       // Note that you need to convert between string and MyString
       regularString := "Regular string"
       // regularString.Reverse() // Error: string doesn't have Reverse method
       
       myStr := MyString(regularString)
       reversed = myStr.Reverse()
       
       // Converting back
       backToRegular := string(myStr)
   }
   ```
   
   This limitation exists because:
   
   1. **Package coherence**: Methods must be defined in the same package as the type, but built-in types are in a special package.
   
   2. **Type safety**: Allowing everyone to add methods to built-in types could lead to conflicts and unpredictable behavior.
   
   3. **Design choice**: Go emphasizes composition over inheritance or monkey-patching.
   
   When adding methods to type definitions, be aware that the new type is not interchangeable with the original type—explicit conversion is required. This is different from "type aliases" (using `type MyString = string`), which creates a fully interchangeable alias but still doesn't allow adding methods.

3. **Q: How are methods different from functions, and why use one over the other?**
   
   A: Methods and functions serve different purposes and have important differences:
   
   **Key differences:**
   
   1. **Syntax and invocation**:
      ```go
      // Function
      func CalculateArea(r Rectangle) float64 {
          return r.Width * r.Height
      }
      area := CalculateArea(rect)  // Called with arguments
      
      // Method
      func (r Rectangle) Area() float64 {
          return r.Width * r.Height
      }
      area := rect.Area()  // Called with dot notation
      ```
   
   2. **Receiver parameter**:
      - Methods have a special receiver parameter that binds the function to a type
      - Functions have no receiver and operate on their parameters
   
   3. **Namespace**:
      - Methods are "attached" to a type, creating a namespace
      - Functions exist at the package level
   
   **When to use methods:**
   
   1. **Object-oriented design**:
      ```go
      user.Save()           // More intuitive than SaveUser(user)
      user.ValidateEmail()  // Clearly operates on the user
      ```
   
   2. **Method chaining**:
      ```go
      query.Where("age > ?", 18).OrderBy("name").Limit(10).Offset(20)
      ```
   
   3. **Interface implementation**:
      ```go
      // io.Reader interface requires a Read method
      func (f *File) Read(p []byte) (n int, err error) {
          // Implementation
      }
      ```
   
   4. **When behavior is intrinsic to the type**:
      ```go
      time.Now().Add(5 * time.Minute)  // Adding is intrinsic to time
      ```
   
   **When to use functions:**
   
   1. **Operations on multiple types**:
      ```go
      Equal(point1, point2)  // Comparing two entities of equal standing
      ```
   
   2. **Standalone utility functions**:
      ```go
      filepath.Join("dir", "file")  // Not tied to a specific type
      ```
   
   3. **When no clear "owner" exists**:
      ```go
      sort.Slice(people, func(i, j int) bool {
          return people[i].Name < people[j].Name
      })
      ```
   
   4. **Pure functions with no state**:
      ```go
      math.Sqrt(16)  // Pure mathematical operation
      ```
   
   Methods make code more readable when operations are conceptually tied to a type, while functions are better for operations that aren't tied to a specific type or that operate on multiple types equally. Methods also enable polymorphism through interfaces, which is a powerful tool in Go for creating flexible and testable code.

### 2. Value Receivers vs. Pointer Receivers

**Concise Explanation:**
Methods in Go can have either value receivers or pointer receivers. Value receivers operate on a copy of the value, while pointer receivers operate on the original value via a pointer. This choice affects whether the method can modify the original value, how the method behaves with nil values, and the performance characteristics when dealing with large structs.

**Where to Use:**
- Value receivers for immutable operations or small structs
- Pointer receivers when methods need to modify the receiver
- Pointer receivers for large structs to avoid copying
- Pointer receivers when nil handling is needed
- Value receivers to enforce immutability
- Consistent receiver types across all methods of a type

**Code Snippet:**
```go
type Counter struct {
    count int
}

// Value receiver - works on a copy of the counter
func (c Counter) Value() int {
    return c.count
}

// Pointer receiver - can modify the original counter
func (c *Counter) Increment() {
    c.count++
}

// Pointer receiver - handles nil case
func (c *Counter) IncrementSafely() int {
    // Handle nil pointer gracefully
    if c == nil {
        return 0
    }
    c.count++
    return c.count
}

func main() {
    // Using value and pointer methods
    counter := Counter{count: 5}
    
    // Call method with value receiver
    fmt.Println("Value:", counter.Value()) // Output: Value: 5
    
    // Call method with pointer receiver
    counter.Increment()
    fmt.Println("After increment:", counter.Value()) // Output: After increment: 6
    
    // Go permits calling pointer methods on values
    // This works, Go automatically takes the address
    c2 := Counter{count: 10}
    c2.Increment() // Go converts to (&c2).Increment()
    fmt.Println("c2 after increment:", c2.Value()) // Output: c2 after increment: 11
    
    // Nil pointer handling
    var c3 *Counter
    value := c3.IncrementSafely() // Handles nil safely
    fmt.Println("Nil counter value:", value) // Output: Nil counter value: 0
    
    // Create a slice of Counters
    counters := []Counter{
        {count: 1},
        {count: 2},
        {count: 3},
    }
    
    // This won't modify the original elements because we're iterating over copies
    for _, c := range counters {
        c.Increment() // Operates on a copy, not the original slice element
    }
    fmt.Println("Counters after loop:", counters[0].Value()) // Still 1
    
    // To modify elements, use a pointer or index
    for i := range counters {
        counters[i].Increment() // Modifies the element in the slice
    }
    fmt.Println("Counters after proper loop:", counters[0].Value()) // Now 2
}
```

**Real-World Example:**
Implementing a cache system with different receiver types:

```go
package cache

import (
    "sync"
    "time"
)

// Item represents a cached item
type Item struct {
    Value      interface{}
    Expiration int64
    Created    time.Time
}

// IsExpired checks if the item has expired
func (item Item) IsExpired() bool {
    if item.Expiration == 0 {
        return false
    }
    return time.Now().UnixNano() > item.Expiration
}

// Cache represents an in-memory cache
type Cache struct {
    items    map[string]Item
    mu       sync.RWMutex
    interval time.Duration
    stopCh   chan struct{}
}

// NewCache creates a new cache with the given cleanup interval
func NewCache(interval time.Duration) *Cache {
    cache := &Cache{
        items:    make(map[string]Item),
        interval: interval,
        stopCh:   make(chan struct{}),
    }
    
    // Start cleanup routine
    go cache.cleanupRoutine()
    
    return cache
}

// cleanupRoutine periodically removes expired items
func (c *Cache) cleanupRoutine() {
    ticker := time.NewTicker(c.interval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            c.DeleteExpired()
        case <-c.stopCh:
            return
        }
    }
}

// Set adds an item to the cache with an optional expiration time
func (c *Cache) Set(key string, value interface{}, duration time.Duration) {
    var expiration int64
    
    if duration > 0 {
        expiration = time.Now().Add(duration).UnixNano()
    }
    
    item := Item{
        Value:      value,
        Expiration: expiration,
        Created:    time.Now(),
    }
    
    c.mu.Lock()
    c.items[key] = item
    c.mu.Unlock()
}

// Get retrieves an item from the cache
func (c *Cache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    item, found := c.items[key]
    c.mu.RUnlock()
    
    if !found {
        return nil, false
    }
    
    if item.IsExpired() {
        // Item has expired but hasn't been removed yet
        c.mu.Lock()
        delete(c.items, key)
        c.mu.Unlock()
        return nil, false
    }
    
    return item.Value, true
}

// Delete removes an item from the cache
func (c *Cache) Delete(key string) {
    c.mu.Lock()
    delete(c.items, key)
    c.mu.Unlock()
}

// Count returns the number of items in the cache
func (c *Cache) Count() int {
    c.mu.RLock()
    count := len(c.items)
    c.mu.RUnlock()
    return count
}

// DeleteExpired removes all expired items from the cache
func (c *Cache) DeleteExpired() {
    now := time.Now().UnixNano()
    
    c.mu.Lock()
    for key, item := range c.items {
        if item.Expiration > 0 && now > item.Expiration {
            delete(c.items, key)
        }
    }
    c.mu.Unlock()
}

// Clear removes all items from the cache
func (c *Cache) Clear() {
    c.mu.Lock()
    c.items = make(map[string]Item)
    c.mu.Unlock()
}

// Stop halts the cleanup routine
func (c *Cache) Stop() {
    close(c.stopCh)
}

// ItemsSnapshot returns a copy of the current items map
// (Value receiver example - returns a copy without affecting the original)
func (c Cache) ItemsSnapshot() map[string]Item {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    snapshot := make(map[string]Item, len(c.items))
    for k, v := range c.items {
        snapshot[k] = v
    }
    
    return snapshot
}

// Usage example
func Example() {
    // Create a new cache with cleanup every 5 minutes
    cache := NewCache(5 * time.Minute)
    defer cache.Stop()
    
    // Add items to the cache
    cache.Set("key1", "value1", 1*time.Hour)
    cache.Set("key2", 42, 30*time.Second)
    cache.Set("key3", true, 0) // Never expires
    
    // Get items
    val1, found1 := cache.Get("key1")
    if found1 {
        fmt.Printf("Found key1: %v\n", val1)
    }
    
    val2, found2 := cache.Get("key2")
    if found2 {
        fmt.Printf("Found key2: %v\n", val2)
    }
    
    // Wait for key2 to expire
    time.Sleep(31 * time.Second)
    _, found2Again := cache.Get("key2")
    fmt.Printf("key2 found after expiry: %v\n", found2Again) // false
    
    // Get a snapshot
    snapshot := cache.ItemsSnapshot()
    fmt.Printf("Snapshot size: %d\n", len(snapshot))
    
    // Clear the cache
    cache.Clear()
    fmt.Printf("Items after clear: %d\n", cache.Count()) // 0
}
```

**Common Pitfalls:**
- Modifying values with value receivers (changes won't persist)
- Forgetting to handle nil pointers in pointer receiver methods
- Inconsistency in receiver types across a single type's methods
- Unnecessary pointer receivers for small struct types (adds garbage collection pressure)
- Not considering method sets when implementing interfaces
- Modifying fields of a value receiver (changes apply to the copy, not the original)
- Not understanding when Go automatically converts between value and pointer for method calls

**Confusion Questions:**

1. **Q: If I have a pointer receiver method, can I call it on a value, and vice versa?**
   
   A: Go provides convenient automatic conversions in some cases, but there are important rules to understand:
   
   **1. Calling a pointer receiver method on a value**:
   
   ```go
   type Person struct {
       Name string
   }
   
   func (p *Person) SetName(name string) {
       p.Name = name  // Modifies the receiver
   }
   
   func main() {
       person := Person{Name: "Alice"}
       person.SetName("Bob")  // Go automatically takes the address (&person)
       fmt.Println(person.Name)  // "Bob"
   }
   ```
   
   This works because Go automatically gets the address of the value. However, this only works if the value is addressable. For example, this doesn't work:
   
   ```go
   func getPerson() Person {
       return Person{Name: "Alice"}
   }
   
   getPerson().SetName("Bob")  // Error: can't take the address of a temporary value
   ```
   
   **2. Calling a value receiver method on a pointer**:
   
   ```go
   func (p Person) GetName() string {
       return p.Name
   }
   
   func main() {
       personPtr := &Person{Name: "Charlie"}
       name := personPtr.GetName()  // Go automatically dereferences the pointer
       fmt.Println(name)  // "Charlie"
   }
   ```
   
   This works because Go automatically dereferences the pointer for you.
   
   **3. Method sets**:
   
   When it comes to interfaces, these automatic conversions don't apply:
   
   ```go
   type Namer interface {
       GetName() string
   }
   
   type Editor interface {
       SetName(string)
   }
   
   func main() {
       var p Person = Person{Name: "Dave"}
       var pPtr *Person = &Person{Name: "Eve"}
       
       var n1 Namer = p      // Works: value satisfies Namer interface
       var n2 Namer = pPtr   // Works: pointer satisfies Namer interface
       
       var e1 Editor = pPtr  // Works: pointer satisfies Editor interface
       // var e2 Editor = p  // Error: Person does not implement Editor (SetName has pointer receiver)
   }
   ```
   
   The key rule is:
   - Methods with value receivers can be called on both values and pointers
   - Methods with pointer receivers can only be called on pointers or addressable values
   - For interface satisfaction, a type only implements an interface if all methods of the interface are in the type's method set

2. **Q: Why does modifying a value inside a value receiver method not affect the original value?**
   
   A: When you use a value receiver, Go creates a copy of the value for the method to work with. Any modifications to this copy don't affect the original value:
   
   ```go
   type Counter struct {
       count int
   }
   
   // Value receiver method - works on a copy
   func (c Counter) Increment() {
       c.count++ // Modifies the copy, not the original
   }
   
   func main() {
       counter := Counter{count: 5}
       counter.Increment()
       fmt.Println(counter.count) // Still 5, not 6
   }
   ```
   
   This happens because of Go's pass-by-value semantics. When the `Increment` method is called, Go creates a copy of the `counter` value and passes it to the method. The method then increments the `count` field of this copy, not the original `counter` variable.
   
   To modify the original value, you must use a pointer receiver:
   
   ```go
   // Pointer receiver method - works on the original
   func (c *Counter) Increment() {
       c.count++ // Modifies the original counter
   }
   
   func main() {
       counter := Counter{count: 5}
       counter.Increment()
       fmt.Println(counter.count) // Now 6
   }
   ```
   
   With a pointer receiver, Go passes the address of the original value to the method. The method then follows this address to modify the original value.
   
   This behavior is consistent with how parameters work in regular functions:
   
   ```go
   // Function with value parameter
   func incrementValue(c Counter) {
       c.count++ // Modifies the copy, not the original
   }
   
   // Function with pointer parameter
   func incrementPointer(c *Counter) {
       c.count++ // Modifies the original
   }
   ```
   
   Understanding this distinction is crucial for writing methods that behave as expected.

3. **Q: How do I handle nil receivers in methods with pointer receivers?**
   
   A: When using pointer receivers, you need to explicitly check for nil receivers if there's a possibility that your method might be called on a nil pointer:
   
   ```go
   type Logger struct {
       Level string
       Output io.Writer
   }
   
   // Unsafe - will panic if l is nil
   func (l *Logger) LogUnsafe(message string) {
       fmt.Fprintf(l.Output, "[%s] %s\n", l.Level, message)
   }
   
   // Safe - handles nil receiver
   func (l *Logger) Log(message string) {
       if l == nil {
           // Handle nil logger case
           fmt.Println("[DEFAULT] " + message)
           return
       }
       
       if l.Output == nil {
           // Handle nil output case
           fmt.Println("[" + l.Level + "] " + message)
           return
       }
       
       fmt.Fprintf(l.Output, "[%s] %s\n", l.Level, message)
   }
   
   func main() {
       // Normal usage
       logger := &Logger{Level: "INFO", Output: os.Stdout}
       logger.Log("Normal log message")
       
       // Nil logger
       var nilLogger *Logger
       nilLogger.Log("This won't panic") // Uses nil handling logic
       // nilLogger.LogUnsafe("This will panic") // Would panic with nil pointer dereference
   }
   ```
   
   Here are some strategies for handling nil receivers:
   
   1. **Default behavior**:
      ```go
      if l == nil {
          // Provide default behavior
          return defaultValue
      }
      ```
   
   2. **No-op pattern**:
      ```go
      if l == nil {
          // Do nothing and return
          return
      }
      ```
   
   3. **Error return**:
      ```go
      func (l *Logger) LogWithError(message string) error {
          if l == nil {
              return errors.New("logger is nil")
          }
          // Normal operation
          return nil
      }
      ```
   
   4. **Using the nil receiver as a sentinel value**:
      ```go
      func (l *Logger) IsConfigured() bool {
          return l != nil && l.Output != nil
      }
      ```
   
   5. **Creating a NullObject implementation**:
      ```go
      var NullLogger = &Logger{Level: "NULL"} // Special logger that does nothing
      
      func GetLogger(name string) *Logger {
          logger := findLogger(name)
          if logger == nil {
              return NullLogger // Return null object instead of nil
          }
          return logger
      }
      ```
   
   Always consider nil receivers when working with pointer receiver methods, especially for public APIs where you don't control how your methods will be called.

### 3. Method Sets

**Concise Explanation:**
A method set is the set of methods that are available for a given type. The method set determines which interfaces a type satisfies. For a type `T`, the method set includes all methods with receiver type `T`. For a pointer type `*T`, the method set includes all methods with receiver type `T` or `*T`. This asymmetry is important for understanding interface satisfaction and method availability.

**Where to Use:**
- Understanding interface satisfaction rules
- Determining which interfaces a type implements
- Making design decisions about receiver types
- Implementing interfaces correctly
- Ensuring types satisfy required method sets
- Debugging issues with interface implementation

**Code Snippet:**
```go
package main

import "fmt"

// Interfaces to demonstrate method set rules
type Adder interface {
    Add(int)
}

type Valuer interface {
    Value() int
}

type Counter struct {
    count int
}

// Method with pointer receiver - modifies the receiver
func (c *Counter) Add(n int) {
    c.count += n
}

// Method with value receiver - just returns a value
func (c Counter) Value() int {
    return c.count
}

// Method with value receiver - returns a string representation
func (c Counter) String() string {
    return fmt.Sprintf("Counter: %d", c.count)
}

func main() {
    // Create a Counter value and pointer
    counter := Counter{count: 5}
    counterPtr := &Counter{count: 10}
    
    // Calling methods directly
    counter.Add(2)     // Auto-conversion to (&counter).Add(2)
    counterPtr.Add(3)
    
    fmt.Println(counter.Value())    // 7
    fmt.Println(counterPtr.Value()) // Auto-conversion to (*counterPtr).Value()
    
    // Interface satisfaction
    // 1. Pointer type satisfies both interfaces
    var adder Adder = counterPtr    // Works: *Counter has Add method
    var valuer1 Valuer = counterPtr // Works: *Counter has Value method (from Counter)
    
    adder.Add(5)
    fmt.Println(valuer1.Value()) // 15
    
    // 2. Value type satisfies only Valuer
    // var adder2 Adder = counter // Error: Counter does not implement Adder (Add method has pointer receiver)
    var valuer2 Valuer = counter // Works: Counter has Value method
    
    fmt.Println(valuer2.Value()) // 7
    
    // Method expressions - convert methods to regular functions
    // Value receiver method expression
    valueMethod := Counter.Value    // Type is func(Counter) int
    fmt.Println(valueMethod(counter)) // 7
    
    // Pointer receiver method expression
    addMethod := (*Counter).Add     // Type is func(*Counter, int)
    addMethod(counterPtr, 10)
    fmt.Println(counterPtr.Value()) // 25
    
    // Compile-time check for interface satisfaction
    // This will fail compilation if Counter doesn't implement fmt.Stringer
    var _ fmt.Stringer = counter // Counter satisfies fmt.Stringer because of String() method
    
    // This would fail compilation:
    // var _ Adder = counter // Counter doesn't satisfy Adder (only *Counter does)
}
```

**Real-World Example:**
Building a geometry system with method sets and interfaces:

```go
package geometry

import (
    "fmt"
    "math"
)

// Shape defines the interface for all geometric shapes
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Resizable defines an interface for shapes that can change size
type Resizable interface {
    Resize(factor float64)
}

// Movable defines an interface for shapes that can be moved
type Movable interface {
    Move(dx, dy float64)
}

// Point represents a point in 2D space
type Point struct {
    X, Y float64
}

// Move implements the Movable interface for Point
func (p *Point) Move(dx, dy float64) {
    p.X += dx
    p.Y += dy
}

// Distance calculates the distance between two points
func (p Point) Distance(other Point) float64 {
    dx := p.X - other.X
    dy := p.Y - other.Y
    return math.Sqrt(dx*dx + dy*dy)
}

// String implements the fmt.Stringer interface for Point
func (p Point) String() string {
    return fmt.Sprintf("(%g, %g)", p.X, p.Y)
}

// Rectangle represents a rectangle with width and height
type Rectangle struct {
    Origin Point
    Width  float64
    Height float64
}

// Area calculates the area of the rectangle
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Perimeter calculates the perimeter of the rectangle
func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// Move implements the Movable interface for Rectangle
func (r *Rectangle) Move(dx, dy float64) {
    r.Origin.Move(dx, dy)
}

// Resize implements the Resizable interface for Rectangle
func (r *Rectangle) Resize(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

// String implements the fmt.Stringer interface for Rectangle
func (r Rectangle) String() string {
    return fmt.Sprintf("Rectangle at %v with width=%g, height=%g",
        r.Origin, r.Width, r.Height)
}

// Circle represents a circle with a center and radius
type Circle struct {
    Center Point
    Radius float64
}

// Area calculates the area of the circle
func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

// Perimeter calculates the perimeter (circumference) of the circle
func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

// Move implements the Movable interface for Circle
func (c *Circle) Move(dx, dy float64) {
    c.Center.Move(dx, dy)
}

// Resize implements the Resizable interface for Circle
func (c *Circle) Resize(factor float64) {
    c.Radius *= factor
}

// String implements the fmt.Stringer interface for Circle
func (c Circle) String() string {
    return fmt.Sprintf("Circle at %v with radius=%g",
        c.Center, c.Radius)
}

// MovableShape combines both Shape and Movable interfaces
type MovableShape interface {
    Shape
    Movable
}

// ResizableShape combines Shape and Resizable interfaces
type ResizableShape interface {
    Shape
    Resizable
}

// FullFeaturedShape combines all shape interfaces
type FullFeaturedShape interface {
    Shape
    Movable
    Resizable
    fmt.Stringer
}

// PrintShapeInfo prints detailed information about a shape
func PrintShapeInfo(s Shape) {
    fmt.Printf("Area: %g\n", s.Area())
    fmt.Printf("Perimeter: %g\n", s.Perimeter())
    
    // Type assertions to check additional capabilities
    if movable, ok := s.(Movable); ok {
        fmt.Println("This shape is movable")
        movable.Move(1, 1)  // Move it by 1,1
    }
    
    if resizable, ok := s.(Resizable); ok {
        fmt.Println("This shape is resizable")
        resizable.Resize(1.5)  // Resize by 150%
    }
    
    if stringer, ok := s.(fmt.Stringer); ok {
        fmt.Printf("String representation: %s\n", stringer)
    }
    
    fmt.Println()
}

// Usage example
func Example() {
    // Create some shapes
    rect := Rectangle{
        Origin: Point{10, 10},
        Width:  5,
        Height: 3,
    }
    
    circle := Circle{
        Center: Point{0, 0},
        Radius: 5,
    }
    
    // Use them as Shapes
    shapes := []Shape{rect, circle}
    for _, shape := range shapes {
        fmt.Printf("Shape: %T\n", shape)
        fmt.Printf("Area: %g\n", shape.Area())
        fmt.Printf("Perimeter: %g\n", shape.Perimeter())
        fmt.Println()
    }
    
    // These will work with pointers
    movableShapes := []Movable{&rect, &circle}
    for _, shape := range movableShapes {
        shape.Move(1, 1)
    }
    
    resizableShapes := []Resizable{&rect, &circle}
    for _, shape := range resizableShapes {
        shape.Resize(2.0)
    }
    
    // FullFeaturedShape can only be satisfied by pointers to our shapes
    var fullRect FullFeaturedShape = &rect
    PrintShapeInfo(fullRect)
    
    // The following won't compile:
    // var badRect FullFeaturedShape = rect  // Error: Rectangle does not implement Resizable
    
    // Type assertions for runtime checks
    var genericShape Shape = &circle
    
    if fullShape, ok := genericShape.(FullFeaturedShape); ok {
        fmt.Println("This shape has all features!")
        fullShape.Move(5, 5)
        fullShape.Resize(0.5)
        fmt.Println(fullShape.String())
    }
}
```

**Common Pitfalls:**
- Not understanding that value types don't implement interfaces with pointer receiver methods
- Forgetting that pointer types implement interfaces with value receiver methods
- Attempting to use a method expression with incorrect receiver type
- Mixing value and pointer receivers inconsistently
- Not considering method sets when designing interfaces
- Forgetting that nil pointers are valid receiver values for pointer methods
- Confusion about which methods are in the method set of a given type

**Confusion Questions:**

1. **Q: Why doesn't a value type implement an interface with pointer receiver methods?**
   
   A: This is one of Go's most important rules for method sets and interface satisfaction. A value type `T` doesn't implement an interface that has methods with pointer receivers `*T` because it's impossible to get a pointer to an arbitrary value.
   
   Here's an example to illustrate:
   
   ```go
   type Modifier interface {
       Modify()
   }
   
   type Data struct {
       Value int
   }
   
   // Method with pointer receiver
   func (d *Data) Modify() {
       d.Value++
   }
   
   func ProcessModifier(m Modifier) {
       m.Modify()
   }
   
   func main() {
       d := Data{Value: 5}
       // ProcessModifier(d)  // Error: cannot use d (type Data) as type Modifier
                             // Data does not implement Modifier (Modify method has pointer receiver)
       
       // This works
       ProcessModifier(&d)
   }
   ```
   
   The reason for this restriction is that if Go allowed value types to implement interfaces with pointer methods, scenarios like this could occur:
   
   ```go
   func CreateAndProcess() {
       ProcessModifier(Data{Value: 5})  // Passing a temporary value
   }
   ```
   
   When `ProcessModifier` calls the `Modify` method, it would need a pointer to the value, but the value `Data{Value: 5}` is a temporary that may not exist in memory after the function call. Even if it did exist, modifying it would be pointless since it's a temporary copy.
   
   In contrast, pointer types `*T` implement interfaces with value receiver methods because the pointer can always be dereferenced to get the value:
   
   ```go
   type Printer interface {
       Print()
   }
   
   func (d Data) Print() {
       fmt.Println(d.Value)
   }
   
   var p Printer = &Data{Value: 10}  // This works fine
   ```
   
   This asymmetry in method sets is a deliberate design choice that enforces safety and predictability in Go's type system.

2. **Q: How can I ensure my type correctly implements an interface?**
   
   A: Go provides several ways to ensure that your type correctly implements an interface:
   
   **1. Use a compile-time check**:
   
   ```go
   type Reader interface {
       Read(p []byte) (n int, err error)
   }
   
   type MyReader struct {
       // fields
   }
   
   func (r *MyReader) Read(p []byte) (n int, err error) {
       // implementation
       return len(p), nil
   }
   
   // Compile-time check
   var _ Reader = (*MyReader)(nil)  // Will fail to compile if *MyReader doesn't implement Reader
   ```
   
   This trick uses a variable declaration with the blank identifier (`_`) and tries to assign a nil pointer of your type to the interface type. If your type doesn't implement the interface, the compiler will generate an error.
   
   **2. Use explicit interface implementation checks**:
   
   ```go
   func EnsureImplements() {
       // These will fail to compile if the types don't implement the interfaces
       var _ io.Reader = (*MyReader)(nil)
       var _ io.Writer = (*MyWriter)(nil)
       var _ io.Closer = (*MyCloser)(nil)
   }
   ```
   
   **3. Write tests that use interface values**:
   
   ```go
   func TestMyReaderImplementsReader(t *testing.T) {
       var r io.Reader = &MyReader{}  // Should compile
       
       // Test the implementation
       data := make([]byte, 10)
       n, err := r.Read(data)
       if err != nil {
           t.Errorf("unexpected error: %v", err)
       }
       if n != 10 {
           t.Errorf("expected to read 10 bytes, got %d", n)
       }
   }
   ```
   
   **4. Check interface implementation at runtime**:
   
   ```go
   func IsReader(v interface{}) bool {
       _, ok := v.(io.Reader)
       return ok
   }
   
   func TestMyReaderImplementsReader(t *testing.T) {
       reader := &MyReader{}
       if !IsReader(reader) {
           t.Error("MyReader does not implement io.Reader")
       }
   }
   ```
   
   **5. Use tools like `golint` and `go vet`**:
   
   These tools can help catch certain issues with method signatures that might prevent your type from correctly implementing an interface.
   
   **6. Documentation comments**:
   
   ```go
   // MyReader implements io.Reader
   type MyReader struct {
       // fields
   }
   ```
   
   While this doesn't enforce anything at compile time, it clearly documents your intent, making it easier to catch bugs during code review.
   
   The compile-time check (approach #1) is the most common and recommended approach because it catches errors early without adding runtime overhead.

3. **Q: What are method expressions and when are they useful?**
   
   A: Method expressions are a way to convert methods into regular functions. They're useful when you want to use methods as first-class values or when you need to separate the receiver from the method call.
   
   **Syntax and basic usage**:
   
   ```go
   type Point struct {
       X, Y float64
   }
   
   func (p Point) Distance(q Point) float64 {
       dx := p.X - q.X
       dy := p.Y - q.Y
       return math.Sqrt(dx*dx + dy*dy)
   }
   
   func (p *Point) Scale(factor float64) {
       p.X *= factor
       p.Y *= factor
   }
   
   func main() {
       // Method expressions - convert methods to regular functions
       
       // Value receiver method expression
       distance := Point.Distance  // Type is func(Point, Point) float64
       
       p1 := Point{1, 2}
       p2 := Point{4, 6}
       fmt.Println(distance(p1, p2))  // Equivalent to p1.Distance(p2)
       
       // Pointer receiver method expression
       scale := (*Point).Scale  // Type is func(*Point, float64)
       scale(&p1, 2)            // Equivalent to p1.Scale(2)
       fmt.Println(p1)          // {2 4}
   }
   ```
   
   **When method expressions are useful**:
   
   **1. Passing methods to higher-order functions**:
   
   ```go
   type Rect struct {
       Width, Height float64
   }
   
   func (r Rect) Area() float64 {
       return r.Width * r.Height
   }
   
   func (r Rect) Perimeter() float64 {
       return 2 * (r.Width + r.Height)
   }
   
   func MeasureRectangles(rects []Rect, measure func(Rect) float64) []float64 {
       results := make([]float64, len(rects))
       for i, rect := range rects {
           results[i] = measure(rect)
       }
       return results
   }
   
   func main() {
       rects := []Rect{{2, 3}, {3, 4}, {4, 5}}
       
       // Use method expressions to pass methods as arguments
       areas := MeasureRectangles(rects, Rect.Area)
       perimeters := MeasureRectangles(rects, Rect.Perimeter)
       
       fmt.Println("Areas:", areas)
       fmt.Println("Perimeters:", perimeters)
   }
   ```
   
   **2. Building method dispatch tables**:
   
   ```go
   type Calculator struct {
       // fields
   }
   
   func (c Calculator) Add(a, b float64) float64 { return a + b }
   func (c Calculator) Subtract(a, b float64) float64 { return a - b }
   func (c Calculator) Multiply(a, b float64) float64 { return a * b }
   func (c Calculator) Divide(a, b float64) float64 { return a / b }
   
   func main() {
       calc := Calculator{}
       
       // Create a method dispatch table
       operations := map[string]func(Calculator, float64, float64) float64{
           "add":      Calculator.Add,
           "subtract": Calculator.Subtract,
           "multiply": Calculator.Multiply,
           "divide":   Calculator.Divide,
       }
       
       // Use the dispatch table
       op := "multiply"
       if fn, ok := operations[op]; ok {
           result := fn(calc, 10, 5)
           fmt.Printf("%s: %.2f\n", op, result)
       }
   }
   ```
   
   **3. Partial application and currying**:
   
   ```go
   type Greeter struct {
       Prefix string
   }
   
   func (g Greeter) Greet(name string) string {
       return g.Prefix + ", " + name + "!"
   }
   
   func main() {
       greet := Greeter.Greet  // Method expression
       
       // Partial application - fix the receiver
       formalGreeter := Greeter{"Hello"}
       formalGreet := func(name string) string {
           return greet(formalGreeter, name)
       }
       
       // Use the partially applied function
       fmt.Println(formalGreet("Alice"))  // "Hello, Alice!"
   }
   ```
   
   Method expressions provide a bridge between Go's object-oriented and functional programming styles, allowing you to treat methods as regular functions that can be passed around, stored, and manipulated just like any other function value.

### 4. Interface Definition and Implementation

**Concise Explanation:**
Interfaces in Go define a set of method signatures that a type must implement to satisfy the interface. Unlike many object-oriented languages, Go interfaces are implemented implicitly—types satisfy interfaces by implementing all the required methods, without needing to declare that they implement the interface. This approach, known as "duck typing," provides loose coupling between packages and enables powerful patterns for abstraction and polymorphism.

**Where to Use:**
- Defining contracts between components
- Enabling polymorphic behavior
- Creating abstraction layers
- Facilitating unit testing with mock implementations
- Decoupling code from specific implementations
- Building composable and reusable components
- Defining common behaviors across different types

**Code Snippet:**
```go
// Interface definition
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Composite interface
type ReadWriter interface {
    Reader
    Writer
}

// Implementing interfaces
type FileHandler struct {
    filename string
    contents []byte
    position int
}

// Implement the Read method - satisfies Reader interface
func (f *FileHandler) Read(p []byte) (n int, err error) {
    if f.position >= len(f.contents) {
        return 0, io.EOF
    }
    
    n = copy(p, f.contents[f.position:])
    f.position += n
    return n, nil
}

// Implement the Write method - satisfies Writer interface
func (f *FileHandler) Write(p []byte) (n int, err error) {
    // Ensure capacity
    if f.position+len(p) > len(f.contents) {
        // Grow the buffer
        newContents := make([]byte, f.position+len(p))
        copy(newContents, f.contents)
        f.contents = newContents
    }
    
    // Copy data
    n = copy(f.contents[f.position:], p)
    f.position += n
    return n, nil
}

// Now FileHandler implicitly satisfies Reader, Writer, and ReadWriter

func main() {
    // Create a FileHandler
    file := &FileHandler{
        filename: "test.txt",
        contents: []byte{},
    }
    
    // Use it as a Writer
    var writer Writer = file
    writer.Write([]byte("Hello, World!"))
    
    // Reset position
    file.position = 0
    
    // Use it as a Reader
    var reader Reader = file
    data := make([]byte, 5)
    n, _ := reader.Read(data)
    fmt.Printf("Read %d bytes: %s\n", n, data) // "Read 5 bytes: Hello"
    
    // Use it as a ReadWriter
    var rw ReadWriter = file
    rw.Write([]byte(" Go!"))
    file.position = 0
    
    fullData := make([]byte, 100)
    n, _ = rw.Read(fullData)
    fmt.Printf("Read %d bytes: %s\n", n, fullData[:n]) // "Read 15 bytes: Hello, World! Go!"
    
    // Compile-time check for interface implementation
    var _ Reader = (*FileHandler)(nil)
    var _ Writer = (*FileHandler)(nil)
    var _ ReadWriter = (*FileHandler)(nil)
}
```

**Real-World Example:**
Building a payment processing system with interfaces:

```go
package payment

import (
    "errors"
    "fmt"
    "time"
)

// Money represents a monetary amount
type Money struct {
    Amount   float64
    Currency string
}

// String formats the monetary amount
func (m Money) String() string {
    return fmt.Sprintf("%.2f %s", m.Amount, m.Currency)
}

// PaymentMethod is the interface that all payment methods must implement
type PaymentMethod interface {
    Pay(amount Money) (TransactionID, error)
    Refund(transactionID TransactionID, amount Money) error
    Validate() error
}

// TransactionID uniquely identifies a payment transaction
type TransactionID string

// CreditCard represents a credit card payment method
type CreditCard struct {
    Number     string
    CVV        string
    ExpiryMonth int
    ExpiryYear  int
    CardholderName string
}

// Pay processes a credit card payment
func (c *CreditCard) Pay(amount Money) (TransactionID, error) {
    if err := c.Validate(); err != nil {
        return "", err
    }
    
    // In a real implementation, this would call a payment gateway
    fmt.Printf("Processing credit card payment for %s\n", amount)
    fmt.Printf("Card: %s (ending in %s)\n", c.CardholderName, c.Number[len(c.Number)-4:])
    
    // Generate a transaction ID
    txID := TransactionID(fmt.Sprintf("CC-%d", time.Now().UnixNano()))
    return txID, nil
}

// Refund processes a credit card refund
func (c *CreditCard) Refund(transactionID TransactionID, amount Money) error {
    if err := c.Validate(); err != nil {
        return err
    }
    
    // In a real implementation, this would call a payment gateway
    fmt.Printf("Processing credit card refund for %s\n", amount)
    fmt.Printf("Transaction ID: %s\n", transactionID)
    
    return nil
}

// Validate checks if the credit card is valid
func (c *CreditCard) Validate() error {
    if len(c.Number) < 13 || len(c.Number) > 19 {
        return errors.New("invalid card number length")
    }
    
    if len(c.CVV) < 3 || len(c.CVV) > 4 {
        return errors.New("invalid CVV")
    }
    
    // Check expiry date
    currentYear, currentMonth, _ := time.Now().Date()
    if c.ExpiryYear < currentYear || 
       (c.ExpiryYear == currentYear && c.ExpiryMonth < int(currentMonth)) {
        return errors.New("card has expired")
    }
    
    if c.CardholderName == "" {
        return errors.New("cardholder name is required")
    }
    
    return nil
}

// PayPal represents a PayPal payment method
type PayPal struct {
    Email    string
    Password string
    Token    string
}

// Pay processes a PayPal payment
func (p *PayPal) Pay(amount Money) (TransactionID, error) {
    if err := p.Validate(); err != nil {
        return "", err
    }
    
    // In a real implementation, this would call PayPal's API
    fmt.Printf("Processing PayPal payment for %s\n", amount)
    fmt.Printf("PayPal account: %s\n", p.Email)
    
    // Generate a transaction ID
    txID := TransactionID(fmt.Sprintf("PP-%d", time.Now().UnixNano()))
    return txID, nil
}

// Refund processes a PayPal refund
func (p *PayPal) Refund(transactionID TransactionID, amount Money) error {
    if err := p.Validate(); err != nil {
        return err
    }
    
    // In a real implementation, this would call PayPal's API
    fmt.Printf("Processing PayPal refund for %s\n", amount)
    fmt.Printf("Transaction ID: %s\n", transactionID)
    
    return nil
}

// Validate checks if the PayPal account is valid
func (p *PayPal) Validate() error {
    if p.Email == "" {
        return errors.New("email is required")
    }
    
    if p.Token == "" && p.Password == "" {
        return errors.New("either token or password is required")
    }
    
    return nil
}

// BankTransfer represents a bank transfer payment method
type BankTransfer struct {
    AccountName   string
    AccountNumber string
    BankCode      string
    Reference     string
}

// Pay processes a bank transfer payment
func (b *BankTransfer) Pay(amount Money) (TransactionID, error) {
    if err := b.Validate(); err != nil {
        return "", err
    }
    
    // In a real implementation, this would initiate a bank transfer
    fmt.Printf("Processing bank transfer payment for %s\n", amount)
    fmt.Printf("To account: %s (%s)\n", b.AccountName, b.AccountNumber)
    
    // Generate a transaction ID
    txID := TransactionID(fmt.Sprintf("BT-%d", time.Now().UnixNano()))
    return txID, nil
}

// Refund processes a bank transfer refund
func (b *BankTransfer) Refund(transactionID TransactionID, amount Money) error {
    if err := b.Validate(); err != nil {
        return err
    }
    
    // In a real implementation, this would initiate a bank refund
    fmt.Printf("Processing bank transfer refund for %s\n", amount)
    fmt.Printf("Transaction ID: %s\n", transactionID)
    
    return nil
}

// Validate checks if the bank transfer details are valid
func (b *BankTransfer) Validate() error {
    if b.AccountName == "" {
        return errors.New("account name is required")
    }
    
    if b.AccountNumber == "" {
        return errors.New("account number is required")
    }
    
    if b.BankCode == "" {
        return errors.New("bank code is required")
    }
    
    return nil
}

// PaymentProcessor handles payment processing using various payment methods
type PaymentProcessor struct {
    availableMethods map[string]PaymentMethod
    transactions     map[TransactionID]PaymentRecord
}

// PaymentRecord stores information about a processed payment
type PaymentRecord struct {
    Method        string
    Amount        Money
    TransactionID TransactionID
    Timestamp     time.Time
    Refunded      bool
    RefundAmount  Money
}

// NewPaymentProcessor creates a new payment processor
func NewPaymentProcessor() *PaymentProcessor {
    return &PaymentProcessor{
        availableMethods: make(map[string]PaymentMethod),
        transactions:     make(map[TransactionID]PaymentRecord),
    }
}

// RegisterPaymentMethod adds a payment method to the processor
func (p *PaymentProcessor) RegisterPaymentMethod(name string, method PaymentMethod) {
    p.availableMethods[name] = method
}

// ProcessPayment processes a payment using the specified method
func (p *PaymentProcessor) ProcessPayment(methodName string, amount Money) (TransactionID, error) {
    method, found := p.availableMethods[methodName]
    if !found {
        return "", fmt.Errorf("payment method '%s' not found", methodName)
    }
    
    txID, err := method.Pay(amount)
    if err != nil {
        return "", err
    }
    
    // Record the transaction
    p.transactions[txID] = PaymentRecord{
        Method:        methodName,
        Amount:        amount,
        TransactionID: txID,
        Timestamp:     time.Now(),
    }
    
    return txID, nil
}

// ProcessRefund processes a refund for a previous transaction
func (p *PaymentProcessor) ProcessRefund(transactionID TransactionID, amount Money) error {
    record, found := p.transactions[transactionID]
    if !found {
        return fmt.Errorf("transaction '%s' not found", transactionID)
    }
    
    if record.Refunded {
        return fmt.Errorf("transaction '%s' already refunded", transactionID)
    }
    
    if amount.Amount > record.Amount.Amount {
        return fmt.Errorf("refund amount (%s) exceeds original payment amount (%s)",
            amount, record.Amount)
    }
    
    method, found := p.availableMethods[record.Method]
    if !found {
        return fmt.Errorf("payment method '%s' no longer available", record.Method)
    }
    
    err := method.Refund(transactionID, amount)
    if err != nil {
        return err
    }
    
    // Update the transaction record
    record.Refunded = true
    record.RefundAmount = amount
    p.transactions[transactionID] = record
    
    return nil
}

// GetTransactionDetails returns details for a specific transaction
func (p *PaymentProcessor) GetTransactionDetails(transactionID TransactionID) (PaymentRecord, error) {
    record, found := p.transactions[transactionID]
    if !found {
        return PaymentRecord{}, fmt.Errorf("transaction '%s' not found", transactionID)
    }
    
    return record, nil
}

// Usage example
func Example() {
    // Create payment methods
    creditCard := &CreditCard{
        Number:         "4111111111111111",
        CVV:            "123",
        ExpiryMonth:    12,
        ExpiryYear:     2025,
        CardholderName: "John Doe",
    }
    
    paypal := &PayPal{
        Email: "john.doe@example.com",
        Token: "secure-token-123",
    }
    
    bankTransfer := &BankTransfer{
        AccountName:   "ACME Corp",
        AccountNumber: "12345678",
        BankCode:      "BANKCODE123",
        Reference:     "Invoice 12345",
    }
    
    // Create payment processor
    processor := NewPaymentProcessor()
    
    // Register payment methods
    processor.RegisterPaymentMethod("credit_card", creditCard)
    processor.RegisterPaymentMethod("paypal", paypal)
    processor.RegisterPaymentMethod("bank_transfer", bankTransfer)
    
    // Process payments
    payment := Money{Amount: 100.50, Currency: "USD"}
    
    // Credit card payment
    ccTxID, err := processor.ProcessPayment("credit_card", payment)
    if err != nil {
        fmt.Printf("Credit card payment error: %v\n", err)
    } else {
        fmt.Printf("Credit card payment successful. Transaction ID: %s\n", ccTxID)
    }
    
    // PayPal payment
    ppTxID, err := processor.ProcessPayment("paypal", payment)
    if err != nil {
        fmt.Printf("PayPal payment error: %v\n", err)
    } else {
        fmt.Printf("PayPal payment successful. Transaction ID: %s\n", ppTxID)
    }
    
    // Process refund
    refundAmount := Money{Amount: 50.25, Currency: "USD"}
    err = processor.ProcessRefund(ccTxID, refundAmount)
    if err != nil {
        fmt.Printf("Refund error: %v\n", err)
    } else {
        fmt.Printf("Refund processed successfully for transaction: %s\n", ccTxID)
    }
    
    // Get transaction details
    if txDetails, err := processor.GetTransactionDetails(ccTxID); err == nil {
        fmt.Printf("Transaction details: %+v\n", txDetails)
    }
}
```

**Common Pitfalls:**
- Forgetting that interfaces are satisfied implicitly (no "implements" keyword needed)
- Using pointer receivers when value receivers would work and vice versa
- Creating interfaces that are too large or complex
- Not handling nil interface values properly
- Assuming an interface value has a specific concrete type without checking
- Exporting interface methods but not the interface itself
- Embedding non-interface types in interface definitions (not allowed)
- Defining methods on interface types (not allowed)

**Confusion Questions:**

1. **Q: How does Go's implicit interface implementation differ from explicit interfaces in other languages?**
   
   A: Go's approach to interfaces differs significantly from languages like Java or C#:
   
   **Go's implicit interfaces**:
   ```go
   // Define an interface
   type Writer interface {
       Write([]byte) (int, error)
   }
   
   // Implement the interface (implicitly)
   type FileWriter struct {
       // fields
   }
   
   func (fw *FileWriter) Write(data []byte) (int, error) {
       // implementation
       return len(data), nil
   }
   
   // Use the interface
   var writer Writer = &FileWriter{}
   ```
   
   **Java's explicit interfaces** (for comparison):
   ```java
   // Define an interface
   interface Writer {
       int write(byte[] data) throws IOException;
   }
   
   // Implement the interface (explicitly)
   class FileWriter implements Writer {
       @Override
       public int write(byte[] data) throws IOException {
           // implementation
           return data.length;
       }
   }
   
   // Use the interface
   Writer writer = new FileWriter();
   ```
   
   **Key differences**:
   
   1. **Declaration vs. Implementation Focus**:
      - Go focuses on behavior: "If a type has these methods, it can be used wherever the interface is expected"
      - Java focuses on classification: "This type is declared as implementing this interface"
   
   2. **Decoupling**:
      - Go allows types to satisfy interfaces they didn't know about when created
      - Traditional OOP requires knowledge of the interface at implementation time
   
   3. **Retroactive Implementation**:
      - Go lets you create interfaces that existing types already satisfy
      - Java/C# require modifying existing types to implement new interfaces
   
   4. **Interface Evolution**:
      - Go can add methods to interfaces without breaking existing implementations
      - Java/C# require all implementers to be updated when interfaces change
   
   **Benefits of Go's approach**:
   
   - **Loose Coupling**: Types don't need to know about the interfaces they satisfy
   - **Adaptability**: New interfaces can be created for existing types
   - **Simplicity**: Less boilerplate code and no inheritance hierarchies
   - **Composition**: Interfaces can be composed of other interfaces easily
   
   **Example of Go's flexibility**:
   
   ```go
   // You can create an interface that matches the standard library's methods
   type MyReadCloser interface {
       Read([]byte) (int, error)
       Close() error
   }
   
   // os.File already satisfies this interface without any modification
   file, _ := os.Open("file.txt")
   var rc MyReadCloser = file // Works perfectly
   ```
   
   This power and flexibility allow for more modular, adaptable code but requires understanding Go's approach to interface satisfaction.

2. **Q: When should I define interfaces in my code, and how large should they be?**
   
   A: Interfaces are a powerful tool in Go, but there are important guidelines for when and how to define them:
   
   **When to define interfaces**:
   
   1. **When you have multiple implementations** of the same behavior:
      ```go
      type Storage interface {
          Save(data []byte) error
          Load() ([]byte, error)
      }
      
      // Can be implemented by FileStorage, S3Storage, MemoryStorage, etc.
      ```
   
   2. **At the point of use** (not necessarily with the implementation):
      ```go
      // In the package that needs the behavior, not the implementing package
      type Logger interface {
          Log(level string, message string)
      }
      
      func Process(data []byte, logger Logger) {
          // Use logger interface here
      }
      ```
   
   3. **For testing and mocking**:
      ```go
      // Real code uses interfaces that can be mocked in tests
      type EmailSender interface {
          Send(to, subject, body string) error
      }
      
      // In production code
      func NotifyUser(user User, sender EmailSender) {
          sender.Send(user.Email, "Notification", "Important message")
      }
      
      // In test code
      type MockEmailSender struct {
          SentEmails []Email
      }
      
      func (m *MockEmailSender) Send(to, subject, body string) error {
          m.SentEmails = append(m.SentEmails, Email{To: to, Subject: subject, Body: body})
          return nil
      }
      ```
   
   4. **To define behavioral contracts** between packages:
      ```go
      // Package database defines interfaces that drivers must implement
      type Driver interface {
          Connect(dsn string) (Connection, error)
      }
      
      type Connection interface {
          Query(query string, args ...interface{}) (Rows, error)
          Close() error
      }
      ```
   
   **Interface size guidelines**:
   
   The Go proverb "The bigger the interface, the weaker the abstraction" provides good guidance:
   
   1. **Prefer small interfaces** with focused behavior:
      ```go
      // Good: Focused on a single responsibility
      type Reader interface {
          Read(p []byte) (n int, err error)
      }
      
      type Writer interface {
          Write(p []byte) (n int, err error)
      }
      
      // Compose for more complex behaviors
      type ReadWriter interface {
          Reader
          Writer
      }
      ```
   
   2. **Follow the "interface segregation principle"**:
      ```go
      // Better to have multiple small interfaces
      type Opener interface {
          Open(name string) error
      }
      
      type Closer interface {
          Close() error
      }
      
      // Than one large one
      type FileHandler interface {
          Open(name string) error
          Close() error
          Read(p []byte) (n int, err error)
          Write(p []byte) (n int, err error)
          Seek(offset int64, whence int) (int64, error)
      }
      ```
   
   3. **Single method interfaces are often the most reusable**:
      ```go
      type Stringer interface {
          String() string
      }
      
      type Formatter interface {
          Format(f State, c rune)
      }
      ```
   
   4. **Let interfaces grow through composition**:
      ```go
      type Reader interface {
          Read(p []byte) (n int, err error)
      }
      
      type ReaderAt interface {
          ReadAt(p []byte, off int64) (n int, err error)
      }
      
      type ReadSeeker interface {
          Reader
          Seeker
      }
      ```
   
   In summary, interfaces should be defined where they're used (not with implementations), focus on specific behaviors, stay small, and grow through composition rather than large interface definitions.

3. **Q: How can I check if a type implements an interface at runtime?**
   
   A: Go provides several techniques for checking interface implementation at runtime:
   
   **1. Type assertions**:
   
   A type assertion provides access to an interface value's underlying concrete value:
   
   ```go
   // Check if value implements io.Reader
   func processReader(v interface{}) {
       if reader, ok := v.(io.Reader); ok {
           // v implements io.Reader, use reader
           data := make([]byte, 100)
           n, _ := reader.Read(data)
           fmt.Printf("Read %d bytes\n", n)
       } else {
           fmt.Println("Value does not implement io.Reader")
       }
   }
   ```
   
   **2. Type switches**:
   
   A type switch performs multiple type assertions in series:
   
   ```go
   func process(v interface{}) {
       switch x := v.(type) {
       case io.Reader:
           // v is an io.Reader
           data := make([]byte, 100)
           n, _ := x.Read(data)
           fmt.Printf("Read %d bytes\n", n)
           
       case io.Writer:
           // v is an io.Writer
           n, _ := x.Write([]byte("hello"))
           fmt.Printf("Wrote %d bytes\n", n)
           
       case io.Closer:
           // v is an io.Closer
           x.Close()
           fmt.Println("Closed")
           
       case fmt.Stringer:
           // v is a fmt.Stringer
           fmt.Println(x.String())
           
       default:
           fmt.Printf("Unsupported type: %T\n", v)
       }
   }
   ```
   
   **3. Reflect package**:
   
   For more advanced checks, Go's reflect package can be used:
   
   ```go
   func implementsReader(v interface{}) bool {
       // Get the type of io.Reader
       readerType := reflect.TypeOf((*io.Reader)(nil)).Elem()
       
       // Get the type of the value
       valueType := reflect.TypeOf(v)
       
       // Check if the value's type implements the interface
       return valueType.Implements(readerType)
   }
   
   func main() {
       file, _ := os.Open("file.txt")
       fmt.Println(implementsReader(file))          // true
       fmt.Println(implementsReader(&bytes.Buffer{})) // true
       fmt.Println(implementsReader("string"))      // false
   }
   ```
   
   **4. Helper function with type assertions**:
   
   ```go
   // IsReader checks if a value implements io.Reader
   func IsReader(v interface{}) bool {
       _, ok := v.(io.Reader)
       return ok
   }
   
   // IsWriteCloser checks if a value implements both Writer and Closer
   func IsWriteCloser(v interface{}) bool {
       _, ok := v.(io.WriteCloser)
       return ok
   }
   
   func main() {
       var x interface{} = &bytes.Buffer{}
       fmt.Println(IsReader(x))        // true
       fmt.Println(IsWriteCloser(x))   // true
   }
   ```
   
   **Important considerations**:
   
   1. **Performance**: Type assertions and type switches are fairly efficient, but the reflect package has more overhead.
   
   2. **Zero values**: Be careful about checking nil pointers:
      ```go
      var f *os.File // nil pointer
      // This is true! A nil *os.File still implements io.Reader
      fmt.Println(implementsReader(f)) // true
      
      // But using it would cause a nil pointer panic
      if r, ok := f.(io.Reader); ok {
          // r is not nil, but the underlying value is
          data := make([]byte, 10)
          r.Read(data) // PANIC: nil pointer dereference
      }
      ```
   
   3. **Interface vs. Implementation**: A value might implement an interface but be a nil pointer:
      ```go
      var w io.Writer
      var f *os.File // nil pointer
      
      w = f // This works! w is a non-nil interface with a nil pointer value
      
      if w != nil {
          // This condition is true because the interface is not nil
          w.Write([]byte("hello")) // But this will panic
      }
      ```
   
   For safe runtime checks of interface implementation, always check both that a value implements an interface and that the underlying value isn't nil if it's a pointer type.

### 5. The Empty Interface

**Concise Explanation:**
The empty interface, `interface{}` (or just `any` in Go 1.18+), is an interface type with no methods. Since every type implements at least zero methods, all types satisfy the empty interface. This makes it a generic container that can hold values of any type, serving as Go's approach to generic programming before the introduction of parameterized types in Go 1.18.

**Where to Use:**
- When a function needs to accept arguments of any type
- For containers that need to store mixed types
- When working with dynamic or unknown data (like JSON)
- For plugins or extension systems with unknown types
- When implementing generic algorithms
- As a placeholder for values whose type will be determined at runtime
- With type assertions to recover the concrete type

**Code Snippet:**
```go
// Function that takes any type
func PrintAny(v interface{}) {
    fmt.Printf("Value: %v, Type: %T\n", v, v)
}

// Slice that can hold elements of any type
func MixedSlice() {
    var mixed []interface{}
    
    // Add various types
    mixed = append(mixed, 42)              // int
    mixed = append(mixed, "hello")         // string
    mixed = append(mixed, 3.14)            // float64
    mixed = append(mixed, struct{}{})      // empty struct
    mixed = append(mixed, []int{1, 2, 3})  // slice of int
    
    // Process each item
    for i, item := range mixed {
        fmt.Printf("mixed[%d] is a %T with value %v\n", i, item, item)
    }
}

// Map using interface{} values
func ConfigMap() {
    // Map with string keys and values of any type
    config := map[string]interface{}{
        "port":      8080,
        "debug":     true,
        "rates":     []float64{0.1, 0.2, 0.3},
        "timeout":   30 * time.Second,
        "handlers":  map[string]string{"get": "/api/get", "post": "/api/post"},
    }
    
    // Accessing values requires type assertions
    if port, ok := config["port"].(int); ok {
        fmt.Printf("Port number: %d\n", port)
    }
    
    if rates, ok := config["rates"].([]float64); ok {
        for _, rate := range rates {
            fmt.Printf("Rate: %.2f\n", rate)
        }
    }
    
    // This will fail since "timeout" is not a string
    if timeout, ok := config["timeout"].(string); ok {
        fmt.Printf("Timeout: %s\n", timeout)
    } else {
        fmt.Println("Timeout is not a string")
        // Use type switch to handle various possibilities
        switch v := config["timeout"].(type) {
        case int:
            fmt.Printf("Timeout is an integer: %d\n", v)
        case time.Duration:
            fmt.Printf("Timeout is a duration: %s\n", v)
        default:
            fmt.Printf("Timeout is some other type: %T\n", v)
        }
    }
}

func main() {
    // Using PrintAny with different types
    PrintAny(123)
    PrintAny("hello")
    PrintAny([]string{"a", "b", "c"})
    
    MixedSlice()
    ConfigMap()
    
    // Container for mixed types
    var data interface{}
    
    // Store an int
    data = 42
    if v, ok := data.(int); ok {
        fmt.Printf("data contains int: %d\n", v)
    }
    
    // Store a struct
    type Person struct {
        Name string
        Age  int
    }
    data = Person{"Alice", 30}
    
    // Type assertion to access the struct
    if p, ok := data.(Person); ok {
        fmt.Printf("Person: %s is %d years old\n", p.Name, p.Age)
    }
    
    // Type switch for multiple possibilities
    switch v := data.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    case Person:
        fmt.Printf("Person: %s, %d\n", v.Name, v.Age)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}
```

**Real-World Example:**
Building a flexible configuration system using the empty interface:

```go
package config

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "os"
    "reflect"
    "strconv"
    "strings"
    "time"
)

// Config represents a flexible configuration store that can handle
// values of any type and supports environment variable overrides
type Config struct {
    values map[string]interface{}
}

// NewConfig creates an empty configuration
func NewConfig() *Config {
    return &Config{
        values: make(map[string]interface{}),
    }
}

// LoadFromJSON loads configuration from a JSON file
func (c *Config) LoadFromJSON(filename string) error {
    data, err := ioutil.ReadFile(filename)
    if err != nil {
        return err
    }
    
    return json.Unmarshal(data, &c.values)
}

// Override overrides configuration with environment variables
func (c *Config) Override() {
    // Process all environment variables
    for _, env := range os.Environ() {
        // Only process variables with our prefix
        if !strings.HasPrefix(env, "CONFIG_") {
            continue
        }
        
        // Split into key and value
        parts := strings.SplitN(env, "=", 2)
        if len(parts) != 2 {
            continue
        }
        
        // Convert CONFIG_KEY_NAME to key.name
        key := strings.TrimPrefix(parts[0], "CONFIG_")
        key = strings.ToLower(key)
        key = strings.ReplaceAll(key, "_", ".")
        
        // Set the value
        c.Set(key, parts[1])
    }
}

// Set sets a configuration value
func (c *Config) Set(key string, value interface{}) {
    // Handle nested keys like "database.host"
    keys := strings.Split(key, ".")
    
    // Navigate to the correct nested map
    current := c.values
    for i := 0; i < len(keys)-1; i++ {
        k := keys[i]
        
        // If this level doesn't exist, create it
        if _, exists := current[k]; !exists {
            current[k] = make(map[string]interface{})
        }
        
        // If it's not a map, make it one (replacing the existing value)
        nest, ok := current[k].(map[string]interface{})
        if !ok {
            nest = make(map[string]interface{})
            current[k] = nest
        }
        
        current = nest
    }
    
    // Set the final value
    lastKey := keys[len(keys)-1]
    current[lastKey] = value
}

// Get retrieves a configuration value
func (c *Config) Get(key string) interface{} {
    keys := strings.Split(key, ".")
    
    // Navigate nested maps
    var current interface{} = c.values
    for _, k := range keys {
        // Get the map at this level
        m, ok := current.(map[string]interface{})
        if !ok {
            return nil
        }
        
        // Get the value at this level
        value, exists := m[k]
        if !exists {
            return nil
        }
        
        current = value
    }
    
    return current
}

// GetString returns a configuration value as a string
func (c *Config) GetString(key string, defaultValue string) string {
    value := c.Get(key)
    if value == nil {
        return defaultValue
    }
    
    // Handle different types
    switch v := value.(type) {
    case string:
        return v
    case int, int64, float64, bool:
        return fmt.Sprintf("%v", v)
    default:
        return defaultValue
    }
}

// GetInt returns a configuration value as an integer
func (c *Config) GetInt(key string, defaultValue int) int {
    value := c.Get(key)
    if value == nil {
        return defaultValue
    }
    
    // Handle different types
    switch v := value.(type) {
    case int:
        return v
    case int64:
        return int(v)
    case float64:
        return int(v)
    case string:
        if i, err := strconv.Atoi(v); err == nil {
            return i
        }
    }
    
    return defaultValue
}

// GetBool returns a configuration value as a boolean
func (c *Config) GetBool(key string, defaultValue bool) bool {
    value := c.Get(key)
    if value == nil {
        return defaultValue
    }
    
    // Handle different types
    switch v := value.(type) {
    case bool:
        return v
    case string:
        lv := strings.ToLower(v)
        return lv == "true" || lv == "yes" || lv == "1"
    case int:
        return v != 0
    }
    
    return defaultValue
}

// GetDuration returns a configuration value as a time.Duration
func (c *Config) GetDuration(key string, defaultValue time.Duration) time.Duration {
    value := c.Get(key)
    if value == nil {
        return defaultValue
    }
    
    // Handle different types
    switch v := value.(type) {
    case time.Duration:
        return v
    case int:
        return time.Duration(v) * time.Second
    case float64:
        return time.Duration(v * float64(time.Second))
    case string:
        if d, err := time.ParseDuration(v); err == nil {
            return d
        }
        // Try parsing as seconds
        if i, err := strconv.Atoi(v); err == nil {
            return time.Duration(i) * time.Second
        }
    }
    
    return defaultValue
}

// GetStringSlice returns a configuration value as a string slice
func (c *Config) GetStringSlice(key string, defaultValue []string) []string {
    value := c.Get(key)
    if value == nil {
        return defaultValue
    }
    
    // Handle different types
    switch v := value.(type) {
    case []string:
        return v
    case []interface{}:
        result := make([]string, len(v))
        for i, item := range v {
            result[i] = fmt.Sprintf("%v", item)
        }
        return result
    case string:
        if v == "" {
            return []string{}
        }
        return strings.Split(v, ",")
    }
    
    return defaultValue
}

// Bind binds configuration values to a struct
func (c *Config) Bind(prefix string, target interface{}) error {
    v := reflect.ValueOf(target)
    if v.Kind() != reflect.Ptr || v.Elem().Kind() != reflect.Struct {
        return fmt.Errorf("target must be a pointer to a struct")
    }
    
    v = v.Elem()
    t := v.Type()
    
    for i := 0; i < v.NumField(); i++ {
        field := v.Field(i)
        fieldType := t.Field(i)
        
        // Skip unexported fields
        if !field.CanSet() {
            continue
        }
        
        // Get the field tag or use the field name
        key := fieldType.Tag.Get("config")
        if key == "" {
            key = strings.ToLower(fieldType.Name)
        }
        
        // Prepend prefix if provided
        if prefix != "" {
            key = prefix + "." + key
        }
        
        // Get value from config
        value := c.Get(key)
        if value == nil {
            continue
        }
        
        // Set the field based on its type
        switch field.Kind() {
        case reflect.String:
            if str, ok := value.(string); ok {
                field.SetString(str)
            }
        case reflect.Int, reflect.Int64:
            var intValue int64
            switch v := value.(type) {
            case int:
                intValue = int64(v)
            case int64:
                intValue = v
            case float64:
                intValue = int64(v)
            case string:
                if i, err := strconv.ParseInt(v, 10, 64); err == nil {
                    intValue = i
                }
            }
            field.SetInt(intValue)
        case reflect.Bool:
            if b, ok := c.GetBool(key, false).(bool); ok {
                field.SetBool(b)
            }
        case reflect.Float64:
            if f, ok := value.(float64); ok {
                field.SetFloat(f)
            }
        case reflect.Slice:
            if strs, ok := c.GetStringSlice(key, nil); ok && field.Type().Elem().Kind() == reflect.String {
                sliceValue := reflect.MakeSlice(field.Type(), len(strs), len(strs))
                for i, str := range strs {
                    sliceValue.Index(i).SetString(str)
                }
                field.Set(sliceValue)
            }
        case reflect.Struct:
            // Recursively bind nested structs
            if field.Type() == reflect.TypeOf(time.Duration(0)) {
                if d, ok := c.GetDuration(key, 0).(time.Duration); ok {
                    field.Set(reflect.ValueOf(d))
                }
            } else if field.CanAddr() {
                c.Bind(key, field.Addr().Interface())
            }
        case reflect.Map:
            if m, ok := value.(map[string]interface{}); ok && field.Type().Key().Kind() == reflect.String {
                mapValue := reflect.MakeMap(field.Type())
                for k, v := range m {
                    mapValue.SetMapIndex(reflect.ValueOf(k), reflect.ValueOf(v))
                }
                field.Set(mapValue)
            }
        }
    }
    
    return nil
}

// Usage example
func Example() {
    // Create a new configuration
    cfg := NewConfig()
    
    // Load configuration from a JSON file
    err := cfg.LoadFromJSON("config.json")
    if err != nil {
        fmt.Printf("Error loading config: %v\n", err)
        return
    }
    
    // Override with environment variables
    cfg.Override()
    
    // Use the configuration
    serverPort := cfg.GetInt("server.port", 8080)
    debug := cfg.GetBool("debug", false)
    timeout := cfg.GetDuration("timeout", 30*time.Second)
    
    fmt.Printf("Server port: %d\n", serverPort)
    fmt.Printf("Debug mode: %t\n", debug)
    fmt.Printf("Timeout: %s\n", timeout)
    
    // Using nested values
    dbHost := cfg.GetString("database.host", "localhost")
    dbPort := cfg.GetInt("database.port", 5432)
    
    fmt.Printf("Database: %s:%d\n", dbHost, dbPort)
    
    // Bind to a struct
    type ServerConfig struct {
        Port    int           `config:"port"`
        Host    string        `config:"host"`
        Timeout time.Duration `config:"timeout"`
        Debug   bool          `config:"debug"`
    }
    
    var serverConfig ServerConfig
    err = cfg.Bind("server", &serverConfig)
    if err != nil {
        fmt.Printf("Error binding config: %v\n", err)
        return
    }
    
    fmt.Printf("Server config: %+v\n", serverConfig)
    
    // Set a new value
    cfg.Set("feature.experimental", true)
    
    fmt.Printf("Experimental features: %t\n", cfg.GetBool("feature.experimental", false))
}
```

**Common Pitfalls:**
- Type assertions without the "comma ok" check leading to panics
- Forgetting what type is stored in the empty interface
- Returning interface{} from functions without documentation about the actual type
- Excessive use of empty interfaces leading to "stringly typed" code
- Loss of compile-time type safety
- Unnecessary use of interface{} when concrete types or more specific interfaces would work
- Not handling all possible types in a type switch
- Performance overhead from type assertions and boxing/unboxing

**Confusion Questions:**

1. **Q: When should I use interface{} versus a specific interface?**
   
   A: The choice between `interface{}` (or `any` in Go 1.18+) and specific interfaces is a trade-off between flexibility and type safety:
   
   **Use a specific interface when**:
   
   1. **You know the behaviors needed**, not the concrete types:
      ```go
      // Better than interface{} because it defines required behavior
      func Copy(dst io.Writer, src io.Reader) (int64, error) {
          // Uses specific methods from the interfaces
          return io.Copy(dst, src)
      }
      ```
   
   2. **The code needs to work with multiple implementations** that share methods:
      ```go
      type Logger interface {
          Log(message string)
      }
      
      // Can accept any type that has a Log method
      func Process(data []byte, logger Logger) {
          logger.Log("Processing data...")
      }
      ```
   
   3. **You want compile-time type safety**:
      ```go
      // This will catch type errors at compile time
      func Sort(data sort.Interface) {
          // Can only call methods defined in sort.Interface
          sort.Sort(data)
      }
      ```
   
   4. **You want to express a contract that multiple types can implement**:
      ```go
      type Validator interface {
          Validate() error
      }
      ```
   
   **Use interface{} when**:
   
   1. **The type is truly unknown** or could be anything:
      ```go
      // Must handle any data type from JSON
      func ParseJSON(data []byte) (interface{}, error) {
          var result interface{}
          err := json.Unmarshal(data, &result)
          return result, err
      }
      ```
   
   2. **You're implementing a container that needs to store heterogeneous types**:
      ```go
      type Any struct {
          Value interface{}
      }
      
      func NewAny(v interface{}) *Any {
          return &Any{Value: v}
      }
      ```
   
   3. **The available types can't share a more specific interface**:
      ```go
      // These types don't have common methods
      func PrintAny(v interface{}) {
          fmt.Printf("%v\n", v)
      }
      ```
   
   4. **For dynamic behavior similar to reflection**:
      ```go
      func Call(obj interface{}, method string, args ...interface{}) {
          // Use reflection to call the method dynamically
      }
      ```
   
   **Decision guidelines**:
   
   1. **Start specific**: Begin with concrete types or narrow interfaces
   2. **Expand as needed**: Only generalize to `interface{}` when necessary
   3. **Consider code boundaries**: Use `interface{}` at API boundaries that must accept any type
   4. **Document expectations**: Always document what types your functions expect when using `interface{}`
   5. **Type assertions**: Use type assertions defensively with the "comma ok" idiom
   
   Here's a comparison of approaches:
   
   ```go
   // Bad: Too generic
   func ProcessData(data interface{}) {
       // Type switches or assertions needed
   }
   
   // Better: Specific interface
   func ProcessData(data Processor) {
       data.Process()
   }
   
   // Also good: Generic since Go 1.18
   func ProcessData[T any](data T) {
       // Type-safe operations on T
   }
   ```
   
   In Go 1.18 and later, consider using generics instead of `interface{}` for type-safe generic programming.

2. **Q: Why does a type assertion for an interface value sometimes panic?**
   
   A: Type assertions can panic if not used correctly. Understanding the potential panic scenarios helps you write more robust code.
   
   **Case 1: Missing "comma ok" check**:
   
   ```go
   func process(v interface{}) {
       // DANGEROUS: This will panic if v is not a string
       s := v.(string)
       fmt.Println(s)
   }
   
   process(42) // PANIC: interface conversion: interface {} is int, not string
   ```
   
   The safe way is to use the "comma ok" idiom:
   
   ```go
   func processSafely(v interface{}) {
       s, ok := v.(string)
       if ok {
           fmt.Println(s)
       } else {
           fmt.Printf("Not a string: %T\n", v)
       }
   }
   
   processSafely(42) // Prints: "Not a string: int"
   ```
   
   **Case 2: Asserting nil interface to concrete type**:
   
   ```go
   var v interface{} // v is nil
   
   // DANGEROUS: This will panic
   s := v.(string)
   fmt.Println(s)
   ```
   
   Again, use the "comma ok" check:
   
   ```go
   if s, ok := v.(string); ok {
       fmt.Println(s)
   } else {
       fmt.Println("Not a string or nil value")
   }
   ```
   
   **Case 3: Interface holds a nil pointer but is not nil itself**:
   
   This is subtle and can be confusing:
   
   ```go
   type MyReader struct{}
   func (m *MyReader) Read(p []byte) (int, error) {
       return 0, nil
   }
   
   func main() {
       var r io.Reader
       var m *MyReader // nil pointer
       
       r = m // r is NOT nil, it's an interface value containing a nil pointer
       
       if r != nil {
           // This condition is TRUE!
           fmt.Println("r is not nil")
           
           // But this will PANIC
           r.Read(make([]byte, 10)) // Panic: nil pointer dereference
       }
   }
   ```
   
   The safe way to handle this is to check both the interface and the concrete value:
   
   ```go
   if r != nil {
       // Check if r contains a nil MyReader pointer
       if mr, ok := r.(*MyReader); ok && mr == nil {
           fmt.Println("r contains a nil MyReader pointer")
       } else {
           // Safe to use
           r.Read(make([]byte, 10))
       }
   }
   ```
   
   **Best practices for safe type assertions**:
   
   1. **Always use "comma ok" checks**:
      ```go
      if value, ok := v.(Type); ok {
          // Safe to use value
      }
      ```
   
   2. **Use type switches with a default case**:
      ```go
      switch x := v.(type) {
      case string:
          // Handle string
      case int:
          // Handle int
      default:
          // Handle other types
      }
      ```
   
   3. **Check for nil before using**:
      ```go
      if ptr, ok := v.(*MyType); ok {
          if ptr != nil {
              // Safe to use ptr
          }
      }
      ```
   
   4. **Consider helper functions**:
      ```go
      // SafeString gets a string value or a default
      func SafeString(v interface{}, def string) string {
          if s, ok := v.(string); ok {
              return s
          }
          return def
      }
      ```
   
   By following these practices, you can avoid the most common causes of type assertion panics.

3. **Q: How do type switches work and when should I use them?**
   
   A: Type switches provide a concise way to check the actual type of an interface value and handle each possibility differently. They're especially useful when dealing with values of unknown or multiple possible types.
   
   **Basic syntax**:
   
   ```go
   func process(v interface{}) {
       switch x := v.(type) {
       case nil:
           // v is nil
           fmt.Println("nil value")
       case int:
           // x is an int
           fmt.Printf("Integer: %d\n", x)
       case float64:
           // x is a float64
           fmt.Printf("Float: %.2f\n", x)
       case string:
           // x is a string
           fmt.Printf("String: %s\n", x)
       case []byte:
           // x is a byte slice
           fmt.Printf("Byte slice of length %d\n", len(x))
       case error:
           // x is an error
           fmt.Printf("Error: %v\n", x)
       default:
           // None of the above types
           fmt.Printf("Unknown type: %T\n", x)
       }
   }
   ```
   
   **When to use type switches**:
   
   1. **Processing mixed data from external sources**:
      ```go
      // Processing JSON data
      func processJSON(data interface{}) {
          switch v := data.(type) {
          case map[string]interface{}:
              // Handle JSON object
              for key, value := range v {
                  fmt.Printf("Key: %s, Value: %v\n", key, value)
              }
          case []interface{}:
              // Handle JSON array
              for i, item := range v {
                  fmt.Printf("Item %d: %v\n", i, item)
              }
          case string:
              // Handle JSON string
              fmt.Printf("String value: %s\n", v)
          case float64:
              // Handle JSON number
              fmt.Printf("Number value: %.2f\n", v)
          case bool:
              // Handle JSON boolean
              fmt.Printf("Boolean value: %t\n", v)
          case nil:
              // Handle JSON null
              fmt.Println("Null value")
          default:
              fmt.Printf("Unexpected type: %T\n", v)
          }
      }
      ```
   
   2. **Handling multiple interface implementations**:
      ```go
      func processResource(r Resource) {
          switch res := r.(type) {
          case *File:
              // Handle file resource
              fmt.Printf("File: %s\n", res.Path)
          case *HTTPResource:
              // Handle HTTP resource
              fmt.Printf("URL: %s\n", res.URL)
          case *DatabaseResource:
              // Handle database resource
              fmt.Printf("Query: %s\n", res.Query)
          default:
              fmt.Printf("Unknown resource type: %T\n", res)
          }
      }
      ```
   
   3. **Type-specific operations without multiple function variants**:
      ```go
      func formatValue(v interface{}) string {
          switch val := v.(type) {
          case time.Time:
              return val.Format("2006-01-02")
          case time.Duration:
              return val.String()
          case fmt.Stringer:
              return val.String()
          case error:
              return "Error: " + val.Error()
          case []byte:
              return string(val)
          default:
              return fmt.Sprintf("%v", val)
          }
      }
      ```
   
   4. **Hierarchical checking** (checking interfaces before concrete types):
      ```go
      func describe(v interface{}) {
          switch x := v.(type) {
          // Check interfaces first
          case io.ReadWriter:
              fmt.Println("This is both a reader and writer")
              // Further distinguish if needed
              if _, ok := x.(io.Closer); ok {
                  fmt.Println("It's also a closer")
              }
          case io.Reader:
              fmt.Println("This is a reader")
          case io.Writer:
              fmt.Println("This is a writer")
              
          // Then check concrete types
          case *bytes.Buffer:
              fmt.Println("This is a bytes buffer")
          case *strings.Reader:
              fmt.Println("This is a strings reader")
          default:
              fmt.Printf("Unknown type: %T\n", x)
          }
      }
      ```
   
   **Key considerations**:
   
   1. **Order matters**: More specific types should come before more general ones
   2. **Interface checks**: You can check for interface satisfaction in a type switch
   3. **Variable scope**: The variable in each case is scoped only to that case block
   4. **Performance**: Type switches are generally efficient and often clearer than multiple type assertions
   5. **Exhaustiveness**: Use the default case to handle unexpected types
   
   Type switches provide a clean, idiomatic way to handle different types without excessive if-else chains of type assertions. They're particularly valuable when working with data of dynamic or unknown types.

### 6. Interface Values and Nil Interfaces

**Concise Explanation:**
An interface value in Go consists of two components: a concrete type and a value of that type. It's important to understand that an interface is only nil if both its type and value are nil. This distinction leads to sometimes surprising behavior when working with nil pointers assigned to interfaces, as an interface containing a nil pointer is not itself nil.

**Where to Use:**
- Understanding nil-check behavior with interfaces
- Proper handling of interface values
- Designing functions that return interfaces
- Debugging nil-related errors
- Working with optional interface implementations
- Understanding error handling
- Testing for interface nil values

**Code Snippet:**
```go
import (
    "fmt"
    "io"
)

func main() {
    // A nil interface value - both type and value are nil
    var i interface{}
    fmt.Println("i == nil:", i == nil) // true
    
    // A nil pointer of type *int - the pointer is nil
    var p *int
    fmt.Println("p == nil:", p == nil) // true
    
    // Assign nil pointer to interface - interface is NOT nil
    i = p
    fmt.Println("After i = p, i == nil:", i == nil) // false!
    
    // Interface has a type (*int) but its value is nil
    fmt.Printf("Type: %T, Value: %v\n", i, i) // Type: *int, Value: <nil>
    
    // Working with typed nil
    type MyError struct {
        Message string
    }
    
    func (e *MyError) Error() string {
        if e == nil {
            return "<nil>"
        }
        return e.Message
    }
    
    // Create a nil *MyError
    var myErr *MyError
    fmt.Println("myErr == nil:", myErr == nil) // true
    
    // Assign to error interface - now it's not nil
    var err error = myErr
    fmt.Println("err == nil:", err == nil) // false!
    
    // But the method can still be called
    fmt.Println("err.Error():", err.Error()) // <nil>
    
    // Common pattern with io.Reader
    var r io.Reader
    fmt.Println("r == nil:", r == nil) // true
    
    // Create a nil *bytes.Buffer
    var buf *bytes.Buffer = nil
    
    // Assign nil buffer to io.Reader
    r = buf
    fmt.Println("After r = buf, r == nil:", r == nil) // false!
    
    // This is often a source of bugs in error handling
    func returnsError() error {
        var p *MyError = nil
        // ...
        return p // Returns a non-nil error interface containing a nil pointer
    }
    
    err = returnsError()
    if err != nil {
        // This will execute even though the error's value is nil!
        fmt.Println("Error is not nil, but its value is:", err) // Error is not nil, but its value is: <nil>
    }
    
    // Proper pattern for returning nil error
    func properErrorReturn() error {
        var p *MyError = nil
        // ...
        if someCondition {
            return p // Returns a non-nil error interface
        }
        return nil // Returns a nil interface
    }
}
```

**Real-World Example:**
Building a plugin system with nil interface handling:

```go
package plugin

import (
    "errors"
    "fmt"
    "sync"
)

// Plugin defines the interface that all plugins must implement
type Plugin interface {
    Name() string
    Initialize() error
    Execute(args map[string]interface{}) (interface{}, error)
    Shutdown() error
}

// Registry manages the available plugins
type Registry struct {
    plugins map[string]Plugin
    mu      sync.RWMutex
}

// NewRegistry creates a plugin registry
func NewRegistry() *Registry {
    return &Registry{
        plugins: make(map[string]Plugin),
    }
}

// Register adds a plugin to the registry
func (r *Registry) Register(p Plugin) error {
    // Check if plugin is nil
    if p == nil {
        return errors.New("cannot register a nil plugin")
    }
    
    name := p.Name()
    if name == "" {
        return errors.New("plugin must have a non-empty name")
    }
    
    r.mu.Lock()
    defer r.mu.Unlock()
    
    if _, exists := r.plugins[name]; exists {
        return fmt.Errorf("plugin '%s' is already registered", name)
    }
    
    r.plugins[name] = p
    return nil
}

// Get returns a plugin by name
func (r *Registry) Get(name string) Plugin {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    // Return nil (not a typed nil) if not found
    return r.plugins[name]
}

// Execute runs a plugin with the given name and arguments
func (r *Registry) Execute(name string, args map[string]interface{}) (interface{}, error) {
    plugin := r.Get(name)
    
    // Proper nil check
    if plugin == nil {
        return nil, fmt.Errorf("plugin '%s' not found", name)
    }
    
    return plugin.Execute(args)
}

// InitializeAll initializes all registered plugins
func (r *Registry) InitializeAll() []error {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    var errors []error
    for name, plugin := range r.plugins {
        // This check shouldn't be necessary (map values can't be nil)
        // But included for defensive programming
        if plugin == nil {
            errors = append(errors, fmt.Errorf("plugin '%s' is nil", name))
            continue
        }
        
        if err := plugin.Initialize(); err != nil {
            errors = append(errors, fmt.Errorf("failed to initialize plugin '%s': %w", name, err))
        }
    }
    
    if len(errors) == 0 {
        return nil
    }
    return errors
}

// ShutdownAll shuts down all plugins
func (r *Registry) ShutdownAll() []error {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    var errors []error
    for name, plugin := range r.plugins {
        // Shouldn't be necessary, but defensive programming
        if plugin == nil {
            continue
        }
        
        if err := plugin.Shutdown(); err != nil {
            errors = append(errors, fmt.Errorf("failed to shut down plugin '%s': %w", name, err))
        }
    }
    
    if len(errors) == 0 {
        return nil
    }
    return errors
}

// BasePlugin provides default implementations for Plugin methods
type BasePlugin struct {
    PluginName string
}

// Name returns the plugin name
func (b *BasePlugin) Name() string {
    return b.PluginName
}

// Initialize provides a default implementation
func (b *BasePlugin) Initialize() error {
    return nil
}

// Shutdown provides a default implementation
func (b *BasePlugin) Shutdown() error {
    return nil
}

// LoggingPlugin is an example plugin that logs messages
type LoggingPlugin struct {
    BasePlugin
    LogLevel string
}

// NewLoggingPlugin creates a new logging plugin
func NewLoggingPlugin(level string) *LoggingPlugin {
    return &LoggingPlugin{
        BasePlugin: BasePlugin{PluginName: "logger"},
        LogLevel:   level,
    }
}

// Execute implements the Plugin interface
func (l *LoggingPlugin) Execute(args map[string]interface{}) (interface{}, error) {
    msg, ok := args["message"].(string)
    if !ok {
        return nil, errors.New("message must be a string")
    }
    
    fmt.Printf("[%s] %s\n", l.LogLevel, msg)
    return nil, nil
}

// MathPlugin is an example plugin that performs calculations
type MathPlugin struct {
    BasePlugin
}

// NewMathPlugin creates a new math plugin
func NewMathPlugin() *MathPlugin {
    return &MathPlugin{
        BasePlugin: BasePlugin{PluginName: "math"},
    }
}

// Execute implements the Plugin interface
func (m *MathPlugin) Execute(args map[string]interface{}) (interface{}, error) {
    op, ok := args["operation"].(string)
    if !ok {
        return nil, errors.New("operation must be a string")
    }
    
    x, xok := args["x"].(float64)
    y, yok := args["y"].(float64)
    
    if !xok || !yok {
        return nil, errors.New("x and y must be numbers")
    }
    
    switch op {
    case "add":
        return x + y, nil
    case "subtract":
        return x - y, nil
    case "multiply":
        return x * y, nil
    case "divide":
        if y == 0 {
            return nil, errors.New("division by zero")
        }
        return x / y, nil
    default:
        return nil, fmt.Errorf("unknown operation: %s", op)
    }
}

// OptionalFeature represents a plugin that may not be available
type OptionalFeature struct {
    Name     string
    Available bool
    plugin   Plugin
}

// NewOptionalFeature creates a new optional feature
func NewOptionalFeature(name string, p Plugin) *OptionalFeature {
    return &OptionalFeature{
        Name:     name,
        Available: p != nil,
        plugin:   p,
    }
}

// Execute runs the plugin if available
func (o *OptionalFeature) Execute(args map[string]interface{}) (interface{}, error) {
    if !o.Available {
        return nil, fmt.Errorf("feature '%s' is not available", o.Name)
    }
    
    // This nil check is important - the plugin might be unavailable
    // but o.plugin could be a typed nil (not nil interface, but a nil pointer)
    if o.plugin == nil {
        return nil, fmt.Errorf("feature '%s' has no implementation", o.Name)
    }
    
    return o.plugin.Execute(args)
}

// Usage example
func Example() {
    // Create plugin registry
    registry := NewRegistry()
    
    // Register plugins
    logger := NewLoggingPlugin("INFO")
    math := NewMathPlugin()
    
    registry.Register(logger)
    registry.Register(math)
    
    // Initialize all plugins
    if errs := registry.InitializeAll(); errs != nil {
        for _, err := range errs {
            fmt.Printf("Initialization error: %v\n", err)
        }
    }
    
    // Execute logging plugin
    _, err := registry.Execute("logger", map[string]interface{}{
        "message": "Hello, World!",
    })
    if err != nil {
        fmt.Printf("Error executing logger: %v\n", err)
    }
    
    // Execute math plugin
    result, err := registry.Execute("math", map[string]interface{}{
        "operation": "add",
        "x":         10.5,
        "y":         5.2,
    })
    if err != nil {
        fmt.Printf("Error executing math plugin: %v\n", err)
    } else {
        fmt.Printf("Result: %.2f\n", result.(float64))
    }
    
    // Try to execute a non-existent plugin
    _, err = registry.Execute("unknown", nil)
    if err != nil {
        fmt.Printf("Expected error: %v\n", err)
    }
    
    // Create an optional feature (unavailable)
    analytics := NewOptionalFeature("analytics", nil)
    fmt.Printf("Analytics available: %t\n", analytics.Available)
    
    _, err = analytics.Execute(nil)
    if err != nil {
        fmt.Printf("Expected error: %v\n", err)
    }
    
    // Shutdown all plugins
    registry.ShutdownAll()
}

// Demonstrating nil interface behavior
func NilInterfaceDemo() {
    // Define a function that might return nil
    getPlugin := func(name string) Plugin {
        if name == "real" {
            return &LoggingPlugin{
                BasePlugin: BasePlugin{PluginName: "logger"},
                LogLevel:   "DEBUG",
            }
        }
        
        // Returning nil interface
        if name == "missing" {
            return nil
        }
        
        // Returning interface containing nil pointer (problematic)
        var nilPlugin *LoggingPlugin
        return nilPlugin // This is a non-nil interface containing a nil pointer!
    }
    
    // Case 1: Truly nil interface
    p1 := getPlugin("missing")
    fmt.Println("p1 == nil:", p1 == nil) // true
    
    // Safe way to use
    if p1 != nil {
        p1.Execute(nil) // Won't execute
    } else {
        fmt.Println("p1 is nil, not executing")
    }
    
    // Case 2: Interface with nil pointer
    p2 := getPlugin("nil")
    fmt.Println("p2 == nil:", p2 == nil) // false!
    
    // This can cause bugs
    if p2 != nil {
        // This will execute even though the pointer is nil!
        fmt.Println("p2 is not nil, trying to execute...")
        // p2.Execute(nil) // Would panic with nil pointer dereference
        
        // Safe way to check:
        switch v := p2.(type) {
        case *LoggingPlugin:
            if v == nil {
                fmt.Println("p2 contains a nil *LoggingPlugin")
            }
        }
    }
    
    // Case 3: Normal non-nil interface
    p3 := getPlugin("real")
    fmt.Println("p3 == nil:", p3 == nil) // false
    
    // Safe to use
    if p3 != nil {
        p3.Execute(map[string]interface{}{"message": "Hello"})
    }
}
```

**Common Pitfalls:**
- Mistakenly assuming an interface containing a nil pointer is itself nil
- Returning a typed nil instead of a nil interface in functions
- Not checking for both interface nil and contained nil pointers
- Confusing nil interfaces with zero value structs
- Incorrectly comparing interface values for equality
- Not understanding the difference between a nil interface value and an interface value containing nil
- Using nil interfaces without type assertion checks
- Not documenting functions that return interface types

**Confusion Questions:**

1. **Q: Why doesn't `if err != nil { ... }` work as expected with typed nil values?**
   
   A: This is one of the most common sources of bugs with Go interfaces. The issue occurs because an interface value is only nil if both its concrete type and value are nil.
   
   Here's what happens in the troublesome case:
   
   ```go
   func returnsError() error {
       var p *CustomError = nil  // A nil pointer of type *CustomError
       return p                 // Implicitly converted to error interface
   }
   
   func main() {
       err := returnsError()
       
       fmt.Println(err == nil)  // Prints: false
       
       // This condition will be true even though the error's value is nil!
       if err != nil {
           fmt.Println("Error is not nil, but its value is:", err)
           // Often prints: Error is not nil, but its value is: <nil>
       }
   }
   ```
   
   What's happening:
   
   1. The `returnsError` function creates a nil pointer of type `*CustomError`
   2. When returning this as an `error` interface, Go creates an interface value with:
      - Concrete type: `*CustomError`
      - Value: `nil`
   3. The resulting interface is not nil because it has a concrete type
   4. The `if err != nil` check returns `true` because the interface itself is not nil
   
   **The correct way** to return a nil error:
   
   ```go
   func returnsError() error {
       var p *CustomError = nil
       
       if someCondition {
           return p  // Returns a non-nil interface containing nil pointer
       }
       
       return nil  // Returns a nil interface
   }
   ```
   
   **How to safely check for both**:
   
   ```go
   // Option 1: Check for specific types
   if err != nil {
       if customErr, ok := err.(*CustomError); ok && customErr == nil {
           // Handle the case of a nil *CustomError
           fmt.Println("Error is a nil *CustomError")
       } else {
           // Handle normal non-nil error
           fmt.Println("Error:", err)
       }
   }
   
   // Option 2: Use reflection (more generic but more overhead)
   func isNilValue(i interface{}) bool {
       if i == nil {
           return true
       }
       
       val := reflect.ValueOf(i)
       return val.Kind() == reflect.Ptr && val.IsNil()
   }
   ```
   
   This behavior is a consequence of Go's interface design and is important to understand for correct error handling.

2. **Q: How exactly are interface values represented in memory?**
   
   A: Understanding the internal representation of interface values helps explain their behavior, particularly with nil values:
   
   **Interface value structure**:
   
   An interface value consists of two components:
   
   1. **Type pointer** (or type descriptor): Points to runtime information about the concrete type
   2. **Data pointer**: Points to the actual data of the concrete type
   
   Conceptually, you can think of an interface value like this:
   
   ```go
   type interfaceValue struct {
       typePtr *typeDescriptor  // Information about the concrete type
       dataPtr unsafe.Pointer   // Pointer to the data
   }
   ```
   
   **Different scenarios**:
   
   1. **Nil interface value**:
      ```go
      var i interface{}
      // i's representation:
      // typePtr = nil
      // dataPtr = nil
      ```
      Both pointers are nil, so the interface is nil.
   
   2. **Interface with concrete value**:
      ```go
      var i interface{} = 42
      // i's representation:
      // typePtr = pointer to int's type information
      // dataPtr = pointer to the int value 42
      ```
   
   3. **Interface with nil pointer**:
      ```go
      var p *int = nil
      var i interface{} = p
      // i's representation:
      // typePtr = pointer to *int's type information
      // dataPtr = nil
      ```
      The type pointer is not nil, but the data pointer is nil. The interface itself is not nil.
   
   **Visual representation**:
   
   ```
   Nil interface:
   +----------+----------+
   | typePtr  | dataPtr  |
   |   nil    |   nil    |
   +----------+----------+
   
   Interface with value:
   +----------+----------+      +------+
   | typePtr  | dataPtr  |----->|  42  |
   | non-nil  | non-nil  |      +------+
   +----------+----------+
   
   Interface with nil pointer:
   +----------+----------+
   | typePtr  | dataPtr  |
   | non-nil  |   nil    |
   +----------+----------+
   ```
   
   **This explains the behavior**:
   
   1. `i == nil` checks if both the type and data pointers are nil
   2. Method calls on interfaces use the type pointer to find the method
   3. A nil interface cannot have methods called on it (both pointers are nil)
   4. An interface with a nil data pointer can have methods called on it (the type pointer is not nil)
   
   This internal representation is why:
   
   ```go
   var p *int = nil  // A nil pointer
   var i interface{} = p  // i has type but nil data
   
   fmt.Println(p == nil)  // true - p is nil
   fmt.Println(i == nil)  // false - i's type pointer is not nil
   ```
   
   Understanding this representation helps explain the sometimes confusing behavior of nil interfaces and interfaces containing nil pointers.

3. **Q: How should I properly handle nil values with interfaces?**
   
   A: Working correctly with nil interfaces requires understanding both interface nil checks and contained value nil checks. Here are patterns to handle them properly:
   
   **1. Return true nil interfaces, not typed nil values**:
   
   ```go
   // GOOD: Returns a nil interface for the error case
   func GoodFunction() error {
       if succeeded {
           return nil
       }
       
       // Return a real error, not a typed nil
       return errors.New("something went wrong")
   }
   
   // BAD: Returns a typed nil
   func BadFunction() error {
       var err *MyError = nil
       return err  // Returns a non-nil interface holding a nil *MyError
   }
   ```
   
   **2. Use nil checks with type assertions**:
   
   ```go
   func processError(err error) {
       if err == nil {
           return // No error
       }
       
       // Type-specific error handling with nil check
       if customErr, ok := err.(*CustomError); ok {
           if customErr == nil {
               // Handle the case of nil *CustomError
               fmt.Println("Got a nil custom error")
               return
           }
           // Handle non-nil custom error
           fmt.Println("Custom error:", customErr.Message)
           return
       }
       
       // Handle other error types
       fmt.Println("Generic error:", err)
   }
   ```
   
   **3. Implement safe nil handling in interface methods**:
   
   ```go
   type Resource struct {
       Name string
       Data []byte
   }
   
   func (r *Resource) Process() error {
       // Safe nil check inside the method
       if r == nil {
           return errors.New("cannot process nil resource")
       }
       
       // Now safe to use r
       fmt.Printf("Processing resource: %s\n", r.Name)
       return nil
   }
   ```
   
   **4. Careful comparison of interface values**:
   
   ```go
   func compareInterfaces(a, b interface{}) bool {
       // Direct comparison only works if types match exactly
       
       // If both are nil, they're equal
       if a == nil && b == nil {
           return true
       }
       
       // If only one is nil, they're not equal
       if a == nil || b == nil {
           return false
       }
       
       // For dynamic value comparison, use reflection
       return reflect.DeepEqual(a, b)
   }
   ```
   
   **5. Create helper functions for safe handling**:
   
   ```go
   // IsNil safely checks if an interface or its value is nil
   func IsNil(v interface{}) bool {
       if v == nil {
           return true
       }
       
       // Use reflection to check for nil pointer, slice, map, etc.
       val := reflect.ValueOf(v)
       return val.Kind() == reflect.Ptr && val.IsNil()
   }
   
   // SafeCall calls a function only if the receiver isn't nil
   func SafeCall(obj interface{}, fn func() error) error {
       if IsNil(obj) {
           return errors.New("cannot call method on nil value")
       }
       return fn()
   }
   ```
   
   **6. Design interfaces with nil safety in mind**:
   
   ```go
   // NullObject pattern - provide a non-nil implementation
   type Logger interface {
       Log(msg string)
   }
   
   type NullLogger struct{}
   
   func (NullLogger) Log(msg string) {
       // Do nothing
   }
   
   // Instead of returning nil, return a null implementation
   func GetLogger(name string) Logger {
       logger := findLogger(name)
       if logger == nil {
           return NullLogger{}
       }
       return logger
   }
   ```
   
   **7. Document nil behavior clearly**:
   
   ```go
   // ProcessData handles data processing.
   // Returns nil if processing succeeds.
   // May return *ProcessError or *ValidationError.
   // If input is nil, returns ErrNilInput.
   func ProcessData(data []byte) error {
       if data == nil {
           return ErrNilInput
       }
       // Processing...
   }
   ```
   
   By following these patterns consistently, you can avoid the common bugs and surprises that come from nil interface values.

### 7. Type Assertions and Type Switches

**Concise Explanation:**
Type assertions and type switches provide mechanisms to access the underlying concrete value of an interface. A type assertion checks if an interface value holds a specific type and extracts the value if it does. A type switch allows for conditional execution based on the concrete type of an interface value, handling multiple possible types in a concise way.

**Where to Use:**
- Extracting concrete values from interfaces
- Handling different types in a polymorphic way
- Working with data of unknown or multiple types
- Conditional processing based on types
- Implementing type-specific behavior
- Pattern matching on types
- Converting between related types

**Code Snippet:**
```go
import (
    "fmt"
    "io"
    "os"
)

func main() {
    // ---------- Type Assertions ----------
    
    // Basic type assertion
    var i interface{} = "hello"
    
    // Check if i contains a string
    s, ok := i.(string)
    if ok {
        fmt.Println("i is a string:", s)
    } else {
        fmt.Println("i is not a string")
    }
    
    // Type assertion without check (DANGEROUS)
    // If the assertion fails, this will panic
    // s := i.(string)
    
    // Type assertion with different type (fails gracefully)
    n, ok := i.(int)
    if ok {
        fmt.Println("i is an int:", n)
    } else {
        fmt.Println("i is not an int")
    }
    
    // Type assertions with interfaces
    var r io.Reader = os.Stdin
    
    // Check if r is also a Writer
    file, ok := r.(*os.File)
    if ok {
        fmt.Println("r is an *os.File:", file.Name())
    }
    
    // Check if r implements io.Writer
    writer, ok := r.(io.Writer)
    if ok {
        fmt.Println("r is also a Writer, writing 'hello'")
        writer.Write([]byte("hello"))
    }
    
    // ---------- Type Switches ----------
    
    // Function using type switch
    processValue := func(v interface{}) {
        switch x := v.(type) {
        case nil:
            fmt.Println("v is nil")
        case int:
            fmt.Printf("v is an int: %d\n", x)
        case float64:
            fmt.Printf("v is a float64: %.2f\n", x)
        case string:
            fmt.Printf("v is a string: %q\n", x)
        case bool:
            fmt.Printf("v is a bool: %t\n", x)
        case []int:
            fmt.Printf("v is a slice of ints with %d elements\n", len(x))
        case io.Reader:
            fmt.Println("v is an io.Reader")
        default:
            fmt.Printf("v is of an unknown type: %T\n", v)
        }
    }
    
    // Try with different values
    processValue(42)
    processValue(3.14)
    processValue("hello")
    processValue(true)
    processValue([]int{1, 2, 3})
    processValue(os.Stdin)
    processValue(struct{ name string }{"Alice"})
    
    // Type switch with multiple types per case
    categorize := func(v interface{}) string {
        switch v.(type) {
        case int, int8, int16, int32, int64:
            return "integer"
        case float32, float64:
            return "floating-point"
        case string:
            return "string"
        case bool:
            return "boolean"
        case nil:
            return "nil"
        default:
            return "other"
        }
    }
    
    fmt.Println(categorize(42))         // "integer"
    fmt.Println(categorize(3.14))       // "floating-point"
    fmt.Println(categorize("hello"))    // "string"
    
    // Type assertion inside a type switch
    processReader := func(r io.Reader) {
        switch reader := r.(type) {
        case *os.File:
            fmt.Println("Processing file:", reader.Name())
        case *bytes.Buffer:
            fmt.Println("Processing bytes buffer, len:", reader.Len())
        case nil:
            fmt.Println("Reader is nil")
        default:
            fmt.Printf("Unknown reader type: %T\n", reader)
        }
    }
    
    processReader(os.Stdin)
    processReader(bytes.NewBufferString("hello"))
}
```

**Real-World Example:**
Building a flexible data processing pipeline with type assertions and switches:

```go
package processor

import (
    "encoding/json"
    "encoding/xml"
    "errors"
    "fmt"
    "io"
    "strings"
    "time"
)

// Result represents processed data
type Result struct {
    Data        interface{}
    Type        string
    ProcessedAt time.Time
    Metadata    map[string]interface{}
}

// Processor defines the interface for data processors
type Processor interface {
    Process(data interface{}) (*Result, error)
    CanProcess(data interface{}) bool
}

// Pipeline orchestrates multiple processors
type Pipeline struct {
    processors []Processor
}

// NewPipeline creates a new processing pipeline
func NewPipeline(processors ...Processor) *Pipeline {
    return &Pipeline{
        processors: processors,
    }
}

// AddProcessor adds a processor to the pipeline
func (p *Pipeline) AddProcessor(processor Processor) {
    p.processors = append(p.processors, processor)
}

// Process processes data through the pipeline
func (p *Pipeline) Process(data interface{}) (*Result, error) {
    for _, processor := range p.processors {
        if processor.CanProcess(data) {
            return processor.Process(data)
        }
    }
    
    return nil, fmt.Errorf("no processor available for type %T", data)
}

// StringProcessor processes string data
type StringProcessor struct{}

func (p StringProcessor) CanProcess(data interface{}) bool {
    _, ok := data.(string)
    return ok
}

func (p StringProcessor) Process(data interface{}) (*Result, error) {
    str, ok := data.(string)
    if !ok {
        return nil, errors.New("stringProcessor: not a string")
    }
    
    // Process the string
    processed := strings.TrimSpace(str)
    wordCount := len(strings.Fields(processed))
    
    return &Result{
        Data:        processed,
        Type:        "string",
        ProcessedAt: time.Now(),
        Metadata: map[string]interface{}{
            "length":    len(processed),
            "wordCount": wordCount,
        },
    }, nil
}

// NumberProcessor processes numeric data
type NumberProcessor struct{}

func (p NumberProcessor) CanProcess(data interface{}) bool {
    switch data.(type) {
    case int, int32, int64, float32, float64:
        return true
    }
    return false
}

func (p NumberProcessor) Process(data interface{}) (*Result, error) {
    var value float64
    
    // Convert various numeric types to float64
    switch v := data.(type) {
    case int:
        value = float64(v)
    case int32:
        value = float64(v)
    case int64:
        value = float64(v)
    case float32:
        value = float64(v)
    case float64:
        value = v
    default:
        return nil, fmt.Errorf("numberProcessor: unsupported type %T", data)
    }
    
    // Process the number
    return &Result{
        Data:        value,
        Type:        "number",
        ProcessedAt: time.Now(),
        Metadata: map[string]interface{}{
            "isInteger": value == float64(int(value)),
            "squared":   value * value,
            "doubled":   value * 2,
        },
    }, nil
}

// JSONProcessor processes JSON data
type JSONProcessor struct{}

func (p JSONProcessor) CanProcess(data interface{}) bool {
    switch data := data.(type) {
    case string:
        // Check if it looks like JSON
        return len(data) > 0 && (data[0] == '{' || data[0] == '[')
    case []byte:
        return len(data) > 0 && (data[0] == '{' || data[0] == '[')
    case io.Reader:
        return true // We'll try to parse it as JSON
    }
    return false
}

func (p JSONProcessor) Process(data interface{}) (*Result, error) {
    var jsonData interface{}
    var err error
    
    switch v := data.(type) {
    case string:
        err = json.Unmarshal([]byte(v), &jsonData)
    case []byte:
        err = json.Unmarshal(v, &jsonData)
    case io.Reader:
        dec := json.NewDecoder(v)
        err = dec.Decode(&jsonData)
    default:
        return nil, fmt.Errorf("jsonProcessor: unsupported type %T", data)
    }
    
    if err != nil {
        return nil, fmt.Errorf("jsonProcessor: %w", err)
    }
    
    // Extract metadata based on JSON structure
    metadata := make(map[string]interface{})
    
    switch v := jsonData.(type) {
    case map[string]interface{}:
        metadata["type"] = "object"
        metadata["keys"] = len(v)
        
        // Extract top-level keys
        keys := make([]string, 0, len(v))
        for k := range v {
            keys = append(keys, k)
        }
        metadata["keyNames"] = keys
        
    case []interface{}:
        metadata["type"] = "array"
        metadata["length"] = len(v)
        
        // Try to determine array item type
        if len(v) > 0 {
            metadata["firstItemType"] = fmt.Sprintf("%T", v[0])
        }
    }
    
    return &Result{
        Data:        jsonData,
        Type:        "json",
        ProcessedAt: time.Now(),
        Metadata:    metadata,
    }, nil
}

// XMLProcessor processes XML data
type XMLProcessor struct{}

func (p XMLProcessor) CanProcess(data interface{}) bool {
    switch data := data.(type) {
    case string:
        return strings.HasPrefix(strings.TrimSpace(data), "<")
    case []byte:
        return len(data) > 0 && data[0] == '<'
    case io.Reader:
        return true // We'll try to parse it as XML
    }
    return false
}

func (p XMLProcessor) Process(data interface{}) (*Result, error) {
    var xmlMap map[string]interface{}
    var err error
    
    switch v := data.(type) {
    case string:
        err = xml.Unmarshal([]byte(v), &xmlMap)
    case []byte:
        err = xml.Unmarshal(v, &xmlMap)
    case io.Reader:
        dec := xml.NewDecoder(v)
        err = dec.Decode(&xmlMap)
    default:
        return nil, fmt.Errorf("xmlProcessor: unsupported type %T", data)
    }
    
    if err != nil {
        return nil, fmt.Errorf("xmlProcessor: %w", err)
    }
    
    return &Result{
        Data:        xmlMap,
        Type:        "xml",
        ProcessedAt: time.Now(),
        Metadata: map[string]interface{}{
            "type": "xml document",
        },
    }, nil
}

// SliceProcessor processes slice data
type SliceProcessor struct{}

func (p SliceProcessor) CanProcess(data interface{}) bool {
    v := reflect.ValueOf(data)
    return v.Kind() == reflect.Slice
}

func (p SliceProcessor) Process(data interface{}) (*Result, error) {
    v := reflect.ValueOf(data)
    if v.Kind() != reflect.Slice {
        return nil, errors.New("sliceProcessor: not a slice")
    }
    
    length := v.Len()
    metadata := map[string]interface{}{
        "length": length,
    }
    
    // Attempt to determine element type
    if length > 0 {
        firstElem := v.Index(0).Interface()
        metadata["elementType"] = fmt.Sprintf("%T", firstElem)
        
        // Check if elements are all the same type
        sameType := true
        for i := 1; i < length && sameType; i++ {
            elem := v.Index(i).Interface()
            if fmt.Sprintf("%T", elem) != metadata["elementType"] {
                sameType = false
            }
        }
        metadata["homogeneous"] = sameType
        
        // For numeric slices, compute some statistics
        if sameType {
            switch firstElem.(type) {
            case int, int32, int64, float32, float64:
                sum := 0.0
                for i := 0; i < length; i++ {
                    val := reflect.ValueOf(v.Index(i).Interface()).Convert(reflect.TypeOf(float64(0))).Float()
                    sum += val
                }
                metadata["sum"] = sum
                metadata["average"] = sum / float64(length)
            }
        }
    }
    
    return &Result{
        Data:        data,
        Type:        "slice",
        ProcessedAt: time.Now(),
        Metadata:    metadata,
    }, nil
}

// FallbackProcessor handles any data type
type FallbackProcessor struct{}

func (p FallbackProcessor) CanProcess(data interface{}) bool {
    // Always returns true as a fallback
    return true
}

func (p FallbackProcessor) Process(data interface{}) (*Result, error) {
    result := &Result{
        Data:        data,
        Type:        fmt.Sprintf("%T", data),
        ProcessedAt: time.Now(),
        Metadata:    make(map[string]interface{}),
    }
    
    // Add type information
    v := reflect.ValueOf(data)
    result.Metadata["kind"] = v.Kind().String()
    
    // Add more metadata based on reflection
    switch v.Kind() {
    case reflect.Struct:
        result.Metadata["numFields"] = v.NumField()
    case reflect.Map:
        result.Metadata["length"] = v.Len()
    case reflect.Ptr:
        if !v.IsNil() {
            elem := v.Elem()
            result.Metadata["pointsTo"] = elem.Type().String()
        } else {
            result.Metadata["isNil"] = true
        }
    }
    
    return result, nil
}

// ProcessorRegistry manages available processors
type ProcessorRegistry struct {
    processors map[string]Processor
}

// NewProcessorRegistry creates a new processor registry
func NewProcessorRegistry() *ProcessorRegistry {
    return &ProcessorRegistry{
        processors: make(map[string]Processor),
    }
}

// Register adds a processor to the registry
func (r *ProcessorRegistry) Register(name string, processor Processor) {
    r.processors[name] = processor
}

// Get returns a processor by name
func (r *ProcessorRegistry) Get(name string) (Processor, bool) {
    processor, exists := r.processors[name]
    return processor, exists
}

// CreateDefaultPipeline builds a standard processing pipeline
func CreateDefaultPipeline() *Pipeline {
    return NewPipeline(
        &StringProcessor{},
        &NumberProcessor{},
        &JSONProcessor{},
        &XMLProcessor{},
        &SliceProcessor{},
        &FallbackProcessor{},
    )
}

// Usage example
func Example() {
    pipeline := CreateDefaultPipeline()
    
    // Process different types of data
    processAndPrint := func(data interface{}) {
        result, err := pipeline.Process(data)
        if err != nil {
            fmt.Printf("Error: %v\n", err)
            return
        }
        
        fmt.Printf("Processed %s:\n", result.Type)
        fmt.Printf("  Data: %v\n", result.Data)
        fmt.Printf("  Metadata: %v\n", result.Metadata)
        fmt.Println()
    }
    
    // Process a string
    processAndPrint("  Hello, World!  ")
    
    // Process a number
    processAndPrint(42)
    
    // Process JSON
    processAndPrint(`{"name": "Alice", "age": 30, "active": true}`)
    
    // Process XML
    processAndPrint("<user><name>Bob</name><age>25</age></user>")
    
    // Process a slice
    processAndPrint([]int{1, 2, 3, 4, 5})
    
    // Process a custom struct
    type Person struct {
        Name string
        Age  int
    }
    processAndPrint(Person{Name: "Charlie", Age: 35})
}
```

**Common Pitfalls:**
- Using type assertions without the "comma ok" check, leading to panics
- Forgetting that a type switch with no cases doesn't match anything
- Not handling the default case in a type switch
- Trying to access variables from one case in another case
- Using overly complex nested type switches
- Not considering that interfaces can be nil or contain nil pointers
- Confusing type assertions with type conversions
- Assuming the order of type checks doesn't matter (it does when interfaces are involved)

**Confusion Questions:**

1. **Q: What's the difference between type assertion and type conversion?**
   
   A: Type assertion and type conversion are related but fundamentally different mechanisms in Go:
   
   **Type Assertion** works only with interface types and attempts to extract the underlying concrete value:
   
   ```go
   var i interface{} = "hello"
   
   // Type assertion - extracting the string from interface{}
   s, ok := i.(string)  // s = "hello", ok = true
   
   // Type assertion without a check (dangerous, could panic)
   s2 := i.(string)  // s2 = "hello"
   
   // Failed type assertion with check
   n, ok := i.(int)  // n = 0, ok = false (no panic)
   
   // Failed type assertion without check
   // n2 := i.(int)  // PANIC: interface conversion: interface {} is string, not int
   ```
   
   **Type Conversion** changes a value from one type to another compatible type:
   
   ```go
   // Type conversion between numeric types
   var x int = 10
   var y float64 = float64(x)  // Convert int to float64
   
   // Type conversion between string and byte slice
   s := "hello"
   b := []byte(s)  // Convert string to []byte
   s2 := string(b) // Convert []byte back to string
   
   // Type conversion with named types
   type MyInt int
   var m MyInt = 5
   var n int = int(m)  // Convert MyInt to int
   ```
   
   **Key differences**:
   
   1. **Purpose**:
      - Type assertion: Extracts concrete values from interfaces
      - Type conversion: Changes a value from one type to another
   
   2. **Applicable types**:
      - Type assertion: Works only on interface types
      - Type conversion: Works between compatible non-interface types
   
   3. **Syntax**:
      - Type assertion: `value.(Type)` or `value.(type)` in a switch
      - Type conversion: `DestType(value)`
   
   4. **Runtime behavior**:
      - Type assertion: Can fail at runtime (panic or return false for ok)
      - Type conversion: Compile-time checked for compatibility, though some conversions can lose information
   
   5. **Error handling**:
      - Type assertion: Use "comma ok" idiom for safe check
      - Type conversion: Compiler prevents invalid conversions
   
   6. **Use cases**:
      - Type assertion: Working with interfaces and polymorphism
      - Type conversion: Working with numeric types, strings, and user-defined types
   
   **Examples in context**:
   
   ```go
   func process(val interface{}) {
       // Type assertion - check if val is a string
       if str, ok := val.(string); ok {
           fmt.Println("String:", str)
       }
       
       // Type conversion - if val is an int, convert it to string
       if num, ok := val.(int); ok {
           numStr := strconv.Itoa(num)  // Type conversion from int to string
           // OR
           numStr = fmt.Sprintf("%d", num)
           fmt.Println("Number as string:", numStr)
       }
       
       // Type assertion with interface types
       if reader, ok := val.(io.Reader); ok {
           data := make([]byte, 100)
           reader.Read(data)
       }
   }
   ```
   
   Understanding the distinction helps avoid common errors and use the appropriate mechanism for each situation.

2. **Q: How do type switches compare to if-else chains with type assertions?**
   
   A: Type switches and if-else chains with type assertions can accomplish similar goals, but they have important differences in readability, efficiency, and behavior:
   
   **Type Switch**:
   
   ```go
   func process(v interface{}) string {
       switch x := v.(type) {
       case string:
           return "String: " + x
       case int:
           return fmt.Sprintf("Int: %d", x)
       case bool:
           return fmt.Sprintf("Bool: %t", x)
       case []byte:
           return fmt.Sprintf("Bytes: %v", x)
       default:
           return fmt.Sprintf("Unknown type: %T", v)
       }
   }
   ```
   
   **If-Else Chain**:
   
   ```go
   func process(v interface{}) string {
       if x, ok := v.(string); ok {
           return "String: " + x
       } else if x, ok := v.(int); ok {
           return fmt.Sprintf("Int: %d", x)
       } else if x, ok := v.(bool); ok {
           return fmt.Sprintf("Bool: %t", x)
       } else if x, ok := v.([]byte); ok {
           return fmt.Sprintf("Bytes: %v", x)
       } else {
           return fmt.Sprintf("Unknown type: %T", v)
       }
   }
   ```
   
   **Key differences**:
   
   1. **Readability**:
      - Type switches are more readable, especially with many cases
      - The purpose is clear at a glance - "I'm switching on the type"
      - The variable is declared once in the switch statement
   
   2. **Efficiency**:
      - Type switches can be more efficient as the Go runtime can potentially optimize the checks
      - Type assertions are checked separately in if-else chains, potentially doing redundant work
   
   3. **Variable Scoping**:
      - Type switch: Each case has its own scope for the variable
      - If-else: Variables from one clause can leak into others
   
   4. **Special Cases**:
      - Type switches can handle `nil` as a special case
      - Type switches can check multiple types in a single case
      - Type switches can check for interfaces that the value implements
   
   **Examples of type switch advantages**:
   
   1. **Multiple types in a single case**:
   ```go
   switch v.(type) {
   case int, int8, int16, int32, int64:
       return "some integer type"
   case float32, float64:
       return "some float type"
   }
   ```
   
   2. **Nil handling**:
   ```go
   switch x := v.(type) {
   case nil:
       return "nil value"
   // other cases...
   }
   ```
   
   3. **Interface checking**:
   ```go
   switch v.(type) {
   case io.Reader:
       return "implements Reader"
   case io.Writer:
       return "implements Writer"
   }
   ```
   
   **When to use each**:
   
   - **Use type switch when**:
     - Handling many different types
     - The logic focuses mainly on type differentiation
     - You need multiple types per case
     - You want clearer, more readable code
     - Special handling for nil is needed
   
   - **Use if-else with assertions when**:
     - Checking just one or two types
     - Type check is just one part of a more complex condition
     - You need to combine type checks with other conditions
     - You want to share variables between different clauses
     
   In most cases, type switches are preferred for pure type-based branching because they're more idiomatic, readable, and potentially more efficient.

3. **Q: Can type assertions and switches check if a value implements an interface?**
   
   A: Yes, type assertions and type switches can check if a value implements an interface, which is one of their most powerful features:
   
   **Type Assertions with Interfaces**:
   
   ```go
   // Check if a value implements io.Reader
   func processValue(v interface{}) {
       // Try to get v as an io.Reader
       reader, ok := v.(io.Reader)
       
       if ok {
           // v implements io.Reader, use the reader
           data := make([]byte, 100)
           n, err := reader.Read(data)
           fmt.Printf("Read %d bytes, error: %v\n", n, err)
       } else {
           fmt.Printf("%T does not implement io.Reader\n", v)
       }
   }
   ```
   
   **Type Switches with Interfaces**:
   
   ```go
   func categorize(v interface{}) string {
       switch v.(type) {
       case io.Reader:
           return "is a Reader"
       case io.Writer:
           return "is a Writer"
       case io.Closer:
           return "is a Closer"
       case io.ReadWriter:
           return "is both Reader and Writer"
       default:
           return "unknown interface type"
       }
   }
   ```
   
   **Important behavior for interfaces**:
   
   1. **Order matters** in type switches with interfaces:
   
   ```go
   func checkInterfaces(v interface{}) string {
       switch v.(type) {
       case io.Reader:
           return "Reader"          // This will match io.ReadWriter too!
       case io.ReadWriter:
           return "ReadWriter"      // Never reached for io.ReadWriter values
       }
       return "neither"
   }
   ```
   
   The correct order is from most specific to most general:
   
   ```go
   func checkInterfaces(v interface{}) string {
       switch v.(type) {
       case io.ReadWriter:
           return "ReadWriter"      // More specific interface first
       case io.Reader:
           return "Reader only"     // Less specific interface later
       }
       return "neither"
   }
   ```
   
   2. **Nil interface check**: Unlike regular nil checks, interface assertions can distinguish between nil interfaces and interfaces with nil values:
   
   ```go
   func checkNilInterface(v interface{}) {
       switch v.(type) {
       case nil:
           fmt.Println("nil interface (both type and value are nil)")
       default:
           if v == nil {
               fmt.Println("v is nil but has a type (rare case)")
           } else {
               fmt.Println("non-nil interface")
           }
       }
   }
   ```
   
   3. **Static vs. dynamic types**:
   
   ```go
   var r io.Reader = &bytes.Buffer{}
   
   // This checks the dynamic type
   switch r.(type) {
   case *bytes.Buffer:
       fmt.Println("It's a *bytes.Buffer")  // This will match
   case io.ReadWriter:
       fmt.Println("It's a ReadWriter")     // Would match if above case wasn't there
   }
   ```
   
   **Real-world example - HTTP handler multiplexer**:
   
   ```go
   func HandleRequest(handler interface{}, w http.ResponseWriter, r *http.Request) {
       switch h := handler.(type) {
       case func(http.ResponseWriter, *http.Request):
           // Simple handler function
           h(w, r)
           
       case http.Handler:
           // Standard http.Handler interface
           h.ServeHTTP(w, r)
           
       case func(string, http.ResponseWriter, *http.Request):
           // Handler with path parameter
           pathParam := r.URL.Path
           h(pathParam, w, r)
           
       case interface{ Process(http.ResponseWriter, *http.Request) error }:
           // Custom handler interface
           if err := h.Process(w, r); err != nil {
               http.Error(w, err.Error(), http.StatusInternalServerError)
           }
           
       default:
           http.Error(w, "Invalid handler", http.StatusInternalServerError)
       }
   }
   ```
   
   Type assertions and switches with interfaces enable flexible polymorphism without inheritance hierarchies, making them a cornerstone of Go's approach to interface-based design.

### 8. Interface Composition

**Concise Explanation:**
Interface composition is the practice of building larger interfaces by combining smaller ones. Instead of creating large, monolithic interfaces, Go encourages composing interfaces from smaller, focused interfaces. This promotes flexibility, code reuse, and the principle that types should only implement the methods they need.

**Where to Use:**
- Creating layered interface hierarchies
- Building modular APIs with specific capabilities
- Defining common behavior groups
- Extending existing interfaces
- Implementing the interface segregation principle
- Making code more testable with focused interfaces
- Enforcing minimal requirements for types

**Code Snippet:**
```go
// Basic interface composition
// Small, focused interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

// Composed interfaces
type ReadWriter interface {
    Reader
    Writer
}

type ReadCloser interface {
    Reader
    Closer
}

type WriteCloser interface {
    Writer
    Closer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// Using composed interfaces
func process(rw ReadWriter) error {
    // Can read and write
    data := make([]byte, 100)
    n, err := rw.Read(data)
    if err != nil {
        return err
    }
    
    // Process data...
    processed := bytes.ToUpper(data[:n])
    
    // Write back
    _, err = rw.Write(processed)
    return err
}

// Custom interface composition
type Sizer interface {
    Size() int64
}

type Stater interface {
    Stat() (os.FileInfo, error)
}

type ReadSizer interface {
    Reader
    Sizer
}

// Function using composed interface
func readIfSmall(rs ReadSizer, maxSize int64) ([]byte, error) {
    if rs.Size() > maxSize {
        return nil, errors.New("content too large")
    }
    
    // Content is an acceptable size, read it
    data := make([]byte, rs.Size())
    _, err := io.ReadFull(rs, data)
    return data, err
}
```

**Real-World Example:**
Building an extensible content management system with interface composition:

```go
package cms

import (
    "context"
    "errors"
    "io"
    "time"
)

// Core interfaces for content items

// Identifier provides a unique ID for content
type Identifier interface {
    ID() string
}

// Titled provides a title for content
type Titled interface {
    Title() string
}

// Describable provides a description for content
type Describable interface {
    Description() string
}

// Timestamped provides creation and modification times
type Timestamped interface {
    CreatedAt() time.Time
    UpdatedAt() time.Time
}

// Taggable provides tags/categories for content
type Taggable interface {
    Tags() []string
    AddTag(tag string)
    RemoveTag(tag string)
}

// Serializable can be serialized to and from bytes
type Serializable interface {
    MarshalContent() ([]byte, error)
    UnmarshalContent(data []byte) error
}

// Validatable can validate its own data integrity
type Validatable interface {
    Validate() error
}

// Permissioned controls access to content
type Permissioned interface {
    CanView(userID string) bool
    CanEdit(userID string) bool
    CanDelete(userID string) bool
}

// Versionable provides version history
type Versionable interface {
    Version() int
    PreviousVersions() []string // IDs of previous versions
}

// Composed interfaces for content types

// BasicContent represents the minimal content interface
type BasicContent interface {
    Identifier
    Titled
    Timestamped
}

// RichContent adds more metadata capabilities
type RichContent interface {
    BasicContent
    Describable
    Taggable
}

// Document represents a complete document interface
type Document interface {
    RichContent
    Validatable
    Serializable
    Permissioned
    Content() io.Reader
    SetContent(content io.Reader) error
    ContentType() string
    Size() int64
}

// VersionedDocument adds version history
type VersionedDocument interface {
    Document
    Versionable
}

// Storage interfaces for backend implementations

// ContentReader can fetch content
type ContentReader interface {
    Get(ctx context.Context, id string) (BasicContent, error)
    List(ctx context.Context, filter map[string]interface{}) ([]BasicContent, error)
}

// ContentWriter can save content
type ContentWriter interface {
    Save(ctx context.Context, content BasicContent) error
    Delete(ctx context.Context, id string) error
}

// ContentStorage combines reading and writing
type ContentStorage interface {
    ContentReader
    ContentWriter
}

// SearchProvider allows searching content
type SearchProvider interface {
    Search(ctx context.Context, query string) ([]BasicContent, error)
    Index(ctx context.Context, content BasicContent) error
}

// FullStorage combines all storage features
type FullStorage interface {
    ContentStorage
    SearchProvider
    
    // Additional methods specific to full storage
    Backup(ctx context.Context) (io.Reader, error)
    Stats(ctx context.Context) (StorageStats, error)
}

// StorageStats provides statistics about the storage
type StorageStats struct {
    DocumentCount int
    TotalSize     int64
    OldestDoc     time.Time
    NewestDoc     time.Time
}

// Implementation structs

// BaseContentItem implements the common content methods
type BaseContentItem struct {
    id          string
    title       string
    description string
    created     time.Time
    updated     time.Time
    tags        []string
}

// Implement BasicContent interface
func (b *BaseContentItem) ID() string {
    return b.id
}

func (b *BaseContentItem) Title() string {
    return b.title
}

func (b *BaseContentItem) SetTitle(title string) {
    b.title = title
    b.updated = time.Now()
}

// Implement Timestamped interface
func (b *BaseContentItem) CreatedAt() time.Time {
    return b.created
}

func (b *BaseContentItem) UpdatedAt() time.Time {
    return b.updated
}

// Implement Describable interface
func (b *BaseContentItem) Description() string {
    return b.description
}

func (b *BaseContentItem) SetDescription(desc string) {
    b.description = desc
    b.updated = time.Now()
}

// Implement Taggable interface
func (b *BaseContentItem) Tags() []string {
    return append([]string{}, b.tags...) // Return a copy
}

func (b *BaseContentItem) AddTag(tag string) {
    // Check if tag already exists
    for _, t := range b.tags {
        if t == tag {
            return
        }
    }
    b.tags = append(b.tags, tag)
    b.updated = time.Now()
}

func (b *BaseContentItem) RemoveTag(tag string) {
    for i, t := range b.tags {
        if t == tag {
            // Remove tag by replacing it with the last one and truncating
            b.tags[i] = b.tags[len(b.tags)-1]
            b.tags = b.tags[:len(b.tags)-1]
            b.updated = time.Now()
            break
        }
    }
}

// TextDocument implements the Document interface
type TextDocument struct {
    BaseContentItem
    content     []byte
    permissions map[string]int // UserID -> permission level
    version     int
    history     []string
}

// NewTextDocument creates a new text document
func NewTextDocument(id, title string) *TextDocument {
    now := time.Now()
    return &TextDocument{
        BaseContentItem: BaseContentItem{
            id:      id,
            title:   title,
            created: now,
            updated: now,
        },
        permissions: make(map[string]int),
        version:     1,
    }
}

// Implement Content methods
func (d *TextDocument) Content() io.Reader {
    return bytes.NewReader(d.content)
}

func (d *TextDocument) SetContent(content io.Reader) error {
    data, err := io.ReadAll(content)
    if err != nil {
        return err
    }
    
    // Save current version to history
    if len(d.content) > 0 {
        d.history = append(d.history, d.id+"_v"+strconv.Itoa(d.version))
        d.version++
    }
    
    d.content = data
    d.updated = time.Now()
    return nil
}

func (d *TextDocument) ContentType() string {
    return "text/plain"
}

func (d *TextDocument) Size() int64 {
    return int64(len(d.content))
}

// Implement Serializable
func (d *TextDocument) MarshalContent() ([]byte, error) {
    return json.Marshal(struct {
        ID          string    `json:"id"`
        Title       string    `json:"title"`
        Description string    `json:"description"`
        Content     string    `json:"content"`
        Created     time.Time `json:"created_at"`
        Updated     time.Time `json:"updated_at"`
        Tags        []string  `json:"tags"`
        Version     int       `json:"version"`
        History     []string  `json:"history"`
    }{
        ID:          d.id,
        Title:       d.title,
        Description: d.description,
        Content:     string(d.content),
        Created:     d.created,
        Updated:     d.updated,
        Tags:        d.tags,
        Version:     d.version,
        History:     d.history,
    })
}

func (d *TextDocument) UnmarshalContent(data []byte) error {
    var temp struct {
        ID          string    `json:"id"`
        Title       string    `json:"title"`
        Description string    `json:"description"`
        Content     string    `json:"content"`
        Created     time.Time `json:"created_at"`
        Updated     time.Time `json:"updated_at"`
        Tags        []string  `json:"tags"`
        Version     int       `json:"version"`
        History     []string  `json:"history"`
    }
    
    if err := json.Unmarshal(data, &temp); err != nil {
        return err
    }
    
    d.id = temp.ID
    d.title = temp.Title
    d.description = temp.Description
    d.content = []byte(temp.Content)
    d.created = temp.Created
    d.updated = temp.Updated
    d.tags = temp.Tags
    d.version = temp.Version
    d.history = temp.History
    
    return nil
}

// Implement Validatable
func (d *TextDocument) Validate() error {
    if d.id == "" {
        return errors.New("document must have an ID")
    }
    
    if d.title == "" {
        return errors.New("document must have a title")
    }
    
    if len(d.content) > 10*1024*1024 { // 10MB
        return errors.New("document content too large")
    }
    
    return nil
}

// Implement Permissioned
const (
    PermView   = 1
    PermEdit   = 2
    PermDelete = 4
    PermAdmin  = 7 // View + Edit + Delete
)

func (d *TextDocument) CanView(userID string) bool {
    perm, exists := d.permissions[userID]
    return exists && (perm&PermView) != 0
}

func (d *TextDocument) CanEdit(userID string) bool {
    perm, exists := d.permissions[userID]
    return exists && (perm&PermEdit) != 0
}

func (d *TextDocument) CanDelete(userID string) bool {
    perm, exists := d.permissions[userID]
    return exists && (perm&PermDelete) != 0
}

func (d *TextDocument) SetPermission(userID string, perm int) {
    d.permissions[userID] = perm
    d.updated = time.Now()
}

// Implement Versionable
func (d *TextDocument) Version() int {
    return d.version
}

func (d *TextDocument) PreviousVersions() []string {
    return append([]string{}, d.history...)
}

// MemoryStorage implements ContentStorage in memory
type MemoryStorage struct {
    documents map[string]Document
    mu        sync.RWMutex
}

// NewMemoryStorage creates a new in-memory storage
func NewMemoryStorage() *MemoryStorage {
    return &MemoryStorage{
        documents: make(map[string]Document),
    }
}

// Implement ContentReader
func (m *MemoryStorage) Get(ctx context.Context, id string) (BasicContent, error) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    doc, exists := m.documents[id]
    if !exists {
        return nil, errors.New("document not found")
    }
    
    return doc, nil
}

func (m *MemoryStorage) List(ctx context.Context, filter map[string]interface{}) ([]BasicContent, error) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    results := make([]BasicContent, 0)
    
    for _, doc := range m.documents {
        // Apply filters
        match := true
        for key, value := range filter {
            switch key {
            case "tag":
                // Check if document has this tag
                tagVal, ok := value.(string)
                if !ok {
                    continue
                }
                
                if taggable, ok := doc.(Taggable); ok {
                    hasTag := false
                    for _, tag := range taggable.Tags() {
                        if tag == tagVal {
                            hasTag = true
                            break
                        }
                    }
                    match = match && hasTag
                } else {
                    match = false
                }
                
            case "after":
                // Check if document was created after this time
                timeVal, ok := value.(time.Time)
                if !ok {
                    continue
                }
                
                if timestamped, ok := doc.(Timestamped); ok {
                    match = match && timestamped.CreatedAt().After(timeVal)
                } else {
                    match = false
                }
            }
            
            if !match {
                break
            }
        }
        
        if match {
            results = append(results, doc)
        }
    }
    
    return results, nil
}

// Implement ContentWriter
func (m *MemoryStorage) Save(ctx context.Context, content BasicContent) error {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    // We need a Document, not just BasicContent
    doc, ok := content.(Document)
    if !ok {
        return errors.New("content must implement Document interface")
    }
    
    // Validate if possible
    if validatable, ok := doc.(Validatable); ok {
        if err := validatable.Validate(); err != nil {
            return err
        }
    }
    
    // Store the document
    m.documents[doc.ID()] = doc
    return nil
}

func (m *MemoryStorage) Delete(ctx context.Context, id string) error {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    if _, exists := m.documents[id]; !exists {
        return errors.New("document not found")
    }
    
    delete(m.documents, id)
    return nil
}

// ContentService provides high-level content operations
type ContentService struct {
    storage ContentStorage
}

// NewContentService creates a new content service
func NewContentService(storage ContentStorage) *ContentService {
    return &ContentService{
        storage: storage,
    }
}

// CreateDocument creates a new document
func (s *ContentService) CreateDocument(ctx context.Context, title, content string) (Document, error) {
    id := generateID()
    doc := NewTextDocument(id, title)
    err := doc.SetContent(strings.NewReader(content))
    if err != nil {
        return nil, err
    }
    
    err = s.storage.Save(ctx, doc)
    if err != nil {
        return nil, err
    }
    
    return doc, nil
}

// ShareDocument grants permissions to a user
func (s *ContentService) ShareDocument(ctx context.Context, docID, userID string, canEdit bool) error {
    content, err := s.storage.Get(ctx, docID)
    if err != nil {
        return err
    }
    
    doc, ok := content.(*TextDocument)
    if !ok {
        return errors.New("invalid document type")
    }
    
    perm := PermView
    if canEdit {
        perm |= PermEdit
    }
    
    doc.SetPermission(userID, perm)
    return s.storage.Save(ctx, doc)
}

// Usage example
func Example() {
    // Create storage
    storage := NewMemoryStorage()
    
    // Create service
    service := NewContentService(storage)
    
    // Create a document
    ctx := context.Background()
    doc, err := service.CreateDocument(ctx, "Hello World", "This is my first document")
    if err != nil {
        fmt.Println("Error creating document:", err)
        return
    }
    
    // Add metadata
    if desc, ok := doc.(RichContent); ok {
        d := desc.(interface {
            SetDescription(string)
        })
        d.SetDescription("A sample document")
        
        if taggable, ok := desc.(Taggable); ok {
            taggable.AddTag("sample")
            taggable.AddTag("hello")
        }
    }
    
    // Share with a user
    err = service.ShareDocument(ctx, doc.ID(), "user123", true)
    if err != nil {
        fmt.Println("Error sharing document:", err)
        return
    }
    
    // Retrieve the document
    retrieved, err := storage.Get(ctx, doc.ID())
    if err != nil {
        fmt.Println("Error retrieving document:", err)
        return
    }
    
    // Use the document based on what interfaces it implements
    fmt.Println("Document ID:", retrieved.ID())
    fmt.Println("Title:", retrieved.Title())
    
    if ts, ok := retrieved.(Timestamped); ok {
        fmt.Println("Created:", ts.CreatedAt().Format(time.RFC3339))
        fmt.Println("Updated:", ts.UpdatedAt().Format(time.RFC3339))
    }
    
    if desc, ok := retrieved.(Describable); ok {
        fmt.Println("Description:", desc.Description())
    }
    
    if tags, ok := retrieved.(Taggable); ok {
        fmt.Println("Tags:", tags.Tags())
    }
    
    if doc, ok := retrieved.(Document); ok {
        content, _ := io.ReadAll(doc.Content())
        fmt.Println("Content:", string(content))
        fmt.Println("Size:", doc.Size(), "bytes")
    }
    
    // Check permissions
    if perm, ok := retrieved.(Permissioned); ok {
        fmt.Println("User123 can view:", perm.CanView("user123"))
        fmt.Println("User123 can edit:", perm.CanEdit("user123"))
        fmt.Println("User456 can view:", perm.CanView("user456"))
    }
}

// Helper function to generate an ID
func generateID() string {
    b := make([]byte, 16)
    rand.Read(b)
    return hex.EncodeToString(b)
}
```

**Common Pitfalls:**
- Creating large interfaces instead of composing smaller ones
- Not recognizing interface composition opportunities
- Circular dependencies between interfaces
- Adding too many methods to a single interface
- Not following the Interface Segregation Principle
- Including unnecessary methods in interfaces
- Embedding concrete types in interfaces (not allowed)
- Creating interfaces that are too specific to be reusable
- Making interfaces public that should be package-private

**Confusion Questions:**

1. **Q: What is the Interface Segregation Principle and why is it important in Go?**
   
   A: The Interface Segregation Principle (ISP) is a design principle stating that clients should not be forced to depend on interfaces they don't use. In Go, this means creating small, focused interfaces rather than large, monolithic ones.
   
   **Core concept of ISP**:
   
   > "Clients should not be forced to depend upon interfaces that they do not use."
   
   In Go terms, this translates to:
   
   1. **Define minimal interfaces** where each interface has only the methods needed for specific functionality
   2. **Compose larger interfaces** from smaller ones when needed
   3. **Keep client dependencies focused** on just the methods they actually use
   
   **Example - Violating ISP**:
   
   ```go
   // Bad: Large, monolithic interface
   type FileSystem interface {
       Open(name string) (File, error)
       Create(name string) (File, error)
       Remove(name string) error
       Rename(oldpath, newpath string) error
       Stat(name string) (FileInfo, error)
       ReadDir(dirname string) ([]FileInfo, error)
       MkdirAll(path string, perm os.FileMode) error
       Chmod(name string, mode os.FileMode) error
       Chown(name string, uid, gid int) error
       // ...many more methods
   }
   
   // Client only needs read access but depends on the whole interface
   func ReadConfig(fs FileSystem, path string) ([]byte, error) {
       f, err := fs.Open(path)  // Only needs Open
       if err != nil {
           return nil, err
       }
       defer f.Close()
       return io.ReadAll(f)
   }
   ```
   
   **Example - Following ISP**:
   
   ```go
   // Good: Small, focused interfaces
   type FileOpener interface {
       Open(name string) (File, error)
   }
   
   type FileCreator interface {
       Create(name string) (File, error)
   }
   
   type FileRemover interface {
       Remove(name string) error
   }
   
   // Composed interface for clients that need more functionality
   type FileManager interface {
       FileOpener
       FileCreator
       FileRemover
   }
   
   // Client depends only on what it needs
   func ReadConfig(opener FileOpener, path string) ([]byte, error) {
       f, err := opener.Open(path)
       if err != nil {
           return nil, err
       }
       defer f.Close()
       return io.ReadAll(f)
   }
   ```
   
   **Benefits of ISP in Go**:
   
   1. **Easier testing**: Mock only the methods you need
      ```go
      type MockFileOpener struct{}
      
      func (m MockFileOpener) Open(name string) (File, error) {
          return NewMockFile(testData), nil
      }
      
      // Test is simpler - only need to implement Open
      ReadConfig(MockFileOpener{}, "config.json")
      ```
   
   2. **Flexibility**: Types only need to implement what makes sense
      ```go
      type ReadOnlyFS struct{}
      
      // Only implements what it can actually do
      func (fs ReadOnlyFS) Open(name string) (File, error) { /* ... */ }
      // No need to implement Create, Remove, etc.
      
      // Works with ReadConfig because it only needs Open
      ```
   
   3. **Evolution**: Can change implementations without affecting clients
      ```go
      // Original client works with any of these without changes
      ReadConfig(LocalFileSystem{}, "config.json")
      ReadConfig(S3FileSystem{}, "configs/app.json")
      ReadConfig(InMemoryFS{}, "test-config.json")
      ```
   
   4. **Clear intent**: Code expresses exactly what functionality it needs
   
   ISP is particularly important in Go because of its structural typing system and implicit interface implementation. Following ISP leads to more modular, flexible, and maintainable code that aligns with Go's philosophy of simplicity and composability.

2. **Q: How does interface composition differ from struct embedding?**
   
   A: Interface composition and struct embedding are related concepts in Go, but they serve different purposes and have important differences:
   
   **Interface Composition**:
   
   ```go
   type Reader interface {
       Read(p []byte) (n int, err error)
   }
   
   type Closer interface {
       Close() error
   }
   
   // Interface composition
   type ReadCloser interface {
       Reader
       Closer
   }
   ```
   
   In interface composition, the composed interface includes all methods from embedded interfaces.
   
   **Struct Embedding**:
   
   ```go
   type Buffer struct {
       bytes.Buffer  // Embedded struct
       count int
   }
   
   // Methods from bytes.Buffer are promoted to Buffer
   b := Buffer{}
   b.Write([]byte("hello"))  // Uses embedded Buffer's Write method
   ```
   
   In struct embedding, the embedded type's methods and fields are promoted to the containing struct.
   
   **Key differences**:
   
   1. **Purpose**:
      - Interface composition: Builds method sets for abstraction
      - Struct embedding: Implements code reuse and delegation
   
   2. **What's being composed**:
      - Interface composition: Only method signatures (no implementation)
      - Struct embedding: Both data (fields) and behavior (methods)
   
   3. **Implementation requirement**:
      - Interface composition: Types must still implement all methods
      - Struct embedding: Implementation is inherited from embedded type
   
   4. **Relationship to concrete types**:
      - Interface composition: No connection to any concrete type
      - Struct embedding: Directly includes a specific concrete type
   
   5. **Type relationship**:
      - Interface composition: Creates "is-a" relationships at the interface level
      - Struct embedding: Creates "has-a" relationships with delegation
   
   **Example showing the difference**:
   
   ```go
   // Interface composition
   type Walker interface {
       Walk(distance int)
   }
   
   type Swimmer interface {
       Swim(distance int)
   }
   
   // Composed interface
   type Amphibian interface {
       Walker
       Swimmer
   }
   
   // To implement Amphibian, a type needs to provide ALL methods
   type Frog struct {}
   
   func (f Frog) Walk(distance int) {
       fmt.Printf("Frog hops %d meters\n", distance)
   }
   
   func (f Frog) Swim(distance int) {
       fmt.Printf("Frog swims %d meters\n", distance)
   }
   
   // vs. Struct embedding
   type Animal struct {
       Name string
   }
   
   func (a Animal) Eat() {
       fmt.Printf("%s is eating\n", a.Name)
   }
   
   // Dog embeds Animal and automatically gets Eat method
   type Dog struct {
       Animal      // Embedded struct
       Breed string
   }
   
   // No need to implement Eat - it's inherited
   dog := Dog{Animal: Animal{Name: "Buddy"}, Breed: "Labrador"}
   dog.Eat()  // "Buddy is eating"
   ```
   
   **When to use each**:
   
   - **Use interface composition when**:
     - Defining contracts that types must implement
     - Building hierarchies of related behaviors
     - Creating abstractions for polymorphism
     - Following the Interface Segregation Principle
   
   - **Use struct embedding when**:
     - Reusing implementation code
     - Extending existing types
     - Delegating to embedded functionality
     - Implementing the Decorator pattern
   
   These two mechanisms complement each other in Go's type system, with interface composition providing behavioral contracts and struct embedding providing implementation reuse.

3. **Q: What are the best practices for designing interface hierarchies in Go?**
   
   A: Designing effective interface hierarchies in Go requires following certain principles aligned with Go's philosophy:
   
   **1. Start small and specific**:
   
   ```go
   // Good: Start with small, focused interfaces
   type Reader interface {
       Read(p []byte) (n int, err error)
   }
   
   type Writer interface {
       Write(p []byte) (n int, err error)
   }
   
   // Bad: Starting with large interfaces
   type FileHandler interface {
       Open() error
       Read(p []byte) (n int, err error)
       Write(p []byte) (n int, err error)
       Seek(offset int64, whence int) (int64, error)
       Close() error
   }
   ```
   
   **2. Compose interfaces rather than extending them**:
   
   ```go
   // Good: Composition of small interfaces
   type ReadWriter interface {
       Reader
       Writer
   }
   
   type ReadWriteCloser interface {
       Reader
       Writer
       Closer
   }
   
   // Bad: Monolithic interface or inheritance-like structure
   type ReaderPlus interface {
       Read(p []byte) (n int, err error)
       ReadAt(p []byte, off int64) (n int, err error)
       ReadFrom(r Reader) (n int64, err error)
   }
   ```
   
   **3. Define interfaces at the consumer, not the implementation**:
   
   ```go
   // Good: Define interfaces where they're used
   func Copy(dst Writer, src Reader) (int64, error) {
       // Uses interfaces defined by what Copy needs
   }
   
   // Bad: Defining interfaces with the implementation
   type Database interface { /* methods */ }
   
   type PostgresDB struct{}
   type MySQLDB struct{}
   // Both implement Database
   ```
   
   **4. Use embedding to inherit behavior, not to create type hierarchies**:
   
   ```go
   // Good: Embedding for behavior reuse
   type BaseHandler struct{}
   
   func (b BaseHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // Common handling logic
   }
   
   type UserHandler struct {
       BaseHandler  // Embed for reuse, not hierarchy
       userService UserService
   }
   
   // Bad: Trying to create class-like hierarchies
   type Animal interface { /* ... */ }
   type Mammal interface { Animal; /* ... */ }  // Not how Go works!
   ```
   
   **5. Follow the Interface Segregation Principle**:
   
   ```go
   // Good: Client-specific interfaces
   type ConfigLoader interface {
       LoadConfig() (Config, error)
   }
   
   type ConfigSaver interface {
       SaveConfig(Config) error
   }
   
   // Bad: Forcing clients to depend on methods they don't need
   type ConfigManager interface {
       LoadConfig() (Config, error)
       SaveConfig(Config) error
       ValidateConfig(Config) error
       MigrateConfig(Config) (Config, error)
   }
   ```
   
   **6. Discover interfaces, don't design them upfront**:
   
   ```go
   // Good: Extract interfaces after patterns emerge
   // Start with concrete implementation:
   type fileStore struct { /* ... */ }
   func (f *fileStore) Get(id string) ([]byte, error) { /* ... */ }
   func (f *fileStore) Set(id string, data []byte) error { /* ... */ }
   
   // Later extract interface when needed:
   type Store interface {
       Get(id string) ([]byte, error)
       Set(id string, data []byte) error
   }
   
   // Function that works with any Store
   func Process(s Store, id string) error {
       // ...
   }
   
   // Bad: Creating interfaces before implementations
   type UserRepository interface {
       GetUser(id string) (User, error)
       SaveUser(User) error
       DeleteUser(id string) error
       // Methods with no implementations yet
   }
   ```
   
   **7. Keep interfaces in the same package as the code that uses them**:
   
   ```go
   // Good: Interface defined where it's used
   package http
   
   type Handler interface {
       ServeHTTP(ResponseWriter, *Request)
   }
   
   // Bad: Interfaces defined with implementations
   package postgres
   
   // This should be in a higher-level package
   type Database interface {
       Query(query string, args ...interface{}) (Rows, error)
   }
   ```
   
   **8. Use embedding for nesting or layering behavior**:
   
   ```go
   // Good: Layered interfaces
   type LoggingHandler struct {
       Next http.Handler
   }
   
   func (l LoggingHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // Log request
       log.Printf("Request: %s %s", r.Method, r.URL.Path)
       
       // Call next handler
       l.Next.ServeHTTP(w, r)
       
       // Log completion
       log.Println("Request completed")
   }
   
   // Bad: Trying to use inheritance-like patterns
   type BaseHandler struct{}
   func (b BaseHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       // This approach is less flexible than composition
   }
   ```
   
   **9. Use acceptor and provider interfaces**:
   
   ```go
   // Good: Separate interfaces for providing vs. accepting
   
   // Provider interface (implemented by sources)
   type LogProvider interface {
       GetLogs() []LogEntry
   }
   
   // Acceptor interface (implemented by consumers)
   type LogProcessor interface {
       ProcessLogs([]LogEntry) error
   }
   
   // Function using both interfaces
   func TransferLogs(provider LogProvider, processor LogProcessor) error {
       logs := provider.GetLogs()
       return processor.ProcessLogs(logs)
   }
   ```
   
   **10. Test interface adherence using compile-time checks**:
   
   ```go
   // Compile-time check that *fileStore implements Store
   var _ Store = (*fileStore)(nil)
   
   // For implementations required to satisfy multiple interfaces
   var (
       _ io.Reader = (*CustomBuffer)(nil)
       _ io.Writer = (*CustomBuffer)(nil)
       _ io.Closer = (*CustomBuffer)(nil)
   )
   ```
   
   Following these best practices leads to more maintainable, flexible, and idiomatic Go code. The key principle is to keep interfaces small, focused, and composed rather than creating deep, inheritance-like hierarchies common in other object-oriented languages.

### 9. Common Interfaces in the Standard Library

**Concise Explanation:**
Go's standard library includes many well-designed interfaces that serve as excellent patterns for your own code. These interfaces are typically small, focused on a single responsibility, and composable. Learning these common interfaces helps you understand Go's design philosophy and gives you ready-made abstractions to work with when designing your own packages.

**Where to Use:**
- When interacting with standard library functions
- As models for designing your own interfaces
- When implementing standard patterns for your types
- To achieve compatibility with existing Go ecosystem
- For common operations like I/O, comparison, and formatting
- When designing frameworks that extend standard library functionality

**Code Snippet:**
```go
import (
    "fmt"
    "io"
    "sort"
)

// Common interfaces and their implementations

// 1. io.Reader and io.Writer
type LogEntry struct {
    Message string
    Level   string
}

// Implement io.Writer interface
func (l *LogEntry) Write(p []byte) (n int, err error) {
    l.Message = string(p)
    return len(p), nil
}

func processReader(r io.Reader) {
    // Read up to 100 bytes
    data := make([]byte, 100)
    n, err := r.Read(data)
    if err != nil && err != io.EOF {
        fmt.Println("Error reading:", err)
        return
    }
    fmt.Printf("Read %d bytes: %s\n", n, data[:n])
}

// 2. fmt.Stringer
type Person struct {
    Name string
    Age  int
}

// Implement fmt.Stringer interface
func (p Person) String() string {
    return fmt.Sprintf("%s (%d years old)", p.Name, p.Age)
}

// 3. sort.Interface
type ByAge []Person

// Implement sort.Interface methods
func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
    // Using io.Reader and io.Writer
    src := strings.NewReader("Hello, Gophers!")
    dst := &LogEntry{}
    
    // Copy from reader to writer
    n, err := io.Copy(dst, src)
    if err != nil {
        fmt.Println("Error copying:", err)
    }
    fmt.Printf("Copied %d bytes\n", n)
    fmt.Printf("Log entry: %s\n", dst.Message)
    
    // Using fmt.Stringer
    alice := Person{Name: "Alice", Age: 30}
    fmt.Println(alice) // Uses String() method
    
    // Using sort.Interface
    people := []Person{
        {"Bob", 31},
        {"John", 42},
        {"Michael", 17},
        {"Jenny", 26},
    }
    
    sort.Sort(ByAge(people))
    fmt.Println("People sorted by age:")
    for _, p := range people {
        fmt.Println(p)
    }
    
    // Using sort.Slice (modern alternative to sort.Interface)
    sort.Slice(people, func(i, j int) bool {
        return people[i].Name < people[j].Name
    })
    fmt.Println("People sorted by name:")
    for _, p := range people {
        fmt.Println(p)
    }
}
```

**Real-World Example:**
Building a data processing pipeline using standard interfaces:

```go
package main

import (
    "bufio"
    "bytes"
    "compress/gzip"
    "crypto/sha256"
    "encoding/csv"
    "encoding/json"
    "errors"
    "fmt"
    "io"
    "os"
    "sort"
    "strings"
)

// Record represents a data record
type Record struct {
    ID      string   `json:"id"`
    Name    string   `json:"name"`
    Email   string   `json:"email"`
    Tags    []string `json:"tags"`
    Score   float64  `json:"score"`
    Active  bool     `json:"active"`
}

// Implement fmt.Stringer for better display
func (r Record) String() string {
    return fmt.Sprintf("Record{%s, %s, %s, %v}", r.ID, r.Name, r.Email, r.Active)
}

// RecordProcessor defines the interface for processing records
type RecordProcessor interface {
    Process(records []Record) ([]Record, error)
}

// Filter is a processor that filters records based on a predicate
type Filter struct {
    Predicate func(Record) bool
}

func (f Filter) Process(records []Record) ([]Record, error) {
    var result []Record
    
    for _, record := range records {
        if f.Predicate(record) {
            result = append(result, record)
        }
    }
    
    return result, nil
}

// Transformer modifies records
type Transformer struct {
    Transform func(Record) Record
}

func (t Transformer) Process(records []Record) ([]Record, error) {
    result := make([]Record, len(records))
    
    for i, record := range records {
        result[i] = t.Transform(record)
    }
    
    return result, nil
}

// Sorter sorts records based on a comparison function
type Sorter struct {
    Less func(a, b Record) bool
}

func (s Sorter) Process(records []Record) ([]Record, error) {
    result := make([]Record, len(records))
    copy(result, records)
    
    sort.Slice(result, func(i, j int) bool {
        return s.Less(result[i], result[j])
    })
    
    return result, nil
}

// Pipeline chains multiple processors together
type Pipeline []RecordProcessor

func (p Pipeline) Process(records []Record) ([]Record, error) {
    current := records
    var err error
    
    for _, processor := range p {
        current, err = processor.Process(current)
        if err != nil {
            return nil, err
        }
    }
    
    return current, nil
}

// RecordReader defines an interface for reading records from a source
type RecordReader interface {
    ReadRecords() ([]Record, error)
}

// RecordWriter defines an interface for writing records to a destination
type RecordWriter interface {
    WriteRecords([]Record) error
}

// JSONRecordReader reads records from JSON
type JSONRecordReader struct {
    Reader io.Reader
}

func (r JSONRecordReader) ReadRecords() ([]Record, error) {
    var records []Record
    dec := json.NewDecoder(r.Reader)
    err := dec.Decode(&records)
    return records, err
}

// JSONRecordWriter writes records as JSON
type JSONRecordWriter struct {
    Writer io.Writer
    Pretty bool
}

func (w JSONRecordWriter) WriteRecords(records []Record) error {
    enc := json.NewEncoder(w.Writer)
    if w.Pretty {
        enc.SetIndent("", "  ")
    }
    return enc.Encode(records)
}

// CSVRecordReader reads records from CSV
type CSVRecordReader struct {
    Reader io.Reader
}

func (r CSVRecordReader) ReadRecords() ([]Record, error) {
    csvReader := csv.NewReader(r.Reader)
    
    // Read header
    header, err := csvReader.Read()
    if err != nil {
        return nil, err
    }
    
    // Map header columns to indices
    colMap := make(map[string]int)
    for i, col := range header {
        colMap[strings.ToLower(col)] = i
    }
    
    // Check required columns
    required := []string{"id", "name", "email", "tags", "score", "active"}
    for _, req := range required {
        if _, ok := colMap[req]; !ok {
            return nil, fmt.Errorf("missing required column: %s", req)
        }
    }
    
    var records []Record
    
    // Read data rows
    for {
        row, err := csvReader.Read()
        if err == io.EOF {
            break
        }
        if err != nil {
            return nil, err
        }
        
        // Parse row into record
        record := Record{
            ID:    row[colMap["id"]],
            Name:  row[colMap["name"]],
            Email: row[colMap["email"]],
            Tags:  strings.Split(row[colMap["tags"]], ","),
        }
        
        // Parse score
        if _, err := fmt.Sscanf(row[colMap["score"]], "%f", &record.Score); err != nil {
            return nil, fmt.Errorf("invalid score in row with ID %s: %v", record.ID, err)
        }
        
        // Parse active status
        record.Active = strings.ToLower(row[colMap["active"]]) == "true"
        
        records = append(records, record)
    }
    
    return records, nil
}

// CSVRecordWriter writes records to CSV
type CSVRecordWriter struct {
    Writer io.Writer
}

func (w CSVRecordWriter) WriteRecords(records []Record) error {
    csvWriter := csv.NewWriter(w.Writer)
    
    // Write header
    header := []string{"ID", "Name", "Email", "Tags", "Score", "Active"}
    if err := csvWriter.Write(header); err != nil {
        return err
    }
    
    // Write records
    for _, record := range records {
        row := []string{
            record.ID,
            record.Name,
            record.Email,
            strings.Join(record.Tags, ","),
            fmt.Sprintf("%.2f", record.Score),
            fmt.Sprintf("%t", record.Active),
        }
        
        if err := csvWriter.Write(row); err != nil {
            return err
        }
    }
    
    csvWriter.Flush()
    return csvWriter.Error()
}

// LineRecordReader reads records from a simple line-based format
type LineRecordReader struct {
    Reader io.Reader
}

func (r LineRecordReader) ReadRecords() ([]Record, error) {
    var records []Record
    scanner := bufio.NewScanner(r.Reader)
    
    for scanner.Scan() {
        line := scanner.Text()
        if line == "" {
            continue
        }
        
        // Simple format: id|name|email|tags|score|active
        parts := strings.Split(line, "|")
        if len(parts) != 6 {
            return nil, fmt.Errorf("invalid line format: %s", line)
        }
        
        var score float64
        if _, err := fmt.Sscanf(parts[4], "%f", &score); err != nil {
            return nil, fmt.Errorf("invalid score in line: %s", line)
        }
        
        records = append(records, Record{
            ID:     parts[0],
            Name:   parts[1],
            Email:  parts[2],
            Tags:   strings.Split(parts[3], ","),
            Score:  score,
            Active: parts[5] == "true",
        })
    }
    
    if err := scanner.Err(); err != nil {
        return nil, err
    }
    
    return records, nil
}

// CompressedReader wraps an io.Reader to provide transparent decompression
type CompressedReader struct {
    reader io.Reader
}

func NewCompressedReader(r io.Reader) (*CompressedReader, error) {
    // Read the first few bytes to check for gzip header
    header := make([]byte, 2)
    
    // Peek at the header
    if _, err := r.Read(header); err != nil {
        return nil, err
    }
    
    // Create a reader that puts the header back
    reader := io.MultiReader(bytes.NewReader(header), r)
    
    // Check for gzip magic number
    if header[0] == 0x1f && header[1] == 0x8b {
        // It's gzip data, create a gzip reader
        gzReader, err := gzip.NewReader(reader)
        if err != nil {
            return nil, err
        }
        return &CompressedReader{reader: gzReader}, nil
    }
    
    // Not compressed, just return the original reader
    return &CompressedReader{reader: reader}, nil
}

func (cr *CompressedReader) Read(p []byte) (int, error) {
    return cr.reader.Read(p)
}

// DataProcessor handles data processing using interfaces
type DataProcessor struct {
    Reader     RecordReader
    Processors []RecordProcessor
    Writer     RecordWriter
}

// ProcessData reads, processes, and writes records
func (dp *DataProcessor) ProcessData() error {
    // Read records
    records, err := dp.Reader.ReadRecords()
    if err != nil {
        return fmt.Errorf("reading records: %w", err)
    }
    
    // Create pipeline from processors
    pipeline := Pipeline(dp.Processors)
    
    // Process records
    processed, err := pipeline.Process(records)
    if err != nil {
        return fmt.Errorf("processing records: %w", err)
    }
    
    // Write processed records
    err = dp.Writer.WriteRecords(processed)
    if err != nil {
        return fmt.Errorf("writing records: %w", err)
    }
    
    return nil
}

// Additional processors

// Deduplicate removes duplicate records based on ID
type Deduplicate struct{}

func (d Deduplicate) Process(records []Record) ([]Record, error) {
    seen := make(map[string]bool)
    var result []Record
    
    for _, record := range records {
        if !seen[record.ID] {
            seen[record.ID] = true
            result = append(result, record)
        }
    }
    
    return result, nil
}

// Validator checks records for validity
type Validator struct{}

func (v Validator) Process(records []Record) ([]Record, error) {
    var validRecords []Record
    var invalidIDs []string
    
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    
    for _, record := range records {
        valid := true
        
        // Check ID
        if record.ID == "" {
            invalidIDs = append(invalidIDs, "record with empty ID")
            valid = false
        }
        
        // Check Name
        if record.Name == "" {
            invalidIDs = append(invalidIDs, record.ID+": empty name")
            valid = false
        }
        
        // Check Email
        if !emailRegex.MatchString(record.Email) {
            invalidIDs = append(invalidIDs, record.ID+": invalid email")
            valid = false
        }
        
        // Check Score
        if record.Score < 0 {
            invalidIDs = append(invalidIDs, record.ID+": negative score")
            valid = false
        }
        
        if valid {
            validRecords = append(validRecords, record)
        }
    }
    
    if len(invalidIDs) > 0 {
        return validRecords, fmt.Errorf("found %d invalid records: %s", 
                                       len(invalidIDs), strings.Join(invalidIDs[:min(3, len(invalidIDs))], ", "))
    }
    
    return validRecords, nil
}

// Hasher adds a hash to each record name
type Hasher struct{}

func (h Hasher) Process(records []Record) ([]Record, error) {
    for i := range records {
        hash := sha256.Sum256([]byte(records[i].Email))
        records[i].ID = fmt.Sprintf("%x", hash[:8])
    }
    
    return records, nil
}

// Helper function for min of two integers
func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}

// Example usage
func main() {
    // Create sample data
    records := []Record{
        {ID: "1", Name: "Alice", Email: "alice@example.com", Tags: []string{"vip", "customer"}, Score: 94.5, Active: true},
        {ID: "2", Name: "Bob", Email: "bob@example.com", Tags: []string{"customer"}, Score: 82.0, Active: true},
        {ID: "3", Name: "Charlie", Email: "charlie@example.com", Tags: []string{"trial"}, Score: 65.5, Active: false},
        {ID: "4", Name: "Dave", Email: "dave@example.com", Tags: []string{"vip", "trial"}, Score: 91.2, Active: true},
        {ID: "5", Name: "Eve", Email: "eve@example.com", Tags: []string{"customer"}, Score: 77.8, Active: true},
    }
    
    // Create JSON data
    jsonBuffer := &bytes.Buffer{}
    jsonWriter := JSONRecordWriter{Writer: jsonBuffer, Pretty: true}
    jsonWriter.WriteRecords(records)
    
    // Create a reader from the JSON
    jsonReader := JSONRecordReader{Reader: jsonBuffer}
    
    // Create processors
    activeFilter := Filter{
        Predicate: func(r Record) bool { return r.Active },
    }
    
    highScoreFilter := Filter{
        Predicate: func(r Record) bool { return r.Score >= 80.0 },
    }
    
    nameTransformer := Transformer{
        Transform: func(r Record) Record {
            r.Name = strings.ToUpper(r.Name)
            return r
        },
    }
    
    scoreSort := Sorter{
        Less: func(a, b Record) bool { return a.Score > b.Score }, // Descending
    }
    
    // Create pipeline
    processor := DataProcessor{
        Reader: jsonReader,
        Processors: []RecordProcessor{
            activeFilter,
            highScoreFilter,
            nameTransformer,
            scoreSort,
        },
        Writer: JSONRecordWriter{
            Writer: os.Stdout,
            Pretty: true,
        },
    }
    
    // Run the pipeline
    fmt.Println("Processing records...")
    if err := processor.ProcessData(); err != nil {
        fmt.Printf("Error: %v\n", err)
        os.Exit(1)
    }
    
    // CSV example
    fmt.Println("\nConverting to CSV...")
    csvBuffer := &bytes.Buffer{}
    csvWriter := CSVRecordWriter{Writer: csvBuffer}
    
    // Create a new processor for CSV conversion
    csvProcessor := DataProcessor{
        Reader: jsonReader,
        Processors: []RecordProcessor{
            scoreSort, // Sort by score
        },
        Writer: csvWriter,
    }
    
    if err := csvProcessor.ProcessData(); err != nil {
        fmt.Printf("Error: %v\n", err)
        os.Exit(1)
    }
    
    fmt.Println("CSV output:")
    fmt.Println(csvBuffer.String())
}
```

**Common Pitfalls:**
- Not knowing when to use existing interfaces versus creating new ones
- Implementing interfaces incorrectly (missing methods or incorrect signatures)
- Not checking documentation for interface behavior expectations
- Using interfaces that are unnecessarily complex for your needs
- Ignoring context-specific interfaces (like context.Context usage)
- Not understanding the performance implications of some interfaces
- Misusing error handling patterns in interfaces like io.Reader
- Attempting to modify standard library interfaces

**Confusion Questions:**

1. **Q: What are the most important interfaces in the Go standard library and when should I use them?**
   
   A: The Go standard library contains several fundamental interfaces that appear frequently and serve as excellent examples of interface design. Here's a guide to the most important ones:
   
   **1. io.Reader and io.Writer**: The foundation of I/O operations
   
   ```go
   type Reader interface {
       Read(p []byte) (n int, err error)
   }
   
   type Writer interface {
       Write(p []byte) (n int, err error)
   }
   ```
   
   **When to use**: Whenever you deal with reading or writing data, regardless of the source or destination. These interfaces abstract away the details of where data comes from or goes to.
   
   **Examples**:
   - Reading files, network connections, or in-memory buffers
   - Writing to files, HTTP responses, or string buffers
   - Processing streams of data
   
   **2. fmt.Stringer**: For string representation
   
   ```go
   type Stringer interface {
       String() string
   }
   ```
   
   **When to use**: When you want to provide a human-readable string representation of your type, especially for debugging or display.
   
   **Examples**:
   - Custom data types that need a string representation
   - Types that will be printed with `fmt.Print` family of functions
   
   **3. error**: For error handling
   
   ```go
   type error interface {
       Error() string
   }
   ```
   
   **When to use**: When you need to create custom error types that carry additional context or behavior.
   
   **Examples**:
   - Application-specific errors with special handling
   - Errors that wrap other errors
   - Errors with additional context or metadata
   
   **4. sort.Interface**: For sorting
   
   ```go
   type Interface interface {
       Len() int
       Less(i, j int) bool
       Swap(i, j int)
   }
   ```
   
   **When to use**: When you need to sort custom collections or in custom ways. Note that since Go 1.8, `sort.Slice` provides a simpler alternative.
   
   **Examples**:
   - Sorting custom slice types
   - Implementing custom sort orders
   
   **5. context.Context**: For request-scoped data and cancellation
   
   ```go
   type Context interface {
       Deadline() (deadline time.Time, ok bool)
       Done() <-chan struct{}
       Err() error
       Value(key interface{}) interface{}
   }
   ```
   
   **When to use**: When you need to carry request-scoped values, deadlines, or cancellation signals across API boundaries and between goroutines.
   
   **Examples**:
   - HTTP handlers passing context through the call chain
   - Operations that need timeouts or cancellation
   - Services needing to propagate request metadata
   
   **6. encoding.TextMarshaler/TextUnmarshaler**: For text encoding/decoding
   
   ```go
   type TextMarshaler interface {
       MarshalText() (text []byte, err error)
   }
   
   type TextUnmarshaler interface {
       UnmarshalText(text []byte) error
   }
   ```
   
   **When to use**: When you need to convert your types to and from text representations.
   
   **Examples**:
   - Custom types that need to marshal to/from JSON
   - Types that have a canonical string representation
   
   **7. http.Handler**: For HTTP request handling
   
   ```go
   type Handler interface {
       ServeHTTP(ResponseWriter, *Request)
   }
   ```
   
   **When to use**: When you need to create HTTP handlers for web servers or middleware.
   
   **Examples**:
   - Web server endpoints
   - Middleware components
   - API handlers
   
   **8. database/sql.Scanner and driver.Valuer**: For database interactions
   
   ```go
   type Scanner interface {
       Scan(value interface{}) error
   }
   
   type Valuer interface {
       Value() (Value, error)
   }
   ```
   
   **When to use**: When you need to customize how your types are stored in or retrieved from databases.
   
   **Examples**:
   - Custom database types
   - Objects that need special serialization for storage
   
   **9. io.Closer**: For resource cleanup
   
   ```go
   type Closer interface {
       Close() error
   }
   ```
   
   **When to use**: When your type manages resources that need explicit cleanup.
   
   **Examples**:
   - File handlers
   - Network connections
   - Database connections
   
   **10. image.Image**: For image processing
   
   ```go
   type Image interface {
       ColorModel() color.Model
       Bounds() Rectangle
       At(x, y int) color.Color
   }
   ```
   
   **When to use**: When you're working with image data or implementing custom image types.
   
   These standard interfaces provide consistent patterns that you can build upon in your own code, promoting interoperability and code reuse throughout the Go ecosystem.

2. **Q: How should I implement the io.Reader and io.Writer interfaces correctly?**
   
   A: Implementing `io.Reader` and `io.Writer` correctly requires understanding their contracts and expected behaviors. Here's a guide to proper implementation:
   
   **io.Reader Interface**:
   
   ```go
   type Reader interface {
       Read(p []byte) (n int, err error)
   }
   ```
   
   **Key requirements**:
   
   1. It should read up to `len(p)` bytes into `p`
   2. It should return the number of bytes read (`0 <= n <= len(p)`) and any error
   3. It should return `io.EOF` when there's no more input
   4. It should return `n > 0` bytes even if an error occurs (other than EOF)
   5. It should not retain `p`
   
   **Correct implementation**:
   
   ```go
   type StringReader struct {
       s string
       i int // Current reading index
   }
   
   func (r *StringReader) Read(p []byte) (n int, err error) {
       // Check if we're at the end
       if r.i >= len(r.s) {
           return 0, io.EOF
       }
       
       // Calculate how many bytes to read
       n = copy(p, r.s[r.i:])
       r.i += n
       
       // Return EOF if we've read everything
       if r.i >= len(r.s) {
           err = io.EOF
       }
       
       return n, err
   }
   ```
   
   **io.Writer Interface**:
   
   ```go
   type Writer interface {
       Write(p []byte) (n int, err error)
   }
   ```
   
   **Key requirements**:
   
   1. It should write `len(p)` bytes from `p` to the underlying data stream
   2. It should return the number of bytes written (`0 <= n <= len(p)`) and any error
   3. If `n < len(p)`, it must return an error explaining why the write is short
   4. It should not modify the data in `p`
   5. It should not retain `p`
   
   **Correct implementation**:
   
   ```go
   type BufferWriter struct {
       buffer []byte
   }
   
   func (w *BufferWriter) Write(p []byte) (n int, err error) {
       // Append the data to our buffer
       w.buffer = append(w.buffer, p...)
       return len(p), nil
   }
   ```
   
   **Common errors to avoid**:
   
   1. **Returning wrong number of bytes**:
      ```go
      // WRONG!
      func (r *BadReader) Read(p []byte) (n int, err error) {
          // Always claiming to have read len(p) bytes when we didn't
          return len(p), nil
      }
      ```
   
   2. **Not returning io.EOF properly**:
      ```go
      // WRONG!
      func (r *BadReader) Read(p []byte) (n int, err error) {
          // Returning EOF even when we read some bytes
          n = copy(p, r.data[r.pos:])
          r.pos += n
          return n, io.EOF  // Should only return EOF when n == 0
      }
      ```
   
   3. **Modifying the slice passed to Write**:
      ```go
      // WRONG!
      func (w *BadWriter) Write(p []byte) (n int, err error) {
          // Modifying the input is not allowed
          for i := range p {
              p[i] = byte(unicode.ToUpper(rune(p[i])))
          }
          // ...
      }
      ```
   
   4. **Returning inconsistent errors**:
      ```go
      // WRONG!
      func (w *BadWriter) Write(p []byte) (n int, err error) {
          // Returning error but saying all bytes were written
          return len(p), errors.New("write failed")
      }
      ```
   
   **Testing your implementations**:
   
   ```go
   func TestReader(t *testing.T) {
       r := &MyReader{data: "hello world"}
       buf := make([]byte, 5)
       
       n, err := r.Read(buf)
       if n != 5 || err != nil || string(buf) != "hello" {
           t.Errorf("First read failed: got %d bytes, error %v, content %q", n, err, buf)
       }
       
       n, err = r.Read(buf)
       if n != 5 || err != nil || string(buf) != " worl" {
           t.Errorf("Second read failed: got %d bytes, error %v, content %q", n, err, buf)
       }
       
       n, err = r.Read(buf)
       if n != 1 || err != io.EOF || string(buf[:n]) != "d" {
           t.Errorf("Third read failed: got %d bytes, error %v, content %q", n, err, buf[:n])
       }
       
       n, err = r.Read(buf)
       if n != 0 || err != io.EOF {
           t.Errorf("Fourth read failed: expected 0, EOF got %d, %v", n, err)
       }
   }
   ```
   
   **Real-world examples**:
   
   - `bytes.Buffer` implements both interfaces
   - `os.File` implements both interfaces
   - `strings.Reader` implements io.Reader
   
   Correctly implementing these interfaces ensures your code works seamlessly with the entire ecosystem of I/O functions in the standard library.

3. **Q: How do error interfaces work in Go and what's the best way to implement custom errors?**
   
   A: Error handling in Go is centered around the `error` interface, which is simple yet powerful. Understanding how to implement and work with custom errors is essential for robust error handling.
   
   **The error interface**:
   
   ```go
   type error interface {
       Error() string
   }
   ```
   
   This minimal interface requires just a single method that returns a string representation of the error.
   
   **Basic custom error implementation**:
   
   ```go
   // Simple string-based error
   type SimpleError string
   
   func (e SimpleError) Error() string {
       return string(e)
   }
   
   // Usage
   const ErrNotFound = SimpleError("resource not found")
   ```
   
   **Structured error with fields**:
   
   ```go
   type ValidationError struct {
       Field string
       Value string
       Reason string
   }
   
   func (e ValidationError) Error() string {
       return fmt.Sprintf("validation failed for field %s with value %s: %s", 
                         e.Field, e.Value, e.Reason)
   }
   ```
   
   **Error wrapping (Go 1.13+)**:
   
   Go 1.13 introduced improved error handling with `errors.Unwrap`, `errors.Is`, and `errors.As`:
   
   ```go
   type QueryError struct {
       Query string
       Err   error // Wrapped error
   }
   
   func (e *QueryError) Error() string {
       return fmt.Sprintf("query error for %q: %v", e.Query, e.Err)
   }
   
   // Implement Unwrap method for error wrapping
   func (e *QueryError) Unwrap() error {
       return e.Err
   }
   
   // Usage
   if err := executeQuery("SELECT * FROM users"); err != nil {
       var qe *QueryError
       if errors.As(err, &qe) {
           fmt.Printf("Query failure: %s\n", qe.Query)
       }
       
       if errors.Is(err, sql.ErrNoRows) {
           fmt.Println("No records found")
       }
   }
   ```
   
   **Best practices for custom errors**:
   
   1. **Use error variables for sentinel errors**:
      ```go
      var (
          ErrNotFound = errors.New("resource not found")
          ErrPermissionDenied = errors.New("permission denied")
      )
      
      func GetResource(id string) (*Resource, error) {
          // ...
          return nil, ErrNotFound
      }
      
      // Check by comparing
      err := GetResource("123")
      if errors.Is(err, ErrNotFound) {
          // Handle not found case
      }
      ```
   
   2. **Use error types for detailed errors**:
      ```go
      type NotFoundError struct {
          ResourceType string
          ID           string
      }
      
      func (e *NotFoundError) Error() string {
          return fmt.Sprintf("%s with ID %s not found", e.ResourceType, e.ID)
      }
      ```
   
   3. **Implement custom behavior methods**:
      ```go
      type HTTPError struct {
          Code    int
          Message string
      }
      
      func (e HTTPError) Error() string {
          return fmt.Sprintf("HTTP error %d: %s", e.Code, e.Message)
      }
      
      // Custom method for HTTP status code
      func (e HTTPError) StatusCode() int {
          return e.Code
      }
      
      // Usage
      if err != nil {
          var httpErr HTTPError
          if errors.As(err, &httpErr) {
              http.Error(w, httpErr.Error(), httpErr.StatusCode())
              return
          }
      }
      ```
   
   4. **Proper error wrapping**:
      ```go
      // Pre-Go 1.13:
      func doSomething() error {
          err := doSomethingElse()
          if err != nil {
              return fmt.Errorf("something failed: %v", err)
          }
          return nil
      }
      
      // Go 1.13+ with %w verb for wrapping:
      func doSomething() error {
          err := doSomethingElse()
          if err != nil {
              return fmt.Errorf("something failed: %w", err)
          }
          return nil
      }
      ```
   
   5. **Testing error types and values**:
      ```go
      func TestErrors(t *testing.T) {
          // Test sentinel error
          err := processItem("invalid")
          if !errors.Is(err, ErrInvalidInput) {
              t.Errorf("expected ErrInvalidInput, got %v", err)
          }
          
          // Test error type
          var validationErr *ValidationError
          if !errors.As(err, &validationErr) {
              t.Errorf("expected ValidationError, got %v", err)
          }
          
          // Test wrapped error
          if !errors.Is(err, ErrBase) {
              t.Errorf("expected wrapped ErrBase, got %v", err)
          }
      }
      ```
   
   **Common error handling patterns**:
   
   1. **Return errors unmodified for leaf functions**:
      ```go
      func readFile(path string) ([]byte, error) {
          return os.ReadFile(path) // Return error directly
      }
      ```
   
   2. **Add context when returning errors up the call stack**:
      ```go
      func processConfig(path string) error {
          data, err := readFile(path)
          if err != nil {
              return fmt.Errorf("reading config file %s: %w", path, err)
          }
          // ...
      }
      ```
   
   3. **Check error type for special handling**:
      ```go
      if err != nil {
          var pathErr *fs.PathError
          if errors.As(err, &pathErr) {
              log.Printf("Path error on %s: %v", pathErr.Path, pathErr.Err)
              return ErrInvalidPath
          }
          return err
      }
      ```
   
   By following these best practices, you can create a robust error handling system that provides clear, informative error messages while also allowing programmatic error inspection and handling.

## Next Actions

### Exercises and Micro-Projects

1. **Methods Implementation**
   - Create a custom time duration type with helper methods
   - Implement geometric shapes with area and perimeter methods
   - Build a chain-able query builder using method receivers
   - Design a custom string type with transformation methods

2. **Interface Design**
   - Create a pluggable logging system with various backends
   - Design a file filter system with composable filters
   - Implement a simple plugin architecture using interfaces
   - Build a processor pipeline with chainable transformations

3. **Interface Composition**
   - Create layered interfaces for a database access pattern
   - Build a middleware system with composable handlers
   - Implement an event system with listener interfaces
   - Design a content format converter with reader/writer interfaces

4. **Standard Interface Usage**
   - Create custom types implementing io.Reader and io.Writer
   - Implement a custom error type with wrapping
   - Build a custom sort interface for complex data
   - Design a string formatter with the Stringer interface

### Real-World Project: Task Management API with Interfaces

Build a RESTful task management API that demonstrates the use of methods and interfaces:

1. **Project Structure**:
   ```
   task-manager/
   ├── cmd/
   │   └── server/
   │       └── main.go
   ├── internal/
   │   ├── api/
   │   │   ├── handlers.go
   │   │   └── router.go
   │   ├── domain/
   │   │   ├── task.go
   │   │   └── user.go
   │   └── storage/
   │       ├── interfaces.go
   │       ├── memory.go
   │       └── file.go
   ├── pkg/
   │   ├── logging/
   │   │   └── logger.go
   │   └── validation/
   │       └── validator.go
   ├── go.mod
   └── go.sum
   ```

2. **Core Interfaces**:
   - `TaskRepository` for storage operations
   - `TaskService` for business logic
   - `Logger` for customizable logging
   - `Validator` for input validation
   - `Authorizer` for access control

3. **Implementation Requirements**:
   - Storage implementations: in-memory, file-based, and database
   - RESTful API for task CRUD operations
   - Filtering and sorting tasks by various criteria
   - User authentication and authorization
   - Proper error handling with custom error types

4. **Interface and Method Focus**:
   - Use method receivers for domain models
   - Implement standard interfaces where appropriate
   - Design small, focused interfaces
   - Use interface composition for layering
   - Ensure proper error handling patterns

5. **Extension Ideas**:
   - Add notification interfaces for task events
   - Implement different output formats (JSON, CSV, etc.)
   - Add middleware interfaces for request/response processing
   - Create a plugin system for task automation

This project will exercise the concepts of methods, interfaces, interface composition, and standard library interfaces while building a practical application.

## Success Criteria

You've mastered Methods and Interfaces when you can:

1. **Methods**
   - Choose correctly between value and pointer receivers
   - Design clean, consistent method sets for your types
   - Properly implement methods that modify receivers
   - Use method expressions and values when appropriate
   - Handle nil receivers safely with pointer methods

2. **Interfaces**
   - Design small, focused interfaces with clear purposes
   - Implement interfaces implicitly and correctly
   - Use interface composition effectively
   - Perform type assertions and switches safely
   - Handle nil interfaces and nil interface values properly

3. **Interface Usage**
   - Recognize when to use existing standard interfaces
   - Design interfaces at the consumer rather than the implementation
   - Follow the interface segregation principle
   - Test interface compliance with compile-time checks
   - Use interfaces for decoupling and testing

4. **Error Handling**
   - Create proper custom error types
   - Implement error wrapping and unwrapping correctly
   - Use errors.Is and errors.As properly
   - Design clear error hierarchies
   - Follow idiomatic error handling patterns

5. **Interface Best Practices**
   - Keep interfaces small and focused
   - Use composition over inheritance
   - Design for extensibility and testability
   - Follow consistent naming conventions
   - Document interface behavior expectations

## Troubleshooting

### Common Issues and Solutions

1. **Method Receiver Issues**
   - **Problem**: Changes to receiver not persisting
   - **Solution**: Use pointer receivers for methods that modify state
   ```go
   // Wrong:
   func (c Counter) Increment() { c.value++ } // Modifies copy, not original
   
   // Right:
   func (c *Counter) Increment() { c.value++ } // Modifies original
   ```

2. **Nil Pointer Panics**
   - **Problem**: Panic when calling methods on nil pointers
   - **Solution**: Add nil checks in pointer receiver methods
   ```go
   func (c *Counter) Increment() {
       if c == nil {
           return // Gracefully handle nil receiver
       }
       c.value++
   }
   ```

3. **Interface Type Assertion Panics**
   - **Problem**: Panic when type assertion fails
   - **Solution**: Use the "comma, ok" syntax for safe type assertions
   ```go
   // Wrong:
   value := data.(string) // Panics if data is not a string
   
   // Right:
   value, ok := data.(string)
   if ok {
       // Safe to use value as string
   } else {
       // Handle case where data is not a string
   }
   ```

4. **Incorrect Interface Implementation**
   - **Problem**: Type doesn't satisfy interface due to method signatures
   - **Solution**: Use compile-time checks and verify method signatures
   ```go
   // Compile-time check
   var _ io.Reader = (*MyReader)(nil)
   
   // Check method signature
   func (r *MyReader) Read(p []byte) (n int, err error) {
       // Implementation must match interface exactly
   }
   ```

5. **Nil Interface Confusion**
   - **Problem**: Checking if interfaces with nil values are nil
   - **Solution**: Understand that an interface is only nil if both type and value are nil
   ```go
   var err *MyError = nil
   var e error = err        // e is not nil because it has a type (*MyError)
   
   // To safely check:
   if e != nil {
       fmt.Printf("Type: %T, Value: %v\n", e, e)
       if reflect.ValueOf(e).IsNil() {
           fmt.Println("Interface value is nil")
       }
   }
   ```

### Debugging Tips

1. **Print Interface Details**:
   ```go
   func DebugInterface(val interface{}) {
       fmt.Printf("Type: %T, Value: %v\n", val, val)
       
       // Show if interface is nil
       fmt.Printf("Is nil? %v\n", val == nil)
       
       // Show if interface value is nil (for pointer types)
       if val != nil {
           rv := reflect.ValueOf(val)
           if rv.Kind() == reflect.Ptr || rv.Kind() == reflect.Interface {
               fmt.Printf("Points to nil? %v\n", rv.IsNil())
           }
       }
   }
   ```

2. **Verify Interface Satisfaction**:
   ```go
   // At compile time
   var _ MyInterface = (*MyImplementation)(nil)
   
   // At runtime
   func ImplementsInterface(v interface{}, interfaceType interface{}) bool {
       return reflect.TypeOf(v).Implements(
           reflect.TypeOf(interfaceType).Elem())
   }
   ```

3. **Check Method Sets**:
   ```go
   func PrintMethodSet(v interface{}) {
       t := reflect.TypeOf(v)
       fmt.Printf("Method set for %v:\n", t)
       
       for i := 0; i < t.NumMethod(); i++ {
           method := t.Method(i)
           fmt.Printf(" - %s\n", method.Name)
       }
   }
   ```

4. **Trace Type Assertions**:
   ```go
   func SafeTypeAssertion(v interface{}, target interface{}) (interface{}, bool) {
       targetType := reflect.TypeOf(target)
       fmt.Printf("Attempting to assert %T as %v\n", v, targetType)
       
       result := reflect.ValueOf(v).Convert(targetType)
       return result.Interface(), true
   }
   ```

5. **Investigate Interface Wrapping**:
   ```go
   func UnwrapError(err error) {
       fmt.Printf("Error: %v\n", err)
       
       for err != nil {
           fmt.Printf("Type: %T\n", err)
           
           // Try to unwrap
           u, ok := err.(interface{ Unwrap() error })
           if !ok {
               break
           }
           err = u.Unwrap()
           fmt.Printf("Unwrapped: %v\n", err)
       }
   }
   ```

By understanding these common issues and debugging approaches, you'll be well-equipped to diagnose and solve problems with methods and interfaces in your Go code.
