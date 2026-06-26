# Fetch API - C.R.U.D Documentation

## Overview

**`fetch()` is an in-built JavaScript API for making HTTP requests** and responses. It returns `Promise`, making it ideal for asynchronous operations.
JS fetch API as a technology comes under the umbrella of AJAX.

`AJAX` is **not** a technology it's a set of web development techniques, a concept/philosophy for:

- Making background HTTP requests

- Update UI without refreshing

- Build responsive, app-like experiences

- Update parts of a page dynamically

- Create single-page applications (SPAs)

---

**Syntax:**
```javascript
fetch(input, [init])
```

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `input` | `string` or `Request` | URL endpoint or Request object |
| `init` (optional) | `object` | Configuration options for the request |

---

## The `init` Options Object

| Option | Type | Description |
|--------|------|-------------|
| `method` | `string` | **HTTP method** (GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS) |
| `headers` | `object` or `Headers` | Request headers |
| `body` | `Blob`, `FormData`, `URLSearchParams`, `string`, `ReadableStream` | Request body for POSTing (not allowed for GET/HEAD) |
| `mode` | `cors`, `no-cors`, `same-origin` | CORS mode |
| `credentials` | `omit`, `same-origin`, `include` | Include cookies? |
| `cache` | `default`, `no-store`, `reload`, `no-cache`, `force-cache`, `only-if-cached` | Cache behavior |
| `redirect` | `follow`, `error`, `manual` | Redirect handling |
| `referrer` | `string` or `about:client` | Referrer policy |
| `signal` | `AbortSignal` | Abort the request |
| `keepalive` | `boolean` | Keep request alive after page unload |

**Here we discuss only the CRUD methods, so we will only discuss the top three - `method` `headers` `body`**

---

## REST Methods :

**`fetch()` supports all REST (HTTP) methods** via the `method` property in `init`:

### GET - Retrieve data.

**The default mehtod for fetch**

```javascript
fetch('https://api.example.com/users')
  .then(response => response.json())
```


### POST - Create new resource

```
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'John', email: 'john@example.com' })
})
```

### PUT - Update an entire resource

```
fetch('https://api.example.com/users/123', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Johnathan', email: 'john@new.example.com' })
})
```

### PATCH - Partial updates

```
fetch('https://api.example.com/users/123', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Johnahtan' })
})
```

### DELETE - Remove resource

```
fetch('https://api.example.com/users/123', {
  method: 'DELETE'
})
```

### HEAD - Get headers only

Here the HEAD method allows you to ask for metadata and headers only, without the actual response that comes with it.

It's identical to GET but without the response body. You get:

- HTTP status code

- Response headers (Content-Type, Content-Length, Last-Modified, etc.)

- Metadata about the resource

- **NO response body/content**

```
fetch('https://api.example.com/users', {
  method: 'HEAD'
})

// Returns:
// - Status: 200 OK
// - Headers: Content-Type, Content-Length, etc.
// - Body: (empty) <-- NO data
```

### OPTIONS - List allowed methods

OPTIONS method allow you to ask server - What HTTP methods are allowed for this resource and the rules for communicating with the server.

```
fetch('https://api.example.com/users', {
  method: 'OPTIONS'
})

/ Response headers might include:
// Access-Control-Allow-Methods: GET, POST, PUT, DELETE
// Access-Control-Allow-Headers: Content-Type, Authorization
// Access-Control-Allow-Origin: *
// Access-Control-Max-Age: 86400
// Allow: GET, POST, HEAD
```

---

## Response Object Properties

| Property | Type | Description |
|----------|------|-------------|
| `ok` | `boolean` | `true` if status is 200-299 |
| `status` | `number` | HTTP status code 200, 304, 400 etc. |
| `statusText` | `string` | Status message 400 BAD_REQUEST, 404 NOT_FOUND ,200 OK etc, Gives human readable text|
| `headers` | `Headers` | Response headers |
| `url` | `string` | Final URL after redirects |
| `redirected` | `boolean` | Whether a redirect occurred |
| `type` | `string` | Response type (basic, cors, opaque) |
| `bodyUsed` | `boolean` | Whether body has been read |

---

## Response Methods (Body Parsing)

| Method | Returns | Description |
|--------|---------|-------------|
| `json()` | `Promise<object>` | Parse as JSON |
| `text()` | `Promise<string>` | Parse as text |
| `blob()` | `Promise<Blob>` | Parse as blob |
| `arrayBuffer()` | `Promise<ArrayBuffer>` | Parse as ArrayBuffer |
| `formData()` | `Promise<FormData>` | Parse as FormData |
| `clone()` | `Response` | Clone the response |

---

## Practical Examples

### 1. Basic GET with error handling
```javascript
async function fetchUsers() {
  try {
    const response = await fetch('https://api.example.com/users');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch failed:', error);
  }
}
```

### 2. POST with authentication
```javascript
async function createUser(userData) {
  const response = await fetch('https://api.example.com/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer your-token-here'
    },
    body: JSON.stringify(userData)
  });
  
  if (!response.ok) throw new Error('Failed to create user');
  return await response.json();
}
```

### 3. Upload file with FormData
```javascript
async function uploadFile(file) {
  const formData = new FormData();
  formData.append('file', file);
  
  const response = await fetch('https://api.example.com/upload', {
    method: 'POST',
    body: formData
    // Don't set Content-Type header - browser sets it with boundary
  });
  
  return await response.json();
}
```

Check my other article on updating file with FormData

---

## Browser Compatibility
- Chrome 42+
- Firefox 39+
- Safari 10.1+
- Edge 14+
- Node.js (requires `node-fetch` or `undici`)


## Key Takeaways

- **`fetch()` supports all REST methods** via the `method` property  
- Returns Promises - use `async/await` or `.then()`  
- Always check `response.ok` before parsing   
- `fetch` doesn't throw on HTTP errors (400-500) - handle manually  

