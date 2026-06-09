---
status: approved
name: backend-patterns
source: everything-claude-code
domain: backend
description: >
  Backend patterns for a Go+CDK+AWS serverless monorepo.
  Adapted from everything-claude-code for neobank context.
---

# Backend Patterns

## API Patterns

### Request/Response Lifecycle
- Parse and validate input at the handler layer - fail fast with typed errors
- Business logic lives exclusively in the service layer
- Data access is isolated in the repository layer
- Return structured error responses with domain-specific error codes

### Error Handling
- Use typed error hierarchies (e.g., `ApiError` with domain codes)
- Wrap errors with context at each layer boundary
- Log errors once at the outermost boundary - no duplicate logging
- Map internal errors to appropriate HTTP status codes at the handler

### Idempotency
- All write operations should be idempotent where possible
- Use DynamoDB condition expressions to prevent duplicate processing
- Include idempotency keys in request contracts for critical operations

## Caching Patterns

### SSM Parameter Store
- Branch-scoped namespace: `/{branchName}/core/{resourceRef}`
- Cache SSM values with TTL; refresh on Lambda cold start
- Never hardcode configuration values - always resolve from SSM

### DynamoDB Patterns
- Atomic updates via condition expressions - never full-object overwrites
- Single-table design where domain boundaries allow
- Use GSIs for access patterns, not scan operations
- Optimistic locking with version attributes for concurrent writes

## Service Communication
- Prefer direct Lambda invocation for synchronous cross-service calls
- Use EventBridge for asynchronous event-driven communication
- Step Functions for multi-step workflows with retry and error handling
- All inter-service contracts defined explicitly
