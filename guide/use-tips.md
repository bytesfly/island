
## 页面跳转

- 当前页面锚点跳转

```md
[跳转到 -> 插入数学公式](#mathjax)
```

效果： [跳转到 -> 插入数学公式](#mathjax)

注意： 我这里设置了二级标题的`id`属性，如果没有设置`id`的话直接用二级标题就行，即`[跳转到 -> 插入数学公式](#插入数学公式)`

可参考： [https://docsify.js.org/#/zh-cn/helpers?id=设置标题的-id-属性](https://docsify.js.org/#/zh-cn/helpers?id=设置标题的-id-属性)

- 跳转到当前站点其他页面

比如： 

```md
[跳转到 -> 快速部署-Docker部署](/guide/quick-deploy#补充Docker部署)
```

效果如下： [跳转到 -> 快速部署-Docker部署](/guide/quick-deploy#补充Docker部署)


## 下载本站文件

参考： [https://docsify.js.org/#/zh-cn/helpers?id=忽略编译链接](https://docsify.js.org/#/zh-cn/helpers?id=忽略编译链接)

为了方便其他人下载文档项目中的某些文件(比如`pdf`、`zip`、`jar`等)，这时也可以用超链接的方式，但需用`ignore`显示告诉`docsify`不要编译这个链接。 如下：

```md
[link](/path/file ':ignore title')
```

比如这里提供一个当前站点的文件的下载链接：

[use-tips.md](/guide/use-tips.md ':ignore 点击下载文件')

注意： `GitHub Pages`部署的方式不能用这种方式下载，我用`nginx`部署是可以的，详情见： [快速部署-本地部署](/guide/quick-deploy#本地部署)

## 插入数学公式 :id=mathjax

参考： [https://github.com/docsifyjs/docsify/issues/96](https://github.com/docsifyjs/docsify/issues/96)

使用`MathJax`渲染数学公式，在`index.html`中添加如下：

```html

<script>
    window.$docsify = {
        plugins: [
            function (hook) {
                hook.doneEach(function () {
                    if (typeof MathJax !== 'undefined') {
                        MathJax.Hub.Queue(["Typeset", MathJax.Hub])
                    }
                })
            }
        ]
    }
</script>

<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
    tex2jax: { inlineMath: [['$','$'], ['\\(','\\)']], processClass: 'math', processEscapes: true },
    TeX: {
    equationNumbers: { autoNumber: ['AMS'], useLabelIds: true },
    extensions: ['extpfeil.js', 'mediawiki-texvc.js'],
    Macros: {bm: "\\boldsymbol"}
    },
    'HTML-CSS': { linebreaks: { automatic: true } },
    SVG: { linebreaks: { automatic: true } }
    });
</script>
<script src="https://mathjax.cnblogs.com/2_7_5/MathJax.js?config=TeX-AMS-MML_HTMLorMML&amp;v=20200504"></script>
```

- 行内公式

语法如下：

```md
行内公式举例： $\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt$
```

行内公式举例： $\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt$


- 单独的公式块

语法如下：

```md
$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt
$$
```


显示效果：

$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt
$$

关于在Markdown中编辑数学符号与公式，可参考我之前整理过的博客： [https://www.cnblogs.com/bytesfly/p/markdown-formula.html](https://www.cnblogs.com/bytesfly/p/markdown-formula.html)

## 持续更新

使用 [docsify](https://docsify.js.org/#/zh-cn/) 作为一个开发人员的文档站点还是挺实用的。以需求为驱动，这里会持续更新使用技巧。
