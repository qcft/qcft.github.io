---
layout: post
title: 读“在框架设计中寻求平衡”
date: 2019-12-30
categories: 阅读 
tags:
  - 框架设计
  - Vue
coauthor: mileOfSunshine
---


今年 JSConf.Asia 会议上，Vue.js 的作者 @尤雨溪 在关于 前端框架设计取舍 上作了分享。 分享主要从两个部分展开介绍： 职责范围（Scope）和 渲染机制（Render Mechanism）。

#### 职责范围

小的职责范围具有很好的灵活度，但关于构建抽象上要求开发者有自主学习能力。相反，大的职责范围致力于提供抽象概念，倘若抽象不适用，想改变时，却缺乏灵活度。

Vue 在职责范围这个问题的处理方式上，采取了渐进式。


![](https://p1.ssl.qhimg.com/t01fe27088aa20d3ded.png)


通过渐进的方式来选择特性，让更多的人专注于开发，而不是学习一堆在当前开发中可能不需要的概念。对于当前一些问题，开发者仍然能通过文档来获取一些解决方案。

这种渐进式一来让 Vue 也存在着大职责范围中的统一维护面问题；二来 Vue 生态也无法像小职责范围那样的多样化。
<!--more-->
#### 渲染机制

JSX 具有完整的 JavaScript 表现力，但由于渲染函数的动态特性使其难以优化。模板的约束允许编译器对你的意图作做预判，从而给它更多的空间执行优化；但因受限于模板语法，从而失去了 JavaScript 表达能力。

单纯取舍不能解决问题，Vue 在渲染机制方面选择了 VDOM 结合模板编译，在模板编译上入手优化，引入了SVELTE，只对动态节点作 VDOM Diff 算法，从而提高了运行速度。

![svelte-logo-horizontal.svg](https://p1.ssl.qhimg.com/t01352166f663a62c22.png)

接下来看看 SVELTE 是怎么处理的。

场景1：只有一个动态节点

![image](https://p3.ssl.qhimg.com/t01712c2274c39bc718.png)

当模板中只有一个动态节点时，整个节点结构是静态的、不会改变，我们只需直接更新 message 字符串这一个动态节点。

场景2：存在结构指令

![v-if Demo](https://p1.ssl.qhimg.com/t01514e25d142756025.png)

若存在结构指令 `v-if`，此时这个节点可能存在或可能不存在。处理办法是进行拆分，以 `v-if` 为界拆分成两个嵌套块。 

- `v-if` 外部：是一个静态节点结构，只有 `v-if` 这一个动态节点
- `v-if` 内部：也是一个静态节点结构，只有 `{{ message }}` 一个动态节点

唯一要处理的就是将每个块内的动态节点数组扁平化。

> 数组扁平化是指将一个多维数组变为一维数组

![v-for Demo](https://p5.ssl.qhimg.com/t01df369a48ee8b05df.png)

又或者存在的是结构指令 `v-for`，处理方式也是一样，将代码拆分成嵌套块...

![more v-if, v-for Demo](https://p5.ssl.qhimg.com/t01a85190317eae945e.png)

若模板中存在多个 `v-if`、`v-for`，亦是如此。以结构指令为界，将模板拆分成嵌套块。每个块内的节点结构是静态的，动态节点维护在一个单一扁平化数组中。

![Block Tree](https://p1.ssl.qhimg.com/t015cf5539238c56211.png)

![Block Tree Demo](https://p1.ssl.qhimg.com/t01dc325e2cba4ce2e2.png)

假设这个区块树存在上图的节点，那么得出的扁平化数组（结构指令 v-if, v-for 等本身也可以看出一个节点）为：

```js
B5: [node5, node6, node7]
B4: [node1, node2, node3, node4]
B3: [B5]
B2: [B4]
B1: [B2, B3]
```
如此一来，对于同一模板的Diff， vue2.x 和 vue3.x 在做法上就存在比较大的差异。vue2.x 会做一个完整的 Diff 操作，而 vue3.x 是比较一个单一扁平化数组中的文本是否发生变化。有实验表明，vue3.0 中使用的新的编译策略的速度比 vue2.x 快了6倍多。虽说是个实验数据，但或多或少，vue3 都是更快。

![实验数据](https://p1.ssl.qhimg.com/t015efdcb0ccb5fb4a3.png)

#### 总结

最后，我们回到本次分享中的话题。通过 @尤雨溪 的分享，我们能感受到他在框架设计中是如何寻求平衡的，这对我们今后的框架设计存在着许多借鉴意义。

如何获取一个最佳平衡点，我们都曾有过努力寻找最佳的方式来解决问题的经历，我想这就是一个寻求最佳平衡点的过程。这个最佳方式不仅能解决现有问题，并且与后期的发展也一致，在我看来，这应该就是最佳平衡点。


> 本文作者： mileOfSunshine
> 本文链接： https://qwebfe.github.io/2019/12/30/seeking-the-balance-in-framework-design/
> 版权声明：文章是原创作品。转载请注明出处！