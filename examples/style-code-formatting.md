# Fix Code Formatting with Prettier

## Description

Apply Prettier formatting to entire codebase to ensure consistent code style. Currently, files have inconsistent indentation, quote styles, and line endings.

**Type**: Style
**Priority**: P2
**Estimated Time**: 2 hours

## Current Issues

- Mixed tabs and spaces (especially in older files)
- Inconsistent quote styles (some single, some double)
- Mixed line endings (CRLF vs LF)
- Trailing whitespace and commas
- Inconsistent object/array formatting

## Changes Needed

### Prettier Configuration
Create `.prettierrc`:
```json
{
  "semi": true,
  "singleQuote": false,
  "trailingComma": "es5",
  "tabWidth": 2,
  "printWidth": 100,
  "endOfLine": "lf"
}
```

### Files to Format
- All `.ts` and `.tsx` files
- All `.js` and `.jsx` files
- All `.json` files
- All `.md` files

### Ignore Patterns
Add `.prettierignore`:
```
node_modules/
dist/
build/
coverage/
*.min.js
package-lock.json
```

## Implementation Steps

1. **Install Prettier**: `npm install --save-dev prettier`
2. **Add config files**: Create `.prettierrc` and `.prettierignore`
3. **Format all files**: `npx prettier --write "src/**/*.{ts,tsx,js,jsx,json,md}"`
4. **Verify changes**: Git diff to review formatting changes
5. **Add pre-commit hook**: Use husky to auto-format on commit
6. **Update CI**: Add prettier check to CI pipeline

## Pre-commit Hook Setup

```bash
npm install --save-dev husky lint-staged
npx husky init
```

Add to `.husky/pre-commit`:
```bash
npx lint-staged
```

Add to `package.json`:
```json
{
  "lint-staged": {
    "*.{ts,tsx,js,jsx,json,md}": "prettier --write"
  }
}
```

## Verification

- [ ] Run prettier on all files
- [ ] Review git diff (should only be formatting changes)
- [ ] Verify no functionality changes
- [ ] Test pre-commit hook works
- [ ] Verify CI prettier check passes

## Success Criteria

- All files pass `prettier --check`
- Git commit history shows clean formatting changes
- Pre-commit hook prevents future formatting issues
- Team adopts prettier in their editors
