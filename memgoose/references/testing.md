# Testing With Memgoose

Use this reference when replacing Mongoose in tests or writing backend model/API tests.

## Primary Pattern

Mock `mongoose` before importing any code that imports mongoose:

```typescript
import { vi, beforeAll, afterAll, beforeEach } from 'vitest'
import { connect, disconnect, dropDatabase } from 'memgoose'

vi.mock('mongoose', () => import('memgoose'))

beforeAll(() => {
  connect({ storage: 'memory' })
})

afterAll(async () => {
  await disconnect()
})

beforeEach(async () => {
  await dropDatabase()
})
```

Production models can then be imported and used unchanged. The docs show this pattern for Vitest, Jest, and Express/Fastify API tests.

## API Route Tests

The docs show building the app after the mongoose mock is registered, then issuing requests through the app test interface. In the Fastify example, the app is built once in `beforeAll()` and exercised with `app.inject()`.

## Isolation

Use `dropDatabase()` in `beforeEach` to clear data between tests. Use `disconnect()` in teardown so storage resources and TTL intervals are cleaned up.

## Migration Notes

- Vitest: `vi.mock('mongoose', () => import('memgoose'))`.
- Jest: `jest.mock('mongoose', () => require('memgoose'))`.
- Mock early, before imports that depend on mongoose.
- Existing Schema/model/query patterns should work for documented APIs.
- If code uses advanced Mongoose features, verify support in the memgoose docs before relying on it.
