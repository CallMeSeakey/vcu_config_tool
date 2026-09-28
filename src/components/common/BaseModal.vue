<template>
  <Teleport to="body">
    <Transition name="modal-fade">
      <div 
        v-if="modelValue" 
        class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-xs select-none"
        @click.self="handleBackdropClick"
        @keydown.esc="handleClose"
        tabindex="-1"
      >
        <div 
          :class="[
            'bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden transition-all',
            sizeClass
          ]"
        >
          <!-- 统一头部 -->
          <div class="h-12 px-4 border-b border-slate-200 dark:border-slate-800 flex items-center justify-between shrink-0 bg-slate-50/50 dark:bg-slate-900/40">
            <slot name="header">
              <h3 class="text-sm font-semibold text-slate-800 dark:text-slate-200">{{ title }}</h3>
            </slot>
            <button 
              v-if="showClose" 
              @click="handleClose" 
              class="p-1.5 hover:bg-slate-200 dark:hover:bg-slate-800 rounded-md text-slate-400 hover:text-slate-600 dark:hover:text-slate-200 transition cursor-pointer"
            >
              <X class="w-4 h-4" />
            </button>
          </div>

          <!-- 内容主体插槽 -->
          <div class="p-4 overflow-y-auto flex-1 text-xs">
            <slot></slot>
          </div>

          <!-- 统一底部插槽 -->
          <div v-if="$slots.footer" class="h-12 px-4 border-t border-slate-200 dark:border-slate-800 flex items-center justify-end gap-2 shrink-0 bg-slate-50/50 dark:bg-slate-900/30">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { X } from 'lucide-vue-next';

const props = withDefaults(defineProps<{
  modelValue: boolean;
  title?: string;
  size?: 'sm' | 'md' | 'lg' | 'xl';
  showClose?: boolean;
  closeOnBackdrop?: boolean;
}>(), {
  modelValue: false,
  title: '',
  size: 'md',
  showClose: true,
  closeOnBackdrop: true
});

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void;
  (e: 'close'): void;
}>();

const sizeClass = computed(() => {
  switch (props.size) {
    case 'sm': return 'w-84';
    case 'lg': return 'w-[680px]';
    case 'xl': return 'w-[840px]';
    default: return 'w-[480px]';
  }
});

function handleClose() {
  emit('update:modelValue', false);
  emit('close');
}

function handleBackdropClick() {
  if (props.closeOnBackdrop) {
    handleClose();
  }
}
</script>

<style scoped>
.modal-fade-enter-active, .modal-fade-leave-active { transition: opacity 0.2s ease, transform 0.2s ease; }
.modal-fade-enter-from, .modal-fade-leave-to { opacity: 0; transform: scale(0.97); }
</style>