<script setup lang="ts">

const props = defineProps<{
  title: string;
  value: number;
  loading: boolean;
}>();

</script>

<template>
  <div
    :class="[
      'border border-gray-200 rounded-lg p-4 flex flex-col justify-between gap-2 bg-white shadow',
      loading ? 'animate-pulse' : '',
    ]"
  >
    <slot name="icon"></slot>
    <h3 class="text-lg font-semibold text-gray-900">{{ title }}</h3>
    <p v-if="!loading" class="text-gray-500">${{ value?.toLocaleString() }}</p>
  </div>
</template>
