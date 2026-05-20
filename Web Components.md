#WebComponents
It is a suite of different technologies that allows to create reusable custom elements, with their functionality encapsulated away from the rest of the code, and utilize them in web apps.

It consists of three man technologies, which can be used together to create versatile custom elements with encapsulated functionality that can be reused wherever you like without fear of code colisions.
- [[Custom Elements]]: A set of Javascript APIs that allow you to define custom elements and their behavior, which can then be used as desired in your user interface.
- [[Shadow DOM]]:  A set of Javascript APIs for attaching an encapsulated "shadow" DOM tree to an element, which is rendered separately from the main document DOM, and controlling associated functionality.
- [[Templates and slots]]: The `<template>` and `<slot>`elements enable to write markup templates that are not displayed in the rendered page. These can the be reused multiple times as the basis of a custom element's structure.

The basic approach for implementing a web component generally looks something like this:
1. Create a class in which you specify your web component functionality, using the [[Javascript Classes |class]] syntax.
2. Register new custom element using the  `CustomElementRegistry.define()`method, passing it the element name to be defined, the class or function in which its functionality is specified, and optionally, what element it inherits from.
3. If required, define an HTML template using `<template>` and `<slot>`. Again use regular DOM methods to clone the template and attach it to your shadow DOM.
4. Use your custom element wherever you like on your page, just like you would any regular HTML element.