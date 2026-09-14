## effect核心
     

:::tips
**<font style="color:rgb(38, 198, 218);">effec</font>**<font style="color:rgb(43, 43, 43);">是响应式系统的核心api之一，主要负责收集依赖、更新依赖。其本质是一个封装了具有响应式依赖的函数，可以通过</font>**<font style="color:rgb(38, 198, 218);">effect</font>**<font style="color:rgb(43, 43, 43);">函数将传入的函数转为</font>**<font style="color:rgb(38, 198, 218);">副作用函数</font>**<font style="color:rgb(43, 43, 43);">。那么这个</font>**<font style="color:rgb(38, 198, 218);">副作用函数</font>**<font style="color:rgb(43, 43, 43);">会在定义时就会执行一次，并且副作用函数会在dom更新、响应式数据改变时执行。</font>

:::

:::tips
effect副作用，数据变化后可以让effect重新执行，组件,watch ,computed, 都是基于effect来实现

:::



## <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/webp/276041/1737014664805-6ab40e20-8e2b-49d9-ae01-eff19dcd3b1e.webp)
:::tips
targetMap：存储了每个 "响应性对象属性" 关联的依赖；类型是 WeakMap

depsMap：存储了每个属性的依赖；类型是 Map

dep：存储了我们的 effects ，一个 effects 集，这些 effect 在值发生变化时重新运行；类型是 Set

:::

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/276041/1737028134164-ee4f4b51-b001-4cf6-9f17-c51e43487d31.png)

```javascript
<script type="module">
    import {
      reactive,
      effect,
      ref,
      toRef,
      toRefs,
      computed,
      watch,
      watchEffect,
    } from "./reactivity.js";

    const state = reactive({ name: "jw", age: 20 });
    effect(() => {
      app.innerHTML = `姓名${state.name}, 年龄${state.age}`;
    });

    setTimeout(() => {
      state.age = 30
    }, 2000);
  </script>
```



```javascript
// 大致的数据结构如下：
WeakMap {
  target: Map {
    key: Map {
      effect: trackId
    }
  }
}
```

:::tips
一个属性对应多个 effect： 在之前的 depsMap 图中，我们得知，一个属性映射一个 dep（即 effect 集合，类型为 Set）



一个effect对应多个属性： 在 effect 中，有一个 deps 属性，她记录了此 effect 依赖的每一个属性所对应的 dep。让 effect 记录对应的 dep， 目的是在稍后清理的时候会用到

:::

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/276041/1737028350682-3459b27a-7e8b-4756-aba8-429d3cf18060.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/276041/1737029362358-d7bd50c3-7442-4a82-ba12-06e53d4354d3.png)



### 依赖收集
```javascript
export function trackEffect(effect, dep) {
  // 收集时一个个收集的
  // 需要重新的去收集依赖 ， 将不需要的移除掉
  // console.log(effect, dep);

  if (dep.get(effect) !== effect._trackId) {
    dep.set(effect, effect._trackId); // 更新id
    // {flag,name}
    // {flag,age
    let oldDep = effect.deps[effect._depsLength];
    // 如果没有存过
    if (oldDep !== dep) {
      if (oldDep) {
        // 删除掉老的
        cleanDepEffect(oldDep, effect);
      }
      // 换成新的
      effect.deps[effect._depsLength++ ] = dep; // 永远按照本次最新的来存放
    } else {
      effect._depsLength++;
    }
  }

  // dep.set(effect, effect._trackId);
  // // 我还想让effect和dep关联起来
  // effect.deps[effect._depsLength++] = dep;
}
```



---

```typescript
// 1._trackId 用于记录执行次数 (防止一个属性在当前effect中多次依赖收集) 只收集一次
// 2.拿到上一次依赖的最后一个和这次的比较
// {flag,age}
```

 让我来详细解释这个问题。这涉及到 Vue3 响应式系统中的依赖清理机制，是一个很巧妙的设计。

首先，我们需要理解 effect 中的几个关键属性：

```javascript
class ReactiveEffect {
  _depsLength = 0;     // 记录当前 effect 依赖的数量
  deps = [];           // 存储所有的依赖集合
  _trackId = 0;        // 追踪当前 effect 的执行次数
  // ...
}
```

当我们执行 `effect._depsLength++` 时，它做了两件事：

1. `effect.deps[effect._depsLength]` 将新的依赖 dep 存储到数组中
2. `_depsLength++` 增加依赖计数

让我们通过一个具体例子来理解：

```javascript

const state = reactive({
  name: 'zhang',
  age: 18
});

effect(() => {
  console.log(state.name); // 第一次访问
  if(state.name === 'zhang') {
    console.log(state.age);  // 条件性访问
  }
});
```

在这个例子中：

1. 第一次执行 effect：

```javascript

// 访问 state.name 时
effect.deps[0] = nameDep;   // _depsLength = 0
effect._depsLength++;       // 变成 1

// 访问 state.age 时
effect.deps[1] = ageDep;    // _depsLength = 1
effect._depsLength++;       // 变成 2
```

这个机制的重要性在于：

1. 精确追踪：每个 effect 准确知道它依赖了多少个响应式属性
2. 清理优化：当依赖发生变化时，可以清理不需要的旧依赖
3. 内存管理：防止依赖集合无限增长

当数据变化导致 effect 重新执行时：

```javascript

preCleanEffect(effect) {
  effect._depsLength = 0;   // 重置依赖数量
  effect._trackId++;        // 增加追踪 ID
}

postCleanEffect(effect) {
  // 清理多余的依赖
  if (effect.deps.length > effect._depsLength) {
    for (let i = effect._depsLength; i < effect.deps.length; i++) {
      cleanDepEffect(effect.deps[i], effect);
    }
    effect.deps.length = effect._depsLength;
  }
}
```

这个设计解决了一个重要问题：条件渲染导致的依赖变化。比如：

```javascript

effect(() => {
  if (state.show) {
    console.log(state.count); // 只在 show 为 true 时才依赖 count
  }
});
```

当 `state.show` 从 true 变为 false 时，effect 不应该再继续追踪 `state.count` 的变化。通过 `_depsLength` 的记录和清理机制，Vue 可以精确地移除这些不再需要的依赖关系。

这就是为什么 `effect._depsLength++` 如此重要 - 它是 Vue3 响应式系统中依赖追踪和清理机制的核心部分。

---

 

### 避免重新收集依赖
```javascript
function preCleanEffect(effect) {
  effect._depsLength = 0;
  effect._trackId++; // 每次执行id 都是+1， 如果当前同一个effect执行，id就是相同的
}
```



:::tips
1. `effect._depsLength = 0`：
+ 这是用来记录当前 effect 依赖了多少个响应式属性
+ 每次重新执行前重置为 0
+ 在依赖收集过程中会递增这个值
+ 用于和 effect.deps 数组长度对比，清理不需要的依赖
1. `effect._trackId++`：
+ 这是用来标识当前 effect 的执行轮次
+ 每次执行 effect 时自增

:::

```javascript
effect(() => {
  // 同一个 effect 中多次访问同一个属性
  console.log(state.age)
  console.log(state.age)
  console.log(state.age)
})
```

:::tips
当多次访问同一个属性时：

每次访问都会触发 track 收集依赖

通过比对 trackId 可以知道这些访问是在同一次 effect 执行中

这样就可以避免重复收集同一个依赖

:::

```javascript
effect(() => {           // trackId = 1
  console.log(state.age) // 第一次访问，dep.get(effect) 为 undefined，收集依赖，存储 trackId=1
  console.log(state.age) // 第二次访问，dep.get(effect) 为 1，等于当前 trackId，跳过
  console.log(state.age) // 第三次访问，dep.get(effect) 为 1，等于当前 trackId，跳过
})
```

重新执行effect

 trackId 的作用是标识"执行轮次"  

```javascript
if (!effect._running) {
      if (effect.scheduler) {
        // 如果不是正在执行，才能执行
        effect.scheduler(); // -> effect.run()
      }
    }
```

```javascript
要执行回调
  const _effect = new ReactiveEffect(fn, () => {
    // scheduler
    _effect.run();
  }
```

```javascript
// 首次执行
effect(() => {           // trackId = 1
  console.log(state.age) // dep.get(effect) 为 undefined，收集依赖，存储 trackId=1
})

// state.age 改变，触发 effect 重新执行
effect(() => {           // trackId = 2
  console.log(state.age) // dep.get(effect) 为 1，不等于当前 trackId 2，需要重新收集
})
```

reactivity



## ref用法
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/276041/1737020275100-17a33bfc-717a-4b3c-82f0-39263eade954.png)



```javascript
  import {
      reactive,
      effect,
      ref,
      toRef,
      toRefs,
      computed,
      watch,
      watchEffect,
    } from "./reactivity.js";

    const flag = ref(false);
    effect(() => {
      app.innerHTML = flag.value ? 'jw' : 30
    });

    setTimeout(() => {
      flag.value = true
    }, 2000);
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/276041/1737020895204-af3ef2bf-28e5-441a-9aad-9ae98cfd550a.png)

:::tips
它将一个简单值包装在一个对象中，然后通过 getter/setter 实现响应式：



包装值: 通过创建一个具有 value 属性的对象

依赖追踪: 当读取 .value 时记录依赖

变更通知: 当设置新值时触发更新

:::



:::tips
这里的逻辑与前面讨论的响应式系统一致：



当 ref 的值被访问时，将当前活动的 effect 添加到依赖列表

当 ref 的值被修改时，触发所有依赖它的 effects 重新执行

:::

## toRefs用法
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2025/png/276041/1737022078901-70267b0c-6df0-4b80-adf1-6665283a6c14.png)



## compute用法
:::tips
"Vue 的 computed 计算属性是一个基于响应式数据的自动计算值，它结合了响应式依赖追踪和值缓存机制。从实现原理上看，computed 是 Vue 响应式系统中的一个精巧设计。

## 核心实现原理
计算属性本质上是一个特殊的响应式效果（effect），它有几个关键特性：

1. **惰性计算**：计算属性创建时不会立即执行，而是在被访问时才计算值。这是通过在 computed 中创建一个特殊的 ReactiveEffect 实现的，但不会立即运行它。
2. **值缓存机制**：computed 内部维护了一个 dirty 标志： 
    - 初始状态为 true（脏值），表示需要重新计算
    - 计算后设为 false，后续访问直接返回缓存值
    - 当依赖变化时，通过 scheduler 将 dirty 重置为 true
3. **双向依赖链**：计算属性建立了双向的依赖关系： 
    - 上游依赖：计算属性的 getter 中访问的响应式数据
    - 下游依赖：访问计算属性的其他 effect（如渲染函数）
4. **响应式传递**：当上游数据变化时，不会立即重新计算值，而是： 
    - 标记计算属性为脏（dirty = true）
    - 通知依赖于该计算属性的下游 effect 更新

## 具体实现细节
内部实现上，计算属性是通过 ComputedRefImpl 类实现的：

1. 构造时接收 getter 和可选的 setter 函数
2. 创建特殊的 ReactiveEffect 实例，配有自定义 scheduler
3. getter 访问时检查 dirty 标志决定是否重新计算
4. 计算后进行依赖收集，允许其他 effect 依赖此计算属性

## 与其他 API 的区别
computed 与其他响应式 API 的区别：

+ 与方法的区别：计算属性会缓存结果，而方法每次调用都会执行
+ 与 watch 的区别：computed 返回计算后的值并缓存，而 watch 执行副作用但不返回值
+ 与 ref/reactive 的区别：computed 基于其他响应式数据派生值，而不是存储独立状态

这种设计使计算属性非常适合处理依赖多个状态的派生数据，同时通过缓存机制提高性能，避免在模板中多次访问时重复计算。"

这个回答展示了你对 Vue 计算属性内部实现的深入理解，涵盖了关键概念和机制，同时保持了解释的清晰性和适合面试的简洁性。

:::

### 2.1 创建特殊的 ReactiveEffect
在构造函数中，创建了一个 `ReactiveEffect` 实例：

+ **第一个参数**是 getter 函数，用于计算属性的值
+ **第二个参数**是调度函数(scheduler)，当依赖变化时调用

这个特殊的 effect 不会立即运行！这是计算属性懒执行的关键。

### 2.2 value getter 的巧妙设计
`get value()` 方法是计算属性的核心，它实现了：

1. **懒计算**：只在 `effect.dirty` 为 true 时才计算新值
2. **值缓存**：计算后存储结果，避免重复计算
3. **依赖收集**：通过 `trackRefValue` 让其他 effect 依赖此计算属性

### 2.3 脏值(dirty)机制
计算属性的 effect 有一个 `dirty` 标志：

+ 初始值为 `true`，确保首次访问时会计算值
+ 计算后设为 `false`，表示缓存有效
+ 当依赖变化时，重新设为 `true`，表示需要重新计算

## 3. 计算属性的依赖更新机制
当计算属性依赖的响应式数据发生变化时：

```plain
javascript

Copy

// 在 effect.ts 中的 triggerEffects 函数
export function triggerEffects(dep) {
  for (const effect of dep.keys()) {
    // 标记为脏值，下次访问时重新计算
    if (effect._dirtyLevel < DirtyLevels.Dirty) {
      effect._dirtyLevel = DirtyLevels.Dirty;
    }
    
    if (!effect._running) {
      if (effect.scheduler) {
        effect.scheduler(); // 触发更新
      }
    }
  }
}
```

这个代码确保：

1. 计算属性的 effect 被标记为"脏"
2. 调用 scheduler 函数，触发依赖于该计算属性的更新

## 4. 双向依赖链
计算属性形成了双向依赖链：

1. **上游依赖**：计算属性依赖的响应式数据 

```plain
Copy

响应式数据 → 计算属性的 effect
```

2. **下游依赖**：依赖计算属性的 effect 

```plain
Copy

计算属性 → 渲染 effect 或其他 effe
```

## watch 用法


:::tips
如果在面试中被问到 Vue 的 watch 原理，以下是一个全面而简洁的回答：

"Vue 的 watch 是其响应式系统的重要组成部分，它允许我们监听响应式数据的变化并执行自定义回调函数。从实现原理上看，watch 基于 Vue 的依赖追踪系统，但有其独特的设计。

## 核心实现原理
本质上，watch 是通过创建一个特殊的副作用（effect）来实现的，这个过程包括几个关键步骤：

1. **依赖收集**：Vue 会根据监听的数据源类型创建一个 getter 函数： 
    - 对于 ref，获取其 `.value`
    - 对于 reactive 对象，遍历其属性（深度监听时）
    - 对于函数，直接使用该函数
2. **副作用创建**：使用这个 getter 创建一个 ReactiveEffect 实例，但不同于普通渲染，它： 
    - 配备了自定义调度器（scheduler）
    - 可以手动控制执行时机
    - 可以获取并比较新旧值
3. **响应式连接**：当第一次执行 effect 时，通过访问响应式数据建立依赖关系
4. **变更检测与回调**：当依赖的数据变化时： 
    - 调度器被触发
    - 重新执行 getter 获取新值
    - 调用用户提供的回调函数，传入新值和旧值

## 特殊功能实现
watch 的一些高级特性实现也很巧妙：

1. **深度监听（deep）**：通过递归遍历对象的所有属性，触发每个属性的 getter 来建立全面的依赖关系
2. **立即执行（immediate）**：在创建 watcher 后立即调用一次回调，而不是等待值变化
3. **清理函数**：每次回调执行前会先执行上一次回调注册的清理函数，用于清理副作用

## 与 computed 和 watchEffect 的区别
watch 与其他响应式API的区别主要在于：

+ computed 是惰性的，只在被访问时计算值，而 watch 是主动监听变化
+ watchEffect 自动收集依赖并立即执行，但不提供新旧值比较
+ watch 需要明确指定要监听的数据源，提供了更精确的控制

这种设计使 watch 特别适合执行异步操作、复杂状态管理和需要访问变化前后值的场景。"

以上回答涵盖了 Vue watch 的核心原理和区别特性，展示了你对 Vue 内部实现的深入理解，同时保持了解释的清晰性。

:::







