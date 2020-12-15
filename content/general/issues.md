---
title: "Common Errors"
date: 2020-12-15T18:32:11+01:00
---

## Why does my document show 'server did not return any content' error message?

Please bear in mind that while TEI Publisher aims to be a universal tool, the specific components may make certain assumptions about data they are getting and if your documents do not follow the same encoding conventions it may be required to adjust parameters passed to the components from the page template - or add required information in the document.

For example - table of content component assumes that the document structure is represented by means of nested `div` elements and section titles are given in the `head` element. If your project rather chooses numbered divisions (`div1, div2`) etc it may be advisable to adjust this to avoid customizing all navigation, table of contents and so on, but it is one of very rare cases where TEI Publisher exposes any predilection for a particular flavour of TEI.

Similarly, template with aligned transcription/translation panels is parametrized with an *XPath* expression pointing to relevant fragments of the document which store the transcription and translation. This expression most probably will have to be adjusted (unless of course you also have Latin texts with Polish translation structured in a similar way).

## Page view doesn't work

Displaying documents page by page requires that page breaks are encoded. In TEI you are expected to explicitly place `pb` elements where each page begins.

Please note that certain vocabularies, like DocBook, do not recognize the concept of a page at all, so page view cannot possibly work for these.