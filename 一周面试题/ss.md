## React 中

### 本地调试 React 代码的方法

- 先将 React 代码下载到本地，进入项目文件夹后`yarn build`
- 利用 create-react-app 创建一个自己的项目
- 把 react 源码和自己刚刚创建的项目关联起来，之前 build 源码到 build 文件夹下面，然后 cd 到 react 文件夹下面的 build 文件夹下。里面有 node_modules 文件夹，进入此文件夹。发现有 react 文件夹和 react-dom 文件夹。分别进入到这两个文件夹。分别运行 yarn link。此时创建了两个快捷方式。react 和 react-dom
- cd 到自己项目的目录下，运行 yarn link react react-dom 。此时在你项目里就使用了 react 源码下的 build 的相关文件。如果你对 react 源码有修改，就刷新下项目，就能里面体现在你的项目里。

### 场景

假设有这样一个场景，父组件传递子组件一个 A 参数，子组件需要监听 A 参数的变化转换为 state。

### 16 之前

在 React 以前我们可以使用`componentWillReveiveProps`来监听`props`的变换

### 16 之后

在最新版本的 React 中可以使用新出的`getDerivedStateFromProps`进行 props 的监听，`getDerivedStateFromProps`可以返回`null`或者一个对象，如果是对象，则会更新`state`

### getDerivedStateFromProps 触发条件

我们的目标就是找到 `getDerivedStateFromProps`的 触发条件

我们知道，只要调用`setState`就会触发`getDerivedStateFromProps`，并且`props`的值相同，也会触发`getDerivedStateFromProps`(16.3 版本之后)

`setState`在`react.development.js`当中

```js
Component.prototype.setState = function(partialState, callback) {
  !(
    typeof partialState === "object" ||
    typeof partialState === "function" ||
    partialState == null
  )
    ? invariant(
        false,
        "setState(...): takes an object of state variables to update or a function which returns an object of state variables."
      )
    : void 0;
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};
```

```js
ReactNoopUpdateQueue {
    //...部分省略

    enqueueSetState: function (publicInstance, partialState, callback, callerName) {
    warnNoop(publicInstance, 'setState');
  }
}
```

执行的是一个警告方法

```js
function warnNoop(publicInstance, callerName) {
  {
    // 实例的构造体
    var _constructor = publicInstance.constructor;
    var componentName =
      (_constructor && (_constructor.displayName || _constructor.name)) ||
      "ReactClass";
    // 组成一个key 组件名称+方法名（列如setState）
    var warningKey = componentName + "." + callerName;
    // 如果已经输出过警告了就不会再输出
    if (didWarnStateUpdateForUnmountedComponent[warningKey]) {
      return;
    }
    // 在开发者工具的终端里输出警告日志 不能直接使用 component.setState来调用
    warningWithoutStack$1(
      false,
      "Can't call %s on a component that is not yet mounted. " +
        "This is a no-op, but it might indicate a bug in your application. " +
        "Instead, assign to `this.state` directly or define a `state = {};` " +
        "class property with the desired state in the %s component.",
      callerName,
      componentName
    );
    didWarnStateUpdateForUnmountedComponent[warningKey] = true;
  }
}
```

看来`ReactNoopUpdateQueue`是一个抽象类，实际的方法并不是在这里实现的，同时我们看下最初`updater`赋值的地方，初始化`Component`时，会传入实际的`updater`

```js
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}
```

我们在组件的构造方法当中将`this`进行打印

```js
class App extends Component {
  constructor(props) {
    super(props);
    //..省略

    console.log("constructor", this);
  }
}
```

![-w766](https://camo.githubusercontent.com/880e90113d658b5375215d152266d5c84ce73a40/687474703a2f2f696d672e63646e2e6b74667465616d2e636f6d2f31353430313136323232383936342e6a7067)

方法指向的是，在`react-dom.development.js`的`classComponentUpdater`

```js
var classComponentUpdater = {
  // 是否渲染
  isMounted: isMounted,
  enqueueSetState: function(inst, payload, callback) {
    // inst 是fiber
    inst = inst._reactInternalFiber;
    // 获取时间
    var currentTime = requestCurrentTime();
    currentTime = computeExpirationForFiber(currentTime, inst);
    // 根据更新时间初始化一个标识对象
    var update = createUpdate(currentTime);
    update.payload = payload;
    void 0 !== callback && null !== callback && (update.callback = callback);
    // 排队更新 将更新任务加入队列当中
    enqueueUpdate(inst, update);
    //
    scheduleWork(inst, currentTime);
  }
  // ..省略
};
```

enqueueUpdate
就是将更新任务加入队列当中

```js
function enqueueUpdate(fiber, update) {
  var alternate = fiber.alternate;
  // 如果alternat为空并且更新队列为空则创建更新队列
  if (null === alternate) {
    var queue1 = fiber.updateQueue;
    var queue2 = null;
    null === queue1 &&
      (queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState));
  } else
    (queue1 = fiber.updateQueue),
      (queue2 = alternate.updateQueue),
      null === queue1
        ? null === queue2
          ? ((queue1 = fiber.updateQueue = createUpdateQueue(
              fiber.memoizedState
            )),
            (queue2 = alternate.updateQueue = createUpdateQueue(
              alternate.memoizedState
            )))
          : (queue1 = fiber.updateQueue = cloneUpdateQueue(queue2))
        : null === queue2 &&
          (queue2 = alternate.updateQueue = cloneUpdateQueue(queue1));
  null === queue2 || queue1 === queue2
    ? appendUpdateToQueue(queue1, update)
    : null === queue1.lastUpdate || null === queue2.lastUpdate
    ? (appendUpdateToQueue(queue1, update), appendUpdateToQueue(queue2, update))
    : (appendUpdateToQueue(queue1, update), (queue2.lastUpdate = update));
}
```

我们看 scheduleWork 下

```js
function scheduleWork(fiber, expirationTime) {
  // 获取根 node
  var root = scheduleWorkToRoot(fiber, expirationTime);
  null !== root &&
    (!isWorking &&
      0 !== nextRenderExpirationTime &&
      expirationTime < nextRenderExpirationTime &&
      ((interruptedBy = fiber), resetStack()),
    markPendingPriorityLevel(root, expirationTime),
    (isWorking && !isCommitting$1 && nextRoot === root) ||
      requestWork(root, root.expirationTime),
    nestedUpdateCount NESTED_UPDATE_LIMIT &&
      ((nestedUpdateCount = 0), reactProdInvariant("185")));
}
```

```js
function requestWork(root, expirationTime) {
  // 将需要渲染的root进行记录
  addRootToSchedule(root, expirationTime);
  if (isRendering) {
    // Prevent reentrancy. Remaining work will be scheduled at the end of
    // the currently rendering batch.
    return;
  }

  if (isBatchingUpdates) {
    // Flush work at the end of the batch.
    if (isUnbatchingUpdates) {
      // ...unless we're inside unbatchedUpdates, in which case we should
      // flush it now.
      nextFlushedRoot = root;
      nextFlushedExpirationTime = Sync;
      performWorkOnRoot(root, Sync, true);
    }
    // 执行到这边直接return，此时setState()这个过程已经结束
    return;
  }

  // TODO: Get rid of Sync and use current time?
  if (expirationTime === Sync) {
    performSyncWork();
  } else {
    scheduleCallbackWithExpirationTime(root, expirationTime);
  }
}
```

太过复杂，一些方法其实还没有看懂，但是根据断点可以把执行顺序先理一下，在`setState`之后会执行`performSyncWork`，随后是如下的一个执行顺序

performSyncWork =performWorkOnRoot =renderRoot =workLoop =performUnitOfWork =beginWork =applyDerivedStateFromProps

最终方法是执行

```js
function applyDerivedStateFromProps(
  workInProgress,
  ctor,
  getDerivedStateFromProps,
  nextProps
) {
  var prevState = workInProgress.memoizedState;
  {
    if (
      debugRenderPhaseSideEffects ||
      (debugRenderPhaseSideEffectsForStrictMode &&
        workInProgress.mode & StrictMode)
    ) {
      // Invoke the function an extra time to help detect side-effects.
      getDerivedStateFromProps(nextProps, prevState);
    }
  }
  // 获取改变的state
  var partialState = getDerivedStateFromProps(nextProps, prevState);
  {
    // 对一些错误格式进行警告
    warnOnUndefinedDerivedState(ctor, partialState);
  } // Merge the partial state and the previous state.
  // 判断getDerivedStateFromProps返回的格式是否为空，如果不为空则将由原的state和它的返回值合并
  var memoizedState =
    partialState === null || partialState === undefined
      ? prevState
      : _assign({}, prevState, partialState);
  // 设置state
  // 一旦更新队列为空，将派生状态保留在基础状态当中
  workInProgress.memoizedState = memoizedState; // Once the update queue is empty, persist the derived state onto the
  // base state.
  var updateQueue = workInProgress.updateQueue;

  if (updateQueue !== null && workInProgress.expirationTime === NoWork) {
    updateQueue.baseState = memoizedState;
  }
}
```

## Vue

vue 监听变量变化依靠的是`watch`，因此我们先从源码中看看，`watch`是在哪里触发的。

### Watch 触发条件

在`src/core/instance`中有`initState()`

`/core/instance/state.js`

在数据初始化时`initData()`，会将每 vue 的 data 注册到`objerserver`中

```js
function initData(vm: Component) {
  // ...省略部分代码

  // observe data
  observe(data, true /* asRootData */);
}
```

```js
/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */
export function observe(value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value) || value instanceof VNode) {
    return;
  }
  let ob: Observer | void;
  if (hasOwn(value, "__ob__") && value.__ob__ instanceof Observer) {
    ob = value.__ob__;
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    // 创建observer
    ob = new Observer(value);
  }
  if (asRootData && ob) {
    ob.vmCount++;
  }
  return ob;
}
```

来看下`observer`的构造方法，不管是 array 还是 obj，他们最终都会调用的是`this.walk()`

```js
constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      // 遍历array中的每个值，然后调用walk
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }
```

我们再来看下 walk 方法，walk 方法就是将 object 中的执行`defineReactive()`方法，而这个方法实际就是改写`set`和`get`方法

```js
/**
* Walk through each property and convert them into
* getter/setters. This method should only be called when
* value type is Object.
*/
walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
}
```

`/core/observer/index.js`

`defineReactive`方法最为核心，它将 set 和 get 方法改写，如果我们重新对变量进行赋值，那么会判断变量的新值是否等于旧值，如果不相等，则会触发`dep.notify()`从而回调 watch 中的方法。

```js
/**
 * Define a reactive property on an Object.
 */
export function defineReactive(
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  // dep当中存放的是watcher数组
  const dep = new Dep();

  const property = Object.getOwnPropertyDescriptor(obj, key);
  if (property && property.configurable === false) {
    return;
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get;
  const setter = property && property.set;
  if ((!getter || setter) && arguments.length === 2) {
    // 如果第三个值没有传。那么val就直接从obj中根据key的值获取
    val = obj[key];
  }

  let childOb = !shallow && observe(val);

  Object.defineProperty(obj, key, {
    enumerable: true,
    // 可设置值
    configurable: true,
    get: function reactiveGetter() {
      const value = getter ? getter.call(obj) : val;
      if (Dep.target) {
        // dep中生成个watcher
        dep.depend();
        if (childOb) {
          childOb.dep.depend();
          if (Array.isArray(value)) {
            dependArray(value);
          }
        }
      }
      return value;
    },
    // 重点看set方法
    set: function reactiveSetter(newVal) {
      // 获取变量原始值
      const value = getter ? getter.call(obj) : val;
      /* eslint-disable no-self-compare */
      // 进行重复值比较 如果相等直接return
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return;
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== "production" && customSetter) {
        // dev环境可以直接自定义set
        customSetter();
      }

      // 将新的值赋值
      if (setter) {
        setter.call(obj, newVal);
      } else {
        val = newVal;
      }
      childOb = !shallow && observe(newVal);
      // 触发watch事件
      // dep当中是一个wacher的数组
      // notify会执行wacher数组的update方法，update方法触发最终的watcher的run方法，触发watch回调
      dep.notify();
    }
  });
}
```

## 小程序

### 自定义 Watch

小程序的 data 本身是不支持 watch 的，但是我们可以自行添加，我们参照`Vue`的写法自己写一个。
`watcher.js`

```js
export function defineReactive (obj, key, callbackObj, val) {
  const property = Object.getOwnPropertyDescriptor(obj, key);
  console.log(property);

  const getter = property && property.get;
  const setter = property && property.set;

  val = obj[key]

  const callback = callbackObj[key];

  Object.defineProperty(obj, key, {
    enumerable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val

      return value
    },
    set: (newVal) ={
      console.log('start set');
      const value = getter ? getter.call(obj) : val

      if (typeof callback === 'function') {
        callback(newVal, val);
      }

      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      console.log('finish set', newVal);
    }
  });
}

export function watch(cxt, callbackObj) {
  const data = cxt.data
  for (const key in data) {
    console.log(key);
    defineReactive(data, key, callbackObj)
  }
}
```

### 使用

我们在执行 watch 回调前没有对新老赋值进行比较，原因是微信当中对 data 中的变量赋值，即使给引用变量赋值还是相同的值，也会因为引用地址不同，判断不相等。如果想对新老值进行比较就不能使用`===`，可以先对 obj 或者 array 转换为 json 字符串再比较。

```js
//index.js
//获取应用实例
const app = getApp();

import { watch } from "../../utils/watcher";

Page({
  data: {
    motto: "hello world",
    userInfo: {},
    hasUserInfo: false,
    canIUse: wx.canIUse("button.open-type.getUserInfo"),
    tableData: []
  },
  onLoad: function() {
    this.initWatcher();
  },
  initWatcher() {
    watch(this, {
      motto(newVal, oldVal) {
        console.log("newVal", newVal, "oldVal", oldVal);
      },

      userInfo(newVal, oldVal) {
        console.log("newVal", newVal, "oldVal", oldVal);
      },

      tableData(newVal, oldVal) {
        console.log("newVal", newVal, "oldVal", oldVal);
      }
    });
  },
  onClickChangeStringData() {
    this.setData({
      motto: "hello"
    });
  },
  onClickChangeObjData() {
    this.setData({
      userInfo: {
        name: "helo"
      }
    });
  },
  onClickChangeArrayDataA() {
    const tableData = [];
    this.setData({
      tableData
    });
  }
});
```

## 参考

- [如何阅读 React 源码](https://github.com/JesseZhao1990/blog/issues/132)
- [React 16.3 ~ React 16.5 一些比较重要的改动](http://lizimeow.cn/2018/09/20/React%2016.3%20~%20React%2016.5%20%E4%B8%80%E4%BA%9B%E6%AF%94%E8%BE%83%E9%87%8D%E8%A6%81%E7%9A%84%E6%94%B9%E5%8A%A8/)

## 广而告之

本文发布于[薄荷前端周刊](https://github.com/BooheeFE/weekly)，欢迎 Watch & Star ★，转载请注明出处。

### 欢迎讨论，点个赞再走吧 ｡◕‿◕｡ ～
