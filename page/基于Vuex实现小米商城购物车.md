# 基于Vuex实现小米商城购物车

## 前言

上学期利用课余时间学习了Vue.js、Node.js，一直想做个完整的项目 实践 一下，但之前在学校并没有那么多的时间。现在恰好有时间，就想着做一个项目巩固之前学到的东西。

思来想去，最后决定模仿 [小米商城 ](www.mi.com)做一个电商项目，目前已经差不多做完了，本文就购物车模块的实现进行总结。

## 说明

> 完整项目代码仓库：[https://github.com/hai-27/vue-store](https://github.com/hai-27/vue-store)。

>  项目部署在阿里云服务器，预览链接：[ http://106.15.179.105 ]( http://106.15.179.105)(没有兼容移动端，请使用PC访问)。

> 本文仅对前端部分进行总结，后端采用 Node.js(Koa)+Mysql 实现，详细代码请移步 [https://github.com/hai-27/store-server](https://github.com/hai-27/store-server)。

> 新人发帖，若有不对的地方，请多多指教 ^_^ 

## 效果

话不多说，先看效果
![](https://user-gold-cdn.xitu.io/2020/3/7/170b4d1a7b892de3?w=1919&h=1189&f=gif&s=1816428)

## 实现步骤

### 1. 静态页面准备

![](https://user-gold-cdn.xitu.io/2020/3/7/170b4eb5b5a5bd80?w=1901&h=449&f=png&s=37994)

页面使用了 [element-ui]( https://element.eleme.cn/#/zh-CN/component/installation ) 的`Icon 图标`、` el-checkbox `、`el-input-number`、`el-popover`、`el-button`，所有在main.js需要引入element-ui。

```javascript
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
Vue.use(ElementUI);
```

页面代码如下：

**说明：** 为了方便，此处直接放最终的代码。

```html
<template>
  <div class="shoppingCart">
    <!-- 购物车头部 -->
    <div class="cart-header">
      <div class="cart-header-content">
        <p>
          <i class="el-icon-shopping-cart-full" style="color:#ff6700; font-weight: 600;">
          </i>
          我的购物车
        </p>
        <span>温馨提示：产品是否购买成功，以最终下单为准哦，请尽快结算</span>
      </div>
    </div>
    <!-- 购物车头部END -->

    <!-- 购物车主要内容区 -->
    <div class="content" v-if="getShoppingCart.length>0">
      <ul>
        <!-- 购物车表头 -->
        <li class="header">
          <div class="pro-check">
            <el-checkbox v-model="isAllCheck">全选</el-checkbox>
          </div>
          <div class="pro-img"></div>
          <div class="pro-name">商品名称</div>
          <div class="pro-price">单价</div>
          <div class="pro-num">数量</div>
          <div class="pro-total">小计</div>
          <div class="pro-action">操作</div>
        </li>
        <!-- 购物车表头END -->

        <!-- 购物车列表 -->
        <li class="product-list" v-for="(item,index) in getShoppingCart" :key="item.id">
          <div class="pro-check">
            <el-checkbox :value="item.check" @change="checkChange($event,index)">
            </el-checkbox>
          </div>
          <div class="pro-img">
            <router-link :to="{ 
            path: '/goods/details', 
            query: {productID:item.productID} 
            }">
              <img :src="$target + item.productImg" />
            </router-link>
          </div>
          <div class="pro-name">
            <router-link
              :to="{ path: '/goods/details', query: {productID:item.productID} }"
            >{{item.productName}}</router-link>
          </div>
          <div class="pro-price">{{item.price}}元</div>
          <div class="pro-num">
            <el-input-number
              size="small"
              :value="item.num"
              @change="handleChange($event,index,item.productID)"
              :min="1"
              :max="item.maxNum"
            ></el-input-number>
          </div>
          <div class="pro-total pro-total-in">{{item.price*item.num}}元</div>
          <div class="pro-action">
            <el-popover placement="right">
              <p>确定删除吗？</p>
              <div style="text-align: right; margin: 10px 0 0">
                <el-button
                  type="primary"
                  size="mini"
                  @click="deleteItem($event,item.id,item.productID)"
                >确定</el-button>
              </div>
              <i class="el-icon-error" slot="reference" style="font-size: 18px;"></i>
            </el-popover>
          </div>
        </li>
        <!-- 购物车列表END -->
      </ul>
      <div style="height:20px;background-color: #f5f5f5"></div>
      <!-- 购物车底部导航条 -->
      <div class="cart-bar">
        <div class="cart-bar-left">
          <span>
            <router-link to="/goods">继续购物</router-link>
          </span>
          <span class="sep">|</span>
          <span class="cart-total">
            共
            <span class="cart-total-num">{{getNum}}</span> 件商品，已选择
            <span class="cart-total-num">{{getCheckNum}}</span> 件
          </span>
        </div>
        <div class="cart-bar-right">
          <span>
            <span class="total-price-title">合计：</span>
            <span class="total-price">{{getTotalPrice}}元</span>
          </span>
          <router-link :to="getCheckNum > 0 ? '/confirmOrder' : ''">
            <div :class="getCheckNum > 0 ? 'btn-primary' : 'btn-primary-disabled'">
            去结算
            </div>
          </router-link>
        </div>
      </div>
      <!-- 购物车底部导航条END -->
    </div>
    <!-- 购物车主要内容区END -->

    <!-- 购物车为空的时候显示的内容 -->
    <div v-else class="cart-empty">
      <div class="empty">
        <h2>您的购物车还是空的！</h2>
        <p>快去购物吧！</p>
      </div>
    </div>
    <!-- 购物车为空的时候显示的内容END -->
  </div>
</template>
```

### 2. 创建Vuex

/store/index.js

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

import shoppingCart from './modules/shoppingCart'

Vue.use(Vuex)

export default new Vuex.Store({
  strict: true,
  modules: {
    shoppingCart
  }
})
```

/store/modules/shoppingCart.js

```javascript
export default {
  state: {
    shoppingCart: []
    // shoppingCart结构
    /* 
    shoppingCart = {
      id: "", // 购物车id
      productID: "", // 商品id
      productName: "", // 商品名称
      productImg: "", // 商品图片
      price: "", // 商品价格
      num: "", // 商品数量
      maxNum: "", // 商品限购数量
      check: false // 是否勾选
    }
    */
  }
}
```

### 3. 同步购物车状态

**思路：**

- 在根组件App.vue监听用户登录状态；
- 如果用户已经登录，从数据库获取用户的购物车数据，把获取到数据更新到vuex；
- 用户没有登录，把vuex中购物车的状态设置为空。

代码如下：

```javascript
import { mapActions } from "vuex";
import { mapGetters } from "vuex";

computed: {
  ...mapGetters(["getUser", "getNum"])
},
methods: {
  ...mapActions(["setShoppingCart"]),
}
watch: {
  // 获取vuex的登录状态
  getUser: function(val) {
    if (val === "") {
      // 用户没有登录
      this.setShoppingCart([]);
    } else {
      // 用户已经登录,获取该用户的购物车信息
      this.$axios
        .post("/api/user/shoppingCart/getShoppingCart", {
          user_id: val.user_id
        })
        .then(res => {
          if (res.data.code === "001") {
            // 001 为成功, 更新vuex购物车状态
            this.setShoppingCart(res.data.shoppingCartData);
          } else {
            // 提示失败信息
            this.notifyError(res.data.msg);
          }
        })
        .catch(err => {
          return Promise.reject(err);
        });
    }
  }
}
```

vuex的mutations：

```javascript
setShoppingCart (state, data) {
	// 设置购物车状态
	state.shoppingCart = data;
},
```

vuex的actions

```javascript
setShoppingCart({ commit }, data) {
  commit('setShoppingCart', data);
}
```

### 4. 动态生成购物车页面

**思路：**

- 通过vuex中的getters.getShoppingCart获取购物车的状态；
- 使用v-if判断购物车是否存在商品；
- 如果存在，使用v-for生成购物车列表；
- 如果不存在，显示购物车为空的时候显示的内容；

购物车html伪代码：

```html
<div class="shoppingCart">
  <div class="content" v-if="getShoppingCart.length>0">
    <ul>
      <li class="header">
        <!-- 购物车表头部分,省略详细代码 -->
      </li>
      <li class="product-list" v-for="(item,index) in getShoppingCart" :key="item.id">
        <!-- 购物车列表部分，省略详细代码 -->
      </li>
    </ul>
  </div>
  <!-- 购物车为空的时候显示的内容 -->
  <div v-else class="cart-empty">
    <div class="empty">
      <h2>您的购物车还是空的！</h2>
      <p>快去购物吧！</p>
    </div>
    
  </div>
</div>
```

vuex的getters：

```javascript
getShoppingCart(state) {
  // 获取购物车状态
  return state.shoppingCart;
}
```

### 5.添加商品到购物车

**思路：**

- 用户在商品的详情页，通过点击加入购物车按钮，调用点击事件addShoppingCart；
- 先向后端发起添加购物车的请求，根据返回信息操作vuex；
- 该商品第一次加入购物车，通过vuex的 Actions (unshiftShoppingCart)把后端返回的购物车信息插入vuex；
- 该商品已经在购物车，通过vuex的 Actions (addShoppingCartNum)把该商品数量+1；
- 商品数量达到限购数量，禁止点击加入购物车按钮。

html：

```html
<el-button class="shop-cart" :disabled="dis" @click="addShoppingCart">
  加入购物车
</el-button>
```

逻辑代码如下：

```javascript
methods: {
  ...mapActions(["unshiftShoppingCart", "addShoppingCartNum"]),
  // 加入购物车
  addShoppingCart() {
    // 判断是否登录,没有登录则显示登录组件
    if (!this.$store.getters.getUser) {
      this.$store.dispatch("setShowLogin", true);
      return;
    }
    // 向后端发起请求，把商品信息插入数据库的购物车表
    this.$axios
      .post("/api/user/shoppingCart/addShoppingCart", {
        user_id: this.$store.getters.getUser.user_id,
        product_id: this.productID
      })
      .then(res => {
        switch (res.data.code) {
          case "001":
            // 新加入购物车成功
            this.unshiftShoppingCart(res.data.shoppingCartData[0]);
            this.notifySucceed(res.data.msg);
            break;
          case "002":
            // 该商品已经在购物车，数量+1
            this.addShoppingCartNum(this.productID);
            this.notifySucceed(res.data.msg);
            break;
          case "003":
            // 商品数量达到限购数量
            this.dis = true;
            this.notifyError(res.data.msg);
            break;
          case "401":
            // 没有登录
            this.$store.dispatch("setShowLogin", true);
            this.notifyError(res.data.msg);
            break;
          default:
            this.notifyError(res.data.msg);
        }
      })
      .catch(err => {
        return Promise.reject(err);
      });
  }
}
```

vuex的mutations：

```javascript
unshiftShoppingCart(state, data) {
  // 添加商品到购物车
  // 用于在商品详情页点击添加购物车,后台添加成功后，更新vuex状态
  state.shoppingCart.unshift(data);
},
addShoppingCartNum(state, productID) {
  // 增加购物车商品数量
  // 用于在商品详情页点击添加购物车,后台返回002，“该商品已在购物车，数量 +1”，更新vuex的商品数量
  for (let i = 0; i < state.shoppingCart.length; i++) {
    const temp = state.shoppingCart[i];
    if (temp.productID == productID) {
      if (temp.num < temp.maxNum) {
        temp.num++;
      }
    }
  }
}
```

vuex的actions：

```javascript
unshiftShoppingCart({ commit }, data) {
  commit('unshiftShoppingCart', data);
},
addShoppingCartNum({ commit }, productID) {
  commit('addShoppingCartNum', productID);
}
```

### 6. 删除购物车中的商品

**思路：**

- 购物车每个商品都有一个删除按钮，用户点击删除按钮，先弹出确认对话框；
- 当用户选择确认删除，调用点击事件deleteItem($event,item.id,item.productID)；
- 通过点击事件获取到购物车id和商品id；
- 先向后端发起删除购物车的请求，根据返回信息操作vuex；
- 删除成功，通过vuex的 Actions (deleteShoppingCart)，把该商品从购物车删除；
- 如果删除失败，提示相关信息。

html：

```html
<div class="pro-action">
  <el-popover placement="right">
    <p>确定删除吗？</p>
    <div style="text-align: right; margin: 10px 0 0">
      <el-button type="primary" size="mini" 
      @click="deleteItem($event,item.id,item.productID)">确定</el-button>
    </div>
    <i class="el-icon-error" slot="reference" style="font-size: 18px;"></i>
  </el-popover>
</div>
```

逻辑代码如下：

```javascript
methods: {
  // 向后端发起删除购物车的数据库信息请求
  deleteItem(e, id, productID) {
    this.$axios
      .post("/api/user/shoppingCart/deleteShoppingCart", {
        user_id: this.$store.getters.getUser.user_id,
        product_id: productID
      })
      .then(res => {
        switch (res.data.code) {
          case "001":
            // 删除成功
            // 更新vuex状态
            this.deleteShoppingCart(id);
            // 提示删除成功信息
            this.notifySucceed(res.data.msg);
            break;
          default:
            // 提示删除失败信息
            this.notifyError(res.data.msg);
        }
      })
      .catch(err => {
        return Promise.reject(err);
      });
  }
}
```

vuex的mutations：

```javascript
deleteShoppingCart(state, id) {
  // 根据购物车id删除购物车商品
  for (let i = 0; i < state.shoppingCart.length; i++) {
    const temp = state.shoppingCart[i];
    if (temp.id == id) {
      state.shoppingCart.splice(i, 1);
    }
  }
}
```

vuex的actions：

```javascript
deleteShoppingCart({ commit }, id) {
  commit('deleteShoppingCart', id);
}
```

### 7. 修改购物车商品的数量

**思路：**

- 购物车每个商品都有一个计数器，可以点击加减按钮修改商品数量，或者直接在input输入框输入商品数量进行修改。计数器使用了 [element-ui]( https://element.eleme.cn/#/zh-CN/component/installation ) 的`el-input-number`实现。
- 通过计数器的 change 事件获取到新的数量、购物车商品的索引(即数组的索引)、商品id；
- 先向后端发起修改购物车商品数量的请求，根据返回信息操作vuex；
- 修改成功，通过vuex的 Actions (updateShoppingCart)，修改购物车商品的数量；
- 修改失败，提示失败信息。其中：数量小于1、数量是否达到限购数量（这两种情况，前端有设置校验，一般不会出现）。

html：

```html
<div class="pro-num">
  <el-input-number 
  size="small" 
  :value="item.num" 
  @change="handleChange($event,index,item.productID)" 
  :min="1"
  :max="item.maxNum"
  >
</el-input-number>
```

逻辑代码如下：

```javascript
// 修改商品数量的时候调用该函数
handleChange(currentValue, key, productID) {
  // 当修改数量时，默认勾选
  this.updateShoppingCart({ key: key, prop: "check", val: true });
  // 向后端发起修改购物车商品数量的请求
  this.$axios
    .post("/api/user/shoppingCart/updateShoppingCart", {
      user_id: this.$store.getters.getUser.user_id,
      product_id: productID,
      num: currentValue
    })
    .then(res => {
      switch (res.data.code) {
        case "001":
          // 001代表修改成功
          // 更新vuex状态
          this.updateShoppingCart({
            key: key,
            prop: "num",
            val: currentValue
          });
          // 提示修改成功信息
          this.notifySucceed(res.data.msg);
          break;
        default:
          // 提示修改失败信息
          this.notifyError(res.data.msg);
      }
    })
    .catch(err => {
      return Promise.reject(err);
    });
}
```

vuex的mutations：

```javascript
updateShoppingCart(state, payload) {
  // 更新购物车
  // 可更新商品数量和是否勾选
  // 用于购物车点击勾选及加减商品数量
  if (payload.prop == "num") {
    // 判断效果的商品数量是否大于限购数量或小于1
    if (state.shoppingCart[payload.key].maxNum < payload.val) {
      return;
    }
    if (payload.val < 1) {
      return;
    }
  }
  // 根据商品在购物车的数组的索引和属性更改
  state.shoppingCart[payload.key][payload.prop] = payload.val;
}
```

vuex的actions：

```javascript
updateShoppingCart({ commit }, payload) {
  commit('updateShoppingCart', payload);
}
```

### 8. 是否勾选商品

**思路：**

- 购物车每个商品都有一个勾选框，使用[element-ui]( https://element.eleme.cn/#/zh-CN/component/installation ) 的`el-checkbox`实现，结算时提交全部勾选的商品。
- 通过 change 事件获取到勾选框的状态（true或false）、购物车商品的索引(即数组的索引)；
- 通过vuex的 Actions (updateShoppingCart)，修改购物车商品的勾选状态；

html:

```html
<div class="pro-check">
  <el-checkbox :value="item.check" @change="checkChange($event,index)"></el-checkbox>
</div>
```

逻辑代码如下：

```javascript
checkChange(val, key) {
  // 更新vuex中购物车商品是否勾选的状态
  this.updateShoppingCart({ key: key, prop: "check", val: val });
}
```

**说明：** 此处使用的vuex的mutationsvuex和actions，和**修改商品数量**的是同一个，两个场景，通过传递的参数不同进行区分。**修改商品数量**时传递参数是{ key: key, prop: "num", val: val }，**是否勾选商品**传递的参数是{ key: key, prop: "check", val: val }，请注意**prop**的变化。

### 9. 是否全选商品

**思路：**

- 购物车设置了一个全选框，通过v-model绑定**isAllCheck**。
- **isAllCheck**值是通过计算属性的 getter获取vuex中的getters.getIsAllCheck;
- vuex中的getters.getIsAllCheck通过遍历购物车数组，判断每一个商品勾选状态，只要有一个商品没有勾选，getIsAllCheck均为false，否则为true；
- 当点击全选框，通过计算属性的 setter 调用vuex的Actions (checkAll)，更改每个商品的勾选状态，从而修改全选框的状态。

html：

```html
<div class="pro-check">
  <el-checkbox v-model="isAllCheck">全选</el-checkbox>
</div>
```

逻辑代码如下：

```javascript
computed: { 
  isAllCheck: {
    get() {
      return this.$store.getters.getIsAllCheck;
    },
    set(val) {
      this.checkAll(val);
    }
  }
}
```

vuex的getters：

```javascript
getIsAllCheck(state) {
  // 判断是否全选
  let isAllCheck = true;
  for (let i = 0; i < state.shoppingCart.length; i++) {
    const temp = state.shoppingCart[i];
    // 只要有一个商品没有勾选立即return false;
    if (!temp.check) {
      isAllCheck = false;
      return isAllCheck;
    }
  }
  return isAllCheck;
}
```

vuex的mutations：

```javascript
checkAll(state, data) {
  // 点击全选按钮，更改每个商品的勾选状态
  for (let i = 0; i < state.shoppingCart.length; i++) {
    state.shoppingCart[i].check = data;
  }
}
```

vuex的actions

```javascript
checkAll({ commit }, data) {
  commit('checkAll', data);
}
```

### 10. 计算购物车中商品的总数量

在购物车页面和根组件的顶部导航栏使用。

vuex的getters：

```javascript
getNum(state) {
  // 购物车商品总数量
  let totalNum = 0;
  for (let i = 0; i < state.shoppingCart.length; i++) {
    const temp = state.shoppingCart[i];
    totalNum += temp.num;
  }
  return totalNum;
}
```

### 11. 计算购物车中勾选的商品总数量

在购物车页面和结算页面使用。

vuex的getters：

```javascript
getCheckNum(state) {
  // 获取购物车勾选的商品总数量
  let totalNum = 0;
  for (let i = 0; i < state.shoppingCart.length; i++) {
    const temp = state.shoppingCart[i];
    if (temp.check) {
      totalNum += temp.num;
    }
  }
  return totalNum;
}
```

### 12. 计算购物车中勾选的商品总价格

在购物车页面和结算页面使用。

vuex的getters：

```javascript
getTotalPrice(state) {
  // 购物车勾选的商品总价格
  let totalPrice = 0;
  for (let i = 0; i < state.shoppingCart.length; i++) {
    const temp = state.shoppingCart[i];
    if (temp.check) {
      totalPrice += temp.price * temp.num;
    }
  }
  return totalPrice;
}
```

### 13.生成购物车中勾选的商品详细信息

在结算页面使用。

vuex的getters：

```javascript
getCheckGoods(state) {
  // 获取勾选的商品信息
  // 用于确认订单页面
  let checkGoods = [];
  for (let i = 0; i < state.shoppingCart.length; i++) {
    const temp = state.shoppingCart[i];
    if (temp.check) {
      checkGoods.push(temp);
    }
  }
  return checkGoods;
}
```

## 总结

至此，购物车的前端部分已经全部实现：从数据库同步购物车数据，根据购物车数据动态生成购物车页面，添加商品到购物车，删除购物车中的商品，修改购物车商品的数量，是否勾选购物车商品，是否全选购物车商品， 计算购物车中商品的总数量，计算购物车中勾选的商品总数量，计算购物车中勾选的商品总价格，生成购物车中勾选的商品详细信息。

## 后记

> 结束了，新人第一次发帖，若有不对的地方，请多多指教 ^_^ 

> 本文是基于完整项目，就购物车模块的实现进行总结。

> 完整项目代码仓库：[https://github.com/hai-27/vue-store](https://github.com/hai-27/vue-store)。

> 项目预览链接：[ http://106.15.179.105 ]( http://106.15.179.105)(没有兼容移动端，请使用PC访问)。

> 喜欢本文的同学，不妨点个赞，如果能给完整项目代码仓库加个Star就更好了，谢谢 ^_^ 

> 对这个项目会做更多的总结，感兴趣的同学可以点个关注。

> 感谢你的阅读！







**作者** [hai-27](https://github.com/hai-27)

2020年3月8日