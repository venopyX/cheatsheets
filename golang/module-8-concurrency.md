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

### 10. Context for Cancellation and Timeouts

**Concise Explanation:**
The context package in Go provides a way to carry deadlines, cancellation signals, and request-scoped values across API boundaries and between goroutines. It's particularly useful in concurrent programs to propagate cancellation signals, implement timeouts, and ensure resources are freed when operations are no longer needed. Context allows parent operations to control child operations, enabling clean cancellation hierarchies.

**Where to Use:**
- To propagate cancellation signals across goroutines
- To implement timeouts for operations
- To carry request-scoped values through your program
- For controlling the lifetime of concurrent operations
- To implement graceful shutdown of services
- When working with APIs that accept context
- To manage resources across concurrent operations
- To implement deadline-aware processing

**Code Snippet:**
```go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "time"
)

func main() {
    // Create a base context
    ctx := context.Background()
    
    // Example 1: Context with a deadline (timeout)
    fmt.Println("Example 1: Context with deadline")
    deadlineCtx, cancel := context.WithDeadline(ctx, time.Now().Add(2*time.Second))
    defer cancel() // Always call cancel to avoid resource leaks
    
    go func() {
        // Simulate work that respects the deadline
        if err := doWorkWithDeadline(deadlineCtx); err != nil {
            fmt.Printf("Deadline work error: %v\n", err)
        }
    }()
    
    // Wait to see the result
    time.Sleep(3 * time.Second)
    
    // Example 2: Context with timeout
    fmt.Println("\nExample 2: Context with timeout")
    timeoutCtx, cancel := context.WithTimeout(ctx, 1*time.Second)
    defer cancel()
    
    go func() {
        // Simulate work that respects timeout
        if err := doWorkWithTimeout(timeoutCtx); err != nil {
            fmt.Printf("Timeout work error: %v\n", err)
        }
    }()
    
    // Wait to see the result
    time.Sleep(2 * time.Second)
    
    // Example 3: Context with cancellation
    fmt.Println("\nExample 3: Context with cancellation")
    cancelCtx, cancel := context.WithCancel(ctx)
    
    go func() {
        // Simulate work that respects cancellation
        if err := doWorkWithCancellation(cancelCtx); err != nil {
            fmt.Printf("Cancellation work error: %v\n", err)
        }
    }()
    
    // Let it run briefly, then cancel
    time.Sleep(500 * time.Millisecond)
    fmt.Println("Cancelling the operation")
    cancel()
    time.Sleep(1 * time.Second)
    
    // Example 4: Context with value
    fmt.Println("\nExample 4: Context with value")
    type key string
    userKey := key("user")
    
    valueCtx := context.WithValue(ctx, userKey, "john_doe")
    
    go func() {
        processWithUserInfo(valueCtx, userKey)
    }()
    
    // Wait to see the result
    time.Sleep(1 * time.Second)
    
    // Example 5: HTTP request with context
    fmt.Println("\nExample 5: HTTP request with context")
    serverCtx, serverCancel := context.WithCancel(ctx)
    defer serverCancel()
    
    // Start HTTP server
    server := startServer(serverCtx)
    
    // Make a request with timeout
    makeRequestWithTimeout()
    
    // Shut down the server
    serverCancel()
    server.Shutdown(context.Background())
    
    fmt.Println("\nAll examples complete")
}

// Function that does work respecting a deadline
func doWorkWithDeadline(ctx context.Context) error {
    // Check if deadline is set
    deadline, ok := ctx.Deadline()
    if ok {
        fmt.Printf("Deadline set to: %v (in %v)\n", deadline, time.Until(deadline))
    } else {
        fmt.Println("No deadline set")
    }
    
    // Simulate work that takes 3 seconds (longer than the deadline)
    for i := 0; i < 3; i++ {
        // Check if context is done before each unit of work
        select {
        case <-ctx.Done():
            return fmt.Errorf("work cancelled due to deadline: %v", ctx.Err())
        case <-time.After(1 * time.Second):
            fmt.Printf("Completed %d/3 of deadline work\n", i+1)
        }
    }
    
    return nil
}

// Function that does work respecting a timeout
func doWorkWithTimeout(ctx context.Context) error {
    // Loop until context is cancelled or timeout reached
    for i := 0; ; i++ {
        // Check if we should stop
        select {
        case <-ctx.Done():
            return fmt.Errorf("work cancelled due to timeout: %v", ctx.Err())
        default:
            // Continue working
        }
        
        // Simulate some work
        fmt.Printf("Doing timeout work unit %d\n", i+1)
        time.Sleep(400 * time.Millisecond)
    }
}

// Function that does work respecting cancellation
func doWorkWithCancellation(ctx context.Context) error {
    for i := 0; ; i++ {
        select {
        case <-ctx.Done():
            return fmt.Errorf("work cancelled by caller: %v", ctx.Err())
        default:
            // Continue working
        }
        
        // Simulate work
        fmt.Printf("Doing cancellable work unit %d\n", i+1)
        time.Sleep(200 * time.Millisecond)
    }
}

// Function that uses context values
func processWithUserInfo(ctx context.Context, userKey interface{}) {
    // Extract user from context
    if username, ok := ctx.Value(userKey).(string); ok {
        fmt.Printf("Processing request for user: %s\n", username)
    } else {
        fmt.Println("No user found in context")
    }
}

// HTTP server that uses context for shutdown
func startServer(ctx context.Context) *http.Server {
    server := &http.Server{
        Addr: ":8080",
        Handler: http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Get context from request
            reqCtx := r.Context()
            
            fmt.Println("Processing request...")
            
            // Simulate work with potential timeout from request context
            select {
            case <-time.After(1 * time.Second):
                fmt.Fprintln(w, "Request processed successfully")
                fmt.Println("Request completed")
            case <-reqCtx.Done():
                // Client cancelled the request
                fmt.Println("Request cancelled by client:", reqCtx.Err())
                return
            }
        }),
    }
    
    // Start server in a goroutine
    go func() {
        fmt.Println("Starting HTTP server on :8080")
        if err := server.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatalf("HTTP server error: %v", err)
        }
        fmt.Println("HTTP server stopped")
    }()
    
    // Monitor the context for cancellation to shut down server
    go func() {
        <-ctx.Done()
        fmt.Println("Shutting down HTTP server...")
        
        // Create a timeout for graceful shutdown
        shutdownCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()
        
        if err := server.Shutdown(shutdownCtx); err != nil {
            log.Fatalf("HTTP server shutdown error: %v", err)
        }
    }()
    
    return server
}

// Make an HTTP request with timeout
func makeRequestWithTimeout() {
    // Create a context with timeout for the request
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    // Create request with context
    req, err := http.NewRequestWithContext(ctx, "GET", "http://localhost:8080", nil)
    if err != nil {
        fmt.Printf("Error creating request: %v\n", err)
        return
    }
    
    fmt.Println("Sending HTTP request with 2-second timeout")
    
    // Send the request
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Printf("HTTP request error: %v\n", err)
        return
    }
    defer resp.Body.Close()
    
    fmt.Printf("HTTP response status: %s\n", resp.Status)
}
```

**Real-World Example:**
A service that performs distributed operations with context-based coordination:

```go
package main

import (
    "context"
    "encoding/json"
    "errors"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "sync"
    "syscall"
    "time"
)

// Request represents an incoming job request
type Request struct {
    ID         string          `json:"id"`
    Operations []string        `json:"operations"`
    Timeout    int             `json:"timeout_seconds"`
    Data       json.RawMessage `json:"data"`
}

// Result represents the outcome of a job
type Result struct {
    RequestID string                 `json:"request_id"`
    Status    string                 `json:"status"`
    Results   map[string]interface{} `json:"results,omitempty"`
    Error     string                 `json:"error,omitempty"`
    StartTime time.Time              `json:"start_time"`
    EndTime   time.Time              `json:"end_time"`
}

// Service handles processing of distributed operations
type Service struct {
    server         *http.Server
    operationMap   map[string]Operation
    activeRequests sync.Map
    mu             sync.RWMutex
}

// Operation represents a unit of work that can be performed
type Operation func(ctx context.Context, data json.RawMessage) (interface{}, error)

// NewService creates a new service
func NewService() *Service {
    service := &Service{
        operationMap: make(map[string]Operation),
    }
    
    // Register available operations
    service.registerOperations()
    
    // Set up HTTP server with handlers
    mux := http.NewServeMux()
    mux.HandleFunc("/process", service.handleProcess)
    mux.HandleFunc("/status/", service.handleStatus)
    mux.HandleFunc("/cancel/", service.handleCancel)
    
    service.server = &http.Server{
        Addr:    ":8080",
        Handler: mux,
    }
    
    return service
}

// Start begins the service
func (s *Service) Start() {
    // Start HTTP server
    go func() {
        log.Println("Starting service on :8080")
        if err := s.server.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatalf("HTTP server error: %v", err)
        }
    }()
}

// Shutdown gracefully stops the service
func (s *Service) Shutdown(ctx context.Context) error {
    log.Println("Shutting down service...")
    
    // First, shutdown HTTP server
    if err := s.server.Shutdown(ctx); err != nil {
        return err
    }
    
    // Then cancel all active requests
    var wg sync.WaitGroup
    
    s.activeRequests.Range(func(key, value interface{}) bool {
        requestID := key.(string)
        cancelFunc := value.(context.CancelFunc)
        
        wg.Add(1)
        go func() {
            defer wg.Done()
            log.Printf("Cancelling request %s due to shutdown", requestID)
            cancelFunc()
        }()
        return true
    })
    
    // Wait for all cancellations to complete or timeout
    done := make(chan struct{})
    go func() {
        wg.Wait()
        close(done)
    }()
    
    select {
    case <-done:
        log.Println("All requests cancelled successfully")
    case <-ctx.Done():
        log.Println("Timeout waiting for request cancellations")
    }
    
    return nil
}

// Register operations that this service can perform
func (s *Service) registerOperations() {
    // Simulated database operation
    s.operationMap["fetch_data"] = func(ctx context.Context, data json.RawMessage) (interface{}, error) {
        // Parse parameters
        var params struct {
            Query string `json:"query"`
            Limit int    `json:"limit"`
        }
        
        if err := json.Unmarshal(data, &params); err != nil {
            return nil, fmt.Errorf("invalid parameters: %w", err)
        }
        
        // Check context before expensive operation
        if err := ctx.Err(); err != nil {
            return nil, err
        }
        
        // Simulate database fetch with timeout awareness
        return performDatabaseOperation(ctx, params.Query, params.Limit)
    }
    
    // Simulated external API call
    s.operationMap["external_api"] = func(ctx context.Context, data json.RawMessage) (interface{}, error) {
        // Parse parameters
        var params struct {
            Endpoint string `json:"endpoint"`
            Method   string `json:"method"`
            Payload  json.RawMessage `json:"payload"`
        }
        
        if err := json.Unmarshal(data, &params); err != nil {
            return nil, fmt.Errorf("invalid parameters: %w", err)
        }
        
        // Make API call with timeout awareness
        return callExternalAPI(ctx, params.Endpoint, params.Method, params.Payload)
    }
    
    // Simulated computation
    s.operationMap["compute"] = func(ctx context.Context, data json.RawMessage) (interface{}, error) {
        // Parse parameters
        var params struct {
            Algorithm string `json:"algorithm"`
            Input     json.RawMessage `json:"input"`
        }
        
        if err := json.Unmarshal(data, &params); err != nil {
            return nil, fmt.Errorf("invalid parameters: %w", err)
        }
        
        // Perform computation with timeout awareness
        return performComputation(ctx, params.Algorithm, params.Input)
    }
}

// HTTP handler for processing requests
func (s *Service) handleProcess(w http.ResponseWriter, r *http.Request) {
    // Only allow POST method
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // Parse the request
    var req Request
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, fmt.Sprintf("Invalid request: %v", err), http.StatusBadRequest)
        return
    }
    
    // Generate ID if not provided
    if req.ID == "" {
        req.ID = fmt.Sprintf("req-%d", time.Now().UnixNano())
    }
    
    // Validate timeout
    if req.Timeout <= 0 {
        req.Timeout = 30 // Default timeout
    }
    
    // Create context with timeout
    ctx, cancel := context.WithTimeout(r.Context(), time.Duration(req.Timeout)*time.Second)
    
    // Store cancellation function for potential explicit cancellation
    s.activeRequests.Store(req.ID, cancel)
    defer func() {
        s.activeRequests.Delete(req.ID)
    }()
    
    // Start processing in a goroutine
    go func() {
        defer cancel() // Ensure context is cancelled when done
        
        result := s.processRequest(ctx, req)
        
        // Store the result (in a real system, this would be persistent)
        s.storeResult(req.ID, result)
    }()
    
    // Return accepted response
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusAccepted)
    json.NewEncoder(w).Encode(map[string]string{
        "request_id": req.ID,
        "status":     "processing",
        "message":    "Request accepted for processing",
    })
}

// HTTP handler for checking request status
func (s *Service) handleStatus(w http.ResponseWriter, r *http.Request) {
    // Extract request ID from URL path
    requestID := r.URL.Path[len("/status/"):]
    if requestID == "" {
        http.Error(w, "Missing request ID", http.StatusBadRequest)
        return
    }
    
    // Check if request is still active
    if _, ok := s.activeRequests.Load(requestID); ok {
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]string{
            "request_id": requestID,
            "status":     "processing",
            "message":    "Request is still processing",
        })
        return
    }
    
    // Check for stored result
    result, found := s.getResult(requestID)
    if !found {
        http.Error(w, "Request not found", http.StatusNotFound)
        return
    }
    
    // Return the result
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(result)
}

// HTTP handler for cancelling requests
func (s *Service) handleCancel(w http.ResponseWriter, r *http.Request) {
    // Extract request ID from URL path
    requestID := r.URL.Path[len("/cancel/"):]
    if requestID == "" {
        http.Error(w, "Missing request ID", http.StatusBadRequest)
        return
    }
    
    // Check if request exists and can be cancelled
    if cancelFunc, ok := s.activeRequests.Load(requestID); ok {
        // Call the cancellation function
        cancelFunc.(context.CancelFunc)()
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]string{
            "request_id": requestID,
            "status":     "cancelled",
            "message":    "Request has been cancelled",
        })
    } else {
        // Request not found or already completed
        http.Error(w, "Request not found or already completed", http.StatusNotFound)
    }
}

// Process a request with its operations
func (s *Service) processRequest(ctx context.Context, req Request) Result {
    result := Result{
        RequestID: req.ID,
        Status:    "processing",
        Results:   make(map[string]interface{}),
        StartTime: time.Now(),
    }
    
    // Check if we have valid operations
    if len(req.Operations) == 0 {
        result.Status = "failed"
        result.Error = "no operations specified"
        result.EndTime = time.Now()
        return result
    }
    
    // Set up a wait group to track operations
    var wg sync.WaitGroup
    resultsMutex := sync.Mutex{}
    
    // Execute operations concurrently
    for _, opName := range req.Operations {
        // Check if the operation exists
        s.mu.RLock()
        operation, ok := s.operationMap[opName]
        s.mu.RUnlock()
        
        if !ok {
            resultsMutex.Lock()
            result.Results[opName] = map[string]interface{}{
                "status": "failed",
                "error":  "operation not found",
            }
            resultsMutex.Unlock()
            continue
        }
        
        // Start this operation
        wg.Add(1)
        go func(name string, op Operation) {
            defer wg.Done()
            
            // Execute the operation with a deadline
            opResult, err := op(ctx, req.Data)
            
            // Store the result
            resultsMutex.Lock()
            defer resultsMutex.Unlock()
            
            if err != nil {
                result.Results[name] = map[string]interface{}{
                    "status": "failed",
                    "error":  err.Error(),
                }
            } else {
                result.Results[name] = map[string]interface{}{
                    "status": "success",
                    "data":   opResult,
                }
            }
        }(opName, operation)
    }
    
    // Wait for all operations to complete or context to be cancelled
    done := make(chan struct{})
    go func() {
        wg.Wait()
        close(done)
    }()
    
    // Wait for completion or cancellation
    select {
    case <-done:
        result.Status = "completed"
    case <-ctx.Done():
        result.Status = "cancelled"
        result.Error = ctx.Err().Error()
    }
    
    result.EndTime = time.Now()
    return result
}

// In a real system, results would be stored persistently
// For simplicity, we're using an in-memory map
var resultStore = make(map[string]Result)
var resultStoreMutex sync.RWMutex

func (s *Service) storeResult(id string, result Result) {
    resultStoreMutex.Lock()
    defer resultStoreMutex.Unlock()
    resultStore[id] = result
}

func (s *Service) getResult(id string) (Result, bool) {
    resultStoreMutex.RLock()
    defer resultStoreMutex.RUnlock()
    result, found := resultStore[id]
    return result, found
}

// Simulated operations
func performDatabaseOperation(ctx context.Context, query string, limit int) (interface{}, error) {
    // Check if already cancelled
    if err := ctx.Err(); err != nil {
        return nil, err
    }
    
    // Simulate query execution that takes time
    log.Printf("Executing database query: %s (limit: %d)", query, limit)
    
    // Check context periodically during execution
    select {
    case <-time.After(1500 * time.Millisecond):
        // Continue with operation
    case <-ctx.Done():
        return nil, fmt.Errorf("database operation cancelled: %w", ctx.Err())
    }
    
    // Generate some fake results
    results := make([]map[string]interface{}, 0, limit)
    for i := 0; i < limit; i++ {
        results = append(results, map[string]interface{}{
            "id":    fmt.Sprintf("row-%d", i),
            "value": fmt.Sprintf("Value %d for %s", i, query),
        })
        
        // Check context after each item (allows for fast cancellation)
        if i%10 == 0 && ctx.Err() != nil {
            return nil, fmt.Errorf("database operation cancelled while generating results: %w", ctx.Err())
        }
    }
    
    return results, nil
}

func callExternalAPI(ctx context.Context, endpoint string, method string, payload json.RawMessage) (interface{}, error) {
    // Create a request with timeout from context
    log.Printf("Calling API %s %s", method, endpoint)
    
    // Create HTTP client that respects context deadlines
    client := &http.Client{}
    req, err := http.NewRequestWithContext(ctx, method, endpoint, nil)
    if err != nil {
        return nil, fmt.Errorf("creating request: %w", err)
    }
    
    // Set content type if we have a payload
    if len(payload) > 0 {
        req.Header.Set("Content-Type", "application/json")
    }
    
    // Make the request
    resp, err := client.Do(req)
    if err != nil {
        // Check if it was due to context cancellation
        if errors.Is(err, context.Canceled) || errors.Is(err, context.DeadlineExceeded) {
            return nil, fmt.Errorf("API call cancelled: %w", ctx.Err())
        }
        return nil, fmt.Errorf("API call failed: %w", err)
    }
    defer resp.Body.Close()
    
    // Parse the response
    var result interface{}
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        return nil, fmt.Errorf("parsing API response: %w", err)
    }
    
    return result, nil
}

func performComputation(ctx context.Context, algorithm string, input json.RawMessage) (interface{}, error) {
    log.Printf("Performing computation using %s algorithm", algorithm)
    
    // Check algorithm type
    var result interface{}
    var err error
    
    switch algorithm {
    case "fibonacci":
        var n int
        if err := json.Unmarshal(input, &n); err != nil {
            return nil, fmt.Errorf("invalid input for fibonacci: %w", err)
        }
        
        result, err = computeFibonacci(ctx, n)
        
    case "sort":
        var items []int
        if err := json.Unmarshal(input, &items); err != nil {
            return nil, fmt.Errorf("invalid input for sort: %w", err)
        }
        
        result, err = sortItems(ctx, items)
        
    default:
        return nil, fmt.Errorf("unknown algorithm: %s", algorithm)
    }
    
    if err != nil {
        return nil, err
    }
    
    return result, nil
}

// Compute Fibonacci number with context cancellation support
func computeFibonacci(ctx context.Context, n int) (int, error) {
    // Base cases
    if n <= 0 {
        return 0, nil
    }
    if n == 1 {
        return 1, nil
    }
    
    // Check context before computation
    if err := ctx.Err(); err != nil {
        return 0, err
    }
    
    // For larger values, use iterative approach with periodic context checks
    if n > 10 {
        a, b := 0, 1
        for i := 2; i <= n; i++ {
            a, b = b, a+b
            
            // Periodically check for cancellation
            if i%100 == 0 {
                if err := ctx.Err(); err != nil {
                    return 0, fmt.Errorf("fibonacci computation cancelled: %w", err)
                }
            }
        }
        return b, nil
    }
    
    // For smaller values, use recursive approach
    // (In a real implementation, you'd use memoization)
    fib1, err := computeFibonacci(ctx, n-1)
    if err != nil {
        return 0, err
    }
    
    fib2, err := computeFibonacci(ctx, n-2)
    if err != nil {
        return 0, err
    }
    
    return fib1 + fib2, nil
}

// Sort items with context cancellation support
func sortItems(ctx context.Context, items []int) ([]int, error) {
    // Check context before starting
    if err := ctx.Err(); err != nil {
        return nil, err
    }
    
    // Make a copy to avoid modifying the input
    result := make([]int, len(items))
    copy(result, items)
    
    // Use a simple bubble sort with context checks
    // (In production, you'd use a more efficient algorithm)
    for i := 0; i < len(result); i++ {
        // Check context periodically
        if i%100 == 0 {
            if err := ctx.Err(); err != nil {
                return nil, fmt.Errorf("sort cancelled: %w", err)
            }
        }
        
        for j := 0; j < len(result)-i-1; j++ {
            if result[j] > result[j+1] {
                result[j], result[j+1] = result[j+1], result[j]
            }
        }
    }
    
    return result, nil
}

func main() {
    // Create a new service
    service := NewService()
    
    // Start the service
    service.Start()
    
    // Set up signal handling for graceful shutdown
    signalCh := make(chan os.Signal, 1)
    signal.Notify(signalCh, syscall.SIGINT, syscall.SIGTERM)
    
    // Wait for termination signal
    sig := <-signalCh
    log.Printf("Received signal: %v, initiating graceful shutdown", sig)
    
    // Create a shutdown context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    // Shutdown the service
    if err := service.Shutdown(ctx); err != nil {
        log.Fatalf("Service shutdown error: %v", err)
    }
    
    log.Println("Service shutdown complete")
}
```

**Common Pitfalls:**
- Not cancelling contexts to release resources, causing leaks
- Passing the wrong context through API boundaries
- Using context.Background() or context.TODO() inappropriately
- Storing request-scoped values in global variables instead of context
- Using context.Value() for critical parameters instead of function arguments
- Over-nesting contexts, making them hard to track
- Not checking ctx.Err() in long-running operations
- Using context for long-term storage rather than request-scoped data
- Creating deeply nested context hierarchies that are hard to reason about
- Not handling timeouts correctly in resource cleanup

**Confusion Questions:**

1. **Q: When should I use context.Background() vs. context.TODO()?**

   A: Understanding when to use `context.Background()` versus `context.TODO()` is important for writing clear and maintainable Go code. Although both functions return an empty context with no values, deadlines, or cancellation, they have different semantic meanings:

   **context.Background()**

   ```go
   ctx := context.Background()
   ```

   **When to use context.Background()**:

   1. **As the root context** for your application or long-running process
      ```go
      func main() {
          ctx := context.Background()
          // Derive other contexts from this root
          ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
          defer cancel()
          // ...
      }
      ```

   2. **For the "top-level" of a request chain**
      ```go
      func startProcess() {
          ctx := context.Background()
          processRequest(ctx, req)
      }
      ```

   3. **For initialization code** where no existing context is available
      ```go
      func initializeApp() {
          ctx := context.Background()
          db, err := connectToDatabase(ctx)
          // ...
      }
      ```

   4. **For tests** when you need a simple context
      ```go
      func TestMyFunction(t *testing.T) {
          ctx := context.Background()
          result, err := myFunction(ctx, args)
          // ...
      }
      ```

   **context.TODO()**

   ```go
   ctx := context.TODO()
   ```

   **When to use context.TODO()**:

   1. **As a placeholder** when you're not sure which context to use but plan to fix it later
      ```go
      // FIXME: Replace with proper context once we refactor the API
      ctx := context.TODO()
      ```

   2. **During incremental code migration** to a context-aware API
      ```go
      // We're in the process of adding context to all functions
      func legacyFunction() {
          ctx := context.TODO() // Will be replaced with a proper context later
          newContextAwareFunction(ctx)
      }
      ```

   3. **When you expect a context will be needed** but don't have one yet
      ```go
      func someFunction() {
          // We know we'll need a context eventually but don't have one yet
          ctx := context.TODO()
          // ...
      }
      ```

   **Practical differences:**

   While both functions are functionally identical (they both return an empty context), they signal different intent:

   - **Background()**: "This is deliberately a new, empty context, appropriate for this root position"
   - **TODO()**: "I should have a specific context here, but I don't yet, and I intend to fix this later"

   **Best practices:**

   1. **Prefer explicit context passing** over creating new root contexts
      ```go
      // Better design: Pass context from caller
      func processTask(ctx context.Context, task Task) error {
          // Use passed context
      }
      
      // Avoid creating new root contexts inside functions
      func badProcessTask(task Task) error {
          ctx := context.Background() // Don't do this inside functions
          // ...
      }
      ```

   2. **Use Background() for the main entry points** of your application
      ```go
      func main() {
          ctx := context.Background()
          // Create derived contexts with timeouts, cancellation, etc.
      }
      ```

   3. **Use TODO() for temporary solutions** that will be fixed
      ```go
      // Mark clearly that this is temporary
      // TODO: Refactor to accept context from caller
      ctx := context.TODO()
      ```

   4. **Document why you're using TODO()** to make refactoring easier
      ```go
      // TODO: Replace with request context once we update the API
      ctx := context.TODO()
      ```

   5. **Set up lint rules** to flag instances of `context.TODO()` in production code

   In summary, use `context.Background()` when you explicitly need a new root context, and `context.TODO()` when you're temporarily filling a gap that should be addressed in future refactoring. In practice, you'll use `context.Background()` much more frequently in production code.

2. **Q: How do I properly pass contexts through my application?**

   A: Properly passing contexts through your application ensures effective cancellation signals, timeouts, and request-scoped values. Here's a comprehensive guide:

   **Core principles for passing contexts:**

   1. **Pass contexts explicitly as the first parameter**
      ```go
      func ProcessOrder(ctx context.Context, orderID string) error {
          // Use context here
      }
      ```

   2. **Never store contexts in structs** as a general field
      ```go
      // DON'T do this
      type BadService struct {
          ctx context.Context // Bad practice
          // ...
      }
      
      // DO this instead
      type GoodService struct {
          // No context as a field
      }
      
      // Pass context in method calls
      func (s *GoodService) Process(ctx context.Context, data Data) {
          // ...
      }
      ```

   3. **Pass contexts through all layers** of your application
      ```go
      // Controller layer
      func (c *OrderController) CreateOrder(w http.ResponseWriter, r *http.Request) {
          // Get context from request
          ctx := r.Context()
          
          // Pass to service layer
          order, err := c.orderService.CreateOrder(ctx, orderData)
          // ...
      }
      
      // Service layer
      func (s *OrderService) CreateOrder(ctx context.Context, data OrderData) (*Order, error) {
          // Pass to repository layer
          return s.orderRepo.Create(ctx, data)
      }
      
      // Repository layer
      func (r *OrderRepository) Create(ctx context.Context, data OrderData) (*Order, error) {
          // Use context for database operations
          return r.db.ExecContext(ctx, "INSERT...", data.Fields...)
      }
      ```

   **For HTTP servers:**

   ```go
   func handler(w http.ResponseWriter, r *http.Request) {
       // Get context from the request
       ctx := r.Context()
       
       // Add request-specific values if needed
       ctx = context.WithValue(ctx, userIDKey, getUserID(r))
       
       // Create a timeout for this specific request
       ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
       defer cancel()
       
       // Use the context in downstream operations
       result, err := processRequest(ctx, r.URL.Query())
       // ...
   }
   ```

   **For long-running workers:**

   ```go
   func Worker(ctx context.Context) {
       for {
           select {
           case <-ctx.Done():
               // Context cancelled, exit worker
               log.Printf("Worker exiting: %v", ctx.Err())
               return
               
           case job := <-jobQueue:
               // Create a child context for this job with timeout
               jobCtx, cancel := context.WithTimeout(ctx, 30*time.Second)
               
               // Process the job with its own context
               processJob(jobCtx, job)
               
               // Clean up this job's context
               cancel()
           }
       }
   }
   ```

   **Context hierarchy management:**

   ```go
   // Starting with an application context
   appCtx, appCancel := context.WithCancel(context.Background())
   defer appCancel()
   
   // Create a request context from app context
   requestCtx, requestCancel := context.WithTimeout(appCtx, 10*time.Second)
   defer requestCancel()
   
   // Create operation-specific context
   operationCtx, operationCancel := context.WithTimeout(requestCtx, 2*time.Second)
   defer operationCancel()
   
   // If parent contexts are cancelled, all children are also cancelled
   // If operation times out, request can still continue with other operations
   ```

   **Handling context across API boundaries:**

   ```go
   // API Gateway pattern
   func (g *Gateway) CallExternalService(ctx context.Context, req Request) (*Response, error) {
       // Create a specific timeout for external call
       callCtx, cancel := context.WithTimeout(ctx, g.callTimeout)
       defer cancel()
       
       // Make the HTTP request with the derived context
       httpReq, err := http.NewRequestWithContext(callCtx, http.MethodPost, g.serviceURL, body)
       if err != nil {
           return nil, err
       }
       
       // Add any required headers
       for k, v := range g.headers {
           httpReq.Header.Set(k, v)
       }
       
       // Make the call
       resp, err := g.client.Do(httpReq)
       // ...
   }
   ```

   **Context in asynchronous operations:**

   ```go
   func ProcessAsync(ctx context.Context, data Data) <-chan Result {
       resultCh := make(chan Result, 1)
       
       go func() {
           defer close(resultCh)
           
           // Use the same context for this goroutine
           select {
           case <-ctx.Done():
               // Context cancelled before we even start
               resultCh <- Result{Err: ctx.Err()}
               return
           default:
               // Continue with processing
           }
           
           // Do work that respects the context
           result, err := doWork(ctx, data)
           
           // Send results, but respect context cancellation
           select {
           case resultCh <- Result{Value: result, Err: err}:
               // Result sent successfully
           case <-ctx.Done():
               // Context cancelled, result no longer needed
               return
           }
       }()
       
       return resultCh
   }
   ```

   **Middleware approach for web servers:**

   ```go
   // Timeout middleware
   func TimeoutMiddleware(timeout time.Duration) func(http.Handler) http.Handler {
       return func(next http.Handler) http.Handler {
           return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
               // Get the existing context
               ctx := r.Context()
               
               // Create a timeout context
               ctx, cancel := context.WithTimeout(ctx, timeout)
               defer cancel()
               
               // Create new request with the timeout context
               r = r.WithContext(ctx)
               
               // Call the next handler
               next.ServeHTTP(w, r)
           })
       }
   }
   
   // Usage
   http.Handle("/api/", TimeoutMiddleware(5*time.Second)(apiHandler))
   ```

   **Request tracing with context:**

   ```go
   // Define a key for the trace ID
   type traceIDKey struct{}
   
   // Middleware to add trace ID
   func TracingMiddleware(next http.Handler) http.Handler {
       return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
           // Generate or get trace ID from request
           traceID := r.Header.Get("X-Trace-ID")
           if traceID == "" {
               traceID = generateTraceID()
           }
           
           // Add to context
           ctx := context.WithValue(r.Context(), traceIDKey{}, traceID)
           
           // Add to response headers
           w.Header().Set("X-Trace-ID", traceID)
           
           // Continue with modified context
           next.ServeHTTP(w, r.WithContext(ctx))
       })
   }
   
   // Helper to extract trace ID
   func GetTraceID(ctx context.Context) string {
       if id, ok := ctx.Value(traceIDKey{}).(string); ok {
           return id
       }
       return "unknown"
   }
   
   // Usage in handlers and services
   func processRequest(ctx context.Context) {
       traceID := GetTraceID(ctx)
       log.Printf("[Trace: %s] Processing request", traceID)
       // ...
   }
   ```

   **Best practices for context cancellation:**

   ```go
   // Parent function
   func handleRequest(w http.ResponseWriter, r *http.Request) {
       ctx := r.Context()
       
       // For operations that should have an independent timeout
       ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
       defer cancel() // Always call cancel to avoid resource leaks
       
       // Call function with context
       result, err := processOperation(ctx)
       
       // Cancel here instead if you want to stop early
       // cancel()
   }
   ```

   **Key takeaways:**

   1. **Pass context as first argument** in all functions that do I/O or may be long-running
   2. **Do not store context in structs** except in rare cases for request-scope objects
   3. **Cancel contexts when operations complete** to free resources
   4. **Create specific contexts with appropriate timeouts** for different operations
   5. **Only store request-scoped values** in context, not general application configuration
   6. **Document the expected contents** of your context values
   7. **Use custom types for context keys** to avoid collisions

   By consistently following these patterns, you'll ensure that cancellation signals, deadlines, and request-scoped values flow properly through your application, making it more robust and maintainable.

3. **Q: How do I handle derived contexts and cancellation properly?**

   A: Handling derived contexts and cancellation properly is essential for resource management and controlling the flow of concurrent operations. Here's a comprehensive guide:

   **Understanding Context Hierarchies**

   In Go, contexts form a tree-like hierarchy:
   - When you derive a new context from a parent, the child inherits the cancellation signal, deadlines, and values from its parent
   - Cancelling a parent context cancels all its derived children
   - Cancelling a child context does not affect its parent or siblings

   **Creating Derived Contexts**

   There are four main ways to derive contexts:

   1. **WithCancel**: Create a context that can be cancelled manually
      ```go
      childCtx, cancel := context.WithCancel(parentCtx)
      defer cancel() // Always call cancel to avoid resource leaks
      ```

   2. **WithDeadline**: Create a context that will be cancelled at a specific time
      ```go
      deadline := time.Now().Add(5 * time.Minute)
      childCtx, cancel := context.WithDeadline(parentCtx, deadline)
      defer cancel() // Always call cancel to free resources early if completed before deadline
      ```

   3. **WithTimeout**: Create a context that will be cancelled after a duration
      ```go
      childCtx, cancel := context.WithTimeout(parentCtx, 30 * time.Second)
      defer cancel() // Always call cancel, even if timeout occurs
      ```

   4. **WithValue**: Create a context with a key-value pair
      ```go
      childCtx := context.WithValue(parentCtx, userIDKey, "user-123")
      // No cancel function, as WithValue doesn't set up a cancellation mechanism
      ```

   **Proper Cancellation Patterns**

   **1. Always defer cancel() immediately after creating a cancellable context**

   ```go
   func processRequest(ctx context.Context, req Request) (*Response, error) {
       // Create a timeout for this operation
       ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
       defer cancel() // <-- Always defer immediately after creation
       
       // Rest of function...
   }
   ```

   **2. Cancel explicitly when operation completes early**

   ```go
   func searchWithEarlyTermination(ctx context.Context, query string) ([]Result, error) {
       ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
       defer cancel() // Ensures cleanup even in case of errors
       
       results := make([]Result, 0)
       
       for _, source := range dataSources {
           result, err := searchSource(ctx, source, query)
           if err != nil {
               continue
           }
           
           results = append(results, result...)
           
           // If we found enough results, cancel early to release resources
           if len(results) >= 10 {
               cancel() // Cancel explicitly for early termination
               break
           }
       }
       
       return results, nil
   }
   ```

   **3. Handling nested cancellation**

   ```go
   func processWithSubtasks(ctx context.Context, task Task) error {
       // Create a context for the entire task
       taskCtx, taskCancel := context.WithTimeout(ctx, 30*time.Second)
       defer taskCancel()
       
       // Process each subtask with its own shorter timeout
       for _, subtask := range task.Subtasks {
           // Create a context specifically for this subtask
           subtaskCtx, subtaskCancel := context.WithTimeout(taskCtx, 5*time.Second)
           
           // Process the subtask
           err := processSubtask(subtaskCtx, subtask)
           
           // Always cancel the subtask context when done
           subtaskCancel()
           
           if err != nil {
               return err
           }
           
           // Check if parent context was cancelled
           if taskCtx.Err() != nil {
               return taskCtx.Err()
           }
       }
       
       return nil
   }
   ```

   **4. Controlling cancellation propagation**

   Sometimes you want to prevent cancellation from propagating to certain operations, particularly cleanup:

   ```go
   func processWithCleanup(ctx context.Context, resource Resource) error {
       // Use original context for the main operation
       err := performOperation(ctx, resource)
       
       // Use a fresh context for cleanup to ensure it runs even if ctx is cancelled
       cleanupCtx := context.Background()
       timeoutCtx, cancel := context.WithTimeout(cleanupCtx, 5*time.Second)
       defer cancel()
       
       // This cleanup will run regardless of whether the original context was cancelled
       cleanupErr := resource.Cleanup(timeoutCtx)
       
       // Return the original error if there was one
       if err != nil {
           return err
       }
       
       return cleanupErr
   }
   ```

   **5. Checking cancellation properly**

   ```go
   func processItems(ctx context.Context, items []Item) error {
       for i, item := range items {
           // Check for cancellation before each iteration
           if err := ctx.Err(); err != nil {
               return fmt.Errorf("processing cancelled at item %d: %w", i, err)
           }
           
           if err := processItem(ctx, item); err != nil {
               return err
           }
       }
       return nil
   }
   ```

   **6. Select with context.Done() for cancellation**

   ```go
   func worker(ctx context.Context) {
       for {
           select {
           case <-ctx.Done():
               // Context cancelled, exit the goroutine
               log.Printf("Worker exiting: %v", ctx.Err())
               return
               
           case task := <-taskCh:
               processTask(ctx, task)
               
           case <-time.After(idleTimeout):
               // Idle timeout reached
               log.Println("Worker idle timeout reached")
               return
           }
       }
   }
   ```

   **7. Handling multiple contexts**

   Sometimes you need to respect multiple cancellation sources:

   ```go
   // Function to create a context that's cancelled when either input context is cancelled
   func mergeContexts(ctx1, ctx2 context.Context) (context.Context, context.CancelFunc) {
       ctx, cancel := context.WithCancel(context.Background())
       
       go func() {
           select {
           case <-ctx1.Done():
               cancel()
           case <-ctx2.Done():
               cancel()
           case <-ctx.Done():
               // Our context was directly cancelled
           }
       }()
       
       return ctx, cancel
   }
   
   // Usage
   func processWithMultipleDeadlines(requestCtx, systemCtx context.Context, data Data) error {
       // Create a context that's cancelled if either input context is cancelled
       mergedCtx, cancel := mergeContexts(requestCtx, systemCtx)
       defer cancel()
       
       return process(mergedCtx, data)
   }
   ```

   **8. Handling timeouts and cancellation separately**

   ```go
   func operationWithDistinctCancellation(ctx context.Context) error {
       // Set up a timeout for this operation
       timeoutCtx, timeoutCancel := context.WithTimeout(ctx, 5*time.Second)
       defer timeoutCancel()
       
       // Create a channel for operation completion
       done := make(chan struct{})
       var opErr error
       
       // Run the operation in a goroutine
       go func() {
           defer close(done)
           opErr = performActualOperation(timeoutCtx)
       }()
       
       // Wait for completion, timeout, or cancellation
       select {
       case <-done:
           // Operation completed
           return opErr
       case <-timeoutCtx.Done():
           // Check if it was our timeout or parent cancellation
           if ctx.Err() != nil {
               return fmt.Errorf("operation cancelled by caller: %w", ctx.Err())
           }
           return fmt.Errorf("operation timed out after 5 seconds")
       }
   }
   ```

   **9. Graceful shutdown with context**

   ```go
   func main() {
       // Create a background context
       ctx, cancel := context.WithCancel(context.Background())
       
       // Start services
       go startService1(ctx)
       go startService2(ctx)
       
       // Wait for termination signal
       sigCh := make(chan os.Signal, 1)
       signal.Notify(sigCh, syscall.SIGINT, syscall.SIGTERM)
       <-sigCh
       
       // Signal services to shut down
       log.Println("Shutdown signal received, initiating graceful shutdown")
       cancel() // This will propagate cancellation to all services
       
       // Allow time for graceful shutdown
       time.Sleep(5 * time.Second)
       log.Println("Shutdown complete")
   }
   ```

   **10. Testing cancellation behavior**

   ```go
   func TestCancellation(t *testing.T) {
       ctx, cancel := context.WithCancel(context.Background())
       
       // Start a function that respects cancellation
       done := make(chan struct{})
       go func() {
           defer close(done)
           err := functionUnderTest(ctx)
           if err != ctx.Err() {
               t.Errorf("Expected context error, got: %v", err)
           }
       }()
       
       // Cancel after a short time
       time.Sleep(100 * time.Millisecond)
       cancel()
       
       // Check if function exited correctly
       select {
       case <-done:
           // Function exited as expected
       case <-time.After(time.Second):
           t.Fatal("Function did not respect cancellation")
       }
   }
   ```

   **Key Principles for Context Cancellation:**

   1. **Call `cancel()` when you're done**, even if the context deadline has passed
   2. **Defer cancel immediately after creation** to avoid resource leaks
   3. **Check ctx.Err() periodically** in long-running operations
   4. **Respect cancellation by returning quickly** when context is done
   5. **Propagate context.Err()** when returning from a cancelled operation
   6. **Cancel explicitly for early termination** when appropriate
   7. **Create separate contexts for unrelated operations** to prevent unwanted cancellation propagation
   8. **Use Background() for operations that should run to completion** regardless of caller cancellation

   Following these patterns will ensure that your application properly handles cancellation signals, allowing for resource cleanup and graceful shutdown in concurrent operations.

### 11. Rate Limiting

**Concise Explanation:**
Rate limiting is a technique to control the frequency of operations, such as API calls, resource access, or processing operations. In Go's concurrent programs, rate limiting helps manage load, prevent resource exhaustion, and comply with external API limits. Go provides several patterns for implementing rate limiting, including token bucket, leaky bucket, and time window counters, all of which can be implemented using goroutines, channels, and time management.

**Where to Use:**
- When calling external APIs with usage limits
- To prevent overwhelming downstream services
- For controlling resource usage in high-concurrency environments
- To manage database query rates
- When implementing public-facing APIs
- To protect against denial of service attacks
- For fair usage enforcement across multiple users
- When processing high-volume data streams

**Code Snippet:**
```go
package main

import (
    "context"
    "fmt"
    "log"
    "sync"
    "time"
)

// RateLimiter provides rate limiting capabilities
type RateLimiter interface {
    Wait(ctx context.Context) error
    TryAcquire() bool
}

// TokenBucket implements a token bucket rate limiter
type TokenBucket struct {
    tokens        chan struct{}
    refillRate    time.Duration
    bucketSize    int
    mu            sync.Mutex
    lastRefillTime time.Time
}

// NewTokenBucket creates a new token bucket rate limiter
func NewTokenBucket(rate int, per time.Duration, bucketSize int) *TokenBucket {
    // Calculate token refill interval
    refillInterval := per / time.Duration(rate)
    
    // Create token bucket
    tb := &TokenBucket{
        tokens:        make(chan struct{}, bucketSize),
        refillRate:    refillInterval,
        bucketSize:    bucketSize,
        lastRefillTime: time.Now(),
    }
    
    // Initial fill
    for i := 0; i < bucketSize; i++ {
        tb.tokens <- struct{}{}
    }
    
    // Start refill goroutine
    go tb.refill()
    
    return tb
}

// refill periodically adds tokens to the bucket
func (tb *TokenBucket) refill() {
    ticker := time.NewTicker(tb.refillRate)
    defer ticker.Stop()
    
    for range ticker.C {
        tb.mu.Lock()
        // Add token if bucket is not full
        select {
        case tb.tokens <- struct{}{}:
            // Token added
        default:
            // Bucket is full
        }
        tb.mu.Unlock()
    }
}

// Wait blocks until a token is available or the context is cancelled
func (tb *TokenBucket) Wait(ctx context.Context) error {
    select {
    case <-tb.tokens:
        // Token acquired
        return nil
    case <-ctx.Done():
        // Context cancelled
        return ctx.Err()
    }
}

// TryAcquire tries to acquire a token without blocking
func (tb *TokenBucket) TryAcquire() bool {
    select {
    case <-tb.tokens:
        return true
    default:
        return false
    }
}

// Example usage
func main() {
    // Create a rate limiter allowing 5 operations per second
    limiter := NewTokenBucket(5, time.Second, 5)
    
    // Create a wait group to wait for all operations
    var wg sync.WaitGroup
    
    // Launch 20 operations
    for i := 0; i < 20; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            // Create context with timeout
            ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
            defer cancel()
            
            // Wait for rate limiter
            start := time.Now()
            if err := limiter.Wait(ctx); err != nil {
                log.Printf("Operation %d cancelled: %v", id, err)
                return
            }
            
            // Perform the operation
            elapsed := time.Since(start)
            log.Printf("Operation %d started after waiting %v", id, elapsed)
            
            // Simulate work
            time.Sleep(100 * time.Millisecond)
            log.Printf("Operation %d completed", id)
        }(i)
    }
    
    // Wait for all operations to complete
    wg.Wait()
    
    // Example with non-blocking acquisition
    for i := 0; i < 10; i++ {
        if limiter.TryAcquire() {
            fmt.Printf("Acquired token immediately for operation %d\n", i)
        } else {
            fmt.Printf("Could not acquire token for operation %d\n", i)
        }
    }
}
```

**Real-World Example:**
A rate-limited API client for interacting with an external service:

```go
package apiclient

import (
    "context"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "sync"
    "time"
)

// APIClient provides access to a remote API with rate limiting
type APIClient struct {
    client      *http.Client
    baseURL     string
    rateLimiter *RateLimiter
    authToken   string
}

// RateLimiter manages rate limits for different endpoints
type RateLimiter struct {
    limits     map[string]*TokenBucket
    globalRate *TokenBucket
    mu         sync.RWMutex
}

// NewRateLimiter creates a new rate limiter
func NewRateLimiter(globalRate int) *RateLimiter {
    return &RateLimiter{
        limits:     make(map[string]*TokenBucket),
        globalRate: NewTokenBucket(globalRate, time.Second, globalRate),
    }
}

// TokenBucket implements a token bucket algorithm
type TokenBucket struct {
    tokens     chan struct{}
    ticker     *time.Ticker
    quit       chan struct{}
    bucketSize int
    mu         sync.Mutex
}

// NewTokenBucket creates a token bucket that refills at the specified rate
func NewTokenBucket(rate int, per time.Duration, bucketSize int) *TokenBucket {
    refillInterval := per / time.Duration(rate)
    tb := &TokenBucket{
        tokens:     make(chan struct{}, bucketSize),
        ticker:     time.NewTicker(refillInterval),
        quit:       make(chan struct{}),
        bucketSize: bucketSize,
    }
    
    // Initial fill
    for i := 0; i < bucketSize; i++ {
        tb.tokens <- struct{}{}
    }
    
    // Start refill process
    go func() {
        for {
            select {
            case <-tb.ticker.C:
                tb.mu.Lock()
                select {
                case tb.tokens <- struct{}{}:
                    // Token added
                default:
                    // Bucket is full
                }
                tb.mu.Unlock()
            case <-tb.quit:
                tb.ticker.Stop()
                return
            }
        }
    }()
    
    return tb
}

// Wait blocks until a token is available or context is cancelled
func (tb *TokenBucket) Wait(ctx context.Context) error {
    select {
    case <-tb.tokens:
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}

// Close stops the token bucket
func (tb *TokenBucket) Close() {
    close(tb.quit)
}

// SetEndpointLimit sets a rate limit for a specific endpoint
func (rl *RateLimiter) SetEndpointLimit(endpoint string, rate int) {
    rl.mu.Lock()
    defer rl.mu.Unlock()
    
    // Close existing limiter if present
    if existing, ok := rl.limits[endpoint]; ok {
        existing.Close()
    }
    
    rl.limits[endpoint] = NewTokenBucket(rate, time.Second, rate)
}

// Wait respects both global and endpoint-specific rate limits
func (rl *RateLimiter) Wait(ctx context.Context, endpoint string) error {
    // First wait for global limit
    if err := rl.globalRate.Wait(ctx); err != nil {
        return err
    }
    
    // Then check for endpoint-specific limit
    rl.mu.RLock()
    limiter, exists := rl.limits[endpoint]
    rl.mu.RUnlock()
    
    if exists {
        return limiter.Wait(ctx)
    }
    
    return nil
}

// NewAPIClient creates a new API client with rate limiting
func NewAPIClient(baseURL string, globalRateLimit int) *APIClient {
    return &APIClient{
        client:      &http.Client{Timeout: 10 * time.Second},
        baseURL:     baseURL,
        rateLimiter: NewRateLimiter(globalRateLimit),
    }
}

// SetAuthToken sets the authentication token for API requests
func (c *APIClient) SetAuthToken(token string) {
    c.authToken = token
}

// SetEndpointLimit sets a rate limit for a specific API endpoint
func (c *APIClient) SetEndpointLimit(endpoint string, ratePerSecond int) {
    c.rateLimiter.SetEndpointLimit(endpoint, ratePerSecond)
}

// Get performs a rate-limited GET request
func (c *APIClient) Get(ctx context.Context, endpoint string, result interface{}) error {
    // Wait for rate limiter
    if err := c.rateLimiter.Wait(ctx, endpoint); err != nil {
        return fmt.Errorf("rate limiting: %w", err)
    }
    
    // Create request
    url := c.baseURL + endpoint
    req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
    if err != nil {
        return fmt.Errorf("creating request: %w", err)
    }
    
    // Add headers
    if c.authToken != "" {
        req.Header.Set("Authorization", "Bearer "+c.authToken)
    }
    req.Header.Set("Accept", "application/json")
    
    // Perform request
    resp, err := c.client.Do(req)
    if err != nil {
        return fmt.Errorf("executing request: %w", err)
    }
    defer resp.Body.Close()
    
    // Check status code
    if resp.StatusCode < 200 || resp.StatusCode >= 300 {
        body, _ := io.ReadAll(resp.Body)
        return fmt.Errorf("API error %d: %s", resp.StatusCode, body)
    }
    
    // Parse response
    if err := json.NewDecoder(resp.Body).Decode(result); err != nil {
        return fmt.Errorf("parsing response: %w", err)
    }
    
    return nil
}

// Example usage
func Example() {
    // Create client with global rate limit of 10 requests/second
    client := NewAPIClient("https://api.example.com", 10)
    
    // Set endpoint-specific limits
    client.SetEndpointLimit("/users", 5)    // 5 requests/second
    client.SetEndpointLimit("/products", 2) // 2 requests/second
    
    // Set authentication
    client.SetAuthToken("my-api-token")
    
    // Make concurrent API requests
    var wg sync.WaitGroup
    ctx := context.Background()
    
    // Function to fetch user data
    fetchUser := func(id int) {
        defer wg.Done()
        
        var user struct {
            ID   int    `json:"id"`
            Name string `json:"name"`
        }
        
        endpoint := fmt.Sprintf("/users/%d", id)
        err := client.Get(ctx, endpoint, &user)
        if err != nil {
            fmt.Printf("Error fetching user %d: %v\n", id, err)
            return
        }
        
        fmt.Printf("User %d: %s\n", user.ID, user.Name)
    }
    
    // Start concurrent requests
    for i := 1; i <= 20; i++ {
        wg.Add(1)
        go fetchUser(i)
    }
    
    // Wait for all requests to complete
    wg.Wait()
}
```

**Common Pitfalls:**
- Using static time delays instead of proper rate limiting
- Creating a new limiter for each request, defeating the purpose
- Not accounting for rate limits across multiple instances of an application
- Forgetting to add timeouts, leading to blocked operations if rate limits are too restrictive
- Neglecting to handle error cases when operations are rejected due to rate limits
- Not considering different rate limits for different operations or users
- Using overly complex rate limiting algorithms when simpler ones would suffice
- Allocating rate limit tokens too slowly, causing excessive delays

**Confusion Questions:**

1. **Q: What's the difference between token bucket, leaky bucket, and fixed window rate limiting?**

   A: Token bucket, leaky bucket, and fixed window are different rate limiting algorithms with distinct characteristics:

   **Token Bucket:**
   
   ```go
   // Token Bucket implementation
   type TokenBucket struct {
       tokens     chan struct{}
       capacity   int
       refillRate time.Duration
   }
   ```
   
   - **How it works**: Maintains a bucket of tokens that refills at a steady rate. Each operation consumes a token.
   - **Characteristics**:
     - Allows bursts up to the bucket capacity
     - Smooths out long-term rate while permitting short bursts
     - Tokens accumulate when not used (up to capacity)
   - **Best for**: API clients with occasional bursts of activity
   
   **Leaky Bucket:**
   
   ```go
   // Leaky Bucket implementation
   type LeakyBucket struct {
       queue      chan struct{}
       processRate time.Duration
       quit       chan struct{}
   }
   ```
   
   - **How it works**: Requests enter a queue with a fixed size, and are processed at a constant rate
   - **Characteristics**:
     - Enforces a constant outflow rate
     - Buffers requests up to queue capacity
     - No accumulation of unused capacity
     - Excess requests are dropped when the bucket (queue) is full
   - **Best for**: Traffic shaping and ensuring steady processing rates
   
   **Fixed Window:**
   
   ```go
   // Fixed Window implementation
   type FixedWindow struct {
       mu         sync.Mutex
       count      int
       limit      int
       windowSize time.Duration
       lastReset  time.Time
   }
   ```
   
   - **How it works**: Counts events in fixed time windows (e.g., per minute)
   - **Characteristics**:
     - Simplest to implement
     - Resets counter at the end of each window
     - Can allow twice the rate at window boundaries (edge condition)
     - No memory between windows
   - **Best for**: Simple rate limiting scenarios with defined time periods
   
   **Sliding Window:**
   
   ```go
   // Sliding Window implementation
   type SlidingWindow struct {
       mu         sync.Mutex
       events     []time.Time
       limit      int
       windowSize time.Duration
   }
   ```
   
   - **How it works**: Tracks timestamps of events within a moving time window
   - **Characteristics**:
     - More accurate than fixed window
     - Prevents boundary spike issues
     - Higher memory usage as it tracks individual events
     - Smoothly handles transitions between windows
   - **Best for**: More precise rate limiting with gradual window transitions

   **Practical decision guide:**

   - For client-side API rate limiting where bursts are acceptable: **Token Bucket**
   - For steady, predictable throughput with queue behavior: **Leaky Bucket**
   - For server-side rate limiting with minimal overhead: **Fixed Window**
   - For precise rate limiting without boundary issues: **Sliding Window**

   The token bucket approach is often the most versatile and is widely used in Go applications because it balances good properties of both allowing short bursts while maintaining long-term rate constraints.

2. **Q: How do I implement per-user or distributed rate limiting?**

   A: Implementing per-user or distributed rate limiting requires additional techniques beyond a simple rate limiter. Here are approaches for both scenarios:

   **Per-User Rate Limiting:**

   For per-user rate limiting, create and manage separate rate limiters for each user:

   ```go
   type UserRateLimiter struct {
       limiters   map[string]*TokenBucket // Key is user identifier
       mu         sync.RWMutex
       rate       int           // Default tokens per interval
       interval   time.Duration // Default refill interval
       bucketSize int           // Default bucket size
   }

   func NewUserRateLimiter(rate int, interval time.Duration, bucketSize int) *UserRateLimiter {
       return &UserRateLimiter{
           limiters:   make(map[string]*TokenBucket),
           rate:       rate,
           interval:   interval,
           bucketSize: bucketSize,
       }
   }

   // GetLimiter returns the rate limiter for a specific user
   func (u *UserRateLimiter) GetLimiter(userID string) *TokenBucket {
       u.mu.RLock()
       limiter, exists := u.limiters[userID]
       u.mu.RUnlock()

       if exists {
           return limiter
       }

       // Create new limiter for user with default settings
       u.mu.Lock()
       defer u.mu.Unlock()
       
       // Double-check after acquiring write lock
       if limiter, exists = u.limiters[userID]; exists {
           return limiter
       }

       // Create a new limiter
       limiter = NewTokenBucket(u.rate, u.interval, u.bucketSize)
       u.limiters[userID] = limiter
       return limiter
   }

   // SetUserLimit sets custom limits for a specific user
   func (u *UserRateLimiter) SetUserLimit(userID string, rate int, bucketSize int) {
       u.mu.Lock()
       defer u.mu.Unlock()

       // Close existing limiter if present
       if existingLimiter, ok := u.limiters[userID]; ok {
           existingLimiter.Close()
       }

       // Create new limiter with custom settings
       u.limiters[userID] = NewTokenBucket(rate, u.interval, bucketSize)
   }

   // Wait waits for a token for the specified user
   func (u *UserRateLimiter) Wait(ctx context.Context, userID string) error {
       limiter := u.GetLimiter(userID)
       return limiter.Wait(ctx)
   }
   ```

   **Distributed Rate Limiting:**

   For distributed rate limiting across multiple application instances, you need a shared storage mechanism like Redis:

   ```go
   type DistributedRateLimiter struct {
       redisClient *redis.Client
       keyPrefix   string
       window      time.Duration
       limit       int
   }

   func NewDistributedRateLimiter(redisAddr, keyPrefix string, limit int, window time.Duration) (*DistributedRateLimiter, error) {
       client := redis.NewClient(&redis.Options{
           Addr: redisAddr,
       })

       // Check connection
       if _, err := client.Ping().Result(); err != nil {
           return nil, fmt.Errorf("redis connection error: %w", err)
       }

       return &DistributedRateLimiter{
           redisClient: client,
           keyPrefix:   keyPrefix,
           window:      window,
           limit:       limit,
       }, nil
   }

   // Allow checks if a request is allowed and increments counter if so
   func (d *DistributedRateLimiter) Allow(ctx context.Context, key string) (bool, error) {
       // Create a full key with prefix
       fullKey := fmt.Sprintf("%s:%s", d.keyPrefix, key)
       
       // Execute Redis script for atomic increment and check
       // This Lua script:
       // 1. Increments the counter for the key
       // 2. Sets expiry time if it's a new key
       // 3. Returns the current count
       script := redis.NewScript(`
           local current = redis.call("INCR", KEYS[1])
           if current == 1 then
               redis.call("EXPIRE", KEYS[1], ARGV[1])
           end
           return current
       `)

       // Convert window duration to seconds for Redis expiry
       windowSeconds := int(d.window.Seconds())
       
       // Run the script
       result, err := script.Run(ctx, d.redisClient, []string{fullKey}, windowSeconds).Int()
       if err != nil {
           return false, fmt.Errorf("redis script error: %w", err)
       }

       // Check if we're under the limit
       return result <= d.limit, nil
   }

   // Wait blocks until a request is allowed or context is cancelled
   func (d *DistributedRateLimiter) Wait(ctx context.Context, key string) error {
       for {
           allowed, err := d.Allow(ctx, key)
           if err != nil {
               return err
           }
           
           if allowed {
               return nil
           }

           // Not allowed, wait and retry
           select {
           case <-time.After(100 * time.Millisecond):
               // Continue and retry
           case <-ctx.Done():
               return ctx.Err()
           }
       }
   }
   ```

   **Combining both approaches:**

   For a comprehensive rate limiting strategy in a distributed system with per-user limits:

   ```go
   type CompleteRateLimiter struct {
       redis         *redis.Client
       globalLimiter *DistributedRateLimiter
       userLimiters  map[string]*DistributedRateLimiter
       mu            sync.RWMutex
   }

   func NewCompleteRateLimiter(redisAddr string) (*CompleteRateLimiter, error) {
       client := redis.NewClient(&redis.Options{Addr: redisAddr})
       
       // Create global limiter
       globalLimiter, err := NewDistributedRateLimiter(
           redisAddr,
           "global",
           1000,           // 1000 requests per minute globally
           time.Minute,
       )
       if err != nil {
           return nil, err
       }
       
       return &CompleteRateLimiter{
           redis:         client,
           globalLimiter: globalLimiter,
           userLimiters:  make(map[string]*DistributedRateLimiter),
       }, nil
   }

   // GetUserLimiter returns or creates a rate limiter for a user
   func (c *CompleteRateLimiter) GetUserLimiter(userID string) (*DistributedRateLimiter, error) {
       c.mu.RLock()
       limiter, exists := c.userLimiters[userID]
       c.mu.RUnlock()
       
       if exists {
           return limiter, nil
       }
       
       // Create new limiter with default user settings
       c.mu.Lock()
       defer c.mu.Unlock()
       
       // Double-check after acquiring write lock
       if limiter, exists = c.userLimiters[userID]; exists {
           return limiter, nil
       }
       
       var err error
       limiter, err = NewDistributedRateLimiter(
           c.redis.Options().Addr,
           fmt.Sprintf("user:%s", userID),
           60,            // Default 60 requests per minute per user
           time.Minute,
       )
       if err != nil {
           return nil, err
       }
       
       c.userLimiters[userID] = limiter
       return limiter, nil
   }

   // Wait respects both global and per-user rate limits
   func (c *CompleteRateLimiter) Wait(ctx context.Context, userID string) error {
       // First check global limit
       if err := c.globalLimiter.Wait(ctx, "requests"); err != nil {
           return err
       }
       
       // Then check user-specific limit
       userLimiter, err := c.GetUserLimiter(userID)
       if err != nil {
           return err
       }
       
       return userLimiter.Wait(ctx, "requests")
   }
   ```

   **Key principles for distributed/per-user rate limiting:**

   1. **Use centralized storage** (like Redis) for distributed state
   2. **Implement atomic operations** to avoid race conditions in counters
   3. **Add sliding window or decay** for more accurate limits
   4. **Consider performance impact** of network calls for each rate limit check
   5. **Implement fallback mechanisms** in case the rate limiting service fails
   6. **Cache results briefly** when possible to reduce storage load
   7. **Ensure consistent time** across distributed system nodes 
   8. **Provide feedback to users** about their rate limit status (headers, etc.)

   These approaches provide scalable ways to limit API usage per user while maintaining global limits across a distributed system.

3. **Q: How do I handle rate limiting failures gracefully in a client application?**

   A: Handling rate limiting failures gracefully in client applications requires a comprehensive strategy including retries, backoff, circuit breaking, and user feedback. Here's how to implement these techniques:

   **1. Detecting Rate Limit Responses**

   First, properly identify when rate limiting is occurring:

   ```go
   func isRateLimited(resp *http.Response, err error) bool {
       // Check for explicit rate limit status codes
       if resp != nil && (resp.StatusCode == 429 || resp.StatusCode == 503) {
           return true
       }
       
       // Check error types that might indicate rate limiting
       var netErr net.Error
       if errors.As(err, &netErr) && netErr.Timeout() {
           return true
       }
       
       // Check for specific API error responses
       if resp != nil && resp.StatusCode == 200 {
           // Some APIs return 200 with error in body
           var apiResp struct {
               Error struct {
                   Code    string `json:"code"`
                   Message string `json:"message"`
               } `json:"error"`
           }
           
           if err := json.NewDecoder(resp.Body).Decode(&apiResp); err == nil {
               return apiResp.Error.Code == "RATE_LIMITED"
           }
           
           // Reset the body for further processing
           resp.Body.Close()
           // Re-create the body (simplified, would need proper implementation)
       }
       
       return false
   }
   ```

   **2. Exponential Backoff with Jitter**

   Implement smart retry logic that backs off exponentially and adds jitter to prevent thundering herd problems:

   ```go
   func retryWithBackoff(ctx context.Context, fn func() (*http.Response, error)) (*http.Response, error) {
       var resp *http.Response
       var err error
       
       maxRetries := 5
       initialBackoff := 100 * time.Millisecond
       maxBackoff := 10 * time.Second
       
       for attempt := 0; attempt <= maxRetries; attempt++ {
           resp, err = fn()
           
           // If not rate limited or final attempt, return result
           if !isRateLimited(resp, err) || attempt == maxRetries {
               return resp, err
           }
           
           // Calculate backoff duration with exponential increase
           backoff := initialBackoff * time.Duration(1<<attempt) // 2^attempt
           if backoff > maxBackoff {
               backoff = maxBackoff
           }
           
           // Add jitter (random variance) to prevent synchronized retries
           jitter := time.Duration(rand.Int63n(int64(backoff) / 2))
           backoff = backoff/2 + jitter
           
           // Extract retry-after header if present
           if resp != nil && resp.Header.Get("Retry-After") != "" {
               if retryAfter, err := strconv.Atoi(resp.Header.Get("Retry-After")); err == nil {
                   backoff = time.Duration(retryAfter) * time.Second
               }
           }
           
           // Log the retry attempt
           log.Printf("Rate limited. Retrying in %v (attempt %d/%d)", backoff, attempt+1, maxRetries)
           
           // Wait for backoff duration or context cancellation
           select {
           case <-time.After(backoff):
               // Continue with retry
           case <-ctx.Done():
               return nil, ctx.Err()
           }
       }
       
       return resp, err
   }
   ```

   **3. Circuit Breaker Pattern**

   Implement a circuit breaker to prevent repeated failures when an API is consistently rate limiting:

   ```go
   type CircuitBreaker struct {
       mu                  sync.Mutex
       state               string // "closed", "open", "half-open"
       failureThreshold    int
       failureCount        int
       resetTimeout        time.Duration
       lastStateChangeTime time.Time
   }

   func NewCircuitBreaker(failureThreshold int, resetTimeout time.Duration) *CircuitBreaker {
       return &CircuitBreaker{
           state:            "closed",
           failureThreshold: failureThreshold,
           resetTimeout:     resetTimeout,
       }
   }

   // Execute runs the given function if the circuit breaker allows it
   func (cb *CircuitBreaker) Execute(fn func() error) error {
       cb.mu.Lock()
       state := cb.state
       
       // If circuit is open but enough time has passed, try half-open
       if state == "open" && time.Since(cb.lastStateChangeTime) > cb.resetTimeout {
           state = "half-open"
           cb.state = "half-open"
           cb.lastStateChangeTime = time.Now()
       }
       cb.mu.Unlock()
       
       if state == "open" {
           return fmt.Errorf("circuit breaker is open")
       }
       
       // Execute the function
       err := fn()
       
       cb.mu.Lock()
       defer cb.mu.Unlock()
       
       if err != nil && isRateLimited(nil, err) {
           // Function failed with rate limiting
           if state == "half-open" {
               // Return to open state on failure during half-open
               cb.state = "open"
               cb.lastStateChangeTime = time.Now()
           } else {
               // Increment failure counter in closed state
               cb.failureCount++
               if cb.failureCount >= cb.failureThreshold {
                   cb.state = "open"
                   cb.lastStateChangeTime = time.Now()
               }
           }
           return err
       }
       
       // Success, reset circuit breaker
       if state == "half-open" {
           cb.state = "closed"
           cb.lastStateChangeTime = time.Now()
       }
       cb.failureCount = 0
       
       return err
   }
   ```

   **4. Request Queuing/Batching**

   Queue requests during rate limited periods and process them when allowed:

   ```go
   type RequestQueue struct {
       queue      chan Request
       processing bool
       rateLimiter *TokenBucket
       mu         sync.Mutex
   }

   type Request struct {
       Fn       func() error
       ResultCh chan error
   }

   func NewRequestQueue(rateLimiter *TokenBucket) *RequestQueue {
       rq := &RequestQueue{
           queue:       make(chan Request, 100), // Queue size
           rateLimiter: rateLimiter,
       }
       
       go rq.processor()
       return rq
   }

   func (rq *RequestQueue) processor() {
       for req := range rq.queue {
           // Wait for rate limiter with reasonable timeout
           ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
           err := rq.rateLimiter.Wait(ctx)
           cancel()
           
           if err != nil {
               req.ResultCh <- fmt.Errorf("rate limiting timeout: %w", err)
               continue
           }
           
           // Execute the request
           err = req.Fn()
           req.ResultCh <- err
       }
   }

   // Enqueue adds a request to the queue
   func (rq *RequestQueue) Enqueue(fn func() error) chan error {
       resultCh := make(chan error, 1)
       
       // Try to add to queue, non-blocking
       select {
       case rq.queue <- Request{Fn: fn, ResultCh: resultCh}:
           // Added to queue
       default:
           // Queue is full
           resultCh <- errors.New("request queue is full")
       }
       
       return resultCh
   }
   ```

   **5. Complete API Client with Rate Limit Handling**

   A comprehensive client that combines all these techniques:

   ```go
   type ResilientClient struct {
       client       *http.Client
       baseURL      string
       rateLimiter  *TokenBucket
       circuitBreaker *CircuitBreaker
       requestQueue *RequestQueue
   }

   func NewResilientClient(baseURL string, ratePerSecond int) *ResilientClient {
       limiter := NewTokenBucket(ratePerSecond, time.Second, ratePerSecond)
       
       rc := &ResilientClient{
           client:        &http.Client{Timeout: 10 * time.Second},
           baseURL:       baseURL,
           rateLimiter:   limiter,
           circuitBreaker: NewCircuitBreaker(5, 30*time.Second),
       }
       
       rc.requestQueue = NewRequestQueue(limiter)
       return rc
   }

   // Get performs a GET request with full resiliency features
   func (rc *ResilientClient) Get(ctx context.Context, path string, result interface{}) error {
       // Create the request function
       requestFn := func() error {
           return rc.circuitBreaker.Execute(func() error {
               var resp *http.Response
               var err error
               
               // Use retryWithBackoff
               resp, err = retryWithBackoff(ctx, func() (*http.Response, error) {
                   // Create request
                   req, err := http.NewRequestWithContext(ctx, "GET", rc.baseURL+path, nil)
                   if err != nil {
                       return nil, err
                   }
                   
                   // Execute request
                   return rc.client.Do(req)
               })
               
               if err != nil {
                   return err
               }
               defer resp.Body.Close()
               
               // Check status
               if resp.StatusCode >= 400 {
                   body, _ := io.ReadAll(resp.Body)
                   return fmt.Errorf("API error %d: %s", resp.StatusCode, body)
               }
               
               // Parse response
               return json.NewDecoder(resp.Body).Decode(result)
           })
       }
       
       // Choose execution strategy based on urgency
       urgent := ctx.Value("urgent") != nil
       
       if urgent {
           // For urgent requests, try to execute immediately
           if rc.rateLimiter.TryAcquire() {
               return requestFn()
           }
       }
       
       // Queue the request
       resultCh := rc.requestQueue.Enqueue(requestFn)
       
       // Wait for result or context cancellation
       select {
       case err := <-resultCh:
           return err
       case <-ctx.Done():
           return ctx.Err()
       }
   }
   ```

   **Key techniques for graceful rate limit handling:**

   1. **Proper identification** of rate limit responses
   2. **Smart retry logic** with exponential backoff and jitter
   3. **Circuit breaking** to prevent cascading failures
   4. **Request queuing** to smooth out traffic spikes
   5. **Priority handling** for different request types
   6. **Resource pooling** to maximize utilization within limits
   7. **Monitoring and alerting** for persistent rate limit issues
   8. **User feedback** about rate limiting status

   By implementing these techniques, your client application can handle rate limiting gracefully, providing the best possible experience while respecting API limits.

### 12. Generators and Iterators

**Concise Explanation:**
Generators and iterators in Go are patterns for lazily producing sequences of values. Unlike languages with native generator functions, Go implements this pattern using goroutines and channels to create pipelines that produce values on demand. This approach allows for efficient memory usage when processing large data sets, as values are generated only when needed rather than all at once. Generators produce values and send them to a channel, while iterators consume these values, creating a clean separation between data production and consumption.

**Where to Use:**
- For processing large data sets without loading everything into memory
- When generating infinite sequences or streams of data
- For implementing lazy evaluation of computation-heavy operations
- In data processing pipelines that transform sequences step by step
- When reading records from databases or files that are too large for memory
- For implementing custom iteration patterns over complex data structures
- When simulating streams of events (like user actions or sensor readings)
- For testing systems with sequences of generated input values

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

// Generator creates a channel that produces values
func Generator(ctx context.Context, values ...int) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        
        for _, v := range values {
            select {
            case <-ctx.Done():
                return
            case out <- v:
                // Value sent
            }
        }
    }()
    
    return out
}

// RangeGenerator creates a channel that produces a range of integers
func RangeGenerator(ctx context.Context, start, end int) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        
        for i := start; i < end; i++ {
            select {
            case <-ctx.Done():
                return
            case out <- i:
                // Value sent
            }
        }
    }()
    
    return out
}

// InfiniteGenerator creates a channel that produces an infinite sequence
func InfiniteGenerator(ctx context.Context) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        
        i := 0
        for {
            select {
            case <-ctx.Done():
                return
            case out <- i:
                i++
            }
        }
    }()
    
    return out
}

// RandomGenerator creates a channel that produces random integers
func RandomGenerator(ctx context.Context, min, max int) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        
        for {
            select {
            case <-ctx.Done():
                return
            case out <- min + rand.Intn(max-min+1):
                // Random value sent
            }
        }
    }()
    
    return out
}

// Take creates a channel that takes n items from a source channel
func Take(ctx context.Context, src <-chan int, n int) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        
        for i := 0; i < n; i++ {
            select {
            case <-ctx.Done():
                return
            case v, ok := <-src:
                if !ok {
                    return // Source channel is closed
                }
                out <- v
            }
        }
    }()
    
    return out
}

// Map applies a function to each value from a source channel
func Map(ctx context.Context, src <-chan int, fn func(int) int) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        
        for v := range src {
            select {
            case <-ctx.Done():
                return
            case out <- fn(v):
                // Transformed value sent
            }
        }
    }()
    
    return out
}

// Filter creates a channel with values that satisfy a predicate
func Filter(ctx context.Context, src <-chan int, fn func(int) bool) <-chan int {
    out := make(chan int)
    
    go func() {
        defer close(out)
        
        for v := range src {
            if fn(v) {
                select {
                case <-ctx.Done():
                    return
                case out <- v:
                    // Filtered value sent
                }
            }
        }
    }()
    
    return out
}

// Reduce applies a function cumulatively to all values from a source
func Reduce(ctx context.Context, src <-chan int, initial int, fn func(int, int) int) <-chan int {
    out := make(chan int, 1) // Buffer to ensure value is sent even if receiver is slow
    
    go func() {
        defer close(out)
        
        result := initial
        for v := range src {
            result = fn(result, v)
        }
        
        select {
        case <-ctx.Done():
            return
        case out <- result:
            // Final result sent
        }
    }()
    
    return out
}

// Iterator wraps a channel to provide Next/Value operations
type Iterator struct {
    ch      <-chan int
    current int
    valid   bool
    mu      sync.RWMutex
}

// NewIterator creates an iterator from a channel
func NewIterator(ch <-chan int) *Iterator {
    return &Iterator{
        ch:    ch,
        valid: false,
    }
}

// Next advances the iterator to the next value
func (it *Iterator) Next() bool {
    it.mu.Lock()
    defer it.mu.Unlock()
    
    value, ok := <-it.ch
    if !ok {
        it.valid = false
        return false
    }
    
    it.current = value
    it.valid = true
    return true
}

// Value returns the current value
func (it *Iterator) Value() (int, bool) {
    it.mu.RLock()
    defer it.mu.RUnlock()
    
    return it.current, it.valid
}

func main() {
    // Create a context for cancellation
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    // Example 1: Simple generator and iteration
    fmt.Println("Example 1: Generator with specific values")
    gen := Generator(ctx, 1, 2, 3, 4, 5)
    
    for v := range gen {
        fmt.Printf("%d ", v)
    }
    fmt.Println()
    
    // Example 2: Range generator
    fmt.Println("\nExample 2: Range generator")
    rangeGen := RangeGenerator(ctx, 10, 15)
    
    for v := range rangeGen {
        fmt.Printf("%d ", v)
    }
    fmt.Println()
    
    // Example 3: Infinite generator with Take
    fmt.Println("\nExample 3: Infinite generator with Take")
    infiniteGen := InfiniteGenerator(ctx)
    limitedGen := Take(ctx, infiniteGen, 5)
    
    for v := range limitedGen {
        fmt.Printf("%d ", v)
    }
    fmt.Println()
    
    // Example 4: Transformation pipeline
    fmt.Println("\nExample 4: Data pipeline with Map and Filter")
    // Generate numbers -> Map (double them) -> Filter (only even) -> Take first 5
    pipeline := Take(
        ctx,
        Filter(
            ctx,
            Map(
                ctx,
                InfiniteGenerator(ctx),
                func(x int) int { return x * 2 },
            ),
            func(x int) bool { return x%4 == 0 },
        ),
        5,
    )
    
    for v := range pipeline {
        fmt.Printf("%d ", v)
    }
    fmt.Println()
    
    // Example 5: Random generator with reducer
    fmt.Println("\nExample 5: Random numbers with sum reducer")
    randGen := RandomGenerator(ctx, 1, 10)
    limitedRand := Take(ctx, randGen, 5)
    
    fmt.Print("Random values: ")
    nums := []int{}
    for v := range limitedRand {
        fmt.Printf("%d ", v)
        nums = append(nums, v)
    }
    fmt.Println()
    
    // Sum them using a new generator and reducer
    sumCh := Reduce(
        ctx,
        Generator(ctx, nums...),
        0,
        func(acc, val int) int { return acc + val },
    )
    
    sum := <-sumCh
    fmt.Printf("Sum: %d\n", sum)
    
    // Example 6: Using iterator interface
    fmt.Println("\nExample 6: Iterator interface")
    rangeGen2 := RangeGenerator(ctx, 100, 105)
    iter := NewIterator(rangeGen2)
    
    for iter.Next() {
        val, _ := iter.Value()
        fmt.Printf("%d ", val)
    }
    fmt.Println()
}
```

**Real-World Example:**
A file processing system that reads and processes large files line by line:

```go
package main

import (
    "bufio"
    "compress/gzip"
    "context"
    "encoding/csv"
    "encoding/json"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "strings"
    "sync"
    "time"
)

// LogEntry represents a parsed log entry
type LogEntry struct {
    Timestamp time.Time          `json:"timestamp"`
    Level     string             `json:"level"`
    Message   string             `json:"message"`
    Metadata  map[string]interface{} `json:"metadata"`
    Raw       string             `json:"raw"`
}

// Stats contains aggregated statistics from logs
type Stats struct {
    TotalEntries int            `json:"total_entries"`
    ErrorCount   int            `json:"error_count"`
    WarnCount    int            `json:"warn_count"`
    InfoCount    int            `json:"info_count"`
    DebugCount   int            `json:"debug_count"`
    TopErrors    []string       `json:"top_errors"`
    TimeRange    [2]time.Time   `json:"time_range"`
}

// FileScanner is a generator that finds files matching a pattern
func FileScanner(ctx context.Context, root string, pattern string) <-chan string {
    out := make(chan string)
    
    go func() {
        defer close(out)
        
        // Walk the directory tree
        err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            
            // Check for cancellation
            select {
            case <-ctx.Done():
                return ctx.Err()
            default:
                // Continue
            }
            
            // Skip directories
            if info.IsDir() {
                return nil
            }
            
            // Check if file matches pattern
            matched, err := filepath.Match(pattern, filepath.Base(path))
            if err != nil {
                return err
            }
            
            if matched {
                // Send matching file path
                select {
                case out <- path:
                    // File path sent
                case <-ctx.Done():
                    return ctx.Err()
                }
            }
            
            return nil
        })
        
        if err != nil && err != ctx.Err() {
            fmt.Fprintf(os.Stderr, "Error scanning files: %v\n", err)
        }
    }()
    
    return out
}

// LineReader is a generator that reads lines from a file
func LineReader(ctx context.Context, filePath string) <-chan string {
    out := make(chan string)
    
    go func() {
        defer close(out)
        
        // Open the file
        file, err := os.Open(filePath)
        if err != nil {
            fmt.Fprintf(os.Stderr, "Error opening %s: %v\n", filePath, err)
            return
        }
        defer file.Close()
        
        // Create appropriate reader based on file extension
        var reader io.Reader = file
        
        // Handle gzip files
        if strings.HasSuffix(filePath, ".gz") {
            gzReader, err := gzip.NewReader(file)
            if err != nil {
                fmt.Fprintf(os.Stderr, "Error creating gzip reader for %s: %v\n", filePath, err)
                return
            }
            defer gzReader.Close()
            reader = gzReader
        }
        
        // Create buffered reader
        scanner := bufio.NewScanner(reader)
        
        // Read line by line
        for scanner.Scan() {
            line := scanner.Text()
            
            // Send the line
            select {
            case out <- line:
                // Line sent
            case <-ctx.Done():
                return
            }
        }
        
        if err := scanner.Err(); err != nil {
            fmt.Fprintf(os.Stderr, "Error reading %s: %v\n", filePath, err)
        }
    }()
    
    return out
}

// CSVReader is a generator that parses CSV files
func CSVReader(ctx context.Context, filePath string, hasHeader bool) <-chan []string {
    out := make(chan []string)
    
    go func() {
        defer close(out)
        
        // Open the file
        file, err := os.Open(filePath)
        if err != nil {
            fmt.Fprintf(os.Stderr, "Error opening %s: %v\n", filePath, err)
            return
        }
        defer file.Close()
        
        // Create CSV reader
        reader := csv.NewReader(file)
        
        // Handle header row
        if hasHeader {
            if _, err := reader.Read(); err != nil {
                fmt.Fprintf(os.Stderr, "Error reading header row: %v\n", err)
                return
            }
        }
        
        for {
            // Read a row
            record, err := reader.Read()
            if err == io.EOF {
                break
            }
            if err != nil {
                fmt.Fprintf(os.Stderr, "Error reading CSV record: %v\n", err)
                continue
            }
            
            // Send the record
            select {
            case out <- record:
                // Record sent
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return out
}

// LogParser transforms log lines into structured entries
func LogParser(ctx context.Context, lines <-chan string) <-chan LogEntry {
    out := make(chan LogEntry)
    
    go func() {
        defer close(out)
        
        for line := range lines {
            // Parse log entry (simplified example)
            entry := parseLogEntry(line)
            
            // Send parsed entry
            select {
            case out <- entry:
                // Entry sent
            case <-ctx.Done():
                return
            }
        }
    }()
    
    return out
}

// LogFilter filters log entries based on level
func LogFilter(ctx context.Context, entries <-chan LogEntry, minLevel string) <-chan LogEntry {
    out := make(chan LogEntry)
    
    // Define log level priorities
    levels := map[string]int{
        "DEBUG": 0,
        "INFO":  1,
        "WARN":  2,
        "ERROR": 3,
        "FATAL": 4,
    }
    
    minLevelVal := levels[strings.ToUpper(minLevel)]
    
    go func() {
        defer close(out)
        
        for entry := range entries {
            // Check if entry meets minimum level
            entryLevel := strings.ToUpper(entry.Level)
            if levels[entryLevel] >= minLevelVal {
                // Send filtered entry
                select {
                case out <- entry:
                    // Entry sent
                case <-ctx.Done():
                    return
                }
            }
        }
    }()
    
    return out
}

// StatAggregator collects statistics from log entries
func StatAggregator(ctx context.Context, entries <-chan LogEntry) <-chan Stats {
    out := make(chan Stats, 1) // Buffer size 1 to ensure stats are sent even if receiver is slow
    
    go func() {
        defer close(out)
        
        stats := Stats{
            TimeRange: [2]time.Time{time.Now(), time.Time{}},
        }
        
        errorMap := make(map[string]int)
        
        for entry := range entries {
            // Update entry count
            stats.TotalEntries++
            
            // Update level counts
            switch strings.ToUpper(entry.Level) {
            case "DEBUG":
                stats.DebugCount++
            case "INFO":
                stats.InfoCount++
            case "WARN":
                stats.WarnCount++
            case "ERROR", "FATAL":
                stats.ErrorCount++
                errorMap[entry.Message]++
            }
            
            // Update time range
            if entry.Timestamp.Before(stats.TimeRange[0]) {
                stats.TimeRange[0] = entry.Timestamp
            }
            if entry.Timestamp.After(stats.TimeRange[1]) {
                stats.TimeRange[1] = entry.Timestamp
            }
        }
        
        // Calculate top errors
        type errorCount struct {
            message string
            count   int
        }
        
        var errors []errorCount
        for msg, count := range errorMap {
            errors = append(errors, errorCount{msg, count})
        }
        
        // Sort errors by count (simplified - in real code, use sort package)
        // This is a simple bubble sort for demonstration
        for i := 0; i < len(errors); i++ {
            for j := i + 1; j < len(errors); j++ {
                if errors[i].count < errors[j].count {
                    errors[i], errors[j] = errors[j], errors[i]
                }
            }
        }
        
        // Get top 5 errors or fewer if there are less than 5
        topCount := 5
        if len(errors) < 5 {
            topCount = len(errors)
        }
        
        stats.TopErrors = make([]string, topCount)
        for i := 0; i < topCount; i++ {
            stats.TopErrors[i] = errors[i].message
        }
        
        // Send aggregated stats
        select {
        case out <- stats:
            // Stats sent
        case <-ctx.Done():
            return
        }
    }()
    
    return out
}

// JSONWriter writes entries to a JSON file
func JSONWriter(ctx context.Context, entries <-chan LogEntry, outputPath string, batchSize int) <-chan int {
    out := make(chan int, 1)
    
    go func() {
        defer close(out)
        
        // Create output file
        file, err := os.Create(outputPath)
        if err != nil {
            fmt.Fprintf(os.Stderr, "Error creating output file: %v\n", err)
            out <- 0
            return
        }
        defer file.Close()
        
        // Create JSON encoder
        encoder := json.NewEncoder(file)
        
        // Write opening bracket for array
        file.WriteString("[\n")
        
        count := 0
        first := true
        batch := make([]LogEntry, 0, batchSize)
        
        for entry := range entries {
            // Add to batch
            batch = append(batch, entry)
            count++
            
            // Write batch if full
            if len(batch) >= batchSize {
                writeEntryBatch(file, batch, first, encoder)
                first = false
                batch = batch[:0] // Clear batch
            }
            
            // Check for cancellation
            select {
            case <-ctx.Done():
                break
            default:
                // Continue
            }
        }
        
        // Write any remaining entries
        if len(batch) > 0 {
            writeEntryBatch(file, batch, first, encoder)
        }
        
        // Write closing bracket for array
        file.WriteString("\n]")
        
        // Send total count
        select {
        case out <- count:
            // Count sent
        case <-ctx.Done():
            return
        }
    }()
    
    return out
}

// Helper function to write a batch of entries
func writeEntryBatch(file *os.File, batch []LogEntry, first bool, encoder *json.Encoder) {
    for i, entry := range batch {
        if !first || i > 0 {
            file.WriteString(",\n")
        }
        
        // Write entry
        encoder.Encode(entry)
    }
}

// Helper function to parse a log entry (simplified)
func parseLogEntry(line string) LogEntry {
    // This is a very simplified parser
    // In a real implementation, you'd use regex or proper parsers
    
    parts := strings.SplitN(line, " ", 4)
    entry := LogEntry{
        Raw:      line,
        Metadata: make(map[string]interface{}),
    }
    
    if len(parts) >= 1 {
        // Try to parse timestamp
        timestamp, err := time.Parse("2006-01-02T15:04:05", parts[0])
        if err == nil {
            entry.Timestamp = timestamp
        } else {
            entry.Timestamp = time.Now()
        }
    }
    
    if len(parts) >= 2 {
        entry.Level = parts[1]
    }
    
    if len(parts) >= 3 {
        entry.Message = parts[2]
    }
    
    if len(parts) >= 4 {
        // Try to extract metadata
        metaStr := parts[3]
        if strings.Contains(metaStr, "=") {
            metaParts := strings.Split(metaStr, " ")
            for _, p := range metaParts {
                kv := strings.SplitN(p, "=", 2)
                if len(kv) == 2 {
                    entry.Metadata[kv[0]] = kv[1]
                }
            }
        }
    }
    
    return entry
}

func main() {
    // Create context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    // Directory to scan
    logDir := "./logs"
    
    // Start the processing pipeline
    fmt.Println("Starting log processing pipeline...")
    
    // Stage 1: Find log files
    fmt.Println("Finding log files...")
    logFiles := FileScanner(ctx, logDir, "*.log")
    
    // Set up a wait group to track file processing
    var wg sync.WaitGroup
    
    // Process each file
    fileCount := 0
    for filePath := range logFiles {
        fileCount++
        wg.Add(1)
        
        go func(path string) {
            defer wg.Done()
            
            fmt.Printf("Processing file: %s\n", path)
            
            // Create file-specific context
            fileCtx, fileCancel := context.WithCancel(ctx)
            defer fileCancel()
            
            // Stage 2: Read lines from file
            lines := LineReader(fileCtx, path)
            
            // Stage 3: Parse log entries
            entries := LogParser(fileCtx, lines)
            
            // Stage 4: Filter entries by level
            filteredEntries := LogFilter(fileCtx, entries, "INFO")
            
            // Stage 5: Aggregate statistics
            statsCh := StatAggregator(fileCtx, filteredEntries)
            
            // Get stats
            stats, ok := <-statsCh
            if !ok {
                fmt.Printf("Failed to get stats for %s\n", path)
                return
            }
            
            // Output results
            fmt.Printf("File: %s\n", path)
            fmt.Printf("  Total entries: %d\n", stats.TotalEntries)
            fmt.Printf("  Error entries: %d\n", stats.ErrorCount)
            fmt.Printf("  Warning entries: %d\n", stats.WarnCount)
            fmt.Printf("  Info entries: %d\n", stats.InfoCount)
            fmt.Printf("  Debug entries: %d\n", stats.DebugCount)
            fmt.Printf("  Time range: %v to %v\n", stats.TimeRange[0], stats.TimeRange[1])
            
            if len(stats.TopErrors) > 0 {
                fmt.Println("  Top errors:")
                for i, err := range stats.TopErrors {
                    fmt.Printf("    %d. %s\n", i+1, err)
                }
            }
        }(filePath)
    }
    
    // Wait for all files to be processed
    wg.Wait()
    
    if fileCount == 0 {
        fmt.Println("No log files found")
    } else {
        fmt.Printf("Processed %d log files\n", fileCount)
    }
    
    fmt.Println("Log processing complete")
}
```

**Common Pitfalls:**
- Creating generators without proper cancellation, leading to goroutine leaks
- Not closing channels when done generating, causing deadlocks in consumers
- Forgetting to check for closed channels in iterators
- Using unbuffered channels for generators that need to produce many values quickly
- Creating complex generator chains that are hard to debug
- Overlooking the memory overhead of goroutines in very large generator networks
- Not implementing proper error handling in generators
- Using recursive generators without base cases, causing infinite loops
- Missing synchronization when accessing iterator state from multiple goroutines

**Confusion Questions:**

1. **Q: How do generators in Go differ from generators in languages like Python or JavaScript?**

   A: Generators in Go differ significantly from those in languages like Python or JavaScript, primarily in implementation approach and concurrency model:

   **Implementation Approach Differences:**

   **In Python/JavaScript:**
   ```python
   # Python generator using yield
   def range_generator(start, end):
       for i in range(start, end):
           yield i

   # Usage
   for num in range_generator(1, 5):
       print(num)
   ```

   ```javascript
   // JavaScript generator using yield
   function* rangeGenerator(start, end) {
       for (let i = start; i < end; i++) {
           yield i;
       }
   }

   // Usage
   for (const num of rangeGenerator(1, 5)) {
       console.log(num);
   }
   ```

   **In Go:**
   ```go
   // Go generator using goroutines and channels
   func rangeGenerator(ctx context.Context, start, end int) <-chan int {
       out := make(chan int)
       
       go func() {
           defer close(out)
           
           for i := start; i < end; i++ {
               select {
               case <-ctx.Done():
                   return
               case out <- i:
                   // Value sent
               }
           }
       }()
       
       return out
   }

   // Usage
   for num := range rangeGenerator(ctx, 1, 5) {
       fmt.Println(num)
   }
   ```

   **Key differences:**

   1. **Language Support**:
      - Python/JavaScript: Native language feature with `yield` keyword
      - Go: Pattern implemented using goroutines and channels

   2. **Execution Model**:
      - Python/JavaScript: Pause and resume execution within the same function
      - Go: Separate goroutine runs concurrently, communicating via channels

   3. **Memory Overhead**:
      - Python/JavaScript: Minimal overhead to save function state
      - Go: Goroutine overhead (initially ~2KB per goroutine)

   4. **Concurrency**:
      - Python/JavaScript: Generators are not inherently concurrent
      - Go: Generators are built on Go's concurrency primitives

   5. **Cancellation**:
      - Python/JavaScript: Requires manual handling
      - Go: Natural cancellation through context or closing channels

   6. **Composition**:
      - Python/JavaScript: Can use generator delegation (yield from/yield*)
      - Go: Pipeline composition by connecting channels

   **Performance implications:**

   Go's approach offers several advantages:

   1. **Parallelism**: Go generators can truly run in parallel on multiple cores
   2. **Backpressure**: Unbuffered channels provide natural backpressure
   3. **Cancellation**: Easy integration with Go's context package
   4. **Distribution**: Can distribute generators across network boundaries

   However, there are trade-offs:

   1. **Overhead**: Higher memory usage due to goroutine allocation
   2. **Complexity**: More complex to implement simple iterative patterns
   3. **Debugging**: Harder to trace execution flow across goroutines

   **When to use Go's approach vs. traditional iteration:**

   - Use Go generators when:
     - Processing large data sets that benefit from parallelism
     - Building pipelines with multiple transformation stages
     - Dealing with I/O bound operations that can run concurrently
     - Creating truly infinite sequences where memory is a concern

   - Use traditional iteration when:
     - Dealing with small, finite collections
     - Maximum performance is critical and data fits in memory
     - The complexity of channels and goroutines isn't justified
     - Sequential processing is more appropriate for the problem

   Go's generator pattern is a powerful tool, but it represents a different philosophy: build concurrency into your fundamental data flow patterns, rather than adding it as an afterthought.

2. **Q: How do I handle errors in generator pipelines?**

   A: Error handling in Go generator pipelines requires special attention because errors can occur in any stage. Here are comprehensive strategies for handling errors in generator pipelines:

   **Strategy 1: Include errors in the output type**

   Create a result type that includes both the value and any error:

   ```go
   type Result struct {
       Value int
       Err   error
   }

   func Generator(ctx context.Context, data []int) <-chan Result {
       out := make(chan Result)
       
       go func() {
           defer close(out)
           
           for _, v := range data {
               // Simulate potential error
               var err error
               if v < 0 {
                   err = fmt.Errorf("negative value: %d", v)
               }
               
               select {
               case <-ctx.Done():
                   return
               case out <- Result{Value: v, Err: err}:
                   // Result sent
               }
           }
       }()
       
       return out
   }

   // Consumer checks errors
   for result := range Generator(ctx, data) {
       if result.Err != nil {
           log.Printf("Error: %v", result.Err)
           continue // or handle error differently
       }
       
       // Process valid result
       fmt.Println("Value:", result.Value)
   }
   ```

   **Strategy 2: Separate error channel**

   Use a separate channel for errors, especially useful for centralized error handling:

   ```go
   func GeneratorWithErrCh(ctx context.Context, data []int) (<-chan int, <-chan error) {
       valueCh := make(chan int)
       errCh := make(chan error, 1) // Buffer prevents blocking if consumer only reads values
       
       go func() {
           defer close(valueCh)
           defer close(errCh)
           
           for _, v := range data {
               if v < 0 {
                   select {
                   case errCh <- fmt.Errorf("negative value: %d", v):
                       // Error sent, continue with next value
                   case <-ctx.Done():
                       return
                   }
                   continue
               }
               
               select {
               case <-ctx.Done():
                   return
               case valueCh <- v:
                   // Value sent
               }
           }
       }()
       
       return valueCh, errCh
   }

   // Consumer handles both channels
   valuesCh, errCh := GeneratorWithErrCh(ctx, data)
   
   // Handle errors in separate goroutine
   go func() {
       for err := range errCh {
           log.Printf("Error: %v", err)
       }
   }()
   
   // Process values
   for v := range valuesCh {
       fmt.Println("Value:", v)
   }
   ```

   **Strategy 3: Error pipeline stage**

   Create a dedicated pipeline stage for error handling:

   ```go
   func HandleErrors(ctx context.Context, in <-chan Result) (<-chan int, <-chan error) {
       outCh := make(chan int)
       errCh := make(chan error, 10) // Buffered to prevent blocking
       
       go func() {
           defer close(outCh)
           defer close(errCh)
           
           for result := range in {
               if result.Err != nil {
                   select {
                   case errCh <- result.Err:
                       // Error sent
                   case <-ctx.Done():
                       return
                   }
                   continue
               }
               
               select {
               case outCh <- result.Value:
                   // Value sent
               case <-ctx.Done():
                   return
               }
           }
       }()
       
       return outCh, errCh
   }
   
   // In use:
   resultCh := Generator(ctx, data)
   valuesCh, errCh := HandleErrors(ctx, resultCh)
   
   // Handle errors and values separately
   ```

   **Strategy 4: Error aggregation**

   Collect and aggregate errors from multiple stages:

   ```go
   type ErrorCollector struct {
       errors []error
       mu     sync.Mutex
   }

   func (ec *ErrorCollector) Add(err error) {
       ec.mu.Lock()
       defer ec.mu.Unlock()
       ec.errors = append(ec.errors, err)
   }

   func (ec *ErrorCollector) Errors() []error {
       ec.mu.Lock()
       defer ec.mu.Unlock()
       return ec.errors
   }
   
   func Process(ctx context.Context, data []int) ([]int, []error) {
       // Create pipeline with error collector
       collector := &ErrorCollector{}
       
       // Source generator
       source := func() <-chan Result {
           out := make(chan Result)
           go func() {
               defer close(out)
               for _, v := range data {
                   if v < 0 {
                       out <- Result{Err: fmt.Errorf("negative input: %d", v)}
                   } else {
                       out <- Result{Value: v}
                   }
               }
           }()
           return out
       }()
       
       // Process stage
       processed := func(in <-chan Result) <-chan Result {
           out := make(chan Result)
           go func() {
               defer close(out)
               for r := range in {
                   // Skip results with errors
                   if r.Err != nil {
                       collector.Add(r.Err)
                       continue
                   }
                   
                   // Process value
                   if r.Value%2 == 0 {
                       out <- Result{Value: r.Value * 2}
                   } else {
                       err := fmt.Errorf("odd value not supported: %d", r.Value)
                       collector.Add(err)
                   }
               }
           }()
           return out
       }(source)
       
       // Collect results
       var results []int
       for r := range processed {
           if r.Err != nil {
               collector.Add(r.Err)
               continue
           }
           results = append(results, r.Value)
       }
       
       return results, collector.Errors()
   }
   ```

   **Strategy 5: Context-based cancellation on first error**

   Cancel the entire pipeline when any stage encounters an error:

   ```go
   func PipelineWithEarlyExit(parentCtx context.Context, data []int) ([]int, error) {
       ctx, cancel := context.WithCancel(parentCtx)
       defer cancel() // Ensure cancellation
       
       var firstErr error
       var errOnce sync.Once
       
       setError := func(err error) {
           errOnce.Do(func() {
               firstErr = err
               cancel() // Cancel context on first error
           })
       }
       
       // First stage generator
       stage1 := func() <-chan int {
           out := make(chan int)
           go func() {
               defer close(out)
               for _, v := range data {
                   if v < 0 {
                       setError(fmt.Errorf("negative value: %d", v))
                       return
                   }
                   
                   select {
                   case out <- v:
                       // Value sent
                   case <-ctx.Done():
                       return
                   }
               }
           }()
           return out
       }()
       
       // Second stage processor
       stage2 := func(in <-chan int) <-chan int {
           out := make(chan int)
           go func() {
               defer close(out)
               for v := range in {
                   result := v * 2
                   if result > 100 {
                       setError(fmt.Errorf("result too large: %d", result))
                       return
                   }
                   
                   select {
                   case out <- result:
                       // Processed value sent
                   case <-ctx.Done():
                       return
                   }
               }
           }()
           return out
       }(stage1)
       
       // Collect results
       var results []int
       for v := range stage2 {
           results = append(results, v)
           
           // Check for cancellation
           if ctx.Err() != nil {
               break
           }
       }
       
       if firstErr != nil {
           return results, firstErr
       }
       
       if parentCtx.Err() != nil {
           return results, parentCtx.Err()
       }
       
       return results, nil
   }
   ```

   **Best practices for error handling in generators:**

   1. **Be consistent**: Use the same error handling approach throughout your pipeline
   2. **Favor early error detection**: Validate inputs before processing when possible
   3. **Consider error severity**: Not all errors should terminate the pipeline
   4. **Provide context**: Wrap errors with stage-specific information
   5. **Handle resource cleanup**: Ensure resources are freed even when errors occur
   6. **Document error handling**: Make it clear how errors propagate through your pipeline
   7. **Test error paths**: Verify your error handling logic works as expected

   The best approach depends on your specific requirements - whether you need to process partial results despite errors (strategies 1-3) or fail fast on any error (strategy 5).

3. **Q: How can I limit memory usage when processing very large data sets with generators?**

   A: Processing very large data sets with generators requires careful memory management to avoid exhaustion. Here are comprehensive strategies for limiting memory usage:

   **1. Use backpressure with unbuffered channels**

   Unbuffered channels naturally implement backpressure, preventing producers from outpacing consumers:

   ```go
   func DataGenerator(ctx context.Context) <-chan LargeData {
       // Unbuffered channel forces synchronization
       out := make(chan LargeData) // Not buffered
       
       go func() {
           defer close(out)
           
           for {
               data, err := fetchNextChunk()
               if err != nil {
                   if errors.Is(err, io.EOF) {
                       return
                   }
                   log.Printf("Error fetching chunk: %v", err)
                   continue
               }
               
               select {
               case out <- data:
                   // Will block until consumer is ready to receive
               case <-ctx.Done():
                   return
               }
           }
       }()
       
       return out
   }
   ```

   **2. Process data in fixed-size chunks**

   Break large datasets into manageable chunks:

   ```go
   const chunkSize = 1000 // Process 1000 records at a time

   func ChunkedProcessor(ctx context.Context, source <-chan Record) <-chan ProcessedChunk {
       out := make(chan ProcessedChunk)
       
       go func() {
           defer close(out)
           
           chunk := make([]Record, 0, chunkSize)
           for record := range source {
               chunk = append(chunk, record)
               
               // When chunk is full, process it
               if len(chunk) >= chunkSize {
                   result := processChunk(chunk)
                   
                   select {
                   case out <- result:
                       // Sent processed chunk
                   case <-ctx.Done():
                       return
                   }
                   
                   // Create new chunk (reuse memory)
                   chunk = make([]Record, 0, chunkSize)
               }
           }
           
           // Process final partial chunk
           if len(chunk) > 0 {
               select {
               case out <- processChunk(chunk):
                   // Sent final chunk
               case <-ctx.Done():
                   return
               }
           }
       }()
       
       return out
   }
   ```

   **3. Implement object pooling for large structures**

   Reuse memory with sync.Pool:

   ```go
   type DataBlock struct {
       Buffer []byte
       // Other fields...
   }

   var blockPool = sync.Pool{
       New: func() interface{} {
           return &DataBlock{Buffer: make([]byte, 64*1024)} // 64KB blocks
       },
   }

   func StreamProcessor(ctx context.Context, reader io.Reader) <-chan Result {
       out := make(chan Result)
       
       go func() {
           defer close(out)
           
           for {
               // Get block from pool
               block := blockPool.Get().(*DataBlock)
               
               // Read data into block
               n, err := reader.Read(block.Buffer)
               if err != nil {
                   if err != io.EOF {
                       log.Printf("Read error: %v", err)
                   }
                   
                   // Return block to pool
                   blockPool.Put(block)
                   break
               }
               
               // Process block and send result
               if n > 0 {
                   // Clone only the data we need
                   dataToProcess := make([]byte, n)
                   copy(dataToProcess, block.Buffer[:n])
                   
                   result := processData(dataToProcess)
                   
                   // Return block to pool early
                   blockPool.Put(block)
                   
                   select {
                   case out <- result:
                       // Sent result
                   case <-ctx.Done():
                       return
                   }
               }
           }
       }()
       
       return out
   }
   ```

   **4. Implement windowing for time-series or sequential data**

   Process data in sliding windows to limit memory usage:

   ```go
   func SlidingWindowProcessor(ctx context.Context, source <-chan DataPoint, windowSize int, slideSize int) <-chan WindowResult {
       out := make(chan WindowResult)
       
       go func() {
           defer close(out)
           
           // Circular buffer implementation
           buffer := make([]DataPoint, windowSize)
           bufferIdx := 0
           totalPoints := 0
           
           for point := range source {
               // Add to buffer, overwriting oldest data
               buffer[bufferIdx] = point
               bufferIdx = (bufferIdx + 1) % windowSize
               
               totalPoints++
               
               // Process window when we have enough data and hit slide boundary
               if totalPoints >= windowSize && totalPoints%slideSize == 0 {
                   // Create a window from buffer
                   window := make([]DataPoint, windowSize)
                   
                   // Arrange in chronological order
                   for i := 0; i < windowSize; i++ {
                       window[i] = buffer[(bufferIdx+i)%windowSize]
                   }
                   
                   result := processWindow(window)
                   
                   select {
                   case out <- result:
                       // Window result sent
                   case <-ctx.Done():
                       return
                   }
               }
           }
       }()
       
       return out
   }
   ```

   **5. Use streaming algorithms**

   For certain types of processing, use algorithms that don't require holding all data:

   ```go
   func StreamingStatsGenerator(ctx context.Context, source <-chan float64) <-chan Stats {
       out := make(chan Stats)
       
       go func() {
           defer close(out)
           
           var (
               count int64
               sum, min, max float64
               mean float64
               m2 float64 // For online variance calculation
           )
           
           // Initialize
           first := true
           
           // Process stream one value at a time
           for value := range source {
               count++
               
               if first {
                   min = value
                   max = value
                   mean = value
                   first = false
               } else {
                   // Update min/max
                   if value < min {
                       min = value
                   }
                   if value > max {
                       max = value
                   }
                   
                   // Welford's algorithm for online mean and variance
                   delta := value - mean
                   mean += delta / float64(count)
                   m2 += delta * (value - mean)
               }
               
               // Periodically emit stats (e.g., every 1000 values)
               if count%1000 == 0 {
                   var variance float64
                   if count > 1 {
                       variance = m2 / float64(count-1)
                   }
                   
                   stats := Stats{
                       Count:    count,
                       Mean:     mean,
                       Min:      min,
                       Max:      max,
                       Variance: variance,
                   }
                   
                   select {
                   case out <- stats:
                       // Stats sent
                   case <-ctx.Done():
                       return
                   }
               }
           }
           
           // Send final stats
           var variance float64
           if count > 1 {
               variance = m2 / float64(count-1)
           }
           
           select {
           case out <- Stats{
               Count:    count,
               Mean:     mean,
               Min:      min,
               Max:      max,
               Variance: variance,
           }:
               // Final stats sent
           case <-ctx.Done():
               return
           }
       }()
       
       return out
   }
   ```

   **6. Implement memory tracking and adaptive throttling**

   Monitor memory usage and slow down processing if needed:

   ```go
   func MemoryAwareGenerator(ctx context.Context, source <-chan LargeData) <-chan ProcessedData {
       out := make(chan ProcessedData)
       
       go func() {
           defer close(out)
           
           ticker := time.NewTicker(100 * time.Millisecond)
           defer ticker.Stop()
           
           for {
               var data LargeData
               var ok bool
               
               // Try to get next item
               select {
               case data, ok = <-source:
                   if !ok {
                       return // Source is closed
                   }
               case <-ctx.Done():
                   return
               }
               
               // Check memory pressure
               if isHighMemoryUsage() {
                   log.Println("High memory usage detected, throttling")
                   
                   // Wait for memory pressure to reduce or context to cancel
                   select {
                   case <-time.After(1 * time.Second):
                       // Continue with processing after delay
                   case <-ctx.Done():
                       return
                   }
               }
               
               // Process data
               result := process(data)
               
               // Send result
               select {
               case out <- result:
                   // Result sent
               case <-ctx.Done():
                   return
               }
           }
       }()
       
       return out
   }
   
   func isHighMemoryUsage() bool {
       var m runtime.MemStats
       runtime.ReadMemStats(&m)
       
       // If using more than 80% of available memory, throttle
       return m.Alloc > (m.Sys * 8/10)
   }
   ```

   **7. Implement disk-backed buffers for overflow**

   Use disk storage when memory pressure is too high:

   ```go
   type DiskBackedBuffer struct {
       memoryBuffer []Data
       memoryLimit  int
       diskBuffer   *os.File
       count        int64
       mu           sync.Mutex
   }

   func NewDiskBackedBuffer(memoryLimit int) (*DiskBackedBuffer, error) {
       // Create temporary file
       tempFile, err := os.CreateTemp("", "buffer-*.tmp")
       if err != nil {
           return nil, err
       }
       
       return &DiskBackedBuffer{
           memoryBuffer: make([]Data, 0, memoryLimit),
           memoryLimit:  memoryLimit,
           diskBuffer:   tempFile,
       }, nil
   }

   func (b *DiskBackedBuffer) Add(data Data) error {
       b.mu.Lock()
       defer b.mu.Unlock()
       
       // If buffer isn't full, keep in memory
       if len(b.memoryBuffer) < b.memoryLimit {
           b.memoryBuffer = append(b.memoryBuffer, data)
           b.count++
           return nil
       }
       
       // Otherwise, write to disk
       bytes, err := json.Marshal(data)
       if err != nil {
           return err
       }
       
       // Write length prefix and data
       lenBytes := make([]byte, 8)
       binary.LittleEndian.PutUint64(lenBytes, uint64(len(bytes)))
       
       if _, err := b.diskBuffer.Write(lenBytes); err != nil {
           return err
       }
       
       if _, err := b.diskBuffer.Write(bytes); err != nil {
           return err
       }
       
       b.count++
       return nil
   }

   // Implementation of iterator interface omitted for brevity
   ```

   **8. Dynamic generator pipeline resizing**

   Adjust the number of parallel workers based on memory usage:

   ```go
   func AdaptiveParallelProcessor(ctx context.Context, source <-chan LargeData, initialWorkers int) <-chan Result {
       out := make(chan Result)
       
       // Channel for worker communication
       workCh := make(chan LargeData)
       
       // Worker count control
       var (
           activeWorkers int
           workerMu      sync.Mutex
           workerWg      sync.WaitGroup
       )
       
       // Monitoring goroutine
       go func() {
           ticker := time.NewTicker(1 * time.Second)
           defer ticker.Stop()
           
           for {
               select {
               case <-ticker.C:
                   // Check memory usage
                   var m runtime.MemStats
                   runtime.ReadMemStats(&m)
                   
                   memUsagePercent := float64(m.Alloc) / float64(m.Sys)
                   
                   workerMu.Lock()
                   
                   if memUsagePercent > 0.8 && activeWorkers > 1 {
                       // Too high memory usage, reduce workers
                       log.Printf("High memory usage (%.1f%%), reducing workers from %d to %d", 
                           memUsagePercent*100, activeWorkers, activeWorkers-1)
                       activeWorkers--
                   } else if memUsagePercent < 0.5 && activeWorkers < initialWorkers*2 {
                       // Memory usage is fine, can increase workers
                       log.Printf("Low memory usage (%.1f%%), increasing workers from %d to %d", 
                           memUsagePercent*100, activeWorkers, activeWorkers+1)
                       
                       // Launch new worker
                       workerWg.Add(1)
                       activeWorkers++
                       go worker(workCh, out, &workerWg)
                   }
                   
                   workerMu.Unlock()
                   
               case <-ctx.Done():
                   return
               }
           }
       }()
       
       // Start initial workers
       workerMu.Lock()
       activeWorkers = initialWorkers
       workerMu.Unlock()
       
       for i := 0; i < initialWorkers; i++ {
           workerWg.Add(1)
           go worker(workCh, out, &workerWg)
       }
       
       // Forward data from source to work channel
       go func() {
           defer close(workCh) // Signal workers to stop
           
           for data := range source {
               select {
               case workCh <- data:
                   // Data sent to a worker
               case <-ctx.Done():
                   return
               }
           }
       }()
       
       // Wait for workers to finish and close output
       go func() {
           workerWg.Wait()
           close(out)
       }()
       
       return out
   }
   
   func worker(in <-chan LargeData, out chan<- Result, wg *sync.WaitGroup) {
       defer wg.Done()
       
       for data := range in {
           result := process(data)
           out <- result
       }
   }
   ```

   **9. Implement lazy loading for nested data**

   Only load details when needed:

   ```go
   type LazyRecord struct {
       ID        string
       BasicInfo *BasicInfo
       detailsKey string
       detailsLoader func(key string) (*Details, error)
       details   *Details
       detailsLoaded bool
       mu        sync.Mutex
   }

   func (r *LazyRecord) GetDetails() (*Details, error) {
       r.mu.Lock()
       defer r.mu.Unlock()
       
       if r.detailsLoaded {
           return r.details, nil
       }
       
       details, err := r.detailsLoader(r.detailsKey)
       if err != nil {
           return nil, err
       }
       
       r.details = details
       r.detailsLoaded = true
       return details, nil
   }
   ```

   **Key principles for memory-efficient generators:**

   1. **Process incrementally**: Never load the entire dataset at once
   2. **Use streaming algorithms**: Choose algorithms that don't require the full dataset
   3. **Implement backpressure**: Prevent producers from overwhelming consumers
   4. **Monitor resource usage**: Track memory and adjust behavior accordingly
   5. **Pool and reuse objects**: Minimize allocations for frequently used structures
   6. **Control parallelism**: Match concurrency to available resources
   7. **Use appropriate data structures**: Choose memory-efficient representations
   8. **Benchmark and profile**: Measure memory usage to identify bottlenecks

   By combining these techniques, you can process datasets much larger than available memory while maintaining good performance.

## Next Actions

### Exercises and Micro-Projects

#### Exercise 1: Rate Limited API Client
Build a robust API client with rate limiting features, including:
- Token bucket rate limiting for API endpoints
- Concurrent request handling with bounded parallelism
- Automatic retries with exponential backoff
- Proper error handling and timeout management
- Support for different rate limits per endpoint

Test your client with a real or mock API service to ensure it respects rate limits while maximizing throughput.

#### Exercise 2: Infinite Sequence Generator Library
Create a library that implements various infinite sequence generators:
- Fibonacci sequence generator
- Prime number generator
- Random number generator with various distributions
- Date/time sequence generator
- Custom sequence generator with user-provided functions

Include utilities to limit, transform, filter, and combine these generators. Make sure all generators support proper cancellation via context.

#### Exercise 3: Concurrent Web Crawler
Build a web crawler that:
- Crawls websites while respecting robots.txt
- Implements rate limiting per domain
- Uses worker pools to download and process pages
- Extracts links and follows them within specified constraints
- Saves content or performs analytics
- Implements graceful shutdown

Focus on respecting website limits while maximizing crawl efficiency.

#### Exercise 4: Memory-Efficient Log Processor
Develop a log processing system that:
- Processes log files that are too large to fit in memory
- Uses generator patterns to stream data
- Implements multiple processing stages (parsing, filtering, aggregation)
- Provides statistical analysis of log data
- Handles multiple log formats
- Exports results in various formats (CSV, JSON, etc.)

Test with increasingly large log files to ensure memory usage remains constant.

#### Exercise 5: Adaptive Rate Limiter
Create an advanced rate limiter that:
- Automatically adjusts rate limits based on response times
- Implements different algorithms (token bucket, leaky bucket, etc.)
- Supports distributed rate limiting with Redis
- Provides monitoring and metrics
- Handles different priority levels for requests
- Gracefully degrades under high load

### Micro-Project: Data Pipeline Framework

Create a reusable data pipeline framework that combines both rate limiting and generator patterns:

**Requirements:**
1. A pipeline architecture with stages for source, transformation, and sink
2. Support for both batch and stream processing modes
3. Rate limiting capabilities at multiple points in the pipeline
4. Memory-efficient processing using generator patterns
5. Error handling with optional retry mechanisms
6. Monitoring and metrics collection
7. Graceful shutdown and resource cleanup
8. Pluggable components for custom processing logic

**Core Components:**
- Pipeline coordinator that manages overall flow
- Source connectors (files, databases, APIs, etc.)
- Transformation stages with customizable functions
- Rate limiters for controlling throughput
- Memory management utilities
- Error handling middleware
- Sink connectors for output destinations

**Advanced Features:**
- Back pressure mechanisms
- Dynamic scaling based on system load
- Checkpointing for resumable processing
- Priority queues for important data
- Circuit breakers for external dependencies

## Success Criteria

For rate limiting, you've mastered the concept when you can:

1. **Implement different rate limiting algorithms**
   - Create token bucket, leaky bucket, and fixed window rate limiters
   - Choose the appropriate algorithm for different use cases
   - Tune rate limiters for optimal performance

2. **Apply rate limiting at different levels**
   - Implement per-user, per-resource, and global rate limiting
   - Create hierarchical rate limiting systems
   - Coordinate rate limits across distributed systems

3. **Handle rate limit violations gracefully**
   - Implement backoff and retry mechanisms
   - Provide appropriate feedback to users or systems
   - Gracefully degrade service under high load

For generators and iterators, you've mastered the concept when you can:

1. **Design efficient data processing pipelines**
   - Create multi-stage generators that process data incrementally
   - Compose generator stages into complex pipelines
   - Implement proper error handling throughout the pipeline

2. **Process large datasets with constant memory**
   - Handle datasets larger than available RAM
   - Implement streaming algorithms for common operations
   - Monitor and control memory usage during processing

3. **Implement specialized generators for different data sources**
   - Create generators for files, databases, APIs, and other sources
   - Build custom iterators for complex data structures
   - Design generators that respect resource constraints

## Troubleshooting

### Common Issues and Solutions for Rate Limiting

1. **Too Restrictive Rate Limits**
   - **Problem**: Legitimate traffic is being blocked
   - **Solution**: Adjust rate limits based on actual usage patterns, implement burst allowances, or use dynamic rate limiting based on system capacity

2. **Bypassed Rate Limits**
   - **Problem**: Rate limits are ineffective due to distributed clients
   - **Solution**: Implement rate limiting based on user identity rather than source IP, use authentication tokens for rate limit tracking

3. **Poor Performance Under Load**
   - **Problem**: Rate limiting code becomes a bottleneck
   - **Solution**: Use efficient data structures, consider distributed rate limiting with Redis, implement local caching of rate limit status

4. **Inconsistent Distributed Rate Limiting**
   - **Problem**: Different servers apply different limits
   - **Solution**: Use a centralized rate limit store (Redis), implement consistent hashing, or use a distributed rate limit protocol

5. **Rate Limiter Memory Leaks**
   - **Problem**: Tracking too many users causes memory issues
   - **Solution**: Implement expiration for rate limit records, use efficient storage like Redis with TTL, implement cleanup for inactive users

### Common Issues and Solutions for Generators

1. **Goroutine Leaks**
   - **Problem**: Generators are created but never fully consumed
   - **Solution**: Always use context for cancellation, implement timeouts, ensure proper channel closing

2. **Memory Growth**
   - **Problem**: Generator chain uses more memory than expected
   - **Solution**: Process data in smaller chunks, implement backpressure, monitor memory usage, use object pooling

3. **Pipeline Deadlocks**
   - **Problem**: Generator chain has deadlocked stages
   - **Solution**: Use buffered channels where appropriate, ensure proper channel closing, implement timeouts for all operations

4. **Poor Performance**
   - **Problem**: Generator pipeline is slower than expected
   - **Solution**: Profile to find bottlenecks, adjust buffer sizes, balance worker counts, consider batch processing

5. **Error Handling Confusion**
   - **Problem**: Errors get lost in the pipeline or handling is inconsistent
   - **Solution**: Use consistent error handling strategy, propagate context cancellation, consider using structured error types

### Debugging Techniques

1. **Rate Limiter Debugging**
   ```go
   // Add debug logging to track rate limit decisions
   func (tb *TokenBucket) Wait(ctx context.Context) error {
       start := time.Now()
       result := tb.tryAcquire()
       
       if result {
           log.Printf("Rate limit: allowed immediately")
       } else {
           log.Printf("Rate limit: blocked, waiting for token")
           // Wait for token or context cancellation
           select {
           case <-tb.tokens:
               log.Printf("Rate limit: allowed after %v wait", time.Since(start))
               return nil
           case <-ctx.Done():
               log.Printf("Rate limit: cancelled after %v wait", time.Since(start))
               return ctx.Err()
           }
       }
       
       return nil
   }
   ```

2. **Channel State Visualization**
   ```go
   // Monitor channel capacity usage
   func monitorChannel(ch chan interface{}, name string) {
       ticker := time.NewTicker(time.Second)
       defer ticker.Stop()
       
       for range ticker.C {
           log.Printf("Channel %s: %d/%d (%.1f%% full)",
               name, len(ch), cap(ch), float64(len(ch))/float64(cap(ch))*100)
       }
   }
   ```

3. **Pipeline Stage Timing**
   ```go
   // Wrap a generator stage with timing measurements
   func timeStage(name string, in <-chan Data) <-chan Data {
       out := make(chan Data)
       
       go func() {
           defer close(out)
           
           count := 0
           start := time.Now()
           
           for data := range in {
               stageStart := time.Now()
               
               // Forward the data
               out <- data
               
               count++
               if count%100 == 0 {
                   elapsed := time.Since(stageStart)
                   log.Printf("Stage %s: processed %d items, last 100 took %v (%.2f items/sec)",
                       name, count, elapsed, float64(100)/elapsed.Seconds())
               }
           }
           
           totalTime := time.Since(start)
           log.Printf("Stage %s complete: %d items in %v (%.2f items/sec)",
               name, count, totalTime, float64(count)/totalTime.Seconds())
       }()
       
       return out
   }
   ```

4. **Memory Usage Tracking**
   ```go
   // Track memory usage during pipeline processing
   func trackMemory(ctx context.Context, interval time.Duration) {
       ticker := time.NewTicker(interval)
       defer ticker.Stop()
       
       var lastAlloc uint64
       
       for {
           select {
           case <-ticker.C:
               var m runtime.MemStats
               runtime.ReadMemStats(&m)
               
               currentAlloc := m.Alloc / 1024 / 1024 // MB
               allocDiff := int64(currentAlloc) - int64(lastAlloc)
               
               log.Printf("Memory: %d MB (Δ%+d MB)", currentAlloc, allocDiff)
               lastAlloc = currentAlloc
               
           case <-ctx.Done():
               return
           }
       }
   }
   ```

5. **Context Cancellation Tracing**
   ```go
   // Trace context cancellation through a pipeline
   func traceCtx(ctx context.Context, name string) context.Context {
       ctx, cancel := context.WithCancel(ctx)
       
       go func() {
           select {
           case <-ctx.Done():
               log.Printf("Context %s cancelled: %v", name, ctx.Err())
           }
       }()
       
       return ctx
   }
   ```

By understanding and addressing these common issues with rate limiters and generators, you'll be able to build robust, efficient, and memory-friendly concurrent applications in Go.

### Exercises and Micro-Projects

#### Exercise 1: Concurrent Data Processing Pipeline
Create a data processing pipeline that reads data from multiple files, processes them in parallel, and aggregates the results. Implement the following stages:
- File finding stage (searches for files)
- Data reading stage (reads the contents)
- Processing stage (transforms the data)
- Aggregation stage (combines results)
- Output stage (writes to file or console)

Use goroutines, channels, and proper error handling to build a robust pipeline that can process large amounts of data efficiently.

#### Exercise 2: Rate-Limited API Client
Build an HTTP client that can make concurrent API requests but respects rate limits. The client should:
- Allow specifying a maximum number of concurrent requests
- Implement a token bucket rate limiter
- Handle retries with exponential backoff
- Support request timeouts and cancellation
- Provide a clean API for users to make requests

Test your client against a real API (like GitHub's) or use a mock server.

#### Exercise 3: Worker Pool with Priority Queue
Implement a worker pool that processes tasks from a priority queue. The system should:
- Maintain a fixed number of worker goroutines
- Process tasks based on priority, not just arrival time
- Allow tasks to be cancelled
- Provide statistics on processing times and queue lengths
- Handle graceful shutdown

Use this to process different types of tasks with varying priorities.

#### Exercise 4: Distributed Counter
Create a system for maintaining counter statistics across multiple goroutines without locks. Implement:
- A set of counter goroutines that receive increment/decrement operations
- A dispatcher that distributes counter operations
- A way to retrieve the current count
- Benchmarks comparing this approach vs. mutex-based counters

#### Exercise 5: Context-Aware Cache
Implement a cache that respects context cancellation and deadlines. The cache should:
- Store values with optional TTL (time to live)
- Support context-aware get/set operations
- Allow batch operations
- Implement background cleanup of expired items
- Support cache size limits with eviction policies

### Micro-Project: Real-time Chat System

Develop a simple real-time chat system using Go's concurrency features.

**Requirements:**
1. Server that accepts WebSocket connections from clients
2. Support for multiple chat rooms
3. User presence tracking (online/offline status)
4. Message broadcasting to room participants
5. Private messaging between users
6. Rate limiting to prevent spam
7. Graceful shutdown that preserves messages

**Components to Implement:**
- Connection manager (tracks active WebSocket connections)
- Room manager (handles room membership)
- Message broker (distributes messages to appropriate recipients)
- User manager (handles user states and permissions)
- Rate limiter (prevents message flooding)
- Persistence layer (saves messages for retrieval)

**Advanced Features:**
- Typing indicators
- Read receipts
- Message history retrieval
- File sharing
- User authentication

## Success Criteria

You've mastered Go concurrency when you can:

1. **Design concurrent systems**
   - Choose appropriate concurrency patterns for a given problem
   - Balance parallelism with resource constraints
   - Structure code to avoid race conditions and deadlocks
   - Apply fan-out, fan-in, pipelines, and worker pools correctly

2. **Use goroutines effectively**
   - Create the right number of goroutines for a given workload
   - Ensure goroutines terminate properly
   - Handle errors in concurrent code
   - Prevent goroutine leaks

3. **Master channel operations**
   - Choose between buffered and unbuffered channels
   - Properly close channels
   - Handle closed channel detection
   - Apply select statements for multiplexing
   - Use channel direction constraints

4. **Implement cancellation and timeouts**
   - Use context to propagate cancellation signals
   - Apply appropriate timeouts for operations
   - Handle context values properly
   - Clean up resources when operations are cancelled

5. **Manage shared state**
   - Choose between channels and mutexes appropriately
   - Implement thread-safe data structures
   - Avoid race conditions
   - Use atomics when appropriate

6. **Write robust concurrent code**
   - Handle edge cases like early termination
   - Implement graceful shutdown procedures
   - Write comprehensive tests for concurrent code
   - Debug concurrency issues effectively

## Troubleshooting

### Common Issues and Solutions

1. **Deadlocks**
   - **Problem**: All goroutines are blocked, waiting for each other
   - **Detection**: Go runtime reports "fatal error: all goroutines are asleep - deadlock!"
   - **Solution**: Ensure proper channel usage (don't forget to close), avoid circular dependencies, use buffered channels when appropriate, or implement timeouts

2. **Goroutine leaks**
   - **Problem**: Goroutines never terminate, consuming resources
   - **Detection**: Increasing memory usage, `runtime.NumGoroutine()` shows growing count
   - **Solution**: Always provide a way for goroutines to exit (context cancellation, done channels), ensure channels are properly closed

3. **Race conditions**
   - **Problem**: Unpredictable behavior due to concurrent access to shared data
   - **Detection**: Use `-race` flag with `go test` or `go build`
   - **Solution**: Use synchronization primitives (mutex, channels) or ensure data is only accessed by one goroutine at a time

4. **Channel send on closed channel**
   - **Problem**: Sending on a closed channel causes panic
   - **Detection**: Runtime panic: "send on closed channel"
   - **Solution**: Ensure only one goroutine is responsible for closing a channel, use sync.Once for closing if multiple goroutines might close it

5. **Context leaks**
   - **Problem**: Not calling cancel() function leads to context leaks
   - **Detection**: Growing number of goroutines, resource usage
   - **Solution**: Always defer cancel() immediately after creating a context with cancellation

6. **Oversynchronization**
   - **Problem**: Too much synchronization leading to performance issues
   - **Detection**: Poor performance, excessive blocking
   - **Solution**: Reduce lock contention, use read-write locks, consider lock-free algorithms

7. **Channel backpressure**
   - **Problem**: Fast producers overwhelm slow consumers
   - **Detection**: Growing memory usage, delayed processing
   - **Solution**: Use buffered channels appropriately, implement rate limiting, or use worker pools with controlled concurrency

### Debugging Techniques

1. **Race detector**
   ```bash
   go test -race ./...
   go run -race myprogram.go
   ```
   Identifies data races in your code

2. **Logging goroutine information**
   ```go
   log.Printf("Goroutines: %d", runtime.NumGoroutine())
   ```
   Track how many goroutines are running at different points

3. **Stack traces for all goroutines**
   ```go
   buf := make([]byte, 64*1024)
   n := runtime.Stack(buf, true)
   log.Printf("=== Stack trace for %d goroutines ===\n%s", runtime.NumGoroutine(), buf[:n])
   ```
   Print stack traces for all goroutines to see what they're doing

4. **Timeout-based debugging**
   ```go
   func debugDeadlock() {
       time.AfterFunc(10*time.Second, func() {
           log.Println("Possible deadlock detected")
           buf := make([]byte, 64*1024)
           n := runtime.Stack(buf, true)
           log.Printf("Stack: %s", buf[:n])
           os.Exit(1)
       })
   }
   ```
   Force a stack dump if your program appears stuck

5. **Channel state inspection**
   ```go
   // For buffered channels, check capacity and length
   fmt.Printf("Channel: %d/%d\n", len(ch), cap(ch))
   ```
   Monitor channel buffer utilization

6. **Context tracing**
   ```go
   func withContextTracing(parent context.Context, name string) context.Context {
       ctx, cancel := context.WithCancel(parent)
       
       go func() {
           select {
           case <-parent.Done():
               log.Printf("Context %s cancelled by parent: %v", name, parent.Err())
           case <-ctx.Done():
               log.Printf("Context %s cancelled directly: %v", name, ctx.Err())
           }
       }()
       
       return ctx
   }
   ```
   Track when and why contexts are being cancelled

By mastering these troubleshooting techniques and understanding common concurrency pitfalls, you'll be well-equipped to build robust, efficient concurrent applications in Go.
