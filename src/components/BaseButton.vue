<template>
  <button :class="classes" v-if="!href">
    <slot />
  </button>
  <a v-else :href="href" target="_blank" :class="classes"> <slot /></a>
</template>

<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps({
  look: {
    type: String,
    default: 'primary'
  },
  href: {
    type: String,
    default: undefined
  }
})

const stylesMap: { [key: string]: string } = {
  primary:
    'rounded bg-indigo-500 px-2 py-1 text-xs font-semibold text-white shadow-sm hover:bg-indigo-400 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-500',
  secondary:
    'rounded bg-white px-2 py-1 text-xs font-semibold text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 hover:bg-gray-50'
}

const classes = computed(() => {
  return stylesMap[props.look]
})
</script>
