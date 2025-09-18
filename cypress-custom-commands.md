# 🛠️ Cypress Custom Commands — Complete Guide

Custom commands let you extend Cypress’s DSL with **reusable, readable** actions, like `cy.login()`, `cy.dataCy()`, or `cy.drag()`.

---

## Table of Contents
1. [What are Custom Commands?](#what-are-custom-commands)
2. [Where to Define Them](#where-to-define-them)
3. [Adding Commands (Parent/Child/Dual)](#adding-commands-parentchilddual)
4. [Overwriting Built-ins](#overwriting-built-ins)
5. [TypeScript Support (declaration merging)](#typescript-support-declaration-merging)
6. [Examples](#examples)
   - [data-cy selector helper](#data-cy-selector-helper)
   - [Login command](#login-command)
   - [Drag command (mouse events)](#drag-command-mouse-events)
   - [Assert toast/notification](#assert-toastnotification)
   - [Network-aware submit](#network-aware-submit)
7. [Best Practices & Pitfalls](#best-practices--pitfalls)
8. [Cheatsheet](#cheatsheet)

---

## What are Custom Commands?
A **custom command** is a function you add to the `cy` namespace so your tests read like a story:

```js
cy.login('admin', 'secret');
cy.dataCy('submit').click();
cy.drag('#item', '#dropzone');
```

They improve **reusability**, **readability**, and reduce duplication.

---

## Where to Define Them
Create commands in **`cypress/support/commands.js`** (or `.ts`), which is auto-loaded by Cypress.

**cypress/support/e2e.js** (or `e2e.ts`) imports them once per spec file:
```js
// cypress/support/e2e.js
import './commands';
```

> In Cypress v10+, the `supportFile` default is `cypress/support/e2e.(js|ts)`.

---

## Adding Commands (Parent/Child/Dual)

Cypress supports **three types of commands** via `Cypress.Commands.add`:

### 1) Parent Command
- **No subject**; starts a chain.
```js
// commands.js
Cypress.Commands.add('login', (username, password) => {
  cy.session([username, password], () => {
    cy.visit('/login');
    cy.get('input[name="username"]').type(username);
    cy.get('input[name="password"]').type(password, { log: false });
    cy.get('button[type="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```
Usage:
```js
cy.login('admin', 'secret');
```

### 2) Child Command
- Requires a **previous subject** (uses `.prevSubject: true`).  
```js
Cypress.Commands.add('dataCy', { prevSubject: 'optional' }, (subject, value) => {
  // if subject provided: search inside it, else global
  const sel = `[data-cy="${value}"]`;
  return subject ? cy.wrap(subject).find(sel) : cy.get(sel);
});
```
Usage:
```js
cy.dataCy('card');                 // parent usage
cy.get('.list').dataCy('item');    // child usage (scoped)
```

### 3) Dual Command
- Works **with or without** a previous subject (`prevSubject: 'optional'` as above).  
(Example shown in `dataCy`.)

> Other options: `'element'`, `'document'`, `'window'` to only accept those as subjects.

---

## Overwriting Built-ins
You can **monkey-patch** Cypress commands to enhance behavior.

```js
// Example: add default options to cy.visit
Cypress.Commands.overwrite('visit', (originalFn, url, options = {}) => {
  const defaults = { timeout: 20000 };
  return originalFn(url, { ...defaults, ...options });
});
```

Another example: log requests automatically when using `cy.request`:
```js
Cypress.Commands.overwrite('request', (originalFn, ...args) => {
  Cypress.log({ name: 'request', message: JSON.stringify(args[0]) });
  return originalFn(...args);
});
```

> Use overwrites sparingly—make behavior **predictable** for your team.

---

## TypeScript Support (declaration merging)

Create a `cypress/support/commands.d.ts` (or any `.d.ts` loaded by TS) and **augment** the Cypress namespace:

```ts
// cypress/support/commands.d.ts
/// <reference types="cypress" />

declare global {
  namespace Cypress {
    interface Chainable {
      /**
       * Login once and cache session.
       * @example cy.login('admin', 'secret')
       */
      login(username: string, password: string): Chainable<void>;

      /**
       * Query by [data-cy="value"] selector, optionally scoped to a subject.
       * @example cy.dataCy('submit').click()
       * @example cy.get('form').dataCy('submit')
       */
      dataCy(value: string): Chainable<JQuery<HTMLElement>>;

      /**
       * Drag one element to a target selector.
       * @example cy.drag('#item', '#dropzone')
       */
      drag(sourceSelector: string, targetSelector: string): Chainable<void>;

      /**
       * Assert a toast appears with text.
       * @example cy.expectToast(/saved/i)
       */
      expectToast(text: string | RegExp): Chainable<void>;
    }
  }
}

export {};
```

Make sure your **tsconfig** includes the support folder (e.g., `include: ["cypress", "**/*.ts"]`).

---

## Examples

### data-cy selector helper
```js
// cypress/support/commands.js
Cypress.Commands.add('dataCy', { prevSubject: 'optional' }, (subject, value) => {
  const sel = `[data-cy="${value}"]`;
  return subject ? cy.wrap(subject).find(sel) : cy.get(sel);
});
```
Usage:
```js
cy.dataCy('login-form').within(() => {
  cy.dataCy('username').type('admin');
  cy.dataCy('password').type('secret');
  cy.dataCy('submit').click();
});
```

### Login command
```js
// caches across specs (Cypress v12+): cy.session
Cypress.Commands.add('login', (username, password) => {
  cy.session([username, password], () => {
    cy.request('POST', '/api/login', { username, password }).then(({ body }) => {
      window.localStorage.setItem('token', body.token);
    });
  });
});
```
Usage:
```js
beforeEach(() => {
  cy.login(Cypress.env('USER'), Cypress.env('PASS'));
  cy.visit('/dashboard');
});
```

### Drag command (mouse events)
```js
Cypress.Commands.add('drag', (from, to) => {
  cy.get(from).trigger('mousedown', { which: 1 });
  cy.get(to).trigger('mousemove').trigger('mouseup', { force: true });
});
```
Usage:
```js
cy.drag('#item-1', '#dropzone');
```

### Assert toast/notification
```js
Cypress.Commands.add('expectToast', (text) => {
  cy.get('[role="alert"], .toast, .notification')
    .should('be.visible')
    .contains(text);
});
```
Usage:
```js
cy.expectToast(/saved successfully/i);
```

### Network-aware submit
```js
Cypress.Commands.add('submitAndWait', (alias) => {
  cy.dataCy('submit').click();
  cy.wait(alias); // e.g., '@saveUser'
});
```
Usage:
```js
cy.intercept('POST', '/api/users').as('saveUser');
cy.submitAndWait('@saveUser');
```

---

## Best Practices & Pitfalls
- ✅ **Return** the Cypress chain from your command (so it remains chainable & retryable).
- ✅ Prefer **selectors like `[data-cy="..."]`** over CSS classes that may change.
- ✅ Keep commands **small, focused, and pure** (no test-specific assertions inside shared commands).
- ✅ For secrets, use **`Cypress.env()`** not hard-coded values.
- ✅ For heavy logic or file-system tasks, bridge via **`cy.task()`** (Node side).
- ⚠️ **Don’t return raw Promises**; wrap with `cy.wrap()` if needed.
- ⚠️ Avoid `async/await` inside commands—Cypress chains are **not Promises**.
- ⚠️ Use `Cypress.Commands.overwrite` **sparingly**—surprises confuse teammates.
- ⚠️ Beware of **subject types** (`prevSubject`) to avoid awkward command errors.

---

## Cheatsheet
```js
// Add (parent)
Cypress.Commands.add('login', (u, p) => { /* ... */ });

// Add (child)
Cypress.Commands.add('withinByData', { prevSubject: true }, (subject, val) => {
  return cy.wrap(subject).find(`[data-cy="${val}"]`);
});

// Add (dual)
Cypress.Commands.add('dataCy', { prevSubject: 'optional' }, (subject, val) => {
  const sel = `[data-cy="${val}"]`;
  return subject ? cy.wrap(subject).find(sel) : cy.get(sel);
});

// Overwrite
Cypress.Commands.overwrite('visit', (orig, url, opts) => orig(url, { timeout: 20000, ...opts }));

// Export types (TS): cypress/support/commands.d.ts (see section above)
```

---

Happy testing & extending! 🚀
