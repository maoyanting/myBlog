---
title: hexo-NexT的优化
date: 2018-02-07 10:59:24
tags:categories: 
- hexo
tags:
- hexo
---

一些基于NexT的优化

<!--more-->

## 写在前面

基本的就不讲了，可参考：[Hexo+Next个人博客主题优化](https://www.jianshu.com/p/efbeddc5eb19)

只记录一些自己的修改

个人觉得blog还是清爽一点的好，东西弄多了过于喧宾夺主；

特别是背景，之前看一个blog背景是个二次元萌妹子，太花了，看背景我就看不下去文章了。

## 字数统计

Next的_config.yml文件里面推荐使用`hexo-symbols-count-time`

```yml
# Post wordcount display settings
# Dependencies: https://github.com/theme-next/hexo-symbols-count-time
```

具体使用可见[GitHub](https://github.com/theme-next/hexo-symbols-count-time)

### 阅读时间

因为每个人的基础不同速度不同，个人不建议放上去。

注意：这里显示的时间是以min为单位，依据个人习惯，可以修改`./node_modules/_hexo-symbols-count-time@0.2.0@hexo-symbols-count-time/lib/helper.js`里面`getFormatTime`的return来改变显示方式。

### 字数统计

个人觉得是有点争议的，关于这一点Issues里面有人和开发者争论过，详见[No use by changing the parameter `awl` and `wpm`. [some html tags can add excess additions in symbols counting] #3](https://github.com/theme-next/hexo-symbols-count-time/issues/3)

***

## 看板娘

如果你去过b站的直播，大概就对那个bilibili娘有点印象

这东西叫live2D，当然你也可以放入自己的blog

GitHub地址：[EYHN/hexo-helper-live2d](https://link.zhihu.com/?target=https%3A//github.com/EYHN/hexo-helper-live2d)

把它添加进你的hexo中，就能在页面上看见一个萌妹子的身体随着你的鼠标晃啊晃了～

也可以自己制作模型，但是制作这个好像还是蛮费神的，需要不停地调。

***

## 背景

修改背景很简单，这里只说一下，我改好背景以后，本地浏览是没问题的，但是上传到git上去就是显示不出来。后来无意中clean了一下就好了Ծ ̮ Ծ。

背景是我自己做的，白色为主，只有右侧有图案，看文章的时候侧边栏会完全挡住背景。

```css
body {
  background:url(https://github.com/maoyanting/readme_pic/blob/master/blog/background.jpeg?raw=true);
  background-attachment: fixed;
  background-size:cover
  +mobile() {
    background: white;
  }
}
```

这里有个问题，开始手机端的浏览体验不是很好╮(￣▽￣")╭，手机端背景显示有问题

索性把手机端的背景图片去掉，加上这句就行了，`+mobile() {background: white; }`，完美！

***

## 首页隐藏指定文章

参考：[首页隐藏部分文章#474](https://github.com/iissnan/hexo-theme-next/issues/474)

next/layout/index.swig文件中， post_template.render(post, true) 这一部分修改为

```
{% if post.visible !== 'hide' %}
{{ post_template.render(post, true) }}
{% endif %}
```

在文章里面加上：visible: hide 在首页隐藏指定文章



## 问题

### 一、Host key verification failed. fatal: Could not read from remote repository.错误

原因：莫名其妙的，有一段时间没去碰blog就这样了

解决方案：查看你的ssh有没有问题，没有的话运行一下

```
ssh git@github.com
```

然后就好了