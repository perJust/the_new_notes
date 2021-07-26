###  虚拟Dom过程     snabbdom

```note
写在前面的话：
	整个虚拟DOM，就是比较新旧vnode，达到最小操作dom的次数来实现改变；
	
	当你的逻辑这阶段处理完毕后，会产生新的 vnode,这时候就需要diff来比对新旧结点，实现每个元素的一次性的改变，这个‘一次性’的意思是对于这个元素而言的，并不是对于整个dom操作而言的，别以为patch过后就只操作一次dom就实现了dom的更新。
		譬如：旧结点的div的内部是文本，新节点的div里是两个span，patch执行到这的时候，就会将旧结点的文本删除了，将新的 创建的两个span进行insert进去。
		
	可能会疑问：这样怎么减少dom操作的？
		首先明白虚拟DOM就是解决频繁修改dom的，当你在一个逻辑流程中，对一个dom进行了频繁的操作，譬如：修改class，修改style。举例说明：A();B();都执行的时候，在A函数内将width修改成a；等到执行B函数时根据条件判断又把width修改为b；对于width的修改，真实dom就要操作两次了，但是虚拟dom最后patch时只操作了一次；
```

patch的过程就是 oldVnode 根据 newVnode 进行改变，并不是直接将newVnode构造出真实dom。

```js
	// 开始处理虚拟DOM

	var patch = init({ // 会返回个patch函数
        classModule,	// 配置一些处理的模块
        propsModule,
        styleModule,
        eventListenersModule
    })

	var container = document.getElementById('app');

	var vnode = h('div#app', { on: {click: func}}, [h(/**/), h(/**/)]); 
	//  h函数是生成vnode的，h函数内最后会 return vnode(...)  调用vnode函数进行返回
	// 上面这步就可知 var vnode = {...children: { ...children: { ... } }} vnode结构
	// 别只看到第一个h函数的执行，这里是有多种 h的执行 构成一个整体的 vnode

	// 页面首次加载时，会执行第一次patch
	patch(container, vnode);
```

```javascript
	function h(sel, data, children) {
        // ...   经过一些处理转变成
        return vnode(sel, data, children, text, undefined);
    }
	// 下面的所有函数执行都是在 init 内
```


```javascript
function vnode(sel, data, children, text, elm) {
	const key = data === undefined? undefined : data.key; // 有data时，就从data里拿到key（也可能里面没定义key） 这就是vue里定义key的用途
	 return {sel, data, children, text, elm, key}; 
	 // 一个vnode包含：
	 // 		sel: 选择器，即标签
	 //			data: 标签上的属性与事件
	 //			children与text：只能有一个，要么是有子选择器的，要么只有纯文本；又有子元素又有text，是放在children的
	 //			elm： 真实dom
	 //			key： 唯一的索引
	}
```

```javascript
	function init(modules: Array<Partial<Module>>, domApi?: DOMAPI) {
      let i: number;
      let j: number;
      const cbs: ModuleHooks = { // 把 modules 里引入的各个模块的钩子函数 分类别放好  方便之后的统一执行
        create: [],
        update: [],
        remove: [],
        destroy: [],
        pre: [],
        post: [],
      };

  const api = domApi !== undefined ? domApi : htmlDomApi; // api是后面处理patch完成后要操作dom的方法      最后会解释domApi 与 htmlDomApi

	// ... 定义一些处理函数 如：createElm，addVnodes
	
	return function patch() {
		// ... 下面会单独解释
	}

​```javascript

```

```

​```javascript
	function patch (oldVnode, vnode) {
		//...
		if (sameVnode(oldVnode, vnode)) {
			// 相同，则进行比对更新
			patchVnode(oldVnode, vnode, insertedVnodeQueue); 
		} else {
			// 创建新的dom元素
			
			// 然后插入新的dom元素，删除旧的dom元素  相当于更新了 vnode.elm的指向
		}
	}

	sameVnode(oldVnode, vnode) 来判断新旧 vnode 是否相同，根据：标签名是否相同与key是否相同，如果都相同的话，则认为是同一个结点
	
	创建好的vnode上会有个属性是elm，这是指向真实dom的
```



```note
譬如：
	api.setTextContent(elm, ''); 将elm的文本清空
	这是立即执行dom操作的
	
	记住：!!!
		进行patch的操作，通过diff发现一处不一样，就立马将对应的元素进行dom操作；
		
		通俗来理解就是：你在patch之前，操作虚拟dom几百次，最后虚拟dom也只是产生一个新的vnode，并不会将真实dom操作几百次
			等到开始patch时，就代表虚拟dom的改变已经结束了，这时候就要将虚拟DOM的改变与旧的进行对比，发现哪里改变了就立马改变哪里。api的操作就是这样的，哪里改变了，就立马改变对应的真实dom。
			额外说一说：js是单线程的，肯定diff发现一处虚拟dom新旧不同，就立马操作dom进行改变了。难不成你还想等到同一时刻，将多处需要改变的dom，统一一起操作了？？js这个单线程会告诉你不行的。
			这时候你会想，我将需要改变的放到 document中的fragment里进行操作，最后一次append到#app上不就实现一次操作了吗？想一想，原本document的元素，你转移到fragment里操作，是不是页面就没元素了，空白；
				假如你是将每处不同的地方记录下位置，再放进fragment里操作，这是不是更多了移出、移入的dom操作；
				假如你是想先用fragment装下跟真实dom一样的元素，这样就可以只操作fragment里的最后append进去就可以了，真实dom也存在也不会移出空白了，这就可以了吧？错，这不仅document要重新分配更多的空间，这是不是达不到复用真实dom，而频繁的创建dom，更不要说了：真实dom的styleSheet，事件，每个dom之间的关联，你是需要多大的时间复杂度与空间复杂度才能拷贝一个一摸一样的真实dom结构。
```



```javascript
	function patchVnode(oldVnode, vnode, insertedVnodeQueue) {
		const elm = vnode.elm = oldVnode.elm; // 将旧的dom元素设置到新节点的elm上，因为新的本来就没有真实dom指向，没elm
		const oldCh = oldVnode.children; // 获取旧的children
		const ch = vnode.children; // 获取新的children
		if(OldCh === ch) return ; // 相同则表明不需要用patch更新
		
		// 这里根据个概念：有children时，就没有text；有text时，就没有children；
		if(isUndef(vnode.text)) { // 如果新的没text ==》 表示可能有children
			// isUndef ===> is undefined   表示vnode.text是undefined的
			if(isDef(oleCh) && isDef(ch)) { // 都有children
				updateChildren(elm, oldCh, ch, insertedVnodeQueue)
			} else if(isDef(ch)) { // 新的有children
				if(isDef(oldCh.text)){ // 如果旧的有text 则清空text
					api.setTextContent(elm, '');
				}
				addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue); // 添加children 且dom会进行api.insertBefore插入操作
			} else if (isDef(oldCh)) { // 旧的有children
				// 移除旧的children  因为新的无chilren无text
				removeVnodes(elm, oldCh, 0, oldCh.length - 1);
			} else if (isDef(oldCh.text)) {
				// 移除text 因为新的无chilren无text
				api.setTextContent(elm, '');
			}
		} else if (oldVnode.text !== vnode.text) { // 新的有text，无children；且与旧的text不相等
			if(isDef(ch)) { // 如果旧的有children ==》 表示无text
				removeVnodes(elm, oldCh, 0, oldCh.length - 1); // 则移除旧vnode里的children;  0 => oldCh.length - 1表示全移除；
			}
			api.setTextContent(elm, vnode.text); // 设置新的text
		}
	}
	
	// 补充
		function isUndef(s){
			return s === undefined;
		}
		function isDef(s){
			return s !== undefined;
		}		
```

​							 



​						oldStartIdx								oldEndIdx

​		旧children		a			b			c			d

​		新children		b			d			c			e

​							newStartIdx						newEndIdx



```javascript
	// 新旧结点对比
	/*
	* 整个过程就是：
	*		新旧结点各自首尾开始遍历（相当于对撞指针）,
	*		除了空的判断外，分为四种情况：新节点开始与旧结点开始、新节点结束与旧结点结束、新结点开始与旧结点结束、新结点结束与旧结点开始
	*		每次满足一个条件时，指针进行对应的左右移动
	*/
	function updateChildren(
		 	parentElm: Node,
            oldCh: VNode[],
            newCh: VNode[],
            insertedVnodeQueue: VNodeQueue
		) {
            let oldStartIdx = 0;
            let newStartIdx = 0;
            let oldEndIdx = oldCh.length - 1;
            let oldStartVnode = oldCh[0];
            let oldEndVnode = oldCh[oldEndIdx];
            let newEndIdx = newCh.length - 1;
            let newStartVnode = newCh[0];
            let newEndVnode = newCh[newEndIdx];
        	let oldKeyToIdx; // 这是个key对应index的map结构
            let idxInOld: number;
            let elmToMove: VNode;
    		let before: any;
		while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
            if () {// ...  判断为空的情况

            } 
                else if(sameVnode(oldStartVnode, newStartVnode)) {
                    patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue);// patch比对更新
                    oldStartVnode = oldCh[++oldStartIdx]; // oldStartIdx与oldStartVnode 都指向下一个
                    newStartVnode = newCh[++newStartIdx];
                }else if(sameVnode(oldEndVnode, newEndVnode)) {
                    patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue);
                    oldEndVnode = oldCh[--oldEndIdx]; // 右边的指针往左移
                    newEndVnode = newCh[--newEndIdx];
                }else if(sameVnode(oldStartVnode, newEndVnode)) {
                    patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue);
                    api.insertBefore(	// 将oldStartVnode放到当前oldEndVnode之后
                      parentElm,
                      oldStartVnode.elm!,
                      api.nextSibling(oldEndVnode.elm!)
                    );
                    oldStartVnode = oldCh[++oldStartIdx]; // 往右移
                    newEndVnode = newCh[--newEndIdx];	// 往左移
                }else if(sameVnode(oldEndVnode, newStartVnode)) {
                    patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue);
                    api.insertBefore(parentElm, oldEndVnode.elm!, oldStartVnode.elm!); // 将 oldEndVnode.elm 放到当前的 oldStartVnode.elm之前
                    oldEndVnode = oldCh[--oldEndIdx];
                    newStartVnode = newCh[++newStartIdx];
                } else {	// 以上四种情况都不是
                    idxInOld = oldKeyToIdx[newStartVnode.key as string]; // 拿新结点开始位置的key，用oldKeyToIdx找到old内是否有对应的key，返回索引，注意oldKeyToIdx是个map结构，没找到的话就返回undefined
                     if (isUndef(idxInOld)) { // 旧结点没有新结点start的key
                      api.insertBefore(	// 那么直接将新结点变成真实dom，插入到旧结点当前的start指向之前
                        parentElm,
                        createElm(newStartVnode, insertedVnodeQueue),
                        oldStartVnode.elm!
                      );
                    } else {
                          elmToMove = oldCh[idxInOld]; // 先存下原来的元素
                      	    if (elmToMove.sel !== newStartVnode.sel) { // 标签元素不相同
                                api.insertBefore( // 还是插入到oldStart之前，跟上面一样
                                  parentElm,
                                  createElm(newStartVnode, insertedVnodeQueue),
                                  oldStartVnode.elm!
                                );
                              } else { // 相同的话，就进行patchVnode比对
                                patchVnode(elmToMove, newStartVnode, insertedVnodeQueue);
                                oldCh[idxInOld] = undefined as any; // 旧结点原来的位置清空
                                api.insertBefore(parentElm, elmToMove.elm!, oldStartVnode.elm!); // 将事先保存好的oldvnode的元素，插入到当前start之前
                          }
                     }
                    newStartVnode = newCh[++newStartIdx]; // 将新结点指向下一个
                }
        }
	}
```

```note
整个过程就是： 从patch函数开始，先比对新旧vnode是否相似，不相似的话就创建dom元素进行挂载elm上；相似的话，就进入patchVnode进行比对，patchVnode完成的功能就是将oldVnode根据newVnode进行改变，最后得到的oldVnode肯定就是更新后(除了两个都是children的情况，其余时dom也更新了)的vnode了；在patchVnode过程中，遇到新旧结点都有children的情况时,这时就要进行updateChildren了，进行比对children，children内每个vnode对比时，相似的vnode又会进行新旧vnode的比对即patchVnode；
	这样就相当于 patchVnode 与 updateChildren 交替执行，实现整个vnode的比对、旧vnode的更新、真实dom的更新完成.
```



```javascript
// domApi 与 htmlDomApi

// domApi
// 就是自定义的dom操作方法，需要提供和 htmlDomApi 一样的操作函数
// 其实也就是为了实现自定义话

// htmlDomApi
// 就是封装一些操作dom的方法，提供给外部使用
// 如下：
function createElement(
  tagName: any,
  options?: ElementCreationOptions
): HTMLElement {
  return document.createElement(tagName, options);
}

function createElementNS(
  namespaceURI: string,
  qualifiedName: string,
  options?: ElementCreationOptions
): Element {
  return document.createElementNS(namespaceURI, qualifiedName, options);
}

function createTextNode(text: string): Text {
  return document.createTextNode(text);
}

function createComment(text: string): Comment {
  return document.createComment(text);
}

function insertBefore(
  parentNode: Node,
  newNode: Node,
  referenceNode: Node | null
): void {
  parentNode.insertBefore(newNode, referenceNode);
}

function removeChild(node: Node, child: Node): void {
  node.removeChild(child);
}

function appendChild(node: Node, child: Node): void {
  node.appendChild(child);
}

function parentNode(node: Node): Node | null {
  return node.parentNode;
}

function nextSibling(node: Node): Node | null {
  return node.nextSibling;
}

function tagName(elm: Element): string {
  return elm.tagName;
}

function setTextContent(node: Node, text: string | null): void {
  node.textContent = text;
}

function getTextContent(node: Node): string | null {
  return node.textContent;
}

function isElement(node: Node): node is Element {
  return node.nodeType === 1;
}

function isText(node: Node): node is Text {
  return node.nodeType === 3;
}

function isComment(node: Node): node is Comment {
  return node.nodeType === 8;
}

export const htmlDomApi: DOMAPI = {
  createElement,
  createElementNS,
  createTextNode,
  createComment,
  insertBefore,
  removeChild,
  appendChild,
  parentNode,
  nextSibling,
  tagName,
  setTextContent,
  getTextContent,
  isElement,
  isText,
  isComment,
};

```

> 像小程序之类的，因为底层操作的不是dom了，可能就传的是 自定义的domApi