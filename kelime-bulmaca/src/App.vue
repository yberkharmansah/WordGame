<template>
  <div id="app">
    <KelimeBulmaca />
  </div>
</template>

<script>
import KelimeBulmaca from './components/KelimeBulmaca.vue';

export default {
  name: 'App',
  components: {
    KelimeBulmaca
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
