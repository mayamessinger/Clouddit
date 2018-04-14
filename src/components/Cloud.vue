<!-- https://vuejs.org/v2/guide/components.html -->
<!-- https://vuejs.org/v2/guide/components.html -->

<template>
	<form>
	  <!-- <input id="subreddit" type="text" placeholder="subreddit (no /r/prefix)" v-model:="subreddit" /> -->
	  <!-- <dropdown id="sort" :options="sorts" :selected="" v-on:updateOption=""></dropdown> -->
	  <!-- <label for="limit">Number of posts:</label> -->
	  <!-- <input id="limit" type="range" min="1" max="500" step="1" v-model:="limit" /> -->
	  <!-- <input id="limitVal" type="text" size="3" placeholder="300" value="" v-model:="limit" /> -->
	  <!-- <dropdown id="time" :options="times" :selected="" v-on:updateOption=""></dropdown> -->
	  <!-- <input id="submit" type="button" onclick="$emit('visualize')" value="Visualize"> -->
	</form>
</template>

<script>
import dropdown from "vue-dropdowns";

export default {
  name: 'MyCloud',
  data () {
    return {
    	sorts: ["best", "hot", "new", "controversial", "top", "rising"],
    	times: ["hour", "day", "week", "month", "year", "all"]
    }
  },
  props:  [
    "subreddit",
    "sort",
    "limit",
    "time"
  ]
}
</script>

<style lang="scss">

</style>