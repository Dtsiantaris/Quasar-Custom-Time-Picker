<template>
  <div class="flex flex-col items-center gap-1">
    <CustomStepperButton
      type="up"
      @click="incrementValue"
    />
    <q-input
      ref="inputRef"
      :model-value="valueRef"
      :mask="mask"
      :label="!valueRef ? (props.type==='hours' ? $t('hh') :$t('mm')) :undefined"
      filled
      class="border border-primary rounded w-[4rem]"
      input-class="text-center font-semibold text-2xl"
      @keydown="handleInput"
    />

    <CustomStepperButton
      type="down"
      @click="decrementValue"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, watch, PropType } from 'vue'
import CustomStepperButton from './CustomStepperButton.vue'
import { QInput } from 'quasar'
import _ from 'lodash'

const props = defineProps({
  value: {
    type: String,
    required: true
  },
  type: {
    type: String as PropType<'hours' | 'minutes'>,
    required: true
  }
})

const emits = defineEmits(['update:value', 'change-focus'])

const inputRef = ref<QInput>()
const valueRef = ref(props.value.toString())
const mask = '##'

const focusInput = () => {
  inputRef.value?.focus()
}

defineExpose({ focusInput })

const handleInput = (event: KeyboardEvent) => {
  event.preventDefault()
  const key = event.key
  // Check if input is allowed digit
  if (!isValidKey(key)) {
    return
  }
  // Define if part of input is selected
  const selectionStart = (event.target as HTMLInputElement)?.selectionStart || 0
  const selectionEnd = (event.target as HTMLInputElement)?.selectionEnd || 0
  const selectionDiff = selectionEnd - selectionStart

  // Handle backspace
  if (key === 'Backspace') {
    valueRef.value = valueRef.value.slice(0, (event.ctrlKey || selectionDiff === 2) ? -2 : -1)
    if (props.type === 'minutes' && valueRef.value.length === 0) emits('change-focus')
    return
  }
  // Handle tab
  if (key === 'Tab') {
    emits('change-focus')
    return
  }
  let currentValue = _.cloneDeep(valueRef.value)
  // check if value is 2 digit. Delete value and apply input if true
  if (currentValue.length === 2 && selectionDiff === 0) currentValue = ''
  let potentialValue = currentValue
  if (selectionDiff === 2) potentialValue = key
  else if (selectionDiff === 1) {
  // One digit is selected, replace the specific character
    const chars = currentValue.split('')
    chars[selectionStart] = key
    potentialValue = chars.join('')
  } else potentialValue = currentValue + key

  // If its the second digit and it is not an arrow key
  // Arrow key check prevents change of focus when not wanted
  if (currentValue.length === 2 && key !== 'ArrowLeft' && key !== 'ArrowRight') {
    // If nothing is selected allow changing of focus
    if (selectionDiff === 0) {
      emits('change-focus')
      return
    }
  } else if (key === 'ArrowLeft' || key === 'ArrowRight') return

  if (!isValidValue(potentialValue)) {
    return
  }
  valueRef.value = potentialValue
  // change focus when model value has reached max length
  if (potentialValue.length === 2) {
    emits('change-focus')
    // prefix with 0 when necessary
  } else if (potentialValue.length === 1 && props.type === 'hours' ? Number(key) > 2 : Number(key) > 5) {
    valueRef.value = valueRef.value.padStart(2, '0')
    emits('change-focus')
  }
}

const isValidKey = (key: string) => {
  const controlKeys = ['Backspace', 'ArrowLeft', 'ArrowRight', 'Delete', 'Tab']
  return !isNaN(Number(key)) || controlKeys.includes(key)
}

const isValidValue = (value: string) => {
  const numberValue = parseInt(value, 10)
  if (props.type === 'hours') {
    return numberValue >= 0 && numberValue <= 23
  } else if (props.type === 'minutes') {
    return numberValue >= 0 && numberValue <= 59
  }
  return false
}

const incrementValue = () => {
  let currentValue = parseInt(valueRef.value)
  if (isNaN(currentValue)) currentValue = 0

  if (props.type === 'hours') {
    currentValue = (currentValue + 1) % 24
  } else if (props.type === 'minutes') {
    currentValue = (currentValue + 1) % 60
  }

  valueRef.value = currentValue.toString().padStart(2, '0')
}

const decrementValue = () => {
  let currentValue = parseInt(valueRef.value)
  if (isNaN(currentValue)) currentValue = 0

  if (props.type === 'hours') {
    currentValue = (currentValue + 23) % 24
  } else if (props.type === 'minutes') {
    currentValue = (currentValue + 59) % 60
  }

  valueRef.value = currentValue.toString().padStart(2, '0')
}

watch(() => props.value, (newValue) => {
  valueRef.value = newValue.toString()
}, { deep: true })

watch(() => valueRef.value, (newValue) => {
  emits('update:value', newValue)
})
</script>
