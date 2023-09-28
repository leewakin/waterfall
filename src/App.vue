<script setup lang="ts">
import { ref } from 'vue'
import Waterfall from './Waterfall.vue'

function test() {
  return [
    {
      title: '偶像剧女主怎么穿',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: '周愚昧',
    },
    {
      title: 'asdadfdghfghfdghasdadfdghfghfdgh',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: 'lk',
    },
    {
      title: '90斤宅男在酒屋里有什么不可告人的秘密',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: '李冬宝',
    },
    {
      title: 'bbb',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: 'aaa',
    },
    {
      title: '9asdadsaddsa不可告人的秘密asda',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: 'aaabbb',
    },
    {
      title: 'aaa',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: 'ccc',
    },
    {
      title: '电饭锅都是官方',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: '撒地方',
    },
  ]
}

const tests = ref(test())

setTimeout(() => {
  tests.value.pop()
}, 1000)
setTimeout(() => {
  tests.value.shift()
}, 2000)

setTimeout(() => {
  tests.value.push(
    {
      title: '偶像剧女主怎么穿111',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: '周愚昧333',
    },
    {
      title: '偶像剧女主怎么穿222',
      imgUrl: 'https://picsum.photos/120/150',
      nickname: '周愚昧555',
    }
  )
}, 3000)
</script>

<template>
  <div class="wrapper">
    <Waterfall :datas="tests">
      <template #default="{ data }">
        <div
          style="width: 100%; height: 150px; overflow: hidden"
          :style="{
            background: `url(${data.imgUrl}) no-repeat center 100%/100%`,
          }"
        ></div>

        <div style="padding: 0 6px">
          <div style="font-size: 14px; word-wrap: break-word">
            {{ data.title }}
          </div>
          <div
            style="
              display: flex;
              justify-content: space-between;
              align-items: center;
              font-size: 12px;
              padding: 6px 0;
              color: #909399;
            "
          >
            <div>{{ data.nickname }}</div>
            <div>321</div>
          </div>
        </div>
      </template>
    </Waterfall>
  </div>
</template>

<style>
.wrapper {
  height: 100vh;
  width: 100%;
  padding: 6px;
  display: flex;
  justify-content: center;
}
</style>
