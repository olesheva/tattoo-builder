<template>
  <div class="relative flex items-start">
    <div class="flex h-6 items-center">
      <input
        :name="name"
        :value="value"
        :checked="isChecked"
        @change="onInput"
        :id="name"
        type="checkbox"
        class="h-4 w-4 rounded border-gray-300 text-[--checkbox-background-color] focus:ring-[--checkbox-background-color] cursor-pointer"
      />
    </div>
    <div class="ml-3 text-sm leading-6">
      <label :for="name" class="font-medium text-gray-900 cursor-pointer">{{ label }}</label>
    </div>
  </div>
</template>

<script setup lang="ts">

import { computed } from 'vue'
const props = defineProps({
    label: {
        type: String,
        default: ''
    },
    name: {
        type: String,
        default: '',
    },
    value: {
        type: String,
        required: true
    },
    modelValue: {
        type: [String, Number, Array],
        required: true,
    },
})

const emits = defineEmits(['update:modelValue'])
const isChecked = computed(() => {
    if (Array.isArray(props.modelValue)) {
        return props.modelValue.includes(props.value)
      }
      return !!props.modelValue
})

const onInput = (payload: Event) => {
    // console.log('emits', (payload.target as HTMLInputElement))
    // emits('update:modelValue', (payload.target as HTMLInputElement).checked)
    const isChecked = (payload.target as HTMLInputElement).checked
      if (Array.isArray(props.modelValue)) {
        let newValue = [...props.modelValue]
        if (isChecked) {
          newValue.push(props.value)
        } else {
          newValue.splice(newValue.indexOf(props.value), 1)
        }
        emits('update:modelValue', newValue)
      } else {
        emits('update:modelValue', !!isChecked)
      }
}

</script>
