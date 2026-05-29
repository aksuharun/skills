# Aggregation

Use this reference for analytics and transformation pipelines.

## Basic Shape

```typescript
const results = await Model.aggregate([
  { $match: { status: 'active' } },
  { $group: { _id: '$category', count: { $sum: 1 } } },
  { $sort: { count: -1 } }
])
```

Place `$match` early when possible to reduce the number of documents later stages process.

## Pipeline Stages

Documented stages include:

- `$match`
- `$group`
- `$project`
- `$sort`
- `$limit`
- `$skip`
- `$count`
- `$unwind`
- `$lookup`
- `$addFields`
- `$replaceRoot`
- `$sample`
- `$bucket`
- `$bucketAuto`
- `$facet`
- `$out`
- `$merge`

Use `$out` carefully because it replaces the target collection. Use `$merge` for incremental writes.

## Accumulators and Expressions

Documented accumulators include `$sum`, `$avg`, `$min`, `$max`, `$first`, `$last`, `$push`, and `$addToSet`.

The docs also cover date, string, array, type conversion, conditional, and object expression operators. Inspect `docs/AGGREGATION.md` before using a less common expression.

## Common Patterns

Grouped analytics:

```typescript
await Sale.aggregate([
  { $match: { inStock: true } },
  {
    $group: {
      _id: '$category',
      count: { $sum: 1 },
      avgPrice: { $avg: '$price' }
    }
  }
])
```

Join related collections:

```typescript
await Book.aggregate([
  {
    $lookup: {
      from: 'Author',
      localField: 'authorId',
      foreignField: '_id',
      as: 'author'
    }
  },
  { $unwind: '$author' }
])
```

Faceted results:

```typescript
await Product.aggregate([
  { $match: { category: 'Electronics' } },
  {
    $facet: {
      topBrands: [
        { $group: { _id: '$brand', count: { $sum: 1 } } },
        { $sort: { count: -1 } },
        { $limit: 5 }
      ]
    }
  }
])
```
