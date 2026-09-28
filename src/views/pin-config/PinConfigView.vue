<template>
  <main class="flex-1 flex flex-col bg-slate-50 dark:bg-slate-950 relative overflow-hidden transition-colors">
    <template v-if="store.currentVcu && store.currentConfig">
      <!-- 顶部操作工具条 -->
      <div class="h-11 px-4 border-b border-slate-200 dark:border-slate-800 flex items-center justify-between bg-white/50 dark:bg-slate-900/20 backdrop-blur shrink-0">
        <!-- 左侧：搜索与模式切换 -->
        <div class="flex items-center gap-3">
          <div class="relative w-56">
            <Search class="w-3.5 h-3.5 absolute left-2.5 top-2.5 text-slate-400" />
            <input 
              v-model="pinSearch" 
              :placeholder="t('vcu.searchPinPlaceholder')" 
              class="w-full bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-md px-2.5 py-1 pl-8 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500"
            />
          </div>

          <div class="flex items-center bg-slate-200/60 dark:bg-slate-800 p-0.5 rounded-lg border border-slate-300 dark:border-slate-700">
            <button 
              @click="mode = 'graphic'"
              :class="['p-1 rounded-md transition cursor-pointer', mode === 'graphic' ? 'bg-white dark:bg-slate-900 text-cyan-600 dark:text-cyan-400 shadow-xs' : 'text-slate-400 hover:text-slate-600 dark:hover:text-slate-200']"
              :title="t('sidebar.viewIcon')"
            >
              <LayoutGrid class="w-4 h-4" />
            </button>
            <button 
              @click="mode = 'table'"
              :class="['p-1 rounded-md transition cursor-pointer', mode === 'table' ? 'bg-white dark:bg-slate-900 text-cyan-600 dark:text-cyan-400 shadow-xs' : 'text-slate-400 hover:text-slate-600 dark:hover:text-slate-200']"
              :title="t('sidebar.viewList')"
            >
              <List class="w-4 h-4" />
            </button>
          </div>
        </div>

        <!-- 右侧：取消/应用配置 + 导出/下发 -->
        <div class="flex items-center gap-2">
          <!-- 取消按钮 -->
          <button 
            @click="handleDiscard" 
            :disabled="!store.isDirty"
            class="px-3 py-1 bg-white hover:bg-slate-100 dark:bg-slate-800 dark:hover:bg-slate-700 text-xs rounded-md border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 transition cursor-pointer disabled:opacity-40 disabled:cursor-not-allowed flex items-center gap-1"
          >
            <Undo class="w-3.5 h-3.5" /> {{ t('common.action.undo') }}
          </button>

          <!-- 应用按钮 -->
          <button 
            @click="handleApply" 
            :disabled="!store.isDirty"
            :class="[
              'px-3.5 py-1 text-xs rounded-md font-semibold transition cursor-pointer flex items-center gap-1.5 shadow-xs',
              store.isDirty 
                ? 'bg-emerald-600 hover:bg-emerald-500 text-white animate-pulse' 
                : 'bg-slate-200 dark:bg-slate-800 text-slate-400 opacity-50 cursor-not-allowed'
            ]"
          >
            <Check class="w-3.5 h-3.5" /> {{ t('common.action.apply') }}
          </button>

          <div class="h-4 w-[1px] bg-slate-200 dark:border-slate-800 mx-1"></div>

          <button @click="exportJson" class="px-2.5 py-1 bg-white hover:bg-slate-50 dark:bg-slate-800 dark:hover:bg-slate-700 text-xs rounded-md border border-slate-200 dark:border-slate-700 flex items-center gap-1 text-slate-700 dark:text-slate-200 shadow-xs cursor-pointer">
            <FileCode class="w-3.5 h-3.5 text-cyan-500" /> {{ t('export.json') }}
          </button>
          <button @click="exportExcel" class="px-2.5 py-1 bg-white hover:bg-slate-50 dark:bg-slate-800 dark:hover:bg-slate-700 text-xs rounded-md border border-slate-200 dark:border-slate-700 flex items-center gap-1 text-slate-700 dark:text-slate-200 shadow-xs cursor-pointer">
            <FileSpreadsheet class="w-3.5 h-3.5 text-emerald-500" /> {{ t('export.excel') }}
          </button>
          <button @click="isUdsOpen = true" class="px-3 py-1 bg-indigo-600 hover:bg-indigo-500 text-white text-xs rounded-md font-medium shadow-xs flex items-center gap-1 cursor-pointer">
            <Send class="w-3.5 h-3.5" /> {{ t('uds.deploy') }}
          </button>
        </div>
      </div>

      <!-- 核心工作画布 -->
      <div class="flex-1 p-5 overflow-y-auto">
        <GraphicConnector ref="graphicRef" v-if="mode === 'graphic'" :search-keyword="pinSearch" />
        <TableConnector ref="tableRef" v-else :search-keyword="pinSearch" />
      </div>
    </template>

    <div v-else class="flex-1 flex flex-col items-center justify-center text-slate-400 dark:text-slate-600">
      <Cpu class="w-12 h-12 mb-2 stroke-1" />
      <p class="text-xs">{{ t('vcu.emptyTip') }}</p>
    </div>

    <!-- 基于 BaseModal 的下发向导 -->
    <UdsWizardModal v-model="isUdsOpen" />
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import * as XLSX from 'xlsx';
import GraphicConnector from './GraphicConnector.vue';
import TableConnector from './TableConnector.vue';
import UdsWizardModal from '@/views/modals/UdsWizardModal.vue';
import { Search, FileSpreadsheet, FileCode, Send, Cpu, LayoutGrid, List, Check, Undo } from 'lucide-vue-next';

const emit = defineEmits<{(e: 'validation-failed', errors: string[]): void; (e: 'apply-success'): void;}>();

const { t } = useI18n();
const store = useVcuStore();

const pinSearch = ref('');
const mode = ref<'graphic' | 'table'>('graphic');
const isUdsOpen = ref(false);

const graphicRef = ref<InstanceType<typeof GraphicConnector> | null>(null);
const tableRef = ref<InstanceType<typeof TableConnector> | null>(null);

function handleDiscard() {
  if (confirm(t('vcu.discardConfirm'))) {
    store.discardChanges();
  }
}

async function handleApply() {
  const { success, result } = await store.applyConfig();
  if (success) {
    emit('apply-success');
  } else {
    emit('validation-failed', result.errors);
  }
}

function exportExcel() {
  if (!store.currentConfig) return;
  const rows = Object.entries(store.currentConfig.assignments).map(([pin, detail]) => ({
    [t('vcu.pinNo')]: pin,
    [t('vcu.function')]: detail.selected_function,
    [t('vcu.range')]: detail.analog_range 
      ? `${detail.analog_range.min} ~ ${detail.analog_range.max} ${detail.analog_range.unit}` 
      : 'N/A'
  }));
  const ws = XLSX.utils.json_to_sheet(rows);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, t('export.sheetName'));
  XLSX.writeFile(wb, `${store.currentConfig.config_name}.xlsx`);
}

function exportJson() {
  if (!store.currentConfig) return;
  const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(store.currentConfig, null, 2));
  const downloadAnchor = document.createElement('a');
  downloadAnchor.setAttribute("href", dataStr);
  downloadAnchor.setAttribute("download", `${store.currentConfig.config_name}.json`);
  document.body.appendChild(downloadAnchor);
  downloadAnchor.click();
  downloadAnchor.remove();
}

function locatePin(pinLabel: string) {
  if (mode.value === 'graphic') {
    graphicRef.value?.scrollToPin(pinLabel);
  } else {
    tableRef.value?.scrollToPin(pinLabel);
  }
}

defineExpose({ locatePin });
</script>