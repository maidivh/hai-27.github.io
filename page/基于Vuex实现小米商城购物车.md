# 基于Vuex实现小米商城购物车

## 前言

2020年寒假尤其特殊，因为新型冠状病毒肺炎疫情，学校至今没有正式开学。想起上学期利用课余时间学习了`vue`、`node.js`，一直想做个完整的项目实战一下，但之前在学校并没有那么多的时间。现在恰好有时间，就想着做一个项目巩固之前学到的东西。

思来想去，最后决定模仿 [小米商城 ](www.mi.com)做一个电商项目，目前已经差不多做完了，本文就购物车的实现进行总结。

## 说明

> 完整项目地址：[https://github.com/hai-27/vue-store](https://github.com/hai-27/vue-store)。

>  项目部署在阿里云服务器，预览链接：[ http://106.15.179.105 ]( http://106.15.179.105)。

> 本文仅对前端部分进行总结，后端采用 Node.js(Koa)+Mysql 实现，详细代码请移步 [https://github.com/hai-27/store-server](https://github.com/hai-27/store-server)。

> 新人发帖，若有不对的地方，请多多指教 ^_^ 

## 效果

话不多说，先看效果
![](https://user-gold-cdn.xitu.io/2020/3/7/170b4d1a7b892de3?w=1919&h=1189&f=gif&s=1816428)
## 实现步骤

### 静态页面准备

![](https://user-gold-cdn.xitu.io/2020/3/7/170b4eb5b5a5bd80?w=1901&h=449&f=png&s=37994)

页面使用了 [element-ui]( https://element.eleme.cn/#/zh-CN/component/installation ) 的`Icon 图标`、` el-checkbox `、`el-input-number`、`el-popover`、`el-button`，所有在main.js需要引入element-ui

```
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
Vue.use(ElementUI);
```

页面代码如下：

```
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
    <div class="content">
      <ul>
        <!-- 购物车表头 -->
        <li class="header">
          <div class="pro-check">
            <el-checkbox :value="true">全选</el-checkbox>
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
        <li class="product-list">
          <div class="pro-check">
            <el-checkbox :value="true"></el-checkbox>
          </div>
          <div class="pro-img">
            <router-link :to="{ path: '/goods/details', query: {productID:1} }">
              <img :src="$target + 'public/imgs/phone/Redmi-k30-5G.png'" />
            </router-link>
          </div>
          <div class="pro-name">
            <router-link
              :to="{ path: '/goods/details', query: {productID:1} }"
            >Redmi K30 5G</router-link>
          </div>
          <div class="pro-price">2599元</div>
          <div class="pro-num">
            <el-input-number
              size="small"
              :value="1"
              :min="1"
              :max="10"
            ></el-input-number>
          </div>
          <div class="pro-total pro-total-in">2599元</div>
          <div class="pro-action">
            <el-popover placement="right">
              <p>确定删除吗？</p>
              <div style="text-align: right; margin: 10px 0 0">
                <el-button
                  type="primary"
                  size="mini"
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
            <span class="cart-total-num">1</span> 件商品，已选择
            <span class="cart-total-num">1</span> 件
          </span>
        </div>
        <div class="cart-bar-right">
          <span>
            <span class="total-price-title">合计：</span>
            <span class="total-price">2599元</span>
          </span>
          <router-link :to="10 > 0 ? '/confirmOrder' : ''">
            <div :class="10 > 0 ? 'btn-primary' : 'btn-primary-disabled'">去结算</div>
          </router-link>
        </div>
      </div>
      <!-- 购物车底部导航条END -->
    </div>
    <!-- 购物车主要内容区END -->

    <!-- 购物车为空的时候显示的内容 -->
    <div v-if="false" class="cart-empty">
      <div class="empty">
        <h2>您的购物车还是空的！</h2>
        <p>快去购物吧！</p>
      </div>
    </div>
    <!-- 购物车为空的时候显示的内容END -->
  </div>
```

### 创建Vuex

/store/index.js

```
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

/store//modules/shoppingCart.js

```
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

### 同步购物车状态

在根组件App.vue监听用户登录状态，如果用户已经登录，从数据库获取用户的购物车数据

代码如下：

```
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

```
setShoppingCart (state, data) {
	// 设置购物车状态
	state.shoppingCart = data;
},
```

vuex的actions

```
setShoppingCart({ commit }, data) {
  commit('setShoppingCart', data);
}
```

### 添加购物车：



### 删除购物车：

### 修改商品数量：

### 是否勾选商品：

### 是否全选商品：











