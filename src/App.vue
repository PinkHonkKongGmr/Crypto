<template>
  <div id="app">
    <cointable></cointable>
  </div>
</template>

<script>
import Cointable from "@/components/table.vue";
export default {
  components: {
    cointable: Cointable
  }
};
</script>


<style lang="scss">
* {
  padding: 0;
  margin: 0;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
