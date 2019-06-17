## 目录
* [简介](#intro)
* [生命周期](#lifecircle)
* [组件](#components)  
  * [父向子传递数据](#c-dataFlow1)  
  * [子向传递数据](#c-dataFlow2)

<span id="intro"></span>
### 简介

**vue -- mvvm框架，数据驱动视图，单向数据流**

>当你把一个普通的 JavaScript 对象传入 Vue 实例作为 data 选项，Vue 将遍历此对象所有的属性，并使用 Object.defineProperty 把这些属性全部转为 getter/setter。Object.defineProperty 是 ES5 中一个无法 shim 的特性，这也就是 Vue 不支持 IE8 以及更低版本浏览器的原因

![官网数据响应原理图](https://cn.vuejs.org/images/data.png)

*上图是官网的响应式的原理图*
>这张图比较清晰地展示了整个流程，首先通过一次渲染操作触发Data的getter（这里保证只有视图中需要被用到的data才会触发getter）进行依赖收集，这时候其实Watcher与data可以看成一种被绑定的状态（实际上是data的闭包中有一个Deps订阅者，在修改的时候会通知所有的Watcher观察者），在data发生变化的时候会触发它的setter，setter通知Watcher，Watcher进行回调通知组件重新渲染的函数，之后根据diff算法来决定是否发生视图的更新。  
>
>Vue在初始化组件数据时，在生命周期的beforeCreate与created钩子函数之间实现了对data、props、computed、methods、events以及watch的处理。

<span id="lifecircle"></span>
### 生命周期

![官网生命周期图](https://cn.vuejs.org/images/lifecycle.png)

*生命周期的钩子：*

**beforeCreate:**  
在实例初始化之后，数据观测(data observer) 和 event/watcher 事件配置之前被调用。

**created:**  
实例已经创建完成之后被调用。在这一步，实例已完成以下的配置：数据观测(data observer)，属性和方法的运算， watch/event 事件回调。然而，挂载阶段还没开始，$el 属性目前不可见。

**beforeMount:**  
在挂载开始之前被调用: 主要是将template编译到render方法---render方法会生成VNode用于之后生命周期中的updated来比较VNode节点区别并更新真实DOM

**mounted:**  
已经将VNode挂载到真实DOM上时调用

**beforeUpdate:**  
数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。 你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。

**updated:**  
由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。
>当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。然而在大多数情况下，你应该避免在此期间更改状态，因为这可能会导致更新无限循环。  

**beforeDestroy:**  
实例销毁之前调用。在这一步，实例仍然完全可用

**destroyed:**  
Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。 该钩子在服务器端渲染期间不被调用。

<span id="components"></span>
### 组件
<span id="c-dataFlow1"></span>
##### 1.父向子传递数据
通过props来进行
```html
/** 子组件 child.vue **/
<template>
  <h3>{{ title }}</h3> <!-- 在这里使用父级组件传来的title -->
</tamplate>
<scrpit>
  export default {
    props: ['title'] // 这里定义使用子组件时，可通过对应字段传入数据
  }
</script>

/** 父组件 **/
<template>
  <Child title="父组件的数据" /> <!-- title就是子组件定义的props里的字段 -->
</tamplate>
<scrpit>
  import Child from 'child.vue'
  export default {
    components: {
      Child // 在这里引入子组件就可以在template使用
    }
  }
</script>
```
*备注：由于单向数据流的原则，理论上子组件不应该做某种操作来直接改变props传来的值，如果需要需要对props传来的值进行处理，可以使用[computed计算属性](https://cn.vuejs.org/v2/guide/computed.html#%E5%9F%BA%E7%A1%80%E4%BE%8B%E5%AD%90)*

<span id="c-dataFlow2"></span>
##### 2.子向父传递数据
通过vue的自定义事件来传递-分两部分实现  
*1) 在父组件中使用  **@事件名|v-on:事件名**形式来监听子组件*  
*2) 在子组件中使用 **this.$emit(事件名)** 来触发*
```html
/** 子组件 child.vue **/
<template>
  <button @click="handleClick">子组件按钮</button>
</tamplate>
<scrpit>
  export default {
    methods: {
      handleClick() {
        this.$emit('childMsg', '子组件消息')
      }
    }
  }
</script>

/** 父组件 **/
<template>
  <Child @childMsg="handleChildMsg" />
</tamplate>
<scrpit>
  import Child from 'child.vue'
  export default {
    components: {
      Child // 在这里引入子组件就可以在template使用
    },
    methods: {
      handleChildMsg(msg) {
        // 这个msg 就是子组件中this.$emit方法的第二个参数
        console.log(msg) // ---输出'子组件消息'
      }
    }
  }
</script>
```
