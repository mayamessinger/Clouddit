<template>
  <div class="param stopList">
    This word cloud uses a stop list.<br />
    To re-include a word, please check it
    <div class="scrollable">
      <div v-for="word in oldStop">
        <input type="checkbox" value="word" :checked="true" v-on:change="$emit('addBackWord', word)" />{{word}}
      </div>
      <div v-for="word in stopList">
        <input type="checkbox" value="word" :checked="false" v-on:change="$emit('addBackWord', word)" />{{word}}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "StopList",
  props:  [
    "stopList",
    "oldStop"
  ]
}
</script>

<style lang="scss">
.stopList {
  height: 30%;
  max-height: 250px;
  margin-top: 10%;
}

.scrollable {
  max-height: 100%;
  overflow-y: scroll;
}
</style>