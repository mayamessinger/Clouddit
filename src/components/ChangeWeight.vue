<template>
  <div>
    <label class="param">Weigh words by:</label>
    <button v-for="option in weightOptions" value="option" v-on:click="$emit('weigh', option)" >{{option}}</button>
</div>
</template>

<script>
export default {
  name: "ChangeVisualize",
  props:  [
    "weightOptions"
  ]
}
</script>

<style lang="scss">

</style>