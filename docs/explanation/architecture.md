# Architecture

This document describes the system architecture and design principles of the Fastify web framework.

## System Context

Fastify operates within the Node.js ecosystem as a high-performance web framework that connects developers, applications, and external systems.

```mermaid
C4Context
    title System Context - Fastify Web Framework
    
    Person(developers, "Developers", "Build web applications and APIs")
    Person(users, "End Users", "Access applications via HTTP")
    
    System(fastify, "Fastify Framework", "High-performance Node.js web framework with plugin architecture")
    
    System_Ext(nodejs, "Node.js Runtime", "JavaScript runtime environment")
    System_Ext(databases, "Databases", "PostgreSQL, MongoDB, Redis, MySQL")
    System_Ext(apis, "External APIs", "Microservices, third-party APIs")
    System_Ext(monitoring, "Monitoring Tools", "APM, logging, metrics collection")
    System_Ext(registry, "NPM Registry", "Package distribution and dependencies")
    
    Rel(developers, fastify, "Build applications with")
    Rel(users, fastify, "Send HTTP requests to")
    Rel(fastify, nodejs, "Runs on")
    Rel(fastify, databases, "Connects to via plugins")
    Rel(fastify, apis, "Integrates with via HTTP")
    Rel(fastify, monitoring, "Reports metrics and logs to")
    Rel(fastify, registry, "Installs plugins from")
```

## Container Architecture

Fastify's internal architecture consists of several key containers that work together to provide high-performance request processing.

```mermaid
C4Container
    title Container Diagram - Fastify Framework Internal Architecture
    
    Container(server, "HTTP Server", "Node.js HTTP/HTTPS/HTTP2", "Handles incoming connections and protocol termination")
    Container(router, "Router Engine", "find-my-way", "High-performance route matching and parameter extraction")
    Container(validator, "Schema Validator", "@fastify/ajv-compiler", "JSON schema compilation and validation")
    Container(serializer, "JSON Serializer", "fast-json-stringify", "High-speed response serialization")
    Container(hooks, "Lifecycle Hooks", "JavaScript", "Request/response lifecycle management")
    Container(plugins, "Plugin System", "avvio", "Plugin loading, dependency injection, and encapsulation")
    Container(logger, "Logger", "Pino", "Structured JSON logging with performance optimization")
    Container(context, "Request Context", "JavaScript", "Request-scoped state and dependency injection")
    
    Rel(server, router, "Routes requests to")
    Rel(router, hooks, "Triggers lifecycle events in")
    Rel(hooks, validator, "Validates input using")
    Rel(hooks, context, "Manages state in")
    Rel(context, plugins, "Accesses services from")
    Rel(hooks, serializer, "Serializes responses with")
    Rel(hooks, logger, "Logs events to")
```

## Core Component Interactions

This sequence diagram shows how the main components interact during a typical HTTP request lifecycle.

```mermaid
sequenceDiagram
    participant Client
    participant Server as HTTP Server
    participant Router
    participant Hooks as Lifecycle Hooks
    participant Validator as Schema Validator
    participant Handler as Route Handler
    participant Serializer as JSON Serializer
    participant Logger
    
    Client->>Server: HTTP Request
    Server->>Router: Parse and route request
    Router->>Hooks: Trigger onRequest hooks
    Hooks->>Validator: Validate request schema
    
    alt Validation fails
        Validator->>Client: 400 Bad Request
    else Validation succeeds
        Validator->>Hooks: Trigger preHandler hooks
        Hooks->>Handler: Execute route handler
        Handler->>Hooks: Return response data
        Hooks->>Serializer: Serialize response
        Serializer->>Hooks: Trigger onSend hooks
        Hooks->>Server: Send HTTP response
        Server->>Client: HTTP Response
        Server->>Logger: Log request completion
    end
```

## Technology Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Runtime** | Node.js 18+ | JavaScript execution environment |
| **HTTP Server** | Node.js http/https/http2 | Protocol handling and connection management |
| **Router** | find-my-way | High-performance URL routing and parameter extraction |
| **Schema Validation** | @fastify/ajv-compiler | JSON Schema compilation and request validation |
| **JSON Serialization** | fast-json-stringify | High-speed response serialization |
| **Plugin System** | avvio | Dependency injection and plugin lifecycle management |
| **Logging** | Pino | Structured JSON logging with minimal overhead |
| **Testing** | light-my-request | HTTP injection for unit testing |

## Key Design Patterns

### Plugin Architecture
- **Encapsulation**: Each plugin operates in its own context with isolated scope
- **Dependency Injection**: Plugins can declare and inject dependencies
- **Lifecycle Management**: Automatic plugin loading and initialization ordering

### Hook System
- **Observer Pattern**: Lifecycle events allow non-invasive request processing
- **Chain of Responsibility**: Hooks execute in defined order with ability to halt processing
- **Decorator Pattern**: Request and reply objects can be enhanced by plugins

### Schema-Driven Development
- **Contract-First**: Define JSON schemas for validation and serialization
- **Performance**: Pre-compiled validators and serializers for speed
- **Type Safety**: Schema definitions provide runtime type checking

## Key Design Decisions

### Performance-First Architecture
**Decision**: Prioritize speed and low overhead in every architectural choice.

**Rationale**: Web framework performance directly impacts application scalability and user experience. Fastify is designed for high-throughput scenarios.

**Implementation**: 
- Pre-compiled route matching with radix trees
- Schema pre-compilation for validation and serialization
- Minimal object creation in hot paths
- Optimized logging with async writing

### Plugin-Based Extensibility
**Decision**: Use a formal plugin system instead of middleware-only approach.

**Rationale**: Provides better encapsulation, dependency management, and testing compared to traditional middleware chains.

**Implementation**:
- Avvio-based plugin system with dependency resolution
- Context isolation between plugins
- Declarative plugin dependencies and loading order

### Schema-Centric Validation
**Decision**: Make JSON Schema validation a first-class feature.

**Rationale**: Type safety and performance benefits from pre-compiled validation, plus automatic documentation generation.

**Implementation**:
- AJV-based schema compilation
- Schema-driven serializer generation
- Integration with TypeScript definitions

### HTTP Protocol Agnostic
**Decision**: Support multiple HTTP protocol versions transparently.

**Rationale**: Future-proof the framework as HTTP/2 and HTTP/3 adoption increases.

**Implementation**:
- Abstract server creation supporting HTTP/1.1, HTTP/2, and HTTPS
- Protocol-agnostic request/response interfaces
- Automatic protocol detection and optimization

## Performance Characteristics

| Metric | Typical Value | Notes |
|--------|---------------|-------|
| **Requests/sec** | 100,000+ | On modern hardware with simple routes |
| **Latency (p99)** | < 1ms | For in-memory operations |
| **Memory Usage** | ~50MB base | Framework overhead, excluding application code |
| **Plugin Load Time** | < 100ms | For typical plugin with dependencies |
| **Schema Compilation** | < 10ms | Per schema during application startup |

## Scalability Considerations

### Horizontal Scaling
- **Stateless Design**: No server-side session state in core framework
- **Cluster Mode**: Supports Node.js cluster module for multi-process deployment
- **Load Balancer Ready**: Designed for deployment behind reverse proxies

### Vertical Scaling
- **Non-blocking I/O**: Leverages Node.js event loop for high concurrency
- **Memory Efficiency**: Minimal object allocation in request processing
- **CPU Optimization**: Pre-compiled components reduce runtime computation