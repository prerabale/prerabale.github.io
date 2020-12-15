title: Monad

speaker: prerabale

url: /monad

transition: slide2

theme: moon

date: 2018-10-15 20:09:16

[slide]

## Monad

发音:  ['mɑnæd]  {:&.moveIn}  

[slide]

[magic data-transition="cover-circle"]

### Monad是什么?

---

如果你有在网上搜索"Monad"，你可能看到的是各种范畴学理论和一些专业的术语。  {:&.flexbox.vleft.moveIn}



例如： {:&.flexbox.vleft.moveIn}

* Haskell {:&.flexbox.vleft.moveIn}
* 单子
* Functor
* Functor Context
* Monad Context
* Applicative
* ...

[/magic]

[slide]

## 看完的感觉

这一大堆的什么东西，看不懂，头晕！！！ {:&.moveIn}

![](/monad/img/just-do-it.jpeg) {:&.moveIn}

[slide]

## Monad到底是什么?

* Monad 是一种组合函数的方式，它除了返回值以外，还需要一个 context。 {:&.flexbox.vleft.fadeIn}

* 一种设计模式，将一个运算过程拆分成多个互相链接的独立部分，只需要提供下一步运算所需要的函数，整个运算就会自动运行下去。

* 有一种更利于理解它的作用的叫法：`computation builder`。

[slide]

## 举个🌰 

有一个基本数值 {:&.moveIn}

![](/monad/img/1.png) {:&.moveIn}

[slide]

经过函数处理，然后输出结果

![](/monad/img/2.png) {:&.moveIn}

这个过程就是我们的函数处理 {:&.moveIn}

[slide]

我们不妨把这个处理的中间过程

比作一个管道 {:&.moveIn}

![](/monad/img/3.png) {:&.moveIn}

数据从管道上方进入，从下方输出 {:&.moveIn}

[slide]

当我们有多个过程的时候

我们只需要把管道串联起来就行了 {:&.moveIn}

![](/monad/img/4.png) {:&.moveIn}

[slide]



talk is cheap {:&.moveIn}

show me your code {:&.moveIn}

假设：我们有如下一个用户的列表 {:&.moveIn}

```js
const users = [
    {name: 'Eddard Stark', age: 17, sex: 'male', gift: false},
    {name: 'Catelyn Stark', age: 21, sex: 'female', gift: false},
    {name: 'Rickard Stark', age: 31, sex: 'male', gift: false}
]
```

 {:&.moveIn}

现在有3个需求 {:&.moveIn}

1. 客户英文不好，看大写字母吃力：把所有用户的名字转换为小写  {:&.flexbox.vleft.moveIn}
2. 新年在即，所有人又都大了一岁：把所有用户的年龄加上1
3. 还没有女朋友，好想做个舔狗啊：给所有女性送一份🎁

[slide]

## 常规写法

```js
const sendGift = (user) => user.sex === 'female' ? user.gift = true : null;
const ageAddOne = (user) => user.age += 1;
const lowerCase = (user) => user.name = user.name.toLowerCase();

users.forEach(user => {
  lowerCase(user);
  ageAddOne(user);
  sendGift(user);
})
```

 {:&.fadeIn}

三个处理函数是逐一调用的  {:&.fadeIn}

[slide]

## Monad 方式

```js
const sendGift = user => (user.sex === 'female' ? user.gift = true : null, user);
const ageAddOne = user => (user.age += 1, user);
const lowerCase = user => (user.name = user.name.toLowerCase(), user);

const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);                                                                                                                                   

const mix = compose(sendGift, ageAddOne, lowerCase);

users.forEach(mix);
```

把三个处理函数合并成了一个  {:&.fadeIn}

当然，本质上也还是逐一调用😂  {:&.fadeIn}

[slide]

## 这就是一个Monad？

---

是的 {:&.moveIn}

将所需要的操作组合起来，正是Monad要做的事情 {:&.moveIn}

但也不是 {:&.moveIn}

因为组合的方式很不Monad {:&.moveIn}

[slide]

让我们再看看刚才的例子

![](/monad/img/5.png) {:&.moveIn}

单纯的功能函数，其实是不需要返回这些数据的 {:&.moveIn}

就因为后续能够组合在一起，所以不得不做特殊处理 {:&.moveIn}

所以这并不是一个通用的手段 {:&.moveIn}

或者说并不能有效的结合多种功能函数  {:&.moveIn}

[slide]

## 存在的问题

* 不同的返回方式如何处理 {:&.moveIn}
* I/O 等异步请求的数据如何处理
* ...（以及我所不知道的其它问题）

需要一种方式能够统一规范处理 {:&.moveIn}

[slide]

## Monad 的做法

`type lift（类型提升）` {:&.flexbox.vleft.moveIn}

指的是将一个类型提升到对应的 context 中，值因此被赋予了对应 context 拥有的 API 用于计算，驱动 context 相关计算等等。类型提升可以描述为 a => F(a)。（Monad 也是一种 Functor，所以这里我们用了 F 表示 Monad）  {:&.flexbox.vleft.moveIn}

[slide]

`flatten（展平）` {:&.flexbox.vleft}

指的是去除值的 context 包裹。即: F(a) => a。 {:&.flexbox.vleft.moveIn}

`map（映射）` {:&.flexbox.vleft.moveIn}

map 指的是，“应用一个函数到 a，返回 b”。即给定某输入，返回某输出。 {:&.flexbox.vleft.moveIn}

[slide]

`context` {:&.flexbox.vleft}

 context 是一个 Monad 组合（包括 type lift，flatten 和 map）的计算细节。Functor/Monad 的 API 用到了 context，这些 API 允许你在应用的某些部分组合 Monad。Functor 及 Monad 的核心在于将 context 进行抽象，使我们在进行组合的时候不需要关注其中细节。在 context 内部进行 map 意味着你可以在 context 内部应用一个 map 函数完成 a => b，而新返回的 b 又被包裹了相同的 context。如果 a 的 context 是 Observable，那么 b 的 context 就也是 Observable，即 Observable(a) => Observable(b)。同理有，Array(a) => Array(b)。 {:&.flexbox.vleft.moveIn}

[slide]

## Monad 除此之外还需要了解的

* `组合函数`:  {:&.moveIn}

  compose(f, g)(x) = f(g(x))，注意顺序，靠后的参数是先执行的

* `Functor`: 

  functor就是一个函数，接收一个数值、一个 handler，拆开传入的原始值，得到其中的各个分解后的值，然后调用 handler 依次处理每一个上一步的到的数据，再将处理后的多个数据再封装成一个新的结构体，最后返回这个新的结构体。例如：`Array.map`

* `Fcuntor Context`: Array

[slide]

## ~~深入浅出~~由浅入深



简单的组合函数 {:&.moveIn}

```js
g:           a => b
f:                b => c
h = f(g(a)): a    =>   c
```

 {:&.moveIn}

因为函数的输入输出都有整齐划一的类型

只需要匹配输出类型 `b` 为 输入类型 `b` 即可

[magic]

![](/monad/img/4.png)

[/magic]

[slide]

Functor 的组合函数

```js
g:             F(a) => F(b)
f:                     F(b) => F(c)
h = f(g(Fa)):  F(a)    =>      F(c)
```

 {:&.moveIn}

这里的 F 可以理解为 Array.map，a 为 a 数组，b 为 b数组.... {:&.moveIn}

[magic]

![](/monad/img/6.png)

[/magic]

[slide]

但是如果你想要从 a => F(b)，b => F(c) 这样的形式进行函数组合

你就需要 Monad {:&.moveIn}

我们把 F() 换为 M() {:&.moveIn}

```js
g:                  a => M(b)
f:                       b => M(c)
h = composeM(f, g): a    =>   M(c)
```

 {:&.moveIn}

[slide]

```js
g:                  a => M(b)
f:                       b => M(c)
h = composeM(f, g): a    =>   M(c)
```

在这个例子中，管道中流通在函数之间的数据类型没有整齐划一。 {:&.moveIn}

函数 f 接收的输入是类型 b，但是上一步中，f 从 g 处拿到的类型却是 M(b)（装有 b 的 Monad） {:&.flexbox.vleft.moveIn}

[magic data-transition="move"]

![](/monad/img/7.png) {:&.moveIn} 

====

![](/monad/img/7.png)  ![](/monad/img/8.png)

[/magic]

[slide]

由于这一不对称性，composeM() 需要展开 g 输出的 M(b)，把获得的 b 传给 f，因为 f 想要的类型是 b 而不是 M(b)。 {:&.flexbox.vleft.moveIn}

这一过程（通常称为 .bind() 或者  .chain()） 就是 flatten 和 map 发生的地方 {:&.flexbox.vleft.moveIn}

[magic data-transition="move"]

![](/monad/img/flatten.png) {:&.moveIn}

====

![](/monad/img/flatten2.png)

[/magic]

~~正因为大多数函数都不是简单的 a => b 映射，因此 Monad 是需要的~~ {:&.flexbox.vleft.moveIn}

[slide]

## 再举一个🌰



假设：一个请求进来，我们要校验这个请求的用户是否有权限访问

* 你是谁？（我们首先得知道这个请求是谁发起的） {:&.flexbox.vleft.moveIn}
* 有没有权限？（18岁以下🚫禁止入内）

[magic data-transition="move"]

```js
getUserById(token: String) => Promise(User)

hasPermision(User) => Promise(Boolean)
```

 {:&.moveIn}

====

```js
g:                  a => M(b)
f:                       b => M(c)
h = composeM(f, g): a    =>   M(c)
```

====

```js
// a => Promise(b)
const getUserById = id => id === 3 ?
      Promise.resolve({ name: 'Kurt', role: 'Author' }) : undefined;
// b => Promise(c)
const hasPermission = ({ role }) => (
  Promise.resolve(role === 'Author')
);
// 尝试组合上面两个任务，注意：这个例子会失败
const authUser = compose(hasPermission, getUserById);
// 总是输出 false
authUser(3)
```

[/magic]

[slide]

当我们尝试组合 hasPermission() 和 getUserById() 为 authUser() 时 {:&.moveIn}

我们遇到了一个大问题 {:&.moveIn}

由于 hasPermission() 接收一个 User 对象作为输入 {:&.moveIn}

但却得到的是 Promise(User) {:&.moveIn}

为了解决这个问题，我们需要创建一个特别地组合函数 composePromises() 来替换掉原来的 compose() {:&.moveIn}

这个组合函数知道使用 .then() 去完成函数组合 {:&.moveIn}

```js
const composeM = chainMethod => (...ms) => (
  ms.reduce((f, g) => x => g(x)[chainMethod](f))
);
const composePromises = composeM('then');
```

 {:&.moveIn}

[slide]

于是我们得到了：

```js
const composeM = chainMethod => (...ms) => (
  ms.reduce((f, g) => x => g(x)[chainMethod](f))
);
const composePromises = composeM('then');
const label = 'API call composition';
// a => Promise(b)
const getUserById = id => id === 3 ?
      Promise.resolve({ name: 'Kurt', role: 'Author' }) :
undefined
;
// b => Promise(c)
const hasPermission = ({ role }) => (
  Promise.resolve(role === 'Author')
);
// 组合函数，这次大功告成了！
const authUser = composePromises(hasPermission, getUserById);
authUser(3); // true
```

[slide]

Monad 的实质：

- 函数 map： a => b {:&.flexbox.vleft.moveIn}
- 具有 Functor context 的 map： Functor(a) => Functor(b)
- 具备 Monad context，且需要 flatten 的 map：Monad(Monad(a)) => Monad(b)

---

在这个例子中，我们的 Monad 是 Promise，所以当我们组合这些返回 Promise 的函数时，对于 hasPermission() 函数，它得到的是 Promise(User) 而不是 Promise 中装有的 User 。注意到，如果你去除了 Monad(Monad(a)) 中外层 Monad() 的包裹，就剩下了 Monad(a) => Monad(b)，这就是 Functor 中的 .map()。如果我们再有某种手段能够展开 Monad(x) => x 的话，就走上正轨了。 {:&.flexbox.vleft.moveIn}

[slide]

所以 {:&.flexbox.vleft}

其实 Promise 也是 Monad {:&.flexbox.vleft.moveIn}

只是 {:&.flexbox.vleft.moveIn}

并不是严格意义上的 Monad {:&.flexbox.vleft.moveIn}

这是因为只有 Promise 包裹的值是 Promise 对象时.then(flatten) 才会去除外层 Promise 的包裹 {:&.flexbox.vleft.moveIn}

否则它会直接做 .map()，而不需要 flatten {:&.flexbox.vleft.moveIn}

* type lift: new Proimse() {:&.flexbox.vleft.moveIn}
* flatten: .then()

对于 Promise 对象，.then() 就用来替代 .chain()，但其实二者完成的是同一件事儿 {:&.flexbox.vleft.moveIn}

[slide]

Monads 必须满足三个定律（公理），合在一起称之为 Monad 定律：

* 左同一律： `unit(x).chain(f) ==== f(x)`（译注：将 `x` 提升到 Monad context 后，使用 `f()` 进行 map，等同于直接对 `x` 直接使用 `f` 进行 map）

* 右同一律： `m.chain(unit) ==== m`（译注：Monad 对象进行 map 操作的结果等于原对象 ）

* 结合律： `m.chain(f).chain(g) ==== m.chain(x => f(x).chain(g))`

[slide]

## 附录

结语：我们不生产水，只是大自然的搬运工。 {:&.moveIn}

* [Haskell](https://www.haskell.org/)(是一种标准化的，通用的纯函数编程语言，有非限定性语义和强静态类型) {:&.flexbox.vleft.moveIn}
* [图解 Monad](http://www.ruanyifeng.com/blog/2015/07/monad.html)
* [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
* [JS 函数式编程指南](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/)
* [what-is-a-monad](https://stackoverflow.com/questions/44965/what-is-a-monad)
* [[译]JavaScript 让 Monad 更简单（软件编写）](https://juejin.im/post/59e55dbbf265da43333d7652)
* [如何在 JS 代码中消灭 for 循环](https://zhuanlan.zhihu.com/p/51549583)
* [transducers-js](https://github.com/cognitect-labs/transducers-js)
