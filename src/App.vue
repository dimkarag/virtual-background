<template>
  <virtual-background
    :content-src="require('@/assets/reporter-green-screen.mp4')"
    :background-image-src="require('@/assets/nature.jpg')"
  />
</template>

<script>

import VirtualBackground from '@/components/VirtualBackground'
export default {
  components: { VirtualBackground },
  name: 'App'
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
