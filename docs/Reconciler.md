## Reconciler 的作用

`Reconciler`是 react 中运行时的模块，根据 jsx 产生的 element,调用宿主环境中的 api,最终显示到 ui 界面

Reconciler 操作 element 和 Fiber
已知 react jsx 会产出 element,结构如下：

```js
const ReactElement = function (
  type: Type,
  key: Key,
  ref: Ref,
  props: Props
): ReactElement {
  const element = {
    $$typeof: REACT_ELEMENT_TYPE,
    type,
    key,
    ref,
    props,
    _mark: 'wy set',
  };
  return element;
};
```

element 除了和子 element 之间有关联关系，和别的 element 之间的关系难以获知,且不便于扩展。

> fiberNode 关联节点,结构如下

fiberNode 通过 return, sibling, child 关联父，兄弟，孩子 fiberNode,通过 flags 标价增删改等副作用

```js
// react-source-learn/packages/react-Reconciler/src/ReactFiber.ts
import { Key, Ref, Props } from 'shared/ReactTypes';
import { WorkTag } from './ReactWorkTags';
import { Flags, NoFlags } from './ReactFiberFlags';
export class FiberNode {
  tag: WorkTag; // fiberNode 是什么类型的节点
  ref: Ref;

  key: Key;
  stateNode: any; // 当前fiberNode对应的真实的dom 例如 div
  type: any; // 对应的element的函数， 例如functionComponent的函数

  return: FiberNode | null;
  sibling: FiberNode | null;
  child: FiberNode | null;
  alternate: FiberNode | null; // current 和 wip 之间的fiberNode相互指向
  index: number;

  pendingProps: Props; // 在fiberNode作为工作单元刚开始工作的时候的props
  memoizedProps: Props; // 在fiberNode作为工作单元结束工作的时候props

  flags: Flags; // 当前节点的操作类型 例如 插入，删除 flags 被统称为副作用

  constructor(tag: WorkTag, pendingProps: Props, key: Key) {
    this.tag = tag;
    this.ref = null;
    this.key = key;
    this.stateNode = null;
    this.type = null;

    this.return = null;
    this.sibling = null;
    this.child = null;
    this.index = 0;

    this.alternate = null;

    this.pendingProps = pendingProps;
    this.memoizedProps = null;

    this.flags = NoFlags;
  }
}
```

## Reconciler 工作方式

Reconciler 工作方式是比较 reactElement 和 fiberNode，来创建 fiberNode 或者产生子 fiberNode

- 当 reactElement 存在，fiberNode 为 null，就创建当前 reactElement 对应的 fiberNode
- 当 reactElement，fiberNode 都存在，根据 reactElement，fiberNode 的差异，做增删改

reactElement，fiberNode 比较之后，会产生 current fiber tree 和 workInProgress fiber Tree

- current: 真实 ui 对应的 fiber tree
- workInProgress：更新时，在后台 Reconciler 中计算产生的 workInProgress

当前 workInProgress 完成之后，current 和 workInProgress 立即相互交换，这个过程称为双缓冲

## Reconciler 访问 reactElement 方式

Reconciler 中访问 reactElement 采用深度优先遍历的方式，这是一个递归的过程。Reconciler 把这个过程分为“递（beginWork）”和“归（completeWork）”

> 递归的操作伪代码

```js
// react-source-learn/packages/react-Reconciler/src/ReactFiberWorkLoop.ts
import { FiberNode } from './ReactFiber';
import { beginWork } from './ReactFiberBeginWork';
import { completeWork } from './ReactFiberCompleteWork';
let workInProgressRoot: FiberNode | null; // 正在被执行的fiberNode

function prepareFreshStack(root: FiberNode) {
  workInProgressRoot = root;
}

function renderRoot(root: FiberNode) {
  prepareFreshStack(root);
  do {
    try {
      workLoop();
    } catch (e) {
      console.warn('workLoop出错', e);
      workInProgressRoot = null;
    }
  } while (true);
}

function workLoop() {
  while (workInProgressRoot !== null) {
    performUnitOfWork(workInProgressRoot);
  }
}

// 递操作
function performUnitOfWork(fiber: FiberNode) {
  const next = beginWork(fiber); // 子fiberNode
  if (next !== null) {
    workInProgressRoot = next;
  } else {
    completeUnitWork(fiber);
  }
}

// 归操作
function completeUnitWork(fiber: FiberNode) {
  let node: FiberNode | null = fiber;
  do {
    completeWork(node);
    const sibling = node.sibling;
    if (sibling !== null) {
      workInProgressRoot = sibling;
      return;
    }

    node = node.return;
    workInProgressRoot = node;
  } while (node !== null);
}
```
