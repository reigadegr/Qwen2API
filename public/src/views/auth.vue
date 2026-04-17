<template>
  <div class="auth-page flex min-h-screen items-center justify-center px-4 py-10">
    <transition name="fade-slide">
      <div
        class="auth-panel flex w-full max-w-xl flex-col rounded-[28px] px-8 py-10 shadow-2xl"
        v-if="showPanel">
        <div class="mb-8">
          <div class="mb-3 inline-flex rounded-full border border-sky-200 bg-sky-50 px-3 py-1 text-xs font-semibold tracking-[0.18em] text-sky-700">QWEN2API CONSOLE</div>
          <h1 class="block text-3xl font-bold text-slate-800">{{ t('auth.title') }}</h1>
          <p class="mt-3 text-sm leading-6 text-slate-500">使用管理 API Key 登录控制台，所有配置与账号操作集中在一个清晰的浅色工作区内。</p>
        </div>
        <input type="text"
          class="auth-input h-14 w-full rounded-2xl border pl-5 placeholder:text-slate-400 focus:outline-none"
          :placeholder="t('auth.placeholder')" v-model="apiKey" @keyup.enter="handleLogin">
        <button class="auth-button mt-6 h-14 w-full rounded-2xl font-semibold text-white transition-all duration-200"
          @click="handleLogin">{{ t('auth.login') }}</button>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import axios from 'axios'
import { useRouter } from 'vue-router'

const { t } = useI18n()
const router = useRouter()
const apiKey = ref('')
const showPanel = ref(false)

const handleLogin = async () => {
  try {
    const res = await axios.post('/verify', {
      apiKey: apiKey.value
    }, {
      headers: {
        'Content-Type': 'application/json'
      }
    })
    if (res.data.status == 200) {
      localStorage.setItem('apiKey', apiKey.value)
      router.push({ path: '/', replace: true })
    } else {
      alert(t('auth.error'))
    }
  } catch (err) {
    alert(t('auth.error'))
  }
}

onMounted(() => {
  setTimeout(() => {
    showPanel.value = true
  }, 80)
})
</script>

<style lang="css" scoped>
.auth-panel {
  border: 1px solid rgba(189, 210, 234, 0.9);
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.96), rgba(247, 251, 255, 0.92));
  box-shadow: 0 30px 70px rgba(41, 92, 145, 0.16);
}

.auth-input {
  border-color: var(--border);
  background: rgba(248, 251, 255, 0.95);
  color: var(--text-primary);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.7);
  transition: border-color 0.2s ease, box-shadow 0.2s ease, background-color 0.2s ease;
}

.auth-input:focus {
  border-color: #68a5e3;
  box-shadow: 0 0 0 4px rgba(102, 161, 224, 0.18);
  background: #ffffff;
}

.auth-button {
  background: linear-gradient(135deg, #3f89d8, #2a6db7);
  box-shadow: 0 18px 30px rgba(48, 104, 170, 0.24);
}

.auth-button:hover {
  background: linear-gradient(135deg, #347dcb, #205f9f);
  transform: translateY(-1px);
}

.fade-slide-enter-active, .fade-slide-leave-active {
  transition: opacity 0.5s, transform 0.5s;
}
.fade-slide-enter-from, .fade-slide-leave-to {
  opacity: 0;
  transform: translateY(40px);
}
.fade-slide-enter-to, .fade-slide-leave-from {
  opacity: 1;
  transform: translateY(0);
}
</style>
