# 🎯 Cypress Events Cheat Sheet

Cypress provides powerful event hooks for customizing behavior during test execution.  
Events can be divided into **Runner Events (Cypress.on)** and **DOM/App Events (cy.on, cy.trigger)**.

---

## 🌀 Runner Events (Global Lifecycle)

Use **`Cypress.on(event, callback)`** to subscribe to test runner events.

### Common Runner Events

| Event | Triggered When | Example Usage |
|-------|----------------|---------------|
| `uncaught:exception` | App throws an uncaught error | Ignore known errors |
| `fail` | A command or assertion fails | Log errors or capture screenshot |
| `window:before:load` | Before `window.onload` executes | Stub functions |
| `window:load` | After app window loads | Setup logic |
| `test:before:run` | Before each test runs | Add custom setup |
| `test:after:run` | After each test finishes | Cleanup, reporting |

### Examples

```js
// Ignore uncaught app exceptions
Cypress.on('uncaught:exception', (err, runnable) => {
  console.warn('App error ignored:', err.message);
  return false;
});

// Capture and log test failures
Cypress.on('fail', (err, runnable) => {
  console.error('Test failed:', runnable.title, err.message);
  throw err; // rethrow to still fail the test
});

// Modify window before app code runs
Cypress.on('window:before:load', win => {
  cy.stub(win.console, 'error').callsFake(msg => {
    console.log('Console error intercepted:', msg);
  });
});
```

---

## 🖱️ DOM & App Events

### Listening to DOM events with `cy.on`
```js
cy.get('button').click().then($btn => {
  $btn.on('click', e => {
    console.log('Button clicked!', e);
  });
});
```

### Triggering events with `cy.trigger`
```js
cy.get('input#name').trigger('focus');
cy.get('input#name').trigger('keydown', { keyCode: 13 });
```

---

## 🔔 Custom Application Events

Listen to or trigger custom events in your app using `cy.window()`.

```js
cy.window().then(win => {
  win.document.addEventListener('custom-event', e => {
    console.log('Custom event fired:', e.detail);
  });

  // Trigger a custom event
  win.document.dispatchEvent(
    new CustomEvent('custom-event', { detail: { foo: 123 } })
  );
});
```

---

## 📋 Best Practices

- Avoid overusing `uncaught:exception` to hide real bugs.  
- Use `Cypress.removeAllListeners()` to clean up global event listeners if needed.  
- Prefer `cy.trigger` for simulating user interactions (hover, drag, keydown).  
- Combine with `cy.intercept` to test network-driven events.  

---

# ✅ Summary

- **Runner events**: lifecycle hooks (`uncaught:exception`, `fail`, `window:before:load`, etc.).  
- **DOM events**: listen (`cy.on`) or simulate (`cy.trigger`).  
- **Custom events**: trigger and listen via `cy.window()`.  
- Great for **debugging, controlling failures, stubbing behavior, and simulating interactions**.

---