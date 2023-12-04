<template>
  <main>
    <h2>네트워크 탭을 열어보세요</h2>
    <button @click="reqJsonPlaceholder">100개 병렬 요청</button>
    <button @click="reqJsonPlaceholderDiv">6개 단위 병렬 요청</button>
    <button @click="reqJsonPlaceholderOneSecond">6개 단위 1초 지연 병렬 요청</button>
  </main>
</template>

<script setup lang="ts">
import { ref, onBeforeUnmount } from 'vue'

const isVisible = ref(true)

async function reqJsonPlaceholder() {
  const promises = Array.from({ length: 100 }, (_, idx) =>
    fetch(`https://jsonplaceholder.typicode.com/posts/${idx + 1}`).then((response) =>
      response.json()
    )
  )

  const res = await Promise.all(promises)
  console.log(res)
}

async function reqJsonPlaceholderDiv() {
  const windowSize = 6
  for (let i = 0; i < Math.ceil(100 / windowSize); i++) {
    const res = await Promise.all([
      fetch(`https://jsonplaceholder.typicode.com/posts/${i + 1}`),
      fetch(`https://jsonplaceholder.typicode.com/posts/${i + 2}`),
      fetch(`https://jsonplaceholder.typicode.com/posts/${i + 3}`),
      fetch(`https://jsonplaceholder.typicode.com/posts/${i + 4}`),
      fetch(`https://jsonplaceholder.typicode.com/posts/${i + 5}`),
      fetch(`https://jsonplaceholder.typicode.com/posts/${i + 6}`)
    ])
    console.log(res.map(async (elem) => await elem.json()))
  }
}

async function reqJsonPlaceholderOneSecond() {
  const windowSize = 6
  for (let i = 0; i < Math.ceil(100 / windowSize); i++) {
    if (!isVisible.value) break

    const Promises = []
    for (let j = 0; j < windowSize; j++) {
      Promises.push(
        new Promise<Response>((res) => {
          const result = fetch(`https://jsonplaceholder.typicode.com/posts/${j + 1 + i * 6}`)
          setTimeout(() => {
            res(result)
          }, 1000)
        })
      )
    }
    const res = await Promise.all(Promises)
    console.log(res.map(async (elem) => await elem.json()))
  }
}

// async function reqJsonPlaceholderOneSecondSerialize() {
//   for (let i = 0; i < 100; i++) {
//     setTimeout(() => {}, 100)
//     await fetch(`https://jsonplaceholder.typicode.com/posts/${i + 1}`).then((res) => res.json())
//   }
// }

onBeforeUnmount(() => {
  isVisible.value = false
})
</script>
