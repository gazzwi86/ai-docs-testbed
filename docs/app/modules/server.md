# Server Module

HTTP server creation and lifecycle management for multiple protocol support.

## Module Structure

```mermaid
flowchart TD
    createServer[createServer] --> getServerInstance[getServerInstance]
    createServer --> listen[listen function]
    
    getServerInstance --> HTTP[HTTP Server]
    getServerInstance --> HTTPS[HTTPS Server]
    getServerInstance --> HTTP2[HTTP2 Server]
    
    listen --> multipleBindings[multipleBindings]
    listen --> onListenHookRunner[onListenHookRunner]
    
    multipleBindings --> IPv4[IPv4 Binding]
    multipleBindings --> IPv6[IPv6 Binding]
    
    subgraph "Protocol Support"
        HTTP
        HTTPS
        HTTP2
    end
```

## Public API

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `createServer` | `options, httpHandler` | `Object` | Creates server instance with listen method |
| `listen` | `listenOptions, callback` | `Promise` | Starts server listening on specified address |
| `close` | `callback` | `Promise` | Gracefully shuts down server and connections |
| `forceCloseConnections` | None | `void` | Forcibly closes all active connections |

## Key Types

| Type | Properties | Description |
|------|------------|-------------|
| `ListenOptions` | `port, host, path, backlog, exclusive` | Server listening configuration |
| `ServerOptions` | `http2, https, serverFactory` | Server creation options |
| `AddressInfo` | `address, port, family` | Server address information |

## Usage Example

```javascript
const { createServer } = require('./lib/server')

// Create HTTP server
const server = createServer(options, httpHandler)

// Listen on port 3000
const listen = server.listen.bind(fastifyInstance)
await listen({ port: 3000, host: '0.0.0.0' })

// Close server gracefully
await server.close()
```

## Dependencies

| Dependency | Type | Purpose |
|------------|------|---------|
| `node:http` | Built-in | HTTP/1.1 server creation |
| `node:https` | Built-in | HTTPS server creation |
| `node:http2` | Built-in | HTTP/2 server creation |
| `node:dns` | Built-in | DNS resolution for dual-stack |
| `find-my-way` | External | High-performance router |

## Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `serverFactory` | `function` | `undefined` | Custom server factory function |
| `http2` | `boolean \| object` | `false` | Enable HTTP/2 with optional settings |
| `https` | `object` | `undefined` | HTTPS options (key, cert, etc.) |
| `trustProxy` | `boolean \| string \| number` | `false` | Proxy trust configuration |
| `maxRequestsPerSocket` | `number` | `0` | Max requests per keep-alive socket |

## Protocol Detection Flow

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Protocol as Protocol Handler
    
    Client->>Server: Incoming Connection
    Server->>Protocol: Detect Protocol
    
    alt HTTP/2
        Protocol->>Server: HTTP/2 Handler
        Server->>Client: HTTP/2 Response
    else HTTPS
        Protocol->>Server: HTTPS Handler
        Server->>Client: HTTPS Response
    else HTTP/1.1
        Protocol->>Server: HTTP Handler
        Server->>Client: HTTP Response
    end
```