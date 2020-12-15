---
title: "Navigating the Document"
date: 2020-12-13
tags: ["ODD", "processing model", "navigation"]
---

## How can I include elements located elsewhere (in the same or another document)?

For example, it is common practice to keep notes in the back of the document and reference them using ref. When processing the ref, you want to include the note's content into the note behaviour:

```xml
<elementSpec mode="change" ident="ref">
  <model behaviour="note">
    <param name="place" value="'margin'"/>
    <!-- Look up the note by xml:id -->
    <param name="content" value="id(substring-after(@target, '#'), root($parameters?root))/node()"/>
  </model>
</elementSpec>
```

Here we use the fn:id XPath function to look up the element whose ID is given in the @target attribute. It will usually start with an '#', so we have to strip this out. The second parameter to fn:id specifies the document-node to search for the id. Note that we're using $parameters?root here and not just root(.). See next topic below for an explanation.

## How do I navigate to the root of the document in XPath?
Normally you would get to the document-node of the current document by calling `root(.)` in an XPath within the ODD. This works, but there is a caveat:

TEI Publisher will not always pass the entire document to the ODD! For example, if documents are viewed page by page, TEI Publisher will first construct a virtual TEI document containing only the relevant content of the page. This is necessary to make sure the content is well-formed XML. Calling `root(.)` may thus not return the original document-node but just the constructed root.

To compensate for this, TEI Publisher always passes in an external parameter pointing to the original document-node. It can be accessed via the variable `$parameters?root`.