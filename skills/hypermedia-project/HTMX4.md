# htmx v4 reference (differences that matter in practice)

Source: htmx 4 migration guide, https://four.htmx.org/migration-guide-htmx-4/. v4 is recent: re-check that page when behavior surprises you.

## Requests
- Transport is `fetch()`, not XHR.
- `hx-get` / `hx-delete` no longer include the enclosing form's inputs: add `hx-include="closest form"`.
- Default timeout is 60 s.
- Headers: `HX-Source` (`tag#id`) replaces `HX-Trigger` / `HX-Trigger-Name`; `HX-Target` is `tag#id`; `HX-Request-Type` is `full` or `partial`; `Accept` is `text/html`. Branch layout vs fragment on these.

## Inheritance is explicit
Attributes do not inherit by default. Write `hx-confirm:inherited="..."` to flow down the DOM, `:append` to extend an inherited value. `hx-disinherit` / `hx-inherit` are gone.

## Swap
- Every response swaps except `204` and `304`. 4xx and 5xx swap too: render the error as HTML (re-rendered form with messages) and it lands in place.
- Per-status control with `hx-status:422="..."`, `hx-status:5xx="..."`; keys: `swap`, `target`, `select`, `push`, `replace`, `transition`.
- New styles: `innerMorph`, `outerMorph` (idiomorph), `textContent`, `delete`; aliases `before`, `after`, `prepend`, `append`.
- OOB swaps run after the main swap.
- `show` / `scroll` take `showTarget` / `scrollTarget` keys instead of `selector:position`.

## Several updates from one response
`<hx-partial>` blocks, each with its own `hx-target` and `hx-swap`, as an alternative to `hx-swap-oob`. Prefer it when one action touches several regions.

## Other
- `hx-config` sets per-element request options; `hx-request` is removed.
- `hx-disable` is now what `hx-disabled-elt` was; the old `hx-disable` is `hx-ignore`.
- History: no localStorage cache; back re-fetches. `htmx.config.history` can be `"reload"` or `false`. The `hx-history-cache` extension restores from sessionStorage.
- SSE and WebSocket are core extensions `hx-sse` / `hx-ws`.
- Events follow `htmx:<phase>:<action>`: `htmx:before:request`, `htmx:after:swap`, `htmx:after:init`, `htmx:before:response`, `htmx:finally:request`; errors via `htmx:error` and `htmx:response:error`.

## Motion with htmx
- Morph swaps keep element identity, so CSS transitions can animate between states without JS.
- Use CSS and View Transitions (the `transition` key in `hx-status`; see the view transition events in the migration guide). Animate composite-only properties. For easing, duration and review, use the `animate`, `animation-performance` and `review-animations` skills.
