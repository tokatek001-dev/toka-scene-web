<template>
  <textarea
    v-bind="$attrs"
    :value="modelValue"
    @input="(e) => emit('update:modelValue', (e.target as HTMLTextAreaElement).value)"
    class="flex min-h-[80px] w-full rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50 resize-none"
  />
</template>

<script setup lang="ts">
defineOptions({ inheritAttrs: false });
defineProps<{ modelValue?: string }>();
const emit = defineEmits<{ (e: "update:modelValue", value: string): void }>();
</script>
