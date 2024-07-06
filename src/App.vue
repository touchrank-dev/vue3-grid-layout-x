<script setup lang="ts">
import {computed, ref} from "vue"
import BasicDemo from "./example/BasicDemo.vue"
import MultiGrids from "./example/MultiGrids.vue"

const demos = ref({
  basicDemo: BasicDemo,
  multiGrids: MultiGrids
})

const key = ref<string>("Multi Grids")

const switchComponent = (which: string) => {
  key.value = which
}

const component = computed(() => {
  switch (key.value) {
    case "Basic Grid":
      return demos.value.basicDemo
    case "Multi Grids":
      return demos.value.multiGrids
  }
  return undefined
})
</script>

<template>
  <div class="container">
    <div class="toolbar">
      <strong>{{ key }}</strong> /
      <button @click="switchComponent('Basic Grid')">Basic Grid</button>
      <button @click="switchComponent('Multi Grids')">Multi Grids</button>
    </div>
    <div class="components">
      <component :is="component" />
    </div>
  </div>
</template>
<style lang="css">
body {
  margin: 0px;
  padding: 0px;
  overflow-x: hidden;
  overflow-y: scroll;
  padding: 10px;
}
.container {
  width: 100%;
  font-size: 14px;
}
.components {
  padding: 10px;
  background-color: #eee;
}
.toolbar {
  background-color: #ddd;
  padding: 10px;
  border-bottom: 1px solid #000;
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 10px;
  font-size: 16px;
}
</style>
