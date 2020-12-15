title: React diff 时间复杂度是如何从O(n^3)到O(n)
author: prerabale
tags:
  - React
categories:
  - 前端
date: 2020-02-28 00:13:00
---

## 背景


我们都知道 React 将 dom 树的 diff，从 O(n^3) 变成了 O(n)，带来了大幅的性能提升。
那么
编辑距离  
1973年，Burkhard和Keller提出的BK树有效地解决了这个问题

- [A Survey on Tree Edit Distance and Related Problems](https://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf)
- [一个基于树编辑距离及其相关问题的调查](https://raw.githubusercontent.com/migijs/migi/master/lib/%E4%B8%80%E4%B8%AA%E5%9F%BA%E4%BA%8E%E6%A0%91%E7%BC%96%E8%BE%91%E8%B7%9D%E7%A6%BB%E5%8F%8A%E5%85%B6%E7%9B%B8%E5%85%B3%E9%97%AE%E9%A2%98%E7%9A%84%E8%B0%83%E6%9F%A5.pdf)
- [react的diff 从O(n^3)到 O(n) ，请问 O(n^3) 和O(n) 是怎么算出来？](https://www.zhihu.com/question/66851503)