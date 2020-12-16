---
title: "Can alternate content be loaded asynchronously?"
menuTitle: "Asynchronous Popover"
date: 2020-12-12T14:21:16+01:00
weight: 2
tags: ['webcomponents', 'pb-popover']
---

If you output a lot of **alternate** blocks within your ODD, e.g. for places, people or terms, it can slow down rendering of the text, in particular, if the content to be shown as alternate requires further processing or is loaded from an external database system via an API.

To help with this, the _pb-popover_ webcomponent allows popup content to be loaded asynchronously. _pb-popover_ is the default component which is output for `<model behaviour="alternate">`.

To use this feature, change your model to use a template expression instead of the _alternate_ behaviour. In the template, output a _pb-popover_ and give it an attribute __remote__, containing the URL from which the content for this particular popover should be loaded. The popover will then defer loading content until it is actually shown.

For example, the following _elementSpec_ outputs a _pb-popover_ for any **placeName** having an **@ref** attribute. Instead of constructing the contents of the alternate immediately, it just outputs a _loading_ message. The `remote="api/places/[[ref]]"` attribute specifies the endpoint from which the actual content will be loaded once the popover is displayed. The **@ref** is passed to the endpoint as last path component.

```xml
<elementSpec ident="placeName" mode="add">
    <model behaviour="inline" cssClass="place" predicate="@ref">
        <param name="ref" value="@ref"/>
        <pb:template xmlns="" xml:space="preserve">
            <pb-popover remote="api/places/[[ref]]">[[content]]<span slot="alternate">Loading ...</span></pb-popover>
        </pb:template>
    </model>
    <model behaviour="inline"/>
</elementSpec>
```

For sure this assumes that you created a custom API endpoint `/api/places/{ref}` on your TEI Publisher instance.