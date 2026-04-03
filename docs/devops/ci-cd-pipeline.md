# CI/CD Pipeline

This document describes the continuous integration and deployment pipeline for the Fastify web framework.

## Pipeline Overview

The CI/CD pipeline ensures code quality, compatibility, and reliability through automated testing and validation across multiple environments.

```mermaid
flowchart TD
    A[Code Push/PR] --> B[Dependency Review]
    A --> C[License Check]
    A --> D[Lint Code]
    
    D --> E[Coverage Tests]
    E --> F[Unit Tests Matrix]
    E --> G[Integration Tests]
    E --> H[Type Tests]
    E --> I[Performance Tests]
    
    F --> J[Security Scans]
    G --> J
    H --> J
    I --> J
    
    J --> K{All Tests Pass?}
    K -->|No| L[Block Merge]
    K -->|Yes| M[Allow Merge]
    
    M --> N{Is Tagged Release?}
    N -->|Yes| O[Publish to NPM]
    N -->|No| P[Update Documentation]
    
    O --> Q[Update Website]
    P --> Q
    Q --> R[Notify Community]
    
    subgraph "Test Matrix"
        F1[Node 20 - Ubuntu]
        F2[Node 22 - Ubuntu]  
        F3[Node 24 - Ubuntu]
        F4[Node 20 - macOS]
        F5[Node 22 - macOS]
        F6[Node 24 - macOS]
        F7[Node 20 - Windows]
        F8[Node 22 - Windows]
        F9[Node 24 - Windows]
    end
    
    F --> F1
    F --> F2
    F --> F3
    F --> F4
    F --> F5
    F --> F6
    F --> F7
    F --> F8
    F --> F9
```

## Trigger Conditions

| Event | Workflows Triggered | Purpose |
|-------|-------------------|---------|
| **Push to main** | Full CI pipeline | Validate production-ready code |
| **Push to next** | Full CI pipeline | Test upcoming features |
| **Push to *.x** | Full CI pipeline | Validate maintenance releases |
| **Pull Request** | Full CI pipeline | Review and validate changes |
| **Manual Dispatch** | Custom Node.js version testing | Test against specific Node versions |
| **Scheduled** | Security scans, performance benchmarks | Regular maintenance |
| **Release Tag** | Publication pipeline | Deploy to NPM registry |

## Pipeline Stages

### 1. Code Quality Gates

**Dependency Review** (Pull requests only)
- Scans new dependencies for security vulnerabilities
- Reviews license compatibility
- Blocks PRs with vulnerable dependencies

**License Verification**
- Validates all production dependencies use approved licenses
- Allowed: 0BSD, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, MIT
- Fails if unlicensed or non-approved licenses found

**Code Linting**
- Runs ESLint with project-specific rules
- Enforces coding standards and best practices
- Must pass before other tests execute

### 2. Testing Matrix

**Coverage Testing**
```mermaid
flowchart LR
    A[Coverage Tests] --> B[Unix Coverage]
    A --> C[Windows Coverage]
    B --> D[Generate Reports]
    C --> D
    D --> E[Combine Coverage]
    E --> F[Verify 100% Requirement]
```

**Unit Testing Matrix**
- **Node.js Versions**: 20, 22, 24 (LTS and current)
- **Operating Systems**: Ubuntu, macOS, Windows
- **Total Combinations**: 9 test environments
- **Execution**: Parallel across all combinations
- **Failure Policy**: Any failure blocks the pipeline

**TypeScript Validation**
- Compiles TypeScript definitions
- Validates type compatibility
- Runs type-specific tests with tsd

**Integration Testing**
- Tests with alternative Node.js runtimes
- Validates plugin ecosystem compatibility
- Cross-version compatibility testing

### 3. Performance Validation

**Benchmark Testing**
- HTTP request throughput testing with AutoCannon
- Parser performance validation
- Regression detection against baseline
- Performance metrics reporting

**Load Testing Scripts**
| Script | Purpose | Metrics |
|--------|---------|---------|
| `npm run benchmark` | Basic HTTP performance | Requests/sec, latency |
| `npm run benchmark:parser` | JSON parsing speed | Parser throughput |
| `npm run benchmark:parser:error` | Error handling performance | Error path latency |

### 4. Security Scanning

**Vulnerability Assessment**
- NPM audit for known vulnerabilities
- Dependency chain security analysis
- License compliance validation
- Security advisory integration

**Code Security**
- Static analysis for security patterns
- Injection vulnerability detection
- Secrets scanning (keys, tokens)

### 5. Publication Pipeline

**Pre-publication Validation**
```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant NPM as NPM Registry
    participant Users as End Users
    
    Dev->>GH: Create release tag
    GH->>GH: Run full test suite
    GH->>GH: Validate package integrity
    GH->>GH: Build documentation
    GH->>NPM: Publish package
    NPM->>Users: Package available
    GH->>GH: Update website
    GH->>Users: Notify community
```

**Publication Steps**
1. **Integrity Check**: Validates built files match source
2. **Version Sync**: Updates version across all files  
3. **Package Build**: Creates distribution package
4. **NPM Publication**: Publishes to public registry
5. **Documentation Update**: Refreshes docs and website
6. **Community Notification**: Updates changelog and announcements

## Environment Variables

The CI/CD pipeline uses several environment variables for configuration:

| Variable | Usage | Scope |
|----------|--------|-------|
| `PREPUBLISH` | Publication validation flag | Pre-publication |
| `GITHUB_TOKEN` | Repository access | All workflows |
| `NPM_TOKEN` | Registry publishing | Publication only |
| `NODE_VERSION` | Custom Node.js version testing | Manual dispatch |

**Note**: All secrets are managed through GitHub Secrets and never exposed in logs.

## Quality Metrics

### Test Coverage Requirements
- **Minimum Coverage**: 100% line coverage required
- **Coverage Tools**: c8 (V8 JavaScript coverage)
- **Coverage Verification**: Automated check blocks failing builds
- **Coverage Reporting**: HTML reports generated for analysis

### Performance Thresholds
- **Request Latency**: p99 < 1ms for simple routes
- **Throughput**: > 100k requests/sec baseline
- **Memory Usage**: < 50MB framework overhead
- **Startup Time**: < 100ms for basic server

### Security Standards
- **Dependency Scanning**: Daily vulnerability checks
- **License Compliance**: Only approved open-source licenses
- **Security Patches**: Released within 48 hours of disclosure
- **CVE Response**: Coordinated disclosure process

## Failure Handling

### Build Failures
- **Immediate Notification**: Maintainers notified within 5 minutes
- **Failure Analysis**: Automated logs and test reports
- **Quick Recovery**: Rollback procedures for critical issues
- **Post-Mortem**: Analysis for repeated failures

### Recovery Procedures
- **Test Flakiness**: Re-run failing tests up to 3 times
- **Infrastructure Issues**: Automatic retry with different runners  
- **Dependency Failures**: Fallback to cached dependencies
- **Publication Failures**: Manual intervention with maintainer approval

## Monitoring and Alerting

### Pipeline Health
- **Success Rate**: Track percentage of successful builds
- **Duration Monitoring**: Alert on pipeline duration increases
- **Flaky Test Detection**: Identify and fix unstable tests
- **Resource Usage**: Monitor GitHub Actions minutes consumption

### Release Monitoring
- **Download Metrics**: Track NPM package downloads
- **Version Adoption**: Monitor version distribution in the wild
- **Issue Correlation**: Link releases to issue reports
- **Performance Impact**: Monitor real-world performance metrics