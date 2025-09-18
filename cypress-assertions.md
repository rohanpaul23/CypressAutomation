# Cypress Assertions: Implicit vs Explicit

## 🔹 Implicit Assertions

-   Happen automatically after a Cypress command.
-   Cypress **retries** the command until the assertion passes or times
    out (built-in retry-ability).
-   You don't have to explicitly call `should()` or `and()` --- but
    often you do to extend checks.

### Example

``` js
// Cypress will keep retrying until element is found and visible
cy.get('button').should('be.visible');

// Chain multiple implicit assertions
cy.get('input')
  .should('have.value', 'Hello')
  .and('be.enabled');
```

👉 **Key point:** Cypress automatically waits for conditions to be true
before proceeding.

------------------------------------------------------------------------

## 🔹 Explicit Assertions

-   Require using **assertion libraries** like **Chai**,
    **Chai-jQuery**, or **expect/assert**.
-   They do **not retry** automatically --- they check once at the time
    they are executed.
-   Useful when you want more **direct control** over conditions.

### Example

``` js
// Using Chai expect
cy.get('h1').then(($el) => {
  expect($el.text()).to.equal('Welcome');
});

// Using Chai assert
cy.get('h1').then(($el) => {
  assert.equal($el.text(), 'Welcome');
});
```

👉 **Key point:** Explicit assertions require `.then()` to grab the
subject and run synchronous checks.

------------------------------------------------------------------------

## ✅ Quick Comparison

  --------------------------------------------------------------------------------------------------------------
  Feature           Implicit Assertion (`should`, `and`)     Explicit Assertion (`expect`, `assert`)
  ----------------- ---------------------------------------- ---------------------------------------------------
  Retry-ability     ✅ Yes                                   ❌ No

  Ease of use       ✅ Simple, chainable                     ⚠️ Requires `.then()`

  Assertion         Built into Cypress + Chai-JQuery         Chai `expect` / `assert`
  libraries used                                             

  Example           `cy.get('input').should('be.visible')`   `cy.get('input').then(el => expect(el).to.exist)`
  --------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## ⚡ Rule of Thumb

-   Use **implicit assertions** when checking UI conditions (visibility,
    text, value).
-   Use **explicit assertions** when validating complex logic, custom
    calculations, or multiple values after grabbing elements.
