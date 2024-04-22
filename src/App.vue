<template>
  <div>
    <div>
      <div>抢哪个！</div>
      <button @click="getproductsList">手动请求</button>
      <button @click="getCartList()">请求列表</button>
      <button @click="autoproductsList">自动请求单天抢D</button>
      <button @click="selsecD">抢购单天全部D席</button>
      <button @click="changeNew">切换new</button>
      <div>
        <div v-for="(item,index) in productsList" :key="index">
           <label :for="item.id">{{item.store_name}} <input @click.prevent="checkList(item.id)" class="checksku" :id="item.id" :value="item.id" type="checkbox"></label>
        </div>
      </div>
      <div class="twolist">
        <div style="margin-right:20px;" v-for="(item,index) in attr_value" :key="index">
          <div v-for="(item1,index1) in seat" :key="index1">
            <div v-if="cartList[item+','+item1].stock != 0">
            <!-- <div v-if="cartList[item].stock != 0 && (cartList[item].suk.slice(6,7)=='F')"> -->
              <label :for="cartList[item+','+item1].id">{{cartList[item+','+item1].suk}}<input @click.prevent="checkuuid(cartList[item+','+item1])" class="checksku" :id="cartList[item+','+item1].id" :value="JSON.stringify(cartList[item+','+item1])" type="checkbox"></label>
              <span>价格{{cartList[item+','+item1].price}}</span>
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
            'Authori-zation': 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwd2QiOiJkNDFkOGNkOThmMDBiMjA0ZTk4MDA5OThlY2Y4NDI3ZSIsImlzcyI6InFsc2hvcC55dW5tZWxsLnZpcCIsImF1ZCI6InFsc2hvcC55dW5tZWxsLnZpcCIsImlhdCI6MTcxMzUwMjk0NSwibmJmIjoxNzEzNTAyOTQ1LCJleHAiOjE3MTYwOTQ5NDUsImp0aSI6eyJpZCI6NzcwMiwidHlwZSI6ImFwaSJ9fQ.erbmY5q8uwH4tXoQGsSKilrpAQhtyEmcHjYxb96owvA',
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
let newType = ref(0)
function changeNew() {
  if (newType.value == 0) {
    newType.value = 1
  }else{
    newType.value = 0
  }
}
let productsList = ref([])
// 设定指定时间抢D席位
function autoproductsList() {
  // 设定抢购时间
  let time = 1712289600000
  // 定时器检测
  // let timer = setInterval(() => {
  //   console.log('检测中');
  //   if (new Date().getTime() >= time) {
  //     clearInterval(timer)
  //     getJson({
  //       method: 'get',
  //       host: '/api',
  //       url: 'products?sid=35&keyword=&priceOrder=&salesOrder=&news=0&page=1&limit=20&cid=0&coupon_category_id=&productId=',
  //     }).then(async (res) => {
  //       productsList.value = res.data
  //       await getCartList(productsList.value[0].id)
  //       await selsecD()
  //     })
  //   }
  // }, 10);

  let timer = setInterval(async () => {
    console.log('检测中');
    if (new Date().getTime() >= time) {
      clearInterval(timer)
      await getproductsList()
      // await getCartList()
      // await selsecD()
    }
  }, 10);
  
}
// 手动请求
function getproductsList() {
  getJson({
    method: 'get',
    host: '/api',
    url: 'products?sid=35&keyword=&priceOrder=&salesOrder=&news=0&page=1&limit=20&cid=0&coupon_category_id=&productId=',
  }).then((res) => {
     productsList.value = res.data
  })
}
let useId = ref('')
// 选择哪天摊位链接
function checkList(id){
  useId.value = id
  getCartList()
}

// 请求列表
let cartList = ref([])
let attr_value = ref([])
let seat = ref([])
let arr3 = ref([])
function getCartList() {
  let url = 'product/detail/453'
  if (useId.value) {
    url = 'product/detail/'+useId.value
  }
  getJson({
    method: 'get',
    host: '/api',
    url,
  }).then((res) => {
    cartList.value = res.data.productValue
    //日期
    attr_value.value = res.data.productAttr[0].attr_values
    // 座位号
    if (res.data.productAttr[1]) {
      seat.value = res.data.productAttr[1].attr_values
    }
  })
}

// 抢所有D席位
function selsecD() {
  seat.value.forEach(item1=>{
    if (item1.slice(0,1)=='D') {
      attr_value.value.forEach(item=>{
        if (cartList.value[item+','+item1].stock != 0) {
          checkItem.value = cartList.value[item+','+item1]
          getcartId(checkItem)
        }
      })
    }
  })
}

// 选中项目
let checkItem = ref({})
function checkuuid(item) {
  checkItem.value = item
  getcartId(checkItem)
}
// 获取id
let cartId = ref("")
function getcartId(checkItem) {
  getJson({
    method: 'post',
    host: '/api',
    url: 'cart/add',
    params:{
    "productId": checkItem.value.product_id,
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
    params:{"cartId":id,"new":newType.value}
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
    params:{"cartId":id,"new":newType.value,"addressId":0,"shipping_type":type}
  }).then((res) => {
    confirm.value = res.data
    creatOrder(confirm.value)
  })
}

// 创建订单
function creatOrder(confirm) {
  getJson({
    method: 'post',
    host: '/api',
    url: 'order/create/'+confirm.orderKey,
    params:{
        "custom_form": confirm.custom_form,
        "real_name": "史文琳",
        "phone": "16643563081",
        "addressId": 0,
        "formId": "",
        "couponId": 0,
        "useIntegral": false,
        "bargainId": confirm.bargain_id,
        "combinationId":  confirm.combination_id,
        "discountId": null,
        "pinkId": 0,
        "advanceId": confirm.advance_id,
        "seckill_id": confirm.seckill_id,
        "mark": "",
        "store_id": 5,
        "from": "routine",
        "shipping_type": type.value,
        "new": newType.value,
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
