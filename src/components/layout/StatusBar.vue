<template>
  <footer class="h-6 bg-slate-100 dark:bg-slate-900 border-t border-slate-200 dark:border-slate-800 px-3 flex items-center justify-between text-[11px] text-slate-500 dark:text-slate-400 select-none">
    <div class="flex items-center gap-4">
      <!-- 可点击的诊断规则入口 -->
      <button 
        @click="$emit('open-problems')" 
        class="flex items-center gap-1.5 px-1.5 py-0.5 rounded hover:bg-slate-200 dark:hover:bg-slate-800 transition cursor-pointer"
        :title="t('problems.title')"
      >
        <span class="w-2 h-2 rounded-full" :class="store.validationLogs.errors.length ? 'bg-rose-500 animate-pulse' : 'bg-emerald-500'"></span>
        <span>{{ t('vcu.rules.ruleStatus') }}:</span>
        <span :class="store.validationLogs.errors.length ? 'text-rose-500 font-bold underline decoration-rose-500/50' : 'text-emerald-500'">
          {{ store.validationLogs.errors.length ? `${store.validationLogs.errors.length} ${t('vcu.rules.conflictItems')}` : t('common.status.normal') }}
        </span>
      </button>

      <span v-if="store.validationLogs.warnings.length" class="text-amber-500">
        ({{ store.validationLogs.warnings.length }} {{ t('vcu.rules.warningItems') }})
      </span>
    </div>

    <div class="flex items-center gap-4 font-mono">
      <span>VCU Core: Rust v2</span>
      <span>UDS: Standby</span>
    </div>
  </footer>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';

defineEmits<{(e: 'open-problems'): void}>();

const { t } = useI18n();
const store = useVcuStore();
</script>