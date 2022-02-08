<template>
  <!-- 顶部导航 -->
  <van-nav-bar title="购物车" ></van-nav-bar>

  <!-- 购物车列表 -->
  <div class="cart-list" v-if="hasItem">
    <cart-item
      v-for="item in cartList"
      :key="item.id"
      :itemData="item"
    />
  </div>
  <!-- 当购物车为空时 -->
  <van-empty v-else description="购物车还没有商品哦~" />
  <!-- 总计区域 -->
  <van-submit-bar
    :price="store.getters['cart/totalPrice'] * 100"
    button-text="去结算"
    @submit="handleClick"
  >
    <van-checkbox v-model="checkedAll" checked-color="#ee0a24">全选</van-checkbox>
  </van-submit-bar>
  <!-- 公共资源 -->
  <layout-footer></layout-footer>
</template>

<script setup>
import LayoutFooter from '@/components/LayoutFooter.vue'
import CartItem from './components/CartItem.vue'
import { computed } from '@vue/reactivity'
import { useStore } from 'vuex'
import { nextTick } from '@vue/runtime-core'
import { useRouter } from 'vue-router'
const store = useStore()

// 引入接口
import { getCartList } from '@/api/cart'

// 1列表数据处理
// 存储数据
const cartList = computed(() => store.state.cart.cartList)
// 检测购物车是否为空
const hasItem = computed(() => cartList.value.length !== 0)

// 初始化购物车数据
const initCartList = async () => {
  const { data } = await getCartList()
  if (data.status !== 200) { return }

  // 请求新数据后，先将原始数据清空
  store.commit('cart/clear')
  
  await nextTick()

  // 数据处理
  data.data.valid.forEach(item => {
    // 提交给 addToCart 的值必须符合要求
    store.commit('cart/addToCart', {
      id: item.id,
      checked: true,
      count: item.cart_num,
      image: item.productInfo.image,
      title: item.productInfo.store_name,
      price: item.truePrice,
      stock: item.trueStock,
      productId: item.product_id
    })
  })
}
initCartList()

const checkedAll = computed({
  get: () => store.getters['cart/checkedAll'],
  set (newStatus) {
    store.commit('cart/changeAll', { checked: newStatus })
  }
})

const router = useRouter()
const handleClick = () => {
  router.push({
    name: 'order-confirm',
    params: {
      cartId: store.getters['cart/checkedItems'].map(item => item.id).toString()
    }
  })
}
</script>

<style lang="scss" scoped>
// 导航区域
.van-nav-bar {
  position: fixed !important;
  width: 100%;
  top: 0;
}

// 列表区域
.cart-list {
  margin: 50px 0 100px;
}

// 总计区域
.van-submit-bar {
  bottom: 48px;
}
</style>
