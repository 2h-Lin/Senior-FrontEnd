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

在vue初始化阶段，我们把`template`模板用`vnode`类实例化成js对象并缓存下来，当数据发生变化重新渲染页面的时候，我们把数据发生变化后用`vnode`类实例化的js对象与前一次缓存下来描述dom节点的js对象进行对比，找出差异；然后根据有差异的节点创建出真实的节点插入视图当中

### 3.1.3  总结

 虚拟dom就是用以对象的形式去描述真实的dom，用js计算的性能换取操作真实dom所消耗的性能

## 3.2 diff算法


