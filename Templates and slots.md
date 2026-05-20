#WebComponents 
# The truth about templates
When you have to reuse the same markup structures repeatedly on a web page, it makes sense to use some kind of a template rather than repeating the same structure over and over again. This is possible by the HTML `<template>` element. This element and its contents are not rendered in the DOM, but it can still be referenced using Javascript.
```html
<template id="custom-paragraph">
  <p>My paragraph</p>
</template>
```
This won't appear in your page until you grab a reference to it with Javascript and then append it to the DOM, using something like the followong:
```javascript
let template = document.getElementById("custom-paragraph");
let templateContent = template.content;
document.body.appendChild(templateContent);
```
# Using templates with web components
Templates are useful on their own, but they work even better with web components. Let's define a web component that uses our template as the content of its shadow DOM.
```js
customElements.define(
  "my-paragraph",
  class extends HTMLElement {
    constructor() {
      super();
      let template = document.getElementById("custom-paragraph");
      let templateContent = template.content;

      const shadowRoot = this.attachShadow({ mode: "open" });
      shadowRoot.appendChild(document.importNode(templateContent, true));
    }
  },
);
```
The key point to note here is that we append a clone of the template content to the shadow root, created using the `Document.importNode()` method.

And because we are pending its contents to a shadow DOM, we can include some styling information inside the template in a `<style>` element, which is then encapsulated inside the custom element. This wouldn't work if we just appended it to the standard DOM.
```html
<template id="custom-paragraph">
  <style>
    p {
      color: white;
      background-color: #666666;
      padding: 5px;
    }
  </style>
  <p>My paragraph</p>
</template>
```

Now we can use it by just adding it to our HTML document:
```html
<my-paragraph></my-paragraph>
```
# Adding flexibility with slots
We can make it possible to display different text in each element instance in a nice declarative way using the `<slot>` element.

Slot are identified by their `name` attribute, and allow you to define placeholders in your template that can be filled with any markup fragment you want when the element is used in the markup.

So, if we want to add a slot into our trivial example, we could update our template's paragraph element like this:
```html
<p><slot name="my-text">My default text</slot></p>
```
>[!IMPORTANT]
>The `name` attribute should be unique per shadow root: if you have two slots with the same name, all of these elements with a matching `slot` attribute will be asigned to the first slot with that name.

If the slot's content isn't defined when the element is included in the markup,or if the browser doesn't support slots, `<my-paragraph>` just contains the fallback content "My default text".

To define the slot's content, we include an HTML structure inside the `<my-paragrapg>`element with a `slot` attribute whose value is equal to the name of the slot we want it to fill.
```html
<my-paragraph>
  <span slot="my-text">Let's have some different text!</span>
</my-paragraph>
```
or
```html
<my-paragraph>
  <ul slot="my-text">
    <li>Let's have some different text!</li>
    <li>In a list!</li>
  </ul>
</my-paragraph>
```
>[!NOTE]
>The `slot` attribute does not need to be unique: a `<slot>` can be filled by multiple elements that all have a matching `slot` attributes.
>```html
><my-paragraph>
>	<span slot="my-text">This will be displayed</span>
>	<span slot="my-text">This will be displayed twice</span>
 ></my-paragraph>
>```

The `name` and `slot` attributes both default to the empty string, so elements with no `slot` attributes are assigned to the `<slot>`with no `name` attribute (the unnamed slot, or default slot).
```html
<template id="custom-paragraph">
  <style>
    p {
      color: white;
      background-color: #666666;
      padding: 5px;
    }
  </style>
  <p>
    <slot name="my-text">My default text</slot>
    <slot></slot>
  </p>
</template>
```
```html
<my-paragraph>
  <span slot="my-text">Let's have some different text!</span>
  <span>This will go into the unnamed slot</span>
  <span>This will also go into the unnamed slot</span>
</my-paragraph>
```
In this example:
- Content with `slot="my-text"` goes into the named slot.
- All other content automatically goes into the unnamed slot.
# A more involved example
>[!NOTE]
>It it is technically possible to use `<slot>` element without a `<template>` element, e.g., within say a regular `<div>` element, and still take advantage of the place-holder features of `<slot>` for Shadow DOM content, and doing so may indeed avoid the small trouble of needing to first access the template element's content property (and clone it). However, it is generally more practical to add slots within a `<template>` element, since you are unlikely to need to define a pattern based on an already-rendered element.
>In addition, even if it is not already rendered, the purpose of the container as a template should be more semantically clear when using the [`<template>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/template). In addition, [`<template>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/template) can have items directly added to it, like [`<td>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/td), which would disappear when added to a [`<div>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/div).

## Creating a template with some slots
```html
<template id="element-details-template">
  <style>
    details {
      font-family: "Open Sans Light", "Helvetica", "Arial";
    }
    .name {
      font-weight: bold;
      color: #217ac0;
      font-size: 120%;
    }
    h4 {
      margin: 10px 0 -8px 0;
    }
    h4 span {
      background: #217ac0;
      padding: 2px 6px;
    }
    h4 span {
      border: 1px solid #cee9f9;
      border-radius: 4px;
    }
    h4 span {
      color: white;
    }
    .attributes {
      margin-left: 22px;
      font-size: 90%;
    }
    .attributes p {
      margin-left: 16px;
      font-style: italic;
    }
    dl {
	  margin-left: 6px;
	}
	dt {
	  color: #217ac0;
	  font-family: "Consolas", "Liberation Mono", "Courier New";
	  font-size: 110%;
	  font-weight: bold;
	}
	dd {
	  margin-left: 16px;
	}
  </style>
  <details>
    <summary>
      <span>
        <code class="name"
          >&lt;<slot name="element-name">NEED NAME</slot>&gt;</code
        >
        <span class="desc"
          ><slot name="description">NEED DESCRIPTION</slot></span
        >
      </span>
    </summary>
    <div class="attributes">
      <h4><span>Attributes</span></h4>
      <slot name="attributes"><p>None</p></slot>
    </div>
  </details>
  <hr />
</template>
```
```js
customElements.define(
  "element-details",
  class extends HTMLElement {
    constructor() {
      super();
      const template = document.getElementById(
        "element-details-template",
      ).content;
      const shadowRoot = this.attachShadow({ mode: "open" });
      shadowRoot.appendChild(document.importNode(template, true));
    }
  },
);
```
## Using the `<element-details>` custom element with named slots
```html
<element-details>
  <span slot="element-name">slot</span>
  <span slot="description"
    >A placeholder inside a web component that users can fill with their own
    markup, with the effect of composing different DOM trees together.</span
  >
  <dl slot="attributes">
    <dt>name</dt>
    <dd>The name of the slot.</dd>
  </dl>
</element-details>

<element-details>
  <span slot="element-name">template</span>
  <span slot="description"
    >A mechanism for holding client- side content that is not to be rendered
    when a page is loaded but may subsequently be instantiated during runtime
    using JavaScript.</span
  >
</element-details>
```