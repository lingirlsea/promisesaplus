

**一个健全的，通用的 JavaScript Promise 开放标准，由开发者制定，供开发者参考**

一个 *promise* 代表了一个异步操作的最终结果。与 promise 交互的基本方法是使用它提供的 `then` 方法，该方法注册了不同的回调函数，用来接收 promise 的最终值，或者该 promise 没有完成的原因。

此规范详细描述了 then 方法的行为，提供了一个可互操作的基础，这个基础依赖于实现了所有 Promises/A+ 规范的 promise 来提供。因此，这个规范应当被认为非常稳定。尽管 Promises/A+ 组织可能不定期的修改此规范，并进行微小的向后兼容的更改以解决新发现的极端情况，但只有经过仔细考虑、讨论和测试后，我们才会整合大的或向后不兼容的更改。

从历史上看，Promises/A+ 阐明了早期 Promises/A 提案的行为条款，将其扩展到实际上的行为，并删减了不足或有问题的部分。

最后，核心 Promises/A+ 规范没有涉及如何创建，执行或拒绝 promises，而是选择专注于提供可操作的 `then` 方法。 后期的配套规范中可能涉及这些主题。


### 1、术语
1.1 “promise” 是具有 `then` 方法的对象或函数，其行为符合此规范。

1.2 “thenable” 是定义 `then` 方法的对象或函数。

1.3 “value” 是任意合法的JavaScript值（包括 undefined，thenable 或 promise）。

1.4 “exception” 是使用 `throw` 语句抛出的值。

1.5 “reason” 是一个值，用来表明 promise 被拒绝的原因。


### 2、要求
#### 2.1 Promise 状态
一个 promise 必须是以下三种状态之一：待处理, 已处理, 或已拒绝。

2.1.1 当 promise 处于待处理状态时：
- 2.1.1.1 可以转换至已处理或已拒绝状态。

2.1.2 当 promise 处于已处理状态时：
- 2.1.2.1 不可以转换至其他状态。
- 2.1.2.2 必须有一个值，该值不可改变。

2.1.3 当 promise 处于已拒绝状态时：
- 2.1.3.1 不可以转换至其他状态。
- 2.1.3.2 必须有一个原因，该原因不可改变。

这里, “不可改变” 意味者恒等 (即通过 === 判断), 而不意味者更深层次的不可变。
> *译者注：深层次的不可变应该是针对非基本类型值，只考虑引用地址相同*

#### 2.2 `then` 方法
promise 必须提供一个 `then` 方法来访问其当前值，或最终值，或被拒绝的理由。

promise 的 `then` 方法接受两个参数：
```
promise.then(onFulfilled, onRejected)  
```

2.2.1 `onFulfilled` 和 `onRejected` 都是可选的参数：
- 2.2.1.1 如果 `onFulfilled` 不是函数，它必须被忽略。

- 2.2.1.2 如果 `onRejected` 不是函数，它必须被忽略。

2.2.2 如果 `onFulfilled` 是函数：
- 2.2.2.1 当 promise 处于已处理状态时，该函数必须被调用并将 promise 的值作为第一个参数。

- 2.2.2.2 该函数一定不能在 promise 处于已处理状态之前调用。

- 2.2.2.3 该函数被调用次数不超过一次。

2.2.3 如果 `onRejected` 是函数：
- 2.2.3.1 当 promise 处于已拒绝状态时，该函数必须被调用并将 promise 拒绝的原因作为第一个参数。

- 2.2.3.2 该函数一定不能在 promise 处于已拒绝状态之前调用。

- 2.2.3.3 该函数被调用次数不超过一次。

2.2.4 在[执行上下文](https://es5.github.io/#x10.3)堆栈仅包含平台代码之前，不得调用 `onFulfilled` 或 `onRejected` 。<sup>注解 3.1</sup>

2.2.5 必须将 `onFulfilled` 和 `onRejected` 作为函数调用（即没有 `this` 值）。<sup>注解 3.2</sup>

2.2.6 `then` 可以在同一个 promise 上多次调用。
 - 2.2.6.1 如果/当 `promise` 处于已处理状态时，所有相应的 `onFulfilled` 回调必须按照它们对 `then` 的组织顺序依次调用。

 - 2.2.6.2 如果/当 `promise` 处于已拒绝状态时，所有相应的 `onRejected` 回调必须按照它们对 `then` 的组织顺序依次调用。

2.2.7 `then` 必须返回一个 promise。<sup>注解 3.3</sup>
```
promise2 = promise1.then(onFulfilled, onRejected);
```

- 2.2.7.1 如果 `onFulfilled` 或 `onRejected` 返回值 `x`，执行 Promise 解决步骤 `[[Resolve]](promise2, x)`。

- 2.2.7.2 如果 `onFulfilled` 或 `onRejected` 抛出异常 `e`, `promise2` 必须被拒绝，并把 `e` 作为被拒绝的原因。

- 2.2.7.3 如果 `onFulfilled` 不是函数，且 `promise1`处于已处理状态，则 `promise2` 也必须处于已处理状态，且拥有和 `promise1` 已处理状态相同的值。

- 2.2.7.4 如果 `onRejected` 不是函数，且 `promise1` 处于已拒绝状态，则 `promise2` 必须处于已拒绝状态，且拥有和 `promise1` 相同的拒绝原因。



#### 2.3 Promise 解决步骤

`promise 解决步骤` 是一个抽象操作，它将 promise 和 value 作为输入，我们将其表示为 `[[Resolve]](promise, x)`。如果 `x` 具有 thenable 特性，我们就假设 `x` 的行为至少有点像 promise，它将试图使 promise 接收 `x` 的状态。否则，它使用值 `x` 执行 `promise`。

对 thenables 的这种处理允许 promise 实现互操作，只要它们暴露符合 Promises/A+ 规范的 `then` 方法即可。它还允许基于 Promises/A+ 的实现，“同化”不一致但合理的 `then` 方法。

要运行 `[[Resolve]](promise, x)`，请执行以下步骤：

- 2.3.1 如果 `promise` 和 `x` 引用同一个对象，则以一个 TypeError 类型的值作为拒绝 `promise` 的原因。

- 2.3.2 如果 `x` 是一个 promise，接收其状态：<sup>注解 3.4</sup>
	- 2.3.2.1 如果 `x` 处于未处理状态，则 promise 必须保持未处理状态，直到 `x` 被处理或被拒绝。

	- 2.3.2.2 如果/当 `x` 处于已处理状态，用相同的值执行 promise。

	- 2.3.2.2 如果/当 `x` 处于已拒绝状态时，以同样的理由拒绝 promise。

- 2.3.3 否则，当 `x` 是对象或函数,
	- 2.3.3.1 用 `x.then` 代替 `then`。<sup>注解 3.5</sup>

	- 2.3.3.2 如果在获取属性 `x.then` 的过程中导致抛出异常 `e`，则拒绝 `promise` 并用 `e` 作为拒绝原因。

	- 2.3.3.3 如果 `then` 是函数，则用 `x` 作为 `this`，`resolvePromise` 作为第一个参数，`rejectPromise` 作为第二个参数，调用该函数，其中：
		- 2.3.3.3.1 如果/当使用值 `y` 调用 `resolvePromise` 时，运行 `[[Resolve]](promise, y)`。

		- 2.3.3.3.2 如果/当使用 `r` 作为原因调用 `rejectPromise` 时，用 `r` 拒绝 `promise`。

		- 2.3.3.3.3 如果同时调用 `resolvePromise` 和 `rejectPromise`，或者对同一个参数进行多次调用，则第一次调用优先，并且忽略任何后来的的调用。

		- 2.3.3.3.4 如果调用 `then` 时抛出异常 `e`，
			- 2.3.3.3.4.1 如果 `resolvePromise` 或 `rejectPromise` 已经被调用，忽略该异常。

			- 2.3.3.3.4.2 否则，拒绝 `promise` 并以 `e` 作为拒绝原因。

	- 2.3.3.4 如果 `then` 不是函数，用 `x` 去执行 `promise`。

- 2.3.4 如果 `x` 不是对象或函数，用 `x` 去执行 `promise`。

如果 promise 被一个 thenable 解决，且该 thenable 参与一个 thenable 循环链，那么 `[[Resolve]](promise, thenable)` 的递归性质最终导致 `[[Resolve]](promise, thenable)` 被再次调用，上述算法将导致无限递归。鼓励实现此类递归检测，并以一个 `TypeError` 类型的值作为原因拒绝 `promise`，但此类检测不是必须的。<sup>注3.6</sup>


### 3、注释

- 3.1 这里的“平台代码”意味着引擎，环境和 promise 实现代码。实际上，这个要求确保了 `onFulfilled` 和 `onRejected` 的异步执行，在事件循环结束后用新的堆栈调用 `then`。这可以使用诸如 `setTimeout` 或 `setImmediate` 之类的“宏任务”机制，或者使用诸如 `MutationObserver` 或 `process.nextTick` 之类的“微任务”机制来实现。由于 promise 实现被认为是平台代码，因此它本身可能包含一个任务调度队列或 “trampoline”，在任务调度队列或 “trampoline” 中，处理程序被调用。

- 3.2 也就是说，在严格模式下 `this` 的值是 `undefined`；在非严格模式下，`this` 的值将是全局对象。

- 3.3 实现中，可以允许 `promise2 === promise1`，前提是实现过程满足所有要求。每个实现都应该记录它是否可以产生 `promise2 === promise1` 以及在什么条件下可以。

- 3.4 一般来说，只有当 `x` 来自当前的实现时才会知道它是真正的 promise。该条款允许使用基于特定实现的方法，来接收符合已知要求的 promise 的状态。

- 3.5 这个步骤首先存储对 `x.then` 的引用，然后测试该引用，然后调用该引用，避免多次访问 `x.then` 属性。在面对属性访问器时，这些预防措施对于确保一致性非常重要，，因为属性访问器的值可能会在两次检索之间发生变化。

- 3.6 实现 *不* 应该对 thenable 调用链的深度设置任意限制，并假设超出该限制，递归将是无限的。只有真正的循环才应该导致 `TypeError`；如果一个无限循环链上的 thenable 具有明显不同，那么永远递归下去就是正确的行为。

<br>
<br>

**参考资料：**
<br>
<br>

[【翻译】Promises/A+ 规范](http://www.ituring.com.cn/article/66566)

[“macro-task” 与 “micro-task” 在 stackoverflow 上的相关解读](https://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)

[You-Dont-Know-JS 中关于Promise的详细解读](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch3.md)


<br>
<br>

---

<br>
<br>

**An open standard for sound, interoperable JavaScript promises—by implementers, for implementers.**  

A *promise* represents the eventual result of an asynchronous operation. The primary way of interacting with a promise is through its `then` method, which registers callbacks to receive either a promise’s eventual value or the reason why the promise cannot be fulfilled.

This specification details the behavior of the then method, providing an interoperable base which all Promises/A+ conformant promise implementations can be depended on to provide. As such, the specification should be considered very stable. Although the Promises/A+ organization may occasionally revise this specification with minor backward-compatible changes to address newly-discovered corner cases, we will integrate large or backward-incompatible changes only after careful consideration, discussion, and testing.

Historically, Promises/A+ clarifies the behavioral clauses of the earlier Promises/A proposal, extending it to cover *de facto* behaviors and omitting parts that are underspecified or problematic.

Finally, the core Promises/A+ specification does not deal with how to create, fulfill, or reject promises, choosing instead to focus on providing an interoperable `then` method. Future work in companion specifications may touch on these subjects.

### 1. Terminology
1.1 “promise” is an object or function with a `then` method whose behavior conforms to this specification.  

1.2 “thenable” is an object or function that defines a `then` method.

1.3 “value” is any legal JavaScript value (including undefined, a thenable, or a promise).

1.4 “exception” is a value that is thrown using the `throw` statement.

1.5 “reason” is a value that indicates why a promise was rejected.

### 2. Requirements

#### 2.1 Promise States
A promise must be in one of three states: pending, fulfilled, or rejected.

2.1.1 When pending, a promise:  
- 2.1.1.1 may transition to either the fulfilled or rejected state.  

2.1.2 When fulfilled, a promise:  
-	2.1.2.1 must not transition to any other state.

-	2.1.2.2 must have a value, which must not change.  

2.1.3 When rejected, a promise:  
-	2.1.3.1 must not transition to any other state.

-	2.1.3.2 must have a reason, which must not change.  

Here, “must not change” means immutable identity (i.e. ===), but does not imply deep immutability.

#### 2.2 The `then` Method
A promise must provide a `then` method to access its current or eventual value or reason.  

A promise’s `then` method accepts two arguments:
```
promise.then(onFulfilled, onRejected)  
```

2.2.1 Both `onFulfilled` and `onRejected` are optional arguments:
- 2.2.1.1 If `onFulfilled` is not a function, it must be ignored.

- 2.2.1.2 If `onRejected` is not a function, it must be ignored.

2.2.2 If `onFulfilled` is a function:
- 2.2.2.1 it must be called after promise is fulfilled, with promise’s value as its first argument.

- 2.2.2.2 it must not be called before promise is fulfilled.

- 2.2.2.3 it must not be called more than once.

2.2.3 If `onRejected` is a function:
- 2.2.3.1 it must be called after promise is rejected, with promise’s reason as its first argument.

- 2.2.3.2 it must not be called before promise is rejected.

- 2.2.3.3 it must not be called more than once.

2.2.4 `onFulfilled` or `onRejected` must not be called until the [execution context](https://es5.github.io/#x10.3) stack contains only platform code. [3.1]

2.2.5 `onFulfilled` and `onRejected` must be called as functions (i.e. with no `this` value). [3.2]

2.2.6 `then` may be called multiple times on the same promise.
- 2.2.6.1 If/when `promise` is fulfilled, all respective `onFulfilled` callbacks must execute in the order of their originating calls to `then`.

- 2.2.6.2 If/when `promise` is rejected, all respective `onRejected` callbacks must execute in the order of their originating calls to `then`.

2.2.7 `then` must return a promise. [3.3]
```
promise2 = promise1.then(onFulfilled, onRejected);
```

- 2.2.7.1 If either `onFulfilled` or `onRejected` returns a value `x`, run the Promise Resolution Procedure `[[Resolve]](promise2, x)`.

- 2.2.7.2 If either `onFulfilled` or `onRejected` throws an exception `e`, `promise2` must be rejected with `e` as the reason.

- 2.2.7.3 If `onFulfilled` is not a function and `promise1` is fulfilled, `promise2` must be fulfilled with the same value as `promise1`.

- 2.2.7.4 If `onRejected` is not a function and `promise1` is rejected, `promise2` must be rejected with the same reason as `promise1`.


#### 2.3 The Promise Resolution Procedure

The `promise resolution procedure` is an abstract operation taking as input a promise and a value, which we denote as `[[Resolve]](promise, x)`. If `x` is a thenable, it attempts to make promise adopt the state of `x`, under the assumption that `x` behaves at least somewhat like a promise. Otherwise, it fulfills `promise` with the value `x`.

This treatment of thenables allows promise implementations to interoperate, as long as they expose a Promises/A+-compliant `then` method. It also allows Promises/A+ implementations to “assimilate” nonconformant implementations with reasonable `then` methods.

To run `[[Resolve]](promise, x)`, perform the following steps:

- 2.3.1 If promise and x refer to the same object, reject promise with a TypeError as the reason.

- 2.3.2 If `x` is a promise, adopt its state: [3.4]
  - 2.3.2.1 If `x` is pending, `promise` must remain pending until `x` is fulfilled or rejected.

  - 2.3.2.2 If/when `x` is fulfilled, fulfill `promise` with the same value.

  - 2.3.2.2 If/when `x` is rejected, reject `promise` with the same reason.

- 2.3.3 Otherwise, if `x` is an object or function,

	- 2.3.3.1 Let `then` be `x.then`. [3.5]

	- 2.3.3.2 If retrieving the property `x.then` results in a thrown exception `e`, reject `promise` with `e` as the reason.

	- 2.3.3.3 If `then` is a function, call it with `x` as `this`, first argument `resolvePromise`, and second argument `rejectPromise`, where:

		- 2.3.3.3.1 If/when `resolvePromise` is called with a value `y`, run `[[Resolve]](promise, y)`.

		- 2.3.3.3.2 If/when `rejectPromise` is called with a reason `r`, reject `promise` with `r`.

		- 2.3.3.3.3 If both `resolvePromise` and `rejectPromise` are called, or multiple calls to the same argument are made, the first call takes precedence, and any further calls are ignored.

		- 2.3.3.3.4 If calling `then` throws an exception `e`,

			- 2.3.3.3.4.1 If `resolvePromise` or `rejectPromise` have been called, ignore it.

			- 2.3.3.3.4.2 Otherwise, reject `promise` with `e` as the reason.

	- 2.3.3.4 If `then` is not a function, fulfill `promise` with `x`.

- 2.3.4 If `x` is not an object or function, fulfill `promise` with `x`.

If a promise is resolved with a thenable that participates in a circular thenable chain, such that the recursive nature of `[[Resolve]](promise, thenable)` eventually causes `[[Resolve]](promise, thenable)` to be called again, following the above algorithm will lead to infinite recursion. Implementations are encouraged, but not required, to detect such recursion and reject `promise` with an informative `TypeError` as the reason. [3.6]

### 3. Notes

- 3.1 Here “platform code” means engine, environment, and promise implementation code. In practice, this requirement ensures that `onFulfilled` and `onRejected` execute asynchronously, after the event loop turn in which `then` is called, and with a fresh stack. This can be implemented with either a “macro-task” mechanism such as `setTimeout` or `setImmediate`, or with a “micro-task” mechanism such as `MutationObserver` or `process.nextTick`. Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or “trampoline” in which the handlers are called.

- 3.2 That is, in strict mode `this` will be `undefined` inside of them; in sloppy mode, it will be the global object.

- 3.3 Implementations may allow `promise2 === promise1`, provided the implementation meets all requirements. Each implementation should document whether it can produce `promise2 === promise1` and under what conditions.

- 3.4 Generally, it will only be known that `x` is a true promise if it comes from the current implementation. This clause allows the use of implementation-specific means to adopt the state of known-conformant promises.

- 3.5 This procedure of first storing a reference to `x.then`, then testing that reference, and then calling that reference, avoids multiple accesses to the `x.then` property. Such precautions are important for ensuring consistency in the face of an accessor property, whose value could change between retrievals.

- 3.6 Implementations should *not* set arbitrary limits on the depth of thenable chains, and assume that beyond that arbitrary limit the recursion will be infinite. Only true cycles should lead to a `TypeError`; if an infinite chain of distinct thenables is encountered, recursing forever is the correct behavior.

