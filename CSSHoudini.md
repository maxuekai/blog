# CSS Houdini

> 想要使用某种 CSS 特性，但是因为浏览器兼容性问题而没法使用？更糟糕一点，所有浏览器都支持这种特性，但支持度不完全，在某些情况下会有 bug 出现、支持性不一致，甚至于完全不兼容。如果你曾经遇到过上述情况（我肯定你一定遇到过），那你得好好关注 Houdini。

可以说 `CSS Houdini` 是近几年大家比较关注的 CSS 技术，它的终极目标是实现 css 属性的完全兼容，它提出了一种设想：将 CSS API 开放给开发者，使其可以通过这套接口自行扩展 CSS

## CSS Houdini 是什么？

让我们先来回顾一下浏览器在渲染网页的过程中，经历了什么？

![浏览器渲染网页流程](./img/browser_render.png)

开发者们能操作的就是通过 JS 去控制 `DOM` 与 `CSSOM`，来影响页面的变化，但是对于接下來的 `Layout`、`Paint` 与 `Composite` 就几乎沒有控制权了

CSS Houdini 是由一群来自 Mozilla, Apple, Opera, Microsoft, HP, Intel, IBM, Adobe 与 Google 的工程师所组成的工作小组，志在建立一系列的 API，让开发者能够介入浏览器的 CSS engine 操作，带给开发者更多的解決方案，用来解決 CSS 长久以来的问题：
* css 属性跨浏览器的问题
* css 编译转换成支持属性的制作困难

## CSS Houdini 提供的 API

* [Typed OM](https://drafts.css-houdini.org/css-typed-om/)
* [Properties / Values](https://drafts.css-houdini.org/css-properties-values-api/)
* [Worklets](https://drafts.css-houdini.org/worklets/)
* [Paint API](https://drafts.css-houdini.org/css-paint-api/)
* [Layout API](https://drafts.css-houdini.org/css-layout-api/)
* [Animation Worklet](https://wicg.github.io/animation-worklet/)
* [Parser API](https://wicg.github.io/CSS-Parser-API/)
* [Font Metrics](https://github.com/w3c/css-houdini-drafts/blob/master/font-metrics-api/README.md)

可以在 [Is Houdini Ready yet](https://ishoudinireadyyet.com/) 这里查到主流浏览器对这些 api 的支持情况

## CSS Type OM

`CSS Typed OM`可以被认为是现在使用的`CSSOM`的第二个版本。它的目标是解决很多现有模型的问题并且会引入`新的CSS解析器API`和`CSS属性和值API`的特性。

Typed OM的另一个主要目标是`改进性能`，将当前CSSOM的字符串值转化成`有意义的类型化的 JavaScript 表达式`会产生显著的性能提升。

旧的 CSSOM 获取 DOM 的样式返回的都是字符串，如
```js
el.style.opacity = 0.3;
typeof el.style.opacity;  // string
```
由于返回的样式都是字符串，所以我们处理的时候都需要再做一层类型转换；而 CSS Type OM 可以帮助我们获取到类型化的数据，我们只需要按照对应的类型做直接处理即可
```js
el.attributeStyleMap.set('opacity', 0.3);
typeof el.attributeStyleMap.get('opacity').value; // number
```
`attributeStyleMap` 属性用来获取元素样式表规则，返回一个 `StylePropertyMap` 对象。 StylePropertyMap 对象类似 Map 对象，所以它们支持所有常见的操作`get / set / keys / values / entries`，处理起来更加灵活高效