<template>
  <div class="border border-slate-200 dark:border-slate-800 rounded-lg overflow-hidden">
    <table class="w-full text-left text-xs">
      <thead class="bg-slate-50 dark:bg-slate-800/60 border-b border-slate-200 dark:border-slate-800 text-slate-500">
        <tr>
          <th class="p-2 font-medium">VCU</th>
          <th class="p-2 font-medium">{{ t('vcu.summary') }}</th>
        </tr>
      </thead>
      <tbody class="divide-y divide-slate-200 dark:divide-slate-800">
        <tr 
          v-for="vcu in list" 
          :key="vcu.id"
          @click="$emit('select', vcu)"
          @dblclick="$emit('select', vcu)"
          @contextmenu.prevent="$emit('open-context', $event, vcu)"
          :class="[
            'cursor-pointer transition',
            store.currentVcu?.id === vcu.id ? 'bg-cyan-50/80 dark:bg-cyan-950/40 text-cyan-600 dark:text-cyan-400' : 'hover:bg-slate-50 dark:hover:bg-slate-800/40'
          ]"
        >
          <td class="p-2 font-semibold truncate max-w-[100px]">
            {{ resolveI18nText(vcu.name, locale) }}
          </td>
          <td class="p-2 text-slate-500 dark:text-slate-400 truncate max-w-[140px]">
            {{ resolveI18nText(vcu.summary, locale) }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import { resolveI18nText, type VcuDescription } from '@/types/vcu';

defineProps<{ list: VcuDescription[] }>();
defineEmits<{
  (e: 'select', vcu: VcuDescription): void;
  (e: 'open-context', event: MouseEvent, vcu: VcuDescription): void;
}>();

const { t, locale } = useI18n();
const store = useVcuStore();
</script>