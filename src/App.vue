<template>
  <div>
    <div>
      <div>抢哪个！</div>
      <div v-for="(item,index) in cartList" :key="index">
        <div v-if="item.stock != 0">
          <label @click="checkuuid(item)" :for="item.id">{{item.suk}}<input class="checksku" :id="item.id" :value="JSON.stringify(item)" type="checkbox"></label>
          <span>价格{{item.price}}</span>
        </div>
        <!-- <div v-for="(item1,index1) in item.sku_detail" :key="item1.goods_id">
          <label @click="checkuuid(item.sku_list[index1])" :for="item.sku_list[index1].goods_id">{{item1.goods_name}}<input class="checksku" :id="item.sku_list[index1].goods_id" :value="JSON.stringify(item.sku_list[index1])" type="checkbox"></label>
          <span>价格{{item1.price}}</span>
        </div> -->
      </div>
    </div>
    <button @click="getproductsList">请求数据1</button>
    <button @click="queryorder">开始抢购</button>
    <button @click="getproductsList">ewfewf</button>
  </div>
</template>

<script setup>
import { ref,reactive } from 'vue'
import HelloWorld from './components/HelloWorld.vue'
import axios from 'axios'
const params = reactive({})
function ajax({ method = 'POST', url = '', params = {}, token = '', host = "/api", headers = {} }) {
    method = method.toUpperCase()
    url = host + url
    return new Promise((resolve, reject) => {
        let baseHeaders = {
            'Content-Type': 'application/json;charset=UTF-8',
            'Authori-zation': 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwd2QiOiJkNDFkOGNkOThmMDBiMjA0ZTk4MDA5OThlY2Y4NDI3ZSIsImlzcyI6InFsc2hvcC55dW5tZWxsLnZpcCIsImF1ZCI6InFsc2hvcC55dW5tZWxsLnZpcCIsImlhdCI6MTcxMDc1MjkwMCwibmJmIjoxNzEwNzUyOTAwLCJleHAiOjE3MTMzNDQ5MDAsImp0aSI6eyJpZCI6NzcwMiwidHlwZSI6ImFwaSJ9fQ.rjdHkvfgd2UMXXwHg7hPLm0jFUBZWwTmqU3uWrZRK68',
        }
        if (token) {
          baseHeaders.accesstoken= token
        }
        let opt = {
            method: method,
            url: url,
            timeout: 1000 * 60,
            headers: Object.assign(baseHeaders, headers)
        } 
        if (method === 'POST'){
            opt.data = params
        } else if (method === 'GET') {
            opt.params = params
        }
        axios( opt )
        .then((res) => {
          resolve(res.data)
        })
        .catch((err) => {
            reject(err.data);
        })
    })
}
function getJson({ method = 'POST', url = '', params = {}, token = '',uid = '', host = "/api", headers = {} }) {
  return new Promise((resolve, reject)=>{
    ajax({ method, url, params, token, uid, host, headers }).then(data => resolve(data)).catch(err => reject(err))
  })
}

// 请求分类
let productsList = ref([])
function getproductsList() {
  getJson({
    method: 'get',
    host: '/api',
    url: 'products?sid=35&keyword=&priceOrder=&salesOrder=&news=0&page=1&limit=20&cid=0&coupon_category_id=&productId=',
  }).then((res) => {
     productsList.value = res.data
     getCartList(productsList.value[0].id)
  })
}
// 请求列表
let cartList = ref([])
function getCartList(id) {
  getJson({
    method: 'get',
    host: '/api',
    url: 'product/detail/'+id,
  }).then((res) => {
     cartList.value = res.data.productValue
     console.log(cartList.value,111111);
  })
}

// 选中项目
let checkItem = ref({})
function checkuuid(item) {
  checkItem.value = item
  getcartId()
}
// 获取id
let cartId = ref("")
function getcartId() {
  getJson({
    method: 'post',
    host: '/api',
    url: 'cart/add',
    params:{
      "productId": productsList.value[0].id,
      "cartNum": 1,
      "new": 1,
      "uniqueId": checkItem.value.unique,
      "virtual_type": 0
    }
  }).then((res) => {
     cartId.value = res.data.cartId
     console.log(cartId.value,22222222);
     getType()
  })
}

// 获取type
let type = ref('')
function getType() {
  getJson({
    method: 'post',
    host: '/api',
    url: 'order/check_shipping',
    params:{"cartId":cartId.value,"new":1}
  }).then((res) => {
    type.value = res.data.type
    confirmOrder()
  })
}

// 提交订单
let confirm = ref({})
function confirmOrder() {
  getJson({
    method: 'post',
    host: '/api',
    url: 'order/confirm',
    params:{"cartId":cartId.value,"new":1,"addressId":0,"shipping_type":type.value}
  }).then((res) => {
    confirm.value = res.data
    creatOrder()
  })
}

// 创建订单
function creatOrder() {
  getJson({
    method: 'post',
    host: '/api',
    url: 'order/create/'+confirm.value.orderKey,
    params:{
        "custom_form": cartList.value.custom_form,
        "real_name": "史文琳",
        "phone": "16643563081",
        "addressId": 0,
        "formId": "",
        "couponId": 0,
        "useIntegral": false,
        "bargainId": cartList.value.bargain_id,
        "combinationId":  cartList.value.combination_id,
        "discountId": null,
        "pinkId": 0,
        "advanceId": cartList.value.advance_id,
        "seckill_id": cartList.value.seckill_id,
        "mark": "",
        "store_id": 5,
        "from": "routine",
        "shipping_type": type.value,
        "new": 1,
        "invoice_id": ""
    }
  }).then((res) => {

  })
}



</script>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
