---
title: "Can I use a custom table of contents?"
menuTitle: "Custom TOC"
date: 2020-12-18T17:29:19+01:00
tags: ["templates", "open api", "xquery"]
---

By default, the table of contents shown in TEI Publisher is generated per document. In some cases this does not make much sense, e.g. for an edition of letters without internal structure. Fortunately it is not too difficult to replace the code which generates the ToC with your own, assuming you have a basic understanding of XQuery.

The following applies to TEI Publisher 7, which provides a clear structure for custom code like this. Publisher 7 uses the Open API standard to describe how server-side functionality can be called by clients.

{{% notice note %}}
When we talk about the *client* in this context, we refer to the browser, while *server* means the TEI Publisher instance this client is talking to. In Publisher 7, both communicate via a properly documented API.
{{% /notice %}}

For our custom table of contents, we first need to expose an endpoint in the API, which - when called - returns the HTML to be shown to the user. TEI Publisher's [default API](https://teipublisher.com/exist/apps/tei-publisher/api.html) should not be changed. Custom endpoints should instead go into:

1. `modules/custom-api.json` - containing the formal definition and documentation of the API
2. `modules/custom-api.xql` - into which we add the actual code as an XQuery function

### Changing the Open API definition

Custom endpoints may overwrite default endpoints, so the easiest approach for the table of contents is to simply copy the corresponding definition from the standard API into `modules/custom-api.json`. The route for the ToC is `/api/document/{id}/contents`, which means that it would handle requests to e.g. `http://localhost:8080/exist/apps/tei-publisher/api/document/macbeth.xml/contents`.

Just open `modules/lib/api.json`, search for `/api/document/{id}/contents` and copy the whole entry into the `paths` object in your `modules/custom-api.json`:

```json
"paths": {
    "/api/document/{id}/contents": {
        "get": {
            "summary": "Retrieve a formatted table of contents for the document",
            "tags": [
                "documents"
            ],
            "operationId": "custom:table-of-contents",
            "parameters": [
                {
                    "name": "id",
                    "in": "path",
                    "required": true,
                    "schema": {
                        "type": "string",
                        "example": "test/kant_rvernunft_1781.TEI-P5.xml"
                    }
                },
                {
                    "name": "view",
                    "in": "query",
                    "schema": {
                        "type": "string",
                        "enum": [
                            "page",
                            "div",
                            "single"
                        ]
                    },
                    "example": "div",
                    "description": "The view type used by the main view which displays the document, e.g. 'page' or 'div'. This has an influence on the generated links, which need to differ when linking to a page rather than a section."
                },
                {
                    "name": "target",
                    "in": "query",
                    "schema": {
                        "type": "string"
                    },
                    "description": "The target channel into which link selection events should be send (if the user clicks on a link)"
                },
                {
                    "name": "icons",
                    "in": "query",
                    "schema": {
                        "type": "boolean",
                        "default": true
                    },
                    "description": "Should an expand/collapse icon be displayed next to headings having nested child sections?"
                }
            ],
            "responses": {
                "200": {
                    "description": "Returns the formatted table of contents as HTML",
                    "content": {
                        "text/html": {
                            "schema": {
                                "type": "string"
                            }
                        }
                    }
                },
                "404": {
                    "description": "Document not found",
                    "content": {
                        "application/json": {
                            "schema": {
                                "type": "object"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

The only property we need to change is `operationId`. This contains the name of an XQuery function to be called whenever the route is requested. By default this points to `dapi:table-of-contents`, but we change it to our own implementation, called `custom:table-of-contents`.

Next we should implement this function. Like all API functions it takes one argument only, representing the request as an XQuery map with a number of keys. Most important is `$request?parameters` which should contain a key/value pair for every parameter declared for the request.

Open the XQuery module `modules/custom-api.xql` and add a function:

```xquery
declare function api:table-of-contents($request as map(*)) {
<ul>
    {
        for $doc in collection($config:data-default)/tei:TEI
        let $relPath := substring-after(document-uri(root($doc)), $config:data-default || "/")
        return
            <li>
                <pb-link path="{$relPath}" emit="transcription">
                {$doc//tei:teiHeader/tei:fileDesc/tei:titleStmt/tei:title/string()}
                </pb-link>
            </li>
    }
    </ul>
};
```

In this simple version we just iterate over all TEI documents in the data collection and output the title from the teiHeader as a string enclosed in a `<pb-link>` element. `<pb-link>` expects a path relative to the data collection as link, so we construct one in `$relPath` first.

Since we're using variables from the **config** module, we also need to import that. We also have to declare a namespace for `tei`. So the top of the XQuery module should be changed to this:

```xquery
xquery version "3.1";

module namespace api="http://teipublisher.com/api/custom";

declare namespace tei="http://www.tei-c.org/ns/1.0";

import module namespace bapi="http://teipublisher.com/api/blog" at "blog.xql";
import module namespace rutil="http://exist-db.org/xquery/router/util";
import module namespace config = "http://www.tei-c.org/tei-simple/config" at "config.xqm";
```

### Testing the changes

It is best to test the API call independantly first. Every TEI Publisher instance includes an API viewer, which can be used for testing. In your local instance, log in and use the *Documentation/TEI Publisher API* link from the menu. Scroll to the bottom of the page to see your custom API. It should now show this:

![Custom API screenshot](/images/custom-api.png)

Click on the entry for `/api/document/{id}/contents` to see its definition. You can test the endpoint by clicking on **Try it out** and then **Execute**, which should return the HTML output you expected.

### Improving the display of titles

Instead of just printing out the raw string content of the title of each document, it would be nicer if we could format it somehow, including other information from the header. Obviously the way to do it is to pass the header through the ODD for processing. For this we first need to import two more utility modules in the header of the XQuery:

```xquery
import module namespace pm-config="http://www.tei-c.org/tei-simple/pm-config" at "pm-config.xql";
import module namespace tpu="http://www.tei-c.org/tei-publisher/util" at "lib/util.xql";
```

The first allows us to call the TEI processing model to format our TEI. The second is needed to properly configure the correct settings which apply to the current document, i.e. which ODD to use. Our function might then look as follows:

```xquery
declare function api:table-of-contents($request as map(*)) {
    <ul>
    {
        for $doc in collection($config:data-default)/tei:TEI
        let $relPath := substring-after(document-uri(root($doc)), $config:data-default || "/")
        let $pconfig := tpu:parse-pi(root($doc), ())
        let $title := $doc/tei:teiHeader/tei:fileDesc/tei:titleStmt
        return
            <li>
                <pb-link path="{$relPath}" emit="transcription">
                { $pm-config:web-transform($title, map { "headers": "short" }, $pconfig?odd) }
                </pb-link>
            </li>
    }
    </ul>
};
```

We're calling `tpu:parse-pi` to retrieve the specific configuration to be used for the current document. The ODD we determined this way is then passed into `$pm-config:web-transform` as last argument. The first argument is the XML node we want to format. In this case we're limiting it to `tei:titleStmt` and ignore the rest.

The second argument to `$pm-config:web-transform` is a map of parameters which will be forwarded to the ODD. Use this to pass external processing information. In the example above, we're passing `"headers": "short"`, which from within an ODD processing model rule would be available as `$parameters?headers='short'`. All the default ODDs in TEI Publisher check this particular parameter to output a short summary of metadata about the document, as seen in the browsing view of TEI Publisher. But feel free to add your own parameters and processing logic here.