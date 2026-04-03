# Configuration Options Reference

Complete reference of all Fastify configuration options and their usage.

## Core Server Options

| Option | Type | Default | Description | Environment Variable |
|--------|------|---------|-------------|---------------------|
| `logger` | `boolean \| object` | `false` | Enable Pino logger or pass logger configuration | `LOG_LEVEL` |
| `disableRequestLogging` | `boolean` | `false` | Disable automatic request logging | - |
| `serverFactory` | `function` | HTTP server | Custom server factory function | - |
| `caseSensitive` | `boolean` | `true` | Enable case-sensitive routing | - |
| `ignoreTrailingSlash` | `boolean` | `false` | Ignore trailing slashes in routes | - |
| `ignoreDuplicateSlashes` | `boolean` | `false` | Ignore duplicate slashes in routes | - |
| `maxParamLength` | `number` | `100` | Maximum length of route parameters | - |
| `bodyLimit` | `number` | `1048576` | Maximum request body size in bytes (1MB) | `BODY_LIMIT` |
| `keepAliveTimeout` | `number` | `72000` | Keep-alive timeout in milliseconds | - |
| `connectionTimeout` | `number` | `0` | Connection timeout in milliseconds (0 = no limit) | - |
| `pluginTimeout` | `number` | `60000` | Plugin registration timeout in milliseconds | - |
| `requestIdHeader` | `string` | `"request-id"` | Header name for request ID | - |
| `requestIdLogLabel` | `string` | `"reqId"` | Log property name for request ID | - |
| `http2` | `boolean \| object` | `false` | Enable HTTP/2 support | - |
| `https` | `object` | `null` | HTTPS options (key, cert, etc.) | - |
| `trustProxy` | `boolean \| string \| number \| function` | `false` | Trust proxy headers | `TRUST_PROXY` |

## Logger Configuration

### Basic Logger Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `logger` | `boolean` | `false` | Enable with default settings |
| `logger.level` | `string` | `"info"` | Log level (trace, debug, info, warn, error, fatal) |
| `logger.stream` | `stream` | `process.stdout` | Output stream |
| `logger.redact` | `array \| object` | `[]` | Fields to redact from logs |
| `logger.serializers` | `object` | `{}` | Custom serializers for objects |

### Development Logger

```javascript
{
  logger: {
    transport: {
      target: 'pino-pretty',
      options: {
        colorize: true,
        translateTime: 'HH:MM:ss Z',
        ignore: 'pid,hostname'
      }
    }
  }
}
```

### Production Logger

```javascript
{
  logger: {
    level: 'warn',
    redact: ['req.headers.authorization', 'req.body.password'],
    serializers: {
      req: require('pino-std-serializers').req,
      res: require('pino-std-serializers').res
    }
  }
}
```

## Request Handling Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `querystringParser` | `function` | `qs.parse` | Custom querystring parser |
| `exposeHeadRoutes` | `boolean` | `true` | Automatically generate HEAD routes for GET routes |
| `constraints` | `object` | `{}` | Route constraints configuration |
| `return503OnClosing` | `boolean` | `true` | Return 503 when server is closing |
| `ajv` | `object` | `{}` | AJV schema validation options |
| `serializerOpts` | `object` | `{}` | Serializer options |
| `http2SessionTimeout` | `number` | `72000` | HTTP/2 session timeout |

### Custom Request ID Generator

```javascript
{
  genReqId: (req) => {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`
  }
}
```

### Custom Querystring Parser

```javascript
{
  querystringParser: (str) => {
    // Custom parsing logic
    const params = new URLSearchParams(str)
    const result = {}
    for (const [key, value] of params) {
      result[key] = value
    }
    return result
  }
}
```

## Schema Validation Options

### AJV Configuration

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `ajv.removeAdditional` | `boolean \| string` | `false` | Remove additional properties |
| `ajv.useDefaults` | `boolean \| string` | `false` | Use default values from schema |
| `ajv.coerceTypes` | `boolean \| string` | `false` | Coerce types to match schema |
| `ajv.allErrors` | `boolean` | `false` | Collect all errors instead of stopping at first |
| `ajv.nullable` | `boolean` | `false` | Support nullable keyword |

```javascript
{
  ajv: {
    removeAdditional: true,
    useDefaults: true,
    coerceTypes: true,
    allErrors: true,
    formats: {
      'custom-format': /^[A-Z]{3}$/
    }
  }
}
```

### Schema Controller Options

```javascript
{
  schemaController: {
    compilersFactory: {
      buildValidator: function (externalSchemas, options) {
        // Custom validator compiler
      },
      buildSerializer: function (externalSchemas, options) {
        // Custom serializer compiler  
      }
    }
  }
}
```

## HTTP/2 Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `http2` | `boolean` | `false` | Enable HTTP/2 |
| `http2.allowHTTP1` | `boolean` | `true` | Allow HTTP/1.1 on HTTP/2 socket |
| `http2SessionTimeout` | `number` | `72000` | Session timeout in milliseconds |

```javascript
{
  http2: {
    allowHTTP1: true,
    settings: {
      enablePush: false,
      initialWindowSize: 2147483647
    }
  }
}
```

## HTTPS Options

```javascript
{
  https: {
    key: fs.readFileSync('path/to/private-key.pem'),
    cert: fs.readFileSync('path/to/certificate.pem'),
    // Optional: Certificate Authority
    ca: fs.readFileSync('path/to/ca-certificate.pem'),
    // Request client certificate
    requestCert: true,
    // Reject unauthorized certificates  
    rejectUnauthorized: true
  }
}
```

## Trust Proxy Configuration

| Value | Type | Description |
|-------|------|-------------|
| `true` | `boolean` | Trust all proxies |
| `false` | `boolean` | Trust no proxies |
| `"loopback"` | `string` | Trust loopback addresses |
| `"linklocal"` | `string` | Trust link-local addresses |
| `"uniquelocal"` | `string` | Trust unique local addresses |
| `1` | `number` | Trust first proxy |
| `["192.168.1.1", "10.0.0.1"]` | `array` | Trust specific IPs |
| `function` | `function` | Custom trust function |

```javascript
// Custom trust function
{
  trustProxy: (address, hop) => {
    return address === '127.0.0.1' || address === '192.168.1.1'
  }
}
```

## Performance Tuning Options

### High Concurrency

```javascript
{
  logger: false, // Disable logging for performance
  disableRequestLogging: true,
  keepAliveTimeout: 30000,
  connectionTimeout: 5000,
  bodyLimit: 524288, // 512KB
  maxParamLength: 50,
  pluginTimeout: 5000
}
```

### Memory Optimization

```javascript
{
  bodyLimit: 1048576, // 1MB
  maxParamLength: 100,
  caseSensitive: true,
  ignoreTrailingSlash: false,
  ignoreDuplicateSlashes: false
}
```

## Environment-Specific Configurations

### Development Environment

```javascript
const isDev = process.env.NODE_ENV === 'development'

{
  logger: isDev ? {
    transport: {
      target: 'pino-pretty',
      options: { colorize: true }
    }
  } : { level: 'warn' },
  disableRequestLogging: !isDev,
  ignoreTrailingSlash: isDev,
  caseSensitive: !isDev
}
```

### Testing Environment

```javascript
const isTest = process.env.NODE_ENV === 'test'

{
  logger: !isTest, // Disable logging in tests
  pluginTimeout: isTest ? 0 : 60000, // No timeout in tests
  keepAliveTimeout: isTest ? 1000 : 72000
}
```

### Production Environment

```javascript
{
  logger: {
    level: process.env.LOG_LEVEL || 'warn',
    redact: ['req.headers.authorization', 'res.headers["set-cookie"]']
  },
  trustProxy: true,
  keepAliveTimeout: 65000, // Slightly less than load balancer timeout
  connectionTimeout: 10000,
  bodyLimit: parseInt(process.env.MAX_BODY_SIZE) || 1048576,
  return503OnClosing: true
}
```

## Content Type Parser Options

```javascript
{
  addContentTypeParser: [
    {
      type: 'application/xml',
      parseAs: 'string',
      handler: (req, body, done) => {
        // Custom XML parsing
        done(null, parseXML(body))
      }
    }
  ]
}
```

## Custom Server Factory

```javascript
{
  serverFactory: (handler, opts) => {
    const server = require('http').createServer((req, res) => {
      // Custom server logic
      handler(req, res)
    })
    
    // Custom server configuration
    server.keepAliveTimeout = 61000
    server.headersTimeout = 65000
    
    return server
  }
}
```

## Validation and Error Handling

### Schema Error Options

```javascript
{
  schemaErrorFormatter: (errors, dataVar) => {
    return new Error(`Schema validation failed: ${errors.map(e => e.message).join(', ')}`)
  },
  
  // Custom error handler
  errorHandler: function (error, request, reply) {
    if (error.validation) {
      reply.status(400).send({
        error: 'Validation Error',
        message: error.message,
        details: error.validation
      })
    } else {
      reply.status(500).send({ error: 'Internal Server Error' })
    }
  }
}
```

## Complete Example Configuration

```javascript
const fastify = require('fastify')({
  // Core options
  logger: {
    level: process.env.LOG_LEVEL || 'info',
    transport: process.env.NODE_ENV === 'development' ? {
      target: 'pino-pretty',
      options: { colorize: true }
    } : undefined
  },
  
  // Request handling
  bodyLimit: parseInt(process.env.BODY_LIMIT) || 1048576,
  caseSensitive: process.env.NODE_ENV === 'production',
  ignoreTrailingSlash: process.env.NODE_ENV !== 'production',
  maxParamLength: 100,
  
  // Connection settings
  keepAliveTimeout: 65000,
  connectionTimeout: 10000,
  trustProxy: process.env.TRUST_PROXY === 'true',
  
  // Schema validation
  ajv: {
    removeAdditional: true,
    useDefaults: true,
    coerceTypes: false,
    allErrors: false
  },
  
  // Custom request ID
  genReqId: (req) => `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
  requestIdHeader: 'x-request-id',
  
  // Error handling
  return503OnClosing: true
})
```

---

*For practical configuration examples, see the [Configuration Guide](../guides/configuration.md).*