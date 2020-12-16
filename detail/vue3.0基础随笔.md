## vue3.0基础随笔.md

> 概括：基础api

1. 响应式方法
```javascript
import { reactive, ref } from 'vue'

export default {
    setup() {
        let user = reactive({
            name: '1',
            age: 18,
            height: 180
        });
        let userRef = ref({
            name: '2',
            age: 18,
            height: 180
        });
        function change() {
            user.name = '11';
            userRef.value.name = '22';
        }
        return { user, userRef, change };
    }
}
```
```html
<div>{{ user.name }}</div>
<div>{{ userRef.name }}</div>
<button @click="change">change</button>

<!-- 
区别与相同：
    1、ref与reactive都是绑定响应式数据；
    2、ref需要采用的ref.value的形式修改数据，reactive正常修改；
    3、ref可以绑定基本类型,如let num = ref(1);而reactive需绑定对象;
    4、ref如果绑定的是基本类型，则直接可以使用，如上一条中num，模板中可以直接使用num;
    5、ref相当于基于reactive实现的，相当于 ref = reactive({ value: refVal }), 只不过模板里使用时没写userRef.value.name的形式(是因为模板编译时判断了当前数据类型是否为ref，为ref的话则省去.value);
    6、都是递归响应式数据;
 -->
```

2. 相关api
```javascript
import { isRef, isReactive } from 'vue';
// 判断类型为ref、reactive

import { shadowRef, shadowReactive } from 'vue';
export default {
    setup() {
        let user = shadowReactive({
            name: '1',
            age: 18,
            height: {
                x: 1
            }
        });
        let userRef = shadowRef({
            name: '2',
            age: 18,
            height: : {
                x: 1
            }
        });
        return { user, userRef };
    }
}
/*
    1、shadowRef与shadowReactive是创建的非递归数据，其余用法都同上一点；
    2、user.height.x的改变，不是响应式的，但第一层数据是响应式的,如：user.name = '11'是会引起相关改变的；userRef.name的改变不是响应式的，要想引起改变，需重新负值,如：userRef = { name: '22' }；(由此可以间接理解ref与reactive区别的第5点，相当于shadowRef基于shadowReactvie也是监听第一层，只不过是.value)
*/

import { triggerRef } from 'vue';
// 由上可知，要想在shadowRef内部改变后立即触发相关改变，需要调用triggerRef进行触发，triggerRef(userRef)；
```