# Optimize Database Queries for User Dashboard

## Description

Optimize slow database queries causing 5-10 second page load times on the user dashboard. Add indexes, reduce N+1 queries, and implement query result caching.

**Type**: Performance
**Impact**: High (affects all users)
**Estimated Time**: 3 days

## Performance Issues

### Current Metrics:
- Dashboard load time: 8.3s (p95)
- Database queries: 127 queries per page load
- N+1 query pattern: Users → Posts → Comments
- Missing indexes on frequently queried columns

### Slow Query Analysis:

**Query 1: Fetch user posts** (2.3s)
```sql
SELECT * FROM posts WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 50;
```
❌ No index on `user_id`
❌ No index on `created_at`

**Query 2: Fetch post comments** (3.8s, runs 50 times)
```sql
SELECT * FROM comments WHERE post_id = ?;
```
❌ N+1 problem (50 separate queries)
❌ No index on `post_id`

**Query 3: Count user followers** (1.2s)
```sql
SELECT COUNT(*) FROM followers WHERE following_id = ?;
```
❌ Full table scan
❌ No index on `following_id`

## Optimization Plan

### Phase 1: Add Database Indexes (Day 1)

```sql
-- Add indexes for common queries
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_followers_following_id ON followers(following_id);

-- Composite index for user posts with date filter
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at DESC);
```

**Expected Impact:** 60% reduction in query time

### Phase 2: Fix N+1 Queries (Day 2)

**Before (N+1 problem):**
```typescript
const posts = await Post.findAll({ where: { userId } });
for (const post of posts) {
  post.comments = await Comment.findAll({ where: { postId: post.id } });
}
```

**After (Single query with JOIN):**
```typescript
const posts = await Post.findAll({
  where: { userId },
  include: [{ model: Comment, as: 'comments' }]
});
```

**Expected Impact:** 50 queries → 1 query

### Phase 3: Implement Query Caching (Day 3)

**Cache Strategy:**
- User posts: Cache for 5 minutes
- Follower counts: Cache for 10 minutes
- User profile: Cache for 15 minutes

```typescript
// Before: No caching
const posts = await Post.findAll({ where: { userId } });

// After: Redis caching
const cacheKey = `user:${userId}:posts`;
let posts = await redis.get(cacheKey);
if (!posts) {
  posts = await Post.findAll({ where: { userId } });
  await redis.setex(cacheKey, 300, JSON.stringify(posts));
}
```

**Expected Impact:** 70% cache hit rate = 70% faster for repeat loads

## Implementation Steps

1. **Analyze current queries:**
   - Run EXPLAIN on slow queries
   - Identify missing indexes
   - Document N+1 patterns

2. **Create database migrations:**
   - Write migration for new indexes
   - Test on staging database
   - Verify query plans improve

3. **Refactor code for eager loading:**
   - Replace lazy loading with eager loading
   - Use JOIN instead of separate queries
   - Add Sequelize/Prisma `include` options

4. **Implement caching layer:**
   - Set up Redis client
   - Add cache-aside pattern
   - Implement cache invalidation

5. **Test and measure:**
   - Load test dashboard (100 concurrent users)
   - Measure query counts and latency
   - Verify cache hit rates

## Performance Targets

### Before Optimization:
- Dashboard load: 8.3s (p95)
- Query count: 127 queries
- Database CPU: 85%

### After Optimization:
- Dashboard load: < 1.5s (p95) ✅
- Query count: < 10 queries ✅
- Database CPU: < 40% ✅
- Cache hit rate: > 70% ✅

## Testing Plan

- [ ] Benchmark queries before/after indexes
- [ ] Verify N+1 queries eliminated
- [ ] Test cache invalidation works
- [ ] Load test with 100 concurrent users
- [ ] Verify no regressions in functionality
- [ ] Monitor cache memory usage

## Monitoring

**Metrics to Track:**
- Query execution time (per query)
- Total dashboard load time
- Cache hit/miss ratio
- Database connection pool usage
- Redis memory usage

**Alerts:**
- Query time > 1 second
- Cache hit rate < 60%
- Database CPU > 70%

## Rollback Plan

If performance degrades:
1. Revert code changes (remove eager loading)
2. Keep database indexes (safe to keep)
3. Disable Redis caching
4. Investigate issue in separate branch

## Success Criteria

- [ ] Dashboard loads in < 1.5s (p95)
- [ ] Query count reduced by > 90%
- [ ] Cache hit rate > 70%
- [ ] No functionality regressions
- [ ] Database CPU usage reduced
- [ ] User satisfaction improved
