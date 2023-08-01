<script setup>
import '@picocss/pico'
import { LRUCache } from 'lru-cache'
import { onBeforeMount, ref } from 'vue'
import { hash as ohash } from 'ohash'

const state = ref({})
const posts = ref([])

const promiseCache = new LRUCache({
  max: 500,
  ttl: 1000 * 60, // 1 minute
})

// onBeforeMount(() => {
//   fetchPosts('https://jsonplaceholder.typicode.com/posts').then(
//     data => (posts.value = data)
//   )
// })

function fetchPosts(url, params) {
  console.log('fetchPosts init...')
  const hash = ohash([url, params])

  if (!promiseCache.has(hash)) {
    console.log('fetchPosts promiseCache...')
    promiseCache.set(
      hash,
      fetch(url, params)
        .then(res => {
          console.log('fetchPosts res.json()...')
          return res.json()
        })
        .then(data => {
          state.value[hash] = data
          posts.value = data
          return data
        })
        .catch(e => {
          promiseCache.delete(hash)
          throw e
        })
    )
  }
  return promiseCache.get(hash)
}
</script>

<template>
  <button @click="fetchPosts('https://jsonplaceholder.typicode.com/posts')">
    Fetch posts
  </button>
  <article v-for="post in posts" :key="post.id">
    <header>{{ post.title }}</header>
    <p>{{ post.body }}</p>
  </article>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
