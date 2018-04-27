<template>
	<span>
		<input id="subreddit" type="text" placeholder="subreddit (no /r/prefix)" v-bind:value="subreddit" v-on:input="$emit('input', $event.target.value)" />
	</span>
</template>

<script>
export default {
  name: "Subreddit",
  props:  [
    "subreddit"
  ]
}
</script>

<style lang="scss">

</style>