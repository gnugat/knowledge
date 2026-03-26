# Reach SPA Quality with HTMX and Twig

> **Talk Abstract**
>
> Discover how HTMX can transform the way you develop by bringing the power and
> flexibility of Single Page Applications (SPA) to your projects, without
> writing any JavaScript!

_Originally in French_
Speaker: Damien Alexandre
* [Video](https://www.youtube.com/watch?v=wjmCTfVr4Q8)
* [Related article](https://jolicode.com/blog/making-a-single-page-application-with-htmx-and-symfony)

## Recap

Damien Alexandre works at JoliCode and describes HTMX as a small JavaScript
library whose main purpose is to let you avoid writing JavaScript.

The talk is aimed at backend developers.

### The problem with modern web applications

Users expect web applications to behave like native apps: no full-page reloads
when opening an email, instant cart additions, real-time search results.
Plain HTML cannot deliver this.
HTML was designed for links and forms, full-page navigation, and nothing more.

To bridge the gap, the industry built SPAs.
But SPAs come at a cost: the backend stops returning HTML and returns JSON
instead, a JavaScript router manages navigation, a JavaScript rendering engine
replaces the browser's own, client-side state management is needed, and the
team splits into frontend and backend disciplines with an API contract between
them.

This is the thick client model: HTML becomes a rendering target, all
application behavior lives in JavaScript, and everything rests on JSON.

### The HTMX approach

HTMX takes the opposite direction.
Instead of layering a full application framework on top of HTML, it enriches
HTML itself with the capabilities it was missing.
The application state lives entirely on the server.
The backend returns HTML, not JSON.
There is no frontend layer.

This is sometimes called HATEOAS: Hypermedia as the Engine of Application State.

HTMX was created by Carson Gross in 2020 as a rewrite of Intercooler.js (2013).
It gained traction through memes.
The library has zero dependencies, ships as a single file, and weighs 50KB,
compared to jQuery at ~80KB, Unpoly at ~160KB, and Hotwire Turbo at ~200KB.

### The four problems HTMX fixes

HTML has four limitations that HTMX addresses:

1. Only `<a>` and `<form>` elements can make HTTP requests.
2. Those requests are only triggered by click and submit events.
3. Only GET and POST are supported, not PUT, DELETE, PATCH, etc.
4. Every response replaces the entire page.

### Core attributes

HTMX works through HTML attributes.
Four attributes cover the vast majority of use cases.

**`hx-get`, `hx-post`, `hx-delete`, etc.** declare the HTTP verb and the URL
for an Ajax request.
When a button carries `hx-post="/like/42"`, clicking it sends a POST request
and places the response in the DOM.

**`hx-trigger`** sets the event that fires the request.
The default is sensible: click for buttons, submit for forms.
Any JavaScript event works, including custom ones.
Modifiers add behavior: `once` fires only once, `changed` fires only when the
input value has actually changed, `delay:200ms` debounces, `every 5s` polls.

**`hx-target`** is a CSS selector pointing to the element that will receive
the response.
The response does not have to replace the element that triggered the request.
A button at the bottom of the page can update the navigation menu at the top.

**`hx-swap`** controls how the response is inserted: `innerHTML` (default),
`outerHTML`, `afterend`, `beforeend`, `afterbegin`, `beforebegin`.

### Demos

**Like button.**
A button carries `hx-post="/like/42"` and `hx-swap="outerHTML"`.
The backend saves the toggle to the session and returns only the button
template, with a different CSS class when liked.
Adding an `id` attribute to the button enables HTMX morphing, which preserves
the DOM node during replacement and allows CSS transitions to animate smoothly.
No JavaScript written.

**Live search.**
An input carries four attributes:
- `hx-get="/search"` to call the search endpoint
- `hx-trigger="input changed delay:200ms"` to debounce keystrokes
- `hx-target="#search-results"` to update only the results list
- `hx-push-url="true"` to keep the address bar in sync, so a page refresh
  restores the same results

On the backend, the `HX-Request` header (present on all HTMX requests) signals
that only the results block should be rendered, not the full page.
This avoids rendering the full layout for nothing.

**Boosted navigation.**
Adding `hx-boost="true"` to a container makes all links inside it behave like
TurboDrive: instead of a full navigation, the browser makes an Ajax request,
replaces only the `<body>`, merges any changed `<head>` elements, and updates
the history.
This is progressive enhancement: the page works without JavaScript and gains
a smoother feel with it.

**Lazy loading.**
`hx-get="/user-block"` combined with `hx-trigger="load"` fires a request the
moment the element enters the page.
This is useful for pages served from an aggressive cache: the cached page
contains no session-dependent content, and the personalized block (a greeting,
a user badge) is loaded separately on every visit.

**Infinite pagination.**
On the last item in a loop, if there is a next page, add:
- `hx-get="/emojis?page=2"` pointing to the next page
- `hx-trigger="revealed"` to fire when the element enters the viewport
- `hx-swap="afterend"` to append results below rather than replace

Scrolling to the bottom of the list automatically loads the next page.
Four attributes for infinite pagination.

**Forms.**
Adding `hx-boost="true"` to a form submits it via Ajax and merges the response
into the page without a full reload, including the PRG (Post/Redirect/Get)
pattern becoming unnecessary.
Symfony returns HTTP 422 for validation errors by default.
HTMX ignores non-2xx responses by default, so a small configuration is needed
to allow 422 responses to update the DOM:

```html
<meta name="htmx-config" content='{"responseHandling": [{"code":"422", "swap": true}]}'>
```

**Out-of-band swaps.**
A single Ajax response can update multiple independent parts of the page.
Any element in the response carrying `hx-swap-oob="true"` and matching an `id`
in the page will be swapped into that position, regardless of where the
triggering element is.
A form submission can update the form itself and a results panel elsewhere in
the layout in one round trip.

**Server-sent events.**
HTMX has an extension for server-sent events and Mercure.
Connecting a `<div>` to a Mercure hub and rendering streamed updates into it
requires two attributes.

### Tips

HTMX offers several ways to achieve the same result (boost, explicit attributes,
full-page vs partial renders).
Without agreed conventions the codebase becomes inconsistent quickly.
Teams should settle on one approach and stick to it.

HTMX emits JavaScript events at each stage of a request (before request, before
swap, after swap).
These are the hooks for any JavaScript that still needs to be written.
Enabling `htmx.logAll()` in the browser console lists every event.

PHPStorm has full HTMX support and prompts to enable it when an `hx-*` attribute
is detected.

Error handling requires a different mental model.
When a request comes from HTMX, a 404 or 500 response produces no visible
feedback, just silence from the user's perspective.
The backend should return a 200 with a human-readable error message instead.

### When to use HTMX

HTMX fits well on smaller projects, CRUD applications, backend admin panels,
and any application that is mostly forms and text.
Adding a handful of attributes to a traditional server-rendered application is
enough to make it feel noticeably more responsive.

It is the wrong tool for applications with heavy client-side interdependencies,
offline requirements, or spreadsheet-like interfaces where many elements update
simultaneously.

### Hotwire users

Developers already using Hotwire with Stimulus should not switch to HTMX.
Turbo already covers the same ground.
HTMX is the right choice for teams that are not already invested in the Hotwire
stack and want to add reactivity without committing to a full frontend layer.

### Q&A

On testing: backend response tests work as normal.
Testing that HTMX correctly swaps the right DOM element in the right place
requires a real browser, so end-to-end tests with Playwright or equivalent are
still needed for full coverage.

On why HTMX is not more widely used in production: Damien's view is that it is
difficult to adopt mid-project once a JavaScript codebase already exists.
It works best when chosen from the start.
He also mentions Unpoly as a competing library that covers both Ajax behavior
and UI interactions (modals, overlays) in a single package.

On accessibility: HTMX itself provides nothing.
The progressive enhancement approach, building accessible behavior first and
layering HTMX on top, is the right path, but the accessibility work still falls
on the developer.
