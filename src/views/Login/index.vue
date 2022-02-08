<template>
  <van-form @submit="submitHandle">
    <img class="logo" :src="state.logoUrl" alt="">
    <van-cell-group inset>
      <van-field
        v-model="state.username"
        name="用户名"
        label="用户名"
        placeholder="用户名"
      />
      <van-field
        v-if="state.isPassword"
        v-model="state.password"
        type="password"
        name="密码"
        label="密码"
        placeholder="密码"
      />
      <van-field
        v-else
        v-model="state.captcha"
        center
        clearable
        name="验证码"
        label="短信验证码"
        placeholder="短信验证码"
      >
        <template #button>
          <van-button
            size="small"
            type="primary"
            @click="sendCaptcha"
          >
            {{ state.currentText }}
          </van-button>
        </template>
      </van-field>
    </van-cell-group>
    <div style="margin: 16px;">
      <van-button round block type="primary" native-type="submit">
        登录
      </van-button>
      <span
        class="change-button"
        v-text="state.changeText"
        @click="ChangeMode"
      ></span>
    </div>
  </van-form>
</template>

<script setup>
import { reactive, computed } from '@vue/reactivity'
import { Toast } from 'vant'
import { useCountDown } from '@vant/use'
import { useStore } from 'vuex'
const store = useStore()
import { useRouter, useRoute } from 'vue-router'
const router = useRouter()
const route = useRoute()


const state = reactive({
  // 'captcha' 登录模式
  loginMode: 'password',
  // 检测是否为密码登录模式
  isPassword: computed(() => state.loginMode === 'password'),
  // 切换按钮文本处理
  changeText: computed(() => state.isPassword ? '快速登录' : '密码登录'),
  username: '17201234567',
  password: 'qwer1234',
  captcha: '',
  // 存储发送状态，用于控制显示效果
  isSend: false,
  // 倒计时实例
  countDown: null,
  // 根据状态设置要显示的内容
  currentText: computed(() => state.isSend ? state.countDown.seconds : '发送验证码'),
  // logo地址
  logoUrl: ''
})

// 切换登录模式处理
const ChangeMode = () => {
  state.loginMode = state.isPassword ? 'captcha' : 'password'
  // 清除密码和验证码的内容
  state.password = ''
  state.captcha = ''
}

// 验证码处理
import { getVerifyCode, getVerify, loginByPassword, loginByCaptcha } from '@/api/user'
// 发送验证码请求
const sendCaptcha = async () => {
  // 手机号规则校验
  if (!/^1\d{10}$/.test(state.username)) {
    return Toast('请检查用户名')
  }

  // 发送校验请求
  const { data: v1 } = await getVerifyCode()
  if (v1.status !== 200) { return }
  console.log(v1)

  // 发送验证码请求
  const { data: v2 } = await getVerify({
    type: 'login',
    phone: state.username,
    key: v1.data.key
  })
  if (v2.status !== 200) { Toast('网络开小差了，请稍后再试') }

  // 验证码发送完成后设置倒计时实例
  const countDown = useCountDown({
    time: 60 * 1000,
    onFinish () {
      state.isSend = false
    }
  })
  // 开启倒计时
  countDown.start()
  state.countDown = countDown.current
  // 更改发送状态
  state.isSend = true
}

// 登录处理
const submitHandle = async () => {
  // 用户名检测
  const username = state.username.trim()
  if (username === '') { return Toast('请检测用户名') }

  // 统一存储响应结果
  let data = ''
  if (state.isPassword) {
    // 1.密码模式
    const password = state.password.trim()
    if (password === '') { return Toast('请检查密码') }
    ({ data } = await loginByPassword({
      account: username,
      password
    }))
  } else {
    // 验证码模式
    const captcha = state.captcha.trim()
    if (captcha === ''){ return Toast('请检查验证码') }
    ({ data } = await loginByCaptcha({
      phone: username,
      captcha
    }))
  }
  // 接受到响应数据
  if (data.status !== 200) { return Toast('用户名或密码错误') }
  // 成功时，通过mutation 提交新的token 
  store.commit('user/setUser', data.data.token)
  // 跳转页面
  router.push(route.query.redirect ?? '/user')
}

// 头像处理
import { getLogo } from '@/api/index'
const loadLogo = async () => {
  const { data } = await getLogo()
  if (data.status !== 200) { return }
  state.logoUrl = data.data.logo_url
}
loadLogo()

</script>

<style lang="scss" scoped>
.van-form {
  display: flex;
  flex-direction: column;
  // align-items: center;
}
.logo {
  width: 100%;
  align-self: center;
  margin: 30px 0 10px;
}
.change-button {
  display: block;
  text-align: center;
  font-size: 14px;
  color: #999;
  margin-top: 10px;
}
</style>
