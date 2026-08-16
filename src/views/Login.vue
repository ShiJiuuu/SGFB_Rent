<template>
  <div class="login-container">
    <div class="login-blob blob-1"></div>
    <div class="login-blob blob-2"></div>
    <div class="login-blob blob-3"></div>
    <div class="login-card">
      <div class="login-badge">
        <img src="../../SGFBLogo.png" alt="Logo" class="login-logo" />
      </div>
      <h2>影视器材预约系统后台管理</h2>
      <p class="login-hint">（此为后台管理页面，预约请选择"预约申请"页面）</p>
      <form @submit.prevent="handleLogin">
        <div class="form-item">
          <label>用户名：</label>
          <input type="text" v-model="loginForm.username" placeholder="请输入用户名" :class="{ 'error-input': errorMessage }">
        </div>
        <div class="form-item">
          <label>密码：</label>
          <input type="password" v-model="loginForm.password" placeholder="请输入密码" :class="{ 'error-input': errorMessage }">
        </div>
        <transition name="error">
          <div class="error-message" v-if="errorMessage" :key="errorMessage">
            {{ errorMessage }}
          </div>
        </transition>
        <div class="form-item">
          <button type="submit" class="login-btn" :disabled="loading">
            {{ loading ? '登录中...' : '登录' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const loginForm = ref({
  username: '',
  password: ''
})
const loading = ref(false)
const errorMessage = ref('')

const handleLogin = async () => {
  errorMessage.value = ''
  
  if (!loginForm.value.username || !loginForm.value.password) {
    errorMessage.value = '请输入用户名和密码'
    return
  }
  
  loading.value = true
  
  try {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(loginForm.value)
    })
    
    const data = await response.json()
    
    if (data.success) {
      localStorage.setItem('token', data.token)
      localStorage.setItem('user', JSON.stringify(data.user))
      router.push('/borrow-application')
    } else {
      errorMessage.value = data.message || '用户名或密码错误'
    }
  } catch (error) {
    console.error('登录失败:', error)
    errorMessage.value = '登录失败，请稍后重试'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.login-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #1e2a3a 0%, #2d4a6b 45%, #3a6ea5 100%);
  overflow: hidden;
}

.login-blob {
  position: absolute;
  border-radius: 50%;
  filter: blur(60px);
  opacity: 0.45;
  pointer-events: none;
}

.blob-1 {
  width: 420px;
  height: 420px;
  background: radial-gradient(circle, #66b1ff, transparent 70%);
  top: -120px;
  left: -100px;
  animation: float 14s ease-in-out infinite;
}

.blob-2 {
  width: 360px;
  height: 360px;
  background: radial-gradient(circle, #8cc5ff, transparent 70%);
  bottom: -100px;
  right: -80px;
  animation: float 18s ease-in-out infinite reverse;
}

.blob-3 {
  width: 260px;
  height: 260px;
  background: radial-gradient(circle, #a0cfff, transparent 70%);
  bottom: 15%;
  left: 12%;
  animation: float 22s ease-in-out infinite;
  animation-delay: -6s;
}

.login-card {
  position: relative;
  width: 400px;
  padding: 40px;
  background: rgba(255, 255, 255, 0.92);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.6);
  border-radius: 16px;
  box-shadow: 0 20px 50px rgba(15, 40, 80, 0.35);
}

.login-badge {
  display: flex;
  justify-content: center;
  margin-bottom: 14px;
}

.login-logo {
  height: 84px;
  object-fit: contain;
  filter: drop-shadow(0 4px 10px rgba(64, 158, 255, 0.3));
}

.login-card h2 {
  text-align: center;
  margin-bottom: 10px;
  color: #304156;
  font-size: 20px;
}

.login-hint {
  text-align: center;
  font-size: 13px;
  color: #909399;
  margin-bottom: 30px;
  line-height: 1.5;
}

.form-item {
  margin-bottom: 20px;
}

.form-item label {
  display: block;
  margin-bottom: 8px;
  color: #606266;
  font-weight: 500;
}

.form-item input {
  width: 100%;
  padding: 11px 12px;
  border: 1px solid #dcdfe6;
  border-radius: 8px;
  font-size: 14px;
  box-sizing: border-box;
  background: rgba(255, 255, 255, 0.8);
  transition: border-color 0.25s ease, box-shadow 0.25s ease;
}

.form-item input:hover {
  border-color: #c0c4cc;
}

.form-item input:focus {
  outline: none;
  border-color: #409eff;
  box-shadow: 0 0 0 3px rgba(64, 158, 255, 0.18);
}

.form-item input.error-input {
  border-color: #f56c6c;
}

.form-item input.error-input:focus {
  box-shadow: 0 0 0 3px rgba(245, 108, 108, 0.2);
}

.error-message {
  color: #f56c6c;
  font-size: 14px;
  margin-bottom: 20px;
  padding: 8px 12px;
  background: #fef0f0;
  border-radius: 8px;
  border: 1px solid #fde2e2;
  animation: shake 0.4s ease;
}

.error-enter-active,
.error-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
}

.error-enter-from,
.error-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}

.login-btn {
  width: 100%;
  padding: 12px;
  background: linear-gradient(135deg, #409eff 0%, #66b1ff 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  letter-spacing: 4px;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.25, 0.8, 0.35, 1);
}

.login-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #66b1ff 0%, #8cc5ff 100%);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(64, 158, 255, 0.4);
}

.login-btn:active:not(:disabled) {
  transform: translateY(0);
}

.login-btn:disabled {
  background: linear-gradient(135deg, #a0cfff 0%, #b3d8ff 100%);
  cursor: not-allowed;
}

@media screen and (max-width: 768px) {
  .login-card {
    width: calc(100% - 40px);
    padding: 30px 24px;
  }
}
</style>
