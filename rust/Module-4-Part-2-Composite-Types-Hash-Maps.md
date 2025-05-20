# Module 4: Composite Types - Part 2: Hash Maps

## Core Problem and Key Assumptions
- **Problem**: Understanding how to effectively use hash maps to store and retrieve data with key-value associations
- **Key Assumptions**:
  - You understand basic Rust syntax and primitive types
  - You need efficient key-value storage with fast lookups
  - You want to leverage Rust's type safety while working with key-value data
  - You aim to write efficient code using idiomatic Rust patterns

## Essential Concepts to Cover
1. Creating and Initializing Hash Maps
2. Adding, Accessing, and Removing Elements
3. Entry API
4. Iterating Over Hash Maps
5. Custom Hash Functions

---

## 1. Creating and Initializing Hash Maps

### Concise Explanation
HashMaps in Rust (`HashMap<K, V>`) are collections that store key-value pairs with efficient lookup, insertion, and deletion operations. Keys must be unique and implement the `Hash` and `Eq` traits. HashMap provides approximately O(1) average-case performance for lookups, inserts, and removals.

Unlike arrays and vectors, hash maps are unordered collections, meaning the elements aren't stored in any particular sequence. The actual ordering depends on the hash function and may change as elements are added or removed.

Key characteristics:
- Fast lookups by key (average O(1) complexity)
- Keys must be unique; adding a duplicate key overwrites the previous value
- Keys must implement `Hash` and `Eq` traits
- Stores data in key-value pairs, like dictionaries in other languages
- Unordered collection (no guaranteed iteration order)

### Where to Use
- When you need to associate values with keys
- For fast lookups when you have the key
- When implementing caches, indexes, or lookup tables
- For counting occurrences (frequency maps)
- When working with unique identifiers
- For implementing relationships between different data types

### Code Snippet
```rust
use std::collections::HashMap;

fn main() {
    // Creating an empty HashMap
    let mut scores: HashMap<String, i32> = HashMap::new();
    
    // Creating with initial capacity
    let mut population: HashMap<&str, u64> = HashMap::with_capacity(10);
    
    // Creating from tuples using collect
    let teams = vec!["Blue", "Red", "Green"];
    let initial_scores = vec![10, 50, 30];
    
    // Zip two iterators together and collect into HashMap
    let team_scores: HashMap<_, _> = teams
        .into_iter()
        .zip(initial_scores.into_iter())
        .collect();
    
    println!("Team scores: {:?}", team_scores);
    
    // Creating from array of tuples
    let mut capitals = HashMap::from([
        ("Japan", "Tokyo"),
        ("France", "Paris"),
        ("Germany", "Berlin"),
    ]);
    
    println!("Capitals: {:?}", capitals);
    
    // Initialize with default values using iterators
    let numbers: HashMap<i32, bool> = (1..=5).map(|i| (i, i % 2 == 0)).collect();
    println!("Even numbers: {:?}", numbers);
    
    // Creating using the from_iter method
    use std::iter::FromIterator;
    let pairs = vec![("apple", 5), ("banana", 8), ("orange", 3)];
    let fruit_counts = HashMap::from_iter(pairs);
    println!("Fruit counts: {:?}", fruit_counts);
    
    // Getting HashMap capacity and len
    println!("Capacity: {}", capitals.capacity());
    println!("Number of elements: {}", capitals.len());
    
    // Checking if a HashMap is empty
    println!("Is empty? {}", capitals.is_empty());
    
    // Creating a HashMap using macros (requires external crate)
    // For demonstration purposes, commented out:
    // let map = maplit::hashmap!{
    //     "a" => 1,
    //     "b" => 2,
    // };
}
```

### Real-World Example
A word frequency counter:

```rust
use std::collections::HashMap;
use std::fs::File;
use std::io::{self, BufRead, BufReader};

fn count_words(filename: &str) -> io::Result<HashMap<String, usize>> {
    let file = File::open(filename)?;
    let reader = BufReader::new(file);
    
    let mut word_counts: HashMap<String, usize> = HashMap::new();
    
    for line in reader.lines() {
        let line = line?;
        
        // Split the line into words and count them
        for word in line
            .split(|c: char| !c.is_alphanumeric())
            .filter(|s| !s.is_empty())
            .map(|s| s.to_lowercase())
        {
            // Increment the count for this word
            *word_counts.entry(word).or_insert(0) += 1;
        }
    }
    
    Ok(word_counts)
}

fn find_most_common_words(word_counts: &HashMap<String, usize>, limit: usize) -> Vec<(String, usize)> {
    // Convert the HashMap into a Vec of (word, count) pairs
    let mut counts: Vec<(String, usize)> = word_counts
        .iter()
        .map(|(word, &count)| (word.clone(), count))
        .collect();
    
    // Sort by count (descending) and then by word (ascending)
    counts.sort_by(|a, b| b.1.cmp(&a.1).then_with(|| a.0.cmp(&b.0)));
    
    // Take the top `limit` elements
    counts.truncate(limit);
    
    counts
}

fn main() -> io::Result<()> {
    // Count words in a text file
    let word_counts = count_words("sample.txt")?;
    
    println!("Total unique words: {}", word_counts.len());
    
    // Find the most common words
    let common_words = find_most_common_words(&word_counts, 10);
    
    println!("\nMost common words:");
    for (i, (word, count)) in common_words.iter().enumerate() {
        println!("{}. '{}': {} occurrences", i + 1, word, count);
    }
    
    // Create a HashMap that categorizes words by their frequency
    let mut frequency_categories: HashMap<usize, Vec<String>> = HashMap::new();
    
    for (word, &count) in &word_counts {
        frequency_categories
            .entry(count)
            .or_insert_with(Vec::new)
            .push(word.clone());
    }
    
    // Print words that appear exactly 5 times
    if let Some(words) = frequency_categories.get(&5) {
        println!("\nWords that appear exactly 5 times:");
        for word in words {
            println!("- {}", word);
        }
    }
    
    Ok(())
}
```

### Common Pitfalls
- **Pitfall**: Forgetting that HashMap keys need to implement both `Hash` and `Eq` traits.
  - **Solution**: Use types that implement these traits, or implement them for your custom types.
- **Pitfall**: Expecting HashMap iteration to be in a specific order.
  - **Solution**: Do not rely on iteration order; if order matters, use a `BTreeMap` or manually sort the entries.
- **Pitfall**: Using mutable references to HashMap values across iterations.
  - **Solution**: Be aware of ownership and borrowing rules; you generally can't modify a HashMap while iterating it.
- **Pitfall**: Not considering hash collisions in performance-critical code.
  - **Solution**: For types with potentially poor distribution, consider implementing a custom hasher.
- **Pitfall**: Using complex or mutable objects as keys.
  - **Solution**: Use simple, immutable types as keys to ensure hash consistency.

### Confusion Questions with Answers
1. **Q**: What's the difference between `HashMap` and `BTreeMap`?
   **A**: `HashMap` provides average O(1) lookups but doesn't maintain any ordering, while `BTreeMap` maintains keys in sorted order but has O(log n) lookup time. Use `HashMap` for pure performance, and `BTreeMap` when you need the keys to be ordered.

2. **Q**: Can I use any type as a key in a `HashMap`?
   **A**: No, key types must implement both the `Hash` and `Eq` traits. Most standard library types (like `String`, `i32`, etc.) implement these, but for custom types, you need to implement or derive these traits.

3. **Q**: Is the iteration order of a `HashMap` guaranteed to be consistent?
   **A**: No, the iteration order of a `HashMap` is not guaranteed to be consistent, even between program runs with the same data. If you need consistent ordering, consider using a `BTreeMap` or sorting the entries.

---

## 2. Adding, Accessing, and Removing Elements

### Concise Explanation
Working with hash map elements involves operations to insert key-value pairs, retrieve values by their keys, and remove entries. Rust provides straightforward methods for these operations while maintaining its strong safety guarantees.

Key operations:
- Inserting: Add or update key-value pairs
- Accessing: Retrieve values by key
- Checking: Determine if a key exists
- Removing: Delete key-value pairs
- Clearing: Remove all elements

These operations provide the core functionality that makes hash maps useful for efficiently managing key-value data.

### Where to Use
- Insertion: When adding new data or updating existing entries
- Access: When retrieving values based on a key
- Checking: For conditional logic based on key presence
- Removal: When data is no longer needed
- Clearing: When resetting a collection for reuse

### Code Snippet
```rust
use std::collections::HashMap;

fn main() {
    // Create a new hash map
    let mut menu_prices: HashMap<String, f64> = HashMap::new();
    
    // Inserting elements
    menu_prices.insert("Coffee".to_string(), 3.50);
    menu_prices.insert("Tea".to_string(), 2.80);
    menu_prices.insert("Cake".to_string(), 5.00);
    println!("Menu: {:?}", menu_prices);
    
    // Overwriting existing value
    menu_prices.insert("Coffee".to_string(), 3.75); // Price update
    println!("Updated menu: {:?}", menu_prices);
    
    // Inserting only if key doesn't exist
    menu_prices.entry("Tea".to_string()).or_insert(2.50); // Won't change
    menu_prices.entry("Sandwich".to_string()).or_insert(6.25); // New entry
    println!("Menu with sandwich: {:?}", menu_prices);
    
    // Accessing values
    
    // Using indexing (panics if key doesn't exist)
    let coffee_price = menu_prices["Coffee"];
    println!("Coffee costs ${:.2}", coffee_price);
    
    // Using get (returns Option<&V>)
    match menu_prices.get("Tea") {
        Some(&price) => println!("Tea costs ${:.2}", price),
        None => println!("Tea is not on the menu"),
    }
    
    // Using get_mut to modify a value in place
    if let Some(price) = menu_prices.get_mut("Cake") {
        *price *= 0.9; // 10% discount
        println!("Discounted cake price: ${:.2}", price);
    }
    
    // Checking if a key exists
    let item = "Burger";
    println!("Do we sell {}? {}", item, menu_prices.contains_key(item));
    
    // Getting a value or a default
    let soda_price = menu_prices.get("Soda").copied().unwrap_or(2.00);
    println!("Soda costs ${:.2} (default)", soda_price);
    
    // Removing elements
    
    // Remove a specific key-value pair
    if let Some(price) = menu_prices.remove("Tea") {
        println!("Removed Tea, which cost ${:.2}", price);
    }
    
    // Removing conditionally
    let expensive_threshold = 5.0;
    menu_prices.retain(|_, &mut price| price < expensive_threshold);
    println!("After removing expensive items: {:?}", menu_prices);
    
    // Getting the number of elements
    println!("Number of menu items: {}", menu_prices.len());
    
    // Clearing all elements
    menu_prices.clear();
    println!("After clearing: {:?}", menu_prices);
    println!("Is menu empty? {}", menu_prices.is_empty());
}
```

### Real-World Example
A simple cache implementation:

```rust
use std::collections::HashMap;
use std::hash::Hash;
use std::time::{Duration, Instant};

struct Cache<K, V> {
    map: HashMap<K, (V, Instant)>,
    ttl: Duration, // Time to live
}

impl<K, V> Cache<K, V>
where
    K: Eq + Hash + Clone,
    V: Clone,
{
    fn new(ttl_secs: u64) -> Self {
        Cache {
            map: HashMap::new(),
            ttl: Duration::from_secs(ttl_secs),
        }
    }
    
    fn insert(&mut self, key: K, value: V) {
        self.map.insert(key, (value, Instant::now()));
    }
    
    fn get(&mut self, key: &K) -> Option<V> {
        // First, check if the entry exists and is valid
        if let Some((value, timestamp)) = self.map.get(key) {
            // Check if the entry is still valid
            if timestamp.elapsed() < self.ttl {
                return Some(value.clone());
            } else {
                // Entry is expired, remove it
                self.map.remove(key);
            }
        }
        None
    }
    
    fn remove(&mut self, key: &K) -> Option<V> {
        self.map.remove(key).map(|(value, _)| value)
    }
    
    fn clear(&mut self) {
        self.map.clear();
    }
    
    fn len(&self) -> usize {
        self.map.len()
    }
    
    // Clean expired entries
    fn cleanup(&mut self) {
        self.map.retain(|_, (_, timestamp)| timestamp.elapsed() < self.ttl);
    }
    
    // Get all valid keys
    fn keys(&mut self) -> Vec<K> {
        // First cleanup expired entries
        self.cleanup();
        
        // Then collect valid keys
        self.map.keys().cloned().collect()
    }
}

fn expensive_computation(input: i32) -> String {
    // Simulate an expensive operation
    std::thread::sleep(Duration::from_millis(100));
    format!("Result for input {}: {}", input, input * input)
}

fn main() {
    // Create a cache with 5-second TTL
    let mut cache = Cache::new(5);
    
    // Function to get a result, using the cache if available
    let mut get_result = |input: i32| {
        let start = Instant::now();
        
        // Try to get from cache first
        if let Some(result) = cache.get(&input) {
            println!("Cache hit for input {}: {} (took {:?})", 
                     input, result, start.elapsed());
            return result;
        }
        
        // Not in cache, compute and store
        let result = expensive_computation(input);
        println!("Cache miss for input {}: {} (took {:?})", 
                 input, result, start.elapsed());
        cache.insert(input, result.clone());
        result
    };
    
    // First access (cache miss)
    get_result(42);
    
    // Second access to same value (cache hit)
    get_result(42);
    
    // Different value (cache miss)
    get_result(7);
    
    // Check cache state
    println!("Cache has {} entries", cache.len());
    println!("Cache keys: {:?}", cache.keys());
    
    // Remove an item
    if let Some(removed) = cache.remove(&42) {
        println!("Removed from cache: {}", removed);
    }
    
    // Try accessing removed item (will be a cache miss)
    get_result(42);
    
    // Wait for entries to expire
    println!("Waiting for cache entries to expire...");
    std::thread::sleep(Duration::from_secs(6));
    
    // This will be a cache miss because the entry expired
    get_result(7);
    
    // Check cache state after expiry
    println!("Cache has {} entries after expiry", cache.len());
}
```

### Common Pitfalls
- **Pitfall**: Using indexing (`map[key]`) which panics if the key doesn't exist.
  - **Solution**: Use `get` or `get_mut` which return `Option` types, or use the Entry API.
- **Pitfall**: Not handling missing keys properly.
  - **Solution**: Always consider the case where a key might not exist by using pattern matching or combinators like `unwrap_or`.
- **Pitfall**: Inefficient operations from repeatedly accessing the same key.
  - **Solution**: Use the Entry API for compound operations on the same key.
- **Pitfall**: Using complex expressions as map keys directly.
  - **Solution**: Store complex expressions in variables before using them as keys to avoid recomputation.
- **Pitfall**: Forgetting that `get` returns a reference, not the value itself.
  - **Solution**: Use `copied` or `cloned` for primitive or copyable types, or dereference and clone for owned values.

### Confusion Questions with Answers
1. **Q**: What happens if I try to access a key that doesn't exist using the index syntax (`map[key]`)?
   **A**: The program will panic at runtime. For safe access, use `get()` (which returns an `Option`) or the Entry API.

2. **Q**: How can I update a value in a hash map based on the old value?
   **A**: You have several options:
   - Use `get_mut` to get a mutable reference and update in place
   - Use `entry().and_modify()` to update existing entries
   - Use `entry().or_insert()` to update or insert
   - Remove and re-insert the updated value

3. **Q**: Is there a performance difference between checking if a key exists with `contains_key` and then accessing it with `get` versus just using `get` directly?
   **A**: Yes, using both `contains_key` and then `get` would be less efficient as it performs two lookups. It's better to just use `get` and match on the `Option` result, which requires only one lookup.

---

## 3. Entry API

### Concise Explanation
The Entry API is a powerful feature of Rust's HashMap that provides an efficient way to manipulate values based on whether a key exists. It gives you a single entry point for handling both the insertion of new keys and modification of existing values, without having to do repeated lookups.

Key aspects:
- The `entry` method returns an `Entry` enum with variants for vacant or occupied entries
- Provides methods like `or_insert`, `or_insert_with`, and `and_modify` for conditional operations
- Eliminates redundant lookups when you need to check and manipulate a value
- Enables efficient compound operations on a single key
- Maintains type safety through the borrow checker

The Entry API is particularly useful for counter patterns, caching, and other scenarios where you need to conditionally modify or initialize values.

### Where to Use
- When performing operations that depend on whether a key exists
- For counting occurrences or accumulating values
- In "upsert" (update-or-insert) scenarios
- When building caches or memoization tables
- For efficiently implementing complex update logic
- When you want to avoid redundant lookups

### Code Snippet
```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    
    // Basic or_insert - insert a value if the key doesn't exist
    scores.entry("Alice").or_insert(10);
    println!("Scores after inserting Alice: {:?}", scores);
    
    // or_insert returns a mutable reference to the value
    let score = scores.entry("Bob").or_insert(0);
    *score += 5; // Modify the value through the reference
    println!("Scores after inserting and updating Bob: {:?}", scores);
    
    // Counting occurrences with entry API
    let text = "hello world hello rust";
    let mut word_count = HashMap::new();
    
    for word in text.split_whitespace() {
        *word_count.entry(word).or_insert(0) += 1;
    }
    println!("Word counts: {:?}", word_count);
    
    // or_insert_with - use a function to create the value
    let mut cafeteria = HashMap::new();
    cafeteria.entry("Monday").or_insert_with(|| vec!["Soup", "Salad"]);
    println!("Cafeteria menu: {:?}", cafeteria);
    
    // and_modify - only modify if the key exists
    cafeteria.entry("Monday").and_modify(|menu| menu.push("Pasta"));
    cafeteria.entry("Tuesday").and_modify(|menu| menu.push("Pizza")); // Does nothing
    println!("Updated cafeteria menu: {:?}", cafeteria);
    
    // Combined operations
    cafeteria.entry("Tuesday")
        .and_modify(|menu| menu.push("Burger"))  // Only runs if key exists
        .or_insert_with(|| vec!["Sandwich"]);    // Runs if key doesn't exist
    println!("Cafeteria menu with Tuesday: {:?}", cafeteria);
    
    // Vacant and Occupied entries
    let mut inventory = HashMap::new();
    
    match inventory.entry("Apple") {
        std::collections::hash_map::Entry::Vacant(entry) => {
            // Key doesn't exist
            println!("No apples in inventory, adding some");
            entry.insert(5);
        },
        std::collections::hash_map::Entry::Occupied(mut entry) => {
            // Key exists
            let count = entry.get_mut();
            println!("Found {} apples, adding more", count);
            *count += 10;
        }
    }
    println!("Inventory: {:?}", inventory);
    
    // More complex logic with entry patterns
    let mut user_posts = HashMap::new();
    
    // Function to add a post for a user
    let add_post = |name: &str, post_id: i32| {
        user_posts
            .entry(name.to_string())
            .or_insert_with(Vec::new)
            .push(post_id);
    };
    
    add_post("alice", 1);
    add_post("bob", 2);
    add_post("alice", 3);
    
    println!("User posts: {:?}", user_posts);
    
    // Getting and removing an entry if it exists
    let mut cache = HashMap::new();
    cache.insert("key1", "value1");
    
    // Remove an entry if it exists and get its value
    if let Some(value) = cache.remove("key1") {
        println!("Removed: {}", value);
    }
    
    // Taking ownership of a value with Entry API
    let mut map = HashMap::new();
    map.insert("key", "old_value".to_string());
    
    // Replace and take ownership of the old value
    let old_value = map.entry("key")
        .and_modify(|v| *v = "new_value".to_string())
        .or_insert_with(|| "default".to_string());
    
    println!("Current value: {}, Map: {:?}", old_value, map);
}
```

### Real-World Example
An issue tracking system that uses the Entry API for updates:

```rust
use std::collections::HashMap;
use std::time::{Duration, Instant};

#[derive(Debug, Clone, PartialEq, Eq)]
enum Status {
    Open,
    InProgress,
    Resolved,
    Closed,
}

#[derive(Debug, Clone)]
struct Issue {
    id: u32,
    title: String,
    description: String,
    status: Status,
    assigned_to: Option<String>,
    created_at: Instant,
    updated_at: Instant,
    comments: Vec<String>,
    tags: Vec<String>,
}

impl Issue {
    fn new(id: u32, title: &str, description: &str) -> Self {
        let now = Instant::now();
        Issue {
            id,
            title: title.to_string(),
            description: description.to_string(),
            status: Status::Open,
            assigned_to: None,
            created_at: now,
            updated_at: now,
            comments: Vec::new(),
            tags: Vec::new(),
        }
    }
    
    fn add_comment(&mut self, comment: &str) {
        self.comments.push(comment.to_string());
        self.updated_at = Instant::now();
    }
    
    fn assign_to(&mut self, user: &str) {
        self.assigned_to = Some(user.to_string());
        self.updated_at = Instant::now();
        
        if self.status == Status::Open {
            self.status = Status::InProgress;
        }
    }
    
    fn update_status(&mut self, status: Status) {
        self.status = status;
        self.updated_at = Instant::now();
    }
    
    fn add_tag(&mut self, tag: &str) {
        if !self.tags.contains(&tag.to_string()) {
            self.tags.push(tag.to_string());
            self.updated_at = Instant::now();
        }
    }
    
    fn time_since_update(&self) -> Duration {
        self.updated_at.elapsed()
    }
}

struct IssueTracker {
    issues: HashMap<u32, Issue>,
    next_id: u32,
    tags: HashMap<String, Vec<u32>>,   // Tag to issue IDs mapping
    assigned: HashMap<String, Vec<u32>>, // User to issue IDs mapping
}

impl IssueTracker {
    fn new() -> Self {
        IssueTracker {
            issues: HashMap::new(),
            next_id: 1,
            tags: HashMap::new(),
            assigned: HashMap::new(),
        }
    }
    
    fn create_issue(&mut self, title: &str, description: &str) -> u32 {
        let id = self.next_id;
        self.next_id += 1;
        
        let issue = Issue::new(id, title, description);
        self.issues.insert(id, issue);
        
        id
    }
    
    fn add_tag(&mut self, issue_id: u32, tag: &str) -> bool {
        if let Some(issue) = self.issues.get_mut(&issue_id) {
            // Only add tag if it doesn't already exist
            if !issue.tags.contains(&tag.to_string()) {
                issue.add_tag(tag);
                
                // Update the tags index using Entry API
                self.tags
                    .entry(tag.to_string())
                    .or_insert_with(Vec::new)
                    .push(issue_id);
                
                return true;
            }
        }
        false
    }
    
    fn assign_issue(&mut self, issue_id: u32, user: &str) -> bool {
        // Use Entry API to handle the issue update
        match self.issues.entry(issue_id) {
            std::collections::hash_map::Entry::Occupied(mut entry) => {
                let issue = entry.get_mut();
                
                // Remove from previous assignment if any
                if let Some(prev_user) = &issue.assigned_to {
                    if let Some(issues) = self.assigned.get_mut(prev_user) {
                        issues.retain(|&id| id != issue_id);
                    }
                }
                
                // Assign to new user
                issue.assign_to(user);
                
                // Update the assigned index using Entry API
                self.assigned
                    .entry(user.to_string())
                    .or_insert_with(Vec::new)
                    .push(issue_id);
                
                true
            },
            std::collections::hash_map::Entry::Vacant(_) => false,
        }
    }
    
    fn add_comment(&mut self, issue_id: u32, comment: &str) -> bool {
        // Example of using get_mut instead of Entry (for comparison)
        if let Some(issue) = self.issues.get_mut(&issue_id) {
            issue.add_comment(comment);
            return true;
        }
        false
    }
    
    fn update_status(&mut self, issue_id: u32, status: Status) -> bool {
        // Another approach using the Entry API
        self.issues
            .entry(issue_id)
            .and_modify(|issue| issue.update_status(status.clone()))
            .is_occupied()
    }
    
    fn issues_by_tag(&self, tag: &str) -> Vec<&Issue> {
        match self.tags.get(tag) {
            Some(issue_ids) => issue_ids
                .iter()
                .filter_map(|id| self.issues.get(id))
                .collect(),
            None => Vec::new(),
        }
    }
    
    fn issues_assigned_to(&self, user: &str) -> Vec<&Issue> {
        self.assigned
            .get(user)
            .map_or_else(
                Vec::new,
                |issue_ids| issue_ids
                    .iter()
                    .filter_map(|id| self.issues.get(id))
                    .collect()
            )
    }
    
    fn get_issue(&self, id: u32) -> Option<&Issue> {
        self.issues.get(&id)
    }
}

fn main() {
    let mut tracker = IssueTracker::new();
    
    // Create some issues
    let issue1 = tracker.create_issue(
        "Login screen not responsive",
        "The login screen doesn't adapt to mobile resolutions."
    );
    
    let issue2 = tracker.create_issue(
        "Add dark mode support",
        "Implement a dark theme for the entire application."
    );
    
    let issue3 = tracker.create_issue(
        "Performance issues in dashboard",
        "Dashboard is slow to load with large datasets."
    );
    
    // Add tags using Entry API indirectly
    tracker.add_tag(issue1, "frontend");
    tracker.add_tag(issue1, "ui");
    tracker.add_tag(issue2, "frontend");
    tracker.add_tag(issue2, "enhancement");
    tracker.add_tag(issue3, "performance");
    tracker.add_tag(issue3, "backend");
    
    // Assign issues
    tracker.assign_issue(issue1, "alice");
    tracker.assign_issue(issue2, "bob");
    tracker.assign_issue(issue3, "alice");
    
    // Add comments
    tracker.add_comment(issue1, "This should be a high priority item.");
    tracker.add_comment(issue1, "I'll start working on this today.");
    
    // Update status
    tracker.update_status(issue1, Status::InProgress);
    tracker.update_status(issue2, Status::Open);
    
    // Print issues by tag
    println!("Frontend issues:");
    for issue in tracker.issues_by_tag("frontend") {
        println!("  #{}: {} - {:?}", issue.id, issue.title, issue.status);
    }
    
    // Print issues assigned to Alice
    println!("\nIssues assigned to Alice:");
    for issue in tracker.issues_assigned_to("alice") {
        println!("  #{}: {} - {:?}", issue.id, issue.title, issue.status);
        println!("    Tags: {:?}", issue.tags);
        if !issue.comments.is_empty() {
            println!("    Comments:");
            for comment in &issue.comments {
                println!("      - {}", comment);
            }
        }
    }
    
    // Resolve an issue
    tracker.update_status(issue1, Status::Resolved);
    
    // Get a specific issue
    if let Some(issue) = tracker.get_issue(issue1) {
        println!("\nIssue #{}:", issue.id);
        println!("  Title: {}", issue.title);
        println!("  Status: {:?}", issue.status);
        println!("  Assigned to: {:?}", issue.assigned_to);
        println!("  Last updated: {:?} ago", issue.time_since_update());
    }
}
```

### Common Pitfalls
- **Pitfall**: Forgetting that `or_insert` returns a mutable reference to the value.
  - **Solution**: Remember to dereference the returned reference (`*value_ref`) when modifying primitive types.
- **Pitfall**: Not using the returned value from `entry` operations.
  - **Solution**: Capture and use the return value when needed, especially for further modifications.
- **Pitfall**: Using `or_insert` with expensive operations for values that might already exist.
  - **Solution**: Use `or_insert_with` with a closure to lazily generate values only when needed.
- **Pitfall**: Unnecessary use of Entry API for simple lookups without modifications.
  - **Solution**: Use `get`/`get_mut` for simple lookups, and Entry API for complex insert/update operations.
- **Pitfall**: Using multiple Entry API operations on the same key in separate statements.
  - **Solution**: Chain Entry API operations to avoid redundant lookups.

### Confusion Questions with Answers
1. **Q**: What's the difference between `or_insert` and `or_insert_with`?
   **A**: `or_insert(val)` takes a value directly and uses it if the key doesn't exist, meaning the value is always evaluated even if not used. `or_insert_with(|| val)` takes a closure that is only evaluated if the key doesn't exist, which is more efficient for expensive operations.

2. **Q**: How does `and_modify` work with `or_insert`?
   **A**: When chained together, `and_modify` runs first and only executes if the key exists, while `or_insert` only executes if the key doesn't exist. This allows you to handle both cases efficiently in a single operation.

3. **Q**: Can I use Entry API to access both the key and value in a HashMap?
   **A**: Yes, you can use pattern matching on the result of `entry()` to distinguish between `Vacant` and `Occupied` entries. `Occupied` entries provide methods to access both the key and value, such as `key()`, `get()`, and `get_mut()`.

---

## 4. Iterating Over Hash Maps

### Concise Explanation
Iterating over a HashMap allows you to process all key-value pairs in the collection. Rust provides different types of iterators for hash maps: immutable iterators for keys, values, or both; and mutable iterators that allow modifying values.

Key ways to iterate:
- `iter()`: Borrow keys and values immutably as `(&K, &V)`
- `iter_mut()`: Borrow keys immutably and values mutably as `(&K, &mut V)`
- `into_iter()`: Consume the HashMap, yielding owned keys and values as `(K, V)`
- Iterate over just the keys (`keys()`) or just the values (`values()`)
- Use various iterator methods to filter, transform, or process the entries

Remember that HashMaps are unordered collections, so iteration order is not guaranteed and may change between program runs.

### Where to Use
- When you need to process all entries in a hash map
- For transforming or filtering a hash map's contents
- When accumulating or computing aggregate results from a collection
- To display or output all key-value pairs
- When converting a hash map to another collection type
- For batched updates or calculations across all entries

### Code Snippet
```rust
use std::collections::HashMap;

fn main() {
    // Create a sample hash map
    let mut population = HashMap::from([
        ("New York", 8_419_000),
        ("Tokyo", 9_273_000),
        ("London", 8_982_000),
        ("Paris", 2_161_000),
        ("Beijing", 21_540_000),
    ]);
    
    // Basic iteration (immutable)
    println!("Cities and their populations:");
    for (city, pop) in &population {
        println!("  {}: {}", city, pop);
    }
    
    // Iterate with indices
    println!("\nNumbered list of cities:");
    for (i, (city, pop)) in population.iter().enumerate() {
        println!("  {}. {} - {}", i + 1, city, pop);
    }
    
    // Iterate over only the keys
    println!("\nList of cities:");
    for city in population.keys() {
        println!("  {}", city);
    }
    
    // Iterate over only the values
    println!("\nList of populations:");
    for pop in population.values() {
        println!("  {}", pop);
    }
    
    // Mutable iteration (modify values)
    println!("\nIncreasing all populations by 10%");
    for (_city, pop) in &mut population {
        *pop = (*pop as f64 * 1.1) as i32;
    }
    
    // Check the modified values
    println!("\nUpdated populations:");
    for (city, pop) in &population {
        println!("  {}: {}", city, pop);
    }
    
    // Sort entries during iteration
    println!("\nCities sorted by name:");
    let mut cities: Vec<(&str, i32)> = population.iter().map(|(&k, &v)| (k, v)).collect();
    cities.sort_by(|a, b| a.0.cmp(b.0));
    
    for (city, pop) in cities {
        println!("  {}: {}", city, pop);
    }
    
    // Sort by values instead
    println!("\nCities sorted by population (descending):");
    let mut by_population: Vec<(&str, i32)> = population.iter().map(|(&k, &v)| (k, v)).collect();
    by_population.sort_by(|a, b| b.1.cmp(&a.1)); // Note the reversed comparison
    
    for (city, pop) in by_population {
        println!("  {}: {}", city, pop);
    }
    
    // Filter entries during iteration
    println!("\nCities with population over 9 million:");
    let large_cities: HashMap<&str, i32> = population
        .iter()
        .filter(|&(_city, &pop)| pop > 9_000_000)
        .map(|(&k, &v)| (k, v))
        .collect();
    
    for (city, pop) in &large_cities {
        println!("  {}: {}", city, pop);
    }
    
    // Transform a hash map into another structure
    let population_in_millions: HashMap<&str, f64> = population
        .iter()
        .map(|(&city, &pop)| (city, pop as f64 / 1_000_000.0))
        .collect();
    
    println!("\nPopulations in millions:");
    for (city, pop) in &population_in_millions {
        println!("  {}: {:.2} million", city, pop);
    }
    
    // Consuming iteration (into_iter)
    println!("\nConsuming iteration:");
    for (city, pop) in population {
        println!("  Final entry: {} with {}", city, pop);
    }
    // population is no longer usable here
    
    // Create a new hash map
    let mut letter_count = HashMap::new();
    letter_count.insert('a', 5);
    letter_count.insert('b', 2);
    letter_count.insert('c', 7);
    
    // Find max value using iteration
    if let Some((max_char, max_count)) = letter_count
        .iter()
        .max_by_key(|&(_k, v)| v)
        .map(|(k, v)| (*k, *v))
    {
        println!("\nMost frequent letter: {} (appears {} times)", max_char, max_count);
    }
    
    // Using iterator methods for aggregation
    let total_count: i32 = letter_count.values().sum();
    println!("Total count of all letters: {}", total_count);
    
    // Check if any entry satisfies a condition
    let has_frequent = letter_count.values().any(|&count| count > 5);
    println!("Has a letter appearing more than 5 times? {}", has_frequent);
}
```

### Real-World Example
An analytics system that processes event data:

```rust
use std::collections::HashMap;
use std::time::{Duration, SystemTime};

#[derive(Debug, Clone)]
struct Event {
    id: String,
    event_type: String,
    user_id: Option<String>,
    timestamp: SystemTime,
    properties: HashMap<String, String>,
}

impl Event {
    fn new(id: &str, event_type: &str, user_id: Option<&str>) -> Self {
        Event {
            id: id.to_string(),
            event_type: event_type.to_string(),
            user_id: user_id.map(|s| s.to_string()),
            timestamp: SystemTime::now(),
            properties: HashMap::new(),
        }
    }
    
    fn add_property(&mut self, key: &str, value: &str) {
        self.properties.insert(key.to_string(), value.to_string());
    }
}

struct Analytics {
    events: Vec<Event>,
}

impl Analytics {
    fn new() -> Self {
        Analytics {
            events: Vec::new(),
        }
    }
    
    fn track(&mut self, event: Event) {
        self.events.push(event);
    }
    
    // Count events by type
    fn event_counts(&self) -> HashMap<String, usize> {
        let mut counts = HashMap::new();
        
        for event in &self.events {
            *counts.entry(event.event_type.clone()).or_insert(0) += 1;
        }
        
        counts
    }
    
    // Get unique users
    fn unique_users(&self) -> usize {
        let user_set: std::collections::HashSet<&String> = self
            .events
            .iter()
            .filter_map(|event| event.user_id.as_ref())
            .collect();
        
        user_set.len()
    }
    
    // Get most common property values for a specific property key
    fn property_value_distribution(&self, property_key: &str) -> HashMap<String, usize> {
        let mut distribution = HashMap::new();
        
        for event in &self.events {
            if let Some(value) = event.properties.get(property_key) {
                *distribution.entry(value.clone()).or_insert(0) += 1;
            }
        }
        
        distribution
    }
    
    // Get user journey (sequence of events) for a specific user
    fn user_journey(&self, user_id: &str) -> Vec<&Event> {
        self.events
            .iter()
            .filter(|event| event.user_id.as_ref().map_or(false, |id| id == user_id))
            .collect()
    }
    
    // Get conversion rate between two event types
    fn conversion_rate(&self, from_event: &str, to_event: &str) -> f64 {
        let user_with_from: std::collections::HashSet<&String> = self
            .events
            .iter()
            .filter(|event| event.event_type == from_event)
            .filter_map(|event| event.user_id.as_ref())
            .collect();
        
        let users_with_both: std::collections::HashSet<&String> = self
            .events
            .iter()
            .filter(|event| event.event_type == to_event)
            .filter_map(|event| event.user_id.as_ref())
            .filter(|user_id| user_with_from.contains(user_id))
            .collect();
        
        if user_with_from.is_empty() {
            0.0
        } else {
            users_with_both.len() as f64 / user_with_from.len() as f64
        }
    }
    
    // Aggregate metrics by time periods
    fn events_by_hour(&self) -> HashMap<u64, usize> {
        let mut counts = HashMap::new();
        let epoch = SystemTime::UNIX_EPOCH;
        
        for event in &self.events {
            if let Ok(duration) = event.timestamp.duration_since(epoch) {
                let hour = duration.as_secs() / 3600;
                *counts.entry(hour).or_insert(0) += 1;
            }
        }
        
        counts
    }
    
    // Print detailed analytics report
    fn print_report(&self) {
        println!("=== Analytics Report ===");
        println!("Total events: {}", self.events.len());
        println!("Unique users: {}", self.unique_users());
        
        println!("\nEvent Counts:");
        let mut event_counts: Vec<(String, usize)> = self.event_counts().into_iter().collect();
        event_counts.sort_by(|a, b| b.1.cmp(&a.1)); // Sort by count (descending)
        
        for (event_type, count) in event_counts {
            println!("  {}: {}", event_type, count);
        }
        
        // Print property distribution for "source" property
        println!("\nTraffic Sources:");
        let sources = self.property_value_distribution("source");
        if !sources.is_empty() {
            let mut sources_vec: Vec<(String, usize)> = sources.into_iter().collect();
            sources_vec.sort_by(|a, b| b.1.cmp(&a.1));
            
            for (source, count) in sources_vec {
                println!("  {}: {}", source, count);
            }
        } else {
            println!("  No source data available");
        }
        
        // Print conversion rate from page_view to signup
        let signup_rate = self.conversion_rate("page_view", "signup");
        println!("\nPage View to Signup conversion rate: {:.2}%", signup_rate * 100.0);
        
        // Print hourly event distribution
        println!("\nEvents by Hour (last 24 hours):");
        let now = SystemTime::now();
        let current_hour = now
            .duration_since(SystemTime::UNIX_EPOCH)
            .unwrap()
            .as_secs() / 3600;
        
        let hourly_counts = self.events_by_hour();
        for hour_offset in 0..24 {
            let hour = current_hour - hour_offset;
            let count = hourly_counts.get(&hour).copied().unwrap_or(0);
            println!("  -{} hours: {} events", hour_offset, count);
        }
    }
}

fn main() {
    let mut analytics = Analytics::new();
    
    // Simulate some events
    let mut event1 = Event::new("e1", "page_view", Some("user1"));
    event1.add_property("page", "/home");
    event1.add_property("source", "google");
    analytics.track(event1);
    
    let mut event2 = Event::new("e2", "signup", Some("user1"));
    event2.add_property("plan", "premium");
    event2.add_property("source", "google");
    analytics.track(event2);
    
    let mut event3 = Event::new("e3", "page_view", Some("user2"));
    event3.add_property("page", "/products");
    event3.add_property("source", "direct");
    analytics.track(event3);
    
    let mut event4 = Event::new("e4", "add_to_cart", Some("user2"));
    event4.add_property("product_id", "123");
    event4.add_property("price", "49.99");
    analytics.track(event4);
    
    let mut event5 = Event::new("e5", "page_view", Some("user3"));
    event5.add_property("page", "/home");
    event5.add_property("source", "twitter");
    analytics.track(event5);
    
    let mut event6 = Event::new("e6", "page_view", None); // Anonymous user
    event6.add_property("page", "/about");
    event6.add_property("source", "facebook");
    analytics.track(event6);
    
    // Print report
    analytics.print_report();
    
    // Get and print user journey for a specific user
    println!("\nUser Journey for user1:");
    let journey = analytics.user_journey("user1");
    for (i, event) in journey.iter().enumerate() {
        println!("  Step {}: {} at {:?}", 
                 i + 1, 
                 event.event_type,
                 event.timestamp);
        
        // Print event properties
        println!("    Properties:");
        for (key, value) in &event.properties {
            println!("      {}: {}", key, value);
        }
    }
}
```

### Common Pitfalls
- **Pitfall**: Expecting consistent iteration order across program runs.
  - **Solution**: Sort the entries if you need a deterministic order.
- **Pitfall**: Mutating a HashMap while iterating over it.
  - **Solution**: Collect the changes to make and apply them after iteration, or use the Entry API where appropriate.
- **Pitfall**: Using `iter` when you need owned values to create a new collection.
  - **Solution**: Use `into_iter` to get owned values when you don't need the original HashMap anymore.
- **Pitfall**: Inefficiently converting between HashMap types.
  - **Solution**: Use iterators with `map` and `collect` for efficient transformations.
- **Pitfall**: Not considering performance when repeatedly iterating the same HashMap.
  - **Solution**: Cache results when iterating repeatedly, or consider transforming the data structure.

### Confusion Questions with Answers
1. **Q**: Why does the order of elements change when I iterate over a HashMap?
   **A**: HashMaps don't guarantee any particular iteration order because they're optimized for fast lookups, not ordering. The order may depend on the hash function, the order of insertions, and even change between program runs. If you need a consistent order, sort the entries or use a `BTreeMap`.

2. **Q**: Can I add or remove elements from a HashMap while iterating over it?
   **A**: No, you can't do this directly due to Rust's borrowing rules. If you need to make changes based on iteration, either collect the changes to make later or use the `retain` method for filtering elements based on a predicate.

3. **Q**: What's the difference between `iter()`, `iter_mut()`, and `into_iter()` for a HashMap?
   **A**: 
   - `iter()` - Provides immutable references to key-value pairs as `(&K, &V)`. The HashMap can be used after iteration.
   - `iter_mut()` - Provides immutable references to keys and mutable references to values as `(&K, &mut V)`. The HashMap can be used after iteration.
   - `into_iter()` - Consumes the HashMap and yields owned key-value pairs as `(K, V)`. The HashMap cannot be used after iteration.

---

## 5. Custom Hash Functions

### Concise Explanation
In Rust, HashMap uses a hashing algorithm to convert keys into indices in its internal data structure. By default, it uses a secure, randomized hashing algorithm that provides protection against hash table collision attacks. However, in some cases, you might want to use a different hashing algorithm, either for performance reasons or to customize the hashing behavior.

Key points:
- The default hasher (`DefaultHasher`) is cryptographically secure but may not be the fastest
- Custom hashers can be used for better performance with trusted inputs
- Types that can be used as HashMap keys must implement the `Hash` trait
- Custom hash functions are implemented by providing a custom `Hasher`
- Common alternatives include FxHasher (faster but not DOS-resistant) and AHash

Custom hashers are particularly useful in performance-critical applications or when you need special behavior like deterministic hashing.

### Where to Use
- Performance-critical applications where hash computation speed matters
- When using keys that have expensive hash functions by default
- For deterministic hashing across program runs
- When working with domain-specific types that need custom hash behavior
- In memory-constrained environments where hash quality vs. speed tradeoffs matter
- For specialized use cases like caching or fingerprinting

### Code Snippet
```rust
use std::collections::HashMap;
use std::hash::{Hash, Hasher, BuildHasher, BuildHasherDefault};
use std::collections::hash_map::DefaultHasher;

// A custom hasher that uses a simple algorithm
#[derive(Default)]
struct SimpleHasher {
    state: u64,
}

impl Hasher for SimpleHasher {
    fn finish(&self) -> u64 {
        self.state
    }
    
    fn write(&mut self, bytes: &[u8]) {
        for &b in bytes {
            self.state = self.state.wrapping_mul(31).wrapping_add(b as u64);
        }
    }
}

// A build hasher for our simple hasher
#[derive(Default, Clone)]
struct SimpleHasherBuilder;

impl BuildHasher for SimpleHasherBuilder {
    type Hasher = SimpleHasher;
    
    fn build_hasher(&self) -> SimpleHasher {
        SimpleHasher::default()
    }
}

// A struct with a custom hash implementation
struct CustomKey {
    id: u64,
    name: String,
}

impl PartialEq for CustomKey {
    fn eq(&self, other: &Self) -> bool {
        self.id == other.id // Equality based on ID only
    }
}

impl Eq for CustomKey {}

impl Hash for CustomKey {
    fn hash<H: Hasher>(&self, state: &mut H) {
        // Only hash the ID, ignore the name
        self.id.hash(state);
    }
}

fn main() {
    // Regular HashMap with default hasher
    let mut default_map: HashMap<String, i32> = HashMap::new();
    default_map.insert("one".to_string(), 1);
    default_map.insert("two".to_string(), 2);
    println!("Default HashMap: {:?}", default_map);
    
    // HashMap with a custom hasher
    let mut simple_map: HashMap<String, i32, SimpleHasherBuilder> = 
        HashMap::with_hasher(SimpleHasherBuilder);
    simple_map.insert("one".to_string(), 1);
    simple_map.insert("two".to_string(), 2);
    println!("HashMap with simple hasher: {:?}", simple_map);
    
    // Compare hashes of the same string with different hashers
    let mut default_hasher = DefaultHasher::new();
    "test string".hash(&mut default_hasher);
    let default_hash = default_hasher.finish();
    
    let mut simple_hasher = SimpleHasher::default();
    "test string".hash(&mut simple_hasher);
    let simple_hash = simple_hasher.finish();
    
    println!("DefaultHasher hash: {:x}", default_hash);
    println!("SimpleHasher hash: {:x}", simple_hash);
    
    // Using a custom key type with custom hash implementation
    let key1 = CustomKey {
        id: 1,
        name: "First Item".to_string(),
    };
    
    let key2 = CustomKey {
        id: 1, // Same ID
        name: "Different Name".to_string(), // Different name
    };
    
    let mut custom_map: HashMap<CustomKey, String> = HashMap::new();
    custom_map.insert(key1, "Value 1".to_string());
    
    // This will overwrite the previous value because the keys are considered equal
    // (they have the same ID, and we defined equality based on ID only)
    custom_map.insert(key2, "Value 2".to_string());
    
    println!("Map size after inserting two 'equal' keys: {}", custom_map.len());
    
    // Using Type aliases for HashMaps with custom hashers
    type FastMap<K, V> = HashMap<K, V, SimpleHasherBuilder>;
    
    let mut fast_map: FastMap<i32, &str> = FastMap::default();
    fast_map.insert(1, "one");
    fast_map.insert(2, "two");
    println!("Fast map: {:?}", fast_map);
    
    // Compare lookup performance
    use std::time::Instant;
    
    const N: usize = 100_000;
    
    // Create maps with a lot of elements
    let mut default_big: HashMap<String, i32> = HashMap::with_capacity(N);
    let mut simple_big: HashMap<String, i32, SimpleHasherBuilder> = 
        HashMap::with_capacity_and_hasher(N, SimpleHasherBuilder);
    
    for i in 0..N {
        let key = format!("key_{}", i);
        default_big.insert(key.clone(), i as i32);
        simple_big.insert(key, i as i32);
    }
    
    // Benchmark lookups (Note: not a proper benchmark, just for illustration)
    let start = Instant::now();
    for i in 0..N {
        let key = format!("key_{}", i);
        default_big.get(&key);
    }
    let default_time = start.elapsed();
    
    let start = Instant::now();
    for i in 0..N {
        let key = format!("key_{}", i);
        simple_big.get(&key);
    }
    let simple_time = start.elapsed();
    
    println!("\nLookup times for {} elements:", N);
    println!("  DefaultHasher: {:?}", default_time);
    println!("  SimpleHasher: {:?}", simple_time);
    
    // Note: In real applications, consider using existing alternatives like FxHasher
    // which are well-tested and optimized
}
```

### Real-World Example
A caching system with customized hashing for domain objects:

```rust
use std::collections::HashMap;
use std::hash::{Hash, Hasher, BuildHasher};
use std::time::{Duration, Instant};

// FastHasher is a wrapper around a faster, non-cryptographic hash function
// In a real application, you might use FxHasher or AHash
// Here we'll simulate one with a simple implementation
#[derive(Default)]
struct FastHasher {
    state: u64,
}

impl Hasher for FastHasher {
    fn finish(&self) -> u64 {
        self.state
    }
    
    fn write(&mut self, bytes: &[u8]) {
        for &b in bytes {
            // Simple FNV-1a style hash
            self.state ^= b as u64;
            self.state = self.state.wrapping_mul(0x100000001b3);
        }
    }
}

#[derive(Clone, Default)]
struct FastHasherBuilder;

impl BuildHasher for FastHasherBuilder {
    type Hasher = FastHasher;
    
    fn build_hasher(&self) -> FastHasher {
        FastHasher::default()
    }
}

// Domain object: A URL with custom hash implementation
#[derive(Clone, Debug, Eq)]
struct Url {
    protocol: String,
    domain: String,
    path: String,
    query_params: HashMap<String, String>,
}

impl Url {
    fn new(url_str: &str) -> Option<Self> {
        // Very basic URL parser for demonstration
        let parts: Vec<&str> = url_str.splitn(2, "://").collect();
        if parts.len() < 2 {
            return None;
        }
        
        let protocol = parts[0].to_lowercase();
        let remaining = parts[1];
        
        let domain_path: Vec<&str> = remaining.splitn(2, '/').collect();
        let domain = domain_path[0].to_lowercase();
        
        let path_query: Vec<&str> = if domain_path.len() > 1 {
            domain_path[1].splitn(2, '?').collect()
        } else {
            vec![""]
        };
        
        let path = if path_query[0].is_empty() {
            "/".to_string()
        } else {
            format!("/{}", path_query[0])
        };
        
        let mut query_params = HashMap::new();
        if path_query.len() > 1 {
            for param in path_query[1].split('&') {
                let kv: Vec<&str> = param.splitn(2, '=').collect();
                if kv.len() == 2 {
                    query_params.insert(kv[0].to_lowercase(), kv[1].to_string());
                }
            }
        }
        
        Some(Url {
            protocol,
            domain,
            path,
            query_params,
        })
    }
    
    fn to_string(&self) -> String {
        let mut result = format!("{}://{}{}", self.protocol, self.domain, self.path);
        
        if !self.query_params.is_empty() {
            result.push('?');
            for (i, (key, value)) in self.query_params.iter().enumerate() {
                if i > 0 {
                    result.push('&');
                }
                result.push_str(&format!("{}={}", key, value));
            }
        }
        
        result
    }
}

// PartialEq implementation that ignores the order of query parameters
impl PartialEq for Url {
    fn eq(&self, other: &Self) -> bool {
        self.protocol == other.protocol &&
        self.domain == other.domain &&
        self.path == other.path &&
        self.query_params == other.query_params
    }
}

// Hash implementation that produces the same hash regardless of query param order
impl Hash for Url {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.protocol.hash(state);
        self.domain.hash(state);
        self.path.hash(state);
        
        // Sort the query params by key to ensure consistent hashing
        let mut sorted_params: Vec<(&String, &String)> = self.query_params.iter().collect();
        sorted_params.sort_by(|a, b| a.0.cmp(b.0));
        
        // Hash the sorted params
        for (key, value) in sorted_params {
            key.hash(state);
            value.hash(state);
        }
    }
}

// A cache entry with expiration
struct CacheEntry<T> {
    value: T,
    expires_at: Instant,
}

// URL cache using custom hasher for better performance
struct UrlCache<T> {
    cache: HashMap<Url, CacheEntry<T>, FastHasherBuilder>,
    default_ttl: Duration,
}

impl<T: Clone> UrlCache<T> {
    fn new(default_ttl_secs: u64) -> Self {
        UrlCache {
            cache: HashMap::with_hasher(FastHasherBuilder),
            default_ttl: Duration::from_secs(default_ttl_secs),
        }
    }
    
    fn get(&mut self, url: &Url) -> Option<T> {
        // Remove expired entries when accessing
        self.cleanup();
        
        self.cache.get(url).map(|entry| {
            if entry.expires_at > Instant::now() {
                Some(entry.value.clone())
            } else {
                None
            }
        }).flatten()
    }
    
    fn set(&mut self, url: Url, value: T) {
    fn set_with_ttl(&mut self, url: Url, value: T, ttl: Duration) {
        let expires_at = Instant::now() + ttl;
        self.cache.insert(url, CacheEntry { value, expires_at });
    }
    
    fn cleanup(&mut self) {
        let now = Instant::now();
        self.cache.retain(|_, entry| entry.expires_at > now);
    }
    
    fn remove(&mut self, url: &Url) -> Option<T> {
        self.cache.remove(url).map(|entry| entry.value)
    }
    
    fn clear(&mut self) {
        self.cache.clear();
    }
    
    fn len(&self) -> usize {
        self.cache.len()
    }
    
    fn is_empty(&self) -> bool {
        self.cache.is_empty()
    }
}

// A fetcher trait for getting URL content
trait UrlFetcher<T> {
    fn fetch(&self, url: &Url) -> Result<T, String>;
}

// Example URL content fetcher implementation
struct WebContentFetcher;

impl UrlFetcher<String> for WebContentFetcher {
    fn fetch(&self, url: &Url) -> Result<String, String> {
        // In a real implementation, this would make an HTTP request
        // For this example, we'll just simulate a response
        println!("Fetching content from {}", url.to_string());
        
        // Simulate network delay
        std::thread::sleep(Duration::from_millis(200));
        
        // Generate some fake content based on the URL
        let content = format!("Content for {} - Generated at {:?}", 
                             url.to_string(), 
                             std::time::SystemTime::now());
        
        Ok(content)
    }
}

// A caching URL fetcher that uses our custom cache
struct CachingUrlFetcher<T, F: UrlFetcher<T>> {
    cache: UrlCache<T>,
    fetcher: F,
}

impl<T: Clone, F: UrlFetcher<T>> CachingUrlFetcher<T, F> {
    fn new(fetcher: F, ttl_secs: u64) -> Self {
        CachingUrlFetcher {
            cache: UrlCache::new(ttl_secs),
            fetcher,
        }
    }
    
    fn get_content(&mut self, url_str: &str) -> Result<T, String> {
        let url = Url::new(url_str).ok_or_else(|| format!("Invalid URL: {}", url_str))?;
        
        // Try to get from cache first
        if let Some(content) = self.cache.get(&url) {
            println!("Cache hit for {}", url_str);
            return Ok(content);
        }
        
        // Not in cache, fetch it
        println!("Cache miss for {}", url_str);
        let content = self.fetcher.fetch(&url)?;
        
        // Store in cache
        self.cache.set(url, content.clone());
        
        Ok(content)
    }
    
    fn cache_stats(&self) -> (usize, usize) {
        (self.cache.len(), self.cache.cache.capacity())
    }
}

fn main() {
    // Create a caching URL fetcher
    let mut fetcher = CachingUrlFetcher::new(
        WebContentFetcher,
        60, // 60 second TTL
    );
    
    // Make some requests
    println!("First request:");
    let content1 = fetcher.get_content("https://example.com/page?id=123&user=smith").unwrap();
    println!("Content length: {} bytes\n", content1.len());
    
    println!("Second request (same URL, should be cached):");
    let content2 = fetcher.get_content("https://example.com/page?id=123&user=smith").unwrap();
    println!("Content length: {} bytes\n", content2.len());
    
    // Same URL but different query param order (should still hit cache due to our custom hash)
    println!("Third request (same URL, different param order):");
    let content3 = fetcher.get_content("https://example.com/page?user=smith&id=123").unwrap();
    println!("Content length: {} bytes\n", content3.len());
    
    // Different URL
    println!("Fourth request (different URL):");
    let content4 = fetcher.get_content("https://example.com/another-page").unwrap();
    println!("Content length: {} bytes\n", content4.len());
    
    // Print cache stats
    let (size, capacity) = fetcher.cache_stats();
    println!("Cache size: {}, capacity: {}", size, capacity);
    
    // Benchmark different hashers for URL objects
    use std::collections::hash_map::DefaultHasher;
    use std::hash::{BuildHasherDefault, Hasher};
    
    let urls = [
        "https://example.com/page1?id=123&user=smith",
        "https://example.com/page2?category=books&sort=price",
        "https://shop.example.com/products?id=456&color=red",
        "https://api.example.com/v1/users/123?fields=name,email",
        "https://example.com/search?q=rust+programming&page=1",
    ];
    
    let parsed_urls: Vec<Url> = urls.iter()
        .filter_map(|url_str| Url::new(url_str))
        .collect();
    
    // Measure hashing time with default hasher
    let start = Instant::now();
    let mut default_hasher = DefaultHasher::new();
    for url in &parsed_urls {
        url.hash(&mut default_hasher);
        let _ = default_hasher.finish();
        default_hasher = DefaultHasher::new();
    }
    let default_time = start.elapsed();
    
    // Measure hashing time with fast hasher
    let start = Instant::now();
    let mut fast_hasher = FastHasher::default();
    for url in &parsed_urls {
        url.hash(&mut fast_hasher);
        let _ = fast_hasher.finish();
        fast_hasher = FastHasher::default();
    }
    let fast_time = start.elapsed();
    
    println!("\nURL Hashing Performance:");
    println!("  Default hasher: {:?}", default_time);
    println!("  Fast hasher: {:?}", fast_time);
    println!("  Speed improvement: {:.2}x", default_time.as_nanos() as f64 / fast_time.as_nanos() as f64);
}
```

### Common Pitfalls
- **Pitfall**: Using custom hashers without understanding their security implications.
  - **Solution**: Only use non-cryptographic hashers like FxHasher for trusted inputs, not for public-facing applications.
- **Pitfall**: Implementing Hash inconsistently with PartialEq/Eq.
  - **Solution**: Ensure that if two objects are equal, they produce the same hash value.
- **Pitfall**: Creating inefficient hash functions that cause excessive collisions.
  - **Solution**: Ensure your hash function distributes values well across the hash space.
- **Pitfall**: Using mutable values as HashMap keys.
  - **Solution**: Only use immutable values as keys, as changing a key's hashable content after insertion can make it unfindable.
- **Pitfall**: Reimplementing existing optimized hashers unnecessarily.
  - **Solution**: Use existing libraries like FxHasher, AHash, or NoHashHasher from standard crates.

### Confusion Questions with Answers
1. **Q**: When should I use a custom hasher instead of the default one?
   **A**: Use a custom hasher when:
   - You need better performance and your application doesn't face untrusted inputs
   - You need deterministic hashing (the same hash across program runs)
   - You have domain-specific knowledge that could make hashing more efficient
   - You're working in a memory-constrained environment and need to balance speed vs. space

2. **Q**: How does implementing a custom Hash trait affect performance?
   **A**: Implementing a custom Hash trait allows you to:
   - Focus on hashing only the relevant fields of your type
   - Ignore fields that don't affect equality
   - Create more efficient hash algorithms for your specific data types
   - Ensure hash consistency even when data representation varies (like query parameters in different orders)

3. **Q**: Is it safe to use non-cryptographic hashers like FxHasher in web applications?
   **A**: It depends on your threat model. Non-cryptographic hashers are vulnerable to hash-DoS attacks, where an attacker deliberately creates keys that hash to the same value, degrading hash table performance. For public-facing web services that accept arbitrary user input as keys, stick with the default hasher. For internal applications or where input is trusted, faster hashers like FxHasher can be used safely.

---

## Next Actions: Exercises and Projects

### Exercises
1. **HashMap Operations Practice**
   - Create a function to merge two hash maps, handling conflicts gracefully
   - Implement a function that inverts a hash map (swapping keys and values)
   - Write a function to find keys in one hash map that aren't in another
   - Create a function that groups values in a vector by a computed property

2. **Entry API Challenge**
   - Implement a frequency counter for words in a text using the Entry API
   - Create a cache that evicts least recently used items when it reaches capacity
   - Implement a function that aggregates values by key from multiple sources
   - Write a function that tracks the history of changes to a value in a hash map

3. **Custom Hashing Workshop**
   - Create a struct with a custom Hash implementation that ignores certain fields
   - Implement a hash map with case-insensitive string keys
   - Write a benchmark comparing different hashers for various types of keys
   - Create a custom hasher that focuses on minimizing collisions for a specific domain

4. **HashMap Performance Optimization**
   - Benchmark different ways of bulking constructing hash maps
   - Compare the performance of hash maps vs. vectors for various operations
   - Optimize a hash map-intensive application by choosing appropriate capacities
   - Compare custom hashers for different workloads

### Micro-Project: Simple Key-Value Database
Build a simple in-memory key-value database that:
- Supports CRUD operations (Create, Read, Update, Delete)
- Implements transaction support with commit and rollback
- Uses the Entry API for efficient operations
- Allows for custom key types with specialized hash functions
- Provides statistics and performance monitoring

```rust
// Example starter code for key-value database micro-project
use std::collections::HashMap;
use std::hash::Hash;

enum TransactionOp<K, V> {
    Insert(K, V),
    Remove(K),
}

struct Transaction<K, V> {
    operations: Vec<TransactionOp<K, V>>,
}

struct KeyValueStore<K, V> {
    data: HashMap<K, V>,
    transaction_stack: Vec<Transaction<K, V>>,
}

impl<K, V> KeyValueStore<K, V>
where
    K: Eq + Hash + Clone,
    V: Clone,
{
    fn new() -> Self {
        KeyValueStore {
            data: HashMap::new(),
            transaction_stack: Vec::new(),
        }
    }
    
    fn get(&self, key: &K) -> Option<&V> {
        self.data.get(key)
    }
    
    fn insert(&mut self, key: K, value: V) -> Option<V> {
        if !self.transaction_stack.is_empty() {
            // Inside a transaction, record the operation
            let current_tx = self.transaction_stack.last_mut().unwrap();
            current_tx.operations.push(TransactionOp::Insert(key.clone(), value.clone()));
        }
        
        // Apply the change
        self.data.insert(key, value)
    }
    
    fn remove(&mut self, key: &K) -> Option<V> {
        if !self.transaction_stack.is_empty() {
            // Inside a transaction, record the operation
            if let Some(value) = self.data.get(key) {
                let current_tx = self.transaction_stack.last_mut().unwrap();
                current_tx.operations.push(TransactionOp::Remove(key.clone()));
            }
        }
        
        // Apply the change
        self.data.remove(key)
    }
    
    fn begin_transaction(&mut self) {
        self.transaction_stack.push(Transaction { operations: Vec::new() });
    }
    
    fn commit_transaction(&mut self) -> Result<(), &'static str> {
        if self.transaction_stack.is_empty() {
            return Err("No transaction to commit");
        }
        
        self.transaction_stack.pop();
        Ok(())
    }
    
    fn rollback_transaction(&mut self) -> Result<(), &'static str> {
        if self.transaction_stack.is_empty() {
            return Err("No transaction to rollback");
        }
        
        // Pop the transaction and reverse its operations
        if let Some(tx) = self.transaction_stack.pop() {
            for op in tx.operations.into_iter().rev() {
                match op {
                    TransactionOp::Insert(key, _) => {
                        self.data.remove(&key);
                    },
                    TransactionOp::Remove(key) => {
                        // This would be more complex in a real implementation
                        // as we would need to store the original values
                    },
                }
            }
        }
        
        Ok(())
    }
    
    // Implement more functionality here...
}

// Complete the implementation with more features and transaction support
```

## Success Criteria
You've mastered hash maps in Rust when you can:
1. Choose appropriate capacity and hasher settings for performance
2. Effectively use the Entry API for compound operations
3. Apply hash maps for common patterns like caching, counting, and lookups
4. Create custom hash implementations for complex types
5. Understand and manage HashMap iteration patterns
6. Optimize HashMap usage for your specific application needs
7. Implement specialized hash functions when needed

## Troubleshooting Advice
1. **HashMap Key Constraints**
   - Remember keys must implement `Hash` and `Eq` traits
   - Don't use types with custom equality that don't have corresponding hash implementations
   - Consider deriving `Hash` and `Eq` for simple struct keys
   - Only use immutable fields in hash calculations

2. **Performance Issues**
   - Preallocate with appropriate capacity when the approximate size is known
   - Consider specialized hasher implementations for performance-critical code
   - Profile to identify bottlenecks in hash calculations or lookups
   - Keep hash keys small and simple when possible

3. **Iteration Challenges**
   - Don't assume consistent iteration order across program runs
   - Avoid borrowing conflicts by not modifying a HashMap during iteration
   - Consider collecting keys/values first if you need to modify during iteration
   - Sort the entries if you need a predictable order

4. **Entry API Confusion**
   - Remember that `or_insert` returns a mutable reference
   - Use `and_modify` followed by `or_insert` to handle both cases in one call
   - Understand that Entry's purpose is to avoid redundant lookups
   - Use `or_insert_with` for values that are expensive to compute

5. **Hash Function Problems**
   - Ensure hash functions are consistent with equality comparison
   - Watch for hash collisions that can degrade performance
   - Use cryptographically secure hashers for untrusted inputs
   - Don't recompute hashes unnecessarily; cache when appropriate
