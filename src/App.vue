<template>
  <div>
    <div>
      <div>抢哪个！</div>
      <button @click="getproductsList">请求数据1</button>
      <button @click="selsecD">抢购全部D席</button>
      <div class="twolist">
        <div style="margin-right:20px;">
          <div v-for="(item,index) in arr1" :key="index">
            <div v-if="cartList[item].stock != 0">
            <!-- <div v-if="cartList[item].stock != 0 && (cartList[item].suk.slice(6,7)=='F')"> -->
              <label :for="cartList[item].id">{{cartList[item].suk}}<input @click.prevent="checkuuid(cartList[item])" class="checksku" :id="cartList[item].id" :value="JSON.stringify(cartList[item])" type="checkbox"></label>
              <span>价格{{cartList[item].price}}</span>
            </div>
          </div>
        </div>
        <div>
          <div v-for="(item1,index) in arr2" :key="index">
            <div v-if="cartList[item1].stock != 0">
            <!-- <div v-if="cartList[item].stock != 0 && (cartList[item].suk.slice(6,7)=='F')"> -->
              <label :for="cartList[item1].id">{{cartList[item1].suk}}<input @click.stop="checkuuid(cartList[item1])" class="checksku" :id="cartList[item1].id" :value="JSON.stringify(cartList[item])" type="checkbox"></label>
              <span>价格{{cartList[item1].price}}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    
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
let arr1 = ref([])
let arr2 = ref([])
function getCartList(id) {
  getJson({
    method: 'get',
    host: '/api',
    url: 'product/detail/'+id,
  }).then((res) => {
    cartList.value = res.data.productValue
    console.log(cartList.value,111111);
    arr1.value = []
    arr2.value = []
    //日期
    let attr_value = res.data.productAttr[0].attr_values
    // 座位号
    let seat = res.data.productAttr[1].attr_values
    seat.forEach(item => {
      let v1 = attr_value[0]+','+item
      let v2 = attr_value[1]+','+item
      if (!!cartList.value[v1].stock) {
        arr1.value.push(v1)
      }
      if (!!cartList.value[v2].stock) {
        arr2.value.push(v2)
      }
    });
  })
}

// 抢所有D席位
function selsecD() {
  arr1.value.forEach(item=>{
    if (cartList.value[item].suk.slice(6,7)=='D') {
      checkItem.value = cartList.value[item]
      getcartId()
    }
  })
  arr2.value.forEach(item=>{
    if (cartList.value[item].suk.slice(6,7)=='D') {
      checkItem.value = cartList.value[item]
      getcartId()
    }
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
     getType(cartId.value)
  })
}

// 获取type
let type = ref('')
function getType(id) {
  getJson({
    method: 'post',
    host: '/api',
    url: 'order/check_shipping',
    params:{"cartId":id,"new":1}
  }).then((res) => {
    type.value = res.data.type
    confirmOrder(id,type.value)
  })
}

// 提交订单
let confirm = ref({})
function confirmOrder(id,type) {
  getJson({
    method: 'post',
    host: '/api',
    url: 'order/confirm',
    params:{"cartId":id,"new":1,"addressId":0,"shipping_type":type}
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
.twolist{
  display: flex;
}
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
