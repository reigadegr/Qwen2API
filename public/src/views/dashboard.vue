<template>
  <div class="dashboard-scroll min-h-screen overflow-y-auto px-4 py-6 md:px-6">
    <div class="mx-auto max-w-7xl">
      <div class="dashboard-hero mb-6 flex flex-col justify-between gap-5 px-5 py-5 md:flex-row md:items-center">
        <div>
          <div class="text-xs font-semibold uppercase tracking-[0.24em] text-sky-700">Operations Console</div>
          <h1 class="mt-2 text-4xl font-bold text-slate-800">{{ t('dash.title') }} <span class="ml-2 text-sm font-medium text-slate-400">by 兜豆子</span></h1>
          <p class="mt-2 text-sm text-slate-500">账号维护、批量刷新和导出操作集中在同一工作台，减少误操作和无效视觉干扰。</p>
        </div>
        <div class="grid w-full grid-cols-2 gap-3 sm:flex sm:w-auto sm:flex-row sm:flex-wrap">
          <button @click="showAddModal = true"
                  class="toolbar-btn toolbar-btn--primary">
            {{ t('dash.addAccount') }}
          </button>
          <button @click="refreshAllAccounts"
                  :disabled="isRefreshingAll"
                  :class="[
                    'toolbar-btn toolbar-btn--accent',
                    isRefreshingAll ? 'toolbar-btn--busy cursor-not-allowed' : ''
                  ]">
            <span v-if="isRefreshingAll" class="flex items-center space-x-2">
              <svg class="animate-spin h-4 w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="m4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              <span>{{ t('dash.refreshing') }}</span>
            </span>
            <span v-else>{{ t('dash.refreshAll') }}</span>
          </button>
          <button @click="forceRefreshAllAccounts"
                  :disabled="isForceRefreshingAll"
                  :class="[
                    'toolbar-btn toolbar-btn--warning',
                    isForceRefreshingAll ? 'toolbar-btn--busy cursor-not-allowed' : ''
                  ]">
            <span v-if="isForceRefreshingAll" class="flex items-center space-x-2">
              <svg class="animate-spin h-4 w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="m4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              <span>{{ t('dash.forceRefreshing') }}</span>
            </span>
            <span v-else>{{ t('dash.forceRefresh') }}</span>
          </button>
          <button @click="exportAccounts"
                  class="toolbar-btn toolbar-btn--neutral">
            {{ t('dash.export') }}
          </button>
          <router-link to="/settings"
                       class="toolbar-btn toolbar-btn--ghost col-span-2 sm:col-span-1">
            {{ t('dash.settings') }}
          </router-link>
        </div>
      </div>

      <!-- 分页控制区 -->
      <div class="info-strip mb-4 flex flex-col justify-between gap-3 px-4 py-4 md:flex-row md:items-center">
        <div class="flex items-center space-x-2">
          <span class="text-slate-600">{{ t('dash.perPage') }}</span>
          <select v-model="pageSize" @change="changePageSize" class="rounded-xl border border-slate-200 bg-white px-3 py-2 shadow-sm focus:border-sky-500 focus:ring-sky-500 transition-all duration-300">
            <option :value="10">10</option>
            <option :value="20">20</option>
            <option :value="50">50</option>
            <option :value="100">100</option>
            <option :value="200">200</option>
          </select>
        </div>
        <div class="flex space-x-2 items-center">
          <span class="text-slate-600">{{ t('dash.totalItems', { n: totalItems }) }}</span>
          <button
            @click="changePage(currentPage - 1)"
            :disabled="currentPage === 1"
            :class="[
              'rounded-xl px-3 py-2 transition-all duration-300',
              currentPage === 1 ? 'bg-slate-100 text-slate-400 cursor-not-allowed' : 'bg-sky-50 text-sky-700 hover:bg-sky-100'
            ]"
          >
            {{ t('dash.prevPage') }}
          </button>
          <span class="text-slate-600">{{ currentPage }}/{{ totalPages }}</span>
          <button
            @click="changePage(currentPage + 1)"
            :disabled="currentPage === totalPages || totalPages === 0"
            :class="[
              'rounded-xl px-3 py-2 transition-all duration-300',
              currentPage === totalPages || totalPages === 0 ? 'bg-slate-100 text-slate-400 cursor-not-allowed' : 'bg-sky-50 text-sky-700 hover:bg-sky-100'
            ]"
          >
            {{ t('dash.nextPage') }}
          </button>
        </div>
      </div>

      <!-- 多选操作区 -->
      <div class="info-strip mb-4 flex flex-col justify-between gap-3 px-4 py-4 md:flex-row md:items-center">
        <div class="flex items-center space-x-3">
          <label class="inline-flex items-center cursor-pointer group">
            <div class="relative">
              <input type="checkbox"
                    v-model="selectAll"
                    @change="toggleSelectAll"
                    class="sr-only peer">
              <div class="w-6 h-6 rounded-lg border-2 border-slate-300 bg-white transition-all duration-300 flex items-center justify-center peer-checked:border-sky-600 peer-checked:bg-sky-600">
                <svg v-show="selectAll" class="w-4 h-4 text-white" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="20 6 9 17 4 12"></polyline>
                </svg>
              </div>
            </div>
            <span class="ml-2 text-slate-600 group-hover:text-sky-700 transition-colors duration-200">{{ t('dash.selectAll') }}</span>
          </label>
          <button
            @click="deleteSelected"
            :disabled="selectedTokens.length === 0"
            :class="[
              'flex items-center space-x-1 rounded-xl border px-4 py-2 transition-all duration-300',
              selectedTokens.length === 0 ? 'cursor-not-allowed border-slate-200 bg-slate-100 text-slate-400' : 'border-rose-200 bg-rose-50 text-rose-700 hover:bg-rose-100'
            ]"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
            </svg>
            <span>{{ t('dash.deleteSelected', { n: selectedTokens.length }) }}</span>
          </button>
        </div>
        <button
          @click="showDeleteAllConfirm = true"
          class="flex items-center space-x-1 rounded-xl border border-rose-300 bg-rose-50 px-4 py-2 text-rose-700 transition-all duration-300 hover:bg-rose-100"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
          </svg>
          <span>{{ t('dash.deleteAll') }}</span>
        </button>
      </div>

      <!-- Token列表 -->
      <div class="max-h-[calc(75vh)] overflow-y-auto pr-2 scrollbar-hidden">
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 p-4">
          <div v-for="token in displayedTokens"
               :key="token.email"
               class="token-card group relative overflow-hidden rounded-[24px] pt-4 transition-all duration-300"
               :class="{'ring-2 ring-sky-500 ring-opacity-75': isSelected(token.email)}">
            <div class="absolute top-3 left-3 z-10">
              <label class="custom-checkbox cursor-pointer">
                <input type="checkbox"
                       :checked="isSelected(token.email)"
                       @change="toggleSelect(token.email)"
                       class="sr-only peer">
                <div class="checkbox-icon flex h-6 w-6 items-center justify-center rounded-lg border-2 border-slate-300 bg-white shadow-sm transition-all duration-300 hover:shadow peer-checked:border-sky-600 peer-checked:bg-sky-600">
                  <svg v-show="isSelected(token.email)" class="w-4 h-4 text-white transform scale-0 peer-checked:scale-100 transition-transform duration-300" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
                    <polyline points="20 6 9 17 4 12"></polyline>
                  </svg>
                </div>
              </label>
            </div>
            <div class="absolute inset-0 border border-white/70 bg-white/88"></div>
            <div class="relative p-6 flex flex-col gap-4">
              <div class="flex flex-col space-y-3">
                <div class="field-row relative flex items-center rounded-xl px-3 py-2">
                  <div class="overflow-x-auto scrollbar-hide flex-1 flex items-center space-x-2">
                    <span class="field-label">Email</span>
                    <span class="field-value whitespace-nowrap text-left">{{ token.email }}</span>
                  </div>
                  <button @click="copyToClipboard(token.email)" class="copy-button absolute right-2 rounded-lg px-2 py-1 text-xs font-medium">复制</button>
                </div>
                <div class="field-row relative flex items-center rounded-xl px-3 py-2">
                  <div class="overflow-x-auto scrollbar-hide flex-1 flex items-center space-x-2">
                    <span class="field-label">Password</span>
                    <span class="field-value whitespace-nowrap text-left">{{ token.password }}</span>
                  </div>
                  <button @click="copyToClipboard(token.password)" class="copy-button absolute right-2 rounded-lg px-2 py-1 text-xs font-medium">复制</button>
                </div>
                <div class="field-row relative flex items-center rounded-xl px-3 py-2">
                  <div class="overflow-x-auto scrollbar-hide flex-1 flex items-center space-x-2">
                    <span class="field-label">Token</span>
                    <span class="field-value whitespace-nowrap text-left text-sm">{{ token.token }}</span>
                  </div>
                  <button @click="copyToClipboard(token.token)" class="copy-button absolute right-2 rounded-lg px-2 py-1 text-xs font-medium">复制</button>
                </div>
                <div class="field-row relative flex items-center rounded-xl px-3 py-2">
                  <div class="overflow-x-auto scrollbar-hide flex-1 flex items-center space-x-2">
                    <span class="field-label">Expire</span>
                    <span class="field-value whitespace-nowrap text-left">{{ new Date(token.expires * 1000).toLocaleString() }}</span>
                  </div>
                  <button @click="copyToClipboard(new Date(token.expires * 1000).toLocaleString())" class="copy-button absolute right-2 rounded-lg px-2 py-1 text-xs font-medium">复制</button>
                </div>
              </div>

              <div class="mt-auto space-y-2 border-t border-slate-200/80 pt-4">
                <button @click="refreshToken(token.email)"
                        :disabled="refreshingTokens.includes(token.email)"
                        :class="[
                          'token-action token-action--primary',
                          refreshingTokens.includes(token.email) ? 'toolbar-btn--busy cursor-not-allowed' : ''
                        ]">
                  <span v-if="refreshingTokens.includes(token.email)" class="flex items-center space-x-2">
                    <svg class="animate-spin h-4 w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                      <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                      <path class="opacity-75" fill="currentColor" d="m4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <span>{{ t('dash.refreshing') }}</span>
                  </span>
                  <span v-else>{{ t('dash.refreshToken') }}</span>
                </button>
                <button @click="deleteToken(token.email)"
                        class="token-action token-action--danger">
                  {{ t('dash.deleteAccount') }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 删除全部确认对话框 -->
    <div v-if="showDeleteAllConfirm"
         class="fixed inset-0 z-50 flex items-center justify-center bg-slate-900/30 backdrop-blur-sm"
         @click.self="showDeleteAllConfirm = false">
      <div class="relative w-11/12 max-w-md rounded-[28px] border border-slate-200 bg-white p-6 shadow-[0_30px_60px_rgba(40,79,123,0.16)] transform transition-all duration-300 scale-100 opacity-100">
        <h2 class="mb-4 text-2xl font-bold text-rose-700">{{ t('dash.dangerTitle') }}</h2>
        <p class="mb-6 text-slate-600">{{ t('dash.dangerText', { n: totalItems }) }}</p>
        <div class="flex justify-end space-x-4">
          <button @click="showDeleteAllConfirm = false"
                  class="rounded-xl bg-slate-100 px-4 py-2 transition-all duration-300 hover:bg-slate-200">
            {{ t('dash.cancel') }}
          </button>
          <button @click="deleteAllAccounts"
                  class="rounded-xl bg-rose-600 px-4 py-2 text-white transition-all duration-300 hover:bg-rose-700">
            {{ t('dash.confirmDelete') }}
          </button>
        </div>
      </div>
    </div>

    <!-- 添加账号模态框 -->
    <div v-if="showAddModal"
         class="fixed inset-0 z-50 overflow-y-auto bg-slate-900/30 px-4 py-6 backdrop-blur-sm"
          @click.self="closeAddModal">
      <div :class="[
            'relative mx-auto flex max-h-[calc(100vh-3rem)] w-full flex-col overflow-hidden rounded-[28px] border border-slate-200 bg-white p-6 shadow-[0_30px_60px_rgba(40,79,123,0.16)] transform transition-all duration-300 scale-100 opacity-100',
            addMode === 'batch' ? 'max-w-3xl' : 'max-w-md'
          ]">
        <div class="mb-6 flex shrink-0 border-b border-slate-200">
          <button :class="['flex-1 rounded-t-xl py-2 font-bold transition-all duration-300', addMode==='single' ? 'border-b-2 border-sky-600 bg-sky-50 text-sky-700' : 'bg-transparent text-slate-500', isBatchAdding ? 'cursor-not-allowed opacity-50' : '']" @click="!isBatchAdding && (addMode='single')">{{ t('dash.singleAdd') }}</button>
          <button :class="['flex-1 rounded-t-xl py-2 font-bold transition-all duration-300', addMode==='batch' ? 'border-b-2 border-sky-600 bg-sky-50 text-sky-700' : 'bg-transparent text-slate-500', isBatchAdding ? 'cursor-not-allowed opacity-50' : '']" @click="!isBatchAdding && (addMode='batch')">{{ t('dash.batchAdd') }}</button>
        </div>
        <div class="modal-scroll min-h-0 flex-1 overflow-y-auto pr-2">
        <transition name="fade" mode="out-in">
          <div v-if="addMode==='single'" key="single" class="pr-1">
            <h2 class="mb-4 text-xl font-bold text-slate-800">{{ t('dash.addTitle') }}</h2>
            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-slate-700">Email</label>
                <input v-model="newAccount.email" type="email"
                       class="form-input mt-1 block h-12 w-full rounded-xl px-4 text-base">
              </div>
              <div>
                <label class="block text-sm font-medium text-slate-700">Password</label>
                <input v-model="newAccount.password" type="password"
                       class="form-input mt-1 block h-12 w-full rounded-xl px-4 text-base">
              </div>
              <div class="flex justify-end space-x-4 pt-4">
                <button @click="closeAddModal"
                        class="rounded-xl bg-slate-100 px-4 py-2 transition-all duration-300 hover:bg-slate-200">
                  {{ t('dash.cancel') }}
                </button>
                <button @click="addToken"
                        class="primary-submit rounded-xl px-4 py-2 text-white transition-all duration-300">
                  {{ t('dash.add') }}
                </button>
              </div>
            </div>
          </div>
          <div v-else key="batch" class="pr-1">
            <h2 class="mb-4 px-4 text-xl font-bold text-slate-800">{{ t('dash.batchTitle') }}</h2>
            <div class="space-y-4">
              <div>
                <label class="block px-4 pb-2 text-sm font-medium text-slate-700">{{ t('dash.batchLabel') }}</label>
                <textarea v-model="batchAccounts"
                          :disabled="isBatchAdding"
                          rows="6"
                          class="form-input mt-1 block h-36 w-full resize-none px-4 py-3 text-base disabled:opacity-70"></textarea>
              </div>
              <div v-if="batchTask" class="mx-4 rounded-2xl border border-slate-200 bg-slate-50/80 p-5 shadow-sm">
                <div class="flex flex-col gap-3 md:flex-row md:items-start md:justify-between">
                  <div>
                    <div class="text-sm text-slate-500">{{ t('dash.taskStatus') }}</div>
                    <div class="mt-1 flex items-center gap-3">
                      <span :class="['inline-flex rounded-full px-3 py-1 text-xs font-semibold', batchTaskStatusClass]">
                        {{ batchTaskStatusText }}
                      </span>
                      <span class="text-sm text-slate-600">{{ batchTask.message }}</span>
                    </div>
                  </div>
                  <div class="text-sm text-slate-500">
                    {{ t('dash.taskId') }} <span class="font-mono text-slate-700">{{ batchTask.taskId }}</span>
                  </div>
                </div>

                <div class="mt-4">
                  <div class="mb-2 flex items-center justify-between text-sm text-slate-600">
                    <span>{{ t('dash.progress') }}</span>
                    <span>{{ batchTask.completed }}/{{ batchTask.total }} · {{ batchTask.progress.toFixed(0) }}%</span>
                  </div>
                  <div class="h-3 overflow-hidden rounded-full bg-slate-200">
                    <div class="h-full rounded-full bg-gradient-to-r from-sky-500 via-cyan-500 to-emerald-500 transition-all duration-500" :style="batchTaskProgressStyle"></div>
                  </div>
                </div>

                <div class="mt-4 grid grid-cols-2 gap-3 md:grid-cols-5">
                  <div class="rounded-xl bg-slate-50 px-4 py-3">
                    <div class="text-xs text-slate-500">{{ t('dash.total') }}</div>
                    <div class="mt-1 text-lg font-semibold text-slate-800">{{ batchTask.total }}</div>
                  </div>
                  <div class="rounded-xl bg-emerald-50 px-4 py-3">
                    <div class="text-xs text-emerald-600">{{ t('dash.success') }}</div>
                    <div class="mt-1 text-lg font-semibold text-emerald-700">{{ batchTask.success }}</div>
                  </div>
                  <div class="rounded-xl bg-rose-50 px-4 py-3">
                    <div class="text-xs text-rose-600">{{ t('dash.failed') }}</div>
                    <div class="mt-1 text-lg font-semibold text-rose-700">{{ batchTask.failed }}</div>
                  </div>
                  <div class="rounded-xl bg-amber-50 px-4 py-3">
                    <div class="text-xs text-amber-600">{{ t('dash.skipped') }}</div>
                    <div class="mt-1 text-lg font-semibold text-amber-700">{{ batchTask.skipped }}</div>
                  </div>
                  <div class="rounded-xl bg-violet-50 px-4 py-3">
                    <div class="text-xs text-violet-600">{{ t('dash.invalid') }}</div>
                    <div class="mt-1 text-lg font-semibold text-violet-700">{{ batchTask.invalid }}</div>
                  </div>
                </div>

                <div v-if="batchTask.activeEmails?.length" class="mt-4 rounded-xl bg-sky-50 px-4 py-3">
                  <div class="text-sm font-medium text-sky-700">{{ t('dash.processing') }}</div>
                  <div class="mt-2 flex flex-wrap gap-2">
                    <span v-for="email in batchTask.activeEmails" :key="email" class="rounded-full bg-white px-3 py-1 text-xs text-sky-700 shadow-sm">
                      {{ email }}
                    </span>
                  </div>
                </div>

                <div v-if="batchTask.recentResults?.length" class="mt-4">
                  <div class="mb-2 text-sm font-medium text-slate-700">{{ t('dash.recentResults') }}</div>
                  <div class="max-h-52 space-y-2 overflow-y-auto pr-1">
                    <div v-for="item in batchTask.recentResults" :key="`${item.email}-${item.status}-${item.message}`" class="flex items-start justify-between rounded-xl border border-slate-200 bg-white px-4 py-3">
                      <div class="min-w-0 pr-4">
                        <div class="truncate text-sm font-medium text-slate-800">{{ item.email }}</div>
                        <div class="mt-1 text-xs text-slate-500">{{ item.message }}</div>
                      </div>
                      <span :class="['inline-flex shrink-0 rounded-full px-2.5 py-1 text-xs font-semibold', item.status === 'success' ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700']">
                        {{ item.status === 'success' ? t('dash.success') : t('dash.failed') }}
                      </span>
                    </div>
                  </div>
                </div>
              </div>
              <div class="flex justify-end space-x-4 pt-4">
                <button @click="closeAddModal"
                        class="rounded-xl bg-slate-100 px-4 py-2 transition-all duration-300 disabled:opacity-50 hover:bg-slate-200">
                  {{ isBatchAdding ? t('dash.continueBackground') : t('dash.close') }}
                </button>
                <button @click="addBatchTokens"
                        :disabled="isBatchAdding"
                        class="primary-submit rounded-xl px-4 py-2 text-white transition-all duration-300 disabled:opacity-50">
                  {{ isBatchAdding ? t('dash.batchRunning') : t('dash.batchStart') }}
                </button>
              </div>
            </div>
          </div>
        </transition>
        </div>
      </div>
    </div>

    <!-- Toast 通知 -->
    <div v-if="toast.show"
         :class="[
           'fixed right-4 top-4 z-50 max-w-md rounded-2xl border px-6 py-4 shadow-lg transform transition-all duration-300',
           toast.type === 'success' ? 'border-emerald-200 bg-emerald-50 text-emerald-800' :
           toast.type === 'warning' ? 'border-amber-200 bg-amber-50 text-amber-800' : 'border-rose-200 bg-rose-50 text-rose-800'
         ]">
      <div class="flex items-center space-x-2">
        <svg v-if="toast.type === 'success'" class="w-5 h-5 flex-shrink-0" fill="currentColor" viewBox="0 0 20 20">
          <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" />
        </svg>
        <svg v-else-if="toast.type === 'warning'" class="w-5 h-5 flex-shrink-0" fill="currentColor" viewBox="0 0 20 20">
          <path fill-rule="evenodd" d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z" clip-rule="evenodd" />
        </svg>
        <svg v-else class="w-5 h-5 flex-shrink-0" fill="currentColor" viewBox="0 0 20 20">
          <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
        </svg>
        <span class="whitespace-pre-line">{{ toast.message }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'
import { useI18n } from 'vue-i18n'
import axios from 'axios'

const { t } = useI18n()

const tokens = ref([])
const showAddModal = ref(false)
const addMode = ref('single')
const newAccount = ref({
  email: '',
  password: ''
})
const batchAccounts = ref('')
const isBatchAdding = ref(false)
const batchTask = ref(null)
const batchTaskPollTimer = ref(null)
const batchTaskNotified = ref(false)

// 分页相关
const displayedTokens = ref([])
const currentPage = ref(1)
const pageSize = ref(10)
const totalItems = ref(0)
const totalPages = computed(() => Math.max(1, Math.ceil(totalItems.value / pageSize.value)))
const isLoading = ref(false)

// 多选相关
const selectedTokens = ref([])
const selectAll = ref(false)
const showDeleteAllConfirm = ref(false)

// 刷新相关
const isRefreshingAll = ref(false)
const isForceRefreshingAll = ref(false)
const refreshingTokens = ref([])

// Toast 通知
const toast = ref({
  show: false,
  message: '',
  type: 'success'
})
const batchTaskStatusText = computed(() => {
  const status = batchTask.value?.status
  if (status === 'completed') return t('dash.statusCompleted')
  if (status === 'failed') return t('dash.statusFailed')
  if (status === 'running') return t('dash.statusRunning')
  if (status === 'pending') return t('dash.statusPending')
  return t('dash.statusNotStarted')
})
const batchTaskStatusClass = computed(() => {
  const status = batchTask.value?.status
  if (status === 'completed') return 'bg-emerald-100 text-emerald-700'
  if (status === 'failed') return 'bg-rose-100 text-rose-700'
  if (status === 'running') return 'bg-sky-100 text-sky-700'
  if (status === 'pending') return 'bg-amber-100 text-amber-700'
  return 'bg-slate-100 text-slate-700'
})
const batchTaskProgressStyle = computed(() => ({
  width: `${Math.max(0, Math.min(100, Number(batchTask.value?.progress || 0)))}%`
}))

const getAuthHeaders = () => ({
  'Authorization': localStorage.getItem('apiKey') || ''
})

const clearBatchTaskPolling = () => {
  if (batchTaskPollTimer.value) {
    clearTimeout(batchTaskPollTimer.value)
    batchTaskPollTimer.value = null
  }
}

const resetBatchTaskState = (clearInput = false) => {
  clearBatchTaskPolling()
  batchTask.value = null
  batchTaskNotified.value = false
  isBatchAdding.value = false

  if (clearInput) {
    batchAccounts.value = ''
  }
}

const buildBatchTaskMessage = (task) => {
  let message = t('msg.batchComplete', { n: task.success })
  if (task.failed > 0) message += ', ' + t('msg.batchFailed', { n: task.failed })
  if (task.skipped > 0) message += ', ' + t('msg.batchSkipped', { n: task.skipped })
  if (task.invalid > 0) message += ', ' + t('msg.batchInvalid', { n: task.invalid })
  if (task.failedEmails?.length > 0) {
    message += '\n' + t('msg.failedAccounts') + ' ' + task.failedEmails.slice(0, 10).join(', ')
  }
  return message
}

const finalizeBatchTask = async (task) => {
  clearBatchTaskPolling()
  batchTask.value = task
  isBatchAdding.value = false

  if (task.success > 0) {
    await getTokens()
  }

  if (!batchTaskNotified.value) {
    batchTaskNotified.value = true
    showToast(
      buildBatchTaskMessage(task),
      task.failed > 0 || task.invalid > 0 ? 'warning' : 'success'
    )
  }
}

const pollBatchTask = async (taskId) => {
  clearBatchTaskPolling()

  try {
    const res = await axios.get(`/api/batchTasks/${taskId}`, {
      headers: getAuthHeaders()
    })

    batchTask.value = res.data

    if (res.data.status === 'completed' || res.data.status === 'failed') {
      await finalizeBatchTask(res.data)
      return
    }

    batchTaskPollTimer.value = setTimeout(() => {
      pollBatchTask(taskId)
    }, 800)
  } catch (error) {
    console.error('pollBatchTask error:', error)

    if (batchTask.value) {
      batchTask.value = {
        ...batchTask.value,
        message: t('msg.progressRetry')
      }
    }

    batchTaskPollTimer.value = setTimeout(() => {
      pollBatchTask(taskId)
    }, 1500)
  }
}

const closeAddModal = () => {
  showAddModal.value = false

  if (!isBatchAdding.value && addMode.value === 'batch' && batchTask.value?.status === 'completed') {
    resetBatchTaskState(batchTask.value.failed === 0 && batchTask.value.invalid === 0)
  }
}

const isSelected = (email) => {
  return selectedTokens.value.includes(email)
}

const toggleSelect = (email) => {
  const index = selectedTokens.value.indexOf(email)
  if (index === -1) {
    selectedTokens.value.push(email)
  } else {
    selectedTokens.value.splice(index, 1)
  }
  selectAll.value = selectedTokens.value.length === displayedTokens.value.length
}

const toggleSelectAll = () => {
  if (selectAll.value) {
    selectedTokens.value = displayedTokens.value.map(token => token.email)
  } else {
    selectedTokens.value = []
  }
}

const deleteSelected = async () => {
  if (selectedTokens.value.length === 0) return

  if (!confirm(t('msg.deleteConfirm', { n: selectedTokens.value.length }))) return

  try {
    const deletePromises = selectedTokens.value.map(email =>
      axios.delete('/api/deleteAccount', {
        data: { email },
        headers: getAuthHeaders()
      })
    )

    await Promise.all(deletePromises)
    await getTokens()
    selectedTokens.value = []
    selectAll.value = false
    showToast(t('msg.deleteSuccess'))
  } catch (error) {
    console.error('deleteSelected error:', error)
    showToast(t('msg.deleteFailed') + error.message, 'error')
  }
}

const deleteAllAccounts = async () => {
  try {
    const res = await axios.get('/api/getAllAccounts', {
      params: { page: 1, pageSize: 10000 },
      headers: getAuthHeaders()
    })
    const allAccounts = res.data.data

    const deletePromises = allAccounts.map(token =>
      axios.delete('/api/deleteAccount', {
        data: { email: token.email },
        headers: getAuthHeaders()
      })
    )

    await Promise.all(deletePromises)
    showDeleteAllConfirm.value = false
    currentPage.value = 1
    await getTokens()
    selectedTokens.value = []
    selectAll.value = false
    showToast(t('msg.allDeleted'))
  } catch (error) {
    console.error('deleteAllAccounts error:', error)
    showToast(t('msg.allDeleteFailed') + error.message, 'error')
  }
}

const changePage = async (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page
    selectedTokens.value = []
    selectAll.value = false
    await getTokens()
  }
}

const changePageSize = async () => {
  currentPage.value = 1
  selectedTokens.value = []
  selectAll.value = false
  await getTokens()
}

const showToast = (message, type = 'success') => {
  toast.value.message = message
  toast.value.type = type
  toast.value.show = true

  setTimeout(() => {
    toast.value.show = false
  }, 3000)
}

const copyToClipboard = async (text) => {
  try {
    await navigator.clipboard.writeText(text)
    showToast(t('dash.copiedToClipboard'))
  } catch (err) {
    console.error('copyToClipboard error:', err)
    showToast(t('msg.copyFailed'), 'error')
  }
}

const getTokens = async () => {
  isLoading.value = true
  try {
    const res = await axios.get('/api/getAllAccounts', {
      params: {
        page: currentPage.value,
        pageSize: pageSize.value
      },
      headers: getAuthHeaders()
    })

    displayedTokens.value = res.data.data
    totalItems.value = res.data.total

    if (currentPage.value > totalPages.value && totalPages.value > 0) {
      currentPage.value = 1
      await getTokens()
      return
    }

    selectedTokens.value = []
    selectAll.value = false

  } catch (error) {
    console.error('getTokens error:', error)
    showToast(t('msg.fetchFailed') + error.message, 'error')
  } finally {
    isLoading.value = false
  }
}

const addToken = async () => {
  try {
    await axios.post('/api/setAccount', newAccount.value, {
      headers: getAuthHeaders()
    })
    closeAddModal()
    newAccount.value = { email: '', password: '' }
    await getTokens()
    showToast(t('msg.addSuccess'))
  } catch (error) {
    console.error('addToken error:', error)
    showToast(t('msg.addFailed') + error.message, 'error')
  }
}

const addBatchTokens = async () => {
  if (isBatchAdding.value) return
  if (!batchAccounts.value.trim()) {
    showToast(t('msg.enterAccounts'), 'error')
    return
  }

  batchTaskNotified.value = false
  clearBatchTaskPolling()
  isBatchAdding.value = true

  try {
    const res = await axios.post('/api/setAccounts', {
      accounts: batchAccounts.value,
      async: true
    }, {
      headers: getAuthHeaders()
    })

    batchTask.value = res.data
    addMode.value = 'batch'
    showAddModal.value = true

    if (res.data.status === 'completed' || res.data.status === 'failed') {
      await finalizeBatchTask(res.data)
      return
    }

    pollBatchTask(res.data.taskId)
  } catch (error) {
    console.error('addBatchTokens error:', error)
    showToast(t('msg.batchAddFailed') + error.message, 'error')
    isBatchAdding.value = false
  }
}

const refreshToken = async (email) => {
  if (refreshingTokens.value.includes(email)) return

  refreshingTokens.value.push(email)

  try {
    await axios.post('/api/refreshAccount', { email }, {
      headers: getAuthHeaders()
    })

    await getTokens()
    showToast(t('msg.refreshSuccess', { email }))
  } catch (error) {
    console.error('refreshToken error:', error)
    showToast(t('msg.refreshFailed') + error.message, 'error')
  } finally {
    const index = refreshingTokens.value.indexOf(email)
    if (index > -1) {
      refreshingTokens.value.splice(index, 1)
    }
  }
}

const refreshAllAccounts = async () => {
  if (isRefreshingAll.value) return

  if (!confirm(t('msg.refreshAllConfirm'))) return

  isRefreshingAll.value = true

  try {
    const response = await axios.post('/api/refreshAllAccounts', {
      thresholdHours: 24
    }, {
      headers: getAuthHeaders()
    })

    await getTokens()
    showToast(t('msg.refreshAllComplete', { n: response.data.refreshedCount }))
  } catch (error) {
    console.error('refreshAllAccounts error:', error)
    showToast(t('msg.refreshAllFailed') + error.message, 'error')
  } finally {
    isRefreshingAll.value = false
  }
}

const forceRefreshAllAccounts = async () => {
  if (isForceRefreshingAll.value) return

  if (!confirm(t('msg.forceRefreshAllConfirm'))) return

  isForceRefreshingAll.value = true

  try {
    const response = await axios.post('/api/forceRefreshAllAccounts', {}, {
      headers: getAuthHeaders()
    })

    await getTokens()
    showToast(t('msg.forceRefreshAllComplete', { n: response.data.refreshedCount }))
  } catch (error) {
    console.error('forceRefreshAllAccounts error:', error)
    showToast(t('msg.forceRefreshAllFailed') + error.message, 'error')
  } finally {
    isForceRefreshingAll.value = false
  }
}

const deleteToken = async (email) => {
  if (!confirm(t('msg.deleteAccountConfirm'))) return

  try {
    await axios.delete('/api/deleteAccount', {
      data: { email },
      headers: getAuthHeaders()
    })
    await getTokens()
    showToast(t('msg.deleteAccountSuccess'))
  } catch (error) {
    console.error('deleteToken error:', error)
    showToast(t('msg.deleteAccountFailed') + error.message, 'error')
  }
}

const exportAccounts = async () => {
  try {
    const res = await axios.get('/api/getAllAccounts', {
      params: { page: 1, pageSize: 10000 },
      headers: getAuthHeaders()
    })
    const allAccounts = res.data.data

    if (allAccounts.length === 0) {
      showToast(t('msg.exportEmpty'), 'error')
      return
    }

    const content = allAccounts.map(token => `${token.email}:${token.password}`).join('\n')

    const blob = new Blob([content], { type: 'text/plain;charset=utf-8' })

    const url = URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = url
    link.download = 'qwen_accounts.txt'
    document.body.appendChild(link)
    link.click()

    setTimeout(() => {
      document.body.removeChild(link)
      URL.revokeObjectURL(url)
    }, 100)

    showToast(t('msg.exportSuccess'))
  } catch (error) {
    console.error('exportAccounts error:', error)
    showToast(t('msg.exportFailed') + error.message, 'error')
  }
}

onMounted(() => {
  getTokens()
})

onBeforeUnmount(() => {
  clearBatchTaskPolling()
})
</script>

<style lang="css" scoped>
@media (max-width: 640px) {
  .dashboard-scroll {
    padding-left: 0.75rem;
    padding-right: 0.75rem;
  }
}

.dashboard-hero,
.info-strip {
  border: 1px solid rgba(214, 228, 243, 0.92);
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.92), rgba(246, 250, 255, 0.88));
  box-shadow: 0 18px 40px rgba(39, 84, 132, 0.08);
  border-radius: 28px;
}

.dashboard-scroll {
  scrollbar-gutter: stable;
  scrollbar-width: thin;
  scrollbar-color: rgba(124, 159, 197, 0.9) rgba(222, 235, 248, 0.9);
}

.modal-scroll {
  scrollbar-width: thin;
  scrollbar-color: rgba(124, 159, 197, 0.75) rgba(233, 241, 250, 0.9);
}

.dashboard-scroll::-webkit-scrollbar {
  width: 8px;
}

.modal-scroll::-webkit-scrollbar {
  width: 6px;
}

.dashboard-scroll::-webkit-scrollbar-track {
  background: rgba(226, 237, 248, 0.95);
  border-radius: 999px;
  border: 1px solid rgba(199, 217, 236, 0.9);
}

.modal-scroll::-webkit-scrollbar-track {
  background: rgba(235, 242, 250, 0.95);
  border-radius: 999px;
}

.dashboard-scroll::-webkit-scrollbar-thumb {
  background: linear-gradient(180deg, rgba(133, 170, 210, 0.95), rgba(105, 146, 190, 0.95));
  border-radius: 999px;
  border: 1px solid rgba(95, 137, 183, 0.78);
  background-clip: padding-box;
}

.modal-scroll::-webkit-scrollbar-thumb {
  background: linear-gradient(180deg, rgba(150, 183, 220, 0.9), rgba(123, 159, 198, 0.9));
  border-radius: 999px;
  border: 1px solid rgba(118, 154, 194, 0.7);
}

.dashboard-scroll::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(180deg, rgba(112, 156, 202, 1), rgba(84, 129, 178, 1));
}

.modal-scroll::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(180deg, rgba(129, 168, 210, 1), rgba(103, 145, 190, 1));
}

.dashboard-scroll::-webkit-scrollbar-thumb:active {
  background: linear-gradient(180deg, rgba(96, 142, 190, 1), rgba(74, 120, 171, 1));
}

.dashboard-scroll::-webkit-scrollbar-corner {
  background: transparent;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s, transform 0.3s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
.fade-enter-to, .fade-leave-from {
  opacity: 1;
  transform: translateY(0);
}

.token-card {
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.95), rgba(245, 249, 254, 0.95));
  border: 1px solid rgba(214, 228, 243, 0.9);
  box-shadow: 0 16px 30px rgba(38, 83, 130, 0.08);
  transform: translateY(0);
  transition: transform 0.25s ease, box-shadow 0.25s ease, border-color 0.25s ease;
}

.token-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 22px 36px rgba(38, 83, 130, 0.12);
  border-color: rgba(162, 196, 231, 0.9);
}

.scrollbar-hide {
  -ms-overflow-style: none;
  scrollbar-width: none;
}

.scrollbar-hide::-webkit-scrollbar {
  display: none;
}

@keyframes slideIn {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.token-card {
  animation: slideIn 0.5s ease-out;
  animation-fill-mode: both;
}

.token-card:nth-child(3n+1) { animation-delay: 0.1s; }
.token-card:nth-child(3n+2) { animation-delay: 0.2s; }
.token-card:nth-child(3n+3) { animation-delay: 0.3s; }

.overflow-x-auto {
  position: relative;
  cursor: pointer;
}

.overflow-x-auto::after {
  content: '';
  position: absolute;
  right: 0;
  top: 0;
  bottom: 0;
  width: 24px;
  background: linear-gradient(to right, transparent, rgba(244, 248, 253, 0.95));
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.3s;
}

.overflow-x-auto:hover::after {
  opacity: 1;
}

/* 隐藏滚动条样式 */
.scrollbar-hidden {
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;  /* Firefox */
}

.scrollbar-hidden::-webkit-scrollbar {
  display: none;  /* Chrome, Safari and Opera */
}

/* 自定义滚动条样式（备用） */
.max-h-\[calc\(100vh-200px\)\]::-webkit-scrollbar {
  width: 6px;
}

.max-h-\[calc\(100vh-200px\)\]::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.05);
  border-radius: 8px;
}

.max-h-\[calc\(100vh-200px\)\]::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

.max-h-\[calc\(100vh-200px\)\]::-webkit-scrollbar-thumb:hover {
  background-color: rgba(0, 0, 0, 0.2);
}

/* 自定义复选框样式 */
.custom-checkbox .checkbox-icon {
  position: relative;
  overflow: hidden;
}

.custom-checkbox .checkbox-icon:before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 0;
  height: 100%;
  background: rgba(71, 134, 204, 0.1);
  transition: width 0.3s ease;
}

.custom-checkbox:hover .checkbox-icon:before {
  width: 100%;
}

.custom-checkbox input:checked + .checkbox-icon svg {
  animation: check-animation 0.5s cubic-bezier(0.17, 0.67, 0.83, 0.67);
  transform: scale(1);
}

@keyframes check-animation {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}

/* 给选中的卡片添加动画效果 */
.token-card.ring-2 {
  animation: selected-pulse 2s infinite;
}

@keyframes selected-pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(76, 143, 216, 0.35);
  }
  70% {
    box-shadow: 0 0 0 6px rgba(76, 143, 216, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(76, 143, 216, 0);
  }
}

.field-row {
  background: linear-gradient(180deg, rgba(240, 247, 255, 0.95), rgba(233, 242, 252, 0.95));
  border: 1px solid rgba(214, 228, 243, 0.9);
}

.field-label {
  min-width: 96px;
  text-align: left;
  font-size: 0.78rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #54708d;
}

.field-value {
  color: #16385d;
  font-weight: 600;
}

.copy-button {
  opacity: 0;
  color: #235f9f;
  background: rgba(214, 232, 251, 0.95);
  border: 1px solid rgba(173, 203, 236, 0.9);
  transition: opacity 0.2s ease, background-color 0.2s ease;
}

.group:hover .copy-button {
  opacity: 1;
}

.copy-button:hover {
  background: rgba(197, 221, 247, 0.98);
}

.toolbar-btn,
.token-action {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  min-height: 44px;
  padding: 0.7rem 1rem;
  border-radius: 14px;
  border: 1px solid transparent;
  font-weight: 700;
  transition: transform 0.2s ease, box-shadow 0.2s ease, background-color 0.2s ease, border-color 0.2s ease;
}

.toolbar-btn:hover,
.token-action:hover {
  transform: translateY(-1px);
}

.toolbar-btn--primary,
.primary-submit,
.token-action--primary {
  background: linear-gradient(135deg, #3d86d3, #2c6fb7);
  border-color: rgba(41, 102, 167, 0.92);
  color: #ffffff;
  box-shadow: 0 14px 28px rgba(45, 104, 171, 0.18);
}

.toolbar-btn--primary:hover,
.primary-submit:hover,
.token-action--primary:hover {
  background: linear-gradient(135deg, #327bc8, #235f9f);
}

.toolbar-btn--accent {
  background: linear-gradient(180deg, rgba(235, 244, 255, 0.98), rgba(219, 234, 251, 0.98));
  border-color: rgba(163, 196, 231, 0.92);
  color: #1f5d9e;
  box-shadow: 0 10px 22px rgba(73, 121, 175, 0.12);
}

.toolbar-btn--neutral,
.toolbar-btn--ghost {
  background: rgba(247, 250, 255, 0.95);
  border-color: rgba(203, 220, 238, 0.92);
  color: #35597d;
}

.toolbar-btn--warning {
  background: linear-gradient(180deg, rgba(255, 244, 223, 0.98), rgba(255, 238, 204, 0.98));
  border-color: rgba(240, 206, 145, 0.95);
  color: #8d5a13;
}

.toolbar-btn--busy {
  background: linear-gradient(135deg, #5f96d0, #3d79bb);
  border-color: rgba(51, 110, 178, 0.95);
  color: #ffffff;
}

.token-action {
  width: 100%;
}

.token-action--danger {
  background: rgba(252, 232, 232, 0.92);
  border-color: rgba(239, 196, 196, 0.95);
  color: #b43b3b;
}

.token-action--danger:hover {
  background: rgba(249, 220, 220, 0.96);
}

.form-input {
  border: 1px solid rgba(214, 228, 243, 0.95);
  background: rgba(248, 251, 255, 0.98);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.85);
  transition: border-color 0.2s ease, box-shadow 0.2s ease, background-color 0.2s ease;
}

.form-input:focus {
  outline: none;
  border-color: #67a5e4;
  box-shadow: 0 0 0 4px rgba(103, 165, 228, 0.18);
  background: #ffffff;
}

@keyframes refresh-pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(92, 149, 210, 0.32);
  }
  70% {
    box-shadow: 0 0 0 6px rgba(92, 149, 210, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(92, 149, 210, 0);
  }
}

.toolbar-btn--busy,
.token-action--primary.toolbar-btn--busy {
  animation: refresh-pulse 1.6s infinite;
}
</style>
