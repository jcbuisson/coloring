<!--
Example usage

<SVGSelect src="/france.svg"
   multiple
   v-model="selectionDict"
   :selectStyle="{ fill: '#fc8e97', stroke: '#FFFFFF' }"
/>

src: url of the SVG document. All paths and objects of the document are selectable, as long as they have an id
model: a dictionnary pathId -> style object
selectStyle: a style to apply to the selected path
-->

<template>
   <div :style="cssVars">
      <svg :viewBox="svg && svg.properties.viewBox" :width="svg && svg.properties.width" :height="svg && svg.properties.height" xmlns="http://www.w3.org/2000/svg">
         <path v-for="(path, index) in pathList" :key="index"
            :d="path.d"
            :style="path.style"
            :transform="path.transform"
            
            @click="selectArea(path)"
            :class="[{'area': isArea(path) && !isSelected(path) && !readonly}]"
         />
      </svg>
   </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { parse } from 'svg-parser'

import toPath from 'element-to-path'


const props = defineProps({
   src: {
      type: String,
      required: true
   },
   value: {
      type: Object,
      required: true
   },
   selectStyle: {
      type: Object,
      required: true
   },
   multiple: {
      type: Boolean,
      default: true
   },
   readonly: {
      type: Boolean,
      default: false
   },
})

const emit = defineEmits(['input'])

onMounted(async() => {
   await buildSVGDocumentStyleDict(props.src)
   updateDOMFromValue()
})

const svg = ref(undefined)
const pathList = ref([])
const pathStyleDict = ref({})

const cssVars = computed(() => ({
   '--hoverColor': props.selectStyle.hoverColor
}))

function selectArea(path) {
   if (path.id && !props.readonly) {
      const isPathSelected = !!props.value[path.id]

      // update model (which in turn changes the DOM)
      if (isPathSelected) {
         // remove element from selection
         delete props.value[path.id]
      } else {
         if (!props.multiple) {
            const [lastSelectedPathId] = Object.keys(props.value)
            if (lastSelectedPathId) {
               // remove previous selection
               delete props.value[lastSelectedPathId]
            }
         }
         // add element to selection
         const selectStyleCopy = Object.assign({}, props.selectStyle)
         props.value[path.id] = selectStyleCopy
      }
      emit('input', props.value)
   }

   updateDOMFromValue()
}

function isSelected(path) {
   return props.value[path.id]
}

function isArea(path) {
   return path.id
}

async function buildSVGDocumentStyleDict(src) {
   pathStyleDict.value = undefined
   let response = await fetch(src)
   let svgText = await response.text()
   // console.log('svgText', svgText)
   let parsedSvg = parse(svgText)
   svg.value = parsedSvg.children[0]

   // build a list of all <path> elements. They must be at the first level and not inside <g> elements
   pathList.value = svg.value.children.map(svgElement => {
      const path = {
         d: toPath({
            type: svgElement.type,
            name: svgElement.tagName,
            attributes: svgElement.properties
         }),
         transform: svgElement.properties.transform,
         id: svgElement.properties.id
      }
      if (svgElement.properties.style) {
         const styleDict = {}
         for (const stylePart of svgElement.properties.style.split(';')) {
            const [key, value] = stylePart.split(':')
            if (key && value) {
               styleDict[key.trim()] = value.trim()
            }
         }
         path.style = styleDict
      } else {
         path.style = Object.assign({}, svgElement.properties)
      }
      return path
   })

   // build a dictionnary of the initial style for each path
   const pathStyleDict_ = {}
   for (const path of pathList.value) {
      // make a copy
      pathStyleDict_[path.id] = { ...path.style }
   }
   pathStyleDict.value = pathStyleDict_
}

function updateDOMFromValue() {
   if (pathStyleDict.value) {
      // svg must have been fetched and processed
      pathList.value.forEach(path => {
         const selectedStyle = props.value[path.id]
         if (selectedStyle) {
            // assign selectedStyle to path
            Object.assign(path.style, selectedStyle)
         } else {
            // set back original style to path
            Object.assign(path.style, pathStyleDict.value[path.id])
         }
      })
   }
}
</script>

<style scoped>
path {
  pointer-events: all;
}

.area:hover {
  /* stroke: rgba(0, 152, 167, 0.3) !important; */
  /* fill: rgba(0,152,167, 0.3) !important; */
  stroke-width: 2;
  stroke: var(--hoverColor);
  fill: var(--hoverColor) !important;
}
</style>