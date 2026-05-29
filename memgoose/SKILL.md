---
name: memgoose
description: Use memgoose for Mongoose-like in-memory databases, backend tests that mock mongoose, schema/model/query implementation, storage backend selection, aggregation pipelines, and performance tuning with indexes or lean queries. Trigger when working with memgoose directly, replacing mongoose in tests with memgoose, writing model tests, configuring memory/file/SQLite/WiredTiger storage, or using memgoose schemas, queries, populate, hooks, virtuals, discriminators, ObjectId, and aggregation.
---

# Memgoose

Use this skill to work efficiently with memgoose using only the documented API. memgoose is a lightweight MongoDB-like, Mongoose-style TypeScript database with memory storage by default and optional persistent storage.

## First Steps

Identify the task and load only the needed reference:

- Testing or mocking mongoose: read `references/testing.md`.
- Basic models, CRUD, and connection setup: read `references/basics.md`.
- Schema fields, validation, indexes, timestamps, methods, or statics: read `references/schemas.md`.
- Queries, updates, chaining, select, sort, pagination, lean, populate, or atomic operations: read `references/queries.md`.
- Aggregation pipelines and analytics: read `references/aggregation.md`.
- Storage backend choice or persistence setup: read `references/storage.md`.
- Performance tuning: read `references/performance.md`.

Prefer docs-backed patterns over assumptions from Mongoose. If a requested Mongoose feature is not in these references, inspect the project docs before claiming support.

## Common Workflows

### Use memgoose in Tests

Mock `mongoose` before importing app code or production models:

```typescript
import { vi } from 'vitest'
import { connect, disconnect, dropDatabase } from 'memgoose'

vi.mock('mongoose', () => import('memgoose'))

beforeAll(() => connect({ storage: 'memory' }))
afterAll(async () => disconnect())
beforeEach(async () => dropDatabase())
```

For route tests, register the mock before building the app and then exercise the app through its test interface. See `references/testing.md`.

### Build a Model

Use `Schema` and `model` for default memory storage, or create a configured database with `connect()` / `createDatabase()` when storage matters.

```typescript
import { Schema, model } from 'memgoose'

const userSchema = new Schema({
  email: { type: String, required: true, unique: true },
  age: { type: Number, min: 0 }
})

userSchema.index('email')

const User = model('User', userSchema)
```

See `references/basics.md` and `references/schemas.md`.

### Query and Update Data

Use MongoDB-like filters and update operators. Query builders are thenable and also support `.exec()`.

```typescript
const users = await User.find({ age: { $gte: 18 } })
  .sort('-age name')
  .limit(20)
  .select('email age')
  .lean()
  .exec()

await User.updateOne({ email }, { $inc: { loginCount: 1 } })
```

See `references/queries.md`.

### Choose Storage

Use memory for tests and transient data. Use file, SQLite, or WiredTiger only when persistence is required or the task explicitly targets those backends. Always disconnect persistent databases when done.

See `references/storage.md`.

## Guardrails

- Do not describe memgoose as full Mongoose or MongoDB parity; use the documented supported APIs.
- Do not add undocumented configuration keys or operators.
- Use indexes for frequently queried fields and compound indexes for common multi-field filters.
- Use lean queries only when virtuals, methods, and `save()` are unnecessary.
