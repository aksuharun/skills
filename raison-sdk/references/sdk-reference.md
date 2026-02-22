# Raison SDK Reference

## Table of Contents
1. [Constructor](#constructor)
2. [Instance Methods](#instance-methods)
3. [Static Methods](#static-methods)
4. [Types](#types)
5. [Error Behavior](#error-behavior)

---

## Constructor

### `new Raison(config)`

Opens a WebSocket connection, initializes an in-memory prompt database, and begins real-time sync.

| Parameter       | Type     | Required | Default                  | Description                                           |
|-----------------|----------|----------|--------------------------|-------------------------------------------------------|
| `config.apiKey` | `string` | Yes      | —                        | Must be non-empty and start with `rsn_`.              |
| `config.baseUrl`| `string` | No       | `https://api.raison.ist` | Override for self-hosted instances. Trailing slashes stripped. |

**Throws:**
- `"API key is required"` — if `apiKey` is empty/falsy
- `"Invalid API key format"` — if `apiKey` does not start with `rsn_`

```typescript
const raison = new Raison({ apiKey: process.env.RAISON_API_KEY });

// Self-hosted
const raison = new Raison({
  apiKey: "rsn_production_xyz",
  baseUrl: "https://api.your-domain.com",
});
```

---

## Instance Methods

### `render(promptId, variables?)`

Renders a Handlebars prompt template with optional variables.

**Signature:** `render(promptId: string, variables?: Record<string, unknown>): Promise<string>`

| Condition                      | Returns                              |
|-------------------------------|--------------------------------------|
| Prompt found, variables given | Compiled Handlebars output           |
| Prompt found, no variables    | Raw template content (not compiled)  |
| Prompt not found              | `""` (empty string)                  |
| Prompt has empty content      | `""` (empty string)                  |
| Template compilation fails    | Raw template content (safe fallback) |

- Awaits initial sync automatically — safe to call immediately after construction.
- `noEscape: true` — HTML characters are never escaped.
- Missing variables in the template render as `""`, not errors.

```typescript
const result = await raison.render("PROMPT_ID", { name: "Alice" });

// Raw template (no variables passed)
const raw = await raison.render("PROMPT_ID");

// Non-existent prompt — returns ""
const empty = await raison.render("nonexistent-id", { name: "Alice" });
```

---

### `find(params?)`

Queries the in-memory database for all prompts matching the given parameters.

**Signature:** `find(params?: Partial<Prompt>): Promise<Prompt[]>`

- No params → returns all prompts deployed to this environment.
- Always returns an array (`[]` if nothing matches, never `null`).
- Awaits initial sync before executing.

```typescript
const all = await raison.find();
const agentPrompts = await raison.find({ agentId: "AGENT_ID" });
const named = await raison.find({ name: "system-prompt" });
const v2 = await raison.find({ version: 2 });
const specific = await raison.find({ agentId: "AGENT_ID", version: 1 });
```

---

### `findOne(params)`

Finds the first prompt matching the given parameters.

**Signature:** `findOne(params: Partial<Prompt>): Promise<Prompt | null>`

- Returns the matching prompt or `null` if not found.
- `{ agentId, name }` uses the composite index — most efficient query pattern.
- Awaits initial sync before executing.

```typescript
const p = await raison.findOne({ id: "PROMPT_ID" });
const p = await raison.findOne({ agentId: "AGENT_ID", name: "system-prompt" });
const missing = await raison.findOne({ name: "nonexistent" }); // null
```

---

### `disconnect()`

Closes the WebSocket connection. Call on process shutdown.

**Signature:** `disconnect(): void`

- Stops real-time updates.
- In-memory database is preserved — existing data is still readable.

```typescript
process.on("SIGTERM", () => {
  raison.disconnect();
  process.exit(0);
});
```

---

## Static Methods

### `Raison.registerHelper(name, fn)`

Registers a global Handlebars helper available in all prompt templates.

**Signature:** `registerHelper(name: string, fn: Handlebars.HelperDelegate): void`

- Helpers are process-global — register once at startup, not per-request.
- Must be registered before `render` is called with a template that uses the helper.

```typescript
Raison.registerHelper("uppercase", (str: string) => str.toUpperCase());
Raison.registerHelper("json", (obj: unknown) => JSON.stringify(obj, null, 2));
```

---

## Types

```typescript
interface RaisonConfig {
  apiKey: string;
  baseUrl?: string;
}

interface Prompt {
  id: string;       // Stable unique identifier
  name: string;     // Human-readable name, unique within an agent
  agentId: string;  // ID of the parent agent
  version: number;  // Version number (1, 2, 3, ...)
  content: string;  // Handlebars template content
}
```

**Exports:**

```typescript
import { Raison } from "raison";             // Main class
import type { RaisonConfig } from "raison";  // Constructor config type
import type { Prompt } from "raison";        // Prompt data type
```

---

## Error Behavior

| Scenario               | Method      | Result                                  |
|------------------------|-------------|-----------------------------------------|
| API key missing        | constructor | throws `"API key is required"`          |
| API key wrong format   | constructor | throws `"Invalid API key format"`       |
| Prompt not found       | `render`    | returns `""`                            |
| Prompt has no content  | `render`    | returns `""`                            |
| Template syntax error  | `render`    | returns raw content (safe fallback)     |
| No match               | `find`      | returns `[]`                            |
| No match               | `findOne`   | returns `null`                          |
| WebSocket disconnect   | all methods | auto-reconnects, readyPromise preserved |
