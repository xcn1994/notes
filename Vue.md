# Vue

## 一、什么是 vue

​			新生前端开发技术，分离页面（html）与控制层（js），运行时打包，解耦开发

## 二、为什么使用 vue

1、使用方便，只需要导入 vue 即可使用

2、使用 npm 管理依赖， webpack 打包工具对 vue.js 打包，减少文件数量，降低 http 请求，提高请求效率

3、官方 CLI 脚本架方便创建 vue.js 工程雏形

## 三、vue 的使用方式

### 1、v-model 

双向数据绑定，只能在以下元素中使用

```html
input
select
textarea
componets （vue组件）
```

### 2、v-text 

插值表达式延迟加载，解决表达式闪烁问题

```html
v-text="" 属性代替插值表达式
```

### 3、v-on 

绑定事件

```html
v-on:click='change' 
```

### 4、v-bind

-  可以将数据绑定在 dom 的任意属性中

-  可以给 dom 对象绑定一个或多个特性，如动态绑定 style 和 class	

```html
v-bind:src=''
v-bind:style='{ fontSzie:size + 'px'}'

缩写模式
:src=''
:style='{ fontSize:size + 'px'}'
```

### 5、v-if 和 v-for

```html
//数组遍历
v-for='(value, index) in myList' >{{index}}--{{value}}<
v-for='(value, index) in myList:key='index' v-if="index % 2 == 0"' >{{index}}--{{value}}<

//对象遍历
v-for='(value, key) in myObject' >{{key}}--{{value}}<

//集合遍历
v-for='(value, key) in myArrList' : name='value.user.name' >{{key}}--{{name}}--{{value.user.age}}<
v-if='value.user.age <= 10' >{{key}}--{{name}}--{{value.user.age}}<
v-else=''>{{key}}--{{name}}--{{value.user.age}}<
```

### 示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
</head>
<body>

<div id="myApp">
    {{name}}
    <input type="text" v-model="numA"> +
    <input type="text" v-model="numB"> =
    {{result}}
    <button v-on:click="change()">计算</button>
    <br>
    <li v-for="(value, index) in myArr" v-if="value % 2 == 0">{{index}}--{{value}}
    </li>

    <a v-bind:style="{fontSize:size+'px', color:color}">测试测试</a>
    <a :href="{url}">百度</a>
    <div v-for="(value, key) in myList">{{key}}--{{value}}</div>

    <div v-for="(item, index) in myArray":key="item.person.user">
        {{index}}--{{item.person.user}}--{{item.person.age}}
    </div>
</div>
<!-- <script src="./build.js"></script> -->
</body>
    <script>
    	var VM = new Vue({
            el: "#myApp",
            data: {
                url:'www.baidu.com',
                name: "测试数据",
                numA:1,
                numB:2,
                result:0,
                size:20,
                color:'blue',
                myArr:
                    [1,2,3,4,5],
                myList:
                    {user:"xiaoli", age:12},
                myArray:
                    [
                        { person:{user:"xiaoli", age:12} },
                        { person:{user:"huahua", age:23} },
                        { person:{user:"paopao", age:32} }
                    ]
            },
            methods:{
                change:function () {
                    this.result = add(Number.parseInt(this.numA),Number.parseInt(this.numB));
                }
            }
        });
    </script>
</html>
```

## 四、nodejs 与 webpack

### 1、安装 nodejs

### 2、安装 npm

​		nodejs 集成了 npm。 npm  -v

​		在 nodejs 目录下创建 npm_modeuls 与 npm_cache

```
npm config ls
npm config set prefix "./npm_modeuls"
npm config set cache "./npm_cache"
```

### 3、安装 cnpm

配置国内镜像，提升下载速度

```cmd
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm -v
cnpm ls
cnpm install -g nrm
nrm use taotao
```

### 4、安装 webpack 和 webpack-server

```
#全局安装
npm install webpack@3.6.0 -g --save-dev

#本项目安装,项目目录下
npm install webpack@3.6.0 webpack-dev-server@2.9.1 html-webpack-plugin@2.30.1 --save-dev
```

### 5、打包测试

- 创建 module.js ，分离 script 代码

- 创建 src/main.js

- 创建 dist 目录

- 将 a.html 与 module.js 放在同一目录下

- 在 a.htnl 目录中安装 webpack 和 webpack-server

- 修改 package.json

- 创建 webpack.config.js 文件

- debug 调试

  #### 测试代码

  ```js
  #module.js 
  
  function add(x, y) {
      #断点调试
      debugger
      return x + y;
  }
  
  function sub(x, y) {
      return x - y;
  }
  
  // module.exports = {add, sub}
  module.exports.add = add;
  // module.exports.sub = sub;
  ```

  ```js
  #mian.js
  
  var {add} = require("../model01.js");
  var Vue = require("../vue.min");
  var VM = new Vue({
      el: "#myApp",
      data: {
          url:'www.baidu.com',
          name: "测试数据",
          numA:1,
          numB:2,
          result:0,
          size:20,
          color:'blue',
          myArr:
              [1,2,3,4,5],
          myList:
              {user:"xiaoli", age:12},
          myArray:
              [
                  { person:{user:"xiaoli", age:12} },
                  { person:{user:"huahua", age:23} },
                  { person:{user:"paopao", age:32} }
              ]
      },
      methods:{
          change:function () {
              this.result = add(Number.parseInt(this.numA),Number.parseInt(this.numB));
          }
      }
  });
  ```

  ```js
  #webpack.config.js
  
  var htmlwp = require('html-webpack-plugin');	// 引入html-webpack-plugin插件
  module.exports={
      entry:'./src/main.js',  		// 指定打包的入口文件
      output:{
          path : __dirname+'/dist',   // 注意：__dirname表示webpack.config.js所在目录的绝对路径
          filename:'build.js'		    // 输出文件
      },
      devtool:'eval-source-map',      //开启debug 调试
      plugins:[						//创建插件对象
          new htmlwp({
              title: '首页',  		   // 生成的页面标题
              filename: 'index.html', // webpack-dev-server在内存中生成的文件名称
              template: 'myVue.html' // 根据vue_02.html这个模板来生成(模板文件需要自己提供)
          })
      ]
  }
  ```

  ```json
  #package.json
  
  {
    "scripts": {	
      "dev": "webpack-dev-server --inline --hot --open --port 5008"
  	},
  	
    "devDependencies": {
      "html-webpack-plugin": "^2.30.1",
      "webpack": "^3.6.0",
      "webpack-dev-server": "^2.9.1"
    }
  }
  
  ```

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>测试</title>
  </head>
  <body>
  
  <div id="myApp">
      {{name}}
      <input type="text" v-model="numA"> +
      <input type="text" v-model="numB"> =
      {{result}}
      <button v-on:click="change()">计算</button>
      <br>
      <li v-for="(value, index) in myArr" v-if="value % 2 == 0">{{index}}--{{value}}
      </li>
  
      <a v-bind:style="{fontSize:size+'px', color:color}">测试测试</a>
      <a :href="{url}">百度</a>
      <div v-for="(value, key) in myList">{{key}}--{{value}}</div>
  
      <div v-for="(item, index) in myArray":key="item.person.user">
          {{index}}--{{item.person.user}}--{{item.person.age}}
      </div>
  </div>
  
  </body>
  </html>
  ```

   使用 web-strom 开发工具，右键 .json 文件，选择 show npm script
  
  
  
  ## 五、vue 工程
  
  ### ![前后端请求响应流程](D:\Java\笔记\前后端请求响应流程.png)

![vue流程](D:\Java\笔记\vue流程.jpg)



### 钩子函数

![钩子函数](D:\Java\笔记\钩子函数.jpg)

![钩子函数2](D:\Java\笔记\钩子函数2.jpg)