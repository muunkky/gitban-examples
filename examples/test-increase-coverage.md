# Increase Test Coverage for Auth Module

## Description

Increase test coverage for the authentication module from current 65% to target 90%. Focus on edge cases, error paths, and integration scenarios currently not covered.

**Type**: Test
**Complexity**: Medium
**Estimated Time**: 3 days

## Current Coverage Analysis

**Overall Auth Module:** 65% coverage (target: 90%)

### Coverage by Component:
| Component | Current | Target | Gap |
|-----------|---------|--------|-----|
| AuthService | 80% | 95% | ❌ Missing error paths |
| TokenService | 90% | 95% | ✅ Nearly complete |
| OAuthService | 45% | 85% | ❌ Missing provider tests |
| EmailService | 60% | 85% | ❌ Missing template tests |
| MFAService | 50% | 90% | ❌ Missing edge cases |

### Uncovered Code Paths:

**AuthService:**
- ❌ Account lockout after failed attempts
- ❌ Password reset token expiry
- ❌ Concurrent login handling
- ❌ Database connection errors

**OAuthService:**
- ❌ OAuth provider timeout
- ❌ Invalid OAuth callback state
- ❌ Token refresh failure
- ❌ Account linking conflicts

**EmailService:**
- ❌ Email delivery failures
- ❌ Template rendering errors
- ❌ Rate limit exceeded

**MFAService:**
- ❌ TOTP time window edge cases
- ❌ Backup code exhaustion
- ❌ Invalid QR code generation

## Test Plan

### Unit Tests to Add

**AuthService (15 tests):**
- [ ] Login fails after 5 attempts (account lockout)
- [ ] Lockout clears after 30 minutes
- [ ] Password reset token expires after 1 hour
- [ ] Expired token rejected
- [ ] Database connection error handled gracefully
- [ ] Concurrent logins from different IPs
- [ ] Invalid email format rejected
- [ ] SQL injection attempt blocked
- [ ] Password complexity validation
- [ ] Case-insensitive email matching
- [ ] User creation with duplicate email fails
- [ ] Session creation with invalid user fails
- [ ] Token generation with missing claims fails
- [ ] Token validation with expired token fails
- [ ] Refresh token rotation works correctly

**OAuthService (10 tests):**
- [ ] OAuth provider timeout returns error
- [ ] Invalid state parameter rejected (CSRF)
- [ ] Missing authorization code handled
- [ ] Token exchange failure handled
- [ ] Account linking with existing OAuth account
- [ ] OAuth token refresh on expiry
- [ ] Provider returns invalid user info
- [ ] Multiple OAuth providers for same user
- [ ] OAuth account unlinking
- [ ] Provider-specific error codes mapped correctly

**EmailService (8 tests):**
- [ ] Email delivery failure logged
- [ ] Template rendering with missing variables
- [ ] HTML escaping prevents XSS
- [ ] Rate limit exceeded returns error
- [ ] Invalid recipient email rejected
- [ ] Email queue full handled
- [ ] Template not found error
- [ ] SMTP connection timeout

**MFAService (10 tests):**
- [ ] TOTP code valid within 30s window
- [ ] TOTP code invalid outside window
- [ ] Time sync issues handled (±1 window)
- [ ] Backup codes work for login
- [ ] Backup code used once can't be reused
- [ ] All backup codes exhausted warning
- [ ] Invalid TOTP secret handled
- [ ] QR code generation with invalid params
- [ ] MFA disable requires password confirmation
- [ ] MFA codes not accepted before enabled

### Integration Tests to Add

- [ ] Full registration → email verification → login flow
- [ ] OAuth login → account linking → profile access
- [ ] Password reset → email → token → new password → login
- [ ] MFA setup → QR scan → verify code → login with MFA
- [ ] Account lockout → wait → retry → success
- [ ] Concurrent sessions from multiple devices
- [ ] Token refresh during active session
- [ ] Logout invalidates tokens across services

### Edge Case Tests

- [ ] Very long passwords (>1000 chars)
- [ ] Unicode characters in emails/passwords
- [ ] Null/undefined inputs
- [ ] Empty strings for required fields
- [ ] Maximum rate limit boundary
- [ ] Clock skew during token validation
- [ ] Leap second handling in timestamps

## Implementation Approach

**Week 1: Unit Tests**
- Days 1-2: AuthService tests (15 tests)
- Day 3: OAuthService tests (10 tests)
- Day 4: EmailService tests (8 tests)
- Day 5: MFAService tests (10 tests)

**Week 2: Integration & Edge Cases**
- Days 1-2: Integration tests (8 tests)
- Day 3: Edge case tests (7 tests)
- Day 4: Coverage analysis and gap fill
- Day 5: Documentation and cleanup

## Tools and Setup

- **Framework**: Jest
- **Mocking**: jest.mock() for external services
- **Coverage**: jest --coverage --coverageThreshold='{"global":{"branches":90,"functions":90,"lines":90}}'
- **CI**: Run on every PR, block merge if coverage drops

## Success Criteria

- [ ] Overall auth module coverage ≥ 90%
- [ ] All components ≥ 85% coverage
- [ ] All critical paths covered (login, OAuth, MFA)
- [ ] Zero flaky tests
- [ ] All tests pass in CI
- [ ] Test execution time < 30 seconds
- [ ] Code review approved by team lead

## Documentation

- [ ] Update test README with new patterns
- [ ] Document mocking strategies
- [ ] Add coverage badge to repo README
- [ ] Create testing best practices doc

## Risks and Mitigations

**Risk:** Tests are slow
**Mitigation:** Use mocks for external services, parallel test execution

**Risk:** Flaky tests due to timing issues
**Mitigation:** Avoid real timers, use jest.useFakeTimers()

**Risk:** Hard to test certain OAuth flows
**Mitigation:** Use recorded HTTP interactions, mock OAuth provider responses
