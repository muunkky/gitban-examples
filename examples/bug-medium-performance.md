# Dashboard Page Loads Slowly with Large Datasets

## Description

The dashboard page takes 10-15 seconds to load when users have more than 1000 items in their workspace. CPU usage spikes to 100% during rendering.

**Type**: Bug (Performance)
**Severity**: Medium
**Impact**: Power users with large datasets
**Reported By**: 3 users via support tickets
**Discovered**: 2025-10-20

## Performance Metrics

| Dataset Size | Load Time | CPU Usage | Memory |
|--------------|-----------|-----------|--------|
| 100 items    | 1.2s      | 40%       | 120MB  |
| 500 items    | 4.5s      | 80%       | 280MB  |
| 1000 items   | 10.8s     | 100%      | 450MB  |
| 2000 items   | 25.3s     | 100%      | 890MB  |

## Steps to Reproduce

1. Create account with 1500+ todo items (use seed script)
2. Navigate to `/dashboard`
3. Observe browser freezes for 10+ seconds
4. Check browser DevTools Performance tab

## Root Cause Analysis

**Investigation Findings:**
1. ❌ Dashboard renders entire list in single render cycle (no virtualization)
2. ❌ No memoization on expensive calculations (statistics, filtering)
3. ❌ Multiple unnecessary re-renders triggered by state updates
4. ❌ Large JSON payload (2MB) loaded upfront, not paginated

**Performance Profiling:**
- 60% time in `calculateStatistics()` - runs on every render
- 30% time in React reconciliation - 1000+ DOM nodes
- 10% time in network fetch

**Problematic Code** (src/pages/Dashboard.tsx):
```tsx
function Dashboard() {
  const { items } = useItems();  // Fetches ALL items

  // Recalculates on every render
  const stats = calculateStatistics(items);
  const filtered = filterItems(items, filter);
  const sorted = sortItems(filtered, sortBy);

  return (
    <>
      <Stats data={stats} />
      {sorted.map(item => <ItemCard key={item.id} item={item} />)}
    </>
  );
}
```

## Proposed Solution

### Short-term Fixes (Week 1)
1. **Add virtualization** with `react-window`
   - Only render visible items (10-20 at a time)
   - Expected improvement: 80% reduction in initial render time

2. **Memoize expensive calculations**
   ```tsx
   const stats = useMemo(() => calculateStatistics(items), [items]);
   const filtered = useMemo(() => filterItems(items, filter), [items, filter]);
   ```

3. **Lazy load statistics**
   - Fetch stats from backend API (pre-calculated)
   - Reduce client-side computation

### Long-term Improvements (Month 1)
4. **Implement pagination**
   - Load 50 items per page
   - Infinite scroll with cursor-based pagination

5. **Add server-side filtering/sorting**
   - Move computation to backend
   - Return only requested page

6. **Optimize data structure**
   - Use Map instead of Array for O(1) lookups
   - Normalize nested data

## Implementation Plan

1. Add react-window for virtualization
2. Add useMemo for calculations
3. Refactor statistics to lazy load
4. Add performance monitoring
5. Test with 2000+ item dataset
6. Deploy to staging for user testing

## Testing Strategy

### Performance Tests
- [ ] Benchmark: Load time < 2s for 1000 items
- [ ] Benchmark: Load time < 3s for 2000 items
- [ ] CPU usage stays < 60% during render
- [ ] Memory usage < 200MB for 1000 items

### Regression Tests
- [ ] Verify all dashboard features still work
- [ ] Filtering and sorting work correctly
- [ ] Statistics calculations are accurate
- [ ] Scroll position maintained

### User Acceptance Tests
- [ ] Get feedback from 3 affected users
- [ ] Monitor performance metrics post-deploy

## Success Metrics

- Dashboard load time < 2 seconds for 1000 items (currently 10s)
- CPU usage < 60% during render (currently 100%)
- Memory usage < 200MB (currently 450MB)
- No user complaints about slowness in first month

## Related Issues

- Similar performance issue in Reports page
- Mobile app also affected (check mobile rendering)
