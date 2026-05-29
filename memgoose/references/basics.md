# Basics

Use this reference for installation, connection setup, model creation, and CRUD.

## Install

```bash
npm install memgoose
```

Optional persistence dependencies:

```bash
npm install better-sqlite3
npm install memgoose-wiredtiger
```

## First Model

```typescript
import { Schema, model } from 'memgoose'

interface User {
  name: string
  email: string
  age: number
}

const userSchema = new Schema<User>({
  name: String,
  email: String,
  age: Number
})

const User = model('User', userSchema)
```

`model()` uses the default database. If `connect()` was not called, memory storage is used.

## CRUD

```typescript
await User.create({ name: 'Alice', email: 'alice@example.com', age: 25 })
await User.insertMany([{ name: 'Bob', email: 'bob@example.com', age: 30 }])

await User.find()
await User.find({ age: { $gte: 18 } })
await User.findOne({ email: 'alice@example.com' })
await User.findById(id)

await User.updateOne({ email }, { $set: { age: 26 } })
await User.updateMany({ age: { $lt: 18 } }, { $set: { status: 'minor' } })

await User.deleteOne({ email })
await User.deleteMany({ status: 'inactive' })
await User.countDocuments({ age: { $gte: 18 } })
```

## Connections

Use `connect(config)` to configure the default database:

```typescript
import { connect } from 'memgoose'

const db = connect({ storage: 'memory' })
const User = db.model('User', userSchema)
```

Use `createDatabase(config)` for separate databases with different storage choices.

Call `disconnect()` or `db.disconnect()` when persistent storage or TTL indexes are involved.
