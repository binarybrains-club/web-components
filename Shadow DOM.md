#WebComponents 
>[!NOTE]
>Shadow DOM enables to attach a DOM tree to an element, and have the internals of this tree hidden from Javascript and CSS running in the page.

*Shadow* DOM allows hidden DOM trees to be attached to elements in the regular DOM tree, this shadow DOM tree starts with a shadow root, underneath which you can attach any element, in the same way as the normal DOM

![[shadow-dom-elements.png]]
There are some bits of shadow DOM terminology to be aware of:
- **Shadow host**: The regular DOM node that the shadow DOM is attached to.
- **Shadow tree**: The DOM inside the shadow DOM.
- **Shadow boundary**: the where the shadow DOM ends, ant the regular DOM begins.
- **Shadow root**: The root node of the shadow tree.
>[!NOTE]
>You can affect the nodes in the shadow DOM in exactly the same way as non-shadow nodes. The difference is that none of the code inside a shadow DOM can affect anything outside it, allowing for handy encapsulation.

## Attribute inheritance
The shadow tree and `<slot>` elements inherit the `dir` and `lang` attributes from their shadow host.
# Shadow DOM and custom elements
Custom elements are implemented as a class which extends either the base `HTMLElement` or a built-in HTML element such as `HTMLParagraphElement`. Typically, the custom element itself is a shadow host, and the element creates multiple elements under that root, to provide the internal implementation of the element.

```javascript
class FilledCircle extends HTMLElement {
  constructor() {
    super();
  }
  connectedCallback() {
    // Create a shadow root
    // The custom element itself is the shadow host
    const shadow = this.attachShadow({ mode: "open" });

    // create the internal implementation
    const svg = document.createElementNS("http://www.w3.org/2000/svg", "svg");
    const circle = document.createElementNS(
      "http://www.w3.org/2000/svg",
      "circle",
    );
    circle.setAttribute("cx", "50");
    circle.setAttribute("cy", "50");
    circle.setAttribute("r", "50");
    circle.setAttribute("fill", this.getAttribute("color"));
    svg.appendChild(circle);

    shadow.appendChild(svg);
  }
}

customElements.define("filled-circle", FilledCircle);
```

# Creating a shadow DOM
## Imperatively with Javascript
The following page contains two elements, a [`<div>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/div) element with an [`id`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/id) of `"host"`, and a [`<span>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/span) element containing some text:
```html
<div id="host"></div>
<span>I'm not in the shadow DOM</span>
```
We're going to use the "host" element as the shadow host. We call `attachShadow()` on the host to create the shadow DOM, and can then add nodes to the shadow DOM just like we would to the main DOM.
```javascript
const host = document.querySelector("#host");
const shadow = host.attachShadow({ mode: "open" });
const span = 
```

The result looks like this:
![[shadow-dom-imperatively.png]]
>[!IMPORTANT]
>Creating a **shadow DOM via Javascript API** might be good option for **client-side rendered applications**.
# Declaratively with HTML
For other applications, a server-side rendered UI might have better performance and a better user experience. In such cases you can use the `<template>` element to declaratively define the shadow DOM. The key to this behaviour is the enumerated `shadowrootmode` attribute, which can be set either `open` or  `closed`, the same values as the mode option of [[#Imperatively with Javascript|`attachShadow()`]] method.
```html
<div id="host">
  <template shadowrootmode="open">
    <span>I'm in the shadow DOM</span>
  </template>
</div>
```
![[shadow-dom-declaratively.png]]
>[!NOTE]
>By default, contents of `<template>` are not displayed. In this case, because the `shadowrootmode="open"` was included, the shadow root is rendered. In supporting browsers, the visible contents within that shadow root are displayed.

After the browser parses the HTML, it replaces `<template>` element with its content wrapped in a shadow root that's attached to the parent element, the `<div id="host">` in the example. The resulting DOM tree looks like this:
```markdown
- DIV id="host"
  - #shadow-root
    - SPAN
      - #text: I'm in the shadow DOM
```

# Encapsulating from Javascript
Clicking the "Uppercase span elements" button finds all `<span>` elements in the page and changes their text to uppercase. Clicking the "Reload" button just reloads the page, so you can try again.
```html
<div id="host"></div>
<span>I'm not in the shadow DOM</span>
<br />

<button id="upper" type="button">Uppercase span elements</button>
<button id="reload" type="button">Reload</button>
```
```javascript
const host = document.querySelector("#host");
const shadow = host.attachShadow({ mode: "open" });
const span = document.createElement("span");
span.textContent = "I'm in the shadow DOM";
shadow.appendChild(span);

const upper = document.querySelector("button#upper");
upper.addEventListener("click", () => {
  const spans = Array.from(document.querySelectorAll("span"));
  for (const span of spans) {
    span.textContent = span.textContent.toUpperCase();
  }
});

const reload = document.querySelector("#reload");
reload.addEventListener("click", () => document.location.reload());
```

If you click "Uppercase span elements", you'll see that `Document.querySelectorAll()` doesn't find the elements in our shadow DOM:
![[encapsulation-from-javascript.png]]
# Element.shadowRoot and the "mode" option
With the `mode` set to `"open"`, the Javascript in the page is able to access the internals of your shadow DOM through the `shadowRoot` property of the shadow host.

This time the "Uppercase" button uses `shadowRoot` to find the `<span>` elements in the DOM:
```html
<div id="host"></div>
<span>I'm not in the shadow DOM</span>
<br />

<button id="upper" type="button">Uppercase shadow DOM span elements</button>
<button id="reload" type="button">Reload</button>
```
```javascript
const host = document.querySelector("#host");
const shadow = host.attachShadow({ mode: "open" });
const span = document.createElement("span");
span.textContent = "I'm in the shadow DOM";
shadow.appendChild(span);

const upper = document.querySelector("button#upper");
upper.addEventListener("click", () => {
  const spans = Array.from(host.shadowRoot.querySelectorAll("span"));
  for (const span of spans) {
    span.textContent = span.textContent.toUpperCase();
  }
});

const reload = document.querySelector("#reload");
reload.addEventListener("click", () => document.location.reload());
```
![[Pasted image 20260101182226.png]]
>[!NOTE]
> The attribute `mode` is a string specifying the encapsulation mode for the shadow DOM tree. This can be one of:
> - `open`
> ```javascript
> element.attachShadow({ mode: "open" });
> element.shadowRoot; // Returns a ShadowRoot object
> ```
> - `closed`
> ```javascript
> element.attachShadow({ mode: "closed" });
> element.shadowRoot; // Returns null
> ```
>You should **not consider this a strong security mechanism**, because there are ways it can be evaded. It's a more of **an indication that the page should not access** the internals of your shadow DOM tree.
# Encapsulation from CSS
This time, we'll have some CSS targeting `<span>` elements in the page:
```html
<div id="host"></div>
<span>I'm not in the shadow DOM</span>
```
```javascript
const host = document.querySelector("#host");
const shadow = host.attachShadow({ mode: "open" });
const span = document.createElement("span");
span.textContent = "I'm in the shadow DOM";
shadow.appendChild(span);
```
```css
span {
  color: blue;
  border: 1px solid black;
}
```
The page CSS does not affect nodes inside the shadow DOM:
![[Pasted image 20260101183538.png]]
# Applying styles inside the shadow DOM
There are two different ways to apply styles inside a shadow DOM tree:
- Programatically, by constructing a `CSSStyleSheet` object and attaching it to the shadow root.
- Declarative, by adding `<style>` element in a `<template>` element's declaration.
In both cases, the styles defined in the shadow DOM tree are scoped to that tree.
## Constructable stylesheets
To style page elements in the shadow DOM with constructable stylesheets, we can:
1. Create any empty `CSSStyleSheet` object
2. Set its content using `CSSStyleSheet.replace()` or `CSSStyleSheet.replaceSync()`
3. Add it to the shadow root by assigning to `ShadowRoot.adoptedStyleSheets`
Rules defined in the `CSSStyleSheet` will be scoped to the shadow DOM tree.
```html
<div id="host"></div>
<span>I'm not in the shadow DOM</span>
```

```javascript
const sheet = new CSSStyleSheet();
sheet.replaceSync("span { color: red; border: 2px dotted black;}");

const host = document.querySelector("#host");

const shadow = host.attachShadow({ mode: "open" });
shadow.adoptedStyleSheets = [sheet];

const span = document.createElement("span");
span.textContent = "I'm in the shadow DOM";
shadow.appendChild(span);
```

The styles defined in the shadow DOM tree are not applied in the rest of the page:
![[Pasted image 20260101193607.png]]
## Adding `<style>` elements in `<template>` declarations
An alternative to constructing `CSSStyleSheet` objects is to include a `<style>` element inside the `<template>` element used to define a web component.

In this case the HTML includes the `<template>` declaration
```html
<template id="my-element">
  <style>
    span {
      color: red;
      border: 2px dotted black;
    }
  </style>
  <span>I'm in the shadow DOM</span>
</template>

<div id="host"></div>
<span>I'm not in the shadow DOM</span>
```
```javascript
const host = document.querySelector("#host");
const shadow = host.attachShadow({ mode: "open" });
const template = document.getElementById("my-element");

shadow.appendChild(template.content);
```
Again, the styles defined in the `<template>` are applied only within the shadow DOM tree, and not in the rest of the page:
![[Pasted image 20260101200310.png]]
## Choosing between programmatic and declarative options
Creating a `CSSStyleSheet` and assigning it to the shadow root using `adoptedStyleSheets` allows you to create a single stylesheet and share it among many DOM trees. The will browser will parse that stylesheet once. Also, you can make dynamic changes to the stylesheet and have them propagate to all components that use the sheet.

The approach of attaching a `<style>` element is great if you want to be declarative, have few style, and don´t need to share style across different components.