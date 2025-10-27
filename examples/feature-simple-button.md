# Add Submit Button to Form

## Description

Add a submit button to the user registration form. The button should be styled according to the design system and properly validate form inputs before submission.

**Type**: Feature
**Complexity**: Simple
**Estimated Time**: 1-2 hours

## Acceptance Criteria

- [ ] Submit button renders at bottom of registration form
- [ ] Button uses primary button styling from design system
- [ ] Button is disabled when form is invalid
- [ ] Button shows loading state during submission
- [ ] Form validation runs before submission
- [ ] Success/error feedback displayed after submission

## Implementation Notes

- Use existing `<Button>` component from `components/ui/button.tsx`
- Hook up form validation using `react-hook-form`
- Display loading spinner during async submission
- Show toast notification on success/error

## Testing Strategy

- [ ] Unit test: Button renders with correct props
- [ ] Unit test: Button disabled state reflects form validity
- [ ] Integration test: Form submission flow with validation
- [ ] Visual test: Button matches design system

## Related Cards

- Blocked by: User registration form layout (#F0012)
