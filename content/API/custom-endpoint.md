---
title: "How to add a custom endpoint?"
menuTitle: "Custom Endpoints"
date: 2022-01-30T12:14:39+01:00
tags: ["api"]
---

### Introduction

Since version 7.0 of TEI Publisher, nearly all requests to the server are handled via an API (the only exception being requests for static resources like images or CSS styles). The entire API is expressed in a formal specification based on the [Open API standard](http://spec.openapis.org/oas/v3.0.3). Within TEI Publisher you can browse the API by opening the `Documentation/TEI Publisher API` page from the menubar. Here you'll see a list of URL paths (or *API endpoints* as we often call them).

You can use the API documentation page to directly try out the endpoints or use the provided information to construct your own URLs. For example, if we look at the very first endpoint on the page, `/api/document/{id}`, we can see that it expects a single required parameter called `id`. Usually parameters are either specified as normal HTTP request parameters (written after the `?` in an URL for a GET request) or within the URL path itself. The `id` parameter is defined as a **path** parameter and it is referenced in the URL path accordingly (the bit in curly brackets: `{id}`).

![API documentation](/images/api-documents.png)

Now let's assume we would like to call this endpoint to retrieve the source XML of Graves' letter, which resides under `data/test/graves6.xml` in a default TEI Publisher install. The `id` parameter should be relative to the data root collection of TEI Publisher, thus `test/graves6.xml`. The Open API specification requires that path parameters are URL-encoded, so the / in the path should be encoded as `%2F`.

Therefore the URL path to use would become `/api/document/test%2Fgraves6.xml`. Now we just prepend the correct server address pointing to the root of TEI Publisher to get a full URL which we can paste into the browser's location bar, e.g. for our local install: <http://localhost:8080/exist/apps/tei-publisher/api/document/test%2Fgraves6.xml>.

If instead of retrieving the raw XML we would like to see the HTML output (after applying an ODD), we can use the endpoint `/api/document/{id}/html`. This requires an additional parameter to specify the ODD, so the full URL on localhost would become: <http://localhost:8080/exist/apps/tei-publisher/api/document/test%2Fgraves6.xml/html?odd=graves.odd>.

### Organization of the API

As can be seen on the API documentation page, the different endpoints are grouped into categories ("tags") like *documents*, *collection*, *odd* etc. This grouping is just for your convenience and has no technical implications. You'll also find that there is a separate section at the bottom, called *Custom API*. This is specifically for endpoints you want to add on top of TEI Publisher's default endpoints. We'll use this below.

More important is another distinction: looking at the endpoint paths we see two kinds: those starting with `/api` and those without prefix, which all appear under the *view* tag. In general we can say that URLs **without** `/api` prefix are intended to be **directly called** by the user when entering a URL into the browser or navigating a link, while those with `/api` prefix are usually not exposed to the user but instead consumed by other code components.

To illustrate this with an example: if you navigate to `/exist/apps/tei-publisher/doc/documentation.xml` in your browser, the endpoint called will be `/{docId}`. This endpoint inspects the requested document and determines that it should be displayed using the HTML template `documentation.html` (residing in `templates/pages/documentation.html`). It thus loads this template, expands it through eXist's HTML templating (which does things like e.g. including the menubar and toolbar), and returns it to the browser.

![API Request Flow](/images/api-flow.png)

The browser will start to render the received HTML. While doing so, it finds a bunch of custom elements (= *webcomponents*) and activates them. Among those webcomponents is the central `pb-view` on the page, which is supposed to display the actual documentation content. When it gets activated, it will send another request to the TEI Publisher API, which would say: "get me the next division of the documentation as HTML transformed via ODD". This second request is triggered by the component (not the user directly) and thus starts with the `/api` prefix (in concrete it will go to `/api/parts/doc%2Fdocumentation.xml/html`).

To summarize, we can distinguish between

1. user-facing endpoints, which produce HTML content to be viewed in a browser
2. machine-processable API endpoints providing information to be consumed by components or other services

In the next section we learn how to create an endpoint to be used by a webcomponent, thus falling under category (2).

### Adding an Endpoint to Browse Places

Assume you have an edition in which all references to places are tagged. The references point to a central authority file containing further information on each place. In the simplest case this might be a separate TEI document with the list of places contained in `/TEI/standoff/listPlace`.

As an alternative entry point into the edition, we may want to provide users a page on which they can browse through all place names and see their location on a map. Since there might a large number of places, we ideally want to group them by first letter, plus provide a search feature for filtering. Fortunately, the `tei-publisher-components` library (since version 1.33.0) provides a webcomponent for this purpose: `pb-split-list`.

The component will retrieve the information to display from an API endpoint. This endpoint should return a JSON object with

1. the grouping categories to be used (e.g. letters of the alphabet) along with an item count for each category
2. the items to be shown in the currently selected category

The returned JSON record may look like this:

```json
{
    "items": [
        "<span class=\"place\"><a href=\"Aix-les-Bains (F)?category=A&amp;search=le\">Aix-les-Bains (F)</a></span>"
    ],
    "categories": [
        {
            "category": "A",
            "count": 1
        }
        {
            "category": "C",
            "count": 1
        },
        {
            "category": "D",
            "count": 2
        },
        // ... more ...
        {
            "category": "All",
            "count": 54
        }
    ]
}
```

This example has only one place to be shown under the currently selected category (letter 'A'): "Aix-les-Bains".

Our first task now is to implement an API endpoint which returns a JSON record as shown above. For this we have to

1. provide a formal definition of the API endpoint in `modules/custom-api.json`
2. write the actual XQuery function which returns the required data

We won't dive into the Open API standard here. Important to note is just that TEI Publisher uses JSON instead of YAML for the specification. For a more detailed explanation of Open API 3, refer to the [tutorial](https://support.smartbear.com/swaggerhub/docs/tutorials/openapi-3-tutorial.html).

In `modules/custom-api.json`, insert the following definition into the `paths` object:

```json
"paths": {
    "/api/places": {
        "get": {
            "summary": "List places",
            "description": "Retrieve list of places in format required by pb-split-list",
            "operationId": "custom:places",
            "parameters": [
                {
                    "name": "category",
                    "in": "query",
                    "schema": {
                        "type": "string",
                        "example": "A"
                    }
                },
                {
                    "name": "limit",
                    "in": "query",
                    "schema": {
                        "type": "integer",
                        "default": 50
                    }
                },
                {
                    "name": "search",
                    "in": "query",
                    "schema":{
                        "type": "string"
                    }
                }
            ],
            "responses": {
                "200": {
                    "description": "Categories and places to display",
                    "content": {
                        "application/json": {
                            "schema":{
                                "type": "object"
                            }
                        }
                    }
                }
            }
        }
    },
    // ... existing definitions
}
```

{{% notice info %}}
TEI Publisher splits the Open API definition into two parts: `modules/lib/api.json` and `modules/custom-api.json`. The first should **never** be modified. When you upgrade TEI Publisher, this file will likely be replaced by a newer version. You are allowed to **overwrite** definitions found in `modules/lib/api.json` within `modules/custom-api.json` though, see the chapter on [Can I use a custom table of contents?](/templates/custom-toc).
{{% /notice %}}

If you reload the API documentation page, you'll already see the new endpoint popping up in the list. However, trying to call it will result in an error: "Function custom:places could not be resolved". We can deduce from this that the underlying library is searching for a function `custom:places` but couldn't find it. The name `custom:places` in fact is what we defined in the `operationId` property of the specification.

Let's implement this function then. As you may have guessed already, it should go into `modules/custom-api.xql`. We first need to declare the TEI namespace and add an import for the `config` module. Paste the following two lines before the first import:

```xquery
declare namespace tei="http://www.tei-c.org/ns/1.0";

import module namespace config="http://www.tei-c.org/tei-simple/config" at "config.xqm";
```

Next we add two functions to the file:

```xquery
declare function api:places($request as map(*)) {
    let $search := normalize-space($request?parameters?search)
    let $letterParam := $request?parameters?category
    let $limit := $request?parameters?limit
    let $places :=
        if ($search and $search != '') then
            doc($config:data-root || "/playground/places.xml")//tei:listPlace/tei:place[matches(@n, "^" || $search, "i")]
        else
            doc($config:data-root || "/playground/places.xml")//tei:listPlace/tei:place
    let $sorted := sort($places, "?lang=de-DE", function($place) { lower-case($place/@n) })
    let $letter := 
        if (count($places) < $limit) then 
            "Alle"
        else if ($letterParam = '') then
            substring($sorted[1], 1, 1) => upper-case()
        else
            $letterParam
    let $byLetter :=
        if ($letter = 'Alle') then
            $sorted
        else
            filter($sorted, function($entry) {
                starts-with(lower-case($entry/@n), lower-case($letter))
            })
    return
        map {
            "items": api:output-place($byLetter, $letter, $search),
            "categories":
                if (count($places) < $limit) then
                    []
                else array {
                    for $index in 1 to string-length('ABCDEFGHIJKLMNOPQRSTUVWXYZ')
                    let $alpha := substring('ABCDEFGHIJKLMNOPQRSTUVWXYZ', $index, 1)
                    let $hits := count(filter($sorted, function($entry) { starts-with(lower-case($entry/@n), lower-case($alpha))}))
                    where $hits > 0
                    return
                        map {
                            "category": $alpha,
                            "count": $hits
                        },
                    map {
                        "category": "Alle",
                        "count": count($sorted)
                    }
                }
        }
};

declare function api:output-place($list, $category as xs:string, $search as xs:string?) {
    array {
        for $place in $list
        let $categoryParam := if ($category = "all") then substring($place/@n, 1, 1) else $category
        let $params := "category=" || $categoryParam || "&amp;search=" || $search
        let $label := $place/@n/string()
        return
            <span class="place">
                <a href="{$label}?{$params}">{$label}</a>
            </span>
    }
};
```

If we now again test the API endpoint via the API documentation page, we should see the expected JSON output. We can also play around with the parameters, e.g. filter by name prefix via parameter `search`.

### Combining Endpoint and Webcomponent

We now have the endpoint ready, so the next step is to actually create an HTML page and plug it in. First, check if you have the [correct version](/webcomponents/version-upgrade) of the tei-publisher-components library (> 1.32.0).

Next, create a file below `templates`, e.g. `templates/places.html` and paste in the following HTML:

```html
<?xml version="1.0" encoding="UTF-8"?>
<html>
    <head>
        <meta charset="utf-8"/>
        <meta name="viewport" content="width=device-width, minimum-scale=1, initial-scale=1, user-scalable=yes"/>
        <link rel="shortcut icon" type="image/x-icon" href="resources/images/favicon.ico"/>
        <link rel="shortcut icon" type="image/png" href="resources/images/favicon-16.png" sizes="16x16"/>
        <link rel="shortcut icon" type="image/png" href="resources/images/favicon-24.png" sizes="24x24"/>
        <link rel="shortcut icon" type="image/png" href="resources/images/favicon-32.png" sizes="32x32"/>
        <link rel="shortcut icon" type="image/png" href="resources/images/favicon-64.png" sizes="64x64"/>

        <title data-template="config:app-title"></title>
        <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Oswald"/>
        <link rel="stylesheet" href="resources/css/theme.css" />
        <script type="module" src="pb-components-bundle.js" data-template="pages:load-components"></script>
        <style>
            main {
                max-width: 41rem;
                margin: 0 auto;
            }
        </style>
    </head>
    <body>
        <pb-page data-template="pages:pb-page" unresolved="unresolved">
            <app-header-layout>
                <app-header slot="header" reveals="reveals" fixed="fixed" effects="waterfall">
                    <app-toolbar class="menubar">
                        <a href="${app}/index.html" class="logo" data-template="lib:parse-params"></a>
                    </app-toolbar>
                </app-header>
                <main>
                    <pb-custom-form id="options" auto-submit="paper-input,paper-icon-button">
                        <paper-input id="query" name="search" label="Suche">
                            <paper-icon-button icon="search" slot="suffix"></paper-icon-button>
                        </paper-input>
                    </pb-custom-form>
                    <pb-split-list url="api/places" subforms="#options" selected="A"></pb-split-list>
                </main>
            </app-header-layout>
        </pb-page>
    </body>
</html>
```

Navigate to the new page by entering [/tei-publisher/places.html](http://localhost:8080/exist/apps/tei-publisher/places.html):

![places.html](/images/places.png)

### Registering a different URL

Right now the user needs to navigate to `places.html` to get to our new page. For convenience, we may want to publish this page under a different URL, e.g. `/tei-publisher/contexts/places`. We could thus register another endpoint in our API definition to map `/contexts/places` to the `places.html` template. We can re-use an existing API function for this, so we don't need to write any XQuery. The existing function is called `vapi:html`. Add the following endpoint definition to `custom-api.json`:

```json
"/contexts/places": {
    "get": {
        "summary": "Landing page for places",
        "description": "Get the HTML template for the places landing page",
        "tags": ["view"],
        "operationId": "vapi:html",
        "x-error-handler": "vapi:handle-error",
        "parameters": [
            {
                "name": "file",
                "in": "query",
                "schema": {
                    "type": "string",
                    "default": "places"
                }
            }
        ],
        "responses": {
            "200": {
                "description": "HTML view for the document",
                "content": {
                    "text/html": {
                        "schema": {
                            "type": "string"
                        }
                    }
                }
            },
            "404": {
                "description": "The document was not found",
                "content": {
                    "text/html": {
                        "schema": {
                            "type": "string"
                        }
                    }
                }
            },
            "500": {
                "description": "An error occurred",
                "content": {
                    "text/html": {
                        "schema": {
                            "type": "string"
                        }
                    }
                }
            }
        }
    }
}
```

The main trick here is to hardcode the `file` parameter to a fixed value by defining a default: `places`. The `vapi:html` function will automatically try to locate this file in the `templates` collection by appending `.html`.

{{% notice info %}}
To learn about other API functions used by TEI Publisher, it's best to have a look at `modules/lib/api.json`.
{{% /notice %}}

After saving the changes, we should be able to access the places landing page via an URL like [/tei-publisher/contexts/places](http://localhost:8080/exist/apps/tei-publisher/contexts/places).

### Showing Places on a Map

In addition to browsing through the places, it would also be nice to display their location on a map. Ideally the map should display all places occurring in the edition, so we can easily see which places are geographically close. We would thus need an additional endpoint, returning a simple list of all places with their geographical coordinates. Let's add an endpoint for `/places/all` above our existing endpoint for `/places`:

```json
"/api/places/all": {
    "get": {
        "summary": "List all places",
        "description": "Retrieve list of all places",
        "operationId": "custom:places-all",
        "responses": {
            "200": {
                "description": "List of all places",
                "content": {
                    "application/json": {
                        "schema":{
                            "type": "array"
                        }
                    }
                }
            }
        }
    }			
}
```

The implementation of `custom:places-all` is straightforward: add the following function to `modules/custom-api.xql`:

```xquery
declare function api:places-all($request as map(*)) {
    let $places := doc($config:data-root || "/playground/places.xml")//tei:listPlace/tei:place
    return 
        array { 
            for $place in $places
                let $tokenized := tokenize($place/tei:location/tei:geo)
                return 
                    map {
                        "latitude":$tokenized[1],
                        "longitude":$tokenized[2],
                        "label":$place/@n/string()
                    }
            }        
};
```

Now we can extend the HTML template, `places.html` to add a map component:

1. import the javascript library for the map component below the last `<script>` in the header:

    ```html
    <script type="module" src="pb-leaflet-map.js" data-template="pages:load-components"/>
    ```
2. into the CSS `<style>` section, add a rule to size the map (otherwise it will be 0 height):
    ```css
    pb-leaflet-map {
        height: 40vh;
        width: 100%;
    }
    ```
3. add the HTML for the map component before the `<main>` element:
    ```html
    <pb-leaflet-map id="map" subscribe="map" emit="map" zoom="10" cluster="" latitude="47.3686498" longitude="8.5391825">
        <pb-map-layer show=""
            base="" 
            label="Mapbox OSM"                                 
            url="https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token={accessToken}" 
            max-zoom="19" 
            access-token="pk.eyJ1Ijoid29sZmdhbmdtbSIsImEiOiJjam1kMjVpMnUwNm9wM3JwMzdsNGhhcnZ0In0.v65crewF-dkNsPF3o1Q4uw" 
            attribution="© Mapbox © OpenStreetMap">
        </pb-map-layer>
    </pb-leaflet-map>
    ```
4. finally we need a bit of javascript before the closing `</body>`:
   ```javascript
   <script>
        window.addEventListener('WebComponentsReady', function() {
            pbEvents.subscribe('pb-page-ready', null, function() {
                const endpoint = document.querySelector("pb-page").getEndpoint();
                const url = `${endpoint}/api/places/all`;
                console.log(`fetching places from: ${url}`);
                fetch(url)                
                .then(function(response) {
                    return response.json();
                })
                .then(function(json) {
                    pbEvents.emit("pb-update-map", "map", json)
                });
            });
        });
    </script>
    ```
    This script waits until webcomponents have been loaded and fully initialized (`pb-page-ready` event). It then uses the browser's [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) function to retrieve the list of places from our endpoint, and sends the resulting JSON array via a `pb-update-map` event to the map for display.

![Map display](/images/map.png)

One last feature is still missing though: the places listed in the browsing view are not connected to the map. Would be nice if we had an icon next to each place name to click on and have the map zoom to the corresponding coordinates. To make this happen, replace the function `api:output-place` in `modules/custom-api.xql` with the following version:

```xquery
    declare function api:output-place($list, $category as xs:string, $search as xs:string?) {
    array {
        for $place in $list
        let $categoryParam := if ($category = "all") then substring($place/@n, 1, 1) else $category
        let $params := "category=" || $categoryParam || "&amp;search=" || $search
        let $label := $place/@n/string()
        let $coords := tokenize($place/tei:location/tei:geo)
        return
            <span class="place">
                <a href="{$label}?{$params}">{$label}</a>
                <pb-geolocation latitude="{$coords[1]}" longitude="{$coords[2]}" label="{$label}" emit="map" event="click">
                    <iron-icon icon="maps:map"></iron-icon>
                </pb-geolocation>
            </span>
    }
};
```

This will add a little map icon next to each place. The `pb-geolocation` component around the icon creates a link, which - when clicked - instructs the map to zoom to the given coordinates.