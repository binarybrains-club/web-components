# Why name it "Light DOM"?
Before [[Web Components]] existed, there was only the DOM. But the concept the concept of [[Shadow DOM]] was introduced, a way was needed to refer to elements outside that tunnel
- **Light DOM**: This is the traditional DOM. These area the nodes you write inside a standard HTML tad or the children of a component that are visible to the browser globally.
- **Shadow DOM**: This is a component's private DOM. It is not accessible from the outside using global selectos such as `document.querySelector`, which prevents CSS styles from "leaking" in or out.

The elements you write inside the tags of a Web Component belong to the Light DOM and therefore behave like any other element on the page
## What does it mean for them to be globally accessible?
It means that the browser treats them as direct "children" of the component in the main tree. Here's what you can and can't do:
- **Global Javascript**: You can select them using `document.querySelector("your-component p")` without any problems.
- **Global CSS**: If you have any rule in your general CSS file that says `p { color: red; }`, it will affect those elements, even if they are "inside" the component.
- **Events**: Event (such as a `click`) that occur on those elements bubble up to the `window` naturally

In other for those Light DOM elements to be visible within your component, the Web Component must use a tag called [[Templates and slots#Adding flexibility with slots|`<slot>`]]

Image that the Shadow DOM is a closed box with a hole (the slot). The Light DOM is an object that you place outside the box, but which peeks through that hole. The object is still "outside" (it is globally accessible), but visually it appears to be inside the box.

**Practical example**
```html
<my-card>
  <h1 class="title">Hello World</h1> 
</my-card>

<script>
  // Thiw works perfectly
  const titulo = document.querySelector('.title');
  console.log(titulo.textContent); // Prints: "Hola Mundo"
</script>
```
# Key Differences

| Feature    | Light DOM (Traditional)                         | Shadow DOM (Encapsulado)                                      |
| ---------- | ----------------------------------------------- | ------------------------------------------------------------- |
| Access     | Accesible by any global script                  | Private to the component                                      |
| Styles     | Global styles affect it                         | Styles are isolated (encapsulated)                            |
| Definition | It is the markup that the component user writes | It is the internal structure defined by the component creator |
**Quick Example**
If you have a custom calendar component:
- The Light DOM would be the text or labels you put bewteen `<my-calendar>...</my-calendar>`
- The Shadow DOM would be all the logic of buttons, tables, and numbers that the component builds internally to function, but that you don't see in the main HTML

>[!NOTE]
>**Fun fact**: It's called “Light” not because it's lightweight, but to contrast with “Shadow.” It's the DOM that is “under the light” of the main document.