<template>
  <div class="space-y-4">
    <!-- 全部折叠/展开快捷条 -->
    <div class="flex items-center justify-between px-1 text-xs text-slate-500 select-none">
      <div class="flex items-center gap-2">
        <span class="font-medium text-slate-700 dark:text-slate-300">
          {{ t('vcu.connector') }} ({{ store.currentVcu?.connectors.length || 0 }})
        </span>
        <span class="text-[11px] text-slate-400">
          • 共计 {{ totalPinCount }} {{ t('common.unit.pin') }}
        </span>
      </div>
      <div class="flex items-center gap-2">
        <button 
          @click="expandAll" 
          class="hover:text-cyan-500 transition cursor-pointer flex items-center gap-1"
        >
          <ChevronsDown class="w-3.5 h-3.5" /> {{ t('vcu.expandAll') }}
        </button>
        <span>|</span>
        <button 
          @click="collapseAll" 
          class="hover:text-cyan-500 transition cursor-pointer flex items-center gap-1"
        >
          <ChevronsUp class="w-3.5 h-3.5" /> {{ t('vcu.collapseAll') }}
        </button>
      </div>
    </div>

    <!-- 连接器列表 -->
    <div 
      v-for="conn in store.currentVcu?.connectors" 
      :key="conn.id" 
      class="bg-white dark:bg-slate-900/60 rounded-xl border border-slate-200 dark:border-slate-800 shadow-xs overflow-hidden transition-all duration-200"
    >
      <!-- 折叠触发栏 -->
      <div 
        @click="toggleCollapse(conn.id)"
        class="h-12 px-4 flex items-center justify-between cursor-pointer select-none bg-slate-50/60 hover:bg-slate-100/80 dark:bg-slate-900/50 dark:hover:bg-slate-800/50 transition-colors border-b border-transparent"
        :class="{ 'border-slate-200 dark:border-slate-800/80': isExpanded(conn.id) }"
      >
        <div class="flex items-center gap-3">
          <div :class="['transition-transform duration-200 text-slate-400', isExpanded(conn.id) ? 'rotate-90 text-cyan-500' : '']">
            <ChevronRight class="w-4 h-4" />
          </div>

          <div class="flex items-center gap-2">
            <Layers class="w-4 h-4 text-cyan-600 dark:text-cyan-400" />
            <h3 class="text-sm font-semibold text-slate-800 dark:text-slate-200">
              {{ t('vcu.connector') }}: {{ conn.name }}
            </h3>
            <span class="ml-2 text-[10px] bg-slate-200/70 dark:bg-slate-800 text-slate-600 dark:text-slate-400 px-2 py-0.5 rounded-full font-mono">
              {{ conn.pin_count }} Pins
            </span>
          </div>
        </div>

        <div class="flex items-center gap-3">
          <span v-if="searchKeyword" class="text-xs text-cyan-500 font-medium">
            {{ t('vcu.matched') }}: {{ filterPins(conn.pins).length }}
          </span>
          <img 
            :src="getConnectorIcon(conn.pin_count)" 
            class="h-6 opacity-70 group-hover:opacity-100 transition" 
          />
        </div>
      </div>

      <!-- 展开内容区 -->
      <div v-show="isExpanded(conn.id)" class="p-5 pt-4 transition-all">
        <div class="grid grid-cols-2 md:grid-cols-3 xl:grid-cols-4 gap-3.5">
          <div 
            v-for="pin in filterPins(conn.pins)" 
            :key="pin.label"
            :id="`pin-card-${pin.label}`"
            :class="[
              'p-3 rounded-lg border transition-all duration-300 relative',
              highlightedPin === pin.label 
                ? 'ring-2 ring-rose-500 scale-[1.02] shadow-lg shadow-rose-500/20' 
                : '',
              pin.is_power 
                ? 'bg-amber-50/60 dark:bg-slate-900/90 border-amber-300 dark:border-amber-900/40 text-amber-700 dark:text-amber-300' 
                : 'bg-white dark:bg-slate-950 border-slate-200 dark:border-slate-800 hover:border-slate-300 dark:hover:border-slate-700 shadow-2xs'
            ]"
          >
            <div class="flex items-center justify-between text-xs mb-2">
              <span class="font-bold tracking-wider font-mono">{{ pin.label }}</span>
              <span v-if="pin.is_power" class="text-[10px] bg-amber-100 dark:bg-amber-950/60 text-amber-700 dark:text-amber-400 border border-amber-300 dark:border-amber-800 px-1.5 py-0.2 rounded font-mono">
                {{ t('vcu.powerBadge') }}
              </span>
              <span v-else class="text-[10px] text-slate-400 dark:text-slate-500 font-mono">#{{ pin.pin_no }}</span>
            </div>

            <div class="relative">
              <select 
                :disabled="pin.is_power"
                :value="store.currentConfig?.assignments[pin.label]?.selected_function"
                @change="(e: any) => store.updatePinFunction(pin.label, e.target.value)"
                class="w-full bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700/80 rounded px-2 py-1 text-xs text-slate-800 dark:text-slate-200 focus:outline-none focus:border-cyan-500 disabled:opacity-60 disabled:cursor-not-allowed cursor-pointer"
              >
                <option v-for="fn in pin.supported_functions" :key="fn" :value="fn">{{ fn }}</option>
              </select>
            </div>

            <div v-if="['AIU', 'AIR', 'AII'].includes(store.currentConfig?.assignments[pin.label]?.selected_function || '')" class="mt-2 pt-2 border-t border-slate-100 dark:border-slate-800/80 flex items-center gap-1.5">
              <span class="text-[10px] text-slate-500">{{ t('vcu.range') }}:</span>
              <input type="number" value="0" class="w-10 bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 px-1 py-0.5 text-[10px] text-center rounded text-slate-800 dark:text-slate-200" />
              <span class="text-[10px] text-slate-500">~</span>
              <input type="number" value="5" class="w-10 bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 px-1 py-0.5 text-[10px] text-center rounded text-slate-800 dark:text-slate-200" />
              <span class="text-[10px] text-cyan-600 dark:text-cyan-400">{{ t('common.unit.voltage') }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick } from 'vue';
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import type { PinDef } from '@/types/vcu';
import { Layers, ChevronRight, ChevronsDown, ChevronsUp } from '@lucide/vue';

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

function expandAll() {
  store.currentVcu?.connectors.forEach(c => expandedIds.value.add(c.id));
}

function collapseAll() {
  expandedIds.value.clear();
}

const totalPinCount = computed(() => {
  return store.currentVcu?.connectors.reduce((sum, c) => sum + c.pin_count, 0) || 0;
});

function filterPins(pins: PinDef[]) {
  if (!props.searchKeyword) return pins;
  return pins.filter(p => p.label.toLowerCase().includes(props.searchKeyword.toLowerCase()));
}

function getConnectorIcon(count: number): string {
  if (count <= 8) return '/src/assets/icons/connector-8pin.svg';
  if (count <= 12) return '/src/assets/icons/connector-12pin.svg';
  return '/src/assets/icons/connector-16pin.svg';
}

async function scrollToPin(pinLabel: string) {
  const targetConn = store.currentVcu?.connectors.find(c => c.pins.some(p => p.label === pinLabel));
  if (targetConn) {
    expandedIds.value.add(targetConn.id);
  }
  await nextTick();
  const el = document.getElementById(`pin-card-${pinLabel}`);
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