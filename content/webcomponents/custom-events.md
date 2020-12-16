---
title: "Custom Event Wiring"
date: 2020-12-16T15:32:18+01:00
tags: ['webcomponents', 'events']
---

## Can I connect components via custom code?

There might be cases where you would like to make a component react to changes in another component, but TEI Publisher does not provide the necessary connection by default. Components in Publisher mainly communicate via [signals sent into channels](https://teipublisher.com/exist/apps/tei-publisher/doc/documentation.xml?id=webcomponents-communication). You can thus always connect two components via custom code by listening to events sent by a first component and - when received - trigger another component.

Let's assume you would like certain parts of your document to be collapsed initially. You are thus outputting a `pb-collapse`. However, instead of preloading the collapsed content, it should be loaded only when the user actually wants to look at it and thus expands the `pb-collapse`. We're thus using a `pb-view` (or `pb-load`) for the content:

```html
<pb-collapse emit="meta">
    <div slot="collapse-trigger">About this text</div>
    <pb-view slot="collapse-content" on-update src="intro" subscribe="meta"></pb-view>
</pb-collapse>
```

The `pb-view` will load content from a different document (defined by a `pb-document` with id *intro*). The `@on-update` property instructs the view to not load content immediately, but only when it receives a **pb-update** event.

We thus need to wire the `pb-collapse` with the `pb-view` by sending a **pb-update** event to the view whenever the collapse is expanded. `pb-collapse` will emit a **pb-collapse-open** event in this case. We can thus connect the **pb-collapse-open** signal with **pb-update** by inserting a few lines of custom code into the HTML template we use to display the text:

```html
<script>
    window.addEventListener('load', () => {
        document.addEventListener('pb-collapse-open', (ev) => {
            if (ev.detail && ev.detail.key && ev.detail.key === 'meta') {
                document.dispatchEvent(new CustomEvent('pb-update', {
                    detail: {
                        key: 'meta'
                    }
                }));
            }
        });
    });
</script>
```

Important here is the `key` property in the event details: within TEI Publisher, it always indicates the *channel* for which an event should apply. It corresponds to the channels we defined with `@subscribe` and `@emit` on the HTML elements above. If there's no key, it means that the event is targetted at the global default channel. For simple cases, emitting events to the default channel might be enough, but if your HTML gets more complex, you want to be sure that this particular `pb-collapse` is sending its events to the correct `pb-view` - and not all of them.

### Using utility methods

Beginning with version **1.13.0** of pb-components, there's a utility class which can be used to simplify the code above:

```html
<script>
window.addEventListener('load', () =>
    pbEvents.subscribe('pb-collapse-open', 'meta', () => pbEvents.emit('pb-update', 'meta'), true)
);
</script>
```

The arguments are:

1. the name of the event
2. the channel to emit to (or null for the default channel)
3. a callback function to be called when the event is triggered
4. should the event handler fire only once?

By passing 'true' for the fourth argument, we prevent the `pb-update` event being resent whenever the user expands the collapse again. Loading the content of the collapse once should be enough.

As an alternative to passing in a callback function, `pbEvents.subscribe` also returns a `Promise`. So here's another formulation of the same:

```html
<script>
window.addEventListener('load', () =>
    pbEvents.subscribe('pb-collapse-open', 'meta', null, true)
        .then(() => pbEvents.emit('pb-update', 'meta'))
);
</script>
```

### Demo

You can find a demo of the code covered in this article in the [pb-components documentation](https://unpkg.com/@teipublisher/pb-components@latest/dist/api.html#pb-collapse.2).