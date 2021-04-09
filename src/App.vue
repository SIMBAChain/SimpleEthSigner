<template>
  <img class="logo" salt="Simbachains logo" src="./assets/logo.png">
  <Signer/>
</template>

<script>
import Signer from './components/Signer.vue'

export default {
  name: 'App',
  components: {
    Signer
  }
}
</script>

<style lang="scss">
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;

    .logo {
      width: 128px;
      height: 128px;
    }
  }
</style>
