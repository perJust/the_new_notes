## vue里的computed原理分析

> 思考：computed的原理也在于利用watcher，将内部执行一遍，利用内部相关联的`data绑定的值`的get，将当前computed环境添加至相关data的dep中，待到更新时，通知watcher更新，进而使得computed重新获取新值。