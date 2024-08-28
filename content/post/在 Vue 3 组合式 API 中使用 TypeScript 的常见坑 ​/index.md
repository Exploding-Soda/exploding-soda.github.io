
+++
author = "Vue Developer"
title = "在 Vue 3 组合式 API 中使用 TypeScript 的坑"
date = "2024-08-28"
description = "分享在 Vue 3 的组合式 API 中使用 TypeScript 遇到的一些常见坑，以及如何避开这些坑的经验。"
tags = [
    "Vue 3",
    "JavaScript",
    "TypeScript",
    "组合式API",
]
categories = [
    "前端开发",
    "JavaScript 框架",
]
series = ["Vue 3 实践问题"]
aliases = ["vue3-composition-api-typescript-pitfalls"]
+++

Vue 3，TypeScript 的类型检查能在写代码的时候发现不少问题，但在实际使用中，难免会遇到一些坑。尤其是在 Vue 3 的组合式 API 中使用 TypeScript 时，有些地方一不留神就踩到雷了。这篇文章总结了一些我在开发中遇到的典型坑，顺便分享一下怎么绕过这些坑的经验。

## 坑一：ref 的类型推断

第一个坑就是 `ref` 的类型推断问题。按理说，`ref` 应该能够自动推断类型，但实际开发中却经常发现它不能完全满足需求。比如这个例子：

```typescript
import { ref } from 'vue';

const count = ref(0);  // 你可能觉得 count 应该是 number 类型
```

但问题来了，如果后面要重新赋值给 `count` 一个 `null`，TypeScript 会直接报错：

```typescript
count.value = null; // 报错：不能把 null 分配给 number 类型
```

解决方法也不复杂，就是在定义的时候显式指定类型，比如这样：

```typescript
const count = ref<number | null>(0);
```

通过显式声明类型，就可以避免后续赋值时报错。

## 坑二：reactive 对象的类型丢失

`reactive` 是用来创建响应式对象的，但是在 TypeScript 下有个坑，就是当对象嵌套时，类型可能会丢失。这是因为 `reactive` 返回的是一个 Proxy 对象，而 TypeScript 对 Proxy 的类型推断还不够完美。

```typescript
import { reactive } from 'vue';

const state = reactive({
  user: {
    name: 'Alice',
    age: 30
  }
});

// 访问 user 时，类型推断可能会失效
state.user.name = 123; // TypeScript 居然不报错
```

在这种情况下，可以使用 `interface` 或 `type` 显式声明类型，然后配合 `reactive` 使用：

```typescript
type User = {
  name: string;
  age: number;
};

const state = reactive<{
  user: User;
}>({
  user: {
    name: 'Alice',
    age: 30
  }
});
```

这样，类型就不会丢失了，TypeScript 也会帮忙检查类型是否正确。

## 坑三：组合式函数中的类型定义

写组合式函数时，定义输入输出的类型是个很好的习惯，但有时候类型定义会搞得人脑袋疼，尤其是在返回复杂的响应式对象时。

```typescript
import { ref } from 'vue';

function useUser() {
  const name = ref('Alice');
  const age = ref(30);
  
  return { name, age };
}

// 如果不显式定义，TypeScript 可能无法推断出返回对象的具体类型
```

解决方法是对返回值进行类型定义，这样可以让使用这个函数时的类型更加明确：

```typescript
function useUser(): { name: Ref<string>; age: Ref<number> } {
  const name = ref('Alice');
  const age = ref(30);
  
  return { name, age };
}
```

通过这种方式，TypeScript 就能够准确地知道 `name` 和 `age` 的类型了。

## 坑四：使用 watch 和 watchEffect 的类型问题

`watch` 和 `watchEffect` 是组合式 API 中很常用的两个函数，但它们的类型有时会让人迷惑。尤其是 `watch`，要传入的回调函数的类型和被监听的对象类型有时不容易搞清楚。

```typescript
import { ref, watch } from 'vue';

const count = ref(0);

watch(count, (newValue, oldValue) => {
  console.log(newValue, oldValue); // newValue 和 oldValue 的类型可能需要手动定义
});
```

为了让 TypeScript 能正确推断类型，建议在使用 `watch` 时显式指定类型：

```typescript
watch<number>(count, (newValue, oldValue) => {
  console.log(newValue, oldValue); // 现在 TypeScript 能正确识别类型了
});
```

## 总结

在 Vue 3 的组合式 API 中使用 TypeScript，虽然能带来强大的类型检查，但确实有不少需要注意的地方。遇到问题时，不妨试试显式定义类型，往往能事半功倍。希望这些踩坑的经验对大家有所帮助，少踩雷，多写 bug-free 的代码！
