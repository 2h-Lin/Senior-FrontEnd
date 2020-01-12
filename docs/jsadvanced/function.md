## 2.1 javaScript内存管理

### 2.1.1 js内存机制

🔥内存空间：栈内存（stack）、堆内存（heap）

1. 栈内存：所有原始数据类型都存储在栈内存中，如果删除一个栈原始数据，遵循先进后出；如下图：a最先进栈，最后出栈。
![](~@/jsasvanced/stack.png)

2. 堆内存：引用数据类型会在堆内存中开辟一个空间，并且会有一个十六进制的内存地址，在栈内存中声明的变量的值就是十六进制的内存地址。

![](~@/jsasvanced/heap.png)

 函数也是引用数据类型，我们定一个函数的时候，会在堆内存中开辟空间，会以字符串的形式存储到堆内存中去，如下图：
 
 ![](~@/jsasvanced/function.png)

 ```javascript
 // 我们直接打印fn会出现一段字符窜
 console.log(fn)
  f fn() {
      var t=10;
      var f=10;
      console.log(i+j)
  }
  // 加上括号才执行里面的代码
  fn() // 20
  ```
 
### 2.1.2 垃圾回收
 🔥概念：（我们平时创建所有的数据类型都需要内存）
   所谓的垃圾回收就是找出那些不再继续使用的变量，然后释放出其所占用的内存，垃圾回收会按照固定的时间间隔周期性的执行这一操作。

 🔥javaScript使用的垃圾回收机制来自动管理内存，垃圾回收是把双刃剑；垃圾回收是不可见的

  -  优势：可以大幅简化程序的内存管理代码，降低程序员的负担，减少因长时间运转而带来的内存泄漏问题。

  - 不足：程序员无法掌控内存，javascript没有暴露任何关于内存的api，无法强迫进行垃圾回收，无法干预内存管理。

🔥 垃圾回收的方式

1. 引用计数（reference counting）

      跟踪记录每个值被引用的次数，如果一个值引用次数是0，就表示这个值不再用到了，因此可以将这块内存释放

      原理：每次引用加1，被释放减1，当这个值的引用次数变成0时，就将其内存空间释放。
 ```javascript
    let obj= {a:10}; // 引用+1
    let obj1={a:10} // 引用+1
    obj ={} //引用减1
    obj1=null //引用为0
 ``` 
引用计数的bug：循环引用
  ```javascript
    // ie8较早的浏览器,现在浏览器不会出现这个问题
    function Fn (){
        var objA={a:10}
        var objB={b:10}
        objA.c=objB
        objB.c=objA
    }
 ``` 
 2. 标记清除（现代浏览采用标记清除的方式）

 🔥概念：标记清除指的是当变量进入环境时，这个变量标记为“进入环境”;而当变量离开环境时，则将其标记为“离开环境”，最后垃圾收集器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间（所谓的环境就是执行环境）
 
 🔥全局执行环境
   - 最外围的执行环境
   - 根据宿主环境的不同表示的执行环境的对象也不一样，在浏览器中全局执行环境被认为是window对象
   - 全局变量和函数都是作为window对象的属性和方法创建的
   - 某个执行环境中的所有代码执行完毕后，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁（全局执行环境只有当关闭网页的时候才会被销毁）

  🔥环境栈（局部）
  - 每个函数都有自己的执行环境，当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。而在函数执行之后，栈将其环境弹出，把控制权返回给之浅的执行环境，ECMAScript程序中的执行流正式由这个方便的机制控制着
 
```javascript
    function foo (){
       var a = 10   // 被标记进入执行环境
       var b = ‘hello’ // 被标记进入执行环境
    }
    foo()  //执行完毕，a 和 b 被标记离开执行环境，内存被回收
```
### 2.1.3 V8内存管理机制

  🔥V8引擎限制内存的原因
   - V8最初为浏览器设计，不太可能遇到大量内存的使用场景（表层原因）
   - 防止因为垃圾回收所导致的线程暂停执行的时间过长（深层原因，按照官方的说法以1.5G的垃圾回收为例，v8做一次小的垃圾回收需要50毫秒以上，做一次非增量的垃圾回收需要1秒以上，这里的时间是指javascript线程暂停执行的时间，这是不可接受的，
   v8直接限制了内存的大小，如果说在node.js中操作大内存的对象，可以通过去修改设置去完成，或者是避开这种限制，1.7g是在v8引擎方面做的限制，我们可以使用buffer对象，而buffer对象的内存分配是在c++层面进行的，c++的内存不受v8的限制）

  🔥V8回收策略
   - v8采用可一种分代回收的策略，将内存分为两个生代；新生代和老生代
   - v8分别对新生代和老生代使用不同的来及回收算法来提升垃圾回收效率

  🔥新生代垃圾回收
  from和to组成一个`Semispace`（半空间）当我们分配对象时，先在from对象中进行分配，当垃圾回收运行时先检查from中的对象，当`obj2`需要回收时将其留在from空间，而`ob1`分配到to空间，然后进行反转，将from空间和to空间进行互换，进行垃圾
  回收时，将to空间的内存进行释放，简而言之from空间存放不被释放的对象，to空间存放被释放的对象，当垃圾回收时将to空间的对象全部进行回收
  ![](~@/jsasvanced/v8.jpg)

  🔥新生代对象的晋升（新生代中用来存放，生命较短的对象，老生代存放生命较长的对象）
   - 在新生代垃圾回收的过程中，当一个对象经过多次复制后依然存活，它将会被认为是生命周期较长的对象，随后会被移动到老生代中，采取新的算法进行管理
   - 在From空间和To空间进行反转的过程中，如果To空间中的使用量已经超过了25%，那么就将From中的对象直接晋升到老生代内存空间中

  🔥老生代垃圾回收（有2种回收方法）
   - 老生代内存空间是一个连续的结构
  ![](~@/jsasvanced/oldshengdai.jpg)
  1. 标记清除（Mark Sweep）
  Mark Sweep 是将需要被回收的对象进行标记，在垃圾回收运行时直接释放相应的地址空间,红色的区域就是需要被回收的
  ![](~@/jsasvanced/marksweep.jpg)
  - 标记合并（Mark Compact）
  Mark Compact将存活的对象移动到一边，将需要被回收的对象移动到另一边，然后对需要被回收的对象区域进行整体的垃圾回收
  ![](~@/jsasvanced/markconpact.jpg)
  

## 2.2 如何保证你的代码质量

### 2.2.1 单元测试 (写一段代码去验证另一段代码，检测的对象可以是样式、功能、组件等)

  🔥概念
   - 测试一种验证我们的代码是否可以按预期工作的方法
   - 单元测试是对软件中的最小可测试单元进行检测和验证

  🔥前端单元测试的意义
   - 检测出潜在的bug
   - 快速反馈功能输出，验证代码是否达到预期
   - 保证代码重构的安全性
   - 方便协作开发

  🔥单元测试代码
   1. 案例1

   ``` javascript
        let add=(a,b)=>a+b //被测试的方法
        let result =add(1,2)
          // 写的测试的代码
        let expect=4
        if(result!==expect){
            throw new Error(`1+2应该等于${expect},但结果确是${result}`)
        }
      // 最后输出：Uncaught Error: 1+2应该等于4,但结果确是3 
   ```
  1. 案例2
   ``` javascript
        //被测试的方法
          let add=(a,b)=>a+b
        // 写的测试的代码
          let  expect = (res)=>{
              return {
                  toBe:(actual)=>{
                      if(res!==actual){
                      throw new Error(`预期值和实际值不符`)
                    }
                  }
              }
          }

      // expect(add(1,2)).toBe(4)
      let test =(desc,fn)=>{
          try{
              fn()
              console.log(`${desc}通过`); 
          }catch(err){
              console.log(`${desc}没有通过`);           
        }
      }
      test('加法测试',()=>{
          expect(add(1,2)).toBe(3)
      })
      // 最后输出：加法测试通过 
   ```      
### 2.2.1 jest的基础使用（facebook的一套测试javascript的框架）
  🔥安装
   1. 安装node
   2. yarn add -D jest
   3. 查看是否安装成功 npm ls jest

 🔥jest的基础使用
  1. 创建一个文件夹，然后npm init -y，然后下载jest：yarn add -D jest
  2. 在文件夹下创建math.js,这个文件是写被测试的代码；如下：
   ``` javascript
    let  add =(a,b) => a+b
      module.exports= {
          add
      }
   ```
   3. 在文件夹下创建math.test.js，这个文件写测试代码；如下。
  ``` javascript
        const { add } = require('./math')
        test ('加法测试',()=>{
            expect(add(1,2)).toBe(3)
        })
   ```
  4. 配置package.json里的script脚本
  ``` javascript
        "scripts": {
         "test": "jest"
        }
  ```
  5. 执行npm test，测试成功会出现以下信息
  ``` bash
        PASS  ./math.test.js
        ✓ 加法测试 (3ms)

      Test Suites: 1 passed, 1 total
      Tests:       1 passed, 1 total
      Snapshots:   0 total
      Time:        0.98s
      Ran all test suites.
       
  ```
  提示：具体代码可以在源码test/2.2/jest目录下查看


## 2.3 提高代码的可靠性

### 2.3.1 函数式编程

 🔥含义：函数式编程是一种编程范式，是一种构建计算机程序结构和元素的风格，它把计算看作是对数据函数的评估，避免了状态的变化和数据的可变。
       将我们的程序分解为一些更可复用，更可靠且更易于理解的部分，然后在将他们组合起来，形成一个更易推理的程序整体。

 1. 案例1:对一个数组每项加+1
   ``` javascript
        // 初级程序员
        let arr =[1,2,3,4]
        let newArr=[]
        for (var i=0;i<arr.length; i++){
          newArr.push(arr[i]+1)
        }
        console.log(newArr) //[2, 3, 4, 5]
  ```
  ``` javascript
      // 函数式编程
        let arr =[1,2,3,4]
        let newArr =(arr,fn)=>{
            let res=[]
          for (var i=0;i<arr.length; i++){
            res.push(fn(arr[i]))
           }
           return res
        }
        let add= item=>item+1 //每项加1
        let multi=item=>item*5 //每项乘5
        let sum =newArr(arr,add)
        let product =newArr(arr,multi)
        console.log(sum,product); // [2, 3, 4, 5] [5, 10, 15, 20]
  ```
  ### 2.3.2 纯函数
  🔥含义：如果函数的调用参数相同，则永远返回相同的结果。它不依赖于程序执行期间函数外部任何状态或数据的变化，必须只依赖于其输入的参数(相同的输入，必须得到相同的输出)。
   ``` javascript
        // 纯函数
        const calculatePrice=（price，discount）=> price * discount
        let price = calculatePrice（200，0，8）
        console.log(price)
    ```
    ``` javascript
        // 不纯函数
        const calculatePrice=（price，discount）=>{
          const dt= new Date().toISOString()
          console.log(`${dt}:${something}`)
          return something
        }
        foo('hello')
  ```
  ### 2.3.3 函数副作用
  - 当调用函数时，除了返回函数值外，还对注调用函数产生附加的影响
  - 例如修改全局变量（函数外的变量）或修改参数
    ``` javascript
      //函数外a被改变，这就是函数的副作用
      let a=5
      let foo =（）=> a = a * 10s
      console.log(a);  // 50
        
      let arr = [1,2,3,4,5,6]
      arr.slice(1,3)  //纯函数，返回[1,2],原数组不改变
      arr.splice(1,3) // 非纯函数，返回[2,3,4],原数组被改变
      arr.pop() // 非纯函数，返回6，原数组改变
    ```
    ``` javascript
    
      //通过依赖注入，对函数进行改进，所谓的依赖注入就是把不存的部分作为参数传入，把不存的代码提取出来；远离父函数；同时这么做不是为了消除副作用
      //主要是为了控制不确定性

      const foo =(d,log,something)=>{
        const dt =d.toISOString();
        return log(`${dt}:${something}}`);
      }
      const something ='你好'
      const d= new Date()
      const log =console.log.bind(console)
      foo(d,log,something)
    ```
     ### 2.3.4 函数副作用可变性和不可变性
     - 可变性是指一个变量创建以后可以任意修改
     - 不可变性指一个变量，一旦被创建，就永远不会发生改变，不可变性是函数式编程的核心概念
    ``` javascript
    // javascript中的对象都是引用类型，可变性使程序具有不确定性，调用函数foo后，我们的对象就发生了改变；这就是可变性，js中没有原生的不可变性
      let data={count:1}
      let foo =(data)=>{
        data.count=3
      }
      console.log(data.cont) // 1
      foo(data)
       console.log(data.cont) // 3

      // 改进后使我们的数据具有不可变性
      let data={count:1}
      let foo =(data)=>{
        let lily= JSON.parse(JSON.stringify(data)) // leg lily= {...data} 使用扩展运算符去做拷贝，只能拷贝第一层
        lily.count=3
      }
      console.log(data.cont) // 1
      foo(data)
       console.log(data.cont) // 1
    ```
    ## 2.4 compose函数pipe函数

    ### 2.4.1 compose函数
    🔥含义：
    - 将需要嵌套执行的函数平铺
    - 嵌套执行指的是一个函数的返回值将作为另一个函数的参数
    🔥作用：实现函数式编程中的Pointfree,使我们专注于转换而不是数据（Pointfree不使用所有处理的值，只合成运算过程，即我们所指的无参数分割）
    🔥案例，计算一个数加10在乘以10
    ``` javascript
      // 一般会这么做
      let calculate => x => (x+10) * 10
      console.log(calculate(10)) 

    ```
    ``` javascript
      // 用compose函数实现
        let add = x => x+10
        let multiply = y => y*10
        console.log(multiply(add(10)))

        let compose=function (){
            let args=[].slice.call(arguments)
           
            return function (x) {
                return args.reduceRight(function (total,current) { //从右往左执行args里的函数
                    console.log(total,current)
                   return current(total)
                },x)
            }
        }
        let calculate = compose(multiply,add)
        console.log(calculate,calculate(10)) // 200
    // 用es6实现 
    const compose = (...args) => x=> args.reduceRight((res,cb)=> cb(res),x)
    ```
    ### 2.4.1 pipe函数
     pipe函数compose类似，只不过从左往右执行

## 2.5 高阶函数
  🔥含义：
  - 高阶函数是对其他函数进行操作的函数，可以将它们作为参数或返回它们
  - 简单来说，高阶函数是一个函数，它接收函数作为参数或将函数作为输出返回 

  🔥map/reduce/filter

  ``` javascript
     // 用redece做累加
      let arr =[1,2,3,4,5]
      let sum =arr.reduce((pre,cur)=>{
        return pre +cur
      },10)

    // 用redece做去重
      let arr =[1,2,3,4,5,3,3,4]
      let newArr =arr.reduce((pre,cur)=>{
          pre.indexOf(cur)===-1 && pre.push(cur)
        return pre
      },[])
    console.log(newArr) //[1, 2, 3, 4, 5]
  ```
  🔥flat
  ``` javascript
        let arr=[[1,2,3,[23,3,[1,2]]]]
        let arr1=arr.flat(Infinity)   // 多维转一维数组
        let arr2=arr.flat(2) // // 多维转二维数组,默认值是1
        console.log(arr1,arr2)  // [1, 2, 3, 23, 3, 1, 2]  [1, 2, 3, 23, 3, Array(2)]
      
  ```
  🔥高阶函数的意义
  1. 参数为函数的高阶函数
  ``` javascript
        // 参数为函数的高阶函数
        function foo (f){
          // 判断是否为函数
          if((typeof f)==="function"){
            f()
          }
        }
        foo(function(){})   
  ```
  2. 返回值为函数的高阶函数
  ``` javascript
          // 回值为函数的高阶函数
          function foo (f){
            rerutn function(){}
          }
          foo()   
  ```
   3. 高阶函数的实际作用
  ``` javascript
        let callback = (value)=>{
          console.log(value)
        }
        let foo = (value,fn) =>{
          if(typeof fn==='function'){
            fn(value)
          }
        }
        foo('hello',callback)s
  ```
  ## 2.5 常用函数

  ### 2.5.1 memozition（缓存函数）
  🔥含义：缓存函数是指将上次的计算结果缓存起来，当下次调用时，如果遇到相同的参数，就直接返回缓存中的数据
  ``` javascript
      let add = (a ,b) => a+b
      // 假设memoize函数可以实现缓存
      let calculate = memoize(add)
      calculate(10,20); // 30
      calculate(10,20); // 相同的参数，第二次调用是，从缓存中取出数据，而并非重新计算一次
  ```
 🔥实现原理：把参数和对应的结果数据存到一个对象中去，调用时，判断参数对应的数据是否存在，存在就返回对应的结果数据
  ``` javascript
  // 缓存函数
     let memoize =function (func) {
       let cache= {}
       return function (key) {
         if(!cache[key]){
           cache[key] = func.apply(this,arguments)
         }
         return cache[key]
       }
     }
  ```
  ``` javascript
     /*
     *hasher也是个函数，是为了计算key，如果传入了hasher，就用hasher函数计算key；
     否则就用memoize函数传入的第一个参数，接着就去判断如果这个key没有被求值过，就去执行，
     最后我们将这个对象返回
     */
     let memoize =function (func,hasher) {
       var memoize = function (key) {
         var cache = memoize.cache
         var address='' + (hasher ? hasher.apply(this,arguments) : key)
         var (!cache[address]) chache[address] = func.apply(this,arguments)
         return cache[address]
       }
       memoize.cache={}
       return memoize
     }
     // 缓存函数可以是fei bo
  ```
  🔥案例：求斐波那且数列
  ``` javascript
      // 不用memoize的情况下，会执行453次
      var count=0
      var fibonacci= function (n) {
          count++
          return n < 2? n : fibonacci(n-1) + fibonacci(n-2)
      }
     for (var i=0;i<=10;i++){
        fibonacci(i) //453 
      }
      console.log(count)

    // 用memoize的情况下，会执行12次
      let memoize =function (func,hasher) {
       var memoize = function (key) {
         var cache = memoize.cache
         var address= '' + (hasher ? hasher.apply(this,arguments) : key);
         if (!cache[address]) cache[address] = func.apply(this,arguments)
         return cache[address]
       }
       memoize.cache={}
       return memoize
     }
      fibonacci =memoize(fibonacci)  
      for (var i=0;i<=10;i++){
        fibonacci(i) //453 12
      }
      //缓存函数能应付大量重复计算，或者大量依赖之前的结果的运算场景
      console.log(count)
  ```
  ### 2.5.2 curry(柯里化函数)
  🔥含义：在数学和计算机科学中，柯里化是一种将使用多个参数的一个函数转换成一些列使用一个参数的函数技术（把接受多个参数的函数转换成几个单一参数的函数）
   ``` javascript
    // 没有柯里化的函数
    function girl(name,age,single) {
      return `${name}${age}${single}`
    }
     girl('张三')(180)('单身')
     // 柯里化的函数
     function girl(name) {
       return function (age){
          return function (single){
            return `${name}${age}${single}`
         }
       }
     }
     girl('张三')(180)('单身')
  ```
  🔥案例1：检测字符串中是否有空格
  ``` javascript
     // 封装函数去检测
     let matching = (reg,str) => reg.test(str)
     matching(/\s+/g,'hello world') // true
     matching(/\s+/g,'abcdefg') // false
     
     // 柯里化
     let curry = (reg) => {
       return (str) =>{
         return reg.test(str)
       }
     }
    let hasSpace = curry(/\s+/g)
    hasSpace('hello word') // true
    hasSpace('abcdefg') // false
  ```
  🔥案例2：获取数组对象中的age的属性值
   ``` javascript
     let persons = [
       {name:'zs',age:21},
       {name:'ls',age:22}
     ]
     // 不柯里化
     let getage = persons.map(item=>{
       return item.age
     })
     // 用loadsh的curry 来实现
     const _=require('loadsh')
     let getProp= _.curry((key,obj)=>{
       return obj[key]
     })
    person.map(getProp('age'))

  ```
  🔥柯里化这个概念实现本身就难，平时写代码很难用到，关键理解其思想

  ### 2.5.3 偏函数
  🔥比较：
  - 柯里化是将一个多参数函数转换成多个单参数的函数，也就是将一个n元函数转换成n个一元函数
  - 偏函数则固定一个函数的一个或多个参数，也就是将一个n元函数转换成一个n-x元的函数

  - 柯里化：f(a,b,c)=f(a)(b)(c)
  - 偏函数：f(a,b,c)=f(a,b)(c)
  ``` javascript
    /*
      用bind函数实现偏函数，bind的另一个用法使一个函数拥有预设的初始参数，将这些参数写在bind的第一个参数后，
      当绑定函数调用时，会插入目标函数的初始位置，调用函数传入的参数会跟在bind传入的后面
    */
     let add = (x,y) => x+y
     let rst =add.bind(null,1)
     rst(2) //3
  ```
   ## 2.5 防抖和节流