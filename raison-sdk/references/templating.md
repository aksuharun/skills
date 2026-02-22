# Raison Templating (Handlebars)

Raison prompts use [Handlebars](https://handlebarsjs.com/) syntax. Key note: the SDK compiles with `noEscape: true` — use `{{value}}` everywhere, triple braces are not needed.

## Table of Contents
1. [Variables](#variables)
2. [Nested Objects](#nested-objects)
3. [Conditionals](#conditionals)
4. [Loops](#loops)
5. [Escaping Braces](#escaping-braces)
6. [Custom Helpers](#custom-helpers)
7. [Template Errors](#template-errors)
8. [Full Example](#full-example)

---

## Variables

```handlebars
Hello, {{userName}}!
You are assisting with a {{context}} inquiry.
```

```typescript
await raison.render("PROMPT_ID", { userName: "Alice", context: "billing" });
// → "Hello, Alice!\nYou are assisting with a billing inquiry."
```

Missing variables render as `""` — no error thrown.

---

## Nested Objects

```handlebars
User: {{user.name}}
Email: {{user.email}}
Account tier: {{user.account.tier}}
```

```typescript
await raison.render("PROMPT_ID", {
  user: { name: "Alice", email: "alice@example.com", account: { tier: "pro" } },
});
```

---

## Conditionals

```handlebars
{{#if isPremium}}
You have access to all premium features.
{{else}}
Upgrade your plan to unlock premium features.
{{/if}}
```

Falsy values (`false`, `null`, `undefined`, `0`, `""`, `[]`) take the `{{else}}` branch. `{{else}}` is optional.

---

## Loops

**Array of strings:**

```handlebars
Available tools:
{{#each tools}}
- {{this}}
{{/each}}
```

```typescript
await raison.render("PROMPT_ID", { tools: ["search", "calculator", "calendar"] });
```

**With index (`@index` is zero-based):**

```handlebars
{{#each steps}}
Step {{@index}}: {{this}}
{{/each}}
```

**Array of objects:**

```handlebars
{{#each messages}}
{{role}}: {{content}}
{{/each}}
```

```typescript
await raison.render("PROMPT_ID", {
  messages: [
    { role: "user", content: "Hello" },
    { role: "assistant", content: "Hi there!" },
  ],
});
```

---

## Escaping Braces

To output a literal `{{` without it being treated as a variable:

```handlebars
Use the syntax \{{variableName}} to insert a variable.
```

Renders as: `Use the syntax {{variableName}} to insert a variable.`

---

## Custom Helpers

Register once at application startup before creating Raison instances:

```typescript
import { Raison } from "raison";

// Date formatting
Raison.registerHelper("formatDate", (date: string | Date) =>
  new Date(date).toLocaleDateString("en-US", { year: "numeric", month: "long", day: "numeric" })
);

// JSON embedding (for structured context)
Raison.registerHelper("json", (obj: unknown) => JSON.stringify(obj, null, 2));

// Case transforms
Raison.registerHelper("uppercase", (str: string) => str.toUpperCase());
Raison.registerHelper("lowercase", (str: string) => str.toLowerCase());

// Truncation
Raison.registerHelper("truncate", (str: string, length: number) =>
  str.length <= length ? str : str.slice(0, length) + "..."
);

// Conditional (block helper)
Raison.registerHelper("ifEqual", function (a, b, options) {
  return a === b ? options.fn(this) : options.inverse(this);
});
```

**Using helpers in templates:**

```handlebars
User: {{uppercase userName}}
Context: {{json metadata}}
Today is {{formatDate currentDate}}.
Summary: {{truncate description 200}}

{{#ifEqual plan "enterprise"}}
You have enterprise-level access.
{{else}}
Contact sales to upgrade.
{{/ifEqual}}
```

---

## Template Errors

If a template is syntactically invalid (e.g., unclosed block), `render` catches the error and returns the raw template content as a safe fallback — the application keeps working.

To detect compilation failures, compare the returned string against `findOne` raw content.

---

## Full Example

```handlebars
You are a {{role}} assistant for {{uppercase companyName}}.

{{#if userContext}}
User information:
{{json userContext}}
{{/if}}

{{#if conversationHistory.length}}
Previous conversation:
{{#each conversationHistory}}
{{role}}: {{content}}
{{/each}}
{{/if}}

Today is {{formatDate currentDate}}.
```

```typescript
const prompt = await raison.render("PROMPT_ID", {
  role: "customer support",
  companyName: "acme corp",
  userContext: { plan: "pro", joinedDate: "2024-01-15" },
  conversationHistory: [
    { role: "user", content: "I need help with my invoice" },
  ],
  currentDate: new Date().toISOString(),
});
```
