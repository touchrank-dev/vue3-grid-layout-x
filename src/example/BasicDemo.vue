<script setup lang="ts">
import {ref, toRaw} from "vue"
import {testData2} from "../test"

import GridLayout from "../components/Grid/GridLayout.vue"
import GridItem from "../components/Grid/GridItem.vue"

const randomText = (w: number) => {
  const len = sampleText.length
  const start = Math.floor(Math.random() * 10)
  const end = Math.floor(Math.random() * w * 200)
  return sampleText.substring(start, Math.max(start + 10, end))
}

testData2.forEach(it => (it.body = randomText(it.w)))
let testLayout = ref(testData2)

const refLayout = ref()
const sampleText =
  "Curabitur ut sapien vel eros ultrices tristique. Aliquam placerat urna nec nisl laoreet facilisis. Vivamus vulputate purus nisl, eget maximus purus molestie eget. Quisque sollicitudin ac lacus ut fringilla. Donec a felis dictum, aliquet turpis in, pharetra enim. Phasellus luctus erat quis odio sollicitudin, pulvinar porta leo ullamcorper. Morbi tempus at enim a vehicula. Vestibulum pharetra sapien lectus, vitae luctus nulla accumsan eu. Phasellus placerat tempor pellentesque. Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos. Cras ac libero suscipit, condimentum orci in, varius eros. Donec congue aliquet mauris sed cursus. Praesent in consectetur nisl. Maecenas condimentum, diam vitae porta vehicula, justo orci bibendum tortor, vitae ultrices tellus neque eget nibh. Cras lacinia pulvinar arcu, at laoreet enim venenatis quis. Quisque ac massa ullamcorper, ultricies massa eu, consectetur elit. "

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

const nextId = ref(100)
/**
 * 演示插入一项
 */
const appendItem = () => {
  // 修改各项以保证新项插入到第一位
  testLayout.value.forEach(it => {
    it.sortIndex += 1
    if (it.x == 0 && it.y == 0) {
      it.y += 1
    }
  })
  //插入一项
  testLayout.value.push({
    sortIndex: 0,
    x: 0,
    y: 0,
    w: 2,
    h: 25,
    i: `${nextId.value++}`,
    body: "[APPEND] " + randomText()
  })
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
  <button @click="exportLayout">Export layout data</button>&nbsp;
  <button @click="appendItem">Append a new GridItem</button>
</template>
<style scoped>
@import url(./index.css);
</style>
