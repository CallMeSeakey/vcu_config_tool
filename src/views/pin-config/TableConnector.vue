<template>
  <div class="space-y-5">
    <div 
      v-for="(conn, idx) in store.currentVcu?.connectors" 
      :key="conn.id" 
      class="rounded-xl border shadow-xs overflow-hidden transition-all duration-200"
      :class="getConnectorTheme(idx).containerClass"
    >
      <!-- 分组标题栏 -->
      <div 
        @click="toggleCollapse(conn.id)"
        class="h-10 px-4 flex items-center justify-between cursor-pointer select-none transition-colors"
        :class="getConnectorTheme(idx).headerClass"
      >
        <div class="flex items-center gap-2">
          <ChevronRight :class="['w-4 h-4 transition-transform duration-200', isExpanded(conn.id) ? 'rotate-90' : '']" />
          <span class="font-bold text-xs font-mono">{{ conn.name }}</span>
          <span class="text-[11px] opacity-70">({{ conn.pin_count }} Pins)</span>
        </div>
        <span class="text-[10px] font-mono font-medium px-2 py-0.5 rounded-full border" :class="getConnectorTheme(idx).badgeClass">
          CONNECTOR #{{ idx + 1 }}
        </span>
      </div>

      <!-- 列表主体表格 -->
      <div v-show="isExpanded(conn.id)">
        <table class="w-full text-left text-xs">
          <thead class="border-b text-slate-400 select-none text-[11px]" :class="getConnectorTheme(idx).theadClass">
            <tr>
              <th class="p-2.5 pl-6 font-medium">{{ t('vcu.pinNo') }}</th>
              <th class="p-2.5 font-medium">{{ t('vcu.function') }}</th>
              <th class="p-2.5 font-medium">{{ t('vcu.range') }}</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 dark:divide-slate-800/40">
            <tr 
              v-for="pin in filterPins(conn.pins)" 
              :key="pin.label" 
              :id="`table-pin-row-${pin.label}`"
              :class="[
                'transition-colors duration-200',
                highlightedPin === pin.label ? 'bg-rose-500/20 font-bold' : getConnectorTheme(idx).rowHoverClass
              ]"
            >
              <td class="p-2.5 pl-6 font-mono font-bold text-slate-700 dark:text-slate-300">
                {{ pin.label }}
                <span v-if="pin.is_power" class="ml-2 text-[10px] bg-amber-100 dark:bg-amber-950/60 text-amber-700 dark:text-amber-400 border border-amber-300 dark:border-amber-800 px-1 py-0.2 rounded font-mono">
                  POWER
                </span>
              </td>
              <td class="p-2.5">
                <select 
                  :disabled="pin.is_power"
                  :value="store.currentConfig?.assignments[pin.label]?.selected_function"
                  @change="(e: any) => store.updatePinFunction(pin.label, e.target.value)"
                  class="bg-white/80 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded px-2 py-1 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500 disabled:opacity-60 cursor-pointer"
                >
                  <option v-for="fn in pin.supported_functions" :key="fn" :value="fn">{{ fn }}</option>
                </select>
              </td>
              <td class="p-2.5">
                <span v-if="['AIU', 'AIR', 'AII'].includes(store.currentConfig?.assignments[pin.label]?.selected_function || '')" class="text-slate-500 font-mono">
                  0 ~ 5 V
                </span>
                <span v-else class="text-slate-400">-</span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue';
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import type { PinDef } from '@/types/vcu';
import { ChevronRight } from 'lucide-vue-next';

const props = defineProps<{ searchKeyword: string }>();
const { t } = useI18n();
const store = useVcuStore();

const expandedIds = ref<Set<string>>(new Set(['conn-jp1', 'conn-jp2', 'conn-jp3', 'conn-jp4', 'conn-jp5']));
const highlightedPin = ref<string | null>(null);

function isExpanded(id: string) {
  return expandedIds.value.has(id);
}

function toggleCollapse(id: string) {
  if (expandedIds.value.has(id)) {
    expandedIds.value.delete(id);
  } else {
    expandedIds.value.add(id);
  }
}

function filterPins(pins: PinDef[]) {
  if (!props.searchKeyword) return pins;
  return pins.filter(p => p.label.toLowerCase().includes(props.searchKeyword.toLowerCase()));
}

// 5种柔和风格的连接器边界配色主题
const THEMES = [
  {
    containerClass: 'bg-cyan-50/20 dark:bg-cyan-950/10 border-cyan-200/80 dark:border-cyan-900/40',
    headerClass: 'bg-cyan-100/40 dark:bg-cyan-900/30 text-cyan-900 dark:text-cyan-200 border-b border-cyan-200/60 dark:border-cyan-900/40',
    badgeClass: 'bg-cyan-100 text-cyan-700 border-cyan-300 dark:bg-cyan-900/50 dark:text-cyan-300 dark:border-cyan-800',
    theadClass: 'bg-cyan-50/30 dark:bg-cyan-950/20 border-cyan-100 dark:border-cyan-900/30',
    rowHoverClass: 'hover:bg-cyan-50/60 dark:hover:bg-cyan-900/20',
  },
  {
    containerClass: 'bg-indigo-50/20 dark:bg-indigo-950/10 border-indigo-200/80 dark:border-indigo-900/40',
    headerClass: 'bg-indigo-100/40 dark:bg-indigo-900/30 text-indigo-900 dark:text-indigo-200 border-b border-indigo-200/60 dark:border-indigo-900/40',
    badgeClass: 'bg-indigo-100 text-indigo-700 border-indigo-300 dark:bg-indigo-900/50 dark:text-indigo-300 dark:border-indigo-800',
    theadClass: 'bg-indigo-50/30 dark:bg-indigo-950/20 border-indigo-100 dark:border-indigo-900/30',
    rowHoverClass: 'hover:bg-indigo-50/60 dark:hover:bg-indigo-900/20',
  },
  {
    containerClass: 'bg-emerald-50/20 dark:bg-emerald-950/10 border-emerald-200/80 dark:border-emerald-900/40',
    headerClass: 'bg-emerald-100/40 dark:bg-emerald-900/30 text-emerald-900 dark:text-emerald-200 border-b border-emerald-200/60 dark:border-emerald-900/40',
    badgeClass: 'bg-emerald-100 text-emerald-700 border-emerald-300 dark:bg-emerald-900/50 dark:text-emerald-300 dark:border-emerald-800',
    theadClass: 'bg-emerald-50/30 dark:bg-emerald-950/20 border-emerald-100 dark:border-emerald-900/30',
    rowHoverClass: 'hover:bg-emerald-50/60 dark:hover:bg-emerald-900/20',
  },
  {
    containerClass: 'bg-amber-50/20 dark:bg-amber-950/10 border-amber-200/80 dark:border-amber-900/40',
    headerClass: 'bg-amber-100/40 dark:bg-amber-900/30 text-amber-900 dark:text-amber-200 border-b border-amber-200/60 dark:border-amber-900/40',
    badgeClass: 'bg-amber-100 text-amber-700 border-amber-300 dark:bg-amber-900/50 dark:text-amber-300 dark:border-amber-800',
    theadClass: 'bg-amber-50/30 dark:bg-amber-950/20 border-amber-100 dark:border-amber-900/30',
    rowHoverClass: 'hover:bg-amber-50/60 dark:hover:bg-amber-900/20',
  },
  {
    containerClass: 'bg-purple-50/20 dark:bg-purple-950/10 border-purple-200/80 dark:border-purple-900/40',
    headerClass: 'bg-purple-100/40 dark:bg-purple-900/30 text-purple-900 dark:text-purple-200 border-b border-purple-200/60 dark:border-purple-900/40',
    badgeClass: 'bg-purple-100 text-purple-700 border-purple-300 dark:bg-purple-900/50 dark:text-purple-300 dark:border-purple-800',
    theadClass: 'bg-purple-50/30 dark:bg-purple-950/20 border-purple-100 dark:border-purple-900/30',
    rowHoverClass: 'hover:bg-purple-50/60 dark:hover:bg-purple-900/20',
  }
];

function getConnectorTheme(idx: number) {
  return THEMES[idx % THEMES.length];
}

async function scrollToPin(pinLabel: string) {
  const targetConn = store.currentVcu?.connectors.find(c => c.pins.some(p => p.label === pinLabel));
  if (targetConn) {
    expandedIds.value.add(targetConn.id);
  }
  await nextTick();
  const el = document.getElementById(`table-pin-row-${pinLabel}`);
  if (el) {
    el.scrollIntoView({ behavior: 'smooth', block: 'center' });
    highlightedPin.value = pinLabel;
    setTimeout(() => {
      highlightedPin.value = null;
    }, 2500);
  }
}

defineExpose({ scrollToPin });
</script>