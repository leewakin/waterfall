<script setup lang="ts" generic="T">
import {
  computed,
  nextTick,
  onMounted,
  onUnmounted,
  ref,
  watch,
} from 'vue'

interface WaterfallProps {
  items: T[]
  columns?: number
  gap?: number
  threshold?: number
  columnCalculator?: (width: number) => number
}

const props = withDefaults(defineProps<WaterfallProps>(), {
  items: () => [],
  columns: 2,
  gap: 6,
  threshold: 200,
  columnCalculator: (width: number) => {
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

const emits = defineEmits(['reachBottom'])

let columns = props.columns
const waterfallItems = ref([]) as Ref<{ item: T; position: { x: number; y: number } }[]>
const containerRef = ref<HTMLDivElement>()
const containerRect = ref()
const columnHeights = ref<number[]>([])
const itemWidth = computed(() =>
  containerRect.value
    ? (containerRect.value.width - (columns - 1) * props.gap) / columns
    : 0
)
const containerHeight = computed(
  () => Math.max(Math.max(...columnHeights.value) - props.gap, 0) + 'px'
)
const minHeight = computed(() => Math.min(...columnHeights.value))
const nextColumnIndex = computed(() => {
  return columnHeights.value.findIndex(height => height === minHeight.value)
})

const items = ref([]) as Ref<T[]>

async function renderWaterfall() {
  updateContainerRect()
  updateColumns()
  prepareRender()
  await nextTick()
  items.value.length && renderItem(items.value[0])
}

const rerender = renderWaterfall
watch(
  () => props.items,
  () => rerender(),
  { deep: true }
)

function updateContainerRect() {
  containerRect.value = containerRef.value?.getBoundingClientRect()
}

function updateColumns() {
  const width = containerRect.value?.width ?? 0
  columns = props.columnCalculator(width) || 2
}

function prepareRender() {
  waterfallItems.value = []
  columnHeights.value = new Array(columns).fill(0)
  items.value = props.items
}

function renderItem(item: T) {
  const position = calculateRenderPosition()
  waterfallItems.value.push({
    item: item,
    position: position,
  })
}

function calculateRenderPosition() {
  return {
    x: nextColumnIndex.value * (itemWidth.value + props.gap) || 0,
    y: minHeight.value || 0,
  }
}

const observer = new MutationObserver(([mutation]) => {
  if (mutation.addedNodes.length) {
    const newNode = mutation.addedNodes[0] as HTMLElement
    const rect = newNode.getBoundingClientRect()
    const index = parseInt(newNode.dataset.index!)
    if (isNaN(index)) return
    const currentColumnIndex = nextColumnIndex.value
    columnHeights.value[currentColumnIndex] += rect.height + props.gap
    const nextIndex = index + 1
    nextIndex < items.value.length && renderItem(items.value[nextIndex])
  }
})

function startObserver() {
  if (!containerRef.value) return
  observer.observe(containerRef.value, {
    childList: true,
  })
}

function handleReachBottom() {
  const element = document.documentElement
  const isBottom =
    Math.abs(element.scrollHeight - element.clientHeight - element.scrollTop) <
    props.threshold
  isBottom && emits('reachBottom')
}

function cleanup() {
  removeEventListeners()
}

function removeEventListeners() {
  window.removeEventListener('resize', renderWaterfall)
  window.removeEventListener('scroll', handleReachBottom, true)
}

function addEventListeners() {
  window.addEventListener('scroll', handleReachBottom, true)
  window.addEventListener('resize', renderWaterfall)
}

onMounted(() => {
  startObserver()
  addEventListeners()
  renderWaterfall()
})

onUnmounted(() => cleanup())
</script>

<template>
  <div ref="containerRef" class="Waterfall">
    <section
      v-for="(waterfallItem, index) in waterfallItems"
      :key="(waterfallItem.item as any)?.id ?? waterfallItem.item"
      :style="{
        width: itemWidth + 'px',
        transform: `translate(${waterfallItem.position.x}px, ${waterfallItem.position.y}px)`,
      }"
      :data-index="index"
      class="item"
    >
      <slot v-bind="waterfallItem" :index="index" />
    </section>
  </div>
</template>

<style scoped>
.Waterfall {
  position: relative;
  width: 100%;
  max-width: 1024px;
  height: v-bind(containerHeight);
  background-color: pink;
  overflow: visible;
}

.item {
  position: absolute;
  border-radius: 4px;
  background-color: white;
}
</style>
