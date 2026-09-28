<template>
  <BaseModal :model-value="modelValue" :title="t('descEditor.title')" size="xl" @close="handleClose">
    <div v-if="formData" class="flex flex-col h-[65vh]">
      <!-- 1. 现代化微光胶囊 Tab 切换栏 -->
      <div class="flex items-center justify-between pb-3 mb-4 border-b border-slate-200/80 dark:border-slate-800 shrink-0">
        <div class="inline-flex p-1 bg-slate-100 dark:bg-slate-900 border border-slate-200/60 dark:border-slate-800 rounded-xl">
          <button 
            @click="activeTab = 'basic'" 
            :class="[
              'px-4 py-1.5 rounded-lg text-xs font-semibold transition-all duration-200 cursor-pointer flex items-center gap-2',
              activeTab === 'basic' 
                ? 'bg-white dark:bg-slate-800 text-cyan-600 dark:text-cyan-400 shadow-sm' 
                : 'text-slate-600 dark:text-slate-400 hover:text-slate-900 dark:hover:text-slate-200'
            ]"
          >
            <FileText class="w-3.5 h-3.5" /> 
            <span>{{ t('descEditor.tabBasic') }}</span>
          </button>
          <button 
            @click="activeTab = 'resources'" 
            :class="[
              'px-4 py-1.5 rounded-lg text-xs font-semibold transition-all duration-200 cursor-pointer flex items-center gap-2',
              activeTab === 'resources' 
                ? 'bg-white dark:bg-slate-800 text-cyan-600 dark:text-cyan-400 shadow-sm' 
                : 'text-slate-600 dark:text-slate-400 hover:text-slate-900 dark:hover:text-slate-200'
            ]"
          >
            <Layers class="w-3.5 h-3.5" /> 
            <span>{{ t('descEditor.tabResources') }}</span>
            <span :class="[
              'text-[10px] px-1.5 py-0.5 rounded-full font-mono transition-colors',
              activeTab === 'resources' ? 'bg-cyan-100 dark:bg-cyan-950 text-cyan-700 dark:text-cyan-300' : 'bg-slate-200 dark:bg-slate-800 text-slate-500'
            ]">
              {{ formData.connectors.length }}
            </span>
          </button>
        </div>

        <!-- 显式配置状态统计胶囊 -->
        <div v-if="activeTab === 'resources'" class="flex items-center gap-2 text-xs">
          <span :class="['flex items-center gap-1.5 px-2.5 py-1 rounded-full text-[11px] font-medium border', unconfiguredPins.length ? 'bg-rose-50 dark:bg-rose-950/40 border-rose-200 dark:border-rose-900/60 text-rose-600 dark:text-rose-400' : 'bg-emerald-50 dark:bg-emerald-950/40 border-emerald-200 dark:border-emerald-900/60 text-emerald-600 dark:text-emerald-400']">
            <span class="w-1.5 h-1.5 rounded-full" :class="unconfiguredPins.length ? 'bg-rose-500 animate-pulse' : 'bg-emerald-500'"></span>
            {{ unconfiguredPins.length ? `未显式配置: ${unconfiguredPins.length} Pins` : '全部引脚显式就绪' }}
          </span>
        </div>
      </div>

      <!-- Tab 1: 基础属性编辑 -->
      <div v-show="activeTab === 'basic'" class="space-y-4 flex-1 overflow-y-auto pr-1">
        <div>
          <label class="block font-medium text-slate-700 dark:text-slate-300 mb-1">{{ t('descEditor.id') }}</label>
          <input v-model="formData.id" disabled class="w-full bg-slate-100 dark:bg-slate-950 border border-slate-200 dark:border-slate-800 rounded p-1.5 text-xs text-slate-400 font-mono" />
        </div>

        <div class="grid grid-cols-2 gap-3">
          <div>
            <label class="block font-medium text-slate-700 dark:text-slate-300 mb-1">{{ t('descEditor.nameZh') }}</label>
            <input v-model="formData.nameZh" class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded p-1.5 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500" />
          </div>
          <div>
            <label class="block font-medium text-slate-700 dark:text-slate-300 mb-1">{{ t('descEditor.nameEn') }}</label>
            <input v-model="formData.nameEn" class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded p-1.5 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500" />
          </div>
        </div>

        <div>
          <label class="block font-medium text-slate-700 dark:text-slate-300 mb-1">{{ t('descEditor.summaryZh') }}</label>
          <textarea rows="3" v-model="formData.summaryZh" class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded p-1.5 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500"></textarea>
        </div>

        <div>
          <label class="block font-medium text-slate-700 dark:text-slate-300 mb-1">{{ t('descEditor.summaryEn') }}</label>
          <textarea rows="3" v-model="formData.summaryEn" class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded p-1.5 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500"></textarea>
        </div>
      </div>

      <!-- Tab 2: 连接器与 Pin 资源图形化编辑器 -->
      <div v-show="activeTab === 'resources'" class="flex-1 flex gap-4 overflow-hidden">
        <!-- 2. 左侧连接器清单 (删除按钮直接内嵌于每个连接器卡片) -->
        <div class="w-60 flex flex-col border border-slate-200 dark:border-slate-800 rounded-xl overflow-hidden shrink-0 bg-slate-50/60 dark:bg-slate-900/40">
          <div class="h-10 px-3 border-b border-slate-200 dark:border-slate-800 flex items-center justify-between bg-slate-100/70 dark:bg-slate-800/60">
            <span class="font-semibold text-slate-700 dark:text-slate-300 text-xs">{{ t('descEditor.connectorList') }}</span>
            <button 
              @click="addConnector" 
              class="p-1 hover:bg-cyan-600 hover:text-white rounded text-cyan-600 dark:text-cyan-400 transition cursor-pointer flex items-center gap-1 text-[11px] font-medium"
              :title="t('descEditor.addConnector')"
            >
              <Plus class="w-3.5 h-3.5" />
              <span>{{ t('common.action.add') }}</span>
            </button>
          </div>

          <div class="flex-1 overflow-y-auto p-2 space-y-1.5">
            <div 
              v-for="(conn, idx) in formData.connectors" 
              :key="conn.id"
              @click="selectedConnIndex = idx"
              :class="[
                'p-2.5 rounded-lg cursor-pointer transition flex items-center justify-between text-xs group relative border',
                selectedConnIndex === idx 
                  ? 'bg-cyan-50/90 dark:bg-cyan-950/70 border-cyan-400/80 dark:border-cyan-800 text-cyan-800 dark:text-cyan-300 font-semibold shadow-xs' 
                  : 'bg-white/80 dark:bg-slate-900/60 border-slate-200/80 dark:border-slate-800/80 hover:border-slate-300 dark:hover:border-slate-700 text-slate-700 dark:text-slate-300'
              ]"
            >
              <div class="flex items-center gap-2 truncate pr-14">
                <Layers class="w-4 h-4 text-cyan-500 shrink-0" />
                <span class="truncate font-mono">{{ conn.name }}</span>
              </div>

              <!-- 右侧信息与直接删除按钮 -->
              <div class="flex items-center gap-1.5">
                <span class="text-[10px] text-slate-400 font-mono group-hover:hidden transition">
                  {{ conn.pin_count }}P
                </span>

                <!-- 直接放置在连接器上的删除按钮 -->
                <button 
                  @click.stop="deleteConnectorByIndex(idx)" 
                  class="p-1 text-slate-400 hover:text-rose-600 hover:bg-rose-50 dark:hover:bg-rose-950/60 rounded transition cursor-pointer"
                  :title="t('descEditor.delConnector')"
                >
                  <Trash2 class="w-3.5 h-3.5" />
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- 右侧选中的连接器详细属性与 Pin 矩阵表格 -->
        <div v-if="currentConn" class="flex-1 flex flex-col border border-slate-200 dark:border-slate-800 rounded-xl overflow-hidden bg-white dark:bg-slate-900/50">
          <!-- 连接器头部属性快速修改 -->
          <div class="px-4 py-2.5 border-b border-slate-200 dark:border-slate-800 bg-slate-50/60 dark:bg-slate-900/40 flex items-center justify-between gap-4 shrink-0">
            <div class="flex items-center gap-5">
              <div class="flex items-center gap-2">
                <span class="text-slate-500 text-xs shrink-0">{{ t('descEditor.connectorName') }}:</span>
                <input 
                  v-model="currentConn.name" 
                  @input="handleConnectorNameChange"
                  class="w-28 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-md px-2.5 py-1 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500 font-bold font-mono" 
                />
              </div>

              <div class="flex items-center gap-2">
                <span class="text-slate-500 text-xs shrink-0">{{ t('descEditor.pinCount') }}:</span>
                <input 
                  type="number" 
                  min="1" 
                  max="120"
                  :value="currentConn.pin_count" 
                  @change="handlePinCountChange"
                  class="w-20 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-md px-2 py-1 text-xs text-center text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500 font-mono" 
                />
              </div>
            </div>

            <!-- 一键将未配置引脚设为 NC -->
            <button 
              @click="setRemainingAsNC"
              class="px-2.5 py-1 rounded bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 text-slate-600 dark:text-slate-300 text-[11px] transition cursor-pointer flex items-center gap-1"
            >
              <CheckSquare class="w-3 h-3 text-cyan-500" /> 一键未配置设为 NC
            </button>
          </div>

          <!-- Pin 端口属性配置列表 -->
          <div class="flex-1 overflow-y-auto p-4 space-y-3">
            <div 
              v-for="pin in currentConn.pins" 
              :key="pin.pin_no"
              :class="[
                'p-3 rounded-xl border transition-all flex flex-col gap-2.5',
                pin.is_power 
                  ? 'bg-amber-50/40 dark:bg-amber-950/10 border-amber-300 dark:border-amber-900/50' 
                  : isPinConfigured(pin).configured 
                    ? 'bg-slate-50/60 dark:bg-slate-950/40 border-slate-200 dark:border-slate-800/80' 
                    : 'bg-rose-50/30 dark:bg-rose-950/20 border-rose-300 dark:border-rose-900/60 ring-1 ring-rose-400/40'
              ]"
            >
              <div class="flex items-center justify-between text-xs">
                <div class="flex items-center gap-2">
                  <span class="font-bold font-mono text-slate-800 dark:text-slate-200 text-sm">{{ pin.label }}</span>
                  <span v-if="pin.is_power" class="text-[10px] bg-amber-100 dark:bg-amber-950/80 text-amber-700 dark:text-amber-400 border border-amber-300 dark:border-amber-800 px-1.5 py-0.2 rounded font-mono">
                    {{ t('vcu.powerBadge') }}
                  </span>
                  <span v-else-if="pin.default_function === 'NC'" class="text-[10px] bg-slate-200 dark:bg-slate-800 text-slate-600 dark:text-slate-400 border border-slate-300 dark:border-slate-700 px-1.5 py-0.2 rounded font-mono">
                    {{ t('descEditor.ncBadge') }}
                  </span>
                  <span v-if="!isPinConfigured(pin).configured" class="text-[10px] text-rose-500 font-semibold flex items-center gap-1">
                    <AlertCircle class="w-3 h-3" /> 未显式配置
                  </span>
                  <span v-if="pin.is_power" class="text-[10px] text-amber-600 dark:text-amber-400">
                    ({{ t('descEditor.powerModeTip') }})
                  </span>
                </div>

                <!-- 默认工作模式下拉选择 -->
                <div class="flex items-center gap-2">
                  <span class="text-slate-500 dark:text-slate-400 text-xs">{{ t('descEditor.defaultMode') }}:</span>
                  <select 
                    v-model="pin.default_function"
                    class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-md px-2.5 py-1 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500 font-mono cursor-pointer"
                  >
                    <option v-for="mode in pin.supported_functions" :key="mode" :value="mode">
                      {{ mode }}
                    </option>
                  </select>
                </div>
              </div>

              <!-- 支持的模式列表 (多选下拉输入组件，支持 NC 模式) -->
              <div>
                <label class="block text-[11px] text-slate-400 mb-1">{{ t('descEditor.supportedModes') }}:</label>
                <MultiSelectDropdown 
                  v-model="pin.supported_functions"
                  :options="ALL_MODES"
                  @change="(newModes) => onPinModesChange(pin, newModes)"
                />
              </div>
            </div>
          </div>
        </div>

        <div v-else class="flex-1 flex items-center justify-center text-slate-400 text-xs">
          请选择或新增连接器
        </div>
      </div>
    </div>

    <!-- 底部操作栏 -->
    <template #footer>
      <button 
        @click="handleClose" 
        class="px-3 py-1.5 rounded border border-slate-200 dark:border-slate-700 text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-800 transition cursor-pointer"
      >
        {{ t('common.action.cancel') }}
      </button>
      <button 
        @click="validateAndSave" 
        class="px-3.5 py-1.5 rounded bg-cyan-600 hover:bg-cyan-500 text-white font-medium transition cursor-pointer flex items-center gap-1.5 shadow-xs"
      >
        <Save class="w-3.5 h-3.5" />
        {{ t('common.action.save') }}
      </button>
    </template>
  </BaseModal>

  <!-- 3. 未显式配置端口拦截警告弹窗 (基于 BaseModal) -->
  <BaseModal 
    v-model="showValidationModal" 
    :title="t('descEditor.validationTitle')" 
    size="md" 
    @close="showValidationModal = false"
  >
    <div class="space-y-3">
      <div class="flex items-start gap-2.5 p-3 rounded-lg bg-rose-50 dark:bg-rose-950/30 border border-rose-200 dark:border-rose-900/60 text-rose-700 dark:text-rose-300 text-xs">
        <AlertTriangle class="w-4 h-4 shrink-0 text-rose-500 mt-0.5" />
        <div>
          <p class="font-semibold">{{ t('descEditor.validationTitle') }}</p>
          <p class="mt-1 text-rose-600/90 dark:text-rose-400/90 leading-relaxed">{{ t('descEditor.validationDesc') }}</p>
        </div>
      </div>

      <!-- 未配置的引脚列表清单 -->
      <div class="max-h-48 overflow-y-auto space-y-1 border border-slate-200 dark:border-slate-800 rounded-lg p-2 bg-slate-50/50 dark:bg-slate-900/50">
        <div 
          v-for="item in unconfiguredPins" 
          :key="item.label"
          class="flex items-center justify-between text-xs p-1.5 rounded hover:bg-white dark:hover:bg-slate-800"
        >
          <span class="font-mono font-bold text-slate-800 dark:text-slate-200">{{ item.label }}</span>
          <span class="text-rose-500 text-[11px]">{{ t(`descEditor.${item.reason}`) }}</span>
        </div>
      </div>
    </div>

    <template #footer>
      <button 
        @click="autoFixAllAsNC" 
        class="px-3 py-1.5 rounded bg-cyan-600 hover:bg-cyan-500 text-white font-medium transition cursor-pointer"
      >
        全部自动设为 NC
      </button>
      <button 
        @click="showValidationModal = false" 
        class="px-3 py-1.5 rounded border border-slate-200 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-800 text-slate-600 dark:text-slate-300 transition cursor-pointer"
      >
        返回手动编辑
      </button>
    </template>
  </BaseModal>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import BaseModal from '@/components/common/BaseModal.vue';
import MultiSelectDropdown from '@/components/common/MultiSelectDropdown.vue';
import type { VcuDescription, ConnectorDef, PinDef, FunctionType } from '@/types/vcu';
import { ALL_MODES, applyPinModeRules, isPinConfigured } from '@/utils/pinRules';
import { FileText, Layers, Plus, Trash2, Save, AlertTriangle, AlertCircle, CheckSquare } from 'lucide-vue-next';

const props = defineProps<{ modelValue: boolean; vcu: VcuDescription | null }>();
const emit = defineEmits<{(e: 'update:modelValue', val: boolean): void}>();

const { t } = useI18n();
const store = useVcuStore();

const activeTab = ref<'basic' | 'resources'>('basic');
const selectedConnIndex = ref(0);
const showValidationModal = ref(false);

const formData = ref<{
  id: string;
  nameZh: string;
  nameEn: string;
  summaryZh: string;
  summaryEn: string;
  icon_type: string;
  connectors: ConnectorDef[];
} | null>(null);

watch(() => props.vcu, (newVcu) => {
  if (newVcu) {
    formData.value = {
      id: newVcu.id,
      nameZh: (newVcu.name as any)?.['zh-CN'] || (typeof newVcu.name === 'string' ? newVcu.name : ''),
      nameEn: (newVcu.name as any)?.['en-US'] || '',
      summaryZh: (newVcu.summary as any)?.['zh-CN'] || (typeof newVcu.summary === 'string' ? newVcu.summary : ''),
      summaryEn: (newVcu.summary as any)?.['en-US'] || '',
      icon_type: newVcu.icon_type || 'ecu-default',
      connectors: JSON.parse(JSON.stringify(newVcu.connectors || []))
    };
    selectedConnIndex.value = 0;
  }
}, { immediate: true });

const currentConn = computed(() => {
  if (!formData.value || !formData.value.connectors.length) return null;
  return formData.value.connectors[selectedConnIndex.value] || null;
});

// 计算所有未显式配置的引脚清单
const unconfiguredPins = computed(() => {
  if (!formData.value) return [];
  const list: { label: string; reason: string }[] = [];
  formData.value.connectors.forEach(c => {
    c.pins.forEach(p => {
      const check = isPinConfigured(p);
      if (!check.configured) {
        list.push({ label: p.label, reason: check.errorReason || 'emptySupported' });
      }
    });
  });
  return list;
});

// 新增连接器
function addConnector() {
  if (!formData.value) return;
  const newIndex = formData.value.connectors.length + 1;
  const name = `JP${newIndex}`;
  const count = 8;
  const pins: PinDef[] = [];
  for (let i = 1; i <= count; i++) {
    pins.push({
      pin_no: String(i),
      label: `${name}.${i}`,
      is_power: false,
      supported_functions: ['NC'], // 默认直接显式分配为 NC 悬空
      default_function: 'NC',
      grid_row: Math.floor((i - 1) / 4) + 1,
      grid_col: ((i - 1) % 4) + 1,
    });
  }

  formData.value.connectors.push({
    id: `conn-${crypto.randomUUID().slice(0, 6)}`,
    name,
    pin_count: count,
    pins,
  });
  selectedConnIndex.value = formData.value.connectors.length - 1;
}

// 直接在连接器上删除指定索引的连接器
function deleteConnectorByIndex(index: number) {
  if (!formData.value) return;
  const targetConn = formData.value.connectors[index];
  if (!targetConn) return;

  const confirmMsg = t('descEditor.delConnectorConfirm', { name: targetConn.name });
  if (confirm(confirmMsg)) {
    formData.value.connectors.splice(index, 1);
    selectedConnIndex.value = Math.max(0, Math.min(selectedConnIndex.value, formData.value.connectors.length - 1));
  }
}

function handleConnectorNameChange() {
  if (!currentConn.value) return;
  const prefix = currentConn.value.name;
  currentConn.value.pins.forEach((p, idx) => {
    p.label = `${prefix}.${idx + 1}`;
  });
}

function handlePinCountChange(e: Event) {
  if (!currentConn.value) return;
  const target = e.target as HTMLInputElement;
  const newCount = Math.max(1, parseInt(target.value) || 1);
  currentConn.value.pin_count = newCount;

  const currentPins = currentConn.value.pins;
  if (newCount > currentPins.length) {
    const prefix = currentConn.value.name;
    for (let i = currentPins.length + 1; i <= newCount; i++) {
      currentPins.push({
        pin_no: String(i),
        label: `${prefix}.${i}`,
        is_power: false,
        supported_functions: ['NC'],
        default_function: 'NC',
        grid_row: Math.floor((i - 1) / 4) + 1,
        grid_col: ((i - 1) % 4) + 1,
      });
    }
  } else if (newCount < currentPins.length) {
    currentConn.value.pins = currentPins.slice(0, newCount);
  }
}

function onPinModesChange(pin: PinDef, newModes: FunctionType[]) {
  const ruleResult = applyPinModeRules(newModes);
  pin.supported_functions = ruleResult.cleanModes;
  pin.is_power = ruleResult.isPower;

  if (!pin.default_function || !pin.supported_functions.includes(pin.default_function)) {
    pin.default_function = pin.supported_functions[0] || undefined;
  }
}

// 一键将当前连接器未配置 Pin 设为 NC
function setRemainingAsNC() {
  if (!currentConn.value) return;
  currentConn.value.pins.forEach(p => {
    if (!p.supported_functions || p.supported_functions.length === 0) {
      p.supported_functions = ['NC'];
      p.default_function = 'NC';
      p.is_power = false;
    }
  });
}

// 一键将全局所有未配置 Pin 设为 NC
function autoFixAllAsNC() {
  if (!formData.value) return;
  formData.value.connectors.forEach(c => {
    c.pins.forEach(p => {
      const check = isPinConfigured(p);
      if (!check.configured) {
        p.supported_functions = ['NC'];
        p.default_function = 'NC';
        p.is_power = false;
      }
    });
  });
  showValidationModal.value = false;
}

function handleClose() {
  emit('update:modelValue', false);
}

// 保存前进行完备的显式配置检测
function validateAndSave() {
  if (!props.vcu || !formData.value) return;

  // 1. 检测所有端口是否显式配置
  if (unconfiguredPins.value.length > 0) {
    activeTab.value = 'resources';
    showValidationModal.value = true;
    return;
  }

  // 2. 校验全部通过，落盘更新
  const updatedVcu: VcuDescription = {
    id: formData.value.id,
    name: { 'zh-CN': formData.value.nameZh, 'en-US': formData.value.nameEn },
    summary: { 'zh-CN': formData.value.summaryZh, 'en-US': formData.value.summaryEn },
    icon_type: formData.value.icon_type,
    connectors: formData.value.connectors,
  };

  store.updateVcu(updatedVcu);
  store.selectVcu(updatedVcu);
  alert(t('descEditor.saveDescSuccess'));
  emit('update:modelValue', false);
}
</script>