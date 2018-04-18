<template>
  <div class="param stopList">
    This word cloud uses a stop list to filter common words.<br />
    To re-include a word, please check it below.
    <div>
      <input type="text" placeholder="Add word" v-model="newStop" />
      <button v-on:click="$emit('addBackWord', newStop)">Add</button>
    </div>
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
  data() {
    return {
      newStop: null
    }
  },
  props:  [
    "stopList",
    "oldStop"
  ]
}
</script>

<style lang="scss">
.stopList {
  height: inherit;
  margin-top: 10%;
  max-height: 15em;
}

.scrollable {
  height: inherit;
  margin-top: 1%;
  overflow-y: scroll;
}
</style>