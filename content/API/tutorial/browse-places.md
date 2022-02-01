---
title: "Adding an Endpoint to Browse Places "
menuTitle: "Custom Endpoint"
date: 2022-01-30T12:14:39+01:00
tags: ["api"]
weight: 3
---

Assume you have an edition in which all references to places are tagged. The references point to a central authority file containing further information on each place. In the simplest case this might be a separate TEI document with the list of places contained in `/TEI/standoff/listPlace`. An example (taken from the *Alfred Escher Briefedition*) is attached below.

{{%attachments title="TEI Source" pattern=".*\.(xml)$"/%}}

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
                                "type": "object",
                                "properties": {
                                    "items": {
                                        "type": "array",
                                        "items": {
                                            "type": "string"
                                        }
                                    },
                                    "categories": {
                                        "type": "array",
                                        "items": {
                                            "type": "object",
                                            "properties": {
                                                "category": {
                                                    "type": "string"
                                                },
                                                "count": {
                                                    "type": "integer"
                                                }
                                            }
                                        }
                                    }
                                }
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