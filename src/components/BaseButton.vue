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
    'rounded bg-[--primary-button-background-color] px-3 py-2 text-xs font-semibold text-white shadow-sm hover:bg-[--primary-button-background-hover-color] focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[--primary-button-background-color]',
  secondary:
    'rounded bg-white px-3 py-2 text-xs font-semibold text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 hover:bg-gray-50'
}

const classes = computed(() => {
  return stylesMap[props.look]
})
</script>
