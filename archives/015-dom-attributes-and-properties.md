# HTML attribute 和 DOM property

在大多数的文章中，attribute 一般被翻译为“特性”，property 被译为“属性”。

## TLDR;

HTML attribute | DOM property
------------ | -------------
值永远是字符串或 `null` | 值可以是任意合法 js 类型
大小写不敏感 | 大小写敏感
不存在时: 如果是标准属性, 返回 `""`; 如果是非标准属性, 返回 `null` | 不存在是返回 `undefined`
对于 `href`, 返回 html 设置的值 | 对于 `href` 返回解析后的完整 uri
更新 `value`, 属性也更新 | 更新 `value`, 特性不更新

## 概述

当我们书写 HTML 代码的时候，我们为 HTML <abbr title="Element">元素</abbr>设置<abbr title="attribute">特性</abbr> ，例如：

```html
<input id="name" value="justjavac" />
```

我们写了一个 `input` 标签，并给他定义了 2 个<abbr title="attribute">特性</abbr> (`id` 和 `value`)。当浏览器解析这段代码的时候，会把 html 源码解析为 DOM 对象，确切的说是解析为了 [`HTMLInputElement`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLInputElement) 对象。`HTMLInputElement` 的继承关系是：

```
HTMLInputElement
  ↓
HTMLElement
  ↓
Element
  ↓
Node
  ↓
EventTarget
  ↓
Object
```

通过查看文档会发现，`HTMLInputElement` 的原型上定义了很多<abbr title="property">属性</abbr>和方法，例如 `form`, `name`, `type`, `alt`, `checked`, `src`, `value` 等等。还有从 `HTMLElement` 继承来的 `id`, `title`, `clientTop` 等等。

如果仔细找找，就不难发现其中就有我们为 `input` 标签定义的<abbr title="attribute">特性</abbr>：`id` 和 `value`。**当浏览器解析网页时，将 HTML <abbr title="attribute">特性</abbr>映射为了 DOM <abbr title="property">属性</abbr>**。

而 `Element` 类还有一个 [`attributes`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/attributes) 属性，里面包含了所有的特性。

但是，**HTML attribute 和 DOM property 并不总是一对一的关系**。

## 1. DOM 属性

当浏览器解析完 HTML 后，生成的 DOM 是一个继承自 Object 的常规 JavaScript 对象，因此我们可以像操作任何 JS 对象那样来操作 DOM 对象。

```js
const el = document.getElementById('name')
el.foo = 'bar'
el.user = { name: 'jjc', age: '18'}
```

也可以为其添加方法。如果你想给每个 html 元素都添加属性或方法，甚至可以直接修改 `Element.prototype`，不过我们不推荐这么做。

## 2. HTML 特性

和 DOM 属性类似，除了那些规范里定义的标准特性外，HTML 也可以添加非标准的属性，例如：

```html
<input id="name" value="justjavac" foo="bar" />
```

当 HTML 特性映射为 DOM 属性时，只映射标准属性，**访问非标准属性将得到 `undefined`**。

```js
const el = document.getElementById('name')
el.foo === undefined
```

好在 DOM 对象也提供了操作特性的 API：

- `Element.hasAttribute(name)` – 判断某个特性是否操作
- `elem.getAttribute(name`) – 获取指定特性的值
- `elem.setAttribute(name, value)` – 设置指定特性的值
- `elem.removeAttribute(name)` – 移除指定特性

以上 API 定义在 `Element` 上。

根据 HTML 规范，标签以及特性名是不区分大小写的，因此以下代码是一样的：

```js
el.getAttribute('id')
el.getAttribute('ID')
el.getAttribute('iD')
```

并且，**特性永远都是字符串或 `null`**。如果我们为特性设置非字符串的值，则引擎会将此值转换为字符串。属性是具有类型的：

```js
el.getAttribute('checked') === '' // 特性是字符串
el.checked === false              // 属性是 boolean 类型的值

el.getAttribute('style') === 'color:blue' // 特性是字符串
typeof el.style === 'object'                 // 属性是 CSSStyleDeclaration 对象
```

即使都是字符串，属性和特性也可能不同，有一个例外就是 `href`：

```js
el.getAttribute('href') === '#tag' // 特性原样返回 html 设置的值
el.href === 'http://jjc.fun#tag'   // 属性返回解析后的完整 uri
```

## 3. 特性和属性的同步

当标准的特性更新时，对应的属性也会更新；反之亦然。

但是 `input.value` 的同步是单向的，只是 `attribute --> property`。当修改特性时，属性也会更新；但是修改属性后，特性却还是原值。

```js
el.setAttribute('value', 'jjc');  // 修改特性
el.value === 'jjc'                // 属性也更新了  

el.value = 'newValue';            // 修改属性 
el.getAttribute('value')) === 'jjc' // 特性没有更新
```

## 4. 非标准特性

非标准 HTML 特性并不会自动映射为 DOM 属性。当我们实用 `data-` 开头的特性时，会映射到 DOM 的 dataset 属性。中划线格式会变成驼峰格式：

```js
el.setAttribute('data-my-name', 'jjc');
el.dataset.myName === 'jjc'

el.setAttribute('data-my-AGE', 18);
el.dataset.myAge === '18'
```

## 自定义特性 VS 非规范特性

HTMl 允许我们自定义标签，也可以扩展标签的特性，但是我们推荐使用已经进入 HTML5 规范的自定义特性 `data-*`。比如我们想为` div` 标签增加一个 `age` 特性，我们可以有 2 种选择：

```html
<div age="18">justjavac</div>
<div data-age="18">justjavac</div>
```

虽然第一种代码更短，但是却有一个潜在的风险。因为 HTML 规范是一直发展变化的，也许在未来的某个版本中，`age` 被添加进了标准特性里面，这将会引起潜在的 bug。

## 参考文献和资料

- [What is the difference between properties and attributes in HTML?](https://stackoverflow.com/questions/6003819/what-is-the-difference-between-properties-and-attributes-in-html)
- [.prop() vs .attr()](https://stackoverflow.com/questions/5874652/prop-vs-attr)
- [HTML5 - 28 October 2014](https://www.w3.org/TR/html50/forms.html#the-input-element)
- [Attributes and properties](https://javascript.info/dom-attributes-and-properties)
- [getAttribute() versus Element object properties?](https://stackoverflow.com/questions/10280250/getattribute-versus-element-object-properties?noredirect=1&lq=1)

-----------

> 阅读原文：[HTML attribute 和 DOM property](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know/blob/master/archives/015-dom-attributes-and-properties.md)
>
> 讨论地址：[#15](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know/issues/15)
> 
> 如果你想参与讨论，请[点击这里](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know)
