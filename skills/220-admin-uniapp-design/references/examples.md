# UniApp原型开发示例

## 项目结构

```
prototype-uniapp/
├── src/
│   ├── api/                    # API接口（Mock）
│   │   ├── modules/
│   │   │   ├── auth.ts
│   │   │   ├── user.ts
│   │   │   └── order.ts
│   │   └── mock-data/          # Mock JSON数据
│   ├── components/             # 组件
│   │   ├── common/             # 通用组件
│   │   └── business/           # 业务组件
│   ├── pages/                  # 页面
│   │   ├── index/              # 首页
│   │   ├── category/           # 分类
│   │   ├── cart/               # 购物车
│   │   ├── user/               # 用户中心
│   │   └── auth/               # 认证
│   ├── static/                 # 静态资源
│   ├── stores/                 # Pinia状态
│   ├── utils/                  # 工具函数
│   ├── App.vue
│   ├── main.ts
│   ├── manifest.json
│   └── pages.json
├── package.json
└── vite.config.ts
```

## 登录页示例

```vue
<!-- src/pages/auth/login.vue -->
<template>
  <view class="login-container">
    <!-- 状态栏占位 -->
    <view class="status-bar" :style="{ height: statusBarHeight + 'px' }"></view>
    
    <!-- Logo区域 -->
    <view class="logo-section">
      <image class="logo" src="/static/logo.png" mode="aspectFit" />
      <text class="welcome-text">欢迎回来</text>
    </view>
    
    <!-- 表单区域 -->
    <view class="form-section">
      <!-- 手机号输入 -->
      <view class="input-item">
        <text class="input-label">手机号</text>
        <input
          v-model="loginForm.phone"
          type="number"
          maxlength="11"
          placeholder="请输入手机号"
          class="input-field"
        />
      </view>
      
      <!-- 验证码输入 -->
      <view class="input-item">
        <text class="input-label">验证码</text>
        <view class="code-input-wrapper">
          <input
            v-model="loginForm.code"
            type="number"
            maxlength="6"
            placeholder="请输入验证码"
            class="input-field code-input"
          />
          <button
            class="code-btn"
            :disabled="countdown > 0"
            @click="handleGetCode"
          >
            {{ countdown > 0 ? `${countdown}s` : '获取验证码' }}
          </button>
        </view>
      </view>
      
      <!-- 登录按钮 -->
      <button class="login-btn" :loading="loading" @click="handleLogin">
        登 录
      </button>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useUserStore } from '@/stores/modules/user'

const userStore = useUserStore()
const systemInfo = uni.getSystemInfoSync()
const statusBarHeight = ref(systemInfo.statusBarHeight)

const loginForm = reactive({ phone: '', code: '' })
const loading = ref(false)
const countdown = ref(0)

const validatePhone = (phone: string): boolean => {
  return /^1[3-9]\d{9}$/.test(phone)
}

const handleGetCode = async () => {
  if (!validatePhone(loginForm.phone)) {
    uni.showToast({ title: '请输入正确的手机号', icon: 'none' })
    return
  }
  
  try {
    await getCodeApi(loginForm.phone)
    uni.showToast({ title: '验证码已发送', icon: 'success' })
    countdown.value = 60
    const timer = setInterval(() => {
      countdown.value--
      if (countdown.value <= 0) clearInterval(timer)
    }, 1000)
  } catch (error) {
    uni.showToast({ title: '发送失败', icon: 'none' })
  }
}

const handleLogin = async () => {
  if (!validatePhone(loginForm.phone)) {
    uni.showToast({ title: '请输入正确的手机号', icon: 'none' })
    return
  }
  
  if (loginForm.code.length !== 6) {
    uni.showToast({ title: '请输入6位验证码', icon: 'none' })
    return
  }
  
  loading.value = true
  try {
    const res = await loginApi(loginForm)
    userStore.setToken(res.token)
    userStore.setUserInfo(res.user)
    uni.showToast({ title: '登录成功', icon: 'success' })
    uni.switchTab({ url: '/pages/index/index' })
  } catch (error) {
    uni.showToast({ title: '登录失败', icon: 'none' })
  } finally {
    loading.value = false
  }
}
</script>

<style scoped lang="scss">
.login-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  
  .logo-section {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 60rpx 0;
    
    .logo {
      width: 160rpx;
      height: 160rpx;
    }
    
    .welcome-text {
      color: #fff;
      font-size: 36rpx;
      font-weight: 600;
    }
  }
  
  .form-section {
    background: #fff;
    border-radius: 24rpx;
    padding: 48rpx 32rpx;
    margin: 0 32rpx;
  }
  
  .input-item {
    margin-bottom: 32rpx;
    
    .input-label {
      color: #333;
      font-size: 28rpx;
      margin-bottom: 16rpx;
    }
    
    .input-field {
      height: 88rpx;
      background: #f5f5f5;
      border-radius: 12rpx;
      padding: 0 24rpx;
    }
    
    .code-btn {
      width: 200rpx;
      height: 88rpx;
      background: #667eea;
      color: #fff;
      border-radius: 12rpx;
    }
  }
  
  .login-btn {
    height: 96rpx;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: #fff;
    border-radius: 48rpx;
    margin-top: 48rpx;
  }
}
</style>
```

## Mock数据配置

```typescript
// src/api/mock-data/auth.json
{
  "login": {
    "code": 200,
    "data": {
      "token": "mock_token_xxx",
      "user": {
        "id": 1,
        "phone": "13800138000",
        "nickname": "测试用户",
        "avatar": "https://placeholder.com/100"
      }
    }
  },
  "code": {
    "code": 200,
    "message": "发送成功"
  }
}
```

## 验证清单

- [ ] 页面布局与线框图一致
- [ ] 适配不同屏幕尺寸
- [ ] 处理安全区和刘海屏
- [ ] 交互功能正常（点击、滑动、下拉刷新）
- [ ] 页面跳转流畅
- [ ] Mock数据正确显示
- [ ] 在微信开发者工具中正常显示
