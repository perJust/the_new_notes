## watch选项的原理

> 总结来说：其实也就是每个watch所监听的data，添加一个watcher，这个watcher的回调函数cb是指向watch监听属性的后的handler；当有`immediate: true`时将立即以表达式的当前值触发回调；当有`deep: true`时将进行深度遍历，触发内部收集当前watch的依赖，当内部改变触发set时进行notify、update，进而以表达式的当前值触发回调；

***
1. watch选项不管设不设置`deep`，对普通数组以下两种方式是监听不到：
>> 当你利用索引直接设置一个数组项时，例如：vm.items[indexOfItem] = newValue
>
>> 当你修改数组的长度时，例如：vm.items.length = newLength
2. 设置`deep: true`对于数组内部包含object的，如果内部object某个属性发生改变，可以监听的到；如: `p: [1, {a:1, n: [{b:1}]}]`，进行`this.p[1].a = 2`或者`this.p[1].n[0].b = 2`，都可以监听的到；
>> vue源码在Observer类可知，data属性是个数组也会遍历进行监听(仅限内部object项)
```javascript
// Observer类
constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
        if (hasProto) {
        protoAugment(value, arrayMethods)
        } else {
        copyAugment(value, arrayMethods, arrayKeys)
        }
        this.observeArray(value)    // 如果是数组  还要进行判断  不管多深  如果内部有对象形式，都会进行Object.defineProperty监听
    } else {
        this.walk(value)
    }
}

/**
 * Observe a list of Array items.
 */
observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
        observe(items[i])
    }
}

/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */
export function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {     // 非对象的  直接返回  不进行监听
    return
  }
  // ...
}
```



##### watch的核心源码分析

```javascript
// 源码截取 并做分析

// watch类
/**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    pushTarget(this)    // Dep.target指向当前
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)     // 主要是这一步  traverse(遍历)  深度遍历读取value  触发内部defineProperty的get  使它们都订阅当前
      }
      popTarget()   // Dep.target移除当前指向
      this.cleanupDeps()
    }
    return value
  }

// traverse.js
/**
 * Recursively traverse an object to evoke all converted
 * getters, so that every nested property inside the object
 * is collected as a "deep" dependency.
 */
export function traverse (val: any) {
  _traverse(val, seenObjects)
  seenObjects.clear()
}

function _traverse (val: any, seen: SimpleSet) {
  let i, keys
  const isA = Array.isArray(val)
  if ((!isA && !isObject(val)) || Object.isFrozen(val) || val instanceof VNode) {
    return
  }
  if (val.__ob__) {
    const depId = val.__ob__.dep.id
    if (seen.has(depId)) {
      return
    }
    seen.add(depId)
  }
  if (isA) {
    i = val.length
    while (i--) _traverse(val[i], seen)     // 对于数组  则继续深入递归
  } else {
    keys = Object.keys(val)
    i = keys.length
    while (i--) _traverse(val[keys[i]], seen)   // 对于对象  也进行深入递归  但是过程中发生了val[keys[i]]  相当于触发get  进而收集当前这个依赖
  }
}
```