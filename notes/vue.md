### vue 优化
+ 一 代码书写习惯
    + 不需要做响应式的数据，不要放在data中（根据原理会调用getter/setter）
    + v-if 和 v-show 的合理使用
        - v-if （display:none/block） 
        - v-show （display: visiblity/hidden）
        - v-if 的切换开销更大
    + v-for 遍历必须为 item 添加 key，可以更快的定位到diff，避免使用 v-if，必要情况下可以使用 computed 属性
    + 能拆分的组件尽量拆分 颗粒度尽可能小
    + 设置缓存（http 缓存、service worker 离线缓存）
    + 使用vuex、结合路由钩子函数 beforeEnter 一起使用，这样组件还没有加载就能用 vuex 的数据了
    + keep-alive 可以实现组件的缓存，把组件的结构和数据都缓存到内存
    + 路由懒加载(2种)
        - component: Cart1 = () => import('./Cart1.vue')
        - component: Cart2 = (resolve) => require('./Cart2.vue', resolve)
    + 按需导入组件
    + Object.freeze 也能实现数据劫持，冻结
+ 二 加载
    1.图片资源懒加载 
    - npm i vue-lazyload -D
    - import VueLazyload from 'vue-lazyload'
    - 最外层static目录下的图片引用
        ```
        Vue.use(VueLazyload,{
                error:'/static/images/logo.png',//图片加载失败时候显示的图片
                loading:'/static/images/loading.gif'//图片还未加载完成时候的loading图片
        })
        ```
    或者
    - src下的assets目录下的图片
        ```
        Vue.use(VueLazyload,{
            error:require('./assets/images/logo.png'),
            loading:require('./assets/images/loading.gif')
        })
        <img  v-lazy="item.image" alt="img">
        ```
    2.第三方插件按需导入
+ 三 提升用户体验
    - 骨架屏
    - pwa
+ 四 SEO
    SSR和预渲染
+ 五 后端角度优化
    1.gzip 压缩
    2.服务器缓存和服务器缓存
    3.http 请求
    比如： 设置 no-cache Expirs Etags 等等
### vue 首屏加载优化
    1.把不常用的库放到 index.html 中，通过 cdn 引入
    2.vue 路由懒加载
    3.不生成 map 文件
    4.vue 组件尽量不要全局引入，可根据情况按需加载
    5.使用更轻量的工具库
    6.开启gzip压缩
    7.首页单独做服务端渲染


