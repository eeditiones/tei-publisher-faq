---
title: "How can I include elements located elsewhere (in the same or another document)?"
menuTitle: "Accessing elements"
date: 2020-12-13
tags: ["ODD", "processing model", "navigation"]
---

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

Here we use the fn:id XPath function to look up the element whose ID is given in the @target attribute. It will usually start with an '#', so we have to strip this out. The second parameter to fn:id specifies the document-node to search for the id. 

{{% notice note %}}
Note that we're using $parameters?root here and not just root(.). [See this article](/odd/accessing-root) for an explanation why.
{{% /notice %}}

## Including content from other documents

It is also possible to redirect processing to arbitrary other documents stored in the database. Let's assume you would like to show additional information about people occurring in the text in an `alternate`. The additional information may be contained in the TEI header of the current file, but in many cases this is not practical and you would rather like to keep a list of people for all documents in a separate authority file, which is easier to maintain.

To look up a person by `@xml:id` in the separate authority file, you just need to change the second parameter to the `id()` function to point to the correct location in the database:

```xml
<model behaviour="alternate">
    <param name="default" value="."/>
    <param name="alternate" value="id(substring-after(@ref, '#'), collection('/db/apps/serafin/data/auxiliary'))"/>
    <outputRendition xml:space="preserve">
    color: #1565c0;
    </outputRendition>
</model>
```

Here we search through an entire collection, but you can as well limit it to a single document by using the `doc()` function instead of `collection()`.