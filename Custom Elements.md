#WebComponents 
# Type of custom element
There are two types of custom element:
- **Autonomous custom elements** [[Inheritance|inherits]] from the HTML element base [[Classes and Objects#Classes|class]] `HTMLElement`. You have to implement their behavior from scratch.
```javascript
class CustomElement extends HTMLElement{
	// ...
}
```
- **Customized built-in elements** inherit from standard HTML elements such as `HTMLImageElement` or `HTMLParagraphElement`. Their implementation extends the behavior of select instances of the standard element.
```javascript
class CustomImageElement extends HTMLImageElement{
	// ...
}
```

For both kinds of custom element, the basic steps to create and use them are the same:
1. Implement its behavior by defining a Javascript class.
2. Register the custom element to the current page.
3. Finally, you can use the custom element in the HTML or Javascript code.
# Implementing a custom Element

Implementation of a minimal custom element that customizes the `<p>` element:
```javascript
class WordCount extends HTMLParagraphElement {
	constructor(){
		super();
	}
}
```
Implementation of a minimal autonomous custom element:
```javascript
class PopUpInfo extends HTMLElement {
	constructor(){
		super();
	}
}
```
>[!NOTE]
>In the class `constructor`, can be set up initial state and default values, register event listeners and perhaps create a shadow root.
## Custom element lifecycle callbacks
Once your custom element is registered, the browser call certain methods of your class when code in the page interacts with your custom element in certain ways. By providing an implementation of these [[Classes and Objects#Defining Methods|methods]]. which the specification calls *lifecycle callbacks*, you can run code in response to these events.

Custom element lifecycle callbacks include:
- `connectedCallback()`: Called each time the element is added to the document.
>[!NOTE]
>The specification recommends that developers should implement custom element setup in this callback rather than the constructor.
- `disconnectedCallback()`: Called each time the element is removed from the document.
- `connectedMoveCallback()`: When defined, this is called instead of `connectedCallback()` and `disconnectedCallback()` each time the element is moved to a different place in the DOM via `Element.moveBefore()`. 
>[!NOTE]
>Use this to avoid running initialization/cleanup code in the `connectedCallback()` and `disconnectedCallback()` callbacks when the element is not actually being added to or removed from the DOM. See [[#Lifecycle callbacks and state-preserving moves]]
- `adoptedCallback()`: Called each time the element is moved to a new document.
-  `attributeChangedCallback()`: Called when attributes are changed, added, removed, or replaced. See [[#Responding to attribute changes]].
![[web-component-lifecycle.png]]

```javascript
// Create a class for the element
class MyCustomElement extends HTMLElement {
  // List of attributes that will be observed, which will trigger `attributeChangedCallback`
  static observedAttributes = ["color", "size"];

  constructor() {
    // Always call super first in constructor
    super();
  }

  connectedCallback() {
    console.log("Custom element added to page.");
  }

  disconnectedCallback() {
    console.log("Custom element removed from page.");
  }

  connectedMoveCallback() {
    console.log("Custom element moved with moveBefore()");
  }

  adoptedCallback() {
    console.log("Custom element moved to new page.");
  }

  attributeChangedCallback(name, oldValue, newValue) {
    console.log(`Attribute ${name} has changed.`);
  }
}

customElements.define("my-custom-element", MyCustomElement);
```
## Lifecycle callbacks and state-preserving moves

The position of a custom element in the DOM can be manipulated just like any regular HTML element, but there are lifecycle side-effects to consider.
Each time a custom element is moved (via methods such as `Element.moveBefore()` or `Node.insertBefore()`), the `disconnectedCallback()` and `connectedCallback()` lifecycle callbacks are fired, because the element is disconnected from and reconnected to the DOM.
If you want to preserve the element's state, you can do so by defining a `connectedMoveCallback()` lifecycle callback inside the element class, and the using the `Element.moveBefore()` method to move the element (instead of similar methods such as `Node.insertBefore()`). This causes the `connectedMoveCallback()` to run instead of `connectedCallback()` and `disconnectedCallback()`.
You could add an empty `connectedMoveCallback()` to stop the other two callbacks running, or include some custom logic to handle the move:
```javascript
class MyComponent {
  // ...
  connectedMoveCallback() {
    console.log("Custom move-handling logic here.");
  }
  // ...
}
```

# Registering a custom element
To make a custom element available in a page, call the `define()` method of `window.customElements`.
The `define()` method takes the following arguments:
- `name`: Name of the element. This must start with a lowercase letter contain a hyphen, and satisfy certain other rules listed in the specification
>[!NOTE]
> A string name is a valid custom element name if all of the following are true:
> - *name* is a valid element local name
> - *name*'s 0th code pount in as ASCII lower alpha
> - *name* does not contain any ASCII upper alphas
> - name is not one of the following ():
> 	- "annotation-xml"
> 	- color-profile"
> 	- "font-face"
> 	- "font-face-src"
> 	- "font-face-uri"
> 	- "font-face-format"
> 	- "font-face-name"
> 	- "missing-glyph"
> 	- ... other hyphen-containing element names from the applicable specificacions, namely SVG2 and MathML
- `constructor`: The custom element's constructor function.
- `options`: Only included for customized built-in elements, this is an object containing a single property a `extends`, which is a string naming the built-in element to extend

For example, this code registers the  `WordCount`  customized built-in element:
```javascript
customElements.define("word-count", WordCount, {
	extends: "p",
});
```

This code registers the `PopupInfo` autonomous custom element:
```javascript
customElements.define("popup-info", PopupInfo);
```
# Using a custom element
To use a customized built-in element, use the built-in element but with the custom name as the value of the `is` attribute:
```html
<p is="word-count"></p>
```
To use an autonomous custom element, use the custom name just like a built-in HTML element:
```html
<popup-info>
  <!-- content of the element -->
</popup-info>
```
# Responding to attribute changes
Like built-in elements, custom elements can use HTML attributes to configure the element's behavior. To use attributes effectively, an element has to be able to respond to changes in an attribute's value. To do this, a custom element needs to add the following members to the class that implements the custom element:
- A static property named `observedAttributes`. This must be an array containing the names of all attributes for which the element needs change notifications
```javascript
class CustomElement extends HTMLElement {
	static observedAttributes = ["size"]; 
	// ...
}

class MyElement extends HTMLElement {
	static get observedAttributes(){
		return ["title", "count"];
	}
}
```
- An implementation of the `attributeChangedCallback()` lifecycle callback
```javascript
class CustomElement extends HTMLElement {
	// ...
	
	attributedChangedCallback(name, oldValue, newValue){
		//...
	}
}
```
The `attributeChangedCallback()` callback is then called whenever an attribute whose name is listed in the element's `observedAttributes` property is added, modified, removed, or replaced.

The callback is passed three arguments:
- The name of the attribute changed.
- The attribute's old value
- The attribute's new value
```javascript
// Create a class for the element
class MyCustomElement extends HTMLElement {
  static observedAttributes = ["size"];

  constructor() {
    super();
  }

  attributeChangedCallback(name, oldValue, newValue) {
    console.log(
      `Attribute ${name} has changed from ${oldValue} to ${newValue}.`,
    );
  }
}

customElements.define("my-custom-element", MyCustomElement);
```
Note that if the element's HTML declaration includes an observed attribute, then `attributeChangedCallback()` will be called after the attribute is initialized, when the element's declaration is parsed for the first time. So in the following example, `attributeChangedCallback()` will be called when the DOM is parsed, even if the attribute is never changed again:
```html
<my-custom-element size="100"></my-custom-element>
```
## Custom states and custom state pseudo-class CSS selectors

Built-in HTML elements can have different states, such as "hover", "disabled", and "read only". Some of these states can be set as attributes using HTML or Javascript, while others are internal, and cannot. Whether external or internal, commonly these states have corresponding CSS pseudo-classes that can be used to select and style the element when it is in a particular state.

Autonomous custom elements (but not elements based on built-in elements) also allow you to define states and select against them using the `:state()` pseudo-classes function. The code below shows how this works using the example of an autonomous custom element that has an internal state `"collapsed"`.

The `collapsed` state is represented as a boolean property that is not visible outside of the element. To make this state selectable in CSS the custom element first calls `HTMLElement.attachInternals()` in its constructor in order to attach an [ElementInternals](https://developer.mozilla.org/en-US/docs/Web/API/ElementInternals) object, which in turn provides access to a [CustomStateSet](https://developer.mozilla.org/en-US/docs/Web/API/CustomStateSet) through the `ElementInternals.state` property. The setter for the (internal) collapsed state adds the _identifier_ `hidden` to the `CustomStateSet` when the state is `true`, and removes it when the state is `false.
```javascript
class MyCustomElement extends HTMLElement {
  constructor() {
    super();
    this._internals = this.attachInternals();
  }

  get collapsed() {
    return this._internals.states.has("hidden");
  }

  set collapsed(flag) {
    if (flag) {
      // Existence of identifier corresponds to "true"
      this._internals.states.add("hidden");
    } else {
      // Absence of identifier corresponds to "false"
      this._internals.states.delete("hidden");
    }
  }
}

// Register the custom element
customElements.define("my-custom-element", MyCustomElement);
```

We can use the identifier added to the custom element's `CustomStateSet` (`this._internals.states`) for matching the element's custom state. This is matched by passing the identifier to the `:state()` pseudo-class. or example, below we select on the hidden state being true (and hence the element's collapsed state) using the `:hidden` selector, and remove the border.

```css
my-custom-element {
  border: dashed red;
}
my-custom-element:state(hidden) {
  border: none;
}
```
The `:state()` pseudo-class can also be used within the `:host()` pseudo-class function to match a custom state [within a custom element's shadow DOM](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/:state#matching_a_custom_state_in_a_custom_elements_shadow_dom). Additionally, the `:state()` pseudo-class can be used after the [`::part()`](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Shadow_parts) pseudo-element to match the shadow parts of a custom element that is in particular state.


Check [[Shadow DOM#Applying styles inside the shadow DOM]]
```typescript
export class CustomElement extends HTMLElement {
  _internals;
  constructor() {
    super();
    this._internals = this.attachInternals();
  }

  connectedCallback() {
    this.attachShadow({
      mode: "open",
    });

    const style = document.createElement("style");
    style.textContent = `
      :host {
        border: dashed red;
      }
      :host(:state(hidden)){
        border: none;
      }
    `;
    this.shadowRoot?.appendChild(style);
  }

  get collapsed() {
    return this._internals.states.has("hidden");
  }

  set collapsed(flag: boolean) {
    if (flag) {
      this._internals.states.add("hidden");
    } else {
      this._internals.states.delete("hidden");
    }
  }
}

customElements.define("custom-element", CustomElement);
```

```javascript
const styles = new CSSStyleSheet();
styles.replaceSync(`
      :host {
        border: dashed red;
      }
      :host(:state(hidden)){
        border: none;
      }
`);

export class CustomElement extends HTMLElement {
  _internals;
  constructor() {
    super();
    this._internals = this.attachInternals();
  }

  connectedCallback() {
    const shadow = this.attachShadow({
      mode: "open",
    });

    shadow.adoptedStyleSheets.push(styles);
  }

  get collapsed() {
    return this._internals.states.has("hidden");
  }

  set collapsed(flag: boolean) {
    if (flag) {
      this._internals.states.add("hidden");
    } else {
      this._internals.states.delete("hidden");
    }
  }
}

customElements.define("custom-element", CustomElement);
```