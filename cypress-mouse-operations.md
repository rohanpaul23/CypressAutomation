# 🖱️ Mouse Operations in Cypress

Cypress provides multiple commands to simulate **mouse operations** such
as clicking, hovering, dragging, and moving.

------------------------------------------------------------------------

## 1. **Click**

Simulates a user clicking on an element.

``` js
// Single click
cy.get('button').click();

// Force click (ignores visibility or overlap issues)
cy.get('button').click({ force: true });

// Double click
cy.get('button').dblclick();

// Right click
cy.get('button').rightclick();
```

------------------------------------------------------------------------

## 2. **Hover**

Cypress doesn't have a direct `.hover()` command.\
Use `.trigger('mouseover')` to simulate hover.

``` js
cy.get('.menu-item').trigger('mouseover');
cy.get('.submenu').should('be.visible');
```

------------------------------------------------------------------------

## 3. **Mouse Down & Mouse Up**

Useful for drag-and-drop or custom UI interactions.

``` js
cy.get('#draggable').trigger('mousedown');
cy.get('#droppable').trigger('mouseup');
```

------------------------------------------------------------------------

## 4. **Mouse Move**

Simulate moving the mouse across coordinates or elements.

``` js
// Move relative to element
cy.get('#slider').trigger('mousemove', { clientX: 200, clientY: 100 });

// Move with chaining (drag-like behavior)
cy.get('#drag-area')
  .trigger('mousedown', { which: 1 })
  .trigger('mousemove', { clientX: 300, clientY: 200 })
  .trigger('mouseup');
```

------------------------------------------------------------------------

## 5. **Scroll Into View**

Ensures the element is visible before interacting.

``` js
cy.get('#hiddenButton').scrollIntoView().click();
```

------------------------------------------------------------------------

## 6. **Drag and Drop (Custom)**

No built-in `drag()` and `drop()` in Cypress.\
Simulate with `mousedown + mousemove + mouseup`.

``` js
cy.get('#item')
  .trigger('mousedown', { which: 1 });

cy.get('#dropzone')
  .trigger('mousemove')
  .trigger('mouseup', { force: true });
```

👉 Alternatively, use the **cypress-drag-drop** plugin:

``` js
import '@4tw/cypress-drag-drop';

cy.get('#item').drag('#dropzone');
```

------------------------------------------------------------------------

## 7. **Context Menu (Right Click)**

``` js
cy.get('.file').rightclick();
cy.get('.context-menu').should('be.visible');
```

------------------------------------------------------------------------

## 📝 Summary Table

  --------------------------------------------------------------------------
  Operation       Cypress Command / Approach
  --------------- ----------------------------------------------------------
  Single click    `cy.get('el').click()`

  Double click    `cy.get('el').dblclick()`

  Right click     `cy.get('el').rightclick()`

  Hover           `cy.get('el').trigger('mouseover')`

  Mouse down/up   `cy.get('el').trigger('mousedown') / trigger('mouseup')`

  Mouse move      `cy.get('el').trigger('mousemove', {clientX, clientY})`

  Drag & drop     `mousedown → mousemove → mouseup` (or plugin)

  Scroll & click  `cy.get('el').scrollIntoView().click()`
  --------------------------------------------------------------------------

------------------------------------------------------------------------
