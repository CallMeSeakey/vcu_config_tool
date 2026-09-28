<template>
  <header 
    data-tauri-drag-region
    @mousedown="handleStartDragging"
    class="h-10 bg-white/90 dark:bg-slate-900/90 border-b border-slate-200 dark:border-slate-800 flex items-center justify-between px-3 z-50 backdrop-blur select-none cursor-default"
  >
    <!-- 左侧：汉堡菜单 + 快捷动作区 (打开/保存/新建/导入) -->
    <div class="flex items-center gap-2" @mousedown.stop>
      <button class="p-1 hover:bg-slate-100 dark:hover:bg-slate-800 rounded transition text-slate-600 dark:text-slate-400 cursor-pointer">
        <Menu class="w-4 h-4" />
      </button>

      <div class="h-4 w-[1px] bg-slate-300 dark:bg-slate-800 mx-0.5"></div>

      <!-- 快捷工具按钮组 -->
      <div class="flex items-center gap-1">
        <button 
          @click="$emit('quick-open')" 
          class="p-1.5 hover:bg-slate-100 dark:hover:bg-slate-800 rounded text-slate-600 dark:text-slate-400 hover:text-cyan-600 transition cursor-pointer"
          :title="t('nav.quickOpen')"
        >
          <FolderOpen class="w-3.5 h-3.5" />
        </button>

        <!-- 保存按钮：依据 isDirty 动态激活/禁用 -->
        <button 
          @click="handleSave"
          :disabled="!store.isDirty"
          :class="[
            'p-1.5 rounded transition flex items-center gap-1 text-xs font-medium cursor-pointer',
            store.isDirty 
              ? 'bg-cyan-600 hover:bg-cyan-500 text-white shadow-xs animate-pulse' 
              : 'text-slate-400 dark:text-slate-600 opacity-50 cursor-not-allowed'
          ]"
          :title="store.isDirty ? t('nav.quickSaveDirtyTip') : t('nav.quickSaveCleanTip')"
        >
          <Save class="w-3.5 h-3.5" />
          <span v-if="store.isDirty" class="text-[10px]">{{ t('common.action.saveDirty') }}</span>
        </button>

        <button 
          @click="$emit('quick-create')" 
          class="p-1.5 hover:bg-slate-100 dark:hover:bg-slate-800 rounded text-slate-600 dark:text-slate-400 hover:text-cyan-600 transition cursor-pointer"
          :title="t('nav.quickCreate')"
        >
          <FilePlus class="w-3.5 h-3.5" />
        </button>

        <button 
          @click="$emit('quick-import')" 
          class="p-1.5 hover:bg-slate-100 dark:hover:bg-slate-800 rounded text-slate-600 dark:text-slate-400 hover:text-cyan-600 transition cursor-pointer"
          :title="t('nav.quickImport')"
        >
          <UploadCloud class="w-3.5 h-3.5" />
        </button>
      </div>
    </div>

    <!-- 中间：软件名 | 当前活动VCU (居中展示，允许拖拽窗口) -->
    <div class="flex-1 flex items-center justify-center text-xs font-medium h-full pointer-events-none px-4">
      <div class="flex items-center gap-2 text-slate-700 dark:text-slate-300">
        <span class="tracking-wider uppercase font-bold text-cyan-600 dark:text-cyan-400">
          {{ t('app.title') }}
        </span>
        <span class="text-slate-400">|</span>
        <span v-if="store.currentVcu" class="font-semibold text-slate-900 dark:text-slate-100">
          {{ resolveI18nText(store.currentVcu.name, locale) }}
          <span v-if="store.isDirty" class="text-amber-500 ml-1 font-mono text-[11px]">{{ t('app.modified') }}</span>
        </span>
        <span v-else class="text-slate-400">{{ t('app.noProject') }}</span>
      </div>
    </div>

    <!-- 右侧：功能控制组与系统窗口按键 -->
    <div class="flex items-center gap-2" @mousedown.stop>
      <button 
        @click="toggleLanguage" 
        class="px-2 py-0.5 text-xs font-medium border border-slate-200 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-800 rounded transition flex items-center gap-1 text-slate-700 dark:text-slate-300 cursor-pointer"
        :title="t('common.language.label')"
      >
        <Languages class="w-3.5 h-3.5" />
        <span>{{ locale === 'zh-CN' ? 'CN' : 'EN' }}</span>
      </button>

      <button 
        @click="cycleTheme" 
        class="p-1.5 hover:bg-slate-100 dark:hover:bg-slate-800 rounded text-slate-600 dark:text-slate-400 transition cursor-pointer"
        :title="themeTooltip"
      >
        <Sun v-if="themeMode === 'light'" class="w-4 h-4 text-amber-500" />
        <Moon v-else-if="themeMode === 'dark'" class="w-4 h-4 text-cyan-400" />
        <Laptop v-else class="w-4 h-4 text-slate-400" />
      </button>

      <button 
        @click="$emit('open-settings')" 
        class="p-1.5 hover:bg-slate-100 dark:hover:bg-slate-800 rounded text-slate-600 dark:text-slate-400 transition cursor-pointer"
        :title="t('common.settings')"
      >
        <Settings class="w-4 h-4" />
      </button>

      <div class="h-4 w-[1px] bg-slate-300 dark:bg-slate-800 mx-1"></div>

      <div class="flex items-center gap-0.5">
        <button @click="minimizeWindow" class="w-7 h-7 flex items-center justify-center hover:bg-slate-200 dark:hover:bg-slate-800 rounded text-slate-500 hover:text-slate-900 dark:hover:text-white cursor-pointer">─</button>
        <button @click="toggleMaximize" class="w-7 h-7 flex items-center justify-center hover:bg-slate-200 dark:hover:bg-slate-800 rounded text-slate-500 hover:text-slate-900 dark:hover:text-white cursor-pointer text-sm">□</button>
        <button @click="closeWindow" class="w-7 h-7 flex items-center justify-center hover:bg-rose-600 rounded text-slate-500 hover:text-white cursor-pointer">✕</button>
      </div>
    </div>
  </header>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useI18n } from 'vue-i18n';
import { getCurrentWindow } from '@tauri-apps/api/window';
import { useVcuStore } from '@/stores/vcuStore';
import { resolveI18nText } from '@/types/vcu';
import { 
  Menu, Languages, Sun, Moon, Laptop, Settings, 
  FolderOpen, Save, FilePlus, UploadCloud 
} from 'lucide-vue-next';

const emit = defineEmits<{
  (e: 'open-settings'): void;
  (e: 'quick-open'): void;
  (e: 'quick-create'): void;
  (e: 'quick-import'): void;
  (e: 'save-failed', errors: string[]): void;
}>();

const { t, locale } = useI18n();
const store = useVcuStore();
const appWindow = getCurrentWindow();

async function handleStartDragging(e: MouseEvent) {
  if (e.button === 0) await appWindow.startDragging();
}

async function minimizeWindow() { await appWindow.minimize(); }
async function toggleMaximize() { await appWindow.toggleMaximize(); }
async function closeWindow() { await appWindow.close(); }

async function handleSave() {
  if (!store.isDirty) return;
  const { success, result } = await store.applyConfig();
  if (!success) {
    emit('save-failed', result.errors);
  }
}

function toggleLanguage() {
  locale.value = locale.value === 'zh-CN' ? 'en-US' : 'zh-CN';
  localStorage.setItem('app-lang', locale.value);
}

type ThemeMode = 'light' | 'dark' | 'auto';
const themeMode = ref<ThemeMode>((localStorage.getItem('app-theme') as ThemeMode) || 'auto');

const themeTooltip = computed(() => {
  if (themeMode.value === 'light') return t('common.theme.light');
  if (themeMode.value === 'dark') return t('common.theme.dark');
  return t('common.theme.auto');
});

function applyTheme(mode: ThemeMode) {
  const root = document.documentElement;
  if (mode === 'dark') root.classList.add('dark');
  else if (mode === 'light') root.classList.remove('dark');
  else {
    if (window.matchMedia('(prefers-color-scheme: dark)').matches) root.classList.add('dark');
    else root.classList.remove('dark');
  }
}

function cycleTheme() {
  if (themeMode.value === 'auto') themeMode.value = 'light';
  else if (themeMode.value === 'light') themeMode.value = 'dark';
  else themeMode.value = 'auto';
  localStorage.setItem('app-theme', themeMode.value);
  applyTheme(themeMode.value);
}
</script>