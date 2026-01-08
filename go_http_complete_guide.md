# Complete Go HTTP Programming Guide: Server to Client Mastery

## TABLE OF CONTENTS
1. **Fundamentals & Theory**
2. **HTTP Server (Basic to Advanced)**
3. **HTTP Status Codes (All Types with Examples)**
4. **HTTP Client (Basic to Advanced)**
5. **Server vs Client Comparison**
6. **Production-Ready Patterns**

---

## PART 1: FUNDAMENTALS & THEORY

### 1.1 What is HTTP?
HTTP (HyperText Transfer Protocol) is a stateless, request-response protocol where:
- **Stateless**: Each request is independent
- **Request-Response**: Client sends request → Server sends response
- **Methods**: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS
- **Status Codes**: 3-digit codes indicating response type (2xx=success, 3xx=redirect, 4xx=client error, 5xx=server error)

### 1.2 Go's HTTP Architecture
```
net/http package provides:
├── Server-side: http.Server, http.Handler, http.ResponseWriter, http.Request
├── Client-side: http.Client, http.Response, http.Request
├── Routing: http.ServeMux (basic) or third-party routers
└── Utilities: Status codes, headers, cookies, etc.
```

---

## PART 2: HTTP SERVER - COMPLETE GUIDE

### 2.1 BASIC HTTP SERVER (Hello World)

```go
package main

import (
	"fmt"           // For formatted output
	"log"           // For logging errors
	"net/http"      // HTTP server/client package
)

func main() {
	// Step 1: Create a simple handler function
	// Handler takes ResponseWriter and Request pointer
	http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
		// w: ResponseWriter - write response to client
		// r: *Request - incoming request from client
		
		// Write plain text response with 200 OK (default)
		fmt.Fprintf(w, "Hello, World!\n")
	})

	// Step 2: Start server
	// Listen on port 8080, nil = use default mux
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**Line-by-line Explanation:**
- `http.HandleFunc("/hello", handler)` - Register handler for path "/hello"
- `w http.ResponseWriter` - Interface to write HTTP response
- `r *http.Request` - Request object containing client data
- `fmt.Fprintf(w, ...)` - Write to response writer (like fmt.Printf but to ResponseWriter)
- `http.ListenAndServe(":8080", nil)` - Start server, nil uses DefaultServeMux
- `log.Fatal()` - Print error and exit if server fails

**Running:**
```bash
go run main.go
# In another terminal:
curl http://localhost:8080/hello
# Output: Hello, World!
```

---

### 2.2 REQUEST OBJECT EXPLORATION

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func inspectRequest(w http.ResponseWriter, r *http.Request) {
	// r.Method: HTTP method (GET, POST, PUT, DELETE, etc.)
	fmt.Fprintf(w, "Method: %s\n", r.Method)
	
	// r.URL: Full URL parsed into components
	fmt.Fprintf(w, "URL Path: %s\n", r.URL.Path)
	fmt.Fprintf(w, "Full URL: %s\n", r.URL.String())
	
	// r.URL.Query(): URL query parameters (e.g., ?name=john&age=30)
	name := r.URL.Query().Get("name")
	age := r.URL.Query().Get("age")
	fmt.Fprintf(w, "Query - name: %s, age: %s\n", name, age)
	
	// r.Header: HTTP headers as key-value map
	fmt.Fprintf(w, "User-Agent: %s\n", r.Header.Get("User-Agent"))
	fmt.Fprintf(w, "Content-Type: %s\n", r.Header.Get("Content-Type"))
	
	// r.Body: Raw request body (for POST/PUT)
	fmt.Fprintf(w, "Content-Length: %d\n", r.ContentLength)
	
	// r.Host: Host header value
	fmt.Fprintf(w, "Host: %s\n", r.Host)
	
	// r.RemoteAddr: Client IP address
	fmt.Fprintf(w, "Client IP: %s\n", r.RemoteAddr)
}

func main() {
	http.HandleFunc("/inspect", inspectRequest)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**Testing:**
```bash
curl "http://localhost:8080/inspect?name=john&age=30" \
  -H "User-Agent: CustomAgent" \
  -H "Content-Type: application/json"

# Output:
# Method: GET
# URL Path: /inspect
# Full URL: http://localhost:8080/inspect?name=john&age=30
# Query - name: john, age: 30
# User-Agent: CustomAgent
# Content-Type: application/json
# Content-Length: 0
# Host: localhost:8080
# Client IP: 127.0.0.1:XXXXX
```

---

### 2.3 RESPONSE WRITER & STATUS CODES

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func successResponse(w http.ResponseWriter, r *http.Request) {
	// Step 1: Set status code (must be before writing body)
	w.WriteHeader(http.StatusOK) // 200 OK
	
	// Step 2: Set Content-Type header
	w.Header().Set("Content-Type", "text/plain; charset=utf-8")
	
	// Step 3: Write response body
	fmt.Fprintf(w, "Success response\n")
}

func createdResponse(w http.ResponseWriter, r *http.Request) {
	// Set status BEFORE writing body
	w.WriteHeader(http.StatusCreated) // 201 Created
	w.Header().Set("Content-Type", "application/json")
	
	// Write JSON response
	fmt.Fprintf(w, `{"id": 1, "message": "Resource created"}`)
}

func notFoundResponse(w http.ResponseWriter, r *http.Request) {
	// 404 Not Found
	w.WriteHeader(http.StatusNotFound)
	w.Header().Set("Content-Type", "text/plain")
	fmt.Fprintf(w, "Resource not found\n")
}

func serverErrorResponse(w http.ResponseWriter, r *http.Request) {
	// 500 Internal Server Error
	w.WriteHeader(http.StatusInternalServerError)
	w.Header().Set("Content-Type", "text/plain")
	fmt.Fprintf(w, "Something went wrong\n")
}

func main() {
	http.HandleFunc("/success", successResponse)
	http.HandleFunc("/created", createdResponse)
	http.HandleFunc("/notfound", notFoundResponse)
	http.HandleFunc("/error", serverErrorResponse)
	
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**CRITICAL: Order matters!**
```go
// ✓ CORRECT
w.WriteHeader(http.StatusOK)
w.Header().Set("X-Custom", "value")
fmt.Fprintf(w, "body")

// ✗ WRONG - header write ignored after WriteHeader
w.Header().Set("X-Custom", "value")
w.WriteHeader(http.StatusOK)  // This sets status but headers after are ignored
```

---

### 2.4 ALL HTTP STATUS CODES WITH EXAMPLES

#### **1xx - Informational (Request received, processing continues)**

```go
// 100 Continue
// 101 Switching Protocols (WebSocket)
// 102 Processing (WebDAV)
// 103 Early Hints (experimental)

func handleSwitchingProtocols(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusSwitchingProtocols)
	// Usually used for WebSocket upgrades
}
```

#### **2xx - Success (Request succeeded)**

```go
// 200 OK - Standard successful response
func handle200(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, `{"status": "ok"}`)
}

// 201 Created - Resource created successfully
func handle201(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusCreated)
	w.Header().Set("Location", "/resource/123") // URL of created resource
	fmt.Fprintf(w, `{"id": 123, "created": true}`)
}

// 202 Accepted - Request accepted but not completed
func handle202(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusAccepted)
	fmt.Fprintf(w, `{"status": "processing"}`)
}

// 204 No Content - Success but no content to send
func handle204(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusNoContent)
	// NO body written
}

// 206 Partial Content - Range request success
func handle206(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusPartialContent)
	w.Header().Set("Content-Range", "bytes 0-1023/2048")
	fmt.Fprintf(w, "partial content...")
}
```

#### **3xx - Redirection (Further action needed)**

```go
// 300 Multiple Choices
func handle300(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusMultipleChoices)
	fmt.Fprintf(w, `[{"url": "/resource/1"}, {"url": "/resource/2"}]`)
}

// 301 Moved Permanently - Permanent redirect
func handle301(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Location", "/new-path")
	w.WriteHeader(http.StatusMovedPermanently)
	fmt.Fprintf(w, "Redirecting...")
}

// 302 Found - Temporary redirect
func handle302(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Location", "/temp-path")
	w.WriteHeader(http.StatusFound)
	fmt.Fprintf(w, "Redirecting temporarily...")
}

// 304 Not Modified - Resource unchanged since last request
func handle304(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusNotModified)
	// No body, client uses cached version
}

// 307 Temporary Redirect - Preserve method
func handle307(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Location", "/temp")
	w.WriteHeader(http.StatusTemporaryRedirect)
}

// 308 Permanent Redirect - Preserve method
func handle308(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Location", "/new")
	w.WriteHeader(http.StatusPermanentRedirect)
}
```

#### **4xx - Client Error (Client's fault)**

```go
// 400 Bad Request - Malformed request
func handle400(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusBadRequest)
	fmt.Fprintf(w, `{"error": "Invalid request format"}`)
}

// 401 Unauthorized - Authentication required
func handle401(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("WWW-Authenticate", `Bearer realm="restricted"`)
	w.WriteHeader(http.StatusUnauthorized)
	fmt.Fprintf(w, `{"error": "Authentication required"}`)
}

// 403 Forbidden - Authenticated but no permission
func handle403(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusForbidden)
	fmt.Fprintf(w, `{"error": "Permission denied"}`)
}

// 404 Not Found - Resource doesn't exist
func handle404(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusNotFound)
	fmt.Fprintf(w, `{"error": "Resource not found"}`)
}

// 405 Method Not Allowed
func handle405(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Allow", "GET, POST")
	w.WriteHeader(http.StatusMethodNotAllowed)
	fmt.Fprintf(w, `{"error": "Method not allowed"}`)
}

// 409 Conflict - Resource conflict
func handle409(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusConflict)
	fmt.Fprintf(w, `{"error": "Resource already exists"}`)
}

// 410 Gone - Resource permanently deleted
func handle410(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusGone)
	fmt.Fprintf(w, `{"error": "Resource permanently deleted"}`)
}

// 413 Payload Too Large
func handle413(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusRequestEntityTooLarge)
	fmt.Fprintf(w, `{"error": "Request body too large"}`)
}

// 429 Too Many Requests - Rate limiting
func handle429(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Retry-After", "60")
	w.WriteHeader(http.StatusTooManyRequests)
	fmt.Fprintf(w, `{"error": "Rate limit exceeded"}`)
}

// 418 I'm a Teapot - Easter egg
func handle418(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusTeapot)
	fmt.Fprintf(w, "☕ I'm a teapot!")
}
```

#### **5xx - Server Error (Server's fault)**

```go
// 500 Internal Server Error - Generic server error
func handle500(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusInternalServerError)
	fmt.Fprintf(w, `{"error": "Internal server error"}`)
}

// 501 Not Implemented - Feature not implemented
func handle501(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusNotImplemented)
	fmt.Fprintf(w, `{"error": "Feature not implemented"}`)
}

// 502 Bad Gateway - Invalid response from upstream
func handle502(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusBadGateway)
	fmt.Fprintf(w, `{"error": "Bad gateway"}`)
}

// 503 Service Unavailable - Temporarily unavailable
func handle503(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Retry-After", "300")
	w.WriteHeader(http.StatusServiceUnavailable)
	fmt.Fprintf(w, `{"error": "Service unavailable"}`)
}

// 504 Gateway Timeout - Upstream took too long
func handle504(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusGatewayTimeout)
	fmt.Fprintf(w, `{"error": "Gateway timeout"}`)
}
```

---

### 2.5 READING REQUEST BODY

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
)

// User struct for JSON parsing
type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
	Age   int    `json:"age"`
}

// Read plain text body
func handlePlainText(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}
	
	// Method 1: Read all body at once (small payloads)
	body, err := io.ReadAll(r.Body)
	if err != nil {
		w.WriteHeader(http.StatusBadRequest)
		fmt.Fprintf(w, "Error reading body: %v", err)
		return
	}
	defer r.Body.Close()
	
	fmt.Fprintf(w, "Received: %s\n", string(body))
}

// Read JSON body
func handleJSON(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}
	
	// Decode JSON directly from request body
	var user User
	decoder := json.NewDecoder(r.Body)
	defer r.Body.Close()
	
	if err := decoder.Decode(&user); err != nil {
		w.WriteHeader(http.StatusBadRequest)
		fmt.Fprintf(w, `{"error": "Invalid JSON"}`)
		return
	}
	
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, `{"received": true, "name": "%s", "email": "%s"}`, user.Name, user.Email)
}

// Handle form data
func handleFormData(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}
	
	// Parse form (max 10MB)
	r.ParseMultipartForm(10 << 20) // 10MB
	
	// Access form values
	name := r.FormValue("name")
	email := r.FormValue("email")
	
	// Access file upload
	file, handler, err := r.FormFile("profile_pic")
	if err != nil {
		w.WriteHeader(http.StatusBadRequest)
		fmt.Fprintf(w, "Error getting file: %v", err)
		return
	}
	defer file.Close()
	
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Received form: name=%s, email=%s, file=%s\n", 
		name, email, handler.Filename)
}

func main() {
	http.HandleFunc("/text", handlePlainText)
	http.HandleFunc("/json", handleJSON)
	http.HandleFunc("/form", handleFormData)
	
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**Testing:**
```bash
# Plain text
curl -X POST http://localhost:8080/text -d "Hello Server" --data-raw

# JSON
curl -X POST http://localhost:8080/json \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com","age":30}'

# Form data with file
curl -X POST http://localhost:8080/form \
  -F "name=John" \
  -F "email=john@example.com" \
  -F "profile_pic=@/path/to/image.jpg"
```

---

### 2.6 CUSTOM HANDLER STRUCT (PRODUCTION PATTERN)

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
)

// CustomHandler implements http.Handler interface
type CustomHandler struct {
	db string // Any dependencies (database, cache, etc.)
}

// ServeHTTP implements http.Handler interface
// Required: method name, parameter types, and receiver must match exactly
func (h *CustomHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// Router logic
	if r.URL.Path == "/users" && r.Method == http.MethodGet {
		h.getUsers(w, r)
	} else if r.URL.Path == "/users" && r.Method == http.MethodPost {
		h.createUser(w, r)
	} else {
		w.WriteHeader(http.StatusNotFound)
		fmt.Fprintf(w, "Not found")
	}
}

func (h *CustomHandler) getUsers(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	users := []map[string]interface{}{
		{"id": 1, "name": "Alice"},
		{"id": 2, "name": "Bob"},
	}
	json.NewEncoder(w).Encode(users)
}

func (h *CustomHandler) createUser(w http.ResponseWriter, r *http.Request) {
	var user map[string]string
	json.NewDecoder(r.Body).Decode(&user)
	defer r.Body.Close()
	
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	fmt.Fprintf(w, `{"id": 3, "name": "%s"}`, user["name"])
}

func main() {
	handler := &CustomHandler{db: "connected"}
	
	// Use handler directly (not HandleFunc)
	http.Handle("/users", handler)
	
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

### 2.7 MIDDLEWARE PATTERN

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

// Middleware wraps a handler and adds functionality
func loggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Before handler
		start := time.Now()
		fmt.Printf("[%s] %s %s\n", time.Now().Format("15:04:05"), r.Method, r.URL.Path)
		
		// Call next handler
		next.ServeHTTP(w, r)
		
		// After handler
		duration := time.Since(start)
		fmt.Printf("Request took: %v\n", duration)
	})
}

func authMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Check authentication
		token := r.Header.Get("Authorization")
		if token == "" {
			w.WriteHeader(http.StatusUnauthorized)
			fmt.Fprintf(w, "Missing Authorization header")
			return
		}
		
		// Token is valid, continue
		next.ServeHTTP(w, r)
	})
}

// Chain multiple middlewares
func chainMiddleware(handler http.Handler, middlewares ...func(http.Handler) http.Handler) http.Handler {
	for _, middleware := range middlewares {
		handler = middleware(handler)
	}
	return handler
}

func protectedHandler(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Protected content accessed!")
}

func main() {
	// Apply middlewares to protected handler
	handler := http.HandlerFunc(protectedHandler)
	handler = chainMiddleware(handler, loggingMiddleware, authMiddleware)
	
	http.Handle("/protected", handler)
	
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

**Test:**
```bash
# Without token - 401
curl http://localhost:8080/protected

# With token - 200
curl -H "Authorization: Bearer token123" http://localhost:8080/protected
```

---

### 2.8 CONTEXT AND TIMEOUTS

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"time"
)

// Handler that respects context deadline
func slowHandler(w http.ResponseWriter, r *http.Request) {
	// Get context from request
	ctx := r.Context()
	
	// Start an operation
	resultChan := make(chan string)
	
	go func() {
		time.Sleep(5 * time.Second)
		resultChan <- "Operation complete"
	}()
	
	select {
	case result := <-resultChan:
		// Operation completed in time
		w.WriteHeader(http.StatusOK)
		fmt.Fprintf(w, result)
		
	case <-ctx.Done():
		// Context cancelled (client disconnected or timeout)
		err := ctx.Err()
		w.WriteHeader(http.StatusRequestTimeout)
		fmt.Fprintf(w, "Request timeout: %v", err)
	}
}

func main() {
	http.HandleFunc("/slow", slowHandler)
	
	// Configure server with timeouts
	server := &http.Server{
		Addr:         ":8080",
		Handler:      nil,
		ReadTimeout:  5 * time.Second,  // Max time to read request
		WriteTimeout: 5 * time.Second,  // Max time to write response
		IdleTimeout:  60 * time.Second, // Max idle connection duration
	}
	
	log.Fatal(server.ListenAndServe())
}
```

---

### 2.9 COMPLETE PRODUCTION SERVER EXAMPLE

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"time"
)

type APIError struct {
	Error  string `json:"error"`
	Status int    `json:"status"`
}

type APIResponse struct {
	Data      interface{} `json:"data,omitempty"`
	Error     string      `json:"error,omitempty"`
	Status    int         `json:"status"`
	Timestamp string      `json:"timestamp"`
}

func jsonResponse(w http.ResponseWriter, status int, data interface{}) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	
	response := APIResponse{
		Status:    status,
		Timestamp: time.Now().UTC().Format(time.RFC3339),
	}
	
	if data != nil {
		response.Data = data
	}
	
	json.NewEncoder(w).Encode(response)
}

func jsonError(w http.ResponseWriter, status int, errMsg string) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	
	response := APIResponse{
		Error:     errMsg,
		Status:    status,
		Timestamp: time.Now().UTC().Format(time.RFC3339),
	}
	
	json.NewEncoder(w).Encode(response)
}

func loggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		log.Printf("[%s] %s %s from %s", 
			time.Now().Format("2006-01-02 15:04:05"),
			r.Method, r.URL.Path, r.RemoteAddr)
		
		next.ServeHTTP(w, r)
		
		log.Printf("Completed in %v", time.Since(start))
	})
}

func healthCheckHandler(w http.ResponseWriter, r *http.Request) {
	jsonResponse(w, http.StatusOK, map[string]interface{}{
		"status": "healthy",
		"uptime": time.Now().Unix(),
	})
}

func getUserHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodGet {
		jsonError(w, http.StatusMethodNotAllowed, "Only GET allowed")
		return
	}
	
	userID := r.URL.Query().Get("id")
	if userID == "" {
		jsonError(w, http.StatusBadRequest, "Missing id parameter")
		return
	}
	
	// Simulate database lookup
	user := map[string]interface{}{
		"id":    userID,
		"name":  "John Doe",
		"email": "john@example.com",
	}
	
	jsonResponse(w, http.StatusOK, user)
}

func createUserHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		jsonError(w, http.StatusMethodNotAllowed, "Only POST allowed")
		return
	}
	
	// Parse JSON body
	var user map[string]interface{}
	if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
		jsonError(w, http.StatusBadRequest, "Invalid JSON")
		return
	}
	defer r.Body.Close()
	
	// Simulate database save
	user["id"] = "123"
	
	jsonResponse(w, http.StatusCreated, user)
}

func notFoundHandler(w http.ResponseWriter, r *http.Request) {
	jsonError(w, http.StatusNotFound, "Endpoint not found")
}

func main() {
	mux := http.NewServeMux()
	
	// Register routes
	mux.HandleFunc("/health", healthCheckHandler)
	mux.HandleFunc("/users/get", getUserHandler)
	mux.HandleFunc("/users/create", createUserHandler)
	mux.HandleFunc("/", notFoundHandler)
	
	// Wrap mux with middleware
	handler := loggingMiddleware(mux)
	
	// Configure server
	server := &http.Server{
		Addr:         ":8080",
		Handler:      handler,
		ReadTimeout:  10 * time.Second,
		WriteTimeout: 10 * time.Second,
		IdleTimeout:  60 * time.Second,
	}
	
	log.Printf("Server starting on %s", server.Addr)
	log.Fatal(server.ListenAndServe())
}
```

---

## PART 3: HTTP CLIENT - COMPLETE GUIDE

### 3.1 BASIC GET REQUEST

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {
	// Simple GET request using default client
	resp, err := http.Get("https://jsonplaceholder.typicode.com/users/1")
	if err != nil {
		log.Fatal("Error:", err)
	}
	defer resp.Body.Close() // Always close body
	
	// Read response status
	fmt.Printf("Status: %s (%d)\n", resp.Status, resp.StatusCode)
	
	// Read response headers
	fmt.Printf("Content-Type: %s\n", resp.Header.Get("Content-Type"))
	fmt.Printf("Server: %s\n", resp.Header.Get("Server"))
	
	// Read response body
	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatal("Error reading body:", err)
	}
	
	fmt.Printf("Body:\n%s\n", string(body))
}
```

---

### 3.2 POST REQUEST

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
}

func main() {
	// Create data
	user := User{
		Name:  "John Doe",
		Email: "john@example.com",
	}
	
	// Marshal to JSON
	jsonData, err := json.Marshal(user)
	if err != nil {
		log.Fatal("Error marshaling JSON:", err)
	}
	
	// Create request body from JSON bytes
	// bytes.NewBuffer: Converts byte slice to io.Reader
	bodyReader := bytes.NewBuffer(jsonData)
	
	// POST request
	// 3rd param is io.Reader (body content)
	resp, err := http.Post(
		"https://jsonplaceholder.typicode.com/users",
		"application/json",
		bodyReader,
	)
	if err != nil {
		log.Fatal("Error:", err)
	}
	defer resp.Body.Close()
	
	// Read response
	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Status: %d\n", resp.StatusCode)
	fmt.Printf("Response: %s\n", string(body))
}
```

---

### 3.3 CUSTOM REQUEST (Advanced)

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {
	// Create custom request with more control
	req, err := http.NewRequest(
		http.MethodGet,                                    // Method
		"https://jsonplaceholder.typicode.com/users/1",  // URL
		nil,                                               // Body (nil for GET)
	)
	if err != nil {
		log.Fatal("Error creating request:", err)
	}
	
	// Add custom headers
	req.Header.Set("User-Agent", "MyCustomClient/1.0")
	req.Header.Set("Accept", "application/json")
	req.Header.Set("Authorization", "Bearer token123")
	
	// Add query parameters
	q := req.URL.Query()
	q.Add("filter", "active")
	q.Add("sort", "name")
	req.URL.RawQuery = q.Encode()
	
	// Execute request with default client
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Fatal("Error:", err)
	}
	defer resp.Body.Close()
	
	// Read response
	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Status: %d\n", resp.StatusCode)
	fmt.Printf("Response: %s\n", string(body))
}
```

---

### 3.4 CUSTOM HTTP CLIENT WITH TIMEOUTS

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"time"
)

func main() {
	// Create custom client with timeouts
	// Timeout: max time for entire request (connect + read + write)
	client := &http.Client{
		Timeout: 5 * time.Second,
	}
	
	// Alternative: Fine-grained timeouts using Transport
	client = &http.Client{
		Timeout: 10 * time.Second,
		Transport: &http.Transport{
			// MaxIdleConns: Max total idle connections
			MaxIdleConns: 100,
			
			// MaxIdleConnsPerHost: Max idle per host
			MaxIdleConnsPerHost: 10,
			
			// IdleConnTimeout: Close idle connections after this duration
			IdleConnTimeout: 90 * time.Second,
			
			// DialTimeout: Max time to establish TCP connection
			// (Can't set directly, use Dialer)
		},
	}
	
	resp, err := client.Get("https://jsonplaceholder.typicode.com/users/1")
	if err != nil {
		// Check if timeout
		if _, ok := err.(net.Error); ok {
			log.Fatal("Network timeout occurred")
		}
		log.Fatal("Error:", err)
	}
	defer resp.Body.Close()
	
	body, _ := io.ReadAll(resp.Body)
	fmt.Printf("Status: %d\nBody: %s\n", resp.StatusCode, string(body))
}
```

---

### 3.5 HANDLING RESPONSE STATUS CODES

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {
	resp, err := http.Get("https://httpbin.org/status/404")
	if err != nil {
		log.Fatal("Error:", err)
	}
	defer resp.Body.Close()
	
	// Status code (without status text)
	fmt.Printf("Status Code: %d\n", resp.StatusCode)
	
	// Status (with status text)
	fmt.Printf("Status: %s\n", resp.Status)
	
	// Handle different status codes
	switch resp.StatusCode {
	case http.StatusOK:
		fmt.Println("Success!")
		
	case http.StatusCreated:
		fmt.Println("Resource created!")
		
	case http.StatusBadRequest:
		body, _ := io.ReadAll(resp.Body)
		fmt.Printf("Bad request: %s\n", string(body))
		
	case http.StatusUnauthorized:
		fmt.Println("Authentication required!")
		
	case http.StatusForbidden:
		fmt.Println("Access forbidden!")
		
	case http.StatusNotFound:
		fmt.Println("Resource not found!")
		
	case http.StatusInternalServerError:
		fmt.Println("Server error!")
		
	case http.StatusServiceUnavailable:
		retryAfter := resp.Header.Get("Retry-After")
		fmt.Printf("Service unavailable. Retry after: %s\n", retryAfter)
		
	default:
		if resp.StatusCode >= 500 {
			fmt.Println("Server error")
		} else if resp.StatusCode >= 400 {
			fmt.Println("Client error")
		}
	}
}
```

---

### 3.6 CLIENT - ALL HTTP METHODS

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"net/url"
)

func main() {
	client := &http.Client{}
	baseURL := "https://httpbin.org"
	
	// ===== GET =====
	fmt.Println("=== GET ===")
	resp, _ := client.Get(baseURL + "/get?name=john")
	fmt.Printf("GET: %d\n\n", resp.StatusCode)
	resp.Body.Close()
	
	// ===== POST =====
	fmt.Println("=== POST ===")
	data := map[string]string{"name": "john", "age": "30"}
	jsonData, _ := json.Marshal(data)
	resp, _ = client.Post(baseURL+"/post", "application/json", bytes.NewBuffer(jsonData))
	fmt.Printf("POST: %d\n\n", resp.StatusCode)
	resp.Body.Close()
	
	// ===== POST FORM =====
	fmt.Println("=== POST FORM ===")
	formData := url.Values{}
	formData.Set("name", "john")
	formData.Set("age", "30")
	resp, _ = client.PostForm(baseURL+"/post", formData)
	fmt.Printf("POST FORM: %d\n\n", resp.StatusCode)
	resp.Body.Close()
	
	// ===== PUT =====
	fmt.Println("=== PUT ===")
	data = map[string]string{"id": "123", "name": "jane"}
	jsonData, _ = json.Marshal(data)
	req, _ := http.NewRequest(http.MethodPut, baseURL+"/put", bytes.NewBuffer(jsonData))
	req.Header.Set("Content-Type", "application/json")
	resp, _ = client.Do(req)
	fmt.Printf("PUT: %d\n\n", resp.StatusCode)
	resp.Body.Close()
	
	// ===== PATCH =====
	fmt.Println("=== PATCH ===")
	data = map[string]string{"name": "jane"}
	jsonData, _ = json.Marshal(data)
	req, _ = http.NewRequest(http.MethodPatch, baseURL+"/patch", bytes.NewBuffer(jsonData))
	req.Header.Set("Content-Type", "application/json")
	resp, _ = client.Do(req)
	fmt.Printf("PATCH: %d\n\n", resp.StatusCode)
	resp.Body.Close()
	
	// ===== DELETE =====
	fmt.Println("=== DELETE ===")
	req, _ = http.NewRequest(http.MethodDelete, baseURL+"/delete?id=123", nil)
	resp, _ = client.Do(req)
	fmt.Printf("DELETE: %d\n\n", resp.StatusCode)
	resp.Body.Close()
	
	// ===== HEAD =====
	fmt.Println("=== HEAD ===")
	resp, _ = client.Head(baseURL + "/get")
	fmt.Printf("HEAD: %d\n", resp.StatusCode)
	resp.Body.Close()
	
	// ===== OPTIONS =====
	fmt.Println("=== OPTIONS ===")
	req, _ = http.NewRequest(http.MethodOptions, baseURL+"/", nil)
	resp, _ = client.Do(req)
	fmt.Printf("OPTIONS: %d\n", resp.StatusCode)
	fmt.Printf("Allow: %s\n", resp.Header.Get("Allow"))
	resp.Body.Close()
}
```

---

### 3.7 ERROR HANDLING & RETRIES

```go
package main

import (
	"errors"
	"fmt"
	"io"
	"log"
	"net"
	"net/http"
	"time"
)

func requestWithRetry(url string, maxRetries int) error {
	client := &http.Client{
		Timeout: 5 * time.Second,
	}
	
	for attempt := 1; attempt <= maxRetries; attempt++ {
		resp, err := client.Get(url)
		
		// Handle connection errors
		if err != nil {
			// Check error type
			var netErr net.Error
			if errors.As(err, &netErr) {
				if netErr.Timeout() {
					fmt.Printf("Attempt %d: Timeout\n", attempt)
				} else {
					fmt.Printf("Attempt %d: Network error\n", attempt)
				}
			}
			
			// Retry with backoff
			if attempt < maxRetries {
				backoff := time.Duration(attempt) * time.Second
				fmt.Printf("Retrying after %v\n", backoff)
				time.Sleep(backoff)
				continue
			}
			return err
		}
		
		defer resp.Body.Close()
		
		// Handle HTTP errors
		if resp.StatusCode >= 500 {
			// Server error, retry
			fmt.Printf("Attempt %d: Server error %d\n", attempt, resp.StatusCode)
			if attempt < maxRetries {
				time.Sleep(time.Duration(attempt) * time.Second)
				continue
			}
			return fmt.Errorf("server error: %d", resp.StatusCode)
		}
		
		if resp.StatusCode == 429 {
			// Rate limited, wait based on Retry-After header
			retryAfter := resp.Header.Get("Retry-After")
			fmt.Printf("Rate limited. Retry after: %s\n", retryAfter)
			time.Sleep(5 * time.Second)
			continue
		}
		
		// Success
		body, _ := io.ReadAll(resp.Body)
		fmt.Printf("Success: %d\nBody: %s\n", resp.StatusCode, string(body))
		return nil
	}
	
	return errors.New("max retries exceeded")
}

func main() {
	err := requestWithRetry("https://jsonplaceholder.typicode.com/users/1", 3)
	if err != nil {
		log.Fatal("Failed:", err)
	}
}
```

---

### 3.8 COMPLETE PRODUCTION CLIENT

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"
)

// HTTPClient wrapper for reusability
type HTTPClient struct {
	client *http.Client
	baseURL string
}

// NewHTTPClient creates client with default settings
func NewHTTPClient(baseURL string, timeout time.Duration) *HTTPClient {
	client := &http.Client{
		Timeout: timeout,
		Transport: &http.Transport{
			MaxIdleConns:        100,
			MaxIdleConnsPerHost: 10,
			IdleConnTimeout:     90 * time.Second,
		},
	}
	
	return &HTTPClient{
		client:  client,
		baseURL: baseURL,
	}
}

// Request wrapper for all HTTP methods
func (h *HTTPClient) Request(method string, endpoint string, body interface{}, headers map[string]string) (int, []byte, error) {
	var bodyReader io.Reader
	
	// Prepare body
	if body != nil {
		jsonData, err := json.Marshal(body)
		if err != nil {
			return 0, nil, fmt.Errorf("error marshaling body: %w", err)
		}
		bodyReader = bytes.NewBuffer(jsonData)
	}
	
	// Create request
	req, err := http.NewRequest(method, h.baseURL+endpoint, bodyReader)
	if err != nil {
		return 0, nil, fmt.Errorf("error creating request: %w", err)
	}
	
	// Set default headers
	req.Header.Set("Content-Type", "application/json")
	
	// Set custom headers
	for k, v := range headers {
		req.Header.Set(k, v)
	}
	
	// Execute request
	resp, err := h.client.Do(req)
	if err != nil {
		return 0, nil, fmt.Errorf("error making request: %w", err)
	}
	defer resp.Body.Close()
	
	// Read response
	respBody, err := io.ReadAll(resp.Body)
	if err != nil {
		return resp.StatusCode, nil, fmt.Errorf("error reading response: %w", err)
	}
	
	return resp.StatusCode, respBody, nil
}

// Helper methods
func (h *HTTPClient) Get(endpoint string) (int, []byte, error) {
	return h.Request(http.MethodGet, endpoint, nil, nil)
}

func (h *HTTPClient) Post(endpoint string, body interface{}) (int, []byte, error) {
	return h.Request(http.MethodPost, endpoint, body, nil)
}

func (h *HTTPClient) Put(endpoint string, body interface{}) (int, []byte, error) {
	return h.Request(http.MethodPut, endpoint, body, nil)
}

func (h *HTTPClient) Delete(endpoint string) (int, []byte, error) {
	return h.Request(http.MethodDelete, endpoint, nil, nil)
}

// Usage
func main() {
	client := NewHTTPClient("https://jsonplaceholder.typicode.com", 10*time.Second)
	
	// GET
	status, body, err := client.Get("/users/1")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("GET: %d\n%s\n\n", status, string(body))
	
	// POST
	data := map[string]interface{}{
		"title":  "New Post",
		"body":   "This is a new post",
		"userId": 1,
	}
	status, body, err = client.Post("/posts", data)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("POST: %d\n%s\n\n", status, string(body))
	
	// PUT
	updateData := map[string]interface{}{
		"id":     1,
		"title":  "Updated Post",
		"body":   "Updated content",
		"userId": 1,
	}
	status, body, err = client.Put("/posts/1", updateData)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("PUT: %d\n%s\n\n", status, string(body))
	
	// DELETE
	status, body, err = client.Delete("/posts/1")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("DELETE: %d\n%s\n", status, string(body))
}
```

---

## PART 4: SERVER vs CLIENT COMPARISON

### 4.1 SIDE-BY-SIDE EXAMPLE

```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"
)

type Message struct {
	Text      string `json:"text"`
	Timestamp string `json:"timestamp"`
}

// ========== SERVER SIDE ==========

func serverMessageHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method == http.MethodPost {
		// Read request body (client sends)
		var msg Message
		if err := json.NewDecoder(r.Body).Decode(&msg); err != nil {
			w.WriteHeader(http.StatusBadRequest)
			fmt.Fprintf(w, `{"error": "Invalid JSON"}`)
			return
		}
		defer r.Body.Close()
		
		// Process message
		msg.Timestamp = time.Now().Format(time.RFC3339)
		
		// Send response back to client
		w.Header().Set("Content-Type", "application/json")
		w.WriteHeader(http.StatusOK)
		json.NewEncoder(w).Encode(map[string]interface{}{
			"received": true,
			"message":  msg,
		})
	}
}

func serverMain() {
	http.HandleFunc("/message", serverMessageHandler)
	log.Fatal(http.ListenAndServe(":8080", nil))
}

// ========== CLIENT SIDE ==========

func client() {
	time.Sleep(1 * time.Second) // Give server time to start
	
	// Create message to send
	msg := Message{
		Text: "Hello from client!",
	}
	
	// Marshal to JSON (serialize)
	jsonData, _ := json.Marshal(msg)
	
	// Create POST request (send to server)
	resp, err := http.Post(
		"http://localhost:8080/message",
		"application/json",
		bytes.NewBuffer(jsonData),
	)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	
	// Read response from server
	body, _ := io.ReadAll(resp.Body)
	
	// Parse response JSON
	var response map[string]interface{}
	json.Unmarshal(body, &response)
	
	fmt.Printf("Server response status: %d\n", resp.StatusCode)
	fmt.Printf("Server response: %v\n", response)
}

func main() {
	// Uncomment to run server or client
	// serverMain()
	// client()
}
```

---

### 4.2 COMPARATIVE TABLE

| Aspect | Server | Client |
|--------|--------|--------|
| **Package** | `net/http` | `net/http` |
| **Create Handler** | `http.HandleFunc()` or `http.Handler` | `http.NewRequest()` |
| **Listen** | `http.ListenAndServe()` | Not applicable |
| **Read Request** | `r.Body`, `r.Header` | `resp.Body`, `resp.Header` |
| **Send Response** | `w.WriteHeader()`, `w.Write()` | Request is sent automatically |
| **Status Codes** | Server decides (writes with `WriteHeader`) | Server sends, client reads from `resp.StatusCode` |
| **Headers** | Server writes with `w.Header().Set()` | Client reads with `resp.Header.Get()` |
| **Request Method** | `r.Method` (receives) | `http.MethodGet`, etc. (sends) |
| **Timeout** | `Server.ReadTimeout`, `Server.WriteTimeout` | `http.Client.Timeout` |
| **Concurrency** | Goroutine per request (automatic) | Manual goroutines if needed |

---

## PART 5: PRODUCTION PATTERNS

### 5.1 COMPLETE FULL-STACK EXAMPLE

**server.go:**
```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"sync"
	"time"
)

// In-memory "database"
var (
	users = make(map[int]map[string]interface{})
	userMu = &sync.RWMutex{}
	nextID = 1
)

type User struct {
	ID    int    `json:"id"`
	Name  string `json:"name"`
	Email string `json:"email"`
}

func handleGetUser(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodGet {
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}
	
	id := r.URL.Query().Get("id")
	if id == "" {
		w.WriteHeader(http.StatusBadRequest)
		fmt.Fprintf(w, `{"error": "Missing id"}`)
		return
	}
	
	var userID int
	_, err := fmt.Sscanf(id, "%d", &userID)
	if err != nil {
		w.WriteHeader(http.StatusBadRequest)
		fmt.Fprintf(w, `{"error": "Invalid id"}`)
		return
	}
	
	userMu.RLock()
	user, exists := users[userID]
	userMu.RUnlock()
	
	if !exists {
		w.WriteHeader(http.StatusNotFound)
		fmt.Fprintf(w, `{"error": "User not found"}`)
		return
	}
	
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(user)
}

func handleCreateUser(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}
	
	var user User
	if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
		w.WriteHeader(http.StatusBadRequest)
		fmt.Fprintf(w, `{"error": "Invalid JSON"}`)
		return
	}
	defer r.Body.Close()
	
	if user.Name == "" || user.Email == "" {
		w.WriteHeader(http.StatusBadRequest)
		fmt.Fprintf(w, `{"error": "Name and email required"}`)
		return
	}
	
	userMu.Lock()
	user.ID = nextID
	nextID++
	users[user.ID] = map[string]interface{}{
		"id":    user.ID,
		"name":  user.Name,
		"email": user.Email,
	}
	userMu.Unlock()
	
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(user)
}

func runServer() {
	http.HandleFunc("/users/get", handleGetUser)
	http.HandleFunc("/users/create", handleCreateUser)
	
	server := &http.Server{
		Addr:         ":8080",
		Handler:      nil,
		ReadTimeout:  10 * time.Second,
		WriteTimeout: 10 * time.Second,
		IdleTimeout:  60 * time.Second,
	}
	
	log.Printf("Server starting on %s", server.Addr)
	log.Fatal(server.ListenAndServe())
}

func main() {
	runServer()
}
```

**client.go:**
```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"
)

type User struct {
	ID    int    `json:"id"`
	Name  string `json:"name"`
	Email string `json:"email"`
}

func createUser(client *http.Client, name, email string) (*User, error) {
	user := User{Name: name, Email: email}
	jsonData, _ := json.Marshal(user)
	
	resp, err := client.Post(
		"http://localhost:8080/users/create",
		"application/json",
		bytes.NewBuffer(jsonData),
	)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	
	if resp.StatusCode != http.StatusCreated {
		body, _ := io.ReadAll(resp.Body)
		return nil, fmt.Errorf("error: %s", string(body))
	}
	
	var result User
	json.NewDecoder(resp.Body).Decode(&result)
	return &result, nil
}

func getUser(client *http.Client, id int) (*User, error) {
	resp, err := client.Get(fmt.Sprintf("http://localhost:8080/users/get?id=%d", id))
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	
	if resp.StatusCode == http.StatusNotFound {
		return nil, fmt.Errorf("user not found")
	}
	
	if resp.StatusCode != http.StatusOK {
		body, _ := io.ReadAll(resp.Body)
		return nil, fmt.Errorf("error: %s", string(body))
	}
	
	var result User
	json.NewDecoder(resp.Body).Decode(&result)
	return &result, nil
}

func runClient() {
	time.Sleep(1 * time.Second) // Wait for server to start
	
	client := &http.Client{
		Timeout: 5 * time.Second,
	}
	
	fmt.Println("=== Creating User ===")
	user, err := createUser(client, "John Doe", "john@example.com")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Created: %+v\n\n", user)
	
	fmt.Println("=== Getting User ===")
	retrievedUser, err := getUser(client, user.ID)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Retrieved: %+v\n", retrievedUser)
}

func main() {
	runClient()
}
```

---

### 5.2 SUMMARY TABLE

## HTTP Status Codes Quick Reference

| Code | Name | When to Use |
|------|------|------------|
| **200** | OK | Request successful |
| **201** | Created | Resource created successfully |
| **204** | No Content | Success but no body |
| **301** | Moved Permanently | Permanent redirect |
| **302** | Found | Temporary redirect |
| **304** | Not Modified | Cached response valid |
| **400** | Bad Request | Malformed request |
| **401** | Unauthorized | Authentication required |
| **403** | Forbidden | Authenticated but no access |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Wrong HTTP method |
| **409** | Conflict | Resource conflict |
| **429** | Too Many Requests | Rate limited |
| **500** | Internal Server Error | Generic server error |
| **503** | Service Unavailable | Temporarily unavailable |

---

## KEY TAKEAWAYS

1. **Server**: Receives requests, processes, sends responses with status codes
2. **Client**: Sends requests, receives responses, checks status codes
3. **Always close response body** on client side with `defer resp.Body.Close()`
4. **Status codes first** on server - call `WriteHeader()` before writing body
5. **Reuse clients** - Create once, use multiple times for connection pooling
6. **Set timeouts** - Both server and client should have timeouts
7. **Error handling** - Check status codes and handle all error types
8. **Use JSON** - Marshal/Unmarshal for structured data communication
9. **Middleware** - Wrap handlers for logging, auth, validation
10. **Production-ready** - Always include timeouts, error handling, logging

This complete guide covers everything from HTTP fundamentals to production patterns!