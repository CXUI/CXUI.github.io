---
layout: post
title:  "Turbolinks 5"
date:   2016-08-15 10:30:00
categories: rails
tags: rails5
author: "Victor"
---

## 简介

Turbolinks 让 Web 应用页面之间的导航变得更快。你不需要引入一个客户端的 JavaScript 框架就能得到单页应用的性能。你只需要和过去一样在服务器端渲染 HTML，然后在上面加几个超链接。当用户点击该链接的时候，Turbolinks 自动获取页面，替换掉 `<body>`，合并 `<head>`, 减少页面载入的时间。

### 功能

* 自动提高页面导航的性能。不需要指定哪个链接或哪部分页面需要被替换。
* 不需要服务端提供额外的支持。服务端开发仍然渲染整个 HTML 页面，而不是局部模板或 JSON。
* 尊重 Web 的体验。后退和刷新按钮仍然会如你期望的一样工作，Turbolinks 对搜索引擎友好。
* 支持移动端。借助 [iOS](https://github.com/turbolinks/turbolinks-ios) 和 [Android](https://github.com/turbolinks/turbolinks-android) 可以让你的应用获得近似于原生应用的体验。

### 浏览器支持

Turbolinks 的功能基于 [HTML5 History API](http://caniuse.com/#search=pushState) 和 [requestAnimationFrame](http://caniuse.com/#search=requestAnimationFrame)，所以它可以在几乎全部现代浏览器下正常工作，对于老旧的浏览器它会自动退回到标准的浏览器导航模式。

### 安装

1. 在 Gemfile 里面添加 `gem 'turbolinks', '~> 5.0.0'` 并执行 `bundle install`
2. 在 JavaScript 的 manifest 文件里面添加 `//= require turbolinks`

## 使用 Turbolinks 进行页面导航

Turbolinks 会接管同一域名下的所有 `<a href>` 的点击事件。当你点击一个符合该条件的链接，Turbolinks 会阻止浏览器默认的跳转行为。而使用 [History API](https://developer.mozilla.org/en-US/docs/Web/API/History) 修改浏览器的 URL，使用 [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 获取新页面内容，然后渲染 HTML 响应。

在渲染的过程中，Turbolinks 完全替换掉 `<body>` 内容，合并 `<head>` 中的内容。JavaScript 的 `window` 和 `document` 对象，以及 HTML 的 `<html>` 元素，会从一个页面携带到下一个页面。

### 所有的跳转都是 Visit

Turbolinks 模式下的页面跳转就像通过一个 action 来访问（visit）一个 URL。

访问（visit） 表示一个页面跳转从点击到渲染完毕的生命周期。包含修改浏览器历史，发起网络请求，从缓存中获取页面数据，渲染最后的响应，更新滚动条。

有两个类型的visit:

* Application Visits: `advance` 和 `replace`
* Restoration Visits: `restore`

### Application Visits

当点击链接或者调用 `Turbolinks.visit(location)` 方法的时候，就会触发 Application Visits 模式。该模式总会发起网络请求，当取回数据的时候，Turbolinks 渲染  HTML 并完成整个 visit。

如果可能，当 visit 开始的时候 Turbolinks 会立即从缓存中取出内容并渲染一个预览页面。这改善了你访问同一个页面的速度。

如果请求的 URL 包含一个锚链接，Turbolinks 会自动滚动到该锚节点。

visit 的动作决定了 Application visits 如何修改浏览器的访问历史记录。

默认的 visit 动作是 `advance`，这种情况下 Turbolinks 使用 [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History_API#The_pushState()_method) 推送一条新记录到浏览器的访问历史中。

iOS 应用会推送一个新的 view controller 到导航栈中，Android 会推送一个新的 activity 到回退栈。

如果不想在栈中添加一个新的历史记录，`replace` 动作使用 [history.replaceState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) 干掉最后的一条访问记录，并用新记录来替换它。

你可以使用下面的方法来触发 `replace` 动作。

```html
<a href="/edit" data-turbolinks-action="replace">Edit</a>
Turbolinks.visit("/edit", { action: "replace" })
```

这种情况下，iOS 会干掉最顶层的 view controller 再推送一个新的 view controller 进入导航栈，**并且没有动画效果**。

### Restoration Visits

当使用浏览器的后退和前进按钮时，Turbolinks 会启用 restoration visit 模式。iOS 或 Android 应用的后退动作也会触发这个模式。

如果可能，Turbolinks 会从缓存中取出页面且 **无需** 发起网络请求。否则，它会从网上获取最新的数据。下面会详细介绍缓存部分。

Turbolinks 会储存离开页面时的滚动条位置，当通过 restoration visits 模式返回这个页面的时候会自动定位到该位置。

Restoration visits 拥有 restore 动作，你无需关心它，Turbolinks 会在内部使用该动作。你无法通过外部链接或 `Turbolinks.visit` 调用 restore 动作。

### Canceling Visits Before They Start

不论是点击一个连接还是通过 `Turbolinks.visit` 发起的请求，都可以被取消掉。

当 visit 开始发起的时候，使用 `event.data.url` 或 jQuery 的 `$event.originalEvent.data.url` 访问 URL，`turbolinks:before-visit` 事件会得到通知。所以你只需要监听这个事件，并调用 `event.preventDefault()` 来取消请求。

Restoration visits 无法被取消，它也不会触发 `turbolinks:before-visit`。也就是说你用浏览器的前进和后退按钮是无法触发 `turbolinks:before-visit` 的。

### Disabling Turbolinks on Specific Links

可以通过如下方法，禁用或启用 Turbolinks

```html
<a href="/" data-turbolinks="false">Disabled</a>
<a href="/" data-turbolinks="true">Enabled</a>
```

