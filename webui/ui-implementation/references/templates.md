# Templates

Use these as the fixed starting skeleton for a new file. Don't invent a different structure — copy the shape, then fill in the specifics for the task at hand. Both assume the shared base class is `BonyanElement`, imported from wherever the project's base class module lives.

## Regular Component Template

For an ordinary component: receives whatever references it needs via the base class's single injection method, has local presentational state only, exposes a public method if a parent needs to trigger an action on it, and shows the standard pattern for listening to a domain event.

```js
import { BonyanElement } from '../base/bonyan-element.js';

const TAG = 'app-example-widget';

class ExampleWidget extends BonyanElement {
  // References are never constructor arguments (see component-basics.md — Custom Elements
  // expects a no-argument constructor). They arrive through the base class's one fixed
  // injection method, called exactly once per instance, before it's appended to the DOM:
  //   const el = document.createElement(TAG);
  //   el.connect({ someService });
  //   parent.appendChild(el);
  // connect() is not an update hook — calling it twice on the same instance is a bug the
  // base class throws on. If references need to change, discard this instance and create a new one.
  #someService = null;

  connect({ someService }) {
    this.#someService = someService;
    return this; // BonyanElement's connect() convention: return `this` so the call can be chained.
  }

  // Local, transient, presentational state only — never product data (see component-basics.md,
  // decision-guide.md). This is genuinely presentational: it has no meaning outside this widget.
  #isOpen = false;

  // Public API — the only methods a parent is allowed to call directly (see composition.md).
  open() {
    this.#isOpen = true;
    this.requestRender(); // provided by BonyanElement; triggers the render cycle, never a direct DOM write.
  }

  close() {
    this.#isOpen = false;
    this.requestRender();
  }

  // Standard pattern for reacting to a domain event (see component-basics.md, architecture-context/naming-conventions.md).
  // The base class calls this for every subscription declared in `subscriptions()`; it subscribes on connect
  // and unsubscribes on disconnect automatically — a component never manages this manually.
  subscriptions() {
    return {
      'transactions:saved': () => {
        // Domain events carry identifying data only, never the full entity (see architecture-context/core-boundaries.md).
        // Re-fetch from the service if this widget needs to reflect the change — never trust the event payload
        // to contain everything, and never assume the payload alone is enough to update local state.
        this.requestRender();
      }
    };
  }

  // One fixed template method — the only place markup is produced (see component-basics.md).
  render() {
    return `
      <div class="widget" part="widget">
        ${this.#isOpen ? '<div class="panel">...</div>' : ''}
        <button data-action="toggle">Toggle</button>
      </div>
    `;
  }

  // One delegated listener per event type, registered once by the base class calling this hook,
  // never re-attached per render (see component-basics.md, Event Binding Rule).
  handleEvent(event) {
    if (event.type === 'click' && event.target.dataset.action === 'toggle') {
      this.#isOpen ? this.close() : this.open();
      // Child -> parent communication, if needed, is a bubbled CustomEvent, never a direct call upward.
      // IMPORTANT: any new `ui:*` event name must already be declared in the UI manifest's `events`
      // section before it's dispatched here — see architecture-context/ui-manifest.md. Don't introduce
      // a new ui:* event type in a component file without that declaration existing first.
      this.dispatchEvent(new CustomEvent('ui:panel-toggled', {
        bubbles: true,
        composed: true,
        detail: { open: this.#isOpen }
      }));
    }
  }
}

customElements.define(TAG, ExampleWidget);
export { ExampleWidget, TAG };
```

## Page-Level Component Template

For a page mounted/unmounted by the Entry Point in response to `router:changed` (see `composition.md`, Component Ownership). Differs from a regular component mainly in: it receives route params through the same `connect()` contract, it typically requires services rather than treating them as optional, and it is never created by another component. It also demonstrates the cancellation-safe pattern for an async fetch that must not touch the component after it's been unmounted.

```js
import { BonyanElement } from '../base/bonyan-element.js';

const TAG = 'app-page-accounts-detail';

class AccountsDetailPage extends BonyanElement {
  #accountsService = null;
  #routeParams = null;

  connect({ accountsService, routeParams }) {
    this.#accountsService = accountsService;
    this.#routeParams = routeParams; // e.g. { id: '42' } from RouterService's matched route
    return this;
  }

  // `#account` here is transient render state, not a cache: it exists only so this page has
  // something to render between "fetch started" and "fetch finished," and it is discarded when
  // this page is unmounted. It is never treated as a source of truth — if the underlying account
  // changes elsewhere, this page re-fetches (see the subscriptions() example above) rather than
  // trusting whatever is already sitting in this field. See decision-guide.md: the account data
  // itself still belongs to the service; this field is just where the last fetched copy is held
  // long enough to be rendered.
  #status = 'loading'; // 'loading' | 'ready' | 'error'
  #account = null;

  // Cancellation guard: an async fetch started while mounted must not update state (or call
  // requestRender()) after the page has since been unmounted — e.g. the user navigated away
  // before the request finished. The reliable, contract-independent part of this guard is the
  // `this.isConnected` check after the await, below — that check alone is enough regardless of
  // what the service does. Passing an `AbortSignal` to the service call, as done here, is an
  // additional optimization to stop unnecessary work early, but it is NOT part of the standard
  // service contract (see architecture-context/service-contract.md) — a service is not guaranteed
  // to accept or honor a `signal` option. Only rely on it if this specific service's manifest
  // documents that it accepts one; otherwise omit the `{ signal }` argument and keep just the
  // `isConnected` check, which is always safe on its own.
  #abortController = null;

  async connectedCallback() {
    super.connectedCallback(); // registers Event Bus subscriptions, delegated listener, etc.
    this.#abortController = new AbortController(); // only useful if the service supports it — see above.
    try {
      const account = await this.#accountsService.getById(
        this.#routeParams.id
        // { signal: this.#abortController.signal } — include only if this service documents support for it.
      );
      if (!this.isConnected) return; // unmounted while the fetch was in flight — do nothing.
      this.#account = account;
      this.#status = 'ready';
    } catch (error) {
      if (!this.isConnected) return;
      this.#status = 'error';
      // Display convention for a service-rejected/error case: see forms-and-i18n.md (code -> translated string).
    }
    this.requestRender();
  }

  disconnectedCallback() {
    this.#abortController?.abort();
    super.disconnectedCallback(); // removes Event Bus subscriptions, delegated listener, etc.
  }

  render() {
    if (this.#status === 'loading') return `<div class="loading">...</div>`;
    if (this.#status === 'error') return `<div class="error">...</div>`;
    return `<div class="account-detail">${/* use this.#account fields */ ''}</div>`;
  }
}

customElements.define(TAG, AccountsDetailPage);
export { AccountsDetailPage, TAG };
```

## What Not to Change Between Components

Regardless of what a specific component does, every new file keeps: one `render()` method, references received only via the base class's `connect()` method, local state limited to presentation (fetched data held only transiently for rendering, never as a substitute for the service), domain-event reactions declared through `subscriptions()`, a delegated event-handling entry point, and a `customElements.define()` + named export at the bottom. Async work that outlives a possible unmount uses the `AbortController` + `isConnected` guard shown above. See `forbidden-patterns.md` before considering any deviation from this shape.
