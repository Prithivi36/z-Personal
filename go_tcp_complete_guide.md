# Complete Go TCP Guide: From Basics to Advanced Network Programming

## TABLE OF CONTENTS
1. **Fundamentals & Theory**
2. **TCP Server (Basic to Advanced)**
3. **TCP Client (Basic to Advanced)**
4. **Connection Management & I/O**
5. **Protocol Implementation**
6. **Concurrent Servers & Scaling**
7. **Graceful Shutdown & Error Handling**
8. **Production Patterns & Best Practices**

---

## PART 1: FUNDAMENTALS & THEORY

### 1.1 TCP/IP Overview

**TCP (Transmission Control Protocol):**
- Connection-oriented (handshake before data)
- Reliable (data arrives in order, no loss)
- Stream-based (continuous flow of bytes)
- Bidirectional (both client and server can send/receive)
- Stateful (maintains connection state)

**TCP Connection Flow:**
```
Client                              Server
  |                                   |
  |--------- Connect Request -------> | (SYN)
  | <------ Connect Response -------- | (SYN-ACK)
  |--------- Acknowledgment -------> | (ACK)
  |  [Connection Established]         |
  |                                   |
  |--------- Send Data ------------> |
  | <------ Send Data --------------- |
  |                                   |
  |--------- Close Connection ------> | (FIN)
  | <------ Close Acknowledge ------ | (FIN-ACK)
  |  [Connection Closed]              |
```

### 1.2 Go's net Package

```
net package provides:
├── Listen(): Create listener (server-side)
├── Dial(): Connect to server (client-side)
├── Accept(): Accept incoming connection (server-side)
├── Conn interface: Read/Write/Close
├── TCPListener: TCP-specific listener
├── TCPConn: TCP-specific connection
└── Dialer: Advanced connection control
```

### 1.3 Key Types

```go
// Listener: Accepts incoming connections
type Listener interface {
    Accept() (Conn, error)
    Close() error
    Addr() Addr
}

// Conn: Represents connection
type Conn interface {
    Read(b []byte) (n int, err error)      // Read data
    Write(b []byte) (n int, err error)     // Write data
    Close() error                           // Close connection
    LocalAddr() Addr                        // Local address
    RemoteAddr() Addr                       // Remote address
    SetDeadline(t time.Time) error          // I/O deadline
    SetReadDeadline(t time.Time) error      // Read deadline
    SetWriteDeadline(t time.Time) error     // Write deadline
}
```

---

## PART 2: TCP SERVER - COMPLETE GUIDE

### 2.1 BASIC TCP SERVER

```go
package main

import (
	"fmt"       // For printing
	"log"       // For logging
	"net"       // TCP/IP networking
	"os"        // OS operations
)

func main() {
	// Step 1: Create listener
	// Listen(network, address)
	// "tcp": Listen on TCP protocol
	// ":8080": Listen on all interfaces (*), port 8080
	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatalf("Listen error: %v", err)
	}
	
	// Step 2: Always defer close to cleanup resources
	defer listener.Close()
	
	fmt.Println("TCP Server listening on :8080")
	
	// Step 3: Accept incoming connections in loop
	for {
		// Accept(): Blocks until client connects
		// Returns net.Conn for communication with client
		conn, err := listener.Accept()
		if err != nil {
			// Continue on error (don't crash server)
			fmt.Printf("Accept error: %v\n", err)
			continue
		}
		
		// Step 4: Handle each connection (in main goroutine for now)
		handleConnection(conn)
	}
}

func handleConnection(conn net.Conn) {
	// Always close connection when done
	defer conn.Close()
	
	// Get remote address (client IP:port)
	fmt.Printf("Client connected: %s\n", conn.RemoteAddr())
	
	// Step 1: Create buffer (4KB buffer for reading)
	buffer := make([]byte, 4096)
	
	// Step 2: Read from connection
	// Read blocks until data available or connection closes
	n, err := conn.Read(buffer)
	if err != nil {
		fmt.Printf("Read error: %v\n", err)
		return
	}
	
	// Step 3: Process data
	data := string(buffer[:n]) // Only use bytes that were read
	fmt.Printf("Received: %s\n", data)
	
	// Step 4: Send response
	response := fmt.Sprintf("Echo: %s", data)
	_, err = conn.Write([]byte(response))
	if err != nil {
		fmt.Printf("Write error: %v\n", err)
		return
	}
	
	fmt.Println("Response sent")
}
```

**Testing:**
```bash
# Terminal 1: Run server
go run server.go

# Terminal 2: Connect with netcat
echo "Hello Server" | nc localhost 8080
# Output: Echo: Hello Server
```

---

### 2.2 CONCURRENT SERVER (Multiple Clients)

```go
package main

import (
	"fmt"
	"log"
	"net"
	"sync"
)

func main() {
	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatalf("Listen error: %v", err)
	}
	defer listener.Close()
	
	fmt.Println("TCP Server listening on :8080")
	var wg sync.WaitGroup
	
	// Accept connections in loop
	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Printf("Accept error: %v\n", err)
			continue
		}
		
		// KEY DIFFERENCE: Launch goroutine for each connection
		// Allows server to accept more connections immediately
		wg.Add(1)
		go func(c net.Conn) {
			defer wg.Done()
			handleConnection(c)
		}(conn)
	}
}

func handleConnection(conn net.Conn) {
	defer conn.Close()
	
	clientAddr := conn.RemoteAddr()
	fmt.Printf("[%s] Connected\n", clientAddr)
	
	// Read loop: Keep reading from same client
	buffer := make([]byte, 4096)
	
	for {
		n, err := conn.Read(buffer)
		if err != nil {
			// EOF (client closed) or error
			fmt.Printf("[%s] Disconnected: %v\n", clientAddr, err)
			return
		}
		
		data := string(buffer[:n])
		fmt.Printf("[%s] Received: %s\n", clientAddr, data)
		
		// Send response back
		response := fmt.Sprintf("Echo: %s", data)
		_, err = conn.Write([]byte(response))
		if err != nil {
			fmt.Printf("[%s] Write error: %v\n", clientAddr, err)
			return
		}
	}
}
```

**Advantages:**
- ✅ Multiple clients connected simultaneously
- ✅ Each client's loop independent
- ✅ Server can keep accepting new connections
- ✅ Better resource utilization

---

### 2.3 READING AND WRITING DATA

```go
package main

import (
	"bufio"        // Buffered I/O
	"fmt"
	"net"
)

// ========== UNBUFFERED I/O ==========
func handleUnbuffered(conn net.Conn) {
	defer conn.Close()
	
	// Direct read from connection (no buffer)
	// Read blocks until data available
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		return
	}
	
	// Only bytes 0 to n-1 are valid
	fmt.Printf("Received: %s\n", string(buffer[:n]))
	
	// Direct write
	conn.Write([]byte("Got it!"))
}

// ========== BUFFERED I/O (PREFERRED) ==========
func handleBuffered(conn net.Conn) {
	defer conn.Close()
	
	// Create buffered reader/writer
	// Buffered I/O is more efficient (batches I/O operations)
	reader := bufio.NewReader(conn)
	writer := bufio.NewWriter(conn)
	
	// ReadString: Read until delimiter (e.g., newline)
	// Blocks until delimiter found
	line, err := reader.ReadString('\n')
	if err != nil {
		return
	}
	
	fmt.Printf("Received line: %s\n", line)
	
	// Write to writer (buffered, not immediately sent)
	writer.WriteString("Got it!\n")
	
	// IMPORTANT: Flush to actually send buffered data
	writer.Flush()
}

// ========== READING EXACT BYTES ==========
func handleExactBytes(conn net.Conn) {
	defer conn.Close()
	
	reader := bufio.NewReader(conn)
	
	// ReadFull: Read exactly n bytes or return error
	// (more reliable than Read for fixed-size messages)
	headerBuf := make([]byte, 4)
	_, err := reader.Read(headerBuf)
	if err != nil {
		return
	}
	
	fmt.Printf("Received %d bytes\n", len(headerBuf))
}

// ========== READING UNTIL DELIMITER ==========
func handleDelimiter(conn net.Conn) {
	defer conn.Close()
	
	reader := bufio.NewReader(conn)
	
	// ReadUntilDelimiter: Read until marker byte
	// Includes the delimiter
	bytes, err := reader.ReadBytes('\n')
	if err != nil {
		return
	}
	
	fmt.Printf("Received: %s\n", string(bytes))
}
```

---

### 2.4 CONNECTION ADDRESSES

```go
package main

import (
	"fmt"
	"net"
)

func handleConnection(conn net.Conn) {
	defer conn.Close()
	
	// LocalAddr: Server's local address
	// Format: IP:Port
	localAddr := conn.LocalAddr()
	fmt.Printf("Local Address: %s\n", localAddr)
	fmt.Printf("Local Network: %s\n", localAddr.Network())
	
	// Cast to TCPAddr for more details
	if tcpAddr, ok := localAddr.(*net.TCPAddr); ok {
		fmt.Printf("Local IP: %s\n", tcpAddr.IP)
		fmt.Printf("Local Port: %d\n", tcpAddr.Port)
	}
	
	// RemoteAddr: Client's address
	remoteAddr := conn.RemoteAddr()
	fmt.Printf("Remote Address: %s\n", remoteAddr)
	
	// Cast to TCPAddr
	if tcpAddr, ok := remoteAddr.(*net.TCPAddr); ok {
		fmt.Printf("Remote IP: %s\n", tcpAddr.IP)
		fmt.Printf("Remote Port: %d\n", tcpAddr.Port)
	}
}
```

---

### 2.5 DEADLINES & TIMEOUTS (Server-side)

```go
package main

import (
	"fmt"
	"net"
	"time"
)

func handleConnection(conn net.Conn) {
	defer conn.Close()
	
	// ========== OVERALL DEADLINE ==========
	// SetDeadline: All I/O operations must complete by deadline
	// Applies to both Read and Write
	deadline := time.Now().Add(5 * time.Second)
	conn.SetDeadline(deadline)
	
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
			fmt.Println("Read timeout")
		}
		return
	}
	
	fmt.Printf("Received: %s\n", string(buffer[:n]))
	
	// ========== READ DEADLINE ==========
	// SetReadDeadline: Only Read operations timeout
	conn.SetReadDeadline(time.Now().Add(3 * time.Second))
	
	buffer = make([]byte, 1024)
	n, err = conn.Read(buffer)
	if err != nil {
		fmt.Printf("Read error: %v\n", err)
		return
	}
	
	// ========== WRITE DEADLINE ==========
	// SetWriteDeadline: Only Write operations timeout
	conn.SetWriteDeadline(time.Now().Add(3 * time.Second))
	
	_, err = conn.Write([]byte("Response"))
	if err != nil {
		fmt.Printf("Write error: %v\n", err)
		return
	}
	
	// ========== REMOVE DEADLINE ==========
	// SetDeadline(time.Time{}) removes deadline
	conn.SetDeadline(time.Time{})
}
```

---

### 2.6 SERVER WITH CONTEXT & GRACEFUL SHUTDOWN

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net"
	"sync"
	"time"
)

type TCPServer struct {
	listener   net.Listener
	addr       string
	ctx        context.Context
	cancel     context.CancelFunc
	wg         sync.WaitGroup
	maxClients int
}

func NewTCPServer(addr string, maxClients int) *TCPServer {
	ctx, cancel := context.WithCancel(context.Background())
	return &TCPServer{
		addr:       addr,
		ctx:        ctx,
		cancel:     cancel,
		maxClients: maxClients,
	}
}

func (s *TCPServer) Start() error {
	listener, err := net.Listen("tcp", s.addr)
	if err != nil {
		return err
	}
	
	s.listener = listener
	fmt.Printf("Server listening on %s\n", s.addr)
	
	// Accept connections in separate goroutine
	s.wg.Add(1)
	go s.acceptConnections()
	
	return nil
}

func (s *TCPServer) acceptConnections() {
	defer s.wg.Done()
	
	for {
		select {
		case <-s.ctx.Done():
			fmt.Println("Accepting connections stopped")
			return
		default:
		}
		
		// Set accept deadline to allow periodic context check
		s.listener.(*net.TCPListener).SetDeadline(time.Now().Add(1 * time.Second))
		
		conn, err := s.listener.Accept()
		if err != nil {
			// Check if timeout (expected) or real error
			if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
				continue
			}
			fmt.Printf("Accept error: %v\n", err)
			return
		}
		
		// Handle connection in goroutine
		s.wg.Add(1)
		go func() {
			defer s.wg.Done()
			s.handleConnection(conn)
		}()
	}
}

func (s *TCPServer) handleConnection(conn net.Conn) {
	defer conn.Close()
	
	fmt.Printf("Client connected: %s\n", conn.RemoteAddr())
	
	buffer := make([]byte, 1024)
	
	for {
		// Check context for cancellation
		select {
		case <-s.ctx.Done():
			fmt.Printf("Server closing, disconnecting %s\n", conn.RemoteAddr())
			return
		default:
		}
		
		n, err := conn.Read(buffer)
		if err != nil {
			fmt.Printf("Client %s disconnected\n", conn.RemoteAddr())
			return
		}
		
		fmt.Printf("Received: %s\n", string(buffer[:n]))
		conn.Write([]byte("Got it!"))
	}
}

func (s *TCPServer) Shutdown(timeout time.Duration) {
	fmt.Println("Shutting down server...")
	s.cancel()  // Cancel context
	
	// Wait with timeout
	done := make(chan struct{})
	go func() {
		s.wg.Wait()
		close(done)
	}()
	
	select {
	case <-done:
		fmt.Println("Server shut down cleanly")
	case <-time.After(timeout):
		fmt.Println("Shutdown timeout")
	}
	
	s.listener.Close()
}

func main() {
	server := NewTCPServer(":8080", 100)
	server.Start()
	
	// Graceful shutdown on signal
	time.Sleep(30 * time.Second)
	server.Shutdown(5 * time.Second)
}
```

---

## PART 3: TCP CLIENT - COMPLETE GUIDE

### 3.1 BASIC TCP CLIENT

```go
package main

import (
	"fmt"
	"log"
	"net"
)

func main() {
	// Step 1: Connect to server
	// Dial(network, address)
	// Blocks until connection established or error
	conn, err := net.Dial("tcp", "localhost:8080")
	if err != nil {
		log.Fatalf("Dial error: %v", err)
	}
	
	// Step 2: Always defer close
	defer conn.Close()
	
	fmt.Printf("Connected to %s\n", conn.RemoteAddr())
	
	// Step 3: Send data to server
	message := "Hello Server!"
	_, err = conn.Write([]byte(message))
	if err != nil {
		log.Fatalf("Write error: %v", err)
	}
	
	fmt.Printf("Sent: %s\n", message)
	
	// Step 4: Read response from server
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		log.Fatalf("Read error: %v", err)
	}
	
	response := string(buffer[:n])
	fmt.Printf("Received: %s\n", response)
}

// Usage:
// go run server.go     (in one terminal)
// go run client.go     (in another)
```

---

### 3.2 CLIENT WITH MULTIPLE MESSAGES

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"net"
	"strings"
)

func main() {
	conn, err := net.Dial("tcp", "localhost:8080")
	if err != nil {
		log.Fatalf("Dial error: %v", err)
	}
	defer conn.Close()
	
	fmt.Println("Connected to server. Type messages (type 'quit' to exit)")
	
	// Create buffered reader for console input
	consoleReader := bufio.NewReader(os.Stdin)
	
	// Create buffered reader/writer for connection
	connReader := bufio.NewReader(conn)
	connWriter := bufio.NewWriter(conn)
	
	for {
		// Read from console
		fmt.Print("> ")
		input, _ := consoleReader.ReadString('\n')
		input = strings.TrimSpace(input)
		
		if input == "quit" {
			break
		}
		
		// Send to server
		connWriter.WriteString(input + "\n")
		connWriter.Flush()
		
		// Read response from server
		response, _ := connReader.ReadString('\n')
		fmt.Printf("Server: %s\n", response)
	}
	
	fmt.Println("Disconnected")
}
```

---

### 3.3 CLIENT WITH TIMEOUT

```go
package main

import (
	"fmt"
	"net"
	"time"
)

func main() {
	// ========== DIAL TIMEOUT ==========
	// Dialer: Advanced connection control
	dialer := net.Dialer{
		Timeout:   5 * time.Second,   // Connection timeout
		KeepAlive: 30 * time.Second,  // TCP keep-alive
	}
	
	conn, err := dialer.Dial("tcp", "localhost:8080")
	if err != nil {
		fmt.Printf("Dial error: %v\n", err)
		return
	}
	defer conn.Close()
	
	fmt.Println("Connected")
	
	// ========== READ/WRITE TIMEOUT ==========
	// Set deadline on connection
	conn.SetDeadline(time.Now().Add(10 * time.Second))
	
	// Send message
	conn.Write([]byte("Hello"))
	
	// Read response (will timeout after 10 seconds)
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
			fmt.Println("Read operation timed out")
		}
		return
	}
	
	fmt.Printf("Received: %s\n", string(buffer[:n]))
}
```

---

### 3.4 CLIENT WITH CONTEXT

```go
package main

import (
	"context"
	"fmt"
	"net"
	"time"
)

func main() {
	// Create context with timeout
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	
	// Create dialer
	dialer := net.Dialer{}
	
	// DialContext: Connect with context (cancellation + timeout)
	conn, err := dialer.DialContext(ctx, "tcp", "localhost:8080")
	if err != nil {
		fmt.Printf("DialContext error: %v\n", err)
		return
	}
	defer conn.Close()
	
	fmt.Println("Connected")
	
	// Use connection with context
	// If context cancels, I/O operations will be affected
	conn.Write([]byte("Hello"))
	
	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		fmt.Printf("Read error: %v\n", err)
		return
	}
	
	fmt.Printf("Received: %s\n", string(buffer[:n]))
}
```

---

### 3.5 KEEP-ALIVE & CONNECTION MANAGEMENT

```go
package main

import (
	"fmt"
	"net"
	"time"
)

func main() {
	dialer := net.Dialer{
		Timeout:   5 * time.Second,
		KeepAlive: 30 * time.Second,  // Send keep-alive probes
	}
	
	conn, err := dialer.Dial("tcp", "localhost:8080")
	if err != nil {
		fmt.Printf("Error: %v\n", err)
		return
	}
	defer conn.Close()
	
	// Get local address
	fmt.Printf("Local: %s\n", conn.LocalAddr())
	
	// Get remote address
	fmt.Printf("Remote: %s\n", conn.RemoteAddr())
	
	// TCP connection info (if TCPConn)
	if tcpConn, ok := conn.(*net.TCPConn); ok {
		// Set linger: How long to linger before forcefully closing
		// 0: Immediate close
		// -1: No linger (graceful close)
		tcpConn.SetLinger(-1)
		
		// Disable Nagle's algorithm (send data immediately)
		tcpConn.SetNoDelay(true)
	}
	
	// Send/receive data
	conn.Write([]byte("Test"))
	
	buffer := make([]byte, 1024)
	conn.Read(buffer)
}
```

---

## PART 4: PROTOCOL IMPLEMENTATION

### 4.1 TEXT PROTOCOL (Line-Delimited)

```go
package main

import (
	"bufio"
	"fmt"
	"net"
	"strings"
)

// ========== SERVER ==========
func handleLineProtocol(conn net.Conn) {
	defer conn.Close()
	
	// Buffered reader reads until newline
	scanner := bufio.NewScanner(conn)
	writer := bufio.NewWriter(conn)
	
	for scanner.Scan() {
		// scanner.Bytes() or scanner.Text() get the line
		line := scanner.Text()
		fmt.Printf("Received: %s\n", line)
		
		// Process command
		parts := strings.Fields(line)
		if len(parts) == 0 {
			continue
		}
		
		cmd := parts[0]
		
		switch cmd {
		case "ECHO":
			response := strings.Join(parts[1:], " ")
			writer.WriteString(response + "\n")
			
		case "TIME":
			writer.WriteString("Current time\n")
			
		case "QUIT":
			writer.WriteString("Goodbye\n")
			writer.Flush()
			return
			
		default:
			writer.WriteString("Unknown command\n")
		}
		
		writer.Flush()
	}
	
	if err := scanner.Err(); err != nil {
		fmt.Printf("Scanner error: %v\n", err)
	}
}

// ========== CLIENT ==========
func clientLineProtocol() {
	conn, _ := net.Dial("tcp", "localhost:8080")
	defer conn.Close()
	
	writer := bufio.NewWriter(conn)
	scanner := bufio.NewScanner(conn)
	
	// Send command
	writer.WriteString("ECHO Hello Server\n")
	writer.Flush()
	
	// Read response
	if scanner.Scan() {
		response := scanner.Text()
		fmt.Printf("Response: %s\n", response)
	}
}
```

---

### 4.2 BINARY PROTOCOL (Length-Prefixed)

```go
package main

import (
	"encoding/binary"
	"fmt"
	"io"
	"net"
)

// ========== PROTOCOL STRUCTURE ==========
// Packet Format:
// [4 bytes: Length] [N bytes: Data]
//
// Example: Message "Hello"
// 0x00 0x00 0x00 0x05 'H' 'e' 'l' 'l' 'o'

// ========== SERVER ==========
func handleBinaryProtocol(conn net.Conn) {
	defer conn.Close()
	
	for {
		// Step 1: Read 4-byte length header
		lengthBuf := make([]byte, 4)
		_, err := io.ReadFull(conn, lengthBuf)
		if err != nil {
			fmt.Printf("Error reading length: %v\n", err)
			return
		}
		
		// Convert bytes to uint32 (big-endian)
		length := binary.BigEndian.Uint32(lengthBuf)
		fmt.Printf("Message length: %d\n", length)
		
		// Step 2: Read message data
		messageBuf := make([]byte, length)
		_, err = io.ReadFull(conn, messageBuf)
		if err != nil {
			fmt.Printf("Error reading message: %v\n", err)
			return
		}
		
		message := string(messageBuf)
		fmt.Printf("Received: %s\n", message)
		
		// Step 3: Send response (same format)
		response := "Got: " + message
		responseBytes := []byte(response)
		
		// Write length
		lenBuf := make([]byte, 4)
		binary.BigEndian.PutUint32(lenBuf, uint32(len(responseBytes)))
		conn.Write(lenBuf)
		
		// Write data
		conn.Write(responseBytes)
	}
}

// ========== CLIENT ==========
func clientBinaryProtocol() {
	conn, _ := net.Dial("tcp", "localhost:8080")
	defer conn.Close()
	
	message := "Hello Server"
	
	// Prepare packet
	packet := make([]byte, 4+len(message))
	
	// Write length (first 4 bytes)
	binary.BigEndian.PutUint32(packet[0:4], uint32(len(message)))
	
	// Write message
	copy(packet[4:], message)
	
	// Send
	conn.Write(packet)
	
	// Read response
	lenBuf := make([]byte, 4)
	io.ReadFull(conn, lenBuf)
	
	length := binary.BigEndian.Uint32(lenBuf)
	msgBuf := make([]byte, length)
	io.ReadFull(conn, msgBuf)
	
	fmt.Printf("Response: %s\n", string(msgBuf))
}
```

---

### 4.3 JSON PROTOCOL

```go
package main

import (
	"bufio"
	"encoding/json"
	"fmt"
	"net"
)

type Message struct {
	Type    string            `json:"type"`
	Payload map[string]string `json:"payload"`
}

// ========== SERVER ==========
func handleJSONProtocol(conn net.Conn) {
	defer conn.Close()
	
	decoder := json.NewDecoder(conn)
	encoder := json.NewEncoder(conn)
	
	for {
		var msg Message
		
		// Decode JSON from connection
		err := decoder.Decode(&msg)
		if err != nil {
			fmt.Printf("Decode error: %v\n", err)
			return
		}
		
		fmt.Printf("Received: %+v\n", msg)
		
		// Create response
		response := Message{
			Type: "response",
			Payload: map[string]string{
				"status": "received",
				"echo":   msg.Payload["data"],
			},
		}
		
		// Encode response
		encoder.Encode(response)
	}
}

// ========== CLIENT ==========
func clientJSONProtocol() {
	conn, _ := net.Dial("tcp", "localhost:8080")
	defer conn.Close()
	
	encoder := json.NewEncoder(conn)
	decoder := json.NewDecoder(conn)
	
	// Send message
	msg := Message{
		Type: "request",
		Payload: map[string]string{
			"data": "Hello JSON",
		},
	}
	
	encoder.Encode(msg)
	
	// Read response
	var response Message
	decoder.Decode(&response)
	
	fmt.Printf("Response: %+v\n", response)
}
```

---

## PART 5: GRACEFUL SHUTDOWN & CLEANUP

### 5.1 HALF-CLOSE (CloseWrite)

```go
package main

import (
	"fmt"
	"net"
)

// ========== SERVER SIDE ==========
// Scenario: Server wants to stop receiving but keep sending responses

func handleHalfClose(conn net.Conn) {
	defer conn.Close()
	
	buffer := make([]byte, 1024)
	
	// Read all data from client
	for {
		n, err := conn.Read(buffer)
		if err != nil {
			// Client closed write side
			fmt.Println("Client closed write side")
			break
		}
		
		fmt.Printf("Received: %s\n", string(buffer[:n]))
	}
	
	// Server still has read side open, but client write closed
	// Now server sends final responses
	fmt.Println("Sending final response...")
	conn.Write([]byte("Processing complete"))
}

// ========== CLIENT SIDE ==========
// Scenario: Client sends all data then waits for responses

func clientHalfClose() {
	conn, _ := net.Dial("tcp", "localhost:8080")
	defer conn.Close()
	
	// Send data
	conn.Write([]byte("Message 1"))
	conn.Write([]byte("Message 2"))
	
	// Signal end of transmission
	// CloseWrite: Close writing side, keep reading
	if tcpConn, ok := conn.(*net.TCPConn); ok {
		tcpConn.CloseWrite()  // Send FIN to server
	}
	
	// Read responses
	buffer := make([]byte, 1024)
	n, _ := conn.Read(buffer)
	fmt.Printf("Response: %s\n", string(buffer[:n]))
	
	// Connection fully closes when tcpConn.Close() called (deferred)
}
```

---

### 5.2 GRACEFUL SHUTDOWN WITH SIGNAL HANDLING

```go
package main

import (
	"context"
	"fmt"
	"net"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

type Server struct {
	listener net.Listener
	clients  map[string]net.Conn
	mu       sync.RWMutex
	done     chan struct{}
}

func NewServer(addr string) (*Server, error) {
	listener, err := net.Listen("tcp", addr)
	if err != nil {
		return nil, err
	}
	
	return &Server{
		listener: listener,
		clients:  make(map[string]net.Conn),
		done:     make(chan struct{}),
	}, nil
}

func (s *Server) Start() {
	go s.acceptLoop()
}

func (s *Server) acceptLoop() {
	for {
		select {
		case <-s.done:
			fmt.Println("Accept loop closed")
			return
		default:
		}
		
		// Set deadline to allow periodic checking of done channel
		s.listener.(*net.TCPListener).SetDeadline(time.Now().Add(1 * time.Second))
		
		conn, err := s.listener.Accept()
		if err != nil {
			if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
				continue
			}
			fmt.Printf("Accept error: %v\n", err)
			return
		}
		
		s.mu.Lock()
		s.clients[conn.RemoteAddr().String()] = conn
		s.mu.Unlock()
		
		go s.handleConnection(conn)
	}
}

func (s *Server) handleConnection(conn net.Conn) {
	defer conn.Close()
	
	clientAddr := conn.RemoteAddr().String()
	fmt.Printf("Client %s connected\n", clientAddr)
	
	buffer := make([]byte, 1024)
	
	for {
		select {
		case <-s.done:
			// Server shutting down
			conn.Write([]byte("Server shutting down\n"))
			return
		default:
		}
		
		// Set read deadline to check done periodically
		conn.SetReadDeadline(time.Now().Add(1 * time.Second))
		
		n, err := conn.Read(buffer)
		if err != nil {
			if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
				continue
			}
			fmt.Printf("Client %s disconnected\n", clientAddr)
			return
		}
		
		fmt.Printf("From %s: %s\n", clientAddr, string(buffer[:n]))
		conn.Write([]byte("Echo: " + string(buffer[:n])))
	}
}

func (s *Server) Shutdown(timeout time.Duration) {
	fmt.Println("Starting graceful shutdown...")
	
	// Close accept loop
	close(s.done)
	
	// Close listener (reject new connections)
	s.listener.Close()
	
	// Notify existing clients
	s.mu.Lock()
	for addr, conn := range s.clients {
		fmt.Printf("Closing client: %s\n", addr)
		conn.Close()
	}
	s.mu.Unlock()
	
	fmt.Println("Graceful shutdown complete")
}

func main() {
	server, err := NewServer(":8080")
	if err != nil {
		panic(err)
	}
	
	fmt.Println("Server starting on :8080")
	server.Start()
	
	// Handle shutdown signals
	sigChan := make(chan os.Signal, 1)
	signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
	
	<-sigChan
	server.Shutdown(5 * time.Second)
}
```

---

## PART 6: ERROR HANDLING

### 6.1 TCP ERROR TYPES

```go
package main

import (
	"fmt"
	"net"
)

func handleErrors(conn net.Conn) {
	defer conn.Close()
	
	buffer := make([]byte, 1024)
	
	// Read with error handling
	n, err := conn.Read(buffer)
	
	if err != nil {
		// Type switch for different error types
		switch err := err.(type) {
		case net.Error:
			// Net error
			if err.Timeout() {
				// Timeout error
				fmt.Println("Timeout error")
			} else if err.Temporary() {
				// Temporary error (retry might work)
				fmt.Println("Temporary error, retrying...")
			} else {
				// Permanent error
				fmt.Println("Permanent error")
			}
			
		default:
			// io.EOF: Connection closed
			if err == io.EOF {
				fmt.Println("Connection closed by client")
			} else {
				fmt.Printf("Other error: %v\n", err)
			}
		}
		return
	}
	
	fmt.Printf("Read %d bytes: %s\n", n, string(buffer[:n]))
}
```

---

## PART 7: PRODUCTION PATTERNS

### 7.1 CONNECTION POOL FOR CLIENT

```go
package main

import (
	"fmt"
	"net"
	"sync"
	"time"
)

type ConnPool struct {
	addr      string
	maxConns  int
	available chan net.Conn
	mu        sync.Mutex
	created   int
	closed    bool
}

func NewConnPool(addr string, maxConns int) *ConnPool {
	return &ConnPool{
		addr:      addr,
		maxConns:  maxConns,
		available: make(chan net.Conn, maxConns),
	}
}

func (p *ConnPool) Get(timeout time.Duration) (net.Conn, error) {
	p.mu.Lock()
	if p.closed {
		p.mu.Unlock()
		return nil, fmt.Errorf("pool closed")
	}
	p.mu.Unlock()
	
	select {
	case conn := <-p.available:
		// Reuse existing connection
		return conn, nil
	default:
	}
	
	p.mu.Lock()
	if p.created < p.maxConns {
		p.created++
		p.mu.Unlock()
		
		// Create new connection
		dialer := net.Dialer{Timeout: timeout}
		conn, err := dialer.Dial("tcp", p.addr)
		if err != nil {
			p.mu.Lock()
			p.created--
			p.mu.Unlock()
			return nil, err
		}
		return conn, nil
	}
	p.mu.Unlock()
	
	// Wait for available connection
	select {
	case conn := <-p.available:
		return conn, nil
	case <-time.After(timeout):
		return nil, fmt.Errorf("timeout waiting for connection")
	}
}

func (p *ConnPool) Put(conn net.Conn) {
	select {
	case p.available <- conn:
		// Returned to pool
	default:
		// Pool full, close connection
		conn.Close()
	}
}

func (p *ConnPool) Close() {
	p.mu.Lock()
	p.closed = true
	p.mu.Unlock()
	
	close(p.available)
	for conn := range p.available {
		conn.Close()
	}
}
```

---

### 7.2 CONCURRENT SERVER WITH LOAD LIMITING

```go
package main

import (
	"fmt"
	"net"
	"sync"
	"time"
)

type LimitedServer struct {
	listener    net.Listener
	maxClients  int
	semaphore   chan struct{}
	connections int
	mu          sync.Mutex
}

func NewLimitedServer(addr string, maxClients int) (*LimitedServer, error) {
	listener, err := net.Listen("tcp", addr)
	if err != nil {
		return nil, err
	}
	
	// Semaphore to limit concurrent connections
	semaphore := make(chan struct{}, maxClients)
	
	return &LimitedServer{
		listener:   listener,
		maxClients: maxClients,
		semaphore:  semaphore,
	}, nil
}

func (s *LimitedServer) Accept() (net.Conn, error) {
	conn, err := s.listener.Accept()
	if err != nil {
		return nil, err
	}
	
	// Acquire semaphore slot
	s.semaphore <- struct{}{}
	
	s.mu.Lock()
	s.connections++
	fmt.Printf("Client connected. Total: %d/%d\n", s.connections, s.maxClients)
	s.mu.Unlock()
	
	return conn, nil
}

func (s *LimitedServer) ReleaseConn() {
	// Release semaphore slot
	<-s.semaphore
	
	s.mu.Lock()
	s.connections--
	fmt.Printf("Client disconnected. Total: %d/%d\n", s.connections, s.maxClients)
	s.mu.Unlock()
}

func main() {
	server, _ := NewLimitedServer(":8080", 5)
	
	for {
		conn, _ := server.Accept()
		
		go func() {
			defer server.ReleaseConn()
			defer conn.Close()
			
			buffer := make([]byte, 1024)
			n, _ := conn.Read(buffer)
			
			fmt.Printf("Received: %s\n", string(buffer[:n]))
			conn.Write([]byte("Got it"))
			
			time.Sleep(2 * time.Second)
		}()
	}
}
```

---

## PART 8: BEST PRACTICES & CHECKLIST

### Server Implementation Checklist

- [ ] **Listener Creation**: Use `net.Listen()` with proper error handling
- [ ] **Defer Close**: Always `defer listener.Close()`
- [ ] **Accept Loop**: Loop to accept multiple connections
- [ ] **Goroutines**: Handle each connection in separate goroutine
- [ ] **Timeouts**: Set read/write deadlines for slow clients
- [ ] **Buffer Management**: Create appropriate buffer sizes
- [ ] **Error Handling**: Check for io.EOF and net.Error types
- [ ] **Resource Cleanup**: Close connections properly
- [ ] **Graceful Shutdown**: Handle signals for clean shutdown
- [ ] **Testing**: Test with concurrent clients

### Client Implementation Checklist

- [ ] **Dial Timeout**: Set timeout when connecting
- [ ] **Defer Close**: Always `defer conn.Close()`
- [ ] **Connection Reuse**: Consider connection pooling for multiple requests
- [ ] **Read Timeouts**: Set read deadlines to prevent hangs
- [ ] **Error Handling**: Handle connection errors and timeouts
- [ ] **Keep-Alive**: Enable keep-alive for long connections
- [ ] **Data Format**: Define clear protocol (text/binary/json)
- [ ] **Testing**: Test server unavailability, slow responses

### TCP Tuning Options

```go
// Server-side tuning
listener.(*net.TCPListener).SetDeadline(timeout)

// Client-side tuning
dialer := net.Dialer{
    Timeout:   5 * time.Second,    // Connection timeout
    KeepAlive: 30 * time.Second,   // TCP keep-alive interval
}

// Connection tuning
tcpConn := conn.(*net.TCPConn)
tcpConn.SetNoDelay(true)          // Disable Nagle's algorithm
tcpConn.SetLinger(-1)             // Graceful close
tcpConn.SetKeepAlive(true)        // Enable keep-alive
```

---

## SUMMARY TABLE

| Concept | Function | Usage |
|---------|----------|-------|
| **Listen** | `net.Listen()` | Create server listener |
| **Accept** | `listener.Accept()` | Accept incoming connection |
| **Dial** | `net.Dial()` | Connect to server |
| **Read** | `conn.Read()` | Read from connection |
| **Write** | `conn.Write()` | Write to connection |
| **Close** | `conn.Close()` | Close connection |
| **SetDeadline** | `conn.SetDeadline()` | Set I/O timeout |
| **CloseWrite** | `conn.CloseWrite()` | Close write side (half-close) |
| **ReadFull** | `io.ReadFull()` | Read exact bytes |
| **Buffered I/O** | `bufio` | Efficient I/O operations |

---

## COMMON ERRORS & FIXES

| Error | Cause | Fix |
|-------|-------|-----|
| "address already in use" | Port still in TIME_WAIT | Wait or use SO_REUSEADDR |
| Goroutine leak | Connection never closed | Always `defer conn.Close()` |
| Slow reads | Large buffer blocking | Use buffered I/O or adjust buffer size |
| Timeout during normal operation | Deadline too aggressive | Set appropriate deadlines |
| "broken pipe" | Writing to closed connection | Check if conn is alive |
| Memory growth | Connections not released | Ensure proper cleanup |

This comprehensive guide covers TCP in Go from basics to production-ready patterns!
