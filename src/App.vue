<template>
  <div class="h-screen w-screen flex flex-col bg-slate-100 dark:bg-slate-950 text-slate-900 dark:text-slate-100 select-none overflow-hidden font-sans">
    <!-- 现代化标题栏 -->
    <TitleBar 
      @open-settings="isSettingsOpen = true" 
      @quick-open="handleQuickOpen"
      @quick-create="handleCreateVcu"
      @quick-import="handleImportVcu"
      @save-failed="handleSaveOrApplyFailed"
    />

    <!-- 工作区主体 -->
    <div class="flex-1 flex overflow-hidden">
      <VcuSidebar 
        @request-create="handleCreateVcu" 
        @request-import="handleImportVcu" 
        @request-edit="handleEditVcu"
      />

      <PinConfigView 
        ref="pinConfigRef" 
        @validation-failed="handleSaveOrApplyFailed"
        @apply-success="handleApplySuccess"
      />
    </div>

    <!-- 底部状态栏 -->
    <StatusBar @open-problems="isProblemsOpen = true" />

    <!-- 全局弹窗群 (统一基于 BaseModal 风格) -->
    <SettingsModal v-model="isSettingsOpen" />
    <ProblemsModal 
      v-model="isProblemsOpen" 
      @jump-to-pin="handleJumpToPin" 
    />
    <EditVcuModal 
      v-model="isEditVcuOpen" 
      :vcu="editingVcu" 
    />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useI18n } from 'vue-i18n';
import { useVcuStore } from '@/stores/vcuStore';
import type { VcuDescription } from '@/types/vcu';
import TitleBar from '@/components/layout/TitleBar.vue';
import StatusBar from '@/components/layout/StatusBar.vue';
import VcuSidebar from '@/views/vcu-manage/VcuSidebar.vue';
import PinConfigView from '@/views/pin-config/PinConfigView.vue';
import SettingsModal from '@/views/modals/SettingsModal.vue';
import ProblemsModal from '@/views/modals/ProblemsModal.vue';
import EditVcuModal from '@/views/modals/EditVcuModal.vue';

const { t } = useI18n();
const store = useVcuStore();

const isSettingsOpen = ref(false);
const isProblemsOpen = ref(false);
const isEditVcuOpen = ref(false);
const editingVcu = ref<VcuDescription | null>(null);
const pinConfigRef = ref<InstanceType<typeof PinConfigView> | null>(null);

onMounted(() => {
  store.fetchVcus();
});

function handleQuickOpen() {
  alert(t('notice.openSuccess'));
}

function handleCreateVcu() {
  alert(t('notice.createTip'));
}

function handleImportVcu() {
  alert(t('notice.importTip'));
}

function handleEditVcu(vcu: VcuDescription) {
  editingVcu.value = vcu;
  isEditVcuOpen.value = true;
}

function handleSaveOrApplyFailed(_errors: string[]) {
  isProblemsOpen.value = true;
}

function handleApplySuccess() {
  alert(t('vcu.applySuccess'));
}

function handleJumpToPin(pinLabel: string) {
  pinConfigRef.value?.locatePin(pinLabel);
}
</script>