# UI Map — User Flows

Covers: multi-page user journeys — the sequence of pages/steps a user goes through to accomplish something, and the reasoning behind why it's structured that way. Does not cover single-page/single-component detail — link out to `component-index.md` or a component's own doc instead of re-describing a page here.

This is different from the Route Table in `component-index.md`: the route table is a flat technical listing of "path → page component." A user flow is the *narrative* — the order steps happen in, what choice points exist, and why, from the user's perspective rather than the router's.

## When to Add a Flow

Add an entry for any journey that spans more than one page/component and has a clear start and end from the user's point of view (onboarding, checkout, account setup, a multi-step form spread across pages). A single page that just does one thing doesn't need an entry here — that's just a row in `component-index.md`.

## Entry Template

```
### <flow name>
**Goal:** <what the user is trying to accomplish>
**Entry point(s):** <where a user starts this flow from — e.g. a specific page, or several>
**Feature flag:** <name of the config/feature flag this flow is gated behind, if any — write "Always on" if it isn't gated. This is what makes it visible, at a glance, which flows aren't universally active — see shared-infrastructure.md if the flag itself needs documenting as a config value.>
**Steps:**
1. <page/component> — <what happens here, any validation/decision made>
2. <page/component> — ...
3. ...
**Exit conditions:** <how the flow ends — success and, if relevant, abandonment/cancel>
**Why structured this way:** <the product/UX reasoning — e.g. "split across three pages instead of one long form to reduce perceived effort and allow saving progress." If this reasoning is substantial enough to be its own standalone decision, log it in `decisions.md` and reference its ID (e.g. "see UI-004") here instead of duplicating it.>
**Related components:** <links into `component-index.md`>
```

## Flows

_(entries go here as flows are built)_
