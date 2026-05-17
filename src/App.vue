<template>
  <div id="app">
    <RouterView />
  </div>
</template>

<script setup>
import { RouterView } from 'vue-router'
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Georgia', serif;
  background-color: #0a0a0a;
  color: #c9a84c;
  min-height: 100vh;
}

#app {
  min-height: 100vh;
}
</style>
