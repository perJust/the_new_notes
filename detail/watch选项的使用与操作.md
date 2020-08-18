## watch的使用

> 概括：通过[官网解释](https://cn.vuejs.org/v2/api/#watch)可知，`watch`这个选项，内部键对应的值可以为：字符串、函数、对象、数组。

```javascript
// 先上官网示例
watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // 方法名
    b: 'someMethod',
    // 该回调会在任何被侦听的对象的 property 改变时被调用，不论其被嵌套多深
    c: {
      handler: function (val, oldVal) { /* ... */ },
      deep: true
    },
    // 该回调将会在侦听开始之后被立即调用
    d: {
      handler: 'someMethod',
      immediate: true
    },
    // 你可以传入回调数组，它们会被逐一调用
    e: [
      'handle1',
      function handle2 (val, oldVal) { /* ... */ },
      {
        handler: function handle3 (val, oldVal) { /* ... */ },
        /* 
        	immediate: true,
        	deep: true
        */
      }
    ],
    // watch 某个data下具体key
    'e.f': function (val, oldVal) { /* ... */ }
  },
 methods: {
     /* someMethod */
     /* handle1 */
 }
```

1. 函数形式

> 针对基本类型的属性，可利用函数形式获取新旧值的改变

2. 字符串形式

> 字符串表示的是方法名，这个方法名会在methods获取，效果跟函数形式一样；这种形式一般可以做一个函数处理对应多个watch观察

3. 对象形式

> 一般对于对象进行观察时的写法，有`immediate`与`deep`进行配置；

> 对象形式中的`handler`也可用方法名的形式

4. 数组形式

> 可将上面三种形式都放在回调数组里，逐一执行

5. 注意点

> `watch`监听的键，对于值是对象的key，`watch`监听的key可以是对象下的某个key，如`'e.prop : function(newval, oldval){}'`

> `watch`监听的是对象时
>
> > 直接进行修改而引起的`watch`监听（如：`this.a.name = '123'`这种形式），则`handler(val, oldVal){}`中`val === oldVal`，因为只是某个对象改变了内部的东西而已，所以`val`与`oldVal`都是传入的相同指向；
> >
> > 直接覆盖而引起的`watch`监听（如：`this.a = { name: '123' }`这种形式），则`handler(val, oldVal){}`中`val`不等于` oldVal`；