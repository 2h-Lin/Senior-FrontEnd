## 4.1 工程设计范式

- 工程设计范式有哪些种类

Rails Style / Domain Style

- 相信很多有一定经验的开发者都会遇到选择这种问题
  - 一个看似分层良好的古老工程，随着业务发展却越来越臃肿和低效
  - 一个新的项目，因为这种隐形问题上的纠结，而耽误了自己时间和精力

### 4.1.1 工程范式分类

🚀 Rails Style

```js
// egg应用典型结构
|── app               
|   ├── config     
|   ├── controller
|   ├── extend
|   ├── public
|   ├── router.ts
|   ├── service
|   └── view      
├── app.ts                    
├── agent.ts
├── config
├── logs
├── test
├── typings
├── README.md
├── package.json            
└── yarn.lock
```
```js
// Rails Style TodoList
root
|── reducers              
|   ├── todoReducer.js
|   └── fileterReducer.js
├── actions                 
|   ├── todoActions.js                         
|   └── filterAction.js
├── components                 
|   ├── todoList.jsx
|   ├── todoItem.jsx               
|   └── filter.jsx
├── containers                 
|   ├── todoListContainer.jsx
|   ├── todoItemContainer.jsx               
|   └── filterContainer.jsx
├── App.jsx
└── index.js
```
- Rails Style的特点
  - 专注于纵向的“层”的划分
  - 同一类文件放置在同一目录下
  
- 优势：
    - 便于合并导出
    - 便于进行“层”的扩展

- 不足：
    - 依赖关系难以直接地分析
    - 对功能的修改会涉及到大量的目录切换
    - 难以水平拆分

🚀 Domain style

```js
// Aant Design
├── ...                            
├── components                
|   ├── util 
        └── ...    
|   ├── alert             
|   |   ├── demo         
|   |   ├── index.en-US.msd    
|   |   ├── index.tsx
|   |   ├── index.zh-CN.md
|   |   └── style
|   ├── anchor             
|   |   ├── Anchor.tsx
|   |   ├── AnchorLink.tsx
|   |   ├── __tests__
|   |   ├── demo
|   |   ├── index.en-US.msd    
|   |   ├── index.tsx
|   |   ├── index.zh-CN.md
|   |   └── style
└── ... 
```
- Domain style的特点
  - 专注于横向的 “功能” 的划分
  - 同一个feature放置在同一目录下

- 优势：
  - 便于水平拆分
  - 便于进行“功能”的扩展

- 不足
  - 会产生大量的重复结构
  - 难以垂直拆分

### 4.1.1 如何选择工程范式
 
- 单一功能的项目
  - 库、三方包：fs-extra、axios等
  - 由于不存在水平拆分的必要性、故可以选择Rails Style
  - 易于扩展
  - 减少重复代码
 
