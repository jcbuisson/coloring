<template>
   <div>
      <div class="title">Coloriage</div>
      <SVGSelect class="map"
         multiple
         src="/france.svg"
         :selectStyle="selectStyle"
         :value="selected"
      />
      <ColorPicker class="picker"
         src="/color-picker.svg"
         :value="currentColor"
         @input="onColorChange"
      />
   </div>
</template>

<script setup>
import { ref } from 'vue'

import SVGSelect from '/src/components/SVGSelect.vue'
import ColorPicker from '/src/components/ColorPicker.vue'

const selected = ref({})
const selectStyle = ref({
   fill: '#fc8e97',
   stroke: '#FFFFFF',
   hoverColor: '#00000011',
})
const currentColor = ref("#fc8e97")

function onColorChange(color) {
   const colorCopy = (' ' + color).slice(1) // force deep copy
   selectStyle.value.fill = colorCopy
}
</script>

<style>
#app {
   font-family:"Helvetica Neue", Helvetica, Arial, sans-serif;
}

.title {
   font-size: 64px;
   font-weight: 100;
   color: #000000de;
   text-align: center;
   padding: 30px;
}
@media (max-width: 576px) {
   .title {
      font-size: 32px;
   }
}

.map {
   position: absolute;
   left: 10%;
   right: 10%;
}

.picker {
   position: fixed;
   left: 80%;
   right: 5%;
   top: 40%;
}
</style>
