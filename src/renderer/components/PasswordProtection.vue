<template>
  <!-- 只有设置了密码才显示密码验证页面 -->
  <div v-if="needPasswordAuth && !isAuthenticated" class="password-overlay">
    <!-- 背景粒子效果 -->
    <div class="particles">
      <div class="particle" v-for="i in 50" :key="i"></div>
    </div>

    <div class="password-modal">
      <!-- 顶部图标 -->
      <div class="modal-icon">
        🔐
      </div>

      <!-- 标题区域 -->
      <div class="modal-header">
        <h2>🎵 音乐播放器</h2>
        <h3>访问验证</h3>
        <p>请输入密码继续访问</p>
      </div>

      <!-- 输入区域 -->
      <div class="input-container">
        <div class="input-wrapper">
          <input
            v-model="password"
            type="password"
            placeholder="请输入访问密码"
            @keyup.enter="checkPassword"
            class="password-input"
          />
          <div class="input-border"></div>
        </div>

        <button @click="checkPassword" class="submit-btn">
          <span class="btn-text">验证访问</span>
          <div class="btn-ripple"></div>
        </button>
      </div>

      <!-- 错误提示 -->
      <div v-if="hasError" class="error-message">
        <span class="error-icon">⚠️</span>
        密码错误，请重试
      </div>

      <!-- 底部装饰 -->
      <div class="modal-footer">
        <div class="security-badge">
          <span class="badge-icon">🛡️</span>
          <span class="badge-text">安全验证</span>
        </div>
      </div>
    </div>
  </div>

  <!-- 主应用内容 -->
  <div v-else>
    <slot />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

// 配置 - 从环境变量获取密码
const getPassword = (): string => {
  return import.meta.env.VITE_PASSWORD || ''; // 没有默认密码
};

// 响应式数据
const isAuthenticated = ref(false);
const needPasswordAuth = ref(false);
const password = ref('');
const hasError = ref(false);

// 检查密码
const checkPassword = () => {
  if (!password.value.trim()) {
    hasError.value = true;
    return;
  }

  const correctPassword = getPassword();

  if (password.value === correctPassword) {
    // 密码正确
    isAuthenticated.value = true;
    hasError.value = false;
    // 保存认证状态，避免刷新后重新输入
    localStorage.setItem('music_auth', 'true');
  } else {
    // 密码错误
    hasError.value = true;
  }

  password.value = '';
};

// 页面加载时检查是否需要密码验证
onMounted(() => {
  const configuredPassword = getPassword();

  // 如果没有配置密码，直接通过
  if (!configuredPassword || configuredPassword.trim() === '') {
    isAuthenticated.value = true;
    needPasswordAuth.value = false;
    return;
  }

  // 有密码配置，需要验证
  needPasswordAuth.value = true;

  // 检查是否已经认证过
  const authStatus = localStorage.getItem('music_auth');
  if (authStatus === 'true') {
    isAuthenticated.value = true;
  }
});
</script>

<style scoped>
/* 简单密码保护样式 */
.password-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  padding: 20px;
  box-sizing: border-box;
  overflow: hidden;
}

/* 背景粒子效果 */
.particles {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.particle {
  position: absolute;
  width: 4px;
  height: 4px;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 50%;
  animation: float 6s ease-in-out infinite;
}

.particle:nth-child(odd) {
  animation-delay: -2s;
  animation-duration: 8s;
}

.particle:nth-child(even) {
  animation-delay: -4s;
  animation-duration: 10s;
}

@keyframes float {
  0%, 100% {
    transform: translateY(100vh) translateX(0px) rotate(0deg);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  100% {
    transform: translateY(-100px) translateX(100px) rotate(360deg);
    opacity: 0;
  }
}

/* 随机位置 */
.particle:nth-child(1) { left: 10%; animation-delay: -1s; }
.particle:nth-child(2) { left: 20%; animation-delay: -2s; }
.particle:nth-child(3) { left: 30%; animation-delay: -3s; }
.particle:nth-child(4) { left: 40%; animation-delay: -4s; }
.particle:nth-child(5) { left: 50%; animation-delay: -5s; }

.password-modal {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 24px;
  padding: 0;
  text-align: center;
  max-width: 420px;
  width: 90%;
  max-height: 80vh;
  overflow: hidden;
  box-shadow:
    0 25px 50px rgba(0, 0, 0, 0.2),
    0 0 0 1px rgba(255, 255, 255, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
  animation: modalSlideIn 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
  position: relative;
}

.password-modal::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg,
    rgba(255, 255, 255, 0.1) 0%,
    rgba(255, 255, 255, 0.05) 100%);
  border-radius: 24px;
  pointer-events: none;
}

@keyframes modalSlideIn {
  from {
    opacity: 0;
    transform: translateY(-30px) scale(0.9) rotateX(10deg);
    filter: blur(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1) rotateX(0deg);
    filter: blur(0px);
  }
}

/* 顶部图标 */
.modal-icon {
  font-size: 48px;
  margin: 30px 0 20px 0;
  animation: iconPulse 2s ease-in-out infinite;
}

@keyframes iconPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

/* 标题区域 */
.modal-header {
  padding: 0 40px 30px 40px;
  position: relative;
  z-index: 1;
}

.modal-header h2 {
  margin: 0 0 15px 0;
  color: white;
  font-size: 24px;
  font-weight: 700;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.modal-header h3 {
  margin: 0 0 8px 0;
  color: rgba(255, 255, 255, 0.9);
  font-size: 18px;
  font-weight: 600;
}

.modal-header p {
  margin: 0;
  color: rgba(255, 255, 255, 0.7);
  font-size: 14px;
  opacity: 0.9;
}

/* 输入容器 */
.input-container {
  padding: 0 40px 40px 40px;
  position: relative;
  z-index: 1;
}

.input-wrapper {
  position: relative;
  margin-bottom: 25px;
}

.input-border {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0;
  height: 2px;
  background: linear-gradient(90deg, #667eea, #764ba2);
  transition: width 0.3s ease;
}

.password-input:focus + .input-border {
  width: 100%;
}

.password-input {
  width: 100%;
  padding: 16px 20px;
  border: none;
  border-bottom: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 12px 12px 0 0;
  font-size: 16px;
  box-sizing: border-box;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  background: rgba(255, 255, 255, 0.1);
  color: white;
  backdrop-filter: blur(10px);
}

.password-input::placeholder {
  color: rgba(255, 255, 255, 0.6);
  transition: all 0.3s ease;
}

.password-input:focus {
  outline: none;
  background: rgba(255, 255, 255, 0.2);
  border-bottom-color: rgba(255, 255, 255, 0.8);
  box-shadow: 0 4px 20px rgba(255, 255, 255, 0.1);
  transform: translateY(-2px);
}

.password-input:focus::placeholder {
  color: rgba(255, 255, 255, 0.4);
  transform: translateY(-20px);
}

.submit-btn {
  position: relative;
  width: 100%;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.2) 0%, rgba(255, 255, 255, 0.1) 100%);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;
  padding: 16px 32px;
  border-radius: 12px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(10px);
  overflow: hidden;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.submit-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(255, 255, 255, 0.2);
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.3) 0%, rgba(255, 255, 255, 0.2) 100%);
}

.submit-btn:active {
  transform: translateY(-1px);
}

.btn-text {
  position: relative;
  z-index: 2;
}

.btn-ripple {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}

.submit-btn:active .btn-ripple {
  width: 300px;
  height: 300px;
}

.error-message {
  color: #ff6b6b;
  font-size: 14px;
  margin-top: 20px;
  padding: 15px 20px;
  background: rgba(255, 107, 107, 0.15);
  border: 1px solid rgba(255, 107, 107, 0.3);
  border-radius: 12px;
  backdrop-filter: blur(10px);
  animation: errorSlideIn 0.5s ease-out, shake 0.6s ease-in-out 0.1s;
  display: flex;
  align-items: center;
  gap: 8px;
}

.error-icon {
  font-size: 16px;
  animation: pulse 1s ease-in-out infinite;
}

@keyframes errorSlideIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-8px); }
  75% { transform: translateX(8px); }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.2); }
}

/* 底部装饰 */
.modal-footer {
  padding: 20px 40px 30px 40px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.05);
}

.security-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  color: rgba(255, 255, 255, 0.6);
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.badge-icon {
  font-size: 14px;
  animation: glow 2s ease-in-out infinite alternate;
}

.badge-text {
  font-weight: 500;
}

@keyframes glow {
  from { opacity: 0.6; }
  to { opacity: 1; }
}

/* 手机适配 */
@media (max-width: 768px) {
  .password-overlay {
    padding: 15px; /* 手机上更小的边距 */
  }

  .password-modal {
    width: 95%;
    padding: 20px;
    max-height: 85vh; /* 手机上可以占更多高度 */
  }

  .password-modal h3 {
    font-size: 16px;
  }

  .password-input {
    font-size: 16px; /* 防止iOS缩放 */
    padding: 12px;
  }

  .submit-btn {
    width: 100%;
    padding: 12px;
    font-size: 16px;
  }
}

@media (max-width: 480px) {
  .password-overlay {
    padding: 10px; /* 小屏幕更紧凑 */
  }

  .password-modal {
    width: 90%;
    padding: 15px;
    max-height: 90vh; /* 小屏幕可以占更多高度 */
  }

  .password-modal h3 {
    font-size: 14px;
  }

  .password-modal p {
    font-size: 12px;
  }
}

/* 横屏手机适配 */
@media (max-height: 500px) and (orientation: landscape) {
  .password-overlay {
    padding: 5px;
  }

  .password-modal {
    max-height: 95vh;
    padding: 10px;
  }

  .password-modal h3 {
    margin-bottom: 5px;
    font-size: 14px;
  }

  .password-modal p {
    margin-bottom: 10px;
    font-size: 12px;
  }

  .password-input {
    margin-bottom: 10px;
    padding: 8px;
  }
}
</style>
