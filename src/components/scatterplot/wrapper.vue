// App.vue - Main Application Component
<template>
  <div class="app-container">
    <WebGLScatterplot
      :data="props.data"
      :metaData="props.metaData"
      :pointCount="pointCount"
      :color-map="colorMap"
      :colorMode="colorMode"
      :startColor="startColor"
      :endColor="endColor"
      :singleColor="singleColor"
      :forceRegenerate="forceRegenerate"
      :key="componentKey"
    />
    <ControlPanel
      v-model:pointCount="pointCount"
      v-model:colorMode="colorMode"
      v-model:startColor="startColor"
      v-model:endColor="endColor"
      v-model:singleColor="singleColor"
      :color-scheme="colorMap"
      :color-map-map="colorMapMap"
      @regenerate="regenerateData"
      @updateColorMap="updateColorMap"
    />
  </div>
</template>

<script setup lang="ts" >
import {ref, defineProps, watch} from 'vue';
import WebGLScatterplot from './fixed-webgl-scatterplot.vue';
import ControlPanel from './control.vue';

const props = defineProps({
  data: {
    type: Array,
    default: []
  },
  metaData: {
    type: Object,
    default: {}
  },

})

// Reactive state
const pointCount = ref(5000);
const colorMode = ref('random');
const startColor = ref('#ff0000');
const endColor = ref('#0000ff');
const singleColor = ref('#4285f4');
const forceRegenerate = ref(false);
const componentKey = ref(0);
const colorMapType = ref("")

const colorMap = ref(new Map());
const colorMapMap = ref(new Map())

watch( () => props.data, () => {
  generateColorMaps();
})


// Method to trigger data regeneration in the scatterplot component
function regenerateData() {
  forceRegenerate.value = !forceRegenerate.value;
  componentKey.value++; // Force component re-creation
}

// Helper function to convert hex to RGB
function hexToRgb(hex) {
  try {
    // Ensure the hex string is properly formatted
    if (!hex || hex.length < 7) {
      console.warn(`Invalid hex color: ${hex}, using fallback`);
      return [1.0, 0.0, 0.0]; // Default to red
    }

    // Extract RGB components and normalize to 0-1 range for WebGL
    const r = parseInt(hex.slice(1, 3), 16) / 255;
    const g = parseInt(hex.slice(3, 5), 16) / 255;
    const b = parseInt(hex.slice(5, 7), 16) / 255;

    // Ensure values are valid numbers
    if (isNaN(r) || isNaN(g) || isNaN(b)) {
      console.warn(`Invalid hex color components in: ${hex}, using fallback`);
      return [1.0, 0.0, 0.0]; // Default to red
    }

    return [r, g, b];
  } catch (err) {
    console.error('Error converting hex to RGB:', err);
    return [1.0, 0.0, 0.0]; // Default to red
  }

}

function generateColorMaps() {

  const hexColors = ["#4269d0","#efb118","#ff725c","#6cc5b0","#3ca951","#ff8ab7","#a463f2","#97bbf5","#9c6b4e","#9498a0"]
  const rgbColors = hexColors.map(color => hexToRgb(color))

  // iterate over Metadata Fields:
  for (let i = 0; i <(props.metaData.schema.length-1); i++) {

    const type = props.metaData.schema[i+1].name
    let varType = colorMapMap.value.get(type)
    if (varType == null) {
      colorMapMap.value.set(type, new Map())
    }
  }

  console.log(colorMapMap)

  for (let i = 0; i < props.data.length; i++) {

    for (let j = 2; j < props.data[i].length; j++) {
      const type = props.metaData.schema[j+1].name
      const value = props.data[i][j]
      let varType = colorMapMap.value.get(type)

      // cellType color
      let varTypeOption = varType.get(value);
      if (varTypeOption == null) {
        varType.set(value, rgbColors[varType.size%rgbColors.length])
        colorMapMap.value.set(type, varType)
      }

    }
  }

}

function updateColorMap(data) {
  colorMapType.value = data[0]
  colorMap.value = data[1]
}
</script>

<style>

.app-container {
  height: 90vh;
  position: relative;
}
</style>