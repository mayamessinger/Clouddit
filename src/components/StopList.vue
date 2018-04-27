<template>
  <div class="param stopList">
    This word cloud uses a stop list to filter common words.<br />
    To re-include a word, please check it below.
    <div>
      <label for="addWord">Add word to filter</label>
      <input type="text" id="addWord" v-model="newStop" />
      <button v-on:click="$emit('addBackWord', newStop)">Add</button>
    </div>
    <div class="scrollable">
      <div v-for="word in oldStop">
        <input v-bind:id="word" type="checkbox" value="word" :checked="true" v-on:change="$emit('addBackWord', word)" />
        <label v-bind:for="word">{{word}}</label>
      </div>
      <div v-for="word in stopList">
        <input v-bind:id="word" type="checkbox" value="word" :checked="false" v-on:change="$emit('addBackWord', word)" />
        <label v-bind:for="word">{{word}}</label>
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