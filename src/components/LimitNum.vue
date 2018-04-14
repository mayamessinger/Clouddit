<template>
	<input id="limitNum" type="text" size="1" placeholder="250" v-bind:value="limit" v-on:input="$emit('input', $event.target.value)" />
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