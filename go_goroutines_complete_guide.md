# Complete Go Goroutines Guide: From Basics to Advanced Concurrency Mastery

## TABLE OF CONTENTS
1. **Fundamentals & Theory**
2. **Goroutines (Basic to Advanced)**
3. **Channels (Unbuffered & Buffered)**
4. **Synchronization Primitives (Mutex, RWMutex, WaitGroup, Once)**
5. **Concurrency Patterns**
6. **Context & Cancellation**
7. **Common Pitfalls & Prevention**
8. **Production-Ready Examples**

---

## PART 1: FUNDAMENTALS & THEORY

### 1.1 What is Concurrency?

**Concurrency vs Parallelism:**
- **Concurrency**: Multiple tasks making progress (can be on single core by interleaving)
- **Parallelism**: Multiple tasks running simultaneously (requires multiple cores)

**Go's Model:**
- Go uses **M:N scheduling** - Multiple goroutines (M) mapped to multiple OS threads (N)
- Goroutines are **lightweight** - thousands can run without exhausting resources
- Go runtime **scheduler** manages goroutine distribution across CPU cores

### 1.2 Key Concurrency Primitives

```
Go's Concurrency Toolkit:
├── Goroutines: Lightweight concurrent functions
├── Channels: Type-safe communication between goroutines
├── Select: Multiplex channel operations
├── Sync Package:
│   ├── Mutex: Binary lock (locked/unlocked)
│   ├── RWMutex: Reader-Writer lock
│   ├── WaitGroup: Wait for goroutine collection
│   ├── Once: Execute code exactly once
│   └── Cond: Conditional variable
└── Context: Cancellation, deadlines, values
```

---

## PART 2: GOROUTINES - COMPLETE GUIDE

### 2.1 BASIC GOROUTINE

```go
package main

import (
	"fmt"      // For output
	"time"     // For sleep/timing
)

func main() {
	// Simple function
	greet := func(name string) {
		for i := 0; i < 3; i++ {
			fmt.Printf("Hello %s (%d)\n", name, i)
			time.Sleep(100 * time.Millisecond)
		}
	}
	
	// Sequential execution (blocking)
	// Takes ~600ms
	greet("Alice")
	greet("Bob")
	
	fmt.Println("Done")
}

// Output:
// Hello Alice (0)
// Hello Alice (1)
// Hello Alice (2)
// Hello Bob (0)
// Hello Bob (1)
// Hello Bob (2)
// Done
```

---

### 2.2 CONCURRENT EXECUTION WITH GOROUTINES

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	greet := func(name string) {
		for i := 0; i < 3; i++ {
			fmt.Printf("Hello %s (%d)\n", name, i)
			time.Sleep(100 * time.Millisecond)
		}
	}
	
	// Launch goroutines with 'go' keyword
	// go: Keyword that starts new lightweight thread
	// These don't block, execution continues immediately
	go greet("Alice")
	go greet("Bob")
	
	// Main goroutine waits so child goroutines have time to run
	time.Sleep(500 * time.Millisecond)
	
	fmt.Println("Done")
}

// Output (interleaved):
// Hello Alice (0)
// Hello Bob (0)
// Hello Alice (1)
// Hello Bob (1)
// Hello Alice (2)
// Hello Bob (2)
// Done
```

**Key Differences:**
- ✅ `go greet()` starts function in background
- ✅ Execution continues immediately (non-blocking)
- ✅ Main function doesn't wait for goroutines automatically
- ✅ Program exits when main() returns (kills child goroutines)

---

### 2.3 GOROUTINE LIFECYCLE

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int, name string) {
	// Function body executes in goroutine
	fmt.Printf("[%d] Starting\n", id)
	
	// Do work
	time.Sleep(2 * time.Second)
	
	// Function return = goroutine terminates
	fmt.Printf("[%d] Finished\n", id)
}

func main() {
	// Start goroutine
	go worker(1, "Task-A")
	go worker(2, "Task-B")
	
	// Main exits after 1 second
	time.Sleep(1 * time.Second)
	fmt.Println("Main exiting...")
}

// Output:
// [1] Starting
// [2] Starting
// Main exiting...
// (workers are killed before finishing)
```

**Problem**: Goroutines didn't finish! Need to wait properly.

---

### 2.4 WAITING FOR GOROUTINES - sync.WaitGroup

```go
package main

import (
	"fmt"
	"sync"    // Synchronization package
	"time"
)

func worker(id int, wg *sync.WaitGroup) {
	// Mark this goroutine (wg.Done() called when exiting)
	defer wg.Done()
	
	fmt.Printf("[%d] Starting\n", id)
	time.Sleep(2 * time.Second)
	fmt.Printf("[%d] Finished\n", id)
}

func main() {
	// WaitGroup: Counter for goroutines
	var wg sync.WaitGroup
	
	numWorkers := 3
	
	// Add 3 to internal counter (tells WaitGroup to expect 3 goroutines)
	wg.Add(numWorkers)
	
	for i := 1; i <= numWorkers; i++ {
		// Pass &wg to goroutine
		go worker(i, &wg)
	}
	
	// Wait: Blocks until counter reaches 0
	// Each wg.Done() decrements counter
	wg.Wait()
	
	fmt.Println("All workers finished!")
}

// Output:
// [1] Starting
// [2] Starting
// [3] Starting
// [1] Finished
// [2] Finished
// [3] Finished
// All workers finished!
```

**WaitGroup Mechanics:**
```
wg.Add(3)      → Counter = 3
defer wg.Done() → Counter = 2, 1, 0 (when goroutine returns)
wg.Wait()      → Blocks until counter = 0
```

---

### 2.5 GOROUTINE PANICS

```go
package main

import (
	"fmt"
	"sync"
)

func riskyTask(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	
	if id == 2 {
		// Panic in goroutine ONLY crashes that goroutine
		panic("Something went wrong!")
	}
	
	fmt.Printf("Task %d completed\n", id)
}

func main() {
	var wg sync.WaitGroup
	
	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go riskyTask(i, &wg)
	}
	
	// If goroutine panics, wg.Wait() still completes eventually
	// But main program continues despite panic in goroutine
	wg.Wait()
	fmt.Println("Done")
}

// Output:
// Task 1 completed
// panic: Something went wrong!
// [goroutine 7 panics, but program continues]
// Task 3 completed
// Done
```

**Important**: Goroutine panics don't crash entire program, only that goroutine!

---

### 2.6 GOROUTINE LEAKS

```go
// ✗ BAD - GOROUTINE LEAK
package main

import "time"

func main() {
	ch := make(chan int) // Channel with no sender/receiver
	
	go func() {
		// This goroutine waits forever for data on channel
		// But main never sends anything
		data := <-ch
		println(data)
	}()
	
	// Main exits, but goroutine waits forever (memory leak!)
	time.Sleep(1 * time.Second)
}

// ✓ GOOD - PROPER CLEANUP
package main

import (
	"fmt"
	"time"
)

func main() {
	done := make(chan struct{}) // Signal channel (no data, just done)
	
	go func() {
		fmt.Println("Goroutine working...")
		time.Sleep(1 * time.Second)
		
		// Send signal on done channel
		done <- struct{}{}
	}()
	
	// Wait for goroutine to signal completion
	<-done
	
	fmt.Println("Goroutine finished cleanly")
}

// Output:
// Goroutine working...
// Goroutine finished cleanly
```

---

## PART 3: CHANNELS - COMPLETE GUIDE

### 3.1 CHANNEL BASICS

```go
package main

import "fmt"

func main() {
	// Declare channel for integers
	// chan int: Channel that sends/receives integers
	var ch chan int
	
	// Create channel
	// make(chan int): Unbuffered channel
	ch = make(chan int)
	
	// Send value (blocks until receiver ready)
	// ch <- 42: Send 42 to channel
	go func() {
		ch <- 42
	}()
	
	// Receive value (blocks until sender sends)
	// value := <-ch: Receive from channel
	value := <-ch
	
	fmt.Printf("Received: %d\n", value)
}

// Output:
// Received: 42
```

**Channel Operations:**
```
make(chan T)          → Create unbuffered channel
ch <- value           → Send (blocks until received)
value := <-ch         → Receive (blocks until sent)
close(ch)             → Close channel (can't send after)
value, ok := <-ch     → Receive with ok check
```

---

### 3.2 UNBUFFERED CHANNELS (Synchronous)

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// Unbuffered: No internal buffer
	ch := make(chan string)
	
	// Sender goroutine
	go func() {
		fmt.Println("[Sender] Sending 'hello'...")
		ch <- "hello"     // Blocks here until receiver ready!
		fmt.Println("[Sender] Sent 'hello'")
		
		ch <- "world"     // Blocks again
		fmt.Println("[Sender] Sent 'world'")
	}()
	
	// Simulate slow receiver
	time.Sleep(2 * time.Second)
	
	fmt.Println("[Receiver] Receiving...")
	msg1 := <-ch        // Receives "hello", sender continues
	fmt.Printf("[Receiver] Got: %s\n", msg1)
	
	msg2 := <-ch        // Receives "world"
	fmt.Printf("[Receiver] Got: %s\n", msg2)
}

// Output:
// [Sender] Sending 'hello'...
// [Receiver] Receiving...
// [Sender] Sent 'hello'
// [Receiver] Got: hello
// [Sender] Sending 'world'...
// [Sender] Sent 'world'
// [Receiver] Got: world
```

**Key Points:**
- Sender blocks until receiver is ready
- Receiver blocks until sender sends
- Acts as synchronization point (handshake)

---

### 3.3 BUFFERED CHANNELS (Asynchronous)

```go
package main

import "fmt"

func main() {
	// Buffered: Internal buffer of 2
	ch := make(chan string, 2)
	
	// Sender goroutine
	go func() {
		fmt.Println("[Sender] Sending 'msg1'...")
		ch <- "msg1"      // Stored in buffer, doesn't block
		fmt.Println("[Sender] Stored 'msg1'")
		
		fmt.Println("[Sender] Sending 'msg2'...")
		ch <- "msg2"      // Stored in buffer, doesn't block
		fmt.Println("[Sender] Stored 'msg2'")
		
		fmt.Println("[Sender] Sending 'msg3'...")
		ch <- "msg3"      // BLOCKS - buffer full!
		fmt.Println("[Sender] Stored 'msg3'")
	}()
	
	// Simulate slow receiver
	time.Sleep(2 * time.Second)
	
	fmt.Println("[Receiver] Receiving...")
	for i := 0; i < 3; i++ {
		msg := <-ch    // Removes from buffer
		fmt.Printf("[Receiver] Got: %s\n", msg)
	}
}

// Output:
// [Sender] Sending 'msg1'...
// [Sender] Stored 'msg1'
// [Sender] Sending 'msg2'...
// [Sender] Stored 'msg2'
// [Sender] Sending 'msg3'...
// (blocks here, waits for receiver)
// [Receiver] Receiving...
// [Receiver] Got: msg1
// [Sender] Stored 'msg3'
// [Receiver] Got: msg2
// [Receiver] Got: msg3
```

---

### 3.4 UNBUFFERED vs BUFFERED COMPARISON

```go
package main

import (
	"fmt"
	"time"
)

func demonstrateUnbuffered() {
	fmt.Println("\n=== UNBUFFERED ===")
	ch := make(chan int)
	
	go func() {
		for i := 1; i <= 5; i++ {
			fmt.Printf("Sending %d...\n", i)
			ch <- i        // Blocks until received
			fmt.Printf("Sent %d\n", i)
		}
		close(ch)
	}()
	
	time.Sleep(1 * time.Second)
	for val := range ch {
		fmt.Printf("Received %d\n", val)
	}
}

func demonstrateBuffered() {
	fmt.Println("\n=== BUFFERED (size=2) ===")
	ch := make(chan int, 2)
	
	go func() {
		for i := 1; i <= 5; i++ {
			fmt.Printf("Sending %d...\n", i)
			ch <- i        // Doesn't block if buffer has space
			fmt.Printf("Sent %d\n", i)
		}
		close(ch)
	}()
	
	time.Sleep(1 * time.Second)
	for val := range ch {
		fmt.Printf("Received %d\n", val)
	}
}

func main() {
	demonstrateUnbuffered()
	demonstrateBuffered()
}
```

**Comparison Table:**
| Feature | Unbuffered | Buffered |
|---------|-----------|----------|
| Capacity | 0 | N > 0 |
| Send Blocks | Always until received | Only if full |
| Receive Blocks | Always until sent | Only if empty |
| Synchronization | Strict (rendezvous) | Decoupled |
| Best For | Handshakes, tight sync | Producer-consumer, rate limiting |

---

### 3.5 CLOSING CHANNELS

```go
package main

import "fmt"

func main() {
	ch := make(chan int)
	
	// Sender
	go func() {
		for i := 1; i <= 3; i++ {
			ch <- i
		}
		
		// Close channel (only sender closes!)
		close(ch)
	}()
	
	// Receiver
	// When channel closes, receiver gets zero value and ok=false
	for {
		val, ok := <-ch
		if !ok {
			fmt.Println("Channel closed")
			break
		}
		fmt.Printf("Received: %d\n", val)
	}
}

// Output:
// Received: 1
// Received: 2
// Received: 3
// Channel closed
```

**Closing Rules:**
- ✓ Only sender closes (prevents panic)
- ✓ Sending on closed channel → panic
- ✓ Closing already closed channel → panic
- ✓ Receiving from closed channel → returns zero value, ok=false

---

### 3.6 RANGE OVER CHANNELS

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 3)
	
	// Send values
	ch <- 10
	ch <- 20
	ch <- 30
	close(ch)
	
	// range automatically stops when channel closes
	for value := range ch {
		fmt.Printf("Value: %d\n", value)
	}
	
	fmt.Println("Loop ended")
}

// Output:
// Value: 10
// Value: 20
// Value: 30
// Loop ended
```

---

### 3.7 SELECT STATEMENT

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	// Select: Wait on multiple channel operations
	ch1 := make(chan string)
	ch2 := make(chan string)
	
	go func() {
		time.Sleep(1 * time.Second)
		ch1 <- "Result from ch1"
	}()
	
	go func() {
		time.Sleep(500 * time.Millisecond)
		ch2 <- "Result from ch2"
	}()
	
	// Select waits for first ready channel
	for i := 0; i < 2; i++ {
		select {
		case msg1 := <-ch1:
			// Executes when ch1 has data
			fmt.Println(msg1)
			
		case msg2 := <-ch2:
			// Executes when ch2 has data
			fmt.Println(msg2)
			
		case <-time.After(2 * time.Second):
			// Executes if timeout
			fmt.Println("Timeout!")
		}
	}
}

// Output:
// Result from ch2 (first, came after 500ms)
// Result from ch1 (second, came after 1000ms)
```

**Select Mechanics:**
- Waits for first channel ready
- Each case is a channel operation
- Default case executes if no channels ready (non-blocking)
- Exits when one case completes

---

### 3.8 SELECT WITH DEFAULT

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)
	
	go func() {
		time.Sleep(2 * time.Second)
		ch <- "Data ready"
	}()
	
	// Non-blocking check
	select {
	case data := <-ch:
		fmt.Println(data)
	default:
		// Executes if ch not ready (non-blocking)
		fmt.Println("Channel not ready, doing something else")
	}
	
	// Wait for data
	time.Sleep(3 * time.Second)
	
	select {
	case data := <-ch:
		// Now channel is ready
		fmt.Println(data)
	default:
		fmt.Println("Still not ready")
	}
}

// Output:
// Channel not ready, doing something else
// Data ready
```

---

## PART 4: SYNCHRONIZATION PRIMITIVES

### 4.1 MUTEX - PROTECTING SHARED STATE

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Bank account with protected balance
type Account struct {
	balance int
	mu      sync.Mutex  // Protects balance
}

func (acc *Account) Deposit(amount int) {
	// Lock: Only one goroutine can execute here at a time
	acc.mu.Lock()
	defer acc.mu.Unlock()  // Unlock when function returns
	
	fmt.Printf("[Deposit] Current: %d, Adding: %d\n", acc.balance, amount)
	temp := acc.balance
	time.Sleep(10 * time.Millisecond)  // Simulate processing
	acc.balance = temp + amount
	fmt.Printf("[Deposit] New balance: %d\n", acc.balance)
}

func (acc *Account) GetBalance() int {
	acc.mu.Lock()
	defer acc.mu.Unlock()
	return acc.balance
}

func main() {
	acc := &Account{balance: 0}
	var wg sync.WaitGroup
	
	// 5 goroutines depositing 10 each
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			acc.Deposit(10)
		}()
	}
	
	wg.Wait()
	
	// Without mutex: Race condition, might not be 50
	// With mutex: Guaranteed to be 50
	fmt.Printf("Final balance: %d\n", acc.GetBalance())
}

// Output:
// [Deposit] Current: 0, Adding: 10
// [Deposit] Current: 10, Adding: 10
// [Deposit] Current: 20, Adding: 10
// [Deposit] Current: 30, Adding: 10
// [Deposit] Current: 40, Adding: 10
// [Deposit] New balance: 10
// [Deposit] New balance: 20
// [Deposit] New balance: 30
// [Deposit] New balance: 40
// [Deposit] New balance: 50
// Final balance: 50
```

**Mutex Pattern:**
```
acc.mu.Lock()
defer acc.mu.Unlock()
// Critical section - only one goroutine at a time
// Use shared resource here
```

---

### 4.2 RWMutex - READER-WRITER LOCKS

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type Database struct {
	data map[string]string
	mu   sync.RWMutex  // Reader-Writer mutex
}

// Multiple readers can hold lock simultaneously
func (db *Database) Read(key string) string {
	db.mu.RLock()       // Read lock (multiple allowed)
	defer db.mu.RUnlock()
	
	fmt.Printf("[Reader] Reading %s\n", key)
	time.Sleep(100 * time.Millisecond)
	value := db.data[key]
	fmt.Printf("[Reader] Read %s = %s\n", key, value)
	
	return value
}

// Only one writer at a time, blocks all readers
func (db *Database) Write(key, value string) {
	db.mu.Lock()        // Write lock (exclusive)
	defer db.mu.Unlock()
	
	fmt.Printf("[Writer] Writing %s = %s\n", key, value)
	time.Sleep(50 * time.Millisecond)
	db.data[key] = value
	fmt.Printf("[Writer] Done\n")
}

func main() {
	db := &Database{data: make(map[string]string)}
	var wg sync.WaitGroup
	
	// 5 readers
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			db.Read("key1")
		}(i)
	}
	
	// 1 writer (blocks all readers)
	wg.Add(1)
	go func() {
		defer wg.Done()
		time.Sleep(50 * time.Millisecond)
		db.Write("key1", "value1")
	}()
	
	wg.Wait()
	fmt.Println("Done")
}
```

**RWMutex Use Cases:**
- Readers: Read lock (multiple concurrent)
- Writers: Write lock (exclusive, blocks all)
- Great for read-heavy workloads

---

### 4.3 SYNC.ONCE - EXECUTE EXACTLY ONCE

```go
package main

import (
	"fmt"
	"sync"
)

var (
	instance string
	once     sync.Once
)

// Singleton pattern with sync.Once
func getInstance() string {
	// Do() executes function exactly once, even with multiple goroutines
	once.Do(func() {
		fmt.Println("Initializing instance...")
		instance = "Singleton Instance"
	})
	
	return instance
}

func main() {
	var wg sync.WaitGroup
	
	// 5 goroutines calling getInstance
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			result := getInstance()
			fmt.Printf("[%d] Got: %s\n", id, result)
		}(i)
	}
	
	wg.Wait()
}

// Output:
// Initializing instance... (only once!)
// [0] Got: Singleton Instance
// [1] Got: Singleton Instance
// [2] Got: Singleton Instance
// [3] Got: Singleton Instance
// [4] Got: Singleton Instance
```

**Use Cases:**
- Lazy initialization
- Singleton pattern
- One-time setup

---

### 4.4 SYNC.COND - CONDITIONAL VARIABLES

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var mu sync.Mutex
	cond := sync.NewCond(&mu)
	
	queue := []int{}
	done := false
	
	// Producer
	go func() {
		for i := 1; i <= 3; i++ {
			mu.Lock()
			queue = append(queue, i)
			fmt.Printf("[Producer] Added %d\n", i)
			
			// Signal: Notify waiting goroutines that condition changed
			cond.Signal()
			mu.Unlock()
		}
		
		mu.Lock()
		done = true
		cond.Broadcast()  // Notify all waiters
		mu.Unlock()
	}()
	
	// Consumer
	go func() {
		mu.Lock()
		for !done {
			// Wait: Releases lock, waits for signal, reacquires lock
			if len(queue) == 0 && !done {
				fmt.Println("[Consumer] Waiting...")
				cond.Wait()
			}
			
			if len(queue) > 0 {
				item := queue[0]
				queue = queue[1:]
				fmt.Printf("[Consumer] Got %d\n", item)
			}
		}
		mu.Unlock()
	}()
	
	time.Sleep(2 * time.Second)
}

// Output:
// [Producer] Added 1
// [Consumer] Waiting...
// [Consumer] Got 1
// [Producer] Added 2
// [Consumer] Got 2
// [Producer] Added 3
// [Consumer] Got 3
```

---

## PART 5: CONCURRENCY PATTERNS

### 5.1 WORKER POOL PATTERN

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Task: Unit of work
type Task struct {
	id    int
	value int
}

// Result: Output from task
type Result struct {
	task   Task
	result int
}

func main() {
	// Configuration
	numWorkers := 3
	numTasks := 10
	
	// Channels
	taskCh := make(chan Task)
	resultCh := make(chan Result)
	
	var wg sync.WaitGroup
	
	// Start workers
	for w := 1; w <= numWorkers; w++ {
		wg.Add(1)
		go func(workerID int) {
			defer wg.Done()
			
			// Worker: Consume tasks until channel closes
			for task := range taskCh {
				fmt.Printf("[Worker %d] Processing task %d\n", workerID, task.id)
				time.Sleep(100 * time.Millisecond)
				
				// Process task and send result
				result := Result{
					task:   task,
					result: task.value * 2,
				}
				resultCh <- result
				fmt.Printf("[Worker %d] Completed task %d\n", workerID, task.id)
			}
		}(w)
	}
	
	// Producer: Send tasks
	go func() {
		for t := 1; t <= numTasks; t++ {
			taskCh <- Task{id: t, value: t * 10}
		}
		close(taskCh)  // Signal workers to stop
	}()
	
	// Collect results in separate goroutine
	resultCount := 0
	go func() {
		for result := range resultCh {
			fmt.Printf("Result: Task %d = %d\n", result.task.id, result.result)
			resultCount++
		}
	}()
	
	// Wait for all workers to finish
	wg.Wait()
	close(resultCh)
	
	fmt.Printf("All %d tasks completed\n", numTasks)
}
```

**Advantages:**
- ✅ Bounded concurrency (fixed worker count)
- ✅ Reuses goroutines efficiently
- ✅ Natural backpressure (queue fills when busy)
- ✅ Easy to control resource usage

---

### 5.2 FAN-OUT/FAN-IN PATTERN

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Fan-Out: Distribute work to multiple goroutines
func fanOut(input []int, numWorkers int) []<-chan int {
	workers := make([]<-chan int, numWorkers)
	
	// Split input among workers
	itemsPerWorker := len(input) / numWorkers
	
	for w := 0; w < numWorkers; w++ {
		start := w * itemsPerWorker
		end := start + itemsPerWorker
		
		// Handle remainder
		if w == numWorkers-1 {
			end = len(input)
		}
		
		// Each worker gets slice of input
		ch := make(chan int)
		go func(items []int, workerID int) {
			defer close(ch)
			
			for _, item := range items {
				fmt.Printf("[Worker %d] Processing %d\n", workerID, item)
				time.Sleep(50 * time.Millisecond)
				ch <- item * 2
			}
		}(input[start:end], w)
		
		workers[w] = ch
	}
	
	return workers
}

// Fan-In: Collect results from multiple goroutines
func fanIn(workers []<-chan int) <-chan int {
	out := make(chan int)
	var wg sync.WaitGroup
	
	// Multiplex: Read from each worker
	for _, w := range workers {
		wg.Add(1)
		go func(ch <-chan int) {
			defer wg.Done()
			for val := range ch {
				out <- val
			}
		}(w)
	}
	
	// Close output when all workers done
	go func() {
		wg.Wait()
		close(out)
	}()
	
	return out
}

func main() {
	input := []int{1, 2, 3, 4, 5, 6}
	numWorkers := 2
	
	// Fan-out
	workers := fanOut(input, numWorkers)
	
	// Fan-in
	results := fanIn(workers)
	
	// Collect results
	var outputs []int
	for result := range results {
		fmt.Printf("Result: %d\n", result)
		outputs = append(outputs, result)
	}
	
	fmt.Printf("Processed %d items\n", len(outputs))
}
```

---

### 5.3 PIPELINE PATTERN

```go
package main

import (
	"fmt"
	"time"
)

// Stage 1: Generate numbers
func generate(nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for _, n := range nums {
			fmt.Printf("[Generate] %d\n", n)
			out <- n
		}
	}()
	return out
}

// Stage 2: Square numbers
func square(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			fmt.Printf("[Square] Input %d\n", n)
			time.Sleep(50 * time.Millisecond)
			result := n * n
			fmt.Printf("[Square] Output %d\n", result)
			out <- result
		}
	}()
	return out
}

// Stage 3: Cube numbers
func cube(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			fmt.Printf("[Cube] Input %d\n", n)
			result := n * n * n
			fmt.Printf("[Cube] Output %d\n", result)
			out <- result
		}
	}()
	return out
}

func main() {
	// Chain: 1,2,3 -> square -> cube -> print
	nums := generate(1, 2, 3, 4, 5)
	squared := square(nums)
	cubed := cube(squared)
	
	// Consume
	for result := range cubed {
		fmt.Printf("Final result: %d\n", result)
	}
}

// Output:
// [Generate] 1
// [Square] Input 1
// [Square] Output 1
// [Cube] Input 1
// [Cube] Output 1
// Final result: 1
// ...
```

---

## PART 6: CONTEXT & CANCELLATION

### 6.1 CONTEXT BASICS

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func doWork(ctx context.Context, workName string) {
	for {
		select {
		case <-ctx.Done():
			// Context cancelled or timed out
			err := ctx.Err()
			fmt.Printf("%s: %v\n", workName, err)
			return
			
		default:
			fmt.Printf("%s: Working...\n", workName)
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	// Create context with 2-second timeout
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()  // Always cancel to free resources
	
	go doWork(ctx, "Task1")
	go doWork(ctx, "Task2")
	
	// Wait for context to timeout
	time.Sleep(3 * time.Second)
	fmt.Println("Done")
}

// Output:
// Task1: Working...
// Task2: Working...
// Task1: Working...
// Task2: Working...
// Task1: context deadline exceeded
// Task2: context deadline exceeded
// Done
```

---

### 6.2 CONTEXT CANCELLATION

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// Create cancellable context
	ctx, cancel := context.WithCancel(context.Background())
	
	go func() {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("Goroutine cancelled!")
				return
			default:
				fmt.Println("Working...")
				time.Sleep(500 * time.Millisecond)
			}
		}
	}()
	
	time.Sleep(2 * time.Second)
	
	// Manually cancel context
	cancel()
	
	time.Sleep(500 * time.Millisecond)
	fmt.Println("Main done")
}

// Output:
// Working...
// Working...
// Working...
// Working...
// Goroutine cancelled!
// Main done
```

---

### 6.3 CONTEXT WITH DEADLINE

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func slowOperation(ctx context.Context) error {
	// Simulate long operation
	select {
	case <-time.After(5 * time.Second):
		return fmt.Errorf("operation completed")
	case <-ctx.Done():
		return ctx.Err()
	}
}

func main() {
	// Deadline: absolute time when context expires
	deadline := time.Now().Add(2 * time.Second)
	ctx, cancel := context.WithDeadline(context.Background(), deadline)
	defer cancel()
	
	// Try operation
	err := slowOperation(ctx)
	
	if err != nil {
		fmt.Printf("Error: %v\n", err)
	}
}

// Output:
// Error: context deadline exceeded
```

---

## PART 7: COMMON PITFALLS & PREVENTION

### 7.1 GOROUTINE LEAKS

```go
// ✗ LEAK 1: Goroutine never terminates
package main

func leak1() {
	ch := make(chan int)
	
	go func() {
		val := <-ch  // Waits forever, never cancelled
		println(val)
	}()
}

// ✓ FIX 1: Use context for cancellation
package main

import "context"

func noLeak1() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	
	ch := make(chan int)
	
	go func() {
		select {
		case val := <-ch:
			println(val)
		case <-ctx.Done():
			return
		}
	}()
}

// ✗ LEAK 2: Unbounded goroutine creation
package main

func leak2() {
	for i := 0; i < 1000000; i++ {
		go func() {
			for {} // Infinite loop, never exits
		}()
	}
}

// ✓ FIX 2: Use worker pool with bounded goroutines
package main

func noLeak2() {
	taskCh := make(chan int)
	
	// Fixed number of workers
	for w := 0; w < 10; w++ {
		go func() {
			for range taskCh {
				// Process task
			}
		}()
	}
	
	// Send tasks (bounded by buffer)
	for i := 0; i < 1000000; i++ {
		taskCh <- i
	}
}
```

---

### 7.2 DEADLOCKS

```go
// ✗ DEADLOCK 1: Circular wait
package main

func deadlock1() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	
	// Goroutine 1: Waits for ch2, sends on ch1
	go func() {
		val := <-ch2
		ch1 <- val
	}()
	
	// Goroutine 2: Waits for ch1, sends on ch2
	go func() {
		val := <-ch1
		ch2 <- val
	}()
	
	// DEADLOCK: Both goroutines waiting forever
}

// ✓ FIX 1: Ensure proper communication
package main

func noDeadlock1() {
	ch := make(chan int)
	
	go func() {
		ch <- 42
	}()
	
	val := <-ch
	println(val)
}

// ✗ DEADLOCK 2: Send on closed channel
package main

func deadlock2() {
	ch := make(chan int)
	close(ch)
	
	ch <- 42  // PANIC: send on closed channel
}

// ✓ FIX 2: Check before sending
package main

func noDeadlock2() {
	ch := make(chan int)
	
	// Use context or flag to control send
	ctx, cancel := context.WithCancel(context.Background())
	
	select {
	case ch <- 42:
	case <-ctx.Done():
		return
	}
	
	cancel()
}
```

---

### 7.3 RACE CONDITIONS

```go
// ✗ RACE CONDITION: Multiple goroutines access shared variable
package main

var counter int

func raceCondition() {
	for i := 0; i < 100; i++ {
		go func() {
			counter++  // RACE! Multiple goroutines modify without synchronization
		}()
	}
}

// ✓ FIX 1: Use Mutex
package main

import "sync"

var counter int
var mu sync.Mutex

func noRaceCondition1() {
	for i := 0; i < 100; i++ {
		go func() {
			mu.Lock()
			counter++
			mu.Unlock()
		}()
	}
}

// ✓ FIX 2: Use channels (preferred in Go)
package main

func noRaceCondition2() {
	ch := make(chan int)
	
	go func() {
		count := 0
		for i := 0; i < 100; i++ {
			count += <-ch
		}
		println(count)
	}()
	
	for i := 0; i < 100; i++ {
		go func() {
			ch <- 1
		}()
	}
}
```

**Detect Races:**
```bash
go run -race main.go    # Run with race detector
go test -race ./...     # Test with race detector
```

---

## PART 8: PRODUCTION-READY EXAMPLES

### 8.1 COMPLETE WORKER POOL WITH CONTEXT

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

type WorkerPool struct {
	tasks   chan Task
	results chan Result
	wg      sync.WaitGroup
}

type Task struct {
	ID    int
	Delay time.Duration
}

type Result struct {
	TaskID int
	Value  int
	Error  error
}

func NewWorkerPool(ctx context.Context, numWorkers int) *WorkerPool {
	pool := &WorkerPool{
		tasks:   make(chan Task, 100),
		results: make(chan Result),
	}
	
	// Start workers
	for i := 1; i <= numWorkers; i++ {
		pool.wg.Add(1)
		go pool.worker(ctx, i)
	}
	
	// Monitor for context cancellation
	go func() {
		<-ctx.Done()
		close(pool.tasks)
	}()
	
	return pool
}

func (p *WorkerPool) worker(ctx context.Context, workerID int) {
	defer p.wg.Done()
	
	for task := range p.tasks {
		select {
		case <-ctx.Done():
			p.results <- Result{
				TaskID: task.ID,
				Error:  ctx.Err(),
			}
			return
		default:
			fmt.Printf("[Worker %d] Processing task %d\n", workerID, task.ID)
			
			// Simulate work
			time.Sleep(task.Delay)
			
			p.results <- Result{
				TaskID: task.ID,
				Value:  task.ID * 10,
			}
		}
	}
}

func (p *WorkerPool) Submit(task Task) {
	p.tasks <- task
}

func (p *WorkerPool) Wait() []Result {
	go func() {
		p.wg.Wait()
		close(p.results)
	}()
	
	results := []Result{}
	for result := range p.results {
		results = append(results, result)
	}
	return results
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	
	pool := NewWorkerPool(ctx, 3)
	
	// Submit tasks
	for i := 1; i <= 10; i++ {
		pool.Submit(Task{
			ID:    i,
			Delay: 200 * time.Millisecond,
		})
	}
	
	// Get results
	results := pool.Wait()
	
	for _, r := range results {
		if r.Error != nil {
			fmt.Printf("Task %d error: %v\n", r.TaskID, r.Error)
		} else {
			fmt.Printf("Task %d result: %d\n", r.TaskID, r.Value)
		}
	}
}
```

---

### 8.2 GRACEFUL SHUTDOWN

```go
package main

import (
	"context"
	"fmt"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

type Service struct {
	workers int
	wg      sync.WaitGroup
}

func (s *Service) Run(ctx context.Context) {
	fmt.Println("Service starting...")
	
	for w := 1; w <= s.workers; w++ {
		s.wg.Add(1)
		go s.worker(ctx, w)
	}
	
	fmt.Println("Service running, press Ctrl+C to shutdown")
	
	// Wait for all workers
	s.wg.Wait()
	fmt.Println("All workers finished")
}

func (s *Service) worker(ctx context.Context, id int) {
	defer s.wg.Done()
	
	ticker := time.NewTicker(500 * time.Millisecond)
	defer ticker.Stop()
	
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("[Worker %d] Shutting down gracefully...\n", id)
			return
			
		case <-ticker.C:
			fmt.Printf("[Worker %d] Working...\n", id)
		}
	}
}

func main() {
	service := &Service{workers: 3}
	
	// Context for graceful shutdown
	ctx, cancel := context.WithCancel(context.Background())
	
	// Handle OS signals
	sigChan := make(chan os.Signal, 1)
	signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
	
	// Start service in goroutine
	go service.Run(ctx)
	
	// Wait for signal
	<-sigChan
	
	fmt.Println("\nShutdown signal received")
	cancel()  // Cancel context, notifies all goroutines
	
	// Wait for graceful shutdown (with timeout)
	done := make(chan struct{})
	go func() {
		service.wg.Wait()
		close(done)
	}()
	
	select {
	case <-done:
		fmt.Println("Service stopped cleanly")
	case <-time.After(10 * time.Second):
		fmt.Println("Shutdown timeout, forcing exit")
	}
}

// Output:
// Service starting...
// Service running, press Ctrl+C to shutdown
// [Worker 1] Working...
// [Worker 2] Working...
// [Worker 3] Working...
// ...
// (Press Ctrl+C)
// Shutdown signal received
// [Worker 1] Shutting down gracefully...
// [Worker 2] Shutting down gracefully...
// [Worker 3] Shutting down gracefully...
// All workers finished
// Service stopped cleanly
```

---

### 8.3 CONCURRENT HTTP REQUESTS

```go
package main

import (
	"context"
	"fmt"
	"io"
	"net/http"
	"sync"
	"time"
)

type URLFetcher struct {
	urls    []string
	workers int
	timeout time.Duration
}

type FetchResult struct {
	URL    string
	Status int
	Size   int
	Error  error
}

func (f *URLFetcher) Fetch(ctx context.Context) []FetchResult {
	taskCh := make(chan string)
	resultCh := make(chan FetchResult)
	var wg sync.WaitGroup
	
	// Start workers
	for w := 0; w < f.workers; w++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			for url := range taskCh {
				result := f.fetchURL(ctx, url)
				resultCh <- result
			}
		}()
	}
	
	// Send URLs
	go func() {
		for _, url := range f.urls {
			taskCh <- url
		}
		close(taskCh)
	}()
	
	// Collect results
	results := []FetchResult{}
	go func() {
		wg.Wait()
		close(resultCh)
	}()
	
	for result := range resultCh {
		results = append(results, result)
	}
	
	return results
}

func (f *URLFetcher) fetchURL(ctx context.Context, url string) FetchResult {
	ctx, cancel := context.WithTimeout(ctx, f.timeout)
	defer cancel()
	
	req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
	
	client := &http.Client{}
	resp, err := client.Do(req)
	
	if err != nil {
		return FetchResult{URL: url, Error: err}
	}
	defer resp.Body.Close()
	
	// Read body size
	data, _ := io.ReadAll(resp.Body)
	
	return FetchResult{
		URL:    url,
		Status: resp.StatusCode,
		Size:   len(data),
	}
}

func main() {
	urls := []string{
		"https://httpbin.org/delay/1",
		"https://httpbin.org/status/200",
		"https://httpbin.org/json",
	}
	
	fetcher := &URLFetcher{
		urls:    urls,
		workers: 3,
		timeout: 5 * time.Second,
	}
	
	ctx := context.Background()
	results := fetcher.Fetch(ctx)
	
	for _, r := range results {
		if r.Error != nil {
			fmt.Printf("%s: ERROR - %v\n", r.URL, r.Error)
		} else {
			fmt.Printf("%s: %d (%d bytes)\n", r.URL, r.Status, r.Size)
		}
	}
}
```

---

## PART 9: CHECKLIST & BEST PRACTICES

### Design Checklist

- [ ] **Goroutine Lifecycle**: Every goroutine must terminate cleanly
- [ ] **Channel Management**: Always close channels from sender side only
- [ ] **Context Usage**: Pass context for cancellation and timeouts
- [ ] **Resource Limits**: Use worker pools to limit concurrency
- [ ] **Error Handling**: Handle errors from all concurrent operations
- [ ] **Synchronization**: Choose Mutex, Channel, or other sync primitives
- [ ] **Testing**: Run with `-race` flag to detect races
- [ ] **Monitoring**: Track goroutine count and memory usage
- [ ] **Documentation**: Document concurrency patterns used

### Performance Tips

| Strategy | Benefit | Use Case |
|----------|---------|----------|
| Buffered channels | Reduce blocking | Producer-consumer when rates differ |
| Worker pools | Bounded resources | IO-bound tasks, API calls |
| Context timeouts | Prevent hangs | External services, long operations |
| RWMutex | More concurrency | Read-heavy workloads |
| Select with default | Non-blocking checks | Check if channel ready |
| Pipeline | Composability | Data transformation stages |

### Common Memory Leaks

| Leak Type | Cause | Prevention |
|-----------|-------|-----------|
| Goroutine leak | Never terminates | Use context cancellation |
| Channel leak | Never closed | Close from sender only |
| Circular references | Objects reference each other | Design for cleanup |
| Unbounded growth | Cache/collection never shrinks | Bounded size or TTL |

---

## SUMMARY TABLE

| Concept | Purpose | Example |
|---------|---------|---------|
| **Goroutine** | Lightweight concurrent task | `go functionName()` |
| **Channel** | Type-safe communication | `ch := make(chan int)` |
| **Unbuffered** | Synchronous handshake | `ch := make(chan int)` |
| **Buffered** | Asynchronous within limit | `ch := make(chan int, 10)` |
| **Select** | Wait on multiple channels | `select { case <-ch1: ...` |
| **Mutex** | Protect shared memory | Lock/Unlock around critical section |
| **RWMutex** | Read-write protection | Multiple readers, exclusive writer |
| **WaitGroup** | Wait for goroutine completion | Add/Done/Wait |
| **Once** | Execute exactly once | Lazy initialization |
| **Context** | Cancellation and timeouts | Pass through call chain |

This comprehensive guide covers everything from concurrent basics to production-ready patterns!