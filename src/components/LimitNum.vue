<template>
	<span>
		<input id="limitNum" type="text" size="1" placeholder="50" v-bind:value="limit" v-on:input="$emit('input', $event.target.value)" />
		<label for="limitNum">(#)</label>
	</span>
</template>

<script>
export default {
  name: 'LimitNum',
  props:  [
    "limit"
  ]
}
</script>

<style lang="scss">

</style>