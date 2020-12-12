---
title: "General Questions"
date: 2020-12-11T20:35:42+01:00
tags: ["webcomponents", "pb-page"]
weight: 1
---

## Why is a request sent to `/api/version` and `/login` when the page loads?

The webcomponent library needs to figure out with which version of TEI Publisher it is communicating on the server side. It thus sends two requests while initializing:

1. one to an endpoint called `/login`
2. followed by one to `/api/version`

If the first request returns in a _not found_, the client can safely assume it is talking to a publisher version > 7. It thus sends a follow up to retrieve the actual version. If `/login` returns a proper response, it means the endpoint is a TEI Publisher before 7. We need to make both requests in this sequence to bypass security restrictions in the browser.

If you know that your application will always talk to a defined TEI Publisher version, you can enforce an API version and thus get rid of the extra requests. To do this, pass the attribute `api-version` to `<pb-page>`, e.g.

```xml
<pb-page data-template="pages:pb-page" unresolved="unresolved" api-version="1.0.0">
...
</pb-page>
```

API versions can be determined as follows:

* for TEI Publisher before version 7 we assume "0.9.0", which causes the webcomponents library to switch to backwards-compatibility mode
* versions after 7.0.0 declare the API version they implement in their Open API descriptor, which you can find in `modules/lib/api.json`. The version is specified in the _info/version_ property:

    ```json
    {
        "openapi": "3.0.0",
        "info": {
            "version": "1.0.0",
            "title": "TEI Publisher API",
            "description": "This is the core API for TEI Publisher 7. It describes all endpoints used by TEI Publisher web components, plus additional operations added for external access."
        },
    ```