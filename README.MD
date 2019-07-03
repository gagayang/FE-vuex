# vuex的源码分析

我们都知道vuex 是vue的全局状态管理机。有些status 我们可以再组件内部。但是比如一些公用的数据（用户信息、等）这些可能再
a组件使用、也可能在b组件使用。这个时候就要状态共享。

而且需要组件中可以编辑对数据进行操作。
这里就产生了全局状态管理机

他们都是 flux的单项数据流的具体实现。 

在react中，典型代表是redux。在vue中就是vuex

我们首先来熟悉下里面的结构和api

里面是由state、getter、mutation、action、module 等组成.

##如何使用
这里详细介绍下

###state
state 这个可以认为是整体的这个状态管理机、通过组件中store.state就可以拿到全部的状态

这里很多时候我们要使用state作为部分的计算属性来使用 这里就提供了mapState 来使用


     mapState({
        // 箭头函数可使代码更简练
        count: state => state.count,
        })
        
###getter        
getter 在我们读取state的时候有时候经常要和一些其他的state进行操作关联。

这个时候可能我们每一个组件里面写一遍感觉是浪费的行为。这个时候vuex 就给我们提供了getter方法。

让我们可以直接复用

    getters: {
        doneTodos: state => {
          return state.todos.filter(todo => todo.done)
        }
      }
      
 mapGetters辅助函数仅仅是将 store 中的 getter 映射到局部计算属性
      
###mutation

mutation是我们规定的唯一可以修改（操作）state数据的方法。

它一般由两部分组成 事件名和方法回掉

    mutations: {
      increment (state, payload) {
        const n = payload.n
        state.count += n
      }
    }
    
调用的时候

    store.commit('increment', {
      amount: 10
    }) 
    
###Action

action 是不能直接操作state的，它是直接commit mutation 它这样做的好处就是我们可以进行异步操作。

### module
如果使用单一的state 这样可能会导致比较庞大。而且不利于理解。这个时候就出现了module。通过分割我们可以更明确项目信息



## 源码分析
首先我们看到其实我们使用的都是编译过后的。在package.json中的mian其实里面是/dist/index.js

这个是编译过后的文件。

真正的入口文件是 src/index.js

       这里其实就是引入了其他位置的东西主要是由
       
        Store
       
        install
       
        mapState,
        
        mapMutations,
        
        mapGetters,
        
        mapActions,
        
        这里index 是export 这些
       
这里就要看vue 怎么用了。

vue使用vuex的方式如下

        Vue.use(store)       

这里就是vue的一种install 机制这里就不多做介绍了。只要记住vue的use 就是调用这里面的install 就行了。

    var store = new vue.store({
    
    })
    
    new Vue({
        el: '#app',
        router,
        store,
        i18n,
        template: '<App/>',
        components: { App }
    })


上面就是一个典型的vuex的使用过程。那么 我们就根据使用顺序来分析下源码

首先是install 方法
    
       这个方法是store的一个方法。但是里面主要是调用mixin里面的一个方法
       通过查看代码发现其实它就做了一件事情
       
       判断vue的版本
       1、如果是2.0以上的 就通过vue的mixin来整体混入一个生命周期函数 beforeCreate
       2、如果是1.0的老版本，那么就通过初始化的时候加入这个store。
       
 接下来我们看看生命周期函数干了啥
       
       看了下说白了就是把store 变成vue 实例的$store.看来是这样就可以通过 this.store来获取了。
       
  
 我们在看看store 干了啥
 
        这里就是初始化store 绑定了commit 和dispath方法。把store 绑定到了rootmodule上
        
        这里读取commit 和dispath源码就发现了 这里dispathc 就是有promise的部分。这里可以传入promise
        
        然后通过all 的then 进行下一步
        
        
        这里就可以看到写了很多注册 的方法 然后action mutation 都是提供一系列的机制进行注册。
        
        
 主要流程就这样。暂时先写到到这里。
 
 主要是1 通过install 在beforecreate 来获取store
    2 产生stroe 绑定到root上       
    
    如果有不对的地方 看到了再改正
                     