---
title: 即时生成Vue组件并挂载（改造已有组件）
date: 2017-08-23 21:57:48
tags: [javascript,Vue]
---


## 问题

开发主要使用的前端框架为Element UI，现在有一个新的需求，就是希望能够点击表格中的一个分组项，该行出现一个下拉表格列表，以更加详细的显示分组内的更详细信息。如果只是一个普通的下拉其实也很好实现，只要使用jQuery在对应的点击事件中加一个DOM操作就可以了。但是更让人烦躁的是，所要添加的内容又需要用到Element UI的组件。显然直接写到`$("tr").after()`中是无效的，因为Element UI的组件需要经过babel转码、编译等一系列操作之后才被渲染在页面上。那么怎么办呢？

如果可以先经过Vue渲染，再加到DOM里面就好了！

## Let's try try try...

首先，在填表格的时候搞一点小小的事情：在`<el-column>`中需要下拉的行的某一列下渲染一个`display: none`的表格，然后在点击的时候利用`jQuery.clone(true, true)`方法将元素和事件克隆一份并追加到相应的`<tr>`下。这样的话下拉的内容也是由Vue渲染好的，组件神马的就都可以用啦~

然而...

当点击按钮时，的确获得了经Vue渲染的表格，但是里面的内容却是...暂无数据...

会不会是移动的时候数据没有加载上？尝试了$nextTick方法以及定时以后没有效果。再审查一下元素发现，原始的表格是有数据的，但是clone()获得的却没有。

## Vue.extend大法

#### Vue.extend(options)

- **Arguments:**

  - `{Object} options`

- **Usage:**

  Create a “subclass” of the base Vue constructor. The argument should be an object containing component options.

  The special case to note here is the `data` option - it must be a function when used with `Vue.extend()`.

  ```
  <div id="mount-point"></div>
  ```

  ```
  // create constructor
  var Profile = Vue.extend({
    template: '<p>{{firstName}} {{lastName}} aka {{alias}}</p>',
    data: function () {
      return {
        firstName: 'Walter',
        lastName: 'White',
        alias: 'Heisenberg'
      }
    }
  })
  // create an instance of Profile and mount it on an element
  new Profile().$mount('#mount-point')
  ```

  Will result in:

  `<p>Walter White aka Heisenberg</p>`

显然，这就像开了挂！...0.0（啥开了挂，不就是又套了个现定义的component嘛...完全可以把参数和必要的函数都传进去构造一个component）但是呢要记得不要把 this 搞乱了…建议在extend前将父组件的 this 使用`let _this = this;`备份，在extend的组件里就可以使用 _this 访问父 this 啦~~

因此我们可以先定位到需要添加的`<tr>`上，在其后用jQuery生成一个新的`<tr>`用于挂载临时组件，然后把这个鬼东西使用`.$mount()`方法挂载上去。

这种方式的话需要注意的是在页面切换走的时候要记得把jQuery创建的对象在beforeDestroy()钩子以及其他必要的函数处即时清除，因为由jQuery创建的对象不会被Vue接管管理，需要自己手动管理。
