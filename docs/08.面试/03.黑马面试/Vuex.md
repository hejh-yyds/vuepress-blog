## Vuex知识点

### 1. vuex

**概念**

- 状态管理

**工作流程**

- 视图更新（事件触发）-->dispatch派发任务--->actions 异步请求 ----> commit提交修改（mutation）---->修改state ----->重新渲染视图

**组成**

- state
- Getter:计算属性，依赖的state发生改变，重新计算
- Mutation：同步修改
- Action：异步请求
- Module：模块化vuex



### 2. 模块化

**主模块**

- store.js

```js
import Vue from 'vue'
import Vuex from 'Vuex'
Vue.use(vuex)
export default new Vuex.Store(
	{
    	state:{},
        getters:{},
        actions:{},
        mutations:{},
        modules:{
            
        }
    }
)
```



**Vue入口**

- main.js

```js
import Vue from 'vue'
import store from './store.js'
import App from './app.vue'
new Vue({
    store,
    render:(h)=>h(App)
}).$mount("#app")
```



**其他模块**

- cart.js

```js
const state={
    count:0,
    cartList:[]
}

const getters={
    myTestList:(state)=>state.cartList
}

const actions={
    getCartListAsyn(context,payload){
        setTimeout(()=>{
            context.commit("getCartList",payload)
        },2000)
    }
}

const mutations={
    getCartList(state,payload){
        state.cartList=payload
    }
}

export default {
    
    state,
    getters,
    actions,
    actions
}
```



**组件中进行使用**

```html
// 标签中进行使用
<div @click="$store.commit('cart/getCartList',[])" @click2="$store.dispatch('cart/getCartListAysn',[])">{{$store.state["cart/cartList"]}}</div>

```

**辅助函数使用**

```js
// 辅助函数使用

computed:{
	// 数组形式调用
	...mapState("cart",['count,cartList'])，
	// 对象形式调用
	...mapState("cart",{count:state=>state.count})
	// 参数1不启用
	...mapState(['cart/count'])
	// 对象形式
	...mapState({count:state=>state.cart.count})
}


methods:{
    // 使用模块参数
    ...mapMutations("cart",['getCartList']),
    ...mapMutations("cart",{get:'getCartList'})    
	
    // 不使用模块参数
    ...mapMutations(['cart/getCartList']),
    ...mapMutations({get:'cart/getCartList'})    
}
```



### 3. 开启严格模式

```js
new Vue.Store({
    strict:'true'
})
```

- 消耗性能，发布版本不开启





### 4.  模拟一个简单的vuex

```js
// 模拟自己的vuex

let _Vue = null

class Store {
  // 传递进来state，getters...
  constructor (options) {
    const { state = {}, getters = {}, mutations = {}, actions = {} } = options

    // 挂载到直接的实例上
    // 注意在vuex中的数据为响应式的
    this.state = _Vue.observable(state)

    this.getters = Object.create(null)
    // 注意到getters为一个函数
    // 我们把函数名获取出来，每次根据函数名来执行函数，得到返回值
    Object.keys(getters).forEach(key => {
      Object.defineProperty(this.getters, key, {
        get: () => getters[key](this.state)
      })
    })

    this._mutations = mutations
    this._actions = actions
  }

  // 实现commit方法和dispatch方法

  // state,payload
  commit (type, payload) {
    // 根据方法名执行函数
    this._mutations[type](this.state, payload)
  }

  // context,payload
  dispatch (type, payload) {
    this._actions[type](this, payload)
  }
}

function install (Vue) {
  _Vue = Vue
  // 在Vue实例被初始化成功后，把store这个选项挂载到Vue的原型上
  _Vue.mixin({
    beforeCreate () {
      if (this.$options.store) {
        Vue.prototype.$store = this.$options.store
      }
    }
  })
}

export default {
  Store,
  install
}

```

