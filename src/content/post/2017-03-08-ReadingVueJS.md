+++
categories = ["Linux"]
keywords = ["Linux"]
description = "Vue.js learning"
title = "Vue.js Learning"
date = "2017-03-08T09:25:54+08:00"

+++
### Question

```
What is ES6? ES6的语法?    
What is MVVM框架? 
```

### Tips
#### 介绍
Website: cn.vuejs.org    

三种组建组合到一个vue文件里:    

![/images/2017_03_08_09_35_42_887x580.jpg](/images/2017_03_08_09_35_42_887x580.jpg)

`.vue`是 vue.js中特有的一种文件，在打包中会被打包成浏览器可理解的html.    
#### 开发环境建立
Ubuntu为例:    

```
$ sudo apt-get install -y npm
$ sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
$ sudo cnpm install -g vue-cli
$ vue init webpack my-project
$ cd my-project
$ cnpm install
```
前端项目的依赖很多，所以需要用cnpm install所有依赖

-g 代表安装到系统目录下     
### Understanding Code
#### First example.
```
### Html
<div id="demo">
	<p>{{ message }}</p> //常用的模板渲染的方式, 此message对应的是
// 新建的Vue对象里的data里的mesage字段
	<input v-model="message"> //
Vue.js指令，将message字段关联到input的value里
</div>

### JS
var demo = new Vue({
el: "#demo", // element is demo
data: {
	message: 'Hello Vue.js!''
}	
})
```
两者相加可以得到一个页面渲染, 双向绑定的一个实例。    

双向绑定:在页面上进行的输入会绑定到JS输入里的message, Js
Message的变动会体现在调用message页面部分里，这就是所谓双向绑定。    
#### 重要选项
Vue的所有数据都是放在data里头的,data也是一个对象:    

```
new Vue({
  data: {
    a: 1, 
    b: []
  },
  methods: {
    doSomething: function() {
      this.a ++
      console.log(this.a)
    }
  },
  watch: {
    'a': function(val, oldVal) {
      console.log(val, oldVal)
     }
  }
})
this.a, this.b也可以取得
methods: vue的方法
<p>{{ a }}</p>
```
watch字面上理解就是一个监听，监听了a, 数据里的a, a对应的是一个方法，监听所有a
的变化，doSomething()方法执行的结果。   

#### 模版指令
数据渲染: v-text, v-html, {{}}    

```
<p>{{ a }}</p>
<p v-text="a"></p>
<p v-html="a"></p>
```
对应到Vue的data里的a    

控制模板隐藏: v-if, v-show    

```
<p v-if="isShow"></p>
<p v-show="isShow"></p>
new Vue({
  data: {
    isShow; true
  }
})
```
差别在于v-if是直接不显示元素，而v-show则是根据css的displayline来对它进行隐藏。代码中可以看到dom元素的

渲染循环列表: v-for:    

```
<ul>
<li v-for='item in items'>
	<p v-text='item.label'></p>
</li>
</ul>

data: {
items: [
{
label: 'apple'
},
{
label: 'banana'
}
]
},
```    
第一行是apple, 第二行是banana.   

事件绑定v-on:    

```
<button v-on:click="doThis"></button>
<button @click="doThis"></button> // 简写指令
methods: {
	dothis: function (someThing) {
	
	}
}
``` 
doThis是methods里的东西，而不是从data里取东西了，data针对固定的数据来操作的。    

属性绑定: v-bind:    

```
<img v-bind:src="imageSrc">

<div :class="{ red: isRed }"></div>
<div :class="[classA, classB]"></div>
<div :class="[classA, {classB: isB, classC: isC }]"></div>
```
注意简写方式.    

### Webpack框架
import就是ES6的语法.
