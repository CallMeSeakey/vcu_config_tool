<template>
  <BaseModal :model-value="modelValue" :title="title" size="sm" @close="handleCancel">
    <div class="py-2 text-slate-600 dark:text-slate-300 leading-relaxed">
      {{ message }}
    </div>
    <template #footer>
      <button 
        @click="handleCancel" 
        class="px-3 py-1.5 rounded border border-slate-200 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-800 text-slate-700 dark:text-slate-300 transition"
      >
        {{ t('common.action.cancel') }}
      </button>
      <button 
        @click="handleConfirm" 
        class="px-3 py-1.5 rounded bg-rose-600 hover:bg-rose-500 text-white font-medium transition shadow-xs"
      >
        {{ t('common.action.confirm') }}
      </button>
    </template>
  </BaseModal>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n';
import BaseModal from './BaseModal.vue';

const { t } = useI18n();

defineProps<{
  modelValue: boolean;
  title: string;
  message: string;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void;
  (e: 'confirm'): void;
  (e: 'cancel'): void;
}>();

function handleConfirm() {
  emit('confirm');
  emit('update:modelValue', false);
}

function handleCancel() {
  emit('cancel');
  emit('update:modelValue', false);
}
</script>