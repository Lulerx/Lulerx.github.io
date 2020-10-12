---
title: VUE使用axios请求后端数据
date: 2020-10-12
categories: [VUE,axios]    #分类
tags: [VUE,axios]          #标签
toc: true  #是否启用内容索引
top: false #是否置顶 true或者注释
typora-root-url: ..
---

### 引言

最近学习vue尝试使用axios请求后端数据，发现了一大堆问题，后端接收不到请求的数据。查了资料之后，在这里总结记录一下。

先说一下常用的几种请求方式和模板：

### Get方式

GET 方法传递参数格式如下：

~~~js
<script type = "text/javascript">
  mounted () {
    axios.get('http://localhost:8022/user/login', {
    params: {
      ID: 12345	//参数ID
    }
  })
  .then(function (response) {	//请求成功
    console.log(response);
    alert(response.data.msg);
  })
  .catch(function (error) {		//请求失败
    console.log(error);
  });
  }
})
</script>
~~~

### Post方式

POST 方法传递参数格式如下：

~~~js
<script type = "text/javascript">
  mounted () {
    axios.post('http://localhost:8022/user/login', {
    params: {
      ID: 12345	//参数ID
    }
  })
  .then(function (response) {	//请求成功
    console.log(response);
    alert(response.data.msg);
  })
  .catch(function (error) {		//请求失败
    console.log(error);
  });
  }
})
</script>
~~~

### axios API 方式

~~~js
<script type = "text/javascript">
  mounted () {
    //  可以将请求方式、URL、请求/响应格式、请求头等参数进行单独配置
	axios({		//GET 请求远程图片
  		method:'get',
 		url:'http://bit.ly/2mTM3nY',
  		responseType:'stream'
	})
  	.then(function(response) {
  		response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
	});
  }
})
</script>
~~~

### axios响应结构

axios请求的响应包含以下信息：

~~~json
{
  // `data` 由服务器提供的响应
  data: {},

  // `status`  HTTP 状态码
  status: 200,

  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: "OK",

  // `headers` 服务器响应的头
  headers: {},

  // `config` 是为请求提供的配置信息
  config: {}
}
~~~

使用 then 时，会接收下面这样的响应：

~~~js
axios.get("/user/12345")
  .then(function(response) {
    console.log(response.data);
    console.log(response.status);
    console.log(response.statusText);
    console.log(response.headers);
    console.log(response.config);
  });
~~~

### axios配置默认值

可以指定将被用在各个请求的配置默认值。

全局的 axios 默认值：

~~~js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
~~~

### post请求出现的错误

我们知道在做 post 请求的时候，我们的传参是 `data: {...}` 或者直接 `{...}` 的形式传递的。

在使用时，axios会帮我们 **转换请求数据和响应数据** 以及 **自动转换 JSON 数据**

而在使用post方式请求时，会出现后端接收不到数据的现象

**解决方案一**

【用 URLSearchParams 传递参数】

~~~js
let param = new URLSearchParams()
param.append('username', 'admin')
param.append('pwd', 'admin')
axios({
	method: 'post',
	url: '/api/lockServer/search',
	data: param
})
~~~

>   需要注意的是： `URLSearchParams` 不支持所有的浏览器，但是总体的支持情况还是 OK 的，所以优先推荐这种简单直接的解决方案

---



**解决方案二**

【使用qs库】

`npm install qs --save`

~~~js
import Qs from 'qs'		//引入qs
let data = {
	"username": "admin",
	"pwd": "admin"
}

axios({
	headers: {
		'deviceCode': 'A95ZEF1-47B5-AC90BF3'
	},
	method: 'post',
	url: '/api/lockServer/search',
	data: Qs.stringify(data)	//使用qs将参数转换为query参数
})

~~~

>   *网上有很多方案说使用*
>
>   axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
>
>   但是我用的时候不需要，没请求成功的可以试试

---



**解决方案三**

可以通过修改 `transformRequest` 来达到我们的目的

~~~js
import Qs from 'qs'
axios({
	url: '/api/lockServer/search',
	method: 'post',
	transformRequest: [function (data) {
	    // 对 data 进行任意转换处理
	    return Qs.stringify(data)
    }],
	headers: {
		'deviceCode': 'A95ZEF1-47B5-AC90BF3'
	},
	data: {
	    username: 'admin',
		pwd: 'admin'
	}
})

~~~



