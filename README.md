# Account Details LWC

A lightweight Lightning Web Component that displays a Salesforce Account Id passed in from an external application, with a waiting/loading state until the value arrives.

## Overview

This component renders a simple card UI that shows the Account Id once it has been received from an outside source (e.g. an external app messaging into the page, a parent component, or a platform event listener). Until that data arrives, it shows a "waiting" placeholder instead of an empty or broken UI.

## UI States

| State | Condition | What's shown |
|---|---|---|
| Waiting | `receivedValue` is falsy | "Waiting for Account Id..." message |
| Received | `receivedValue` is truthy | Account Id value + a success confirmation message |

## Template Structure

- **`.header`** — Icon, title ("Account Details"), and a short description of the component's purpose.
- **`.content`** — Conditionally renders one of the two states above using `template if:true` / `if:false`.
- **`.value-card`** — Displays the label ("Salesforce Account Id") and the resolved value (`{parsedData}`).
- **`.success`** — Confirmation message shown once data has been received.
- **`.waiting`** — Placeholder shown while no data has arrived yet.

## Properties (expected in the JS controller)

| Property | Type | Purpose |
|---|---|---|
| `receivedValue` | Boolean | Tracks whether data has arrived; toggles between the waiting and received states |
| `parsedData` | String | The parsed/display-ready Account Id shown in the value card |

> These are referenced in the template but not shown here — add the corresponding `.js` file's logic (e.g. how `receivedValue`/`parsedData` get set: `postMessage` listener, wire adapter, public `@api` property, etc.) so this section can be filled in accurately.

## Usage

Drop `<c-account-details></c-account-details>` into a parent component or Lightning page. Once the external application delivers the Account Id (via whatever channel the JS controller implements), the component automatically switches from the waiting state to the received state — no manual refresh needed.

## Notes / Assumptions

- This README was written from the template markup only; the JS controller (`.js`), metadata (`.js-meta.xml`), and CSS (`.css`) were not provided.
- The exact mechanism for receiving the Account Id (postMessage from an iframe, platform event, parent-to-child `@api` property, etc.) is unspecified in the template — update the **Properties** and **Usage** sections once that's confirmed.
- Component API name is assumed to be `accountDetails`; rename references if the actual folder/bundle name differs.
