
+++
author = "Vue Developer"
title = "用 shallowRef 和 shallowReactive 优化 Vue 3 的性能"
date = "2024-08-28"
description = "探讨在 Vue 3 中使用 shallowRef 和 shallowReactive 优化性能的经验，避免不必要的响应式开销。"
tags = [
    "Vue 3",
    "JavaScript",
    "性能优化",
    "shallowRef",
    "shallowReactive",
]
categories = [
    "前端开发",
    "JavaScript 框架",
]
series = ["Vue 3 实践问题"]
aliases = ["vue3-shallowref-shallowreactive-performance"]
+++

在 Vue 3 项目中，响应式系统虽然强大，但也可能带来一些性能上的开销。尤其是在处理大型对象或数组时，如果每个属性都被深度地响应式化，性能就可能会受到影响。最近在项目中试用了 `shallowRef` 和 `shallowReactive`，效果不错，在这里分享一下这些经验。

## 为什么要用 shallowRef 和 shallowReactive？

Vue 3 默认会对对象的所有属性进行递归地响应式处理，这在大多数情况下没问题，但对于一些非常大的对象或复杂的数据结构，这种深度响应式就显得有些过度了。这不仅会增加内存的使用，还可能导致不必要的性能损耗。`shallowRef` 和 `shallowReactive` 就是为了解决这个问题而生的。

- **`shallowRef`**：只对 `ref` 引用的对象的第一层进行响应式处理，内部的嵌套对象不会被递归地转换为响应式。
- **`shallowReactive`**：类似地，`shallowReactive` 只对对象的第一层属性进行响应式处理，嵌套的对象不会被深度响应式化。

## 实战示例：用 shallowRef 管理大型对象

在项目中，需要管理一个包含大量数据的表格。最开始直接使用 `ref`，但随着表格数据的增多，发现渲染速度逐渐变慢。于是决定尝试使用 `shallowRef`。

### 使用 `ref` 的原始代码

```javascript
import { ref } from 'vue';

const tableData = ref({
  rows: [
    // 这里是大量的表格数据
  ],
  columns: [
    // 这里是表格的列配置
  ]
});
```

问题是，每次数据更新时，Vue 都会深度监听 `tableData` 对象中的所有属性和嵌套对象，导致性能瓶颈。

### 优化后的代码：使用 `shallowRef`

```javascript
import { shallowRef } from 'vue';

const tableData = shallowRef({
  rows: [
    // 这里是大量的表格数据
  ],
  columns: [
    // 这里是表格的列配置
  ]
});
```

通过 `shallowRef`，只对 `tableData` 的第一层进行响应式处理，避免了对深层嵌套属性的监听。这种优化明显减轻了性能开销，表格渲染速度得到了显著提升。

## 使用 shallowReactive 优化深层对象的处理

在另一个场景中，有一个包含复杂嵌套对象的配置项，原本使用 `reactive` 管理，但是随着配置的增多，性能开始变得不稳定。于是，考虑用 `shallowReactive` 来代替。

### 使用 `reactive` 的原始代码

```javascript
import { reactive } from 'vue';

const config = reactive({
  userSettings: {
    theme: 'dark',
    layout: 'grid',
    // 更多嵌套配置
  },
  systemSettings: {
    notifications: true,
    backups: false,
    // 更多系统配置
  }
});
```

这种方式下，每次修改配置时，Vue 都会递归地监听整个 `config` 对象，性能逐渐下降。

### 优化后的代码：使用 `shallowReactive`

```javascript
import { shallowReactive } from 'vue';

const config = shallowReactive({
  userSettings: {
    theme: 'dark',
    layout: 'grid',
    // 更多嵌套配置
  },
  systemSettings: {
    notifications: true,
    backups: false,
    // 更多系统配置
  }
});
```

使用 `shallowReactive` 后，只对 `config` 对象的第一层进行响应式处理，嵌套的配置项不会被深度监听，性能因此得到了明显改善。

## 需要注意的地方

在使用 `shallowRef` 和 `shallowReactive` 时，有几个点需要特别注意：

1. **响应性限制**：由于只是浅层响应式，所以内部嵌套的对象和属性不会自动更新，必须手动处理。如果确实需要深度响应性，还是得用 `ref` 或 `reactive`。

2. **适用场景**：这些浅层响应式工具适合用在大型数据结构或不需要频繁更新内部属性的场景。要根据实际需求权衡性能和响应性。

## 总结

通过 `shallowRef` 和 `shallowReactive`，可以在 Vue 3 中对性能进行细粒度的控制，避免不必要的性能开销。虽然它们不是万能的，但在特定场景下确实能带来显著的性能提升。希望这些经验能帮助开发中遇到类似问题的同学们，少走弯路，提升应用的性能。
