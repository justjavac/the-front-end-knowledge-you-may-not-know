# #23 Async Clipboard API：异步剪贴板 API

原文：[Unblocking Clipboard Access](https://developers.google.com/web/updates/2018/03/clipboardapi)  
作者：@developit

--------------------

在过去的几年里我们只能使用 [`document.execCommand`](https://developers.google.com/web/updates/2015/04/cut-and-copy-commands) 来操作剪贴板。不过，这种操作剪贴板的操作是同步的，并且只能读取和写入 DOM。

现在 Chrome 66 已经支持了新的 [Async Clipboard API](https://www.w3.org/TR/clipboard-apis/#async-clipboard-api)，作为 `execCommand` 替代品。

这个新的 Async Clipboard API 还可以使用 Promise 来简化剪贴板事件并将它们与 Drag-&-Drop API 一起使用。

演示视频：https://zhuanlan.zhihu.com/p/34698155

## 复制：将文本写入剪贴板

`writeText()` 可以把文本写入剪切板。`writeText()` 是异步的，它返回一个 Promise：

```js
navigator.clipboard.writeText('要复制的文本')
  .then(() => {
    console.log('文本已经成功复制到剪切板');
  })
  .catch(err => {
    // This can happen if the user denies clipboard permissions:
    // 如果用户没有授权，则抛出异常
    console.error('无法复制此文本：', err);
  });
```

还可以使用[异步函数](http://esnext.justjavac.com/proposal/ecmascript-asyncawait.html) 的 `async` 和 `await`：

```js
async function copyPageUrl() {
  try {
    await navigator.clipboard.writeText(location.href);
    console.log('Page URL copied to clipboard');
  } catch (err) {
    console.error('Failed to copy: ', err);
  }
}
```

## 粘贴：从剪贴板中读取文本

和复制一样，可以通过调用 `readText()` 从剪贴板中读取文本，该函数也返回一个 Promise：

```js
navigator.clipboard.readText()
  .then(text => {
    console.log('Pasted content: ', text);
  })
  .catch(err => {
    console.error('Failed to read clipboard contents: ', err);
  });
```

为了保持一致性，下面是等效的异步函数：

```js
async function getClipboardContents() {
  try {
    const text = await navigator.clipboard.readText();
    console.log('Pasted content: ', text);
  } catch (err) {
    console.error('Failed to read clipboard contents: ', err);
  }
}
```

## 处理粘贴事件

有计划推出检测剪贴板更改的新事件，但现在最好使用“粘贴”事件。它很适合用于阅读剪贴板文本的新异步方法：

```js
document.addEventListener('paste', event => {
  event.preventDefault();
  navigator.clipboard.readText().then(text => {
    console.log('Pasted text: ', text);
  });
});
```

## 安全和权限

剪贴板访问一直为浏览器带来安全问题。如果没有适当的权限，页面可能会悄悄地将所有恶意内容复制到用户的剪贴板，粘贴时会产生灾难性的结果。想象一下，一个网页，静静地复制 `rm -rf /` 或[解压缩炸弹图像](http://www.aerasec.de/security/advisories/decompression-bomb-vulnerability.html)到剪贴板。

让网页不受限制地读取剪贴板更加麻烦。用户经常将敏感信息（如密码和个人详细信息）复制到剪贴板，然后可以通过任何页面阅读，而用户根本无法察觉。

与许多新的 API 一样，[`navigator.clipboard`](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard) 仅支持通过 HTTPS 提供的页面。为了防止滥用，只有当页面处于活动选项卡时才允许剪贴板访问。活动选项卡中的页面可以在不请求权限的情况下写入剪贴板，但从剪贴板中读取始终需要权限。

为了更容易，复制和粘贴的两个新权限已添加到 [Permissions API](https://developers.google.com/web/updates/2015/04/permissions-api-for-the-web) 中。当页面处于活动选项卡时，clipboard-write 权限会自动授予页面。当您通过从剪贴板中读取数据时，则必须要求获取 clipboard-read 权限。

```js
{ name: 'clipboard-read' }
{ name: 'clipboard-write' }
```

<img width="405" alt="prompt" src="https://user-images.githubusercontent.com/359395/37577995-b7d86458-2b70-11e8-8c7e-3c47b302546b.png">

与使用权限 API 的任何其它内容一样，可以检查您的应用是否具有与剪贴板交互的权限：

```js
navigator.permissions.query({
  name: 'clipboard-read'
}).then(permissionStatus => {
  // permissionStatus.state 的值是 'granted'、'denied'、'prompt':
  console.log(permissionStatus.state);

  // 监听权限状态改变事件
  permissionStatus.onchange = () => {
    console.log(permissionStatus.state);
  };
});
```

以下是剪贴板 API 的“异步”部分真正派上用场的地方：尝试读取或写入剪贴板数据将自动提示用户获得权限（如果尚未授予）。由于 API 是基于 Promise 的，所以如果用户拒绝剪贴板权限时，Promise 将被 reject，因此页面可以适当地作出响应。

因为只有当页面是当前活动选项卡时，Chrome 才允许剪贴板访问，因此如果直接粘贴到 DevTools 中，则会发现这里的一些示例运行不正确，因为此时 DevTools 本身是活动选项卡（页面不是活动选项卡）。有一个技巧：我们需要使用 setTimeout 推迟剪贴板访问，然后在调用函数之前快速单击页面内部以使页面获取焦点：

```js
setTimeout(async () => {
  const text = await navigator.clipboard.readText();
  console.log(text);
}, 2000);
```

## 回顾

在引入异步剪贴板 API 之前，我们在 Web 浏览器中混合了不同的复制和粘贴实现。

在大多数浏览器中，可以使用 `document.execCommand('copy')` 和触发浏览器自己的复制和粘贴 `document.execCommand('paste')`。如果要复制的文本是不存在于 DOM 中的字符串，我们必须将其插入到 DOM 中并选择它：

```js
button.addEventListener('click', e => {
  const input = document.createElement('input');
  document.body.appendChild(input);
  input.value = text;
  input.focus();
  input.select();
  const result = document.execCommand('copy');
  if (result === 'unsuccessful') {
    console.error('Failed to copy text.');
  }
})
```

同样，以下是您如何在不支持新的 Async Clipboard API 的浏览器中处理粘贴的内容：

```js
document.addEventListener('paste', e => {
  const text = e.clipboardData.getData('text/plain');
  console.log('Got pasted text: ', text);
})
```

在 Internet Explorer 中，我们也可以通过 `window.clipboardData` 访问剪贴板。如果在用户手势内进行访问（例如点击事件） - [以负责任的方式请求权限](https://developers.google.com/web/fundamentals/native-hardware/user-location/#ask_permission_responsibly)的一部分 - 则不显示权限提示。

## 检测和回退

在支持所有浏览器的同时，使用功能检测来利用异步剪贴板是个不错的主意。您可以通过检查 `navigator.clipboard` 来检测对 Async Clipboard API 的支持：

```js
document.addEventListener('paste', async e => {
  let text;
  if (navigator.clipboard) {
    text = await navigator.clipboard.readText()
  }
  else {
    text = e.clipboardData.getData('text/plain');
  }
  console.log('Got pasted text: ', text);
});
```

## 异步剪贴板 API 的下一步是什么？

正如你可能已经注意到的那样，这篇文章只涵盖了 `navigator.clipboard` 的文本部分。规范中有更多的通用 `read()` 和 `write()` 方法，但是这些会带来额外的实现复杂性和安全性问题（请记住那些图像炸弹？）。目前，Chrome 正在推出更简单的 API 文本部分。

## 更多信息

- [Chrome 平台状态](https://www.chromestatus.com/feature/5861289330999296)
- [代码示例](https://github.com/GoogleChrome/samples/tree/gh-pages/async-clipboard)
- [API](https://w3c.github.io/clipboard-apis/)
- [解释](https://github.com/w3c/clipboard-apis/blob/master/explainer.adoc)
- [实施方案](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/epeaao7l13M)
- [讨论](https://discourse.wicg.io/t/proposal-modern-asynchronous-clipboard-api/1513)

-----------

> 阅读原文：[Async Clipboard API：异步剪贴板 API](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know/blob/master/archives/023-clipboardapi.md)
>
> 讨论地址：[#23](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know/issues/23)
> 
> 如果你想参与讨论，请[点击这里](https://github.com/justjavac/the-front-end-knowledge-you-may-not-know)
