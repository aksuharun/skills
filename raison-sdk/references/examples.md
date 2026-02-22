# Raison SDK Examples

## Table of Contents
1. [Anthropic / Claude](#anthropic--claude)
2. [OpenAI](#openai)
3. [Express.js Server](#expressjs-server)
4. [Next.js / Serverless Singleton](#nextjs--serverless-singleton)
5. [Multiple Environments](#multiple-environments)
6. [Error Handling](#error-handling)
7. [Graceful Shutdown](#graceful-shutdown)
8. [Generic LLM Pattern](#generic-llm-pattern)

---

## Anthropic / Claude

```typescript
import Anthropic from "@anthropic-ai/sdk";
import { Raison } from "raison";

const raison = new Raison({ apiKey: process.env.RAISON_API_KEY });
const anthropic = new Anthropic();

async function chat(userMessage: string, userId: string) {
  const systemPrompt = await raison.render("SYSTEM_PROMPT_ID", {
    userId,
    date: new Date().toISOString(),
    modelVersion: "claude-sonnet-4-6",
  });

  const response = await anthropic.messages.create({
    model: "claude-sonnet-4-6",
    max_tokens: 1024,
    system: systemPrompt,
    messages: [{ role: "user", content: userMessage }],
  });

  return response.content[0].type === "text" ? response.content[0].text : "";
}
```

---

## OpenAI

```typescript
import OpenAI from "openai";
import { Raison } from "raison";

const raison = new Raison({ apiKey: process.env.RAISON_API_KEY });
const openai = new OpenAI();

async function chat(userMessage: string) {
  const systemPrompt = await raison.render("SYSTEM_PROMPT_ID", {
    date: new Date().toISOString(),
  });

  const response = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: [
      { role: "system", content: systemPrompt },
      { role: "user", content: userMessage },
    ],
  });

  return response.choices[0].message.content;
}
```

---

## Express.js Server

Create the Raison instance once at startup — the connection is reused across all requests.

```typescript
import { Raison } from "raison";
import express from "express";

const raison = new Raison({ apiKey: process.env.RAISON_API_KEY });
const app = express();
app.use(express.json());

app.post("/chat", async (req, res) => {
  const { userId, message } = req.body;

  const systemPrompt = await raison.render("SYSTEM_PROMPT_ID", {
    userId,
    userRole: req.user?.role ?? "guest",
  });

  if (!systemPrompt) {
    return res.status(500).json({ error: "System prompt not found" });
  }

  // Pass systemPrompt to your LLM...
  res.json({ systemPrompt });
});

app.listen(3000);
```

---

## Next.js / Serverless Singleton

Prevent creating a new connection per request by using a module-level singleton.

```typescript
// lib/raison.ts
import { Raison } from "raison";

let instance: Raison | null = null;

export function getRaison(): Raison {
  if (!instance) {
    instance = new Raison({ apiKey: process.env.RAISON_API_KEY! });
  }
  return instance;
}
```

```typescript
// app/api/chat/route.ts (Next.js App Router)
import { getRaison } from "@/lib/raison";
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const { message } = await request.json();
  const raison = getRaison();

  const systemPrompt = await raison.render("SYSTEM_PROMPT_ID", {
    date: new Date().toISOString(),
  });

  // ... call your LLM with systemPrompt
  return NextResponse.json({ systemPrompt });
}
```

> In Next.js dev mode, HMR may create multiple instances — this is harmless. In production, the singleton ensures a single persistent connection.

---

## Multiple Environments

Each `Raison` instance is fully isolated with its own WebSocket and in-memory store.

```typescript
import { Raison } from "raison";

const dev = new Raison({ apiKey: process.env.RAISON_DEV_KEY! });
const prod = new Raison({ apiKey: process.env.RAISON_PROD_KEY! });

// Useful for canary deployments or A/B testing
async function getPrompt(environment: "dev" | "prod", promptId: string) {
  const client = environment === "dev" ? dev : prod;
  return client.render(promptId, { env: environment });
}
```

---

## Error Handling

Guard against missing prompts (render returns `""` for not-found):

```typescript
const FALLBACK_PROMPT = "You are a helpful assistant.";

async function getSystemPrompt(variables: Record<string, unknown>) {
  try {
    const prompt = await raison.render("SYSTEM_PROMPT_ID", variables);

    if (!prompt) {
      console.warn("System prompt not found, using fallback");
      return FALLBACK_PROMPT;
    }

    return prompt;
  } catch (error) {
    console.error("Failed to render prompt:", error);
    return FALLBACK_PROMPT;
  }
}
```

---

## Graceful Shutdown

```typescript
import { Raison } from "raison";

const raison = new Raison({ apiKey: process.env.RAISON_API_KEY! });

process.on("SIGTERM", () => {
  raison.disconnect();
  process.exit(0);
});

process.on("SIGINT", () => {
  raison.disconnect();
  process.exit(0);
});
```

---

## Generic LLM Pattern

A reusable wrapper that works with any LLM:

```typescript
import { Raison } from "raison";

const raison = new Raison({ apiKey: process.env.RAISON_API_KEY! });

interface ChatOptions {
  promptId: string;
  variables?: Record<string, unknown>;
  userMessage: string;
}

async function buildSystemPrompt(options: ChatOptions): Promise<string> {
  const rendered = await raison.render(options.promptId, options.variables);
  return rendered || "You are a helpful assistant.";
}

// Usage with any LLM SDK:
const system = await buildSystemPrompt({
  promptId: "SYSTEM_PROMPT_ID",
  variables: { userName: "Alice", date: new Date().toISOString() },
  userMessage: "How do I reset my password?",
});
// → pass `system` to your LLM's system message parameter
```
