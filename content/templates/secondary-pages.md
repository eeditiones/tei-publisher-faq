---
title: "How can I add auxiliary pages to the edition?"
menuTitle: "Auxiliary Pages"
date: 2020-12-17T14:51:00+01:00
tags: ["templates", "markdown"]
---

A digital edition usually includes a number of auxiliary pages such as an introduction or "About this project" page. This can be done in a number of ways. The easiest is to write your text in an XML format already supported by Publisher like TEI or Docbook. Just save the document into your data directory (or a subdirectory) alongside the edition texts and give it a descriptive name like `introduction.xml`.

To avoid that your introduction appears in the document listing or in a search targeted at the edition texts, you may want to add it to the list of excluded documents in `modules/config.xqm`:

```xquery
(:~
 : A sequence of root elements which should be excluded from the list of
 : documents displayed in the browsing view.
 :)
declare variable $config:data-exclude := (
    doc($config:data-root || "/taxonomy.xml")/tei:TEI,
    collection($config:data-root || "/doc")/tei:TEI,
    (: Exclude single document :)
    doc($config:data-root || "/introduction.xml")/tei:TEI,
    (: Exclude a whole subcollection :)
    collection($config:data-root || "/project")/tei:TEI
);
```

You may then link to the newly added document, e.g. from the menubar using a standard link like:

```html
<a href="${app}/introduction.xml" data-template="pages:parse-params">Introduction</a>
```

{{% notice note %}}
To ensure the link also works when clicked from a page further below the application root, we're using `${app}` together with the `pages:parse-params` template instruction. The variable will be replaced with the root URL of the app by server-side processing.
{{% /notice %}}

## Using Markdown

Starting with TEI Publisher 7.0.0, you may also use Markdown for auxiliary pages. Markdown is a plain text format and quicker to write than TEI or Docbook. For example, all the pages in this FAQ are written in markdown.

To create a markdown document, just create a file ending with `.md` and store it into your data collection. You can then access it like an XML document. An example is shown in the default [TEI Publisher app](https://teipublisher.com/exist/apps/tei-publisher/about.md).

The markdown is rendered on the client using the HTML template `templates/pages/markdown.html`, which you can change if you would like to rearrange or add things.