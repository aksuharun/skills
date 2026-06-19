# Branch Naming

## Core Rule

Use short, descriptive, kebab-case branch names.

Prefer a flat name:

```text
emoji-reactions
```

over a namespaced one:

```text
feature/emoji-reactions
```

If the branch type matters, fold it into the slug instead of adding a
directory-like prefix:

```text
fix-disabled-like-button
```

over:

```text
fix/disabled-like-button
```

## Why

- A descriptive slug usually explains the work well enough on its own.
- Avoiding slash namespaces removes ceremony that often adds little value.

## Rules

1. Use lowercase letters, numbers, and hyphens only.
2. Describe the outcome or problem, not the implementation detail.
3. Keep names specific enough to distinguish parallel work.
4. Prefer one clear phrase over stacked abbreviations.
5. Add a leading qualifier only when it adds real meaning.

## Good Examples

- `emoji-reactions`
- `fix-disabled-like-button`
- `checkout-tax-rounding`
- `retry-webhook-delivery`
- `audit-log-export`
- `remove-legacy-oauth1`
- `csv-import-validation`
- `search-result-pagination`

## Weak Examples

- `feature/emoji-reactions`
- `fix/disabled-like-button`
- `new-stuff`
- `misc-cleanup`
- `button-update`
- `tmp-branch`

## When a Qualifier Helps

Most branches do not need a qualifier. Add one only when it improves
clarity:

- `fix-checkout-tax-rounding`
- `spike-cache-invalidation`
- `release-2026-06`

The qualifier should be optional context, not a required naming ritual.
