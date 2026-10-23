---
name: hypermedia-project
description: Organize, build and review a web project around HATEOAS and hypermedia with htmx v4 and KISS — server returns HTML, the server owns all state, links and forms carry every state transition, no client-side framework. Language and framework neutral. Use when starting a project, adding a resource or page, deciding what an endpoint should return (full page, fragment, multi-target update), choosing between htmx and JavaScript, or reviewing a codebase that drifted toward JSON APIs and client state. For htmx v4 syntax and migration details, read HTMX4.md.
---

# Hypermedia project

The server is the application. The browser is a hypermedia client: it renders HTML, follows links, submits forms. Every state change is a request to the server that returns a new representation. No client-side state store, no JSON API consumed by our own UI.

## Principles (in priority order)

1. **HTML is the API.** Responses are HTML. JSON exists only for third-party machine clients, and then as a separate, explicit surface.
2. **Hypermedia drives state.** What the user can do next is whatever links and forms the server put in the response. Hide or disable an action by not rendering it; never ship the rule to the client.
3. **Server owns state.** Session, selection, filters, wizard step, validation: server-side or in the URL. The DOM is a view, not a database.
4. **Resources, not RPC.** Nouns in URLs, verbs from HTTP. `POST /orders/42/cancellation`, not `POST /cancelOrder`. If a button needs an endpoint named like a function, find the resource it creates or changes.
5. **Progressive enhancement.** The page works as plain links and forms. htmx upgrades it: partial swaps, no full reload. Test first with JS off.
6. **KISS.** Fewest moving parts that work. Each dependency, build step, and line of JS needs a reason a link, form, or htmx attribute could not cover.

## Decide: what does this endpoint return?

| Request | Return |
|---|---|
| Normal navigation (no `HX-Request`) | Full page |
| htmx request replacing one region | Only that fragment |
| One action changes several places | `<hx-partial>` blocks (v4) or out-of-band swaps, in one response |
| Action succeeded, nothing changes | `204` |
| Validation failed | Re-render the form with errors, with an error status, so the form swaps in place |
| Created / moved | Redirect for navigation; the new representation for in-place swaps |

One template per resource representation. The full page composes the same fragment templates; never maintain a "page version" and a "fragment version" of the same thing. Branch on `HX-Request` / `HX-Request-Type` only to add or omit the layout.

## Project layout (adapt names to the language)

```
routes/        one module per resource: list, show, new, edit, create, update, delete
templates/
  layout       page shell, loaded only for full-page responses
  <resource>/  list, row, detail, form  (fragments, each independently renderable)
domain/        business rules and persistence, no HTML, no HTTP
static/        css, htmx, images; almost no JS
```

Rules: routes are thin (parse, call domain, pick template). Domain knows nothing of HTTP. Templates contain no business logic beyond showing what the server already decided is allowed.

## Adding a resource: checklist

1. Name the resource and its states. Write which transitions exist between states.
2. For each state, list what the user can do next: those become links/forms in that state's representation.
3. Map actions to HTTP: read `GET`, create `POST` on the collection, change `PUT`/`PATCH` on the item, remove `DELETE`; anything else becomes a sub-resource (`/orders/42/cancellation`).
4. Build the fragments, then compose the full page from them.
5. Add htmx attributes last, one at a time, only where a full reload hurts.
6. Check: works without JS? Back button and refresh sane? URL reflects the state worth sharing or bookmarking?

## When JavaScript is allowed

Only for what the server cannot do: focus management, a drag handle, a canvas, a media player, clipboard. Keep it a small, local, framework-free script attached to an element, communicating with the server through htmx events or a form submission. Never mirror server data into client state.

## Smells to flag in review

- Our own UI calling a JSON endpoint and rendering it client-side.
- Endpoints named as verbs or functions.
- Business rules duplicated in JS or template conditionals (`if user.role == ...` deciding permissions).
- Client state: stores, hidden fields mirroring server state, `localStorage` as source of truth.
- Two templates for one representation.
- htmx used as a JS framework: chains of triggers, `hx-on` with logic, polling to fake reactivity where a response could include the update.
- Interactions that break with JS disabled, or that don't update the URL when the state is shareable.
- Success signaled through toasts built in JS rather than a rendered response.

## Output when asked to review

List smells found with file and line, the hypermedia replacement for each, ordered by how much client logic it removes. Do not rewrite unrequested.
