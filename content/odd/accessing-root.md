---
title: "How do I navigate to the root of the document in XPath?"
menuTitle: "Accessing root"
date: 2020-12-16T19:03:51+01:00
tags: ["ODD", "processing model", "navigation"]
---

Normally you would get to the document-node of the current document by calling `root(.)` in an XPath within the ODD. This works, but there is a caveat:

TEI Publisher will not always pass the entire document to the ODD! For example, if documents are viewed page by page, TEI Publisher will first construct a virtual TEI document containing only the relevant content of the page. This is necessary to make sure the content is well-formed XML. Calling `root(.)` may thus not return the original document-node but just the constructed root.

To compensate for this, TEI Publisher always passes in an external parameter pointing to the original document-node. It can be accessed via the variable `$parameters?root`.