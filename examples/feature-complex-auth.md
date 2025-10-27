# Implement OAuth2 Authentication System

## Description

Build complete OAuth2 authentication system supporting Google, GitHub, and email/password login. Includes token management, session handling, and security features.

**Type**: Feature
**Complexity**: Complex
**Estimated Time**: 2 weeks

## Acceptance Criteria

### Core Authentication
- [ ] Users can register with email/password
- [ ] Users can log in with email/password
- [ ] Users can log in with Google OAuth
- [ ] Users can log in with GitHub OAuth
- [ ] Password reset via email works
- [ ] Email verification required for new accounts
- [ ] MFA (TOTP) optional for enhanced security

### Token Management
- [ ] JWT access tokens (15min expiry)
- [ ] Refresh tokens (30 day expiry)
- [ ] Token rotation on refresh
- [ ] Token revocation endpoint
- [ ] Secure token storage (httpOnly cookies)

### Session Management
- [ ] Active sessions list in user settings
- [ ] Ability to revoke individual sessions
- [ ] Logout from all devices
- [ ] Session timeout after 30 days inactivity

### Security
- [ ] Passwords hashed with bcrypt (cost factor 12)
- [ ] Rate limiting on login attempts (5/min)
- [ ] Account lockout after 5 failed attempts
- [ ] CSRF protection on state-changing endpoints
- [ ] Secure password requirements enforced

## Technical Architecture

### Components

**Backend Services:**
- `AuthService`: Core authentication logic
- `TokenService`: JWT generation/validation
- `OAuthService`: OAuth provider integration
- `EmailService`: Verification/reset emails
- `MFAService`: TOTP generation/validation

**Database Tables:**
```sql
users (
  id UUID PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT,  -- null for OAuth-only users
  email_verified BOOLEAN DEFAULT FALSE,
  mfa_enabled BOOLEAN DEFAULT FALSE,
  mfa_secret TEXT,
  created_at TIMESTAMP
)

oauth_accounts (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  provider TEXT NOT NULL,  -- 'google', 'github'
  provider_user_id TEXT NOT NULL,
  access_token TEXT,
  refresh_token TEXT,
  UNIQUE(provider, provider_user_id)
)

refresh_tokens (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  token TEXT UNIQUE NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  revoked BOOLEAN DEFAULT FALSE,
  user_agent TEXT,
  ip_address TEXT
)

password_reset_tokens (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  token TEXT UNIQUE NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  used BOOLEAN DEFAULT FALSE
)
```

**API Endpoints:**
- POST `/auth/register` - Email/password registration
- POST `/auth/login` - Email/password login
- POST `/auth/logout` - Logout current session
- POST `/auth/refresh` - Refresh access token
- POST `/auth/forgot-password` - Request password reset
- POST `/auth/reset-password` - Complete password reset
- GET `/auth/verify-email?token=...` - Verify email address
- GET `/auth/oauth/google` - Initiate Google OAuth
- GET `/auth/oauth/google/callback` - Google OAuth callback
- GET `/auth/oauth/github` - Initiate GitHub OAuth
- GET `/auth/oauth/github/callback` - GitHub OAuth callback
- POST `/auth/mfa/enable` - Enable MFA
- POST `/auth/mfa/verify` - Verify MFA code
- GET `/auth/sessions` - List active sessions
- DELETE `/auth/sessions/:id` - Revoke session

## Implementation Plan

### Phase 1: Core Auth (Week 1)
1. Database schema and migrations
2. Email/password registration
3. Email/password login
4. JWT token generation/validation
5. Basic session management
6. Password hashing

### Phase 2: OAuth Integration (Week 1)
7. Google OAuth setup and configuration
8. GitHub OAuth setup and configuration
9. OAuth callback handling
10. Account linking (email ↔ OAuth)
11. OAuth token storage

### Phase 3: Security Features (Week 2)
12. Email verification flow
13. Password reset flow
14. Rate limiting implementation
15. Account lockout logic
16. CSRF protection
17. MFA (TOTP) implementation

### Phase 4: Session Management (Week 2)
18. Refresh token rotation
19. Token revocation
20. Active sessions UI
21. Logout from all devices
22. Session timeout handling

## Testing Strategy

### Unit Tests
- [ ] Password hashing/verification
- [ ] JWT generation/validation
- [ ] OAuth token exchange
- [ ] MFA code generation/verification
- [ ] Email template rendering

### Integration Tests
- [ ] Full registration flow
- [ ] Full login flow (email + OAuth)
- [ ] Password reset flow
- [ ] Email verification flow
- [ ] Token refresh flow
- [ ] MFA enrollment and validation
- [ ] Session management operations

### Security Tests
- [ ] Brute force protection
- [ ] CSRF attack prevention
- [ ] Token expiry enforcement
- [ ] Authorization bypass attempts
- [ ] SQL injection attempts
- [ ] XSS prevention

### Load Tests
- [ ] 1000 concurrent login requests
- [ ] Token refresh under load
- [ ] OAuth callback handling

## Documentation Requirements

- [ ] API endpoint documentation (OpenAPI)
- [ ] OAuth setup guide for developers
- [ ] Security best practices guide
- [ ] User-facing authentication docs
- [ ] Troubleshooting guide

## Security Considerations

**Passwords:**
- Minimum 8 characters, require uppercase, lowercase, number, symbol
- Bcrypt with cost factor 12
- Never log passwords or password hashes

**Tokens:**
- Short-lived access tokens (15 minutes)
- Longer refresh tokens (30 days)
- Rotate refresh tokens on use
- Store in httpOnly, secure, sameSite cookies
- Revoke on logout

**Rate Limiting:**
- Login: 5 attempts per 5 minutes per IP
- Registration: 3 per hour per IP
- Password reset: 3 per hour per email

**OAuth Security:**
- Validate state parameter (CSRF protection)
- Use PKCE for public clients
- Verify OAuth provider signatures
- Store provider tokens encrypted

**MFA:**
- TOTP with 30-second window
- 6-digit codes
- Backup codes (10 single-use codes)
- Rate limit MFA attempts

## Dependencies

- `bcrypt` - Password hashing
- `jsonwebtoken` - JWT generation/validation
- `passport` - OAuth integration
- `speakeasy` - TOTP generation
- `nodemailer` - Email sending
- `ioredis` - Rate limiting state

## Rollout Plan

1. Deploy to staging
2. Security audit by external firm
3. Penetration testing
4. Beta test with 100 internal users
5. Gradual rollout: 10%, 50%, 100%
6. Monitor error rates and auth failures
7. User communication about new auth features

## Risks and Mitigations

**Risk:** OAuth provider downtime
**Mitigation:** Email/password always available, graceful degradation

**Risk:** Token compromise
**Mitigation:** Short expiry, rotation, revocation endpoint

**Risk:** Database breach
**Mitigation:** Bcrypt makes passwords expensive to crack, separate secrets storage

**Risk:** High authentication latency
**Mitigation:** Caching, connection pooling, geographic distribution

## Success Metrics

- Authentication success rate > 99%
- Login latency < 500ms (p95)
- Zero critical security vulnerabilities
- User satisfaction with auth experience > 4/5
