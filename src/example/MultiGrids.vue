<script setup lang="ts">
import {nextTick, ref, toRaw} from "vue"

import GridLayout from "../components/Grid/GridLayout.vue"
import GridItem from "../components/Grid/GridItem.vue"

export interface DraggingItem {
  x: number
  y: number
  w: number
  h: number
  i: string
  sortIndex: number
  fromGrid: number
  dropGrid: number
}
// 对象与Grid的映射关系
export interface MapToGrid {
  obj: any
  gridIndex: number
}

const gridWrappersRef = Array<MapToGrid>()
const gridsRef = Array<MapToGrid>()
const gridItemsRef = new Map<string | number, MapToGrid>()

// 记录拖动的项，重点记录fromGrid和dropGrid
let draggingItem: DraggingItem | null

// 动态设置ref
const setWrapperRef = function (el: any, gridIndex: number) {
  if (el) gridWrappersRef.push({obj: el, gridIndex})
}

const setGridRef = function (el: any, gridIndex: number) {
  if (el) gridsRef.push({obj: el, gridIndex})
}

const setItemsRef = function (el: any, gridIndex: number) {
  if (el && el.i && !gridItemsRef.has(el.i)) {
    gridItemsRef.set(el.i, {obj: el, gridIndex})
  }
}

const layoutData = ref([
  [
    {x: 0, y: 0, w: 2, h: 2, i: "0", sortIndex: 0},
    {x: 2, y: 0, w: 2, h: 3, i: "1", sortIndex: 0},
    {x: 0, y: 2, w: 2, h: 4, i: "2", sortIndex: 0},
    {x: 4, y: 2, w: 2, h: 2, i: "4", sortIndex: 0}
  ],
  [
    {x: 0, y: 0, w: 2, h: 2, i: "5", sortIndex: 0},
    {x: 2, y: 0, w: 2, h: 2, i: "6", sortIndex: 0},
    {x: 4, y: 0, w: 2, h: 3, i: "7", sortIndex: 0}
  ]
])

const dropId = "drop"

const moveEvent = (
  itemId: string | number,
  newX: number,
  newY: number,
  clientX: number,
  clientY: number
) => {
  const itemRef = gridItemsRef.get(itemId)
  if (!itemRef) {
    removableLog("未找到拖放的元素", itemId)
    return
  }
  removableLog(`BREAK! Got a moveEvent for ${itemId}`)
  const fromGrid = itemRef.gridIndex
  const itemData = layoutData.value[fromGrid].find(ele => ele.i === itemId)
  if (!itemData) {
    removableLog("未找到布局数据")
    return
  }

  if (!draggingItem) {
    draggingItem = {...itemData, fromGrid, dropGrid: -1}
  }

  const gridWrapperRef = gridWrappersRef.find(ele => ele.gridIndex === fromGrid)
  if (!gridWrapperRef) {
    removableLog("未找到网格的容器")
    return
  }

  const sourceWrapper = gridWrapperRef.obj
  const sourceWrapperRect = sourceWrapper.getBoundingClientRect()

  // 是否还在原来的容器中
  const mouseInOriGrid = mouseInRect(clientX, clientY, sourceWrapperRect)
  if (mouseInOriGrid) {
    removableLog("鼠标在同一网格，无须处理")
    return
  }

  // 移出视野
  const gridLayout = gridsRef[fromGrid].obj
  // TODO:仍可能影响源网格布局
  gridLayout.dragEvent("dragend", itemId, -100, -100, 0, 0)

  const targetWrapperRef = findTargetWrapper(clientX, clientY, fromGrid)
  // 是否进入其他的Grid
  if (!targetWrapperRef) {
    removableLog("没有进入其他网格")
    draggingItem.dropGrid = -1
    return
  }
  // 拖放时可能经过了中间网格，需要检查
  if (draggingItem.dropGrid && draggingItem.dropGrid !== targetWrapperRef.gridIndex) {
    // 移除中间网格的drop项
    cleanContext(false)
  }
  draggingItem.dropGrid = targetWrapperRef.gridIndex
  const dropGridIndex = draggingItem.dropGrid

  const targetWrapper = targetWrapperRef.obj
  const targetParentRect = targetWrapper.getBoundingClientRect()
  const layoutArray = layoutData.value[dropGridIndex]
  const dropGridLayout = gridsRef[dropGridIndex].obj

  // 如有需要，先在目标网格增加一个drop方格
  if (!layoutArray.find(item => item.i === dropId)) {
    const temp = {
      x: (layoutArray.length * 2) % 12,
      y: layoutArray.length + 12,
      w: draggingItem.w,
      h: draggingItem.h,
      i: dropId,
      sortIndex: 0
    }
    layoutArray.push(temp)
  }

  const dropItemRef = gridItemsRef.get(dropId)
  if (dropItemRef) {
    removableLog("Got Drop item ref.")
    const dropItem = dropItemRef.obj
    try {
      dropItem.styleObj.display = "none"
    } catch (e) {
      /* empty */
    }

    const newPos = dropItem.calcXY(clientY - targetParentRect.top, clientX - targetParentRect.left)
    dropGridLayout.dragEvent(
      "dragstart",
      dropId,
      newPos.x,
      newPos.y,
      draggingItem.h,
      draggingItem.w
    )
    const mouseInTargetGrid = mouseInRect(clientX, clientY, targetParentRect)
    if (!mouseInTargetGrid) {
      removableLog("mouse is not in target grid")
      dropGridLayout.dragEvent(
        "dragend",
        dropId,
        newPos.x,
        newPos.y,
        draggingItem.h,
        draggingItem.w
      )
      layoutData.value[dropGridIndex] = layoutArray.filter(item => item.i !== dropId)
    }
    draggingItem.x = newPos.x
    draggingItem.y = newPos.y
    removableLog("BREAK! continue dragging, hoping to emit movedEvent.")
  } else {
    removableLog("BREAK! Could not found the Drop item, do it next turn")
  }
}

const movedEvent = (
  itemId: string | number,
  newX: number,
  newY: number,
  clientX: number,
  clientY: number
) => {
  removableLog(`BREAK! Got a movedEvent for ${itemId}`)
  nextTick(() => {
    cleanContext(true)
  })

  if (!draggingItem) return

  const itemRef = gridItemsRef.get(itemId)
  if (!itemRef) {
    removableLog(`BREAK! cannot found a gridItemRef #${itemId}`)
    return
  }

  let item = itemRef.obj
  const sourceWrapperIndex = draggingItem.fromGrid
  const sourceWrapper = gridWrappersRef[sourceWrapperIndex].obj
  const sourceWrapperRect = sourceWrapper.getBoundingClientRect()

  // 是否还在原来的容器中
  const mouseInOriGrid = mouseInRect(clientX, clientY, sourceWrapperRect)
  if (mouseInOriGrid) {
    removableLog(`BREAK! Item is moving in its original grid.`)
    return
  }

  if (draggingItem.dropGrid == -1 || draggingItem.dropGrid == sourceWrapperIndex) {
    removableLog(`BREAK! Same as previous or cannot find new target grid`)
    cleanContext(true)
    return
  }

  const targetWrapperIndex = draggingItem.dropGrid
  const targetWrapper = gridWrappersRef[targetWrapperIndex].obj
  const targetParentRect = targetWrapper.getBoundingClientRect()

  // 是否移入了新容器
  const mouseInTargetGrid = mouseInRect(clientX, clientY, targetParentRect)

  if (!mouseInOriGrid && !mouseInTargetGrid) {
    removableLog(`BREAK! Where is the item #${itemId}?`)
    cleanContext(true)
    return
  }

  // 已经移出了源网格
  if (!mouseInOriGrid) {
    removableLog("已经移出了源网格,清理", `id=${itemId}`)
    // cleanDropItem()
    const draggingIndex = layoutData.value[sourceWrapperIndex].findIndex(ele => ele.i === itemId)
    layoutData.value[sourceWrapperIndex].splice(draggingIndex, 1)
  }

  // 已经移入了新网格
  if (mouseInTargetGrid) {
    removableLog("已经移入了新网格,dragend dropId")
    const layoutArray = layoutData.value[targetWrapperIndex]
    const gridLayout = gridsRef[targetWrapperIndex].obj
    removableLog("try to emit dragent for id=dropId item")
    gridLayout.dragEvent("dragend", dropId, -10, -10, draggingItem.h, draggingItem.w)
    // 清理
    layoutData.value[targetWrapperIndex] = layoutArray.filter(item => item.i !== dropId)

    // 插入移入项
    layoutData.value[targetWrapperIndex].push({
      x: draggingItem.x,
      y: draggingItem.y,
      w: draggingItem.w,
      h: draggingItem.h,
      i: draggingItem.i,
      sortIndex: 1
    })

    gridLayout.dragEvent(
      "dragend",
      draggingItem.i,
      draggingItem.x,
      draggingItem.y,
      draggingItem.h,
      draggingItem.w
    )

    const itemRef = gridItemsRef.get(draggingItem.i)
    if (itemRef) {
      itemRef.gridIndex = draggingItem.dropGrid
      item = itemRef.obj
      item.styleObj.display = ""
    }
  }
  draggingItem = null
}

/**
 * 拖放结束回调
 * @deprecated Don't need this, movedEvent() will call cleanContext() in nextTick
 */
const postDrag = () => {
  // cleanContext(true)
}

/**
 * 清理遗留的中间数据
 */
const cleanContext = (removeDraggingItem: boolean) => {
  if (removeDraggingItem && draggingItem) draggingItem = null
  gridItemsRef.delete(dropId)
  const len = layoutData.value.length
  for (let i = 0; i < len; i++) {
    const array = layoutData.value[i]
    layoutData.value[i] = array.filter(item => item.i !== dropId)
    const grid = gridsRef[i].obj
    grid.dragEvent("dragend", dropId, -100, -100, 0, 0)
  }
}

/**
 * TODO:这里有问题！！！！
 * 遍历所有的Grid，返回鼠标已经移入的那个
 */
const findTargetWrapper = (clientX: number, clientY: number, skipIndex: number) => {
  const wrapperRef = gridWrappersRef.find(el => {
    if (el.gridIndex !== skipIndex) {
      const wrapper = el.obj
      removableLog("Found a wrapper", el.obj, clientX, clientY, wrapper.getBoundingClientRect())
      if (mouseInRect(clientX, clientY, wrapper.getBoundingClientRect())) {
        removableLog("鼠标已经移入另一网格")
        return el
      }
    }
    return null
  })
  return wrapperRef
}
/**
 * 给定坐标是否在指定区域内
 */
const mouseInRect = (clientX: number, clientY: number, rect: DOMRect) => {
  if (clientX > rect.left && clientX < rect.right && clientY > rect.top && clientY < rect.bottom) {
    return true
  }
  return false
}

// 临时日志方法
const removableLog = (...args: any) => {
  console.log(...args)
}

const heightUpdated = (i: string | number, h: number, item: any) => {
  item.h = h
}
</script>

<template>
  <div class="layout">
    <div v-for="gridIndex in 2" :key="gridIndex" class="col50 bg-1">
      <div :ref="el => setWrapperRef(el, gridIndex - 1)">
        <grid-layout
          :ref="(el: any) => setGridRef(el, gridIndex - 1)"
          style="width: 100%; border: 1px solid red"
          :layout="layoutData[gridIndex - 1]"
          :col-num="12"
          :row-height="30"
          :is-draggable="true"
          :is-resizable="true"
          :vertical-compact="true"
          :use-css-transforms="true"
        >
          <grid-item
            v-for="item in layoutData[gridIndex - 1]"
            :ref="(el: any) => setItemsRef(el, gridIndex - 1)"
            :key="item.i"
            :x="item.x"
            :y="item.y"
            :w="item.w"
            :h="item.h"
            :i="item.i"
            :sort-index="item.sortIndex"
            class="test"
            @move="moveEvent"
            @moved="movedEvent"
            @post-drag="postDrag"
            @height-updated="(i, h) => heightUpdated(i, h, item)"
          >
            i:{{ item.i }}, sortIndex: {{ item.sortIndex }}
            <br />
            <br />
            <br />
          </grid-item>
        </grid-layout>
      </div>
    </div>
  </div>
  <p>
    You can drag an item from any grid and drop it into another. The current implementation is not
    perfect, and sometimes drag-and-drop may fail.
  </p>
</template>
<style scoped>
@import url(./index.css);
.layout {
  display: flex;
  flex-direction: row;
}
.col50 {
  width: 50%;
  border: 1px solid #000;
  /* min-height: 90vh; */
}
.bg-1 {
  background-color: rgb(114, 238, 114, 0.2);
}
.bg-2 {
  background-color: rgba(255, 255, 100, 0.4);
}
</style>
