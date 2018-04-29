<template>
  <div class="param stopList">
    Clouddit filters against too-common words and excludes them from the word cloud.<br />
    Below are those filtered words. Check a word to re-include it.
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
  margin-top: 5%;
  max-height: 15em;
}

.scrollable {
  height: inherit;
  margin-top: 1%;
  overflow-y: scroll;
}
</style>