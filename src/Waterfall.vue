// 这里的类型推断有bug
<script setup lang="ts" generic="T">
import {
  computed,
  nextTick,
  onMounted,
  onUnmounted,
  ref,
  watch,
  type Ref,
} from 'vue'

interface Props {
  datas: T[]
  col?: number
  gap?: number
  distance?: number
  breaker?: (width: number) => number
}

const props = withDefaults(defineProps<Props>(), {
  datas: () => [],
  col: 2,
  gap: 6,
  distance: 200,
  breaker: (width: number) => {
    if (width < 320) {
      return 2
    } else if (width < 512) {
      return 3
    } else if (width < 640) {
      return 3
    } else if (width < 768) {
      return 4
    } else {
      return 5
    }
  },
})
const emits = defineEmits(['bottomOut'])

let col = props.col
const list = ref([]) as Ref<{ data: T; coords: { x: number; y: number } }[]>
const containerRef = ref<HTMLDivElement>()
const containerRect = ref()
const colHeights = ref<number[]>([])
const itemWidth = computed(() =>
  containerRect.value
    ? (containerRect.value.width - (col - 1) * props.gap) / col
    : 0
)
const containerHeight = computed(
  () => Math.max(Math.max(...colHeights.value) - props.gap, 0) + 'px'
)
const minY = computed(() => Math.min(...colHeights.value))
const nextRenderColIndex = computed(() => {
  return colHeights.value.findIndex(item => item === minY.value)
})

const datas = ref([]) as Ref<T[]>

async function render() {
  updatecontainerRect()
  updateCol()
  renderPreparation()
  await nextTick()
  datas.value.length && renderer(datas.value[0])
}
const rerender = render
watch(
  () => props.datas,
  () => rerender(),
  { deep: true }
)

function updatecontainerRect() {
  containerRect.value = containerRef.value?.getBoundingClientRect()
}
function updateCol() {
  const width = containerRect.value?.width ?? 0
  col = props.breaker(width) || 2
}

function renderPreparation() {
  list.value = []
  colHeights.value = new Array(col).fill(0)
  datas.value = props.datas
}

function renderer(item: T) {
  const coords = calcCoordsOfRender()
  list.value.push({
    data: item,
    coords: coords,
  })
}

function calcCoordsOfRender() {
  return {
    x: nextRenderColIndex.value * (itemWidth.value + props.gap) || 0,
    y: minY.value || 0,
  }
}

const observer = new MutationObserver(([mutation]) => {
  if (mutation.addedNodes.length) {
    const newNode = mutation.addedNodes[0] as HTMLElement
    const rect = newNode.getBoundingClientRect()
    const index = parseInt(newNode.dataset.index!)
    if (isNaN(index)) return
    const currentRenderColIndex = nextRenderColIndex.value // 此时nextRenderColIndex还未更新
    colHeights.value[currentRenderColIndex] += rect.height + props.gap
    const nextIndex = index + 1
    nextIndex < datas.value.length && renderer(datas.value[nextIndex])
  }
})

function startObserver() {
  if (!containerRef.value) return
  observer.observe(containerRef.value, {
    childList: true, // 观察目标子节点的变化，是否有添加或者删除
  })
}

// TODO 节流
function bottomOutHandler() {
  const element = document.documentElement
  const isBottomOut =
    Math.abs(element.scrollHeight - element.clientHeight - element.scrollTop) <
    props.distance
  isBottomOut && emits('bottomOut')
}

function dispose() {
  releaseEvent()
}

function releaseEvent() {
  window.removeEventListener('resize', render)
  window.removeEventListener('scroll', bottomOutHandler, true)
}

function listenerEvent() {
  window.addEventListener('scroll', bottomOutHandler, true)
  window.addEventListener('resize', render)
}

onMounted(() => {
  startObserver()
  listenerEvent()
  render()
})

onUnmounted(() => dispose())
</script>

<template>
  <div ref="containerRef" class="Waterfall">
    <section
      v-for="(item, index) in list"
      :key="(item.data as any)?.id ?? item.data"
      :style="{
        width: itemWidth + 'px',
        transform: `translate(${item.coords.x}px, ${item.coords.y}px)`,
      }"
      :data-index="index"
      class="item"
    >
      <slot v-bind="item" :index="index" />
    </section>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
  font-size: 16px;
  line-height: 1.4;
  color: #101010;
  padding: 0;
  margin: 0;
}

.Waterfall {
  position: relative;
  top: 0;
  left: 0;
  width: 100%;
  max-width: 1024px;
  height: v-bind(containerHeight);
  background-color: pink;
  overflow: visible;
}

.item {
  position: absolute;
  top: 0;
  left: 0;
  border-radius: 4px;
  overflow: hidden;
  background-color: white;
}

.footer {
  text-align: center;
}
</style>
