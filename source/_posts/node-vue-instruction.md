---
title: 【文档小记】Node & Vue 使用方法
tags:
  - Node.js
  - Vue
  - npm
  - nvm
toc: true
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2023-10-22 18:01:43
---

记录 Node 和 Vue 一些使用方法

<!-- more -->

## 安装 Node
### 安装 nvm 工具
```bash
curl https:# raw.githubusercontent.com/creationix/nvm/master/install.sh | bash 
```

### 通过 nvm 安装 Node, npm
```bash
nvm install 16.15.0

nvm ls

nvm use <node-version>
```

## 搭建 Vue
### 安装 vue-cli 脚手架
```bash
npm install -g @vue/cli

vue create <project-name>

npm run serve

npm run dev
```

### 安装 vue-router
```bash
npm install -g vue-router@3
```

```js
// main.js
import Vue from 'vue'; 
import VueRouter from 'vue-router'; 
Vue.use(VueRouter)
```

### 安装 axios
```bash
npm install -g axios
```

```js
import axios from 'axios'
// 可以将 axios 请求封装成 bash 工具文件
```

### 删除 node_modules
```bash
npm install -g remove-node-modules
```

### 安装 Vuex
```bash
npm install -g vuex@3
```

### 配置 Vuex
* 单独创建一个 `states.js`，编辑 vuex 相关的配置 
  ```js
  // states.js
  import Vue from 'vue' 
  import Vuex from 'vuex' 

  Vue.use(Vuex) 

  export default new Vuex.Store({ 

  state:{ 
      }, 

  mutations:{ 
      }, 

  actions:{ 
      }
  }) 
  ```

* `main.js` 中导入第一步的配置文件，并在 vue 实例中声明一下 
  ```js
  import store from "./states.js" 

  new Vue({ 
  // 使用 router 
  router,
  // 使用 vuex
  store,
    render: h => h(App),
  }).$mount('#app') 
  ```

### 使用 Vuex 
* `state` 的使用 (当成组件中的data)
  ```js
  state: { 
      products: [ 
          {name: '鼠标', price: 20}, 
          {name: '键盘', price: 40}, 
          {name: '耳机', price: 60}, 
          {name: '显示屏', price: 80}
      ]
  }
  ``` 

* `getters` 相当于之前实例中讲过的 `computed` 计算属性 
  ```js
  getters:{ 
      // 获取数据的方法，类似于计算属性 
      saleProducts: (state) => { 
        let saleProducts = state.products.map((product) => { 
          return { 
            name: product.name, 
            price: product.price / 2 
          } 
        })
        return saleProducts;
      }
  } 
  ```

* `mutations` 相当于 `methods`，提供修改数据的方法 
  ```js
  mutations: { 
      minusPrice(state, payload) { 
          let newPrice = state.products.forEach(product => { 
          // 将数据的price价钱，减payload 
          product.price -= payload 
        }) 
      }
  }
  ```

* `actions` 相当于异步的 `methods`，支持一步操作，注意 `actions` 还是要通过 `mutations` 修改数据 
  ```js
  actions: { 
      minusPriceAsync(context, payload ) {
      // setTimeout定时，延迟2000毫秒，回调第一个参数的函数 
          setTimeout(() => { 
          // context提交，调用mutations中的minusPrice函数
          context.commit( 'minusPrice', payload ); 
          }, 2000)  
      } 
  } 
  ```

## 搭建 Electron-Vue
### 安装 electron-vue 脚手架模板
```bash
npm install -g @vue/cli

npm install -g electron

npm install -g @vue/cli-init

# python-shell 模块，非必须
npm install -g python-shell

# 创建 project
vue init simulatedgreg/electron-vue <project-name>

npm run dev
```

### 解决出现的报错
#### libgconf 问题  
  ```bash
  sudo apt-get install libgconf-2-4
  ```

#### 控制台 `Object.formEntries` 问题  
  ```bash
  npm i -g polyfill-object.fromentries
  ```

  ```js
  // dev-client.js
  import 'polyfill-object.fromentries';
  ```

#### `404 not found` 问题  
  ```js
  // dev-runner.js
  // 69th row，解开注释
  app.use(hotMiddleuse) 
  ```