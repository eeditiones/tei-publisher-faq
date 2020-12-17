---
title: "How can I figure out the correct IIIF link to an image?"
menuTitle: "IIIF endpoint configuration"
date: 2020-12-17T11:09:37+01:00
tags: ["iiif", "pb-facsimile", "pb-facs-link"]
---

To correctly connect TEI Publisher's `<pb-facsimile>` component with a IIIF server, it's best to first try in the browser and access one image you know should exist. For any image, the first thing the IIIF viewer requests from the server is the `info.json` containing metadata about the image. You should thus find out how to access this `info.json` from your server.

For example, the TEI Publisher demo loads the facsimile images for the Shakespeare from an URL as follows:

https://apps.existsolutions.com/cantaloupe/iiif/2/axc0671-0.jpg/info.json

This represents the starting point for the IIIF viewer. You can click on the link an you'll see the metadata returned. If you don't get any response here, the image view won't work.

From above URL you can deduce the correct `base-uri` for `<pb-facsimile>`: `https://apps.existsolutions.com/cantaloupe/iiif/2/`, which is the part of the URL before the path to the actual image. You can test if the viewer works by directly passing it the filename of an image in the `facsimiles` property (note this is a JSON array): 

```html
<pb-facsimile base-uri="https://apps.existsolutions.com/cantaloupe/iiif/2/" facsimiles="[&quot;axc0671-0.jpg&quot;]" default-zoom-level="0" show-navigator="show-navigator" show-navigation-control="show-navigation-control"></pb-facsimile>
```

In real use you would normally not hardcode the images, but instead output a `<pb-facs-link>` from within your ODD processing model (as we do for the Shakespeare and other examples), containing the reference to the image in the `@facs` attribute:

```html
<pb-facs-link facs="axc0671-0.jpg">
```

{{% notice note %}}
`<pb-facsimile>` will automatically pick up the images referenced via `<pb-facs-link>` when the content is loaded. For this to work, it is important that the `<pb-view>`, which loads the text content, emits its update events to the channel to which `<pb-facsimile>` is listening.
{{% /notice %}}

## Images in subdirectories

If you keep the images on the server in a subdirectory structure, e.g. `shakespeare/axc0671-0.jpg`, be aware that IIIF servers require you to encode the "/" in the directory path. This is usually done by using the `%2F` character encoding, e.g. `shakespeare%2Faxc0671-0.jpg`. Some IIIF servers also allow using "!" as separator instead of "/".