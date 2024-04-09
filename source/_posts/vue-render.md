---
title: vue-render如何渲染多个同名指令
date: 2020-04-17 14:52:38
tags: [vue]
---
本文介绍如何使用render渲染出多个同名指令(使用clipbard举例，其它同理)
[本文测试代码](/demos/vue/vue-render.html)

```html
<button v-clipbard:copy="doCopy" v-clipbard:success="onSuccess">复制</button>
```

通常在vue中使用clipbard时会这么写绑定事件，这样多个同名的指令不同参数是可以正常使用的。
但在有时候会使用render来渲染这个dom，那怎么来实现呢，通过[vue文档](https://cn.vuejs.org/v2/guide/render-function.html)可以看出render的directives的属性是个数组，说明可以渲染多个指令。

```js
h(
  'button', {
    directives: [
      {
        name: 'clipbard',
        arg: 'copy',
        value: () => {
          // doCopy
        }
      },
      {
        name: 'clipbard',
        arg: 'success',
        value: () => {
          // onSuccess
        }
      }
    ]
  }, '复制'
)
```

看上去好像没什么问题，但实际并没有生效。
通过阅读render的源码我们可以发现，render内部是通过每个对象中的name来区分不同指令，相同的name会被覆盖。那不同的arg就不能渲染了吗，并不是。[在此处](https://github.com/vuejs/vue/blob/v2.6.11/src/core/vdom/modules/directives.js#L108)可以看出内部还读取了rawName。

给之前的指令添加一个唯一的rawName(name:arg)

```js
h(
  'button', {
    directives: [
      {
        name: 'clipbard',
        rawName: "clipbard:copy", // new
        arg: 'copy',
        value: () => {
          // doCopy
        }
      },
      {
        name: 'clipbard',
        rawName: "clipbard:success",  // new
        arg: 'success',
        value: () => {
          // onSuccess
        }
      }
    ]
  }, '复制'
)
```

测试一下两个方法都绑定成功！由此得出rawName才是自定义指令中的唯一key。

### 额外

为什么是name:arg呢，其实通过我们自己实现自定义指令，可以在binding对象中发现template中的写法对象是这样的

```js
{
  name: 'clipbard',
  rawName: 'v-clipbard:copy',
  arg: "copy",
  ...
}
```

而上述render的写法对象是这样的

```js
{
  name: "clipbard"
  arg: "success",
  ...
}
```

template会自动生成rawName以v-name:arg格式，跟template中绑定的指令的写法相同，由此证实rawName确实是区分多指令的唯一key

为了与内部生成的rawName规范统一，我们也可以把rawName写成name:arg或v-name:arg的格式。

[本文测试代码](/demos/vue-render.html)
