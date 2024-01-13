<template>
  <div class="flex flex-col w-full h-full items-center ">
    <div class="flex flex-col no-wrap rounded gap-1 w-full ">
      <div
        v-if="!noHeader"
        class="text-white pl-3.5 py-4 text-[24px] leading-7  bg-primary rounded rounded-b-none"
      >
        {{ $t('Enter Time') }}
      </div>
      <div class="flex flex-row no-wrap justify-center items-center gap-0.5 w-fit self-center">
        <div class="flex no-wrap gap-1.5 items-center justify-center p-2">
          <CustomTimeInput
            ref="hoursInputRef"
            v-model:value="hours"
            type="hours"
            @change-focus="focusNext"
          />
          <span class="font-bold text-2xl">:</span>
          <CustomTimeInput
            ref="minutesInputRef"
            v-model:value="minutes"
            type="minutes"
            @change-focus="focusPrevious"
          />
        </div>
        <!-- Q-time btn -->
        <div class="w-full flex flex-col no-wrap items-center justify-center">
          <CustomButton
            padding="0"
            class="!min-h-0 w-[2.1rem] !h-[1.3rem] rounded-md"
            :label="(Number(hours) >= 0 && Number(hours) <12) ? 'AM' : 'PM'"
            @click="hours = (Number(hours) + ((Number(hours) >= 12) ? -12 : 12)).toString().padStart(2, '0')"
          />
          <CustomButton
            flat
            round
            icon="schedule"
            :tooltip-title="$t('Open clock')"
          >
            <q-menu
              style="z-index: 9999;"
              anchor="top start"
              self="center right"
              :offset="[-90,0]"
              transition-show="scale"
              transition-hide="scale"
            >
              <q-time
                v-model="valueRef"
                class="h-[26rem] w-[20rem]"
                mask="YYYY/MM/DD HH:mm"
                format24h
              />
            </q-menu>
          </CustomButton>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import moment from 'moment-timezone'
import CustomTimeInput from './CustomTimeInput.vue'
import CustomButton from './CustomButton.vue'

/**
 * Custom input by text or by incrementing/decrementing hours and minutes.
 * Designed to work along with q-date and q-time.
 * @value Tested only for Date formatted as strings. Should work with any
 * Date format that has Date and time separated by a space.
 * Default format is `YYYY/MM/DD HH:mm`. This should be the format of `value` and
 * it will also be the format of the emitted value.
 */
const props = defineProps({
  value: {
    type: String,
    required: true
  },
  noHeader: {
    type: Boolean,
    default: undefined
  }
})

const emits = defineEmits(['update:value'])

const valueRef = ref(props.value)
const hours = ref('')
const minutes = ref('')
const minutesInputRef = ref<InstanceType<typeof CustomTimeInput['$props']>>()
const hoursInputRef = ref<InstanceType<typeof CustomTimeInput['$props']>>()

// const handleBadgeClick = () => {
//   hours.value = (Number(hours) + ((Number(hours) >= 12) ? -12 : 12)).toString()
//   focusNext()
// }

const mountedFlag = ref(false)
onMounted(() => {
  if (valueRef.value) {
    const splittedValue = valueRef.value.split(' ')[1].split(':')
    hours.value = splittedValue[0]
    minutes.value = splittedValue[1]
  }
  mountedFlag.value = true
})

watch(() => props.value, () => {
  if (!mountedFlag.value) return
  const splittedValue = props.value.split(' ')[1].split(':')
  hours.value = splittedValue[0]
  minutes.value = splittedValue[1]
  // You can emit the updated value here if needed
  console.log('Updated from props')
}, { deep: true })

watch(() => valueRef.value, () => {
  if (!mountedFlag.value) return
  const dateTime = valueRef.value.split(' ')
  const [_hours, _minutes] = dateTime[1].split(':')
  hours.value = _hours
  minutes.value = _minutes
})

watch(() => [hours.value, minutes.value], () => {
  if (!mountedFlag.value) return
  const [datePart] = props.value.split(' ')
  valueRef.value = `${datePart.length ? datePart : moment(new Date()).format('YYYY/MM/DD')} ${hours.value}:${minutes.value}`

  if (moment(valueRef.value).isValid()) {
    emits('update:value', valueRef.value)
  }
}, { deep: true })

const focusNext = () => {
  minutesInputRef.value?.focusInput()
}

const focusPrevious = () => {
  hoursInputRef.value?.focusInput()
}

</script>
