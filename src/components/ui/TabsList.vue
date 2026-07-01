<script setup lang="ts">
import { TabsList } from 'radix-vue'
import { cn } from '@/lib/utils'

const props = defineProps<{
  class?: string
}>()
</script>

<template>
  <TabsList
    :class="cn(
      'inline-flex h-9 items-center justify-start rounded-lg bg-muted p-1 text-muted-foreground w-full',
      props.class
    )"
  >
    <slot />
  </TabsList>
</template>
