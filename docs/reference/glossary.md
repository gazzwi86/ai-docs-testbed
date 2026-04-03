# Glossary

Domain-specific terms and concepts used in the Fastify ecosystem.

## Core Concepts

| Term | Definition |
|------|------------|
| **AJV** | A JSON Schema validator library used by Fastify for request/response validation |
| **Avvio** | Plugin management system that handles the lifecycle and dependencies of Fastify plugins |
| **Content Type Parser** | Middleware that parses incoming request bodies based on Content-Type header |
| **Context** | Encapsulated scope within Fastify that isolates plugins and their modifications |
| **Encapsulation** | Fastify's system of isolating plugins so they cannot interfere with parent or sibling contexts |
| **Fast JSON Stringify** | High-performance JSON serialization library used for response formatting |
| **Fastify Instance** | The main server object created by calling `fastify()` with configuration options |
| **Find My Way** | High-performance HTTP router used internally by Fastify for route matching |
| **Hook** | Lifecycle method that executes at specific points during request/response processing |
| **Light My Request** | HTTP injection library used for testing Fastify applications without starting a server |
| **Pino** | High-performance JSON logger used by Fastify for structured logging |
| **Plugin** | Reusable module that extends Fastify functionality through the plugin system |
| **Prehandler** | Hook that runs after validation but before the route handler |
| **Reply** | Fastify's response object with methods for sending responses and setting headers |
| **Request** | Fastify's enhanced request object containing parsed headers, body, params, and query |
| **Route** | HTTP endpoint definition with method, path, handler, and optional schema/hooks |
| **Schema** | JSON Schema definition used for validating and serializing request/response data |
| **Serializer** | Function that converts JavaScript objects to JSON strings for HTTP responses |

## HTTP and Networking

| Term | Definition |
|------|------------|
| **Connection Timeout** | Maximum time to wait for a client connection to be established |
| **Head Route** | HTTP HEAD method route automatically generated for GET routes |
| **HTTP/2** | Next-generation HTTP protocol with multiplexing and server push capabilities |
| **Keep-Alive Timeout** | Duration to keep idle connections open for reuse |
| **Request ID** | Unique identifier assigned to each HTTP request for tracing and logging |
| **Trust Proxy** | Configuration to trust proxy headers like X-Forwarded-For for client IP detection |

## Performance and Optimization

| Term | Definition |
|------|------------|
| **Body Limit** | Maximum allowed size for HTTP request bodies to prevent memory exhaustion |
| **Case Sensitive Routing** | Whether routes distinguish between uppercase and lowercase characters |
| **Ignore Trailing Slash** | Setting to treat `/users` and `/users/` as the same route |
| **Max Param Length** | Maximum allowed length for URL parameters to prevent DoS attacks |
| **Plugin Timeout** | Maximum time allowed for plugin registration before timing out |

## Validation and Schemas

| Term | Definition |
|------|------------|
| **Coerce Types** | Automatic type conversion (e.g., string "123" to number 123) during validation |
| **JSON Schema** | Specification for describing and validating JSON data structure |
| **Remove Additional** | Option to strip properties not defined in the schema |
| **Schema Compiler** | Function that converts JSON schemas into validation/serialization functions |
| **Use Defaults** | Option to apply default values from schema when properties are missing |

## Plugin System

| Term | Definition |
|------|------------|
| **Decorated Properties** | Custom properties added to Fastify instance, request, or reply objects |
| **Encapsulation Context** | Isolated scope where plugins operate without affecting parent contexts |
| **Plugin Dependency** | Mechanism to ensure plugins load in correct order based on dependencies |
| **Plugin Factory** | Function that creates and configures a plugin instance |
| **Plugin Metadata** | Information about plugin name, version, and dependencies |
| **Scoped Plugin** | Plugin that only affects the context where it's registered and child contexts |

## Logging and Debugging

| Term | Definition |
|------|------------|
| **Log Level** | Severity level for log messages (trace, debug, info, warn, error, fatal) |
| **Log Redaction** | Process of hiding sensitive information like passwords from log output |
| **Log Serializer** | Function that formats objects for structured JSON logging |
| **Request Logging** | Automatic logging of incoming HTTP requests and their details |
| **Structured Logging** | JSON-formatted logs with consistent fields for machine parsing |

## Development and Testing

| Term | Definition |
|------|------------|
| **Hot Reload** | Development feature to restart server automatically when code changes |
| **HTTP Injection** | Testing technique to simulate HTTP requests without starting a real server |
| **Mock Server** | Test server that simulates real server responses for development |
| **Pretty Printing** | Human-readable log formatting for development environments |
| **Test Isolation** | Ensuring tests don't interfere with each other by using separate server instances |

## Security

| Term | Definition |
|------|------------|
| **CORS** | Cross-Origin Resource Sharing - mechanism for allowing cross-domain requests |
| **CSP** | Content Security Policy - HTTP header to prevent XSS attacks |
| **CSRF** | Cross-Site Request Forgery - attack where unauthorized commands are executed |
| **Helmet** | Security middleware that sets various HTTP headers for protection |
| **Rate Limiting** | Technique to limit number of requests from a client within a time window |
| **Request Validation** | Process of checking incoming data against predefined schemas |
| **Secure Headers** | HTTP headers that improve application security (HSTS, X-Frame-Options, etc.) |

## Deployment and Operations

| Term | Definition |
|------|------------|
| **Blue-Green Deployment** | Strategy using two identical environments for zero-downtime updates |
| **Graceful Shutdown** | Process of closing server cleanly by finishing current requests before exit |
| **Health Check** | Endpoint used by load balancers to verify server availability |
| **Load Balancer** | System that distributes incoming requests across multiple server instances |
| **Process Manager** | Tool like PM2 that manages Node.js application lifecycle |
| **Zero-Downtime Deployment** | Deployment strategy that updates application without service interruption |

## Architecture Patterns

| Term | Definition |
|------|------------|
| **API Gateway** | Central entry point that routes requests to appropriate microservices |
| **Circuit Breaker** | Pattern to prevent cascade failures by stopping requests to failing services |
| **Middleware** | Software layer that intercepts and processes requests/responses |
| **Microservice** | Small, independently deployable service focused on a single business capability |
| **Monolithic Architecture** | Application design where all components are interconnected in a single unit |
| **RESTful API** | Web service following REST principles for resource-based HTTP interactions |
| **Service Discovery** | Method for services to find and communicate with each other |

## Acronyms and Abbreviations

| Acronym | Full Form | Context |
|---------|-----------|---------|
| **API** | Application Programming Interface | Web service endpoints |
| **CORS** | Cross-Origin Resource Sharing | Web security |
| **CPU** | Central Processing Unit | Performance monitoring |
| **CSP** | Content Security Policy | Web security |
| **CSRF** | Cross-Site Request Forgery | Web security |
| **DNS** | Domain Name System | Networking |
| **DoS** | Denial of Service | Security attacks |
| **HTTP** | HyperText Transfer Protocol | Web communication |
| **HTTPS** | HTTP Secure | Secure web communication |
| **I/O** | Input/Output | System operations |
| **IP** | Internet Protocol | Networking |
| **JSON** | JavaScript Object Notation | Data format |
| **JWT** | JSON Web Token | Authentication |
| **MIME** | Multipurpose Internet Mail Extensions | Content types |
| **REST** | Representational State Transfer | API architecture |
| **TLS** | Transport Layer Security | Encryption |
| **URL** | Uniform Resource Locator | Web addresses |
| **UUID** | Universally Unique Identifier | Unique identifiers |
| **XSS** | Cross-Site Scripting | Security vulnerability |

## Common Fastify Patterns

| Pattern | Definition |
|---------|------------|
| **Async Route Handler** | Route handler function marked with `async` keyword for asynchronous operations |
| **Context Decoration** | Adding custom properties to Fastify instance that are available in child contexts |
| **Error First Callback** | Node.js convention where callback's first parameter is an error object |
| **Factory Function** | Function that returns a configured plugin for reuse across projects |
| **Fluent Interface** | API design allowing method chaining like `fastify.get().post()` |
| **Hook Pipeline** | Sequence of lifecycle hooks that process requests/responses |
| **Plugin Composition** | Combining multiple smaller plugins into a larger functional unit |
| **Route Prefixing** | Adding common path prefix to all routes within a plugin context |
| **Schema-First Development** | Approach where JSON schemas are defined before implementing handlers |

---

*For implementation details of these concepts, see the [Getting Started Guide](../getting-started/quickstart.md) and [Configuration Options](configuration-options.md).*