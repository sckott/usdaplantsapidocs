---
title: API Reference

language_tabs:
  - http
  - shell
  - r
  - ruby

toc_footers:
  - <a href='http://plants.usda.gov/java/'>USDA Plants Database</a>
  - <a href='https://plantsdb.xyz'>plantsdb API</a>
  - <a href='https://github.com/sckott/usdaplantsapidocs'>Contribute to these docs</a>
  - <a href='http://recology.info/usdaplantsapistatus/'>API Status!</a>

search: true
---

# Introduction

Welcome to the USDA plants database API!

We have examples for Shell (using `curl` and `jq`), R, and Ruby! You can view code examples in the dark area to the right (or below if on mobile), and you can switch the programming language of the examples with the tabs in the top right.

<aside class="notice">
We're still in the process of filling out examples ...
</aside>

<aside class="warning">
Check on the status of the API at our <a href="http://recology.info/usdaplantsapistatus">status page</a>
</aside>

# Base URL

<https://plantsdb.xyz>

# HTTP methods

This is a `read only` API. That is, we only allow `GET` (and `HEAD`) requests on this API.

Requests of all other types will be rejected with appropriate `405` code, including `POST`, `PUT`, `COPY`, `HEAD`, `DELETE`, etc.

# Response Codes

* 200 (OK) - request good!
* 302 (Found) - the root `/`, redirects to `/heartbeat`, and `/docs` redirects to these documents
* 400 (Bad request) - When you have a malformed request, fix it and try again
* 404 (Not found) - When you request a route that does not exist, fix it and try again
* 405 (Method not allowed) - When you use a prohibited HTTP method (we only allow `GET` and `HEAD`)
* 500 (Internal server error) - Server got itself in trouble; get in touch with us. (in [Issues](https://github.com/ropensci/fishbaseapi/issues))

> `400` responses will look something like

```
HTTP/1.1 400 Bad Request
Access-Control-Allow-Methods: HEAD, GET
Access-Control-Allow-Origin: *
Cache-Control: public, must-revalidate, max-age=60
Content-Length: 85
Content-Type: application/json; charset=utf8
Date: Wed, 13 Jul 2016 22:50:55 GMT
Server: Caddy
Status: 400 Bad Request
X-Content-Type-Options: nosniff

{
  "count": 0,
  "returned": 0,
  "data": null,
  "error": {
    "message": "limit too large (max 5000)"
  }
}
```

> `404` responses will look something like

```
HTTP/1.1 404 Not Found
Access-Control-Allow-Methods: HEAD, GET
Access-Control-Allow-Origin: *
Cache-Control: public, must-revalidate, max-age=60
Content-Length: 27
Content-Type: application/json; charset=utf8
Date: Wed, 13 Jul 2016 22:51:30 GMT
Server: Caddy
Status: 404 Not Found
X-Cascade: pass
X-Content-Type-Options: nosniff

{
    "error": "route not found"
}
```

> `405` responses will look something like (with an empty body)

```
HTTP/1.1 405 Method Not Allowed
Access-Control-Allow-Methods: HEAD, GET
Access-Control-Allow-Origin: *
Cache-Control: public, must-revalidate, max-age=60
Content-Length: 0
Content-Type: application/json; charset=utf8
Date: Wed, 13 Jul 2016 22:52:04 GMT
Server: Caddy
Status: 405 Method Not Allowed
X-Content-Type-Options: nosniff
```

> `500` responses will look something like

```
HTTP/1.1 500 Internal Server Error
Cache-Control: public, must-revalidate, max-age=60
Connection: close
Content-Length: 24
Content-Type: application/json
Date: Wed, 13 Jul 2016 22:52:04 GMT
Server: Caddy
Status: 500 Internal Server Error
X-Content-Type-Options: nosniff

{
    "error": "server error"
}
```

Occassionally you may get a MySQL error message in your `error` slot. Get in touch
with us if that happens as that shouldn't be happening, and we'll need to fix
something on our end.

# Response headers

Response headers give the following noteable things:

* allowed methods on a route
* allowed origins
* cache information
* body content length
* content type (JSON)
* date
* server info
* http status code

> `200` response header will look something like

```
HTTP/1.1 200 OK
Access-Control-Allow-Methods: HEAD, GET
Access-Control-Allow-Origin: *
Cache-Control: public, must-revalidate, max-age=60
Content-Length: 3389
Content-Type: application/json; charset=utf8
Date: Wed, 13 Jul 2016 22:53:10 GMT
Server: Caddy
Status: 200 OK
X-Content-Type-Options: nosniff
```

# Response bodies

> Response bodies generally look like:

```json
{
    "count": 1,
    "returned": 1,
    "data": [
      {
        "id": 1,
        "Symbol": "ABAB",
        "Accepted_Symbol_x": "ABAB",
        "Synonym_Symbol_x": "",
        "Scientific_Name_x": "Abutilon abutiloides (Jacq.) Garcke ex Hochr."
      }
    ],
    "error": null
}
```

Responses have 4 slots:

* count: Number records found
* returned: Number records returned
* error: If an error did not occur this is `null`, otherwise, an error message.
* data: The hash of data if any data returned. If no data found, this is an empty hash (hash of length zero)

Responses with errors also have the same 4 slots, but no data returned, and there should be an
informative error message.

# Media Types

We serve up only JSON in this API. All responses will have `Content-Type: application/json; charset=utf8`.

# Pagination

The query parameters `limit` (default = 10) and `offset` (default = 0) can be used
to toggle the number of results you get back and what record you start at. This
doesn't apply to some routes, which don't accept any parameters (e.g., [docs](#docs)).

The response body from the server will include data on records found in `count` and number returned in `returned`:

* `"count": 1056`
* `"returned": 10`

# Authentication

We don't require any. Cheers :)

# Parameters

## Common parameters

+ limit (integer, optional) `number` of records to return.
    + Default: `10`
+ offset (integer, optional) Record `number` to start at.
    + Default: `0`
+ fields (string, optional) Comma-separated `string` of fieds to return.
    + Example: `Genus,Species`

Above parameters common to all routes except:

* [root](#root)
* [heartbeat](#heartbeat)
* [docs](#docs)

## Additional parameters

Right now, any field that is returned from a route can also be queried on. All of the fields from each route are too long to list here - inspect data returned from a small data request, then change your query as desired.

Right now, parameters that are not found are silently dropped. For example, if you query with `/species?foo=bar` in a query, and `foo` is not a field in `species` route, then the `foo=bar` part is ignored. We may in the future error when parameters are not found.

# Routes

<aside class="notice">
Shell examples below using <code>curl</code> sometimes use the command line tool <code>jq</code>,
a tool to parse JSON. If you don't want to use <code>jq</code> - just remove the pipe and
everything after it. If you do, see <a href="https://stedolan.github.io/jq/">stedolan.github.io/jq</a>
for installation and usage.
</aside>

## Root

`GET https://plantsdb.xyz`

```shell
curl -L "https://plantsdb.xyz"
```

Redirects to `https://plantsdb.xyz/heartbeat`

<aside class="notice">
Note that when using <code>curl</code> you need to use the <code>-L</code> flag
to follow redirects.
</aside>

## Heartbeat

Lists all the routes, and gives some indicaion of what can be passed to them.

```shell
curl "https://plantsdb.xyz/heartbeat" |  jq .
```

The above command returns JSON structured like this:

```shell
{
  "routes": [
    "/search (HEAD, GET)",
    "/heartbeat"
  ]
}
```

```r
library("request")

'https://plantsdb.xyz/heartbeat' %>% api()
```

```r
$routes
[1] "/search (HEAD, GET)" "/heartbeat"
```

```ruby
require 'faraday'
require 'multi_json'

conn = Faraday.new(url: 'https://plantsdb.xyz')
res = conn.get 'heartbeat'
MultiJson.load(res.body)
```

The above command returns a Hash like:

```ruby
=> {"routes"=>["/search (HEAD, GET)", "/heartbeat"]}
```

## docs

> GET [/docs]

Get brief description of each table in the Fishbase database.

+ Response 200 (application/json)
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## search

> GET [/search{?limit}{?offset}{?fields}]

Search for plants species

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://plantsdb.xyz/search?limit=5" |  jq .
```

> The above command returns JSON structured like this:

```json
{
  "routes": [
    "/docs/:table?",
    "/heartbeat",
    "/mysqlping",
    "/listfields",
    "/comnames?<params>",
    "/country?<params>"
  ]
}
```

```r
library("request")

res <- api("https://plantsdb.xyz") %>% api_path(search) %>% api_query(limit = 5)
tibble::as_data_frame(res$data)
```

> returns

```r
# A tibble: 5 x 134
     id Symbol Accepted_Symbol_x Synonym_Symbol_x
* <int>  <chr>             <chr>            <chr>
1     1   ABAB              ABAB
2     2  ABAB2             ABPR3            ABAB2
3     3  ABAB3              ABTH            ABAB3
4     4 ABAB70            ABAB70
5     5   ABAC             ABUMB             ABAC
# ... with 130 more variables: Scientific_Name_x <chr>,
#   Hybrid_Genus_Indicator <chr>, Hybrid_Species_Indicator <chr>,
#   Species <chr>, Subspecies_Prefix <chr>, Hybrid_Subspecies_Indicator <chr>,
#   Subspecies <chr>, Variety_Prefix <chr>, Hybrid_Variety_Indicator <chr>,
#   Variety <chr>, Subvariety_Prefix <chr>, Subvariety <chr>,
#   Forma_Prefix <chr>, Forma <chr>, Genera_Binomial_Author <chr> ...
```

```ruby
require 'faraday'
require 'multi_json'

conn = Faraday.new(url: 'https://plantsdb.xyz')
res = conn.get 'search', {"limit": 5}
dat = MultiJson.load(res.body)
dat['data'].collect{ |x| x['Genus']}
```

> The above command returns a Hash like:

```ruby
=> ["Abutilon", "Abrus", "Abutilon", "Abietinella", "Abronia"]
```
