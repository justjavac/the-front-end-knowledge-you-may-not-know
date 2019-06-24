# HTML DOM 级别以及一些小坑

> API (Web 或 XML 页面) = DOM + JavaScript(脚本语言)

## 问题

问题从一个异常开始。

有网友写了如下代码：

```javascript
function download() {
      console.log(1);
}
```

```html
<a onclick="download()">下载</a>
```

当点击按钮的时候，却报错了：

`Uncaught TypeError: download is not a function`

报错信息很奇怪：`download` 不是一个函数。如果我们在 devtools console 执行 `$0.download` 会得到 `""`，一个空字符串。

我们尝试把 `<a>` 换成 `<button>`，可以正常执行。

## 解读

先说答案：HTML 5 中为 `<a>` 增加了 `download` 属性，所以在 a 上调用 `download()` 会提示 `download is not a function`，因为**所有的属性都是字符串**。

同样的坑还有：如果把表单里面的某个控件 `id` 设置为 `submit`，会导致表单提交 `form.submit()` 出错，错误信息和这个类似，`submit is not a function`。

### DOM

实际上没有没有 DOM0 级的官方标准。

DOM 是 Netscape 最早提出，并且与 JS 的诞生是在同一个时间。Netscape2 浏览器首先实现了 DOM，定义了获取文档中一些元素的入口，比如 `document.forms` 和 `document.images`，后期的浏览器为了实现向后兼容，同样也支持这些接口。**在 JS 事件中，我们经常提及的 DOM 事件，也是在这个阶段定义的**。其它比较常用的还有 CSS 访问、DOM 遍历、等。。。

DOM(Document Object Model) 的繁荣可以追溯至 1990 年代后期微软与 Netscape 的“浏览器大战”，双方为了在 JavaScript 与 JScript 一决生死，于是大规模的赋予浏览器强大的功能。微软在网页技术上加入了不少专属事物，计有 VBScript、ActiveX、以及微软自家的 DHTML 格式等，使不少网页使用非微软平台及浏览器无法正常显示。

1998 年 10 月，DOM1 成为了 W3C 的推荐标准。DOM1 级由两个模块组成：DOM核心（DOM Core）和DOM HTML。分别定义了：

- DOM核心：针对任何结构化文档的标准模型
- DOM HTML：只针对HTML文档的标准模型

DOM1 就像一个刚出生的孩子，肯定有各种不足。于是各种浏览器都在原来的基础上添加新的私有 API，比如 `attachEvent` 和 `addEventListener`，于是 W3C 又推出了 DOM level 2。而这个版本最大的变化就是 **添加事件监听的方法统一成了 `addEventListener`**，并增加了第三个参数。（关于第三个参数可以看我之前的文章 #6 ）

W3C 在 2000 年推出 DOM level 2，直到 9 年后的 2009 年，微软发布的 IE9 才遵循这个标准。WTF！！！

后来陆续推出了 DOM3 和 DOM 4。

## 参考链接

- [DOM概述 - Web API 接口 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction)
- [文档对象模型 (DOM) - Web API 接口 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model)
- 探索 DOM Level 3 Core 的关键特性 [第 1 部分](https://www.ibm.com/developerworks/cn/xml/x-keydom/) [第 2 部分](https://www.ibm.com/developerworks/cn/xml/x-keydom2/index.html)
- [DOM级别 | GCidea&#39;s blog](http://gcidea.info/2016/09/12/dom-rank/)

-----------

> 阅读原文：[HTML DOM 级别以及一些小坑](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know/blob/master/archives/014-dom-level.md)
>
> 讨论地址：[#14](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know/issues/14)
> 
> 如果你想参与讨论，请[点击这里](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know)
