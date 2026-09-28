<template>
  <BaseModal 
    :model-value="modelValue" 
    :title="t('descValidation.modalTitle')" 
    size="lg" 
    @close="emit('update:modelValue', false)"
  >
    <div class="space-y-4">
      <!-- 汇总警告 -->
      <div class="p-3 bg-rose-50 dark:bg-rose-950/40 border border-rose-200 dark:border-rose-900/60 rounded-xl flex items-start gap-3">
        <AlertTriangle class="w-5 h-5 text-rose-500 shrink-0 mt-0.5" />
        <div class="text-xs">
          <p class="font-semibold text-rose-700 dark:text-rose-300">
            {{ t('descValidation.summaryError', { count: uniqueFailedFiles.length }) }}
          </p>
          <p class="text-slate-500 dark:text-slate-400 mt-0.5">
            {{ t('descValidation.jumpTip') }}
          </p>
        </div>
      </div>

      <!-- 不合规项清单 -->
      <div class="max-h-[380px] overflow-y-auto space-y-2.5 pr-1">
        <div 
          v-for="(err, idx) in errors" 
          :key="idx"
          class="p-3 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl flex flex-col gap-1.5 shadow-2xs hover:border-rose-300 dark:hover:border-rose-800 transition"
        >
          <div class="flex items-center justify-between text-xs">
            <!-- 异常文件与引脚 -->
            <div class="flex items-center gap-2">
              <FileCode class="w-4 h-4 text-slate-400" />
              <span class="font-bold text-slate-800 dark:text-slate-200 font-mono">{{ err.file_name }}</span>
              <span v-if="err.pin_label" class="px-1.5 py-0.2 bg-slate-100 dark:bg-slate-800 text-slate-600 dark:text-slate-300 rounded font-mono text-[11px]">
                {{ err.pin_label }}
              </span>
            </div>

            <!-- 行号标签提示 -->
            <span v-if="err.line" class="text-[10px] font-mono bg-rose-50 dark:bg-rose-950/60 text-rose-600 dark:text-rose-400 border border-rose-200 dark:border-rose-800 px-2 py-0.5 rounded-full">
              {{ t('descValidation.locLine', { line: err.line }) }}
            </span>
          </div>

          <!-- 具体违背的规则描述 -->
          <p class="text-xs text-rose-600 dark:text-rose-400 pl-6 leading-relaxed">
            • {{ err.message }}
          </p>
        </div>
      </div>
    </div>

    <template #footer>
      <button 
        @click="emit('update:modelValue', false)" 
        class="px-4 py-1.5 bg-slate-800 hover:bg-slate-700 text-white text-xs font-semibold rounded-lg transition cursor-pointer"
      >
        {{ t('common.action.confirm') }}
      </button>
    </template>
  </BaseModal>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useI18n } from 'vue-i18n';
import BaseModal from '@/components/common/BaseModal.vue';
import { AlertTriangle, FileCode } from '@lucide/vue';

export interface DescValidationError {
  file_name: string;
  pin_label?: string;
  line?: number;
  message: string;
}

const props = defineProps<{
  modelValue: boolean;
  errors: DescValidationError[];
}>();

const emit = defineEmits<{(e: 'update:modelValue', val: boolean): void}>();

const { t } = useI18n();

const uniqueFailedFiles = computed(() => {
  return Array.from(new Set(props.errors.map(e => e.file_name)));
});
</script>