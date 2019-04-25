<template>
  <div >
    <v-touch v-on:swipeleft="swiperleft"  class="wrapper">
      <div>1</div>
    </v-touch>
    <div>2</div>
    <div>3</div>
  </div>
</template>

<script>
export default {
  methods: {
    swiperleft() {
      console.log("1");
    }
  }
};
</script>

<style lang="scss">
</style>
