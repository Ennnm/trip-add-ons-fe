<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <AddOnForm />
</template>

<script>
import AddOnForm from './components/AddOnForm.vue'

export default {
  name: 'App',
  components: {
    AddOnForm
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
  margin-top: 60px;
}
</style>
