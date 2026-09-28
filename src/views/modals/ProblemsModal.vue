<template>
  <BaseModal :model-value="modelValue" :title="t('problems.title')" size="md" @close="emit('update:modelValue', false)">
    <div class="space-y-3">
      <div class="text-[11px] text-slate-500 pb-1 border-b border-slate-200 dark:border-slate-800 flex items-center justify-between">
        <span>{{ t('problems.jumpTip') }}</span>
        <div class="flex items-center gap-2">
          <span v-if="store.validationLogs.errors.length" class="text-rose-500 font-bold flex items-center gap-1">
            <AlertCircle class="w-3.5 h-3.5" /> {{ store.validationLogs.errors.length }} {{ t('problems.errorCount') }}
          </span>
          <span v-if="store.validationLogs.warnings.length" class="text-amber-500 font-bold flex items-center gap-1">
            <AlertTriangle class="w-3.5 h-3.5" /> {{ store.validationLogs.warnings.length }} {{ t('problems.warningCount') }}
          </span>
        </div>
      </div>

      <!-- 无冲突提示 -->
      <div v-if="!store.validationLogs.errors.length && !store.validationLogs.warnings.length" class="py-8 text-center text-slate-400">
        <CheckCircle2 class="w-10 h-10 mx-auto text-emerald-500 mb-2 opacity-80" />
        <p>{{ t('problems.empty') }}</p>
      </div>

      <!-- 错误条目列表 -->
      <div class="space-y-1.5 max-h-[360px] overflow-y-auto">
        <div 
          v-for="(err, idx) in store.validationLogs.errors" 
          :key="`err-${idx}`"
          @click="handleJump(err)"
          class="p-2.5 rounded-lg border border-rose-200 dark:border-rose-950/60 bg-rose-50/50 dark:bg-rose-950/20 hover:bg-rose-100/60 dark:hover:bg-rose-950/40 cursor-pointer transition flex items-start gap-2.5 group"
        >
          <AlertCircle class="w-4 h-4 text-rose-500 shrink-0 mt-0.5" />
          <div class="flex-1 text-xs">
            <div class="text-rose-700 dark:text-rose-300 font-medium group-hover:underline">
              {{ err }}
            </div>
            <span class="text-[10px] text-rose-400 font-mono mt-0.5 block">{{ t('problems.clickToLocate') }}</span>
          </div>
          <ArrowRight class="w-4 h-4 text-rose-400 opacity-0 group-hover:opacity-100 transition shrink-0 self-center" />
        </div>

        <div 
          v-for="(warn, idx) in store.validationLogs.warnings" 
          :key="`warn-${idx}`"
          @click="handleJump(warn)"
          class="p-2.5 rounded-lg border border-amber-200 dark:border-amber-950/60 bg-amber-50/50 dark:bg-amber-950/20 hover:bg-amber-100/60 dark:hover:bg-amber-950/40 cursor-pointer transition flex items-start gap-2.5 group"
        >
          <AlertTriangle class="w-4 h-4 text-amber-500 shrink-0 mt-0.5" />
          <div class="flex-1 text-xs">
            <div class="text-amber-800 dark:text-amber-300 font-medium group-hover:underline">
              {{ warn }}
            </div>
            <span class="text-[10px] text-amber-500/70 font-mono mt-0.5 block">{{ t('problems.clickToLocate') }}</span>
          </div>
          <ArrowRight class="w-4 h-4 text-amber-400 opacity-0 group-hover:opacity-100 transition shrink-0 self-center" />
        </div>
      </div>
    </div>

    <template #footer>
      <button 
        @click="emit('update:modelValue', false)" 
        class="px-3 py-1.5 rounded bg-slate-800 hover:bg-slate-700 text-white font-medium transition cursor-pointer"
      >
        {{ t('common.action.close') }}
      </button>
    </template>
  </BaseModal>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import BaseModal from '@/components/common/BaseModal.vue';
import { AlertCircle, AlertTriangle, CheckCircle2, ArrowRight } from 'lucide-vue-next';

defineProps<{ modelValue: boolean }>();
const emit = defineEmits<{
  (e: 'update:modelValue', val: boolean): void;
  (e: 'jump-to-pin', pinLabel: string): void;
}>();

const { t } = useI18n();
const store = useVcuStore();

function handleJump(msg: string) {
  const match = msg.match(/[A-Z0-9]+\.[0-9]+/);
  if (match) {
    emit('jump-to-pin', match[0]);
    emit('update:modelValue', false);
  }
}
</script>