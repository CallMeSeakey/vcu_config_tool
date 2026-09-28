<template>
  <aside 
    :style="{ width: `${store.sidebarWidth}px` }"
    class="bg-white dark:bg-slate-900/40 border-r border-slate-200 dark:border-slate-800/80 flex flex-col shrink-0 select-none relative"
  >
    <!-- 顶部：搜索与资源筛选 -->
    <div class="p-3 border-b border-slate-200 dark:border-slate-800 space-y-2">
      <div class="relative">
        <Search class="w-3.5 h-3.5 absolute left-2.5 top-2.5 text-slate-400 dark:text-slate-500" />
        <input 
          v-model="store.searchQuery" 
          :placeholder="t('sidebar.searchPlaceholder')" 
          class="w-full bg-slate-50 dark:bg-slate-950 border border-slate-200 dark:border-slate-700/60 rounded px-2.5 py-1.5 pl-8 text-xs text-slate-900 dark:text-slate-200 focus:outline-none focus:border-cyan-500"
        />
      </div>

      <div class="flex items-center justify-between text-xs pt-1">
        <div class="flex items-center gap-1.5 text-slate-500">
          <span>{{ t('sidebar.filterDoh') }}</span>
          <input 
            type="number" 
            v-model.number="store.filterDohCount" 
            min="0" 
            class="w-10 bg-slate-50 dark:bg-slate-950 border border-slate-200 dark:border-slate-700 rounded px-1 text-center text-slate-800 dark:text-slate-200 text-xs py-0.5"
          />
        </div>

        <div class="flex items-center gap-1 border border-slate-200 dark:border-slate-800 rounded p-0.5">
          <button 
            @click="store.viewMode = 'icon'" 
            :class="['p-1 rounded cursor-pointer', store.viewMode === 'icon' ? 'bg-slate-200 dark:bg-slate-800 text-cyan-600 dark:text-cyan-400' : 'text-slate-400']"
            :title="t('sidebar.viewIcon')"
          >
            <LayoutGrid class="w-3.5 h-3.5" />
          </button>
          <button 
            @click="store.viewMode = 'list'" 
            :class="['p-1 rounded cursor-pointer', store.viewMode === 'list' ? 'bg-slate-200 dark:bg-slate-800 text-cyan-600 dark:text-cyan-400' : 'text-slate-400']"
            :title="t('sidebar.viewList')"
          >
            <List class="w-3.5 h-3.5" />
          </button>
        </div>
      </div>
    </div>

    <!-- 中间：VCU 列表卡片区 -->
    <div class="flex-1 overflow-y-auto p-2" @contextmenu.prevent>
      <VcuCardView 
        v-if="store.viewMode === 'icon'" 
        :list="store.filteredVcus" 
        @select="(vcu) => store.selectVcu(vcu)"
        @open-context="handleContextMenu"
        @edit="openEditModal"
        @delete="confirmDeleteVcu"
      />
      <VcuTableView 
        v-else 
        :list="store.filteredVcus" 
        @select="(vcu) => store.selectVcu(vcu)"
        @open-context="handleContextMenu"
      />
    </div>

    <!-- 底部：新建与导入按钮 -->
    <div class="p-3 border-t border-slate-200 dark:border-slate-800 grid grid-cols-2 gap-2 bg-slate-50/50 dark:bg-slate-900/30">
      <button 
        @click="$emit('request-create')" 
        class="h-9 flex items-center justify-center gap-1.5 bg-cyan-600 hover:bg-cyan-500 text-white text-xs rounded-lg font-semibold transition shadow-xs cursor-pointer"
      >
        <Plus class="w-4 h-4" /> {{ t('sidebar.newVcu') }}
      </button>
      <button 
        @click="$emit('request-import')" 
        class="h-9 flex items-center justify-center gap-1.5 bg-white hover:bg-slate-100 dark:bg-slate-800 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-200 text-xs rounded-lg font-semibold transition border border-slate-200 dark:border-slate-700 cursor-pointer"
      >
        <Upload class="w-4 h-4" /> {{ t('sidebar.importDesc') }}
      </button>
    </div>

    <!-- 宽度调节拖拽把手 (Resize Bar) -->
    <div 
      @mousedown="startResize"
      class="absolute -right-1 top-0 bottom-0 w-2 cursor-col-resize hover:bg-cyan-500/50 transition-colors z-30"
      :class="{ 'bg-cyan-500': isResizing }"
    ></div>

    <!-- 右键上下文菜单 -->
    <div 
      v-if="contextMenu.visible" 
      :style="{ top: `${contextMenu.y}px`, left: `${contextMenu.x}px` }"
      class="fixed z-50 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-lg shadow-xl py-1 w-44 text-xs"
      @mouseleave="contextMenu.visible = false"
    >
      <button @click="triggerEditFromContext" class="w-full text-left px-3 py-1.5 hover:bg-slate-100 dark:hover:bg-slate-800 text-slate-700 dark:text-slate-200 flex items-center gap-2 cursor-pointer">
        <Edit3 class="w-3.5 h-3.5 text-cyan-500" /> {{ t('sidebar.contextEditDesc') }}
      </button>
      <div class="h-[1px] bg-slate-200 dark:bg-slate-800 my-1"></div>
      <button @click="triggerDeleteFromContext" class="w-full text-left px-3 py-1.5 hover:bg-rose-50 dark:hover:bg-rose-950/40 text-rose-600 flex items-center gap-2 cursor-pointer">
        <Trash2 class="w-3.5 h-3.5" /> {{ t('sidebar.contextDelete') }}
      </button>
    </div>

    <!-- 基于 BaseModal 封装的二次确认弹窗 -->
    <ConfirmModal 
      v-model="deleteConfirmOpen" 
      :title="t('sidebar.deleteConfirmTitle')" 
      :message="t('sidebar.deleteConfirmMessage')" 
      @confirm="executeDelete"
    />
  </aside>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue';
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import type { VcuDescription } from '@/types/vcu';
import VcuCardView from './VcuCardView.vue';
import VcuTableView from './VcuTableView.vue';
import ConfirmModal from '@/components/common/ConfirmModal.vue';
import { Plus, Upload, Search, LayoutGrid, List, Edit3, Trash2 } from '@lucide/vue';

const emit = defineEmits<{
  (e: 'request-create'): void;
  (e: 'request-import'): void;
  (e: 'request-edit', vcu: VcuDescription): void;
}>();

const { t } = useI18n();
const store = useVcuStore();

const isResizing = ref(false);
const contextMenu = reactive({ visible: false, x: 0, y: 0, targetVcu: null as VcuDescription | null });
const deleteConfirmOpen = ref(false);
const pendingDeleteVcuId = ref<string | null>(null);

function startResize(e: MouseEvent) {
  isResizing.value = true;
  const startX = e.clientX;
  const startWidth = store.sidebarWidth;

  function onMouseMove(moveEvent: MouseEvent) {
    if (!isResizing.value) return;
    const delta = moveEvent.clientX - startX;
    const newWidth = Math.max(220, Math.min(480, startWidth + delta));
    store.sidebarWidth = newWidth;
  }

  function onMouseUp() {
    isResizing.value = false;
    window.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('mouseup', onMouseUp);
  }

  window.addEventListener('mousemove', onMouseMove);
  window.addEventListener('mouseup', onMouseUp);
}

function handleContextMenu(event: MouseEvent, vcu: VcuDescription) {
  contextMenu.visible = true;
  contextMenu.x = event.clientX;
  contextMenu.y = event.clientY;
  contextMenu.targetVcu = vcu;
}

function openEditModal(vcu: VcuDescription) {
  emit('request-edit', vcu);
}

function confirmDeleteVcu(vcu: VcuDescription) {
  pendingDeleteVcuId.value = vcu.id;
  deleteConfirmOpen.value = true;
}

function triggerEditFromContext() {
  contextMenu.visible = false;
  if (contextMenu.targetVcu) openEditModal(contextMenu.targetVcu);
}

function triggerDeleteFromContext() {
  contextMenu.visible = false;
  if (contextMenu.targetVcu) confirmDeleteVcu(contextMenu.targetVcu);
}

function executeDelete() {
  if (pendingDeleteVcuId.value) {
    store.deleteVcu(pendingDeleteVcuId.value);
    pendingDeleteVcuId.value = null;
  }
}
</script>