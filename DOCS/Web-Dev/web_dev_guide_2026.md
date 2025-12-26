# **Ultimate Web Development Master Guide (2026 Edition)**

**Target Audience:** Senior Engineers & Full-Stack Developers  
**Philosophy:** Native Browser Capabilities > External Libraries. Performance by default. Security by design.

---

## **Table of Contents**
1. [Modern CSS](#1-modern-css)
2. [JavaScript Fundamentals](#2-javascript-fundamentals)
3. [TypeScript Configuration](#3-typescript-configuration)
4. [Framework Standards](#4-framework-standards)
5. [State Management](#5-state-management)
6. [Runtimes & Backends](#6-runtimes--backends)
7. [Data Layer](#7-data-layer)
8. [Security Fundamentals](#8-security-fundamentals)
9. [Testing Strategy](#9-testing-strategy)
10. [HTML5 & Media](#10-html5--media)
11. [Build Tools](#11-build-tools)
12. [Observability](#12-observability)
13. [Bleeding Edge APIs](#13-bleeding-edge-apis)
14. [Migration Guide](#14-migration-guide)
15. [Stack Decision Matrix](#15-stack-decision-matrix)

---

## **1. Modern CSS**

### **Fluid Typography**
```css
:root {
  --fluid-type: clamp(1rem, 0.5rem + 2vw, 2rem);
  --fluid-pad: max(1rem, 5vw);
}
h1 { font-size: var(--fluid-type); }
```

### **Container Queries**
```css
.card { container-type: inline-size; }
@container (max-width: 400px) {
  .card-layout { flex-direction: column; }
}
```

### **Parent Selector (`:has`)**
```css
.card:has(img) { display: grid; }
form:has(input:invalid) { border: 1px solid red; }
```

### **CSS Layers**
```css
@layer reset, base, components, utilities;
@layer utilities { .text-center { text-align: center; } }
```

### **Logical Properties**
```css
margin-inline: auto; /* LTR/RTL compatible */
padding-block: 2rem;
```

---

## **2. JavaScript Fundamentals**

### **Intersection Observer**
```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(e => e.target.classList.toggle('visible', e.isIntersecting));
}, { rootMargin: '100px', threshold: 0.1 });

document.querySelectorAll('.animate').forEach(el => observer.observe(el));
```

### **Debounce & Throttle**
```js
const debounce = (fn, delay) => {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
};

const throttle = (fn, limit) => {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
};
```

### **Structured Clone**
```js
const copy = structuredClone(original); // Deep copy with Date, Map, Set support
```

### **AbortController**
```js
const controller = new AbortController();
fetch('/api', { signal: controller.signal });
controller.abort(); // Cancel request
```

---

## **3. TypeScript Configuration**

### **Modern tsconfig.json**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "verbatimModuleSyntax": true,
    "paths": { "@/*": ["./src/*"] }
  }
}
```

### **Type Patterns**
```typescript
// Branded types (prevent primitive confusion)
type UserId = string & { readonly __brand: unique symbol };

// Discriminated unions (type-safe state)
type Result<T> = 
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// Template literals (type-safe strings)
type Endpoint = `/api/${string}`;
```

---

## **4. Framework Standards**

### **React 19: Server Components + Actions**
```tsx
// Server Component
async function Page() {
  const data = await fetch('/api/data').then(r => r.json());
  return <Client dataPromise={data} />;
}

// Server Action
'use server';
export async function saveData(formData: FormData) {
  const email = formData.get('email');
  await db.update({ email });
  return { success: true };
}

// Client usage
<form action={saveData}>
  <input name="email" required />
  <button>Save</button>
</form>
```

### **Svelte 5: Runes**
```svelte
<script>
  let count = $state(0);
  let double = $derived(count * 2);
  
  $effect(() => {
    console.log(`Count: ${count}`);
    return () => console.log('Cleanup');
  });
</script>

<button onclick={() => count++}>{count}</button>
```

### **Vue 3.5: Composition API**
```vue
<script setup>
const model = defineModel();
const count = ref(0);
const double = computed(() => count.value * 2);
</script>

<template>
  <input v-model="model" />
  <button @click="count++">{{ count }}</button>
</template>
```

---

## **5. State Management**

### **React: Context + `use` Hook**
```tsx
const StoreContext = createContext<Store | null>(null);

export function useStore() {
  return use(StoreContext);
}
```

### **Svelte 5: Runes (No Stores)**
```ts
export const store = (() => {
  let user = $state(null);
  return {
    get user() { return user; },
    setUser(u) { user = u; }
  };
})();
```

### **Zustand (Cross-Framework)**
```ts
import { create } from 'zustand';

export const useStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```

---

## **6. Runtimes & Backends**

### **Node.js v24 (Enterprise)**
```js
// Native test runner
import { test } from 'node:test';
import assert from 'node:assert/strict';

test('returns 200', async () => {
  const res = await fetch('http://localhost:3000');
  assert.equal(res.status, 200);
});
```

### **Bun v1.2+ (Speed)**
```ts
// All-in-one server
Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response("Hello!");
  },
});

// Shell scripting
import { $ } from 'bun';
await $`rm -rf dist && mkdir dist`;
```

### **Rust + Axum (Infrastructure)**
```rust
use axum::{routing::get, Json, Router};

#[tokio::main]
async fn main() {
    let app = Router::new().route("/", get(|| async { "Hello" }));
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

---

## **7. Data Layer**

### **Drizzle ORM**
```ts
import { pgTable, serial, text } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull(),
});

// Query (fully typed)
const result = await db.select().from(users).where(eq(users.id, 1));

// Migrations
// bun drizzle-kit generate:pg
// bun drizzle-kit push:pg
```

### **Zod Validation**
```ts
import { z } from 'zod';

const UserSchema = z.object({
  email: z.string().email(),
  age: z.number().positive().optional(),
});

type User = z.infer<typeof UserSchema>;

const result = UserSchema.safeParse(data);
if (!result.success) return result.error;
```

---

## **8. Security Fundamentals**

### **Content Security Policy**
```ts
// Next.js middleware
response.headers.set('Content-Security-Policy', `
  default-src 'self';
  script-src 'self' 'strict-dynamic';
  style-src 'self' 'unsafe-inline';
`);
response.headers.set('X-Frame-Options', 'DENY');
```

### **CSRF Protection**
```ts
'use server';
export async function action(formData: FormData) {
  const token = formData.get('csrf-token');
  const sessionToken = cookies().get('csrf-token')?.value;
  
  if (token !== sessionToken) return { error: 'Invalid' };
  // Proceed safely
}
```

### **Input Sanitization**
```ts
import DOMPurify from 'isomorphic-dompurify';

const clean = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ['b', 'i', 'a', 'p'],
});
```

### **Rate Limiting**
```ts
import { Ratelimit } from '@upstash/ratelimit';

const limit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10s'),
});

const { success } = await limit.limit(ip);
if (!success) return new Response('Too many requests', { status: 429 });
```

---

## **9. Testing Strategy**

### **Unit: Node Native**
```ts
import { test } from 'node:test';
import assert from 'node:assert/strict';

test('sum function', () => {
  assert.equal(sum(1, 2), 3);
});
```

### **Component: Testing Library**
```tsx
import { render, screen } from '@testing-library/react';

test('button renders', () => {
  render(<Button>Click</Button>);
  expect(screen.getByText('Click')).toBeInTheDocument();
});
```

### **E2E: Playwright**
```ts
import { test, expect } from '@playwright/test';

test('homepage loads', async ({ page }) => {
  await page.goto('/');
  await expect(page.getByRole('heading')).toBeVisible();
});
```

### **Visual Regression: Chromatic**
```bash
npm install --save-dev chromatic
npx chromatic --project-token=<token>
```

---

## **10. HTML5 & Media**

### **Responsive Images**
```html
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Description" loading="lazy">
</picture>
```

### **Aspect Ratio (No Padding Hacks)**
```css
img, video { 
  width: 100%; 
  aspect-ratio: 16 / 9; 
  object-fit: cover; 
}
```

### **Accessibility**
```css
:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
}
```

---

## **11. Build Tools**

### **Vite v6**
```js
// vite.config.js
export default {
  server: { proxy: { '/api': 'http://localhost:3000' } },
  build: {
    rollupOptions: {
      output: { manualChunks: { vendor: ['react'] } }
    }
  }
};
```

### **Bundle Analysis**
```bash
bun build ./src/index.ts --outdir ./dist --minify
# Target: <50kb for initial bundle
```

---

## **12. Observability**

### **OpenTelemetry**
```ts
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('app');
const span = tracer.startSpan('fetchUser');
// ... operation
span.end();
```

### **Structured Logging**
```ts
import pino from 'pino';

const logger = pino();
logger.info({ userId: 123, action: 'login' }, 'User logged in');
```

### **Error Tracking: Sentry**
```ts
import * as Sentry from '@sentry/react';

Sentry.init({ dsn: process.env.SENTRY_DSN });
```

---

## **13. Bleeding Edge APIs**

### **View Transitions**
```css
@view-transition {
  navigation: auto;
}
```

### **Speculation Rules (Prefetch)**
```html
<script type="speculationrules">
{
  "prefetch": [{ "source": "list", "urls": ["/about", "/contact"] }]
}
</script>
```

### **Popover API**
```html
<button popovertarget="menu">Open</button>
<div id="menu" popover>Menu content</div>
```

---

## **14. Migration Guide**

### **From Webpack → Vite**
1. Replace `webpack.config.js` with `vite.config.js`
2. Change imports: `require()` → `import`
3. Update scripts: `webpack serve` → `vite`

### **From useEffect Fetching → React 19**
```tsx
// Old (Client-side)
useEffect(() => {
  fetch('/api').then(setData);
}, []);

// New (Server Component)
async function Page() {
  const data = await fetch('/api').then(r => r.json());
  return <Client data={data} />;
}
```

### **From Redux → Zustand**
```ts
// Old: 100+ lines of boilerplate
// New: 10 lines
export const useStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```

---

## **15. Stack Decision Matrix**

| Use Case | Language | Runtime | Framework | Database |
|:---------|:---------|:--------|:----------|:---------|
| **Enterprise SaaS** | TypeScript | Node.js | React 19 | Postgres + Drizzle |
| **Startup/MVP** | TypeScript | Bun | Svelte 5 | SQLite + Drizzle |
| **Microservice** | Rust | Rust | Axum | Redis/Postgres |
| **Internal Tool** | JavaScript | Bun | HTMX | SQLite |
| **Desktop App** | Rust/TS | Tauri v2 | Svelte | SQLite |

---

## **Key Principles**

1. **Browser-first**: Use native APIs before libraries
2. **Type safety**: TypeScript + Zod + Drizzle = runtime + compile-time safety
3. **Performance**: Bundle size <50kb, Time to Interactive <2s
4. **Security**: CSP, CSRF, rate limiting, input sanitization
5. **Testing**: Unit (Node) + Component (Testing Library) + E2E (Playwright)
6. **Observability**: Structured logging, tracing, error tracking

---

**Bundle Size Targets (2026):**
- Initial JS: <50kb gzipped
- Total Page Weight: <200kb
- Time to Interactive: <2s on 3G

**Framework Decision Tree:**
- Choose React for: Enterprise, large teams, job market
- Choose Svelte for: Greenfield, performance-critical, small teams
- Choose Vue for: PHP/Laravel ecosystem, gradual adoption

**Runtime Decision Tree:**
- Choose Node for: Enterprise, stability, ecosystem compatibility
- Choose Bun for: Startups, local dev, tooling, shell scripts
- Choose Rust for: Infrastructure, performance-critical services, WebAssembly