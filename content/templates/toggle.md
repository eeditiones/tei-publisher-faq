---
title: "How can I dynamically toggle components to show/hide?"
menuTitle: "Toggling components"
date: 2021-01-27T11:51:25+01:00
tags: ['webcomponents', 'events', 'views', 'diplomatic', 'normalized']
---

The `pb-toggle-feature` and `pb-select-feature` components can be used to toggle the state of selected elements or components on a page. Both emit a `pb-toggle` event if the selection changes, which is handled by `pb-view` and `pb-load`. There are two modes of operation as described in the [webcomponent documentation](https://unpkg.com/@teipublisher/pb-components@latest/dist/api.html#pb-toggle-feature.0) for `pb-toggle-feature`:

1. **server side toggling**: causes the receiving `pb-view` or `pb-load` to reload the currently displayed content, leaving it to the ODD to process the state changes
2. **client side toggling**: applies state changes client side (i.e. without server roundtrip)

In this article we'll concentrate on **client side toggling**.

## An Example

Assume you want to provide the user two different views on the content: a *diplomatic view*, which tries to reproduce the original text with all its details, and a *normalized view*, featuring an easier to read, edited version. Let's say the normalized version differs as follows:

* line breaks are hidden
* we show popups for personal or place names with additional information
* the facsimile is not displayed

Using `pb-select-feature` we can define the two states as below:

```xml
<pb-select-feature id="select-view1" name="view1" label="View" items='[
        {"name": "Normalized Text", "selectors": [{"selector": "br", "state": true}, {"selector": "pb-popover.name", "command": "disable", "state": false}]},
        {"name": "Diplomatic Text", "selectors": [{"selector": ".popover,.annotation", "state": true}, {"selector": "br,.underline", "state": false}, {"selector": "pb-popover.name", "command": "disable", "state": true}]}
        ]' emit="transcription" subscribe="transcription"></pb-select-feature>
```

`pb-select-feature` is implemented as a dropdown list, so the user could choose between more than two states. The list of items has to be defined as a JSON array in property `items`. Each member of the array is an object with two properties:

1. **name**: the label to display in the dropdown
2. **selectors**: an array of objects defining CSS3 selectors which will be used to find the elements to which the state change applies

Looking at *Normalized Text*, we have two selector entries: the first targets all `<br>` elements and sets their *state* to `true`. Because there is no explicit command given, the default command, `toggle`, will apply. All the `toggle` command does is to add a CSS class `toggle` to the target element if state=true and remove it if state=false.

To hide the line breaks, you can thus add a rule as follows to your CSS:

```css
br.toggle {
    display: none;
}
```

{{% notice note %}}
Note that this rule has to go into the CSS applied to the text transformed via the ODD, so it must be linked to either the ODD or `pb-view` (see the [documentation](https://teipublisher.com/exist/apps/tei-publisher/doc/documentation.xml?id=css-styling-external)).
{{% /notice %}}

The second selector for *Normalized Text* additionally defines a command: `disable` and passes `state=false`. Just toggling a CSS class would not be enough to achieve the desired effect in this case: we don't want the `pb-popover` to disappear completely in the diplomatic view. After all the inline text it wraps around should still be shown. Instead, we want the `pb-popover` to no longer react when the user moves the mouse over it. This is what `disable` essentially does: disable the functionality of a component. For the normalized text we don't want to disable, so we pass `state=false` (which essentally means: *enable*), and `state=true` for the diplomatic view.

Only some components react to `disable`, at the time of writing: `pb-popover`, `pb-highlight` and `pb-navigation`. See the [web components documentation](https://unpkg.com/@teipublisher/pb-components@latest/dist/api.html#pb-select-feature.2) for a full example.

## Toggling the facsimile and other elements elsewhere

The section above covered toggling of elements shown as part of the edition text, i.e. inside a `pb-view` or `pb-load`. But what about other, separate components also shown on the page, such as the facsimile view? Beginning with version **1.18.0** of the TEI Publisher webcomponents (check how to [upgrade your version](/webcomponents/version-upgrade/)), this can be done as well. We just add one selector for `pb-facsimile` as shown below:

```xml
<pb-select-feature id="select-view1" name="view1" label="Ansicht" items='[
        {"name": "Edierter Text", "selectors": [{"selector": "br", "state": true}, {"selector": "pb-popover.name", "command": "disable", "state": false}, {"selector": "pb-facsimile", "state": true}]},
        {"name": "Diplomatischer Text", "selectors": [{"selector": ".popover,.annotation", "state": true}, {"selector": "br,.underline", "state": false}, {"selector": "pb-popover.name", "command": "disable", "state": true}, {"selector": "pb-facsimile", "state": false}]}
        ]' emit="transcription" subscribe="transcription"></pb-select-feature>
```

However, we need one more step: because the `pb-facsimile` can be anywhere on the page, we have to tell the `pb-page` which wraps around it to listen for the `pb-toggle` event and dispatch it to the facsimile viewer. All we need is thus to subscribe `pb-page` to the event channel into which `pb-select-feature` emits events, i.e. *transcription*:

```html
<pb-page data-template="pages:pb-page" unresolved="unresolved" subscribe="transcription">
...
</pb-page>
```