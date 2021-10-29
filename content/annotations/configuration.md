---
title: "Why are my annotations not applied to some divisions?"
menuTitle: "Configuration Pitfalls"
date: 2020-12-16T15:32:18+01:00
tags: ['annotations', 'configuration']
---

After generating a stand-alone application, you may find that annotations are skipped for some divisions of the text, i.e. they are not merged properly into the TEI. 

Most likely this is a configuration issue: by default, TEI Publisher paginates a text by division. If a division starts with only a short text before its subdivision, TEI Publisher tries to fill it up by pulling the first subdivisions into the page.

In the annotation editor this won't work as it leads to wrong character offsets! There are two global configuration variables to control pagination and the threshold below which publisher tries to fill up the page, namely:

```xquery
declare variable $config:pagination-depth := 1;

declare variable $config:pagination-fill := 0;
```

For an application used **exclusively** for annotating documents, the variables should be set to `pagination-depth=1` and `pagination-fill=0` as shown above. Another approach would be to configure those parameters only for the collection used for annotating. This is what TEI Publisher app does:

```xquery
declare function config:collection-config($collection as xs:string?, $docUri as xs:string?) {
    switch ($collection)
        (: For annotations we need to overwrite document-specific settings :)
        case "annotate" return
            map {
                "template": "annotate.html",
                "overwrite": true(),
                "depth": 1,
                "fill": 0
            }
        default return
            (: Return empty sequence to use default config :)
            ()
};
```

In this example, we also ensure that the correct template is chosen (`annotate.html`) and any local configuration in the document (via a processing instruction) is disabled (`overwrite: true()`).