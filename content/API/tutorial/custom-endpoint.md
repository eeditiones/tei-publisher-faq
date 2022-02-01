---
title: "Introduction "
menuTitle: "Introduction"
date: 2022-01-30T12:14:39+01:00
tags: ["api"]
weight: 1
---

Since version 7.0 of TEI Publisher, nearly all requests to the server are handled via an API (the only exception being requests for static resources like images or CSS styles). The entire API is expressed in a formal specification based on the [Open API standard](http://spec.openapis.org/oas/v3.0.3). Within TEI Publisher you can browse the API by opening the `Documentation/TEI Publisher API` page from the menubar. Here you'll see a list of URL paths (or *API endpoints* as we often call them).

You can use the API documentation page to directly try out the endpoints or use the provided information to construct your own URLs. For example, if we look at the very first endpoint on the page, `/api/document/{id}`, we can see that it expects a single required parameter called `id`. Usually parameters are either specified as normal HTTP request parameters (written after the `?` in an URL for a GET request) or within the URL path itself. The `id` parameter is defined as a **path** parameter and it is referenced in the URL path accordingly (the bit in curly brackets: `{id}`).

![API documentation](/images/api-documents.png)

Now let's assume we would like to call this endpoint to retrieve the source XML of Graves' letter, which resides under `data/test/graves6.xml` in a default TEI Publisher install. The `id` parameter should be relative to the data root collection of TEI Publisher, thus `test/graves6.xml`. The Open API specification requires that path parameters are URL-encoded, so the / in the path should be encoded as `%2F`.

Therefore the URL path to use would become `/api/document/test%2Fgraves6.xml`. Now we just prepend the correct server address pointing to the root of TEI Publisher to get a full URL which we can paste into the browser's location bar, e.g. for our local install: <http://localhost:8080/exist/apps/tei-publisher/api/document/test%2Fgraves6.xml>.

If instead of retrieving the raw XML we would like to see the HTML output (after applying an ODD), we can use the endpoint `/api/document/{id}/html`. This requires an additional parameter to specify the ODD, so the full URL on localhost would become: <http://localhost:8080/exist/apps/tei-publisher/api/document/test%2Fgraves6.xml/html?odd=graves.odd>.