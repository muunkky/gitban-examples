# Implement User Profile API Endpoint

## Description

Create REST API endpoint for fetching and updating user profile information. Includes authentication, validation, and database operations.

**Type**: Feature
**Complexity**: Medium
**Estimated Time**: 1 day

## Acceptance Criteria

- [ ] GET /api/users/:id endpoint returns user profile
- [ ] PUT /api/users/:id endpoint updates user profile
- [ ] Endpoints require authentication (JWT)
- [ ] Authorization checks ensure users can only access their own profile
- [ ] Input validation rejects invalid data
- [ ] Database queries are efficient with proper indexing
- [ ] API returns appropriate HTTP status codes
- [ ] Error responses follow standard format
- [ ] Audit log records profile changes

## Technical Design

### Endpoints

**GET /api/users/:id**
- Returns: `{ id, email, name, avatar_url, created_at, updated_at }`
- Auth: Bearer token required
- Errors: 401 Unauthorized, 403 Forbidden, 404 Not Found

**PUT /api/users/:id**
- Body: `{ name?, avatar_url? }`
- Returns: Updated user object
- Auth: Bearer token required
- Validation: Name 1-100 chars, avatar_url must be valid URL
- Errors: 400 Bad Request, 401 Unauthorized, 403 Forbidden

### Database Schema

```sql
-- Already exists, no migrations needed
users (
  id UUID PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  avatar_url TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
)
```

## Implementation Steps

1. Create route handlers in `src/routes/users.ts`
2. Add authentication middleware
3. Add authorization checks (user can only edit self)
4. Implement input validation with Zod schema
5. Add database queries with Prisma
6. Add audit logging for profile updates
7. Write error handling
8. Add rate limiting (10 requests/minute per user)

## Testing Strategy

- [ ] Unit tests: Route handlers with mocked DB
- [ ] Unit tests: Validation schema edge cases
- [ ] Integration tests: Full request/response cycle
- [ ] Integration tests: Authentication/authorization flows
- [ ] Integration tests: Database operations
- [ ] Load test: 100 concurrent requests
- [ ] Security test: Authorization bypass attempts

## Documentation Updates

- [ ] Add API endpoint docs to OpenAPI spec
- [ ] Update API documentation website
- [ ] Add example requests/responses

## Security Considerations

- JWT token validated and not expired
- User ID in token matches requested resource
- Input sanitized to prevent SQL injection (using Prisma)
- Rate limiting prevents abuse
- Sensitive fields (password hash) excluded from responses
