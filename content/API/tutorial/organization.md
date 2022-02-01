---
title: "Organization of the API"
menuTitle: "Organization"
date: 2022-01-30T12:14:39+01:00
tags: ["api"]
weight: 2
---

As can be seen on the API documentation page, the different endpoints are grouped into categories ("tags") like *documents*, *collection*, *odd* etc. This grouping is just for your convenience and has no technical implications. You'll also find that there is a separate section at the bottom, called *Custom API*. This is specifically for endpoints you want to add on top of TEI Publisher's default endpoints. We'll use this below.

More important is another distinction: looking at the endpoint paths we see two kinds: those starting with `/api` and those without prefix, which all appear under the *view* tag. In general we can say that URLs **without** `/api` prefix are intended to be **directly called** by the user when entering a URL into the browser or navigating a link, while those with `/api` prefix are usually not exposed to the user but instead consumed by other code components.

To illustrate this with an example: if you navigate to `/exist/apps/tei-publisher/doc/documentation.xml` in your browser, the endpoint called will be `/{docId}`. This endpoint inspects the requested document and determines that it should be displayed using the HTML template `documentation.html` (residing in `templates/pages/documentation.html`). It thus loads this template, expands it through eXist's HTML templating (which does things like e.g. including the menubar and toolbar), and returns it to the browser.

![API Request Flow](/images/api-flow.png)

The browser will start to render the received HTML. While doing so, it finds a bunch of custom elements (= *webcomponents*) and activates them. Among those webcomponents is the central `pb-view` on the page, which is supposed to display the actual documentation content. When it gets activated, it will send another request to the TEI Publisher API, which would say: "get me the next division of the documentation as HTML transformed via ODD". This second request is triggered by the component (not the user directly) and thus starts with the `/api` prefix (in concrete it will go to `/api/parts/doc%2Fdocumentation.xml/html`).

To summarize, we can distinguish between

1. user-facing endpoints, which produce HTML content to be viewed in a browser
2. machine-processable API endpoints providing information to be consumed by components or other services

In the next section we learn how to create an endpoint to be used by a webcomponent, thus falling under category (2).