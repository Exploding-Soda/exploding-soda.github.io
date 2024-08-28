
+++
author = "React Developer"
title = "将 TypeScript 引入现有的 React 项目"
date = "2024-08-28"
description = "在这篇文章中，分享了将 TypeScript 引入到现有 React 项目的实际经验，包括迁移过程中遇到的挑战与解决方案。"
tags = [
    "React",
    "JavaScript",
    "TypeScript",
    "迁移",
]
categories = [
    "前端开发",
    "JavaScript 框架",
]
series = ["React 项目实践"]
aliases = ["react-introducing-typescript-migration"]
+++

开发过程中，之前一直使用 JavaScript 开发，但随着项目规模的扩大，发现有些问题光靠代码检查工具已经不够用了。于是，决定将 TypeScript 引入现有的 React 项目。这个过程虽然有些繁琐，但最终结果令人满意。在这里总结一下整个迁移的过程和一些踩过的坑。

## 为什么要迁移到 TypeScript？

在现有项目中，JavaScript 确实带来了很多灵活性，但随着代码量的增加，类型相关的错误也逐渐增多，特别是在处理复杂的数据结构和 API 时。引入 TypeScript 后，可以通过静态类型检查，在开发阶段就发现许多潜在的问题，而不是等到运行时才爆出错误。

## 迁移的准备工作

在开始迁移之前，先做好准备工作，确保过程尽可能顺利。

### 1. 安装 TypeScript 和相关依赖

首先，安装 TypeScript 及其相关的依赖：

```bash
npm install --save-dev typescript @types/react @types/react-dom
```

安装完成后，在项目根目录下创建 `tsconfig.json` 文件，用来配置 TypeScript 编译器的选项：

```bash
npx tsc --init
```

生成的 `tsconfig.json` 文件可以根据项目需求进行调整，以下是一些常见的配置项：

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
```

### 2. 将文件扩展名改为 .ts 和 .tsx

接下来，将现有的 `.js` 文件改为 `.ts`，React 组件的 `.jsx` 文件改为 `.tsx`。这一步可能会引发一些类型错误，这是正常现象，接下来会逐步解决这些问题。

## 处理类型错误

将文件扩展名修改后，TypeScript 可能会提示大量的类型错误，特别是对于那些之前没有显式类型定义的变量和函数。

### 1. 为 Props 和 State 定义类型

React 组件的 `Props` 和 `State` 是类型定义的重点。以下是一个简单的示例：

```tsx
interface MyComponentProps {
  title: string;
  count?: number;
}

const MyComponent: React.FC<MyComponentProps> = ({ title, count = 0 }) => (
  <div>
    <h1>{title}</h1>
    <p>{count}</p>
  </div>
);
```

通过定义 `Props` 接口，可以让 TypeScript 检查组件使用时是否传入了正确的参数类型。

### 2. 引入 @types/* 处理第三方库

对于使用的第三方库，如果 TypeScript 无法自动推断类型，可以通过 `@types/*` 包来获取类型定义。例如：

```bash
npm install --save-dev @types/lodash
```

这样，在代码中使用 `lodash` 时，TypeScript 就能够进行类型检查了。

### 3. 处理 any 和 unknown

在迁移过程中，为了快速解决类型错误，可能会倾向于使用 `any` 类型。但需要注意的是，`any` 会绕过类型检查，尽量避免滥用。在无法明确类型时，可以先使用 `unknown`，然后在需要的地方进行类型断言。

```tsx
function processData(data: unknown) {
  if (typeof data === 'string') {
    console.log(data.toUpperCase());
  }
}
```

## 逐步迁移的策略

不需要一次性将整个项目迁移到 TypeScript，可以通过以下策略逐步推进：

### 1. 从新功能开始使用 TypeScript

可以先从新开发的功能模块开始使用 TypeScript，逐步积累经验和信心，然后再逐步将旧代码迁移过来。

### 2. 分模块迁移现有代码

对于现有代码，可以按模块或功能区块逐步迁移，这样可以减少一次性迁移带来的风险和工作量。

### 3. 使用 ts-ignore 临时解决问题

在迁移过程中，如果遇到难以解决的类型问题，可以暂时使用 `// @ts-ignore` 来忽略特定行的类型检查，但这应该是最后的手段。

## 总结

将 TypeScript 引入现有的 React 项目是一项值得投资的工作。虽然在迁移过程中可能会遇到各种各样的类型问题，但通过合理的策略和逐步推进，最终可以显著提高代码的稳定性和可维护性。希望这些迁移经验能为有类似需求的开发者提供帮助。
