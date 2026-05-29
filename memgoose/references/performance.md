# Performance

Use this reference for memgoose optimization decisions.

## Indexes

Indexes are the main documented performance tool. Without indexes, queries scan documents. With indexes, equality lookups on indexed fields are much faster.

```typescript
schema.index('email')
schema.index(['city', 'age'])
schema.index(['status', 'createdAt'])
```

Use indexes for:

- frequently queried fields
- equality and range filters
- common sort fields
- unique constraints
- common multi-field filters via compound indexes

Avoid unnecessary indexes for rarely queried fields or very small datasets.

## Query Shape

Prefer selective indexed filters early. Avoid regex on large datasets when an equality query can work. Use `limit` for large result sets and select only required fields.

```typescript
await User.find(
  { status: 'active' },
  {
    limit: 20,
    select: ['name', 'email']
  }
)
```

## Lean Queries

Use `lean: true` or `.lean()` when virtuals, instance methods, and `save()` are not needed.

```typescript
const users = await User.find({ status: 'active' }).lean().exec()
```

Lean queries return plain objects and skip document features.

## Batch Operations

Prefer `insertMany()` for bulk inserts and `updateMany()` / `deleteMany()` for broad changes.

## Storage Performance

Memory is fastest and best for tests. File provides simple persistence. SQLite is documented as ACID persistent storage with native SQL query execution. WiredTiger targets high-performance embedded persistence.

Choose storage based on the task's durability and dependency constraints, not just raw speed.
