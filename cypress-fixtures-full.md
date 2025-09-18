# 📂 Fixtures in Cypress

Fixtures are **static external data files** that Cypress can load and reuse in your tests.  
They are commonly used for **test data** and **mocking API responses**.

---

## Table of Contents
1. [What are Fixtures?](#what-are-fixtures)
2. [Default Location](#default-location)
3. [Basic Fixture Usage](#basic-fixture-usage)
4. [Using Fixtures in Tests](#using-fixtures-in-tests)
5. [Using Fixtures with `this` Context](#using-fixtures-with-this-context)
6. [Using Fixtures with `cy.intercept()`](#using-fixtures-with-cyintercept)
7. [Non-JSON Fixtures](#non-json-fixtures)
8. [Best Practices](#best-practices)
9. [Cheatsheet](#cheatsheet)

---

## What are Fixtures?
- Fixtures are **predefined static data** stored in files.
- Default directory: `cypress/fixtures/`.
- Supported formats: JSON, TXT, CSV, PNG, JPG, etc.
- Useful for:
  - Mocking API responses
  - Supplying test input (e.g., login credentials)
  - Reusing datasets across multiple tests

---

## Default Location
```
cypress/
  ├── fixtures/
  │     ├── users.json
  │     ├── products.json
  │     └── example.txt
```

---

## Basic Fixture Usage
```js
describe('Fixture Example', () => {
  it('loads user data', () => {
    cy.fixture('users').then((data) => {
      cy.log(data.username);
      cy.log(data.password);
    });
  });
});
```

👉 Cypress automatically looks inside `cypress/fixtures/`.

---

## Using Fixtures in Tests
Fill form fields with fixture data:

```js
cy.fixture('users').then((user) => {
  cy.get('input[name="username"]').type(user.username);
  cy.get('input[name="password"]').type(user.password);
  cy.get('button[type="submit"]').click();
});
```

---

## Using Fixtures with `this` Context
```js
describe('Login with fixture', () => {
  beforeEach(function () {
    cy.fixture('users').then((user) => {
      this.user = user;
    });
  });

  it('logs in using fixture data', function () {
    cy.get('input[name="username"]').type(this.user.username);
    cy.get('input[name="password"]').type(this.user.password);
    cy.get('button[type="submit"]').click();
  });
});
```

⚠️ Must use `function () {}` instead of arrow functions, otherwise `this` is undefined.

---

## Using Fixtures with `cy.intercept()`
Mock network responses with fixture files:

```js
cy.intercept('GET', '/api/products', { fixture: 'products.json' }).as('getProducts');

cy.visit('/products');
cy.wait('@getProducts');
```

👉 Cypress serves fixture data instead of calling the real backend.

---

## Non-JSON Fixtures
Cypress can also load `.txt`, `.csv`, `.png`, etc.

```js
cy.fixture('example.txt').then((text) => {
  cy.log(text);
});
```

---

## Best Practices
- ✅ Keep fixture files **small and modular**.  
- ✅ Name files meaningfully (`users.json`, `orders.json`).  
- ✅ Use **separate fixtures** for different test cases.  
- ✅ Combine with `cy.intercept()` for mocking APIs.  
- ❌ Don’t store sensitive data (use environment variables instead).  

---

## Cheatsheet
```js
// Load fixture
cy.fixture('users').then((u) => { ... });

// Alias fixture
cy.fixture('users').as('userData');
cy.get('@userData').then((u) => { ... });

// With intercept
cy.intercept('GET', '/api/products', { fixture: 'products.json' }).as('getProducts');
```

---

✅ Fixtures make Cypress tests **predictable, fast, and independent** of backend state.
