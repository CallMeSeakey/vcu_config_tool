<template>
  <BaseModal :model-value="modelValue" :title="t('settings.title')" size="md" @close="emit('update:modelValue', false)">
    <div class="space-y-4">
      <div class="space-y-1">
        <label class="font-medium text-slate-700 dark:text-slate-300">{{ t('settings.workdir') }}</label>
        <input readonly value="saved_configs/" class="w-full bg-slate-100 dark:bg-slate-950 border border-slate-200 dark:border-slate-800 rounded p-1.5 text-xs text-slate-500 font-mono" />
      </div>
      <div class="flex items-center justify-between pt-2">
        <span class="text-slate-700 dark:text-slate-300 font-medium">{{ t('settings.autoSave') }}</span>
        <input type="checkbox" checked class="rounded border-slate-300 text-cyan-600 focus:ring-cyan-500 cursor-pointer" />
      </div>
      <div class="space-y-1">
        <label class="font-medium text-slate-700 dark:text-slate-300">{{ t('settings.protocolVersion') }}</label>
        <select class="w-full bg-slate-50 dark:bg-slate-950 border border-slate-200 dark:border-slate-800 rounded p-1.5 text-xs text-slate-800 dark:text-slate-200 cursor-pointer">
          <option>ISO 14229-1:2020 (UDS on CAN)</option>
          <option>ISO 14229-2:2013</option>
        </select>
      </div>
    </div>
    <template #footer>
      <button @click="emit('update:modelValue', false)" class="px-3 py-1.5 bg-cyan-600 hover:bg-cyan-500 text-white rounded font-medium transition cursor-pointer">
        {{ t('common.action.confirm') }}
      </button>
    </template>
  </BaseModal>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n';
import BaseModal from '@/components/common/BaseModal.vue';

defineProps<{ modelValue: boolean }>();
const emit = defineEmits<{(e: 'update:modelValue', val: boolean): void}>();
const { t } = useI18n();
</script>