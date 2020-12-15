# 编写 3.x 单组件

项目搭好了，第一个要了解的肯定是组件的变化，由于这部分篇幅会非常大，所以会分成很多个小节，一部分一部分按照开发顺序来逐步了解。

btw: 出于对Vue3的尊敬，以及前端的发展趋势，我们这一次是打算直接使用 `TypeScript` 来编写组件，对ts不太熟悉的同学，建议先对ts有一定的了解，然后一边写一边加深印象。

## 组件的生命周期

写组件之前，需要先了解组件的生命周期，你才能够灵活的把控好每一处代码的执行结果达到你的预期。

### 升级变化

从 2.x 升级到 3.x，在保留对 2.x 的生命周期支持的同时，3.x 也带来了一定的调整。

:::tip
3.x 依然支持 2.x 的生命周期，但是不建议混搭使用，前期你可以继续使用 2.x 的生命周期，但还是**建议尽快熟悉并完全使用 3.x 的生命周期来编写你的组件**。
:::

生命周期的变化，可以直观的从下表了解：

2.x 生命周期|3.x 生命周期|执行时间说明
:-:|:-:|:-:
beforeCreate|setup|组件创建前执行
created|setup|组件创建后执行
beforeMount|onBeforeMount|组件挂载到节点上之前执行
mounted|onMounted|组件挂载完成后执行
beforeUpdate|onBeforeUpdate|组件更新之前执行
updated|onUpdated|组件更新完成之后执行
beforeDestroy|onBeforeUnmount|组件卸载之前执行
destroyed|onUnmounted|组件卸载完成后执行
errorCaptured|onErrorCaptured|当捕获一个来自子孙组件的异常时激活钩子函数

其中，在3.x，`setup` 的执行时机比 2.x 的 `beforeCreate` 和 `created` 还早，可以完全代替原来的这2个钩子函数。

另外，被包含在 `<keep-alive>` 中的组件，会多出两个生命周期钩子函数：

2.x 生命周期|3.x 生命周期|执行时间说明
:-:|:-:|:-:
activated|onActivated|被激活时执行
deactivated|onDeactivated|切换组件后，原组件消失前执行

### 使用 3.x 的生命周期

在3.x，**每个生命周期函数都要先导入才可以使用**，并且所有生命周期函数统一放在 `setup` 里运行。

如果你需要在达到2.x的 `beforeCreate` 和 `created` 目的的话，直接把函数执行在 `setup` 里即可。

比如：

```ts
import { defineComponent, onBeforeMount, onMounted } from 'vue'

export default defineComponent({
  setup () {

    console.log(1);
    
    onBeforeMount( () => {
      console.log(2);
    });
    
    onMounted( () => {
      console.log(3);
    });

    console.log(4);

    return {}
  }
})
```

最终将按照生命周期的顺序输出：

```js
// 1
// 4
// 2
// 3
```

## 组件的基本写法

btw：官网的例子片段挺多，使用 `JavaScript` 基本上没啥问题，故这里只讲述如何通过 `TypeScript` 来编写一个组件。

如果你是从 2.x 就开始写ts的话，应该知道在 2.x 的时候就已经有了 `extend` 和 `class component` 的基础写法；3.x在保留class写法的同时，还推出了 `defineComponent` + `composition api`的新写法。

加上视图部分又有 `template` 和 `tsx` 的写法、以及3.x对不同版本的生命周期兼容，累计下来，在Vue里写ts，至少有9种不同的组合方式（我的认知内，未有更多的尝试），堪比孔乙己的回字（甚至吊打回字……

我们先来回顾一下这些写法组合分别是什么，了解一下 3.x 最好使用哪种写法：

### 回顾 2.x

在2.x，为了更好的ts推导，用的最多的还是 `class component` 的写法。

适用版本|基本写法|视图写法
:-:|:-:|:-:
2.x|Vue.extend|template
2.x|class component|template
2.x|class component|tsx

### 了解 3.x

目前3.x从官方对版本升级的态度来看， `defineComponent` 就是为了解决之前2.x对ts推导不完善等问题而推出的，尤大也是更希望大家习惯 `defineComponent` 的使用。

适用版本|基本写法|视图写法|生命周期版本|官方是否推荐
:-:|:-:|:-:|:-:|:-:
3.x|class component|template|2.x|×
3.x|defineComponent|template|2.x|×
3.x|defineComponent|template|3.x|√
3.x|class component|tsx|2.x|×
3.x|defineComponent|tsx|2.x|×
3.x|defineComponent|tsx|3.x|√

btw: 我本来还想把每种写法都演示一遍，但写到这里，看到这么多种组合，我累了……

所以从接下来开始，都会以 `defineComponent` + `composition api` + `template` 的写法，并且按照 3.x 的生命周期来作为示范案例。

接下来，使用 `composition api` 来编写组件，先来实现一个最简单的 `Hello World!`。

:::warning
在 3.x ，只要你的数据要在 `template` 中使用，就必须在 `setup` 里return出来。

当然，只在函数中调用到，而不需要渲染到模板里的，则无需return。
:::

```vue
<template>
  <p class="msg">{{ msg }}</p>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  setup () {
    const msg = 'Hello World!';

    return {
      msg
    }
  }
})
</script>

<style lang="stylus" scoped>
.msg
  font-size 14px
</style>
```

和 2.x 一样，都是 `template` + `script` + `style` 三段式组合，上手非常简单。

`template` 和 2.x 可以说是完全一样（会有一些不同，比如 `router-link` 移除了 `tag` 属性等等，后面讲到了会说明）

`style` 则是根据你熟悉的预处理器或者原生css来写的，完全没有变化。

变化最大的就是 `script` 部分了。

## 响应式数据的变化

响应式数据是MVVM数据驱动编程的特色，相信大部分人当初入坑MVVM框架，都是因为响应式数据编程比传统的操作DOM要来得方便，而选择Vue，则是方便中的方便。

在 3.x，响应式数据当然还是得到了保留，但是对比 2.x 的写法， 3.x 的步伐迈的有点大。

先放上官方文档：[响应性 API | Vue.js](https://v3.cn.vuejs.org/api/reactivity-api)

:::tip
虽然官方文档做了一定的举例，但实际用起来还是会有一定的坑，比如可能你有些数据用着用着就失去了响应……这些情况不是bug，_(:з)∠)_而是你用的姿势不对……

相对来说官方文档并不会那么细致的去提及各种场景的用法，包括在 `TypeScript` 中的类型定义，所以本章节主要通过踩坑心得的思路来复盘一下这些响应式数据的使用。
:::

相对于 2.x 在 `data` 里定义后即可通过 `this.xxx` 来调用响应式数据，3.x 的生命周期里取消了Vue实例的 `this`，你要用到的比如 `ref` 、`reactive` 等响应式api，都必须通过导入才能使用。

## 响应式 API 之 ref

`ref` 是最常用的一个响应式api，它可以用来定义所有类型的数据，包括Node节点。

没错，在2.x常用的 `this.$refs.xxx` 来取代 `document.querySelector('.xxx')` 获取Node节点的方式，也是用这个api来取代。

### 类型声明

在开始使用api之前，要先了解一下在 `TypeScript` 中，`ref` 需要如何进行类型声明。

平时我们在定义变量的时候，都是这样给他们进行类型声明的：

```ts
// 单类型
const msg: string = 'Hello World!';

// 多类型
const phoneNumber: number | string = 13800138000;
```

但是在使用 `ref` 时，不能这样子声明，会报错，正确的声明方式应该是使用 `<>` 来包裹类型定义，紧跟在 `ref` API之后：

```ts
// 单类型
const msg = ref<string>('Hello World!');

// 多类型
const phoneNumber = ref<number | string>(13800138000);
```

### 变量的定义

了解了如何进行类型声明之后，对变量的定义就没什么问题了，前面说了它可以用来定义所有类型的数据，包括Node节点，但不同类型的值之间还是有少许差异和注意事项，具体可以参考如下。

#### 基本类型

对字符串、布尔值等基本类型的定义方式，比较简单：

```ts
// 字符串
const msg = ref<string>('Hello World!');

// 数值
const count = ref<number>(1);

// 布尔值
const isVip = ref<boolean>(false);
```

#### 引用类型

对于对象、数组等引用类型也适用，比如要定义一个对象：

```ts
// 声明对象的格式
interface Member {
  id: number,
  name: string
};

// 定义一个成员对象
const userInfo = ref<Member>({
  id: 1,
  name: 'Tom'
});
```

定义一个普通数组：

```ts
// 数字数组
const uids = ref<number[]>([ 1, 2, 3 ]);

// 字符串数组
const names = ref<string[]>([ 'Tom', 'Petter', 'Andy' ]);
```

定义一个对象数组：

```ts
// 声明对象的格式
interface Member {
  id: number,
  name: string
};

// 定义一个成员组
const memberList = ref<Member[]>([
  {
    id: 1,
    name: 'Tom'
  },
  {
    id: 2,
    name: 'Petter'
  }
]);
```

#### HTML DOM

对于 `2.x` 常用的 `this.$refs.xxx` 来获取Node节点信息，该api的使用方式也是同样：

模板部分依然是熟悉的用法，把ref挂到你要引用的DOM上。

```html
<template>
  <p ref="msg">
    留意该节点，有一个ref属性
  </p>
</template>
```

`script` 部分有三个最基本的注意事项：

:::tip
1. 定义挂载节点后，也是必须通过 `xxx.value` 才能正确操作到DOM（详见下方的[变量的读取与赋值](#变量的读取与赋值)）；

2. 请保证视图渲染完毕后再执行DOM的相关操作（需要放到 `onMounted` 或者 `nextTick` 里，这一点在 `2.x` 也是一样）；

3. 该变量必须 `return` 出去才可以给到 `template` 使用（这一点是 `3.x` 生命周期的硬性要求）。
:::

具体请看例子：

```ts
import { defineComponent, nextTick, ref } from 'vue'

export default defineComponent({
  setup () {
    // 定义挂载节点
    const msg = ref<HTMLElement>(null);

    // 请保证视图渲染完毕后再执行dom的操作
    nextTick( () => {
      console.log(msg.value.innerText);
    });

    // 必须return出去才可以给到template使用
    return {
      msg
    }
  }
})
```

### 变量的读取与赋值

被 `ref` 包裹的变量会全部变成对象，不管你定义的是什么类型的值，都会转化为一个ref对象，其中ref对象具有指向内部值的单个property `.value`。

:::tip
读取任何ref对象的值都必须通过 `xxx.value` 才可以正确获取到。
:::

请牢记上面这句话，初拥3.x的同学很多bug都是由于这个问题引起的（包括我……

对于普通变量的值，读取的时候直接读变量名即可：

```ts
// 读取一个字符串
const msg: string = 'Hello World!';
console.log('msg的值', msg);

// 读取一个数组
const uids: number[] = [ 1, 2, 3 ];
console.log('第二个uid', uids[1]);
```

对 ref对象的值的读取，切记！必须通过value！

```ts
// 读取一个字符串
const msg: string = 'Hello World!';
console.log('msg的值', msg.value);

// 读取一个数组
const uids = ref<number[]>([ 1, 2, 3 ]);
console.log('第二个uid', uids.value[1]);
```

普通变量都必须使用 `let` 才可以修改值，由于ref对象是个引用类型，所以可以在 `const` 定义的时候，直接通过 `.value` 来修改。

```ts
// 定义一个字符串变量
const msg = ref<string>('Hi!');

// 1s后修改它的值
setTimeout(() => {
  msg.value = 'Hello!'
}, 1000);
```

因此你在对接接口数据的时候，可以自由的使用 `forEach`、`map`、`filter` 等遍历函数来操作你的ref数组，或者直接重置它。

```ts
const data = ref<string[]>([]);

// 提取接口的数据
data.value = api.data.map( (item: any) => item.text );

// 重置数组
data.value = [];
```

问我为什么突然要说这个？因为涉及到下一部分的知识，关于 `reactive` 的。

## 响应式 API 之 reactive

`reactive` 是继 `ref` 之后最常用的一个响应式api了，相对于 `ref`，它的局限性在于只适合对象、数组。

:::tip
使用 `reactive` 的好处就是写法跟平时的对象、数组几乎一模一样，但它也带来了一些特殊注意点，请留意赋值部分的特殊说明。
:::

### 类型声明

待完善…

### 变量的定义

待完善…

### 变量的读取与赋值

待完善…

## 函数使用

待完善…

## 本节结语

这一节内容不多，但非常重要，组件的生命周期关系着你后续在开发过程中，数据获取和呈现之间的关系，以及什么功能应该放在哪个阶段调用，所谓磨刀不误砍柴工。