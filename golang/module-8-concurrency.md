# Module 8: Concurrency

## Core Problem
Building concurrent software that efficiently uses multiple CPU cores for parallel execution while maintaining correctness, managing shared resources, and avoiding race conditions and deadlocks.

## Key Assumptions
- You understand basic Go syntax and functions
- You're familiar with Go's value and reference types
- You understand how functions and methods work in Go
- You have a basic understanding of what concurrency means

## Essential Concepts

### 1. Goroutines

**Concise Explanation:**
Goroutines are lightweight threads managed by the Go runtime. They enable concurrent execution with minimal resources (starting at around 2KB of stack space) and allow thousands of concurrent functions to run efficiently. Unlike OS threads, goroutines are multiplexed onto a smaller number of OS threads, with the Go scheduler handling their distribution and execution.

**Where to Use:**
- For concurrent I/O operations (network requests, file operations)
- For parallel computation across multiple cores
- To handle multiple client requests simultaneously
- When implementing background processing
- For tasks that can run independently of the main program flow
- In servers handling multiple connections

**Code Snippet:**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create a goroutine using the go keyword
    go sayHello("World")
    
    // Create multiple goroutines
    for i := 0; i < 5; i++ {
        // Capture loop variable to avoid closure issues
        i := i
        go func() {
            fmt.Printf("Goroutine %d\n", i)
        }()
    }
    
    // Alternative approach with parameter
    for i := 0; i < 5; i++ {
        go func(n int) {
            fmt.Printf("Goroutine %d\n", n)
        }(i)
    }
    
    // Main function continues execution without waiting
    fmt.Println("Main function")
    
    // Sleep to allow goroutines to execute
    // (In real code, use proper synchronization instead)
    time.Sleep(time.Second)
}

func sayHello(name string) {
    fmt.Printf("Hello, %s!\n", name)
}
```

**Real-World Example:**
A web server handling multiple client requests concurrently:

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "time"
)

func main() {
    // Define HTTP handler for incoming requests
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // Each request is processed in its own goroutine automatically
        fmt.Fprintf(w, "Processing request for: %s\n", r.URL.Path)
        
        // Simulate work
        time.Sleep(100 * time.Millisecond)
        
        fmt.Fprintf(w, "Request processed at: %s\n", time.Now().Format(time.RFC3339))
    })
    
    // Start server (blocking call)
    log.Println("Starting server on :8080")
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        log.Fatalf("Server failed: %v", err)
    }
}
```

**Common Pitfalls:**
- Creating too many goroutines, leading to resource exhaustion
- Not properly synchronizing access to shared data, causing race conditions
- Forgetting that the main function doesn't wait for goroutines to finish
- Capturing loop variables incorrectly in goroutines
- Not handling panics in goroutines, which can crash your program
- Leaking goroutines that never terminate, consuming resources
- Assuming that goroutines start execution immediately

**Confusion Questions:**

1. **Q: How many goroutines can I create in my Go program?**

   A: Go can handle a very large number of goroutines – easily hundreds of thousands or even millions on modern hardware. Unlike OS threads which might be limited to a few thousand, goroutines are lightweight:

   - Each goroutine initially uses **about 2KB of stack space** (which can grow as needed)
   - The Go runtime efficiently multiplexes goroutines onto OS threads
   - The practical limit depends on your system's available memory and the work each goroutine performs

   Here's a simple benchmark that creates a million goroutines:

   ```go
   package main

   import (
       "fmt"
       "runtime"
       "sync"
       "time"
   )

   func main() {
       // Print initial stats
       printStats("Before")
       
       var wg sync.WaitGroup
       
       // Create a million goroutines
       for i := 0; i < 1_000_000; i++ {
           wg.Add(1)
           go func() {
               defer wg.Done()
               time.Sleep(time.Millisecond) // Simulate some work
           }()
           
           // Print progress
           if i > 0 && i%100000 == 0 {
               fmt.Printf("Created %d goroutines...\n", i)
               printStats("Current")
           }
       }
       
       fmt.Println("Waiting for goroutines to finish...")
       wg.Wait()
       
       // Print final stats
       printStats("After")
   }

   func printStats(phase string) {
       var stats runtime.MemStats
       runtime.ReadMemStats(&stats)
       
       fmt.Printf("[%s] Goroutines: %d, Memory: %.2f MB\n", 
           phase, runtime.NumGoroutine(), float64(stats.Alloc)/1024/1024)
   }
   ```

   **Guidelines for goroutine usage:**

   1. **Consider using worker pools** to limit concurrent goroutines
   2. **Monitor resource usage** if your program creates many goroutines
   3. **Ensure proper synchronization** for all shared data
   4. **Always ensure goroutines will terminate** to avoid leaks
   5. **Consider the scheduler overhead** when creating very short-lived goroutines

   In most real-world applications, the limiting factor isn't the number of goroutines but how they're managed and synchronized.

2. **Q: What happens if a goroutine panics? Will it crash my entire program?**

   A: When a goroutine panics, by default it will print the panic message and stack trace, then terminate that specific goroutine. However, if the panic occurs in the main goroutine, it will terminate the entire program.

   Here's how panic propagation works:

   ```go
   func main() {
       // Panic in a goroutine
       go func() {
           fmt.Println("Goroutine starting...")
           panic("something went wrong") // This only terminates this goroutine
           // Code here never executes
       }()
       
       // Main continues executing
       time.Sleep(time.Second)
       fmt.Println("Main function still running")
   }
   ```

   **To handle panics in goroutines** and prevent them from crashing, use `recover()`:

   ```go
   func main() {
       go func() {
           // Recover must be called inside a deferred function
           defer func() {
               if r := recover(); r != nil {
                   fmt.Printf("Recovered from panic: %v\n", r)
                   // Log the stack trace for debugging
                   debug.PrintStack()
               }
           }()
           
           fmt.Println("Goroutine starting...")
           panic("something went wrong") // This panic will be recovered
           // Code here never executes
       }()
       
       // Main continues executing
       time.Sleep(time.Second)
       fmt.Println("Main function still running")
   }
   ```

   **Best practices for handling panics in goroutines:**

   1. **Always add recovery** for long-running goroutines
   2. **Log recoverable panics** with sufficient context for debugging
   3. **In servers**, add panic recovery middleware for all request handlers
   4. **Consider restarting** goroutines that panic, if appropriate
   5. **Use a global panic handler** pattern for consistent handling:

   ```go
   func SafeGo(fn func()) {
       go func() {
           defer func() {
               if r := recover(); r != nil {
                   log.Printf("Goroutine panic: %v\n%s", r, debug.Stack())
               }
           }()
           
           fn()
       }()
   }
   
   // Usage
   SafeGo(func() {
       // Do work that might panic
   })
   ```

   Remember that you should generally avoid letting panics happen in production code. Proper error handling is usually better than relying on panic recovery.

3. **Q: How does Go manage to run so many goroutines efficiently with just a few OS threads?**

   A: Go's efficient goroutine management comes from its runtime scheduler, which implements a form of M:N scheduling. This means it multiplexes M goroutines onto N OS threads. Here's how it works:

   **Go's scheduler uses three main components:**

   1. **G (Goroutine)**: A goroutine, representing a concurrent function execution
   2. **M (Machine)**: An OS thread (machine), which actually executes code
   3. **P (Processor)**: A logical processor that manages a queue of goroutines

   The scheduler follows these principles:

   **1. Work stealing design**:
   - When a P runs out of goroutines, it tries to steal goroutines from other Ps
   - This naturally balances work across processors

   **2. Non-blocking I/O operations**:
   - When a goroutine performs I/O, it's moved off the thread
   - Another goroutine is scheduled on that thread while waiting
   - When I/O completes, the goroutine is placed back in a queue

   **3. Preemptive scheduling**:
   - The scheduler can interrupt goroutines that run too long
   - This prevents a single CPU-bound goroutine from blocking others

   **4. Fast context switching**:
   - Goroutine context switches are much faster than OS thread switches
   - Only user-space registers need to be saved, not kernel state

   **Visualizing the scheduler:**

   ```
   OS Thread (M1)  OS Thread (M2)    OS Thread (M3)   ... (OS Thread MN)
        ↑               ↑                ↑                    ↑
   Processor (P1)   Processor (P2)   Processor (P3)   ... (Processor PN)
        ↑               ↑                ↑                    ↑
     Local Queue     Local Queue      Local Queue       ... Local Queue
     G1 G2 G3...     G4 G5 G6...     G7 G8 G9...          Gx Gy Gz...
                           ↑
                      Global Queue
                      GA GB GC...
   ```

   **Practical implications:**

   1. **GOMAXPROCS** controls the number of Ps (logical processors)
      ```go
      // Use all available CPU cores
      runtime.GOMAXPROCS(runtime.NumCPU())
      
      // Or set via environment variable: GOMAXPROCS=4
      ```

   2. **CPU-bound vs I/O-bound workloads**:
      - For CPU-bound work, you benefit from GOMAXPROCS ≈ number of cores
      - For I/O-bound work, many goroutines can efficiently share fewer threads

   3. **Blocking operations**:
      - Syscalls may block an OS thread temporarily
      - The runtime creates new threads when needed to keep CPU cores busy

   By understanding this model, you can see why Go can efficiently run thousands of goroutines on just a few OS threads, making it ideal for highly concurrent applications.

### 2. Synchronization with WaitGroups

**Concise Explanation:**
WaitGroup is a synchronization primitive in Go used to wait for a collection of goroutines to finish executing. It provides a simple counting mechanism: you increment the counter for each goroutine you start and decrement it when each goroutine completes. The main goroutine can then wait for the counter to reach zero, ensuring all goroutines have finished before proceeding.

**Where to Use:**
- When launching multiple goroutines and need to wait for all to complete
- For parallel processing tasks where results must be collected
- To ensure cleanup happens only after all goroutines are done
- In batch operations where you need to know when all work is complete
- For coordinating multiple independent tasks that must complete before proceeding
- In server shutdown procedures to wait for active requests to finish

**Code Snippet:**
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    // Create a new WaitGroup
    var wg sync.WaitGroup
    
    // Launch 5 goroutines
    for i := 1; i <= 5; i++ {
        // Increment the counter before launching goroutine
        wg.Add(1)
        
        // Launch the goroutine with the current value of i
        go worker(i, &wg)
    }
    
    // Alternative approach: add all at once
    // wg.Add(5)
    
    // Wait for all goroutines to finish
    fmt.Println("Waiting for all workers to finish...")
    wg.Wait()
    fmt.Println("All workers finished!")
}

func worker(id int, wg *sync.WaitGroup) {
    // Ensure the counter is decremented when the function returns
    defer wg.Done() // Same as wg.Add(-1)
    
    fmt.Printf("Worker %d starting\n", id)
    
    // Simulate work
    time.Sleep(time.Duration(id) * 200 * time.Millisecond)
    
    fmt.Printf("Worker %d done\n", id)
}
```

**Real-World Example:**
Processing multiple files concurrently:

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "path/filepath"
    "sync"
)

// FileDownloader downloads multiple files concurrently
type FileDownloader struct {
    TargetDir string
    MaxConcurrent int
}

// DownloadResult contains the result of a download operation
type DownloadResult struct {
    URL      string
    FilePath string
    Size     int64
    Error    error
}

// DownloadFiles downloads multiple files and waits for completion
func (d *FileDownloader) DownloadFiles(urls []string) []DownloadResult {
    var wg sync.WaitGroup
    results := make([]DownloadResult, len(urls))
    
    // Limit concurrency if needed
    semaphore := make(chan struct{}, d.MaxConcurrent)
    
    for i, url := range urls {
        wg.Add(1)
        
        go func(index int, fileURL string) {
            // Acquire semaphore slot
            semaphore <- struct{}{}
            
            // Ensure we release the semaphore and mark the task as done
            defer func() {
                <-semaphore   // Release semaphore
                wg.Done()     // Mark task as done
            }()
            
            // Process the file
            result := d.downloadFile(fileURL)
            results[index] = result
            
            log.Printf("Downloaded %s to %s (%d bytes)", 
                fileURL, result.FilePath, result.Size)
            
        }(i, url)
    }
    
    // Wait for all downloads to finish
    wg.Wait()
    return results
}

// downloadFile downloads a single file
func (d *FileDownloader) downloadFile(url string) DownloadResult {
    result := DownloadResult{URL: url}
    
    // Extract filename from URL
    filename := filepath.Base(url)
    filePath := filepath.Join(d.TargetDir, filename)
    result.FilePath = filePath
    
    // Create file
    out, err := os.Create(filePath)
    if err != nil {
        result.Error = fmt.Errorf("creating file: %w", err)
        return result
    }
    defer out.Close()
    
    // Download the file
    resp, err := http.Get(url)
    if err != nil {
        result.Error = fmt.Errorf("downloading: %w", err)
        return result
    }
    defer resp.Body.Close()
    
    // Check server response
    if resp.StatusCode != http.StatusOK {
        result.Error = fmt.Errorf("bad response: %s", resp.Status)
        return result
    }
    
    // Copy data
    size, err := io.Copy(out, resp.Body)
    if err != nil {
        result.Error = fmt.Errorf("saving: %w", err)
        return result
    }
    
    result.Size = size
    return result
}

func main() {
    // Create download directory
    if err := os.MkdirAll("downloads", 0755); err != nil {
        log.Fatalf("Failed to create download directory: %v", err)
    }
    
    // Initialize downloader
    downloader := FileDownloader{
        TargetDir:     "downloads",
        MaxConcurrent: 3, // Limit to 3 concurrent downloads
    }
    
    // Sample URLs
    urls := []string{
        "https://example.com/file1.txt",
        "https://example.com/file2.txt",
        "https://example.com/file3.txt",
        "https://example.com/file4.txt",
        "https://example.com/file5.txt",
    }
    
    // Download files
    results := downloader.DownloadFiles(urls)
    
    // Report results
    fmt.Println("\nDownload Results:")
    for _, r := range results {
        if r.Error != nil {
            fmt.Printf("Failed: %s - Error: %v\n", r.URL, r.Error)
        } else {
            fmt.Printf("Success: %s - Saved to: %s (%d bytes)\n", 
                r.URL, r.FilePath, r.Size)
        }
    }
}
```

**Common Pitfalls:**
- Forgetting to call `wg.Done()`, leading to deadlocks
- Not passing the WaitGroup as a pointer when needed in functions
- Calling `wg.Add()` after the goroutine has started (may cause undercounting)
- Calling `wg.Done()` more times than `wg.Add()`, causing a panic
- Not using `defer wg.Done()` to ensure it gets called even if there's a panic
- Overusing WaitGroups for complex synchronization scenarios
- Reusing a WaitGroup before previous `Wait()` has returned

**Confusion Questions:**

1. **Q: Why does WaitGroup need to be passed as a pointer to functions?**

   A: WaitGroup must be passed as a pointer to functions because its methods modify its internal counter state. If you pass it by value (as a copy), the `Done()` method would decrement the counter of the copy, not the original WaitGroup, causing the main goroutine to wait forever.

   **Example of incorrect usage:**
   ```go
   func main() {
       var wg sync.WaitGroup
       wg.Add(1)
       
       // Passing WaitGroup by value (copy) - WRONG
       go worker(wg)
       
       wg.Wait() // Will wait forever - the original wg is never decremented
   }
   
   func worker(wg sync.WaitGroup) {
       defer wg.Done() // Decrements the copied WaitGroup, not the original
       // Do work...
   }
   ```

   **Correct usage:**
   ```go
   func main() {
       var wg sync.WaitGroup
       wg.Add(1)
       
       // Passing WaitGroup by pointer - CORRECT
       go worker(&wg)
       
       wg.Wait() // Will properly wait for worker to call Done()
   }
   
   func worker(wg *sync.WaitGroup) {
       defer wg.Done() // Decrements the original WaitGroup
       // Do work...
   }
   ```

   **The problem is similar to this simplified example:**
   ```go
   func main() {
       x := 10
       increment(x)    // Passes a copy of x
       fmt.Println(x)  // Still prints 10, not 11
       
       increment2(&x)  // Passes a pointer to x
       fmt.Println(x)  // Prints 11
   }
   
   func increment(n int) {
       n++ // Modifies the copy, not the original
   }
   
   func increment2(n *int) {
       *n++ // Modifies the original via the pointer
   }
   ```

   When designing functions that accept a WaitGroup:

   1. **Always use a pointer parameter**: `func worker(wg *sync.WaitGroup)`
   2. **Consider making WaitGroup a field** in a struct if many functions need it
   3. **Document clearly** that the function calls `wg.Done()` when complete
   4. **Use defer for wg.Done()** to ensure it's called even on early returns or panics

2. **Q: Can I reuse a WaitGroup after calling Wait() on it?**

   A: Yes, you can reuse a WaitGroup after calling `Wait()` on it, but you must ensure that `Wait()` has returned before reusing it. This means all previously added goroutines must have called `Done()`.

   **Safe reuse pattern:**
   ```go
   func main() {
       var wg sync.WaitGroup
       
       // First batch of goroutines
       for i := 0; i < 3; i++ {
           wg.Add(1)
           go func(n int) {
               defer wg.Done()
               fmt.Printf("First batch: %d\n", n)
           }(i)
       }
       
       // Wait for first batch to complete
       wg.Wait()
       fmt.Println("First batch done")
       
       // Second batch of goroutines (reusing WaitGroup)
       for i := 0; i < 3; i++ {
           wg.Add(1)
           go func(n int) {
               defer wg.Done()
               fmt.Printf("Second batch: %d\n", n)
           }(i)
       }
       
       // Wait for second batch to complete
       wg.Wait()
       fmt.Println("Second batch done")
   }
   ```

   **Unsafe reuse pattern (potential race condition):**
   ```go
   func main() {
       var wg sync.WaitGroup
       
       // First batch
       for i := 0; i < 3; i++ {
           wg.Add(1)
           go func(n int) {
               defer wg.Done()
               fmt.Printf("Task: %d\n", n)
           }(i)
       }
       
       // UNSAFE: Adding more before waiting
       // May cause race condition if first batch finishes quickly
       for i := 3; i < 6; i++ {
           wg.Add(1)
           go func(n int) {
               defer wg.Done()
               fmt.Printf("Task: %d\n", n)
           }(i)
       }
       
       // Wait for all goroutines
       wg.Wait()
   }
   ```

   **Best practices for WaitGroup reuse:**

   1. **Ensure clean separation** between usage cycles
   2. **Always call `Wait()` before reusing** the WaitGroup
   3. **Consider using separate WaitGroups** for unrelated goroutine groups
   4. **For complex workflows**, consider using a more advanced synchronization primitive or a state machine
   5. **When in doubt**, create a new WaitGroup instance

   In production code, if you need complex reuse patterns, it's often clearer to encapsulate WaitGroup usage in a dedicated type with appropriate methods.

3. **Q: What happens if I call wg.Add() after the goroutines have started or if I forget to call wg.Done()?**

   A: Both scenarios can cause serious synchronization issues, including deadlocks and panics:

   **Scenario 1: Calling `wg.Add()` after the goroutines have started**

   ```go
   func main() {
       var wg sync.WaitGroup
       
       // Starting goroutines WITHOUT incrementing the counter first
       for i := 0; i < 5; i++ {
           go func(n int) {
               defer wg.Done() // Will decrement a counter that may not be set
               fmt.Printf("Working on %d\n", n)
               time.Sleep(time.Millisecond * 100)
           }(i)
       }
       
       // Adding to the counter AFTER launching goroutines
       wg.Add(5) // WRONG: goroutines might have already finished!
       
       wg.Wait()
   }
   ```

   **Problems with this approach:**
   - If any goroutine completes and calls `wg.Done()` before `wg.Add(5)`, the final counter will be incorrect
   - In extreme cases, if all goroutines finish before `wg.Add(5)`, `wg.Done()` calls will cause a panic
   - The program may appear to work sometimes due to timing, making the bug difficult to find

   **Scenario 2: Forgetting to call `wg.Done()`**

   ```go
   func main() {
       var wg sync.WaitGroup
       
       for i := 0; i < 5; i++ {
           wg.Add(1)
           go func(n int) {
               fmt.Printf("Working on %d\n", n)
               // Missing wg.Done() here!
           }(i)
       }
       
       wg.Wait() // Will wait forever
   }
   ```

   **Problems with this approach:**
   - The program will deadlock because `wg.Wait()` will block forever
   - In Go's runtime, this deadlock will be detected with a message like `fatal error: all goroutines are asleep - deadlock!`

   **Best practices to avoid these issues:**

   1. **Always increment the counter before starting the goroutine**:
      ```go
      wg.Add(1)
      go func() {
          defer wg.Done()
          // Work here
      }()
      ```

   2. **Use `defer wg.Done()` as the first statement in the goroutine**:
      ```go
      go func() {
          defer wg.Done() // Ensures it's called even if the goroutine panics
          // Work here that might panic or return early
      }()
      ```

   3. **For batch operations, add to the counter once**:
      ```go
      taskCount := len(tasks)
      wg.Add(taskCount) // Add exact count once before launching any goroutines
      
      for _, task := range tasks {
          go func(t Task) {
              defer wg.Done()
              process(t)
          }(task)
      }
      ```

   4. **Consider encapsulating WaitGroup usage in helper functions**:
      ```go
      func runConcurrently(tasks []Task, fn func(Task)) {
          var wg sync.WaitGroup
          wg.Add(len(tasks))
          
          for _, task := range tasks {
              task := task // Capture for closure
              go func() {
                  defer wg.Done()
                  fn(task)
              }()
          }
          
          wg.Wait()
      }
      ```

   Following these practices will help you avoid subtle synchronization bugs that can be difficult to debug.

### 3. Channel Basics

**Concise Explanation:**
Channels are typed conduits for communication and synchronization between goroutines. They allow safe data transfer between concurrent processes, avoiding race conditions when accessing shared data. A channel can be used to send and receive values of a specific type, with operations that block until another goroutine is ready to send or receive, creating natural synchronization points.

**Where to Use:**
- To communicate between goroutines safely
- To synchronize execution between goroutines
- For signaling events (like completion or errors)
- To implement producer-consumer patterns
- To distribute work among multiple goroutines
- For implementing timeouts and cancellation
- As an alternative to shared memory and locks

**Code Snippet:**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create an unbuffered channel of integers
    ch := make(chan int)
    
    // Send values in a goroutine
    go func() {
        fmt.Println("Sending values...")
        
        for i := 1; i <= 5; i++ {
            fmt.Printf("Sending: %d\n", i)
            ch <- i // Send blocks until another goroutine receives
            fmt.Printf("Sent: %d\n", i)
        }
        
        close(ch) // Close channel when done sending
        fmt.Println("Sender finished")
    }()
    
    // Receive values in main goroutine
    fmt.Println("Receiving values...")
    
    // Method 1: Receive until channel is closed
    for value := range ch {
        fmt.Printf("Received: %d\n", value)
        time.Sleep(time.Millisecond * 100) // Simulate processing time
    }
    
    // Method 2: Check if channel is closed
    ch2 := make(chan int)
    go sendValues(ch2)
    
    for {
        value, ok := <-ch2
        if !ok {
            // Channel is closed
            break
        }
        fmt.Printf("Received from ch2: %d\n", value)
    }
    
    fmt.Println("All done!")
}

func sendValues(ch chan int) {
    for i := 10; i <= 15; i++ {
        ch <- i
    }
    close(ch)
}
```

**Real-World Example:**
A concurrent web scraper that fetches multiple pages and processes them:

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "strings"
    "sync"
    "time"
)

// Page represents a fetched web page
type Page struct {
    URL      string
    Content  string
    Links    []string
    Error    error
    FetchedAt time.Time
}

// Fetcher manages concurrent page fetching
func Fetcher(urls []string, concurrency int) []Page {
    // Create channels
    urlCh := make(chan string)      // For distributing URLs
    resultCh := make(chan Page)     // For collecting results
    
    // Start worker goroutines
    var wg sync.WaitGroup
    for i := 0; i < concurrency; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            for url := range urlCh {
                // Fetch the page
                page := fetchPage(url)
                
                // Send result back
                resultCh <- page
            }
            
            fmt.Printf("Worker %d finished\n", id)
        }(i)
    }
    
    // Close the result channel once all workers are done
    go func() {
        wg.Wait()
        close(resultCh)
        fmt.Println("All workers finished")
    }()
    
    // Start sending URLs to workers
    go func() {
        for _, url := range urls {
            urlCh <- url
        }
        close(urlCh) // No more URLs to process
        fmt.Println("All URLs sent to workers")
    }()
    
    // Collect results
    var results []Page
    for page := range resultCh {
        results = append(results, page)
        fmt.Printf("Received result for: %s (Error: %v)\n", 
            page.URL, page.Error != nil)
    }
    
    return results
}

// fetchPage downloads a page and extracts its links
func fetchPage(url string) Page {
    page := Page{
        URL:       url,
        FetchedAt: time.Now(),
    }
    
    // Make HTTP request
    resp, err := http.Get(url)
    if err != nil {
        page.Error = err
        return page
    }
    defer resp.Body.Close()
    
    // Check status code
    if resp.StatusCode != http.StatusOK {
        page.Error = fmt.Errorf("bad status: %d", resp.StatusCode)
        return page
    }
    
    // Read body
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        page.Error = err
        return page
    }
    
    // Store content
    page.Content = string(body)
    
    // Extract links (simplified)
    page.Links = extractLinks(page.Content)
    
    return page
}

// extractLinks finds URLs in HTML content (simplified)
func extractLinks(content string) []string {
    var links []string
    
    // Very basic link extraction (not for production use)
    startIdx := 0
    for {
        hrefStart := strings.Index(content[startIdx:], "href=\"")
        if hrefStart == -1 {
            break
        }
        
        hrefStart += startIdx + 6 // "href=" plus quote
        hrefEnd := strings.Index(content[hrefStart:], "\"")
        if hrefEnd == -1 {
            break
        }
        
        link := content[hrefStart : hrefStart+hrefEnd]
        if strings.HasPrefix(link, "http") {
            links = append(links, link)
        }
        
        startIdx = hrefStart + hrefEnd
    }
    
    return links
}

func main() {
    // List of URLs to fetch
    urls := []string{
        "https://example.com",
        "https://example.org",
        "https://example.net",
        "https://example.edu",
    }
    
    fmt.Println("Starting concurrent fetcher...")
    results := Fetcher(urls, 2) // Use 2 concurrent workers
    
    // Print summary
    fmt.Printf("\nResults Summary:\n")
    fmt.Printf("----------------\n")
    for _, page := range results {
        if page.Error != nil {
            fmt.Printf("❌ %s - Error: %v\n", page.URL, page.Error)
        } else {
            fmt.Printf("✓ %s - %d bytes, %d links\n", 
                page.URL, len(page.Content), len(page.Links))
        }
    }
}
```

**Common Pitfalls:**
- Deadlocks caused by blocking operations without corresponding receivers
- Forgetting to close channels when done sending
- Sending on a closed channel, which causes a panic
- Forgetting that a nil channel blocks forever
- Leaking goroutines that are stuck trying to send or receive
- Not handling the "channel closed" signal correctly
- Using channels where simpler synchronization would be appropriate

**Confusion Questions:**

1. **Q: What's the difference between buffered and unbuffered channels?**

   A: Buffered and unbuffered channels differ in how they handle sends and receives, which affects the synchronization behavior between goroutines.

   **Unbuffered Channels:**

   ```go
   // Create an unbuffered channel
   ch := make(chan int)
   ```

   **Key characteristics:**
   - **Synchronous**: A send operation blocks until another goroutine receives the value
   - **Direct handoff**: Values are directly handed from sender to receiver
   - **Strong coordination**: Both sender and receiver must be ready for the transfer to occur
   - **Zero capacity**: Cannot store values; each send must have a corresponding receive

   **Buffered Channels:**

   ```go
   // Create a buffered channel with capacity 3
   ch := make(chan int, 3)
   ```

   **Key characteristics:**
   - **Asynchronous** up to buffer capacity: Sender only blocks when buffer is full
   - **Temporary storage**: Can hold values until a receiver is ready
   - **Looser coordination**: Sender and receiver don't have to operate at the same time
   - **Specified capacity**: Can store a limited number of values

   **Behavior comparison:**

   ```go
   // Unbuffered channel example
   ch1 := make(chan int)
   
   go func() {
       fmt.Println("Sender: About to send")
       ch1 <- 42        // Blocks until someone receives
       fmt.Println("Sender: Value sent") // Prints after receiver gets value
   }()
   
   time.Sleep(time.Second) // Simulate delay before receiving
   fmt.Println("Receiver: About to receive")
   v := <-ch1
   fmt.Println("Receiver: Got", v)
   
   // Buffered channel example
   ch2 := make(chan int, 2)
   
   fmt.Println("Sender: Sending values to buffer")
   ch2 <- 1  // Doesn't block - goes into buffer
   ch2 <- 2  // Doesn't block - goes into buffer
   fmt.Println("Sender: Values in buffer")
   
   // This would block because buffer is full
   // ch2 <- 3
   
   fmt.Println("Receiver: Getting first value")
   v1 := <-ch2
   fmt.Println("Receiver: Got", v1)
   
   // Now buffer has space again
   fmt.Println("Sender: Sending one more")
   ch2 <- 3  // Doesn't block - buffer has space
   
   fmt.Println("Receiver: Getting remaining values")
   v2 := <-ch2
   v3 := <-ch2
   fmt.Println("Receiver: Got", v2, "and", v3)
   ```

   **When to use each type:**

   **Use unbuffered channels when:**
   - You need guaranteed delivery (every send has a corresponding receive)
   - You need synchronization between operations
   - You want "rendezvous" points where goroutines meet
   - You need strict handoff of data with confirmation

   **Use buffered channels when:**
   - You want to decouple send and receive operations
   - You need to handle temporary mismatch in send/receive rates
   - You want to limit the amount of work in progress
   - You need a producer to enqueue multiple items before a consumer starts processing

   **Capacity sizing for buffered channels:**

   The buffer size should generally be based on:
   1. Expected burst rate of the producer
   2. Processing rate of the consumer
   3. Acceptable latency or memory usage

   In practice, common buffer sizes are powers of 2 (1, 2, 4, 8, 16, etc.) or match the number of workers/CPUs in the system.

2. **Q: What happens if I try to send to or receive from a closed channel?**

   A: Operations on closed channels have specific behaviors that are important to understand:

   **1. Sending to a closed channel:**

   ```go
   ch := make(chan int, 1)
   close(ch)
   
   ch <- 5  // This will PANIC: "send on closed channel"
   ```

   - **Result**: Always causes a runtime panic
   - **Fix**: Ensure the channel is open before sending, or use a mutex to protect the close operation

   **2. Receiving from a closed channel:**

   ```go
   ch := make(chan int, 1)
   ch <- 42     // Put one value in the channel
   close(ch)    // Close the channel
   
   // Successful receive - gets the buffered value
   value1 := <-ch    // value1 = 42
   
   // Receiving again after channel is empty
   value2 := <-ch    // value2 = 0 (zero value for int)
   
   // Checking if channel is closed
   value3, ok := <-ch    // value3 = 0, ok = false
   
   if !ok {
       fmt.Println("Channel is closed and empty")
   }
   ```

   - **Result**: Returns the zero value for the channel type
   - **Check**: Use the two-value form `value, ok := <-ch` to detect if a channel is closed
   - **Usage**: The `for range` loop automatically stops when the channel is closed

   **3. Closing a nil channel:**

   ```go
   var ch chan int  // nil channel
   close(ch)        // This will PANIC: "close of nil channel"
   ```

   - **Result**: Causes a runtime panic
   - **Fix**: Always check that a channel is not nil before closing it

   **4. Closing a channel more than once:**

   ```go
   ch := make(chan int)
   close(ch)
   close(ch)  // This will PANIC: "close of closed channel"
   ```

   - **Result**: Causes a runtime panic
   - **Fix**: Track channel state or use a sync.Once for closing

   **Best practices for working with closed channels:**

   **1. Establish clear ownership**
   ```go
   // Typically, the sender should own and close the channel
   func producer(ch chan<- int) {
       for i := 0; i < 5; i++ {
           ch <- i
       }
       close(ch) // Producer closes when done sending
   }
   
   func consumer(ch <-chan int) {
       // Range automatically handles closed channel
       for v := range ch {
           fmt.Println(v)
       }
   }
   ```

   **2. Use a done channel for signaling**
   ```go
   done := make(chan struct{})
   
   // Signal completion
   close(done) // Closing is the signal
   
   // Check for completion
   select {
   case <-done:
       // Done was signaled
   default:
       // Not done yet
   }
   ```

   **3. Safe channel closing pattern**
   ```go
   type safeChannel struct {
       ch chan int
       mu sync.Mutex
       closed bool
   }
   
   func (sc *safeChannel) Send(value int) error {
       sc.mu.Lock()
       defer sc.mu.Unlock()
       
       if sc.closed {
           return errors.New("channel closed")
       }
       
       sc.ch <- value
       return nil
   }
   
   func (sc *safeChannel) Close() {
       sc.mu.Lock()
       defer sc.mu.Unlock()
       
       if !sc.closed {
           close(sc.ch)
           sc.closed = true
       }
   }
   ```

   **4. Using context for cancellation**
   ```go
   ctx, cancel := context.WithCancel(context.Background())
   
   // Instead of managing channel closing directly
   go func() {
       defer cancel() // Signal done
       // Do work
   }()
   
   // Check for done
   select {
   case <-ctx.Done():
       // Work is done or cancelled
   default:
       // Still working
   }
   ```

   Understanding these behaviors is essential for writing correct concurrent programs in Go and avoiding common pitfalls with channel operations.

3. **Q: How do I know when to use channels versus mutexes for concurrency?**

   A: Choosing between channels and mutexes depends on your concurrent design needs. Here's a decision framework to help you choose the right tool:

   **Use channels when:**

   1. **Communicating between goroutines**
      ```go
      // Worker sends results back to coordinator
      resultCh := make(chan Result)
      go func() {
          result := process()
          resultCh <- result
      }()
      result := <-resultCh
      ```

   2. **Signaling events or state changes**
      ```go
      done := make(chan struct{})
      go func() {
          // Do work...
          close(done) // Signal completion
      }()
      <-done // Wait for work to complete
      ```

   3. **Distributing work**
      ```go
      tasks := make(chan Task, 100)
      for i := 0; i < 10; i++ {
          go worker(tasks)
      }
      // Send tasks to workers
      for _, t := range allTasks {
          tasks <- t
      }
      ```

   4. **Controlling resource access with limiting**
      ```go
      // Semaphore pattern
      sem := make(chan struct{}, maxConcurrent)
      for _, task := range tasks {
          sem <- struct{}{} // Acquire
          go func(t Task) {
              defer func() { <-sem }() // Release
              process(t)
          }(task)
      }
      ```

   5. **Timeouts and cancellation**
      ```go
      select {
      case result := <-resultCh:
          // Process result
      case <-time.After(5 * time.Second):
          // Handle timeout
      case <-ctx.Done():
          // Handle cancellation
      }
      ```

   **Use mutexes when:**

   1. **Protecting shared state**
      ```go
      var (
          counter int
          mu      sync.Mutex
      )
      
      func increment() {
          mu.Lock()
          counter++
          mu.Unlock()
      }
      ```

   2. **Multiple readers, occasional writers (use RWMutex)**
      ```go
      var (
          data map[string]string
          mu   sync.RWMutex
      )
      
      func read(key string) string {
          mu.RLock()
          defer mu.RUnlock()
          return data[key]
      }
      
      func write(key, value string) {
          mu.Lock()
          defer mu.Unlock()
          data[key] = value
      }
      ```

   3. **Simple, short-lived critical sections**
      ```go
      var (
          items []string
          mu    sync.Mutex
      )
      
      func addItem(item string) {
          mu.Lock()
          items = append(items, item)
          mu.Unlock()
      }
      ```

   4. **Updating multiple related variables atomically**
      ```go
      type Counter struct {
          mu         sync.Mutex
          count      int
          lastUpdated time.Time
      }
      
      func (c *Counter) Increment() {
          c.mu.Lock()
          defer c.mu.Unlock()
          
          c.count++
          c.lastUpdated = time.Now()
      }
      ```

   **Decision framework:**

   Ask these questions to choose between channels and mutexes:

   1. **Are you transferring data ownership?**
      - **Yes**: Use channels to safely hand off data
      - **No**: Consider mutexes for shared access

   2. **Is communication/coordination the primary goal?**
      - **Yes**: Channels make message passing explicit
      - **No**: Mutexes might be simpler for just protection

   3. **Do you need to coordinate sequences of operations?**
      - **Yes**: Channels provide natural sequencing
      - **No**: Mutexes protect isolated operations

   4. **Does one goroutine own the data at a time?**
      - **Yes**: Channels enforce this ownership model
      - **No**: Mutexes allow multiple goroutines shared access

   5. **Do you need complex synchronization patterns?**
      - **Yes**: Select with channels can handle complex cases
      - **No**: Simple mutex might be clearer

   **Combined approach example:**

   Sometimes the best approach combines both techniques:

   ```go
   type Worker struct {
       tasks   chan Task
       results chan Result
       data    map[string]interface{}
       mu      sync.RWMutex
   }
   
   func (w *Worker) Start() {
       go func() {
           for task := range w.tasks {
               // Use mutex for accessing shared state
               w.mu.RLock()
               value := w.data[task.Key]
               w.mu.RUnlock()
               
               // Process with value
               result := process(task, value)
               
               // Use channel to communicate result
               w.results <- result
               
               // Update shared state if needed
               if result.UpdateNeeded {
                   w.mu.Lock()
                   w.data[task.Key] = result.NewValue
                   w.mu.Unlock()
               }
           }
       }()
   }
   ```

   **"Go proverb" guidance:**

   *"Don't communicate by sharing memory; share memory by communicating."*

   This suggests preferring channels for communication between goroutines, but pragmatic Go code often uses both mechanisms, choosing the right tool for each synchronization need.

### 4. Buffered vs. Unbuffered Channels

**Concise Explanation:**
Buffered channels have a capacity for storing values, allowing senders to proceed without an immediate receiver up to the buffer limit. Unbuffered channels have no capacity and synchronize by requiring both a sender and receiver to be ready simultaneously. This fundamental difference affects the synchronization characteristics: unbuffered channels provide strong synchronization guarantees with direct handoffs, while buffered channels decouple senders and receivers for more flexible timing.

**Where to Use:**
- **Unbuffered Channels (make(chan T)):**
  - When precise coordination between goroutines is needed
  - For guaranteed delivery and processing of each message
  - In request-response patterns where each request must be processed
  - When the sender needs confirmation that the receiver has processed the value
  - For implementing synchronization points ("rendezvous")

- **Buffered Channels (make(chan T, size)):**
  - When sender and receiver should operate independently
  - To handle bursts of values (e.g., in producer-consumer patterns)
  - To implement throttling and rate limiting
  - When performance is critical and some decoupling is acceptable
  - For batching or queuing operations

**Code Snippet:**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Unbuffered channel example
    fmt.Println("Demonstrating unbuffered channel:")
    unbufferedDemo()
    
    // Buffered channel example
    fmt.Println("\nDemonstrating buffered channel:")
    bufferedDemo()
    
    // Practical comparison
    fmt.Println("\nComparing performance:")
    comparePerformance()
}

func unbufferedDemo() {
    ch := make(chan int) // Unbuffered channel
    
    go func() {
        fmt.Println("Sender: Ready to send")
        fmt.Println("Sender: Sending 42")
        start := time.Now()
        ch <- 42 // Will block until there's a receiver
        fmt.Printf("Sender: Sent after %v\n", time.Since(start))
        
        fmt.Println("Sender: Sending 43")
        ch <- 43
        fmt.Println("Sender: Sent 43")
    }()
    
    // Small delay to ensure the goroutine starts
    time.Sleep(100 * time.Millisecond)
    
    fmt.Println("Receiver: Waiting 1 second before receiving")
    time.Sleep(1 * time.Second)
    
    fmt.Println("Receiver: Receiving first value")
    value := <-ch
    fmt.Printf("Receiver: Got %d\n", value)
    
    fmt.Println("Receiver: Receiving second value")
    value = <-ch
    fmt.Printf("Receiver: Got %d\n", value)
}

func bufferedDemo() {
    ch := make(chan int, 2) // Buffered channel with capacity 2
    
    go func() {
        fmt.Println("Sender: Ready to send")
        fmt.Println("Sender: Sending 42")
        start := time.Now()
        ch <- 42 // Won't block as buffer has space
        fmt.Printf("Sender: Sent 42 after %v\n", time.Since(start))
        
        fmt.Println("Sender: Sending 43")
        ch <- 43 // Still won't block
        fmt.Printf("Sender: Sent 43\n")
        
        fmt.Println("Sender: Sending 44")
        start = time.Now()
        ch <- 44 // Will block as buffer is full
        fmt.Printf("Sender: Sent 44 after %v\n", time.Since(start))
    }()
    
    // Small delay to ensure the goroutine starts
    time.Sleep(100 * time.Millisecond)
    
    fmt.Println("Receiver: Waiting 1 second before receiving")
    time.Sleep(1 * time.Second)
    
    fmt.Println("Receiver: Receiving first value")
    value := <-ch
    fmt.Printf("Receiver: Got %d\n", value)
    
    fmt.Println("Receiver: Receiving second value")
    value = <-ch
    fmt.Printf("Receiver: Got %d\n", value)
    
    fmt.Println("Receiver: Receiving third value")
    value = <-ch
    fmt.Printf("Receiver: Got %d\n", value)
}

func comparePerformance() {
    dataSize := 1000
    
    // Unbuffered channel
    fmt.Println("Unbuffered channel:")
    unbufferedCh := make(chan int)
    start := time.Now()
    
    go func() {
        for i := 0; i < dataSize; i++ {
            unbufferedCh <- i
        }
        close(unbufferedCh)
    }()
    
    // Receive all values
    count := 0
    for range unbufferedCh {
        count++
    }
    
    fmt.Printf("Processed %d items in %v\n", count, time.Since(start))
    
    // Buffered channel
    fmt.Println("Buffered channel:")
    bufferedCh := make(chan int, 100) // Buffer size 100
    start = time.Now()
    
    go func() {
        for i := 0; i < dataSize; i++ {
            bufferedCh <- i
        }
        close(bufferedCh)
    }()
    
    // Receive all values
    count = 0
    for range bufferedCh {
        count++
    }
    
    fmt.Printf("Processed %d items in %v\n", count, time.Since(start))
}
```

**Real-World Example:**
A task scheduler that uses both buffered and unbuffered channels:

```go
package taskscheduler

import (
    "context"
    "fmt"
    "log"
    "sync"
    "time"
)

// Task represents a unit of work
type Task struct {
    ID          string
    Description string
    ExecuteFunc func() (interface{}, error)
    Priority    int // Higher value means higher priority
}

// Result represents the outcome of a task
type Result struct {
    TaskID     string
    Value      interface{}
    Err        error
    StartTime  time.Time
    FinishTime time.Time
}

// TaskScheduler manages the execution of tasks with different priorities
type TaskScheduler struct {
    workers          int
    highPriorityQ    chan Task      // Buffered - for high priority tasks
    normalPriorityQ  chan Task      // Buffered - for normal priority tasks
    resultCh         chan Result    // Buffered - for collecting results
    controlCh        chan struct{}  // Unbuffered - for precise control operations
    shutdownCh       chan struct{}  // Unbuffered - for synchronized shutdown
    wg               sync.WaitGroup
    ctx              context.Context
    cancel           context.CancelFunc
}

// NewTaskScheduler creates a new task scheduler
func NewTaskScheduler(workers int, highPrioritySize, normalPrioritySize, resultSize int) *TaskScheduler {
    ctx, cancel := context.WithCancel(context.Background())
    
    return &TaskScheduler{
        workers:          workers,
        highPriorityQ:    make(chan Task, highPrioritySize),  // Buffered - allows queuing
        normalPriorityQ:  make(chan Task, normalPrioritySize), // Buffered - allows queuing
        resultCh:         make(chan Result, resultSize),       // Buffered - allows collection
        controlCh:        make(chan struct{}),                // Unbuffered - for synchronization
        shutdownCh:       make(chan struct{}),                // Unbuffered - for synchronization
        ctx:              ctx,
        cancel:           cancel,
    }
}

// Start begins task processing
func (ts *TaskScheduler) Start() {
    log.Printf("Starting task scheduler with %d workers\n", ts.workers)
    
    // Start workers
    for i := 0; i < ts.workers; i++ {
        ts.wg.Add(1)
        go ts.worker(i)
    }
}

// worker processes tasks from the queues
func (ts *TaskScheduler) worker(id int) {
    defer ts.wg.Done()
    log.Printf("Worker %d starting\n", id)
    
    for {
        select {
        case <-ts.ctx.Done():
            log.Printf("Worker %d shutting down due to context cancellation\n", id)
            return
            
        // Check high priority queue first (priority scheduling)
        case task, ok := <-ts.highPriorityQ:
            if !ok {
                // High priority channel was closed
                continue
            }
            ts.executeTask(id, task)
            
        // Then check normal priority queue
        case task, ok := <-ts.normalPriorityQ:
            if !ok {
                // Normal priority channel was closed
                continue
            }
            
            // Before executing, check if a high priority task arrived
            select {
            case highPriorityTask, ok := <-ts.highPriorityQ:
                if ok {
                    // Push back the normal task if possible
                    select {
                    case ts.normalPriorityQ <- task:
                        // Successfully pushed back
                    default:
                        // Queue full, execute normal task first
                        ts.executeTask(id, task)
                        ts.executeTask(id, highPriorityTask)
                        continue
                    }
                    // Execute the high priority task first
                    ts.executeTask(id, highPriorityTask)
                } else {
                    // High priority channel was closed
                    ts.executeTask(id, task)
                }
            default:
                // No high priority task, execute the normal task
                ts.executeTask(id, task)
            }
            
        // Check for control signals - unbuffered for immediate action
        case <-ts.controlCh:
            log.Printf("Worker %d received control signal\n", id)
            // Perform maintenance or special actions
            ts.controlCh <- struct{}{} // Acknowledge receipt
        }
    }
}

// executeTask runs a task and sends its result
func (ts *TaskScheduler) executeTask(workerID int, task Task) {
    log.Printf("Worker %d executing task %s\n", workerID, task.ID)
    
    result := Result{
        TaskID:    task.ID,
        StartTime: time.Now(),
    }
    
    // Execute the task function
    value, err := task.ExecuteFunc()
    
    result.Value = value
    result.Err = err
    result.FinishTime = time.Now()
    
    // Send result, but don't block indefinitely
    select {
    case ts.resultCh <- result:
        // Successfully sent result
    case <-ts.ctx.Done():
        // Context cancelled, discard result
    default:
        // Result channel is full
        log.Printf("WARNING: Result channel full, discarding result for task %s\n", task.ID)
    }
}

// SubmitTask adds a task to the appropriate queue
func (ts *TaskScheduler) SubmitTask(task Task) error {
    // Check context to make sure we're not shutting down
    if ts.ctx.Err() != nil {
        return fmt.Errorf("scheduler is shutting down")
    }
    
    // Route to appropriate queue based on priority
    if task.Priority > 5 { // Arbitrary threshold
        // Try to send to high priority queue
        select {
        case ts.highPriorityQ <- task:
            log.Printf("Task %s added to high priority queue\n", task.ID)
            return nil
        default:
            return fmt.Errorf("high priority queue is full")
        }
    } else {
        // Try to send to normal priority queue
        select {
        case ts.normalPriorityQ <- task:
            log.Printf("Task %s added to normal priority queue\n", task.ID)
            return nil
        default:
            return fmt.Errorf("normal priority queue is full")
        }
    }
}

// Results returns a channel for receiving results
func (ts *TaskScheduler) Results() <-chan Result {
    return ts.resultCh
}

// Control sends a control signal and waits for acknowledgment
// Uses unbuffered channel for direct synchronization
func (ts *TaskScheduler) Control() error {
    select {
    case ts.controlCh <- struct{}{}:
        // Wait for acknowledgment
        select {
        case <-ts.controlCh:
            log.Println("Control signal acknowledged")
            return nil
        case <-time.After(5 * time.Second):
            return fmt.Errorf("control signal acknowledgment timed out")
        }
    case <-time.After(1 * time.Second):
        return fmt.Errorf("could not send control signal, workers might be busy")
    }
}

// Shutdown stops the task scheduler and waits for workers to finish
func (ts *TaskScheduler) Shutdown() {
    log.Println("Shutting down task scheduler...")
    
    // Cancel context to signal shutdown
    ts.cancel()
    
    // Signal all workers using unbuffered channel for synchronization
    log.Println("Sending shutdown signal")
    close(ts.shutdownCh)
    
    // Wait for workers to finish
    ts.wg.Wait()
    
    // Close channels
    close(ts.highPriorityQ)
    close(ts.normalPriorityQ)
    close(ts.resultCh)
    close(ts.controlCh)
    
    log.Println("Task scheduler shutdown complete")
}

// Example usage
func ExampleUsage() {
    // Create scheduler with 3 workers
    // 10 high priority slots, 50 normal priority slots, 100 result slots
    scheduler := NewTaskScheduler(3, 10, 50, 100)
    
    // Start processing
    scheduler.Start()
    
    // Start a goroutine to collect results
    go func() {
        for result := range scheduler.Results() {
            if result.Err != nil {
                log.Printf("Task %s failed: %v\n", result.TaskID, result.Err)
            } else {
                log.Printf("Task %s completed with result: %v (took %v)\n", 
                    result.TaskID, result.Value, result.FinishTime.Sub(result.StartTime))
            }
        }
    }()
    
    // Submit some tasks
    for i := 0; i < 20; i++ {
        priority := i % 10 // Vary priority
        
        task := Task{
            ID:          fmt.Sprintf("task-%d", i),
            Description: fmt.Sprintf("Sample task %d", i),
            Priority:    priority,
            ExecuteFunc: func() (interface{}, error) {
                // Simulate work
                time.Sleep(100 * time.Millisecond)
                return "completed", nil
            },
        }
        
        if err := scheduler.SubmitTask(task); err != nil {
            log.Printf("Failed to submit task: %v\n", err)
        }
    }
    
    // Send a control signal
    if err := scheduler.Control(); err != nil {
        log.Printf("Control error: %v\n", err)
    }
    
    // Let tasks process for a while
    time.Sleep(3 * time.Second)
    
    // Shutdown
    scheduler.Shutdown()
}
```

**Common Pitfalls:**
- Miscalculating buffer size, leading to blocked senders or memory issues
- Using buffered channels where synchronization guarantees are needed
- Deadlocks when all goroutines are blocked on channel operations
- Forgetting that a full buffered channel still blocks senders
- Not closing channels properly, causing goroutine leaks
- Assuming a buffered channel behaves like a proper queue data structure
- Using very large buffer sizes instead of implementing proper flow control

**Confusion Questions:**

1. **Q: How do I choose the right buffer size for my channels?**

   A: Selecting an appropriate buffer size for channels requires understanding your program's messaging patterns and performance characteristics. Here's a systematic approach:

   **Step 1: Start by determining if you need a buffer at all**

   - **Use unbuffered (size 0) when:**
     - You need synchronization guarantees between goroutines
     - Each message must be processed before the next is sent
     - You want direct handoff semantics
   
   - **Use buffered when:**
     - Producer and consumer work at different rates
     - You want to reduce goroutine blocking
     - You need to handle bursts of messages

   **Step 2: For buffered channels, consider these factors:**

   1. **Producer-consumer rate difference**:
      - If producer is faster than consumer: buffer ≈ (producer rate / consumer rate) × expected burst duration
      - If consumer is faster than producer: small buffer (1-10) is usually sufficient

   2. **Memory constraints**:
      - Estimate memory per item × buffer size to ensure it won't cause memory pressure
      - For large items, keep buffer smaller

   3. **Burstiness of workload**:
      - More bursty workloads benefit from larger buffers
      - Steady workloads need smaller buffers

   4. **Critical path considerations**:
      - Smaller buffers for critical paths to detect backpressure sooner
      - Larger buffers for non-critical background work

   **Step 3: Common sizing patterns**

   1. **Number of producers/consumers**:
      ```go
      // Common pattern: buffer = number of producers or consumers
      numWorkers := runtime.NumCPU()
      workCh := make(chan Task, numWorkers)
      ```

   2. **Expected burst size**:
      ```go
      // Buffer based on expected maximum burst
      // For a server that might get up to 100 requests in a burst
      requestCh := make(chan Request, 100)
      ```

   3. **Fixed processing batch**:
      ```go
      // If you process in batches of N, use at least N
      batchSize := 10
      dataCh := make(chan Item, batchSize)
      ```

   4. **Minimal buffering**:
      ```go
      // Small buffer to reduce blocking without consuming much memory
      resultCh := make(chan Result, 1)
      ```

   **Step 4: Measure and tune**

   Performance testing is crucial. Compare different buffer sizes:

   ```go
   func benchmarkChannelSize(size int, messages int) time.Duration {
       ch := make(chan int, size)
       start := time.Now()
       
       var wg sync.WaitGroup
       wg.Add(2)
       
       // Producer
       go func() {
           defer wg.Done()
           for i := 0; i < messages; i++ {
               ch <- i
           }
           close(ch)
       }()
       
       // Consumer
       go func() {
           defer wg.Done()
           for range ch {
               // Simulate processing
               time.Sleep(time.Microsecond)
           }
       }()
       
       wg.Wait()
       return time.Since(start)
   }
   ```

   **Real-world examples**:

   1. **Web server request handling**:
      ```go
      // For a web server, buffer size might be proportional to
      // max concurrent connections or requests per second
      requestQueue := make(chan Request, maxConcurrentRequests)
      ```

   2. **Data pipeline**:
      ```go
      // In a pipeline, each stage might have a different optimal buffer
      sourceData := make(chan Data, 100)   // Large input buffer
      processed := make(chan Result, 10)   // Smaller intermediate buffer
      output := make(chan FinalResult, 50) // Medium output buffer
      ```

   3. **Worker pool**:
      ```go
      // For a worker pool, often use buffer = number of workers
      workers := 8
      tasks := make(chan Task, workers)
      results := make(chan Result, workers)
      ```

   Remember that the ideal buffer size is workload-dependent, and there's no universal "correct" size for all situations. Start with a reasonable size based on the guidelines above, then measure and adjust as needed.

2. **Q: Why do unbuffered channels provide stronger synchronization guarantees?**

   A: Unbuffered channels provide stronger synchronization guarantees because they create a direct "rendezvous" between sender and receiver. This has important implications for program correctness and behavior.

   **Core synchronization guarantee**:

   With an unbuffered channel, a sender goroutine will block until a receiver goroutine is ready to receive the message, and vice versa. This creates a perfect synchronization point between the two goroutines.

   ```go
   ch := make(chan int) // Unbuffered channel

   go func() {
       fmt.Println("Sender: About to send")
       ch <- 42  // Will block until receiver is ready
       fmt.Println("Sender: Sent") // Only prints after receiver has received
   }()

   time.Sleep(2 * time.Second) // Delay before receiving
   fmt.Println("Receiver: About to receive")
   value := <-ch  // Unblocks sender
   fmt.Println("Receiver: Received", value)
   ```

   **Key guarantees provided by unbuffered channels**:

   1. **Message delivery confirmation**:
      When a send operation completes on an unbuffered channel, you are guaranteed that:
      - The receiver has received the value
      - The receiver has started executing code after the receive operation

      This serves as an implicit acknowledgment that the message was delivered.

   2. **Sequential ordering**:
      With an unbuffered channel, messages are processed in the exact order they are sent:
      ```go
      go func() {
          ch <- 1
          ch <- 2  // Won't be sent until 1 is received
          ch <- 3  // Won't be sent until 2 is received
      }()

      // Will always receive 1, 2, 3 in that order
      ```

   3. **Memory barrier/visibility**:
      Channel operations serve as memory barriers in Go's memory model. With unbuffered channels:
      - All memory writes in the sender goroutine before the send are visible to the receiver after the receive
      - This ensures proper visibility of other shared variables without additional synchronization

   4. **Limited concurrency**:
      An unbuffered channel naturally limits concurrency - the sender cannot proceed past the send operation until the receiver is ready, which can prevent resource overuse.

   **Comparing to buffered channels**:

   ```go
   buffCh := make(chan int, 2) // Buffered channel with capacity 2

   go func() {
       fmt.Println("Sender: Sending 1")
       buffCh <- 1  // Doesn't block (buffer has space)
       fmt.Println("Sender: Sent 1, immediately sending 2")
       buffCh <- 2  // Still doesn't block
       fmt.Println("Sender: Sent 2")

       // These all printed before receiver even starts!
   }()

   time.Sleep(2 * time.Second)
   fmt.Println("Receiver: Starting to receive now")
   value1 := <-buffCh
   fmt.Println("Receiver: Got", value1)
   ```

   With buffered channels:
   - Senders can proceed after sending (up to buffer capacity) without a receiver being ready
   - There's no guarantee the receiver has processed a message when the sender continues
   - Multiple messages can be "in flight" at once

   **Real-world examples where unbuffered channels are crucial**:

   1. **Request-response patterns**:
      ```go
      requestCh := make(chan Request)   // Unbuffered
      responseCh := make(chan Response) // Unbuffered

      // Client
      go func() {
          requestCh <- Request{ID: 1, Data: "query"}
          response := <-responseCh
          // Now guaranteed to have the response
      }()

      // Server
      go func() {
          request := <-requestCh
          // Process request
          responseCh <- Response{RequestID: request.ID, Result: "data"}
      }()
      ```

   2. **Two-phase commit**:
      ```go
      // Coordinator signals workers to prepare
      prepareCh := make(chan struct{})
      // Workers report back readiness
      readyCh := make(chan bool)
      // Coordinator signals commit
      commitCh := make(chan struct{})

      for i := 0; i < numWorkers; i++ {
          go func(id int) {
              <-prepareCh           // Wait for prepare signal
              ready := prepare(id)  // Do preparation work
              readyCh <- ready      // Report readiness
              <-commitCh            // Wait for commit signal
              commit(id)            // Commit changes
          }(i)
      }

      // Coordinator
      close(prepareCh)  // Signal all workers to prepare
      // Collect readiness signals
      allReady := true
      for i := 0; i < numWorkers; i++ {
          if !<-readyCh {
              allReady = false
          }
      }
      // Signal commit or abort based on readiness
      if allReady {
          close(commitCh)  // All workers proceed to commit phase
      } else {
          // Handle abort case
      }
      ```

   The key insight is that unbuffered channels not only transfer data but also synchronize execution flow between goroutines. When strong coordination is needed between concurrent processes, unbuffered channels provide the necessary guarantees.

3. **Q: Can I change a channel from buffered to unbuffered (or vice versa) once it's created?**

   A: No, you cannot change a channel's buffer size after it's created. The buffer capacity is set when the channel is created with `make()` and remains fixed for the channel's lifetime.

   **When creating channels**:
   ```go
   // Unbuffered channel
   unbufferedCh := make(chan int)
   // or explicitly
   alsoUnbufferedCh := make(chan int, 0)

   // Buffered channel with capacity 10
   bufferedCh := make(chan int, 10)
   ```

   **Why can't you change the buffer size?**

   The buffer is part of the channel's internal structure and memory allocation. Allowing dynamic resizing would be complex and potentially unsafe due to concurrent access.

   **Workarounds for when you need to change capacity:**

   1. **Create a new channel with the desired size**:
   ```go
   func resizeChannel(oldCh chan int, newSize int) chan int {
       // Create new channel with desired size
       newCh := make(chan int, newSize)
       
       // Start a goroutine to transfer values
       go func() {
           // Transfer existing values
           for {
               value, ok := <-oldCh
               if !ok {
                   // Old channel is closed
                   close(newCh)
                   return
               }
               newCh <- value
           }
       }()
       
       return newCh
   }
   ```

   2. **Use a dynamic buffer implementation**:
   ```go
   type DynamicChannel struct {
       input     chan interface{}
       output    chan interface{}
       buffer    []interface{}
       mu        sync.Mutex
       threshold int
   }

   func NewDynamicChannel(initialSize int, growThreshold int) *DynamicChannel {
       dc := &DynamicChannel{
           input:     make(chan interface{}),
           output:    make(chan interface{}),
           buffer:    make([]interface{}, 0, initialSize),
           threshold: growThreshold,
       }
       
       go dc.process()
       return dc
   }

   func (dc *DynamicChannel) process() {
       for {
           // If buffer is empty, we need to wait for an input
           if len(dc.buffer) == 0 {
               val, ok := <-dc.input
               if !ok {
                   // Input channel closed, close output and exit
                   close(dc.output)
                   return
               }
               dc.buffer = append(dc.buffer, val)
           }
           
           // Try to send or receive
           select {
           case val, ok := <-dc.input:
               if !ok {
                   // Input channel closed
                   close(dc.output)
                   return
               }
               dc.buffer = append(dc.buffer, val)
               
               // Grow buffer if needed
               dc.mu.Lock()
               if len(dc.buffer) > dc.threshold {
                   newBuffer := make([]interface{}, len(dc.buffer), cap(dc.buffer)*2)
                   copy(newBuffer, dc.buffer)
                   dc.buffer = newBuffer
               }
               dc.mu.Unlock()
               
           case dc.output <- dc.buffer[0]:
               // Value sent, remove from buffer
               dc.buffer = dc.buffer[1:]
           }
       }
   }

   // Input and output channels for external use
   func (dc *DynamicChannel) In() chan<- interface{} {
       return dc.input
   }

   func (dc *DynamicChannel) Out() <-chan interface{} {
       return dc.output
   }
   ```

   3. **Use a channel of channels pattern**:
   ```go
   type ChannelManager struct {
       current chan int
       control chan controlMsg
   }

   type controlMsg struct {
       newSize int
       reply   chan chan int
   }

   func NewChannelManager(initialSize int) *ChannelManager {
       cm := &ChannelManager{
           current: make(chan int, initialSize),
           control: make(chan controlMsg),
       }
       
       go cm.manage()
       return cm
   }

   func (cm *ChannelManager) manage() {
       for ctrl := range cm.control {
           // Create new channel with requested size
           newCh := make(chan int, ctrl.newSize)
           
           // Transfer values from old to new channel
           go func(old chan int, new chan int) {
               for {
                   select {
                   case val, ok := <-old:
                       if !ok {
                           return
                       }
                       new <- val
                   default:
                       return
                   }
               }
           }(cm.current, newCh)
           
           // Update the current channel
           cm.current = newCh
           
           // Reply with the new channel
           ctrl.reply <- newCh
       }
   }

   func (cm *ChannelManager) Resize(newSize int) chan int {
       reply := make(chan chan int)
       cm.control <- controlMsg{newSize: newSize, reply: reply}
       return <-reply
   }

   func (cm *ChannelManager) Channel() chan int {
       return cm.current
   }
   ```

   **Best practices since channels can't be resized**:

   1. **Choose an appropriate initial size**:
      - Consider the peak message rate and consumption rate
      - Often, buffer size = number of producers or consumers is a good starting point

   2. **Use dynamic buffer pattern when message rates are highly variable**:
      - Buffer in a slice and use a goroutine to mediate between channels

   3. **Implement backpressure mechanisms**:
      - Use non-blocking sends with `select` to detect when buffer is full
      - Slow down producers when buffers are filling up

   4. **Monitor channel behavior**:
      - Add metrics for channel size and blocking duration
      - Use these metrics to tune buffer sizes

   5. **Consider higher-level solutions**:
      - Use a proper queue implementation for very dynamic workloads
      - Consider streaming libraries for high-throughput needs

   In most real-world Go applications, a well-chosen fixed buffer size is sufficient. Focus on designing your concurrent system with appropriate flow control, rather than trying to dynamically resize channels.

### 5. Channel Directions

**Concise Explanation:**
Channel directions in Go allow you to restrict channel usage to either send-only or receive-only operations. By specifying a direction in a function parameter or variable declaration, you express intent and prevent accidental misuse. This provides a form of type safety, ensuring that channels are used as intended and improves code clarity by explicitly documenting the intended use of a channel.

**Where to Use:**
- In function parameters to clarify expected channel usage
- To enforce proper channel usage at compile time
- When passing channels between different parts of a system
- To improve API design by expressing intent clearly
- When separating producer and consumer responsibilities
- To prevent accidental sends or receives in the wrong place
- In public APIs to restrict how channels can be used

**Code Snippet:**
```go
package main

import (
    "fmt"
    "time"
)

// Function that only sends on a channel
func produce(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Printf("Sent: %d\n", i)
    }
    close(ch) // Only the sender should close the channel
}

// Function that only receives from a channel
func consume(ch <-chan int, done chan<- bool) {
    for num := range ch {
        fmt.Printf("Received: %d\n", num)
    }
    done <- true
}

// Function that converts a bidirectional channel to a receive-only channel
func makeReadOnly(ch chan int) <-chan int {
    return ch
}

// Function that converts a bidirectional channel to a send-only channel
func makeWriteOnly(ch chan int) chan<- int {
    return ch
}

func main() {
    // Bidirectional channel
    ch := make(chan int)
    
    // Unidirectional views of the same channel
    var sendCh chan<- int = ch // Send-only
    var recvCh <-chan int = ch // Receive-only
    
    // Attempting to receive from send-only channel
    // This will not compile:
    // value := <-sendCh
    
    // Attempting to send to receive-only channel
    // This will not compile:
    // recvCh <- 5
    
    // Use our channel direction functions
    done := make(chan bool)
    go produce(ch) // ch automatically converts to chan<- int
    go consume(ch, done) // ch automatically converts to <-chan int
    
    <-done // Wait for consumer to finish
}
```

**Real-World Example:**
Building a monitoring system with separate components for collection and processing:

```go
package monitoring

import (
    "context"
    "fmt"
    "log"
    "sync"
    "time"
)

// Metric represents a single data point
type Metric struct {
    Name      string
    Value     float64
    Timestamp time.Time
    Labels    map[string]string
}

// MetricCollector collects metrics from various sources
type MetricCollector interface {
    // Collect returns a receive-only channel that emits metrics
    Collect(ctx context.Context) <-chan Metric
}

// MetricProcessor processes incoming metrics
type MetricProcessor interface {
    // Process accepts a receive-only channel of metrics to process
    Process(ctx context.Context, metrics <-chan Metric) <-chan ProcessedMetric
}

// ProcessedMetric represents a metric after processing
type ProcessedMetric struct {
    Original Metric
    Derived  map[string]float64
    Error    error
}

// MetricStore stores processed metrics
type MetricStore interface {
    // Store accepts processed metrics via a receive-only channel
    Store(ctx context.Context, metrics <-chan ProcessedMetric) <-chan error
}

// SystemMonitor coordinates the collection, processing, and storage of metrics
type SystemMonitor struct {
    collectors []MetricCollector
    processor  MetricProcessor
    store      MetricStore
    interval   time.Duration
}

// NewSystemMonitor creates a new monitoring system
func NewSystemMonitor(
    collectors []MetricCollector,
    processor MetricProcessor,
    store MetricStore,
    interval time.Duration,
) *SystemMonitor {
    return &SystemMonitor{
        collectors: collectors,
        processor:  processor,
        store:      store,
        interval:   interval,
    }
}

// Start begins the monitoring process
func (s *SystemMonitor) Start(ctx context.Context) error {
    log.Println("Starting system monitoring")
    
    // Run monitoring until context is cancelled
    ticker := time.NewTicker(s.interval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ctx.Done():
            log.Println("Monitoring stopped by context cancellation")
            return ctx.Err()
            
        case <-ticker.C:
            if err := s.collectAndProcess(ctx); err != nil {
                log.Printf("Error during collection cycle: %v", err)
            }
        }
    }
}

// collectAndProcess performs a single cycle of collection, processing, and storage
func (s *SystemMonitor) collectAndProcess(ctx context.Context) error {
    // Create a cancellable sub-context for this collection cycle
    cycleCtx, cancel := context.WithTimeout(ctx, s.interval*9/10)
    defer cancel()
    
    // Merge metrics from all collectors
    metrics := s.mergeCollectors(cycleCtx)
    
    // Process metrics
    processed := s.processor.Process(cycleCtx, metrics)
    
    // Store processed metrics
    errCh := s.store.Store(cycleCtx, processed)
    
    // Wait for storage to complete and check for errors
    var lastErr error
    for err := range errCh {
        if err != nil {
            log.Printf("Storage error: %v", err)
            lastErr = err
        }
    }
    
    return lastErr
}

// mergeCollectors combines metrics from all collectors into a single channel
func (s *SystemMonitor) mergeCollectors(ctx context.Context) <-chan Metric {
    // Create output channel
    merged := make(chan Metric)
    
    // WaitGroup to track when all collectors are done
    var wg sync.WaitGroup
    
    // Start a goroutine for each collector
    for i, collector := range s.collectors {
        wg.Add(1)
        
        go func(id int, c MetricCollector) {
            defer wg.Done()
            
            // Get metrics channel from this collector
            metrics := c.Collect(ctx)
            
            // Forward all metrics to the merged channel
            for metric := range metrics {
                select {
                case merged <- metric:
                    // Successfully sent
                case <-ctx.Done():
                    // Context cancelled
                    return
                }
            }
            
            log.Printf("Collector %d completed", id)
        }(i, collector)
    }
    
    // Close the merged channel when all collectors are done
    go func() {
        wg.Wait()
        close(merged)
        log.Println("All collectors completed")
    }()
    
    return merged
}

//
// Example implementations of the interfaces
//

// CPUCollector collects CPU metrics
type CPUCollector struct{}

func (c *CPUCollector) Collect(ctx context.Context) <-chan Metric {
    ch := make(chan Metric)
    
    go func() {
        defer close(ch)
        
        // Simulate collecting CPU metrics
        for i := 0; i < 5; i++ {
            select {
            case <-ctx.Done():
                return
            default:
                ch <- Metric{
                    Name:      "cpu.usage",
                    Value:     float64(50 + i*5), // Simulated value
                    Timestamp: time.Now(),
                    Labels:    map[string]string{"cpu": fmt.Sprintf("cpu%d", i)},
                }
                time.Sleep(100 * time.Millisecond)
            }
        }
    }()
    
    return ch
}

// MemoryCollector collects memory metrics
type MemoryCollector struct{}

func (c *MemoryCollector) Collect(ctx context.Context) <-chan Metric {
    ch := make(chan Metric)
    
    go func() {
        defer close(ch)
        
        // Simulate collecting memory metrics
        select {
        case <-ctx.Done():
            return
        case ch <- Metric{
                Name:      "memory.used",
                Value:     8 * 1024 * 1024 * 1024, // 8GB
                Timestamp: time.Now(),
                Labels:    map[string]string{"type": "physical"},
            }:
        }
        
        select {
        case <-ctx.Done():
            return
        case ch <- Metric{
                Name:      "memory.free",
                Value:     8 * 1024 * 1024 * 1024, // 8GB
                Timestamp: time.Now(),
                Labels:    map[string]string{"type": "physical"},
            }:
        }
    }()
    
    return ch
}

// StandardProcessor processes metrics with standard algorithms
type StandardProcessor struct{}

func (p *StandardProcessor) Process(ctx context.Context, metrics <-chan Metric) <-chan ProcessedMetric {
    processed := make(chan ProcessedMetric)
    
    go func() {
        defer close(processed)
        
        for metric := range metrics {
            // Simulate processing
            derived := make(map[string]float64)
            
            // Add derived metrics based on metric name
            switch metric.Name {
            case "cpu.usage":
                derived["load"] = metric.Value / 100.0
            case "memory.used":
                derived["used_gb"] = metric.Value / (1024 * 1024 * 1024)
            }
            
            // Send processed metric
            select {
            case <-ctx.Done():
                return
            case processed <- ProcessedMetric{
                    Original: metric,
                    Derived:  derived,
                }:
            }
        }
    }()
    
    return processed
}

// DatabaseStore stores metrics in a database
type DatabaseStore struct{}

func (s *DatabaseStore) Store(ctx context.Context, metrics <-chan ProcessedMetric) <-chan error {
    errCh := make(chan error)
    
    go func() {
        defer close(errCh)
        
        for metric := range metrics {
            // Simulate database storage
            log.Printf("Storing metric: %s = %f %v",
                metric.Original.Name,
                metric.Original.Value,
                metric.Original.Labels)
            
            // Simulate occasional errors
            if metric.Original.Value > 60 && metric.Original.Name == "cpu.usage" {
                select {
                case <-ctx.Done():
                    return
                case errCh <- fmt.Errorf("simulated error storing high CPU metric"):
                }
            }
        }
    }()
    
    return errCh
}

// Example usage
func ExampleUsage() {
    // Create collectors
    collectors := []MetricCollector{
        &CPUCollector{},
        &MemoryCollector{},
    }
    
    // Create processor and store
    processor := &StandardProcessor{}
    store := &DatabaseStore{}
    
    // Create monitor
    monitor := NewSystemMonitor(
        collectors,
        processor,
        store,
        5*time.Second, // Collect every 5 seconds
    )
    
    // Start monitoring with cancellation after 30 seconds
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    // Run the monitor (normally this would block until cancelled)
    go func() {
        if err := monitor.Start(ctx); err != nil {
            log.Printf("Monitor stopped with error: %v", err)
        }
    }()
    
    // For example purposes, wait a bit then cancel
    time.Sleep(12 * time.Second)
    
    log.Println("Cancelling monitoring")
    cancel()
    
    // Allow time for graceful shutdown
    time.Sleep(1 * time.Second)
}
```

**Common Pitfalls:**
- Trying to send on a receive-only channel or vice versa
- Closing a receive-only channel (only the sender should close)
- Using bidirectional channels everywhere, missing documentation benefits
- Over-restricting channel directions when the code needs bidirectional access
- Not understanding that channel direction conversion is one-way (can't convert receive-only to bidirectional)
- Unnecessarily converting channel directions within the same function
- Forgetting that nil channels of any direction will block indefinitely

**Confusion Questions:**

1. **Q: When should I use channel direction restrictions in my code?**

   A: Channel direction restrictions should be used strategically to improve code safety, clarity, and design. Here's a comprehensive guide to when and how to use them:

   **1. Use in function parameters for clarity and safety**

   ```go
   // Without direction - unclear what the function does with the channel
   func process(ch chan int) {
       // Could send, receive, or both
   }
   
   // With directions - clear intention
   func produceData(out chan<- int) {
       // Clearly only sends on the channel
       for i := 0; i < 5; i++ {
           out <- i
       }
   }
   
   func consumeData(in <-chan int) {
       // Clearly only receives from the channel
       for v := range in {
           fmt.Println(v)
       }
   }
   ```

   **2. When separating concerns between producers and consumers**

   ```go
   type Producer struct {
       output chan<- int // Can only send
   }
   
   type Consumer struct {
       input <-chan int // Can only receive
   }
   
   func NewProducerConsumerPair() (*Producer, *Consumer) {
       ch := make(chan int)
       return &Producer{output: ch}, &Consumer{input: ch}
   }
   ```

   **3. When designing public APIs**

   ```go
   type EventEmitter struct {
       // Private bidirectional channel for internal use
       events chan Event
   }
   
   // Public method returns receive-only channel to prevent
   // external code from sending or closing
   func (e *EventEmitter) Events() <-chan Event {
       return e.events // Automatically converts to receive-only
   }
   
   // Only EventEmitter methods can send events
   func (e *EventEmitter) Emit(event Event) {
       e.events <- event
   }
   ```

   **4. To enforce the "only sender closes" pattern**

   ```go
   func processItems(items []Item) <-chan Result {
       results := make(chan Result)
       
       go func() {
           defer close(results) // Safe: we own this channel
           
           for _, item := range items {
               results <- process(item)
           }
       }()
       
       return results // Return as receive-only to prevent accidental closing
   }
   ```

   **5. In complex systems to prevent mistakes**

   ```go
   type Pipeline struct {
       input  chan<- Data    // Restricted to send-only
       output <-chan Result  // Restricted to receive-only
       control chan<- Signal // Control channel (send-only)
   }
   
   func NewPipeline() *Pipeline {
       in := make(chan Data)
       out := make(chan Result)
       ctrl := make(chan Signal)
       
       go runPipeline(in, out, ctrl)
       
       return &Pipeline{
           input:   in,
           output:  out,
           control: ctrl,
       }
   }
   ```

   **6. When documenting ownership**

   ```go
   // Worker owns (creates, writes to, and closes) the results channel
   func worker(tasks <-chan Task) <-chan Result {
       results := make(chan Result)
       
       go func() {
           defer close(results)
           for task := range tasks {
               results <- processTask(task)
           }
       }()
       
       return results
   }
   ```

   **Practical recommendations:**

   1. **Public APIs and interfaces**: Always use channel direction constraints
      ```go
      type MessageSource interface {
          // Clearly communicates this only produces messages
          Messages() <-chan Message
      }
      ```

   2. **Internal functions**: Use when it adds clarity or prevents misuse
      ```go
      func processInBackground(data <-chan Item, results chan<- Result) {
          // Function signature makes usage clear
      }
      ```

   3. **Short local functions**: May not need restrictions
      ```go
      // Simple function with local usage might not need restrictions
      func localHelper(ch chan int) {
          ch <- 42
      }
      ```

   4. **Closure-based goroutines**: Often don't need restrictions
      ```go
      // Channel is in closure scope, usage is clear
      go func() {
          for item := range inputCh {
              outputCh <- process(item)
          }
      }()
      ```

   The guiding principle is to use channel direction constraints to make your code's intentions clear and to catch potential mistakes at compile time rather than runtime.

2. **Q: How do channel directions affect channel closing? Who should close a channel?**

   A: Channel directions have important implications for channel closing. Understanding who should close channels and how direction constraints affect closing is critical for correct concurrent programs.

   **The fundamental rule: Only the sender should close a channel**

   This rule exists because:
   1. Sending on a closed channel causes a panic
   2. Multiple senders could race to close the channel
   3. Closing signals "no more values will be sent"

   **Channel directions enforce this rule:**

   ```go
   // Receive-only channel cannot be closed
   func consumer(ch <-chan int) {
       for v := range ch {
           fmt.Println(v)
       }
       
       // This won't compile:
       // close(ch)
       // Error: invalid operation: cannot close receive-only channel
   }
   
   // Send-only channel can be closed
   func producer(ch chan<- int) {
       for i := 0; i < 5; i++ {
           ch <- i
       }
       close(ch) // Legal: we're the sender
   }
   ```

   **Who should close a channel depends on ownership patterns:**

   **1. Single sender pattern (most common)**

   ```go
   func singleSender() <-chan int {
       ch := make(chan int)
       
       go func() {
           defer close(ch) // The sender closes
           for i := 0; i < 5; i++ {
               ch <- i
           }
       }()
       
       return ch // Return as receive-only
   }
   ```

   **2. Multiple senders require coordination**

   ```go
   func multipleSenders(n int) <-chan int {
       ch := make(chan int)
       done := make(chan struct{}) // Signal channel
       
       // Launch n senders
       var wg sync.WaitGroup
       wg.Add(n)
       for i := 0; i < n; i++ {
           go func(id int) {
               defer wg.Done()
               
               // Send values until done signal
               for j := 0; ; j++ {
                   select {
                   case <-done:
                       return
                   case ch <- id*1000 + j:
                       time.Sleep(time.Millisecond)
                   }
               }
           }(i)
       }
       
       // Launch the closing goroutine
       go func() {
           wg.Wait()         // Wait for all senders to finish
           close(done)       // Signal senders to stop
           close(ch)         // Safe to close now
       }()
       
       return ch
   }
   ```

   **3. Use a done channel pattern**

   ```go
   func workerPool(numWorkers int, tasks []Task) <-chan Result {
       taskCh := make(chan Task)
       resultCh := make(chan Result)
       doneCh := make(chan struct{})
       
       // Launch workers that process tasks
       var wg sync.WaitGroup
       wg.Add(numWorkers)
       for i := 0; i < numWorkers; i++ {
           go func() {
               defer wg.Done()
               for {
                   select {
                   case task, ok := <-taskCh:
                       if !ok {
                           return // taskCh was closed
                       }
                       resultCh <- processTask(task)
                   case <-doneCh:
                       return // Signal to stop
                   }
               }
           }()
       }
       
       // Feed tasks to workers
       go func() {
           for _, task := range tasks {
               taskCh <- task
           }
           close(taskCh) // No more tasks
       }()
       
       // Close result channel when all workers done
       go func() {
           wg.Wait()
           close(resultCh)
           close(doneCh)
       }()
       
       return resultCh
   }
   ```

   **4. Use a dedicated closer**

   ```go
   type SafeChannel struct {
       ch     chan int
       closed bool
       mu     sync.Mutex
   }
   
   func NewSafeChannel() *SafeChannel {
       return &SafeChannel{
           ch: make(chan int),
       }
   }
   
   func (sc *SafeChannel) Send(val int) bool {
       sc.mu.Lock()
       defer sc.mu.Unlock()
       
       if sc.closed {
           return false
       }
       
       sc.ch <- val
       return true
   }
   
   func (sc *SafeChannel) Receive() (int, bool) {
       val, ok := <-sc.ch
       return val, ok
   }
   
   func (sc *SafeChannel) Close() {
       sc.mu.Lock()
       defer sc.mu.Unlock()
       
       if !sc.closed {
           close(sc.ch)
           sc.closed = true
       }
   }
   
   func (sc *SafeChannel) Channel() <-chan int {
       return sc.ch // Return as receive-only
   }
   ```

   **5. Context-based cancellation**

   ```go
   func streamValues(ctx context.Context) <-chan int {
       ch := make(chan int)
       
       go func() {
           defer close(ch)
           
           for i := 0; ; i++ {
               select {
               case <-ctx.Done():
                   return
               case ch <- i:
                   time.Sleep(100 * time.Millisecond)
               }
           }
       }()
       
       return ch
   }
   
   // Usage
   ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
   defer cancel()
   
   for val := range streamValues(ctx) {
       fmt.Println(val)
   }
   ```

   **Best practices for channel closing:**

   1. **Establish clear ownership**: Define who is responsible for closing each channel
   2. **Direction constraints make it explicit**: Use to prevent unintended closing
   3. **For fan-in/fan-out**: Use a dedicated goroutine to close after all inputs finish
   4. **Prefer returning receive-only channels** from functions that create goroutines
   5. **Never close a channel from the receiver side**
   6. **For channels that may never need closing**, consider not closing them at all
   7. **Use synchronization primitives** when multiple goroutines might close a channel

   By following these practices and understanding the relationship between channel directions and closing responsibility, you can avoid common concurrency bugs in Go programs.

3. **Q: Can I convert a receive-only or send-only channel back to a bidirectional channel?**

   A: No, you cannot convert a receive-only (`<-chan T`) or send-only (`chan<- T`) channel back to a bidirectional channel (`chan T`). Channel direction conversion is deliberately one-way in Go.

   **Channel direction conversions**:

   ```go
   // Bidirectional channel
   ch := make(chan int)
   
   // These conversions are allowed:
   var sendCh chan<- int = ch // Bidirectional to send-only: OK
   var recvCh <-chan int = ch // Bidirectional to receive-only: OK
   
   // These conversions are NOT allowed:
   var biCh1 chan int = sendCh // Send-only to bidirectional: COMPILE ERROR
   var biCh2 chan int = recvCh // Receive-only to bidirectional: COMPILE ERROR
   ```

   **Why Go prevents converting back to bidirectional**:

   1. **Type safety**: Once you've restricted a channel's operations, removing that restriction would violate type safety guarantees
   
   2. **Preventing mistakes**: The one-way restriction exists to prevent accidental misuse
   
   3. **API contracts**: Direction constraints are part of function contracts that should be respected

   **How to handle situations where you need bidirectional access**:

   **1. Keep the original bidirectional channel when needed**:

   ```go
   func process(data []int) (<-chan int, chan<- int) {
       ch := make(chan int) // Create bidirectional channel
       
       // Return as two direction-constrained views
       return ch, ch
   }
   
   // Or store the original and provide restricted access
   type Processor struct {
       dataCh chan int // Internal bidirectional channel
   }
   
   func (p *Processor) Input() chan<- int {
       return p.dataCh
   }
   
   func (p *Processor) Output() <-chan int {
       return p.dataCh
   }
   ```

   **2. Use separate channels for different directions**:

   ```go
   type Service struct {
       requests  chan Request  // Incoming requests
       responses chan Response // Outgoing responses
   }
   
   func (s *Service) RequestChannel() chan<- Request {
       return s.requests
   }
   
   func (s *Service) ResponseChannel() <-chan Response {
       return s.responses
   }
   ```

   **3. Use a wrapper type to manage channels**:

   ```go
   type ChannelPair struct {
       ch chan int // Private bidirectional channel
   }
   
   func NewChannelPair() *ChannelPair {
       return &ChannelPair{ch: make(chan int)}
   }
   
   func (cp *ChannelPair) Send(v int) {
       cp.ch <- v
   }
   
   func (cp *ChannelPair) Receive() (int, bool) {
       v, ok := <-cp.ch
       return v, ok
   }
   
   func (cp *ChannelPair) Close() {
       close(cp.ch)
   }
   
   func (cp *ChannelPair) SendChannel() chan<- int {
       return cp.ch
   }
   
   func (cp *ChannelPair) ReceiveChannel() <-chan int {
       return cp.ch
   }
   ```

   **4. Design around the constraint**:

   ```go
   // Instead of trying to get back bidirectional access,
   // design functions to accept the channel direction they need
   
   func producer(out chan<- int) {
       // Send values
   }
   
   func consumer(in <-chan int) {
       // Receive values
   }
   
   func coordinator() {
       ch := make(chan int)
       go producer(ch)
       go consumer(ch)
   }
   ```

   **5. Using type assertions (NOT recommended)**:

   ```go
   // UNSAFE: This circumvents type safety and is not recommended
   func unsafeRecover(ch interface{}) chan int {
       switch ch := ch.(type) {
       case chan int:
           return ch
       case chan<- int:
           return ch.(chan int) // Will panic at runtime!
       case <-chan int:
           return ch.(chan int) // Will panic at runtime!
       default:
           panic("not a channel")
       }
   }
   ```

   **Best practices**:

   1. **Design for unidirectionality from the start**:
      - Plan channel ownership and usage patterns
      - Use separate channels for separate concerns
   
   2. **Keep bidirectional channels internal**:
      - Expose only restricted views externally
      - Use encapsulation to maintain proper control
   
   3. **Create channels at the right scope level**:
      - Create channels where both directions are needed
      - Pass constrained views to other functions
   
   4. **Use function parameters to express intent**:
      ```go
      func process(in <-chan Request, out chan<- Response)
      ```

   5. **Return the right direction from functions**:
      ```go
      // If caller only needs to receive from the channel
      func generateNumbers() <-chan int
      
      // If caller only needs to send to the channel
      func createProcessor() chan<- Task
      ```

   The restriction on converting back to bidirectional channels is an intentional design decision in Go to enforce safer concurrent programming patterns. By working with this constraint rather than fighting against it, you'll develop clearer, safer concurrent code.

### 6. Select Statement

**Concise Explanation:**
The `select` statement in Go provides a way to work with multiple channels simultaneously. It blocks until one of its cases can proceed, or executes the default case if provided. If multiple cases are ready, one is chosen at random. This allows goroutines to coordinate multiple events, implement timeouts, handle cancellations, and avoid deadlocks when working with multiple channels.

**Where to Use:**
- When working with multiple channels simultaneously
- To implement non-blocking channel operations
- For adding timeouts to channel operations
- To handle cancellation signals
- For multiplexing input from multiple sources
- To implement advanced concurrency patterns like fan-in
- To prioritize processing between multiple ready channels

**Code Snippet:**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create channels
    ch1 := make(chan string)
    ch2 := make(chan string)
    done := make(chan bool)
    
    // Sender for ch1
    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "Message from channel 1"
    }()
    
    // Sender for ch2
    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "Message from channel 2"
    }()
    
    // Basic select usage
    fmt.Println("Waiting for messages...")
    
    select {
    case msg := <-ch1:
        fmt.Println("Received:", msg)
    case msg := <-ch2:
        fmt.Println("Received:", msg)
    }
    
    // Select with timeout
    fmt.Println("\nSelect with timeout:")
    
    select {
    case msg := <-ch1:
        fmt.Println("Received:", msg)
    case <-time.After(500 * time.Millisecond):
        fmt.Println("Timeout after 500ms")
    }
    
    // Non-blocking channel operations
    fmt.Println("\nNon-blocking receive:")
    
    select {
    case msg := <-ch1:
        fmt.Println("Received:", msg)
    default:
        fmt.Println("No message available")
    }
    
    // Non-blocking send
    fmt.Println("\nNon-blocking send:")
    
    select {
    case ch1 <- "Message":
        fmt.Println("Sent message")
    default:
        fmt.Println("Cannot send message")
    }
    
    // Receiving from multiple channels in a loop
    fmt.Println("\nReceiving from multiple channels in a loop:")
    
    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "First message"
        time.Sleep(500 * time.Millisecond)
        ch2 <- "Second message"
        time.Sleep(500 * time.Millisecond)
        done <- true
    }()
    
    for {
        select {
        case msg := <-ch1:
            fmt.Println("Received from ch1:", msg)
        case msg := <-ch2:
            fmt.Println("Received from ch2:", msg)
        case <-done:
            fmt.Println("Done signal received, exiting loop")
            return
        }
    }
}
```

**Real-World Example:**
A service that handles client requests with timeouts and cancellation:

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "log"
    "math/rand"
    "net/http"
    "sync"
    "time"
)

// RequestHandler processes client requests with timeouts and cancellation
type RequestHandler struct {
    maxConcurrent int
    timeout       time.Duration
    rateLimiter   <-chan time.Time
    activeConns   int
    mu            sync.Mutex
}

// NewRequestHandler creates a new request handler
func NewRequestHandler(maxConcurrent int, timeout time.Duration, rateLimit int) *RequestHandler {
    return &RequestHandler{
        maxConcurrent: maxConcurrent,
        timeout:       timeout,
        rateLimiter:   time.Tick(time.Second / time.Duration(rateLimit)),
    }
}

// HandleRequest processes a client request with timeouts
func (h *RequestHandler) HandleRequest(w http.ResponseWriter, r *http.Request) {
    // Check if we're at capacity
    h.mu.Lock()
    isOverloaded := h.activeConns >= h.maxConcurrent
    if !isOverloaded {
        h.activeConns++
    }
    h.mu.Unlock()
    
    if isOverloaded {
        w.WriteHeader(http.StatusServiceUnavailable)
        fmt.Fprintln(w, "Server is at capacity, please try again later")
        return
    }
    
    // Ensure we decrease the connection count when done
    defer func() {
        h.mu.Lock()
        h.activeConns--
        h.mu.Unlock()
    }()
    
    // Apply rate limiting
    select {
    case <-h.rateLimiter:
        // Allowed to proceed
    default:
        // Rate limit exceeded
        w.WriteHeader(http.StatusTooManyRequests)
        fmt.Fprintln(w, "Rate limit exceeded")
        return
    }
    
    // Create context with timeout
    ctx, cancel := context.WithTimeout(r.Context(), h.timeout)
    defer cancel()
    
    // Process the request
    result, err := h.processRequest(ctx, r)
    
    // Check the result
    if err != nil {
        if errors.Is(err, context.DeadlineExceeded) {
            w.WriteHeader(http.StatusGatewayTimeout)
            fmt.Fprintln(w, "Request processing timed out")
        } else if errors.Is(err, context.Canceled) {
            // Client canceled the request
            log.Println("Request canceled by client")
            return
        } else {
            w.WriteHeader(http.StatusInternalServerError)
            fmt.Fprintf(w, "Error: %v", err)
        }
        return
    }
    
    // Return success
    fmt.Fprintf(w, "Result: %s\n", result)
}

// processRequest simulates processing a request
func (h *RequestHandler) processRequest(ctx context.Context, r *http.Request) (string, error) {
    // Channels for the different processing stages
    dataCh := make(chan []byte)
    processedCh := make(chan string)
    errCh := make(chan error, 2) // Buffer to prevent goroutine leaks
    
    // Start data retrieval
    go func() {
        // Simulating data retrieval (e.g., database query)
        data, err := h.fetchData(r)
        if err != nil {
            errCh <- fmt.Errorf("data fetch error: %w", err)
            return
        }
        
        // Try to send the data, or abort if context is done
        select {
        case dataCh <- data:
            // Data sent successfully
        case <-ctx.Done():
            // Context canceled, abort
            return
        }
    }()
    
    // Start data processing when data is available
    go func() {
        // Wait for data from previous stage
        var data []byte
        
        select {
        case d := <-dataCh:
            data = d
        case <-ctx.Done():
            // Context canceled, abort
            return
        }
        
        // Process the data (simulated)
        result, err := h.processData(data)
        if err != nil {
            errCh <- fmt.Errorf("processing error: %w", err)
            return
        }
        
        // Try to send the result, or abort if context is done
        select {
        case processedCh <- result:
            // Result sent successfully
        case <-ctx.Done():
            // Context canceled, abort
            return
        }
    }()
    
    // Wait for result or error or timeout
    select {
    case result := <-processedCh:
        return result, nil
    case err := <-errCh:
        return "", err
    case <-ctx.Done():
        return "", ctx.Err()
    }
}

// fetchData simulates retrieving data (e.g., from a database)
func (h *RequestHandler) fetchData(r *http.Request) ([]byte, error) {
    // Simulate variable processing time
    processingTime := time.Duration(100+rand.Intn(500)) * time.Millisecond
    time.Sleep(processingTime)
    
    // 10% chance of error
    if rand.Intn(10) == 0 {
        return nil, errors.New("data fetch failed")
    }
    
    return []byte("sample data for " + r.URL.Path), nil
}

// processData simulates processing the retrieved data
func (h *RequestHandler) processData(data []byte) (string, error) {
    // Simulate variable processing time
    processingTime := time.Duration(200+rand.Intn(800)) * time.Millisecond
    time.Sleep(processingTime)
    
    // 10% chance of error
    if rand.Intn(10) == 0 {
        return "", errors.New("data processing failed")
    }
    
    return fmt.Sprintf("Processed: %s (length: %d)", data, len(data)), nil
}

// Simulate a simple HTTP server
func main() {
    // Set up the request handler with:
    // - 10 max concurrent requests
    // - 2 second timeout per request
    // - 5 requests per second rate limit
    handler := NewRequestHandler(10, 2*time.Second, 5)
    
    // Create a server mux
    mux := http.NewServeMux()
    mux.HandleFunc("/process", handler.HandleRequest)
    
    // Start the server
    server := &http.Server{
        Addr:    ":8080",
        Handler: mux,
    }
    
    // Initialize random seed
    rand.Seed(time.Now().UnixNano())
    
    log.Println("Server starting on :8080")
    if err := server.ListenAndServe(); err != nil {
        log.Fatalf("Server error: %v", err)
    }
}
```

**Common Pitfalls:**
- Forgetting the `default` case can make a `select` block forever
- Not handling closed channels in `select` statements
- Creating deadlocks by having all goroutines blocked on `select`
- Assuming deterministic order of case execution when multiple are ready
- Using select with a single case (just use a direct channel operation)
- Not using timeouts in `select` when appropriate
- Forgetting that empty `select{}` blocks forever

**Confusion Questions:**

1. **Q: What happens when multiple cases in a select statement are ready at the same time?**

   A: When multiple cases in a `select` statement are ready simultaneously, Go chooses one of them at random. This deliberate non-determinism prevents certain classes of bugs and deadlocks by avoiding starvation patterns.

   **Example showing randomness:**

   ```go
   package main

   import (
       "fmt"
       "time"
   )

   func main() {
       // Create two ready-to-receive channels
       ch1 := make(chan string, 1)
       ch2 := make(chan string, 1)
       
       // Place values in both channels (both cases will be ready)
       ch1 <- "from channel 1"
       ch2 <- "from channel 2"
       
       // Run the select multiple times to show randomness
       fmt.Println("Running select 10 times with multiple ready cases:")
       
       countCh1 := 0
       countCh2 := 0
       
       for i := 0; i < 10; i++ {
           select {
           case msg := <-ch1:
               fmt.Printf("Received: %s\n", msg)
               ch1 <- msg // Put it back for next iteration
               countCh1++
           case msg := <-ch2:
               fmt.Printf("Received: %s\n", msg)
               ch2 <- msg // Put it back for next iteration
               countCh2++
           }
       }
       
       fmt.Printf("\nResults: ch1 selected %d times, ch2 selected %d times\n", 
           countCh1, countCh2)
   }
   ```

   The output varies between runs, but both channels will typically be selected multiple times, showing the random selection behavior.

   **Why random selection is important:**

   1. **Prevents starvation**: If `select` had a deterministic order (like checking cases from top to bottom), the first ready channel would always be chosen, potentially starving other channels.

   2. **Avoids predictability bugs**: Randomized selection prevents developers from relying on selection order, which could lead to subtle bugs if the implementation changed.

   3. **Fair distribution**: Randomness provides a basic form of fairness between competing channels, which is important in concurrent systems.

   **Implementing prioritization when needed:**

   If you need priority between channels rather than random selection, you can use nested `select` statements:

   ```go
   // Prioritize ch1 over ch2
   select {
   case msg := <-ch1:
       // Handle high priority message
       fmt.Println("High priority:", msg)
   default:
       // No message on high priority channel, check low priority
       select {
       case msg := <-ch2:
           fmt.Println("Low priority:", msg)
       default:
           // No message on any channel
           fmt.Println("No messages")
       }
   }
   ```

   **Ensuring fair handling of multiple ready channels:**

   If you need to process all ready channels, not just a randomly selected one, you can use a loop with non-blocking receives:

   ```go
   processedSomething := true
   for processedSomething {
       processedSomething = false
       
       select {
       case msg1, ok1 := <-ch1:
           if ok1 {
               fmt.Println("Processed ch1:", msg1)
               processedSomething = true
           }
       default:
       }
       
       select {
       case msg2, ok2 := <-ch2:
           if ok2 {
               fmt.Println("Processed ch2:", msg2)
               processedSomething = true
           }
       default:
       }
       
       // Add more channels as needed
   }
   ```

   **Real-world application: Load balancer**

   In a load balancer or request router, random selection among available workers can provide a simple form of load distribution:

   ```go
   func loadBalancer(requests <-chan Request, workers []chan<- Request) {
       for req := range requests {
           // Randomly select among all available workers
           // (all ready to receive a request)
           select {
           case workers[0] <- req:
           case workers[1] <- req:
           case workers[2] <- req:
           // ...more workers...
           }
       }
   }
   ```

   The random selection behavior of `select` is a key design feature in Go's concurrency model, not a limitation. Understanding and working with this behavior leads to more robust concurrent programs.

2. **Q: How can I implement a timeout for a channel operation using select?**

   A: The `select` statement combined with Go's `time` package provides an elegant way to implement timeouts for channel operations. Here are several patterns for implementing timeouts:

   **1. Basic timeout pattern for channel operations**

   ```go
   // Timeout for a receive operation
   select {
   case data := <-ch:
       // Use the received data
       fmt.Println("Received:", data)
   case <-time.After(2 * time.Second):
       fmt.Println("Timed out after waiting 2 seconds")
   }
   ```

   **2. Timeout for send operations**

   ```go
   // Timeout for a send operation
   select {
   case ch <- data:
       fmt.Println("Sent data successfully")
   case <-time.After(2 * time.Second):
       fmt.Println("Timed out after waiting 2 seconds")
   }
   ```

   **3. Using context for operation timeouts**

   ```go
   // Create a context with timeout
   ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
   defer cancel()
   
   // Use context for timeout
   select {
   case data := <-dataCh:
       fmt.Println("Received:", data)
   case <-ctx.Done():
       fmt.Println("Operation timed out:", ctx.Err())
   }
   ```

   **4. Implementing a timeout for a function call**

   ```go
   func executeWithTimeout(timeout time.Duration, operation func() (interface{}, error)) (interface{}, error) {
       resultCh := make(chan struct {
           result interface{}
           err    error
       }, 1)
       
       // Execute operation in goroutine
       go func() {
           result, err := operation()
           resultCh <- struct {
               result interface{}
               err    error
           }{result, err}
       }()
       
       // Wait for result or timeout
       select {
       case r := <-resultCh:
           return r.result, r.err
       case <-time.After(timeout):
           return nil, errors.New("operation timed out")
       }
   }
   
   // Usage
   result, err := executeWithTimeout(5*time.Second, func() (interface{}, error) {
       // Some long operation
       time.Sleep(3 * time.Second)
       return "operation result", nil
   })
   ```

   **5. Implementing a per-request timeout in an HTTP server**

   ```go
   func handleRequest(w http.ResponseWriter, r *http.Request) {
       // Get timeout from request or use default
       timeout := 5 * time.Second
       if t := r.URL.Query().Get("timeout"); t != "" {
           if parsedTimeout, err := time.ParseDuration(t); err == nil {
               timeout = parsedTimeout
           }
       }
       
       // Context carries the timeout
       ctx, cancel := context.WithTimeout(r.Context(), timeout)
       defer cancel()
       
       // Channel for the result
       resultCh := make(chan string, 1)
       
       // Process in a goroutine
       go func() {
           result, err := processRequest(ctx, r.URL.Path)
           if err != nil {
               resultCh <- fmt.Sprintf("Error: %v", err)
               return
           }
           resultCh <- result
       }()
       
       // Wait for result or timeout
       select {
       case result := <-resultCh:
           fmt.Fprintln(w, result)
       case <-ctx.Done():
           w.WriteHeader(http.StatusGatewayTimeout)
           fmt.Fprintln(w, "Request timed out")
       }
   }
   ```

   **6. Using a reusable timer to avoid excessive timer creation**

   ```go
   // For repeated timeout checks, reuse the timer
   timer := time.NewTimer(5 * time.Second)
   defer timer.Stop()
   
   for {
       timer.Reset(5 * time.Second)
       
       select {
       case data := <-dataCh:
           // Process data
           fmt.Println("Received:", data)
       case <-timer.C:
           fmt.Println("Timed out, no data received")
           return
       }
   }
   ```

   **7. Adding timeouts to multiple channel operations**

   ```go
   func processWithTimeouts(inputs []<-chan int, timeout time.Duration) []int {
       results := make([]int, 0, len(inputs))
       
       for _, ch := range inputs {
           select {
           case val := <-ch:
               results = append(results, val)
           case <-time.After(timeout):
               // Skip this channel on timeout
               continue
           }
       }
       
       return results
   }
   ```

   **8. Global timeout for a batch of operations**

   ```go
   func processWithGlobalTimeout(inputs []<-chan int, timeout time.Duration) []int {
       results := make([]int, 0, len(inputs))
       timer := time.NewTimer(timeout)
       defer timer.Stop()
       
       for _, ch := range inputs {
           select {
           case val := <-ch:
               results = append(results, val)
           case <-timer.C:
               // Global timeout reached, return what we have so far
               return results
           }
       }
       
       return results
   }
   ```

   **Important considerations when using timeouts:**

   1. **Timer creation overhead**: `time.After()` creates a new timer for each call, which can be expensive if used in tight loops. Use `time.NewTimer()` and `timer.Reset()` for repeated timeout operations.

   2. **Memory leaks**: Be cautious with goroutines that might be blocked indefinitely after a timeout. Use cancellation or contexts to ensure proper cleanup.

   3. **Appropriate timeout values**: Choose timeouts based on your application's needs. Too short may cause unnecessary failures, too long may tie up resources.

   4. **Cascading timeouts**: Consider cascading timeout strategies where inner operations get less time than the overall operation.

   ```go
   func cascadingTimeouts() {
       outerCtx, outerCancel := context.WithTimeout(context.Background(), 10*time.Second)
       defer outerCancel()
       
       // First stage gets 4 seconds
       stage1Ctx, stage1Cancel := context.WithTimeout(outerCtx, 4*time.Second)
       defer stage1Cancel()
       
       result1, err := stageOne(stage1Ctx)
       if err != nil {
           return handleError(err)
       }
       
       // Second stage gets 3 seconds
       stage2Ctx, stage2Cancel := context.WithTimeout(outerCtx, 3*time.Second)
       defer stage2Cancel()
       
       result2, err := stageTwo(stage2Ctx, result1)
       if err != nil {
           return handleError(err)
       }
       
       // Remaining time is available for final stage
       return finalize(outerCtx, result2)
   }
   ```

   Timeouts are essential for creating robust services that can recover from unexpected delays or failures. By using `select` with timeouts, you can ensure your concurrent operations don't block indefinitely and that resources are properly managed.

3. **Q: How can I implement non-blocking channel operations with select?**

   A: The `select` statement with a `default` case provides an elegant way to implement non-blocking operations on channels. This pattern allows your code to check channel status without waiting.

   **Basic non-blocking receive:**

   ```go
   // Try to receive from a channel without blocking
   select {
   case value := <-ch:
       // Successfully received a value
       fmt.Printf("Received value: %v\n", value)
   default:
       // No value available, channel would block
       fmt.Println("No value available")
   }
   ```

   **Basic non-blocking send:**

   ```go
   // Try to send to a channel without blocking
   select {
   case ch <- value:
       // Successfully sent the value
       fmt.Println("Sent value")
   default:
       // Channel would block (e.g., buffer full)
       fmt.Println("Cannot send value, channel busy")
   }
   ```

   **Common use cases for non-blocking operations:**

   **1. Checking channel state**

   ```go
   // Check if a channel is closed without blocking
   func isClosed(ch <-chan T) bool {
       select {
       case _, ok := <-ch:
           return !ok
       default:
           return false // Not closed or would block
       }
   }
   ```

   **2. Polling multiple channels**

   ```go
   // Process all available values from multiple channels without blocking
   func pollChannels(ch1, ch2 <-chan int) {
       for {
           valueReceived := false
           
           select {
           case v, ok := <-ch1:
               if ok {
                   fmt.Printf("Received from ch1: %d\n", v)
                   valueReceived = true
               }
           default:
               // No value on ch1
           }
           
           select {
           case v, ok := <-ch2:
               if ok {
                   fmt.Printf("Received from ch2: %d\n", v)
                   valueReceived = true
               }
           default:
               // No value on ch2
           }
           
           if !valueReceived {
               // No values available on any channel
               break
           }
       }
   }
   ```

   **3. Implementing a try-send or try-receive operation**

   ```go
   // TrySend attempts to send a value to a channel, returning success status
   func TrySend[T any](ch chan<- T, value T) bool {
       select {
       case ch <- value:
           return true
       default:
           return false
       }
   }
   
   // TryReceive attempts to receive a value from a channel
   func TryReceive[T any](ch <-chan T) (T, bool) {
       var zero T
       select {
       case value, ok := <-ch:
           if !ok {
               return zero, false
           }
           return value, true
       default:
           return zero, false
       }
   }
   ```

   **4. Implementing a background worker with non-blocking job submission**

   ```go
   type Worker struct {
       tasks chan Task
       quit  chan struct{}
   }
   
   func NewWorker(bufferSize int) *Worker {
       w := &Worker{
           tasks: make(chan Task, bufferSize),
           quit:  make(chan struct{}),
       }
       
       go w.run()
       return w
   }
   
   func (w *Worker) run() {
       for {
           select {
           case task := <-w.tasks:
               // Process task
               task.Process()
           case <-w.quit:
               return
           }
       }
   }
   
   // SubmitTask tries to add a task without blocking
   func (w *Worker) SubmitTask(task Task) bool {
       select {
       case w.tasks <- task:
           return true
       default:
           // Worker busy (buffer full)
           return false
       }
   }
   
   // Stop the worker
   func (w *Worker) Stop() {
       close(w.quit)
   }
   ```

   **5. Using in combination with timeouts**

   ```go
   // Try to receive for a limited time, then fall back to default value
   func receiveWithTimeout(ch <-chan int, timeout time.Duration, defaultValue int) int {
       select {
       case value := <-ch:
           return value
       case <-time.After(timeout):
           return defaultValue
       default:
           // Immediately return default if nothing available
           return defaultValue
       }
   }
   ```

   **6. Batching send operations**

   ```go
   // Collect values to send in a batch when buffer is available
   func batchSender(values []int, ch chan<- int) {
       var batch []int
       
       for _, v := range values {
           select {
           case ch <- v:
               // Direct send succeeded
               if len(batch) > 0 {
                   // Try to send any batched values first
                   for i := 0; i < len(batch); i++ {
                       select {
                       case ch <- batch[i]:
                           // Sent successfully
                           batch = batch[1:] // Remove from batch
                           i-- // Adjust index
                       default:
                           // Still can't send batched values
                           break
                       }
                   }
               }
           default:
               // Channel would block, add to batch for later
               batch = append(batch, v)
           }
       }
       
       // Try to send remaining batched values
       for _, v := range batch {
           ch <- v // Blocking send for remaining values
       }
   }
   ```

   **7. Implementing a throttling mechanism**

   ```go
   func throttledSender(input <-chan int, output chan<- int, maxPerSecond int) {
       throttle := time.NewTicker(time.Second / time.Duration(maxPerSecond))
       defer throttle.Stop()
       
       for val := range input {
           select {
           case output <- val:
               // Sent without throttling (non-blocking)
           default:
               // Output channel would block, so use throttle
               select {
               case <-throttle.C:
                   // Wait for throttle tick
                   output <- val
               }
           }
       }
   }
   ```

   **Common pitfalls with non-blocking operations:**

   1. **Busy polling**: Using non-blocking operations in a tight loop can waste CPU. Consider adding a small sleep or using a blocking operation with timeout instead.

   2. **Race conditions**: If you check a channel and then act based on that check, conditions might change between the check and the action.

   3. **Complexity**: Non-blocking operations can make control flow complex. Sometimes a simpler blocking approach with proper goroutine management is clearer.

   4. **Overlooking channel closure**: When using non-blocking receives, be sure to check the second return value (`ok`) to detect if the channel is closed.

   Non-blocking channel operations are powerful tools for building responsive concurrent systems. They allow you to check channel readiness without committing to potentially blocking operations, enabling more flexible communication patterns between goroutines.

### 7. Concurrency Patterns: Fan-out, Fan-in

**Concise Explanation:**
Fan-out, Fan-in is a concurrency pattern where work is distributed across multiple goroutines (fan-out) and then their results are collected and combined (fan-in). This pattern enables efficient parallel processing of tasks, particularly when each task is independent and can benefit from concurrent execution. The fan-out stage distributes work to multiple workers, while the fan-in stage aggregates results from all workers into a single stream or result.

**Where to Use:**
- When processing a large number of independent tasks in parallel
- For CPU-bound operations that can benefit from multiple cores
- When performing multiple I/O operations concurrently
- For network requests to different endpoints
- When implementing parallel search or data processing
- For batch processing where each item can be handled independently
- When you need to speed up processing by parallelizing work

**Code Snippet:**
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Task represents a unit of work to be done
type Task int

// Result represents the output of processing a task
type Result struct {
    Task  Task
    Value int
}

// Process simulates doing work on a task
func Process(task Task) Result {
    // Simulate work by sleeping
    time.Sleep(100 * time.Millisecond)
    
    // Compute some result based on the task
    return Result{
        Task:  task,
        Value: int(task) * 2, // Simple transformation
    }
}

// Basic fan-out, fan-in pattern
func main() {
    tasks := make([]Task, 20)
    for i := range tasks {
        tasks[i] = Task(i + 1)
    }
    
    // Set up the pipeline
    numWorkers := 5
    
    // Fan-out: distribute tasks to workers
    taskCh := distributeWork(tasks)
    
    // Create multiple workers to process tasks concurrently
    resultChannels := make([]<-chan Result, numWorkers)
    for i := 0; i < numWorkers; i++ {
        resultChannels[i] = worker(i+1, taskCh)
    }
    
    // Fan-in: collect results from all workers
    resultCh := collectResults(resultChannels)
    
    // Process the final results
    var totalValue int
    results := make(map[Task]int)
    for result := range resultCh {
        results[result.Task] = result.Value
        totalValue += result.Value
        fmt.Printf("Task %d completed with value %d\n", result.Task, result.Value)
    }
    
    fmt.Printf("\nAll tasks completed!\n")
    fmt.Printf("Total tasks: %d\n", len(results))
    fmt.Printf("Total value: %d\n", totalValue)
}

// distributeWork creates a channel and sends all tasks to it
func distributeWork(tasks []Task) <-chan Task {
    taskCh := make(chan Task)
    
    go func() {
        defer close(taskCh)
        for _, task := range tasks {
            taskCh <- task
        }
    }()
    
    return taskCh
}

// worker processes tasks from the input channel and sends results to the output channel
func worker(id int, taskCh <-chan Task) <-chan Result {
    resultCh := make(chan Result)
    
    go func() {
        defer close(resultCh)
        for task := range taskCh {
            fmt.Printf("Worker %d processing task %d\n", id, task)
            result := Process(task)
            resultCh <- result
        }
    }()
    
    return resultCh
}

// collectResults merges results from multiple channels into a single channel
func collectResults(resultChannels []<-chan Result) <-chan Result {
    mergedCh := make(chan Result)
    
    var wg sync.WaitGroup
    wg.Add(len(resultChannels))
    
    // Start a goroutine for each input channel
    for _, ch := range resultChannels {
        go func(ch <-chan Result) {
            defer wg.Done()
            for result := range ch {
                mergedCh <- result
            }
        }(ch)
    }
    
    // Close the merged channel when all input channels are closed
    go func() {
        wg.Wait()
        close(mergedCh)
    }()
    
    return mergedCh
}
```

**Real-World Example:**
A parallel file processor that searches for content across multiple files:

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "path/filepath"
    "strings"
    "sync"
    "time"
)

// SearchRequest represents a file to be searched
type SearchRequest struct {
    FilePath string
    Term     string
}

// SearchResult represents the results of a search
type SearchResult struct {
    FilePath    string
    Term        string
    LineMatches []LineMatch
    Error       error
}

// LineMatch represents a single matching line
type LineMatch struct {
    LineNum  int
    LineText string
}

// parallelSearch performs a parallel search across files
func parallelSearch(root, term string, maxWorkers int) []SearchResult {
    // Step 1: Find all files (could also be parallelized for large directories)
    var files []string
    filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return nil // Skip errors in directory walking
        }
        if !info.IsDir() && filepath.Ext(path) == ".txt" {
            files = append(files, path)
        }
        return nil
    })
    
    fmt.Printf("Found %d files to search\n", len(files))
    
    // Step 2: Fan-out - Create search requests channel
    requestCh := make(chan SearchRequest)
    go func() {
        defer close(requestCh)
        for _, file := range files {
            requestCh <- SearchRequest{FilePath: file, Term: term}
        }
    }()
    
    // Step 3: Fan-out - Create workers to process search requests
    resultChannels := make([]<-chan SearchResult, maxWorkers)
    for i := 0; i < maxWorkers; i++ {
        resultChannels[i] = searchWorker(requestCh)
    }
    
    // Step 4: Fan-in - Merge results from all workers
    resultCh := mergeSearchResults(resultChannels)
    
    // Step 5: Collect all results
    var results []SearchResult
    for result := range resultCh {
        results = append(results, result)
    }
    
    return results
}

// searchWorker searches files for the specified term
func searchWorker(requests <-chan SearchRequest) <-chan SearchResult {
    results := make(chan SearchResult)
    
    go func() {
        defer close(results)
        
        for req := range requests {
            // Search a single file
            result := searchFile(req.FilePath, req.Term)
            results <- result
        }
    }()
    
    return results
}

// searchFile searches a single file for occurrences of a term
func searchFile(filePath, term string) SearchResult {
    result := SearchResult{
        FilePath:    filePath,
        Term:        term,
        LineMatches: []LineMatch{},
    }
    
    file, err := os.Open(filePath)
    if err != nil {
        result.Error = fmt.Errorf("error opening file: %w", err)
        return result
    }
    defer file.Close()
    
    scanner := bufio.NewScanner(file)
    lineNum := 0
    
    for scanner.Scan() {
        lineNum++
        line := scanner.Text()
        
        if strings.Contains(strings.ToLower(line), strings.ToLower(term)) {
            result.LineMatches = append(result.LineMatches, LineMatch{
                LineNum:  lineNum,
                LineText: line,
            })
        }
    }
    
    if err := scanner.Err(); err != nil {
        result.Error = fmt.Errorf("error reading file: %w", err)
    }
    
    return result
}

// mergeSearchResults combines multiple result channels into one
func mergeSearchResults(channels []<-chan SearchResult) <-chan SearchResult {
    var wg sync.WaitGroup
    merged := make(chan SearchResult)
    
    // Start a goroutine for each input channel
    wg.Add(len(channels))
    for _, ch := range channels {
        go func(ch <-chan SearchResult) {
            defer wg.Done()
            for result := range ch {
                merged <- result
            }
        }(ch)
    }
    
    // Close the merged channel when all input channels have closed
    go func() {
        wg.Wait()
        close(merged)
    }()
    
    return merged
}

func main() {
    start := time.Now()
    
    // Define search parameters
    rootDir := "." // Current directory
    searchTerm := "TODO"
    workerCount := 4
    
    // Perform the parallel search
    fmt.Printf("Searching for '%s' with %d workers...\n", searchTerm, workerCount)
    results := parallelSearch(rootDir, searchTerm, workerCount)
    
    // Display results
    fmt.Printf("\nSearch results for '%s':\n", searchTerm)
    fmt.Printf("------------------------\n")
    
    totalMatches := 0
    filesWithMatches := 0
    
    for _, result := range results {
        if result.Error != nil {
            fmt.Printf("Error searching %s: %v\n", result.FilePath, result.Error)
            continue
        }
        
        matchCount := len(result.LineMatches)
        if matchCount > 0 {
            filesWithMatches++
            totalMatches += matchCount
            
            fmt.Printf("\nFile: %s\n", result.FilePath)
            fmt.Printf("  %d matches found\n", matchCount)
            
            // Display up to 3 matches per file
            for i, match := range result.LineMatches {
                if i >= 3 {
                    fmt.Printf("  ... and %d more matches\n", matchCount-3)
                    break
                }
                fmt.Printf("  Line %d: %s\n", match.LineNum, match.LineText)
            }
        }
    }
    
    fmt.Printf("\nSearch complete!\n")
    fmt.Printf("Found %d matches in %d files (searched %d total files)\n", 
        totalMatches, filesWithMatches, len(results))
    fmt.Printf("Search took %v\n", time.Since(start))
}
```

**Common Pitfalls:**
- Not closing channels properly, leading to goroutine leaks
- Creating too many goroutines for the available CPU cores
- Handling errors improperly in worker goroutines
- Not limiting concurrency, potentially overwhelming system resources
- Forgetting to synchronize the fan-in operation
- Creating unnecessary contention with shared resources
- Not considering the overhead of goroutine creation for small tasks

**Confusion Questions:**

1. **Q: How do I determine the optimal number of workers for a fan-out, fan-in pattern?**

   A: Determining the optimal number of workers depends on several factors including the nature of your workload, available hardware resources, and the type of operations being performed. Here's a systematic approach:

   **For CPU-bound tasks:**

   The most efficient number of workers is typically related to the number of available CPU cores:

   ```go
   import "runtime"
   
   // A good default for CPU-bound tasks
   numWorkers := runtime.NumCPU()
   
   // For tasks with significant CPU waiting (OS operations, system calls)
   // you might want to use more workers
   numWorkers := runtime.NumCPU() * 2
   ```

   **For I/O-bound tasks:**

   I/O-bound tasks (like network requests, file operations) spend most of their time waiting, so you can use many more workers than CPU cores:

   ```go
   // For network operations, a higher number might be appropriate
   numWorkers := 50 // or even higher for HTTP clients
   
   // For database connections, consider the connection pool size
   numWorkers := dbPool.MaxConnections()
   ```

   **Practical approach: Benchmark and scale**

   The most reliable way is to benchmark with different worker counts:

   ```go
   func findOptimalWorkerCount(tasks []Task) int {
       bestTime := time.Duration(1<<63 - 1) // Max duration
       bestCount := 1
       
       // Test different worker counts
       for workers := 1; workers <= 2*runtime.NumCPU(); workers++ {
           start := time.Now()
           
           // Run the fan-out, fan-in with this worker count
           _ = processWithWorkers(tasks, workers)
           
           elapsed := time.Since(start)
           fmt.Printf("%d workers: %v\n", workers, elapsed)
           
           if elapsed < bestTime {
               bestTime = elapsed
               bestCount = workers
           }
       }
       
       return bestCount
   }
   ```

   **Adaptive worker pools**

   For long-running services, consider an adaptive approach:

   ```go
   type AdaptivePool struct {
       minWorkers   int
       maxWorkers   int
       activeWorkers int
       taskQueue    chan Task
       resultQueue  chan Result
       controlCh    chan workerControl
       mu           sync.Mutex
   }
   
   func (p *AdaptivePool) adjustWorkerCount() {
       // Monitor metrics like queue length, processing time, system load
       for {
           time.Sleep(time.Second * 5)
           
           p.mu.Lock()
           queueLength := len(p.taskQueue)
           active := p.activeWorkers
           p.mu.Unlock()
           
           if queueLength > 100 && active < p.maxWorkers {
               // Queue is building up, add workers
               p.addWorkers(min(5, p.maxWorkers-active))
           } else if queueLength < 10 && active > p.minWorkers {
               // Queue is small, reduce workers
               p.removeWorkers(min(2, active-p.minWorkers))
           }
       }
   }
   ```

   **Worker count considerations by workload type:**

   1. **CPU-intensive computation** (e.g., image processing, data encoding)
      - Start with worker count = CPU cores
      - Avoid over-subscription that causes context switching overhead
      - Consider GOMAXPROCS setting

   2. **I/O waiting operations** (e.g., HTTP requests, file reads)
      - Can use many more workers than CPU cores
      - Limit based on external resource constraints (connection limits, file handles)
      - Monitor system for resource exhaustion

   3. **Mixed workloads**
      - For tasks with both CPU and I/O components
      - Start with worker count = CPU cores * (1 + I/O percentage)
      - Example: For tasks that are ~50% I/O waiting, try CPU cores * 1.5

   4. **Resource-constrained environments**
      - Consider memory usage per worker
      - Account for goroutine stack size (initial 2KB, can grow)
      - Limit based on available memory

   **Practical example: Worker sizing strategy**

   ```go
   func calculateWorkerCount(workType string) int {
       cpus := runtime.NumCPU()
       
       switch workType {
       case "cpu_intensive":
           return cpus
           
       case "io_intensive":
           // For I/O bound, we can have many more workers
           return cpus * 10
           
       case "balanced":
           return cpus * 2
           
       case "memory_constrained":
           // If each worker uses significant memory
           memoryPerWorker := 50 * 1024 * 1024 // 50MB per worker
           totalMemory := getTotalMemory()
           maxWorkersByMemory := totalMemory / memoryPerWorker / 2 // Use half of memory max
           
           return min(maxWorkersByMemory, cpus*2)
           
       default:
           return cpus
       }
   }
   ```

   Remember that the optimal number varies based on your specific workload and environment. Start with a reasonable value based on the guidelines above, then measure and adjust based on real performance metrics.

2. **Q: How do I handle errors in a fan-out, fan-in pattern?**

   A: Handling errors in a fan-out, fan-in pattern requires careful consideration to ensure that errors from worker goroutines are properly propagated and don't get lost. There are several effective strategies:

   **Strategy 1: Include errors in the result type**

   This is the most common approach, where each result includes an error field:

   ```go
   type Result struct {
       TaskID int
       Data   interface{}
       Error  error
   }
   
   func worker(tasks <-chan Task) <-chan Result {
       resultCh := make(chan Result)
       
       go func() {
           defer close(resultCh)
           
           for task := range tasks {
               // Try to process
               data, err := process(task)
               
               // Always return a result, with error if it occurred
               resultCh <- Result{
                   TaskID: task.ID,
                   Data:   data,
                   Error:  err,
               }
           }
       }()
       
       return resultCh
   }
   
   // In the consumer
   for result := range resultCh {
       if result.Error != nil {
           log.Printf("Task %d failed: %v", result.TaskID, result.Error)
           // Handle error - retry, skip, or abort
           continue
       }
       
       // Process successful result
       handleSuccess(result.Data)
   }
   ```

   **Strategy 2: Separate error channel**

   Use a dedicated channel for errors:

   ```go
   func processInParallel(tasks []Task, numWorkers int) ([]Output, []error) {
       taskCh := make(chan Task)
       resultCh := make(chan Output)
       errCh := make(chan error)
       
       // Start workers
       var wg sync.WaitGroup
       wg.Add(numWorkers)
       for i := 0; i < numWorkers; i++ {
           go func() {
               defer wg.Done()
               
               for task := range taskCh {
                   output, err := processTask(task)
                   if err != nil {
                       errCh <- fmt.Errorf("task %d: %w", task.ID, err)
                   } else {
                       resultCh <- output
                   }
               }
           }()
       }
       
       // Send all tasks
       go func() {
           for _, task := range tasks {
               taskCh <- task
           }
           close(taskCh)
           
           // Close channels when all workers done
           wg.Wait()
           close(resultCh)
           close(errCh)
       }()
       
       // Collect results and errors
       var outputs []Output
       var errors []error
       
       // Use a wait group to collect from both channels
       var collectorWg sync.WaitGroup
       collectorWg.Add(2)
       
       go func() {
           defer collectorWg.Done()
           for output := range resultCh {
               outputs = append(outputs, output)
           }
       }()
       
       go func() {
           defer collectorWg.Done()
           for err := range errCh {
               errors = append(errors, err)
           }
       }()
       
       collectorWg.Wait()
       return outputs, errors
   }
   ```

   **Strategy 3: Error aggregation with error groups**

   Using the `errgroup` package from `golang.org/x/sync`:

   ```go
   import (
       "context"
       
       "golang.org/x/sync/errgroup"
   )
   
   func processWithErrorGroup(ctx context.Context, tasks []Task) ([]Result, error) {
       g, ctx := errgroup.WithContext(ctx)
       
       // Channel for results
       results := make(chan Result, len(tasks))
       
       // Process tasks
       for _, task := range tasks {
           task := task  // Capture for closure
           g.Go(func() error {
               result, err := processTask(ctx, task)
               if err != nil {
                   return fmt.Errorf("task %v: %w", task.ID, err)
               }
               
               select {
               case results <- result:
                   // Result sent
               case <-ctx.Done():
                   // Context canceled
               }
               return nil
           })
       }
       
       // Close results channel when all goroutines complete
       go func() {
           g.Wait()
           close(results)
       }()
       
       // Collect results
       var collected []Result
       for r := range results {
           collected = append(collected, r)
       }
       
       // If any goroutine returns an error, g.Wait() will return that error
       if err := g.Wait(); err != nil {
           return collected, err
       }
       
       return collected, nil
   }
   ```

   **Strategy 4: Context cancellation on first error**

   Cancel all operations when any worker encounters an error:

   ```go
   func processWithCancellation(tasks []Task) ([]Result, error) {
       // Create cancellable context
       ctx, cancel := context.WithCancel(context.Background())
       defer cancel()
       
       taskCh := make(chan Task)
       resultCh := make(chan Result)
       errCh := make(chan error, 1) // Buffered to avoid blocking
       
       // Launch workers
       var wg sync.WaitGroup
       const numWorkers = 5
       wg.Add(numWorkers)
       
       for i := 0; i < numWorkers; i++ {
           go func() {
               defer wg.Done()
               
               for {
                   select {
                   case task, ok := <-taskCh:
                       if !ok {
                           return
                       }
                       
                       result, err := processTask(ctx, task)
                       if err != nil {
                           select {
                           case errCh <- fmt.Errorf("task %d: %w", task.ID, err):
                               cancel() // Cancel other workers
                           default:
                               // Error channel full, another error was already sent
                           }
                           return
                       }
                       
                       select {
                       case resultCh <- result:
                           // Result sent
                       case <-ctx.Done():
                           return
                       }
                       
                   case <-ctx.Done():
                       return
                   }
               }
           }()
       }
       
       // Send tasks
       go func() {
           defer close(taskCh)
           for _, task := range tasks {
               select {
               case taskCh <- task:
                   // Task sent
               case <-ctx.Done():
                   return
               }
           }
       }()
       
       // Close result channel when all workers done
       go func() {
           wg.Wait()
           close(resultCh)
           close(errCh)
       }()
       
       // Collect results
       var results []Result
       var resultErr error
       
       for {
           select {
           case result, ok := <-resultCh:
               if !ok {
                   // Result channel closed
                   return results, resultErr
               }
               results = append(results, result)
               
           case err, ok := <-errCh:
               if ok && resultErr == nil {
                   resultErr = err
               }
               if !ok {
                   // Error channel closed
                   return results, resultErr
               }
           }
       }
   }
   ```

   **Strategy 5: Process-as-much-as-possible approach**

   Continue processing despite errors:

   ```go
   func processAllTasks(tasks []Task) ([]Result, []error) {
       taskCh := make(chan Task)
       resultCh := make(chan Result)
       errorCh := make(chan error)
       
       // Start workers
       var wg sync.WaitGroup
       for i := 0; i < 5; i++ {
           wg.Add(1)
           go func() {
               defer wg.Done()
               for task := range taskCh {
                   result, err := processTask(task)
                   if err != nil {
                       errorCh <- fmt.Errorf("task %d: %w", task.ID, err)
                   } else {
                       resultCh <- result
                   }
               }
           }()
       }
       
       // Send tasks and close channels when done
       go func() {
           for _, task := range tasks {
               taskCh <- task
           }
           close(taskCh)
           
           wg.Wait()
           close(resultCh)
           close(errorCh)
       }()
       
       // Collect results and errors
       var results []Result
       var errors []error
       
       remaining := 2 // Number of channels to collect from
       for remaining > 0 {
           select {
           case result, ok := <-resultCh:
               if !ok {
                   resultCh = nil
                   remaining--
                   continue
               }
               results = append(results, result)
               
           case err, ok := <-errorCh:
               if !ok {
                   errorCh = nil
                   remaining--
                   continue
               }
               errors = append(errors, err)
           }
       }
       
       return results, errors
   }
   ```

   **Choosing a strategy:**

   1. **Results with error fields** - Best for most situations where each task can fail independently
   2. **Separate error channel** - Good when you want to process errors differently from results
   3. **Error groups** - Best when any error should abort the entire operation
   4. **Context cancellation** - Good for early termination on first error
   5. **Process-all approach** - Best for batch jobs where maximum completion is preferred

   **Best practices for error handling in fan-out, fan-in:**

   1. **Preserve error context** - Wrap errors with task identifiers and additional context
   2. **Consider error handling in the consumer** - Design for how errors will be used
   3. **Avoid silent failures** - Ensure errors are always captured somewhere
   4. **Manage resources** - Ensure proper cleanup even when errors occur
   5. **Test error cases** - Explicitly test how your pattern handles failures

   The right strategy depends on your application's error handling requirements - whether tasks can continue independently despite errors in some tasks, or whether any error requires stopping all processing immediately.

3. **Q: How does the fan-out, fan-in pattern compare to a worker pool?**

   A: Fan-out, fan-in and worker pools are related concurrency patterns with some key differences in their implementation, flexibility, and use cases.

   **Fan-Out, Fan-In Pattern**

   ```go
   func fanOutFanIn(tasks []Task, numWorkers int) <-chan Result {
       // Fan-Out: Distribute tasks
       tasksCh := make(chan Task)
       go func() {
           defer close(tasksCh)
           for _, task := range tasks {
               tasksCh <- task
           }
       }()
       
       // Create multiple workers, each returning its own results channel
       channels := make([]<-chan Result, numWorkers)
       for i := 0; i < numWorkers; i++ {
           channels[i] = worker(tasksCh)
       }
       
       // Fan-In: Combine result channels
       return merge(channels)
   }
   
   // Each worker processes tasks and returns its own results channel
   func worker(tasksCh <-chan Task) <-chan Result {
       resultsCh := make(chan Result)
       go func() {
           defer close(resultsCh)
           for task := range tasksCh {
               result := process(task)
               resultsCh <- result
           }
       }()
       return resultsCh
   }
   
   // Merge combines multiple channels into one
   func merge(channels []<-chan Result) <-chan Result {
       mergedCh := make(chan Result)
       var wg sync.WaitGroup
       
       // Start a goroutine for each input channel
       for _, ch := range channels {
           wg.Add(1)
           go func(ch <-chan Result) {
               defer wg.Done()
               for result := range ch {
                   mergedCh <- result
               }
           }(ch)
       }
       
       // Close merged channel once all input channels are done
       go func() {
           wg.Wait()
           close(mergedCh)
       }()
       
       return mergedCh
   }
   ```

   **Worker Pool Pattern**

   ```go
   func workerPool(tasks []Task, numWorkers int) <-chan Result {
       tasksCh := make(chan Task)
       resultsCh := make(chan Result)
       
       // Start fixed number of workers
       var wg sync.WaitGroup
       wg.Add(numWorkers)
       for i := 0; i < numWorkers; i++ {
           go func() {
               defer wg.Done()
               for task := range tasksCh {
                   result := process(task)
                   resultsCh <- result
               }
           }()
       }
       
       // Close results channel when all workers done
       go func() {
           wg.Wait()
           close(resultsCh)
       }()
       
       // Feed tasks to the workers
       go func() {
           defer close(tasksCh)
           for _, task := range tasks {
               tasksCh <- task
           }
       }()
       
       return resultsCh
   }
   ```

   **Key Differences:**

   1. **Channel Structure**
      - **Fan-out, fan-in**: Each worker has its own output channel, which are then merged
      - **Worker pool**: All workers share a single output channel

   2. **Flexibility**
      - **Fan-out, fan-in**: More modular, easier to compose pipeline stages
      - **Worker pool**: Simpler implementation, less overhead

   3. **Use Cases**
      - **Fan-out, fan-in**: Better for complex data flow with multiple stages
      - **Worker pool**: Better for simple task distribution

   4. **Memory Usage**
      - **Fan-out, fan-in**: More channels, potentially higher overhead
      - **Worker pool**: Fewer channels, potentially lower overhead

   5. **Control Flow**
      - **Fan-out, fan-in**: More explicit about data flow between stages
      - **Worker pool**: Simpler loop structure

   **Choosing Between the Patterns**

   **Use Fan-Out, Fan-In when:**

   1. **Building a processing pipeline** with multiple stages
      ```go
      stage1Results := fanOutFanIn(inputData, 5, stage1Process)
      stage2Results := fanOutFanIn(stage1Results, 3, stage2Process)
      finalResults := fanOutFanIn(stage2Results, 2, finalProcess)
      ```

   2. **Different workers perform different roles** in the pipeline
      ```go
      extractorResults := extractorFanOut(files)
      transformerResults := transformerFanOut(extractorResults)
      loaderResults := loaderFanOut(transformerResults)
      ```

   3. **Encapsulating stages** of processing
      ```go
      // Each stage returns a channel and can be composed
      func ProcessStage1(in <-chan Data) <-chan Result {
          return fanOutFanIn(in, 5, processFunc)
      }
      ```

   **Use Worker Pool when:**

   1. **Simple task distribution** is needed
      ```go
      results := workerPool(tasks, 10)
      for result := range results {
          // Process result
      }
      ```

   2. **Managing resource usage** is important
      ```go
      // Limit concurrency to avoid overwhelming resources
      pool := NewWorkerPool(maxConcurrency)
      pool.Start()
      ```

   3. **Long-running workers** that process multiple tasks
      ```go
      pool := NewWorkerPool(10)
      pool.Start()
      
      // Submit tasks over time
      for task := range incomingTasks {
          pool.Submit(task)
      }
      ```

   **Real-world Hybrid Example**

   Many real-world implementations combine aspects of both patterns:

   ```go
   type WorkerPool struct {
       workerCount  int
       taskQueue    chan Task
       resultQueue  chan Result
       errorQueue   chan error
       quit         chan struct{}
       wg           sync.WaitGroup
   }
   
   func NewWorkerPool(workerCount int) *WorkerPool {
       return &WorkerPool{
           workerCount:  workerCount,
           taskQueue:    make(chan Task, workerCount*2),
           resultQueue:  make(chan Result, workerCount*2),
           errorQueue:   make(chan error, workerCount),
           quit:         make(chan struct{}),
       }
   }
   
   func (p *WorkerPool) Start() {
       // Start workers
       p.wg.Add(p.workerCount)
       for i := 0; i < p.workerCount; i++ {
           go p.worker(i)
       }
   }
   
   func (p *WorkerPool) worker(id int) {
       defer p.wg.Done()
       
       for {
           select {
           case <-p.quit:
               return
           case task, ok := <-p.taskQueue:
               if !ok {
                   return
               }
               
               // Process task
               result, err := task.Process()
               if err != nil {
                   p.errorQueue <- fmt.Errorf("worker %d: %w", id, err)
               } else {
                   p.resultQueue <- result
               }
           }
       }
   }
   
   func (p *WorkerPool) Submit(task Task) {
       p.taskQueue <- task
   }
   
   func (p *WorkerPool) Results() <-chan Result {
       return p.resultQueue
   }
   
   func (p *WorkerPool) Errors() <-chan error {
       return p.errorQueue
   }
   
   func (p *WorkerPool) Stop() {
       close(p.quit)
       p.wg.Wait()
       close(p.resultQueue)
       close(p.errorQueue)
   }
   
   // Usage
   pool := NewWorkerPool(10)
   pool.Start()
   
   go func() {
       for _, task := range tasks {
           pool.Submit(task)
       }
       pool.Stop()
   }()
   
   for result := range pool.Results() {
       // Process results
   }
   
   for err := range pool.Errors() {
       // Handle errors
   }
   ```

   In practice, many Go applications use a blend of these patterns, adapting them to specific needs. The choice depends on your application's requirements for clarity, flexibility, and performance.

### 8. Pipeline Pattern

**Concise Explanation:**
The pipeline pattern in Go structures data processing as a series of stages connected by channels, where each stage receives values, performs some operations, and sends the results to the next stage. Each stage typically runs in its own goroutine, allowing for concurrent processing. This pattern enables clean separation of concerns, making the code more modular and easier to maintain while naturally handling concurrent operations.

**Where to Use:**
- For sequential data transformations that can operate concurrently
- When processing large datasets in smaller chunks
- To separate distinct processing steps in data workflows
- For ETL (Extract, Transform, Load) operations
- When performing multiple operations on streams of data
- To create reusable processing components
- For handling I/O operations while keeping CPU busy

**Code Snippet:**
```go
package main

import (
    "fmt"
    "math"
    "sync"
)

// Simple pipeline that generates numbers, squares them, and filters out odd squares
func main() {
    // Pipeline stages
    done := make(chan struct{})
    defer close(done)
    
    // Stage 1: Generate integers
    nums := generate(done, 1, 10)
    
    // Stage 2: Square the numbers
    squares := square(done, nums)
    
    // Stage 3: Keep only even squares
    evenSquares := filter(done, squares, func(n int) bool {
        return n%2 == 0
    })
    
    // Final stage: Consume and print the results
    for n := range evenSquares {
        fmt.Printf("%d ", n)
    }
    fmt.Println()
    
    // More complex example with fan-out, fan-in for the squaring operation
    fmt.Println("Pipeline with fan-out, fan-in:")
    
    // Stage 1: Generate more integers
    moreNums := generate(done, 1, 20)
    
    // Stage 2: Square numbers using multiple goroutines (fan-out, fan-in)
    squaresParallel := squareParallel(done, moreNums, 4)
    
    // Stage 3: Filter even squares
    evenSquaresParallel := filter(done, squaresParallel, func(n int) bool {
        return n%2 == 0
    })
    
    // Final stage: Consume and print the results
    sum := 0
    for n := range evenSquaresParallel {
        sum += n
        fmt.Printf("%d ", n)
    }
    fmt.Println("\nSum:", sum)
}

// generate creates a pipeline stage that emits integers from start to end
func generate(done <-chan struct{}, start, end int) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        for i := start; i <= end; i++ {
            select {
            case <-done:
                return
            case out <- i:
                // Value sent
            }
        }
    }()
    
    return out
}

// square creates a pipeline stage that receives integers and emits their squares
func square(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        for n := range in {
            select {
            case <-done:
                return
            case out <- n * n:
                // Squared value sent
            }
        }
    }()
    
    return out
}

// filter creates a pipeline stage that filters values based on a predicate
func filter(done <-chan struct{}, in <-chan int, predicate func(int) bool) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        for n := range in {
            if predicate(n) {
                select {
                case <-done:
                    return
                case out <- n:
                    // Filtered value sent
                }
            }
        }
    }()
    
    return out
}

// squareParallel demonstrates fan-out, fan-in for parallel squaring
func squareParallel(done <-chan struct{}, in <-chan int, numWorkers int) <-chan int {
    // Create channels for distribution of work
    channels := make([]<-chan int, numWorkers)
    
    // Fan out: Create multiple workers for the square operation
    for i := 0; i < numWorkers; i++ {
        channels[i] = square(done, in)
    }
    
    // Fan in: Combine results from all workers
    return merge(done, channels...)
}

// merge combines multiple channels into a single channel
func merge(done <-chan struct{}, channels ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)
    
    // Function to collect output from a channel
    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            select {
            case <-done:
                return
            case out <- n:
                // Value sent
            }
        }
    }
    
    // Start a goroutine for each input channel
    wg.Add(len(channels))
    for _, c := range channels {
        go output(c)
    }
    
    // Start a goroutine to close the output channel when all input goroutines are done
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}
```

**Real-World Example:**
A data processing pipeline for analyzing log files:

```go
package main

import (
    "bufio"
    "context"
    "encoding/json"
    "fmt"
    "os"
    "path/filepath"
    "regexp"
    "strings"
    "sync"
    "time"
)

// LogEntry represents a parsed log line
type LogEntry struct {
    Timestamp time.Time
    Level     string
    Message   string
    Source    string
    IP        string
    Path      string
    UserAgent string
    ResponseTime int
    StatusCode   int
    RawLine   string
}

// LogStats holds aggregated statistics about logs
type LogStats struct {
    TotalRequests    int
    ErrorCount       int
    SlowResponses    int // Responses over 500ms
    StatusCodeCounts map[int]int
    PathCounts       map[string]int
    IPAddresses      map[string]int
    MinResponseTime  int
    MaxResponseTime  int
    AvgResponseTime  float64
    TotalTime        int
    TopPaths         []PathStat
}

// PathStat holds statistics for a specific path
type PathStat struct {
    Path         string
    Count        int
    AvgTime      float64
    ErrorCount   int
}

// LogProcessor implements a pipeline for processing log files
type LogProcessor struct {
    concurrency int
}

// NewLogProcessor creates a new log processor with the specified concurrency
func NewLogProcessor(concurrency int) *LogProcessor {
    return &LogProcessor{
        concurrency: concurrency,
    }
}

// FindLogs finds all log files in the specified directory
func (p *LogProcessor) FindLogs(ctx context.Context, root string) <-chan string {
    out := make(chan string)
    
    go func() {
        defer close(out)
        
        err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            
            // Check if the context has been canceled
            select {
            case <-ctx.Done():
                return ctx.Err()
            default:
                // Continue
            }
            
            // Check if it's a log file
            if !info.IsDir() && (strings.HasSuffix(path, ".log") ||
                                strings.HasSuffix(path, ".txt")) {
                select {
                case out <- path:
                    // Sent file path
                case <-ctx.Done():
                    return ctx.Err()
                }
            }
            
            return nil
        })
        
        if err != nil && err != ctx.Err() {
            fmt.Fprintf(os.Stderr, "Error walking directory: %v\n", err)
        }
    }()
    
    return out
}

// ReadLogFiles reads and outputs log lines from multiple files
func (p *LogProcessor) ReadLogFiles(ctx context.Context, paths <-chan string) <-chan string {
    out := make(chan string)
    
    // Fan-out: multiple goroutines reading files in parallel
    var wg sync.WaitGroup
    
    // Limit concurrency
    semaphore := make(chan struct{}, p.concurrency)
    
    go func() {
        defer close(out)
        
        for path := range paths {
            // Check if the context has been canceled
            select {
            case <-ctx.Done():
                return
            default:
                // Continue
            }
            
            // Acquire semaphore slot
            semaphore <- struct{}{}
            
            wg.Add(1)
            go func(path string) {
                defer wg.Done()
                defer func() { <-semaphore }() // Release semaphore slot
                
                // Open and read the file
                file, err := os.Open(path)
                if err != nil {
                    fmt.Fprintf(os.Stderr, "Error opening %s: %v\n", path, err)
                    return
                }
                defer file.Close()
                
                scanner := bufio.NewScanner(file)
                for scanner.Scan() {
                    line := scanner.Text()
                    
                    select {
                    case out <- line:
                        // Line sent
                    case <-ctx.Done():
                        return
                    }
                }
                
                if err := scanner.Err(); err != nil {
                    fmt.Fprintf(os.Stderr, "Error reading %s: %v\n", path, err)
                }
            }(path)
        }
        
        wg.Wait()
    }()
    
    return out
}

// ParseLogLines parses log lines into structured LogEntry objects
func (p *LogProcessor) ParseLogLines(ctx context.Context, lines <-chan string) <-chan LogEntry {
    out := make(chan LogEntry)
    
    // Example regex patterns for parsing (simplified)
    timestampPattern := regexp.MustCompile(`(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})`)
    ipPattern := regexp.MustCompile(`(\d+\.\d+\.\d+\.\d+)`)
    statusPattern := regexp.MustCompile(`status=(\d+)`)
    timePattern := regexp.MustCompile(`time=(\d+)ms`)
    pathPattern := regexp.MustCompile(`path="([^"]+)"`)
    levelPattern := regexp.MustCompile(`level=(\w+)`)
    
    go func() {
        defer close(out)
        
        for line := range lines {
            // Check context
            select {
            case <-ctx.Done():
                return
            default:
                // Continue processing
            }
            
            // Skip empty lines
            if line == "" {
                continue
            }
            
            // Extract information using regex
            entry := LogEntry{RawLine: line}
            
            // Extract timestamp
            if match := timestampPattern.FindStringSubmatch(line); len(match) > 1 {
                if ts, err := time.Parse("2006-01-02 15:04:05", match[1]); err == nil {
                    entry.Timestamp = ts
                }
            }
            
            // Extract IP address
            if match := ipPattern.FindStringSubmatch(line); len(match) > 1 {
                entry.IP = match[1]
            }
            
            // Extract status code
            if match := statusPattern.FindStringSubmatch(line); len(match) > 1 {
                fmt.Sscanf(match[1], "%d", &entry.StatusCode)
            }
            
            // Extract response time
            if match := timePattern.FindStringSubmatch(line); len(match) > 1 {
                fmt.Sscanf(match[1], "%d", &entry.ResponseTime)
            }
            
            // Extract path
            if match := pathPattern.FindStringSubmatch(line); len(match) > 1 {
                entry.Path = match[1]
            }
            
            // Extract log level
            if match := levelPattern.FindStringSubmatch(line); len(match) > 1 {
                entry.Level = match[1]
            }
            
            // Extract message (simplified)
            parts := strings.Split(line, "msg=")
            if len(parts) > 1 {
                msgPart := parts[1]
                if strings.Contains(msgPart, `"`) {
                    quoted := strings.Split(msgPart, `"`)
                    if len(quoted) > 1 {
                        entry.Message = quoted[1]
                    }
                }
            }
            
            // Send to output channel
            select {
            case out <- entry:
                // Sent successfully
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return out
}

// FilterLogEntries filters log entries based on criteria
func (p *LogProcessor) FilterLogEntries(
    ctx context.Context,
    entries <-chan LogEntry,
    minLevel string,
    pathFilter string,
) <-chan LogEntry {
    out := make(chan LogEntry)
    
    // Convert minLevel to priority
    levelPriority := map[string]int{
        "debug": 0,
        "info":  1,
        "warn":  2,
        "error": 3,
        "fatal": 4,
    }
    minPriority := levelPriority[strings.ToLower(minLevel)]
    
    go func() {
        defer close(out)
        
        for entry := range entries {
            // Check context
            select {
            case <-ctx.Done():
                return
            default:
                // Continue
            }
            
            // Apply level filter
            entryPriority := levelPriority[strings.ToLower(entry.Level)]
            if minPriority > 0 && entryPriority < minPriority {
                continue
            }
            
            // Apply path filter
            if pathFilter != "" && !strings.Contains(entry.Path, pathFilter) {
                continue
            }
            
            // Send filtered entry
            select {
            case out <- entry:
                // Sent successfully
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return out
}

// AnalyzeLogs processes multiple log entries to generate statistics
func (p *LogProcessor) AnalyzeLogs(ctx context.Context, entries <-chan LogEntry) <-chan LogStats {
    out := make(chan LogStats)
    
    go func() {
        defer close(out)
        
        stats := LogStats{
            StatusCodeCounts: make(map[int]int),
            PathCounts:       make(map[string]int),
            IPAddresses:      make(map[string]int),
            MinResponseTime:  -1,
        }
        
        // For calculating path-specific statistics
        pathStats := make(map[string]*struct {
            TotalTime  int
            Count      int
            ErrorCount int
        })
        
        // Process each log entry
        for entry := range entries {
            // Update total requests
            stats.TotalRequests++
            
            // Check for errors (status code >= 400)
            if entry.StatusCode >= 400 {
                stats.ErrorCount++
            }
            
            // Check for slow responses
            if entry.ResponseTime > 500 {
                stats.SlowResponses++
            }
            
            // Update status code counts
            stats.StatusCodeCounts[entry.StatusCode]++
            
            // Update path counts
            stats.PathCounts[entry.Path]++
            
            // Update IP address counts
            stats.IPAddresses[entry.IP]++
            
            // Update response time statistics
            if stats.MinResponseTime == -1 || entry.ResponseTime < stats.MinResponseTime {
                stats.MinResponseTime = entry.ResponseTime
            }
            if entry.ResponseTime > stats.MaxResponseTime {
                stats.MaxResponseTime = entry.ResponseTime
            }
            stats.TotalTime += entry.ResponseTime
            
            // Update path-specific statistics
            pathStat, exists := pathStats[entry.Path]
            if !exists {
                pathStat = &struct {
                    TotalTime  int
                    Count      int
                    ErrorCount int
                }{}
                pathStats[entry.Path] = pathStat
            }
            pathStat.Count++
            pathStat.TotalTime += entry.ResponseTime
            if entry.StatusCode >= 400 {
                pathStat.ErrorCount++
            }
            
            // Check if context is done
            select {
            case <-ctx.Done():
                return
            default:
                // Continue processing
            }
        }
        
        // Calculate average response time
        if stats.TotalRequests > 0 {
            stats.AvgResponseTime = float64(stats.TotalTime) / float64(stats.TotalRequests)
        }
        
        // Populate top paths statistics
        stats.TopPaths = make([]PathStat, 0, len(pathStats))
        for path, pathStat := range pathStats {
            stats.TopPaths = append(stats.TopPaths, PathStat{
                Path:       path,
                Count:      pathStat.Count,
                AvgTime:    float64(pathStat.TotalTime) / float64(pathStat.Count),
                ErrorCount: pathStat.ErrorCount,
            })
        }
        
        // Sort top paths by count (simplified - would use sort.Slice in a full implementation)
        // This is a simple bubble sort for demonstration
        for i := 0; i < len(stats.TopPaths)-1; i++ {
            for j := i + 1; j < len(stats.TopPaths); j++ {
                if stats.TopPaths[i].Count < stats.TopPaths[j].Count {
                    stats.TopPaths[i], stats.TopPaths[j] = stats.TopPaths[j], stats.TopPaths[i]
                }
            }
        }
        
        // Limit to top 10 paths
        if len(stats.TopPaths) > 10 {
            stats.TopPaths = stats.TopPaths[:10]
        }
        
        // Send the final statistics
        select {
        case out <- stats:
            // Stats sent successfully
        case <-ctx.Done():
            return
        }
    }()
    
    return out
}

// OutputStats formats and outputs the statistics
func (p *LogProcessor) OutputStats(ctx context.Context, statsCh <-chan LogStats) <-chan string {
    out := make(chan string)
    
    go func() {
        defer close(out)
        
        for stats := range statsCh {
            // Create a formatted report
            report := &strings.Builder{}
            
            fmt.Fprintf(report, "\n=== Log Analysis Report ===\n\n")
            fmt.Fprintf(report, "Total Requests: %d\n", stats.TotalRequests)
            fmt.Fprintf(report, "Error Count: %d (%.1f%%)\n", 
                stats.ErrorCount, float64(stats.ErrorCount)/float64(stats.TotalRequests)*100)
            fmt.Fprintf(report, "Slow Responses (>500ms): %d (%.1f%%)\n",
                stats.SlowResponses, float64(stats.SlowResponses)/float64(stats.TotalRequests)*100)
            fmt.Fprintf(report, "Response Time: Min=%dms, Max=%dms, Avg=%.1fms\n",
                stats.MinResponseTime, stats.MaxResponseTime, stats.AvgResponseTime)
            
            // Status code breakdown
            fmt.Fprintf(report, "\nStatus Code Breakdown:\n")
            for code, count := range stats.StatusCodeCounts {
                fmt.Fprintf(report, "  %d: %d (%.1f%%)\n", code, count, float64(count)/float64(stats.TotalRequests)*100)
            }
            
            // Top paths
            fmt.Fprintf(report, "\nTop Paths:\n")
            for i, pathStat := range stats.TopPaths {
                fmt.Fprintf(report, "  %d. %s - %d requests, %.1fms avg, %d errors\n",
                    i+1, pathStat.Path, pathStat.Count, pathStat.AvgTime, pathStat.ErrorCount)
            }
            
            // Top IPs (limited to 5)
            fmt.Fprintf(report, "\nTop IP Addresses:\n")
            count := 0
            for ip, hits := range stats.IPAddresses {
                if count >= 5 {
                    break
                }
                fmt.Fprintf(report, "  %s: %d requests\n", ip, hits)
                count++
            }
            
            // Send the report
            select {
            case out <- report.String():
                // Report sent
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return out
}

// Process runs the full log processing pipeline
func (p *LogProcessor) Process(ctx context.Context, logDir string, minLevel string, pathFilter string) string {
    // Create a pipeline
    logFiles := p.FindLogs(ctx, logDir)
    logLines := p.ReadLogFiles(ctx, logFiles)
    logEntries := p.ParseLogLines(ctx, logLines)
    filteredEntries := p.FilterLogEntries(ctx, logEntries, minLevel, pathFilter)
    statsCh := p.AnalyzeLogs(ctx, filteredEntries)
    reportCh := p.OutputStats(ctx, statsCh)
    
    // Get the final report
    var finalReport string
    for report := range reportCh {
        finalReport = report
    }
    
    return finalReport
}

func main() {
    // Set up context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    // Create and run the log processor
    processor := NewLogProcessor(4) // Use 4 worker goroutines
    
    // Process logs
    report := processor.Process(ctx, "./logs", "info", "/api")
    
    // Print the final report
    fmt.Println(report)
    
    // Alternatively, write to file
    if err := os.WriteFile("log_report.txt", []byte(report), 0644); err != nil {
        fmt.Printf("Error writing report: %v\n", err)
    }
}
```

**Common Pitfalls:**
- Not handling cancellation correctly (forgetting to check for context cancellation)
- Incorrect channel closing, leading to panics or deadlocks
- Memory leaks from abandoned goroutines when pipelines shut down early
- Not buffering channels when appropriate, causing unnecessary blocking
- Creating too many goroutines, overwhelming system resources
- Not handling errors properly through the pipeline stages
- Creating complex pipelines that are difficult to debug
- Missing backpressure mechanisms for processing imbalances

**Confusion Questions:**

1. **Q: How do I handle errors in a pipeline pattern?**

   A: Handling errors in a pipeline pattern requires careful consideration of error propagation, cancellation, and recovery. There are several approaches:

   **Approach 1: Include error in the output values**

   Each stage produces values that contain both the result and any error:

   ```go
   type Result struct {
       Value interface{}
       Err   error
   }

   func transform(in <-chan int) <-chan Result {
       out := make(chan Result)
       
       go func() {
           defer close(out)
           
           for v := range in {
               result, err := doTransformation(v)
               out <- Result{Value: result, Err: err}
           }
       }()
       
       return out
   }
   
   // In the final consumer:
   for result := range finalStage {
       if result.Err != nil {
           log.Printf("Error: %v", result.Err)
           continue // or handle differently
       }
       
       // Use result.Value
       fmt.Println(result.Value)
   }
   ```

   **Approach 2: Use context for cancellation**

   Use a context to signal errors upstream and downstream:

   ```go
   func processStage(ctx context.Context, in <-chan int) (<-chan int, <-chan error) {
       out := make(chan int)
       errCh := make(chan error, 1) // Buffer for error
       
       go func() {
           defer close(out)
           defer close(errCh)
           
           for v := range in {
               // Check if we should stop
               select {
               case <-ctx.Done():
                   return
               default:
                   // Continue processing
               }
               
               result, err := doTransformation(v)
               if err != nil {
                   errCh <- fmt.Errorf("processing %d: %w", v, err)
                   return
               }
               
               select {
               case out <- result:
                   // Value sent successfully
               case <-ctx.Done():
                   return
               }
           }
       }()
       
       return out, errCh
   }
   
   // Usage:
   ctx, cancel := context.WithCancel(context.Background())
   defer cancel()
   
   out1, errs1 := stage1(ctx, input)
   out2, errs2 := stage2(ctx, out1)
   
   // Error handling goroutine
   go func() {
       select {
       case err := <-errs1:
           log.Printf("Stage 1 error: %v", err)
           cancel() // Cancel the entire pipeline
       case err := <-errs2:
           log.Printf("Stage 2 error: %v", err)
           cancel()
       }
   }()
   
   // Process results
   for result := range out2 {
       // Process result
   }
   ```

   **Approach 3: Separate error channels per stage**

   Each stage has its own error channel:

   ```go
   func stage(in <-chan int, errIn <-chan error) (<-chan int, <-chan error) {
       out := make(chan int)
       errOut := make(chan error, 1) // Buffer for error
       
       go func() {
           defer close(out)
           defer close(errOut)
           
           for {
               select {
               case v, ok := <-in:
                   if !ok {
                       // Input channel closed
                       return
                   }
                   
                   result, err := process(v)
                   if err != nil {
                       errOut <- fmt.Errorf("processing error: %w", err)
                       return
                   }
                   
                   out <- result
                   
               case err, ok := <-errIn:
                   if !ok {
                       continue
                   }
                   
                   // Forward error from previous stage
                   errOut <- err
                   return
               }
           }
       }()
       
       return out, errOut
   }
   ```

   **Approach 4: Error handling middleware**

   Create a special stage just for error handling:

   ```go
   func handleErrors(in <-chan Result) <-chan Result {
       out := make(chan Result)
       
       go func() {
           defer close(out)
           
           for result := range in {
               if result.Err != nil {
                   // Handle specific error types
                   if errors.Is(result.Err, ErrTemporary) {
                       // Retry logic
                       newResult, err := retry(result.Value)
                       out <- Result{Value: newResult, Err: err}
                       continue
                   }
                   
                   // Forward other errors
                   out <- result
                   continue
               }
               
               // Forward successful results
               out <- result
           }
       }()
       
       return out
   }
   ```

   **Approach 5: Stop on first error (fail-fast)**

   Use `done` channel pattern to signal errors:

   ```go
   func pipeline(done <-chan struct{}, values ...interface{}) <-chan interface{} {
       out := make(chan interface{})
       
       go func() {
           defer close(out)
           
           for _, v := range values {
               result, err := process(v)
               if err != nil {
                   // Signal error and exit
                   select {
                   case <-done:
                   case out <- err: // Send error as a value
                   }
                   return
               }
               
               // Send result or exit if done
               select {
               case out <- result:
               case <-done:
                   return
               }
           }
       }()
       
       return out
   }
   ```

   **Guidelines for error handling in pipelines:**

   1. **Be consistent** about how errors propagate through the pipeline
   2. **Plan for cancellation** to stop unnecessary work when errors occur
   3. **Consider error recovery** for transient failures
   4. **Add context** to errors as they move through the pipeline
   5. **Keep track of originating data** that caused errors
   6. **Log appropriately** at the right stages of the pipeline

   The best approach depends on your specific requirements, such as:

   - Do you need to stop the pipeline on first error or continue processing?
   - Are some errors recoverable while others are fatal?
   - Do you need detailed context about where in the pipeline errors occurred?
   - Does error handling need special logic or can it be standardized?

   A combination of context cancellation for fatal errors and result embedding for per-item errors often provides a good balance of control and simplicity.

2. **Q: How do I manage backpressure in a pipeline to prevent memory issues?**

   A: Backpressure is crucial in pipelines to prevent fast producers from overwhelming slow consumers, which can lead to excessive memory usage or even system crashes. Here are several strategies to manage backpressure effectively:

   **1. Use unbuffered channels between stages**

   Unbuffered channels naturally provide backpressure as they block sends until a receiver is ready:

   ```go
   func simpleStage(in <-chan int) <-chan int {
       out := make(chan int) // Unbuffered channel for automatic backpressure
       
       go func() {
           defer close(out)
           for val := range in {
               processed := process(val)
               out <- processed // Will block if downstream is slow
           }
       }()
       
       return out
   }
   ```

   **2. Use fixed-size buffers to handle bursts**

   Small fixed-size buffers allow for some decoupling while still providing backpressure:

   ```go
   func bufferedStage(in <-chan int) <-chan int {
       // Small buffer for handling temporary bursts
       out := make(chan int, 10)
       
       go func() {
           defer close(out)
           for val := range in {
               processed := process(val)
               out <- processed // Will block when buffer is full
           }
       }()
       
       return out
   }
   ```

   **3. Implement rate limiting**

   Throttle fast producers to match the expected consumption rate:

   ```go
   func rateLimitedStage(in <-chan int, rps int) <-chan int {
       out := make(chan int)
       
       go func() {
           defer close(out)
           
           // Create a ticker for rate limiting
           ticker := time.NewTicker(time.Second / time.Duration(rps))
           defer ticker.Stop()
           
           for val := range in {
               processed := process(val)
               
               // Wait for next available tick before sending
               <-ticker.C
               out <- processed
           }
       }()
       
       return out
   }
   ```

   **4. Use worker pools with fixed concurrency**

   Limit the number of items being processed concurrently:

   ```go
   func workerPoolStage(in <-chan int, maxConcurrent int) <-chan int {
       out := make(chan int)
       
       // Create a semaphore to limit concurrency
       semaphore := make(chan struct{}, maxConcurrent)
       
       go func() {
           defer close(out)
           
           var wg sync.WaitGroup
           for val := range in {
               val := val // Capture for closure
               
               // Acquire semaphore slot (blocks when maxConcurrent reached)
               semaphore <- struct{}{}
               
               wg.Add(1)
               go func() {
                   defer wg.Done()
                   defer func() { <-semaphore }() // Release slot
                   
                   processed := process(val)
                   out <- processed
               }()
           }
           
           // Wait for all workers to complete
           wg.Wait()
       }()
       
       return out
   }
   ```

   **5. Implement batch processing**

   Process items in batches instead of individually:

   ```go
   func batchProcessingStage(in <-chan int, batchSize int) <-chan []int {
       out := make(chan []int)
       
       go func() {
           defer close(out)
           
           batch := make([]int, 0, batchSize)
           for val := range in {
               batch = append(batch, val)
               
               // Send batch when it reaches the target size
               if len(batch) >= batchSize {
                   processedBatch := processBatch(batch)
                   out <- processedBatch
                   batch = make([]int, 0, batchSize)
               }
           }
           
           // Send any remaining items
           if len(batch) > 0 {
               out <- processBatch(batch)
           }
       }()
       
       return out
   }
   ```

   **6. Implement dynamic throttling based on resource usage**

   Adjust processing rate based on system metrics:

   ```go
   func adaptiveRateStage(in <-chan int) <-chan int {
       out := make(chan int)
       
       go func() {
           defer close(out)
           
           var delay time.Duration = 0
           
           for val := range in {
               // Process the value
               processed := process(val)
               
               // Apply current delay for backpressure
               if delay > 0 {
                   time.Sleep(delay)
               }
               
               // Send result
               out <- processed
               
               // Check system load and adjust delay
               memUsage := getMemoryUsage()
               cpuUsage := getCPUUsage()
               
               // Adjust delay based on resource usage
               if memUsage > 80 || cpuUsage > 90 {
                   delay += 5 * time.Millisecond
               } else if delay > 0 && memUsage < 50 && cpuUsage < 70 {
                   delay -= time.Millisecond
                   if delay < 0 {
                       delay = 0
                   }
               }
           }
       }()
       
       return out
   }
   ```

   **7. Drop or sample data under heavy load**

   For some use cases, it's acceptable to drop data when overloaded:

   ```go
   func samplingStage(in <-chan int, maxQueueSize int) <-chan int {
       out := make(chan int, maxQueueSize)
       
       go func() {
           defer close(out)
           
           for val := range in {
               processed := process(val)
               
               // Try to send without blocking
               select {
               case out <- processed:
                   // Sent successfully
               default:
                   // Channel full, log and drop the value
                   log.Printf("Warning: dropping value %v due to backpressure", val)
               }
           }
       }()
       
       return out
   }
   ```

   **8. Implement priority-based processing**

   Process high-priority items first when under load:

   ```go
   type PriorityItem struct {
       Value    int
       Priority int // Higher number means higher priority
   }
   
   func priorityStage(in <-chan PriorityItem) <-chan int {
       out := make(chan int)
       
       go func() {
           defer close(out)
           
           // Keep a priority queue of items
           var queue []PriorityItem
           
           // Helper to send highest priority item
           sendHighestPriority := func() bool {
               if len(queue) == 0 {
                   return false
               }
               
               // Find highest priority item
               highestIdx := 0
               for i := 1; i < len(queue); i++ {
                   if queue[i].Priority > queue[highestIdx].Priority {
                       highestIdx = i
                   }
               }
               
               // Process and send it
               processed := process(queue[highestIdx].Value)
               out <- processed
               
               // Remove from queue
               queue = append(queue[:highestIdx], queue[highestIdx+1:]...)
               return true
           }
           
           for {
               // Try to receive a new item or process from queue
               if len(queue) == 0 {
                   // Queue empty, must receive new item
                   item, ok := <-in
                   if !ok {
                       return // Input channel closed
                   }
                   queue = append(queue, item)
               } else {
                   // Have items in queue, try to receive or send
                   select {
                   case item, ok := <-in:
                       if !ok {
                           // Input channel closed, drain queue
                           for len(queue) > 0 {
                               sendHighestPriority()
                           }
                           return
                       }
                       queue = append(queue, item)
                       
                   default:
                       // No new items, process from queue
                       sendHighestPriority()
                   }
               }
           }
       }()
       
       return out
   }
   ```

   **Comprehensive backpressure management example:**

   ```go
   type BackpressuredPipeline struct {
       // Configuration
       batchSize      int
       maxConcurrency int
       bufferSize     int
       maxMemoryMB    int
       
       // Channels
       input          chan interface{}
       output         chan interface{}
       
       // Control
       done           chan struct{}
       wg             sync.WaitGroup
       
       // Metrics
       processed      int64
       dropped        int64
       processing     int64
       
       mu             sync.Mutex
   }
   
   func NewBackpressuredPipeline(config Config) *BackpressuredPipeline {
       return &BackpressuredPipeline{
           batchSize:      config.BatchSize,
           maxConcurrency: config.MaxConcurrency,
           bufferSize:     config.BufferSize,
           maxMemoryMB:    config.MaxMemoryMB,
           
           input:          make(chan interface{}, config.InputBuffer),
           output:         make(chan interface{}, config.OutputBuffer),
           done:           make(chan struct{}),
       }
   }
   
   func (p *BackpressuredPipeline) Start() {
       p.wg.Add(1)
       go func() {
           defer p.wg.Done()
           p.process()
       }()
   }
   
   func (p *BackpressuredPipeline) process() {
       // Semaphore for concurrency limiting
       sem := make(chan struct{}, p.maxConcurrency)
       
       // Batch accumulator
       batch := make([]interface{}, 0, p.batchSize)
       
       // Create a ticker for periodic system checks
       ticker := time.NewTicker(time.Second)
       defer ticker.Stop()
       
       for {
           // Check memory usage and apply backpressure if needed
           memoryPressure := p.checkMemoryPressure()
           
           select {
           case <-p.done:
               // Pipeline is shutting down
               return
               
           case <-ticker.C:
               // Periodic check - process any accumulated batch
               if len(batch) > 0 {
                   p.processBatch(batch, sem)
                   batch = make([]interface{}, 0, p.batchSize)
               }
               
           case item, ok := <-p.input:
               if !ok {
                   // Input channel closed
                   if len(batch) > 0 {
                       p.processBatch(batch, sem)
                   }
                   // Wait for all processing to complete
                   for i := 0; i < p.maxConcurrency; i++ {
                       sem <- struct{}{}
                   }
                   close(p.output)
                   return
               }
               
               // Apply backpressure if under memory pressure
               if memoryPressure > 0.8 {
                   // Under high memory pressure, sample data
                   if rand.Float64() > 0.2 {  // Drop 80% of data
                       atomic.AddInt64(&p.dropped, 1)
                       continue
                   }
               }
               
               // Add to batch
               batch = append(batch, item)
               
               // Process batch if it's full
               if len(batch) >= p.batchSize {
                   p.processBatch(batch, sem)
                   batch = make([]interface{}, 0, p.batchSize)
               }
           }
       }
   }
   
   func (p *BackpressuredPipeline) processBatch(batch []interface{}, sem chan struct{}) {
       // Apply concurrency control
       sem <- struct{}{}
       
       atomic.AddInt64(&p.processing, 1)
       go func(items []interface{}) {
           defer atomic.AddInt64(&p.processing, -1)
           defer func() { <-sem }()
           
           // Process the batch
           results := processBatchItems(items)
           
           // Send results with backpressure
           for _, result := range results {
               select {
               case p.output <- result:
                   atomic.AddInt64(&p.processed, 1)
               case <-p.done:
                   return
               }
           }
       }(batch)
   }
   
   func (p *BackpressuredPipeline) checkMemoryPressure() float64 {
       var m runtime.MemStats
       runtime.ReadMemStats(&m)
       
       memUsageMB := float64(m.Alloc) / (1024 * 1024)
       pressure := memUsageMB / float64(p.maxMemoryMB)
       
       return pressure
   }
   
   func (p *BackpressuredPipeline) Stop() {
       close(p.done)
       p.wg.Wait()
   }
   
   func (p *BackpressuredPipeline) Input() chan<- interface{} {
       return p.input
   }
   
   func (p *BackpressuredPipeline) Output() <-chan interface{} {
       return p.output
   }
   
   func (p *BackpressuredPipeline) Stats() PipelineStats {
       return PipelineStats{
           Processed:  atomic.LoadInt64(&p.processed),
           Dropped:    atomic.LoadInt64(&p.dropped),
           Processing: atomic.LoadInt64(&p.processing),
       }
   }
   ```

   The key principles for managing backpressure in pipelines are:

   1. **Make backpressure explicit** in your design
   2. **Use unbuffered or small buffered channels** to naturally propagate backpressure
   3. **Monitor resource usage** and adjust processing accordingly
   4. **Limit concurrency** to prevent resource exhaustion
   5. **Consider batch processing** for efficiency
   6. **Implement graceful degradation** strategies for extreme load

   By applying these techniques, you can create robust pipelines that handle varying loads efficiently while maintaining system stability.

3. **Q: How do I design a pipeline that can be shut down gracefully?**

   A: Designing a pipeline with graceful shutdown capabilities is crucial for proper resource management and preventing goroutine leaks. Here's a comprehensive approach:

   **1. Use a cancellation mechanism**

   The most common approach is to use a `context.Context` or a dedicated `done` channel:

   ```go
   func stageWithCancel(ctx context.Context, in <-chan int) <-chan int {
       out := make(chan int)
       
       go func() {
           defer close(out)
           
           for {
               select {
               case <-ctx.Done():
                   // Context canceled, shut down gracefully
                   return
                   
               case val, ok := <-in:
                   if !ok {
                       // Input channel closed
                       return
                   }
                   
                   result := process(val)
                   
                   // Send result, but also check for cancellation
                   select {
                   case out <- result:
                       // Value sent successfully
                   case <-ctx.Done():
                       // Context canceled during send
                       return
                   }
               }
           }
       }()
       
       return out
   }
   ```

   **2. Propagate cancellation through the entire pipeline**

   Ensure all stages respond to the same cancellation signal:

   ```go
   func buildPipeline(ctx context.Context, input <-chan Data) <-chan Result {
       // Each stage receives the same context
       stage1 := stageOne(ctx, input)
       stage2 := stageTwo(ctx, stage1)
       stage3 := stageThree(ctx, stage2)
       
       return stage3
   }
   
   // Usage:
   ctx, cancel := context.WithCancel(context.Background())
   defer cancel() // Ensure cancellation when function exits
   
   results := buildPipeline(ctx, inputData)
   
   // Later, when you want to shut down:
   cancel() // This will propagate cancellation to all stages
   ```

   **3. Use WaitGroups to track goroutine completion**

   Track all goroutines to ensure they've completed:

   ```go
   type Pipeline struct {
       stages []Stage
       wg     sync.WaitGroup
       ctx    context.Context
       cancel context.CancelFunc
   }
   
   func NewPipeline() *Pipeline {
       ctx, cancel := context.WithCancel(context.Background())
       return &Pipeline{
           ctx:    ctx,
           cancel: cancel,
       }
   }
   
   func (p *Pipeline) AddStage(stage Stage) {
       p.stages = append(p.stages, stage)
   }
   
   func (p *Pipeline) Run(input <-chan interface{}) <-chan interface{} {
       current := input
       
       // Start each stage
       for _, stage := range p.stages {
           p.wg.Add(1)
           current = stage.Process(p.ctx, current, &p.wg)
       }
       
       // Create a final channel that will be closed when all stages complete
       finalOutput := make(chan interface{})
       
       p.wg.Add(1)
       go func() {
           defer p.wg.Done()
           defer close(finalOutput)
           
           // Forward all values from the last stage
           for v := range current {
               select {
               case finalOutput <- v:
                   // Value sent
               case <-p.ctx.Done():
                   return
               }
           }
       }()
       
       return finalOutput
   }
   
   func (p *Pipeline) Shutdown() {
       p.cancel() // Signal all stages to stop
       p.wg.Wait() // Wait for all goroutines to exit
   }
   
   // Stage interface
   type Stage interface {
       Process(ctx context.Context, in <-chan interface{}, wg *sync.WaitGroup) <-chan interface{}
   }
   ```

   **4. Drain input channels on shutdown**

   Process remaining items when possible:

   ```go
   func drainOnShutdown(ctx context.Context, in <-chan int) <-chan int {
       out := make(chan int)
       
       go func() {
           defer close(out)
           
           // Normal processing loop
           for {
               select {
               case val, ok := <-in:
                   if !ok {
                       return
                   }
                   
                   // Try to send output
                   select {
                   case out <- process(val):
                       // Sent successfully
                   case <-ctx.Done():
                       // Start drain mode - process remaining inputs but don't accept new ones
                       drainMode(in)
                       return
                   }
                   
               case <-ctx.Done():
                   // Start drain mode
                   drainMode(in)
                   return
               }
           }
       }()
       
       return out
   }
   
   // Process remaining items but don't produce more output
   func drainMode(in <-chan int) {
       // Read and discard any remaining inputs to prevent blocking upstream
       for range in {
           // Optionally log that we're discarding values
       }
   }
   ```

   **5. Implement timeouts for shutdown**

   Don't wait indefinitely for stages to complete:

   ```go
   func (p *Pipeline) ShutdownWithTimeout(timeout time.Duration) error {
       // Signal shutdown
       p.cancel()
       
       // Create a channel that's closed when the WaitGroup is done
       done := make(chan struct{})
       go func() {
           p.wg.Wait()
           close(done)
       }()
       
       // Wait for completion or timeout
       select {
       case <-done:
           return nil
       case <-time.After(timeout):
           return fmt.Errorf("pipeline shutdown timed out after %v", timeout)
       }
   }
   ```

   **6. Use buffered output channels for clean shutdown**

   Buffered channels can help prevent deadlocks during shutdown:

   ```go
   func bufferedStage(ctx context.Context, in <-chan int) <-chan int {
       // Buffer helps handle shutdown more gracefully
       out := make(chan int, 100)
       
       go func() {
           defer close(out)
           
           for {
               select {
               case <-ctx.Done():
                   return
               case val, ok := <-in:
                   if !ok {
                       return
                   }
                   
                   result := process(val)
                   
                   // With buffer, less likely to block on shutdown
                   select {
                   case out <- result:
                       // Sent successfully
                   case <-ctx.Done():
                       return
                   }
               }
           }
       }()
       
       return out
   }
   ```

   **7. Comprehensive pipeline with clean shutdown**

   Here's a complete pipeline implementation with graceful shutdown:

   ```go
   type Pipeline struct {
       ctx      context.Context
       cancel   context.CancelFunc
       wg       sync.WaitGroup
       errChan  chan error
       finished chan struct{}
   }
   
   func NewPipeline() *Pipeline {
       ctx, cancel := context.WithCancel(context.Background())
       return &Pipeline{
           ctx:      ctx,
           cancel:   cancel,
           errChan:  make(chan error, 1),
           finished: make(chan struct{}),
       }
   }
   
   // AddStage adds a processing stage that transforms data
   func (p *Pipeline) AddStage(name string, processor func(context.Context, <-chan interface{}) <-chan interface{}) {
       p.wg.Add(1)
       go func() {
           defer p.wg.Done()
           
           // Signal when all stages have completed
           if p.wg.Counter() == 1 {  // (Hypothetical method, just for illustration)
               defer close(p.finished)
           }
           
           // Run the stage, handle any panics
           defer func() {
               if r := recover(); r != nil {
                   select {
                   case p.errChan <- fmt.Errorf("stage %s panicked: %v", name, r):
                   default:
                       // Error channel full, log it
                       log.Printf("stage %s panicked: %v", name, r)
                   }
                   p.cancel() // Cancel the pipeline on panic
               }
           }()
           
           processor(p.ctx)
       }()
   }
   
   // Run starts the pipeline with given input
   func (p *Pipeline) Run(source <-chan interface{}) <-chan interface{} {
       // Implementation omitted for brevity
       return processedOutput
   }
   
   // Shutdown gracefully shuts down the pipeline
   func (p *Pipeline) Shutdown(timeout time.Duration) error {
       // Signal shutdown
       p.cancel()
       
       // Wait for completion or timeout
       select {
       case <-p.finished:
           return nil
           
       case err := <-p.errChan:
           return fmt.Errorf("pipeline error: %w", err)
           
       case <-time.After(timeout):
           return fmt.Errorf("pipeline shutdown timed out after %v", timeout)
       }
   }
   ```

   **8. Testing graceful shutdown**

   Verify your shutdown mechanism works correctly:

   ```go
   func TestPipelineShutdown(t *testing.T) {
       p := NewPipeline()
       
       // Add stages
       input := make(chan interface{})
       output := p.Run(input)
       
       // Send some data
       go func() {
           for i := 0; i < 10; i++ {
               input <- i
           }
       }()
       
       // Receive some data
       received := 0
       timeout := time.After(100 * time.Millisecond)
       
   receiveLoop:
       for {
           select {
           case _, ok := <-output:
               if !ok {
                   break receiveLoop
               }
               received++
           case <-timeout:
               break receiveLoop
           }
       }
       
       // Initiate shutdown with 500ms timeout
       err := p.Shutdown(500 * time.Millisecond)
       if err != nil {
           t.Errorf("Shutdown failed: %v", err)
       }
       
       // Verify channels are closed properly
       if _, ok := <-output; ok {
           t.Error("Output channel not closed after shutdown")
       }
   }
   ```

   **Key principles for graceful pipeline shutdown:**

   1. **Use a coordinated cancellation mechanism** (context or done channel)
   2. **Close channels in the right order** (generally from source to sink)
   3. **Track all goroutines** with wait groups
   4. **Handle remaining data** appropriately (process, discard, or save)
   5. **Apply timeouts** to prevent hanging
   6. **Clean up resources** (files, connections, etc.)
   7. **Log shutdown progress** for debugging

   Following these patterns will help ensure your pipelines shut down cleanly, avoiding goroutine leaks and resource exhaustion.

### 9. Worker Pools

**Concise Explanation:**
A worker pool is a concurrency pattern that maintains a collection of worker goroutines that process tasks from a shared queue. This pattern limits the number of concurrent goroutines to a fixed size, providing controlled parallelism and resource usage. Worker pools are ideal for processing large numbers of similar, independent tasks without spawning an unbounded number of goroutines.

**Where to Use:**
- When processing many similar tasks concurrently
- To limit resource usage by controlling parallelism
- For CPU-bound processing across multiple cores
- For I/O operations with bounded concurrency
- To handle background processing in servers
- For implementing job queues with parallel processing
- When the number of tasks is much larger than the optimal concurrency level

**Code Snippet:**
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Task represents a unit of work to be performed
type Task struct {
    ID      int
    Data    interface{}
    Handler func(interface{}) (interface{}, error)
}

// Result represents the outcome of a task
type Result struct {
    TaskID  int
    Value   interface{}
    Error   error
    WorkerID int
}

// WorkerPool manages a pool of worker goroutines
type WorkerPool struct {
    tasks       chan Task
    results     chan Result
    numWorkers  int
    workersWg   sync.WaitGroup
}

// NewWorkerPool creates a new worker pool with the specified number of workers
func NewWorkerPool(numWorkers, taskQueueSize int) *WorkerPool {
    return &WorkerPool{
        tasks:      make(chan Task, taskQueueSize),
        results:    make(chan Result, taskQueueSize),
        numWorkers: numWorkers,
    }
}

// Start initializes the worker goroutines
func (p *WorkerPool) Start() {
    // Start the worker goroutines
    for i := 1; i <= p.numWorkers; i++ {
        p.workersWg.Add(1)
        go p.worker(i)
    }
}

// worker processes tasks from the task queue
func (p *WorkerPool) worker(id int) {
    defer p.workersWg.Done()
    
    fmt.Printf("Worker %d starting\n", id)
    
    for task := range p.tasks {
        fmt.Printf("Worker %d processing task %d\n", id, task.ID)
        
        startTime := time.Now()
        value, err := task.Handler(task.Data)
        duration := time.Since(startTime)
        
        // Send the result
        p.results <- Result{
            TaskID:   task.ID,
            Value:    value,
            Error:    err,
            WorkerID: id,
        }
        
        fmt.Printf("Worker %d completed task %d in %v\n", id, task.ID, duration)
    }
    
    fmt.Printf("Worker %d stopping\n", id)
}

// Submit adds a task to the pool
func (p *WorkerPool) Submit(task Task) {
    p.tasks <- task
}

// Results returns the channel to collect results
func (p *WorkerPool) Results() <-chan Result {
    return p.results
}

// Stop gracefully shuts down the worker pool
func (p *WorkerPool) Stop() {
    close(p.tasks)       // Signal workers to stop
    p.workersWg.Wait()   // Wait for all workers to finish
    close(p.results)     // Close the results channel
}

// Example usage
func main() {
    // Create a worker pool with 3 workers and a task queue size of 10
    pool := NewWorkerPool(3, 10)
    
    // Start the worker pool
    pool.Start()
    
    // Create a wait group to wait for all results
    var resultsWg sync.WaitGroup
    
    // Start a goroutine to collect results
    resultsWg.Add(1)
    go func() {
        defer resultsWg.Done()
        for result := range pool.Results() {
            if result.Error != nil {
                fmt.Printf("Task %d failed: %v\n", result.TaskID, result.Error)
            } else {
                fmt.Printf("Task %d completed by worker %d with result: %v\n", 
                    result.TaskID, result.WorkerID, result.Value)
            }
        }
    }()
    
    // Submit 10 tasks
    for i := 1; i <= 10; i++ {
        id := i
        
        // Create a task
        task := Task{
            ID:   id,
            Data: fmt.Sprintf("Task data for %d", id),
            Handler: func(data interface{}) (interface{}, error) {
                // Simulate work
                time.Sleep(time.Duration(id*200) * time.Millisecond)
                return fmt.Sprintf("Processed %v", data), nil
            },
        }
        
        // Submit the task
        pool.Submit(task)
    }
    
    // Allow some time for tasks to be processed
    time.Sleep(time.Second)
    
    // Stop the worker pool
    fmt.Println("Stopping worker pool...")
    pool.Stop()
    
    // Wait for result collection to finish
    resultsWg.Wait()
    
    fmt.Println("All tasks completed")
}
```

**Real-World Example:**
An HTTP server using a worker pool to process requests:

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "strconv"
    "sync"
    "syscall"
    "time"
)

// JobRequest represents a job to be processed
type JobRequest struct {
    ID       string          `json:"id"`
    Type     string          `json:"type"`
    Priority int             `json:"priority"`
    Data     json.RawMessage `json:"data"`
}

// JobResult represents the result of a processed job
type JobResult struct {
    RequestID string      `json:"request_id"`
    Success   bool        `json:"success"`
    Message   string      `json:"message"`
    Data      interface{} `json:"data,omitempty"`
    Error     string      `json:"error,omitempty"`
    StartTime time.Time   `json:"start_time"`
    EndTime   time.Time   `json:"end_time"`
}

// JobProcessor handles the processing of different job types
type JobProcessor struct {
    workers        int
    queue          chan job
    results        chan JobResult
    activeJobs     map[string]context.CancelFunc
    jobsMutex      sync.RWMutex
    quit           chan struct{}
    externalAPI    *ExternalAPIClient
    db             *DatabaseClient
    wg             sync.WaitGroup
}

type job struct {
    request  JobRequest
    ctx      context.Context
    resultCh chan<- JobResult
}

// NewJobProcessor creates a new job processor
func NewJobProcessor(numWorkers int, queueSize int) *JobProcessor {
    return &JobProcessor{
        workers:     numWorkers,
        queue:       make(chan job, queueSize),
        results:     make(chan JobResult, queueSize),
        activeJobs:  make(map[string]context.CancelFunc),
        quit:        make(chan struct{}),
        externalAPI: NewExternalAPIClient(),
        db:          NewDatabaseClient(),
    }
}

// Start begins the worker pool
func (p *JobProcessor) Start() {
    // Start the worker goroutines
    for i := 0; i < p.workers; i++ {
        p.wg.Add(1)
        go p.worker(i)
    }
    
    log.Printf("Started job processor with %d workers", p.workers)
}

// worker processes jobs from the queue
func (p *JobProcessor) worker(id int) {
    defer p.wg.Done()
    
    log.Printf("Worker %d started", id)
    
    for {
        select {
        case j, ok := <-p.queue:
            if !ok {
                log.Printf("Worker %d shutting down - queue closed", id)
                return
            }
            
            // Process the job
            p.processJob(id, j)
            
        case <-p.quit:
            log.Printf("Worker %d shutting down - quit signal", id)
            return
        }
    }
}

// processJob handles the actual job processing
func (p *JobProcessor) processJob(workerID int, j job) {
    ctx := j.ctx
    req := j.request
    
    // Create result with defaults
    result := JobResult{
        RequestID: req.ID,
        Success:   false,
        StartTime: time.Now(),
    }
    
    // Ensure we record the end time
    defer func() {
        result.EndTime = time.Now()
        j.resultCh <- result
        
        // Remove from active jobs
        p.jobsMutex.Lock()
        delete(p.activeJobs, req.ID)
        p.jobsMutex.Unlock()
    }()
    
    // Check for cancellation before we start
    select {
    case <-ctx.Done():
        result.Error = "job cancelled before processing"
        return
    default:
        // Continue processing
    }
    
    log.Printf("Worker %d processing job %s of type %s", workerID, req.ID, req.Type)
    
    // Process based on job type
    switch req.Type {
    case "data_analysis":
        result = p.processDataAnalysis(ctx, req)
        
    case "image_processing":
        result = p.processImageProcessing(ctx, req)
        
    case "notification":
        result = p.processNotification(ctx, req)
        
    default:
        result.Error = fmt.Sprintf("unknown job type: %s", req.Type)
    }
}

// Submit adds a new job to the queue with a context
func (p *JobProcessor) Submit(request JobRequest) error {
    // Create a cancellable context for this job
    ctx, cancel := context.WithCancel(context.Background())
    
    // Add to active jobs
    p.jobsMutex.Lock()
    p.activeJobs[request.ID] = cancel
    p.jobsMutex.Unlock()
    
    // Create and submit the job
    j := job{
        request:  request,
        ctx:      ctx,
        resultCh: p.results,
    }
    
    select {
    case p.queue <- j:
        return nil
    default:
        // Queue is full
        p.jobsMutex.Lock()
        delete(p.activeJobs, request.ID)
        p.jobsMutex.Unlock()
        
        cancel() // Clean up the context
        return fmt.Errorf("job queue is full")
    }
}

// Cancel cancels a job by ID if it's still in the queue or processing
func (p *JobProcessor) Cancel(jobID string) bool {
    p.jobsMutex.Lock()
    defer p.jobsMutex.Unlock()
    
    cancel, exists := p.activeJobs[jobID]
    if exists {
        cancel()
        return true
    }
    return false
}

// Results returns the channel for collecting job results
func (p *JobProcessor) Results() <-chan JobResult {
    return p.results
}

// Shutdown gracefully stops the job processor
func (p *JobProcessor) Shutdown(timeout time.Duration) {
    log.Println("Shutting down job processor...")
    
    // Signal all workers to stop accepting new jobs
    close(p.quit)
    
    // Signal all active jobs to cancel
    p.jobsMutex.Lock()
    for _, cancel := range p.activeJobs {
        cancel()
    }
    p.jobsMutex.Unlock()
    
    // Close the job queue to prevent new submissions
    close(p.queue)
    
    // Wait for workers to finish with timeout
    done := make(chan struct{})
    go func() {
        p.wg.Wait()
        close(done)
    }()
    
    select {
    case <-done:
        log.Println("All workers have completed gracefully")
    case <-time.After(timeout):
        log.Println("Shutdown timed out, some jobs may not have completed")
    }
    
    // Close the results channel
    close(p.results)
    
    // Clean up resources
    p.externalAPI.Close()
    p.db.Close()
}

// Specific job processing methods
func (p *JobProcessor) processDataAnalysis(ctx context.Context, req JobRequest) JobResult {
    result := JobResult{
        RequestID: req.ID,
        StartTime: time.Now(),
    }
    
    // Parse job-specific data
    var data struct {
        Dataset string `json:"dataset"`
        Query   string `json:"query"`
    }
    
    if err := json.Unmarshal(req.Data, &data); err != nil {
        result.Error = fmt.Sprintf("invalid job data: %v", err)
        return result
    }
    
    // Simulate data retrieval from database
    rows, err := p.db.Query(ctx, data.Query, data.Dataset)
    if err != nil {
        result.Error = fmt.Sprintf("database query failed: %v", err)
        return result
    }
    
    // Process data (simulated)
    processedData := make(map[string]interface{})
    for i, row := range rows {
        // Check for cancellation periodically
        if i%100 == 0 {
            select {
            case <-ctx.Done():
                result.Error = "job cancelled during processing"
                return result
            default:
                // Continue processing
            }
        }
        
        // Process row (simplified)
        processedData[fmt.Sprintf("row_%d", i)] = row
    }
    
    // Complete the result
    result.Success = true
    result.Message = fmt.Sprintf("Successfully analyzed %d records", len(rows))
    result.Data = processedData
    
    return result
}

func (p *JobProcessor) processImageProcessing(ctx context.Context, req JobRequest) JobResult {
    result := JobResult{
        RequestID: req.ID,
        StartTime: time.Now(),
    }
    
    // Parse job-specific data
    var data struct {
        ImageURL string            `json:"image_url"`
        Options  map[string]string `json:"options"`
    }
    
    if err := json.Unmarshal(req.Data, &data); err != nil {
        result.Error = fmt.Sprintf("invalid job data: %v", err)
        return result
    }
    
    // Download image (simulated)
    imageData, err := p.externalAPI.DownloadImage(ctx, data.ImageURL)
    if err != nil {
        result.Error = fmt.Sprintf("failed to download image: %v", err)
        return result
    }
    
    // Process the image (simulated)
    processedImage, err := p.processImage(ctx, imageData, data.Options)
    if err != nil {
        result.Error = fmt.Sprintf("image processing failed: %v", err)
        return result
    }
    
    // Upload the processed image (simulated)
    uploadURL, err := p.externalAPI.UploadImage(ctx, processedImage)
    if err != nil {
        result.Error = fmt.Sprintf("failed to upload processed image: %v", err)
        return result
    }
    
    // Complete the result
    result.Success = true
    result.Message = "Image processed successfully"
    result.Data = map[string]string{
        "processed_image_url": uploadURL,
        "original_size":       fmt.Sprintf("%d bytes", len(imageData)),
        "processed_size":      fmt.Sprintf("%d bytes", len(processedImage)),
    }
    
    return result
}

func (p *JobProcessor) processNotification(ctx context.Context, req JobRequest) JobResult {
    result := JobResult{
        RequestID: req.ID,
        StartTime: time.Now(),
    }
    
    // Parse job-specific data
    var data struct {
        Recipients []string `json:"recipients"`
        Subject    string   `json:"subject"`
        Message    string   `json:"message"`
        Channel    string   `json:"channel"` // email, sms, push
    }
    
    if err := json.Unmarshal(req.Data, &data); err != nil {
        result.Error = fmt.Sprintf("invalid job data: %v", err)
        return result
    }
    
    // Check for required fields
    if len(data.Recipients) == 0 {
        result.Error = "no recipients specified"
        return result
    }
    
    if data.Message == "" {
        result.Error = "empty message"
        return result
    }
    
    // Process notifications (simulated)
    sentCount := 0
    failedCount := 0
    
    for _, recipient := range data.Recipients {
        // Check for cancellation between recipients
        select {
        case <-ctx.Done():
            result.Error = "job cancelled during processing"
            return result
        default:
            // Continue processing
        }
        
        // Send notification based on channel type
        var err error
        switch data.Channel {
        case "email":
            err = p.externalAPI.SendEmail(ctx, recipient, data.Subject, data.Message)
        case "sms":
            err = p.externalAPI.SendSMS(ctx, recipient, data.Message)
        case "push":
            err = p.externalAPI.SendPushNotification(ctx, recipient, data.Subject, data.Message)
        default:
            err = fmt.Errorf("unknown notification channel: %s", data.Channel)
        }
        
        if err != nil {
            failedCount++
        } else {
            sentCount++
        }
    }
    
    // Complete the result
    if failedCount == 0 {
        result.Success = true
        result.Message = fmt.Sprintf("Successfully sent %d notifications", sentCount)
    } else {
        result.Success = sentCount > 0
        result.Message = fmt.Sprintf("Sent %d notifications, %d failed", sentCount, failedCount)
    }
    
    result.Data = map[string]int{
        "sent":   sentCount,
        "failed": failedCount,
        "total":  sentCount + failedCount,
    }
    
    return result
}

// Helper method for image processing
func (p *JobProcessor) processImage(ctx context.Context, imageData []byte, options map[string]string) ([]byte, error) {
    // Simulate image processing
    time.Sleep(500 * time.Millisecond)
    
    // Check for cancellation
    select {
    case <-ctx.Done():
        return nil, ctx.Err()
    default:
        // Continue processing
    }
    
    // Return "processed" data (just a placeholder)
    return append([]byte("PROCESSED:"), imageData...), nil
}

// HTTP server to handle job submissions
type JobServer struct {
    processor *JobProcessor
    server    *http.Server
}

func NewJobServer(processor *JobProcessor, port int) *JobServer {
    mux := http.NewServeMux()
    
    server := &JobServer{
        processor: processor,
        server: &http.Server{
            Addr:    fmt.Sprintf(":%d", port),
            Handler: mux,
        },
    }
    
    // Register HTTP handlers
    mux.HandleFunc("/api/jobs", server.handleJobs)
    mux.HandleFunc("/api/jobs/", server.handleJobStatus)
    
    return server
}

func (s *JobServer) Start() {
    // Start the job processor
    s.processor.Start()
    
    // Start HTTP server in a goroutine
    go func() {
        log.Printf("Starting HTTP server on %s", s.server.Addr)
        if err := s.server.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatalf("HTTP server error: %v", err)
        }
    }()
}

func (s *JobServer) Shutdown(timeout time.Duration) {
    // Create a context with timeout for shutting down the HTTP server
    ctx, cancel := context.WithTimeout(context.Background(), timeout)
    defer cancel()
    
    // Shutdown the HTTP server
    log.Println("Shutting down HTTP server...")
    if err := s.server.Shutdown(ctx); err != nil {
        log.Printf("HTTP server shutdown error: %v", err)
    }
    
    // Shutdown the job processor
    s.processor.Shutdown(timeout)
}

// HTTP handler for job submissions
func (s *JobServer) handleJobs(w http.ResponseWriter, r *http.Request) {
    if r.Method == "POST" {
        // Parse the job request
        var req JobRequest
        if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
            http.Error(w, fmt.Sprintf("Invalid request: %v", err), http.StatusBadRequest)
            return
        }
        
        // Validate the job
        if req.ID == "" {
            req.ID = fmt.Sprintf("job-%d", time.Now().UnixNano())
        }
        
        if req.Type == "" {
            http.Error(w, "Job type is required", http.StatusBadRequest)
            return
        }
        
        // Submit the job
        err := s.processor.Submit(req)
        if err != nil {
            http.Error(w, fmt.Sprintf("Failed to submit job: %v", err), http.StatusServiceUnavailable)
            return
        }
        
        // Return success response
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusAccepted)
        json.NewEncoder(w).Encode(map[string]string{
            "status":  "accepted",
            "job_id":  req.ID,
            "message": "Job submitted successfully",
        })
    } else {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

// HTTP handler for job status
func (s *JobServer) handleJobStatus(w http.ResponseWriter, r *http.Request) {
    // Extract job ID from URL
    jobID := r.URL.Path[len("/api/jobs/"):]
    
    if r.Method == "GET" {
        // Get job status (simplified - in a real app, we'd query job status from storage)
        http.Error(w, "Not implemented", http.StatusNotImplemented)
    } else if r.Method == "DELETE" {
        // Cancel the job
        if cancelled := s.processor.Cancel(jobID); cancelled {
            w.WriteHeader(http.StatusOK)
            json.NewEncoder(w).Encode(map[string]string{
                "status":  "cancelled",
                "job_id":  jobID,
                "message": "Job cancelled successfully",
            })
        } else {
            http.Error(w, "Job not found or already completed", http.StatusNotFound)
        }
    } else {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

// Simulated external API client
type ExternalAPIClient struct {
    // Configuration, authentication, etc. would go here
}

func NewExternalAPIClient() *ExternalAPIClient {
    return &ExternalAPIClient{}
}

func (c *ExternalAPIClient) DownloadImage(ctx context.Context, url string) ([]byte, error) {
    // Simulate network delay
    select {
    case <-time.After(300 * time.Millisecond):
        return []byte(fmt.Sprintf("SAMPLE_IMAGE_DATA_%s", url)), nil
    case <-ctx.Done():
        return nil, ctx.Err()
    }
}

func (c *ExternalAPIClient) UploadImage(ctx context.Context, data []byte) (string, error) {
    // Simulate network delay
    select {
    case <-time.After(300 * time.Millisecond):
        return fmt.Sprintf("https://example.com/images/%d.jpg", time.Now().Unix()), nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}

func (c *ExternalAPIClient) SendEmail(ctx context.Context, recipient, subject, message string) error {
    // Simulate sending email
    time.Sleep(200 * time.Millisecond)
    
    // Simulate occasional failures
    if len(recipient) % 5 == 0 {
        return fmt.Errorf("failed to send email to %s", recipient)
    }
    return nil
}

func (c *ExternalAPIClient) SendSMS(ctx context.Context, recipient, message string) error {
    // Simulate sending SMS
    time.Sleep(150 * time.Millisecond)
    
    // Simulate occasional failures
    if len(recipient) % 7 == 0 {
        return fmt.Errorf("failed to send SMS to %s", recipient)
    }
    return nil
}

func (c *ExternalAPIClient) SendPushNotification(ctx context.Context, recipient, title, message string) error {
    // Simulate sending push notification
    time.Sleep(100 * time.Millisecond)
    
    // Simulate occasional failures
    if len(recipient) % 3 == 0 {
        return fmt.Errorf("failed to send push notification to %s", recipient)
    }
    return nil
}

func (c *ExternalAPIClient) Close() {
    // Clean up any resources
}

// Simulated database client
type DatabaseClient struct {
    // Connection pool, configuration, etc. would go here
}

func NewDatabaseClient() *DatabaseClient {
    return &DatabaseClient{}
}

func (db *DatabaseClient) Query(ctx context.Context, query, dataset string) ([]map[string]interface{}, error) {
    // Simulate database query
    select {
    case <-time.After(500 * time.Millisecond):
        // Generate some fake rows
        rows := make([]map[string]interface{}, 0, 20)
        for i := 0; i < 20; i++ {
            rows = append(rows, map[string]interface{}{
                "id":    i,
                "name":  fmt.Sprintf("Item %d", i),
                "value": i * 10,
                "date":  time.Now().AddDate(0, 0, -i).Format(time.RFC3339),
            })
        }
        return rows, nil
    case <-ctx.Done():
        return nil, ctx.Err()
    }
}

func (db *DatabaseClient) Close() {
    // Close database connections
}

// Main application entry point
func main() {
    // Get configuration from environment or use defaults
    numWorkers, _ := strconv.Atoi(getEnvOrDefault("NUM_WORKERS", "4"))
    queueSize, _ := strconv.Atoi(getEnvOrDefault("QUEUE_SIZE", "100"))
    port, _ := strconv.Atoi(getEnvOrDefault("PORT", "8080"))
    
    // Create the job processor and server
    processor := NewJobProcessor(numWorkers, queueSize)
    server := NewJobServer(processor, port)
    
    // Start the server
    server.Start()
    
    // Start a goroutine to collect and handle results
    go func() {
        for result := range processor.Results() {
            duration := result.EndTime.Sub(result.StartTime).Milliseconds()
            
            if result.Success {
                log.Printf("Job %s completed in %dms: %s", result.RequestID, duration, result.Message)
            } else {
                log.Printf("Job %s failed in %dms: %s", result.RequestID, duration, result.Error)
            }
            
            // In a real application, you would store these results
            // or notify interested parties about completion
        }
    }()
    
    // Set up graceful shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    
    // Wait for shutdown signal
    <-quit
    log.Println("Shutting down server...")
    
    // Give a 30-second grace period for shutdown
    server.Shutdown(30 * time.Second)
    log.Println("Server gracefully stopped")
}

// Helper function to get environment variable with default
func getEnvOrDefault(key, defaultValue string) string {
    if value, exists := os.LookupEnv(key); exists {
        return value
    }
    return defaultValue
}
```

**Common Pitfalls:**
- Creating too many goroutines or too few workers for the workload
- Not properly handling worker goroutine termination
- Deadlocks in worker pool shutdown
- Leaking goroutines if tasks can block indefinitely
- Not limiting queue size, leading to unbounded memory growth
- Race conditions when accessing shared resources from multiple workers
- Not handling panics in worker goroutines, causing the entire application to crash
- Inefficiently distributing tasks to workers

**Confusion Questions:**

1. **Q: How do I determine the optimal number of workers for my workload?**

   A: Determining the optimal number of workers depends on your workload characteristics and system resources. Here's a systematic approach:

   **For CPU-bound tasks:**
   The optimal number typically relates to the number of available CPU cores:

   ```go
   import "runtime"

   // Starting point for CPU-bound tasks
   numWorkers := runtime.NumCPU()
   ```

   **For I/O-bound tasks:**
   I/O-bound workloads (network, disk, database operations) can efficiently use many more workers than available CPU cores:

   ```go
   // For I/O-bound tasks, often 2-4x the number of cores works well
   numWorkers := runtime.NumCPU() * 3
   ```

   **Consider these factors when tuning:**

   1. **Task execution time**: 
      - Short tasks (<1ms): Fewer workers to minimize overhead
      - Long tasks (>100ms): More workers to keep CPUs busy

   2. **External resource limitations**:
      ```go
      // Example: Database connection pool size might limit workers
      numWorkers = min(runtime.NumCPU()*4, dbPool.MaxConnections())
      ```

   3. **Memory constraints**:
      ```go
      // Estimate memory per worker
      memoryPerWorker := 50 * 1024 * 1024 // 50MB example
      availableMemory := getTotalAvailableMemory() * 0.8 // Use 80% of available
      maxWorkersByMemory := availableMemory / memoryPerWorker
      
      numWorkers = min(runtime.NumCPU()*4, maxWorkersByMemory)
      ```

   4. **Workload characteristics**:
      - **Mixed workloads**: Start with `runtime.NumCPU() * (1 + %IO_time)`
      - **Embarrassingly parallel**: Can use more workers 
      - **Interdependent tasks**: May benefit from fewer workers

   **Practical benchmarking approach:**

   ```go
   func findOptimalWorkers(tasks []Task) int {
       bestTime := time.Duration(1<<63 - 1) // Max duration
       bestCount := 1
       
       results := make(map[int]time.Duration)
       
       // Test different worker counts
       for workers := 1; workers <= runtime.NumCPU()*4; workers *= 2 {
           // Run multiple times to get reliable measurements
           var totalDuration time.Duration
           const runs = 5
           
           for i := 0; i < runs; i++ {
               start := time.Now()
               
               pool := NewWorkerPool(workers, len(tasks))
               pool.Start()
               
               for _, task := range tasks {
                   pool.Submit(task)
               }
               
               pool.Stop() // Wait for completion
               
               totalDuration += time.Since(start)
           }
           
           avgDuration := totalDuration / runs
           results[workers] = avgDuration
           
           if avgDuration < bestTime {
               bestTime = avgDuration
               bestCount = workers
           }
       }
       
       // Log all results
       for workers, duration := range results {
           fmt.Printf("%d workers: %v\n", workers, duration)
       }
       
       return bestCount
   }
   ```

   **Adaptive worker pool:**
   For long-running services with varying load, consider dynamic scaling:

   ```go
   type AdaptiveWorkerPool struct {
       // ... other fields
       minWorkers int
       maxWorkers int
       activeWorkers int
       pendingTasks int
       mu sync.Mutex
   }
   
   func (p *AdaptiveWorkerPool) adjustWorkerCount() {
       for {
           time.Sleep(10 * time.Second) // Check periodically
           
           p.mu.Lock()
           pending := p.pendingTasks
           active := p.activeWorkers
           p.mu.Unlock()
           
           // Scale up if there's a backlog
           if pending > active*10 && active < p.maxWorkers {
               // Add workers
               workersToAdd := min(pending/10, p.maxWorkers-active)
               p.addWorkers(workersToAdd)
               
           } else if pending < active/2 && active > p.minWorkers {
               // Scale down if there's excess capacity
               workersToRemove := min(active/2, active-p.minWorkers)
               p.removeWorkers(workersToRemove)
           }
       }
   }
   ```

   **Final guidelines:**

   1. **Start with a reasonable default**: `runtime.NumCPU()` for CPU-bound, `runtime.NumCPU() * 2-4` for I/O-bound
   2. **Benchmark with your actual workload**: Real-world performance can differ from theoretical estimates
   3. **Monitor resource usage**: CPU, memory, external services
   4. **Consider task affinity**: Some workloads benefit from consistent worker assignment
   5. **For servers**: Leave some CPU capacity for handling requests and system tasks

   Remember that the optimal number of workers is not static – it depends on your specific workload characteristics, available resources, and may change over time.

2. **Q: How can I prioritize tasks in a worker pool?**

   A: Implementing task prioritization in a worker pool allows you to process more important tasks first. Here are several approaches to prioritize tasks:

   **1. Multiple queue approach**

   Use separate queues for different priority levels:

   ```go
   type PriorityWorkerPool struct {
       highPriorityQueue   chan Task
       mediumPriorityQueue chan Task
       lowPriorityQueue    chan Task
       results             chan Result
       workers             int
       wg                  sync.WaitGroup
   }
   
   func NewPriorityWorkerPool(workers int, highCap, medCap, lowCap int) *PriorityWorkerPool {
       return &PriorityWorkerPool{
           highPriorityQueue:   make(chan Task, highCap),
           mediumPriorityQueue: make(chan Task, medCap),
           lowPriorityQueue:    make(chan Task, lowCap),
           results:             make(chan Result, highCap+medCap+lowCap),
           workers:             workers,
       }
   }
   
   func (p *PriorityWorkerPool) Start() {
       for i := 0; i < p.workers; i++ {
           p.wg.Add(1)
           go func(workerID int) {
               defer p.wg.Done()
               
               for {
                   // Check queues in priority order using select with default
                   var task Task
                   var ok bool
                   
                   // First check high priority queue
                   select {
                   case task, ok = <-p.highPriorityQueue:
                       if !ok {
                           // High priority channel closed, check medium priority
                           goto checkMediumPriority
                       }
                   default:
                       goto checkMediumPriority // No high priority tasks
                   }
                   
                   // Process high priority task
                   p.processTask(workerID, task)
                   continue
                   
               checkMediumPriority:
                   // Check medium priority queue
                   select {
                   case task, ok = <-p.mediumPriorityQueue:
                       if !ok {
                           // Medium priority channel closed, check low priority
                           goto checkLowPriority
                       }
                   default:
                       goto checkLowPriority // No medium priority tasks
                   }
                   
                   // Process medium priority task
                   p.processTask(workerID, task)
                   continue
                   
               checkLowPriority:
                   // Check low priority queue
                   select {
                   case task, ok = <-p.lowPriorityQueue:
                       if !ok {
                           // All channels are closed
                           return
                       }
                   default:
                       // No tasks in any queue, wait for tasks with priority bias
                       select {
                       case task, ok = <-p.highPriorityQueue:
                           if !ok {
                               goto finalCheck // High priority closed
                           }
                       case task, ok = <-p.mediumPriorityQueue:
                           if !ok {
                               goto finalCheck // Medium priority closed
                           }
                       case task, ok = <-p.lowPriorityQueue:
                           if !ok {
                               goto finalCheck // Low priority closed
                           }
                       }
                       
                       // Process the task we got
                       p.processTask(workerID, task)
                       continue
                   }
                   
                   // Process low priority task
                   p.processTask(workerID, task)
                   continue
                   
               finalCheck:
                   // Final check to see if all channels closed
                   select {
                   case task, ok = <-p.highPriorityQueue:
                       if ok {
                           p.processTask(workerID, task)
                           continue
                       }
                   case task, ok = <-p.mediumPriorityQueue:
                       if ok {
                           p.processTask(workerID, task)
                           continue
                       }
                   case task, ok = <-p.lowPriorityQueue:
                       if ok {
                           p.processTask(workerID, task)
                           continue
                       }
                       // All channels are closed
                       return
                   }
               }
           }(i)
       }
   }
   
   func (p *PriorityWorkerPool) processTask(workerID int, task Task) {
       // Process task and send result to results channel
       // ...
   }
   
   func (p *PriorityWorkerPool) Submit(task Task, priority Priority) error {
       switch priority {
       case HighPriority:
           select {
           case p.highPriorityQueue <- task:
               return nil
           default:
               return ErrHighPriorityQueueFull
           }
       case MediumPriority:
           select {
           case p.mediumPriorityQueue <- task:
               return nil
           default:
               return ErrMediumPriorityQueueFull
           }
       case LowPriority:
           select {
           case p.lowPriorityQueue <- task:
               return nil
           default:
               return ErrLowPriorityQueueFull
           }
       default:
           return ErrInvalidPriority
       }
   }
   ```

   **2. Priority queue approach**

   Use a priority queue data structure:

   ```go
   // Import priority queue implementation
   import "container/heap"
   
   // PriorityTask adds priority to tasks
   type PriorityTask struct {
       Task     Task
       Priority int // Higher number = higher priority
       Index    int // Used by the heap implementation
   }
   
   // PriorityQueue implements heap.Interface
   type PriorityQueue []*PriorityTask
   
   // Required heap.Interface methods
   func (pq PriorityQueue) Len() int { return len(pq) }
   func (pq PriorityQueue) Less(i, j int) bool {
       // Higher priority value means higher priority
       return pq[i].Priority > pq[j].Priority
   }
   func (pq PriorityQueue) Swap(i, j int) {
       pq[i], pq[j] = pq[j], pq[i]
       pq[i].Index = i
       pq[j].Index = j
   }
   func (pq *PriorityQueue) Push(x interface{}) {
       n := len(*pq)
       task := x.(*PriorityTask)
       task.Index = n
       *pq = append(*pq, task)
   }
   func (pq *PriorityQueue) Pop() interface{} {
       old := *pq
       n := len(old)
       task := old[n-1]
       old[n-1] = nil  // Avoid memory leak
       task.Index = -1 // For safety
       *pq = old[0 : n-1]
       return task
   }
   
   // WorkerPool with priority queue
   type PriorityWorkerPool struct {
       tasks       chan *PriorityTask
       results     chan Result
       done        chan struct{}
       queue       PriorityQueue
       queueMu     sync.Mutex
       queueCond   *sync.Cond
       wg          sync.WaitGroup
       numWorkers  int
   }
   
   func NewPriorityWorkerPool(numWorkers int) *PriorityWorkerPool {
       p := &PriorityWorkerPool{
           tasks:      make(chan *PriorityTask),
           results:    make(chan Result, numWorkers),
           done:       make(chan struct{}),
           queue:      make(PriorityQueue, 0),
           numWorkers: numWorkers,
       }
       p.queueCond = sync.NewCond(&p.queueMu)
       heap.Init(&p.queue)
       return p
   }
   
   func (p *PriorityWorkerPool) Start() {
       // Start the queue manager
       go p.queueManager()
       
       // Start workers
       for i := 0; i < p.numWorkers; i++ {
           p.wg.Add(1)
           go p.worker(i)
       }
   }
   
   func (p *PriorityWorkerPool) queueManager() {
       for {
           p.queueMu.Lock()
           // Wait for tasks to be added
           for p.queue.Len() == 0 {
               p.queueCond.Wait()
               
               // Check if we're done
               select {
               case <-p.done:
                   p.queueMu.Unlock()
                   close(p.tasks)
                   return
               default:
                   // Continue
               }
           }
           
           // Pop highest priority task
           task := heap.Pop(&p.queue).(*PriorityTask)
           p.queueMu.Unlock()
           
           // Send to worker
           select {
           case p.tasks <- task:
               // Task sent to worker
           case <-p.done:
               close(p.tasks)
               return
           }
       }
   }
   
   func (p *PriorityWorkerPool) worker(id int) {
       defer p.wg.Done()
       
       for pt := range p.tasks {
           // Process task
           result := processTask(pt.Task)
           
           // Send result
           select {
           case p.results <- result:
               // Result sent
           case <-p.done:
               return
           }
       }
   }
   
   func (p *PriorityWorkerPool) Submit(task Task, priority int) {
       pt := &PriorityTask{
           Task:     task,
           Priority: priority,
       }
       
       p.queueMu.Lock()
       heap.Push(&p.queue, pt)
       p.queueMu.Unlock()
       
       // Signal that new task is available
       p.queueCond.Signal()
   }
   
   func (p *PriorityWorkerPool) Stop() {
       close(p.done)
       p.queueCond.Broadcast() // Wake up queue manager
       p.wg.Wait()
       close(p.results)
   }
   ```

   **3. Dedicated worker pools by priority**

   Dedicate specific workers to different priority levels:

   ```go
   type MultiTierWorkerPool struct {
       highPriorityPool  *WorkerPool
       mediumPriorityPool *WorkerPool
       lowPriorityPool   *WorkerPool
       results           chan Result
   }
   
   func NewMultiTierWorkerPool(highWorkers, mediumWorkers, lowWorkers int) *MultiTierWorkerPool {
       results := make(chan Result)
       
       return &MultiTierWorkerPool{
           // More workers for high priority tasks
           highPriorityPool: NewWorkerPoolWithResults(highWorkers, 100, results),
           
           // Medium number of workers for medium priority
           mediumPriorityPool: NewWorkerPoolWithResults(mediumWorkers, 100, results),
           
           // Fewer workers for low priority
           lowPriorityPool: NewWorkerPoolWithResults(lowWorkers, 100, results),
           
           results: results,
       }
   }
   
   func (p *MultiTierWorkerPool) Start() {
       p.highPriorityPool.Start()
       p.mediumPriorityPool.Start()
       p.lowPriorityPool.Start()
   }
   
   func (p *MultiTierWorkerPool) Stop() {
       p.highPriorityPool.Stop()
       p.mediumPriorityPool.Stop()
       p.lowPriorityPool.Stop()
       close(p.results)
   }
   
   func (p *MultiTierWorkerPool) Submit(task Task, priority Priority) error {
       switch priority {
       case HighPriority:
           return p.highPriorityPool.Submit(task)
       case MediumPriority:
           return p.mediumPriorityPool.Submit(task)
       case LowPriority:
           return p.lowPriorityPool.Submit(task)
       default:
           return ErrInvalidPriority
       }
   }
   ```

   **4. Time-based priority with weighted fair queuing**

   Give more CPU time to higher priority tasks:

   ```go
   type WeightedWorkerPool struct {
       tasks       chan Task
       results     chan Result
       quit        chan struct{}
       highRatio   int
       mediumRatio int
       lowRatio    int
       wg          sync.WaitGroup
   }
   
   func NewWeightedWorkerPool(workers, highRatio, mediumRatio, lowRatio int) *WeightedWorkerPool {
       return &WeightedWorkerPool{
           tasks:       make(chan Task, workers*2),
           results:     make(chan Result, workers*2),
           quit:        make(chan struct{}),
           highRatio:   highRatio,
           mediumRatio: mediumRatio,
           lowRatio:    lowRatio,
       }
   }
   
   func (p *WeightedWorkerPool) worker() {
       defer p.wg.Done()
       
       // Track slots for each priority
       highSlots := p.highRatio
       mediumSlots := p.mediumRatio
       lowSlots := p.lowRatio
       totalSlots := highSlots + mediumSlots + lowSlots
       
       // Counters for each priority level
       processed := make(map[Priority]int)
       
       for {
           select {
           case <-p.quit:
               return
           case task := <-p.tasks:
               // Determine if we should process based on weighted fair queuing
               shouldProcess := false
               
               switch task.Priority {
               case HighPriority:
                   ratio := float64(processed[HighPriority]) / float64(totalSlots)
                   target := float64(highSlots) / float64(totalSlots)
                   shouldProcess = ratio <= target
               case MediumPriority:
                   ratio := float64(processed[MediumPriority]) / float64(totalSlots)
                   target := float64(mediumSlots) / float64(totalSlots)
                   shouldProcess = ratio <= target
               case LowPriority:
                   ratio := float64(processed[LowPriority]) / float64(totalSlots)
                   target := float64(lowSlots) / float64(totalSlots)
                   shouldProcess = ratio <= target
               }
               
               if shouldProcess {
                   // Process the task
                   result := processTask(task)
                   p.results <- result
                   processed[task.Priority]++
               } else {
                   // Put back in queue for later processing
                   go func() {
                       p.tasks <- task
                   }()
                   
                   // Yield to allow other tasks to be checked
                   runtime.Gosched()
               }
           }
       }
   }
   ```

   **5. Dynamic priority aging**

   Increase priority of waiting tasks over time:

   ```go
   type AgingTask struct {
       Task         Task
       Priority     int
       InitialPriority int
       SubmitTime   time.Time
       mu          sync.Mutex
   }

   func (t *AgingTask) CurrentPriority() int {
       t.mu.Lock()
       defer t.mu.Unlock()
       
       // Age the priority - tasks get more important over time
       waitingTime := time.Since(t.SubmitTime).Seconds()
       agingFactor := int(waitingTime / 10) // Every 10 seconds, priority increases by 1
       
       return t.InitialPriority + agingFactor
   }
   
   type AgingPriorityQueue struct {
       // Standard priority queue with aging method
   }
   
   // Custom Less method that uses current priority
   func (pq AgingPriorityQueue) Less(i, j int) bool {
       return pq[i].CurrentPriority() > pq[j].CurrentPriority()
   }
   ```

   **Choosing the right approach:**

   1. **Multiple queues**: Simple to implement, good for few priority levels
   2. **Priority queue**: More sophisticated, supports many priority levels
   3. **Dedicated pools**: Good for strict isolation between priority levels
   4. **Weighted scheduling**: Best for fair allocation of resources
   5. **Priority aging**: Prevents starvation of low-priority tasks

   Your choice should depend on:
   
   - How critical is the distinction between priority levels?
   - Is task starvation acceptable?
   - Does your workload have consistent or varying priority distribution?
   - How important is implementation complexity versus performance?

   For most applications, either the multiple queue approach or the priority queue approach provides a good balance of simplicity and effectiveness.

3. **Q: How do I handle worker goroutines that panic?**

   A: Properly handling panics in worker goroutines is essential for maintaining a stable worker pool. Here's a comprehensive approach to panic recovery in worker pools:

   **1. Basic panic recovery in workers**

   ```go
   func (p *WorkerPool) worker(id int) {
       defer p.wg.Done()
       
       // Recover from panics
       defer func() {
           if r := recover(); r != nil {
               // Log the panic
               log.Printf("Worker %d panicked: %v\n%s", 
                   id, r, debug.Stack())
               
               // Restart the worker if needed
               go p.worker(id)
           }
       }()
       
       for task := range p.tasks {
           // Process task...
           result, err := p.processTask(task)
           
           // Send result
           p.results <- Result{
               TaskID: task.ID,
               Value:  result,
               Error:  err,
           }
       }
   }
   ```

   **2. Track worker status with panic monitoring**

   ```go
   type WorkerPool struct {
       tasks       chan Task
       results     chan Result
       workerCount int
       activeWorkers int
       panics      int64
       restarts    int64
       mu          sync.RWMutex
       wg          sync.WaitGroup
   }
   
   func (p *WorkerPool) worker(id int) {
       // Register this worker as active
       p.mu.Lock()
       p.activeWorkers++
       p.mu.Unlock()
       
       // Ensure we decrement the active count when done
       defer func() {
           p.mu.Lock()
           p.activeWorkers--
           p.mu.Unlock()
           p.wg.Done()
       }()
       
       // Recover from panics
       defer func() {
           if r := recover(); r != nil {
               // Record the panic
               atomic.AddInt64(&p.panics, 1)
               
               // Log details
               log.Printf("Worker %d panicked: %v\n%s", 
                   id, r, debug.Stack())
               
               // Start a replacement worker if pool is still running
               p.mu.RLock()
               shouldRestart := p.activeWorkers < p.workerCount
               p.mu.RUnlock()
               
               if shouldRestart {
                   atomic.AddInt64(&p.restarts, 1)
                   p.wg.Add(1)
                   go p.worker(id)
               }
           }
       }()
       
       for task := range p.tasks {
           // Process task...
       }
   }
   
   // GetStats returns statistics about the worker pool
   func (p *WorkerPool) GetStats() PoolStats {
       p.mu.RLock()
       active := p.activeWorkers
       p.mu.RUnlock()
       
       return PoolStats{
           ActiveWorkers: active,
           Panics:        atomic.LoadInt64(&p.panics),
           WorkerRestarts: atomic.LoadInt64(&p.restarts),
       }
   }
   ```

   **3. Task-level panic isolation**

   ```go
   func (p *WorkerPool) worker(id int) {
       defer p.wg.Done()
       
       for task := range p.tasks {
           // Execute task with panic recovery
           result := p.executeTaskSafely(id, task)
           
           // Send result
           p.results <- result
       }
   }
   
   func (p *WorkerPool) executeTaskSafely(workerID int, task Task) Result {
       // Prepare default result
       result := Result{
           TaskID:   task.ID,
           Success:  false,
           WorkerID: workerID,
           Error:    nil,
       }
       
       // Capture start time
       startTime := time.Now()
       
       // Execute with panic recovery
       func() {
           defer func() {
               if r := recover(); r != nil {
                   // Convert panic to error
                   err := fmt.Errorf("task panicked: %v", r)
                   result.Error = err
                   
                   // Log the details
                   log.Printf("Task %v panicked in worker %d: %v\n%s",
                       task.ID, workerID, r, debug.Stack())
               }
               
               // Always record execution time
               result.Duration = time.Since(startTime)
           }()
           
           // Execute the actual task handler
           value, err := task.Handler(task.Data)
           
           // If we get here, no panic occurred
           result.Value = value
           result.Error = err
           result.Success = err == nil
       }()
       
       return result
   }
   ```

   **4. Circuit breaker for repeatedly panicking tasks**

   ```go
   type CircuitBreakerPool struct {
       // ... standard pool fields
       panicCounts   map[string]int
       maxPanics     int
       blacklist     map[string]time.Time
       cooldownTime  time.Duration
       mu            sync.RWMutex
   }
   
   func (p *CircuitBreakerPool) executeTask(task Task) Result {
       // Check if task is blacklisted
       p.mu.RLock()
       blacklistedUntil, isBlacklisted := p.blacklist[task.ID]
       p.mu.RUnlock()
       
       // If blacklisted but cooldown expired, remove from blacklist
       if isBlacklisted {
           if time.Now().After(blacklistedUntil) {
               p.mu.Lock()
               delete(p.blacklist, task.ID)
               delete(p.panicCounts, task.ID)
               p.mu.Unlock()
               isBlacklisted = false
           }
       }
       
       // If still blacklisted, return error
       if isBlacklisted {
           return Result{
               TaskID:  task.ID,
               Success: false,
               Error:   fmt.Errorf("task blacklisted due to repeated failures"),
           }
       }
       
       // Execute task with panic recovery
       result := Result{TaskID: task.ID}
       func() {
           defer func() {
               if r := recover(); r != nil {
                   // Increment panic count
                   p.mu.Lock()
                   p.panicCounts[task.ID]++
                   panicCount := p.panicCounts[task.ID]
                   p.mu.Unlock()
                   
                   // Check if we should blacklist this task
                   if panicCount >= p.maxPanics {
                       p.mu.Lock()
                       p.blacklist[task.ID] = time.Now().Add(p.cooldownTime)
                       p.mu.Unlock()
                   }
                   
                   // Set error in result
                   result.Error = fmt.Errorf("task panicked: %v", r)
               }
           }()
           
           // Execute the task
           output, err := task.Handler(task.Data)
           result.Value = output
           result.Error = err
           result.Success = err == nil
       }()
       
       return result
   }
   ```

   **5. Comprehensive worker pool with panic handling**

   ```go
   type ResilientWorkerPool struct {
       // Configuration
       numWorkers    int
       maxPanicsPerWorker int
       maxPanicsPerTask int
       workerRestartDelay time.Duration
       
       // Channels
       tasks        chan Task
       results      chan Result
       control      chan controlMessage
       
       // State
       workers      map[int]*workerState
       tasks        map[string]taskStats
       mu           sync.RWMutex
       wg           sync.WaitGroup
       
       // Metrics
       metrics      PoolMetrics
   }
   
   type workerState struct {
       id          int
       panics      int
       lastPanic   time.Time
       tasks       int64
       active      bool
   }
   
   type taskStats struct {
       panics      int
       executions  int
       lastPanic   time.Time
       blacklisted bool
   }
   
   func (p *ResilientWorkerPool) worker(id int) {
       defer p.wg.Done()
       
       // Register worker
       p.registerWorker(id)
       
       // Recover from panics
       defer func() {
           if r := recover(); r != nil {
               // This should never happen since each task has its own panic recovery,
               // but we add this as an extra safety measure
               log.Printf("CRITICAL: Worker %d panicked outside of task execution: %v\n%s",
                   id, r, debug.Stack())
               
               // Update worker state
               p.recordWorkerPanic(id)
               
               // Restart the worker if not too many panics
               if p.shouldRestartWorker(id) {
                   time.Sleep(p.workerRestartDelay)
                   
                   p.wg.Add(1)
                   go p.worker(id)
               }
           }
           
           // Deregister worker
           p.deregisterWorker(id)
       }()
       
       // Main task processing loop
       for task := range p.tasks {
           // Skip blacklisted tasks
           if p.isTaskBlacklisted(task.ID) {
               p.results <- Result{
                   TaskID:   task.ID,
                   WorkerID: id,
                   Success:  false,
                   Error:    fmt.Errorf("task blacklisted due to repeated failures"),
               }
               continue
           }
           
           // Process task with panic recovery
           p.executeTask(id, task)
       }
   }
   
   func (p *ResilientWorkerPool) executeTask(workerID int, task Task) {
       // Record task execution
       p.recordTaskExecution(task.ID)
       
       // Execute with panic recovery
       result := Result{
           TaskID:   task.ID,
           WorkerID: workerID,
           StartTime: time.Now(),
       }
       
       func() {
           defer func() {
               // Set end time
               result.EndTime = time.Now()
               
               if r := recover(); r != nil {
                   // Log details
                   log.Printf("Task %s panicked in worker %d: %v\n%s",
                       task.ID, workerID, r, debug.Stack())
                   
                   // Record the panic
                   p.recordTaskPanic(task.ID)
                   p.recordWorkerPanic(workerID)
                   
                   // Set error
                   result.Success = false
                   result.Error = fmt.Errorf("panic: %v", r)
                   
                   // Check if task should be blacklisted
                   if p.shouldBlacklistTask(task.ID) {
                       p.blacklistTask(task.ID)
                   }
               }
           }()
           
           // Execute the task
           output, err := task.Handler(task.Data)
           
           result.Value = output
           result.Error = err
           result.Success = err == nil
       }()
       
       // Send the result
       p.results <- result
   }
   
   // Worker state management methods...
   func (p *ResilientWorkerPool) registerWorker(id int) {
       p.mu.Lock()
       defer p.mu.Unlock()
       
       p.workers[id] = &workerState{
           id:     id,
           active: true,
       }
       
       atomic.AddInt64(&p.metrics.ActiveWorkers, 1)
   }
   
   func (p *ResilientWorkerPool) deregisterWorker(id int) {
       p.mu.Lock()
       defer p.mu.Unlock()
       
       if worker, ok := p.workers[id]; ok && worker.active {
           worker.active = false
           atomic.AddInt64(&p.metrics.ActiveWorkers, -1)
       }
   }
   
   func (p *ResilientWorkerPool) recordWorkerPanic(id int) {
       p.mu.Lock()
       defer p.mu.Unlock()
       
       if worker, ok := p.workers[id]; ok {
           worker.panics++
           worker.lastPanic = time.Now()
           atomic.AddInt64(&p.metrics.TotalWorkerPanics, 1)
       }
   }
   
   func (p *ResilientWorkerPool) shouldRestartWorker(id int) bool {
       p.mu.RLock()
       defer p.mu.RUnlock()
       
       worker, ok := p.workers[id]
       if !ok {
           return false
       }
       
       return worker.panics < p.maxPanicsPerWorker
   }
   
   // Task tracking methods...
   func (p *ResilientWorkerPool) recordTaskExecution(taskID string) {
       p.mu.Lock()
       defer p.mu.Unlock()
       
       stats, ok := p.tasks[taskID]
       if !ok {
           stats = taskStats{}
       }
       
       stats.executions++
       p.tasks[taskID] = stats
   }
   
   func (p *ResilientWorkerPool) recordTaskPanic(taskID string) {
       p.mu.Lock()
       defer p.mu.Unlock()
       
       stats, ok := p.tasks[taskID]
       if !ok {
           stats = taskStats{}
       }
       
       stats.panics++
       stats.lastPanic = time.Now()
       p.tasks[taskID] = stats
       
       atomic.AddInt64(&p.metrics.TotalTaskPanics, 1)
   }
   
   func (p *ResilientWorkerPool) shouldBlacklistTask(taskID string) bool {
       p.mu.RLock()
       defer p.mu.RUnlock()
       
       if stats, ok := p.tasks[taskID]; ok {
           return stats.panics >= p.maxPanicsPerTask
       }
       
       return false
   }
   
   func (p *ResilientWorkerPool) blacklistTask(taskID string) {
       p.mu.Lock()
       defer p.mu.Unlock()
       
       if stats, ok := p.tasks[taskID]; ok {
           stats.blacklisted = true
           p.tasks[taskID] = stats
           atomic.AddInt64(&p.metrics.BlacklistedTasks, 1)
           
           log.Printf("Task %s blacklisted after %d panics", taskID, stats.panics)
       }
   }
   
   func (p *ResilientWorkerPool) isTaskBlacklisted(taskID string) bool {
       p.mu.RLock()
       defer p.mu.RUnlock()
       
       if stats, ok := p.tasks[taskID]; ok {
           return stats.blacklisted
       }
       
       return false
   }
   ```

   **Key principles for handling worker panics:**

   1. **Isolate panics**: Ensure a panic in one task doesn't affect others
   2. **Log comprehensively**: Include stack traces, task details, and worker IDs
   3. **Restart strategically**: Consider whether to restart workers after panics
   4. **Track problematic tasks**: Identify and potentially blacklist tasks that consistently panic
   5. **Maintain worker counts**: Ensure the worker pool maintains its desired concurrency level
   6. **Monitor panic rates**: Alert if panic rates exceed normal thresholds
   7. **Recover at the right level**: Recover either per-task or per-worker depending on your needs

   By implementing these techniques, you can create worker pools that gracefully handle panics, maintain stability, and provide detailed diagnostics when issues occur.

### 10. Context for Cancellation

**Concise Explanation:**
The context package in Go provides a way to carry deadlines, cancellation signals, and request-scoped values across API boundaries and between goroutines. It's particularly useful for controlling concurrency by allowing you to explicitly signal cancellation to all goroutines involved in a specific operation. This enables graceful shutdown, timeout handling, and resource cleanup in concurrent systems.

**Where to Use:**
- To implement timeouts for operations
- To propagate cancellation signals across goroutines
- In API handlers to cancel operations when clients disconnect
- To share request-scoped values between goroutines
- For graceful shutdown of concurrent operations
- To implement deadlines for multi-stage operations
- To limit resource usage by cancelling unnecessary work

**Code Snippet:**
```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// Basic context usage with cancellation and timeout
func main() {
    // Example 1: Simple cancellation
    fmt.Println("Example 1: Simple cancellation")
    simpleCancellation()
    
    // Example 2: Timeout context
    fmt.Println("\nExample 2: Context with timeout")
    contextWithTimeout()
    
    // Example 3: Context with values
    fmt.Println("\nExample 3: Context with values")
    contextWithValues()
    
    // Example 4: Propagating cancellation
    fmt.Println("\nExample 4: Propagating cancellation")
    propagatingCancellation()
}

// Example 1: Simple cancellation
func simpleCancellation() {
    // Create a cancellable context
    ctx, cancel := context.WithCancel(context.Background())
    
    // Start a goroutine that uses the context
    go func() {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("Goroutine: Cancellation received, stopping")
                return
            default:
                fmt.Println("Goroutine: Working...")
                time.Sleep(500 * time.Millisecond)
            }
        }
    }()
    
    // Let the goroutine run for a while
    time.Sleep(2 * time.Second)
    
    // Cancel the context
    fmt.Println("Main: Cancelling context")
    cancel()
    
    // Wait to see the goroutine stop
    time.Sleep(1 * time.Second)
}

// Example 2: Context with timeout
func contextWithTimeout() {
    // Create a context with a 2-second timeout
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel() // Always call cancel, even if the timeout expires
    
    // Start a task that might take too long
    result := make(chan int, 1)
    go func() {
        // Simulate work
        fmt.Println("Task: Starting work...")
        workTime := 3 * time.Second // This takes longer than the timeout
        
        // Check for cancellation during work
        select {
        case <-time.After(workTime):
            result <- 42
            fmt.Println("Task: Work completed (but too late)")
        case <-ctx.Done():
            fmt.Printf("Task: Work cancelled due to: %v\n", ctx.Err())
        }
    }()
    
    // Wait for result or timeout
    select {
    case res := <-result:
        fmt.Printf("Main: Got result: %d\n", res)
    case <-ctx.Done():
        fmt.Printf("Main: Operation timed out or cancelled: %v\n", ctx.Err())
    }
}

// Example 3: Context with values
func contextWithValues() {
    // Create a context with values
    ctx := context.Background()
    ctx = context.WithValue(ctx, "user_id", 42)
    ctx = context.WithValue(ctx, "auth_token", "secret-token")
    
    // Pass the context to another function
    processRequestWithContext(ctx)
}

func processRequestWithContext(ctx context.Context) {
    // Extract values from context
    userID, ok := ctx.Value("user_id").(int)
    if !ok {
        fmt.Println("User ID not found or not an integer")
        return
    }
    
    token, ok := ctx.Value("auth_token").(string)
    if !ok {
        fmt.Println("Auth token not found or not a string")
        return
    }
    
    fmt.Printf("Processing request for user %d with token: %s\n", userID, token)
    
    // Use context in a nested function
    performAuthenticatedOperation(ctx)
}

func performAuthenticatedOperation(ctx context.Context) {
    // Still have access to the same values
    userID, _ := ctx.Value("user_id").(int)
    fmt.Printf("Performing operation for user %d\n", userID)
    
    // We can also check for cancellation
    select {
    case <-ctx.Done():
        fmt.Printf("Operation cancelled: %v\n", ctx.Err())
        return
    default:
        // Continue with the operation
    }
}

// Example 4: Propagating cancellation
func propagatingCancellation() {
    // Parent context with cancellation
    parentCtx, parentCancel := context.WithCancel(context.Background())
    defer parentCancel()
    
    // Start multiple worker goroutines
    var wg sync.WaitGroup
    
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        
        // Create child context for each worker
        workerCtx, workerCancel := context.WithCancel(parentCtx)
        
        go func(id int, ctx context.Context, cancel context.CancelFunc) {
            defer wg.Done()
            defer cancel() // Ensure child context is cancelled when goroutine exits
            
            // Worker processing loop
            for {
                select {
                case <-ctx.Done():
                    fmt.Printf("Worker %d: Stopping due to: %v\n", id, ctx.Err())
                    return
                default:
                    // Simulate work
                    fmt.Printf("Worker %d: Working...\n", id)
                    
                    // Randomly fail sometimes
                    if rand.Intn(10) > 7 {
                        fmt.Printf("Worker %d: Encountered an error, cancelling my context\n", id)
                        cancel() // This worker's cancellation doesn't affect parent
                        return
                    }
                    
                    time.Sleep(500 * time.Millisecond)
                }
            }
        }(i, workerCtx, workerCancel)
    }
    
    // Let workers run for a bit
    time.Sleep(2 * time.Second)
    
    // Cancel parent context, which will cascade to all children
    fmt.Println("Main: Cancelling parent context")
    parentCancel()
    
    // Wait for all workers to finish
    wg.Wait()
    fmt.Println("All workers have stopped")
}
```

**Real-World Example:**
A concurrent file processing service with context-aware operations:

```go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "os"
    "path/filepath"
    "sync"
    "time"
)

// FileProcessor handles concurrent file processing operations
type FileProcessor struct {
    concurrency int
    timeout     time.Duration
    logger      *log.Logger
}

// ProcessingResult contains the outcome of file processing
type ProcessingResult struct {
    FilePath  string
    Success   bool
    BytesRead int64
    Error     error
    Duration  time.Duration
}

// NewFileProcessor creates a new file processor
func NewFileProcessor(concurrency int, timeout time.Duration) *FileProcessor {
    return &FileProcessor{
        concurrency: concurrency,
        timeout:     timeout,
        logger:      log.New(os.Stdout, "[FileProcessor] ", log.LstdFlags),
    }
}

// ProcessDirectory processes all files in a directory and its subdirectories
func (fp *FileProcessor) ProcessDirectory(root string, processor func(string) error) error {
    // Create a context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), fp.timeout)
    defer cancel()

    // Create channels
    fileCh := make(chan string, 100)
    errorCh := make(chan error, fp.concurrency)
    doneCh := make(chan struct{})

    // Start a goroutine to walk the directory
    go func() {
        defer close(fileCh)
        err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }

            // Skip directories
            if info.IsDir() {
                return nil
            }

            select {
            case fileCh <- path:
                // File path sent successfully
            case <-ctx.Done():
                // Context cancelled
                return ctx.Err()
            }

            return nil
        })

        if err != nil {
            select {
            case errorCh <- fmt.Errorf("error walking directory: %w", err):
            default:
                // Error channel full or closed
                fp.logger.Printf("Error walking directory: %v", err)
            }
        }
    }()

    // Start worker pool
    var wg sync.WaitGroup
    for i := 0; i < fp.concurrency; i++ {
        wg.Add(1)
        go func(workerID int) {
            defer wg.Done()

            for path := range fileCh {
                // Check if context is cancelled
                select {
                case <-ctx.Done():
                    return
                default:
                    // Continue processing
                }

                fp.logger.Printf("Worker %d processing file: %s", workerID, path)
                if err := processor(path); err != nil {
                    select {
                    case errorCh <- fmt.Errorf("error processing %s: %w", path, err):
                        // Error sent
                    default:
                        // Error channel full or closed
                        fp.logger.Printf("Error processing %s: %v", path, err)
                    }
                }
            }
            
            fp.logger.Printf("Worker %d finished", workerID)
        }(i)
    }

    // Close error channel when all workers are done
    go func() {
        wg.Wait()
        close(errorCh)
        close(doneCh)
    }()

    // Wait for completion or errors
    var errors []error

    for {
        select {
        case err, ok := <-errorCh:
            if !ok {
                // Error channel closed
                errorCh = nil
                continue
            }
            errors = append(errors, err)
        
        case <-doneCh:
            // All workers finished
            doneCh = nil
        
        case <-ctx.Done():
            // Timeout reached
            return fmt.Errorf("processing timed out after %v", fp.timeout)
        }

        // Exit when both channels are closed
        if errorCh == nil && doneCh == nil {
            break
        }
    }

    // Return combined errors if any
    if len(errors) > 0 {
        errorMessages := make([]string, len(errors))
        for i, err := range errors {
            errorMessages[i] = err.Error()
        }
        return fmt.Errorf("encountered %d errors: %s", len(errors), strings.Join(errorMessages, "; "))
    }

    return nil
}

func main() {
    // Create a file processor with 4 workers and 5 minute timeout
    processor := NewFileProcessor(4, 5*time.Minute)
    
    // Define a simple file processor function
    processFile := func(path string) error {
        // Simulate file processing
        time.Sleep(100 * time.Millisecond)
        
        // Example: Skip files larger than 100MB
        info, err := os.Stat(path)
        if err != nil {
            return err
        }
        
        if info.Size() > 100*1024*1024 {
            return fmt.Errorf("file too large: %d bytes", info.Size())
        }
        
        fmt.Printf("Processed file: %s (%d bytes)\n", path, info.Size())
        return nil
    }
    
    // Process all files in the current directory
    if err := processor.ProcessDirectory(".", processFile); err != nil {
        log.Fatalf("Error processing directory: %v", err)
    }
    
    fmt.Println("All files processed successfully!")
}
