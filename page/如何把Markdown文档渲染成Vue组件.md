# 如何把Markdown文档渲染成Vue组件

## 前言

前两天发了篇文章 [基于Vuex实现小米商城购物车](https://juejin.im/post/5e660ef9518825490276748a)，里面放了在线Demo的链接，我在后台看到，有很多人访问。相信看过的人，都会发现，**关于我们**那个页面是没有内容的。我昨天看了一下，觉得太空了，得写点东西。然后就想到README.md，没错就是每个开源项目都会有的介绍文档。

我就想能不能像Github一样，把README.md渲染成一个组件，然后在**关于我们**页面直接使用。

大家都知道，其实不止Github，很多社区都是采取这样的方式，把用Markdown写的文章渲染成页面的。但怎样才能在自己的项目中使用呢?

## 效果

话不多说，先看看我实现的效果

![](https://user-gold-cdn.xitu.io/2020/3/13/170cfd9d996f4601?w=1902&h=754&f=png&s=98278)

是不是有模有样的  ^_^ 

## 实现步骤

### 1.新建一个组件

这个简单就不详细说，只要定义好一个div，class命名为markdown-body就好，至于为什么是这个，后面再说。

```vue
<template>
  <div class="markdown-body">
    
  </div>
</template>
<script>
export default {
  name: "MyMarkdown",
  data() {
    return {
      md: ""
    };
  }
</script>
<style>
.markdown-body {
  box-sizing: border-box;
  margin: 0 auto;
  padding: 0 40px;
}
</style>
```

### 2. 获取Markdown文件

首先需要考虑一个问题，既然想把Markdown文档渲染成组件，那Markdown文件从哪里来？

方法很多，我就说两种：

**静态资源导入**

像外部css文件一样导入，需要使用loader，可以看一下 [markdown-loader](https://www.npmjs.com/package/markdown-loader )，我没有使用这个方法，就不细说了，感兴趣的同学，自己了解一下。

**远程获取**

比如从静态资源服务器、后端服务器等远程服务器获取。

我为什么要选择这种方法呢？

我主要考虑到，README.md不是一成不变的，我可能随时需要修改。如果使用第一种方法，每次修改都需要对项目重新build，然后重新部署，很麻烦。但使用第二种方法，文件修改了，只需要把文件重新上传到服务器就可以了，服务器都不需要重启。

我的后端服务器在之前就已经实现静态资源请求的处理，你在在线Demo中看到的图片，绝大部分都是从后端服务器获取来的。所以我现在只需要在后端服务器的public文件夹下新建一个docs文件夹，然后把README.md放进去。前端在组件创建完成后，使用axios向后端发起请求，就能获取到README.md。

代码：

```javascript
created() {
  // 从后端请求README.md
  this.$axios
    .get("/api/public/docs/README.md", {})
    .then(res => {
      this.md = res.data;
    })
    .catch(err => {
      return Promise.reject(err);
    });
}
```

先在控制台输出看一下获取回来的数据是怎样的

![](https://user-gold-cdn.xitu.io/2020/3/13/170cfda377d543c7?w=1920&h=627&f=png&s=68178)

这个是不是很熟悉，就是README.md源文件，那如何渲染到组件呢？

### 3. 把Markdown文档渲染成html

Markdown文档拿到了，那怎样把它渲染到页面呢，v-html吗？显然不行，我找到了 [vue-markdown ](https://www.npmjs.com/package/vue-markdown)，一个 Markdown解析器，可以把Markdown文档解析成html。

**安装**

```
npm install --save vue-markdown
```

**导入**

```javascript
import VueMarkdown from "vue-markdown";
```

**局部注册组件**

```vue
components: {
  VueMarkdown
}
```

**使用**

```html
<div id="my-markdown" class="markdown-body">
  <vue-markdown :source="md"></vue-markdown>
</div>
```

**效果**

先看一下效果

![](https://user-gold-cdn.xitu.io/2020/3/13/170d23ba057cfb3f?w=1902&h=677&f=png&s=131558)

是不是觉得有点丑，不急，没有css样式，当然是有点丑。

### 4. CSS样式

css样式从哪里？自己写吗，当然不啦，作为一名优秀的**，有现成的，绝不自己写，于是我找到了 [github-markdown-css]( https://www.npmjs.com/package/github-markdown-css) ，为了使用方便，我直接下载下来，放在“../assets/css/”目录下，然后导入。

```css
@import "../assets/css/github-markdown.css";
```

然而，事情并没有像想象的那么美好，页面什么都没有改变。

我看了一下github-markdown-css的文档，才知道要在父div添加class：**markdown-body**，至于为什么，大家应该都懂，就不说了。

看一下效果吧

![](https://user-gold-cdn.xitu.io/2020/3/13/170cfd9d996f4601?w=1902&h=754&f=png&s=98278)

### 总结

到这里，把Markdown文档渲染成Vue组件的需求已经实现，需求很简单，实现也很简单，没有什么技术含量，就当学习的过程分享一下吧。

效果预览：[ http://106.15.179.105/#/about ](http://106.15.179.105/#/about)，感兴趣的同学可以看一下（没有兼容移动端，请使用PC访问）！

完整项目源代码仓库：[https://github.com/hai-27/vue-store](https://github.com/hai-27/vue-store)

感谢你的阅读！



作者：[hai-27](https://github.com/hai-27)
2020年3月13日