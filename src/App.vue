<script setup lang="ts">
import {ref, toRaw} from "vue"
import {testData2} from "./test"

import GridLayout from "./components/Grid/GridLayout.vue"
import GridItem from "./components/Grid/GridItem.vue"

const randomText = () => {
  const len = sampleText.length
  const start = Math.floor(Math.random() * 10)
  const end = Math.floor(Math.random() * len)
  return sampleText.substring(start, Math.max(start + 10, end))
}

testData2.forEach(it => (it.body = randomText()))
let testLayout = ref(testData2)

const refLayout = ref()
const sampleText =
  "中国科学院新疆生态与地理研究所的科学家发现了一种能在火星环境下生存的沙漠苔藓。极端环境生物是生命的奇迹，代表着生命对环境的极限适应能力。为了适应极端环境（干燥、寒冷、高温、辐射等），它们演化出了令人惊叹的特殊“本领”，是极其珍贵的战略生物资源。极端生物及其适应性的研究对于探索生命奥秘、揭示地球生命起源与演化、实现地外星球拓殖等均有重要意义。"

const mapCache: Map<string, any> = new Map()

const responsive = ref<boolean>(true)

function set$Children(vm: any) {
  if (vm && vm.i) {
    mapCache.set(vm.i, vm)
  }
}

const heightUpdated = (i: string | number, h: number, item: any) => {
  item.h = h
}

const exportLayout = () => {
  console.log(toRaw(testLayout.value))
}


</script>

<template>
  <div class="layout">
    <div id="content">
      <GridLayout
        ref="refLayout"
        v-model:layout="testLayout"
        :responsive="responsive"
        :col-num="12"
        :row-height="10"
        :auto-size="true"
        :vertical-compact="true"
        :use-css-transforms="true"
      >
        <grid-item
          v-for="(item, idx) in testLayout"
          :ref="el => set$Children(el)"
          :key="item.i"
          class="test"
          :x="item.x"
          :y="item.y"
          :w="item.w"
          :h="item.h"
          :i="item.i"
          :sort-index="idx"
          :min-w="1"
          @height-updated="(i, h) => heightUpdated(i, h, item)"
        >
          {{ item.sortIndex }}
          <p>{{ item.body }}</p>
        </grid-item>
      </GridLayout>
    </div>
  </div>
  <hr size="1" />
  <button @click="exportLayout">Export</button>
</template>

<style scoped>
.layout {
  background-color: #eee;
  touch-action: none;
}
.vue-grid-item {
  border-radius: 8px;
  padding: 4px;
  margin: 0px;
}
.test {
  background-color: #ddd;
  font-size: 14px;
  line-height: 150%;
}
.droppable-element {
  width: 150px;
  text-align: center;
  background: #fdd;
  border: 1px solid black;
  margin: 10px 0;
  padding: 10px;
}
</style>
