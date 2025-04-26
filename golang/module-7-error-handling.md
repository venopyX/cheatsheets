# Module 7: Error Handling

## Core Problem
Properly handling errors in Go programs to create robust, maintainable software that can gracefully deal with unexpected situations and provide helpful information when things go wrong.

## Key Assumptions
- You understand basic Go syntax and functions
- You know how to write and execute Go programs
- You understand basic flow control (if/else, loops)
- You're familiar with Go's value and return types

## Essential Concepts

### 1. The Error Interface

**Concise Explanation:**
In Go, errors are values that satisfy the built-in `error` interface, which has a single method: `Error() string`. This interface-based approach means errors can be created, passed around, and handled like any other value in Go. The `error` interface allows for simple string error messages and sophisticated custom error types with additional behavior.

**Where to Use:**
- When returning errors from functions
- When checking for and handling error conditions
- When creating custom error types
- When designing packages and APIs that need to report failures
- When distinguishing between different types of errors

**Code Snippet:**
```go
// The error interface definition
type error interface {
    Error() string
}

// Basic function returning an error
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Using the function and handling the error
func main() {
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Result:", result)
}
```

**Real-World Example:**
A file handling library that returns descriptive errors:

```go
package fileutils

import (
    "errors"
    "fmt"
    "os"
    "path/filepath"
)

// ReadTextFile reads a text file and returns its contents as a string
func ReadTextFile(path string) (string, error) {
    // Validate file extension
    ext := filepath.Ext(path)
    if ext != ".txt" && ext != ".md" && ext != ".log" {
        return "", errors.New("unsupported file extension: file must be .txt, .md, or .log")
    }

    // Check if file exists
    info, err := os.Stat(path)
    if err != nil {
        if os.IsNotExist(err) {
            return "", fmt.Errorf("file does not exist: %s", path)
        }
        return "", fmt.Errorf("error accessing file: %w", err)
    }

    // Check if it's a regular file
    if info.IsDir() {
        return "", errors.New("path points to a directory, not a file")
    }

    // Check file size
    if info.Size() > 10*1024*1024 { // 10MB
        return "", fmt.Errorf("file too large: %d bytes (max allowed is 10MB)", info.Size())
    }

    // Read file
    data, err := os.ReadFile(path)
    if err != nil {
        return "", fmt.Errorf("failed to read file: %w", err)
    }

    return string(data), nil
}

// Usage example
func Example() {
    content, err := ReadTextFile("data.txt")
    if err != nil {
        fmt.Println("Error reading file:", err)
        return
    }
    fmt.Println("File content:", content)
}
```

**Common Pitfalls:**
- Ignoring returned errors (the infamous `_ = someFunc()`)
- Using string comparison to check error types which is fragile
- Creating unclear error messages that don't help debugging
- Returning errors without context about what operation failed
- Forgetting to return early after handling an error
- Using unnecessary error variables when `errors.New()` is sufficient
- Not checking for specific error conditions when they matter

**Confusion Questions:**

1. **Q: What's the difference between `errors.New()`, `fmt.Errorf()`, and custom error types?**

   A: These represent three different ways of creating errors in Go, each with different capabilities:

   **`errors.New()`**: The simplest way to create an error with a string message.
   ```go
   err := errors.New("connection timed out")
   ```
   Use it when you need a simple, static error message without variables or special behavior.

   **`fmt.Errorf()`**: Creates an error with a formatted message, similar to `fmt.Printf()`.
   ```go
   err := fmt.Errorf("connection to %s timed out after %d seconds", hostname, timeout)
   ```
   Use it when you need to include dynamic values in your error message.
   
   Since Go 1.13, `fmt.Errorf()` can also wrap another error using `%w` verb:
   ```go
   err := fmt.Errorf("failed to connect: %w", underlyingErr)
   ```
   
   **Custom error types**: Create a type that implements the `error` interface.
   ```go
   type ValidationError struct {
       Field string
       Message string
   }
   
   func (e ValidationError) Error() string {
       return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
   }
   ```
   
   Use custom error types when you need:
   - Additional fields or data with the error
   - Custom behavior or methods
   - Type checking using type assertions or switches
   - A structured way to represent specific error categories

   **When to use each**:
   1. Use `errors.New()` for simple, static error messages
   2. Use `fmt.Errorf()` for formatted error messages or when wrapping other errors
   3. Use custom error types when you need structured error data or specific behavior

   The right choice depends on the complexity of error information needed and how the caller will use the error.

2. **Q: Is returning `nil` the only way to indicate success in Go functions?**

   A: No, returning `nil` for the error value is the conventional way to indicate success, but Go provides several patterns for indicating success or failure:

   **Standard error return pattern** (most common):
   ```go
   func DoSomething() error {
       if successful {
           return nil  // Indicate success
       }
       return errors.New("something went wrong")  // Indicate failure
   }
   ```

   **Multiple return values with error**:
   ```go
   func Divide(a, b float64) (float64, error) {
       if b == 0 {
           return 0, errors.New("division by zero")
       }
       return a / b, nil  // Result and nil error indicate success
   }
   ```

   **Boolean return value** (for simple yes/no operations):
   ```go
   func IsValidEmail(email string) bool {
       // Return true for success/valid, false for failure/invalid
       return emailRegex.MatchString(email)
   }
   ```

   **Error types with success information**:
   ```go
   type Result struct {
       Success bool
       Message string
       Data    interface{}
   }

   func ProcessData(input []byte) Result {
       if valid := validate(input); !valid {
           return Result{Success: false, Message: "Invalid input"}
       }
       processed := process(input)
       return Result{Success: true, Message: "OK", Data: processed}
   }
   ```

   **Using value semantics to indicate status**:
   ```go
   func FindUser(id string) (User, bool) {
       user, exists := userMap[id]
       return user, exists  // Second value indicates if user was found
   }
   ```

   **Using sentinel values**:
   ```go
   func Search(items []string, target string) int {
       for i, item := range items {
           if item == target {
               return i  // Success: return position
           }
       }
       return -1  // Special value indicating "not found"
   }
   ```

   While these alternatives exist, the `error` return pattern is idiomatic in Go because:
   1. It provides a standard way to communicate detailed error information
   2. It works well with Go's multiple return values
   3. It allows for separation between "happy path" and error handling code
   4. It scales from simple errors to complex error types

   In most cases, follow the standard Go convention of returning an error value as the last return value, with `nil` indicating success.

3. **Q: How do I handle multiple error conditions without deeply nesting if-statements?**

   A: Handling multiple error conditions cleanly is essential for readable code. Here are several techniques to avoid the "pyramid of doom":

   **1. Early returns**: Handle errors immediately and return, keeping the main flow unindented.
   
   ```go
   // Instead of this:
   func ProcessFile(path string) error {
       file, err := os.Open(path)
       if err == nil {
           defer file.Close()
           data, err := io.ReadAll(file)
           if err == nil {
               config, err := parseConfig(data)
               if err == nil {
                   return processConfig(config)
               } else {
                   return fmt.Errorf("parse error: %w", err)
               }
           } else {
               return fmt.Errorf("read error: %w", err)
           }
       } else {
           return fmt.Errorf("open error: %w", err)
       }
   }
   
   // Do this:
   func ProcessFile(path string) error {
       file, err := os.Open(path)
       if err != nil {
           return fmt.Errorf("open error: %w", err)
       }
       defer file.Close()
       
       data, err := io.ReadAll(file)
       if err != nil {
           return fmt.Errorf("read error: %w", err)
       }
       
       config, err := parseConfig(data)
       if err != nil {
           return fmt.Errorf("parse error: %w", err)
       }
       
       return processConfig(config)
   }
   ```

   **2. Error handling helper functions**:
   
   ```go
   func handleError(err error, msg string) {
       if err != nil {
           log.Fatalf("%s: %v", msg, err)
       }
   }
   
   func main() {
       file, err := os.Open("config.json")
       handleError(err, "Failed to open config")
       defer file.Close()
       
       data, err := io.ReadAll(file)
       handleError(err, "Failed to read config")
       
       // Continue with processing...
   }
   ```

   **3. Custom error check function**:
   
   ```go
   func check(err error) {
       if err != nil {
           panic(err)  // Will be recovered in a defer function
       }
   }
   
   func ProcessWithPanic() (result string, err error) {
       // Recover from any panics and convert to errors
       defer func() {
           if r := recover(); r != nil {
               if e, ok := r.(error); ok {
                   err = e
               } else {
                   err = fmt.Errorf("%v", r)
               }
           }
       }()
       
       file := check(os.Open("file.txt"))
       defer file.Close()
       
       data := check(io.ReadAll(file))
       config := check(parseConfig(data))
       return processConfig(config), nil
   }
   
   // Helper function to unwrap the error-returning functions
   func check[T any](val T, err error) T {
       if err != nil {
           panic(err)
       }
       return val
   }
   ```

   **4. Using a struct to track error state**:
   
   ```go
   type ErrTracker struct {
       err error
   }
   
   func (e *ErrTracker) Run(fn func() error) {
       if e.err != nil {
           return  // Skip if there's already an error
       }
       e.err = fn()
   }
   
   func ProcessWithTracker() error {
       var et ErrTracker
       var file *os.File
       var data []byte
       
       et.Run(func() error {
           var err error
           file, err = os.Open("config.json")
           return err
       })
       
       defer func() {
           if file != nil {
               file.Close()
           }
       }()
       
       et.Run(func() error {
           var err error
           data, err = io.ReadAll(file)
           return err
       })
       
       et.Run(func() error {
           return processData(data)
       })
       
       return et.err
   }
   ```

   **5. Using Go 1.20+ `errors.Join`** for collecting multiple errors:
   
   ```go
   func ProcessItems(items []Item) error {
       var errs []error
       
       for _, item := range items {
           if err := process(item); err != nil {
               errs = append(errs, fmt.Errorf("item %d: %w", item.ID, err))
           }
       }
       
       return errors.Join(errs...)  // nil if errs is empty
   }
   ```

   **6. For non-error scenarios: separation of concerns**:
   
   ```go
   func ProcessFile(path string) error {
       // Each function handles its own errors
       file, err := openFile(path)
       if err != nil {
           return err
       }
       defer file.Close()
       
       return processFileContents(file)
   }
   
   func openFile(path string) (*os.File, error) {
       // Only handles file opening logic
       return os.Open(path)
   }
   
   func processFileContents(file *os.File) error {
       // Only handles processing logic
       data, err := io.ReadAll(file)
       if err != nil {
           return err
       }
       
       return parseAndProcess(data)
   }
   ```

   The best approach depends on your specific situation, but early returns are generally the most idiomatic Go pattern for error handling, keeping code clean and readable.

### 2. Creating and Returning Errors

**Concise Explanation:**
Go provides several ways to create and return errors, from simple static error messages to complex custom error types. The standard library offers built-in functions for error creation, and you can define your own error types by implementing the `error` interface. Properly creating and returning errors with meaningful context helps callers diagnose and handle issues.

**Where to Use:**
- When a function encounters an invalid input or state
- When an operation fails due to external factors (IO, network, etc.)
- To provide specific information about why an operation failed
- To distinguish between different types of failures
- When you need to include context like function name, input values, or operation details
- When wrapping another error with additional context

**Code Snippet:**
```go
package main

import (
    "errors"
    "fmt"
    "io"
    "os"
)

// Method 1: Using errors.New for simple static messages
func isPositive(n int) error {
    if n < 0 {
        return errors.New("number must be positive")
    }
    return nil
}

// Method 2: Using fmt.Errorf for formatted messages
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide %d by zero", a)
    }
    return a / b, nil
}

// Method 3: Creating a custom error type
type ValidationError struct {
    Field string
    Issue string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation error on field %q: %s", e.Field, e.Issue)
}

// Method 4: Using error wrapping (Go 1.13+)
func readConfig(path string) ([]byte, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("config read failed: %w", err)
    }
    return data, nil
}

// Method 5: Creating constants for sentinel errors
var (
    ErrNotFound = errors.New("item not found")
    ErrTimeout  = errors.New("operation timed out")
)

func findItem(id string) (string, error) {
    // Simulate item not found
    return "", ErrNotFound
}

// Method 6: Using errors.Join to combine multiple errors (Go 1.20+)
func validateUser(user User) error {
    var errs []error
    
    if user.Name == "" {
        errs = append(errs, errors.New("name cannot be empty"))
    }
    
    if user.Age < 0 {
        errs = append(errs, errors.New("age cannot be negative"))
    }
    
    if user.Email == "" {
        errs = append(errs, errors.New("email cannot be empty"))
    }
    
    return errors.Join(errs...)  // Returns nil if errs is empty
}

// Usage examples
func main() {
    // Simple error
    if err := isPositive(-5); err != nil {
        fmt.Println("Error:", err)
    }

    // Formatted error
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("Error:", err)
    }

    // Custom error type
    err := ValidationError{Field: "username", Issue: "too short"}
    fmt.Println("Error:", err)

    // Error wrapping
    _, err = readConfig("nonexistent.conf")
    if err != nil {
        fmt.Println("Error:", err)
        
        // We can unwrap to access the underlying error
        if os.IsNotExist(errors.Unwrap(err)) {
            fmt.Println("The config file doesn't exist!")
        }
    }

    // Sentinel error
    _, err = findItem("123")
    if err == ErrNotFound {
        fmt.Println("Item not found, create a new one")
    }
}

type User struct {
    Name  string
    Age   int
    Email string
}
```

**Real-World Example:**
An HTTP API client library with comprehensive error handling:

```go
package apiclient

import (
    "encoding/json"
    "errors"
    "fmt"
    "io"
    "net/http"
    "time"
)

// Error types for the API client
var (
    ErrNetworkFailure = errors.New("network failure")
    ErrTimeout        = errors.New("request timed out")
    ErrServerError    = errors.New("server error")
    ErrUnauthorized   = errors.New("unauthorized")
    ErrRateLimited    = errors.New("rate limited")
)

// APIError represents an error returned by the API
type APIError struct {
    StatusCode int
    Message    string
    RequestID  string
    Endpoint   string
    Retryable  bool
}

// Error implements the error interface
func (e APIError) Error() string {
    return fmt.Sprintf("API error: %s (status=%d, request=%s, endpoint=%s)", 
        e.Message, e.StatusCode, e.RequestID, e.Endpoint)
}

// IsAPIError checks if an error is an API error
func IsAPIError(err error) (*APIError, bool) {
    var apiErr APIError
    if errors.As(err, &apiErr) {
        return &apiErr, true
    }
    return nil, false
}

// Client is an API client
type Client struct {
    BaseURL     string
    APIKey      string
    HTTPClient  *http.Client
    RetryConfig RetryConfig
}

// RetryConfig configures retry behavior
type RetryConfig struct {
    MaxRetries    int
    InitialDelay  time.Duration
    MaxDelay      time.Duration
    BackoffFactor float64
}

// DefaultRetryConfig returns standard retry settings
func DefaultRetryConfig() RetryConfig {
    return RetryConfig{
        MaxRetries:    3,
        InitialDelay:  100 * time.Millisecond,
        MaxDelay:      2 * time.Second,
        BackoffFactor: 2.0,
    }
}

// NewClient creates a new API client
func NewClient(baseURL, apiKey string) *Client {
    return &Client{
        BaseURL:     baseURL,
        APIKey:      apiKey,
        HTTPClient:  &http.Client{Timeout: 10 * time.Second},
        RetryConfig: DefaultRetryConfig(),
    }
}

// Get performs a GET request to the API
func (c *Client) Get(endpoint string, result interface{}) error {
    // Build the full URL
    url := c.BaseURL + endpoint

    // Try the request with retries
    var resp *http.Response
    var err error
    var attempt int
    delay := c.RetryConfig.InitialDelay

    for attempt = 0; attempt <= c.RetryConfig.MaxRetries; attempt++ {
        // Make the request
        req, err := http.NewRequest(http.MethodGet, url, nil)
        if err != nil {
            return fmt.Errorf("failed to create request: %w", err)
        }
        
        // Add API key
        req.Header.Set("Authorization", "Bearer "+c.APIKey)
        req.Header.Set("Accept", "application/json")
        
        // Perform the request
        resp, err = c.HTTPClient.Do(req)
        
        // Handle client-side errors that shouldn't be retried
        if err != nil {
            var netErr *url.Error
            if errors.As(err, &netErr) && netErr.Timeout() {
                return fmt.Errorf("%w: %v", ErrTimeout, err)
            }
            return fmt.Errorf("%w: %v", ErrNetworkFailure, err)
        }
        
        // Check if we need to retry based on status code
        if resp.StatusCode == 429 || (resp.StatusCode >= 500 && resp.StatusCode < 600) {
            resp.Body.Close()
            
            // Stop if this was our last attempt
            if attempt == c.RetryConfig.MaxRetries {
                break
            }
            
            // Wait before retrying
            time.Sleep(delay)
            delay = time.Duration(float64(delay) * c.RetryConfig.BackoffFactor)
            if delay > c.RetryConfig.MaxDelay {
                delay = c.RetryConfig.MaxDelay
            }
            continue
        }
        
        // If we got here, we don't need to retry
        break
    }
    
    // Ensure body is closed
    defer resp.Body.Close()
    
    // Handle different status codes
    switch {
    case resp.StatusCode == 200:
        // Success - parse the response body
        if result != nil {
            if err := json.NewDecoder(resp.Body).Decode(result); err != nil {
                return fmt.Errorf("failed to decode response: %w", err)
            }
        }
        return nil
        
    case resp.StatusCode == 401 || resp.StatusCode == 403:
        return fmt.Errorf("%w: status code %d", ErrUnauthorized, resp.StatusCode)
        
    case resp.StatusCode == 404:
        return fmt.Errorf("resource not found: %s", endpoint)
        
    case resp.StatusCode == 429:
        return fmt.Errorf("%w: too many requests", ErrRateLimited)
        
    case resp.StatusCode >= 500:
        return fmt.Errorf("%w: status code %d", ErrServerError, resp.StatusCode)
        
    default:
        // Try to get error details from response
        body, _ := io.ReadAll(resp.Body)
        
        // Attempt to parse API error
        var apiResp struct {
            Error struct {
                Message   string `json:"message"`
                RequestID string `json:"request_id"`
            } `json:"error"`
        }
        
        if err := json.Unmarshal(body, &apiResp); err == nil && apiResp.Error.Message != "" {
            return APIError{
                StatusCode: resp.StatusCode,
                Message:    apiResp.Error.Message,
                RequestID:  apiResp.Error.RequestID,
                Endpoint:   endpoint,
                Retryable:  resp.StatusCode >= 500 || resp.StatusCode == 429,
            }
        }
        
        // Fallback for unexpected responses
        return fmt.Errorf("unexpected API response (status=%d): %s", 
            resp.StatusCode, string(body))
    }
}

// Example usage
func Example() {
    client := NewClient("https://api.example.com", "your-api-key")
    
    var users []User
    err := client.Get("/users", &users)
    
    if err != nil {
        // Check for specific error types
        switch {
        case errors.Is(err, ErrUnauthorized):
            fmt.Println("API key is invalid or expired")
            
        case errors.Is(err, ErrRateLimited):
            fmt.Println("Rate limit exceeded, try again later")
            
        case errors.Is(err, ErrNetworkFailure), errors.Is(err, ErrTimeout):
            fmt.Println("Network issue, check your connection")
            
        default:
            // Check if it's a structured API error
            if apiErr, ok := IsAPIError(err); ok {
                fmt.Printf("API error occurred: %s (request ID: %s)\n",
                    apiErr.Message, apiErr.RequestID)
                if apiErr.Retryable {
                    fmt.Println("This error is retryable")
                }
            } else {
                fmt.Printf("Unexpected error: %v\n", err)
            }
        }
        return
    }
    
    // Process users data
    fmt.Printf("Retrieved %d users\n", len(users))
}

type User struct {
    ID    string `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}
```

**Common Pitfalls:**
- Creating vague error messages that lack context
- Not including relevant parameters or values in error messages
- Exposing sensitive information in error messages
- Creating inconsistent error types throughout a package or program
- Not using existing error types when appropriate
- Over-engineering complex error hierarchies when simple errors would do
- Not documenting the errors a function can return
- Using panics instead of returning errors for expected failure cases
- Not providing a way to distinguish between different error conditions

**Confusion Questions:**

1. **Q: When should I use error wrapping versus creating a new error?**

   A: This is an important decision that affects how errors are handled throughout your program. Here's when to use each approach:

   **Use error wrapping (with `fmt.Errorf()` and `%w`) when:**

   1. **You want to preserve the original error** but add context:
      ```go
      file, err := os.Open("config.json")
      if err != nil {
          return fmt.Errorf("failed to open config file: %w", err)
      }
      ```
      This allows code higher up the call stack to still check for specific error types or values using `errors.Is()` or `errors.As()`.

   2. **You're in a middleware-like layer** that needs to pass errors up while adding context:
      ```go
      func processRequest(req Request) error {
          data, err := validateRequest(req)
          if err != nil {
              return fmt.Errorf("invalid request: %w", err)
          }
          
          result, err := performOperation(data)
          if err != nil {
              return fmt.Errorf("operation failed: %w", err)
          }
          
          return nil
      }
      ```

   3. **The error's original type contains important information** that callers might need:
      ```go
      err := db.QueryRow("...").Scan(&result)
      if err != nil {
          // Wrap to preserve that it's a sql.ErrNoRows
          return fmt.Errorf("database query failed: %w", err)
      }
      ```
      Later, callers can still check: `if errors.Is(err, sql.ErrNoRows) { ... }`

   **Create a new error (without wrapping) when:**

   1. **The underlying error is an implementation detail** that callers shouldn't rely on:
      ```go
      func isUserAdmin(id string) (bool, error) {
          user, err := db.GetUser(id)
          if err != nil {
              // Details about DB errors shouldn't leak to callers
              return false, errors.New("failed to check user permissions")
          }
          return user.IsAdmin, nil
      }
      ```

   2. **You want to present a cleaner abstraction** to your API users:
      ```go
      // Instead of exposing internal errors:
      var ErrUserNotFound = errors.New("user not found")
      
      func GetUser(id string) (*User, error) {
          user, err := db.FindUser(id) 
          if err != nil {
              if errors.Is(err, sql.ErrNoRows) {
                  return nil, ErrUserNotFound // Translate to your API's error
              }
              return nil, fmt.Errorf("user lookup failed: %w", err)
          }
          return user, nil
      }
      ```

   3. **For security reasons**, to prevent information leakage:
      ```go
      func authenticate(username, password string) error {
          hashedPassword, err := getStoredHash(username)
          if err != nil {
              // Don't reveal if the user exists or not
              return errors.New("invalid username or password")
          }
          // ...
      }
      ```

   **Guidelines for deciding:**

   1. **Consider the caller's needs**: Do they need to distinguish between different error causes?
   2. **Think about error handling**: Will the caller need to use `errors.Is()` or `errors.As()`?
   3. **Consider abstraction boundaries**: Are you exposing internal implementation details unnecessarily?
   4. **Security implications**: Could error details leak sensitive information?
   5. **Logging needs**: Will you need the detailed original error for logs even if you present a simplified error to callers?

   In general, wrap errors when moving within the same abstraction layer, and create new errors when crossing abstraction boundaries.

2. **Q: What's the difference between errors.New, fmt.Errorf, and custom error types?**

   A: These three approaches to creating errors in Go have different capabilities, use cases, and trade-offs:

   **1. errors.New** - Simple string-based errors

   ```go
   import "errors"
   
   err := errors.New("something went wrong")
   ```

   **Key characteristics:**
   - Creates an immutable error with a fixed string message
   - Simple and straightforward
   - No formatting options for dynamic values
   - Lightweight and efficient
   - Good for predefined error values (sentinel errors)

   **Best for:**
   - Static error messages without variables
   - Defining package-level error constants
   - Very simple error cases

   **Example use case:**
   ```go
   var (
       ErrNotFound = errors.New("resource not found")
       ErrTimeout = errors.New("operation timed out")
   )
   
   func FindItem(id string) (*Item, error) {
       if id == "" {
           return nil, ErrNotFound
       }
       // ...
   }
   ```

   **2. fmt.Errorf** - Formatted error messages

   ```go
   import "fmt"
   
   err := fmt.Errorf("invalid value %q: must be between %d and %d", value, min, max)
   ```

   **Key characteristics:**
   - Creates an error with a formatted message using printf-style verbs
   - Supports including dynamic values in error messages
   - Can wrap other errors using the `%w` verb (since Go 1.13)
   - Returns a concrete type that just satisfies the error interface

   **Best for:**
   - Error messages that need to include dynamic values
   - Adding context to errors
   - Wrapping other errors while adding information

   **Example use case:**
   ```go
   func ValidateAge(age int) error {
       if age < 0 {
           return fmt.Errorf("age cannot be negative: %d", age)
       }
       if age > 150 {
           return fmt.Errorf("age %d exceeds maximum reasonable age of 150", age)
       }
       return nil
   }
   
   // With error wrapping
   func ReadConfig(path string) (*Config, error) {
       data, err := os.ReadFile(path)
       if err != nil {
           return nil, fmt.Errorf("reading config file %s: %w", path, err)
       }
       // ...
   }
   ```

   **3. Custom error types** - Structured errors with behavior

   ```go
   type ValidationError struct {
       Field string
       Message string
   }
   
   func (e ValidationError) Error() string {
       return fmt.Sprintf("validation error on field %q: %s", e.Field, e.Message)
   }
   ```

   **Key characteristics:**
   - Implements the error interface with a custom type
   - Can include additional fields and data
   - Allows type assertions and type switches for handling specific errors
   - Can have additional methods beyond Error()
   - Most flexible but requires more code

   **Best for:**
   - Errors that need to carry structured data
   - When callers need to extract specific error fields
   - When different error handling is needed based on error type
   - Complex error hierarchies

   **Example use case:**
   ```go
   type HTTPError struct {
       StatusCode int
       Message    string
       URL        string
   }
   
   func (e HTTPError) Error() string {
       return fmt.Sprintf("%d %s: %s", e.StatusCode, http.StatusText(e.StatusCode), e.Message)
   }
   
   // Can add helper methods
   func (e HTTPError) IsServerError() bool {
       return e.StatusCode >= 500
   }
   
   // Usage with type assertion
   resp, err := http.Get(url)
   if err != nil {
       if httpErr, ok := err.(*HTTPError); ok {
           fmt.Printf("HTTP error %d: %s\n", httpErr.StatusCode, httpErr.Message)
           if httpErr.IsServerError() {
               // Retry logic
           }
       }
   }
   ```

   **Decision matrix for choosing between approaches:**

   | Approach | When to use |
   |----------|-------------|
   | `errors.New` | Static error messages, package error constants |
   | `fmt.Errorf` | Dynamic error messages, adding context to errors, wrapping errors |
   | Custom error types | Structured error data, error-specific behavior, error type checking |

   **In practice:**
   1. Start with the simplest approach that meets your needs
   2. Use `errors.New` for simple, static error messages
   3. Use `fmt.Errorf` when you need to include values or wrap errors
   4. Move to custom error types when you need structured data or behavior
   5. Be consistent within a package or module

   Remember that these approaches aren't mutually exclusive - many Go programs use all three depending on the specific error handling needs.

3. **Q: How do I include context in my errors without losing the original error information?**

   A: Including context in errors while preserving the original error information is crucial for debugging and proper error handling. Go 1.13+ provides built-in mechanisms for this through error wrapping.

   **1. Use `fmt.Errorf` with the `%w` verb for wrapping**:

   ```go
   func processFile(path string) error {
       file, err := os.Open(path)
       if err != nil {
           return fmt.Errorf("opening %s: %w", path, err)
       }
       defer file.Close()
       
       data, err := io.ReadAll(file)
       if err != nil {
           return fmt.Errorf("reading content from %s: %w", path, err)
       }
       
       return processData(data)
   }
   ```

   This approach:
   - Adds context (what operation was being performed and on what file)
   - Preserves the original error through wrapping
   - Allows calling code to use `errors.Is` and `errors.As` to check error types

   **2. Create custom error types that wrap original errors**:

   ```go
   type FileError struct {
       Path string
       Op   string
       Err  error  // The wrapped error
   }
   
   func (e *FileError) Error() string {
       return fmt.Sprintf("%s %s: %v", e.Op, e.Path, e.Err)
   }
   
   // Implement Unwrap to support errors.Is and errors.As
   func (e *FileError) Unwrap() error {
       return e.Err
   }
   
   // Usage
   func readConfig(path string) (*Config, error) {
       data, err := os.ReadFile(path)
       if err != nil {
           return nil, &FileError{
               Path: path,
               Op:   "read config",
               Err:  err,
           }
       }
       // ...
   }
   ```

   **3. For multiple layers of context, keep wrapping at each level**:

   ```go
   func main() {
       err := processUserData("user123")
       if err != nil {
           fmt.Printf("Error: %v\n", err)
           // Might print:
           // "Error: processing user user123: loading preferences: opening config.json: open config.json: no such file or directory"
       }
   }
   
   func processUserData(userID string) error {
       err := loadUserPreferences(userID)
       if err != nil {
           return fmt.Errorf("processing user %s: %w", userID, err)
       }
       return nil
   }
   
   func loadUserPreferences(userID string) error {
       err := loadConfigFile("config.json")
       if err != nil {
           return fmt.Errorf("loading preferences: %w", err)
       }
       return nil
   }
   
   func loadConfigFile(path string) error {
       _, err := os.Open(path)
       if err != nil {
           return fmt.Errorf("opening %s: %w", path, err)
       }
       return nil
   }
   ```

   **4. Use structured logging alongside errors for more context**:

   ```go
   func processOrder(orderID string) error {
       order, err := db.GetOrder(orderID)
       if err != nil {
           log.WithFields(log.Fields{
               "order_id": orderID,
               "error":    err,
               "time":     time.Now(),
           }).Error("Failed to get order")
           return fmt.Errorf("retrieving order %s: %w", orderID, err)
       }
       // Process order...
   }
   ```

   **5. For APIs, translate internal errors but log the details**:

   ```go
   func handleGetUser(w http.ResponseWriter, r *http.Request) {
       userID := r.PathValue("id")
       user, err := service.GetUser(userID)
       
       if err != nil {
           // Log the detailed error with context
           log.WithError(err).WithField("user_id", userID).Error("Failed to get user")
           
           // Return appropriate error to the client
           if errors.Is(err, sql.ErrNoRows) {
               http.Error(w, "User not found", http.StatusNotFound)
           } else {
               // Don't expose internal error details to clients
               http.Error(w, "Internal server error", http.StatusInternalServerError)
           }
           return
       }
       
       json.NewEncoder(w).Encode(user)
   }
   ```

   **6. Apply the "onion" pattern - add context at every level**:

   ```go
   // Database layer
   func (r *userRepository) FindUser(id string) (*User, error) {
       var user User
       err := r.db.Get(&user, "SELECT * FROM users WHERE id = ?", id)
       if err != nil {
           return nil, fmt.Errorf("database error finding user %s: %w", id, err)
       }
       return &user, nil
   }
   
   // Service layer
   func (s *userService) GetUser(id string) (*User, error) {
       user, err := s.repo.FindUser(id)
       if err != nil {
           return nil, fmt.Errorf("service error getting user %s: %w", id, err)
       }
       return user, nil
   }
   
   // Handler layer
   func (h *Handler) HandleGetUser(w http.ResponseWriter, r *http.Request) {
       userID := r.PathValue("id")
       user, err := h.service.GetUser(userID)
       if err != nil {
           h.handleError(w, r, fmt.Errorf("API error handling user request: %w", err))
           return
       }
       // Respond with user...
   }
   ```

   **Best practices for error context**:

   1. **Be specific but concise**: Include relevant details but don't make errors too verbose
   2. **Include operation names**: Mention what operation was being performed when the error occurred
   3. **Include relevant identifiers**: Add IDs, paths, or names of resources being processed
   4. **Use consistent formatting**: Establish patterns for error messages in your codebase
   5. **Wrap don't replace**: Use `%w` to preserve the original error chain
   6. **Consider privacy/security**: Don't include sensitive data in errors that might be logged or displayed
   7. **Format for both machines and humans**: Make errors useful for both automatic processing and manual debugging

   By following these practices, you create errors that are helpful for debugging while allowing proper programmatic error handling throughout your application.

### 3. Error Handling Strategies

**Concise Explanation:**
Error handling strategies are patterns and approaches for dealing with errors in your Go programs. Go's explicit error handling through return values encourages developers to think about and handle all possible error conditions, making programs more robust. Different strategies can be applied based on the specific needs of your application, from simple error checking and return to more sophisticated approaches involving error wrapping, custom error types, or domain-specific error handling.

**Where to Use:**
- Throughout your Go codebase wherever errors can occur
- When defining error handling policies for packages or applications
- When deciding how to propagate errors across layers of your application
- When determining whether to handle errors locally or pass them up the call stack
- When considering what information should be included with errors
- When designing APIs that need to report various failure modes

**Code Snippet:**
```go
package main

import (
    "errors"
    "fmt"
    "io"
    "log"
    "os"
)

// Strategy 1: Simple error checking and return
// Good for most cases - propagates errors up the call stack
func readConfig(path string) ([]byte, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("failed to read config from %s: %w", path, err)
    }
    return data, nil
}

// Strategy 2: Sentinel errors for expected conditions
// Good for specific error cases that callers need to handle
var (
    ErrInvalidFormat = errors.New("invalid data format")
    ErrEmptyInput    = errors.New("input cannot be empty")
)

func validateInput(input string) error {
    if input == "" {
        return ErrEmptyInput
    }
    
    if !isValidFormat(input) {
        return ErrInvalidFormat
    }
    
    return nil
}

// Strategy 3: Type-based error handling with custom error types
// Good when errors need to carry structured data
type ValidationError struct {
    Field  string
    Reason string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %s: %s", e.Field, e.Reason)
}

func validateUser(user User) error {
    if user.Name == "" {
        return ValidationError{Field: "name", Reason: "cannot be empty"}
    }
    
    if user.Age < 18 {
        return ValidationError{Field: "age", Reason: "must be at least 18"}
    }
    
    return nil
}

// Strategy 4: Error wrapping for context propagation
// Good for adding context as errors traverse layers
func processFile(path string) error {
    file, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("opening file %s: %w", path, err)
    }
    defer file.Close()
    
    data, err := io.ReadAll(file)
    if err != nil {
        return fmt.Errorf("reading file %s: %w", path, err)
    }
    
    return processData(data)
}

// Strategy 5: Partial results with errors
// Good when you can still return some useful data despite errors
type Result struct {
    Data       []string
    Errors     []error
    Successful int
    Failed     int
}

func processAll(items []string) Result {
    result := Result{
        Data: make([]string, 0, len(items)),
    }
    
    for _, item := range items {
        processed, err := process(item)
        if err != nil {
            result.Errors = append(result.Errors, 
                fmt.Errorf("processing %s: %w", item, err))
            result.Failed++
        } else {
            result.Data = append(result.Data, processed)
            result.Successful++
        }
    }
    
    return result
}

// Strategy 6: Domain-specific error handling
// Good for translating low-level errors to domain concepts
func getUserData(userID string) (*UserData, error) {
    // Low-level operation
    data, err := database.Query("SELECT * FROM users WHERE id = ?", userID)
    if err != nil {
        // Translate to domain-specific errors
        if errors.Is(err, database.ErrNoRows) {
            return nil, UserNotFoundError{ID: userID}
        }
        if errors.Is(err, database.ErrTimeout) {
            return nil, ServiceUnavailableError{Reason: "database timeout"}
        }
        // Wrap other errors
        return nil, fmt.Errorf("database error: %w", err)
    }
    
    return parseUserData(data)
}

// Example usage
func main() {
    // Strategy 1: Simple check and return
    config, err := readConfig("config.json")
    if err != nil {
        log.Fatalf("Config error: %v", err)
    }
    
    // Strategy 2: Sentinel error checking
    err = validateInput("some input")
    if err != nil {
        if errors.Is(err, ErrEmptyInput) {
            fmt.Println("Please provide some input")
        } else if errors.Is(err, ErrInvalidFormat) {
            fmt.Println("Input format is not valid")
        } else {
            fmt.Printf("Unexpected error: %v", err)
        }
    }
    
    // Strategy 3: Type-based error handling
    user := User{Name: "", Age: 16}
    if err := validateUser(user); err != nil {
        var validationErr ValidationError
        if errors.As(err, &validationErr) {
            fmt.Printf("Please fix the %s field: %s\n", 
                validationErr.Field, validationErr.Reason)
        } else {
            fmt.Printf("Validation error: %v\n", err)
        }
    }
    
    // Strategy 5: Partial results
    results := processAll([]string{"item1", "item2", "bad_item"})
    fmt.Printf("Processed %d items successfully, %d failures\n", 
        results.Successful, results.Failed)
    for _, err := range results.Errors {
        fmt.Printf("Error: %v\n", err)
    }
}

// Helper types and functions for the examples
type User struct {
    Name string
    Age  int
}

type UserData struct {
    // User data fields
}

type UserNotFoundError struct {
    ID string
}

func (e UserNotFoundError) Error() string {
    return fmt.Sprintf("user %s not found", e.ID)
}

type ServiceUnavailableError struct {
    Reason string
}

func (e ServiceUnavailableError) Error() string {
    return fmt.Sprintf("service unavailable: %s", e.Reason)
}

// Mock functions for the examples
func isValidFormat(input string) bool {
    return len(input) > 5 // Simplified validation
}

func processData(data []byte) error {
    return nil // Simplified implementation
}

func process(item string) (string, error) {
    if item == "bad_item" {
        return "", errors.New("invalid item")
    }
    return "processed_" + item, nil
}

var database = struct {
    Query func(string, ...interface{}) (interface{}, error)
    ErrNoRows error
    ErrTimeout error
}{
    Query: func(query string, args ...interface{}) (interface{}, error) {
        return nil, nil
    },
    ErrNoRows: errors.New("no rows"),
    ErrTimeout: errors.New("timeout"),
}

func parseUserData(data interface{}) (*UserData, error) {
    return &UserData{}, nil
}
```

**Real-World Example:**
A file processing application with comprehensive error handling strategies:

```go
package fileprocessor

import (
    "bufio"
    "errors"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "strings"
    "time"
)

// Error definitions
var (
    ErrInvalidFilePath = errors.New("invalid file path")
    ErrEmptyFile       = errors.New("file is empty")
    ErrInvalidFormat   = errors.New("file format is invalid")
)

// FileError represents an error that occurred while processing a file
type FileError struct {
    Path    string
    Op      string
    Line    int
    Wrapped error
}

func (e *FileError) Error() string {
    if e.Line != 0 {
        return fmt.Sprintf("%s %s:%d: %v", e.Op, e.Path, e.Line, e.Wrapped)
    }
    return fmt.Sprintf("%s %s: %v", e.Op, e.Path, e.Wrapped)
}

func (e *FileError) Unwrap() error {
    return e.Wrapped
}

// ProcessingResult represents the result of processing multiple files
type ProcessingResult struct {
    SuccessCount int
    FailureCount int
    Processed    []string
    Errors       map[string]error
    TotalLines   int
    Duration     time.Duration
}

// Config specifies processing options
type Config struct {
    MaxErrors    int
    IgnoreEmpty  bool
    StopOnError  bool
    IncludeStats bool
}

// DefaultConfig returns a default configuration
func DefaultConfig() *Config {
    return &Config{
        MaxErrors:    0,  // No limit
        IgnoreEmpty:  false,
        StopOnError:  false,
        IncludeStats: true,
    }
}

// Processor handles file processing
type Processor struct {
    config *Config
}

// NewProcessor creates a new file processor
func NewProcessor(config *Config) *Processor {
    if config == nil {
        config = DefaultConfig()
    }
    return &Processor{config: config}
}

// ProcessFiles processes multiple files and returns aggregated results and errors
func (p *Processor) ProcessFiles(paths []string) (*ProcessingResult, error) {
    startTime := time.Now()
    
    result := &ProcessingResult{
        Processed: make([]string, 0, len(paths)),
        Errors:    make(map[string]error),
    }
    
    for _, path := range paths {
        stats, err := p.ProcessFile(path)
        
        if err != nil {
            result.FailureCount++
            result.Errors[path] = err
            
            if p.config.StopOnError {
                break
            }
            
            if p.config.MaxErrors > 0 && result.FailureCount >= p.config.MaxErrors {
                result.Errors["_limit_reached"] = fmt.Errorf(
                    "stopped processing after %d errors", p.config.MaxErrors)
                break
            }
        } else {
            result.SuccessCount++
            result.Processed = append(result.Processed, path)
            
            if stats != nil {
                result.TotalLines += stats.Lines
            }
        }
    }
    
    result.Duration = time.Since(startTime)
    
    // Generate summary error if any failures occurred
    if result.FailureCount > 0 {
        return result, fmt.Errorf("%d out of %d files failed to process",
            result.FailureCount, result.FailureCount+result.SuccessCount)
    }
    
    return result, nil
}

// FileStats contains statistics about processed files
type FileStats struct {
    Lines     int
    Words     int
    Bytes     int
    Processed time.Time
}

// ProcessFile processes a single file and returns statistics
func (p *Processor) ProcessFile(path string) (*FileStats, error) {
    // Validate the file path
    if path == "" {
        return nil, ErrInvalidFilePath
    }
    
    // Clean and validate path
    path = filepath.Clean(path)
    
    // Check if file exists and is accessible
    info, err := os.Stat(path)
    if err != nil {
        if os.IsNotExist(err) {
            return nil, &FileError{
                Path:    path,
                Op:      "stat",
                Wrapped: fmt.Errorf("file does not exist: %w", err),
            }
        }
        return nil, &FileError{
            Path:    path,
            Op:      "stat",
            Wrapped: fmt.Errorf("failed to access file: %w", err),
        }
    }
    
    // Check if it's a regular file
    if !info.Mode().IsRegular() {
        return nil, &FileError{
            Path:    path,
            Op:      "validate",
            Wrapped: fmt.Errorf("not a regular file"),
        }
    }
    
    // Check if file is empty
    if info.Size() == 0 {
        if p.config.IgnoreEmpty {
            // Not an error, but no stats to return
            return &FileStats{
                Lines:     0,
                Words:     0,
                Bytes:     0,
                Processed: time.Now(),
            }, nil
        }
        return nil, &FileError{
            Path:    path,
            Op:      "validate",
            Wrapped: ErrEmptyFile,
        }
    }
    
    // Open the file
    file, err := os.Open(path)
    if err != nil {
        return nil, &FileError{
            Path:    path,
            Op:      "open",
            Wrapped: fmt.Errorf("failed to open file: %w", err),
        }
    }
    defer file.Close()
    
    // Process the file
    stats, err := p.processFileContents(file, path)
    if err != nil {
        return nil, err // Error is already wrapped
    }
    
    return stats, nil
}

// processFileContents processes the content of an open file
func (p *Processor) processFileContents(r io.Reader, path string) (*FileStats, error) {
    stats := &FileStats{
        Processed: time.Now(),
    }
    
    scanner := bufio.NewScanner(r)
    lineNum := 0
    
    for scanner.Scan() {
        lineNum++
        line := scanner.Text()
        
        // Track statistics
        stats.Lines++
        stats.Words += len(strings.Fields(line))
        stats.Bytes += len(scanner.Bytes()) + 1 // +1 for newline
        
        // Validate line format if needed
        if err := validateLineFormat(line); err != nil {
            return nil, &FileError{
                Path:    path,
                Op:      "process",
                Line:    lineNum,
                Wrapped: fmt.Errorf("invalid line format: %w", err),
            }
        }
        
        // Process the line content
        if err := processLine(line, lineNum); err != nil {
            return nil, &FileError{
                Path:    path,
                Op:      "process",
                Line:    lineNum,
                Wrapped: err,
            }
        }
    }
    
    // Check for scanner errors
    if err := scanner.Err(); err != nil {
        return nil, &FileError{
            Path:    path,
            Op:      "scan",
            Wrapped: fmt.Errorf("error reading file: %w", err),
        }
    }
    
    return stats, nil
}

// validateLineFormat checks if a line has valid format
func validateLineFormat(line string) error {
    // This is a placeholder for real validation logic
    if strings.Contains(line, "ERROR:") {
        return ErrInvalidFormat
    }
    return nil
}

// processLine handles processing of a single line
func processLine(line string, lineNum int) error {
    // This is a placeholder for real processing logic
    if strings.Contains(line, "FAIL") {
        return fmt.Errorf("line contains failure marker")
    }
    return nil
}

// Example usage
func Example() {
    // Create processor with custom config
    config := &Config{
        MaxErrors:    5,
        IgnoreEmpty:  true,
        StopOnError:  false,
        IncludeStats: true,
    }
    
    processor := NewProcessor(config)
    
    // Process multiple files
    files := []string{
        "data1.txt",
        "data2.txt",
        "empty.txt",
        "nonexistent.txt",
    }
    
    result, err := processor.ProcessFiles(files)
    
    // Handle overall result
    if err != nil {
        fmt.Println("Processing encountered errors:", err)
    }
    
    // Print statistics
    fmt.Printf("Files processed successfully: %d\n", result.SuccessCount)
    fmt.Printf("Files failed: %d\n", result.FailureCount)
    fmt.Printf("Total lines processed: %d\n", result.TotalLines)
    fmt.Printf("Time taken: %v\n", result.Duration)
    
    // Print specific errors
    if len(result.Errors) > 0 {
        fmt.Println("Errors by file:")
        for file, fileErr := range result.Errors {
            fmt.Printf("  %s: %v\n", file, fileErr)
            
            // Check for specific error types
            var fileError *FileError
            if errors.As(fileErr, &fileError) {
                if errors.Is(fileError.Wrapped, ErrEmptyFile) {
                    fmt.Println("    Note: Empty files can be handled by setting IgnoreEmpty to true")
                }
            }
        }
    }
}
```

**Common Pitfalls:**
- Ignoring errors or using `_ = function()` when errors should be handled
- Handling errors inconsistently across a codebase
- Not adding enough context to errors for effective debugging
- Over-complicating error handling with excessive nesting or too many error types
- Using panics for normal error conditions instead of returning errors
- Not documenting what errors a function might return
- Misusing `fmt.Errorf` with `%v` instead of `%w` when error wrapping is intended
- Not checking for specific error conditions that could be handled specially
- Exposing internal implementation details through errors at API boundaries
- Inconsistent logging of errors, making troubleshooting difficult

**Confusion Questions:**

1. **Q: When should I handle errors locally versus returning them to the caller?**

   A: This is a crucial design decision that affects your code's structure and how errors propagate through your application. Here's a guide for making this choice:

   **Handle errors locally when:**

   1. **The error can be fully resolved at the current level**:
      ```go
      func readConfigWithFallback(path string) (Config, error) {
          config, err := readConfig(path)
          if err != nil {
              // Handle locally by falling back to defaults
              log.Printf("Failed to read config from %s, using defaults: %v", path, err)
              return DefaultConfig(), nil
          }
          return config, nil
      }
      ```

   2. **The caller doesn't need to know about the specific error**:
      ```go
      func getUserName(id string) (string, error) {
          user, err := db.GetUser(id)
          if err != nil {
              if errors.Is(err, sql.ErrNoRows) {
                  // Translate to a more appropriate error for this abstraction
                  return "", errors.New("user not found")
              }
              return "", fmt.Errorf("database error: %w", err)
          }
          return user.Name, nil
      }
      ```

   3. **The error is expected as part of normal operation**:
      ```go
      func readLine(scanner *bufio.Scanner) (string, bool) {
          if scanner.Scan() {
              return scanner.Text(), true
          }
          // End of file is expected, handle it here
          if err := scanner.Err(); err != nil {
              log.Printf("Scanner error: %v", err)
          }
          return "", false
      }
      ```

   4. **The function is performing validation**:
      ```go
      func isValidEmail(email string) bool {
          if email == "" {
              return false  // Handle invalid input locally
          }
          return emailRegex.MatchString(email)
      }
      ```

   5. **For retry logic**:
      ```go
      func fetchWithRetries(url string, maxRetries int) ([]byte, error) {
          var lastErr error
          
          for i := 0; i < maxRetries; i++ {
              resp, err := http.Get(url)
              if err == nil {
                  defer resp.Body.Close()
                  return io.ReadAll(resp.Body)
              }
              
              // Handle error locally with retries
              lastErr = err
              time.Sleep(time.Second * time.Duration(i+1))
          }
          
          // Only return error after all retries failed
          return nil, fmt.Errorf("failed after %d retries: %w", maxRetries, lastErr)
      }
      ```

   **Return errors to the caller when:**

   1. **The caller needs to make decisions based on the error**:
      ```go
      func getUser(id string) (*User, error) {
          // Return the error so caller can decide what to do
          return db.GetUser(id) 
      }
      
      // Caller handles different error cases
      user, err := getUser("123")
      if err != nil {
          if errors.Is(err, sql.ErrNoRows) {
              // Show "user not found" message
          } else {
              // Show general error message
          }
          return
      }
      ```

   2. **You don't have enough context to handle the error properly**:
      ```go
      func processFile(path string) error {
          data, err := os.ReadFile(path)
          if err != nil {
              // Can't make a good decision here without knowing the caller's intent
              return fmt.Errorf("failed to read file %s: %w", path, err)
          }
          // Process data...
      }
      ```

   3. **For API boundaries or public functions**:
      ```go
      // Public API should return errors for callers to handle
      func (api *API) GetUserProfile(id string) (*Profile, error) {
          user, err := api.userService.GetUser(id)
          if err != nil {
              return nil, fmt.Errorf("user profile error: %w", err)
          }
          return mapUserToProfile(user), nil
      }
      ```

   4. **When the error indicates a significant problem**:
      ```go
      func processPayment(payment *Payment) error {
          err := validatePayment(payment)
          if err != nil {
              // Payment validation failures should be returned
              return fmt.Errorf("payment validation failed: %w", err)
          }
          
          err = chargeCustomer(payment)
          if err != nil {
              // Charging failures are significant and should be returned
              return fmt.Errorf("payment processing failed: %w", err)
          }
          
          return nil
      }
      ```

   5. **When you're unsure how to handle it**:
      ```go
      // If in doubt, return the error with context
      func processUserData(data []byte) error {
          user, err := parseUser(data)
          if err != nil {
              // Not sure how to handle this specific error, so return it
              return fmt.Errorf("user data parsing failed: %w", err)
          }
          // Continue processing...
      }
      ```

   **Guidelines for making the decision:**

   1. **Consider the function's contract**: What is this function promising to do?
   2. **Think about error handling capabilities**: Can you meaningfully handle the error here?
   3. **Follow the "separation of concerns" principle**: Error handling should happen at the right level of abstraction
   4. **Consider logging**: Log errors where they occur even if you're returning them
   5. **Be consistent**: Follow the same pattern within a package or module

   **For internal implementation details**:
   - Handle errors locally when possible
   - Translate low-level errors to your domain when appropriate

   **For public APIs**:
   - Return well-defined errors
   - Document possible error types
   - Consider stability of error types if they're part of your API

   The key is balancing prompt error handling against passing errors to code that has more context. There's no one-size-fits-all answer, but with these guidelines, you can make appropriate decisions for your specific situation.

2. **Q: Is it better to use many specific error types or fewer general ones?**

   A: This is a design decision that involves trade-offs between specificity and simplicity. Let's examine both approaches and when each is appropriate.

   **Using many specific error types:**

   ```go
   // Many specific error types
   type NotFoundError struct {
       Resource string
       ID       string
   }

   func (e NotFoundError) Error() string {
       return fmt.Sprintf("%s with ID %s not found", e.Resource, e.ID)
   }

   type ValidationError struct {
       Field string
       Issue string
   }

   func (e ValidationError) Error() string {
       return fmt.Sprintf("validation error on field %s: %s", e.Field, e.Issue)
   }

   type PermissionError struct {
       User     string
       Resource string
       Action   string
   }

   func (e PermissionError) Error() string {
       return fmt.Sprintf("user %s cannot %s on %s", e.User, e.Action, e.Resource)
   }
   ```

   **Advantages:**
   - Very precise error handling possible
   - Rich information specific to each error type
   - Clear documentation of possible errors
   - Type assertions can reliably extract structured data

   **Disadvantages:**
   - More types to maintain
   - Might lead to excessive type checking
   - Can make API evolution more difficult
   - Might expose implementation details

   **Using fewer general error types:**

   ```go
   // Fewer general error types with error codes
   type AppError struct {
       Code    string
       Message string
       Details map[string]interface{}
   }

   func (e AppError) Error() string {
       return fmt.Sprintf("[%s] %s", e.Code, e.Message)
   }

   // Usage
   func GetUser(id string) (*User, error) {
       if id == "" {
           return nil, AppError{
               Code:    "INVALID_INPUT",
               Message: "User ID cannot be empty",
           }
       }
       
       user, err := db.FindUser(id)
       if err != nil {
           if errors.Is(err, sql.ErrNoRows) {
               return nil, AppError{
                   Code:    "NOT_FOUND",
                   Message: "User not found",
                   Details: map[string]interface{}{"id": id},
               }
           }
           return nil, AppError{
               Code:    "DATABASE_ERROR",
               Message: "Error retrieving user",
               Details: map[string]interface{}{"cause": err.Error()},
           }
       }
       
       return user, nil
   }
   ```

   **Advantages:**
   - Simpler to maintain, fewer types
   - More flexible, can add new error conditions without new types
   - Consistent error structure
   - Easy to serialize/deserialize (e.g., for API responses)

   **Disadvantages:**
   - Less type-safety
   - More string comparisons or field checks
   - May require more documentation
   - Field names might not be as descriptive as type names

   **When to use many specific types:**

   1. **For libraries and packages** others will consume
   2. **When errors contain different data structures** specific to each error case
   3. **When errors need custom behavior** through methods
   4. **For complex error hierarchies** where errors have relationships
   5. **When fine-grained error handling** is critical

   ```go
   func handleRequest() {
       user, err := getUser(userID)
       if err != nil {
           var notFoundErr NotFoundError
           if errors.As(err, &notFoundErr) {
               http.Error(w, "User not found", http.StatusNotFound)
               return
           }
           
           var permErr PermissionError
           if errors.As(err, &permErr) {
               http.Error(w, "Unauthorized", http.StatusForbidden)
               return
           }
           
           // General error case
           http.Error(w, "Internal error", http.StatusInternalServerError)
           return
       }
       // Process user...
   }
   ```

   **When to use fewer general types:**

   1. **For applications** where error handling is more standardized
   2. **When error patterns are consistent** across different operations
   3. **For APIs** that need consistent error formats
   4. **When error handling code changes frequently**
   5. **When errors need to be serialized** (e.g., for API responses)

   ```go
   func handleRequest() {
       user, err := getUser(userID)
       if err != nil {
           var appErr AppError
           if errors.As(err, &appErr) {
               switch appErr.Code {
               case "NOT_FOUND":
                   http.Error(w, appErr.Message, http.StatusNotFound)
               case "PERMISSION_DENIED":
                   http.Error(w, appErr.Message, http.StatusForbidden)
               default:
                   http.Error(w, "Internal error", http.StatusInternalServerError)
               }
               return
           }
           http.Error(w, "Internal error", http.StatusInternalServerError)
           return
       }
       // Process user...
   }
   ```

   **A balanced approach:**

   In many cases, a hybrid approach works well:

   ```go
   // Base error type
   type AppError struct {
       Kind    ErrorKind
       Message string
       Err     error
   }

   func (e *AppError) Error() string {
       if e.Err != nil {
           return fmt.Sprintf("%s: %s: %v", e.Kind, e.Message, e.Err)
       }
       return fmt.Sprintf("%s: %s", e.Kind, e.Message)
   }

   func (e *AppError) Unwrap() error {
       return e.Err
   }

   // Error kinds for classification
   type ErrorKind int

   const (
       ErrUnknown ErrorKind = iota
       ErrNotFound
       ErrPermission
       ErrValidation
       ErrDatabase
   )

   func (k ErrorKind) String() string {
       switch k {
       case ErrNotFound:
           return "not found"
       case ErrPermission:
           return "permission denied"
       case ErrValidation:
           return "validation error"
       case ErrDatabase:
           return "database error"
       default:
           return "unknown error"
       }
   }
   ```

   **Final recommendation:**

   1. Start with a small set of error types and expand as needed
   2. Group related errors under common types
   3. Use error wrapping to preserve context and details
   4. Consider your API consumers and their error handling needs
   5. Be consistent in your approach across your codebase
   6. Document your error types and their meanings

   Remember that the goal is to make error handling easy and effective for both the consumers of your code and for yourself when debugging. Choose the approach that best supports your specific application's needs.

3. **Q: Should I log errors where they occur or only at the top level?**

   A: This is a common design question without a one-size-fits-all answer. Both approaches have their place, and often a hybrid strategy works best. Here's a comprehensive guide:

   **Logging at the top level only:**

   ```go
   // Lower-level function - no logging
   func fetchUserData(id string) (*UserData, error) {
       resp, err := http.Get(apiURL + "/users/" + id)
       if err != nil {
           return nil, fmt.Errorf("HTTP request failed: %w", err)
       }
       defer resp.Body.Close()
       
       // Process response...
   }

   // Mid-level function - no logging, just adds context
   func getUser(id string) (*User, error) {
       data, err := fetchUserData(id)
       if err != nil {
           return nil, fmt.Errorf("getting user %s: %w", id, err)
       }
       
       // Convert data to user...
   }

   // Top-level function - logs the error
   func HandleUserRequest(w http.ResponseWriter, r *http.Request) {
       userID := r.URL.Query().Get("id")
       
       user, err := getUser(userID)
       if err != nil {
           log.Printf("User request failed: %v", err)
           http.Error(w, "Failed to get user", http.StatusInternalServerError)
           return
       }
       
       // Return user data...
   }
   ```

   **Advantages:**
   - Avoids duplicate log entries
   - Provides complete context at the point of handling
   - Cleaner logs with fewer entries
   - Easier to correlate errors with specific operations

   **Disadvantages:**
   - Less detail about exactly where the error occurred
   - More difficult to add context-specific logging
   - May lose details if errors are wrapped multiple times
   - Harder to trace execution path

   **Logging at each level:**

   ```go
   // Lower-level function - logs with context
   func fetchUserData(id string) (*UserData, error) {
       resp, err := http.Get(apiURL + "/users/" + id)
       if err != nil {
           log.Printf("API request failed for user %s: %v", id, err)
           return nil, fmt.Errorf("HTTP request failed: %w", err)
       }
       defer resp.Body.Close()
       
       // Process response...
   }

   // Mid-level function - logs with context
   func getUser(id string) (*User, error) {
       log.Printf("Fetching user data for ID %s", id)
       data, err := fetchUserData(id)
       if err != nil {
           log.Printf("Failed to get data for user %s: %v", id, err)
           return nil, fmt.Errorf("getting user %s: %w", id, err)
       }
       
       // Convert data to user...
   }

   // Top-level function - logs with context
   func HandleUserRequest(w http.ResponseWriter, r *http.Request) {
       userID := r.URL.Query().Get("id")
       log.Printf("Handling user request for ID %s", userID)
       
       user, err := getUser(userID)
       if err != nil {
           log.Printf("User request handler error: %v", err)
           http.Error(w, "Failed to get user", http.StatusInternalServerError)
           return
       }
       
       // Return user data...
   }
   ```

   **Advantages:**
   - More detailed logs with precise error locations
   - Better understanding of the execution flow
   - Context-specific information at each level
   - Easier to debug complex issues

   **Disadvantages:**
   - Duplicate log entries for the same error
   - Noisier logs that can be harder to follow
   - Potential performance impact from excessive logging
   - Can clutter code with logging statements

   **A better hybrid approach:**

   ```go
   // Log at the source with context, return error for handling
   func fetchUserData(id string) (*UserData, error) {
       resp, err := http.Get(apiURL + "/users/" + id)
       if err != nil {
           // Log with local context but don't log the full error chain yet
           log.Printf("API request failed for user %s", id)
           return nil, fmt.Errorf("HTTP request failed: %w", err)
       }
       defer resp.Body.Close()
       
       // Process response...
   }

   // Add context but don't log (pass through)
   func getUser(id string) (*User, error) {
       data, err := fetchUserData(id)
       if err != nil {
           // Just add context, don't log here
           return nil, fmt.Errorf("getting user %s: %w", id, err)
       }
       
       // Convert data to user...
   }

   // Log complete error chain with full context
   func HandleUserRequest(w http.ResponseWriter, r *http.Request) {
       userID := r.URL.Query().Get("id")
       
       user, err := getUser(userID)
       if err != nil {
           // Comprehensive logging with full error chain
           log.Printf("User request failed for ID %s: %v", userID, err)
           http.Error(w, "Failed to get user", http.StatusInternalServerError)
           return
       }
       
       // Return user data...
   }
   ```

   **Using structured logging for the best results:**

   ```go
   // Using structured logging (with a hypothetical logger)
   func fetchUserData(ctx context.Context, id string) (*UserData, error) {
       logger := log.FromContext(ctx).With("user_id", id, "operation", "fetchUserData")
       
       resp, err := http.Get(apiURL + "/users/" + id)
       if err != nil {
           // Log at ERROR level with operation-specific context
           logger.Error("API request failed", "error", err)
           return nil, fmt.Errorf("HTTP request failed: %w", err)
       }
       defer resp.Body.Close()
       
       // Log success at DEBUG level
       logger.Debug("API request successful")
       
       // Process response...
   }

   func HandleUserRequest(w http.ResponseWriter, r *http.Request) {
       ctx := r.Context()
       userID := r.URL.Query().Get("id")
       
       // Create a request-scoped logger with request ID
       requestID := uuid.New().String()
       logger := log.New().With(
           "request_id", requestID,
           "path", r.URL.Path,
           "user_id", userID,
       )
       ctx = log.NewContext(ctx, logger)
       
       // Add request ID to response headers
       w.Header().Set("X-Request-ID", requestID)
       
       user, err := getUser(ctx, userID)
       if err != nil {
           // Log the complete error with full context
           logger.Error("Request failed", "error", err)
           http.Error(w, "Failed to get user", http.StatusInternalServerError)
           return
       }
       
       // Return user data...
   }
   ```

   **Best practices for error logging:**

   1. **Log with correlation IDs**: Include request IDs to trace errors across service calls

   2. **Use appropriate log levels**:
      - DEBUG: Detailed information for debugging
      - INFO: Normal application behavior
      - WARN: Unexpected but handled situations
      - ERROR: Errors that need attention
      - FATAL: Critical errors causing shutdown

   3. **Include contextual information**:
      ```go
      logger.Error("Database query failed",
          "user_id", user.ID,
          "query", queryName,
          "error", err,
          "latency_ms", latency.Milliseconds())
      ```

   4. **Be consistent about where you log errors**:
      - Log at the point of handling for complete context
      - Log brief operational failures where they occur
      - Don't log the same error at multiple levels with the same detail

   5. **Consider different strategies for different parts of your application**:
      - Library code: Minimal logging, return errors
      - Application code: Log at boundaries and key decision points
      - Request handlers: Log full errors with request context

   6. **Log error resolution too**:
      ```go
      if err := operation(); err != nil {
          if errors.Is(err, ErrTemporaryFailure) {
              logger.Warn("Operation failed temporarily, retrying", "error", err)
              // Retry logic...
              logger.Info("Retry successful")
              return nil
          }
          logger.Error("Operation failed permanently", "error", err)
          return err
      }
      ```

   **Final recommendation:**

   A hybrid approach generally works best:
   
   1. **At the source of the error**: Log brief operational context without the full error details
   
   2. **At intermediate layers**: Add context to errors but minimal logging
   
   3. **At the handling point**: Log comprehensive details with the full error chain
   
   4. **Use structured logging** to make correlation easier
   
   5. **Adjust based on environment**:
      - Development: More verbose, with locations and stack traces
      - Production: More concise, focusing on actionable information

   This balanced approach gives you the benefits of detailed logs when needed while keeping log noise to a manageable level.

### 4. Custom Error Types

**Concise Explanation:**
Custom error types in Go are user-defined types that implement the `error` interface. By creating your own error types, you can include specific data relevant to the error, implement custom behavior through methods, and enable type-based error handling. This approach allows for more structured error information and better error handling decisions compared to simple string errors.

**Where to Use:**
- When errors need to carry additional data or context
- To create distinct error categories that can be identified through type assertions
- When different error conditions require different handling strategies
- To implement domain-specific error behaviors through methods
- In API design to provide consistent error reporting
- When building a package or library for others to use
- When standard error types don't sufficiently express your error conditions

**Code Snippet:**
```go
package main

import (
    "errors"
    "fmt"
    "net/http"
    "time"
)

// Basic custom error type with fields
type ValidationError struct {
    Field   string
    Message string
}

// Implement the error interface
func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}

// Custom error type with multiple fields
type APIError struct {
    StatusCode int
    Message    string
    Timestamp  time.Time
    RequestID  string
}

func (e APIError) Error() string {
    return fmt.Sprintf("[%d %s] %s (request: %s, time: %s)",
        e.StatusCode,
        http.StatusText(e.StatusCode),
        e.Message,
        e.RequestID,
        e.Timestamp.Format(time.RFC3339),
    )
}

// Custom error type with wrapped error
type QueryError struct {
    Query string
    Err   error
}

func (e QueryError) Error() string {
    return fmt.Sprintf("query error: %s: %v", e.Query, e.Err)
}

// Implement Unwrap for error wrapping (Go 1.13+)
func (e QueryError) Unwrap() error {
    return e.Err
}

// Custom error type with additional behavior
type RetryableError struct {
    Err           error
    RetryAfter    time.Duration
    AttemptsLeft  int
}

func (e RetryableError) Error() string {
    return fmt.Sprintf("%v (retry after %v, attempts left: %d)",
        e.Err, e.RetryAfter, e.AttemptsLeft)
}

func (e RetryableError) Unwrap() error {
    return e.Err
}

// Helper method to check if retry is possible
func (e RetryableError) CanRetry() bool {
    return e.AttemptsLeft > 0
}

// Helper method to get the next retry error
func (e RetryableError) NextRetry() RetryableError {
    return RetryableError{
        Err:          e.Err,
        RetryAfter:   e.RetryAfter,
        AttemptsLeft: e.AttemptsLeft - 1,
    }
}

// Error types with sentinel values
type NotFoundError struct {
    Resource string
    ID       string
}

func (e NotFoundError) Error() string {
    return fmt.Sprintf("%s with ID %s not found", e.Resource, e.ID)
}

// Example of an error type hierarchy
type Error struct {
    Code    string
    Message string
    Op      string    // Operation that failed
    Err     error     // Underlying error
}

func (e *Error) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %s: %v", e.Op, e.Message, e.Err)
    }
    return fmt.Sprintf("%s: %s", e.Op, e.Message)
}

func (e *Error) Unwrap() error {
    return e.Err
}

// Usage examples
func main() {
    // Basic validation error
    email := "invalid-email"
    if !isValidEmail(email) {
        err := ValidationError{
            Field:   "email",
            Message: "invalid email format",
        }
        fmt.Println(err)  // validation failed on email: invalid email format
    }
    
    // API error example
    apiErr := APIError{
        StatusCode: 404,
        Message:    "User not found",
        Timestamp:  time.Now(),
        RequestID:  "req-123456",
    }
    fmt.Println(apiErr)
    
    // Wrapped error example
    originalErr := errors.New("connection refused")
    queryErr := QueryError{
        Query: "SELECT * FROM users",
        Err:   originalErr,
    }
    fmt.Println(queryErr)
    fmt.Println("Unwrapped:", errors.Unwrap(queryErr))
    
    // Error with behavior
    err := fetchWithRetry("https://api.example.com")
    if err != nil {
        var retryErr RetryableError
        if errors.As(err, &retryErr) {
            if retryErr.CanRetry() {
                fmt.Printf("Can retry after %v\n", retryErr.RetryAfter)
                nextErr := retryErr.NextRetry()
                fmt.Printf("Next retry has %d attempts left\n", nextErr.AttemptsLeft)
            }
        }
    }
    
    // Error type hierarchy example
    result, err := findUser("invalid-id")
    if err != nil {
        var notFoundErr NotFoundError
        if errors.As(err, &notFoundErr) {
            fmt.Printf("Not found error: %s with ID %s\n", 
                notFoundErr.Resource, notFoundErr.ID)
        }
        
        // Check error code with type assertion
        var appErr *Error
        if errors.As(err, &appErr) && appErr.Code == "INVALID_INPUT" {
            fmt.Println("Invalid input provided")
        }
    }
}

// Mock functions for examples
func isValidEmail(email string) bool {
    return len(email) > 0 && email != "invalid-email"
}

func fetchWithRetry(url string) error {
    return RetryableError{
        Err:          errors.New("temporary network failure"),
        RetryAfter:   5 * time.Second,
        AttemptsLeft: 3,
    }
}

func findUser(id string) (interface{}, error) {
    if id == "invalid-id" {
        return nil, &Error{
            Code:    "INVALID_INPUT",
            Message: "Invalid user ID format",
            Op:      "findUser",
        }
    }
    return nil, NotFoundError{Resource: "User", ID: id}
}
```

**Real-World Example:**
A user management service with comprehensive custom error types:

```go
package user

import (
    "database/sql"
    "errors"
    "fmt"
    "net/http"
    "time"
)

// Error codes
const (
    ErrInvalidInput  = "INVALID_INPUT"
    ErrNotFound      = "NOT_FOUND"
    ErrAlreadyExists = "ALREADY_EXISTS"
    ErrDatabase      = "DATABASE_ERROR"
    ErrPermission    = "PERMISSION_DENIED"
    ErrInternal      = "INTERNAL_ERROR"
    ErrTimeout       = "TIMEOUT"
)

// UserError is the base error type for user-related operations
type UserError struct {
    Code      string
    Message   string
    Operation string
    Param     string
    Err       error
}

func (e *UserError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("[%s] %s: %v (op=%s, param=%s)",
            e.Code, e.Message, e.Err, e.Operation, e.Param)
    }
    return fmt.Sprintf("[%s] %s (op=%s, param=%s)",
        e.Code, e.Message, e.Operation, e.Param)
}

func (e *UserError) Unwrap() error {
    return e.Err
}

// HTTPResponse converts the error to an HTTP response
func (e *UserError) HTTPResponse() (int, map[string]interface{}) {
    status := http.StatusInternalServerError
    
    // Map error codes to HTTP status codes
    switch e.Code {
    case ErrInvalidInput:
        status = http.StatusBadRequest
    case ErrNotFound:
        status = http.StatusNotFound
    case ErrAlreadyExists:
        status = http.StatusConflict
    case ErrPermission:
        status = http.StatusForbidden
    case ErrTimeout:
        status = http.StatusServiceUnavailable
    }
    
    return status, map[string]interface{}{
        "error": map[string]interface{}{
            "code":      e.Code,
            "message":   e.Message,
            "operation": e.Operation,
            "param":     e.Param,
        },
    }
}

// Specific error types

// ValidationError represents input validation failures
type ValidationError struct {
    Field   string
    Value   interface{}
    Message string
    Err     error
}

func (e *ValidationError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("validation error on field %q: %s (value=%v): %v",
            e.Field, e.Message, e.Value, e.Err)
    }
    return fmt.Sprintf("validation error on field %q: %s (value=%v)",
        e.Field, e.Message, e.Value)
}

func (e *ValidationError) Unwrap() error {
    return e.Err
}

// UserNotFoundError occurs when a user isn't found
type UserNotFoundError struct {
    UserID string
}

func (e UserNotFoundError) Error() string {
    return fmt.Sprintf("user with ID %q not found", e.UserID)
}

// Helper functions to create errors

// NewValidationError creates a validation error wrapped in a UserError
func NewValidationError(field string, value interface{}, message string, err error) error {
    validationErr := &ValidationError{
        Field:   field,
        Value:   value,
        Message: message,
        Err:     err,
    }
    
    return &UserError{
        Code:      ErrInvalidInput,
        Message:   fmt.Sprintf("Invalid %s", field),
        Operation: "validate",
        Param:     field,
        Err:       validationErr,
    }
}

// NewNotFoundError creates a not-found error
func NewNotFoundError(userID string) error {
    return &UserError{
        Code:      ErrNotFound,
        Message:   "User not found",
        Operation: "find",
        Param:     userID,
        Err:       UserNotFoundError{UserID: userID},
    }
}

// NewDatabaseError creates a database-related error
func NewDatabaseError(operation string, err error) error {
    return &UserError{
        Code:      ErrDatabase,
        Message:   "Database operation failed",
        Operation: operation,
        Err:       err,
    }
}

// User represents a user in the system
type User struct {
    ID        string
    Email     string
    Name      string
    CreatedAt time.Time
    UpdatedAt time.Time
}

// UserService provides user-related operations
type UserService struct {
    DB *sql.DB
}

// GetUser retrieves a user by ID
func (s *UserService) GetUser(id string) (*User, error) {
    if id == "" {
        return nil, NewValidationError("id", id, "cannot be empty", nil)
    }
    
    user, err := s.findUserByID(id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, NewNotFoundError(id)
        }
        return nil, NewDatabaseError("GetUser", err)
    }
    
    return user, nil
}

// CreateUser creates a new user
func (s *UserService) CreateUser(email, name string) (*User, error) {
    // Validate email
    if email == "" {
        return nil, NewValidationError("email", email, "cannot be empty", nil)
    }
    if !isValidEmail(email) {
        return nil, NewValidationError("email", email, "invalid format", nil)
    }
    
    // Validate name
    if name == "" {
        return nil, NewValidationError("name", name, "cannot be empty", nil)
    }
    if len(name) < 2 {
        return nil, NewValidationError("name", name, "too short (min 2 chars)", nil)
    }
    
    // Check if email already exists
    exists, err := s.emailExists(email)
    if err != nil {
        return nil, NewDatabaseError("CheckEmail", err)
    }
    if exists {
        return nil, &UserError{
            Code:      ErrAlreadyExists,
            Message:   "Email already in use",
            Operation: "CreateUser",
            Param:     email,
        }
    }
    
    // Create user
    user := &User{
        ID:        generateID(),
        Email:     email,
        Name:      name,
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }
    
    if err := s.saveUser(user); err != nil {
        return nil, NewDatabaseError("SaveUser", err)
    }
    
    return user, nil
}

// UpdateUser updates user details
func (s *UserService) UpdateUser(id string, name string) (*User, error) {
    // Get existing user
    user, err := s.GetUser(id)
    if err != nil {
        return nil, err // Already wrapped appropriately
    }
    
    // Validate name
    if name == "" {
        return nil, NewValidationError("name", name, "cannot be empty", nil)
    }
    if len(name) < 2 {
        return nil, NewValidationError("name", name, "too short (min 2 chars)", nil)
    }
    
    // Update user
    user.Name = name
    user.UpdatedAt = time.Now()
    
    if err := s.saveUser(user); err != nil {
        return nil, NewDatabaseError("UpdateUser", err)
    }
    
    return user, nil
}

// DeleteUser removes a user
func (s *UserService) DeleteUser(id string) error {
    if id == "" {
        return NewValidationError("id", id, "cannot be empty", nil)
    }
    
    // Check if user exists
    if _, err := s.GetUser(id); err != nil {
        return err // Already wrapped appropriately
    }
    
    // Delete user
    if err := s.deleteUserByID(id); err != nil {
        return NewDatabaseError("DeleteUser", err)
    }
    
    return nil
}

// Example HTTP handler to demonstrate error handling
func (s *UserService) HandleGetUser(w http.ResponseWriter, r *http.Request) {
    userID := r.URL.Query().Get("id")
    
    user, err := s.GetUser(userID)
    if err != nil {
        handleError(w, err)
        return
    }
    
    // Return user as JSON
    respondJSON(w, http.StatusOK, user)
}

// Helper to handle errors in HTTP handlers
func handleError(w http.ResponseWriter, err error) {
    var userErr *UserError
    if errors.As(err, &userErr) {
        status, response := userErr.HTTPResponse()
        respondJSON(w, status, response)
        return
    }
    
    // Fallback for unexpected errors
    respondJSON(w, http.StatusInternalServerError, map[string]interface{}{
        "error": map[string]string{
            "code":    "INTERNAL_ERROR",
            "message": "An unexpected error occurred",
        },
    })
}

// Mock helper functions
func (s *UserService) findUserByID(id string) (*User, error) {
    // Mock implementation
    if id == "missing" {
        return nil, sql.ErrNoRows
    }
    return &User{ID: id, Name: "Test User", Email: "test@example.com"}, nil
}

func (s *UserService) emailExists(email string) (bool, error) {
    // Mock implementation
    return email == "existing@example.com", nil
}

func (s *UserService) saveUser(user *User) error {
    // Mock implementation
    return nil
}

func (s *UserService) deleteUserByID(id string) error {
    // Mock implementation
    return nil
}

func isValidEmail(email string) bool {
    // Simplified validation
    return len(email) > 5 && email != "invalid@example"
}

func generateID() string {
    return fmt.Sprintf("user_%d", time.Now().UnixNano())
}

func respondJSON(w http.ResponseWriter, status int, data interface{}) {
    // Mock implementation
    w.WriteHeader(status)
    fmt.Fprintf(w, "%+v\n", data)
}

// Usage example
func Example() {
    service := UserService{DB: nil} // In real code, initialize with a real DB
    
    // Example 1: Get a nonexistent user
    user, err := service.GetUser("missing")
    if err != nil {
        fmt.Printf("Error getting user: %v\n", err)
        
        var userErr *UserError
        if errors.As(err, &userErr) {
            fmt.Printf("Error code: %s\n", userErr.Code)
            
            var notFoundErr UserNotFoundError
            if errors.As(err, &notFoundErr) {
                fmt.Printf("User with ID %s was not found\n", notFoundErr.UserID)
            }
        }
    }
    
    // Example 2: Create a user with invalid email
    user, err = service.CreateUser("invalid@example", "Jane")
    if err != nil {
        fmt.Printf("Error creating user: %v\n", err)
        
        var validationErr *ValidationError
        if errors.As(err, &validationErr) {
            fmt.Printf("Validation failed for field %q: %s\n", 
                validationErr.Field, validationErr.Message)
        }
    }
    
    // Example 3: Create a user with existing email
    user, err = service.CreateUser("existing@example.com", "Jane")
    if err != nil {
        fmt.Printf("Error creating user: %v\n", err)
        
        var userErr *UserError
        if errors.As(err, &userErr) && userErr.Code == ErrAlreadyExists {
            fmt.Printf("User with email %s already exists\n", userErr.Param)
        }
    }
}
```

**Common Pitfalls:**
- Creating overly complex error hierarchies
- Not implementing the `Unwrap()` method for errors that wrap other errors
- Using pointer receivers for some error methods but value receivers for others
- Not checking for specific error types correctly (using `errors.As()` vs type assertion)
- Exposing internal implementation details in error types
- Not documenting the error types a function can return
- Creating inconsistent error types across a package or codebase
- Not providing enough context in error messages
- Forgetting to handle all possible error types in error handling code

**Confusion Questions:**

1. **Q: How do I choose between using type assertions and using errors.As() or errors.Is() when handling custom error types?**

   A: Choosing between type assertions and the error handling functions introduced in Go 1.13 (`errors.As()` and `errors.Is()`) is important for effective error handling. Here's a guide to help you decide:

   **Type assertion (`err.(Type)`):**

   ```go
   err := doSomething()
   if validationErr, ok := err.(*ValidationError); ok {
       fmt.Printf("Validation error on field: %s\n", validationErr.Field)
   }
   ```

   **When to use type assertion:**
   - When working with Go versions prior to 1.13
   - When you are absolutely certain the error is not wrapped
   - When dealing with errors at their source (before wrapping)
   - When the direct concrete type is what matters, not wrapped errors

   **Disadvantages of type assertion:**
   - Doesn't work with wrapped errors
   - Requires checking the second return value (`ok`)
   - More verbose for complex error hierarchies
   - Breaks when error types change

   **errors.As() (Go 1.13+):**

   ```go
   err := doSomething()
   var validationErr *ValidationError
   if errors.As(err, &validationErr) {
       fmt.Printf("Validation error on field: %s\n", validationErr.Field)
   }
   ```

   **When to use errors.As():**
   - When you need to extract data or call methods on a specific error type
   - When errors might be wrapped at any point in the call chain
   - When you care about the concrete type somewhere in the error chain
   - When you need to work with the error object itself

   **errors.Is() (Go 1.13+):**

   ```go
   err := doSomething()
   if errors.Is(err, ErrNotFound) {
       fmt.Println("Resource not found")
   }
   ```

   **When to use errors.Is():**
   - When checking against sentinel or predefined error variables
   - When you only need to check equality, not extract data
   - When comparing against known error constants
   - When you just need to know if a specific error is in the chain

   **Decision matrix:**

   | Method | Use When | Example |
   |--------|----------|---------|
   | Type assertion | Direct error type matters, no wrapping | `if err, ok := err.(*MyError); ok { ... }` |
   | errors.As() | Need data from specific error type, might be wrapped | `var myErr *MyError; if errors.As(err, &myErr) { use myErr... }` |
   | errors.Is() | Checking equality with sentinel errors, might be wrapped | `if errors.Is(err, io.EOF) { ... }` |

   **Examples comparing the approaches:**

   ```go
   // Example 1: Working with sentinel errors
   
   // Define a sentinel error
   var ErrNotFound = errors.New("resource not found")
   
   // Using direct comparison (FRAGILE, breaks with wrapping)
   if err == ErrNotFound {
       // Handle not found case
   }
   
   // Better: Using errors.Is() (works with wrapping)
   if errors.Is(err, ErrNotFound) {
       // Handle not found case
   }
   ```

   ```go
   // Example 2: Working with error types containing data
   
   // Define custom error type
   type ValidationError struct {
       Field string
       Message string
   }
   
   func (e ValidationError) Error() string {
       return fmt.Sprintf("%s: %s", e.Field, e.Message)
   }
   
   // Using type assertion (FRAGILE, breaks with wrapping)
   if validErr, ok := err.(ValidationError); ok {
       fmt.Printf("Field %s has error: %s\n", validErr.Field, validErr.Message)
   }
   
   // Better: Using errors.As() (works with wrapping)
   var validErr ValidationError
   if errors.As(err, &validErr) {
       fmt.Printf("Field %s has error: %s\n", validErr.Field, validErr.Message)
   }
   ```

   ```go
   // Example 3: Complex error chains
   
   // In a deep call stack with error wrapping
   func deepFunction() error {
       err := deeperFunction()
       if err != nil {
           return fmt.Errorf("deep function failed: %w", err)
       }
       return nil
   }
   
   func deeperFunction() error {
       err := deepestFunction()
       if err != nil {
           return fmt.Errorf("deeper function failed: %w", err)
       }
       return nil
   }
   
   func deepestFunction() error {
       return &ValidationError{Field: "username", Message: "too short"}
   }
   
   // At the top level:
   err := deepFunction()
   
   // This fails - type assertion doesn't unwrap errors
   if validErr, ok := err.(*ValidationError); ok {
       // Never reached!
   }
   
   // This works - errors.As unwraps the chain
   var validErr *ValidationError
   if errors.As(err, &validErr) {
       fmt.Printf("Validation error: %s - %s\n", validErr.Field, validErr.Message)
   }
   ```

   **Implementing custom behavior for Is and As**:

   For more advanced cases, you can implement `Is()` and `As()` methods on your error types:

   ```go
   type ResourceError struct {
       ResourceType string
       ResourceID   string
       Action       string
       Err          error
   }

   func (e ResourceError) Error() string {
       return fmt.Sprintf("%s failed on %s '%s': %v", 
           e.Action, e.ResourceType, e.ResourceID, e.Err)
   }

   // Unwrap returns the wrapped error
   func (e ResourceError) Unwrap() error {
       return e.Err
   }
   
   // Is reports whether this error is equivalent to the target error
   func (e ResourceError) Is(target error) bool {
       // Check if targets match by type and resource type
       t, ok := target.(ResourceError)
       if !ok {
           return false
       }
       return e.ResourceType == t.ResourceType && 
              (t.ResourceID == "" || e.ResourceID == t.ResourceID)
   }
   
   // As supports type assertions on this error
   func (e ResourceError) As(target interface{}) bool {
       // Handle specific conversions
       switch v := target.(type) {
       case *ResourceError:
           *v = e
           return true
       default:
           return false
       }
   }
   ```

   **Best Practices:**

   1. **Prefer `errors.Is()` and `errors.As()`** over type assertions in most cases
   2. **Use `errors.Is()` for sentinel error checking** (equality)
   3. **Use `errors.As()` when you need to extract data** from error types
   4. **Only use type assertions when** you're absolutely certain about the direct type
   5. **Remember that `errors.As()` expects a pointer to a pointer** for custom error types
   6. **Implement custom `Is()` and `As()` methods** for complex error matching logic
   7. **Always implement `Unwrap()`** if your error wraps another error

   By following these guidelines, your error handling will be more robust and work better with error wrapping, a key feature of Go's error model.

2. **Q: Should my custom error types be structs, pointers, or interfaces?**

   A: The choice between defining custom error types as structs, pointers to structs, or interfaces involves important trade-offs. Here's a comprehensive guide to help you decide:

   **Struct Values:**

   ```go
   type ValidationError struct {
       Field  string
       Reason string
   }

   func (e ValidationError) Error() string {
       return fmt.Sprintf("%s: %s", e.Field, e.Reason)
   }

   // Usage
   return ValidationError{Field: "email", Reason: "invalid format"}
   ```

   **Advantages:**
   - Immutable by default (callers can't modify error fields)
   - No nil checks needed - a zero value is still a valid error
   - Simple and straightforward
   - Performs well for small error types
   - Works well with errors.As() and errors.Is()

   **Disadvantages:**
   - Can't add methods that modify the error
   - Less efficient for large error structures (copied by value)
   - Can't have nil value semantics

   **Pointers to Structs:**

   ```go
   type ValidationError struct {
       Field  string
       Reason string
   }

   func (e *ValidationError) Error() string {
       return fmt.Sprintf("%s: %s", e.Field, e.Reason)
   }

   // Usage
   return &ValidationError{Field: "email", Reason: "invalid format"}
   ```

   **Advantages:**
   - Methods can modify the error (if needed)
   - More efficient for large error structures
   - Can be nil (representing "no error")
   - Compatible with errors.As() and errors.Is()

   **Disadvantages:**
   - Requires nil checks if used directly
   - Allows mutation, which may be undesirable for errors
   - Slightly more complex memory management
   - Creates heap allocations

   **Interface Types:**

   ```go
   type ValidationError interface {
       error
       Field() string
       Reason() string
   }

   // Concrete implementation
   type validationError struct {
       field  string
       reason string
   }

   func (e validationError) Error() string {
       return fmt.Sprintf("%s: %s", e.field, e.reason)
   }

   func (e validationError) Field() string {
       return e.field
   }

   func (e validationError) Reason() string {
       return e.reason
   }

   // Constructor
   func NewValidationError(field, reason string) ValidationError {
       return validationError{field: field, reason: reason}
   }
   ```

   **Advantages:**
   - Provides a clear API for error users
   - Hides implementation details
   - Allows multiple implementations
   - Provides clear abstraction boundaries

   **Disadvantages:**
   - More complex to implement
   - Extra indirection (potential performance impact)
   - Can be harder to work with errors.As() and errors.Is()
   - Requires careful implementation of Is/As methods for proper error wrapping

   **Guidelines for choosing:**

   **1. Use struct values when:**
   - The error type is small (few fields)
   - Immutability is important
   - You want the simplest approach
   - Error equality checks (via errors.Is) are important

   ```go
   // Good for value errors
   type NotFoundError struct {
       Resource string
       ID       string
   }

   func (e NotFoundError) Error() string {
       return fmt.Sprintf("%s with ID %s not found", e.Resource, e.ID)
   }

   // Usage
   if errors.Is(err, NotFoundError{Resource: "User", ID: ""}) {
       // Handle any user not found error
   }
   ```

   **2. Use pointers to structs when:**
   - The error contains many fields or large data
   - Methods need to modify the error
   - You need nil-ability semantics
   - The error encapsulates other errors (implements Unwrap)

   ```go
   type RequestError struct {
       StatusCode int
       URL        string
       Method     string
       Err        error
   }

   func (e *RequestError) Error() string {
       return fmt.Sprintf("%s %s failed with status %d: %v", 
           e.Method, e.URL, e.StatusCode, e.Err)
   }

   func (e *RequestError) Unwrap() error {
       return e.Err
   }
   ```

   **3. Use interfaces when:**
   - You want to hide implementation details
   - Multiple implementations are needed
   - You're creating a public API or library
   - You need behavioral polymorphism

   ```go
   // Interface defining error behavior
   type RetryableError interface {
       error
       RetryAfter() time.Duration
       Attempts() int
   }

   // Implementation
   type httpRetryError struct {
       statusCode  int
       retryAfter  time.Duration
       attempts    int
       err         error
   }

   func (e httpRetryError) Error() string {
       return fmt.Sprintf("HTTP error %d: %v (retry after %v)", 
           e.statusCode, e.err, e.retryAfter)
   }

   func (e httpRetryError) RetryAfter() time.Duration {
       return e.retryAfter
   }

   func (e httpRetryError) Attempts() int {
       return e.attempts
   }

   func (e httpRetryError) Unwrap() error {
       return e.err
   }
   ```

   **Special considerations for error wrapping:**

   If your error type wraps another error, you should:

   ```go
   // For pointer receivers:
   func (e *MyError) Unwrap() error {
       return e.Err
   }

   // For value receivers:
   func (e MyError) Unwrap() error {
       return e.Err
   }
   ```

   **Best practices for error type design:**

   1. **Be consistent** within a package or module
   2. **Use the simplest approach** that meets your needs
   3. **Document your error types** and how to check for them
   4. **Consider error handling convenience** for users of your code

   ```go
   // Bad: Mixed approach, inconsistent
   func process() error {
       // Sometimes returns struct value
       return ValidationError{Field: "name", Reason: "too short"}
       // Sometimes returns pointer
       return &DatabaseError{Query: "SELECT...", Err: err}
   }

   // Good: Consistent approach
   func process() error {
       // Always returns error pointers
       return &ValidationError{Field: "name", Reason: "too short"}
       return &DatabaseError{Query: "SELECT...", Err: err}
   }
   ```

   5. **Consider the error checking syntax** from the caller's perspective

   ```go
   // With struct values:
   var notFoundErr NotFoundError
   if errors.As(err, &notFoundErr) { ... }

   // With pointers:
   var notFoundErr *NotFoundError
   if errors.As(err, &notFoundErr) { ... }

   // With interfaces:
   var retryErr RetryableError
   if errors.As(err, &retryErr) { ... }
   ```

   **For most real-world code, pointer receivers are the most versatile choice** because they:
   - Support error wrapping seamlessly
   - Work well with `errors.As()` and `errors.Is()`
   - Are efficient for all error sizes
   - Match the patterns used in the standard library

   However, value receivers can be appropriate for small, immutable error types, while interface-based approaches are valuable for library APIs with complex error handling needs.

3. **Q: How can I add behavior to my error types without adding complexity?**

   A: Adding behavior to error types can make error handling more powerful and expressive without adding unnecessary complexity. Here are effective patterns for designing behavior-rich error types:

   **1. Method-based behavior**

   The simplest approach is to add methods to your error types:

   ```go
   type NetworkError struct {
       URL        string
       StatusCode int
       RetryInfo  time.Duration
       Err        error
   }

   func (e *NetworkError) Error() string {
       return fmt.Sprintf("network request to %s failed with status %d: %v",
           e.URL, e.StatusCode, e.Err)
   }

   func (e *NetworkError) Unwrap() error {
       return e.Err
   }

   // Behavior methods:
   
   func (e *NetworkError) CanRetry() bool {
       // 5xx errors are server errors and can be retried
       return e.StatusCode >= 500 && e.StatusCode < 600
   }

   func (e *NetworkError) RetryAfter() time.Duration {
       if e.RetryInfo > 0 {
           return e.RetryInfo
       }
       // Default exponential backoff based on status code
       if e.StatusCode == 503 {
           return 5 * time.Second
       }
       return 1 * time.Second
   }

   func (e *NetworkError) IsClientError() bool {
       return e.StatusCode >= 400 && e.StatusCode < 500
   }

   func (e *NetworkError) IsServerError() bool {
       return e.StatusCode >= 500 && e.StatusCode < 600
   }
   ```

   **Usage:**

   ```go
   resp, err := client.Get(url)
   if err != nil {
       var netErr *NetworkError
       if errors.As(err, &netErr) {
           if netErr.CanRetry() {
               // Wait and retry
               time.Sleep(netErr.RetryAfter())
               return retry(url)
           }
           if netErr.IsClientError() {
               return fmt.Errorf("please check request parameters: %w", err)
           }
       }
       return err
   }
   ```

   **2. Behavioral interfaces**

   Define interfaces for specific error behaviors:

   ```go
   // Define behavior interfaces
   type Retryable interface {
       CanRetry() bool
       RetryAfter() time.Duration
   }

   type Temporary interface {
       Temporary() bool
   }

   type HTTPStatus interface {
       StatusCode() int
   }

   // Implement them in your error types
   type APIError struct {
       Code       int
       Message    string
       RetryDelay time.Duration
       Transient  bool
   }

   func (e *APIError) Error() string {
       return fmt.Sprintf("API error %d: %s", e.Code, e.Message)
   }

   // Implement the behavior interfaces
   func (e *APIError) CanRetry() bool {
       return e.Transient || (e.Code >= 500 && e.Code < 600)
   }

   func (e *APIError) RetryAfter() time.Duration {
       if e.RetryDelay > 0 {
           return e.RetryDelay
       }
       return 1 * time.Second
   }

   func (e *APIError) Temporary() bool {
       return e.Transient
   }

   func (e *APIError) StatusCode() int {
       return e.Code
   }
   ```

   **Usage:**

   ```go
   if err != nil {
       // Check for retry behavior
       if retry, ok := err.(Retryable); ok && retry.CanRetry() {
           time.Sleep(retry.RetryAfter())
           return doOperation()
       }

       // Check for temporary condition
       if temp, ok := err.(Temporary); ok && temp.Temporary() {
           log.Println("Temporary error, might resolve itself")
       }
       
       // Check if it provides HTTP status
       if status, ok := err.(HTTPStatus); ok {
           log.Printf("HTTP status: %d", status.StatusCode())
       }
       
       return err
   }
   ```

   **3. Function helpers for common behaviors**

   Create helper functions that work with error types:

   ```go
   // Helper functions that extract behavior from errors
   
   // IsRetryable checks if an error can be retried
   func IsRetryable(err error) bool {
       var retry interface {
           CanRetry() bool
       }
       return errors.As(err, &retry) && retry.CanRetry()
   }

   // RetryAfter returns the recommended retry delay
   func RetryAfter(err error) time.Duration {
       var retry interface {
           RetryAfter() time.Duration
       }
       if errors.As(err, &retry) {
           return retry.RetryAfter()
       }
       return 1 * time.Second // Default
   }

   // IsTemporary checks if an error is temporary
   func IsTemporary(err error) bool {
       var temp interface {
           Temporary() bool
       }
       return errors.As(err, &temp) && temp.Temporary()
   }

   // GetHTTPStatusCode extracts status code if available
   func GetHTTPStatusCode(err error) (int, bool) {
       var status interface {
           StatusCode() int
       }
       if errors.As(err, &status) {
           return status.StatusCode(), true
       }
       return 0, false
   }
   ```

   **Usage:**

   ```go
   if err != nil {
       if IsRetryable(err) {
           time.Sleep(RetryAfter(err))
           return doOperation()
       }
       
       if IsTemporary(err) {
           log.Println("Temporary error, might resolve itself")
       }
       
       if status, ok := GetHTTPStatusCode(err); ok {
           log.Printf("HTTP status: %d", status)
       }
       
       return err
   }
   ```

   **4. Behavior through composition**

   ```go
   // Base errors with behavior
   type RetryableError struct {
       Err      error
       Delay    time.Duration
       Attempts int
   }

   func (e *RetryableError) Error() string {
       return fmt.Sprintf("%v (retryable, delay=%v, attempts=%d)",
           e.Err, e.Delay, e.Attempts)
   }

   func (e *RetryableError) Unwrap() error {
       return e.Err
   }

   func (e *RetryableError) CanRetry() bool {
       return e.Attempts > 0
   }

   func (e *RetryableError) RetryAfter() time.Duration {
       return e.Delay
   }

   // Compose behaviors
   type APIError struct {
       StatusCode int
       Message    string
   }

   func (e *APIError) Error() string {
       return fmt.Sprintf("API error %d: %s", e.StatusCode, e.Message)
   }

   // Make an error retryable
   func MakeRetryable(err error, delay time.Duration, attempts int) error {
       return &RetryableError{
           Err:      err,
           Delay:    delay,
           Attempts: attempts,
       }
   }
   ```

   **Usage:**

   ```go
   resp, err := client.Get(url)
   if err != nil {
       if resp != nil && resp.StatusCode == 429 {
           // Too many requests, make it retryable
           retryAfter := 30 * time.Second
           if s := resp.Header.Get("Retry-After"); s != "" {
               if secs, err := strconv.Atoi(s); err == nil {
                   retryAfter = time.Duration(secs) * time.Second
               }
           }
           return MakeRetryable(err, retryAfter, 3)
       }
       return err
   }
   ```

   **5. Contextual error handlers**

   ```go
   // ErrorHandler processes errors with context
   type ErrorHandler struct {
       retryEnabled  bool
       maxRetries    int
       defaultDelay  time.Duration
       shouldLogFunc func(error) bool
   }

   // NewErrorHandler creates a configured error handler
   func NewErrorHandler(options ...ErrorHandlerOption) *ErrorHandler {
       h := &ErrorHandler{
           retryEnabled: true,
           maxRetries:   3,
           defaultDelay: 1 * time.Second,
           shouldLogFunc: func(error) bool {
               return true
           },
       }
       
       for _, option := range options {
           option(h)
       }
       
       return h
   }

   // ErrorHandlerOption configures the error handler
   type ErrorHandlerOption func(*ErrorHandler)

   // WithMaxRetries sets maximum retries
   func WithMaxRetries(max int) ErrorHandlerOption {
       return func(h *ErrorHandler) {
           h.maxRetries = max
       }
   }

   // WithDefaultDelay sets default retry delay
   func WithDefaultDelay(delay time.Duration) ErrorHandlerOption {
       return func(h *ErrorHandler) {
           h.defaultDelay = delay
       }
   }

   // Handle processes an error
   func (h *ErrorHandler) Handle(err error, operation string) error {
       if err == nil {
           return nil
       }
       
       // Log if needed
       if h.shouldLogFunc(err) {
           log.Printf("[%s] Error: %v", operation, err)
       }
       
       // Check if retryable
       if h.retryEnabled && IsRetryable(err) {
           retries := h.maxRetries
           
           // Get retry attempts if available
           var retryErr interface {
               Attempts() int
           }
           if errors.As(err, &retryErr) {
               retries = retryErr.Attempts()
           }
           
           if retries > 0 {
               delay := RetryAfter(err)
               if delay == 0 {
                   delay = h.defaultDelay
               }
               
               log.Printf("[%s] Retrying after %v (%d attempts left)",
                   operation, delay, retries)
                   
               time.Sleep(delay)
               return nil // Signal to retry
           }
       }
       
       // Pass through enhanced error
       return fmt.Errorf("%s failed: %w", operation, err)
   }
   ```

   **Usage:**

   ```go
   handler := NewErrorHandler(
       WithMaxRetries(5),
       WithDefaultDelay(2 * time.Second),
   )
   
   for {
       err := doOperation()
       err = handler.Handle(err, "MyOperation")
       if err != nil {
           return err // Terminal error
       }
       
       // nil error means retry or success
       if operationComplete() {
           break
       }
   }
   ```

   **Best practices for behavior-rich errors:**

   1. **Focus on behavior, not implementation**:
      - Design interfaces around what errors can do, not what they are
      - Use method names that clearly express the behavior

   2. **Keep behaviors consistent**:
      - Use the same method names for similar behaviors across error types
      - Ensure consistent return types and semantics

   3. **Document behaviors**:
      - Make it clear which methods are available
      - Document what each method returns and when

   4. **Avoid over-designing**:
      - Start simple and add behaviors as needed
      - Don't try to anticipate every possible error behavior
      - Prefer composition over complex hierarchies

   5. **Consider the caller's perspective**:
      - Make error checks intuitive and readable
      - Avoid requiring complex type assertions
      - Provide helper functions for common operations

   By following these patterns, you can create error types that provide rich behavior while remaining clean, maintainable, and easy to use.

### 5. Error Wrapping and Unwrapping

**Concise Explanation:**
Error wrapping is a technique introduced in Go 1.13 that allows you to add context to errors while preserving the original error for later inspection. This is done using the `%w` verb in `fmt.Errorf()`. The wrapped error can be unwrapped using `errors.Unwrap()`, and the original error can be checked using `errors.Is()` or `errors.As()`. This approach helps create detailed error chains while still allowing code to check for specific error types or values at any level of the chain.

**Where to Use:**
- When propagating errors up the call stack and want to add context
- When you need to preserve the original error type for error checking
- In middleware or layered architectures where error context is important
- When implementing error hierarchies with additional information at each level
- For creating rich error chains that can still be introspected
- When you need to handle specific error types or conditions regardless of wrapping

**Code Snippet:**
```go
package main

import (
    "database/sql"
    "errors"
    "fmt"
    "os"
)

// Basic error wrapping with fmt.Errorf and %w
func readConfig(path string) ([]byte, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("failed to read config file %s: %w", path, err)
    }
    return data, nil
}

// Error wrapping with custom error type
type QueryError struct {
    Query string
    Err   error
}

func (e *QueryError) Error() string {
    return fmt.Sprintf("query error: %s: %v", e.Query, e.Err)
}

// Implement Unwrap() to support Go's error wrapping
func (e *QueryError) Unwrap() error {
    return e.Err
}

// Function that uses custom error wrapping
func queryDatabase(query string) error {
    // Simulate a database error
    err := sql.ErrNoRows
    if err != nil {
        return &QueryError{
            Query: query,
            Err:   err,
        }
    }
    return nil
}

// Multi-level error wrapping
func getUserData(userID string) ([]byte, error) {
    query := fmt.Sprintf("SELECT data FROM users WHERE id = '%s'", userID)
    
    // Query the database
    err := queryDatabase(query)
    if err != nil {
        return nil, fmt.Errorf("getUserData failed for user %s: %w", userID, err)
    }
    
    // Rest of the implementation...
    return []byte("user data"), nil
}

func main() {
    // Example 1: Basic error wrapping
    data, err := readConfig("nonexistent.conf")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        
        // Check if it's a specific error type by unwrapping
        if errors.Is(err, os.ErrNotExist) {
            fmt.Println("The config file doesn't exist!")
        }
    }
    
    // Example 2: Custom error types and unwrapping
    _, err = getUserData("user123")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        
        // Check for specific error type using errors.As
        var queryErr *QueryError
        if errors.As(err, &queryErr) {
            fmt.Printf("Query that failed: %s\n", queryErr.Query)
        }
        
        // Check for root cause using errors.Is
        if errors.Is(err, sql.ErrNoRows) {
            fmt.Println("The user record was not found in the database.")
        }
    }
    
    // Example 3: Manually unwrapping errors
    if err != nil {
        unwrapped := errors.Unwrap(err)
        fmt.Printf("Unwrapped error: %v\n", unwrapped)
        
        // Unwrap again to get to the root cause
        unwrappedAgain := errors.Unwrap(unwrapped)
        fmt.Printf("Root cause: %v\n", unwrappedAgain)
    }
    
    // Example 4: Using errors.Join to combine multiple errors (Go 1.20+)
    err1 := fmt.Errorf("error 1")
    err2 := fmt.Errorf("error 2")
    combined := errors.Join(err1, err2)
    fmt.Printf("Combined errors: %v\n", combined)
}
```

**Real-World Example:**
Building a layered application with comprehensive error wrapping:

```go
package main

import (
    "database/sql"
    "errors"
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "time"
)

// Define sentinel errors
var (
    ErrNotFound      = errors.New("resource not found")
    ErrUnauthorized  = errors.New("unauthorized")
    ErrInvalidInput  = errors.New("invalid input")
    ErrInternal      = errors.New("internal error")
)

// =====================================
// Domain layer error types
// =====================================

// UserError represents errors in the user domain
type UserError struct {
    ID  string
    Err error
}

func (e *UserError) Error() string {
    return fmt.Sprintf("user error (id=%s): %v", e.ID, e.Err)
}

func (e *UserError) Unwrap() error {
    return e.Err
}

// OrderError represents errors in the order domain
type OrderError struct {
    OrderID string
    Op      string
    Err     error
}

func (e *OrderError) Error() string {
    return fmt.Sprintf("order %s error during %s: %v", e.OrderID, e.Op, e.Err)
}

func (e *OrderError) Unwrap() error {
    return e.Err
}

// ValidationError represents input validation failures
type ValidationError struct {
    Field string
    Value interface{}
    Msg   string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %s: %s (got %v)", e.Field, e.Msg, e.Value)
}

// =====================================
// Repository layer errors
// =====================================

// DBError represents database-related errors
type DBError struct {
    Query string
    Err   error
}

func (e *DBError) Error() string {
    return fmt.Sprintf("database error: %v", e.Err)
}

func (e *DBError) Unwrap() error {
    return e.Err
}

// =====================================
// Application services
// =====================================

// UserRepository handles user data access
type UserRepository struct {
    db *sql.DB
}

func (r *UserRepository) FindByID(id string) (map[string]interface{}, error) {
    // Simulate a database query
    if id == "" {
        return nil, &DBError{
            Query: "SELECT * FROM users WHERE id = ?",
            Err:   ErrInvalidInput,
        }
    }
    
    if id == "missing" {
        return nil, &DBError{
            Query: "SELECT * FROM users WHERE id = ?",
            Err:   sql.ErrNoRows,
        }
    }
    
    // Simulate found user
    return map[string]interface{}{
        "id":   id,
        "name": "Test User",
    }, nil
}

// OrderRepository handles order data access
type OrderRepository struct {
    db *sql.DB
}

func (r *OrderRepository) FindByID(id string) (map[string]interface{}, error) {
    // Simulate a database query
    if id == "" {
        return nil, &DBError{
            Query: "SELECT * FROM orders WHERE id = ?",
            Err:   ErrInvalidInput,
        }
    }
    
    if id == "missing" {
        return nil, &DBError{
            Query: "SELECT * FROM orders WHERE id = ?",
            Err:   sql.ErrNoRows,
        }
    }
    
    // Simulate found order
    return map[string]interface{}{
        "id":      id,
        "user_id": "user123",
        "amount":  99.99,
        "status":  "pending",
    }, nil
}

// =====================================
// Application service layer
// =====================================

// UserService provides user-related operations
type UserService struct {
    userRepo *UserRepository
}

func (s *UserService) GetUser(id string) (map[string]interface{}, error) {
    if id == "" {
        return nil, &ValidationError{
            Field: "id",
            Value: id,
            Msg:   "cannot be empty",
        }
    }
    
    user, err := s.userRepo.FindByID(id)
    if err != nil {
        // Wrap the repository error with user context
        return nil, &UserError{
            ID:  id,
            Err: fmt.Errorf("failed to find user: %w", err),
        }
    }
    
    return user, nil
}

// OrderService provides order-related operations
type OrderService struct {
    orderRepo *OrderRepository
    userService *UserService
}

func (s *OrderService) GetOrder(id string) (map[string]interface{}, error) {
    if id == "" {
        return nil, &ValidationError{
            Field: "order_id",
            Value: id,
            Msg:   "cannot be empty",
        }
    }
    
    order, err := s.orderRepo.FindByID(id)
    if err != nil {
        return nil, &OrderError{
            OrderID: id,
            Op:      "get",
            Err:     fmt.Errorf("failed to find order: %w", err),
        }
    }
    
    // Get associated user
    userID := order["user_id"].(string)
    user, err := s.userService.GetUser(userID)
    if err != nil {
        return nil, &OrderError{
            OrderID: id,
            Op:      "get user details",
            Err:     err,
        }
    }
    
    // Add user info to order
    order["user"] = user
    return order, nil
}

// =====================================
// API/HTTP layer
// =====================================

// ErrorResponse formats an error for API responses
type ErrorResponse struct {
    Status  int    `json:"status"`
    Code    string `json:"code"`
    Message string `json:"message"`
    Details string `json:"details,omitempty"`
}

// HTTP handler for getting a user
func handleGetUser(userService *UserService) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        userID := r.URL.Query().Get("id")
        
        user, err := userService.GetUser(userID)
        if err != nil {
            writeErrorResponse(w, err)
            return
        }
        
        // Write successful response
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{"status":"success","data":%v}`, user)
    }
}

// HTTP handler for getting an order
func handleGetOrder(orderService *OrderService) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        orderID := r.URL.Query().Get("id")
        
        order, err := orderService.GetOrder(orderID)
        if err != nil {
            writeErrorResponse(w, err)
            return
        }
        
        // Write successful response
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{"status":"success","data":%v}`, order)
    }
}

// Write an error response based on the error type
func writeErrorResponse(w http.ResponseWriter, err error) {
    // Log the full error details for debugging
    log.Printf("Error occurred: %+v", err)
    
    // Default response
    response := ErrorResponse{
        Status:  http.StatusInternalServerError,
        Code:    "INTERNAL_ERROR",
        Message: "An unexpected error occurred",
    }
    
    // Check for specific error types
    var validationErr *ValidationError
    if errors.As(err, &validationErr) {
        response.Status = http.StatusBadRequest
        response.Code = "VALIDATION_ERROR"
        response.Message = "Invalid input parameters"
        response.Details = validationErr.Error()
    } else if errors.Is(err, ErrNotFound) || errors.Is(err, sql.ErrNoRows) {
        response.Status = http.StatusNotFound
        response.Code = "NOT_FOUND"
        response.Message = "The requested resource was not found"
    } else if errors.Is(err, ErrUnauthorized) {
        response.Status = http.StatusUnauthorized
        response.Code = "UNAUTHORIZED"
        response.Message = "Authentication required"
    } else {
        // Include some error context in dev environments only
        if os.Getenv("ENV") == "development" {
            response.Details = err.Error()
        }
    }
    
    // Send the response
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(response.Status)
    fmt.Fprintf(w, `{"status":"error","error":{"code":"%s","message":"%s","details":"%s"}}`, 
                response.Code, response.Message, response.Details)
}

// =====================================
// Application setup and example
// =====================================

func main() {
    // Initialize repositories (with nil DB for this example)
    userRepo := &UserRepository{db: nil}
    orderRepo := &OrderRepository{db: nil}
    
    // Initialize services
    userService := &UserService{userRepo: userRepo}
    orderService := &OrderService{
        orderRepo:   orderRepo,
        userService: userService,
    }
    
    // Set up HTTP handlers
    http.HandleFunc("/api/user", handleGetUser(userService))
    http.HandleFunc("/api/order", handleGetOrder(orderService))
    
    // Example of error unwrapping
    fmt.Println("Starting error handling examples...")
    
    // Example 1: User not found
    _, err := userService.GetUser("missing")
    if err != nil {
        fmt.Printf("Error: %v\n\n", err)
        
        // Check if it's a not found error regardless of wrapping
        if errors.Is(err, sql.ErrNoRows) {
            fmt.Println("Root cause: User not found in database")
        }
        
        // Unwrap the full error chain
        fmt.Println("Error chain:")
        for e := err; e != nil; e = errors.Unwrap(e) {
            fmt.Printf("- %v\n", e)
        }
        fmt.Println()
    }
    
    // Example 2: Invalid input
    _, err = userService.GetUser("")
    if err != nil {
        fmt.Printf("Error: %v\n\n", err)
        
        // Extract validation details
        var validErr *ValidationError
        if errors.As(err, &validErr) {
            fmt.Printf("Validation error details: field=%s, value=%v, message=%s\n\n",
                validErr.Field, validErr.Value, validErr.Msg)
        }
    }
    
    // Example 3: Order with missing user
    _, err = orderService.GetOrder("order123")
    if err != nil {
        fmt.Printf("Error: %v\n\n", err)
        
        // Extract order error details
        var orderErr *OrderError
        if errors.As(err, &orderErr) {
            fmt.Printf("Order error details: id=%s, operation=%s\n",
                orderErr.OrderID, orderErr.Op)
        }
        
        // We can also check for database errors anywhere in the chain
        var dbErr *DBError
        if errors.As(err, &dbErr) {
            fmt.Printf("Database error details: query=%s\n\n", dbErr.Query)
        }
    }
}
```

**Common Pitfalls:**
- Using `%v` instead of `%w` in `fmt.Errorf()` when you intend to wrap the error
- Forgetting to implement the `Unwrap()` method in custom error types
- Wrapping errors with too much or too little context
- Creating overly deep error chains that are hard to understand
- Exposing sensitive information in error messages
- Using direct error comparisons (`==`) instead of `errors.Is()`
- Using type assertions instead of `errors.As()` for wrapped errors
- Losing the original error by creating a new one
- Not documenting which errors a function might return

**Confusion Questions:**

1. **Q: When should I wrap an error versus creating a new one?**

   A: This is one of the most common questions about error handling in Go. Let me break it down:

   **Wrap an error when:**

   1. **You want to add context while preserving the original error**:
      ```go
      file, err := os.Open("config.json")
      if err != nil {
          return fmt.Errorf("opening configuration file: %w", err)
      }
      ```
      This approach preserves the original error (like `os.ErrNotExist`) while adding context.

   2. **The error's original type or value is important for checks**:
      ```go
      err := db.QueryRow("SELECT id FROM users WHERE email = ?", email).Scan(&id)
      if err != nil {
          return fmt.Errorf("looking up user by email %q: %w", email, err)
      }
      ```
      This lets callers still check `errors.Is(err, sql.ErrNoRows)` higher in the call stack.

   3. **You're adding information along a call chain**:
      ```go
      func fetchUserPreferences(userID string) (Preferences, error) {
          user, err := getUser(userID)
          if err != nil {
              return nil, fmt.Errorf("fetching preferences for user %s: %w", userID, err)
          }
          // ...
      }
      ```
      This builds a descriptive error chain as the error propagates up.

   **Create a new error when:**

   1. **The original error is an implementation detail callers shouldn't depend on**:
      ```go
      func isUserAdmin(id string) (bool, error) {
          user, err := db.GetUser(id)
          if err != nil {
              if errors.Is(err, sql.ErrNoRows) {
                  return false, errors.New("user not found")
              }
              return false, errors.New("error checking user permissions")
          }
          return user.IsAdmin, nil
      }
      ```
      Here, we're hiding the database error details from the caller.

   2. **You're at a boundary between systems or abstraction layers**:
      ```go
      func (api *API) GetUserProfile(id string) (*UserProfile, error) {
          data, err := api.db.QueryUser(id)
          if err != nil {
              if errors.Is(err, database.ErrNoResult) {
                  return nil, ErrUserNotFound
              }
              return nil, errors.New("failed to retrieve user profile")
          }
          // ...
      }
      ```
      Public APIs often present their own error types rather than exposing internal ones.

   3. **You need to create a specific error type for structured handling**:
      ```go
      if len(password) < 8 {
          return &ValidationError{
              Field:   "password",
              Message: "password must be at least 8 characters",
          }
      }
      ```
      When you need specific fields or behaviors in your error.

   4. **For security reasons, to prevent leaking sensitive details**:
      ```go
      func authenticate(username, password string) error {
          hashedPassword, err := getStoredPassword(username)
          if err != nil {
              // Don't leak whether the user exists or not
              return errors.New("invalid username or password")
          }
          // ...
      }
      ```

   **Guidelines for deciding:**

   1. **Consider the consumer's needs**: Will they benefit from details about the underlying error?
      
   2. **Think about error stability**: If you wrap DB-specific errors, are you creating a dependency on your DB implementation?
      
   3. **Evaluate abstraction boundaries**: Should this layer expose details about the errors from lower layers?
      
   4. **Consider security implications**: Could error details leak sensitive information?

   5. **Think about testing**: How will tests interact with these errors?

   A common pattern is to wrap errors within a package or module (adding context at each level), but translate them at API boundaries to present a clean, stable interface to callers.

   **Practical example of a balanced approach**:

   ```go
   // In an internal repository layer:
   func (r *userRepo) FindByID(id string) (*User, error) {
       var user User
       err := r.db.Get(&user, "SELECT * FROM users WHERE id = ?", id)
       if err != nil {
           // Wrap with context while preserving the original error
           return nil, fmt.Errorf("finding user %s: %w", id, err)
       }
       return &user, nil
   }

   // In a service layer (internal to the application)
   func (s *userService) GetUserDetails(id string) (*UserDetails, error) {
       user, err := s.repo.FindByID(id)
       if err != nil {
           // Continue wrapping while staying within the application boundary
           return nil, fmt.Errorf("getting user details: %w", err)
       }
       return mapToDetails(user), nil
   }

   // In a public API handler (boundary layer)
   func (api *API) HandleGetUser(w http.ResponseWriter, r *http.Request) {
       id := r.PathValue("id")
       
       details, err := api.userService.GetUserDetails(id)
       if err != nil {
           // Translate to public error types at the boundary
           if errors.Is(err, sql.ErrNoRows) {
               api.WriteError(w, http.StatusNotFound, "user not found")
               return
           }
           // Log the detailed error but return a generic message
           log.Printf("Error getting user %s: %v", id, err)
           api.WriteError(w, http.StatusInternalServerError, "internal server error")
           return
       }
       
       api.WriteJSON(w, http.StatusOK, details)
   }
   ```

   This approach provides rich context for internal errors while presenting clean, stable errors at public boundaries.

2. **Q: What's the difference between errors.Is(), errors.As(), and errors.Unwrap()?**

   A: Go 1.13 introduced these three functions to improve error handling with wrapped errors. They serve different purposes and should be used in different scenarios:

   **errors.Unwrap(err error) error**

   This function extracts the wrapped error from an error that implements the `Unwrap() error` method.

   ```go
   func Unwrap(err error) error {
       u, ok := err.(interface {
           Unwrap() error
       })
       if !ok {
           return nil
       }
       return u.Unwrap()
   }
   ```

   **When to use**: 
   - When you need to manually extract the next error in an error chain
   - When implementing custom error handling that needs to walk the error chain
   - Less commonly used directly in application code

   **Example**:
   ```go
   err := fmt.Errorf("failed to process: %w", os.ErrNotExist)
   
   // Manually unwrap once
   unwrapped := errors.Unwrap(err) // Returns os.ErrNotExist
   
   // Manually walk the error chain
   fmt.Println("Error chain:")
   for e := err; e != nil; e = errors.Unwrap(e) {
       fmt.Printf("- %v\n", e)
   }
   ```

   **errors.Is(err, target error) bool**

   Checks if `err` or any error in its chain matches `target`. This is like an improved `==` that works with wrapped errors.

   **When to use**:
   - When checking for specific error values or sentinel errors
   - When you want to see if a specific error is anywhere in the error chain
   - When previously you would have used `err == SomeError`

   **Example**:
   ```go
   // Define a sentinel error
   var ErrNotFound = errors.New("not found")
   
   // Create a wrapped error
   err := fmt.Errorf("failed to get user: %w", 
         fmt.Errorf("database error: %w", ErrNotFound))
   
   // Check if ErrNotFound is in the chain
   if errors.Is(err, ErrNotFound) {
       fmt.Println("Resource not found!")
   }
   
   // This would fail because of wrapping
   if err == ErrNotFound {
       fmt.Println("This won't print")
   }
   ```

   **errors.As(err error, target interface{}) bool**

   Checks if `err` or any error in its chain matches the type of `target` and if so, sets `target` to that error value.

   **When to use**:
   - When you need to extract a specific error type from the error chain
   - When you want to access fields or methods of a specific error type
   - When previously you would have used type assertion `err.(SomeErrorType)`

   **Example**:
   ```go
   // Custom error type
   type ValidationError struct {
       Field string
       Err   error
   }
   
   func (e *ValidationError) Error() string {
       return fmt.Sprintf("validation failed for %s: %v", e.Field, e.Err)
   }
   
   func (e *ValidationError) Unwrap() error {
       return e.Err
   }
   
   // Create a wrapped error with our custom type
   valErr := &ValidationError{Field: "email", Err: errors.New("invalid format")}
   err := fmt.Errorf("user creation failed: %w", valErr)
   
   // Extract the validation error to access its fields
   var ve *ValidationError
   if errors.As(err, &ve) {
       fmt.Printf("Validation failed for field: %s\n", ve.Field)
   }
   
   // This would fail because of wrapping
   if ve2, ok := err.(*ValidationError); ok {
       fmt.Println("This won't print", ve2.Field)
   }
   ```

   **Comparison**:

   | Function | Purpose | Input | Return | Use When |
   |----------|---------|-------|--------|----------|
   | `errors.Unwrap()` | Extract nested error | `error` | `error` | Manually walking error chain |
   | `errors.Is()` | Check error identity | `error, error` | `bool` | Comparing against sentinel errors |
   | `errors.As()` | Check error type | `error, pointer-to-error-type` | `bool` | Extracting custom error types |

   **Implementation nuances**:

   1. **errors.Is() can be customized** by implementing an `Is(error) bool` method on your error type:
      ```go
      func (e *MyError) Is(target error) bool {
          t, ok := target.(*MyError)
          if !ok {
              return false
          }
          // Custom comparison logic
          return e.Code == t.Code
      }
      ```

   2. **errors.As() can be customized** by implementing an `As(interface{}) bool` method:
      ```go
      func (e *MyError) As(target interface{}) bool {
          switch t := target.(type) {
          case **APIError:
              *t = &APIError{Code: e.Code, Message: e.Error()}
              return true
          default:
              return false
          }
      }
      ```

   3. **Multiple wrapped errors** (Go 1.20+) use the first error that matches:
      ```go
      err1 := errors.New("error one")
      err2 := errors.New("error two")
      combined := errors.Join(err1, err2)
      
      errors.Is(combined, err1) // true
      errors.Is(combined, err2) // true
      ```

   **Best practices**:

   1. **Use errors.Is() for sentinel errors**: When checking against predefined error values
      ```go
      if errors.Is(err, io.EOF) { ... }
      ```

   2. **Use errors.As() for error types**: When you need to access fields or methods of an error
      ```go
      var netErr *net.OpError
      if errors.As(err, &netErr) {
          fmt.Println("Network operation:", netErr.Op)
      }
      ```

   3. **Use errors.Unwrap() sparingly**: Usually the other functions are more appropriate

   4. **Combine with wrapping**: These functions are designed to work with Go's error wrapping
      ```go
      return fmt.Errorf("operation failed: %w", err)
      ```

   5. **Document behavior**: If your errors implement custom `Is` or `As` methods, document their behavior

   By using these functions appropriately, you can create sophisticated error handling while preserving the ability to inspect and identify specific errors throughout the error chain.

3. **Q: How do I properly wrap errors in a deep call stack without losing context?**

   A: Managing errors in a deep call stack while preserving context requires a systematic approach to error wrapping. Here's a comprehensive guide:

   **1. Add relevant context at each level**

   As errors move up the call stack, wrap them with information specific to each level:

   ```go
   // Low level function
   func readConfigFile(path string) ([]byte, error) {
       data, err := os.ReadFile(path)
       if err != nil {
           return nil, fmt.Errorf("reading config file %q: %w", path, err)
       }
       return data, nil
   }

   // Mid level function
   func loadConfiguration(configPath string) (*Config, error) {
       data, err := readConfigFile(configPath)
       if err != nil {
           return nil, fmt.Errorf("loading configuration: %w", err)
       }
       
       config, err := parseConfig(data)
       if err != nil {
           return nil, fmt.Errorf("parsing configuration: %w", err)
       }
       
       return config, nil
   }

   // High level function
   func initializeApp() error {
       config, err := loadConfiguration(defaultConfigPath)
       if err != nil {
           return fmt.Errorf("initializing application: %w", err)
       }
       
       // Use the configuration...
       return nil
   }
   ```

   The resulting error message will read like a story:
   ```
   initializing application: loading configuration: reading config file "config.json": open config.json: no such file or directory
   ```

   **2. Be concise but informative**

   Include only essential context that helps diagnose the problem:

   ```go
   // Good: Includes relevant parameters
   return fmt.Errorf("connecting to database %q at %s:%d: %w", dbName, host, port, err)

   // Not as good: Too verbose
   return fmt.Errorf("error occurred while attempting to establish a connection to the database named %q located at host %s on port %d: %w", dbName, host, port, err)

   // Not as good: Too sparse
   return fmt.Errorf("database connection error: %w", err)
   ```

   **3. Use consistent formatting patterns**

   Adopt a consistent error message format throughout your codebase:

   ```go
   // Common pattern: "<action>: <error>"
   return fmt.Errorf("validating input: %w", err)
   return fmt.Errorf("connecting to service: %w", err)
   return fmt.Errorf("processing request: %w", err)
   ```

   **4. Add structured context with custom error types**

   For more complex cases, use custom error types to add structured context:

   ```go
   type DatabaseError struct {
       Operation string
       Table     string
       Query     string
       Err       error
   }

   func (e *DatabaseError) Error() string {
       return fmt.Sprintf("database error during %s on table %q: %v", 
           e.Operation, e.Table, e.Err)
   }

   func (e *DatabaseError) Unwrap() error {
       return e.Err
   }

   // Usage
   func queryUsers() ([]*User, error) {
       rows, err := db.Query("SELECT * FROM users WHERE active = true")
       if err != nil {
           return nil, &DatabaseError{
               Operation: "query",
               Table:     "users",
               Query:     "SELECT * FROM users WHERE active = true",
               Err:       err,
           }
       }
       // Process rows...
   }
   ```

   **5. Use errors.Join for aggregating multiple errors**

   When you need to preserve multiple errors from parallel operations:

   ```go
   func validateForm(form Form) error {
       var errs []error
       
       if err := validateName(form.Name); err != nil {
           errs = append(errs, fmt.Errorf("name validation: %w", err))
       }
       
       if err := validateEmail(form.Email); err != nil {
           errs = append(errs, fmt.Errorf("email validation: %w", err))
       }
       
       if err := validatePassword(form.Password); err != nil {
           errs = append(errs, fmt.Errorf("password validation: %w", err))
       }
       
       if len(errs) > 0 {
           return fmt.Errorf("form validation failed: %w", errors.Join(errs...))
       }
       
       return nil
   }
   ```

   **6. Consider the "context chain pattern"**

   Use a helper function to consistently add operation context:

   ```go
   // withOperation wraps an error with operation context
   func withOperation(err error, operation string) error {
       if err == nil {
           return nil
       }
       return fmt.Errorf("%s: %w", operation, err)
   }

   // Usage
   func processOrder(orderID string) error {
       order, err := findOrder(orderID)
       if err != nil {
           return withOperation(err, "finding order")
       }
       
       err = validateOrder(order)
       if err != nil {
           return withOperation(err, "validating order")
       }
       
       err = processPayment(order)
       if err != nil {
           return withOperation(err, "processing payment")
       }
       
       return nil
   }
   ```

   **7. Leverage operation and context in custom errors**

   Incorporate operation and context fields in structured errors:

   ```go
   type OpError struct {
       Op    string // Operation that failed
       Kind  string // Category of error
       Err   error  // Underlying error
       Info  string // Additional information
   }

   func (e *OpError) Error() string {
       if e.Err != nil {
           return fmt.Sprintf("%s: %s: %v", e.Op, e.Kind, e.Err)
       }
       return fmt.Sprintf("%s: %s", e.Op, e.Kind)
   }

   func (e *OpError) Unwrap() error {
       return e.Err
   }

   // Usage
   func getUser(id string) (*User, error) {
       if id == "" {
           return nil, &OpError{
               Op:   "getUser",
               Kind: "validation",
               Info: "empty id",
           }
       }
       
       user, err := db.QueryUser(id)
       if err != nil {
           return nil, &OpError{
               Op:   "getUser",
               Kind: "database",
               Err:  err,
           }
       }
       
       return user, nil
   }
   ```

   **8. Use stack trace capabilities in production**

   For production systems, consider adding stack traces to errors at appropriate points:

   ```go
   import "github.com/pkg/errors"

   // At the lowest level where an error originates
   if err != nil {
       return errors.Wrap(err, "reading configuration file")
   }

   // At higher levels, just add context without capturing stack again
   if err != nil {
       return fmt.Errorf("initializing module: %w", err)
   }
   ```

   **9. Balance detail and cleanliness at API boundaries**

   When errors cross system or package boundaries, consider translating them:

   ```go
   func (api *API) HandleGetUser(w http.ResponseWriter, r *http.Request) {
       userID := r.URL.Query().Get("id")
       
       user, err := api.service.GetUser(userID)
       if err != nil {
           // Log the detailed error for internal use
           log.Printf("Error retrieving user %q: %v", userID, err)
           
           // Return a clean error to the client
           if errors.Is(err, sql.ErrNoRows) || errors.Is(err, ErrUserNotFound) {
               http.Error(w, "User not found", http.StatusNotFound)
               return
           }
           
           http.Error(w, "Internal server error", http.StatusInternalServerError)
           return
       }
       
       // Return the user...
   }
   ```

   **10. Provide helpers for error inspection**

   Create helper functions that make it easy to check for specific error conditions:

   ```go
   // IsNotFound returns true if err indicates a not-found condition
   func IsNotFound(err error) bool {
       return errors.Is(err, sql.ErrNoRows) || 
              errors.Is(err, os.ErrNotExist) ||
              errors.Is(err, ErrNotFound)
   }

   // IsTemporary returns true if the error is temporary and a retry might succeed
   func IsTemporary(err error) bool {
       var netErr net.Error
       return errors.As(err, &netErr) && netErr.Temporary()
   }

   // Usage
   user, err := repo.FindUser(id)
   if err != nil {
       if IsNotFound(err) {
           return nil, ErrUserNotFound
       }
       if IsTemporary(err) {
           return retry(id)
       }
       return nil, fmt.Errorf("finding user: %w", err)
   }
   ```

   **Best Practices Summary**:

   1. **Be consistent** in your error wrapping approach
   2. **Add context at each level**, but don't repeat information
   3. **Include relevant parameters** that help diagnose the issue
   4. **Use structured errors** for complex error information
   5. **Consider the audience** for each error (users, developers, logs)
   6. **Balance detail and abstraction** at API boundaries
   7. **Document error handling expectations** for your functions
   8. **Log detailed errors** while returning appropriate errors to callers

   By following these guidelines, you can create a detailed error trail that preserves context through deep call stacks while keeping error handling clean and effective.

### 6. Sentinel Errors

**Concise Explanation:**
Sentinel errors are predefined error variables that represent specific error conditions in your code. They are created using `errors.New()` and typically exported from packages to allow callers to check for specific error conditions using `errors.Is()`. Using sentinel errors creates a cleaner API than string comparison and allows for error wrapping while maintaining the ability to identify specific errors.

**Where to Use:**
- For well-defined error conditions that callers might want to handle specifically
- When you want to provide standard errors that consumers of your API can check for
- In public API packages to document possible error return values
- For commonly occurring errors that merit custom handling
- When different types of errors require different handling logic
- As a simpler alternative to custom error types for basic error conditions

**Code Snippet:**
```go
package main

import (
    "errors"
    "fmt"
    "io"
    "os"
)

// Define sentinel errors for the main package
var (
    // ErrNotFound is returned when a requested resource doesn't exist
    ErrNotFound = errors.New("resource not found")
    
    // ErrInvalidInput is returned when the user provides invalid input
    ErrInvalidInput = errors.New("invalid input provided")
    
    // ErrUnauthorized is returned when an operation is not permitted
    ErrUnauthorized = errors.New("not authorized")
    
    // ErrTimeout is returned when an operation times out
    ErrTimeout = errors.New("operation timed out")
)

// Using sentinel errors in functions
func getUser(id string) (string, error) {
    if id == "" {
        return "", fmt.Errorf("getUser: %w", ErrInvalidInput)
    }
    
    // Simulate: user with ID "admin" exists
    if id != "admin" {
        return "", fmt.Errorf("user %s: %w", id, ErrNotFound)
    }
    
    return "Admin User", nil
}

// Using sentinel errors in methods
type Resource struct {
    name string
    data []byte
}

func (r *Resource) Read(user string) ([]byte, error) {
    // Check if resource exists
    if r == nil || len(r.data) == 0 {
        return nil, fmt.Errorf("reading %s: %w", r.name, ErrNotFound)
    }
    
    // Check permissions (only admin can read)
    if user != "admin" {
        return nil, fmt.Errorf("user %s reading %s: %w", 
                                user, r.name, ErrUnauthorized)
    }
    
    return r.data, nil
}

// Function that returns both its own and standard library errors
func readFile(path string, maxSize int) ([]byte, error) {
    // Check input
    if path == "" {
        return nil, fmt.Errorf("readFile: %w", ErrInvalidInput)
    }
    
    // Open the file (this may return various os errors)
    file, err := os.Open(path)
    if err != nil {
        return nil, fmt.Errorf("opening %s: %w", path, err)
    }
    defer file.Close()
    
    // Read with size limit
    data := make([]byte, maxSize)
    n, err := file.Read(data)
    if err != nil && err != io.EOF {
        return nil, fmt.Errorf("reading %s: %w", path, err)
    }
    
    return data[:n], nil
}

func main() {
    // Example 1: Checking for sentinel errors
    user, err := getUser("invalid-id")
    if err != nil {
        fmt.Printf("Error getting user: %v\n", err)
        
        // Check if it's a specific error
        if errors.Is(err, ErrNotFound) {
            fmt.Println("The user doesn't exist, please register first.")
        } else if errors.Is(err, ErrInvalidInput) {
            fmt.Println("Please provide a valid user ID.")
        } else {
            fmt.Println("An unexpected error occurred.")
        }
    }
    
    // Example 2: Checking for standard library sentinel errors
    data, err := readFile("nonexistent-file.txt", 1024)
    if err != nil {
        fmt.Printf("Error reading file: %v\n", err)
        
        // Check for specific os errors
        if errors.Is(err, os.ErrNotExist) {
            fmt.Println("File does not exist.")
        } else if errors.Is(err, os.ErrPermission) {
            fmt.Println("Permission denied.")
        } else if errors.Is(err, ErrInvalidInput) {
            fmt.Println("Invalid path provided.")
        }
    } else {
        fmt.Printf("Read %d bytes\n", len(data))
    }
    
    // Example 3: Sentinel errors in resource access
    resource := &Resource{
        name: "secret-document.txt",
        data: []byte("Classified information"),
    }
    
    content, err := resource.Read("guest")
    if err != nil {
        fmt.Printf("Error reading resource: %v\n", err)
        
        if errors.Is(err, ErrUnauthorized) {
            fmt.Println("Please log in as an administrator.")
        }
    } else {
        fmt.Printf("Content: %s\n", content)
    }
}
```

**Real-World Example:**
A file storage service with well-defined sentinel errors:

```go
package storage

import (
    "errors"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "strings"
    "time"
)

// Sentinel errors for the storage package
var (
    // ErrFileNotFound is returned when the requested file doesn't exist
    ErrFileNotFound = errors.New("file not found")
    
    // ErrFileExists is returned when trying to create a file that already exists
    ErrFileExists = errors.New("file already exists")
    
    // ErrInvalidPath is returned when a file path is invalid
    ErrInvalidPath = errors.New("invalid file path")
    
    // ErrFileTooLarge is returned when a file exceeds the maximum allowed size
    ErrFileTooLarge = errors.New("file exceeds maximum size")
    
    // ErrQuotaExceeded is returned when a user's storage quota is exceeded
    ErrQuotaExceeded = errors.New("storage quota exceeded")
    
    // ErrPermissionDenied is returned when a user doesn't have permission for an operation
    ErrPermissionDenied = errors.New("permission denied")
    
    // ErrCorruptFile is returned when a file appears to be corrupt
    ErrCorruptFile = errors.New("file is corrupt or damaged")
)

// FileStorage provides file storage operations
type FileStorage struct {
    RootDir    string
    MaxFileSize int64
    UserQuotas map[string]int64
}

// NewFileStorage creates a new file storage instance
func NewFileStorage(rootDir string) (*FileStorage, error) {
    if rootDir == "" {
        return nil, fmt.Errorf("storage: %w", ErrInvalidPath)
    }
    
    // Create root directory if it doesn't exist
    if err := os.MkdirAll(rootDir, 0755); err != nil {
        return nil, fmt.Errorf("initializing storage at %s: %w", rootDir, err)
    }
    
    return &FileStorage{
        RootDir:    rootDir,
        MaxFileSize: 10 * 1024 * 1024, // 10MB default
        UserQuotas: make(map[string]int64),
    }, nil
}

// validatePath checks if a path is valid and safe
func (fs *FileStorage) validatePath(path string) error {
    // Check for empty path
    if path == "" {
        return fmt.Errorf("empty path: %w", ErrInvalidPath)
    }
    
    // Check for suspicious path components
    if strings.Contains(path, "..") {
        return fmt.Errorf("path contains '..': %w", ErrInvalidPath)
    }
    
    // Check for absolute paths
    if filepath.IsAbs(path) {
        return fmt.Errorf("absolute paths not allowed: %w", ErrInvalidPath)
    }
    
    return nil
}

// getFullPath returns the full filesystem path
func (fs *FileStorage) getFullPath(userID, path string) (string, error) {
    if err := fs.validatePath(path); err != nil {
        return "", err
    }
    
    // Construct user's directory within root
    userDir := filepath.Join(fs.RootDir, userID)
    
    // Create user directory if it doesn't exist
    if err := os.MkdirAll(userDir, 0755); err != nil {
        return "", fmt.Errorf("creating user directory: %w", err)
    }
    
    // Return full path
    return filepath.Join(userDir, path), nil
}

// StoreFile saves a file to storage
func (fs *FileStorage) StoreFile(userID string, path string, data []byte) error {
    // Validate the size
    if int64(len(data)) > fs.MaxFileSize {
        return fmt.Errorf("file size %d bytes: %w", len(data), ErrFileTooLarge)
    }
    
    // Check user quota
    if quota, exists := fs.UserQuotas[userID]; exists {
        currentUsage, err := fs.GetUserUsage(userID)
        if err != nil {
            return fmt.Errorf("checking quota: %w", err)
        }
        
        if currentUsage+int64(len(data)) > quota {
            return fmt.Errorf("user %s quota %d bytes: %w", 
                            userID, quota, ErrQuotaExceeded)
        }
    }
    
    // Get the full path
    fullPath, err := fs.getFullPath(userID, path)
    if err != nil {
        return fmt.Errorf("storing file %s: %w", path, err)
    }
    
    // Check if file exists
    if _, err := os.Stat(fullPath); err == nil {
        return fmt.Errorf("path %s: %w", path, ErrFileExists)
    } else if !os.IsNotExist(err) {
        return fmt.Errorf("checking file %s: %w", path, err)
    }
    
    // Create the directory structure
    dir := filepath.Dir(fullPath)
    if err := os.MkdirAll(dir, 0755); err != nil {
        return fmt.Errorf("creating directories for %s: %w", path, err)
    }
    
    // Write the file
    if err := os.WriteFile(fullPath, data, 0644); err != nil {
        return fmt.Errorf("writing file %s: %w", path, err)
    }
    
    return nil
}

// GetFile retrieves a file from storage
func (fs *FileStorage) GetFile(userID string, path string) ([]byte, error) {
    // Get the full path
    fullPath, err := fs.getFullPath(userID, path)
    if err != nil {
        return nil, fmt.Errorf("getting file %s: %w", path, err)
    }
    
    // Read the file
    data, err := os.ReadFile(fullPath)
    if err != nil {
        if os.IsNotExist(err) {
            return nil, fmt.Errorf("path %s: %w", path, ErrFileNotFound)
        }
        if os.IsPermission(err) {
            return nil, fmt.Errorf("path %s: %w", path, ErrPermissionDenied)
        }
        return nil, fmt.Errorf("reading file %s: %w", path, err)
    }
    
    // Optional: validate file (e.g., check checksum)
    if isCorrupt(data) {
        return nil, fmt.Errorf("file %s: %w", path, ErrCorruptFile)
    }
    
    return data, nil
}

// DeleteFile removes a file from storage
func (fs *FileStorage) DeleteFile(userID string, path string) error {
    // Get the full path
    fullPath, err := fs.getFullPath(userID, path)
    if err != nil {
        return fmt.Errorf("deleting file %s: %w", path, err)
    }
    
    // Check if file exists
    if _, err := os.Stat(fullPath); err != nil {
        if os.IsNotExist(err) {
            return fmt.Errorf("path %s: %w", path, ErrFileNotFound)
        }
        return fmt.Errorf("checking file %s: %w", path, err)
    }
    
    // Delete the file
    if err := os.Remove(fullPath); err != nil {
        if os.IsPermission(err) {
            return fmt.Errorf("path %s: %w", path, ErrPermissionDenied)
        }
        return fmt.Errorf("removing file %s: %w", path, err)
    }
    
    return nil
}

// ListFiles lists all files in a directory
func (fs *FileStorage) ListFiles(userID string, directory string) ([]string, error) {
    // Handle empty directory
    if directory == "" {
        directory = "."
    }
    
    // Get the full directory path
    fullPath, err := fs.getFullPath(userID, directory)
    if err != nil {
        return nil, fmt.Errorf("listing directory %s: %w", directory, err)
    }
    
    // Check if directory exists
    info, err := os.Stat(fullPath)
    if err != nil {
        if os.IsNotExist(err) {
            return nil, fmt.Errorf("directory %s: %w", directory, ErrFileNotFound)
        }
        return nil, fmt.Errorf("checking directory %s: %w", directory, err)
    }
    
    // Make sure it's a directory
    if !info.IsDir() {
        return nil, fmt.Errorf("path %s is not a directory: %w", directory, ErrInvalidPath)
    }
    
    // Read directory contents
    entries, err := os.ReadDir(fullPath)
    if err != nil {
        return nil, fmt.Errorf("reading directory %s: %w", directory, err)
    }
    
    // Extract filenames
    files := make([]string, 0, len(entries))
    for _, entry := range entries {
        name := entry.Name()
        if entry.IsDir() {
            name += "/"
        }
        files = append(files, name)
    }
    
    return files, nil
}

// GetUserUsage calculates the total storage used by a user
func (fs *FileStorage) GetUserUsage(userID string) (int64, error) {
    // Get user directory
    userDir := filepath.Join(fs.RootDir, userID)
    
    // Check if directory exists
    if _, err := os.Stat(userDir); err != nil {
        if os.IsNotExist(err) {
            return 0, nil // No usage yet
        }
        return 0, fmt.Errorf("checking user directory: %w", err)
    }
    
    // Calculate size
    var totalSize int64
    err := filepath.Walk(userDir, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        if !info.IsDir() {
            totalSize += info.Size()
        }
        return nil
    })
    
    if err != nil {
        return 0, fmt.Errorf("calculating usage: %w", err)
    }
    
    return totalSize, nil
}

// SetUserQuota sets the maximum storage quota for a user
func (fs *FileStorage) SetUserQuota(userID string, quota int64) {
    fs.UserQuotas[userID] = quota
}

// Helper function to check if a file is corrupt (simplified example)
func isCorrupt(data []byte) bool {
    // In a real implementation, this might check checksums, file headers, etc.
    return len(data) > 0 && data[0] == 0 && len(data) % 16 != 0
}

// Example client code showing how to use the storage package
func Example() {
    // Create a storage instance
    storage, err := NewFileStorage("/tmp/storage")
    if err != nil {
        fmt.Printf("Failed to initialize storage: %v\n", err)
        return
    }
    
    // Set user quota
    storage.SetUserQuota("alice", 1024*1024) // 1MB
    
    // Store a file
    data := []byte("Hello, world!")
    err = storage.StoreFile("alice", "documents/hello.txt", data)
    if err != nil {
        fmt.Printf("Error storing file: %v\n", err)
        
        if errors.Is(err, ErrQuotaExceeded) {
            fmt.Println("Please upgrade your account for more storage.")
        } else if errors.Is(err, ErrFileExists) {
            fmt.Println("File already exists. Use a different name or overwrite.")
        }
        return
    }
    
    fmt.Println("File stored successfully.")
    
    // Retrieve the file
    retrieved, err := storage.GetFile("alice", "documents/hello.txt")
    if err != nil {
        fmt.Printf("Error retrieving file: %v\n", err)
        
        if errors.Is(err, ErrFileNotFound) {
            fmt.Println("The file does not exist.")
        } else if errors.Is(err, ErrCorruptFile) {
            fmt.Println("The file appears to be damaged.")
        }
        return
    }
    
    fmt.Printf("Retrieved file content: %s\n", retrieved)
    
    // List files
    files, err := storage.ListFiles("alice", "documents")
    if err != nil {
        fmt.Printf("Error listing files: %v\n", err)
        return
    }
    
    fmt.Println("Files in directory:")
    for _, file := range files {
        fmt.Printf("- %s\n", file)
    }
    
    // Get usage
    usage, err := storage.GetUserUsage("alice")
    if err != nil {
        fmt.Printf("Error getting usage: %v\n", err)
        return
    }
    
    quota := storage.UserQuotas["alice"]
    fmt.Printf("Storage usage: %d/%d bytes (%.1f%%)\n",
        usage, quota, float64(usage)/float64(quota)*100)
    
    // Try to store a file that's too large
    largeData := make([]byte, 15*1024*1024) // 15MB
    err = storage.StoreFile("alice", "documents/large_file.bin", largeData)
    if err != nil {
        fmt.Printf("Error storing large file: %v\n", err)
        
        if errors.Is(err, ErrFileTooLarge) {
            fmt.Printf("File exceeds the maximum allowed size of %d bytes\n", 
                    storage.MaxFileSize)
        } else if errors.Is(err, ErrQuotaExceeded) {
            fmt.Println("Not enough storage space available.")
        }
        return
    }
}
```

**Common Pitfalls:**
- Using string comparisons (`if err.Error() == "file not found"`) instead of sentinel errors
- Not exporting sentinel errors that API consumers might want to check for
- Creating too many specific error constants, leading to unwieldy error checking
- Not documenting what functions might return which sentinel errors
- Forgetting to use errors.Is() when checking for wrapped sentinel errors
- Creating non-descriptive or ambiguous error messages
- Using sentinel errors when custom error types with additional data would be more appropriate
- Not wrapping sentinel errors with additional context

**Confusion Questions:**

1. **Q: How do I choose between sentinel errors and custom error types?**

   A: Choosing between sentinel errors and custom error types is an important design decision that depends on several factors. Here's a comprehensive guide to help you decide:

   **Sentinel errors** are predefined, exported error variables:
   ```go
   var (
       ErrNotFound = errors.New("not found")
       ErrTimeout = errors.New("timed out")
   )
   ```

   **Custom error types** are structs that implement the error interface:
   ```go
   type ValidationError struct {
       Field string
       Message string
   }

   func (e ValidationError) Error() string {
       return fmt.Sprintf("validation error on %s: %s", e.Field, e.Message)
   }
   ```

   **Use sentinel errors when:**

   1. **The error condition is simple** and doesn't need additional data
      ```go
      // Good sentinel error use case
      var ErrQueueFull = errors.New("queue full")
      
      func (q *Queue) Push(item interface{}) error {
          if q.isFull() {
              return ErrQueueFull  // Simple condition, no extra context needed
          }
          // ...
      }
      ```

   2. **You want consumers to check for specific error conditions**
      ```go
      // Easily checked by callers
      if errors.Is(err, ErrQueueFull) {
          // Wait and retry
      }
      ```

   3. **The error is a common, expected condition** that callers will handle
      ```go
      var ErrNoMoreItems = errors.New("no more items")
      
      // Used as a signal that's expected by callers
      func (i *Iterator) Next() (Item, error) {
          if i.exhausted {
              return Item{}, ErrNoMoreItems
          }
          // ...
      }
      ```

   4. **For API stability** where error types might change
      ```go
      // Stable API contract that won't break if implementation changes
      var ErrRateLimited = errors.New("rate limit exceeded")
      ```

   5. **For simple packages with few error conditions**
      ```go
      // Simple package with clear error conditions
      var (
          ErrInvalidFormat = errors.New("invalid format")
          ErrUnsupported = errors.New("unsupported operation")
      )
      ```

   **Use custom error types when:**

   1. **You need to include additional context or data**
      ```go
      type HTTPError struct {
          StatusCode int
          URL        string
          Method     string
      }
      
      func (e HTTPError) Error() string {
          return fmt.Sprintf("%s %s failed with status %d", 
                           e.Method, e.URL, e.StatusCode)
      }
      ```

   2. **Callers need to extract specific error fields**
      ```go
      var httpErr HTTPError
      if errors.As(err, &httpErr) {
          fmt.Printf("HTTP error %d calling %s\n", 
                    httpErr.StatusCode, httpErr.URL)
      }
      ```

   3. **The error needs behavior in the form of methods**
      ```go
      type RetryableError struct {
          Err       error
          RetryWait time.Duration
      }
      
      func (e RetryableError) Error() string {
          return fmt.Sprintf("%v (retry after %v)", e.Err, e.RetryWait)
      }
      
      func (e RetryableError) RetryAfter() time.Duration {
          return e.RetryWait
      }
      ```

   4. **You need a hierarchy of error types**
      ```go
      type DatabaseError struct {
          Operation string
          Table     string
          Err       error
      }
      
      // More specific error types
      type QueryError struct {
          DatabaseError // Embedding for hierarchy
          Query        string
      }
      
      type ConnectionError struct {
          DatabaseError // Embedding for hierarchy
          Server       string
      }
      ```

   5. **For more complex error patterns** that sentinel errors can't handle
      ```go
      type ValidationError struct {
          Errors map[string][]string // Multiple errors per field
      }
      ```

   **Hybrid approach: Sentinel errors with custom types**

   You can also combine both approaches:

   ```go
   // Sentinel errors for quick checking
   var (
       ErrValidation = errors.New("validation failed")
       ErrDatabase = errors.New("database error")
   )
   
   // Custom types for detailed information
   type ValidationError struct {
       Field string
       Value interface{}
   }
   
   func (e ValidationError) Error() string {
       return fmt.Sprintf("invalid value for %s: %v", e.Field, e.Value)
   }
   
   // Implement Is to allow errors.Is to match sentinel error
   func (e ValidationError) Is(target error) bool {
       return target == ErrValidation
   }
   
   // Usage
   func validateUser(user User) error {
       if user.Name == "" {
           return ValidationError{Field: "name", Value: user.Name}
       }
       // ...
       return nil
   }
   
   // Caller can do either:
   if errors.Is(err, ErrValidation) {
       // General handling for any validation error
   }
   
   // Or:
   var valErr ValidationError
   if errors.As(err, &valErr) {
       // Specific handling using error fields
   }
   ```

   **Decision guideline:**

   1. Start with sentinel errors for simple cases
   2. Move to custom error types when you need more context/data
   3. Consider a hybrid approach for complex packages
   4. Be consistent within a package
   5. Document your error handling strategy for API consumers
   6. Consider forward compatibility and API stability

   The best choice often depends on how consumers will use your API and what information they need from errors. Sentinel errors provide simplicity, while custom error types provide richness and extensibility.

2. **Q: What's the relationship between sentinel errors and error wrapping?**

   A: Sentinel errors and error wrapping work together in Go's error handling system, but they have different roles and can sometimes seem to conflict. Here's how they relate and how to use them effectively together:

   **How they work together:**

   When you wrap a sentinel error, `errors.Is()` can still find it in the error chain:

   ```go
   var ErrNotFound = errors.New("not found")

   // Wrap a sentinel error with context
   user, err := findUser(id)
   if err != nil {
       return nil, fmt.Errorf("getting user %s: %w", id, err)
   }

   // Later in the code:
   if errors.Is(err, ErrNotFound) {
       // This works even though the error was wrapped!
       return handleNotFound()
   }
   ```

   **Potential conflicts:**

   1. **Message clarity**: Wrapped errors change the error message, which might make direct string comparisons fail:

   ```go
   originalErr := ErrNotFound                       // "not found"
   wrappedErr := fmt.Errorf("user search: %w", err) // "user search: not found"

   // This won't work:
   if err.Error() == "not found" {  // BAD PRACTICE
       // Won't match wrapped error
   }

   // This works correctly:
   if errors.Is(err, ErrNotFound) {  // GOOD PRACTICE
       // Will find ErrNotFound in the error chain
   }
   ```

   2. **Error equality**: Traditional error comparison (`==`) doesn't work with wrapped errors:

   ```go
   // This won't work with wrapped errors:
   if err == ErrNotFound {  // BAD PRACTICE
       // Only checks for exact equality
   }

   // This works correctly:
   if errors.Is(err, ErrNotFound) {  // GOOD PRACTICE
       // Checks throughout the error chain
   }
   ```

   **Best practices for using sentinel errors with wrapping:**

   1. **Always use `errors.Is()` to check for sentinel errors**:

   ```go
   // Correct way to check for sentinel errors
   if errors.Is(err, io.EOF) {
       // Handle end-of-file condition
   }
   ```

   2. **Wrap sentinel errors with context**:

   ```go
   // Good: Add context while preserving original error
   if err != nil {
       if errors.Is(err, sql.ErrNoRows) {
           return fmt.Errorf("user %s not found: %w", userID, err)
       }
       return fmt.Errorf("database error: %w", err)
   }
   ```

   3. **Use `fmt.Errorf()` with `%w` to wrap errors**:

   ```go
   // Correctly wrapping a sentinel error
   return fmt.Errorf("validation failed: %w", ErrInvalidInput)
   ```

   4. **Consider implementing custom `Is` methods** for complex sentinel error relationships:

   ```go
   type NotFoundError struct {
       Resource string
       ID       string
   }

   func (e NotFoundError) Error() string {
       return fmt.Sprintf("%s with ID %s not found", e.Resource, e.ID)
   }

   // Custom Is method to match against sentinel errors
   func (e NotFoundError) Is(target error) bool {
       if target == ErrNotFound {
           return true
       }
       
       // Also check for another NotFoundError with empty fields
       t, ok := target.(NotFoundError)
       if ok {
           return (t.Resource == "" || t.Resource == e.Resource) &&
                 (t.ID == "" || t.ID == e.ID)
       }
       
       return false
   }
   ```

   5. **Be careful with sentinel error wrapping hierarchies**:

   ```go
   // Establish clear relationships between errors
   var (
       ErrDatabase = errors.New("database error")
       ErrQuery    = fmt.Errorf("query error: %w", ErrDatabase)
       ErrConnect  = fmt.Errorf("connection error: %w", ErrDatabase)
   )

   // Now these work as expected:
   errors.Is(ErrQuery, ErrDatabase)    // true
   errors.Is(ErrConnect, ErrDatabase)  // true
   ```

   **Real-world example combining sentinel errors and wrapping:**

   ```go
   package datastore

   import (
       "errors"
       "fmt"
   )

   // Sentinel errors exported by the package
   var (
       ErrNotFound      = errors.New("resource not found")
       ErrAlreadyExists = errors.New("resource already exists")
       ErrInvalidData   = errors.New("invalid data")
       ErrPermission    = errors.New("permission denied")
   )

   // Store implements a simple data store
   type Store struct {
       data map[string][]byte
   }

   func NewStore() *Store {
       return &Store{
           data: make(map[string][]byte),
       }
   }

   // Get retrieves data by key
   func (s *Store) Get(key string) ([]byte, error) {
       // Validate input
       if key == "" {
           return nil, fmt.Errorf("get with empty key: %w", ErrInvalidData)
       }

       // Get data
       data, ok := s.data[key]
       if !ok {
           // Return wrapped sentinel error with context
           return nil, fmt.Errorf("key %q: %w", key, ErrNotFound)
       }

       return data, nil
   }

   // Put stores data by key
   func (s *Store) Put(key string, data []byte) error {
       // Validate input
       if key == "" {
           return fmt.Errorf("put with empty key: %w", ErrInvalidData)
       }

       if len(data) == 0 {
           return fmt.Errorf("put empty data: %w", ErrInvalidData)
       }

       // Check if key exists
       if _, ok := s.data[key]; ok {
           return fmt.Errorf("key %q: %w", key, ErrAlreadyExists)
       }

       // Store data
       s.data[key] = data
       return nil
   }

   // Usage showing sentinel error checking with wrapping
   func Example() {
       store := NewStore()

       // Try to get non-existent key
       data, err := store.Get("user:123")
       if err != nil {
           // Check for specific sentinel error regardless of wrapping
           if errors.Is(err, ErrNotFound) {
               fmt.Println("User not found, creating new record")
               // Create new user...
           } else if errors.Is(err, ErrInvalidData) {
               fmt.Println("Invalid request")
           } else {
               fmt.Printf("Unexpected error: %v\n", err)
           }
       }

       // Try to put data with empty key
       err = store.Put("", []byte("data"))
       if err != nil {
           fmt.Printf("Error: %v\n", err)
           // Output: Error: put with empty key: invalid data
       }
   }
   ```

   The key takeaway is that sentinel errors and error wrapping are designed to work together seamlessly through the `errors.Is()` function, allowing you to add context to errors while still enabling specific error checking throughout your application.

3. **Q: How many sentinel errors should a package define and export?**

   A: Finding the right balance for the number of sentinel errors in a package is important for creating clean, usable APIs. Here's a comprehensive guide:

   **General principles:**

   1. **Define sentinel errors for conditions that callers need to handle specifically** 
   2. **Avoid creating sentinel errors for every possible failure**
   3. **Group similar error conditions under a single sentinel error**
   4. **Export errors that are part of your package's API contract**

   **Quantitative guidelines:**

   While there's no hard rule, most well-designed Go packages follow these patterns:

   - **Small utility packages**: 2-5 sentinel errors
   - **Medium-sized packages**: 5-10 sentinel errors
   - **Large, complex packages**: 10-20 sentinel errors

   Packages with more than 20 exported sentinel errors often become unwieldy and indicate a need for restructuring or using custom error types.

   **Examples from the standard library:**

   The standard library provides good examples of appropriate numbers of sentinel errors:

   1. **io package**: 3 sentinel errors
      ```go
      var (
          ErrShortWrite    = errors.New("short write")
          ErrShortBuffer   = errors.New("short buffer")
          EOF              = errors.New("EOF")
      )
      ```

   2. **os package**: About 10 sentinel errors
      ```go
      var (
          ErrInvalid    = errors.New("invalid argument")
          ErrPermission = errors.New("permission denied")
          ErrExist      = errors.New("file already exists")
          ErrNotExist   = errors.New("file does not exist")
          // ...and a few more
      )
      ```

   3. **http package**: Around 5 sentinel errors
      ```go
      var (
          ErrBodyNotAllowed = errors.New("http: request method or response status code does not allow body")
          ErrHandlerTimeout = errors.New("http: Handler timeout")
          ErrMissingFile    = errors.New("http: no such file")
          // ...and a few more
      )
      ```

   **Decision framework:**

   To decide which sentinel errors to define and export:

   1. **For each error condition, ask**:
      - Do API consumers need to specifically handle this condition?
      - Is this a common/expected error that merits special treatment?
      - Does this error require different handling from other errors?

   2. **Group errors when**:
      - Multiple errors represent the same logical category
      - Callers would handle them the same way
      - The distinction between them is an implementation detail

   3. **Consider custom error types instead when**:
      - You need to include additional context
      - There are many variations of similar errors
      - The error condition needs structured data

   **Signs you have too many sentinel errors:**

   1. **Consumers need complex conditional logic** to handle your errors
   2. **Error checking code is larger** than business logic
   3. **Similar errors** with minor variations
   4. **Errors that leak implementation details**
   5. **Errors that are rarely checked** for specifically

   **Signs you have too few sentinel errors:**

   1. **Consumers resort to string matching** to distinguish error cases
   2. **Important error conditions** can't be distinguished programmatically
   3. **Complex error handling** required within your package

   **Example: Well-balanced approach**

   ```go
   package datastore

   import "errors"

   // Core sentinel errors represent key categories
   var (
       // ErrNotFound indicates the requested resource doesn't exist
       ErrNotFound = errors.New("resource not found")
       
       // ErrAlreadyExists indicates an attempt to create a duplicate resource
       ErrAlreadyExists = errors.New("resource already exists")
       
       // ErrInvalidInput indicates invalid parameters were provided
       ErrInvalidInput = errors.New("invalid input")
       
       // ErrPermission indicates the operation isn't allowed
       ErrPermission = errors.New("permission denied")
       
       // ErrUnavailable indicates the service is temporarily unavailable
       ErrUnavailable = errors.New("service unavailable")
   )

   // Custom error types for when more context is needed
   type ValidationError struct {
       Field   string
       Message string
   }

   func (e ValidationError) Error() string {
       return fmt.Sprintf("validation failed for %s: %s", e.Field, e.Message)
   }

   // Implement Is method to match against sentinel error
   func (e ValidationError) Is(target error) bool {
       return target == ErrInvalidInput
   }
   ```

   **Implementation example:**

   ```go
   // Using the balanced approach above
   func (db *Database) GetUser(id string) (*User, error) {
       if id == "" {
           return nil, fmt.Errorf("getting user: %w", ErrInvalidInput)
       }
       
       user, found := db.users[id]
       if !found {
           return nil, fmt.Errorf("user %s: %w", id, ErrNotFound)
       }
       
       if !db.hasPermission(currentUser, user) {
           return nil, fmt.Errorf("accessing user %s: %w", id, ErrPermission)
       }
       
       return user, nil
   }
   
   func (db *Database) ValidateUser(user *User) error {
       if user.Name == "" {
           // More specific error using custom type
           return ValidationError{Field: "name", Message: "cannot be empty"}
       }
       
       if user.Age < 18 {
           return ValidationError{Field: "age", Message: "must be at least 18"}
       }
       
       return nil
   }
   ```

   **Documenting exported errors:**

   Always document your exported sentinel errors clearly:

   ```go
   // ErrNotFound is returned when a requested resource doesn't exist.
   // It can be returned by GetUser, GetProduct, and similar retrieval methods.
   var ErrNotFound = errors.New("resource not found")
   ```

   **Best practice summary:**

   1. Start with a small set of general error categories
   2. Add specific errors only when there's clear justification
   3. Use error wrapping to add context to sentinel errors
   4. Consider custom error types for complex cases
   5. Document your errors as part of your API contract
   6. Look to the standard library for guidance on idiomatic practices

   By thoughtfully considering these guidelines, you can create a balanced set of sentinel errors that makes your package easy to use without overwhelming API consumers with too many error variations to handle.

### 7. Panic and Recovery

**Concise Explanation:**
Panic in Go is a mechanism for handling exceptional runtime conditions that the program cannot gracefully recover from during normal execution. When a function calls `panic()`, normal execution stops, deferred functions run, and control returns up the call stack until it's either caught by a `recover()` call or the program terminates. Unlike exceptions in other languages, panic is intended for exceptional conditions, not normal error handling. Recovery allows you to catch a panic and resume normal execution, typically used in library code to prevent application crashes and provide more graceful error handling.

**Where to Use:**
- For truly unrecoverable situations, like initialization failures
- When a critical invariant or assumption has been violated
- In development to catch programmer errors that should be fixed
- In wrapper functions to prevent crashes in library code
- To convert unexpected panics into errors at API boundaries
- In testing to check for expected panics
- As a last resort when no graceful error handling option exists

**Code Snippet:**
```go
package main

import (
    "fmt"
    "log"
    "runtime/debug"
)

func main() {
    // Example 1: Basic panic and recover
    fmt.Println("Example 1: Basic panic and recover")
    safelyDo(func() {
        fmt.Println("  Start of dangerous operation")
        panic("something went terribly wrong")
        fmt.Println("  End of dangerous operation") // Never executed
    })
    
    // Example 2: Recovering with returned error
    fmt.Println("\nExample 2: Recovering with returned error")
    err := dangerousOperation(10, 0)
    if err != nil {
        fmt.Printf("  Caught error: %v\n", err)
    }
    
    // Example 3: Panic in a deeper call stack
    fmt.Println("\nExample 3: Panic in a deeper call stack")
    safelyDo(func() {
        a()
    })
    
    // Example 4: Selective recovery
    fmt.Println("\nExample 4: Selective recovery")
    safelyDo(func() {
        recoverSelectively()
    })
    
    fmt.Println("\nProgram completed normally")
}

// safelyDo executes a function and recovers from any panic
func safelyDo(f func()) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("  Recovered from panic: %v\n", r)
            fmt.Printf("  Stack trace: %s\n", debug.Stack())
        }
    }()
    
    f()
}

// dangerousOperation converts panics to errors
func dangerousOperation(a, b int) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("operation failed: %v", r)
        }
    }()
    
    if b == 0 {
        panic("division by zero")
    }
    
    result := a / b
    fmt.Printf("  Result: %d\n", result)
    return nil
}

// Call stack example
func a() {
    b()
}

func b() {
    c()
}

func c() {
    panic("panic in function c")
}

// Selective recovery example
func recoverSelectively() {
    defer func() {
        if r := recover(); r != nil {
            // Only recover from specific types of panics
            if errStr, ok := r.(string); ok && errStr == "expected panic" {
                fmt.Println("  Recovered from expected panic")
            } else {
                fmt.Println("  Unexpected panic type, rethrowing")
                panic(r) // Re-panic for unexpected types
            }
        }
    }()
    
    // This will be recovered
    panic("expected panic")
    
    // If we changed this to:
    // panic(123)
    // It would be rethrown
}

// Real-world HTTP handler example with recover
func handleRequest(w http.ResponseWriter, r *http.Request) {
    defer func() {
        if err := recover(); err != nil {
            // Log the error and stack trace
            log.Printf("Panic in request handler: %v\n%s", err, debug.Stack())
            
            // Return a 500 Internal Server Error
            http.Error(w, "Internal Server Error", http.StatusInternalServerError)
        }
    }()
    
    // Normal request handling here...
    // If a panic occurs, it will be recovered and a 500 error returned
}
```

**Real-World Example:**
A resilient database connection pool with panic recovery:

```go
package dbpool

import (
    "context"
    "database/sql"
    "errors"
    "fmt"
    "log"
    "runtime/debug"
    "sync"
    "time"
)

// Common errors
var (
    ErrNilDB          = errors.New("nil database connection")
    ErrPoolClosed     = errors.New("connection pool is closed")
    ErrNoConnections  = errors.New("no available connections")
    ErrQueryFailed    = errors.New("query execution failed")
    ErrTxFailed       = errors.New("transaction failed")
)

// ConnectionPool manages a pool of database connections
type ConnectionPool struct {
    db           *sql.DB
    maxRetries   int
    maxIdleTime  time.Duration
    mu           sync.RWMutex
    closed       bool
    stats        PoolStats
}

// PoolStats tracks statistics about connection usage
type PoolStats struct {
    OpenConnections int
    InUse           int
    Idle            int
    WaitCount       int64
    WaitDuration    time.Duration
    MaxIdleTime     time.Duration
    MaxLifetime     time.Duration
    MaxOpenConns    int
    MaxIdleConns    int
}

// Config holds pool configuration options
type Config struct {
    MaxOpenConns    int
    MaxIdleConns    int
    MaxLifetime     time.Duration
    MaxIdleTime     time.Duration
    MaxRetries      int
}

// DefaultConfig returns a default configuration
func DefaultConfig() Config {
    return Config{
        MaxOpenConns: 10,
        MaxIdleConns: 5,
        MaxLifetime:  15 * time.Minute,
        MaxIdleTime:  5 * time.Minute,
        MaxRetries:   3,
    }
}

// New creates a new connection pool
func New(dataSourceName string, config Config) (*ConnectionPool, error) {
    if dataSourceName == "" {
        return nil, errors.New("empty data source name")
    }
    
    // Try to open the database - this could panic in some drivers
    var db *sql.DB
    var err error
    
    // Use panic recovery to handle any driver panics
    func() {
        defer func() {
            if r := recover(); r != nil {
                err = fmt.Errorf("database driver panicked: %v", r)
            }
        }()
        
        db, err = sql.Open("postgres", dataSourceName)
    }()
    
    if err != nil {
        return nil, fmt.Errorf("opening database connection: %w", err)
    }
    
    if db == nil {
        return nil, ErrNilDB
    }
    
    // Configure the connection pool
    db.SetMaxOpenConns(config.MaxOpenConns)
    db.SetMaxIdleConns(config.MaxIdleConns)
    db.SetConnMaxLifetime(config.MaxLifetime)
    db.SetConnMaxIdleTime(config.MaxIdleTime)
    
    // Verify connectivity
    if err := db.Ping(); err != nil {
        return nil, fmt.Errorf("verifying database connection: %w", err)
    }
    
    return &ConnectionPool{
        db:          db,
        maxRetries:  config.MaxRetries,
        maxIdleTime: config.MaxIdleTime,
    }, nil
}

// Close closes the connection pool
func (p *ConnectionPool) Close() error {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    if p.closed {
        return ErrPoolClosed
    }
    
    p.closed = true
    return p.db.Close()
}

// Stats returns current pool statistics
func (p *ConnectionPool) Stats() PoolStats {
    p.mu.RLock()
    defer p.mu.RUnlock()
    
    stats := p.db.Stats()
    
    return PoolStats{
        OpenConnections: stats.OpenConnections,
        InUse:           stats.InUse,
        Idle:            stats.Idle,
        WaitCount:       stats.WaitCount,
        WaitDuration:    stats.WaitDuration,
        MaxIdleTime:     p.maxIdleTime,
        MaxOpenConns:    stats.MaxOpenConnections,
        MaxIdleConns:    stats.MaxIdleConnections,
    }
}

// Exec executes a query without returning rows
func (p *ConnectionPool) Exec(ctx context.Context, query string, args ...interface{}) (sql.Result, error) {
    p.mu.RLock()
    if p.closed {
        p.mu.RUnlock()
        return nil, ErrPoolClosed
    }
    p.mu.RUnlock()
    
    var result sql.Result
    var err error
    
    // Use the safe execution function with retries
    err = p.executeWithRetries(ctx, func() error {
        var execErr error
        result, execErr = p.db.ExecContext(ctx, query, args...)
        return execErr
    })
    
    if err != nil {
        return nil, fmt.Errorf("exec query failed: %w", err)
    }
    
    return result, nil
}

// Query executes a query that returns rows
func (p *ConnectionPool) Query(ctx context.Context, query string, args ...interface{}) (*sql.Rows, error) {
    p.mu.RLock()
    if p.closed {
        p.mu.RUnlock()
        return nil, ErrPoolClosed
    }
    p.mu.RUnlock()
    
    var rows *sql.Rows
    var err error
    
    // Use the safe execution function with retries
    err = p.executeWithRetries(ctx, func() error {
        var queryErr error
        rows, queryErr = p.db.QueryContext(ctx, query, args...)
        return queryErr
    })
    
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }
    
    return rows, nil
}

// QueryRow executes a query that returns a single row
func (p *ConnectionPool) QueryRow(ctx context.Context, query string, args ...interface{}) *sql.Row {
    p.mu.RLock()
    defer p.mu.RUnlock()
    
    if p.closed {
        // Return a row that will error when scanned
        return &sql.Row{}
    }
    
    // Use safe execution for the query
    var row *sql.Row
    
    // Use a closure to safely call QueryRowContext
    func() {
        defer func() {
            if r := recover(); r != nil {
                log.Printf("Panic in QueryRow: %v\nStack: %s", r, debug.Stack())
                row = &sql.Row{} // Return an empty row that will error when scanned
            }
        }()
        
        row = p.db.QueryRowContext(ctx, query, args...)
    }()
    
    return row
}

// Transaction represents a database transaction
type Transaction struct {
    tx   *sql.Tx
    pool *ConnectionPool
}

// Begin starts a new transaction
func (p *ConnectionPool) Begin(ctx context.Context) (*Transaction, error) {
    p.mu.RLock()
    if p.closed {
        p.mu.RUnlock()
        return nil, ErrPoolClosed
    }
    p.mu.RUnlock()
    
    var tx *sql.Tx
    var err error
    
    // Start transaction with panic recovery
    err = p.executeWithRetries(ctx, func() error {
        var txErr error
        tx, txErr = p.db.BeginTx(ctx, nil)
        return txErr
    })
    
    if err != nil {
        return nil, fmt.Errorf("begin transaction: %w", err)
    }
    
    return &Transaction{tx: tx, pool: p}, nil
}

// Commit commits the transaction
func (t *Transaction) Commit() error {
    if t.tx == nil {
        return errors.New("transaction is nil or already closed")
    }
    
    var err error
    
    // Safely commit with panic recovery
    func() {
        defer func() {
            if r := recover(); r != nil {
                err = fmt.Errorf("panic during commit: %v", r)
            }
        }()
        
        err = t.tx.Commit()
    }()
    
    // Clear the transaction to prevent reuse
    t.tx = nil
    
    if err != nil {
        return fmt.Errorf("commit transaction: %w", err)
    }
    
    return nil
}

// Rollback rolls back the transaction
func (t *Transaction) Rollback() error {
    if t.tx == nil {
        return nil // Already committed or rolled back
    }
    
    var err error
    
    // Safely rollback with panic recovery
    func() {
        defer func() {
            if r := recover(); r != nil {
                err = fmt.Errorf("panic during rollback: %v", r)
            }
        }()
        
        err = t.tx.Rollback()
    }()
    
    // Clear the transaction to prevent reuse
    t.tx = nil
    
    if err != nil {
        return fmt.Errorf("rollback transaction: %w", err)
    }
    
    return nil
}

// Exec executes a query within the transaction
func (t *Transaction) Exec(ctx context.Context, query string, args ...interface{}) (sql.Result, error) {
    if t.tx == nil {
        return nil, errors.New("transaction is nil or already closed")
    }
    
    var result sql.Result
    var err error
    
    // Execute with panic recovery
    func() {
        defer func() {
            if r := recover(); r != nil {
                err = fmt.Errorf("panic in transaction exec: %v", r)
            }
        }()
        
        result, err = t.tx.ExecContext(ctx, query, args...)
    }()
    
    if err != nil {
        return nil, fmt.Errorf("exec in transaction: %w", err)
    }
    
    return result, nil
}

// Query executes a query within the transaction
func (t *Transaction) Query(ctx context.Context, query string, args ...interface{}) (*sql.Rows, error) {
    if t.tx == nil {
        return nil, errors.New("transaction is nil or already closed")
    }
    
    var rows *sql.Rows
    var err error
    
    // Execute with panic recovery
    func() {
        defer func() {
            if r := recover(); r != nil {
                err = fmt.Errorf("panic in transaction query: %v", r)
            }
        }()
        
        rows, err = t.tx.QueryContext(ctx, query, args...)
    }()
    
    if err != nil {
        return nil, fmt.Errorf("query in transaction: %w", err)
    }
    
    return rows, nil
}

// QueryRow executes a query returning a single row
func (t *Transaction) QueryRow(ctx context.Context, query string, args ...interface{}) *sql.Row {
    if t.tx == nil {
        // Return a row that will error when scanned
        return &sql.Row{}
    }
    
    var row *sql.Row
    
    // Execute with panic recovery
    func() {
        defer func() {
            if r := recover(); r != nil {
                log.Printf("Panic in transaction QueryRow: %v\nStack: %s", r, debug.Stack())
                row = &sql.Row{} // Return an empty row that will error when scanned
            }
        }()
        
        row = t.tx.QueryRowContext(ctx, query, args...)
    }()
    
    return row
}

// WithTransaction runs a function within a transaction, committing if successful or rolling back if not
func (p *ConnectionPool) WithTransaction(ctx context.Context, fn func(*Transaction) error) error {
    tx, err := p.Begin(ctx)
    if err != nil {
        return err
    }
    
    // Ensure rollback happens if panic or error
    defer func() {
        if r := recover(); r != nil {
            // Try to rollback, but if it fails, just log it
            _ = tx.Rollback()
            // Re-panic after cleanup
            panic(r)
        } else if err != nil {
            // Only rollback if error occurred and we didn't panic
            _ = tx.Rollback()
        }
    }()
    
    // Run the user function
    err = fn(tx)
    if err != nil {
        return err
    }
    
    // Commit if everything succeeded
    return tx.Commit()
}

// executeWithRetries executes a function with retries and panic recovery
func (p *ConnectionPool) executeWithRetries(ctx context.Context, fn func() error) error {
    var err error
    var attempt int
    
    for attempt = 0; attempt <= p.maxRetries; attempt++ {
        // If this isn't the first attempt, wait before retrying
        if attempt > 0 {
            // Simple backoff strategy
            backoff := time.Duration(attempt*attempt*100) * time.Millisecond
            select {
            case <-time.After(backoff):
                // Continue with retry
            case <-ctx.Done():
                // Context cancelled during backoff
                return fmt.Errorf("operation cancelled during retry: %w", ctx.Err())
            }
        }
        
        // Execute with panic recovery
        func() {
            defer func() {
                if r := recover(); r != nil {
                    err = fmt.Errorf("database operation panicked: %v", r)
                    log.Printf("Recovered from database panic (attempt %d/%d): %v\nStack: %s",
                        attempt+1, p.maxRetries+1, r, debug.Stack())
                }
            }()
            
            err = fn()
        }()
        
        // If operation succeeded or context is done, return
        if err == nil || ctx.Err() != nil {
            return err
        }
        
        // Check if error is retryable (simplified)
        if !isRetryableError(err) {
            return err
        }
        
        // Log retry attempt
        log.Printf("Retrying database operation after error (attempt %d/%d): %v",
            attempt+1, p.maxRetries+1, err)
    }
    
    return fmt.Errorf("operation failed after %d attempts: %w", attempt, err)
}

// isRetryableError returns true if the error is retryable
func isRetryableError(err error) bool {
    // This is a simplified version - in real code you'd check for specific
    // database errors like connection timeouts, deadlocks, etc.
    if err == nil {
        return false
    }
    
    // Check for common retryable errors
    return errors.Is(err, context.DeadlineExceeded) ||
           strings.Contains(err.Error(), "connection reset by peer") ||
           strings.Contains(err.Error(), "database is locked") ||
           strings.Contains(err.Error(), "deadlock detected") ||
           strings.Contains(err.Error(), "connection refused")
}

// Example usage
func Example() {
    // Create a pool
    ctx := context.Background()
    config := DefaultConfig()
    
    pool, err := New("postgres://user:pass@localhost/mydb?sslmode=disable", config)
    if err != nil {
        log.Fatalf("Failed to create connection pool: %v", err)
    }
    defer pool.Close()
    
    // Execute a query
    rows, err := pool.Query(ctx, "SELECT id, name FROM users WHERE active = $1", true)
    if err != nil {
        log.Printf("Query failed: %v", err)
    }
    defer rows.Close()
    
    // Use a transaction
    err = pool.WithTransaction(ctx, func(tx *Transaction) error {
        // Insert a record
        res, err := tx.Exec(ctx, 
            "INSERT INTO users (name, email) VALUES ($1, $2)",
            "Jane Doe", "jane@example.com")
        if err != nil {
            return err
        }
        
        id, err := res.LastInsertId()
        if err != nil {
            return err
        }
        
        // Update another record
        _, err = tx.Exec(ctx, 
            "UPDATE users SET last_login = $1 WHERE id = $2",
            time.Now(), id)
        return err
    })
    
    if err != nil {
        log.Printf("Transaction failed: %v", err)
    } else {
        log.Println("Transaction completed successfully")
    }
    
    // Print stats
    stats := pool.Stats()
    log.Printf("Pool stats - Open: %d, In use: %d, Idle: %d", 
        stats.OpenConnections, stats.InUse, stats.Idle)
}
```

**Common Pitfalls:**
- Using panic for normal error conditions instead of returning errors
- Not using defer to ensure recover() is called
- Forgetting that recover() only works in deferred functions
- Continuing execution after a partial recovery without proper cleanup
- Using panic/recover as a control flow mechanism like exceptions in other languages
- Not checking the type of value returned by recover() before handling
- Silently swallowing panics without proper logging or error handling
- Rethrowing recovered panics inappropriately
- Not understanding that recover() only catches panics in the same goroutine
- Using panic in libraries without documenting this behavior

**Confusion Questions:**

1. **Q: When should I use panic/recover instead of returning errors?**

   A: The panic/recover mechanism and error returns serve different purposes in Go's error handling strategy. Understanding when to use each approach is crucial for writing robust Go code.

   **Use error returns (the standard approach) when:**

   1. **The failure is an expected part of normal operation**:
      ```go
      func readConfig(path string) (Config, error) {
          data, err := os.ReadFile(path)
          if err != nil {
              return Config{}, fmt.Errorf("reading config file: %w", err)
          }
          // ...
      }
      ```

   2. **The caller can reasonably handle the failure**:
      ```go
      user, err := db.GetUser(id)
      if err != nil {
          // Handle different error cases
          if errors.Is(err, sql.ErrNoRows) {
              return renderNotFound()
          }
          return renderError(err)
      }
      ```

   3. **For most application code**:
      ```go
      func processItem(item Item) error {
          if !item.Valid() {
              return errors.New("invalid item")
          }
          // Process...
          return nil
      }
      ```

   **Use panic/recover when:**

   1. **During initialization when the program cannot start correctly**:
      ```go
      func mustLoadConfig() Config {
          config, err := loadConfig(configPath)
          if err != nil {
              panic(fmt.Sprintf("Failed to load critical configuration: %v", err))
          }
          return config
      }
      ```

   2. **For truly unexpected conditions that indicate bugs**:
      ```go
      func process(data []byte) {
          if data == nil {
              // This should never happen based on our program design
              panic("nil data passed to process()")
          }
          // ...
      }
      ```

   3. **At API boundaries to convert panics to errors**:
      ```go
      // Library function that wraps potential panics
      func SafelyCall(f func() error) (err error) {
          defer func() {
              if r := recover(); r != nil {
                  err = fmt.Errorf("panic recovered: %v", r)
              }
          }()
          return f()
      }
      ```

   4. **For hot code paths where the performance impact of error checking is significant**:
      ```go
      // This is RARELY justified - only in extremely performance-critical code
      func parseToken(data []byte, offset int) string {
          if offset >= len(data) {
              panic("offset out of bounds")
          }
          // Fast parsing without repeated bounds checking
      }
      ```

   **Practical guidelines:**

   1. **Default to returning errors**, they're the idiomatic Go way

   2. **Use panic only when**:
      - The program cannot continue safely
      - It's a programmer error (assertion failure)
      - You're writing library code that needs to protect itself
      - In initialization code where failure should stop the program

   3. **Consider "Must" functions for initialization-only code**:
      ```go
      // Regular function returns an error
      func ParseTemplate(text string) (*Template, error) { ... }
      
      // "Must" wrapper panics instead for initialization use
      func MustParseTemplate(text string) *Template {
          t, err := ParseTemplate(text)
          if err != nil {
              panic(err)
          }
          return t
      }
      ```

   4. **Use recover in API boundaries and server handlers**:
      ```go
      func ServeHTTP(w http.ResponseWriter, r *http.Request) {
          defer func() {
              if rec := recover(); rec != nil {
                  log.Printf("Panic in handler: %v\n%s", rec, debug.Stack())
                  http.Error(w, "Internal Server Error", 500)
              }
          }()
          
          // Normal handler code that might panic
      }
      ```

   **Comparing the approaches**:

   | Aspect | Error Return | Panic/Recover |
   |--------|-------------|---------------|
   | Explicitness | Explicit in function signature | Not visible in signature |
   | Handling location | At call site | Up the call stack |
   | Performance | Small overhead in happy path | Better performance until panic |
   | Recoverability | Highly recoverable | Less controlled |
   | Documentation | Self-documenting (in signature) | Needs explicit docs |
   | Stack trace | Not included by default | Included with debug.Stack() |
   | Idiomatic Go | Standard practice | Exception to the rule |

   **Real-world example of appropriate use**:

   ```go
   // Package initialization
   var templates = map[string]*template.Template{}

   func init() {
       // We want the program to fail completely if templates are invalid
       templates["user"] = template.Must(template.ParseFiles("templates/user.html"))
       templates["admin"] = template.Must(template.ParseFiles("templates/admin.html"))
   }

   // HTTP handler with recovery
   func handleRequest(w http.ResponseWriter, r *http.Request) {
       defer func() {
           if r := recover(); r != nil {
               log.Printf("Handler panic: %v\nStack trace: %s", r, debug.Stack())
               http.Error(w, "Internal Server Error", http.StatusInternalServerError)
           }
       }()
       
       // Normal handler code with error returns for expected conditions
       username := r.URL.Query().Get("user")
       user, err := getUser(username)
       if err != nil {
           http.Error(w, "User not found", http.StatusNotFound)
           return
       }
       
       // Render template
       templates["user"].Execute(w, user)
   }

   // Standard function with error return
   func getUser(username string) (*User, error) {
       if username == "" {
           return nil, errors.New("empty username")
       }
       
       // Database operations that might fail
       // ...
   }
   ```

   In summary, use error returns for nearly all error handling in Go. Reserve panic for exceptional circumstances like unrecoverable initialization failures or protecting against programmer errors, and use recover to convert panics into errors at package boundaries.

2. **Q: How does recover() work and what are its limitations?**

   A: The `recover()` function is Go's mechanism for catching and handling panics. Understanding how it works and its limitations is crucial for using it effectively and safely.

   **How recover() works:**

   `recover()` captures the value provided to `panic()` and returns it, allowing execution to continue normally from the point of the panic. Key points:

   ```go
   func ExampleRecover() {
       defer func() {
           if r := recover(); r != nil {
               fmt.Printf("Recovered from: %v\n", r)
           }
       }()
       
       fmt.Println("Before panic")
       panic("something went wrong")
       fmt.Println("After panic") // Never executed
   }
   ```

   1. **Only works in deferred functions**: `recover()` is only effective when called directly from a deferred function.

   2. **Returns panic value**: It returns the value passed to `panic()`, or `nil` if no panic is occurring.

   3. **Stops the panic sequence**: When successful, it stops unwinding the stack and allows normal execution to resume.

   4. **Executes remaining defers**: Any deferred functions in the current function still run after a successful recovery.

   **Limitations and constraints:**

   1. **Only catches panics in the same goroutine**:

   ```go
   func MainGoroutine() {
       defer func() {
           if r := recover(); r != nil {
               fmt.Println("This will NOT catch the panic in the goroutine")
           }
       }()
       
       go func() {
           panic("panic in goroutine") // This panic won't be caught
       }()
       
       time.Sleep(time.Second) // Wait for goroutine
   }
   ```

   Proper approach:
   ```go
   func ProperlyCatchGoroutinePanic() {
       go func() {
           defer func() {
               if r := recover(); r != nil {
                   fmt.Println("Caught panic inside goroutine")
               }
           }()
           
           panic("panic in goroutine") // This will be caught
       }()
       
       time.Sleep(time.Second) // Wait for goroutine
   }
   ```

   2. **Must be directly called from a deferred function**:

   ```go
   func IndirectRecoverFails() {
       defer func() {
           catchPanic() // recover() inside this function won't work!
       }()
       
       panic("won't be caught")
   }
   
   func catchPanic() {
       if r := recover(); r != nil { // This recover() won't catch anything
           fmt.Println("Caught:", r)
       }
   }
   ```

   Proper approach:
   ```go
   func DirectRecoverWorks() {
       defer func() {
           if r := recover(); r != nil { // Directly within deferred function
               fmt.Println("Caught:", r)
           }
       }()
       
       panic("will be caught")
   }
   ```

   3. **Can't selectively recover specific panics** (without checking the value):

   ```go
   func SelectiveRecovery() {
       defer func() {
           if r := recover(); r != nil {
               // Need to check the panic value manually
               if err, ok := r.(error); ok && errors.Is(err, sql.ErrNoRows) {
                   fmt.Println("Recovered from SQL no rows error")
               } else {
                   // Re-panic for other types
                   panic(r)
               }
           }
       }()
       
       // This will be caught
       panic(fmt.Errorf("wrapping: %w", sql.ErrNoRows))
   }
   ```

   4. **Doesn't clean up resources automatically**:

   ```go
   func ResourceCleanupWithPanic() {
       f, err := os.Open("file.txt")
       if err != nil {
           panic(err)
       }
       // Without defer, this would leak file handle on panic
       defer f.Close()
       
       panic("resource leak prevented by defer")
   }
   ```

   5. **Stack trace not automatically preserved**:

   ```go
   func CaptureStackTrace() {
       defer func() {
           if r := recover(); r != nil {
               // Need to explicitly capture stack trace
               fmt.Printf("Panic: %v\nStack:\n%s", r, debug.Stack())
           }
       }()
       
       deepFunction()
   }
   
   func deepFunction() {
       deeperFunction()
   }
   
   func deeperFunction() {
       panic("deep panic")
   }
   ```

   6. **Doesn't handle runtime fatal errors**:

   ```go
   func CantRecoverFatal() {
       defer func() {
           if r := recover(); r != nil {
               fmt.Println("This won't execute for fatal errors")
           }
       }()
       
       // This will terminate the program without possibility of recovery
       var ptr *int
       *ptr = 42 // Nil pointer dereference - fatal error
   }
   ```

   **Best practices for using recover():**

   1. **Add context and logging**:

   ```go
   func RecoverWithContext() {
       defer func() {
           if r := recover(); r != nil {
               log.Printf("Panic occurred in RecoverWithContext: %v\nStack trace:\n%s", 
                         r, debug.Stack())
               // Consider adding additional context or metrics here
           }
       }()
       
       // Function body
   }
   ```

   2. **Convert panics to errors at API boundaries**:

   ```go
   func SafeOperation() (result string, err error) {
       defer func() {
           if r := recover(); r != nil {
               // Convert panic to error return
               err = fmt.Errorf("operation panicked: %v", r)
               
               // Log the stack trace for debugging
               log.Printf("Panic in SafeOperation: %v\n%s", r, debug.Stack())
           }
       }()
       
       result = riskyOperation()
       return result, nil
   }
   ```

   3. **Handle goroutine panics properly**:

   ```go
   func SafeGoroutine(task func(), errorChan chan<- error) {
       go func() {
           defer func() {
               if r := recover(); r != nil {
                   stack := debug.Stack()
                   err := fmt.Errorf("goroutine panicked: %v\n%s", r, stack)
                   select {
                   case errorChan <- err:
                       // Error reported
                   default:
                       // Channel full or closed, log instead
                       log.Printf("Unhandled panic: %v", err)
                   }
               }
           }()
           
           task()
       }()
   }
   ```

   4. **Use recover() with testing**:

   ```go
   func TestExpectPanic(t *testing.T) {
       defer func() {
           if r := recover(); r == nil {
               t.Errorf("Expected a panic but none occurred")
           }
       }()
       
       functionThatShouldPanic()
   }
   ```

   5. **Implement middleware-style recovery**:

   ```go
   func RecoveryMiddleware(next http.Handler) http.Handler {
       return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
           defer func() {
               if rec := recover(); rec != nil {
                   // Log the panic
                   log.Printf("Request panic: %v\nStack:%s", rec, debug.Stack())
                   
                   // Return error to client
                   http.Error(w, "Internal Server Error", http.StatusInternalServerError)
                   
                   // Optional: notify monitoring systems
                   notifyMonitoring(rec, r.URL.Path)
               }
           }()
           
           next.ServeHTTP(w, r)
       })
   }
   ```

   **Advanced recovery with selective handling**:

   ```go
   type PanicLevel int

   const (
       LevelFatal PanicLevel = iota
       LevelError
       LevelWarning
   )

   type ControlledPanic struct {
       Level   PanicLevel
       Message string
       Err     error
   }

   func (c ControlledPanic) Error() string {
       return c.Message
   }

   func HandleWithGranularity() (err error) {
       defer func() {
           if r := recover(); r != nil {
               switch p := r.(type) {
               case ControlledPanic:
                   // Handle based on panic level
                   switch p.Level {
                   case LevelWarning:
                       log.Printf("Warning: %v", p.Message)
                       // Continue execution
                   case LevelError:
                       log.Printf("Error: %v", p.Message)
                       err = p.Err
                   case LevelFatal:
                       log.Printf("Fatal error: %v", p.Message)
                       err = p.Err
                       // Could also re-panic for fatal issues
                       // panic(p)
                   }
               default:
                   // Unexpected panic type
                   err = fmt.Errorf("unexpected panic: %v", r)
                   log.Printf("Unexpected panic: %v\n%s", r, debug.Stack())
               }
           }
       }()
       
       // Function implementation
       problematicOperation()
       return nil
   }
   
   func problematicOperation() {
       // Could trigger different types of panics
       panic(ControlledPanic{
           Level:   LevelError,
           Message: "Database connection failed",
           Err:     errors.New("connection timeout"),
       })
   }
   ```

   **Summary of recover() limitations:**

   1. Only works in deferred functions called directly by panicking function
   2. Only catches panics in the same goroutine
   3. Cannot automatically differentiate between panic types
   4. Doesn't automatically preserve stack traces
   5. Cannot recover from fatal runtime errors
   6. Requires manual resource cleanup through defer

   By understanding these limitations and following best practices, you can use `recover()` effectively to make your Go programs more robust while maintaining the principles of clear error handling.

3. **Q: How do I handle panics in goroutines?**

   A: Handling panics in goroutines requires special care because Go's standard `recover()` mechanism only catches panics within the same goroutine. Here's a comprehensive guide to managing panics across goroutines:

   **The problem with goroutines and panics:**

   ```go
   func main() {
       // This recover will NOT catch the panic in the goroutine
       defer func() {
           if r := recover(); r != nil {
               fmt.Println("Main recover - this won't work for goroutine panics")
           }
       }()
       
       go func() {
           // This will crash your program
           panic("panic in goroutine")
       }()
       
       time.Sleep(time.Second) // Wait for goroutine to execute
   }
   ```

   **Basic Solution: Recover inside each goroutine**

   ```go
   func main() {
       go func() {
           // Each goroutine needs its own recovery
           defer func() {
               if r := recover(); r != nil {
                   fmt.Printf("Recovered from panic in goroutine: %v\n", r)
               }
           }()
           
           panic("panic in goroutine")
       }()
       
       time.Sleep(time.Second) // Wait for goroutine execution
       fmt.Println("Main function exits normally")
   }
   ```

   **Solutions for goroutine panic handling:**

   **1. Wrap goroutine execution in a safe function:**

   ```go
   // SafeGo runs a function in a goroutine with panic recovery
   func SafeGo(fn func()) {
       go func() {
           defer func() {
               if r := recover(); r != nil {
                   log.Printf("Panic in goroutine: %v\n%s", r, debug.Stack())
               }
           }()
           
           fn()
       }()
   }

   // Usage
   func main() {
       SafeGo(func() {
           // Do work that might panic
           panic("test panic")
       })
       
       // Main continues running
       time.Sleep(time.Second)
   }
   ```

   **2. Communicate panic information back through channels:**

   ```go
   // SafeGoWithNotify runs a function in a goroutine and notifies on completion or panic
   func SafeGoWithNotify(fn func(), errChan chan<- error, doneChan chan<- bool) {
       go func() {
           defer func() {
               if r := recover(); r != nil {
                   stack := debug.Stack()
                   errChan <- fmt.Errorf("panic: %v\n%s", r, stack)
               } else {
                   doneChan <- true
               }
           }()
           
           fn()
       }()
   }

   // Usage
   func main() {
       errChan := make(chan error, 1)
       doneChan := make(chan bool, 1)
       
       SafeGoWithNotify(func() {
           // Do work that might panic
           panic("test panic")
       }, errChan, doneChan)
       
       // Wait for completion or error
       select {
       case err := <-errChan:
           fmt.Printf("Goroutine failed: %v\n", err)
       case <-doneChan:
           fmt.Println("Goroutine completed successfully")
       case <-time.After(5 * time.Second):
           fmt.Println("Goroutine timed out")
       }
   }
   ```

   **3. Using worker pools with panic recovery:**

   ```go
   // Task represents a unit of work
   type Task struct {
       ID  int
       Work func() (interface{}, error)
   }

   // Result represents the outcome of a task
   type Result struct {
       TaskID    int
       Value     interface{}
       Err       error
       Panicked  bool
       PanicVal  interface{}
       Stack     string
   }

   // WorkerPool manages a pool of workers that safely handle panics
   type WorkerPool struct {
       tasks   chan Task
       results chan Result
       wg      sync.WaitGroup
       quit    chan struct{}
   }

   // NewWorkerPool creates a new worker pool
   func NewWorkerPool(numWorkers int) *WorkerPool {
       return &WorkerPool{
           tasks:   make(chan Task, 100),
           results: make(chan Result, 100),
           quit:    make(chan struct{}),
       }
   }

   // Start begins the worker pool
   func (wp *WorkerPool) Start() {
       // Start workers
       for i := 0; i < numWorkers; i++ {
           wp.wg.Add(1)
           go wp.worker(i)
       }
   }

   // Submit adds a task to the pool
   func (wp *WorkerPool) Submit(task Task) {
       wp.tasks <- task
   }

   // Results returns the results channel
   func (wp *WorkerPool) Results() <-chan Result {
       return wp.results
   }

   // Stop shuts down the worker pool
   func (wp *WorkerPool) Stop() {
       close(wp.quit)
       wp.wg.Wait()
       close(wp.results)
   }

   // worker executes tasks and recovers from panics
   func (wp *WorkerPool) worker(id int) {
       defer wp.wg.Done()
       
       for {
           select {
           case <-wp.quit:
               return
               
           case task, ok := <-wp.tasks:
               if !ok {
                   return
               }
               
               // Process the task with panic recovery
               result := Result{TaskID: task.ID}
               
               func() {
                   defer func() {
                       if r := recover(); r != nil {
                           // Capture panic information
                           result.Panicked = true
                           result.PanicVal = r
                           result.Stack = string(debug.Stack())
                           result.Err = fmt.Errorf("panic: %v", r)
                       }
                   }()
                   
                   // Execute the task
                   val, err := task.Work()
                   result.Value = val
                   result.Err = err
               }()
               
               // Send the result
               select {
               case wp.results <- result:
                   // Result sent
               case <-wp.quit:
                   return
               }
           }
       }
   }

   // Usage example
   func main() {
       pool := NewWorkerPool(5)
       pool.Start()
       
       // Submit tasks
       for i := 0; i < 10; i++ {
           id := i
           pool.Submit(Task{
               ID: id,
               Work: func() (interface{}, error) {
                   // Simulate work with occasional panic
                   if id == 3 || id == 7 {
                       panic(fmt.Sprintf("Simulated panic in task %d", id))
                   }
                   
                   // Normal processing
                   return fmt.Sprintf("Result for task %d", id), nil
               },
           })
       }
       
       // Process results
       go func() {
           for result := range pool.Results() {
               if result.Panicked {
                   fmt.Printf("Task %d panicked: %v\nStack:\n%s\n", 
                             result.TaskID, result.PanicVal, result.Stack)
               } else if result.Err != nil {
                   fmt.Printf("Task %d failed with error: %v\n", 
                             result.TaskID, result.Err)
               } else {
                   fmt.Printf("Task %d succeeded: %v\n", 
                             result.TaskID, result.Value)
               }
           }
       }()
       
       // Allow time for processing
       time.Sleep(2 * time.Second)
       pool.Stop()
   }
   ```

   **4. Monitoring goroutine health with heartbeats:**

   ```go
   // Task with heartbeat monitoring
   func MonitoredTask(name string, timeout time.Duration) {
       heartbeat := make(chan struct{})
       done := make(chan struct{})
       
       // Start monitoring goroutine
       go func() {
           select {
           case <-heartbeat:
               fmt.Printf("Task %s completed successfully\n", name)
           case <-time.After(timeout):
               fmt.Printf("Task %s timed out or panicked\n", name)
               // Take recovery action
           }
       }()
       
       // Start worker goroutine
       go func() {
           defer func() {
               if r := recover(); r != nil {
                   fmt.Printf("Task %s panicked: %v\n%s", 
                             name, r, debug.Stack())
                   // Don't send heartbeat - will trigger timeout
               } else {
                   heartbeat <- struct{}{} // Normal completion
               }
               close(done)
           }()
           
           // Do work that might panic
           doRiskyWork(name)
       }()
       
       <-done // Wait for completion
   }
   ```

   **5. Using context for coordinated cancellation with panic handling:**

   ```go
   func SafeWorkerWithContext(ctx context.Context, errorChan chan<- error) {
       go func() {
           defer func() {
               if r := recover(); r != nil {
                   select {
                   case errorChan <- fmt.Errorf("panic: %v\n%s", r, debug.Stack()):
                       // Error reported
                   case <-ctx.Done():
                       // Context was cancelled, just log
                       log.Printf("Panic occurred but context cancelled: %v", r)
                   default:
                       // Channel full or closed, log
                       log.Printf("Panic occurred: %v", r)
                   }
               }
           }()
           
           for {
               select {
               case <-ctx.Done():
                   return
               default:
                   // Do work that might panic
                   riskyOperation()
                   time.Sleep(100 * time.Millisecond)
               }
           }
       }()
   }

   // Usage
   func main() {
       ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
       defer cancel()
       
       errorChan := make(chan error, 10)
       
       // Start workers
       for i := 0; i < 5; i++ {
           SafeWorkerWithContext(ctx, errorChan)
       }
       
       // Process errors from workers
       for {
           select {
           case err := <-errorChan:
               fmt.Printf("Worker error: %v\n", err)
               // Optionally cancel all workers if one fails
               // cancel()
           case <-ctx.Done():
               fmt.Println("Context done, all workers should exit")
               return
           }
       }
   }
   ```

   **6. WaitGroup with panic handling:**

   ```go
   func RunParallel(tasks []func() error) []error {
       var wg sync.WaitGroup
       var mu sync.Mutex
       errors := make([]error, 0)
       
       for i, task := range tasks {
           wg.Add(1)
           
           // Capture task and index for the closure
           t := task
           idx := i
           
           go func() {
               defer func() {
                   if r := recover(); r != nil {
                       err := fmt.Errorf("task %d panicked: %v\n%s", 
                                        idx, r, debug.Stack())
                       
                       // Safely collect the error
                       mu.Lock()
                       errors = append(errors, err)
                       mu.Unlock()
                   }
                   
                   wg.Done()
               }()
               
               if err := t(); err != nil {
                   mu.Lock()
                   errors = append(errors, fmt.Errorf("task %d error: %w", idx, err))
                   mu.Unlock()
               }
           }()
       }
       
       // Wait for all tasks to complete
       wg.Wait()
       
       return errors
   }

   // Usage
   func main() {
       tasks := []func() error{
           func() error { return nil }, // Success
           func() error { return errors.New("failed") }, // Error
           func() error { panic("oops") }, // Panic
       }
       
       errors := RunParallel(tasks)
       
       if len(errors) > 0 {
           for i, err := range errors {
               fmt.Printf("Error %d: %v\n", i, err)
           }
       } else {
           fmt.Println("All tasks completed successfully")
       }
   }
   ```

   **7. Using sync.Once to ensure panic recovery happens only once:**

   ```go
   // Service with safe initialization
   type Service struct {
       once      sync.Once
       initErr   error
       db        *sql.DB
       cache     *Cache
       processor *Processor
   }

   // Initialize starts the service with panic protection
   func (s *Service) Initialize() error {
       s.once.Do(func() {
           // Use a closure with panic recovery for initialization
           func() {
               defer func() {
                   if r := recover(); r != nil {
                       s.initErr = fmt.Errorf("initialization panicked: %v\n%s", 
                                           r, debug.Stack())
                   }
               }()
               
               // Initialization that might panic
               s.db = initDatabase()
               s.cache = initCache()
               s.processor = initProcessor()
           }()
       })
       
       return s.initErr
   }
   ```

   **Best practices for goroutine panic handling:**

   1. **Always recover panics** in each goroutine you create
   
   2. **Log sufficient context for debugging**:
      ```go
      if r := recover(); r != nil {
          log.Printf("Goroutine %d panicked: %v\nStack trace:\n%s", 
                    id, r, debug.Stack())
      }
      ```
   
   3. **Communicate failures back to the parent**:
      ```go
      // Using channels
      errChan <- fmt.Errorf("worker %d panicked: %v", id, r)
      
      // Or using a shared error list with mutex protection
      mu.Lock()
      errors = append(errors, fmt.Errorf("worker %d panicked: %v", id, r))
      mu.Unlock()
      ```
   
   4. **Consider clean termination of other goroutines**:
      ```go
      // If any goroutine panics, cancel all others
      if r := recover(); r != nil {
          log.Printf("Critical error: %v", r)
          cancel() // Cancel the shared context
      }
      ```
   
   5. **Create helper utilities for common patterns**:
      ```go
      // SafeGo is a reusable helper for running recoverable goroutines
      func SafeGo(fn func(), errHandler func(error)) {
          go func() {
              defer func() {
                  if r := recover(); r != nil {
                      err := fmt.Errorf("panic: %v\n%s", r, debug.Stack())
                      errHandler(err)
                  }
              }()
              
              fn()
          }()
      }
      ```
   
   6. **Monitor goroutine health with timeouts**:
      ```go
      // Set a timeout for expected completion
      timer := time.AfterFunc(timeout, func() {
          log.Printf("Goroutine %d may be deadlocked or panicked", id)
          // Take recovery action
      })
      defer timer.Stop() // Cancel timeout if completed normally
      ```

   **Real-world production pattern:**

   ```go
   package worker

   import (
       "context"
       "fmt"
       "log"
       "runtime/debug"
       "sync"
       "time"
   )

   // Job represents a unit of work
   type Job struct {
       ID      string
       Handler func(context.Context) (interface{}, error)
   }

   // Result represents the outcome of a job
   type Result struct {
       JobID      string
       Value      interface{}
       Err        error
       Panicked   bool
       ExecutedAt time.Time
       Duration   time.Duration
   }

   // Worker processes jobs
   type Worker struct {
       id         int
       jobChan    <-chan Job
       resultChan chan<- Result
       errorChan  chan<- error
       wg         *sync.WaitGroup
       ctx        context.Context
       cancel     context.CancelFunc
       logger     *log.Logger
   }

   // NewWorker creates a new worker
   func NewWorker(
       id int,
       jobChan <-chan Job,
       resultChan chan<- Result,
       errorChan chan<- error,
       wg *sync.WaitGroup,
       logger *log.Logger,
   ) *Worker {
       ctx, cancel := context.WithCancel(context.Background())
       return &Worker{
           id:         id,
           jobChan:    jobChan,
           resultChan: resultChan,
           errorChan:  errorChan,
           wg:         wg,
           ctx:        ctx,
           cancel:     cancel,
           logger:     logger,
       }
   }

   // Start begins worker processing
   func (w *Worker) Start() {
       w.wg.Add(1)
       go func() {
           defer w.wg.Done()
           defer func() {
               if r := recover(); r != nil {
                   stack := debug.Stack()
                   err := fmt.Errorf("worker %d panic: %v\n%s", w.id, r, stack)
                   w.logger.Printf("[CRITICAL] %v", err)
                   
                   // Try to report the panic
                   select {
                   case w.errorChan <- err:
                       // Error reported
                   default:
                       // Channel full or closed, just log
                       w.logger.Printf("[ERROR] Could not send error to channel")
                   }
               }
           }()
           
           w.logger.Printf("[INFO] Worker %d starting", w.id)
           w.processJobs()
       }()
   }

   // Stop stops the worker
   func (w *Worker) Stop() {
       w.cancel()
   }

   // processJobs handles the main job processing loop
   func (w *Worker) processJobs() {
       for {
           select {
           case <-w.ctx.Done():
               w.logger.Printf("[INFO] Worker %d shutting down", w.id)
               return
               
           case job, ok := <-w.jobChan:
               if !ok {
                   w.logger.Printf("[INFO] Job channel closed, worker %d exiting", w.id)
                   return
               }
               
               w.executeJob(job)
           }
       }
   }

   // executeJob runs a single job with panic recovery
   func (w *Worker) executeJob(job Job) {
       result := Result{
           JobID:      job.ID,
           ExecutedAt: time.Now(),
           Panicked:   false,
       }
       
       // Define job context with timeout
       jobCtx, cancel := context.WithTimeout(w.ctx, 30*time.Second)
       defer cancel()
       
       // Execute with panic recovery
       func() {
           defer func() {
               if r := recover(); r != nil {
                   stack := debug.Stack()
                   w.logger.Printf("[ERROR] Panic in job %s: %v\n%s", job.ID, r, stack)
                   
                   result.Err = fmt.Errorf("job panicked: %v", r)
                   result.Panicked = true
               }
           }()
           
           // Execute the job
           startTime := time.Now()
           value, err := job.Handler(jobCtx)
           result.Duration = time.Since(startTime)
           result.Value = value
           result.Err = err
           
           if err != nil {
               w.logger.Printf("[WARN] Job %s failed: %v", job.ID, err)
           } else {
               w.logger.Printf("[INFO] Job %s completed in %v", job.ID, result.Duration)
           }
       }()
       
       // Send the result
       select {
       case w.resultChan <- result:
           // Result sent successfully
       case <-w.ctx.Done():
           w.logger.Printf("[WARN] Context cancelled while sending result for job %s", job.ID)
           return
       }
   }

   // Pool manages a group of workers
   type Pool struct {
       workers    []*Worker
       jobChan    chan Job
       resultChan chan Result
       errorChan  chan error
       wg         sync.WaitGroup
       ctx        context.Context
       cancel     context.CancelFunc
       logger     *log.Logger
   }

   // NewPool creates a new worker pool
   func NewPool(numWorkers, queueSize int, logger *log.Logger) *Pool {
       ctx, cancel := context.WithCancel(context.Background())
       
       return &Pool{
           workers:    make([]*Worker, numWorkers),
           jobChan:    make(chan Job, queueSize),
           resultChan: make(chan Result, queueSize),
           errorChan:  make(chan error, numWorkers),
           ctx:        ctx,
           cancel:     cancel,
           logger:     logger,
       }
   }

   // Start begins pool operation
   func (p *Pool) Start() {
       p.logger.Printf("[INFO] Starting worker pool with %d workers", len(p.workers))
       
       // Start workers
       for i := 0; i < len(p.workers); i++ {
           p.workers[i] = NewWorker(
               i,
               p.jobChan,
               p.resultChan,
               p.errorChan,
               &p.wg,
               p.logger,
           )
           p.workers[i].Start()
       }
       
       // Monitor worker errors
       go p.monitorErrors()
   }

   // monitorErrors handles errors from workers
   func (p *Pool) monitorErrors() {
       for {
           select {
           case err := <-p.errorChan:
               p.logger.Printf("[CRITICAL] Worker error: %v", err)
               // Implement recovery strategy here (e.g., restart workers)
               
           case <-p.ctx.Done():
               return
           }
       }
   }

   // Submit adds a job to the pool
   func (p *Pool) Submit(job Job) error {
       select {
       case p.jobChan <- job:
           return nil
       case <-p.ctx.Done():
           return fmt.Errorf("pool is shutting down")
       default:
           return fmt.Errorf("job queue is full")
       }
   }

   // Results provides access to the results channel
   func (p *Pool) Results() <-chan Result {
       return p.resultChan
   }

   // Shutdown stops the pool and waits for all workers to finish
   func (p *Pool) Shutdown(timeout time.Duration) {
       p.logger.Printf("[INFO] Shutting down worker pool")
       
       // Signal shutdown
       p.cancel()
       
       // Close job channel to prevent new submissions
       close(p.jobChan)
       
       // Set up timeout
       doneCh := make(chan struct{})
       go func() {
           p.wg.Wait()
           close(doneCh)
       }()
       
       // Wait for workers to finish or timeout
       select {
       case <-doneCh:
           p.logger.Printf("[INFO] All workers exited cleanly")
       case <-time.After(timeout):
           p.logger.Printf("[WARN] Shutdown timed out after %v", timeout)
       }
       
       // Close result channel
       close(p.resultChan)
       close(p.errorChan)
   }
   ```

   By following these patterns and best practices, you can effectively handle panics in goroutines and prevent them from crashing your application. Remember that the key principles are to recover within each goroutine, communicate failures back to the parent, and provide enough context for debugging.

## Next Actions

### Exercises and Micro-Projects

#### Exercise 1: Custom Error Type Hierarchy
Create a custom error hierarchy for a file processing system. Include errors for file not found, permission issues, format errors, and content validation errors. Implement proper error wrapping and unwrapping, and demonstrate how to check for specific error types using `errors.Is()` and `errors.As()`.

#### Exercise 2: Error Recovery Middleware
Build middleware for an HTTP server that recovers from panics, logs the error with a stack trace, and returns a clean error message to the client. Add functionality to categorize and report different types of panics.

#### Exercise 3: Retry with Backoff
Implement a function that retries operations with exponential backoff when certain types of errors occur. Use error wrapping to provide context about each retry attempt.

#### Exercise 4: Sentinel Error Patterns
Design a package with sentinel errors and demonstrate the different ways to use them effectively. Show how to check for specific errors, wrap them with context, and define behavior based on error types.

#### Exercise 5: Panic Handling in Concurrent Operations
Create a system that runs multiple operations concurrently, each in a separate goroutine, and aggregates results while gracefully handling panics in any goroutine without crashing the entire application.

### Micro-Project: Resilient API Client

Develop a resilient API client library with comprehensive error handling:

1. **Requirements**:
   - Custom error types for different failure modes (network, authentication, rate limiting)
   - Exponential backoff retry logic for transient errors
   - Panic recovery with detailed logging
   - Context-aware operations with timeout and cancellation support
   - Error wrapping that preserves the original error while adding context

2. **Features**:
   - Custom error types with additional data fields
   - Sentinel errors for common scenarios
   - Method to convert errors to user-friendly messages
   - Circuit breaker pattern for failing endpoints
   - Detailed logging of errors with appropriate context

3. **Expected API**:
```go
client := apiclient.NewClient(
    apiclient.WithBaseURL("https://api.example.com"),
    apiclient.WithTimeout(5 * time.Second),
    apiclient.WithRetries(3),
)

resp, err := client.Get(ctx, "/users/123")
if err != nil {
    if errors.Is(err, apiclient.ErrNotFound) {
        // Handle not found
    } else if errors.Is(err, apiclient.ErrRateLimited) {
        // Handle rate limiting
    } else {
        // Handle other errors
    }
}
```

## Success Criteria

You've mastered error handling in Go when you can:

1. **Design Error Types**
   - Create appropriate custom error types for different scenarios
   - Implement error wrapping and unwrapping correctly
   - Use sentinel errors judiciously

2. **Handle Errors Appropriately**
   - Choose appropriate error checking mechanisms (errors.Is, errors.As)
   - Add helpful context to errors without losing information
   - Implement retry mechanisms for transient failures

3. **Panic with Purpose**
   - Use panic only when appropriate 
   - Implement proper recovery patterns
   - Convert panics to errors at API boundaries

4. **Structure Error Information**
   - Create errors with meaningful, actionable messages
   - Add appropriate context to errors when wrapping
   - Differentiate between user-facing and internal error details

5. **Log Errors Effectively**
   - Include relevant context with error logs
   - Use appropriate log levels for different error conditions
   - Avoid logging sensitive information in errors

## Troubleshooting

### Common Issues and Solutions

1. **Error checking is tedious and repetitive**
   - **Problem**: Adding error checks after each operation creates verbose code
   - **Solution**: Use helper functions for common patterns or consider using syntax highlighting in your editor to make error handling more visible

2. **Not enough context in error messages**
   - **Problem**: Errors like "not found" don't provide enough information
   - **Solution**: Always add context with `fmt.Errorf("doing X: %w", err)` to create a chain of operations

3. **Losing original error types when wrapping**
   - **Problem**: Checking for specific errors doesn't work after wrapping
   - **Solution**: Use `errors.Is()` and `errors.As()` instead of direct comparison or type assertions

4. **Panics crashing the entire application**
   - **Problem**: Unexpected panics in one part of the code crash everything
   - **Solution**: Add panic recovery at API boundaries and in goroutine entry points

5. **Too many error types causing confusion**
   - **Problem**: Excessive custom error types make error handling complicated
   - **Solution**: Use a smaller number of well-defined error types with additional context fields

### Debugging Error Handling

1. **Debugging error chains**:
   ```go
   // Print the full error chain
   fmt.Printf("Error: %v\n", err)
   
   // Walk and print the error chain
   for e := err; e != nil; e = errors.Unwrap(e) {
       fmt.Printf("Unwrapped: %v (%T)\n", e, e)
   }
   ```

2. **Testing error behavior**:
   ```go
   // Test that a function returns the expected error
   func TestFunction(t *testing.T) {
       _, err := function()
       if !errors.Is(err, ErrExpected) {
           t.Errorf("Expected ErrExpected, got %v", err)
       }
   }
   ```

3. **Tracking down panic sources**:
   ```go
   defer func() {
       if r := recover(); r != nil {
           fmt.Printf("Panic: %v\nStack trace:\n%s\n", r, debug.Stack())
       }
   }()
   ```

By addressing these common issues and following the patterns described in this module, you'll be well-equipped to implement robust error handling in your Go applications.
