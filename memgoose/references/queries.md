# Queries

Use this reference for filters, update operators, query options, chaining, populate, and atomic operations.

## Find Methods

```typescript
await User.find()
await User.find({ status: 'active' })
await User.findOne({ email })
await User.findById(id)
await User.countDocuments({ age: { $gte: 18 } })
await User.distinct('city', { status: 'active' })
```

`findOne()` returns `null` when no document matches.

## Query Operators

Documented operators:

- Equality: direct value, `$eq`, `$ne`
- Arrays/lists: `$in`, `$nin`, `$all`, `$elemMatch`, `$size`
- Comparisons: `$gt`, `$gte`, `$lt`, `$lte`
- Strings: `$regex`
- Existence: `$exists`
- Logical: `$or`, `$and`, `$nor`, field-level `$not`

Example:

```typescript
await User.find({
  status: 'active',
  age: { $gte: 18, $lte: 65 },
  name: { $in: ['Alice', 'Bob'] }
})
```

## Update Operators

Documented update operators:

- `$set`
- `$unset`
- `$inc`
- `$dec`
- `$push`
- `$pull`
- `$addToSet`
- `$pop`
- `$rename`

Direct updates are treated like setting fields.

```typescript
await User.updateOne(
  { email },
  {
    $set: { status: 'active' },
    $inc: { loginCount: 1 },
    $push: { loginHistory: new Date() }
  }
)
```

## Options and Chaining

Use query options or fluent chaining:

```typescript
await User.find(
  { status: 'active' },
  {
    sort: { age: -1 },
    limit: 10,
    skip: 20,
    lean: true,
    select: ['name', 'email']
  }
)

await User.find({ status: 'active' })
  .sort('-age name')
  .limit(10)
  .skip(20)
  .select('name email')
  .lean()
  .exec()
```

`lean()` returns plain objects. Lean results do not include virtuals or `save()`.

## Populate

Use `ref` in schema fields and `.populate()` in queries:

```typescript
const postSchema = new Schema({
  authorId: { type: ObjectId, ref: 'User' }
})

await Post.findOne({ title }).populate('authorId').exec()
```

Populate also supports arrays and advanced options in the docs, including select, match, nested populate, and explicit model.

## Atomic Operations

Use documented atomic helpers when a task needs read-modify-write behavior:

```typescript
await User.findOneAndUpdate({ email }, { $inc: { loginCount: 1 } })
await User.findOneAndDelete({ email })
```
