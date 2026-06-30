<template>
  <div
    class="relative z-50 bg-background rounded-xl shadow-2xl border border-border w-full mx-4 p-6 overflow-y-auto"
    :class="[maxWidthClass, maxHeightClass]"
    @click.stop
  >
    <slot />
    <button
      class="absolute top-4 right-4 w-7 h-7 rounded-md flex items-center justify-center text-muted-foreground hover:bg-muted hover:text-foreground transition-colors"
      @click="closeDialog"
    >
      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
      </svg>
    </button>
  </div>
</template>

<script setup lang="ts">
import { computed, inject } from "vue";

const props = withDefaults(defineProps<{
  maxWidth?: string;
  maxHeight?: string;
}>(), {
  maxWidth: "max-w-4xl",
  maxHeight: "max-h-[90vh]",
});

const closeDialog = inject<() => void>("dialogClose", () => {});

const maxWidthClass = computed(() => props.maxWidth);
const maxHeightClass = computed(() => props.maxHeight);
</script>
