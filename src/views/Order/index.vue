<template>
  <div class="order">
    <!-- 头部导航区 -->
    <div class="header">
      <van-nav-bar
        title="订单中心"
        left-text="返回"
        left-arrow
        @click-left="onClickLeft"
      />
    </div>
    <!-- 主体内容区 -->
    <div class="main">
      <van-cell-group
        class="order-list"
        v-for="item in orderInfo"
        :key="item.order_id"
        inset="true"
      >
        <van-cell
          class="order-main"
          v-for="index in item.cartInfo"
          :key="index.id"
        >
          <div class="link">
            <img :src="index.productInfo.image" alt="">
            <div class="info">
              <p class="title" v-text="index.productInfo.store_name"></p>
              <p class="price">￥{{ index.truePrice }}</p>
            </div>
          </div>
        </van-cell>
        <div class="order-comment">
          <p class="total">总计：￥{{ item.pay_price }}</p>
          <p class="num" >商品总数：x{{ item.cartInfo.length }}</p>
        </div>
      </van-cell-group>
    </div>
  </div>
</template>

<script setup>
import { getOrderList } from '@/api/order'
import { ref, computed } from '@vue/reactivity'
import { useRouter } from 'vue-router'
const router = useRouter()

// 存储订单列表数据
const orderList = ref({})
const orderInfo = computed(() => orderList.value)
// 获取订单列表数据
const initOrderList = async () => {
  const { data } = await getOrderList()
  console.log(data)
  if(data.status !== 200) { return }
  orderList.value = data.data
}
initOrderList()


// 返回用户页
const onClickLeft = () => {
  router.push({
    name: 'user'
  })
}
</script>

<style lang="scss" scoped>
.order {
  background-color: #f2f2f2;

  .header {
    width: 100%;
    position: fixed !important;
    top: 0;
    z-index: 10001;
  }
  
  .main {
    padding-top: 47px;
    .order-list {
      width: 100%;
      margin: 0 0 15px;

      .order-main {
        width: 100%;
        padding: 0;

        .link {
          display: flex;
          align-items: center;

          img {
            width: 70px;
            height: 70px;
          }
          .info {
            padding-left: 5px;
            .title {
              display: -webkit-box;
              -webkit-box-orient: vertical;
              -webkit-line-clamp: 1;
              overflow: hidden;
              padding: 0 15px 5px 0;
            }
            .price {
              color: #f2270c;
            }
          }
        }
      }

      .order-comment {
        display: flex;
        justify-content: space-between;
        font-size: 14px;
        .total {
          color: #f2270c;
        }

        .num {
          font-size: 16px;
          font-weight: 700;
        }
      }
    }
  }
}
</style>
