<template>
	<input id="limit" type="range" min="1" max="500" step="1" v-bind:value="limit" v-on:input="$emit('input', $event.target.value)" />
</template>

<script>
export default {
  name: 'Limit',
  props:  [
    "limit"
  ]
}
</script>

<style lang="scss">

</style>