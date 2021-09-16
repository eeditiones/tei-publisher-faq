---
title: "I don't have a IIIF server - can I still use pb-facsimile?"
menuTitle: "Using static images without IIIF"
date: 2021-09-16T11:09:37+01:00
tags: ["pb-facsimile", "pb-facs-link"]
---

Even if you do not have a IIIF server at your disposal, you can still use `pb-facsimile` to display static images hosted either in your local TEI Publisher instance or on an external webserver. Key is to set the attribute `type="image"` on `pb-facsimile` and provide the correct base URI for resolving image links. For an example, we may use the static images used by TEI Publisher on the main start page (e.g. `playground.png`). Those are stored in the `data` collection of your local TEI Publisher installation and you could simply add your own images there (or into a subcollection). 

To configure `pb-facsimile`, you just need to set `type="image"` and change the base URI to point to the root of your TEI Publisher install:

```html
<pb-facsimile type="image" base-uri="http://localhost:8080/exist/apps/tei-publisher/" default-zoom-level="0"
    show-navigation-control="show-navigation-control">
</pb-facsimile>
```

Within your ODD transformation, you could then output a `pb-facs-link` wherever you want to reference an image, e.g.:

```html
<pb-facs-link facs="playground.png">Playground</pb-facs-link>
```

This does not differ from using IIIF - just that images will be resolved statically against the base URI instead of going through the IIIF protocol. `pb-facsimile` will still allow you to zoom into the image and pan it with the mouse.