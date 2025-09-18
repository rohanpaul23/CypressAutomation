# 🌐 Browser Navigation in Cypress

Cypress provides commands to **navigate browser history** and **reload pages**, useful for testing flows that depend on back/forward navigation or refresh behavior.

---

## 🔹 `cy.go()`

Navigate through browser history (back/forward).

### Syntax
```js
cy.go(direction)
```

### Parameters
- `-1` → Go **back** one page
- `1` → Go **forward** one page
- `'back'` → Equivalent to `cy.go(-1)`
- `'forward'` → Equivalent to `cy.go(1)`

### Example
```js
cy.visit('/page1');
cy.visit('/page2');

// Go back to page1
cy.go('back');
cy.url().should('include', '/page1');

// Go forward to page2 again
cy.go('forward');
```

Equivalent shorthand:
```js
cy.go(-1);   // back
cy.go(1);    // forward
```

---

## 🔹 `cy.reload()`

Reload the current page.

### Syntax
```js
cy.reload()
```

### Options
- `cy.reload()` → Normal reload (uses cache)
- `cy.reload(true)` → Reload ignoring cache (force refresh)

### Example
```js
cy.visit('/dashboard');

// Trigger a normal reload
cy.reload();

// Force reload ignoring cache
cy.reload(true);
```

---

## ✅ Use Cases

- `cy.go('back')` → Validate navigation back works correctly.  
- `cy.go('forward')` → Validate forward navigation after going back.  
- `cy.reload()` → Ensure state persists after refresh.  
- `cy.reload(true)` → Test APIs or scripts load fresh without cache.  

---

## 📝 Summary

| Command           | Purpose                            | Example            |
|-------------------|------------------------------------|--------------------|
| `cy.go(-1)`       | Navigate **back** one page         | `cy.go('back')`    |
| `cy.go(1)`        | Navigate **forward** one page      | `cy.go('forward')` |
| `cy.reload()`     | Reload current page (with cache)   | `cy.reload()`      |
| `cy.reload(true)` | Reload current page ignoring cache | `cy.reload(true)`  |

---
