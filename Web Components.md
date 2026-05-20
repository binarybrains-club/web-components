#WebComponents
Es un conjunto de diferentes tecnologías que permiten crear elementos personalizados reutilizables, con su funcionalidad encapsulada del resto del código, y utilizarlos en aplicaciones web.

Consta de tres tecnologías principales, que pueden usarse juntas para crear elementos personalizados versátiles con funcionalidad encapsulada que pueden reutilizarse donde sea sin temor a colisiones de código.
- [[Custom Elements]]: Un conjunto de APIs de Javascript que permiten definir elementos personalizados y su comportamiento, que luego pueden usarse según se desee en la interfaz de usuario.
- [[Shadow DOM]]: Un conjunto de APIs de Javascript para adjuntar un árbol DOM "sombra" encapsulado a un elemento, que se renderiza por separado del DOM del documento principal, y controlar la funcionalidad asociada.
- [[Templates and slots]]: Los elementos `<template>` y `<slot>` permiten escribir plantillas de marcado que no se muestran en la página renderizada. Estas pueden luego reutilizarse múltiples veces como base de la estructura de un elemento personalizado.

El enfoque básico para implementar un web component generalmente se ve así:
1. Crear una clase en la que se especifica la funcionalidad del web component, usando la sintaxis de [[Javascript Classes |clase]].
2. Registrar un nuevo elemento personalizado usando el método `CustomElementRegistry.define()`, pasándole el nombre del elemento a definir, la clase o función en la que se especifica su funcionalidad, y opcionalmente, qué elemento hereda.
3. Si es necesario, definir una plantilla HTML usando `<template>` y `<slot>`. De nuevo, usar métodos DOM regulares para clonar la plantilla y adjuntarla al shadow DOM.
4. Usar el elemento personalizado donde sea en la página, como se haría con cualquier elemento HTML normal.
