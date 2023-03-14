## Vue响应式原理



### 1. 数据驱动

**数据驱动**

- 数据响应式：数据为普通的JavaScript对象，修改数据时视图会进行更新，避免了频繁的dom操作

- 双向绑定：数据改变，视图改变，视图改变，数据也会更新
- 发布订阅者模式和观察者模式





### 2. vue2的响应式

- 基于ES5的Object.defineProperty,实现数据的劫持

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div class="" id="app"></div>
    <script>
        // 定义data，需要修改的数据对象
        let data = {
            msg: 'hello'
        }

        // vue实例
        let vm = {}

        // 遍历data节点的属性，全部劫持
        Object.defineProperty(vm, "msg", {
            configurable: true,
            enumerable: true,

            get () {
                return data.msg
            },

            set (newValue) {
                if (data.msg === newValue) {
                    return
                }

                data.msg = newValue

                // 修改#appdom元素节点内容
                // textContent类似InnerHtml，可以获取display为none节点的内容，和script和style节点里面的内容
                document.querySelector("#app").textContent = data.msg
            }
        })

    </script>

</body>

</html>
```



- 实现多属性数据劫持

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div class="" id="app"></div>
    <script>
        // 定义data，需要修改的数据对象
        let data = {
            msg: 'hello',
            count: 0
        }

        // vue实例
        let vm = {}

        Object.keys(data).forEach(key => {
            // 遍历data节点的属性，全部劫持
            Object.defineProperty(vm, key, {
                configurable: true,
                enumerable: true,

                get () {
                    return data[key]
                },

                set (newValue) {
                    if (data[key] === newValue) {
                        return
                    }

                    data[key] = newValue

                    // 修改#appdom元素节点内容
                    // textContent类似InnerHtml，可以获取display为none节点的内容，和script和style节点里面的内容
                    document.querySelector("#app").textContent = data[key]
                }
            })
        })



    </script>

</body>

</html>
```



### 3. vue3的响应式原理

- 基于ES6的Proxy对象

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <div class="" id="app"></div>

    <script>
        // vue3通过Proxy创建data的代理对象实现数据劫持

        let data = {
            msg: 'hello'
        }

        let vm = new Proxy(data, {

            // target，即劫持的对象，key：属性名
            get (target, key) {
                return target[key]
            },
            // newValue 对代理对象进行修改时传入的新值
            set (target, key, newValue) {

                if (target[key] === newValue) {
                    return
                }

                target[key] = newValue

                // 修改dom元素的值
                document.querySelector("#app").textContent = target[key]
            }
        })
    </script>
</body>

</html>
```



### 4. 发布订阅者模式

**基本概念**

- 假定存在一个信号中心，当默认任务处理完，就向信号中心发布一个信号，其他任务可以订阅信号中心，当接收到信号，就可以知道自己何时开始任务

**应用场景**

- vue中的兄弟组件通讯



**发布订阅者模式简单实现**

```js
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <script>

        // 我们先看看vue的使用
        // let eventBus=new Vue()
        // eventBus.$emit('click')
        // eventBus.$on('click',()=>{

        // })

        // {'click':[fn1,fn2],'change':[fn]}

        // 定义的事件中心的类
        class EventEmitter {
            constructor() {
                // 事件到事件处理函数的映射
                this.subs = {}
            }

            $on (eventType, fn) {
                if (!this.subs[eventType]) {
                    this.subs[eventType] = []
                }

                this.subs[eventType].push(fn)
            }

            $emit (eventType) {
                // 找到事件对应的事件处理函数
                let handlers = this.subs[eventType]
                if (handlers) {
                    handlers.forEach(handler => {
                        handler()
                    })
                }
            }
        }

        let em = new EventEmitter()

        em.$on("click", () => {
            console.log('hello world');
        })

        em.$on("click", () => {
            console.log('hello world 123');
        })

        em.$emit("click")
    </script>
</body>

</html>
```



### 5. 观察者模式

- 与发布订阅模式相比，它没有数据中心，但是发布者可以感知到订阅者的存在

```js
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <script>
        // 实现发布订阅者模式

        // 发布者
        class Dep {
            constructor() {
                // 感知到的订阅者数组
                this.subs = []
            }

            addSub (sub) {
                // 判断订阅者是否合法
                if (sub && sub.update) {
                    this.subs.push(sub)
                }
            }

            // 通知所有的订阅者
            notify () {
                this.subs.forEach(sub => {
                    sub.update()
                })
            }
        }

        // 订阅者(观察者)
        class Watcher {

            update () {
                console.log('update something');
            }
        }

        let dep = new Dep()
        let watcher = new Watcher()

        dep.addSub(watcher)

        dep.notify()
    </script>
</body>

</html>
```



### 6. 模拟Vue的响应式

1. 将data对象定义在Vue的实例上，并且是响应式的
2. 将data对象通过observer将只身转换为响应式对象
3. observer监视数据变化，到compiler模块进行差值表达式等指令的解析
4. data自身被转换为响应式时
   1. 每个属性就是一个发布者 Dep
   2. 每个属性被get时，get者，也就是watcher，dep添加watcher
   3. 每个属性被set时，Dep通知watcher进行更新