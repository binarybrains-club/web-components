# La verdad sobre las templates
Cuando tienes que reutilizar las mismas estructuras de marcado repetidamente en una página web, tiene sentido usar algún tipo de plantilla en lugar de repetir la misma estructura una y otra vez. Esto es posible mediante el elemento HTML `<template>`. Este elemento y su contenido no se renderizan en el DOM, pero aún se puede hacer referencia a él usando Javascript.
```html
<template id="custom-paragraph">
  <p>My paragraph</p>
</template>
```
Esto no aparecerá en tu página hasta que obtengas una referencia a él con Javascript y luego lo añadas al DOM, usando algo como lo siguiente:
```javascript
let template = document.getElementById("custom-paragraph");
let templateContent = template.content;
document.body.appendChild(templateContent);
```
# Usando templates con web components
Las templates son útiles por sí solas, pero funcionan aún mejor con web components. Definamos un web component que use nuestra template como el contenido de su shadow DOM.
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
El punto clave a notar aquí es que añadimos un clon del contenido de la template al shadow root, creado usando el método `Document.importNode()`.

Y debido a que estamos añadiendo su contenido a un shadow DOM, podemos incluir información de estilos dentro de la template en un elemento `<style>`, que luego queda encapsulado dentro del elemento personalizado. Esto no funcionaría si simplemente lo añadiéramos al DOM estándar.
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

Ahora podemos usarlo simplemente añadiéndolo a nuestro documento HTML:
```html
<my-paragraph></my-paragraph>
```
# Añadiendo flexibilidad con slots
Podemos hacer posible mostrar diferente texto en cada instancia del elemento de una manera declarativa usando el elemento `<slot>`.

Los slots se identifican por su atributo `name`, y permiten definir marcadores de posición en tu template que pueden llenarse con cualquier fragmento de marcado que quieras cuando el elemento se usa en el marcado.

Entonces, si queremos añadir un slot a nuestro ejemplo trivial, podríamos actualizar el elemento párrafo de nuestra template así:
```html
<p><slot name="my-text">My default text</slot></p>
```
>[!IMPORTANT]
>El atributo `name` debe ser único por shadow root: si tienes dos slots con el mismo nombre, todos los elementos con un atributo `slot` coincidente serán asignados al primer slot con ese nombre.

Si el contenido del slot no está definido cuando el elemento se incluye en el marcado, o si el navegador no soporta slots, `<my-paragraph>` simplemente contiene el contenido de respaldo "My default text".

Para definir el contenido del slot, incluimos una estructura HTML dentro del elemento `<my-paragraph>` con un atributo `slot` cuyo valor es igual al nombre del slot que queremos llenar.
```html
<my-paragraph>
  <span slot="my-text">Let's have some different text!</span>
</my-paragraph>
```
o
```html
<my-paragraph>
  <ul slot="my-text">
    <li>Let's have some different text!</li>
    <li>In a list!</li>
  </ul>
</my-paragraph>
```
>[!NOTE]
>El atributo `slot` no necesita ser único: un `<slot>` puede ser llenado por múltiples elementos que todos tengan un atributo `slot` coincidente.
>```html
><my-paragraph>
>	<span slot="my-text">This will be displayed</span>
>	<span slot="my-text">This will be displayed twice</span>
 ></my-paragraph>
>```

Los atributos `name` y `slot` ambos por defecto son cadena vacía, por lo que los elementos sin atributo `slot` son asignados al `<slot>` sin atributo `name` (el slot sin nombre, o slot por defecto).
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
En este ejemplo:
- El contenido con `slot="my-text"` va al slot nombrado.
- Todo el demás contenido va automáticamente al slot sin nombre.
# Un ejemplo más completo
>[!NOTE]
>Técnicamente es posible usar el elemento `<slot>` sin un elemento `<template>`, por ejemplo dentro de un elemento `<div>` regular, y aún así aprovechar las características de marcador de posición de `<slot>` para contenido Shadow DOM, y hacerlo puede evitar la pequeña molestia de tener que acceder primero a la propiedad content del elemento template (y clonarlo). Sin embargo, generalmente es más práctico añadir slots dentro de un elemento `<template>`, ya que es poco probable que necesites definir un patrón basado en un elemento ya renderizado.
>Además, incluso si no está ya renderizado, el propósito del contenedor como template debería ser semánticamente más claro cuando se usa [`<template>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/template). Además, [`<template>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/template) puede tener elementos directamente añadidos, como [`<td>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/td), que desaparecerían cuando se añaden a un [`<div>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/div).

## Creando una template con algunos slots
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
## Usando el elemento personalizado `<element-details>` con slots nombrados
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
