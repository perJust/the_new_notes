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
> setup函数相当于在：beforeCreate之前执行

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

import { toRow, markRow} from 'vue'

export default {
    setup() {
        let obj = {name: '111'}
        let state = reactive(obj);
        let row = toRow(state);
        // obj === row
        // toRow表示：获取原始的传入对象

        let d = markRow(obj);
        // markRow表示：设定obj为不可响应式的数据
        // 再进行 reactive(obj)是没用的
    }
}

import { toRef, toRefs } from 'vue'

export default {
    setup() {
        let obj = {name: '111', type: 1}
        let state = toRef(obj, 'name')
        state.value = '1'
        // 设定obj里的name为响应式数据，state的value改变也会使obj改变

        let state1 = toRefs(obj)
        state.name.value = '1'
        state.type.value = 2
        // 设定obj都为响应式数据，但set时需对应refs里每个属性，一般理解为：先在toRefs里拿到每个toRef数据，然后对每个ref数据进行.value的赋值。类似 state.name.value
    }
}

<template>
    <div ref="isRef"></div>
</template>
import { ref, onMounted } from 'vue'

export default {
    setup() {
        let isRef = ref(null)
    
        onMounted(() => {
            // isRef.value --> 元素<div ref="isRef"></div>
        })

        // isRef的使用有别于vue2.0，这里是创建一个ref(null)，在template里就直接用ref属性使用了
        return { isRef }
    }
}

import { customRef } from 'vue'

function myRef(value) {
    return customRef((track, trigger) => {
        return {
            get() {
                track();    // 通知vue这个数据需要追踪
                return value;
            },
            set(newVal) {
                value = newVal;
                trigger();  // 触发界面相关更新
            }
        }
    })
}
function getDataRef(url) {
    return customRef((track, trigger) => {
        let value = null;
        fetch(url).then(res=>res.json()).then(res => {
            value = res.data;
            trigger();
        });
        return {
            get() {
                track(); 
                return value;
            },
            set(newVal) {
                value = newVal;
                trigger();
            }
        }
    })
}
export default {
    setup() {
        let state = myRef(123)；
        let data1 = getDataRef('./static/1.json');
        return { state }
    }
}
/*
customRef：自定义ref
    1、自定义控制ref过程的绑定；
    2、可以内部写异步的方法，进行优雅的获取请求数据并相应视图；
*/

import { readonly, isReadonly, shallowReadonly } from 'vue'
export default {
    setup() {
        let readonlyData = readonly({name: '1', msg: { type: 1}})
        readonlyData.name = '11';
        readonlyData.msg.type = 11;
        // 都是不能改变readonlyData

        let shallowReadonlyData = shallowReadonly({name: '2', msg: {type: 2}});
        shallowReadonlyData.name = '22';
        // 不能改变
        shallowReadonlyData.msg.type = 22;
        // 可以改变

        isReadonly(readonlyData); // true
        isReadonly(shallowReadonlyData); // true

        return { readonlyData, shallowReadonlyData };
    }
}
/*
    readonly: 递归进行只读操作；
    shallowReadonly: 只对第一层进行只读操作；

    const声明与readonly区别：
        const: 是赋值保护，不能重新赋值
        readonly: 是属性保护，属性不能重新赋值

        如： const a = {name: '1'};
                a = {}; // error
                a.name = '11'; // ok
            let state = readonly({name: '22'});
                state.name = '11'; // error
                // 不会在想state可不可以赋值吧？
                // state是let声明的，关readonly什么事.
*/
```