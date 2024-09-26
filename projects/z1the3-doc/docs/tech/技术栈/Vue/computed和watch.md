# computed 和 watch

从这个函数名, 就可以看明白 initComputed, 这是初始化计算属性的函数. 它的就是遍历下我们定义的 computed 对象, 然后从中给每一个值定义一个 watcher 实例.

计算属性执行的时候会访问到, this.a 和 this.b. 这时候这两个值因为 Data 初始化的时候就被定义成响应式数据了. 它们内部会有一个 Dep 实例, Dep 实例就会把这个计算 computed watcher 放到自己的 sub 数组里. 待日后自己更新了, 就去通知数组内的 watcher 实例更新.

```js
const computedWatcherOptions = { lazy: true };

// vm: 组件实例 computed 组件内的 计算属性对象
function initComputed(vm: Component, computed: Object) {
  // 遍历所有的计算属性
  for (const key in computed) {
    // 用户定义的 computed
    const userDef = computed[key];
    const getter = typeof userDef === "function" ? userDef : userDef.get;

    watchers[key] = new Watcher( // 👈 这里
      vm,
      getter || noop,
      noop,
      computedWatcherOptions
    );

    defineComputed(vm, key, userDef); //对计算属性本身做响应式处理
  }
}
```

### dirty 的作用

他就是用来记录我们依赖的值有没有变, 如果变了就重新计算一下值, 如果没变, 那就**返回以前的值**. 就像一个懒加载的理念. 这也是计算属性缓存的一种方式

如果包含复杂性的计算，则这一步有机会回避掉

一开始 dirty 为 true, 一旦执行了一次计算,就会设置为 false. 然后当它定义的函数内部依赖的值比如: this.a 和 this.b 发声了变化. 这个值就会重新变为 true;

即
下一次执行计算属性时,就会去重新计算,(只在调用时决定是否重新计算，是同步的；而不是在依赖发生变化后，立即变动)

#### Computed 属性,watch 和组件一样, 本质上都是一个 watcher 实例

监听属性 watch 是异步触发的, 为什么这么说呢?

#### 实际上 watch 的执行逻辑和组件的渲染是一样的. 它们都会被放到一个 nextTick 函数中, 没错就是我们熟悉的 API.它可以让我们的同步逻辑, 放到下一个 Tick 在执行

#### computed 和 watch 对于新值与旧值一样的赋值操作, 都不会做任何变化. 但这点的实现是由响应式系统完成的

计算属性具有"懒计算"功能, 只有依赖的值变化了, 才允许重新计算. 称为"缓存", 个人觉得不准确

#### 在数据更新时, computed 的 dirty 状态会立即改变, 而 watch 与组件重新渲染, 至少都会在下一个"tick"执行

因此 watch 可以异步触发

computed 不可以

## 使用

### computed

接受一个 getter 函数，返回一个只读的响应式 ref 对象。该 ref 通过 .value 暴露 getter 函数的返回值。它也可以接受一个带有 get 和 set 函数的对象来创建一个可写的 ref 对象。

```js
const plusOne = computed(() => count.value + 1);

const plusOne = computed({
  get: () => count.value + 1,
  set: (val) => {
    count.value = val - 1;
  },
});
```

### watch

侦听一个 getter 函数：

```js
const state = reactive({ count: 0 });
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
);
```

侦听多个来源，自动变为数组

```js
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
});
```
