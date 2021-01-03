---
title: "Can I connect components via custom code?"
menuTitle: "Custom Event Wiring"
date: 2020-12-16T15:32:18+01:00
tags: ['webcomponents', 'events']
---

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

Beginning with version **1.14.1** of pb-components ([check your version](/webcomponents/version-upgrade)), there's a utility class which can be used to simplify the code above:

```html
<script>
window.addEventListener('load', () =>
    pbEvents.subscribe('pb-collapse-open', 'meta', () => pbEvents.emit('pb-update', 'meta'), true)
);
</script>
```

The arguments to `subscribe` are:

1. the **name** of the event: you can find the different events emitted by each component in the [component docs](https://unpkg.com/@teipublisher/pb-components@latest/dist/api.html)
2. the **channel(s)** to subscribe to. Can also be null for the default channel. To subscribe to multiple channels at once, pass in an array.
3. a **callback function** to be called when the event is triggered. This will receive the event as parameter.
4. should the event handler fire only **once**?

By passing 'true' for the fourth argument, we prevent the `pb-update` event being resent whenever the user expands the collapse again. Loading the content of the collapse once should be enough.

Alternatively you can use the `subscribeOnce` function if you would like the event handler to only execute once. This function returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), which resolves when the event is received:

```html
<script>
window.addEventListener('load', () =>
    pbEvents.subscribeOnce('pb-collapse-open', 'meta')
        .then(() => pbEvents.emit('pb-update', 'meta'))
);
</script>
```

The first two arguments for `emit` are the same as for `subscribe`. The third argument can be used to pass data to the event. The event handler can access this information in the `ev.detail` property as in the following snippet:

```html
<script>
pbEvents.subscribe('pb-update', ['transcription', 'translation'], (ev) => {
    console.log(ev.detail); // should include foo: 'baz'
}, true);
pbEvents.emit('pb-update', 'translation', {
    foo: 'baz'
});
</script>
```

### Demo

You can find a demo of the code covered in this article in the [pb-components documentation](https://unpkg.com/@teipublisher/pb-components@latest/dist/api.html#pb-collapse.2).