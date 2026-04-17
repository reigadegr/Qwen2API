<template>
    <div class="min-h-screen px-4 py-6 md:px-6">
        <div class="mx-auto max-w-7xl">
            <div class="mb-6 flex flex-col justify-between gap-4 rounded-[28px] border border-slate-200/80 bg-white/80 px-5 py-5 shadow-[0_18px_40px_rgba(39,84,132,0.08)] backdrop-blur-sm md:flex-row md:items-center">
                <div>
                    <div class="text-xs font-semibold uppercase tracking-[0.24em] text-sky-700">System Settings</div>
                    <h1 class="mt-2 text-3xl font-bold text-slate-800">{{ t('settings.title') }}</h1>
                    <p class="mt-2 text-sm text-slate-500">统一管理 API Key、刷新策略和控制台行为，避免分散配置造成的维护成本。</p>
                </div>
                <div class="flex items-center space-x-3">
                    <select v-model="locale" @change="onLocaleChange"
                        class="rounded-xl border border-slate-200 bg-slate-50 px-3 py-2 text-sm text-slate-700 shadow-sm transition-all duration-300 focus:border-sky-500 focus:ring-sky-500">
                        <option value="ru">{{ t('lang.ru') }}</option>
                        <option value="zh">{{ t('lang.zh') }}</option>
                    </select>
                    <router-link to="/"
                        class="action-button px-4 py-2 text-center font-semibold">
                        {{ t('settings.backToDash') }}
                    </router-link>
                </div>
            </div>
            <div class="grid grid-cols-1 gap-6">
                <!-- API Key 管理 -->
                <div class="setting-card relative overflow-hidden rounded-2xl p-6 flex flex-col gap-4">
                    <div class="absolute inset-0 rounded-2xl border border-white/60 bg-white/35"></div>
                    <div class="relative flex flex-col gap-4">
                        <label class="text-lg font-semibold text-slate-700">{{ t('settings.apiKeyTitle') }}</label>

                        <!-- 管理员密钥 -->
                        <div class="rounded-2xl border border-amber-200 bg-amber-50/90 p-4">
                            <div class="flex items-center gap-2 mb-2">
                                <span class="text-yellow-600 font-semibold">{{ t('settings.adminKey') }}</span>
                                <span class="text-xs bg-yellow-200 text-yellow-800 px-2 py-1 rounded">{{ t('settings.adminReadonly') }}</span>
                            </div>
                            <input :value="settings.adminKey" type="text" readonly
                                class="h-10 w-full cursor-not-allowed rounded-lg border border-slate-200 bg-white px-3 text-sm text-slate-600 shadow-sm">
                        </div>

                        <!-- 普通密钥列表 -->
                        <div class="space-y-2">
                            <div class="flex items-center justify-between">
                                <span class="font-semibold text-slate-700">{{ t('settings.regularKeys') }}</span>
                                <button @click="showAddKeyModal = true"
                                    class="rounded-lg bg-sky-600 px-3 py-1 text-sm font-medium text-white transition-all hover:bg-sky-700">
                                    {{ t('settings.addKey') }}
                                </button>
                            </div>

                            <div v-if="settings.regularKeys.length === 0" class="py-4 text-center text-slate-500">
                                {{ t('settings.noKeys') }}
                            </div>

                            <div v-for="(key, index) in settings.regularKeys" :key="index"
                                class="flex items-center gap-2 rounded-xl border border-slate-200 bg-slate-50/80 p-3">
                                <input :value="key" type="text" readonly
                                    class="h-8 flex-1 rounded-lg border border-slate-200 bg-white px-3 text-sm shadow-sm">
                                <button @click="deleteRegularKey(index)"
                                    class="rounded-lg bg-rose-600 px-3 py-1 text-sm text-white transition-all hover:bg-rose-700">
                                    {{ t('settings.delete') }}
                                </button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- 其他设置项 -->
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- 自动刷新 -->
                    <div class="setting-card relative overflow-hidden rounded-2xl p-6 flex flex-col gap-4">
                        <div class="absolute inset-0 rounded-2xl border border-white/60 bg-white/35">
                        </div>
                        <div class="relative flex flex-col gap-2">
                            <label class="font-semibold text-slate-700">{{ t('settings.autoRefresh') }}</label>
                            <div class="flex items-center gap-2">
                                <input v-model="settings.autoRefresh" type="checkbox"
                                class="h-5 w-5 rounded border-slate-300 text-sky-600 focus:ring-sky-500">
                                <span>{{ t('settings.enableAutoRefresh') }}</span>
                            </div>
                            <label class="text-slate-600">{{ t('settings.refreshInterval') }}</label>
                            <input v-model.number="settings.autoRefreshInterval" type="number"
                                class="mt-1 block h-12 w-full rounded-xl border border-slate-200 bg-slate-50/80 px-4 text-base shadow-sm transition-all duration-300 focus:border-sky-500 focus:ring-sky-500">
                            <button @click="saveAutoRefresh"
                                class="save-button mt-2 w-full rounded-xl py-2.5 font-semibold">{{ t('settings.save') }}</button>
                        </div>
                    </div>
                    <!-- 批量登录并发数 -->
                    <div class="setting-card relative overflow-hidden rounded-2xl p-6 flex flex-col gap-4">
                        <div class="absolute inset-0 rounded-2xl border border-white/60 bg-white/35">
                        </div>
                        <div class="relative flex flex-col gap-2">
                            <label class="font-semibold text-slate-700">{{ t('settings.batchConcurrency') }}</label>
                            <label class="text-slate-600">{{ t('settings.batchConcurrencyDesc') }}</label>
                            <input v-model.number="settings.batchLoginConcurrency" type="number" min="1" max="20"
                                class="mt-1 block h-12 w-full rounded-xl border border-slate-200 bg-slate-50/80 px-4 text-base shadow-sm transition-all duration-300 focus:border-sky-500 focus:ring-sky-500">
                            <span class="text-xs text-slate-500">{{ t('settings.batchConcurrencyHint') }}</span>
                            <button @click="saveBatchLoginConcurrency"
                                class="save-button mt-2 w-full rounded-xl py-2.5 font-semibold">{{ t('settings.save') }}</button>
                        </div>
                    </div>
                    <!-- 思考输出 -->
                    <div class="setting-card relative overflow-hidden rounded-2xl p-6 flex flex-col gap-4">
                        <div class="absolute inset-0 rounded-2xl border border-white/60 bg-white/35">
                        </div>
                        <div class="relative flex flex-col gap-2">
                            <label class="font-semibold text-slate-700">{{ t('settings.thinkOutput') }}</label>
                            <div class="flex items-center gap-2">
                                <input v-model="settings.outThink" type="checkbox"
                                class="h-5 w-5 rounded border-slate-300 text-sky-600 focus:ring-sky-500">
                                <span>{{ t('settings.enableThinkOutput') }}</span>
                            </div>
                            <button @click="saveOutThink"
                                class="save-button mt-2 w-full rounded-xl py-2.5 font-semibold">{{ t('settings.save') }}</button>
                        </div>
                    </div>
                    <!-- 搜索信息模式 -->
                    <div class="setting-card relative overflow-hidden rounded-2xl p-6 flex flex-col gap-4">
                        <div class="absolute inset-0 rounded-2xl border border-white/60 bg-white/35">
                        </div>
                        <div class="relative flex flex-col gap-2">
                            <label class="font-semibold text-slate-700">{{ t('settings.searchMode') }}</label>
                            <select v-model="settings.searchInfoMode"
                                class="mt-1 block h-12 w-full rounded-xl border border-slate-200 bg-slate-50/80 px-4 text-base shadow-sm transition-all duration-300 focus:border-sky-500 focus:ring-sky-500">
                                <option value="table">{{ t('settings.searchTable') }}</option>
                                <option value="text">{{ t('settings.searchText') }}</option>
                            </select>
                            <button @click="saveSearchInfoMode"
                                class="save-button mt-2 w-full rounded-xl py-2.5 font-semibold">{{ t('settings.save') }}</button>
                        </div>
                    </div>
                    <!-- 简化模型映射 -->
                    <div class="setting-card relative overflow-hidden rounded-2xl p-6 flex flex-col gap-4">
                        <div class="absolute inset-0 rounded-2xl border border-white/60 bg-white/35">
                        </div>
                        <div class="relative flex flex-col gap-2">
                            <label class="font-semibold text-slate-700">{{ t('settings.simpleModelMap') }}</label>
                            <div class="flex items-center gap-2">
                                <input v-model="settings.simpleModelMap" type="checkbox"
                                class="h-5 w-5 rounded border-slate-300 text-sky-600 focus:ring-sky-500">
                                <span>{{ t('settings.simpleModelMapDesc') }}</span>
                            </div>
                            <button @click="saveSimpleModelMap"
                                class="save-button mt-2 w-full rounded-xl py-2.5 font-semibold">{{ t('settings.save') }}</button>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 添加API Key模态框 -->
            <div v-if="showAddKeyModal"
                class="fixed inset-0 z-50 flex items-center justify-center bg-slate-900/28 px-4 backdrop-blur-sm">
                <div class="w-96 max-w-[90vw] rounded-3xl border border-slate-200 bg-white p-6 shadow-[0_30px_60px_rgba(40,79,123,0.16)]">
                    <h3 class="mb-4 text-lg font-semibold text-slate-800">{{ t('settings.addKeyTitle') }}</h3>
                    <input v-model="newApiKey" type="text" :placeholder="t('settings.addKeyPlaceholder')"
                        class="mb-4 h-10 w-full rounded-lg border border-slate-200 bg-slate-50 px-3 text-sm shadow-sm">
                    <div class="flex gap-2 justify-end">
                        <button @click="showAddKeyModal = false"
                            class="rounded-lg bg-slate-200 px-4 py-2 text-slate-700 transition-all hover:bg-slate-300">
                            {{ t('settings.cancel') }}
                        </button>
                        <button @click="addRegularKey"
                            class="rounded-lg bg-sky-600 px-4 py-2 text-white transition-all hover:bg-sky-700">
                            {{ t('settings.add') }}
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import axios from 'axios'

const { t, locale } = useI18n()

const settings = ref({
    apiKey: localStorage.getItem('apiKey'),
    adminKey: '',
    regularKeys: [],
    defaultHeaders: '',
    defaultCookie: '',
    autoRefresh: false,
    autoRefreshInterval: 21600,
    batchLoginConcurrency: 5,
    outThink: false,
    searchInfoMode: 'table',
    simpleModelMap: false
})

const showAddKeyModal = ref(false)
const newApiKey = ref('')

const onLocaleChange = () => {
    localStorage.setItem('locale', locale.value)
}

const loadSettings = async () => {
    try {
        const res = await axios.get('/api/settings', {
            headers: {
                'Authorization': localStorage.getItem('apiKey')
            }
        })
        settings.value.apiKey = res.data.apiKey
        settings.value.adminKey = res.data.adminKey || ''
        settings.value.regularKeys = res.data.regularKeys || []
        settings.value.defaultHeaders = JSON.stringify(res.data.defaultHeaders)
        settings.value.defaultCookie = res.data.defaultCookie
        settings.value.autoRefresh = res.data.autoRefresh
        settings.value.autoRefreshInterval = res.data.autoRefreshInterval
        settings.value.batchLoginConcurrency = res.data.batchLoginConcurrency
        settings.value.outThink = res.data.outThink
        settings.value.searchInfoMode = res.data.searchInfoMode
        settings.value.simpleModelMap = res.data.simpleModelMap
    } catch (error) {
        console.error('loadSettings error:', error)
    }
}

const saveApiKey = async () => {
    try {
        await axios.post('/api/setApiKey', { apiKey: settings.value.apiKey }, {
            headers: { 'Authorization': localStorage.getItem('apiKey') || '' }
        })
        alert(t('smsg.apiKeySaved'))
    } catch (error) {
        alert(t('smsg.apiKeyFailed') + error.message)
    }
}
const saveAutoRefresh = async () => {
    try {
        await axios.post('/api/setAutoRefresh', {
            autoRefresh: settings.value.autoRefresh,
            autoRefreshInterval: settings.value.autoRefreshInterval
        }, {
            headers: { 'Authorization': localStorage.getItem('apiKey') || '' }
        })
        alert(t('smsg.autoRefreshSaved'))
    } catch (error) {
        alert(t('smsg.autoRefreshFailed') + error.message)
    }
}
const saveBatchLoginConcurrency = async () => {
    try {
        await axios.post('/api/setBatchLoginConcurrency', {
            batchLoginConcurrency: settings.value.batchLoginConcurrency
        }, {
            headers: { 'Authorization': localStorage.getItem('apiKey') || '' }
        })
        alert(t('smsg.batchSaved'))
    } catch (error) {
        alert(t('smsg.batchFailed') + error.message)
    }
}
const saveOutThink = async () => {
    try {
        await axios.post('/api/setOutThink', { outThink: settings.value.outThink }, {
            headers: { 'Authorization': localStorage.getItem('apiKey') || '' }
        })
        alert(t('smsg.thinkSaved'))
    } catch (error) {
        alert(t('smsg.thinkFailed') + error.message)
    }
}
const saveSearchInfoMode = async () => {
    try {
        await axios.post('/api/search-info-mode', { searchInfoMode: settings.value.searchInfoMode }, {
            headers: { 'Authorization': localStorage.getItem('apiKey') || '' }
        })
        alert(t('smsg.searchModeSaved'))
    } catch (error) {
        alert(t('smsg.searchModeFailed') + error.message)
    }
}
const saveSimpleModelMap = async () => {
    try {
        await axios.post('/api/simple-model-map', { simpleModelMap: settings.value.simpleModelMap }, {
            headers: { 'Authorization': localStorage.getItem('apiKey') || '' }
        })
        alert(t('smsg.simpleMapSaved'))
    } catch (error) {
        alert(t('smsg.simpleMapFailed') + error.message)
    }
}

// API Key 管理相关函数
const addRegularKey = async () => {
    if (!newApiKey.value.trim()) {
        alert(t('smsg.enterKey'))
        return
    }

    try {
        await axios.post('/api/addRegularKey', { apiKey: newApiKey.value.trim() }, {
            headers: { 'Authorization': localStorage.getItem('apiKey') || '' }
        })
        alert(t('smsg.keyAdded'))
        newApiKey.value = ''
        showAddKeyModal.value = false
        await loadSettings()
    } catch (error) {
        alert(t('smsg.keyAddFailed') + error.message)
    }
}

const deleteRegularKey = async (index) => {
    if (!confirm(t('smsg.confirmDeleteKey'))) return

    const keyToDelete = settings.value.regularKeys[index]
    try {
        await axios.post('/api/deleteRegularKey', { apiKey: keyToDelete }, {
            headers: { 'Authorization': localStorage.getItem('apiKey') || '' }
        })
        alert(t('smsg.keyDeleted'))
        await loadSettings()
    } catch (error) {
        alert(t('smsg.keyDeleteFailed') + error.message)
    }
}

onMounted(() => {
    loadSettings()
})
</script>

<style lang="css" scoped>
.setting-card {
    background: linear-gradient(180deg, rgba(255, 255, 255, 0.92), rgba(246, 250, 255, 0.9));
    border: 1px solid rgba(214, 228, 243, 0.92);
    box-shadow: 0 16px 32px rgba(35, 78, 124, 0.08);
    transition: box-shadow 0.25s ease, transform 0.25s ease, border-color 0.25s ease;
    position: relative;
}

.setting-card:hover {
    box-shadow: 0 22px 40px rgba(35, 78, 124, 0.12);
    border-color: rgba(151, 189, 227, 0.92);
    transform: translateY(-2px);
}

.action-button {
    border: 1px solid rgba(151, 189, 227, 0.8);
    background: linear-gradient(180deg, rgba(239, 247, 255, 0.96), rgba(222, 237, 252, 0.96));
    color: #174f89;
    box-shadow: 0 8px 20px rgba(54, 108, 168, 0.1);
}

.action-button:hover {
    border-color: rgba(94, 152, 214, 0.8);
    background: linear-gradient(180deg, rgba(230, 242, 255, 1), rgba(214, 233, 252, 1));
}

.save-button {
    border: 1px solid rgba(45, 111, 183, 0.9);
    background: linear-gradient(135deg, #3d86d3, #2c6fb7);
    color: #ffffff;
    box-shadow: 0 14px 30px rgba(46, 104, 171, 0.18);
}

.save-button:hover {
    background: linear-gradient(135deg, #327bc8, #225e9e);
}
</style>
