---
title: VUE-路由配置及跳转方式
date: 2020-10-14
categories: [VUE,router]    #分类
tags: [VUE,router]          #标签
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
typora-root-url: ..
---

### 需求

在vue中，页面的他跳转和卡片切换一般会用到路由router，具体不多说，主要记录一下路由的配置和几种常用的跳转方式



### 路由配置重定向 index.js ：

~~~js
import Home from '../components/Home'
import Page2 from '../components/Page2'
import Chil1 from '../components/Chil1'
import Chil2 from '../components/Chil2'

Vue.use(Router)
export default new Router({
routes: [
    {
      path: '/home',
      name: 'Home',	//可以不用name字段   但有时需要用到name匹配
      component: Home
    } ,
    {
      path: '/page2',
      name: 'Page2',
      component: Page2,
      children:[		//页内切换卡片
                {
                   path: '/chil1',	// 带‘/’ 表示根路径
                   name: 'Chil1',
                   component: Chil1
                },
                {
                    path: 'chil2',	// 不带‘/’ 表示访问路径为/page2/chil2
                    name: 'Chil2',
                    component: Chil2
                }
            ]     
    },
    {
      path: '/',	//匹配根路径 用于默认打开的页面
      name: 'index',
      redirect: '/home'  //redirect重定向到Home组件
    }
  ]  
}）
~~~

### 路由跳转的几种方式

**一、使用标签路由 router-link** 

~~~js
1、不传参

　   <router-link :to="{name:'Home'}"> 
　   <router-link :to="{path:'/home'}">

2、传参

　　<router-link :to="{name:'Home', params: {id:1}}">
　　<router-link :to="{path:'/home', params: {id:1}}"> 
　　// params传参数
　　// 路由配置 path: "/home/:id"
　　// 不配置path ,第一次可请求,刷新页面id会消失
　　// 配置path,刷新页面id会保留
　　// html 取参 $route.params.id
　　// script 取参 this.$route.params.id

　　<router-link :to="{name:'Home', query: {id:1}}">
　　// query传参数 (类似get,页面url后面会显示参数)
　　// 路由可不配置
　　// html 取参 $route.query.id
　　// script 取参 this.$route.query.id
~~~

**二、编程式路由 this.$router.push()**

~~~js
1、不传参
　　this.$router.push('/home')
　　this.$router.push({name:'Home'})
　　this.$router.push({path:'/home'})

2、传参
　　this.$router.push({name:'home',params: {id:'1'}})  // 只能用 name
　　// params传参数
　　// 路由配置 path: "/home/:id"
　　// 不配置path ,第一次可请求,刷新页面id会消失
　　// 配置path,刷新页面id会保留
　　// html 取参 $route.params.id
　　// script 取参 this.$route.params.id

   this.$router.push({name:'Home',query: {id:'1'}})
   this.$router.push({path:'/home',query: {id:'1'}})
　　// query传参数 (类似get,页面url后面会显示参数)
　　// 路由可不配置
　　// html 取参 $route.query.id
　　// script 取参 this.$route.query.id
~~~

>   在axios中使用 this.$router.push() 时，如果使用function会取不到this而报错，而要使用箭头函数
>
>   如下：

~~~js
onSubmit() {
      axios.post('http://localhost:8022/user/login',qs.stringify(this.user))
      .then((response)=>{	//正确；	使用function(response）{}会报错。。。我也不知为啥。。
        alert(response.data.msg);
        if (response.data.url){
          this.$router.push({path: '/home'})
        }
      })
      .catch((error) => alert(error))

    }
~~~

