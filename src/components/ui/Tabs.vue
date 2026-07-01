<script setup lang="ts">
import { TabsRoot } from 'radix-vue'

const props = defineProps<{
  modelValue?: string
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()
</script>

<template>
  <TabsRoot
    :model-value="props.modelValue"
    @update:model-value="(v) => emit('update:modelValue', v)"
    class="flex flex-col"
  >
    <slot />
  </TabsRoot>
</template>
