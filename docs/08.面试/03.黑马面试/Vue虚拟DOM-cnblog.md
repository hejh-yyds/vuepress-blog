## Vue虚拟DOM



### 1. 虚拟dom

**什么是虚拟dom**

- 普通的JS对象描述DOM对象

**虚拟DOM为何出现**

- 手动操作DOM麻烦且减低性能
- 可以跟踪状态的改变更新视图

**虚拟DOM的作用**

- 维护视图与状态的关系
- 复杂视图下提升渲染性能



### 2. SnabbDom

#### 4.1 创建项目

在讲解`Snabbdom`的基本使用之前，我们先来创建一个项目。

打包工具为了方便使用，使用了`parcel`,你也可以使用`webpack`.

下面创建项目，并安装`parcel`

```js
//创建项目目录
md snabbdom-demo
// 进入项目目录
 cd snabbdom-demo
// 创建package.json
npm init -y
//本地安装parcel
npm install parcel-bundler
```

配置`package.json`中的`scripts`'

```js
 "srcipts":{
     "dev":"parcel index.html --open" , //open打开浏览器
     "build"："parcel build index.html"
 }
```

创建目录结构

```
index.html
package.json
---src
    basicusage.js
```

#### 4.2  导入Snabbdom

方法文档：

```
https://github.com/snabbdom/snabbdom
```

下面先安装`Snabbdom`.(这里最新版本有问题，可以先安装0.7.4)

```
npm install snabbdom@0.7.4
```

在项目的`js`文件夹下的`01- basicusage.js`文件中，添加如下代码：

```js
import  snabbdom  from "snabbdom";
console.log(snabbdom);

```

以上代码的意思就是导入`snabbdom`这个包，然后打印其内容。

项目的启动

```
npm run dev
```

这时，开启的端口号为`1234`

```
http://localhost:1234
```

在打开的浏览器中，查看控制台的输出，发现输出的内容为`undefined`

为什么输出的是`undefined`呢?

这里我们需要查看对应的源码，

在`node_modules/snabbdom/snabbdom.js`文件中，

我们可以看到在整个文件中，并没有使用`export default`的方式进行导出，所以就不能使用`import snabbdom`这种方式进行导入。同时在，源码中，我们可以看到导出了三项内容分别为`h`,`thunk`,`init`.

所以，导入的代码修改成如下的形式

```js
import { h, thunk, init } from "snabbdom";
console.log(h, thunk, init);

```

`Snabbdom`的核心仅提供最基本的功能，只导出了三个函数`init()`,`h( )`,`thunk( )`

`init`函数是高阶函数，返回`patch( )`，一会在看该方法

`h`函数返回虚拟节点`VNode`,这个函数我们在使用`Vue.js`的时候见过。

 `  h()`函数用于创建虚拟`DOM`，在`Snabbdom`中用`VNode`描述虚拟节点，也就是虚拟`DOM`。

```vue
new Vue({
router,
render:h=>h(App)
}).$mount('#app')
```

`thunk`函数是一种优化策略，可以在处理不可变数据时使用（用于优化复杂的视图）。



### 2.3 snabbdom基本使用

- 基本使用(basicusage.js)

```js
import  {h,thunk,init}  from "snabbdom";
// console.log(init,thunk,h);
// h函数用于构建一个虚拟dom，
// init函数为一个高阶函数,返回函数patch：新旧DOM比对并更新
// thunk 复杂视图的优化

let vnode=h("div#conatiner.box","hello world")

// 参数数组为模块数组，如class，style，eventListener 等6个模块
let patch=init([])

// 获取页面id为app的元素
let app=document.querySelector("#app")
// 替换更新
let newvNode=patch(app,vnode)


```

- 创建一个注释节点

```js
let a = patch(oldVnode, vnode);
  patch(a, h("!"));//第二个参数，表示创建一个注释节点。
```

- 创建子节点

```js
// 模拟请求更新dom，创建带子节点的vnode

import {init,h} from 'snabbdom'

let patch=init([])

// 第二个参数可以是数组，代表子节点数组
let vnode=h("div",[
    h("span","这是一个span标签"),
    h("p","这是一个p标签")
])

let app=document.querySelector("#app")
patch(app,vnode)

// 模拟请求
setTimeout(()=>{
    let newNode=h("div",[
        h("span","span标签"),
        h("p","p标签")
    ])

    patch(vnode,newNode)
},2000)
```



---

**模块使用**

**常用模块**

官方提供了6个模块

 `attributes:`设置`DOM`元素的属性，内部使用`setAttribute()来设置属性,`处理布尔类型的属性（可以对布尔类型的属性作相应的判断处理，布尔类型的属性，我们比较熟悉的有`selected`,`checked`等）。

**`props`:**  和`attributes`模块类似，设置`DOM`元素的属性`element[attr]=value`,不处理布尔类型的属性。

`class`: 切换样式类，注意：给元素设置类样式是通过`sel`选择器。··

`dataset`:设置` HTML5` 中的 `data-*` 的自定义属性

`eventlisteners`: 注册和移除事件

`style`:设置行内样式，支持动画（内部创建`transitionend`事件）,会增加额外的属性：delayed / remove / destroy

```js
// 使用snabbdom提供修改dom各种属性的模块

import {h,init} from 'snabbdom'

// 样式模块
import styleModule from 'snabbdom/modules/style'
// 事件处理模块
import eventListenersModule from 'snabbdom/modules/eventlisteners'


// 注册模块
let patch=init([styleModule,eventListenersModule])

let app=document.querySelector("#app")

let vnode=h("div#container",{
    style:{
        color:'red'
    },
    on:{
        'click':clickHandler
    }
},[
    h("span","p标签"),
    h("p","p标签")
])

patch(app,vnode)

function clickHandler(){
    console.log('hello world');
}
```





## 3. snabbdom的工作原理

**h函数**

- 输入：
  - 选择器
  - 属性配置，样式配置，事件等配置
  - 子节点
- 输出：vNode
  - vNode.sel 选择器
  - children 子节点数组
  - text:子节点为文本节点内容
  - key 唯一标识
  - elm dom元素

- 工作流程
  - 对输入进行判断，参数2，3是可选的，根据不同的参数输出不同的vNode,对于输入的子节点数组，都转换为vNode类型



**init函数**

- 输入
  - 模块配置数组
  - DOMApi
- 输出
  - patch函数
- 作用
  - 根据输入的模块配置数组，读取模块数组中每个模块的生命周期函数，用于在patch函数执行时机执行



**patch函数**

- 输入
  - 旧vNode|Dom元素
  - 新vNode
- 输出
  - 新vNode



- 工作流程
  - 把参数1转换为vNode,如果参数1不是vNode
  - 比较vNode1和vNode2是否相同，模糊比较：sel和key比较
    - 相同
      - 子节点递归
    - 不同
      - 找到父亲
      - 根据新vNode生成dom元素
      - 老vNode.elm的后面
      - 删除老vNode.elm



**createElm**

**输入**

- vnode

**输出**

- dom元素

**工作流程**

- 处理vNode的data数据(样式，属性等数据，执行样式，属性模块的回调函数处理)

- 虚拟节点转换为对应的dom元素
  - 对vNode.sel 处理，分离出各类选择器
  - 创建一个元素，根据上一步添加class，id等属性
  - 子节点递归上述操作



**patchVnode**

- 两个vNode进行sel和key比较相同时，调用

![](https://img2023.cnblogs.com/blog/3089561/202303/3089561-20230314200740259-1310789053.png)



**updateChildren**

- 新旧节点都有子节点，进行子节点的更新,diff算法



### 4. diff算法

- 新旧节点，vNode表示树形化，存储的时候是数组
- oldVnode：startIndex,endIndex
- newVnode：startIndex,endIndex
- 四种比较情况
  - oldVnode.startIndex：newVnode.startIndex
  - oldVnode.startIndex：newVnode.endIndex
  - oldVnode.endIndex：newVnode.startIndex
  - oldVnode.endIndex：newVnode.endIndex



```js
// 旧节点startIndex与新节点的startIndex相同(sameVnodes比较),patchVnodes
	新节点startIndex++
	旧节点startIndex++

// 不同的情况
	// 旧节点的endIndex与新节点的endIndex比较(sameVnodes比较),patchVnodes
		// 相同 
			新节点endIndex--
			旧节点endIndex--

        // 不同
	// 旧节点的startIndex 与 新节点的endIndex比较(sameVnodes比较)
		// 相同 patchVnodes,然后放到oldend序列的最后
			旧节点startIndex++
			新节点endIndex--


    // 旧节点endIndex 与 新节点startIndex 比较((sameVnodes比较))
		// 相同，	patchVnodes,然后放到oldstart序列的最前面
			旧节点endIndex--
			新节点startIndex++
```



**上述条件都不满足**

```js
// newStartIndex 不变

// 在oldVnodes数组中查找到与newStartIndex相同的
	// 没有找到：直接创建一个全新节点，插入到oldStartIndex节点的前面
		newStartIndex++
	// 找到了：patchVnodes(old,new),然后放在oldStartIndex的前面
		
	

```

