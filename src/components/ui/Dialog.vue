<template>
  <teleport to="body">
    <transition name="dialog-fade">
      <div
        v-if="open"
        class="fixed inset-0 z-50 flex items-center justify-center"
        @click.self="$emit('update:open', false)"
      >
        <!-- Overlay -->
        <div class="absolute inset-0 bg-black/50 backdrop-blur-sm" />
        <!-- Content: inject close handler via provide -->
        <slot />
      </div>
    </transition>
  </teleport>
</template>

<script setup lang="ts">
import { provide } from "vue";

const props = defineProps<{ open?: boolean }>();
const emit = defineEmits<{ (e: "update:open", val: boolean): void }>();

provide("dialogClose", () => emit("update:open", false));
</script>

<style scoped>
.dialog-fade-enter-active,
.dialog-fade-leave-active {
  transition: opacity 0.18s ease;
}
.dialog-fade-enter-from,
.dialog-fade-leave-to {
  opacity: 0;
}
</style>
