# Update Dependencies to Latest Versions

## Description

Update all npm dependencies to their latest stable versions. Focus on security patches and major version upgrades with breaking changes.

**Type**: Chore (Maintenance)
**Priority**: P1 (security updates included)
**Estimated Time**: 1 day

## Motivation

- 3 high-severity security vulnerabilities in current dependencies
- React 18.2 → 18.3 includes performance improvements
- Several packages have reached end-of-life

## Packages to Update

### Critical Security Updates
- [ ] `axios` 1.4.0 → 1.6.2 (CVE-2023-45857 - SSRF vulnerability)
- [ ] `express` 4.18.2 → 4.18.5 (security patches)
- [ ] `jsonwebtoken` 9.0.0 → 9.0.2 (CVE-2022-23529)

### Major Version Updates (Breaking Changes)
- [ ] `react` 18.2.0 → 18.3.1
- [ ] `react-router-dom` 6.10.0 → 6.20.0
- [ ] `typescript` 5.0.4 → 5.3.3
- [ ] `eslint` 8.40.0 → 8.56.0

### Minor/Patch Updates (Safe)
- [ ] `@tanstack/react-query` 4.29.0 → 4.36.1
- [ ] `zod` 3.21.4 → 3.22.4
- [ ] `date-fns` 2.30.0 → 3.0.0
- [ ] `lodash` 4.17.21 → 4.17.21 (no update needed)

## Update Strategy

### Phase 1: Security Patches (Immediate)
```bash
npm update axios express jsonwebtoken
npm audit fix
```

### Phase 2: Major Versions (Carefully)
1. Update one major package at a time
2. Run tests after each update
3. Check for deprecation warnings
4. Update code for breaking changes

### Phase 3: Minor/Patch (Batch)
```bash
npm update
```

## Breaking Changes to Address

### React 18.3
- No breaking changes (patch release)
- New features: React Compiler support

### React Router 6.20
- `useNavigate()` now supports `unstable_viewTransition`
- Deprecated APIs removed (we're not using any)

### TypeScript 5.3
- Stricter import resolution
- May need to update `moduleResolution` in tsconfig.json

### date-fns 3.0
- ESM-first, may need import updates
- Some function signatures changed
- Check date formatting calls

## Testing Plan

### Automated Tests
- [ ] Run full test suite: `npm test`
- [ ] Run type checking: `npm run type-check`
- [ ] Run linter: `npm run lint`
- [ ] Run build: `npm run build`

### Manual Testing
- [ ] Test user login flow
- [ ] Test dashboard rendering
- [ ] Test API calls
- [ ] Test date formatting
- [ ] Test error handling

### Smoke Tests in Staging
- [ ] Deploy to staging environment
- [ ] Run E2E test suite
- [ ] Manual QA of critical paths
- [ ] Performance testing (check for regressions)

## Rollback Plan

If issues discovered:
1. Git revert the dependency update commit
2. Redeploy previous version
3. Investigate issue in separate branch
4. Create hotfix if security-critical

## Implementation Steps

1. Create feature branch: `chore/dependency-updates`
2. Update security patches first
3. Run `npm audit` to verify fixes
4. Update major versions one by one
5. Fix breaking changes
6. Update minor/patch versions
7. Run full test suite
8. Update package-lock.json
9. Document any code changes needed
10. PR review with changelog

## Documentation Updates

- [ ] Update CHANGELOG.md with dependency changes
- [ ] Document any new peer dependency requirements
- [ ] Update README if Node.js version requirement changed
- [ ] Note any breaking changes for other developers

## Post-Update Verification

- [ ] All tests passing
- [ ] No new ESLint warnings
- [ ] No TypeScript errors
- [ ] Bundle size not significantly increased
- [ ] `npm audit` shows 0 vulnerabilities
- [ ] Application runs in dev mode
- [ ] Production build succeeds

## Risk Assessment

**High Risk:**
- Major version updates may break functionality
- TypeScript strictness could reveal hidden bugs

**Medium Risk:**
- Performance regressions from package changes
- Bundle size increase

**Low Risk:**
- Security patch updates (minimal breaking changes)

**Mitigation:**
- Thorough testing at each stage
- Deploy to staging before production
- Have rollback plan ready

## Success Criteria

- [ ] All dependencies updated to latest stable versions
- [ ] Zero high/critical security vulnerabilities
- [ ] All tests passing
- [ ] Application works in staging
- [ ] No performance regressions
- [ ] PR approved and merged
