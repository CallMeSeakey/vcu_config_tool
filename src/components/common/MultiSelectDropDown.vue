<template>
  <div class="relative w-full" ref="containerRef">
    <!-- 触发输入容器 -->
    <div 
      @click="toggleDropdown" 
      class="min-h-[30px] w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700/80 rounded-md px-2 py-1 flex flex-wrap items-center gap-1.5 cursor-pointer text-xs transition focus-within:border-cyan-500"
    >
      <!-- 选中的标签列表 -->
      <span 
        v-for="item in modelValue" 
        :key="item"
        :class="[
          'inline-flex items-center gap-1 px-1.5 py-0.5 rounded text-[11px] font-mono transition',
          isPowerMode(item) 
            ? 'bg-amber-100 dark:bg-amber-950/80 text-amber-800 dark:text-amber-300 border border-amber-300 dark:border-amber-800' 
            : 'bg-cyan-50 dark:bg-cyan-950/80 text-cyan-800 dark:text-cyan-300 border border-cyan-200 dark:border-cyan-800'
        ]"
      >
        <span>{{ item }}</span>
        <button 
          type="button" 
          @click.stop="removeItem(item)" 
          class="hover:text-rose-500 rounded p-0.5 transition cursor-pointer"
        >
          <X class="w-3 h-3" />
        </button>
      </span>

      <!-- 占位文案 -->
      <span v-if="!modelValue.length" class="text-slate-400 select-none">
        {{ placeholder || t('descEditor.selectModePlaceholder') }}
      </span>

      <ChevronDown class="w-3.5 h-3.5 ml-auto text-slate-400 shrink-0" />
    </div>

    <!-- 下拉面板 -->
    <div 
      v-if="isOpen" 
      class="absolute left-0 top-full mt-1 w-full bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-lg shadow-xl z-50 p-1.5 max-h-48 overflow-y-auto"
    >
      <div 
        v-for="opt in availableOptions" 
        :key="opt"
        @click="selectItem(opt)"
        :class="[
          'px-2 py-1 rounded text-xs cursor-pointer flex items-center justify-between transition',
          modelValue.includes(opt) 
            ? 'bg-cyan-50 dark:bg-cyan-950/60 text-cyan-600 dark:text-cyan-400 font-medium' 
            : 'hover:bg-slate-100 dark:hover:bg-slate-800 text-slate-700 dark:text-slate-200'
        ]"
      >
        <span class="font-mono">{{ opt }}</span>
        <Check v-if="modelValue.includes(opt)" class="w-3.5 h-3.5 text-cyan-500" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { useI18n } from 'vue-i18n';
import type { FunctionType } from '@/types/vcu';
import { POWER_MODES } from '@/utils/pinRules';
import { X, ChevronDown, Check } from '@lucide/vue';

const props = defineProps<{
  modelValue: FunctionType[];
  options: FunctionType[];
  placeholder?: string;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', val: FunctionType[]): void;
  (e: 'change', val: FunctionType[]): void;
}>();

const { t } = useI18n();
const isOpen = ref(false);
const containerRef = ref<HTMLElement | null>(null);

function isPowerMode(fn: FunctionType) {
  return POWER_MODES.includes(fn);
}

function toggleDropdown() {
  isOpen.value = !isOpen.value;
}

function selectItem(opt: FunctionType) {
  let updated = [...props.modelValue];
  if (updated.includes(opt)) {
    updated = updated.filter(m => m !== opt);
  } else {
    updated.push(opt);
  }
  emit('update:modelValue', updated);
  emit('change', updated);
}

function removeItem(opt: FunctionType) {
  const updated = props.modelValue.filter(m => m !== opt);
  emit('update:modelValue', updated);
  emit('change', updated);
}

// 点击外部关闭下拉
function handleClickOutside(e: MouseEvent) {
  if (containerRef.value && !containerRef.value.contains(e.target as Node)) {
    isOpen.value = false;
  }
}

onMounted(() => {
  window.addEventListener('click', handleClickOutside);
});
onUnmounted(() => {
  window.removeEventListener('click', handleClickOutside);
});

const availableOptions = props.options;
</script>