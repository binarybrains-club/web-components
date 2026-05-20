# ¿Por qué se llama "Light DOM"?
Antes de que existieran los Web Components, solo existía el DOM. Pero cuando se introdujo el concepto de Shadow DOM, se necesitó una forma de referirse a los elementos fuera de ese túnel.
- **Light DOM**: Es el DOM tradicional. Son los nodos que escribes dentro de una etiqueta HTML estándar o los hijos de un componente que son visibles para el navegador globalmente.
- **Shadow DOM**: Es el DOM privado de un componente. No es accesible desde fuera usando selectores globales como `document.querySelector`, lo que evita que los estilos CSS se "filtren" hacia adentro o hacia afuera.

Los elementos que escribes dentro de las etiquetas de un Web Component pertenecen al Light DOM y por lo tanto se comportan como cualquier otro elemento en la página.
## ¿Qué significa que sean globalmente accesibles?
Significa que el navegador los trata como "hijos" directos del componente en el árbol principal. Esto es lo que se puede y no se puede hacer:
- **Javascript global**: Puedes seleccionarlos usando `document.querySelector("your-component p")` sin problemas.
- **CSS global**: Si tienes alguna regla en tu archivo CSS general que diga `p { color: red; }`, afectará a esos elementos, incluso si están "dentro" del componente.
- **Eventos**: Los eventos (como un `click`) que ocurren en esos elementos burbujean hasta `window` naturalmente.

Para que esos elementos del Light DOM sean visibles dentro de tu componente, el Web Component debe usar una etiqueta llamada `<slot>`.

Imagina que el Shadow DOM es una caja cerrada con un agujero (el slot). El Light DOM es un objeto que colocas fuera de la caja, pero que se asoma a través de ese agujero. El objeto sigue estando "afuera" (es globalmente accesible), pero visualmente parece estar dentro de la caja.

**Ejemplo práctico**
```html
<my-card>
  <h1 class="title">Hello World</h1> 
</my-card>

<script>
  // Esto funciona perfectamente
  const titulo = document.querySelector('.title');
  console.log(titulo.textContent); // Imprime: "Hola Mundo"
</script>
```
# Diferencias clave

| Característica | Light DOM (Tradicional)                         | Shadow DOM (Encapsulado)                                      |
| -------------- | ----------------------------------------------- | ------------------------------------------------------------- |
| Acceso         | Accesible por cualquier script global           | Privado del componente                                        |
| Estilos        | Los estilos globales lo afectan                 | Los estilos están aislados (encapsulados)                     |
| Definición     | Es el marcado que el usuario del componente escribe | Es la estructura interna definida por el creador del componente |

**Ejemplo rápido**

Si tienes un componente de calendario personalizado:
- El Light DOM sería el texto o las etiquetas que pones entre `<my-calendar>...</my-calendar>`
- El Shadow DOM sería toda la lógica de botones, tablas y números que el componente construye internamente para funcionar, pero que no ves en el HTML principal

>[!NOTE]
>**Dato curioso**: Se llama "Light" no porque sea ligero, sino para contrastar con "Shadow". Es el DOM que está "bajo la luz" del documento principal.
