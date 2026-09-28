<template>
  <div class="space-y-2">
    <div 
      v-for="vcu in list" 
      :key="vcu.id"
      @click="$emit('select', vcu)"
      @dblclick="$emit('select', vcu)"
      @contextmenu.prevent="$emit('open-context',$event, vcu)"
      :title="resolveI18nText(vcu.summary, locale)"
      :class="[
        'p-3 rounded-lg border cursor-pointer transition flex gap-3 relative group', 
        store.currentVcu?.id === vcu.id 
          ? 'bg-cyan-50/80 dark:bg-cyan-950/30 border-cyan-500/60 dark:border-cyan-500/50' 
          : 'bg-white dark:bg-slate-900/80 border-slate-200 dark:border-slate-800 hover:border-slate-300 dark:hover:border-slate-700'
      ]"
    >
      <img :src="`/src/assets/icons/${vcu.icon_type}.svg`" class="w-10 h-10 object-contain p-1 bg-slate-50 dark:bg-slate-950 rounded border border-slate-200 dark:border-slate-800 shrink-0" />
      <div class="flex-1 min-w-0 pr-12">
        <h4 class="text-xs font-semibold text-slate-800 dark:text-slate-200 truncate">
          {{ resolveI18nText(vcu.name, locale) }}
        </h4>
        <p class="text-[11px] text-slate-500 dark:text-slate-400 mt-1 line-clamp-2 leading-relaxed">
          {{ resolveI18nText(vcu.summary, locale) }}
        </p>
      </div>

      <!-- 悬浮快捷操作栏 (编辑/删除) -->
      <div class="absolute right-2 top-2 opacity-0 group-hover:opacity-100 transition-opacity flex items-center gap-1 bg-white/90 dark:bg-slate-900/90 rounded border border-slate-200 dark:border-slate-800 p-0.5 shadow-xs">
        <button 
          @click.stop="$emit('edit', vcu)" 
          class="p-1 hover:bg-slate-100 dark:hover:bg-slate-800 rounded text-slate-500 hover:text-cyan-600 transition"
          title="编辑 VCU 描述"
        >
          <Edit3 class="w-3.5 h-3.5" />
        </button>
        <button 
          @click.stop="$emit('delete', vcu)" 
          class="p-1 hover:bg-rose-50 dark:hover:bg-rose-950/40 rounded text-slate-500 hover:text-rose-600 transition"
          title="删除 VCU"
        >
          <Trash2 class="w-3.5 h-3.5" />
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import { resolveI18nText, type VcuDescription } from '@/types/vcu';
import { Edit3, Trash2 } from 'lucide-vue-next';

defineProps<{ list: VcuDescription[] }>();
defineEmits<{
  (e: 'select', vcu: VcuDescription): void;
  (e: 'open-context', event: MouseEvent, vcu: VcuDescription): void;
  (e: 'edit', vcu: VcuDescription): void;
  (e: 'delete', vcu: VcuDescription): void;
}>();

const { locale } = useI18n();
const store = useVcuStore();
</script>