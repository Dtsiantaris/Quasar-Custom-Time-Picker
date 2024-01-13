<template>
  <q-btn
    color="secondary"
    unelevated
    flat
    padding="0"
    :icon="type === 'up' ? 'arrow_drop_up' : 'arrow_drop_down'"
    dense
    class="w-[2.8rem] "
  />
</template>

<script setup lang="ts">
import { PropType } from 'vue'

defineProps({
  type: {
    type: String as PropType<'up' | 'down'>,
    required: true
  }
})
</script>

<style scoped>

</style>
