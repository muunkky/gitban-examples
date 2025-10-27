# Refactor Authentication Module to Reduce Complexity

## Description

Refactor the authentication module to reduce cyclomatic complexity, improve testability, and extract reusable patterns. Current auth code has grown organically and needs cleanup.

**Type**: Refactor
**Motivation**: Code quality, maintainability
**Estimated Time**: 1 week

## Current Problems

### Code Smells Identified:

1. **God Object**: `AuthService` has 2000+ lines, handles too many responsibilities
2. **Duplicate Logic**: Password validation repeated in 4 places
3. **Long Methods**: `loginWithEmail()` is 150 lines
4. **Tight Coupling**: Direct database calls mixed with business logic
5. **Hard to Test**: Mocking requires 10+ dependencies
6. **Low Cohesion**: Email sending logic mixed with token generation

### Metrics (Current):
- **Lines of Code**: 2,147 lines in AuthService.ts
- **Cyclomatic Complexity**: 35 (threshold: 10)
- **Method Length**: Avg 45 lines (threshold: 20)
- **Test Coverage**: 65% (target: 90%)
- **Dependencies**: 15 direct imports

## Refactoring Goals

### Target Metrics:
- ✅ Reduce AuthService to < 500 lines
- ✅ Cyclomatic complexity < 10 per method
- ✅ Average method length < 20 lines
- ✅ Test coverage > 85%
- ✅ Reduce dependencies to < 8

### Design Improvements:
- **Separation of Concerns**: Split by responsibility
- **Single Responsibility**: Each class does one thing
- **Dependency Injection**: Easier testing and flexibility
- **Strategy Pattern**: Pluggable auth providers
- **Repository Pattern**: Abstract data access

## Proposed Architecture

### Before (Current):
```
AuthService (2000 lines)
  ├─ loginWithEmail()
  ├─ loginWithOAuth()
  ├─ registerUser()
  ├─ resetPassword()
  ├─ verifyEmail()
  ├─ generateTokens()
  ├─ validatePassword()
  ├─ sendEmail()
  └─ ... 20 more methods
```

### After (Refactored):
```
AuthFacade (100 lines)
  ├─ AuthenticationService (300 lines)
  │   ├─ EmailAuthProvider
  │   ├─ OAuthProvider
  │   └─ MFAProvider
  ├─ TokenService (200 lines)
  ├─ PasswordService (150 lines)
  ├─ EmailService (150 lines)
  └─ UserRepository (200 lines)
```

## Refactoring Plan

### Phase 1: Extract Services (Days 1-2)

**1. Extract TokenService:**
```typescript
// Before: Mixed in AuthService
generateAccessToken(user) { ... }
generateRefreshToken(user) { ... }
validateToken(token) { ... }

// After: Separate service
class TokenService {
  generateAccessToken(user: User): string
  generateRefreshToken(user: User): string
  validateToken(token: string): TokenPayload
  refreshTokens(refreshToken: string): Tokens
}
```

**2. Extract PasswordService:**
```typescript
class PasswordService {
  hash(password: string): Promise<string>
  verify(password: string, hash: string): Promise<boolean>
  validate(password: string): ValidationResult
  generateResetToken(userId: string): string
  validateResetToken(token: string): boolean
}
```

**3. Extract EmailService:**
```typescript
class EmailService {
  sendVerificationEmail(user: User): Promise<void>
  sendPasswordResetEmail(user: User, token: string): Promise<void>
  sendWelcomeEmail(user: User): Promise<void>
}
```

### Phase 2: Implement Strategy Pattern (Days 3-4)

**Auth Provider Interface:**
```typescript
interface AuthProvider {
  authenticate(credentials: unknown): Promise<User>
  supports(method: AuthMethod): boolean
}

class EmailAuthProvider implements AuthProvider {
  authenticate(credentials: EmailCredentials): Promise<User>
}

class OAuthProvider implements AuthProvider {
  authenticate(credentials: OAuthCredentials): Promise<User>
}

class AuthenticationService {
  private providers: AuthProvider[]

  async authenticate(method: AuthMethod, credentials: unknown) {
    const provider = this.providers.find(p => p.supports(method))
    return provider.authenticate(credentials)
  }
}
```

### Phase 3: Extract Repository (Day 5)

**UserRepository:**
```typescript
class UserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  create(data: CreateUserData): Promise<User>
  update(id: string, data: UpdateUserData): Promise<User>
  delete(id: string): Promise<void>
}
```

### Phase 4: Create Facade (Day 6)

**AuthFacade** (simplified API for consumers):
```typescript
class AuthFacade {
  constructor(
    private authService: AuthenticationService,
    private tokenService: TokenService,
    private passwordService: PasswordService,
    private emailService: EmailService
  ) {}

  async login(email: string, password: string): Promise<AuthResult>
  async register(data: RegisterData): Promise<AuthResult>
  async resetPassword(email: string): Promise<void>
  async verifyEmail(token: string): Promise<void>
}
```

### Phase 5: Migrate and Test (Day 7)

1. Update all consumers to use new AuthFacade
2. Run tests after each migration
3. Remove old AuthService code
4. Verify no regressions

## Testing Strategy

### Unit Tests (Per Service)
- [ ] TokenService: 15 tests
- [ ] PasswordService: 12 tests
- [ ] EmailService: 8 tests
- [ ] EmailAuthProvider: 10 tests
- [ ] OAuthProvider: 10 tests
- [ ] UserRepository: 12 tests
- [ ] AuthFacade: 15 tests

### Integration Tests
- [ ] Full login flow with EmailAuthProvider
- [ ] Full registration flow
- [ ] Password reset flow
- [ ] OAuth login flow
- [ ] Token refresh flow

### Refactoring Safety
- [ ] All tests passing before refactoring
- [ ] Tests passing after each extraction
- [ ] No behavior changes
- [ ] Same API surface for consumers

## Migration Strategy

**Gradual Rollout:**
1. Create new services alongside old code
2. Route 10% of traffic to new code
3. Monitor errors and performance
4. Increase to 50%, then 100%
5. Remove old code after 1 week

**Feature Flags:**
```typescript
if (featureFlags.newAuthArchitecture) {
  return authFacade.login(email, password)
} else {
  return oldAuthService.login(email, password)
}
```

## Benefits

### Maintainability
- ✅ Smaller, focused classes (easier to understand)
- ✅ Clear separation of concerns
- ✅ Easier to locate and fix bugs

### Testability
- ✅ Easier to mock dependencies
- ✅ Isolated unit tests
- ✅ Faster test execution

### Extensibility
- ✅ Easy to add new auth providers (SAML, LDAP)
- ✅ Pluggable email providers
- ✅ Strategy pattern enables customization

### Performance
- ✅ Lazy loading of services
- ✅ Better caching opportunities
- ✅ Parallel execution of independent operations

## Risks

**Risk:** Breaking existing functionality
**Mitigation:** Comprehensive test coverage, gradual rollout, feature flags

**Risk:** Regression in performance
**Mitigation:** Performance benchmarks before/after, profiling

**Risk:** Increased complexity from more classes
**Mitigation:** Good documentation, clear boundaries, facade pattern

## Success Criteria

- [ ] AuthService reduced from 2000 to < 500 lines
- [ ] Cyclomatic complexity < 10 per method
- [ ] Test coverage > 85%
- [ ] All tests passing
- [ ] Zero regressions in production
- [ ] Code review approved
- [ ] Documentation updated
