## 3.1 虚拟DOM
### 3.1.1 前言

🔥 操作真实DOM的代价
  ```javascript
    let div = document.createElement('div')
    let str = ''
    for (const key in div) {
      str += key + ''
    }
    console.log(str)
        
 ```
![](~@/vue2.0/vortual1.png)

从打印结果可以看出，一个dom会有很多属性；真实的dom节点入栈执行会占据很大的内存，当我们频繁的操作会产生性能问题

 我们用传统的开发模式，用原生的js和jq操作DOM时，浏览器会从构建DOM树到绘制从头到尾执行一遍，如果我们更新10个dom节点，浏览器收到第一个dom请求后并不知道后面还有9次更新操作，最终会执行10次。如果第一次计算完，紧接这下一个DOM更新请求更改了前一次的DOM；那么前一次的dom更新就是白白的性能浪费，虽然计算机硬件一直迭代更新，但是操作dom的代价仍然是昂归的，频繁操作还会出现页面卡顿，影响用户体验。

🔥 为什么虚拟DOM？

虚拟dom就是为了解决浏览器性能问题而设计出来的，如果有10次dom更新的操作，虚拟dom不会立即去操作dom，而是将这去10次更新的diff内容保存到本地的一个js对象中，最终将这个js对象一次性patch到DOM树上，再进行后续的操作，避免大量无畏的计算量，所以用js对象模拟DOM节点的好处是页面的更新可以先全部反应到这个js对象上，操作js对象的速度显然更快，等待更新完成后，在将最终的js对象映射真实的DOM。

🔥 vue中虚拟DOM的表现

 ```javascript
 // 通过js对象描述的dom结构
    {
        tag: 'div'
        data: {
            id: 'app',
            class: 'main'
        },
        children: [
            {
                tag: 'p',
                text: 'this is test'
            }
        ]
    }

  //最后真实渲染的dom结构
    <div id="app" class="mian">
       <p>this is test</p>
   </div>
    
 ```
### 3.1.2  VNode类

 ```javascript
// 源码地址：src/core/vdom/vnode.js
// 通过vNode类，实例化出不同的虚拟DOM节点
export default class VNode {

  constructor (
    tag?: string, // 当前节点标签名
    data?: VNodeData, // // 当前节点的数据对象，也就是标签上的属性；包括attrs,style,hook等具体包含的字段可以参考/types/vnode.d.ts
    children?: ?Array<VNode>, ////数组类型，包含当前节点的子节点
    text?: string, // 当前节点的文本
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag // 当前节点标签名
    this.data = data // 当前节点的数据对象，也就是标签上的属性；包括attrs,style,hook等具体包含的字段可以参考/types/vnode.d.ts
    this.children = children //数组类型，包含当前节点的子节点
    this.text = text // 当前节点的文本
    this.elm = elm // 当前虚拟节点对应的真实的dom节点
    this.ns = undefined // 节点的namespace（命名空间）
    this.context = context // 编译作用域，当前节点对应的vue实例
    this.fnContext = undefined // 函数组件化的作用域，当前组件对应的vue实例
    this.fnOptions = undefined  // 函数式组件Option选项
    this.fnScopeId = undefined
    this.key = data && data.key // 节点的key属性，用作节点的标识，有利于patch优化
    this.componentOptions = componentOptions // 创建组件实例时会用到的选项信息
    this.componentInstance = undefined //当前组件节点对应的vue实例
    this.parent = undefined //组件的占位节点
    this.raw = false // 是否为原生HTML或只是普通文本，innerHTML的时候为true，textContent的时候为false
    this.isStatic = false //静态节点标识
    this.isRootInsert = true // 是否作为根节点插入被<transition>包裹的节点，该属性的值为false
    this.isComment = false //当前节点是否是注释节点
    this.isCloned = false //当前节点是否为克隆节点
    this.isOnce = false // 当前节点是否有v-once指令
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }

  // DEPRECATED:向后兼容组件的别名
  /* istanbul ignore next */
  get child (): Component | void {
    return this.componentInstance
  }
}
    
 ```
VNode类中包含了描述一个真实dom节点所需要的一系列属性，通过 VNode类可以描述各种真实dom节点

### 3.1.3 VNode类能描述的类型节点

- EmptyVNode: 没有内容的注释节点

- TextVNode: 文本节点

- CloneVNode: 克隆节点，可以是以上任意类型的节点，唯一的区别在于isCloned属性为true

- ComponentVNode: 组件节点

- FunctionalComponent: 函数式组件节点

- ElementVNode: 普通元素节点

...
1. EmptyVNode（注释节点）
 ```javascript
 // 源码地址：src/core/vdom/vnode.js
 // 创建注释节点
export const createEmptyVNode = (text: string = '') => {
  const node = new VNode()
  node.text = text
  node.isComment = true //isComment为true，说明是一个注释节点
  return node
}
 ```
 从上面可以看出注释节点只需2个属性，text表示是注释内容；isComment表示是否是个注释节点

2. TextVNode（文本节点）
 ```javascript
 // 源码地址：src/core/vdom/vnode.js
// 创建文本节点
export function createTextVNode (val: string | number) {
  return new VNode(undefined, undefined, undefined, String(val))
}

 ```
 文本节点只需传入文本值即可

3. CloneVNode（克隆节点）
 ```javascript
 // 源码地址：src/core/vdom/vnode.js
// 创建克隆节点
export function cloneVNode (vnode: VNode): VNode {
  const cloned = new VNode(
    vnode.tag,
    vnode.data,
    // #7975
    // clone children array to avoid mutating original in case of cloning
    // a child.
    vnode.children && vnode.children.slice(),
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  )
  cloned.ns = vnode.ns
  cloned.isStatic = vnode.isStatic
  cloned.key = vnode.key
  cloned.isComment = vnode.isComment
  cloned.fnContext = vnode.fnContext
  cloned.fnOptions = vnode.fnOptions
  cloned.fnScopeId = vnode.fnScopeId
  cloned.asyncMeta = vnode.asyncMeta
  cloned.isCloned = true
  return cloned
}

 ```
 克隆节点就是把传入的节点的属性全部赋值到新创建的节点上

4. ComponentVNode（组件节点）

   源码地址：src/core/vdom/create-component.js

   组件节点除了有普通元素节点的属性之外，还有2个私有的属性
   - componentOptions  创建组件实例时会用到的选项信息
   - componentInstance 当前组件节点对应的vue实例

5. FunctionalComponent（函数式组件节点）

   源码地址：src/core/vdom/create-functional-component.js

    函数式组件2个私有的属性
   - fnContext  函数组件化的作用域，当前组件对应的vue实例
   - fnOptions  函数式组件Option选项

6. ElementVNode（普通元素节点）

   源码地址：src/core/vdom/create-element.js

### 3.1.4  VNode类的作用

VNode类用js对象形式描述真实的dom，在vue初始化阶段，我们把`template`模板用`vnode`类实例化成js对象并缓存下来，当数据发生变化重新渲染页面的时候，我们把数据发生变化后用`vnode`类实例化的js对象与前一次缓存下来描述dom节点的js对象进行对比，找出差异；然后根据有差异的节点创建出真实的节点插入视图当中

### 3.1.3  总结

 虚拟dom就是用以对象的形式去描述真实的dom，用js计算的性能换取操作真实dom所消耗的性能

## 3.2 diff算法

### 3.2.1 前言

上章我们学习了虚拟dom，知道了渲染真实dom的开销很大，如果我们修改了某个数据，如果直接渲染到真实的dom上会引起整个dom树的重绘和重排；
有没有可能我们只更新修改的那一块dom，diff算法能帮到我们，我们先根据真实的dom生成一个virtual DOM树，当virtual dom树的某个节点发生变化后生成一个新的vnode，然后Vnode和oldVnode进行对比，一边比较一边给真实DOM打补丁，找出差异的过程就是diff的过程。

<font color="black">**vnode：数据变化后要渲染的虚拟的dom节点**</font>

<font color="black">**oldVnode：数据变化前视图对应的虚拟dom节点**</font>

### 3.2.2 patch

<font color="blue">**patch的过程就是以新的Vnode为基准，去改造旧的oldNode，让其跟新的一样；有人会说了，直接把旧的替换成新的就行了吗，如果这样做的话就是更新整个视图，而我们现在想做的是哪里变化了更新哪里。**</font>


🔥 patch的的特点

   pach的过程只会进行同级比较，不会跨级

![](~@/vue2.0/patch.png)

🔥 patch的过程就是做3件事情
- 创建节点：Vnode里有的，oldNode没有，那么就在oldNode里创建
- 删除节点：Vnode里没有，oldNode有，那么旧在oldNode删除
- 更新节点：Vnode和oldNode都有，那么以Vnode基准去更新oldNode

🔥 了解一下oldVnode有哪些属性
  ```javascript
// body下的 <div id="test" class="main"><div> 
// 对应的 oldVnode 就是

{
  elm:  div  //对真实的节点的引用，本例中就是document.querySelector('v#test.main')
  tagName: 'DIV',   //节点的标签
  sel: 'div#test.main'  //节点的选择器
  data: null,       // 一个存储节点属性的对象，对应节点的el[prop]属性，例如onclick , style
  children: [], //存储子节点的数组，每个子节点也是vnode结构
  text: null,    //如果是文本节点，对应文本节点的textContent，否则为null
}
 
 ```
 需要注意的是，el属性引用的是此virtual dom对应的真实dom，patch的vnode参数的elm最初是null，因为patch之前它还没有对应的真实dom      

### 3.2.3 创建节点
从上章我们知道通过Vnode类可以创建6种描述的dom节点的实例，是实际只有3种会被创建，并插入dom当中，3种分别是：元素节点、注释节点、文本节点
```javascript
// 源码位置: /src/core/vdom/patch.js

function createElm (vnode, parentElm, refElm) {
    const data = vnode.data
    const children = vnode.children
    const tag = vnode.tag
    if (isDef(tag)) { //判断是否有tag标签
      vnode.elm = nodeOps.createElement(tag, vnode)   // 创建元素节点
      createChildren(vnode, children, insertedVnodeQueue) // 创建元素节点的子节点
      insert(parentElm, vnode.elm, refElm)       // 插入到DOM中
    } else if (isTrue(vnode.isComment)) {  //判断注释属性是否是true，true的话就是注释节点
      vnode.elm = nodeOps.createComment(vnode.text)  // 创建注释节点
      insert(parentElm, vnode.elm, refElm)           // 插入到DOM中
    } else { // 如果不是元素节点和注释节点，那么就是文本节点
      vnode.elm = nodeOps.createTextNode(vnode.text)  // 创建文本节点
      insert(parentElm, vnode.elm, refElm)           // 插入到DOM中
    }
  }
function isDef (v) {
   return v !== undefined && v !== null
}       
```
### 3.2.4 删除节点
删除节点比较简单，只需调用删除元素的父元素的removeChild方法
```javascript
// 源码位置: /src/core/vdom/patch.js
function removeNode (el) {
    const parent = nodeOps.parentNode(el) //获取父节点
    // element may have already been removed due to v-html / v-text
    if (isDef(parent)) {
      nodeOps.removeChild(parent, el) // 调用父节点的removeChild方法 
    }
  }

function isDef (v) {
   return v !== undefined && v !== null
}

```
### 3.2.5 更新节点
更新节点是vNode和oldVnode都存在时，我们需要细致的找出不同的地方

🔥 先了解一下静态节点

  ```html
      <div>标题<div> 
 ```
像上面的节点就是静态节点，就是不用变化的节点，没有绑定任何变量，第一次渲染后，以后就不会变化了

🔥 这个函数做了一下事情
1. 找到真实的dom节点，称之为elm
2. 判断vNode和oldnode如果是同一个对象，则直接return
3. 如果vNode和oldnode都是静态节点，则直接return
3. 如果vnode没有文本节点
     - 2者都有子节点且不相同，则执行updateChildren比较子节点
     - 若只有vnode存在子节点，在判断oldVnode是否有文本，如果有就清除，然后将vnode的子节点替换到真实的dom中去
     - 若只有oldVnode存在子节点，则清空dom种子节点的存在
     - 若2者都没有子节点，则oldnode种有文本，则清空oldnode的文本
4. 如果vnode和oldVnode都有文本节点且不相同：
则将elm的文本节点设置为vnode的文本节点(文本节点就是text对应的值)

```javascript
// 源码位置: /src/core/vdom/patch.js
  function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
    // vnode和oldVode是完全一样，说明引用一致，没有什么变化；如果是就退出程序
    if (oldVnode === vnode) {
      return
    }
 
    // 让vnode.elm引用到现在的真实dom上，当elm修改时，vnode.elm会同步变化
    const elm = vnode.elm = oldVnode.elm

    // 如果vnode和oldVnode都是静态节点就退出程序，静态节点，无论数据发生任何变化都与它无关
    if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
    ) {
      vnode.componentInstance = oldVnode.componentInstance
      return
    }

    const oldCh = oldVnode.children
    const ch = vnode.children
 
    // vnode如果没有text属性
    if (isUndef(vnode.text)) {
      // 如果vnode的子节点和oldVnode的子节点都存在
      if (isDef(oldCh) && isDef(ch)) {
        //  若都存在且不相同，则更新子节点，这是diff的核心
        if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
      } 
      // 若只有vnode存在子节点
      else if (isDef(ch)) {
        if (process.env.NODE_ENV !== 'production') {
          checkDuplicateKeys(ch)
        }
         // 判断oldVnode是否有文本，如果有则清空dom中的文本，再把vnode的子节点添加到真实的DOM中
        if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
   
      } 
      // 若只有oldVnode存在子节点
      else if (isDef(oldCh)) {
        // 清空dom中的节点
        removeVnodes(oldCh, 0, oldCh.length - 1)
     
      } 
      // 如果oldVnode和node都没有子节点，但oldVnode有text
      else if (isDef(oldVnode.text)) {
        // 那么清空oldVnode文本
        nodeOps.setTextContent(elm, '')
      }

    // 如果vnode和oldVnode有text属性，但是oldVnode和vnode的text不相同
    } else if (oldVnode.text !== vnode.text) {
      // 不相同则用vnode.text替换真实的dom文本
      nodeOps.setTextContent(elm, vnode.text)
    }
  
  }

```

### 3.2.3 diff的流程
从上面看下来,我们了解到了patch要做些什么,无非就是创建、删除、更新；下面我们来分析整个流程

初始化时，通过render函数生成vNode，同时也进行了Watcher的绑定，当数据发生变化时，会执行_update方法，生成一个新的VNode对象，然后调用 __patch__方法，比较VNode和oldNode，最后将节点的差异更新到真实的DOM树上
vue在update的时候会调用以下函数
  ```javascript
   export function lifecycleMixin (Vue: Class<Component>) {
    Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
        const vm: Component = this
        const prevVnode = vpatch__ is injected in entry points
        // based on the rendering backend used.
        if (!prevVnode) { 
          // 初次渲染，会传入原生dom节点和虚拟dom节点
          vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
        } else {
          // 更新
          vm.$el = vm.__patch__(prevVnode, vnode)
        }
      }

   }
        
 ```
vm._patch_才是进行vnode diff的核心，来看看patch是怎么打补丁的（代码只保留核心部分）

patch的会有3种情况
  1. 如果vnode不存在，oldVnode存在；那么需要销毁真实的dom节点
  2. 如果vnode存在，oldVnode不存在；那么需要创建节点
  3. 2者都存在，进行比较
```javascript
 /**
   * oldVnode 旧的真实的DOM节点
   * vnode  节点变化后生成新的Vnode
   */
 function patch (oldVnode, vnode) {

    // 如果vnode不存在，oldVnode存在，需要销毁旧节点，则调用invokeDestroyHook(oldVnode)
    if (isUndef(vnode)) {
      if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
      return
    }

    let isInitialPatch = false
    const insertedVnodeQueue = []
      // 如果oldVnode不存在，Vnode存在，那么创建新节点，则调用createElm()
    if (isUndef(oldVnode)) {
      // empty mount (likely as component), create new root element
      isInitialPatch = true
      createElm(vnode, insertedVnodeQueue)
      // 当Vnode和oldVnode都存在时
    } else {

    }
 }
        
 ```



