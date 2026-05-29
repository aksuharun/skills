# Schemas

Use this reference for schema definitions, validation, indexes, timestamps, subdocuments, methods, and statics.

## Field Definitions

Supported field styles:

```typescript
const schema = new Schema({
  name: String,
  age: Number,
  active: Boolean,
  createdAt: Date,
  tags: [String],
  metadata: Object,
  authorId: { type: ObjectId },
  email: { type: String, required: true, unique: true }
})
```

Subdocuments use nested schemas:

```typescript
const addressSchema = new Schema({
  street: String,
  city: String,
  zipCode: { type: String, match: /^\d{5}$/ }
})

const userSchema = new Schema({
  address: addressSchema,
  addresses: [addressSchema]
})
```

## Field Options

Documented options include:

- `type`
- `required`
- `default`
- `min`, `max`
- `minLength`, `maxLength`
- `enum`
- `match`
- `validate`
- `ref`
- `get`, `set`
- `unique`

Validation errors are thrown as `ValidationError`. Unique indexes enforce uniqueness.

## Defaults, Getters, and Setters

Defaults can be static values or functions. Setters transform data before storage; getters transform data when reading.

```typescript
const schema = new Schema({
  status: { type: String, default: 'active' },
  createdAt: { type: Date, default: () => new Date() },
  email: {
    type: String,
    set: value => value.toLowerCase().trim()
  }
})
```

## Indexes

Use indexes for frequently queried fields:

```typescript
schema.index('email')
schema.index(['city', 'age'])
schema.index({ author: 1, year: -1 })
schema.index('username', { unique: true })
schema.index('createdAt', { ttl: 1800 })
```

`unique: true` on a field also creates a unique index. TTL indexes are single-field indexes and clean up expired documents in the background.

## Timestamps

```typescript
const schema = new Schema(
  { name: String },
  { timestamps: true }
)
```

Custom timestamp names:

```typescript
new Schema(
  { name: String },
  { timestamps: { createdAt: 'created_at', updatedAt: 'updated_at' } }
)
```

## Methods, Statics, and Classes

```typescript
schema.methods.getFullName = function () {
  return `${this.firstName} ${this.lastName}`
}

schema.statics.findByEmail = function (email: string) {
  return this.findOne({ email })
}
```

`schema.loadClass(ClassName)` loads instance methods, static methods, and getters/setters as virtuals.

## Advanced Schema Features

Virtuals are computed properties and are not stored. Hooks support pre/post `save`, `delete`, `update`, `find`, and `findOne`. Discriminators use `discriminatorKey` in schema options.
