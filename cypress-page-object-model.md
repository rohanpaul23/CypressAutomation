# 🧩 Page Object Model (POM) in Cypress — Practical Guide

Page Object Model (POM) is a design pattern that organizes UI automation by **encapsulating page structure and actions** in dedicated classes/objects. Tests then call these page APIs instead of directly querying selectors everywhere.

---

## ✅ Why use POM?
- **Readability**: tests read like user flows (e.g., `loginPage.loginAs(user)`).
- **Reusability**: centralize page actions used by many specs.
- **Maintainability**: change selectors in one place when UI changes.
- **Abstraction**: hide complex flows (e.g., multi-step wizards) behind methods.

> Use **data-cy** (or `data-testid`) attributes for stable selectors in POM files.

---

## 📁 Suggested Folder Structure

```
cypress/
  e2e/
    auth/
      login.cy.ts
    checkout/
      checkout.cy.ts
  fixtures/
    users.json
  support/
    commands.ts         # optional: custom commands
    e2e.ts              # loads commands & global setup
  pages/                # ← POM lives here
    base.page.ts
    login.page.ts
    dashboard.page.ts
    cart.page.ts
```

> Keep POMs under `cypress/pages/` (or `cypress/support/pages`).

---

## 🧱 Base Page (common helpers)

```ts
// cypress/pages/base.page.ts
export abstract class BasePage {
  protected byData(id: string) {
    return cy.get(`[data-cy="${id}"]`);
  }

  visit(path: string) {
    cy.visit(path);
  }

  assertUrlIncludes(partial: string) {
    cy.url().should('include', partial);
  }
}
```

---

## 🔐 Login Page Object

```ts
// cypress/pages/login.page.ts
import { BasePage } from './base.page';

export class LoginPage extends BasePage {
  open() {
    this.visit('/login');
  }

  username() { return this.byData('username'); }
  password() { return this.byData('password'); }
  submit() { return this.byData('submit'); }
  error() { return this.byData('error'); }

  fillCredentials(user: { username: string; password: string }) {
    this.username().clear().type(user.username);
    this.password().clear().type(user.password, { log: false });
  }

  loginAs(user: { username: string; password: string }) {
    this.open();
    this.fillCredentials(user);
    this.submit().click();
  }
}
```

---

## 🏠 Dashboard Page Object

```ts
// cypress/pages/dashboard.page.ts
import { BasePage } from './base.page';

export class DashboardPage extends BasePage {
  greeting() { return this.byData('greeting'); }
  cartLink() { return this.byData('cart-link'); }
  searchBox() { return this.byData('search'); }

  assertWelcome(name: string | RegExp) {
    this.greeting().should('contain.text', name);
  }

  goToCart() {
    this.cartLink().click();
    this.assertUrlIncludes('/cart');
  }
}
```

---

## 🧪 Example Test Using POM

```ts
// cypress/e2e/auth/login.cy.ts
import { LoginPage } from '../../pages/login.page';
import { DashboardPage } from '../../pages/dashboard.page';

const loginPage = new LoginPage();
const dashboard = new DashboardPage();

describe('Login Flow', () => {
  beforeEach(() => {
    cy.fixture('users').as('users');
  });

  it('logs in successfully and lands on dashboard', function () {
    const user = this.users.validUser;
    loginPage.loginAs(user);

    // Assertion is still in the test (keep validation near the test intent)
    dashboard.assertWelcome(user.name);
  });

  it('shows error for invalid credentials', function () {
    const user = this.users.invalidUser;
    loginPage.open();
    loginPage.fillCredentials(user);
    loginPage.submit().click();

    loginPage.error().should('be.visible').and('contain', 'Invalid');
  });
});
```

**Fixture example**
```json
// cypress/fixtures/users.json
{
  "validUser": { "username": "alice", "password": "Secret123", "name": "Alice" },
  "invalidUser": { "username": "alice", "password": "wrong" }
}
```

---

## 🎛️ Tips: Keep Assertions Mostly in Tests
- POM should **expose elements and actions**; keep **test intent & assertions** in specs.
- Exceptions: very common invariant checks (e.g., `assertUrlIncludes`), or “minors” like `should('be.visible')` inside helpers.

---

## 🌐 Intercept within POM (optional)

You may compose network stubs for flows:
```ts
// cypress/pages/login.page.ts (snippet)
loginWithStub(user) {
  cy.intercept('POST', '/api/login', { fixture: 'login-success.json' }).as('login');
  this.loginAs(user);
  cy.wait('@login');
}
```
> Keep heavy network logic in tests or dedicated helpers to avoid POM bloat.

---

## 🧩 Page Components (Widget Objects)
For large pages, split into **component objects** (e.g., `Header`, `CartWidget`, `UserMenu`).

```ts
// cypress/pages/components/header.component.ts
import { BasePage } from '../base.page';

export class Header extends BasePage {
  userMenu() { return this.byData('user-menu'); }
  logout() { return this.byData('logout'); }
  openUserMenu() { this.userMenu().click(); }
}
```
Compose in a page:
```ts
// cypress/pages/dashboard.page.ts (snippet)
import { Header } from './components/header.component';
export class DashboardPage extends BasePage {
  header = new Header();
}
```

---

## 🧠 POM vs Custom Commands
- **POM**: Encapsulates **page-specific** interactions & elements.
- **Custom Commands** (`Cypress.Commands.add`) : Encapsulate **cross-cutting** or **domain** actions (e.g., `cy.loginApi()`, `cy.dataCy()`).
- Use both together: POM for structure; Commands for global utilities.

---

## 🚫 Common Anti‑Patterns
- **Asserting everything inside POM** → hides intent; tests become opaque.
- **Over‑abstracting**: one giant “God” page object with 100 methods.
- **CSS classes for selectors**: prefer stable `data-cy` hooks.
- **Returning raw Promises**: always return Cypress chains (`cy.wrap(...)` if needed).
- **Mixing env/secrets in POM**: store in `Cypress.env()`.

---

## 🧪 Example: Checkout Flow with Multiple Pages

```ts
// cypress/e2e/checkout/checkout.cy.ts
import { LoginPage } from '../../pages/login.page';
import { DashboardPage } from '../../pages/dashboard.page';
import { CartPage } from '../../pages/cart.page';

const login = new LoginPage();
const dashboard = new DashboardPage();
const cart = new CartPage();

describe('Checkout', () => {
  beforeEach(function () {
    cy.fixture('users').as('users');
  });

  it('adds item and checks out', function () {
    login.loginAs(this.users.validUser);
    dashboard.byData('add-item-1').click();
    dashboard.goToCart();

    cart.items().should('have.length.at.least', 1);
    cart.checkout().click();

    cy.url().should('include', '/checkout/confirmation');
    cy.contains(/order confirmed/i).should('be.visible');
  });
});
```

Example Cart page:
```ts
// cypress/pages/cart.page.ts
import { BasePage } from './base.page';

export class CartPage extends BasePage {
  items() { return this.byData('cart-item'); }
  checkout() { return this.byData('checkout'); }
}
```

---

## 🧾 TypeScript Declarations (optional)

If you export singletons for pages, you can type them globally:

```ts
// cypress/pages/index.ts
export * from './base.page';
export * from './login.page';
export * from './dashboard.page';
export * from './cart.page';
```

Then in tests:
```ts
import { LoginPage, DashboardPage } from '../../pages';
```

---

## ✅ Checklist

- [ ] Use a **BasePage** for common helpers like `byData`, `visit`, url assertions.
- [ ] Create **one page object per route** (and component objects for big widgets).
- [ ] Keep **selectors stable** with `data-cy` hooks.
- [ ] Keep **assertions primarily in tests** (POM exposes elements & actions).
- [ ] Prefer **short, focused methods** (e.g., `fillCredentials`, `goToCart`).
- [ ] Avoid mixing POM with **global concerns** (env, tokens) — use commands/fixtures.

---

Happy structuring! 🧱✨
