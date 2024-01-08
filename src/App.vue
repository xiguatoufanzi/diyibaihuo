<template>
  <div>
    <div>
      <div>抢哪个！</div>
      <div v-for="(item) in cartList" :key="item.activity_no">
        <div v-for="(item1,index1) in item.sku_detail" :key="item1.goods_id">
          <label @click="checkuuid(item.sku_list[index1])" :for="item.sku_list[index1].goods_id">{{item1.goods_name}}<input class="checksku" :id="item.sku_list[index1].goods_id" :value="JSON.stringify(item.sku_list[index1])" type="checkbox"></label>
          <span>价格{{item1.price}}</span>
        </div>
      </div>
    </div>
    <button @click="getCartList">请求数据1</button>
    <button @click="queryorder">开始抢购</button>
    <button @click="generateorder">ewfewf</button>
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
function getJson({ method = 'POST', url = '', params = {}, token = 'KQCQOLOE2DGBPVJGET675VQSGNY724PDGZDVP3J4M4WHCFZWH2OA1209141',uid = '', host = "/api", headers = {} }) {
  return new Promise((resolve, reject)=>{
    ajax({ method, url, params, token, uid, host, headers }).then(data => resolve(data)).catch(err => reject(err))
  })
}

let detailList = ref([])
let groupon_id = ref('')
function getData(grouponid) {
  getJson({
    method: 'get',
    host: '/api',
    url: '/api/common/getGrouponSpus?groupon_id='+grouponid+'&size=40&page=1&query=&state=1&category=undefined&r='+ Math.random(),
    token: 'db2e42dafb3e4f4468b43756b7ef9e72',
    uid:'usr1647753967996159148'
  }).then((res) => {
      console.log(res.data.lst);
      // 详情列表
      detailList.value  = res.data.lst
  })
}

// 请求购物车
let cartList = ref([])
function getCartList() {
  getJson({
    method: 'post',
    host: '/api',
    url: '/ktt_gateway/shopcart/query?xcx_version=3.24.554',
  }).then((res) => {
     cartList.value = res.result.shop_cart_item_list
  })
}

// 选中的商品
let goods_sku_list = ref([])
let checkparams = ref({})
let params2 = ref({})
function checkuuid(item1){
  let checksku = []
  let checkskuList = document.querySelectorAll('.checksku')
  checkskuList.forEach((item,index)=>{
    let checked = item.checked
    if (checked) {
      let item2= JSON.parse(item.value)
      checksku.push({
          "sku_id": item2.sku_id,
          "goods_id": item2.goods_id,
          "goods_number": item2.amount
      })
    }
  })
  goods_sku_list.value = checksku
  console.log(goods_sku_list.value,11111111);
  params2.value = {
      "activity_no": item1.activity_no,
      "goods_sku_list": goods_sku_list.value
  }
  checkparams.value = {
    "activity_goods_list": [
        params2.value
    ],
    "anti_content": "3akWfxUkMkVEgUSEs9ynf9YnG_yn0dyXpPYX0XaXpBjy0dynY48X0E8O0TqnUzJXpTaXQwUrnF_hjySMf6cwDIxSIzWT7-3k-3tCS-xZytbZzWE57WKT7BK58vW5EAJpS4R_z-1hSh-_8MqVmlHbDpIzV24V28He",
    "timestamp": 1703833159662,
    "sign_version": "sv2",
    "sign": "DC1D819320F787CC036E924127098F28"
  }
}
// 购物车选中的物品
function checkCart() {
  getJson({
    method: 'post',
    host: '/api',
    url: '/ktt_order_core/customer/batch_activity/ordering/shopping_cart?xcx_version=3.24.554',
    params:checkparams.value
  }).then((res) => {

  })
}

// 订单选中详情
function queryorder() {
  getJson({
    method: 'post',
    host: '/api',
    url: 'ktt_order_core/customer/ordering/query_base_info_on_order_page?xcx_version=3.24.556',
    params:{
      "activity_no": params2.value.activity_no,
      "goods_sku_list": params2.value.goods_sku_list,
      "anti_content": "3akAfx5e-eCEg5SEs9yn19YX5PJX5EqXpg8XYEjn5dyj0dynY48X0E8O0Tqn5zJXpTaXQw5rnU_hjyS-fsoWkIxSIzAT7F3eF3tWSFxVytbVzAEZ7AKT7BKZ8vAZEuJpS4R_zF1hShF_8-qCDlHbkpIzC9jwVjaM",
      "timestamp": 1703833161666,
      "sign_version": "sv2",
      "sign": "BEAAD0EB8BA10A60E5EE8483E44741C4"
    }
  }).then((res) => {
  })
}
// 客户检查创建订单
function generateorder() {
  getJson({
    method: 'post',
    host: '/api',
    url: 'ktt_order_core/customer/ordering/generate_biz_order_no?xcx_version=3.24.556',
    params:{
      "anti_content": "3akAfx5e-eCEg5SEs9yn19YX5PJX5EqXpg8XYEjn5dyj0dynY48X0E8O0Tqn5zJXpTaXQw5rnU_hjyS-fsoWkIxSIzAT7F3eF3tWSFxVytbVzAEZ7AKT7BKZ8vAZEuJpS4R_zF1hShF_8-qCDlHbkpIzC9jwVjaM",
      "timestamp": 1703833161666,
      "sign_version": "sv2",
      "sign": "BEAAD0EB8BA10A60E5EE8483E44741C4"
    }
  }).then((res) => {
  })
}

// 客户检查创建订单
function create_order() {
  getJson({
    method: 'post',
    host: '/api',
    url: 'ktt_order_core/customer/ordering/create_order?xcx_version=3.24.556',
    params:{
      "anti_content": "3akAfx5e-eCEg5SEs9yn19YX5PJX5EqXpg8XYEjn5dyj0dynY48X0E8O0Tqn5zJXpTaXQw5rnU_hjyS-fsoWkIxSIzAT7F3eF3tWSFxVytbVzAEZ7AKT7BKZ8vAZEuJpS4R_zF1hShF_8-qCDlHbkpIzC9jwVjaM",
      "timestamp": 1703833161666,
      "sign_version": "sv2",
      "sign": "BEAAD0EB8BA10A60E5EE8483E44741C4"
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
