<script setup>
import MovieSearch from './components/MovieSearch.vue'
</script>

<template>
  <main>
    <MovieSearch />
  </main>
</template>

<style scoped>
main {
  min-height: 100vh;
  min-width: 100vw;
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f5f5f5;
  overflow: hidden;
  box-sizing: border-box;
}
</style>
<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  width: 100vw;
  overflow-x: hidden;
}
</style>
