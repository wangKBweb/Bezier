<template>
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <demo />
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import demo from './components/demo.vue'

export default {
  name: 'App',
  components: {
    // HelloWorld,
    demo
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
}
</style>
