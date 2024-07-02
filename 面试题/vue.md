两种URL模式有什么区别，哈希模式，历史模式
   
## v-if和v-show的区别

1. 控制手段不同
- v-sho隐藏是为该元素添加css--diplay:none,dom元素依旧还在。v-if显示隐藏是直接将dom元素整个删除
2. 编译过程不同
- v-if切换有一个局部编译/卸载的过程，切换过程中合适地销毁和重建内部的事件监听和子组件。（所以碰到项目中操作数据，前端页面不生效的时候，可以用v-if去强制刷新）v-show只是简单的基于css切换
3. 编译条件不同
- v-if是真正的条件渲染，它会确保在切换过程中条件块内的时间监听器和子组件适当的被销毁和重建。只有渲染条件为假时，并不做操作，直到为真才渲染。
- v-show由false变为true时，v-show不会触发组件的生命周期钩子，而v-if会触发组件的生命周期钩子。会触发beforeCreated,created,beforeMount,mounted.由true变为false时，v-if会触发组件的beforeDestroy和destroyed钩子，而v-show不会触发组件的钩子。
4. 总结
- v-if与v-show都能控制dom元素在页面的显示
- v-if相比v-show开销更大（直接操作dom节点增加与删除）
- 如果需要非常频繁地切换，则使用v-show比较好
- 如果在运行时条件很少改变，则使用v-if较好

## Vue实例挂载的过程
1. 简化过程
- 在调用beforeCreate之前，数据初始化并未完成，像data、props这些属性无法访问到
- 到了create的时候，数据已经初始化完成，能够访问data、props这些属性，但是并没有完成dom的挂载，因此无法访问到dom元素
- 挂载方法是调用vm.$mount
- 初始化顺序 props methods data
- data定义的时候可选择函数形式或者对象形式
- 不要把根元素放在body或者html上
- 可以在对象定义template/render或者直接使用template、el表示元素选择器
- 最终都会解析成render函数，调用compileToFunctions方法，回想template解析成render函数
2. 将template文档解析步骤大致分为以下几步
- 将html文档片段解析成ast描述符
- 将ast描述符解析成字符串
- 生成render函数
**备注(抽象语法树AST，简称语法树，是源代码语法结构的一种抽象表示，它以树状的形式表现编程语言的语法结构，树上的每一个节点都表示源代码的一种结构)**
3. 结论
- new Vue的时候调用会调用的_init方法
- 定义 `$set` `$get` `$delete` ` $watch` 等方法
- 定义 `$on $off $emit $destroy`等事件
- 定义 `_updte $forceUpdate $destroy`等生命周期
- 调用$mount进行页面的挂载
- 挂载的时候主要是通过MountConponent方法
- 定义updateComponent更新函数
- 执行render函数生成虚拟dom
- _update将虚拟dom生成真实dom结构，并且渲染到页面中

## 说说你对生命周期的理解
- 1.生命周期是什么
  - 指的是vue实例从创建到销毁的过程
- 2 生命周期有哪些
  - beforeCreate 组件实例被创建之初
  - created 组件实例已经完全创建
  - beforeMount 组件挂载之前
  - mounted 组件挂载到实例上去之后
  - beforeUpdate 组件数据发生变化，更新之前
  - update 组件数据更新之后
  - beforeDestroy 组件实例销毁之前
  - destroyed 组件实例销毁之后
  - activated keep-alive组件被激活时
  - deactivated keep-alive组件停用是调用
  - errorCaptured 捕获一个来自子孙组件错误是被调用
- 3 具体分析
  - 3.1 beforeCreate -> created得过程初始化vue实例，进行数据观测
  - 3.2 created
    * 完成数据观测，属性与方法得运算，`watch` `event`事件回调得配置
    *  可调用methods中的方法，访问和修改data数据出发响应式渲染dom,可通过computed和watch完成数据计算
    *  此时vm.$el并没有被创建
  - 3.3 created => beforeMount
    *  判断是否存在el选项，若不存在则停止编译，知道调用vm.$mount(el)方法
    - 优先级：render > template > el
    - vm.el获取到的是挂载DOM的
  - 3.4 beforeMount -> mounted
    - 此阶段vm.el完成挂载，vm.$el生成的DOM替换了el选项所对应的DOM
  - 3.5 mounted
    - vm.el已完成DOM的挂载与渲染，此时打印vm.$el，发现之前的挂载点以及内容都被替换成了新的DOM
  - 3.6 beforeUpdate
    - 更新的数据必须是被渲染在模板上的el template render之一
    - 此时view层还未更新
    - 若在beforeUpdate中再次修改数据，不会再次出发更新方法
  - 3.7 updated
    - 完成view层的更新
    - 若在update中再次修改数据，会再次出发更新方法
  - 3.8 beforeDestroy
    - 实例被销毁前调用，此时实例属性和方法仍然可以访问
  - 3.9 destroyed
    - 完全销毁一个实例，可清理它与其它实例的连接，解绑它的全部指令及事件监听器
    - 并不能清除DOM，仅仅销毁实例
- 4 数据在created和mounted的区别
    created是在组件实例一旦创建完成的时候立即调用，这时候页面dom并未生成；mounted实在页面dom节点渲染完毕之后就立即执行。出发时机上，created是比mounted要更早的，两者的相同点是都能拿到对象的属性和方法。讨论这个问题的本质就是出发的时机，放在mounted中的请求有可能导致页面闪动，因为此时页面dom结构已经生成了，但如果在页面加载前完成请求，则不会出现此情况，建议对页面内容的改动放在created生命周期中
## v-if和v-for的优先级
1. 作用
  说一下这两个的作用
2. 优先级
  v-for的优先级比v-if高
3. 注意事项
  - 永远不要把这两个放在同一个元素上，会带来性能上的兰妃
  - 如果避免出现这种情况，则在外层嵌套template，在这一层进行v-if判断，然后在内部进行v-fo循环
  - 如果条件出现在循环内部，可通过计算属性computed提前过滤掉那些不需要显示的项
## SPA首屏加载速度慢的怎么解决
1. 什么是首屏加载
  - 首屏时间，值得是浏览器从响应用户输入网址地址，到首屏内容渲染完成的时间，此时整个网页不一定要全部渲染完成，但需要展示当前视窗所需要的内容
  - 可以用performance.timing计算出首屏时间
2. 加载慢的原因
  - 网络延时的问题
  - 资源文件体积是否过大
  - 资源是否重复发送请求去加载了
  - 加载脚本的时候，渲染内容堵塞了
3. 解决办法
   - 减少入口文件体积
  1: 常见的手段是路由懒加载
   - 静态资源本地缓存
  1：采用HTTP缓存，设置CACHE-control last-modified etag等响应头
  2：采用service worker离线缓存
  3：前端合理利用localstorage
   - UI框架按需加载
   - 图片资源的压缩
  1: 在线字体或者雪碧图，其实主要是为了减少http的请求压力
   - 组件重复打包
  1: minChunks
   - 开启GZip打包
   - 使用SSR 
## 为什么data属性是一个函数而不是一个对象
1. 实例和组件定义data的区别
  Vue实例的时候，data既可以是一个对象，也可以是一个函数，而组件的data必须是一个函数
2. 组件data定义函数与对象的区别
  - Vue组件可能会有多个实例，采用函数返回一个全新data形式，使每个实例对象的数据不会受到其他实例对象数据的污染
3. 原理分析
4. 结论
  - 根实例对象data可以是对象也可以是函数（根实例是单例），不会产生数据污染的情况 
  - 组件实例对象data必须是函数，目的是为了防止多个组件实例对象之间公用一个dat,产生数据污染。采用函数的心事，initData时会将其作为工程函数都返回群星的data对象
## Vue组件间通信方式有哪些
1. 组件间通信的概念
  - 都知道组件是vue最强大的功能之一，vue中每一个.vue文件我们都可以视之为一个组件。通信，指的是发送者通过某种媒体以某种格式来传递信息到收信者以达到某种目的。广义上，任何信息的交通都是通信，组件通信即通过某种方式来传递信息以达到某种目的。
2. 组件通信解决了什么问题
  - 每个组件之间都有独自的作用域，组件间的数据是无法共享的，但实际开发工作中我们常常需要让组件之间共享数据，这也是组件通信的目的。要让他们互相之间能进行通讯，这样才能构成一个有机的完整系统
3. 组件间通信的分类
  - 父子组件的通信
  - 兄弟组件的通信
  - 祖孙与后代之间的通信
  - 非关系组件之间的通信
4. 组件之间通信的方案
  - 通过props传递
  - 通过$emit触发自定义事件
  - 使用ref
  - EventBus
  - $parenr或$root
  - attrs与listeners
  - Provide与Inject
  - Vuex
5. 总结
  - 父子关系的组件数据传递选择 props  与 $emit进行传递，也可选择ref
  - 兄弟关系的组件数据传递可选择$bus，其次可以选择$parent进行传递
  - 祖先与后代组件数据传递可选择attrs与listeners或者 Provide与 Inject
  - 复杂关系的组件数据传递可以通过vuex存放共享的变量
## 说说你对双向绑定的理解

## 说说你对nextTick的理解
1. NextTick是什么
  - VUe在更新DOM时是异步执行。当数据发生变化，Vue将开启一个异步更新队列，视图需要等队列中所有数据变化完成之后，再统一进行更新   
2. 使用场景
  - 如果想要在修改数据后立刻得到更新后的DOM结构，可以使用vue.nextTick()
## mixin是什么
