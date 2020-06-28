# vue里为什么不直接对数组进行监听？

>  官网解释为：由于 JavaScript 的限制，Vue 不能检测以下数组的变动：1、当你利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`;2、当你修改数组的长度时，例如：`vm.items.length = newLength`；


### 解读
> 其实`Object.defineProperty`可以检测到数组索引对应值改变，但是vue却没有这么做。
>>  这个问题有个大佬已经在`github`提过[issue](https://github.com/vuejs/vue/issues/8562)。
>
>>  尤大神给出的答案是==因为性能原因==，这直接将锅指向`Object.defineProperty`。
