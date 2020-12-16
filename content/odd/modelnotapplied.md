---
title: "Why does my added model not have an effect?"
menuTitle: "Model has no effect"
date: 2020-12-13
tags: ["ODD", "processing model"]
---

You added another model rule to an elementSpec but it does not seem to have any effect. This might be due to the rules for selecting a model:

The processor walks through all models in sequence and stops at the first one which either has a matching `@predicate` and/or `@output` or neither of both. **Order is important!** Therefore you should always place models for specific cases first and the most general (without predicate or output) last. If your general model is placed in front of your more specific models, it will always match:

```xml
<elementSpec mode="change" ident="hi">
  <model behaviour="inline"/>
  <model predicate="@rend='bold'" behaviour="inline">
    <outputRendition>font-weight: bold;</outputRendition>
  </model>
</elementSpec>
```

Another common case is missing the step which generates transform code on the basis of your ODD. If you are using the graphic ODD editor this step is performed automatically upon saving the ODD. When editing manually make sure you click on regenerate button in the list of ODDs in the TEI Publisher or execute `modules/lib/regenerate.xql` script. The latter is the only method to regenerate transform code in your custom application without using the graphic ODD editor.