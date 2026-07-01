<template>
  <input
    type="checkbox"
    v-bind="$attrs"
    :checked="checked"
    @change="(e) => emit('update:checked', (e.target as HTMLInputElement).checked)"
    class="h-4 w-4 rounded border border-input bg-background text-primary focus:ring-2 focus:ring-ring focus:ring-offset-2 cursor-pointer disabled:cursor-not-allowed disabled:opacity-50"
  />
</template>

<script setup lang="ts">
defineOptions({ inheritAttrs: false });
const props = defineProps<{ checked?: boolean }>();
const emit = defineEmits<{ (e: "update:checked", value: boolean): void }>();
</script>
