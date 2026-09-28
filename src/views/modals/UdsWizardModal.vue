<template>
  <BaseModal :model-value="modelValue" :title="t('uds.wizardTitle')" size="lg" @close="handleClose">
    <!-- 步骤进度指示器 -->
    <div class="flex items-center justify-between mb-6 px-4">
      <div v-for="i in 4" :key="i" class="flex items-center gap-2">
        <span :class="['w-6 h-6 rounded-full flex items-center justify-center font-bold text-xs', step === i ? 'bg-cyan-600 text-white' : step > i ? 'bg-emerald-500 text-white' : 'bg-slate-200 dark:bg-slate-800 text-slate-400']">
          {{ i }}
        </span>
        <span class="text-slate-600 dark:text-slate-400 font-medium">{{ t(`uds.step${i}`) }}</span>
        <span v-if="i < 4" class="h-[1px] w-8 bg-slate-300 dark:bg-slate-800 mx-2"></span>
      </div>
    </div>

    <!-- 步骤内容区 -->
    <div class="py-4 px-2 space-y-4">
      <div v-if="step === 1" class="space-y-3">
        <label class="block font-medium text-slate-700 dark:text-slate-300">{{ t('uds.channel') }}</label>
        <select class="w-full bg-slate-50 dark:bg-slate-950 border border-slate-200 dark:border-slate-800 rounded-lg p-2 text-xs text-slate-800 dark:text-slate-200 cursor-pointer">
          <option>CAN0 (Peak-System PCAN-USB Pro)</option>
          <option>CAN1 (ZLG USBCAN-II+)</option>
          <option>vcan0 (Virtual SocketCAN Device)</option>
        </select>
      </div>

      <div v-else-if="step === 2" class="space-y-3">
        <label class="block font-medium text-slate-700 dark:text-slate-300">{{ t('uds.baudRate') }}</label>
        <select class="w-full bg-slate-50 dark:bg-slate-950 border border-slate-200 dark:border-slate-800 rounded-lg p-2 text-xs text-slate-800 dark:text-slate-200 cursor-pointer">
          <option>500 Kbps (ISO 11898-2)</option>
          <option>250 Kbps</option>
          <option>1 Mbps (CAN-FD)</option>
        </select>
      </div>

      <div v-else-if="step === 3" class="space-y-2">
        <div class="p-3 bg-emerald-50 dark:bg-emerald-950/30 border border-emerald-300 dark:border-emerald-800 rounded-lg text-emerald-600 text-xs">
          √ 诊断 Session 控制 (0x10) 握手就绪<br/>
          √ 安全访问算法密钥种子校验通过 (0x27)
        </div>
      </div>

      <div v-else-if="step === 4" class="space-y-4">
        <div class="space-y-1">
          <div class="flex justify-between text-xs text-slate-500">
            <span>{{ isFlashing ? t('uds.statusFlashing') : t('uds.statusCompleted') }}</span>
            <span>{{ progress }}%</span>
          </div>
          <div class="w-full h-2 bg-slate-200 dark:bg-slate-800 rounded-full overflow-hidden">
            <div class="h-full bg-cyan-600 transition-all duration-300" :style="{ width: `${progress}%` }"></div>
          </div>
        </div>
      </div>
    </div>

    <template #footer>
      <button 
        v-if="step > 1 && !isFlashing" 
        @click="step--" 
        class="px-3 py-1.5 rounded border border-slate-200 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-800 text-slate-700 dark:text-slate-300 transition cursor-pointer"
      >
        {{ t('common.action.prev') }}
      </button>
      <button 
        v-if="step < 4" 
        @click="step++" 
        class="px-3 py-1.5 rounded bg-cyan-600 hover:bg-cyan-500 text-white font-medium transition cursor-pointer"
      >
        {{ t('common.action.next') }}
      </button>
      <button 
        v-else-if="step === 4 && !isFlashing && progress === 0" 
        @click="startFlash" 
        class="px-3 py-1.5 rounded bg-indigo-600 hover:bg-indigo-500 text-white font-medium transition cursor-pointer"
      >
        {{ t('uds.startFlash') }}
      </button>
      <button 
        v-else 
        @click="handleClose" 
        class="px-3 py-1.5 rounded bg-slate-800 hover:bg-slate-700 text-white transition cursor-pointer"
      >
        {{ t('common.action.finish') }}
      </button>
    </template>
  </BaseModal>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useI18n } from 'vue-i18n';
import BaseModal from '@/components/common/BaseModal.vue';

defineProps<{ modelValue: boolean }>();
const emit = defineEmits<{(e: 'update:modelValue', val: boolean): void}>();

const { t } = useI18n();
const step = ref(1);
const progress = ref(0);
const isFlashing = ref(false);

function handleClose() {
  step.value = 1;
  progress.value = 0;
  isFlashing.value = false;
  emit('update:modelValue', false);
}

function startFlash() {
  isFlashing.value = true;
  const interval = setInterval(() => {
    progress.value += 20;
    if (progress.value >= 100) {
      clearInterval(interval);
      isFlashing.value = false;
    }
  }, 400);
}
</script>