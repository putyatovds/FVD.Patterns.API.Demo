# FVD.Patterns.API.Demo

A minimal, dependency-free example of consuming the **FVD Patterns API** from a web page.

| Environment | Base URL | API description | OpenAPI spec (v4) |
| --- | --- | --- | --- |
| Production | `https://pat4.api.fvd.bz/` | <https://pat4.api.fvd.bz/swagger> | <https://pat4.api.fvd.bz/openapi/v4.json> |
| Staging | `https://staging-pat4.api.fvd.bz/` | <https://staging-pat4.api.fvd.bz/swagger> | <https://staging-pat4.api.fvd.bz/openapi/v4.json> |

## Demo

[`src/simple/index.html`](src/simple/index.html) is a single static page (plain HTML, CSS and JavaScript) that lets a user pick
**Year → Make → Model → Trim** from cascading dropdowns and then lists the matching vinyl patterns with a preview
image, covered vehicles, square footage, difficulty and price.

## How to get access to API

Request access by sending the following information to [api@fvd.bz](mailto:api@fvd.bz):

1. License key number.
2. List of domains/IPs you will be using to make API calls from.
3. Brief description of your project.

You will receive the API key in a reply email.

## Configuration

Edit the two variables at the top of the `<script>` block in `index.html`:

```js
// switch to "https://pat4.api.fvd.bz/" for production
var API = "https://staging-pat4.api.fvd.bz/";   // base URL of the Patterns API
var KEY = "YOUR-API-KEY";                       // API key issued for your domain
```

The demo points at the **staging** slot by default; change `API` to the production URL before going live.

The key is sent in the `fvd-patterns-api-key` header. It is validated together with the request origin, so the page
only works when served from a host that is allowed for that key.

## Running

Serve the `src/simple` folder from an allowed host, for example locally (if `localhost` is allowed for your key):

```sh
npx serve src/simple
```

Opening the file directly via `file://` will not work, because the request origin will not match an allowed host.

## Endpoints used

| Endpoint | Purpose |
| --- | --- |
| `GET /years/{modelid}/{trimid}` | Years with patterns (`0/0` = all) |
| `GET /makes/{year}` | Makes for a year |
| `GET /models/{id}/{year}` | Models of a make (`id` = make ID) for a year |
| `GET /trims/{id}/{year}` | Trims of a model (`id` = model ID) for a year |
| `GET /patterns/{modelid}/{trimid}/{year}` | Patterns for the selected vehicle |
| `GET /patterns/preview/{id}/{size}` | Preview image (no API key; `id` is the short-lived `PatternID` token) |

The preview `size` is a preset, not a width/height pair:

| `size` | Dimensions |
| --- | --- |
| `1` | 360 × 270 |
| `2` | 640 × 480 |
| `3` | 800 × 600 |

All presets are 4:3, matching the demo's thumbnail aspect ratio. The demo uses size `1`.

The API also exposes `GET /types` (pattern types available to your key), `GET /patterndetails/{patternId}` (a single
pattern) and a `GET /` health check (no API key; returns `{ message, version }`) — see the spec for details.

Data responses are wrapped in an envelope `{ Success, Data, Host, Errors }`; access errors (401/403) are returned as
plain text (e.g. `Api key is null`).

> [!WARNING]
> **`PatternID` is a protected, time-limited token, not a permanent identifier.** Every `PatternID` returned by
> `/patterns/...` expires after a short time. Never store it on the client side (`localStorage`, `sessionStorage`,
> cookies, IndexedDB, bookmarks or cached URLs) and never reuse it across sessions. Use it right away for
> `/patterndetails/{patternId}` or `/patterns/preview/...`, and request the pattern list again to get a fresh token.
