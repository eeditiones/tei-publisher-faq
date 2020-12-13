---
title: "Accessing arbitrary nodes in the ODD"
date: 2020-12-13
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