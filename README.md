# 你可能不知道的前端知识点

发掘被我们忽略的前端知识点。所有的讨论以 [issues](https://github.com/justjavac/the-front-end-knowledge-you-may-dont-know/issues) 的形式进行，不定期在 [小密圈](http://t.xiaomiquan.com/EAQvV7Y) 发起话题，任何人都可以在 issues 区围观讨论。

- 须知 [#1](https://github.com/justjavac/the-front-end-knowledge-you-may-dont-know/issues/1)
- 索引 [#2](https://github.com/justjavac/the-front-end-knowledge-you-may-dont-know/issues/2)

## 缘起

前一阵有人在微信群里面遇到了一个问题：

> 当输入框在最底部的时候，弹起的虚拟键盘会把输入框挡住。

于是我发给他一个 API：`Element.scrollIntoViewIfNeeded(opt_center)`，故名思意，就是在需要的时候将元素滚动到可视区域。

对于前端 API 来说，我们最关心的是它的浏览器兼容性：

![scrollIntoViewIfNeeded](./element-scrollIntoViewIfNeeded-can-i-use.png)

随后他又问我：

> 怎么样才能学到这些新的前端技术和API？

首先要知道，这并不是一个新的 API，我们看看它的支持情况:

- 2010-12-06 发布的 Android 2.3（6年前）
- 2011-06-20 发布的 Safari 5.1
- 2011-09-16 发布的 Chrome 15
- 2012-03-07 发布的 iOS 5.1
- ...

这已经是一个有着 6 年历史的 API 了。如果在 GitHub 搜索一下，可以搜索到 38,305 个搜索结果。

![scrollIntoViewIfNeeded search on github](./scrollIntoViewIfNeeded-search-on-github.png)

这已经是一个被广泛使用的 API 了。

所以我创建了这个 repo，整理一些比较实用的但是却不经常见的前端技术。

## 建议

关于碎片化阅读其实我是持反对意见的，碎片化阅读只能作为自己知识的补充，但是真正想学好前端，还是应该多看书，从头构建自己的**完整知识体系**，然后把碎片化阅读作为自己知识体系中知识点的补充。

## License

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/cn/"><img alt="知识共享许可协议" style="border-width:0" src="http://i.creativecommons.org/l/by-nc-sa/3.0/cn/88x31.png" /></a><br />本<span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" rel="dct:type">作品</span>由<a xmlns:cc="http://creativecommons.org/ns#" href="http://justjavac.com" property="cc:attributionName" rel="cc:attributionURL">justjavac</a>创作，采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/3.0/cn/">知识共享署名-非商业性使用-相同方式共享 3.0 中国大陆许可协议</a>进行许可。凡是转载的文章，翻译的文章，或者由其他作者投稿的文章，版权归原作者所有。
