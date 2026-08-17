## Redux

Redux 是 第三方状态管理库

### 说说你对 Redux 的理解

库：react-redux, @reduxjs/toolkit

发布订阅模式

- 特点 : 单向数据流, 数据只读, 使用纯函数来执行修改 (三大原则)
- 基础 : 数据流更新

### Redux, 主要解决什么问题

- 状态管理的复制性
- 状态共享和同步
- 纯函数可调试

### Redux 中异步的请求怎么处理

通过中间件来处理

- redux-thunk
- redux-sage

### Redux 请求中间件如何处理并发

中间件可以添加日志记录

### Redux 状态管理器和变量挂载到 window 中有什么区别

redux : 支持多个 api , 可调试,可观察行，复杂
window: 简单，快速成型

### Redux 中间件是怎么拿到 store 和 action ?

使用高阶函数，第一层传进去的是 store , 第二层是 next, 第三层是 action;
