# web

Batteries-included HTTP for Wyn. A thin, obvious layer over the native
`Http` builtins: request parsing, JSON/HTML/text responses, safe static
files, and HTML page building. Concurrency is the language's: `spawn` one
handler per request and every handler runs on the coroutine scheduler.

## Install

```bash
wyn pkg add web        # resolves to github.com/wynlang/web
```

## A complete server

```wyn
import web

fn handle(req: string) {
    if web.is(req, "GET", "/") == 1 {
        web.html(req, 200, web.page("Hello", "<h1>Hello from Wyn</h1>", ""))
    } else if web.is(req, "GET", "/api/greet") == 1 {
        var name = web.param(req, "name")
        if name.len() == 0 { name = "world" }
        var out = "{\"greeting\":\"hello, "
        out = out + web.json_escape(name)
        out = out + "\"}"
        web.json(req, 200, out)
    } else if web.is_under(req, "GET", "/static/") == 1 {
        web.serve_static(req, "/static/", "public")
    } else {
        web.not_found(req)
    }
}

fn main() {
    var server = web.listen(8080)
    println("http://localhost:8080")
    while true {
        var req = web.accept(server)
        if web.fd(req) > 0 { spawn handle(req) }
    }
}
```

## API

**Server** — `listen(port) -> int`, `accept(server) -> string` (blocks; returns
the request handle to pass to your spawned handler).

**Request** — `method(req)`, `path(req)` (query stripped), `query(req)`,
`param(req, name)`, `body(req)`, `fd(req)`, `is(req, method, path)`,
`is_under(req, method, prefix)`.

**Responses** — `html/json/text(req, status, content)`, `not_found(req)`,
`method_not_allowed(req)`, `bad_request(req, message)` (message JSON-escaped),
`redirect(req, location)`.

**Static files** — `serve_static(req, prefix, dir)`: traversal-safe (`..`
rejected), MIME from extension via `mime_type(path)`, `index.html` default.

**Building output** — `page(title, body_html, extra_head)` (complete styled
HTML5 page; title escaped), `escape(s)` (HTML), `json_escape(s)`.

## Test

```bash
wyn test
```

## Notes

The request is the runtime's packed `METHOD|PATH|BODY|FD` string; accessors
parse what they need. When cross-module struct support lands in the compiler,
this grows a real `Request` type without breaking the function surface.
