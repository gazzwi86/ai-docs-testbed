# Validation Module

JSON schema validation engine providing high-performance request and response validation with compilation caching.

## Module Structure

```mermaid
flowchart TD
    Validation[Validation Module] --> Compilation[Schema Compilation]
    Validation --> Execution[Validation Execution]
    Validation --> Caching[Validation Caching]
    
    Compilation --> SerializationCompiler[Serialization Compiler]
    Compilation --> ValidationCompiler[Validation Compiler]
    Compilation --> SchemaResolver[Schema Resolver]
    
    SerializationCompiler --> FastJsonStringify[fast-json-stringify]
    ValidationCompiler --> AJVCompiler[AJV Compiler]
    
    Execution --> RequestValidation[Request Validation]
    Execution --> ResponseValidation[Response Validation]
    Execution --> ErrorHandling[Validation Errors]
    
    RequestValidation --> BodyValidation[Body Validation]
    RequestValidation --> ParamsValidation[Params Validation]
    RequestValidation --> QueryValidation[Query Validation]
    RequestValidation --> HeadersValidation[Headers Validation]
    
    ResponseValidation --> StatusCodeValidation[Status Code Validation]
    ResponseValidation --> ResponseSchemaValidation[Response Schema Validation]
    
    Caching --> CompilerCache[Compiler Cache]
    Caching --> SchemaCache[Schema Cache]
    
    subgraph "Schema Symbols"
        BodySchema[kSchemaBody]
        ParamsSchema[kSchemaParams]
        QuerySchema[kSchemaQuerystring]
        HeadersSchema[kSchemaHeaders]
        ResponseSchema[kSchemaResponse]
    end
```

## Public API

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `compileSchemasForSerialization` | `context, compile` | `void` | Compiles response serialization schemas |
| `compileSchemasForValidation` | `context, compile` | `void` | Compiles request validation schemas |
| `validateParam` | `params, schema` | `boolean` | Validates route parameters |
| `validateBody` | `body, schema` | `boolean` | Validates request body |
| `validateQuery` | `query, schema` | `boolean` | Validates query parameters |
| `validateHeaders` | `headers, schema` | `boolean` | Validates request headers |

## Key Types

| Type | Properties | Description |
|------|------------|-------------|
| `ValidationContext` | `schema, config, server` | Route validation context |
| `CompiledValidator` | `validate` → `boolean` | Compiled validation function |
| `CompiledSerializer` | `serialize` → `string` | Compiled serialization function |
| `ValidationSchema` | `type, properties, required` | JSON Schema definition |

## Usage Example

```javascript
// Route schema definition
const userSchema = {
  body: {
    type: 'object',
    properties: {
      name: { type: 'string', minLength: 1 },
      email: { type: 'string', format: 'email' },
      age: { type: 'number', minimum: 0 }
    },
    required: ['name', 'email']
  },
  params: {
    type: 'object',
    properties: {
      id: { type: 'string', pattern: '^[0-9]+$' }
    },
    required: ['id']
  },
  querystring: {
    type: 'object',
    properties: {
      format: { type: 'string', enum: ['json', 'xml'] },
      limit: { type: 'number', minimum: 1, maximum: 100 }
    }
  },
  headers: {
    type: 'object',
    properties: {
      'authorization': { type: 'string' }
    },
    required: ['authorization']
  },
  response: {
    200: {
      type: 'object',
      properties: {
        id: { type: 'number' },
        name: { type: 'string' },
        email: { type: 'string' }
      }
    },
    400: {
      type: 'object',
      properties: {
        error: { type: 'string' }
      }
    }
  }
}

// Schema compilation happens automatically during route registration
fastify.post('/users/:id', { schema: userSchema }, async (request, reply) => {
  // All inputs are pre-validated
  const { id } = request.params
  const { name, email, age } = request.body
  const { format, limit } = request.query
  
  // Process validated data
  const user = await updateUser(id, { name, email, age })
  return user // Response is automatically serialized
})
```

## Dependencies

| Dependency | Type | Purpose |
|------------|------|---------|
| `@fastify/ajv-compiler` | External | JSON Schema validation compilation |
| `fast-json-stringify` | External | High-performance JSON serialization |
| `./symbols` | Internal | Schema storage symbols |
| `./errors` | Internal | Validation error types |

## Validation Flow

```mermaid
sequenceDiagram
    participant Request
    participant Router
    participant Validator as Validation Module
    participant Schema as Schema Compiler
    participant Handler
    
    Request->>Router: Incoming Request
    Router->>Validator: Validate Request
    
    Validator->>Schema: Get Compiled Validators
    Schema->>Validator: Return Validators
    
    Validator->>Validator: Validate Headers
    Validator->>Validator: Validate Params
    Validator->>Validator: Validate Query
    Validator->>Validator: Validate Body
    
    alt Validation Success
        Validator->>Handler: Execute Handler
        Handler->>Validator: Return Response
        Validator->>Schema: Get Response Serializer
        Schema->>Validator: Return Serializer
        Validator->>Request: Serialize & Send Response
    else Validation Error
        Validator->>Request: Return 400 Validation Error
    end
```

## Schema Compilation

| HTTP Part | Symbol | Validation Target | Error Code |
|-----------|--------|------------------|------------|
| **Body** | `kSchemaBody` | `request.body` | 400 |
| **Parameters** | `kSchemaParams` | `request.params` | 400 |
| **Query** | `kSchemaQuerystring` | `request.query` | 400 |
| **Headers** | `kSchemaHeaders` | `request.headers` | 400 |
| **Response** | `kSchemaResponse` | `reply.send()` payload | 500 |

## Response Schema Validation

```mermaid
flowchart TD
    ResponsePayload[Response Payload] --> StatusCode{Status Code}
    
    StatusCode --> Status200[200 Schema]
    StatusCode --> Status201[201 Schema]
    StatusCode --> Status400[400 Schema]
    StatusCode --> Status500[500 Schema]
    StatusCode --> DefaultSchema[default Schema]
    
    Status200 --> Serialize200[Serialize with 200 Schema]
    Status201 --> Serialize201[Serialize with 201 Schema]
    Status400 --> Serialize400[Serialize with 400 Schema]
    Status500 --> Serialize500[Serialize with 500 Schema]
    DefaultSchema --> SerializeDefault[Serialize with default Schema]
    
    Serialize200 --> Output[JSON Output]
    Serialize201 --> Output
    Serialize400 --> Output
    Serialize500 --> Output
    SerializeDefault --> Output
```

## Validation Error Format

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "body.email should match format \"email\"",
  "validation": [
    {
      "keyword": "format",
      "dataPath": ".email",
      "schemaPath": "#/properties/email/format",
      "params": {
        "format": "email"
      },
      "message": "should match format \"email\""
    }
  ]
}
```

## Performance Characteristics

| Operation | Performance | Implementation |
|-----------|-------------|----------------|
| **Schema Compilation** | ~10ms per schema | One-time cost during route registration |
| **Request Validation** | ~10μs per request | Pre-compiled AJV validators |
| **Response Serialization** | ~5μs per response | Pre-compiled fast-json-stringify |
| **Memory Overhead** | ~1KB per schema | Compiled function storage |

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `removeAdditional` | `boolean` | `true` | Remove additional properties |
| `useDefaults` | `boolean` | `true` | Fill in default values |
| `coerceTypes` | `boolean` | `true` | Type coercion for params/query |
| `allErrors` | `boolean` | `false` | Return all validation errors |
| `nullable` | `boolean` | `false` | Allow null values |