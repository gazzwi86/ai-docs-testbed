# Application Overview

Fastify is a high-performance Node.js web framework designed for speed and low overhead. It features a plugin-based architecture, built-in JSON schema validation, and comprehensive lifecycle hooks.

## Module Dependency Diagram

```mermaid
flowchart TD
    Entry[fastify.js] --> Server[Server Module]
    Entry --> Request[Request Module]
    Entry --> Reply[Reply Module]
    Entry --> Context[Context Module]
    Entry --> Route[Route Module]
    Entry --> Hooks[Hooks Module]
    Entry --> SchemaController[Schema Controller]
    Entry --> ContentTypeParser[Content Type Parser]
    Entry --> Validation[Validation Module]
    Entry --> ErrorHandler[Error Handler]
    
    Server --> Hooks
    Route --> Context
    Route --> Hooks
    Route --> Validation
    Route --> SchemaController
    Reply --> Hooks
    Reply --> SchemaController
    Request --> Context
    ContentTypeParser --> Validation
    Validation --> SchemaController
    
    %% External dependencies
    Server --> FindMyWay[find-my-way]
    SchemaController --> AJV[@fastify/ajv-compiler]
    SchemaController --> FastJsonStringify[fast-json-stringify]
    Hooks --> Avvio[avvio]
```

## Module Overview Table

| Module | Purpose | Key Files | External Dependencies |
|--------|---------|-----------|----------------------|
| **Server** | HTTP server creation and lifecycle management | `lib/server.js` | `node:http`, `node:https`, `node:http2` |
| **Request** | Request parsing and property management | `lib/request.js` | `@fastify/proxy-addr` |
| **Reply** | Response handling and serialization | `lib/reply.js` | `fast-json-stringify` |
| **Route** | URL routing and route registration | `lib/route.js` | `find-my-way` |
| **Hooks** | Lifecycle hooks and event management | `lib/hooks.js` | `avvio` |
| **Schema Controller** | JSON schema compilation and management | `lib/schema-controller.js` | `@fastify/ajv-compiler`, `@fastify/fast-json-stringify-compiler` |
| **Content Type Parser** | Request body parsing by content type | `lib/content-type-parser.js` | `secure-json-parse`, `toad-cache` |
| **Validation** | Request/response validation engine | `lib/validation.js` | `@fastify/ajv-compiler` |
| **Context** | Request-scoped dependency injection | `lib/context.js` | None |
| **Error Handler** | Error processing and response formatting | `lib/error-handler.js` | None |

## Entry Points

### Primary Entry Point
- **File**: `fastify.js`
- **Export**: Factory function that creates Fastify instances
- **Usage**: `const fastify = require('fastify')(options)`

### Application Startup Flow
1. **Instance Creation**: `fastify.js` creates new Fastify instance with provided options
2. **Plugin Loading**: Avvio-based plugin system loads and initializes plugins
3. **Server Creation**: Server module creates appropriate HTTP/HTTPS/HTTP2 server
4. **Route Registration**: Routes are registered with find-my-way router
5. **Schema Compilation**: JSON schemas are pre-compiled for validation and serialization
6. **Server Listen**: Application listens on specified port/host
7. **Request Processing**: Incoming requests flow through hooks → validation → handler → serialization

## Key Configuration

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `logger` | `boolean \| object` | `false` | Enable Pino logger |
| `bodyLimit` | `number` | `1048576` | Maximum request body size |
| `trustProxy` | `boolean \| string \| number` | `false` | Trust proxy headers |
| `pluginTimeout` | `number` | `60000` | Plugin loading timeout |
| `caseSensitive` | `boolean` | `true` | Case sensitive routing |
| `ignoreTrailingSlash` | `boolean` | `false` | Ignore trailing slashes |

## Performance Characteristics

- **Throughput**: 100,000+ requests/sec on modern hardware
- **Memory Overhead**: ~50MB base framework overhead
- **Request Latency**: <1ms p99 for in-memory operations
- **Schema Compilation**: <10ms per schema during startup
- **Plugin Load Time**: <100ms for typical plugin with dependencies