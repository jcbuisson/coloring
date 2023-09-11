<!--
Example usage

<ColorPicker src="https://static.jcbuisson.dev/svg/color-picker.svg" v-model="currentColor" />

model: '' when not color is selected, '#rrggbb' otherwise
svg: each selectable object must have an id 'c_rrggbb', for example 'c_87ac9f'
-->

<template>
   <SVGSelect
      :src="src"
      :multiple="false"
      :selectStyle="selectStyle"
      :value="selectionObject"
      @input="onSelectionChange"
   />
</template>

<script setup>
import { ref } from 'vue'

import SVGSelect from '/src/components/SVGSelect.vue'

   
const props = defineProps({
   value: {
      type: String,
      required: true
   },
   src: {
      type: String,
      required: true
   }
})

const emit = defineEmits(['input'])

const selectStyle = ref({
   stroke: '#0000FF',
   hoverColor: '#FFFFFF',
})
const selectionObject = ref({})

function onSelectionChange(selectionObject) {
   const [id] = Object.keys(selectionObject)
   if (id) {
      const color = '#' + id.substring(2)
      emit('input', color)
   } else {
      emit('input', '')
   }
}
</script>
