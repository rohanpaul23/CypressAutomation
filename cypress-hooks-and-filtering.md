# 🔄 Hooks in Cypress

Cypress (using Mocha) provides hooks to run **setup and teardown logic**
before/after tests or test suites.

------------------------------------------------------------------------

## 1. **before()**

Runs **once before all tests** in the `describe` block.\
Used for **global setup** (e.g., seeding DB, logging in once).

``` js
describe('User Tests', () => {
  before(() => {
    // Runs once before all tests
    cy.log('Setting up before all tests');
  });

  it('test 1', () => {
    cy.log('Running test 1');
  });
});
```

------------------------------------------------------------------------

## 2. **after()**

Runs **once after all tests** in the `describe` block.\
Used for **cleanup** (e.g., clearing DB, logging out).

``` js
describe('User Tests', () => {
  after(() => {
    // Runs once after all tests
    cy.log('Cleaning up after all tests');
  });

  it('test 1', () => {
    cy.log('Running test 1');
  });
});
```

------------------------------------------------------------------------

## 3. **beforeEach()**

Runs **before each test** inside the `describe` block.\
Common for **repeating setup tasks** like visiting a page.

``` js
describe('Login Tests', () => {
  beforeEach(() => {
    // Runs before every test
    cy.visit('/login');
  });

  it('should display login form', () => {
    cy.get('form').should('exist');
  });

  it('should allow user to type username', () => {
    cy.get('input[name="username"]').type('testuser');
  });
});
```

------------------------------------------------------------------------

## 4. **afterEach()**

Runs **after each test** inside the `describe` block.\
Used for **teardown tasks** (e.g., clearing localStorage).

``` js
describe('Cart Tests', () => {
  afterEach(() => {
    // Runs after each test
    cy.clearLocalStorage();
  });

  it('adds item to cart', () => {
    cy.addToCart('Laptop');
  });

  it('removes item from cart', () => {
    cy.removeFromCart('Laptop');
  });
});
```

------------------------------------------------------------------------

## ⚡ Typical Use Cases

-   `before()` → **One-time setup** (e.g., DB seed, API login).\
-   `after()` → **One-time cleanup** (e.g., DB cleanup, logout).\
-   `beforeEach()` → **Repeatable setup** (e.g., visiting a page,
    resetting state).\
-   `afterEach()` → **Repeatable teardown** (e.g., clearing cookies,
    logs).

------------------------------------------------------------------------

## 📝 Example: Combining Hooks

``` js
describe('E2E Checkout Flow', () => {
  before(() => {
    cy.log('Seed database');
  });

  beforeEach(() => {
    cy.visit('/shop');
  });

  afterEach(() => {
    cy.clearCookies();
  });

  after(() => {
    cy.log('Reset database');
  });

  it('should add product to cart', () => {
    cy.get('.product').first().click();
    cy.get('#add-to-cart').click();
    cy.get('#cart').should('contain', '1 item');
  });

  it('should checkout successfully', () => {
    cy.get('#cart').click();
    cy.get('#checkout').click();
    cy.contains('Order confirmed');
  });
});
```

------------------------------------------------------------------------

# 🎯 Test Filtering in Cypress (`skip` and `only`)

Cypress (via Mocha) allows you to control **which tests run** using
`.only` and `.skip`.

------------------------------------------------------------------------

## 1. **`it.only`**

Runs **only that single test**, skipping all others in the file.

``` js
it.only('should run only this test', () => {
  cy.log('Only this test runs');
});

it('this test will be skipped', () => {
  cy.log('This will NOT run');
});
```

------------------------------------------------------------------------

## 2. **`describe.only`**

Runs **only that suite (block)**, skipping other `describe` blocks.

``` js
describe.only('Checkout Tests', () => {
  it('should add to cart', () => {
    cy.log('Runs');
  });

  it('should place order', () => {
    cy.log('Runs');
  });
});

describe('Login Tests', () => {
  it('should login user', () => {
    cy.log('Skipped');
  });
});
```

------------------------------------------------------------------------

## 3. **`it.skip`**

Marks a test to be **skipped**, but doesn't delete it.

``` js
it.skip('should be skipped temporarily', () => {
  cy.log('Not executed');
});

it('this one will run', () => {
  cy.log('Executed');
});
```

------------------------------------------------------------------------

## 4. **`describe.skip`**

Skips an **entire suite** of tests.

``` js
describe.skip('Profile Tests', () => {
  it('should update name', () => {
    cy.log('Skipped');
  });

  it('should update email', () => {
    cy.log('Skipped');
  });
});
```

------------------------------------------------------------------------

## ⚠️ Gotcha

-   Never leave `.only` in committed code → CI/CD will run only that
    test/suite.\
-   Use `it.skip` instead of commenting out tests (so they show as
    "pending").\
-   Use ESLint plugins (`eslint-plugin-mocha`) to **block `.only` in
    commits**.

------------------------------------------------------------------------

## ✅ Summary Table

  ------------------------------------------------------------------------
  Keyword           Scope              Behavior
  ----------------- ------------------ -----------------------------------
  `it.only`         Single test        Runs only this test, skips others

  `describe.only`   Test suite (block) Runs only this block, skips others

  `it.skip`         Single test        Skips this test

  `describe.skip`   Test suite (block) Skips this suite
  ------------------------------------------------------------------------

------------------------------------------------------------------------
