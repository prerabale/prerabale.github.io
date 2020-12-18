title: 在 React 中用 ts 做路由跳转的检查
author: prerabale
tags:
  - Typescript
  - ''
  - React
categories:
  - 前端
date: 2020-12-18 14:14:00
---
### 为什么需要

我们访问页面的时候，需要知道目标页面的**地址**（`pathname`）和特定的**参数**（`search`、`hash`），然后拼装成完整的 `url` 才能够正常的进行访问。

如果访问了不存在的**地址**，或者设置了错误的**参数**都可能会导致页面无法访问或数据异常。

这些错误不进行一次真实的访问，光靠肉眼保障整个项目的路由没有问题想想就挺头疼的，要知道有些情况这些错误可能是不易察觉的。

例如：

1. 字符拼写错误
> 有位同事定义的路由里面包含了大写的 i: /pages/Iline，有些字体下，大写的 `I` 和小写的 `l` 是很难区分的。同理，参数也会有相同的问题

2. 需要参数不详
> 页面到底需要什么参数？哪些是可选的？哪些是必填的？填写的值是否有效？
> 如果页面之前开发的童鞋没有加上必要的注释说明，或者注释没有及时更新、遗漏了某些参数，又或者发起跳转的参数不符合要求。

3. 页面接收到的参数是 `undefined`、`null`
> 例如：/pages/a?id=undefined&name=null，这种情况是因为发起跳转的地方没有做好校验，`js` 在隐式类型转换的时候，把 `undefined` 和 `null` 转换成了对应的字符串拼接在了 `url` 上。


### 思路

原来在 `React Router` 里需要用

```typescript
history.push('/pages/a?id=1')
```

进行路由跳转。

这样的字符串进行校验是很复杂的，虽然现在 `ts` 也能对字符串的格式进行校验

例如：

```typescript
type hour =
  | '00'
  | '01'
  | '02'
  | '03'
  | '04'
  | '05'
  | '06'
  | '07'
  | '08'
  | '09'
  | '10'
  | '11'
  | '12'
  | '13'
  | '14'
  | '15'
  | '16'
  | '17'
  | '18'
  | '19'
  | '20'
  | '21'
  | '22'
  | '23'
  | '24'
  | '25'
  | '26'
  | '27'
  | '28'
  | '29'
  | '30'
  | '31'
  | '32'
  | '33'
  | '34'
  | '35'
  | '36'
  | '37'
  | '38'
  | '39'
  | '40'
  | '41'
  | '42'
  | '43'
  | '44'
  | '45'
  | '46'
  | '47'
  | '48'
  | '49'
  | '50'
  | '51'
  | '52'
  | '53'
  | '54'
  | '55'
  | '56'
  | '57'
  | '58'
  | '59'

type minute = hour

type time = `${hour}:${minute}`

// TS2322: Type '"60:71"' is not assignable to type '"00:00" | "00:01" | "00:02" | "00:03" | "00:04" | "00:05" | "00:06" | "00:07" | "00:08" | "00:09" | "00:10" | "00:11" | "00:12" | "00:13" | "00:14" | "00:15" | "00:16" | "00:17" | ... 3581 more ... | "59:59"'.
const t: time = '60:71'
```

但是这种校验方式比较适用于相对固定的格式，像 `url` 这种参数的顺序可以随意调换的场景，使用起来并不方便。

撇开实现难度不说，开发体验也极其糟糕，毕竟你总不能要求大家按照固定的格式书写参数的顺序吧。

```typescript
// 通过
history.push('/pages/a?id=1=&name=2')

// 不通过
history.push('/pages/a?name=2&id=1')
```

所以，我们得另辟蹊径

#### 第一步: 拆分 pathname 和参数

为了方便校验，这里我们把 `pathname` 和 `参数` 区分开来，统一封装一个 `navigateTo` 方法（底层还是调用的 `history.push`）。

```typescript
navigateTo({
  url: '/pages/a',
  query: {
	id: 1
  },
  state: {}
})

// 或者
navigateTo('/pages/a', query, state)
```

##### 第二步：校验 pathname 是否存在

首先想到可以利用 `ts` 中的 [联合类型](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#union-types) 和 [枚举](https://www.typescriptlang.org/docs/handbook/enums.html)来进行校验。

例如:

```typescript
// 枚举
enum PAGE_ENUM {
  PAGE_A = '/pages/a',
  PAGE_B = '/pages/b'
}

// TS2322: Type '"/s"' is not assignable to type 'PAGE_ENUM'.
const pageA: PAGE_ENUM = '/s'

// 联合类型
type pageType = '/pages/a' | '/pages/b'

// TS2322: Type '"/s"' is not assignable to type 'pageType'.
const pageB: pageType = '/s'
```
##### 第三步：校验参数类型

参数的校验思路就很明确了，无论是 `interface` 还是 `type` 的校验都可以。

给每个页面需要的参数定义一个 `interface`，调用 `navigateTo` 的时候校验一下参数格式就👌了。


### 实现

#### 导航方法封装

为了方便使用，`navigateTo` 我们设定为一个全局的方法。

因此我们无法使用 `this.props.history` 或者 `useHistory` 的方式使用 `history.push`(`history.replace` 同理)，所以我们需要先拿到 `history` 实例。

##### 获取 hisotry 实例

这里我们使用 `React Router` 文档中的[自定义 history](https://reactrouter.com/web/api/Router/history-object) 即可。

```typescript
// history.ts
import { createBrowserHistory } from "history";

export default createBrowserHistory();

// App.tsx
import React from "react";
import ReactDOM from "react-dom";
import history from "./history";

ReactDOM.render(<Router history={history} />, node);
```

##### 定义路由

这里我们实现两种常见的路由格式定义

第一种：Object 结构
```typescript
import React from 'react'

export const ROUTE_MAP = {
  '/pages/a': React.lazy(() => import('./pages/a'))，
  '/pages/b': React.lazy(() => import('./pages/b'))
}

const routes = Object.entries(ROUTE_MAP).map(([path, component]) => ({ path, component }))

export default routes
```

第二种：数组结构
```typescript
import React from 'react'

export const ROUTES = [
  {
    path: '/pages/a',
    component: React.lazy(() => import('./pages/a'))
  },
  {
    path: '/pages/b',
    component: React.lazy(() => import('./pages/b'))
  }
] as const

export type ROUTE_MAP = typeof ROUTES[number]['path']

export default ROUTES
```

##### navigateTo 实现

```typescript
import qs from 'querystring'

import history from './history'

export const navigateTo = (navigateInfo) => {
  const { url, query, state } = navigateInfo || {}
  
  history.push(`${url}?${qs.stringify(query)}`, state)
}
```

首先我们需要把 ``

[const-assertions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions) 语法，将所有的支持的 `pathname` 收集起来进行检查

例如:

```typescript
const pagePath = ['/pages/a', '/pages/b'] as const
type pageType = typeof pagePath[number]
const page: pageType = '/pages/c' // TS2322: Type '"/pages/c"' is not assignable to type '"/pages/a" | "/pages/b"'.
```