# FVD.Patterns.API.Demo

A minimal, dependency-free example of consuming the **FVD Patterns API** from a web page.

- API description: <https://pat4.api.fvd.bz/swagger>
- OpenAPI spec (v4): <https://pat4.api.fvd.bz/openapi/v4.json>

## Demo

[`src/simple/index.html`](src/simple/index.html) is a single static page (plain HTML, CSS and JavaScript) that lets a user pick
**Year → Make → Model → Trim** from cascading dropdowns and then lists the matching vinyl patterns with a preview
image, covered vehicles, square footage, difficulty and price.

## Configuration

Edit the two variables at the top of the `<script>` block in `index.html`:

```js
var API = "https://pat4.api.fvd.bz/";  // base URL of the Patterns API deployment
var KEY = "YOUR-API-KEY";                       // API key issued for your domain
```

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
| `GET /models/{makeId}/{year}` | Models of a make for a year |
| `GET /trims/{modelId}/{year}` | Trims of a model for a year |
| `GET /patterns/{modelid}/{trimid}/{year}` | Patterns for the selected vehicle |
| `GET /patterns/preview/{id}/{w}/{h}` | Preview image (no API key; `id` is the short-lived `PatternID` token) |

The API also exposes `GET /types`, `GET /patterndetails/{patternId}` and a `GET /` health check — see the spec for details.

Data responses are wrapped in an envelope `{ Success, Data, Errors }`; access errors (401/403) are returned as plain text.

> [!WARNING]
> **`PatternID` is a protected, time-limited token, not a permanent identifier.** Every `PatternID` returned by
> `/patterns/...` expires after a short time. Never store it on the client side (`localStorage`, `sessionStorage`,
> cookies, IndexedDB, bookmarks or cached URLs) and never reuse it across sessions. Use it right away for
> `/patterndetails/{patternId}` or `/patterns/preview/...`, and request the pattern list again to get a fresh token.
