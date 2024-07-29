<template>
  <div id="app">
    <ChatPopup />
  </div>
</template>

<script>
import ChatPopup from './components/ChatPopup.vue'

export default {
  name: 'App',
  components: {
    ChatPopup,
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
