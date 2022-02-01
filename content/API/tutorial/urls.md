---
title: "Registering a Different URL"
menuTitle: "URL Handling"
date: 2022-01-30T12:14:39+01:00
tags: ["api"]
weight: 5
---

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