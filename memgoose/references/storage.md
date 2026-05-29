# Storage

Use this reference when choosing or configuring a memgoose storage backend.

## Backend Choice

- Memory: default, fastest, no persistence. Best for tests, caches, temporary data, and local development without setup.
- File: NDJSON plus write-ahead log. Human-readable, dependency-free persistence.
- SQLite: ACID persistence with native SQL query execution. Requires `better-sqlite3`.
- WiredTiger: high-performance embedded storage. Requires `memgoose-wiredtiger` and build support.

All storage backends use the same model/query API, so application code can usually stay unchanged.

## Memory

```typescript
const db = connect({ storage: 'memory' })
```

Memory is also the default if no storage config is provided.

## File

```typescript
const db = connect({
  storage: 'file',
  file: {
    dataPath: './data',
    persistMode: 'debounced',
    debounceMs: 1000
  }
})
```

Use file storage for simple persistent apps, prototypes, and human-readable storage.

## SQLite

```typescript
const db = connect({
  storage: 'sqlite',
  sqlite: {
    dataPath: './data'
  }
})
```

Install `better-sqlite3` before using SQLite storage. SQLite creates database files per model and is the documented production-ready persistent option.

## WiredTiger

```typescript
const db = connect({
  storage: 'wiredtiger',
  wiredtiger: {
    dataPath: './data/wiredtiger',
    cacheSize: '500M'
  }
})
```

Install `memgoose-wiredtiger` and ensure build tooling is available. See the project docs before changing compressor or cache settings.

## Multiple Databases

Use `createDatabase()` for independent databases:

```typescript
const cacheDb = createDatabase({ storage: 'memory' })
const appDb = createDatabase({ storage: 'sqlite', sqlite: { dataPath: './data' } })
```

Always call `disconnect()` when using persistent backends.
